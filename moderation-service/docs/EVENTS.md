# Events

## `video.moderation.completed`

The completed event remains backward compatible with the existing moderation payload. `evidence` still contains the original fields (`label`, `safeScore`, `nsfwScore`, `timestampSeconds`, `sampledFrameCount`, `thresholds`) and may now include optional audit fields:

- `policy`
- `matchedFrameCount`
- `secondaryDetector`
- `secondaryScore`

Consumers that ignore unknown fields can continue using the existing contract unchanged.
