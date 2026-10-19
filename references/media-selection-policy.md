# Selection, Naming, and Conflict Policy

## Quality preference

Use the configured preference as a ranking signal only after scope completeness is satisfied:

1. complete requested movie/season/episode range;
2. correct work and year;
3. matching subtitle coverage;
4. preferred resolution (`2160p`/`4K`, then `1080p`);
5. HDR/Dolby Vision, source, codec, audio, bitrate, and size;
6. stable uploader/update metadata.

Reject or strongly penalize CAM, TS, TC, trailers, samples, previews, incomplete seasons, and wrong-franchise entries. Never infer quality only from a share title.

## Default naming

```text
Movie folder:  <title> (<year>)
Movie file:    <title> (<year>).<ext>
Season folder: <title> S<season two digits>
Episode file:  <title> S<season two digits>E<episode two digits>.<ext>
Subtitle:      <same canonical stem>.<language-tag>.<ext>
```

Use `zh-CN`, `zh-TW`, `en`, `ja`, and `ko` when the language is known. Preserve a release/edition suffix only when it distinguishes a requested version or prevents a collision.

## Conflict rules

- Same canonical name and same byte size: treat as already present; skip or verify, never copy over it.
- Same canonical name and different byte size: stop and ask the user.
- Destination directory contains unrelated files: do not rename the directory; create a task-owned directory or stop.
- Source is still downloading or its size changes: stop and resume later.
- Unknown year, title, episode mapping, or subtitle pairing: stop and ask rather than guess.

## Destructive actions

The default is copy. Moving, deleting Quark originals, deleting staging files, or replacing an existing target requires an explicit request and a separate confirmation immediately before the operation.
