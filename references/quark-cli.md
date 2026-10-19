# Quark CLI Installation and Authorization

This is the provider boundary for the media-ingestion skill. The skill may be used with a bundled Quark adapter or with an externally installed provider CLI. Do not copy the provider's access token, cookies, or authorization code into a skill file or job state.

## First-use preparation

1. Detect the platform and whether Node.js is available. If the provider package ships an installer, run its installer before any Quark command:

   ```bash
   bash scripts/install.sh
   ```

   On Windows, use the provider's documented PowerShell/Node equivalent when `bash` is unavailable. Do not invent a package URL or silently install an untrusted replacement.

2. Verify the CLI responds to its help or version command. A failed installer is a setup failure; stop and show the user the non-secret error.

3. Check authorization with the provider's account/status command if available. If it reports an authorization error or expired token, do not repeat the media operation first; run the login flow.

The current Quark provider used by the reference deployment follows this contract:

```bash
node scripts/quark-drive.cjs <command> [options]
```

Every provider command must receive the original user request and one stable session ID when the provider requires those audit parameters. Keep the session ID in memory for the current conversation; it is not a secret.

## Login flow

Prefer the browser OAuth login:

```bash
node scripts/quark-drive.cjs login
```

The command may open a browser and wait for authorization. Tell the user exactly what is happening and wait for the command to finish. After success, run a non-destructive account/status check before saving or transferring any media.

## Authorization-code fallback

If browser OAuth cannot complete, the provider may print an authorization URL. Do not stop after displaying only the URL:

1. Show the URL as a clickable link.
2. Tell the user to open it, complete Quark authorization, and copy the `code` parameter from the browser's redirect URL.
3. Explicitly ask the user to paste that code back into the current Agent conversation.
4. Never ask the user to paste a password, cookie, refresh token, or access token.
5. After the user supplies the code, pass it directly to the provider's token-login command in memory:

   ```bash
   node scripts/quark-drive.cjs login --token <authorization-code>
   ```

6. Check the login result. Only after a successful result should the Agent resume the original media-ingestion operation.

An authorization code is sensitive and short-lived. Do not write it to a file, log it, quote it back to the user, or include it in a task manifest.

## Authorization failure handling

- If an operation reports “not authorized”, show the provider's user-facing message, run the login flow once, then retry the original operation once after successful login.
- If login times out, the browser does not open, or the provider returns a non-success result, show the failure message and stop. Wait for the user to provide a code or explicitly request another login attempt; do not loop.
- If the provider says the existing authorization is still valid, stop the login attempt and continue the original operation. Do not repeatedly log in.
- If the user asks to switch accounts, use the provider's explicit unauthorize/logout flow and ask for confirmation before any destructive local cleanup.

## Provider command boundary

The media-ingestion skill needs only these provider capabilities:

```text
install_or_update
login_browser
login_authorization_code
account_status
share_detail
share_save_selected
```

Provider-specific command names may vary. If one capability is unavailable, report the missing capability and stop at the last safe phase. Do not replace Quark operations with downloading files into the Agent workspace.
