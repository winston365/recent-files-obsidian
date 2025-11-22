# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Obsidian plugin that displays recently opened files in the sidebar. Users can filter files by path patterns, frontmatter tags, or bookmarks.

## Build Commands

```bash
npm install          # Install dependencies
npm run dev          # Development mode with watch (auto-rebuild on changes)
npm run build        # Production build (type check + bundle + minify)
npm run eslint       # Lint and auto-fix
```

## Development Setup

1. Clone/symlink this repo to `<vault>/.obsidian/plugins/recent-files-obsidian`
2. Run `npm install` then `npm run dev`
3. Enable plugin in Obsidian settings
4. Reload Obsidian (`Ctrl+R`) after code changes

## Architecture

Single-file plugin (`main.ts`) with three main classes:

- **RecentFilesPlugin** - Main plugin class handling lifecycle, data persistence, and file event handlers (open, rename, delete)
- **RecentFilesListView** - Sidebar view rendering the file list with drag/drop, hover preview, and context menu support
- **RecentFilesSettingTab** - Settings UI for path/tag filters, bookmark exclusion, update trigger, and list length

Data interface (`RecentFilesData`):
- `recentFiles`: Array of `{path, basename}`
- `omittedPaths`: Regex patterns to exclude
- `omittedTags`: Frontmatter tag patterns to exclude
- `omitBookmarks`: Boolean to exclude bookmarked files
- `updateOn`: `'file-open'` or `'file-edit'`
- `maxLength`: Max files in list (default: 50)

## Key Files

- `main.ts` - All plugin source code
- `manifest.json` - Plugin metadata (id, version, author, description)
- `styles.css` - Plugin styles
- `esbuild.config.mjs` - Build configuration

## Code Style

ESLint enforces strict TypeScript rules:
- Explicit return types on functions
- Explicit member accessibility (public/private)
- Prefer arrow functions
- Single quotes, Unix line endings
- No `any` types (use `// eslint-disable-next-line` when unavoidable for Obsidian internals)

## External Integrations

- **Front Matter Title plugin** - Optional integration via `front-matter-plugin-api-provider` to display frontmatter titles instead of filenames
- **Obsidian Bookmarks** - Internal plugin integration to filter bookmarked files
