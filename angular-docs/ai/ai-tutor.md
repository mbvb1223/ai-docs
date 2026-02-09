# Angular AI Tutor
> Source: https://angular.dev/ai/ai-tutor

## Overview

The Angular AI Tutor provides interactive, step-by-step guidance for building a complete modern Angular application. Learners construct a "Smart Recipe Box" project while absorbing current patterns and best practices through concept explanation, code examples, and hands-on exercises.

## Getting Started

Access the tutor via the Angular MCP server:

1. Install the Angular MCP server
2. Create a new Angular project with `ng new <project-name>`
3. Navigate into your project directory
4. Use an AI-powered editor (like Gemini CLI) and enter: `launch the Angular AI tutor`

## Learning Cycle

Each topic follows this three-step pattern:

1. **Learn the Concept** -- The tutor explains a core Angular feature with generic code examples
2. **Apply Your Knowledge** -- Complete a hands-on exercise designed to encourage critical thinking
3. **Get Feedback & Support** -- The tutor reads your project files, verifies solutions, and provides hints or detailed guidance upon request

## Key Features & Commands

### Leave and Come Back
Progress is tied to your project's code. When returning, the tutor automatically analyzes files to determine your exact position and resumes seamlessly.

**Recommendation:** Use Git to commit after completing modules as personal checkpoints.

### Adjust Experience Level
Set your level to Beginner (1-3), Intermediate (4-7), or Experienced (8-10). The tutor adapts teaching style accordingly.

Example prompts:
- "Set my experience level to beginner"
- "Change my rating to 8"

### View the Full Learning Plan
Request the table of contents to see the big picture and track progress.

Example prompts:
- "Where are we?"
- "Show the table of contents"
- "Show the plan"

### Styling
The tutor applies basic styling. You are encouraged to customize with your own styling.

### Skip Current Module
Move to the next topic without completing the current exercise.

Example prompts:
- "Skip this section"
- "Auto-complete this step for me"

### Jump to Any Topic
Learn topics out of sequence. The tutor updates your project to the correct starting point.

Example prompts:
- "Take me to the forms lesson"
- "I want to learn about Route Guards now"
- "Jump to the section on Services"

## Troubleshooting

If issues arise:

1. Type "proceed" to nudge the tutor forward if stuck
2. Correct the tutor if it is mistaken about your progress
3. Ask "What should I see in my UI?" to verify expected appearance
4. Reload the browser window
5. Perform a hard browser restart to clear console errors
6. Start a fresh chat session -- the tutor will read files to find your current step

## Learning Path: Five Phases

### Phase 1: Angular Fundamentals
- Module 1: Getting Started
- Module 2: Dynamic Text with Interpolation
- Module 3: Event Listeners (`(click)`)

### Phase 2: State and Signals
- Module 4: State Management with Writable Signals (Part 1: `set`)
- Module 5: State Management with Writable Signals (Part 2: `update`)
- Module 6: Computed Signals

### Phase 3: Component Architecture
- Module 7: Template Binding (Properties & Attributes)
- Module 8: Creating & Nesting Components
- Module 9: Component Inputs with Signals
- Module 10: Styling Components
- Module 11: List Rendering with `@for`
- Module 12: Conditional Rendering with `@if`

### Phase 4: Advanced Features & Architecture
- Module 13: Two-Way Binding
- Module 14: Services & Dependency Injection (DI)
- Module 15: Basic Routing
- Module 16: Introduction to Forms
- Module 17: Intro to Angular Material

### Phase 5: Experimental Signal Forms

**Critical Note:** Signal Forms are currently an experimental feature. The API may change significantly in future Angular releases.

- Module 18: Introduction to Signal Forms
- Module 19: Submitting & Resetting
- Module 20: Validation in Signal Forms
- Module 21: Field State & Error Messages

## Automated Setup

Some modules require setup steps like creating interfaces or mock data. The tutor presents code and file instructions; you are responsible for creating and modifying files as directed before exercises begin.

## AI & Feedback

The tutor is powered by Large Language Models and can make mistakes. If you spot incorrect explanations or code examples, correct the tutor -- it will adjust accordingly. For bugs or feature requests, submit an issue on GitHub.
