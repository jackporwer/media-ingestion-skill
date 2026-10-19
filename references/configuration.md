# Configuration Contract

The skill is portable. Values below are examples; replace them with the user's deployment values at runtime.

## Endpoints and credentials

```dotenv
MEDIA_SITE_BASE_URL=https://example-media-site.invalid
MEDIA_SITE_USERNAME=
MEDIA_SITE_PASSWORD=

QUARK_BASE_URL=https://pan.quark.cn
QUARK_ACCOUNT_REF=
QUARK_CLI_DIR=

OPENLIST_BASE_URL=http://127.0.0.1:5244
OPENLIST_USERNAME=
OPENLIST_PASSWORD=
```

Do not put real values in a skill, a job file, a public configuration example, or a chat message. The Agent should read these values from an environment/secret manager or ask interactively.

## Storage roots

```yaml
media_roots:
  movies: /library/movies
  tv: /library/tv
  anime: /library/anime
  staging: /quark/staging
```

The roots are deployment-specific. Validate that each destination exists or that the user explicitly permits creating it. Never change a configured root based on a source share's folder names.

## Policy example

```yaml
selection:
  preferred_resolution: 2160p
  allow_fallback: true
  prefer_hdr: true
  prefer_dolby_vision: true
  require_chinese_subtitle: false
  mode: first-qualified

conflicts:
  same_name_same_size: skip
  same_name_different_size: ask
  overwrite: false

naming:
  movie_folder: "{title} ({year})"
  movie_file: "{title} ({year})"
  season_folder: "{title} S{season:02d}"
  episode_file: "{title} S{season:02d}E{episode:02d}"
```

Treat this as a schema, not a required file format. A project may use YAML, JSON, environment variables, or its own settings page as long as it produces the same values.

## Safe configuration checks

Before any side effect, validate:

- OpenList and Quark endpoints are reachable;
- credentials are present without printing them;
- source and destination roots are distinct and correctly classified;
- the selected naming policy has no path traversal or empty required fields;
- overwrite is disabled unless the user explicitly changed it;
- the job state directory is writable and not inside the media library.
