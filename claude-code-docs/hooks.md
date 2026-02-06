# Hooks reference

> Reference for Claude Code hook events, configuration schema, JSON input/output formats, exit codes, async hooks, prompt hooks, and MCP tool hooks.

> **Tip:** For a quickstart guide with examples, see [Automate workflows with hooks](/en/hooks-guide).

Hooks are user-defined shell commands or LLM prompts that execute automatically at specific points in Claude Code's lifecycle. Use this reference to look up event schemas, configuration options, JSON input/output formats, and advanced features like async hooks and MCP tool hooks. If you're setting up hooks for the first time, start with the [guide](/en/hooks-guide) instead.

## Hook lifecycle

Hooks fire at specific points during a Claude Code session. When an event fires and a matcher matches, Claude Code passes JSON context about the event to your hook handler. For command hooks, this arrives on stdin. Your handler can then inspect the input, take action, and optionally return a decision. Some events fire once per session, while others fire repeatedly inside the agentic loop.

| Event                | When it fires                                        |
| :------------------- | :--------------------------------------------------- |
| `SessionStart`       | When a session begins or resumes                     |
| `UserPromptSubmit`   | When you submit a prompt, before Claude processes it |
| `PreToolUse`         | Before a tool call executes. Can block it            |
| `PermissionRequest`  | When a permission dialog appears                     |
| `PostToolUse`        | After a tool call succeeds                           |
| `PostToolUseFailure` | After a tool call fails                              |
| `Notification`       | When Claude Code sends a notification                |
| `SubagentStart`      | When a subagent is spawned                           |
| `SubagentStop`       | When a subagent finishes                             |
| `Stop`               | When Claude finishes responding                      |
| `PreCompact`         | Before context compaction                            |
| `SessionEnd`         | When a session terminates                            |

## Configuration

Hooks are defined in JSON settings files. The configuration has three levels of nesting:

1. Choose a hook event to respond to, like `PreToolUse` or `Stop`
2. Add a matcher group to filter when it fires, like "only for the Bash tool"
3. Define one or more hook handlers to run when matched

### Hook locations

Where you define a hook determines its scope:

| Location                       | Scope                         | Shareable                          |
| :----------------------------- | :---------------------------- | :--------------------------------- |
| `~/.claude/settings.json`      | All your projects             | No, local to your machine          |
| `.claude/settings.json`        | Single project                | Yes, can be committed to the repo  |
| `.claude/settings.local.json`  | Single project                | No, gitignored                     |
| Managed policy settings        | Organization-wide             | Yes, admin-controlled              |
| Plugin `hooks/hooks.json`      | When plugin is enabled        | Yes, bundled with the plugin       |
| Skill or agent frontmatter     | While the component is active | Yes, defined in the component file |

### Matcher patterns

The `matcher` field is a regex string that filters when hooks fire. Use `"*"`, `""`, or omit `matcher` entirely to match all occurrences. Each event type matches on a different field:

