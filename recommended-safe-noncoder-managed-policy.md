# Recommended Safe Codex Policy for Non-Technical Users

## Purpose

This guide describes a balanced managed Codex policy for non-technical users. The policy is designed to let Codex:

- Read files in the current project.
- Create and edit deliverables inside a predictable `Working` folder.
- Perform cached hosted web searches.
- Use the isolated in-app browser.
- Use approved apps and connectors cautiously.
- Run administrator-managed hooks.

At the same time, it prevents Codex from:

- Writing elsewhere in the project or device through local commands.
- Reading common credential and private-key files.
- Giving generated shell or Python code network access.
- Escaping the sandbox through a user approval.
- Controlling desktop applications through Computer Use.
- Controlling the user's normal external browser.
- Loading user, project, session, or plugin hooks.
- Enabling unapproved permission profiles, plugins, or MCP servers.

These controls primarily restrict actions Codex performs on the user's behalf. They do not normally prevent the human user from using their own computer, browser, Terminal, or applications.

## Important file distinction

Codex uses two related kinds of configuration:

1. `requirements.toml` contains administrator-enforced restrictions that users cannot override.
2. `config.toml` contains defaults and preferences. Managed defaults can supply these values, but they are not automatically equivalent to hard requirements.

Do not combine every setting into one file merely because both files use TOML syntax. The recommended policy below uses:

- A primary `requirements.toml` for enforceable security boundaries.
- An optional managed-default `config.toml` section for app behavior and starting preferences.

The managed permission-profile portions require Codex 0.138.0 or later. Codex 0.137.0 and earlier ignore managed `allowed_permission_profiles` and managed `default_permissions`.

## Recommended `requirements.toml`

```toml
# Balanced managed requirements for non-technical users.
# Requires Codex 0.138.0 or later for managed permission profiles.

# Apply the custom least-privilege profile by default.
default_permissions = "safe-noncoder"

# Do not permit approval-based escapes from the sandbox.
allowed_approval_policies = ["never"]

# Permit cached hosted search only. The disabled mode remains implicitly allowed.
allowed_web_search_modes = ["cached"]

# Disable device Remote Control and login-shell behavior.
allow_remote_control = false
allow_login_shell = false

# Enable only administrator-managed hooks. User, project, session, and plugin
# hooks are skipped even though the managed hooks feature is enabled.
allow_managed_hooks_only = true

# This is a complete permission-profile allowlist. Built-in workspace and
# full-device profiles, future built-ins, and user-defined profiles are denied.
[allowed_permission_profiles]
safe-noncoder = true

[features]
# Apps may be used subject to their own app and tool approval controls.
apps = true

# Plugins are disabled for this baseline.
plugins = false

# Allow the isolated in-app browser, but not the normal external browser or
# full Chrome DevTools Protocol access.
in_app_browser = true
browser_use = true
browser_use_external = false
browser_use_full_cdp_access = false

# Do not let Codex click or type in arbitrary desktop applications.
computer_use = false

# Favor the normal execution mode rather than the speed-oriented mode.
fast_mode = false

# Managed hooks may run. Unmanaged hooks remain blocked by
# allow_managed_hooks_only above.
hooks = true

[permissions.safe-noncoder]
description = "Read project files and write only inside each project's Working folder."
extends = ":read-only"

[permissions.safe-noncoder.filesystem]
# Start by denying reads across the filesystem, then reopen only the minimal
# runtime paths and the current workspace roots below.
":root" = "deny"
":minimal" = "read"

# Helps platforms that snapshot recursive deny globs before sandbox startup.
glob_scan_max_depth = 4

[permissions.safe-noncoder.filesystem.":workspace_roots"]
# Read the current project, but write only inside Working.
"." = "read"
"Working" = "write"

# Make protection of repository metadata and Codex configuration explicit.
".codex" = "read"
".git" = "read"

# Deny common environment, authentication, certificate, and private-key files.
"**/.env" = "deny"
"**/.env.*" = "deny"
"**/*.env" = "deny"
"**/.npmrc" = "deny"
"**/.pypirc" = "deny"
"**/.netrc" = "deny"
"**/*.pem" = "deny"
"**/*.key" = "deny"
"**/*.p12" = "deny"
"**/*.pfx" = "deny"
"**/id_rsa" = "deny"
"**/id_ed25519" = "deny"
"**/*secret*" = "deny"
"**/*credential*" = "deny"

[permissions.safe-noncoder.network]
# Sandboxed local commands receive no network access.
enabled = false

# These administrator-enforced denials apply across permission profiles and
# cannot be weakened by user configuration.
[permissions.filesystem]
deny_read = [
  "/**/.env",
  "/**/.env.*",
  "/**/.npmrc",
  "/**/.pypirc",
  "/**/.netrc",
  "/**/*.pem",
  "/**/*.key",
  "/**/*.p12",
  "/**/*.pfx",
  "/**/credentials.json",
  "/**/*service-account*.json",
  "/**/.kube/config",
  "/**/.docker/config.json",
  "~/.ssh",
  "~/.aws",
  "~/.azure",
  "~/.config/gcloud",
]

# Defense-in-depth restrictions for common destructive commands. Filesystem
# permissions remain the main enforcement boundary.
[rules]
prefix_rules = [
  { pattern = [{ token = "rm" }], decision = "forbidden", justification = "Direct file deletion is disabled for this user profile." },
  { pattern = [{ token = "rmdir" }], decision = "forbidden", justification = "Directory deletion is disabled for this user profile." },
  { pattern = [{ token = "shred" }], decision = "forbidden", justification = "Permanent file deletion is disabled." },
  { pattern = [{ token = "git" }, { token = "clean" }], decision = "forbidden", justification = "Deleting untracked files is disabled." },
  { pattern = [{ token = "git" }, { token = "reset" }, { token = "--hard" }], decision = "forbidden", justification = "Destructive repository resets are disabled." },
  { pattern = [{ token = "Remove-Item" }], decision = "forbidden", justification = "PowerShell file deletion is disabled." },
  { pattern = [{ token = "del" }], decision = "forbidden", justification = "Windows file deletion is disabled." },
]

# An empty managed MCP allowlist disables all MCP servers. Remove this table or
# add approved server identities if MCP is required later.
[mcp_servers]
```

