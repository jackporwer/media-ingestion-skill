# OpenList Verification Contract

Use the OpenList HTTP API or a deployment-specific client. Do not assume a particular host port or container name.

## Login

`POST /api/auth/login`

```json
{"username":"configured-user","password":"configured-secret"}
```

Keep `data.token` in memory only. Never print it or put it in job state.

## List and preflight

Use `POST /api/fs/list` for both source and destination. Use `refresh: true` before collision decisions and after a task completes. Confirm file name, `is_dir`, and byte size.

## Copy

`POST /api/fs/copy`

```json
{
  "src_dir":"/configured/staging",
  "dst_dir":"/configured/library/title",
  "names":["exact-name.mkv"],
  "overwrite":false,
  "skip_existing":true
}
```

Record every returned task ID. A successful HTTP response means only that a task was queued.

## Poll and verify

Poll each task with `POST /api/task/copy/info?tid=<task-id>`. Use `state` as the authoritative status; do not infer completion from `progress=100` or a human-readable phase label.

Only mark an item complete when all checks pass:

```text
task state is successful
progress is complete
error is empty
destination listing was refreshed
destination file exists as a non-directory
destination byte size equals the recorded source byte size
```

If a task record has expired, judge the result from a refreshed destination listing and the original manifest. Do not submit a duplicate task until the destination has been checked.

## Rename boundary

Rename only after copy verification. The only safe rename shape is an individual completed file or a newly created task-owned directory whose direct children all belong to the current manifest. Never rename a media-library root, category root, or unrelated collection directory.
