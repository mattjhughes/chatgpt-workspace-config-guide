# ChatGPT Edu Workspace Config Guide

This repository contains a least-privilege managed Codex configuration for non-technical users in a ChatGPT Edu workspace.

## Files

- `requirements.toml` — administrator-enforced security boundaries.
- `config.toml` — managed defaults plus the ChatGPT Edu workspace sign-in restriction.
- `recommended-safe-noncoder-managed-policy.md` — a plain-language explanation of each important setting, including benefits and tradeoffs.

## Required deployment edit

Before deploying `config.toml`, replace:

```toml
forced_chatgpt_workspace_id = "REPLACE_WITH_EDU_WORKSPACE_UUID"
```

with the UUID of the intended ChatGPT Edu workspace. Keep the value quoted.

The restriction does not add users to the workspace or grant them a seat. It only prevents Codex from accepting a ChatGPT login associated with a different workspace.

This policy requires Codex 0.138.0 or later for managed permission profiles. Test it with a pilot group before organization-wide deployment.

## Official documentation

- [Advanced configuration](https://learn.chatgpt.com/docs/config-file/config-advanced)
- [Configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [Managed configuration](https://learn.chatgpt.com/docs/enterprise/managed-configuration)
- [Permissions](https://learn.chatgpt.com/docs/permissions)
