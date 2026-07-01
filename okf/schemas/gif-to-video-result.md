---
type: schema
title: GifToVideoResult
description: convert(source, options?)의 결과물
resource:
tags: [gif-to-video, schema]
timestamp: 2026-07-01T00:00:00Z
---

`GifToVideoModule.convert`의 결과물.

# Schema

| 필드      | 타입                          | 설명                              |
| --------- | ----------------------------- | --------------------------------- |
| `track`   | `MediaStreamVideoTrack`       | 소스를 변환해 생성한 video track  |
| `dispose` | `() => void \| Promise<void>` | 생성된 track과 내부 리소스를 해제 |

# 규칙

- `track.kind`는 `"video"`여야 한다.
- `dispose()`는 idempotent 해야 한다. 여러 번 호출해도 예외를 던지지 않는다.
- `dispose()`는 output track을 stop하고, 내부 render loop와 임시 리소스를 해제한다.
- 호출자는 더 이상 track을 사용하지 않을 때 반드시 `dispose()`를 호출해야 한다.

# 관련

- 모듈: [gif-to-video](../modules/gif-to-video.md)
