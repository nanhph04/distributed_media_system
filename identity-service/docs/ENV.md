# IDENTITY SERVICE ENV

Email:

```text
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USER=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_FROM=noreply@yourdomain.com
```

Kafka topics:

```text
KAFKA_CHANNEL_CREATED_TOPIC=channel.created
KAFKA_MEMBERSHIP_AUTO_RENEW_REMINDER_REQUESTED_TOPIC=membership.auto_renew.reminder_requested
OUTBOX_PUBLISH_BATCH_SIZE=25
OUTBOX_PUBLISH_INTERVAL_MS=5000
```

`membership.auto_renew.reminder_requested` is consumed to send the membership
renewal reminder email. The consumer sends only to verified accounts and uses
Redis/cache idempotency to avoid duplicate emails.

`OUTBOX_PUBLISH_BATCH_SIZE` and `OUTBOX_PUBLISH_INTERVAL_MS` are optional. They
control how many pending outbox messages are claimed per cycle and how often the
publisher polls the `outbox_messages` table.

MinIO avatar storage:

```text
MINIO_ENDPOINT=localhost
MINIO_INTERNAL_ENDPOINT=localhost
MINIO_PORT=9000
MINIO_USE_SSL=false
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_AVATAR_BUCKET=identity-avatars
MINIO_AVATAR_PUBLIC_BASE_URL=http://localhost:9000/identity-avatars
MINIO_AVATAR_MAX_FILE_SIZE=5242880
```

Avatar upload uses `POST /api/user/users/profile/avatar/upload-url` with
`multipart/form-data`. The route name is kept for api-gateway compatibility,
but the endpoint no longer returns a presigned URL. The backend uploads to
MinIO through `MINIO_INTERNAL_ENDPOINT`, so browser CORS to MinIO is not
required for upload.

`MINIO_AVATAR_PUBLIC_BASE_URL` is still used to build `avatarUrl` returned to
clients. Set it to a URL that browsers can open for image display. If frontend
does not run on the same machine as MinIO, do not use `localhost` here.
