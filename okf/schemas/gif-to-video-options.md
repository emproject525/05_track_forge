---
type: schema
title: GifToVideoOptions
description: convert(source, options?)에 넘기는 GIF to video track 변환 옵션
resource: packages/gif-to-video/src/gif-to-video.types.ts
tags: [gif-to-video, schema]
timestamp: 2026-07-01T00:00:00Z
---

`GifToVideoModule.convert`의 두번째 파라미터로 넘기는 옵션.

# Schema

| 필드          | 타입                                        | 기본        | 설명                                           |
| ------------- | ------------------------------------------- | ----------- | ---------------------------------------------- |
| `width?`      | `number`                                    | 원본 너비   | 출력 video track의 너비(px)                    |
| `height?`     | `number`                                    | 원본 높이   | 출력 video track의 높이(px)                    |
| `fps?`        | `number`                                    | GIF 타이밍  | GIF 프레임 타이밍을 무시하고 고정 FPS로 렌더링 |
| `loop?`       | `boolean \| number`                         | `true`      | 반복 재생 여부. 숫자면 지정한 횟수만 반복      |
| `fit?`        | `"contain" \| "cover" \| "fill"`            | `"contain"` | 원본 프레임을 출력 크기에 맞추는 방식          |
| `background?` | `string \| CanvasGradient \| CanvasPattern` | 투명        | 여백 또는 투명 영역에 칠할 배경                |
| `signal?`     | `AbortSignal`                               | -           | 변환 시작 또는 렌더링을 취소하기 위한 signal   |
| `trackLabel?` | `string`                                    | -           | 생성된 video track을 식별하기 위한 label 힌트  |

# 규칙

- `width`와 `height`는 둘 다 생략할 수 있다. 둘 다 생략하면 원본 GIF 크기를 사용한다.
- `width` 또는 `height` 중 하나만 지정하면 원본 비율을 유지해 나머지 값을 계산한다.
- `fps`가 생략되면 GIF에 포함된 프레임 delay를 사용한다.
- `fps`가 지정되면 GIF의 원본 delay 대신 고정된 렌더링 간격을 사용한다.
- `loop: true`는 무한 반복을 의미한다.
- `loop: false`는 마지막 프레임에서 렌더링을 멈춘다. output track을 자동으로 stop할지는 구현 문서에서 명시한다.
- `loop`가 숫자면 해당 횟수만큼 재생한 뒤 마지막 프레임에서 멈춘다.
- `signal`이 abort되면 변환은 실패하거나 진행 중인 렌더링을 중단해야 한다.

# 제약

- `width`, `height`, `fps`는 양수여야 한다.
- `loop`가 숫자일 경우 `0` 이상의 정수여야 한다.
- `fit: "fill"`은 원본 비율을 유지하지 않는다.
- `background`가 지정되지 않으면 투명 영역은 canvas 기본 투명 상태로 둔다.

# 예시

```ts
await convert(file, {
  width: 1280,
  height: 720,
  fit: "contain",
  background: "#000",
  fps: 30,
});
```

# 관련

- 모듈: [gif-to-video](../modules/gif-to-video.md)
