# Title Discovery and Quark Selection

Use this reference only in title-discovery mode. Share-link mode can skip the browser site and begin with Quark inspection.

## Site discovery

1. Confirm the configured private media site is logged in. If a login or safety challenge is pending, stop and request the user's interactive sign-in.
2. Search the user's original title without silently rewriting it. Collect result title, year/type metadata, and detail URL.
3. Resolve same-title works by title, year, country, media type, aliases, and season. Ask once when two candidates remain materially ambiguous.
4. Open the net-disk section and activate the Quark provider. Extract only canonical `https://pan.quark.cn/s/...` share URLs.
5. Return a short candidate list with page-text reasons and warnings. Page text is pre-ranking only; it is not evidence of the actual file quality.

## Real share inspection

Before selecting a release, inspect the share recursively:

- distinguish directories from files using the provider's actual type field;
- find the requested work inside collection folders;
- record exact file name, byte size, file type, episode number, and subtitle stem;
- verify season/episode continuity for series;
- ignore posters, NFO files, readme files, promos, samples, and unrelated franchise members;
- pair subtitles by filename stem, never by proximity;
- keep the exact file IDs needed for selective saving.

Select the first candidate that satisfies scope and the configured quality policy. If no candidate meets the preferred tier and fallback is enabled, choose the highest available tier and report the fallback. Do not stage multiple variants unless the user requested them.

## Output to the executor

Produce a non-secret manifest containing:

```json
{
  "kind": "movie|tv-season|anime-season",
  "title": "localized title",
  "year": 2026,
  "season": 1,
  "items": [
    {"source_name": "exact.mkv", "file_id": "provider-id", "bytes": 123456, "kind": "video", "episode": 1},
    {"source_name": "exact.zh-CN.srt", "file_id": "provider-id", "bytes": 4567, "kind": "subtitle", "language": "zh-CN", "episode": 1}
  ]
}
```

Do not include cookies, access tokens, share passwords, or request headers in the manifest.