## Optional managed-default `config.toml`

The following two values restrict Codex to ChatGPT authentication and one specific ChatGPT Edu workspace. They do not set permissions, web search, app behavior, or other Codex defaults. If a device already has a `config.toml`, paste these two top-level keys into it rather than replacing the file.

```toml
# REQUIRED: Replace this placeholder with the intended ChatGPT Edu workspace
# UUID before deployment.
forced_login_method = "chatgpt"
forced_chatgpt_workspace_id = "REPLACE_WITH_EDU_WORKSPACE_UUID"
```

`forced_login_method = "chatgpt"` prevents API-key authentication from being used instead of the managed ChatGPT login. `forced_chatgpt_workspace_id` limits accepted ChatGPT logins to the named workspace UUID. Keep both keys at the top level of `config.toml`, before the first TOML table header such as `[apps._default]`; otherwise TOML assigns them to that table instead of treating them as Codex authentication settings. These settings do not add a user to the Edu workspace, assign a seat, or replace identity-provider controls. Replace the placeholder before deployment.

## Detailed explanation

### `default_permissions = "safe-noncoder"`

This selects the custom `safe-noncoder` permission profile whenever a task starts.

What it allows:

- The capabilities explicitly opened by that profile.

What it prevents:

- Starting in a broader profile by accident.

Tradeoff:

- Every task starts with the same restrictions, including tasks that legitimately need additional access.

### `allowed_approval_policies = ["never"]`

This means Codex cannot request approval to escape the sandbox.

What it allows:

- Actions already permitted by the active permission profile.

What it prevents:

- A non-technical user accidentally approving broad filesystem or device access.
- Automatic approval reviewers authorizing a sandbox escape.

Tradeoff:

- A legitimate action outside the sandbox fails. The user cannot grant a one-time exception.

This is intentionally stricter than `approval_policy = "on-request"` with either human or automatic review.

### `allowed_web_search_modes = ["cached"]`

This permits the cached hosted search mode. The disabled mode is always implicitly permitted.

What it allows:

- Research using search data already available to the hosted search service.

