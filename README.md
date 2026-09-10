# ChatGPT Edu Workspace Config Guide

This repository contains a least-privilege managed Codex configuration for non-technical users in a ChatGPT Edu workspace.

## Files

- `requirements.toml` — administrator-enforced security boundaries.
- `config.toml` — the two settings that restrict sign-in to the ChatGPT Edu workspace.
- `recommended-safe-noncoder-managed-policy.md` — a plain-language explanation of each important setting, including benefits and tradeoffs.

## Add to an existing `config.toml`

This template is intentionally limited to authentication. It does not set permissions, web search, app, or other Codex behavior.

If a device already has a `config.toml`, do not replace that file. Paste these two settings into it instead:

```toml
forced_login_method = "chatgpt"
forced_chatgpt_workspace_id = "REPLACE_WITH_EDU_WORKSPACE_UUID"
```

Replace the placeholder with the UUID of the intended ChatGPT Edu workspace, and keep the value quoted.

The restriction does not add users to the workspace or grant them a seat. It only prevents Codex from accepting a ChatGPT login associated with a different workspace.

Keep `forced_login_method` and `forced_chatgpt_workspace_id` at the top level of `config.toml`, before the first `[table]` header. TOML keys after a table header belong to that table; placing these keys there would stop Codex from treating them as authentication settings.

## Test before deployment

Use an MDM-managed pilot device before assigning this configuration broadly:

1. Deploy the completed `config.toml` and restart Codex.
2. Sign in with an account that belongs to the intended Edu workspace; confirm Codex starts normally.
3. Attempt to sign in with a personal ChatGPT account or an account in a different workspace; confirm Codex rejects it.
4. Confirm API-key sign-in is unavailable or rejected.
5. Confirm the configuration remains present after the normal MDM sync or device restart cycle.

This policy requires Codex 0.138.0 or later for managed permission profiles. Start with a pilot group before organization-wide deployment.

## Official documentation

- [Advanced configuration](https://learn.chatgpt.com/docs/config-file/config-advanced)
- [Configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [Managed configuration](https://learn.chatgpt.com/docs/enterprise/managed-configuration)
- [Permissions](https://learn.chatgpt.com/docs/permissions)
