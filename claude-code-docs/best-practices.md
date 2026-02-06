# Best Practices for Claude Code

> Tips and patterns for getting the most out of Claude Code, from configuring your environment to scaling across parallel sessions.

Claude Code is an agentic coding environment. Unlike a chatbot that answers questions and waits, Claude Code can read your files, run commands, make changes, and autonomously work through problems while you watch, redirect, or step away entirely.

Most best practices are based on one constraint: Claude's context window fills up fast, and performance degrades as it fills.

## Give Claude a way to verify its work

> Include tests, screenshots, or expected outputs so Claude can check itself. This is the single highest-leverage thing you can do.

Claude performs dramatically better when it can verify its own work, like run tests, compare screenshots, and validate outputs.

| Strategy                              | Before                                                  | After                                                                                                                                                                                                   |
| ------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Provide verification criteria**     | *"implement a function that validates email addresses"* | *"write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing"* |
| **Verify UI changes visually**        | *"make the dashboard look better"*                      | *"\[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them"*                                                            |
| **Address root causes, not symptoms** | *"the build is failing"*                                | *"the build fails with this error: \[paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error"*                                                             |

## Explore first, then plan, then code

> Separate research and planning from implementation to avoid solving the wrong problem.

Use Plan Mode to separate exploration from execution. The recommended workflow has four phases:

1. **Explore**: Enter Plan Mode. Claude reads files and answers questions without making changes.
2. **Plan**: Ask Claude to create a detailed implementation plan.
3. **Implement**: Switch back to Normal Mode and let Claude code, verifying against its plan.
4. **Commit**: Ask Claude to commit with a descriptive message and create a PR.

## Provide specific context in your prompts

> The more precise your instructions, the fewer corrections you'll need.

| Strategy                                                                                         | Before                                               | After                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------ | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scope the task.** Specify which file, what scenario, and testing preferences.                  | *"add tests for foo.py"*                             | *"write a test for foo.py covering the edge case where the user is logged out. avoid mocks."*                                                                                                                                                                                                                                                                    |
| **Point to sources.** Direct Claude to the source that can answer a question.                    | *"why does ExecutionFactory have such a weird api?"* | *"look through ExecutionFactory's git history and summarize how its api came to be"*                                                                                                                                                                                                                                                                             |
| **Reference existing patterns.** Point Claude to patterns in your codebase.                      | *"add a calendar widget"*                            | *"look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. follow the pattern to implement a new calendar widget."* |
| **Describe the symptom.** Provide the symptom, the likely location, and what "fixed" looks like. | *"fix the login bug"*                                | *"users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it"*                                                                                                                                                                                 |

### Provide rich content

You can provide rich data to Claude in several ways:

* **Reference files with `@`** instead of describing where code lives
* **Paste images directly**. Copy/paste or drag and drop images into the prompt
* **Give URLs** for documentation and API references
* **Pipe in data** by running `cat error.log | claude`
* **Let Claude fetch what it needs**

## Configure your environment

### Write an effective CLAUDE.md

Run `/init` to generate a starter CLAUDE.md file based on your current project structure.

CLAUDE.md is a special file that Claude reads at the start of every conversation. Include Bash commands, code style, and workflow rules. Keep it short and human-readable.

| ✅ Include                                            | ❌ Exclude                                          |
| ---------------------------------------------------- | -------------------------------------------------- |
| Bash commands Claude can't guess                     | Anything Claude can figure out by reading code     |
| Code style rules that differ from defaults           | Standard language conventions Claude already knows |
| Testing instructions and preferred test runners      | Detailed API documentation (link to docs instead)  |
| Repository etiquette (branch naming, PR conventions) | Information that changes frequently                |
| Architectural decisions specific to your project     | Long explanations or tutorials                     |
| Developer environment quirks (required env vars)     | File-by-file descriptions of the codebase          |
| Common gotchas or non-obvious behaviors              | Self-evident practices like "write clean code"     |

### Configure permissions

Use `/permissions` to allowlist safe commands or `/sandbox` for OS-level isolation.

### Use CLI tools

Tell Claude Code to use CLI tools like `gh`, `aws`, `gcloud`, and `sentry-cli` when interacting with external services.

### Connect MCP servers

With MCP servers, you can ask Claude to implement features from issue trackers, query databases, analyze monitoring data, integrate designs from Figma, and automate workflows.

### Set up hooks

Hooks run scripts automatically at specific points in Claude's workflow. Unlike CLAUDE.md instructions which are advisory, hooks are deterministic.

### Create skills

Skills extend Claude's knowledge with information specific to your project, team, or domain.

### Create custom subagents

Subagents run in their own context with their own set of allowed tools.

## Communicate effectively

### Ask codebase questions

When onboarding to a new codebase, use Claude Code for learning and exploration:

* How does logging work?
* How do I make a new API endpoint?
* What does `async move { ... }` do on line 134 of `foo.rs`?

### Let Claude interview you

For larger features, have Claude interview you first:

```
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.
```

## Manage your session

### Course-correct early and often

Correct Claude as soon as you notice it going off track.

* **`Esc`**: Stop Claude mid-action
* **`Esc + Esc` or `/rewind`**: Open the rewind menu
* **`"Undo that"`**: Have Claude revert its changes
* **`/clear`**: Reset context between unrelated tasks

### Manage context aggressively

Run `/clear` between unrelated tasks to reset context.

### Use subagents for investigation

Delegate research with `"use subagents to investigate X"`. They explore in a separate context, keeping your main conversation clean.

### Rewind with checkpoints

Every action Claude makes creates a checkpoint. Double-tap `Escape` or run `/rewind` to open the checkpoint menu.

### Resume conversations

Run `claude --continue` to pick up where you left off, or `--resume` to choose from recent sessions.

## Automate and scale

### Run headless mode

Use `claude -p "prompt"` in CI, pre-commit hooks, or scripts.

### Run multiple Claude sessions

Run multiple Claude sessions in parallel using [Claude Desktop](/en/desktop) or [Claude Code on the web](/en/claude-code-on-the-web).

### Fan out across files

Loop through tasks calling `claude -p` for each.

## Avoid common failure patterns

* **The kitchen sink session.** Context is full of irrelevant information. **Fix**: `/clear` between unrelated tasks.
* **Correcting over and over.** Context is polluted with failed approaches. **Fix**: After two failed corrections, `/clear` and write a better initial prompt.
* **The over-specified CLAUDE.md.** If your CLAUDE.md is too long, Claude ignores half of it. **Fix**: Ruthlessly prune.
* **The trust-then-verify gap.** Claude produces plausible-looking implementation that doesn't work. **Fix**: Always provide verification.
* **The infinite exploration.** Claude reads hundreds of files, filling the context. **Fix**: Scope investigations narrowly or use subagents.

## Related resources

* [How Claude Code works](/en/how-claude-code-works) - Understand the agentic loop, tools, and context management
* [Extend Claude Code](/en/features-overview) - Choose between skills, hooks, MCP, subagents, and plugins
* [Common workflows](/en/common-workflows) - Step-by-step recipes for debugging, testing, PRs, and more
* [CLAUDE.md](/en/memory) - Store project conventions and persistent context