What it prevents:

- Users from selecting indexed or live hosted search modes.

Tradeoff:

- Results may be stale, making the setting unsuitable for current news, prices, schedules, releases, or other time-sensitive facts.

This restriction controls hosted web search. It does not make the in-app browser cached-only.

### `allow_remote_control = false`

This disables supported device Remote Control behavior.

What it prevents:

- Operating the device through that remote-control feature.

What it does not prevent:

- Normal local use by the human user.
- Ordinary SSH remote connections.
- Browser Use, which is controlled separately.

### `allow_login_shell = false`

This prevents shell tools from starting a login shell.

Security benefit:

- Shell startup files cannot silently execute or modify the environment merely because Codex started a shell.

Tradeoff:

- Commands depending on PATH changes, environment variables, version managers, or other setup in login-shell startup files may fail.

### `allow_managed_hooks_only = true`

This tells Codex to skip user, project, session, and plugin hooks while retaining administrator-managed hooks.

Security benefit:

- A downloaded or untrusted project cannot add a hook that silently runs code.
- Plugins and local users cannot introduce additional hooks.

Tradeoff:

- Legitimate project automation, formatting, validation, or logging hooks will not run unless they are installed and managed by the administrator.

This setting is paired with `features.hooks = true`. Without a managed hook definition, no hook runs.

### `[allowed_permission_profiles]`

The table is a complete allowlist. Only `safe-noncoder` is permitted.

What it prevents:

- Selecting `:workspace`.
- Selecting `:danger-full-access`.
- Selecting the built-in `:read-only` profile as a way around the custom profile's narrower read rules.
- Selecting a user-created or future built-in profile that an administrator has not reviewed.

Tradeoff:

- Administrators must explicitly add every profile that users genuinely need.

## Feature controls

### `apps = true`

Apps and connectors may be available, subject to app availability, account permissions, and app/tool approval controls.

Important limitation:

- The local filesystem and command-network sandbox does not control apps. Apps connect through separate service-side paths.
- Sensitive apps should receive app-specific and tool-specific managed requirements rather than relying only on `_default` behavior.

### `plugins = false`

This disables plugins in supported clients.

Security benefit:

- Prevents plugin-contributed capabilities, skills, and MCP servers from expanding the environment.

Tradeoff:

- Useful organization-approved plugin functionality is unavailable until the policy is changed.

### `in_app_browser = true`

This enables the browser inside the ChatGPT/Codex application.

What it allows:

- Opening and researching live webpages in an isolated browser context.

What it does not imply:

- It does not grant local shell commands network access.
- It does not grant access to the user's normal browser profile.

### `browser_use = true`

This permits Codex to operate supported browser surfaces.

Benefit:

- Codex can navigate pages and interact with browser interfaces.

Risk:

- Browser actions can affect websites and external accounts. Filesystem permission profiles do not constrain those external actions.

### `browser_use_external = false`

This prevents Codex from controlling the user's ordinary external browser.

Security benefit:

- Protects existing tabs, cookies, extensions, history, and logged-in sessions in the normal browser.

Tradeoff:

- Codex cannot reuse those existing login sessions or extensions.

### `browser_use_full_cdp_access = false`

This disables full Chrome DevTools Protocol access.

Security benefit:

- Removes a powerful low-level browser automation and debugging path.

Tradeoff:

- Advanced web debugging, console inspection, detailed network inspection, and some automated testing may not work.

### `computer_use = false`

This disables Codex Computer Use.

Security benefit:

- Codex cannot click or type in arbitrary desktop applications.
- It cannot use graphical applications as a path around local filesystem command restrictions.

Tradeoff:

- Codex cannot operate applications that lack a dedicated connector, tool, or API.
- Record & Replay and related Computer Use workflows are unavailable.

### `fast_mode = false`

This disables the speed-oriented execution mode.

Benefit:

- Keeps the normal execution posture and avoids optimizing the experience primarily for speed.

Tradeoff:

- Some work may take longer.

This is not a security boundary.

### `hooks = true`

This enables hook processing, but `allow_managed_hooks_only = true` restricts it to administrator-managed hook sources.

Benefit:

- Administrators can add vetted validation, logging, or policy hooks.

