# Job State and Resume Rules

Store non-secret state outside the media library. A JSON file or SQLite row is sufficient.

```json
{
  "job_id": "20261019-abc123",
  "request": "original user text",
  "kind": "movie|tv-season|anime-season",
  "title": "localized title",
  "year": 2026,
  "season": 1,
  "phase": "discovered|selected|staging|copying|verifying|renaming|completed|blocked|failed",
  "source": {"share_url": "https://pan.quark.cn/s/...", "staging_dir": "/quark/staging/title"},
  "destination_dir": "/library/movies/title (2026)",
  "items": [{"source_name":"exact.mkv","bytes":123,"target_name":"title (2026).mkv","task_id":"openlist-task-id"}],
  "updated_at": "2026-10-19T12:00:00+08:00",
  "error": null
}
```

Never store passwords, access tokens, refresh tokens, cookies, share passcodes, or Authorization headers. On resume:

1. read the state;
2. refresh the source and destination listings;
3. reuse known task IDs when still available;
4. verify artifacts before submitting anything new;
5. submit only missing items;
6. continue from the first incomplete phase.

Terminal states:

- `completed`: all manifest items copied, verified, and canonically named;
- `blocked`: user confirmation or genuine ambiguity is required;
- `failed`: deterministic operation failed or a postcondition was not met.
