# Common workflows

> Step-by-step guides for exploring codebases, fixing bugs, refactoring, testing, and other everyday tasks with Claude Code.

This page covers practical workflows for everyday development: exploring unfamiliar code, debugging, refactoring, writing tests, creating PRs, and managing sessions.

## Understand new codebases

### Get a quick codebase overview

Suppose you've just joined a new project and need to understand its structure quickly.

1. Navigate to the project root directory: `cd /path/to/project`
2. Start Claude Code: `claude`
3. Ask for a high-level overview: `give me an overview of this codebase`
4. Dive deeper into specific components:
   - `explain the main architecture patterns used here`
   - `what are the key data models?`
   - `how is authentication handled?`

> **Tips**:
> * Start with broad questions, then narrow down to specific areas
> * Ask about coding conventions and patterns used in the project
> * Request a glossary of project-specific terms

### Find relevant code

Suppose you need to locate code related to a specific feature or functionality.

1. Ask Claude to find relevant files: `find the files that handle user authentication`
2. Get context on how components interact: `how do these authentication files work together?`
3. Understand the execution flow: `trace the login process from front-end to database`

## Fix bugs efficiently

Suppose you've encountered an error message and need to find and fix its source.

1. Share the error with Claude: `I'm seeing an error when I run npm test`
2. Ask for fix recommendations: `suggest a few ways to fix the @ts-ignore in user.ts`
3. Apply the fix: `update user.ts to add the null check you suggested`

## Refactor code

Suppose you need to update old code to use modern patterns and practices.

1. Identify legacy code for refactoring: `find deprecated API usage in our codebase`
2. Get refactoring recommendations: `suggest how to refactor utils.js to use modern JavaScript features`
3. Apply the changes safely: `refactor utils.js to use ES2024 features while maintaining the same behavior`
4. Verify the refactoring: `run tests for the refactored code`

## Use Plan Mode for safe code analysis

Plan Mode instructs Claude to create a plan by analyzing the codebase with read-only operations, perfect for exploring codebases, planning complex changes, or reviewing code safely.

### When to use Plan Mode

* **Multi-step implementation**: When your feature requires making edits to many files
* **Code exploration**: When you want to research the codebase thoroughly before changing anything
* **Interactive development**: When you want to iterate on the direction with Claude

### How to use Plan Mode

**Turn on Plan Mode during a session**

You can switch into Plan Mode during a session using **Shift+Tab** to cycle through permission modes.

**Start a new session in Plan Mode**

```bash
claude --permission-mode plan
```

**Run "headless" queries in Plan Mode**

```bash
claude --permission-mode plan -p "Analyze the authentication system and suggest improvements"
```

## Work with tests

Suppose you need to add tests for uncovered code.

1. Identify untested code: `find functions in NotificationsService.swift that are not covered by tests`
2. Generate test scaffolding: `add tests for the notification service`
3. Add meaningful test cases: `add test cases for edge conditions in the notification service`
4. Run and verify tests: `run the new tests and fix any failures`

## Create pull requests

You can create pull requests by asking Claude directly ("create a pr for my changes") or by using the `/commit-push-pr` skill.

```
> /commit-push-pr
```

For more control over the process, guide Claude through it step-by-step:

1. Summarize your changes: `summarize the changes I've made to the authentication module`
2. Generate a pull request: `create a pr`
3. Review and refine: `enhance the PR description with more context about the security improvements`

## Handle documentation

Suppose you need to add or update documentation for your code.

1. Identify undocumented code: `find functions without proper JSDoc comments in the auth module`
2. Generate documentation: `add JSDoc comments to the undocumented functions in auth.js`
3. Review and enhance: `improve the generated documentation with more context and examples`
4. Verify documentation: `check if the documentation follows our project standards`

## Work with images

Suppose you need to work with images in your codebase.

1. Add an image to the conversation:
   - Drag and drop an image into the Claude Code window
   - Copy an image and paste it into the CLI with ctrl+v
   - Provide an image path: "Analyze this image: /path/to/your/image.png"
2. Ask Claude to analyze the image:
   - `What does this image show?`
   - `Describe the UI elements in this screenshot`
3. Get code suggestions from visual content:
   - `Generate CSS to match this design mockup`
   - `What HTML structure would recreate this component?`

## Reference files and directories

Use @ to quickly include files or directories without waiting for Claude to read them.

- Reference a single file: `Explain the logic in @src/utils/auth.js`
- Reference a directory: `What's the structure of @src/components?`
- Reference MCP resources: `Show me the data from @github:repos/owner/repo/issues`

## Use extended thinking (thinking mode)

Extended thinking is enabled by default, reserving a portion of the output token budget for Claude to reason through complex problems step-by-step. This reasoning is visible in verbose mode, which you can toggle on with `Ctrl+O`.

### Configure thinking mode

| Scope                  | How to configure                                                                     |
| ---------------------- | ------------------------------------------------------------------------------------ |
| **Toggle shortcut**    | Press `Option+T` (macOS) or `Alt+T` (Windows/Linux)                                  |
| **Global default**     | Use `/config` to toggle thinking mode                                                |
| **Limit token budget** | Set `MAX_THINKING_TOKENS` environment variable                                       |

## Resume previous conversations

When starting Claude Code, you can resume a previous session:

* `claude --continue` continues the most recent conversation in the current directory
* `claude --resume` opens a conversation picker or resumes by name
* `claude --from-pr 123` resumes sessions linked to a specific pull request

### Name your sessions

Give sessions descriptive names to find them later:

```
> /rename auth-refactor
```

Resume by name later:

```bash
claude --resume auth-refactor
```

## Run parallel Claude Code sessions with Git worktrees

Suppose you need to work on multiple tasks simultaneously with complete code isolation.

1. Create a new worktree:
   ```bash
   git worktree add ../project-feature-a -b feature-a
   ```
2. Run Claude Code in each worktree:
   ```bash
   cd ../project-feature-a
   claude
   ```
3. Manage your worktrees:
   ```bash
   git worktree list
   git worktree remove ../project-feature-a
   ```

## Use Claude as a unix-style utility

### Add Claude to your verification process

```json
// package.json
{
    "scripts": {
        "lint:claude": "claude -p 'you are a linter. please look at the changes vs. main and report any issues related to typos.'"
    }
}
```

### Pipe in, pipe out

```bash
cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt
```

### Control output format

- Use text format (default): `cat data.txt | claude -p 'summarize this data' --output-format text > summary.txt`
- Use JSON format: `cat code.py | claude -p 'analyze this code for bugs' --output-format json > analysis.json`
- Use streaming JSON format: `cat log.txt | claude -p 'parse this log file for errors' --output-format stream-json`

## Ask Claude about its capabilities

Claude has built-in access to its documentation and can answer questions about its own features:

```
> can Claude Code create pull requests?
> how does Claude Code handle permissions?
> what skills are available?
> how do I use MCP with Claude Code?
```

## Next steps

* [Best practices](/en/best-practices) - Patterns for getting the most out of Claude Code
* [How Claude Code works](/en/how-claude-code-works) - Understand the agentic loop and context management
* [Extend Claude Code](/en/features-overview) - Add skills, hooks, MCP, subagents, and plugins
