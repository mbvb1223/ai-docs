# Angular Installation Guide
> Source: https://angular.dev/installation

## Quick Start

The fastest way to begin with Angular requires no local setup -- simply visit the Angular Playground to start building immediately.

## Local Project Setup

### Prerequisites

Before starting a local Angular project, ensure you have:

- **Node.js**: Version 20.19.0 or newer
- **Text Editor**: Visual Studio Code is recommended
- **Terminal**: Required for executing Angular CLI commands
- **Development Tool**: The Angular Language Service enhances your workflow

### Installation Steps

#### 1. Install Angular CLI

Open your terminal and run the appropriate command for your package manager:

**npm:**

```bash
npm install -g @angular/cli
```

**pnpm:**

```bash
pnpm install -g @angular/cli
```

**yarn:**

```bash
yarn global add @angular/cli
```

**bun:**

```bash
bun install -g @angular/cli
```

Note: Windows or Unix users experiencing issues should consult the CLI documentation.

#### 2. Create a New Project

Execute the create command in your terminal:

```bash
ng new <project-name>
```

Replace `<project-name>` with your desired project name (e.g., `my-first-angular-app`). The CLI will present configuration options; you can press Enter to accept defaults or customize as needed.

Upon successful completion, you'll see:

```
Packages installed successfully.
Successfully initialized git.
```

#### 3. Run Your Project Locally

Navigate to your new project directory:

```bash
cd my-first-angular-app
```

Start the development server:

```bash
npm start
```

You should see output indicating:

```
Watch mode enabled. Watching for file changes...
Local: http://localhost:4200/
press h + enter to show help
```

Visit `http://localhost:4200` in your browser to view your application.

### AI-Powered Development

To leverage AI tools during development, review Angular's prompt guidelines and best practices for your preferred IDE.

## Next Steps

After creating your project, explore the Essentials guide or select specific topics from the in-depth guides to deepen your Angular knowledge.