Tradeoff:

- Managed hook scripts must be securely installed and maintained separately. The configuration does not distribute the script files.

## Filesystem profile

### `extends = ":read-only"`

The custom profile begins with the built-in read-only baseline.

Benefit:

- The profile does not start with general write access.

The additional `:root`, `:minimal`, and workspace rules make the intended read boundaries explicit.

### `":root" = "deny"`

This begins by denying reads across the filesystem.

Benefit:

- Codex does not merely receive a read-only view of the whole device.

The narrower `:minimal` and `:workspace_roots` permissions reopen only the areas needed for work.

### `":minimal" = "read"`

This lets Codex read the limited operating-system and runtime paths that common tools need.

Benefit:

- Utilities such as Git and Python can start and read their required program files.

Tradeoff:

- This is not literally zero access outside the project. Codex determines a minimal runtime set appropriate to the platform.

### `glob_scan_max_depth = 4`

Some platforms snapshot matches for recursive deny-read patterns before sandbox startup. This setting scans up to four directory levels for those matches.

Benefit:

- Improves enforcement of recursive deny patterns on Linux, WSL, and native Windows.

Tradeoff:

- Secrets deeper than four directory levels may need a larger value or explicit-depth patterns.
- Higher values can increase startup work.

### `"." = "read"`

This allows Codex to read each active workspace root.

Benefit:

- Codex can understand, summarize, and use the project files.

Risk:

- An ordinary project file is readable unless a more specific deny rule matches it.

### `"Working" = "write"`

This is the only ordinary project subtree that local Codex commands can modify.

What write access includes:

- Creating files and directories.
- Editing and replacing files.
- Renaming and moving files within the allowed area.
- Deleting files when the operating system allows it.

Operational guidance:

- Keep originals and irreplaceable files outside `Working`.
- Use backups or version history for important deliverables.
- Treat everything inside `Working` as changeable by Codex.

### `".codex" = "read"`

This makes the intended protection of project Codex configuration explicit.

Benefit:

- Codex can inspect project configuration but cannot rewrite project instructions, configuration, or hooks through local file operations.

Tradeoff:

- Codex cannot update project `.codex` configuration under this profile.

### `".git" = "read"`

This makes Git repository metadata explicitly read-only.

Benefit:

- Codex can inspect history and repository state without directly rewriting Git metadata.

Tradeoff:

- Staging, committing, merging, rebasing, branch creation, pulling, and other Git operations that modify `.git` will normally fail.

## Sensitive-file deny patterns

The workspace deny rules prevent local commands from reading or writing matching files even inside otherwise readable or writable project areas.

Protected examples include:

- `.env`, `.env.local`, and files ending in `.env`.
- `.npmrc`, `.pypirc`, and `.netrc` authentication files.
- PEM files and private-key files.
- PKCS #12 certificate bundles such as `.p12` and `.pfx`.
- Common SSH private-key names.
- Filenames containing lowercase `secret` or `credential`.

Limitations:

- Filename matching is not content inspection.
- A secret in a harmless-looking filename may remain readable.
- A harmless file whose name contains `secret` or `credential` may be blocked.
- Case differences may matter; test patterns against the files and platforms used by the organization.

## Global administrator-enforced read denials

`[permissions.filesystem].deny_read` applies beyond the custom profile and cannot be weakened by user configuration.

It protects common credential locations such as:

- SSH keys in `~/.ssh`.
- AWS credentials in `~/.aws`.
- Azure credentials in `~/.azure`.
- Google Cloud credentials in `~/.config/gcloud`.
- Kubernetes and Docker authentication files.
- Service-account JSON files.
- Package-manager authentication files.
- Private keys and certificate bundles.

This is stronger than relying only on workspace-relative deny rules. Native Windows has platform-specific enforcement limitations, so organizations should test direct file tools and shell subprocess behavior separately.

## Network controls

### `[permissions.safe-noncoder.network] enabled = false`

This gives sandboxed local commands no network access.

What it prevents:

- `curl`, `wget`, and network calls from Python or other generated programs.
- Uploading project files from a local command.
- `git fetch`, `git pull`, and `git push`.
- Package downloads such as `npm install` or `pip install` when packages are not already available locally.
- Calls to APIs, remote databases, and internet-facing test services.

