# AI Documentation Collection

A curated collection of AI and framework documentation stored as markdown files for local reference.

## Documentation Sets

| Directory | Description | Source |
|-----------|-------------|--------|
| [claude-code-docs](claude-code-docs/) | Claude Code CLI tool documentation | [claude.ai/code](https://claude.ai/code) |
| [laravel-docs](laravel-docs/) | Laravel PHP framework documentation | [laravel.com/docs](https://laravel.com/docs) |
| [symfony-ai-docs](symfony-ai-docs/) | Symfony AI components and bundles | [symfony.com/doc/current/ai](https://symfony.com/doc/current/ai/) |
| [z-ai-docs](z-ai-docs/) | Z.AI platform (GLM-4.7, GLM-4.6V, GLM-Image, CogVideoX-3) | [docs.z.ai](https://docs.z.ai) |
| [filament-docs](filament-docs/) | Filament PHP admin panel | [filamentphp.com/docs](https://filamentphp.com/docs) |

## Purpose

This repository serves as a local reference for various AI platforms and development frameworks, making documentation accessible offline and enabling AI-assisted coding tools to reference these docs.

## Structure

Each documentation set follows its own organizational structure appropriate to the source material:

```
ai-docs/
├── README.md                 # This file
├── CLAUDE.md                 # Instructions for Claude Code
├── claude-code-docs/         # Claude Code documentation
├── laravel-docs/             # Laravel framework docs
├── symfony-ai-docs/          # Symfony AI docs
├── z-ai-docs/                # Z.AI platform docs
└── filament-docs/            # Filament admin panel docs
```

## Usage

Browse documentation directly or use with AI coding assistants that can reference local files.

## Updating Documentation

To update or add new documentation:

1. Use WebFetch to retrieve pages from the official source
2. Extract content as clean markdown
3. Preserve code examples and section structure
4. Maintain consistent directory hierarchy

## License

Individual documentation sets retain their original licenses:
- Most content is under [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
- Check each directory for specific licensing information
