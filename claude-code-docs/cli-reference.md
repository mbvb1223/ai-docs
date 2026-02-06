# CLI reference

> Complete reference for Claude Code command-line interface, including commands and flags.

## CLI commands

| Command                         | Description                                            | Example                                           |
| :------------------------------ | :----------------------------------------------------- | :------------------------------------------------ |
| `claude`                        | Start interactive REPL                                 | `claude`                                          |
| `claude "query"`                | Start REPL with initial prompt                         | `claude "explain this project"`                   |
| `claude -p "query"`             | Query via SDK, then exit                               | `claude -p "explain this function"`               |
| `cat file \| claude -p "query"` | Process piped content                                  | `cat logs.txt \| claude -p "explain"`             |
| `claude -c`                     | Continue most recent conversation in current directory | `claude -c`                                       |
| `claude -c -p "query"`          | Continue via SDK                                       | `claude -c -p "Check for type errors"`            |
| `claude -r "<session>" "query"` | Resume session by ID or name                           | `claude -r "auth-refactor" "Finish this PR"`      |
| `claude update`                 | Update to latest version                               | `claude update`                                   |
| `claude mcp`                    | Configure Model Context Protocol (MCP) servers         | See the MCP documentation                         |

## CLI flags

| Flag                                   | Description                                                                                                                                                                                               | Example                                                                                            |
| :------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| `--add-dir`                            | Add additional working directories for Claude to access                                                                                                                                                   | `claude --add-dir ../apps ../lib`                                                                  |
| `--agent`                              | Specify an agent for the current session                                                                                                                                                                  | `claude --agent my-custom-agent`                                                                   |
| `--agents`                             | Define custom subagents dynamically via JSON                                                                                                                                                              | `claude --agents '{"reviewer":{"description":"Reviews code","prompt":"You are a code reviewer"}}'` |
| `--allowedTools`                       | Tools that execute without prompting for permission                                                                                                                                                       | `"Bash(git log *)" "Bash(git diff *)" "Read"`                                                      |
| `--append-system-prompt`               | Append custom text to the end of the default system prompt                                                                                                                                                | `claude --append-system-prompt "Always use TypeScript"`                                            |
| `--chrome`                             | Enable Chrome browser integration                                                                                                                                                                         | `claude --chrome`                                                                                  |
| `--continue`, `-c`                     | Load the most recent conversation in the current directory                                                                                                                                                | `claude --continue`                                                                                |
| `--dangerously-skip-permissions`       | Skip all permission prompts (use with caution)                                                                                                                                                            | `claude --dangerously-skip-permissions`                                                            |
| `--debug`                              | Enable debug mode with optional category filtering                                                                                                                                                        | `claude --debug "api,mcp"`                                                                         |
| `--disallowedTools`                    | Tools that are removed from the model's context                                                                                                                                                           | `"Bash(git log *)" "Bash(git diff *)" "Edit"`                                                      |
| `--fallback-model`                     | Enable automatic fallback to specified model when default model is overloaded                                                                                                                             | `claude -p --fallback-model sonnet "query"`                                                        |
| `--fork-session`                       | When resuming, create a new session ID instead of reusing the original                                                                                                                                    | `claude --resume abc123 --fork-session`                                                            |
| `--from-pr`                            | Resume sessions linked to a specific GitHub PR                                                                                                                                                            | `claude --from-pr 123`                                                                             |
| `--ide`                                | Automatically connect to IDE on startup                                                                                                                                                                   | `claude --ide`                                                                                     |
| `--init`                               | Run initialization hooks and start interactive mode                                                                                                                                                       | `claude --init`                                                                                    |
| `--json-schema`                        | Get validated JSON output matching a JSON Schema                                                                                                                                                          | `claude -p --json-schema '{"type":"object","properties":{...}}' "query"`                           |
| `--max-budget-usd`                     | Maximum dollar amount to spend on API calls before stopping                                                                                                                                               | `claude -p --max-budget-usd 5.00 "query"`                                                          |
| `--max-turns`                          | Limit the number of agentic turns                                                                                                                                                                         | `claude -p --max-turns 3 "query"`                                                                  |
| `--mcp-config`                         | Load MCP servers from JSON files or strings                                                                                                                                                               | `claude --mcp-config ./mcp.json`                                                                   |
| `--model`                              | Sets the model for the current session                                                                                                                                                                    | `claude --model claude-sonnet-4-5-20250929`                                                        |
| `--output-format`                      | Specify output format for print mode                                                                                                                                                                      | `claude -p "query" --output-format json`                                                           |
| `--permission-mode`                    | Begin in a specified permission mode                                                                                                                                                                      | `claude --permission-mode plan`                                                                    |
| `--print`, `-p`                        | Print response without interactive mode                                                                                                                                                                   | `claude -p "query"`                                                                                |
| `--remote`                             | Create a new web session on claude.ai                                                                                                                                                                     | `claude --remote "Fix the login bug"`                                                              |
| `--resume`, `-r`                       | Resume a specific session by ID or name                                                                                                                                                                   | `claude --resume auth-refactor`                                                                    |
| `--session-id`                         | Use a specific session ID for the conversation                                                                                                                                                            | `claude --session-id "550e8400-e29b-41d4-a716-446655440000"`                                       |
| `--settings`                           | Path to a settings JSON file or a JSON string                                                                                                                                                             | `claude --settings ./settings.json`                                                                |
| `--system-prompt`                      | Replace the entire system prompt with custom text                                                                                                                                                         | `claude --system-prompt "You are a Python expert"`                                                 |
| `--teleport`                           | Resume a web session in your local terminal                                                                                                                                                               | `claude --teleport`                                                                                |
| `--tools`                              | Restrict which built-in tools Claude can use                                                                                                                                                              | `claude --tools "Bash,Edit,Read"`                                                                  |
| `--verbose`                            | Enable verbose logging                                                                                                                                                                                    | `claude --verbose`                                                                                 |
| `--version`, `-v`                      | Output the version number                                                                                                                                                                                 | `claude -v`                                                                                        |

