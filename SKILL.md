---
name: media-ingestion
description: Discover, stage, copy, verify, and canonically name movies, series, and anime from Quark shares into an OpenList-managed media library. Use when a user asks to add media by title or share link; do not use for deleting sources or arbitrary file management.
---

# Media Ingestion

Use this skill as a **generic Agent workflow**. It must work from any Agent that has the required browser, terminal, and HTTP capabilities; it must not assume Hermes, a particular Agent runtime, a specific NAS path, or a proprietary scheduler.

The workflow is:

```text
user request
  → resolve title and scope
  → discover Quark candidates, or inspect a supplied share
  → inspect real files and select exact video/subtitle files
  → save selected files to Quark staging
  → copy through OpenList
  → poll and verify task completion and byte sizes
  → rename only inside the task-owned destination directory
  → report the verified result
```

## Entry modes

Choose the narrowest mode that satisfies the request:

- **Share-link mode:** use when the user gives a Quark share URL. Inspect the share first; do not search for a different release unless the user asks.
- **Title-discovery mode:** use when the user gives only a movie, series, anime, year, season, or episode range. Read [references/discovery.md](references/discovery.md) before using a browser.
- **Resume mode:** use when a prior job state exists. Read [references/job-state.md](references/job-state.md), reuse recorded task IDs, and never submit a duplicate copy just because a previous run ended.

## Required behavior

1. Load [references/configuration.md](references/configuration.md) when determining endpoints, roots, credentials, naming, or policy.
2. Before the first Quark operation, read [references/quark-cli.md](references/quark-cli.md). Install or update the provider CLI, check authorization, and stop for user authorization when required.
3. Resolve the requested scope before changing anything: movie versus series, year, season, episode range, language, and whether the user wants one version or a collection.
4. Inspect the **real Quark listing**, recursively when necessary. A share title is only a hint; select files by exact name, type, size, episode number, and subtitle pairing.
5. Present an ambiguity or destructive conflict for user confirmation. Never guess between same-title works, overwrite a same-name/different-size target, delete a source, or reclassify an existing media root silently.
6. Use OpenList for copies and moves. Read [references/openlist-verification.md](references/openlist-verification.md) before invoking OpenList operations.
7. Treat an asynchronous copy task as incomplete until its terminal state, error field, refreshed destination listing, and byte-size comparison all pass.
8. Rename only after a successful verified copy, and only inside a newly created or task-dedicated destination directory whose direct children belong to the current manifest. Never rename a media root, category root, or unrelated existing collection.
9. Persist non-secret job state so an interrupted Agent can resume. Store task IDs, source names and sizes, target names, phase, and failure reasons; never persist passwords, tokens, cookies, or Authorization headers.
10. Report the exact phase and verified outcome. `queued`, `progress=100`, or a successful task-creation response is not completion.

## Canonical naming

Read [references/media-selection-policy.md](references/media-selection-policy.md) for configurable preferences and naming rules. Default forms are:

```text
Movie: <Chinese title> (<year>)/<Chinese title> (<year>).mkv
TV:    <Chinese title> S01/<Chinese title> S01E01.mkv
Subs:  <same canonical stem>.<language-tag>.<extension>
```

Keep the user's configured media roots and categories. Do not invent a new root because a source share uses a different folder layout.

## Capability fallback

- Without a browser, accept a supplied share link or return a discovery plan instead of pretending to search.
- Without terminal access, return the exact configuration and execution requirements; do not claim a transfer ran.
- Without persistent storage, perform only a one-shot operation and warn that recovery cannot resume automatically.
- Without OpenList access, stop after discovery and selection; do not download files into the Agent workspace as a substitute.

## Security boundary

Credentials come from the user's configured environment, secret store, or interactive input. Never print them, put them in a prompt, embed them in a skill file, include them in job state, or commit them to a repository. Default to copy; moving or deleting a source requires a separate explicit request and confirmation.

## Supporting references

- [references/configuration.md](references/configuration.md): portable configuration contract and environment variables.
- [references/quark-cli.md](references/quark-cli.md): portable Quark CLI installation, login, authorization-code fallback, and authorization-expiry handling.
- [references/discovery.md](references/discovery.md): browser discovery and Quark candidate extraction.
- [references/media-selection-policy.md](references/media-selection-policy.md): quality, scope, subtitle, conflict, and naming policy.
- [references/openlist-verification.md](references/openlist-verification.md): API operations, task polling, and post-copy proof.
- [references/job-state.md](references/job-state.md): durable state schema and resume rules.
