---
type: module
title: gif-to-video
description: GIF 또는 이미지 소스에서 video MediaStreamTrack을 생성한다.
resource: packages/gif-to-video/src/gif-to-video.module.ts
tags: [gif-to-video]
timestamp: 2026-07-01T00:00:00Z
---

# 개요

GIF 또는 이미지 소스에서 video MediaStreamTrack을 생성한다.
ESM(`import`) 와 UMD 둘 다 제공.

# 공개 API

| 이름      | 종류     | 비동기 | 설명                    |
| --------- | -------- | ------ | ----------------------- |
| `convert` | function | yes    | [description](#convert) |

## convert

`convert()`는 source를 디코딩하고 canvas 기반 렌더링 루프를 구성한 뒤, 생성된 `MediaStreamVideoTrack`과 cleanup 함수를 반환한다.

### Parameters

| 이름      | 타입                                                      | 필수 | 설명                              |
| --------- | --------------------------------------------------------- | ---- | --------------------------------- |
| `source`  | `GifToVideoSource`                                        | yes  | 변환할 GIF 또는 이미지 소스       |
| `options` | [`GifToVideoOptions`](../schemas/gif-to-video-options.md) | no   | 출력 크기, FPS, loop 등 변환 옵션 |

### Source

`GifToVideoSource`는 다음 입력을 지원한다.

| 타입          | 설명                                      |
| ------------- | ----------------------------------------- |
| `Blob`        | 브라우저 파일 입력 또는 fetch 결과        |
| `File`        | `<input type="file">`에서 선택한 GIF 파일 |
| `ArrayBuffer` | 이미 메모리에 로드된 GIF binary           |
| `string`      | GIF URL                                   |

`string` source는 fetch 가능한 URL로 해석한다.

### Returns

| 타입                        | 설명                              |
| --------------------------- | --------------------------------- |
| `Promise<GifToVideoResult>` | 생성된 video track과 cleanup 함수 |

# 규칙

- `convert()` 호출마다 독립적인 `GifToVideoResult`를 반환한다.
- `convert()`는 singleton service가 아니라 conversion factory로 동작한다.
- 각 호출은 독립적인 video track과 내부 리소스를 가진다.
- 호출자는 반환된 result를 더 이상 사용하지 않을 때 `dispose()`를 호출해야 한다.
- `source`가 URL인 경우 네트워크 실패와 CORS 실패가 발생할 수 있다.
- `source`가 GIF가 아니거나 디코딩할 수 없으면 실패해야 한다.
