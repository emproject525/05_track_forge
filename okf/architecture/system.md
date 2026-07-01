---
type: architecture
title: 모노레포 아키텍처
description: Track Forge 패키지들의 workspace, build, bundle, publish 구조
resource: .
tags: [overview, architecture, monorepo, build-system]
timestamp: 2026-07-01T00:00:00Z
---

# 개요

Track Forge는 `MediaStreamTrack`을 생성, 변환, 합성, 정제하는 기능별 패키지 모음이다.

이 문서는 패키지를 어떻게 만들고 빌드하고 배포할지에 대한 시스템 기준을 정의한다.

각 패키지는 하나의 기능 단위를 담당한다.

# 목표

- pnpm workspace로 여러 패키지를 한 repository에서 관리한다.
- 각 패키지는 독립적으로 빌드, 테스트, 배포될 수 있어야 한다.
- 각 패키지는 TDD 기반으로 개발되어야 한다.
- 패키지 빌드는 Vite library mode를 기본으로 한다.
- 번들링은 Vite 8과 Rolldown 기반 구성을 우선한다.
- 브라우저에서 사용하는 ESM 패키지를 기본 산출물로 삼는다.
- 타입 선언 파일을 함께 배포한다.
- 기능별 패키지가 서로 불필요한 의존성을 공유하지 않게 한다.

# 비목표

- Node.js 전용 미디어 처리 시스템을 만들지 않는다.
- 모든 패키지를 하나의 거대한 bundle로 합치지 않는다.
- 공통 계층을 미리 만들지 않는다.
- UI 프레임워크를 패키지 빌드 시스템의 기본 전제로 삼지 않는다.

# 패키지 경계

Track Forge는 기능별 패키지를 기본 단위로 한다.

```txt
packages/
  gif-to-video/
  video-background/
  video-person-center/
  audio-mixer/
```

각 패키지는 자신의 목적, 의존성, 빌드 결과를 독립적으로 가진다.

예시:

```txt
@track-forge/gif-to-video
  GIF 또는 이미지 소스에서 video MediaStreamTrack을 생성한다.

@track-forge/video-background
  video MediaStreamTrack에 배경을 합성한다.

@track-forge/video-person-center
  video MediaStreamTrack에서 사람 위치를 읽고 화면 중앙에 맞춘다.

@track-forge/audio-mixer
  여러 audio MediaStreamTrack을 하나의 audio track으로 합성한다.
```

# 기술스택

TypeScript, pnpm 워크스페이스. Vite 8(Rolldown),

# Workspace

workspace 범위는 `packages/*`를 기본으로 한다.

```yaml
packages:
  - "packages/*"
```

루트 package는 workspace orchestration만 담당한다.

루트가 담당하는 것:

- dependency version policy
- workspace scripts
- TypeScript base config
- lint/test/build orchestration
- release orchestration

루트가 담당하지 않는 것:

- 특정 processor의 runtime 구현
- 특정 processor의 무거운 optional dependency
- 특정 패키지의 브라우저 API 세부 정책

# 패키지 구조

각 패키지는 가능한 한 같은 파일 구조를 따른다.

```txt
packages/<name>/
  package.json
  vite.config.ts
  tsconfig.json
  src/
    <name>.module.ts
    <name>.types.ts
    index.ts
  README.md
```

패키지의 public API는 `src/index.ts`에서만 export한다.

# package.json 규칙

각 패키지는 `@track-forge/<name>` scope를 사용한다.

기본 형태:

```json
{
  "name": "@track-forge/<name>",
  "version": "0.0.0",
  "type": "module",
  "private": false,
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "files": ["dist", "README.md"],
  "scripts": {
    "build": "vite build",
    "typecheck": "tsc --noEmit"
  }
}
```

ESM을 기본 대상으로 삼되, CJS 호환이 필요한 경우에만 CJS 산출물을 유지한다.

CJS 산출물이 필요한 패키지는 `main`, `exports["."].require`, Vite `formats: ["es", "cjs"]`를 해당 패키지에만 추가한다.

