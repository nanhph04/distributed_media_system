# Environment

## NSFW moderation

The default moderation policy is `explicit_only`. A video is auto-rejected only when explicit/porn evidence appears in at least `NSFW_REJECT_MIN_FRAMES` nearby frames and the secondary detector confirms the evidence. Suggestive or sexy content is sent to manual review when it crosses the manual threshold.

| Variable | Default | Description |
| --- | --- | --- |
| `FRAME_INTERVAL_SECONDS` | `2` | Initial video frame sampling interval. |
| `NSFW_POLICY` | `explicit_only` | Auto-reject policy. |
| `NSFW_REJECT_THRESHOLD` | `0.9` | Primary model score required for reject candidates. |
| `NSFW_MANUAL_THRESHOLD` | `0.6` | Primary model score required for manual review candidates. |
| `NSFW_REJECT_MIN_FRAMES` | `2` | Minimum verified frames required for auto reject. |
| `NSFW_REJECT_WINDOW_SECONDS` | `6` | Time window for grouping reject candidate frames. |
| `NSFW_SECONDARY_DETECTOR` | `opennsfw2` | Secondary verifier used for explicit reject candidates. |
| `OPENNSFW2_REJECT_THRESHOLD` | `0.85` | Secondary verifier score required for auto reject. |
| `NSFW_RESAMPLE_RADIUS_SECONDS` | `1` | Seconds before/after a suspicious frame to resample. |
| `NSFW_RESAMPLE_STEP_SECONDS` | `1` | Step size for resampling around suspicious frames. |
| `HF_MODEL_NAME` | `Falconsai/nsfw_image_detection` | Primary Hugging Face image classifier. |

## Worker concurrency

| Variable | Default | Description |
| --- | --- | --- |
| `MODERATION_PARALLEL_ENABLED` | `false` | Enables multiple Kafka moderation workers in one service process. |
| `MODERATION_WORKER_CONCURRENCY` | `1` | Number of moderation workers to start when parallel processing is enabled. Values below `1` are treated as `1`. |

For parallel moderation to improve throughput, the Kafka topic in
`KAFKA_VIDEO_MODERATION_REQUESTED_TOPIC` must have at least as many partitions
as active moderation workers. Kafka assigns at most one consumer in the same
consumer group to each partition.
