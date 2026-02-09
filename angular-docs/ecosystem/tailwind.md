# Using Tailwind CSS with Angular
> Source: https://angular.dev/guide/tailwind

## Overview

Tailwind CSS is a utility-first CSS framework that can be used to build modern websites without ever leaving your HTML. This guide walks through integrating Tailwind CSS into Angular projects.

## Automated Setup with `ng add`

The Angular CLI provides streamlined integration through the `ng add` command:

```bash
ng add tailwindcss
```

This command automatically:
- Installs `tailwindcss` and peer dependencies
- Configures the project for Tailwind CSS
- Adds the Tailwind CSS `@import` statement to styles

After running this command, you can immediately use Tailwind's utility classes in component templates.

## Manual Setup (Alternative Method)

### 1. Create an Angular Project

```bash
ng new my-project
cd my-project
```

### 2. Install Tailwind CSS

Choose your package manager:

**npm:**
```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

**yarn:**
```bash
yarn add tailwindcss @tailwindcss/postcss postcss
```

**pnpm:**
```bash
pnpm add tailwindcss @tailwindcss/postcss postcss
```

**bun:**
```bash
bun add tailwindcss @tailwindcss/postcss postcss
```

### 3. Configure PostCSS Plugins

Create a `.postcssrc.json` file in the project root:

```json
{
  "plugins": {
    "@tailwindcss/postcss": {}
  }
}
```

### 4. Import Tailwind CSS

Add an `@import` to `./src/styles.css`:

```css
@import 'tailwindcss';
```

For SCSS projects, use `./src/styles.scss`:

```scss
@use 'tailwindcss';
```

### 5. Start Using Tailwind

Run your build with `ng serve` and begin using Tailwind's utility classes:

```html
<h1 class="text-3xl font-bold underline">Hello world!</h1>
```

## Additional Resources

- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
