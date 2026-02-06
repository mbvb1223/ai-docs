# Automate workflows with hooks

> Run shell commands automatically when Claude Code edits files, finishes tasks, or needs input. Format code, send notifications, validate commands, and enforce project rules.

Hooks are user-defined shell commands that execute at specific points in Claude Code's lifecycle. They provide deterministic control over Claude Code's behavior, ensuring certain actions always happen rather than relying on the LLM to choose to run them. Use hooks to enforce project rules, automate repetitive tasks, and integrate Claude Code with your existing tools.

For decisions that require judgment rather than deterministic rules, you can also use [prompt-based hooks](#prompt-based-hooks) or [agent-based hooks](#agent-based-hooks) that use a Claude model to evaluate conditions.

> **Tip:** This guide covers common use cases and how to get started. For full event schemas, JSON input/output formats, and advanced features like async hooks and MCP tool hooks, see the [Hooks reference](/en/hooks).

## Set up your first hook

The fastest way to create a hook is through the `/hooks` interactive menu in Claude Code. This walkthrough creates a desktop notification hook, so you get alerted whenever Claude is waiting for your input instead of watching the terminal.

**Steps:**

1. **Open the hooks menu**: Type `/hooks` in the Claude Code CLI. You'll see a list of all available hook events, plus an option to disable all hooks. Select `Notification` to create a hook that fires when Claude needs your attention.

2. **Configure the matcher**: Set the matcher to `*` to fire on all notification types.

3. **Add your command**: Select `+ Add new hook…`. Copy the command for your OS:

   **macOS:**
   ```
   osascript -e 'display notification "Claude Code needs your attention" with title "Claude Code"'
   ```

   **Linux:**
   ```
   notify-send 'Claude Code' 'Claude Code needs your attention'
   ```

4. **Choose a storage location**: Select `User settings` to store it in `~/.claude/settings.json`.

5. **Test the hook**: Press `Esc` to return to the CLI. Ask Claude to do something that requires permission, then switch away from the terminal. You should receive a desktop notification.

## What you can automate

Hooks let you run code at key points in Claude Code's lifecycle: format files after edits, block commands before they execute, send notifications when Claude needs input, inject context at session start, and more.

### Get notified when Claude needs input

Get a desktop notification whenever Claude finishes working and needs your input:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### Auto-format code after edits

Automatically run Prettier on every file Claude edits:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

### Block edits to protected files

Prevent Claude from modifying sensitive files like `.env`, `package-lock.json`, or anything in `.git/`:

```bash
#!/bin/bash
# .claude/hooks/protect-files.sh

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
    exit 2
  fi
done

exit 0
```

### Re-inject context after compaction

When Claude's context window fills up, compaction summarizes the conversation. Use a `SessionStart` hook with a `compact` matcher to re-inject critical context:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Reminder: use Bun, not npm. Run bun test before committing.'"
          }
        ]
      }
    ]
  }
}
```

## How hooks work

Hook events fire at specific lifecycle points in Claude Code:

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

### Read input and return output

Hooks communicate through stdin, stdout, stderr, and exit codes. When an event fires, Claude Code passes event-specific data as JSON to your script's stdin.

#### Exit codes

* **Exit 0**: success, action proceeds
* **Exit 2**: blocking error, action is prevented
* **Any other exit code**: non-blocking error, execution continues

### Configure hook location

| Location                       | Scope               | Shareable                          |
| :----------------------------- | :------------------ | :--------------------------------- |
| `~/.claude/settings.json`      | All your projects   | No, local to your machine          |
| `.claude/settings.json`        | Single project      | Yes, can be committed to the repo  |
| `.claude/settings.local.json`  | Single project      | No, gitignored                     |

## Prompt-based hooks

For decisions that require judgment rather than deterministic rules, use `type: "prompt"` hooks:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if all tasks are complete. If not, respond with {\"ok\": false, \"reason\": \"what remains\"}."
          }
        ]
      }
    ]
  }
}
```

## Agent-based hooks

When verification requires inspecting files or running commands, use `type: "agent"` hooks:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify that all unit tests pass. Run the test suite and check the results.",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

## Limitations and troubleshooting

### Limitations

* Hooks communicate through stdout, stderr, and exit codes only
* Hook timeout is 10 minutes by default
* `PostToolUse` hooks cannot undo actions since the tool has already executed
* `Stop` hooks fire whenever Claude finishes responding, not only at task completion

### Hook not firing

* Run `/hooks` and confirm the hook appears under the correct event
* Check that the matcher pattern matches the tool name exactly
* Verify you're triggering the right event type

### Stop hook runs forever

Check the `stop_hook_active` field from the JSON input and exit early if it's `true`:

```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Allow Claude to stop
fi
# ... rest of your hook logic
```

### Debug techniques

Toggle verbose mode with `Ctrl+O` to see hook output in the transcript, or run `claude --debug` for full execution details.

## Learn more

* [Hooks reference](/en/hooks): full event schemas, JSON output format, async hooks, and MCP tool hooks
* [Security considerations](/en/hooks#security-considerations): review before deploying hooks in shared environments