What it does not control:

- Hosted web search.
- Apps and connectors.
- MCP servers.
- The built-in browser.
- Codex cloud networking.

Those surfaces require their own controls.

## Command rules

The forbidden prefix rules block common direct deletion commands:

- `rm`
- `rmdir`
- `shred`
- `git clean`
- `git reset --hard`
- PowerShell `Remove-Item`
- Windows command-shell `del`

These rules are defense in depth. They do not prove that deletion is impossible. A program can delete files through a programming-language API, graphical application, differently spelled executable path, or another command. The filesystem permission profile is the primary boundary, and `Working` remains intentionally writable and therefore changeable.

The rules also prevent legitimate cleanup. Codex cannot use the blocked commands even to remove an incorrect generated file. The human user may still perform cleanup directly outside Codex.

## Empty MCP allowlist

An empty managed `[mcp_servers]` table disables all MCP servers.

Benefit:

- No local or remote MCP server can add unreviewed tools or external data paths.

Tradeoff:

- Workflows requiring MCP do not work until administrators add approved server identities.

Apps and connectors are controlled separately; disabling MCP does not automatically disable apps.

## Expected user experience

Users should be able to:

- Ask Codex to read and summarize project files.
- Ask Codex to create new deliverables under `Working`.
- Use cached hosted web search.
- Research and interact through the isolated in-app browser.
- Use approved apps, subject to app approval behavior.

Users should expect failure when asking Codex to:

- Modify an original outside `Working`.
- Read a protected credential or private-key file.
- Download a dependency from a local command.
- Push or pull from Git.
- Control a desktop application through Computer Use.
- Use a plugin or MCP server.
- Approve a one-time escape from the sandbox.
- Run a project-supplied hook.

## Deployment checklist

1. Confirm every managed client runs Codex 0.138.0 or later.
2. Replace `REPLACE_WITH_EDU_WORKSPACE_UUID` in `config.toml` with the intended ChatGPT Edu workspace UUID.
3. Keep both authentication keys above the first `[table]` header in `config.toml`.
4. Deploy `config.toml` through MDM and restart Codex on a pilot device.
5. Confirm an account in the intended Edu workspace can sign in and start Codex.
6. Confirm a personal account or account in another ChatGPT workspace is rejected.
7. Confirm API-key sign-in is unavailable or rejected.
8. Confirm the MDM deployment persists after a device restart or normal sync cycle.
9. Remove legacy `sandbox_mode`, `sandbox_workspace_write`, and profile-level `--sandbox` selections from the deployment.
10. Deploy the hard restrictions through a supported managed `requirements.toml` source.
11. Install any managed hook scripts separately and reference them using administrator-controlled absolute paths.
12. Test on every supported operating system. Pay particular attention to native Windows filesystem enforcement and recursive glob behavior.
13. Test that Codex can read a project and write to `Working`.
14. Test that writes outside `Working` fail.
15. Test that protected credential files cannot be read.
16. Test that local command networking fails while hosted search and the in-app browser behave as intended.
17. Test app prompts and verify that destructive and open-world tools are unavailable by default.
18. Start with a small pilot group before an organization-wide rollout.

## Known limitations

- Permission profiles are beta and may change.
- File-name deny patterns cannot detect sensitive content stored under an unexpected name.
- `Working` is writable, so files inside it can be modified or deleted.
- Browser and app actions are outside the local command filesystem sandbox.
- App safety hints depend on correct tool metadata.
- Command prefix rules do not cover every equivalent command or programming-language operation.
- Cached web search may be too stale for time-sensitive questions.
- Disabling local command networking prevents many developer workflows and some document-generation tools that need downloads.
- Enabling only managed hooks requires administrators to distribute and maintain the hook scripts securely.

## Official references

- [Advanced Configuration](https://learn.chatgpt.com/docs/config-file/config-advanced)
- [Configuration Reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [Permissions](https://learn.chatgpt.com/docs/permissions)
- [Managed Configuration](https://learn.chatgpt.com/docs/enterprise/managed-configuration)

Review these sources before each major client rollout because permission profiles and managed configuration are still evolving.
