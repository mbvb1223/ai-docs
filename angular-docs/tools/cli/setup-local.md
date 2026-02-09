# Local Set-up
> Source: https://angular.dev/tools/cli/setup-local

## Overview

This guide explains how to set up your environment for Angular development using the Angular CLI. It covers installing the CLI, creating an initial workspace and starter app, and running that app locally.

## Before You Start

To use Angular CLI, you should be familiar with:

- **JavaScript** - Core web language
- **HTML** - Markup structure
- **CSS** - Styling
- **Command line interface (CLI) tools** - General understanding of command shells
- **TypeScript** - Helpful but not required

## Dependencies

### Node.js Installation

To install Angular CLI locally, you must first install [Node.js](https://nodejs.org/). The Angular CLI uses Node and npm (Node Package Manager) to install and run JavaScript tools outside the browser.

**Requirements:**
- Angular requires an active LTS or maintenance LTS version of Node.js
- Refer to Angular's version compatibility guide for specific version details

[Download and install Node.js](https://nodejs.org/en/download), which includes npm automatically.

## Install the Angular CLI

Open a terminal and run one of these commands based on your package manager:

### npm

```bash
npm install -g @angular/cli
```

### pnpm

```bash
pnpm install -g @angular/cli
```

### yarn

```bash
yarn global add @angular/cli
```

### bun

```bash
bun install -g @angular/cli
```

### Windows PowerShell Setup

On Windows client computers, PowerShell script execution is disabled by default, which may cause installation to fail. Set the execution policy:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Read the displayed message carefully and understand the implications before proceeding.

### Unix/Linux Permissions

On Unix-like systems, global scripts may require root ownership. Use `sudo`:

```bash
sudo npm install -g @angular/cli
```

Or with other package managers:

```bash
sudo pnpm install -g @angular/cli
sudo yarn global add @angular/cli
sudo bun install -g @angular/cli
```

Understand the security implications of running commands as root.

## Create a Workspace and Initial Application

Angular development occurs within a **workspace** context.

To create a new workspace and starter app, execute:

```bash
ng new my-app
```

The CLI will:
- Install necessary Angular npm packages and dependencies (takes several minutes)
- Create a new workspace folder
- Generate a small welcome app ready to run
- Prompt you about features to include

Navigate to your new workspace:

```bash
cd my-app
```

## Run the Application

The Angular CLI includes a built-in development server for local development.

Execute:

```bash
ng serve --open
```

**What this command does:**
- Launches the development server
- Watches your files for changes
- Rebuilds the app automatically
- Reloads the browser when you modify files

The `--open` (or `-o`) flag automatically opens your browser to `http://localhost:4200/` to display your generated application.

## Workspaces and Project Files

### Workspace Structure

The `ng new` command creates an Angular workspace folder and generates a new application inside it.

**Key points:**
- A workspace can contain multiple applications and libraries
- Initial applications are created at the root directory
- Additional applications or libraries go into a `projects/` subfolder by default
- Newly generated apps contain source files for a root component and template
- Each application has a `src` folder containing components, data, and assets

### Editing and Generating Files

You can:
- Edit generated files directly
- Add or modify files using CLI commands
- Use `ng generate` to add new components, directives, pipes, services, and more
- Use `ng add` and `ng generate` commands only within a workspace
- Use `ng new` only outside an existing workspace (it creates a new one)

## Next Steps

After setup, consider:

1. **Learn your workspace** - Study the file structure and configuration options

2. **Test your app** - Run `ng test`

3. **Generate boilerplate** - Use `ng generate` for components, directives, and pipes

4. **Deploy** - Make your app available to users with `ng deploy`

5. **End-to-end testing** - Set up and run tests with `ng e2e`

## Alternative: Try Angular Online

If you're new to Angular, consider starting with the interactive "Try it now!" tutorial on StackBlitz. This introduces Angular essentials in your browser without requiring local setup.