| Event                                                                  | What the matcher filters  | Example matcher values                                                         |
| :--------------------------------------------------------------------- | :------------------------ | :----------------------------------------------------------------------------- |
| `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest` | tool name                 | `Bash`, `Edit\|Write`, `mcp__.*`                                               |
| `SessionStart`                                                         | how the session started   | `startup`, `resume`, `clear`, `compact`                                        |
| `SessionEnd`                                                           | why the session ended     | `clear`, `logout`, `prompt_input_exit`, `bypass_permissions_disabled`, `other` |
| `Notification`                                                         | notification type         | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`       |
| `SubagentStart`                                                        | agent type                | `Bash`, `Explore`, `Plan`, or custom agent names                               |
| `PreCompact`                                                           | what triggered compaction | `manual`, `auto`                                                               |
| `SubagentStop`                                                         | agent type                | same values as `SubagentStart`                                                 |
| `UserPromptSubmit`, `Stop`                                             | no matcher support        | always fires on every occurrence                                               |

### Hook handler fields

Each object in the inner `hooks` array is a hook handler. There are three types:

* **Command hooks** (`type: "command"`): run a shell command
* **Prompt hooks** (`type: "prompt"`): send a prompt to a Claude model for single-turn evaluation
* **Agent hooks** (`type: "agent"`): spawn a subagent that can use tools to verify conditions

#### Common fields

| Field           | Required | Description                                                  |
| :-------------- | :------- | :----------------------------------------------------------- |
| `type`          | yes      | `"command"`, `"prompt"`, or `"agent"`                        |
| `timeout`       | no       | Seconds before canceling. Defaults: 600 for command, 30 for prompt, 60 for agent |
| `statusMessage` | no       | Custom spinner message displayed while the hook runs         |
| `once`          | no       | If `true`, runs only once per session then is removed        |

#### Command hook fields

| Field     | Required | Description                                           |
| :-------- | :------- | :---------------------------------------------------- |
| `command` | yes      | Shell command to execute                              |
| `async`   | no       | If `true`, runs in the background without blocking    |

#### Prompt and agent hook fields

| Field    | Required | Description                                                                     |
| :------- | :------- | :------------------------------------------------------------------------------ |
| `prompt` | yes      | Prompt text to send to the model. Use `$ARGUMENTS` as a placeholder for input   |
| `model`  | no       | Model to use for evaluation. Defaults to a fast model                           |

## Hook input and output

Hooks receive JSON data via stdin and communicate results through exit codes, stdout, and stderr.

### Common input fields

All hook events receive these fields via stdin as JSON:

| Field             | Description                                |
| :---------------- | :----------------------------------------- |
| `session_id`      | Current session identifier                 |
| `transcript_path` | Path to conversation JSON                  |
| `cwd`             | Current working directory                  |
| `permission_mode` | Current permission mode                    |
| `hook_event_name` | Name of the event that fired               |

### Exit code output

* **Exit 0**: success, Claude Code parses stdout for JSON output
* **Exit 2**: blocking error, stderr is fed back to Claude as an error message
* **Any other exit code**: non-blocking error, execution continues

### JSON output

Exit codes let you allow or block, but JSON output gives you finer-grained control:

| Field            | Default | Description                                                   |
| :--------------- | :------ | :------------------------------------------------------------ |
| `continue`       | `true`  | If `false`, Claude stops processing entirely after the hook   |
| `stopReason`     | none    | Message shown to the user when `continue` is `false`          |
| `suppressOutput` | `false` | If `true`, hides stdout from verbose mode output              |
| `systemMessage`  | none    | Warning message shown to the user                             |

## Hook events

### SessionStart

Runs when Claude Code starts a new session or resumes an existing session.

### UserPromptSubmit

Runs when the user submits a prompt, before Claude processes it.

### PreToolUse

Runs after Claude creates tool parameters and before processing the tool call.

### PermissionRequest

Runs when the user is shown a permission dialog.

### PostToolUse

Runs immediately after a tool completes successfully.

### PostToolUseFailure

Runs when a tool execution fails.

### Notification

Runs when Claude Code sends notifications.

### SubagentStart

Runs when a Claude Code subagent is spawned via the Task tool.

### SubagentStop

Runs when a Claude Code subagent has finished responding.

### Stop

Runs when the main Claude Code agent has finished responding.

### PreCompact

Runs before Claude Code is about to run a compact operation.

### SessionEnd

Runs when a Claude Code session ends.

## Prompt-based hooks

Prompt-based hooks (`type: "prompt"`) use an LLM to evaluate whether to allow or block an action. The model responds with structured JSON containing a decision:

```json
{
  "ok": true | false,
  "reason": "Explanation for the decision"
}
```

## Agent-based hooks

Agent-based hooks (`type: "agent"`) spawn a subagent that can use tools like Read, Grep, and Glob to verify conditions before returning a decision.

## Run hooks in the background

Set `"async": true` to run command hooks in the background without blocking Claude. Async hooks cannot block or control Claude's behavior.

## Security considerations

> **Warning:** Hooks run with your system user's full permissions. They can modify, delete, or access any files your user account can access. Review and test all hook commands before adding them to your configuration.

### Security best practices

* **Validate and sanitize inputs**: never trust input data blindly
* **Always quote shell variables**: use `"$VAR"` not `$VAR`
* **Block path traversal**: check for `..` in file paths
* **Use absolute paths**: specify full paths for scripts
* **Skip sensitive files**: avoid `.env`, `.git/`, keys, etc.

## Debug hooks

Run `claude --debug` to see hook execution details. Toggle verbose mode with `Ctrl+O` to see hook progress in the transcript.
