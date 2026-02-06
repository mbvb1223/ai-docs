# Manage Claude's memory

> Learn how to manage Claude Code's memory across sessions with different memory locations and best practices.

Claude Code can remember your preferences across sessions, like style guidelines and common commands in your workflow.

## Determine memory type

Claude Code offers four memory locations in a hierarchical structure:

| Memory Type                | Location                                                                                                                                                        | Purpose                                             | Shared With                     |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- | ------------------------------- |
| **Managed policy**         | • macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br />• Linux: `/etc/claude-code/CLAUDE.md`<br />• Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | Organization-wide instructions managed by IT/DevOps | All users in organization       |
| **Project memory**         | `./CLAUDE.md` or `./.claude/CLAUDE.md`                                                                                                                          | Team-shared instructions for the project            | Team members via source control |
| **Project rules**          | `./.claude/rules/*.md`                                                                                                                                          | Modular, topic-specific project instructions        | Team members via source control |
| **User memory**            | `~/.claude/CLAUDE.md`                                                                                                                                           | Personal preferences for all projects               | Just you (all projects)         |
| **Project memory (local)** | `./CLAUDE.local.md`                                                                                                                                             | Personal project-specific preferences               | Just you (current project)      |

All memory files are automatically loaded into Claude Code's context when launched. Files higher in the hierarchy take precedence and are loaded first.

> **Note**: CLAUDE.local.md files are automatically added to .gitignore, making them ideal for private project-specific preferences.

## CLAUDE.md imports

CLAUDE.md files can import additional files using `@path/to/import` syntax:

```
See @README for project overview and @package.json for available npm commands for this project.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

Both relative and absolute paths are allowed. Relative paths resolve relative to the file containing the import, not the working directory.

> **Warning**: The first time Claude Code encounters external imports in a project, it shows an approval dialog listing the specific files. This is a one-time decision per project.

Imported files can recursively import additional files, with a max-depth of 5 hops.

## How Claude looks up memories

Claude Code reads memories recursively: starting in the cwd, Claude Code recurses up to (but not including) the root directory and reads any CLAUDE.md or CLAUDE.local.md files it finds.

Claude will also discover CLAUDE.md nested in subtrees under your current working directory. Instead of loading them at launch, they are only included when Claude reads files in those subtrees.

### Load memory from additional directories

The `--add-dir` flag gives Claude access to additional directories outside your main working directory. By default, CLAUDE.md files from these directories are not loaded.

To also load memory files from additional directories, set the `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` environment variable:

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared-config
```

## Directly edit memories with `/memory`

Use the `/memory` command during a session to open any memory file in your system editor.

## Set up project memory

Bootstrap a CLAUDE.md for your codebase with the following command:

```
> /init
```

> **Tips**:
> * Include frequently used commands (build, test, lint) to avoid repeated searches
> * Document code style preferences and naming conventions
> * Add important architectural patterns specific to your project

## Modular rules with `.claude/rules/`

For larger projects, you can organize instructions into multiple files using the `.claude/rules/` directory.

### Basic structure

Place markdown files in your project's `.claude/rules/` directory:

```
your-project/
├── .claude/
│   ├── CLAUDE.md           # Main project instructions
│   └── rules/
│       ├── code-style.md   # Code style guidelines
│       ├── testing.md      # Testing conventions
│       └── security.md     # Security requirements
```

All `.md` files in `.claude/rules/` are automatically loaded as project memory.

### Path-specific rules

Rules can be scoped to specific files using YAML frontmatter with the `paths` field:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All API endpoints must include input validation
- Use the standard error response format
- Include OpenAPI documentation comments
```

Rules without a `paths` field are loaded unconditionally.

### Glob patterns

The `paths` field supports standard glob patterns:

| Pattern                | Matches                                  |
| ---------------------- | ---------------------------------------- |
| `**/*.ts`              | All TypeScript files in any directory    |
| `src/**/*`             | All files under `src/` directory         |
| `*.md`                 | Markdown files in the project root       |
| `src/components/*.tsx` | React components in a specific directory |

### Subdirectories

Rules can be organized into subdirectories:

```
.claude/rules/
├── frontend/
│   ├── react.md
│   └── styles.md
├── backend/
│   ├── api.md
│   └── database.md
└── general.md
```

### Symlinks

The `.claude/rules/` directory supports symlinks, allowing you to share common rules across multiple projects:

```bash
# Symlink a shared rules directory
ln -s ~/shared-claude-rules .claude/rules/shared

# Symlink individual rule files
ln -s ~/company-standards/security.md .claude/rules/security.md
```

### User-level rules

You can create personal rules that apply to all your projects in `~/.claude/rules/`:

```
~/.claude/rules/
├── preferences.md    # Your personal coding preferences
└── workflows.md      # Your preferred workflows
```

## Organization-level memory management

Organizations can deploy centrally managed CLAUDE.md files that apply to all users.

1. Create the managed memory file at the **Managed policy** location shown in the memory types table above.
2. Deploy via your configuration management system (MDM, Group Policy, Ansible, etc.)

## Memory best practices

* **Be specific**: "Use 2-space indentation" is better than "Format code properly".
* **Use structure to organize**: Format each individual memory as a bullet point and group related memories under descriptive markdown headings.
* **Review periodically**: Update memories as your project evolves.