# 빌드 시스템

패키지 빌드는 Vite library mode를 사용한다.

기본 기준:

- Vite 8
- Rolldown 기반 bundle
- TypeScript source
- ESM output
- declaration output via Vite plugin or TypeScript emit
- browser-first package

패키지는 브라우저 API를 직접 사용할 수 있으므로, 빌드 결과가 Node.js polyfill에 의존하면 안 된다.

권장 Vite 설정 형태:

```ts
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    lib: {
      entry: "src/index.ts",
      formats: ["es"],
      fileName: "index",
    },
    sourcemap: true,
    minify: false,
  },
});
```

CJS 산출물이 필요한 패키지는 해당 패키지에서만 명시한다.

```ts
formats: ["es", "cjs"];
```

# TypeScript

TypeScript 설정은 루트 base config를 두고 각 패키지가 상속한다.

```txt
tsconfig.base.json
packages/<name>/tsconfig.json
```

기본 기준:

- strict mode 사용
- DOM type 사용
- ESM module 사용
- declaration 생성은 build pipeline에서 처리
- package 내부 import는 가능한 상대 경로 사용

# 의존성

의존성은 필요한 패키지에만 둔다.

예를 들어 사람 감지 모델이 `video-person-center`에 필요하더라도, `gif-to-video`나 `audio-mixer`가 그 의존성을 가져서는 안 된다.

공통 의존성은 다음 조건을 만족할 때만 루트 또는 shared package로 올린다.

- 세 개 이상의 패키지에서 반복된다.
- 버전 정책을 반드시 통일해야 한다.
- 중복 제거가 실제 bundle 크기나 유지보수에 의미가 있다.

공통 패키지는 미리 만들지 않는다. 필요가 생기면 역할이 드러나는 이름으로 만든다.

예시:

```txt
packages/track-types/
packages/track-pipeline/
packages/browser-support/
```

# Script 규칙

루트 script는 workspace 전체 작업을 실행한다.

```json
{
  "scripts": {
    "build": "pnpm -r build",
    "typecheck": "pnpm -r typecheck",
    "test": "pnpm -r test"
  }
}
```

패키지 script는 자기 패키지 안의 작업만 실행한다.

```json
{
  "scripts": {
    "build": "vite build",
    "typecheck": "tsc --noEmit",
    "test": "vitest run"
  }
}
```

# 테스트

브라우저 API 의존성이 있는 패키지는 Node.js 단위 테스트만으로 충분하지 않을 수 있다.

테스트 계층:

- 순수 함수와 옵션 검증은 Vitest로 테스트한다.
- `MediaStreamTrack`, `Canvas`, `AudioContext` 동작은 브라우저 환경 테스트로 검증한다.
- 실제 카메라, 마이크, GPU, 모델 의존성이 있는 기능은 demo 또는 e2e 테스트에서 검증한다.

# Demo

demo app은 package runtime 확인을 위해 별도 workspace package로 둘 수 있다.

```txt
packages/demo/
```

demo는 배포 패키지가 아니라 검증 도구다.

# Publish

각 기능 패키지는 독립 배포 가능해야 한다.

배포 전 확인 항목:

- `pnpm build`
- `pnpm typecheck`
- package exports 확인
- `dist` 산출물 확인
- README의 기본 사용 예시 확인

# 설계 판단

Track Forge는 기능별 패키지 구조를 기본으로 한다.

`video`, `audio`, `core` 같은 큰 분류 패키지는 현재 만들지 않는다. track 종류가 다르더라도 같은 빌드 시스템과 같은 package contract를 사용할 수 있고, 일부 기능은 video/audio 경계를 동시에 가질 수 있기 때문이다.

공통화는 실제 중복과 필요가 생긴 뒤에 진행한다. 공통 패키지가 생기더라도 `core`처럼 넓은 이름보다 `track-types`, `track-pipeline`, `browser-support`처럼 역할이 드러나는 이름을 우선한다.

이 구조의 기준은 도메인 분류가 아니라 package 단위의 독립성이다.
