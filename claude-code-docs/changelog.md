# Claude Code Changelog

This is the changelog for Claude Code, Anthropic's official CLI for Claude. Here are the major recent updates:

## Latest Version: 2.1.32

**Claude Opus 4.6 is now available!**

Key features and fixes:

- **Agent Teams**: Research preview of multi-agent collaboration (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`)
- **Memory System**: Claude automatically records and recalls memories as it works
- **Conversation Summarization**: "Summarize from here" option in message selector for partial conversation summarization
- **Skill Auto-loading**: Skills in `.claude/skills/` within additional directories (`--add-dir`) are now loaded automatically
- **Bug Fixes**:
  - Fixed `@` file completion showing incorrect relative paths when running from subdirectory
  - Fixed bash tool heredoc issues with JavaScript template literals like `${index + 1}`
  - Fixed Thai/Lao spacing vowels rendering in input field
  - VSCode: Fixed slash commands executing incorrectly when pressing Enter with preceding text

## Notable Recent Updates (2.1.31 - 2.1.20)

### Version 2.1.31
- Session resume hint on exit
- Full-width space input support from Japanese IME
- Fixed PDF too large errors locking up sessions
- Fixed temperature override being ignored in streaming API
- Improved system prompts to prefer dedicated tools over bash equivalents

### Version 2.1.30
- **PDF Pages Parameter**: Added `pages` parameter to Read tool (e.g., `pages: "1-5"`)
- Large PDFs (>10 pages) now return lightweight references instead of being inlined
- OAuth support for MCP servers without Dynamic Client Registration
- `/debug` command for troubleshooting
- Added token count, tool uses, and duration metrics to Task results

### Version 2.1.27
- `--from-pr` flag to resume sessions linked to specific GitHub PR
- Automatic PR linking when creating sessions via `gh pr create`
- Fixed context command colored output
- Windows: Fixed bash execution with `.bashrc` files

### Version 2.1.23
- Customizable spinner verbs via `spinnerVerbs` setting
- Fixed mTLS and proxy connectivity
- Fixed ripgrep search timeouts

### Version 2.1.20
- Arrow key history navigation in vim normal mode
- PR review status indicator in prompt footer
- Support for loading `CLAUDE.md` from additional directories
- Task deletion via `TaskUpdate` tool
- Improved task list dynamic adjustment based on terminal height

### Version 2.1.18
- **Customizable Keyboard Shortcuts**: Run `/keybindings` to configure. Learn more at https://code.claude.com/docs/en/keybindings

### Version 2.1.16
- New task management system with dependency tracking
- VSCode: Native plugin management support
- Fixed out-of-memory crashes with heavy subagent usage

## Key Environment Variables

- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` - Enable research preview agent teams
- `CLAUDE_CODE_ENABLE_TASKS=false` - Keep old task system temporarily
- `CLAUDE_CODE_TMPDIR` - Override temp directory
- `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` - Disable background task functionality
- `IS_DEMO=1` - Hide email and organization from UI
- `FORCE_AUTOUPDATE_PLUGINS=1` - Allow plugin autoupdate

## Installation & Getting Started

Claude Code requires installation through official channels:
- **Official Installation**: Run `claude install` or visit https://docs.anthropic.com/en/docs/claude-code/getting-started
- **npm**: Deprecated - use official installation instead
- **Windows Package Manager**: Supported with `winget`

## Common Features

- **Skills & Slash Commands**: Merged for simplified workflow - create in `.claude/skills/`
- **Hot Reload**: Skills automatically reload when created/modified
- **Context Management**: `/context` command shows token usage and percentage
- **Session Management**: `--resume` to continue previous conversations
- **Multi-Agent**: Fork contexts with `context: fork` in skill frontmatter
- **Keyboard Shortcuts**: Fully customizable via `/keybindings`

## Security & Permissions

- Advanced permission system with unreachable rule detection
- OAuth token management with automatic refresh
- Secure handling of sensitive data in debug logs
- Trust dialog for features requiring repo access

The changelog contains 130+ version entries documenting the evolution from version 0.2.21 through 2.1.32, tracking feature additions, bug fixes, performance improvements, and security enhancements.