> **Tip**: The `--output-format json` flag is particularly useful for scripting and automation.

### Agents flag format

The `--agents` flag accepts a JSON object that defines one or more custom subagents. Each subagent requires a unique name (as the key) and a definition object with the following fields:

| Field         | Required | Description                                                                                                                         |
| :------------ | :------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `description` | Yes      | Natural language description of when the subagent should be invoked                                                                 |
| `prompt`      | Yes      | The system prompt that guides the subagent's behavior                                                                               |
| `tools`       | No       | Array of specific tools the subagent can use. If omitted, inherits all tools                                                        |
| `model`       | No       | Model alias to use: `sonnet`, `opus`, `haiku`, or `inherit`. Defaults to `inherit`                                                  |

### System prompt flags

| Flag                          | Behavior                                    | Modes               | Use Case                                                             |
| :---------------------------- | :------------------------------------------ | :------------------ | :------------------------------------------------------------------- |
| `--system-prompt`             | **Replaces** entire default prompt          | Interactive + Print | Complete control over Claude's behavior                              |
| `--system-prompt-file`        | **Replaces** with file contents             | Print only          | Load prompts from files                                              |
| `--append-system-prompt`      | **Appends** to default prompt               | Interactive + Print | Add specific instructions while keeping defaults                     |
| `--append-system-prompt-file` | **Appends** file contents to default prompt | Print only          | Load additional instructions from files                              |

## See also

* [Chrome extension](/en/chrome) - Browser automation and web testing
* [Interactive mode](/en/interactive-mode) - Shortcuts, input modes, and interactive features
* [Quickstart guide](/en/quickstart) - Getting started with Claude Code
* [Common workflows](/en/common-workflows) - Advanced workflows and patterns
* [Settings](/en/settings) - Configuration options
