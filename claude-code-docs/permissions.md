# Configure permissions

> Control what Claude Code can access and do with fine-grained permission rules, modes, and managed policies.

Claude Code supports fine-grained permissions so that you can specify exactly what the agent is allowed to do and what it cannot.

## Permission system

Claude Code uses a tiered permission system to balance power and safety:

| Tool type         | Example          | Approval required | "Yes, don't ask again" behavior               |
| :---------------- | :--------------- | :---------------- | :-------------------------------------------- |
| Read-only         | File reads, Grep | No                | N/A                                           |
| Bash commands     | Shell execution  | Yes               | Permanently per project directory and command |
| File modification | Edit/write files | Yes               | Until session end                             |

## Manage permissions

You can view and manage Claude Code's tool permissions with `/permissions`. This UI lists all permission rules and the settings.json file they are sourced from.

* **Allow** rules let Claude Code use the specified tool without manual approval.
* **Ask** rules prompt for confirmation whenever Claude Code tries to use the specified tool.
* **Deny** rules prevent Claude Code from using the specified tool.

Rules are evaluated in order: **deny -> ask -> allow**. The first matching rule wins.

## Permission modes

| Mode                | Description                                                                           |
| :------------------ | :------------------------------------------------------------------------------------ |
| `default`           | Standard behavior: prompts for permission on first use of each tool                   |
| `acceptEdits`       | Automatically accepts file edit permissions for the session                           |
| `plan`              | Plan Mode: Claude can analyze but not modify files or execute commands                |
| `dontAsk`           | Auto-denies tools unless pre-approved via `/permissions` or `permissions.allow` rules |
| `bypassPermissions` | Skips all permission prompts (requires safe environment)                              |

> **Warning**: `bypassPermissions` mode disables all permission checks. Only use this in isolated environments like containers or VMs.

## Permission rule syntax

Permission rules follow the format `Tool` or `Tool(specifier)`.

### Match all uses of a tool

| Rule       | Effect                         |
| :--------- | :----------------------------- |
| `Bash`     | Matches all Bash commands      |
| `WebFetch` | Matches all web fetch requests |
| `Read`     | Matches all file reads         |

### Use specifiers for fine-grained control

| Rule                           | Effect                                                   |
| :----------------------------- | :------------------------------------------------------- |
| `Bash(npm run build)`          | Matches the exact command `npm run build`                |
| `Read(./.env)`                 | Matches reading the `.env` file in the current directory |
| `WebFetch(domain:example.com)` | Matches fetch requests to example.com                    |

### Wildcard patterns

Bash rules support glob patterns with `*`:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Bash(git * main)",
      "Bash(* --version)",
      "Bash(* --help *)"
    ],
    "deny": [
      "Bash(git push *)"
    ]
  }
}
```

## Tool-specific permission rules

### Bash

* `Bash(npm run build)` matches the exact Bash command
* `Bash(npm run test *)` matches Bash commands starting with `npm run test`
* `Bash(npm *)` matches any command starting with `npm `

### Read and Edit

`Edit` rules apply to all built-in tools that edit files. Read and Edit rules both follow the gitignore specification with four distinct pattern types:

| Pattern            | Meaning                                | Example                          |
| ------------------ | -------------------------------------- | -------------------------------- |
| `//path`           | **Absolute** path from filesystem root | `Read(//Users/alice/secrets/**)` |
| `~/path`           | Path from **home** directory           | `Read(~/Documents/*.pdf)`        |
| `/path`            | Path **relative to settings file**     | `Edit(/src/**/*.ts)`             |
| `path` or `./path` | Path **relative to current directory** | `Read(*.env)`                    |

### WebFetch

* `WebFetch(domain:example.com)` matches fetch requests to example.com

### MCP

* `mcp__puppeteer` matches any tool provided by the `puppeteer` server
* `mcp__puppeteer__puppeteer_navigate` matches a specific MCP tool

### Task (subagents)

Use `Task(AgentName)` rules to control which subagents Claude can use:

```json
{
  "permissions": {
    "deny": ["Task(Explore)"]
  }
}
```

## Working directories

By default, Claude has access to files in the directory where it was launched. You can extend this access:

* **During startup**: use `--add-dir <path>` CLI argument
* **During session**: use `/add-dir` command
* **Persistent configuration**: add to `additionalDirectories` in settings files

## How permissions interact with sandboxing

Permissions and sandboxing are complementary security layers:

* **Permissions** control which tools Claude Code can use
* **Sandboxing** provides OS-level enforcement that restricts the Bash tool's filesystem and network access

## Managed settings

For organizations that need centralized control, administrators can deploy `managed-settings.json` files to system directories:

* **macOS**: `/Library/Application Support/ClaudeCode/managed-settings.json`
* **Linux and WSL**: `/etc/claude-code/managed-settings.json`
* **Windows**: `C:\Program Files\ClaudeCode\managed-settings.json`

### Managed-only settings

| Setting                           | Description                                                                                                                                        |
| :-------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| `disableBypassPermissionsMode`    | Set to `"disable"` to prevent `bypassPermissions` mode                                                                                             |
| `allowManagedPermissionRulesOnly` | When `true`, prevents user and project settings from defining permission rules                                                                     |
| `allowManagedHooksOnly`           | When `true`, prevents loading of user, project, and plugin hooks                                                                                   |
| `strictKnownMarketplaces`         | Controls which plugin marketplaces users can add                                                                                                   |

## Settings precedence

Permission rules follow settings precedence: managed settings have the highest precedence, followed by command line arguments, local project, shared project, and user settings.

## See also

* [Settings](/en/settings): complete configuration reference
* [Sandboxing](/en/sandboxing): OS-level filesystem and network isolation
* [Authentication](/en/authentication): set up user access
* [Security](/en/security): security safeguards and best practices
* [Hooks](/en/hooks-guide): automate workflows and extend permission evaluation
