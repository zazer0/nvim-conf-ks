# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Neovim configuration based on kickstart.nvim - a minimal, single-file Neovim setup designed as a starting point for personal configuration. The configuration uses lazy.nvim for plugin management and includes custom plugins for enhanced development experience.

## Key Commands

### Plugin Management
- `:Lazy` - Open lazy.nvim plugin manager interface
- `:Lazy update` - Update all plugins
- `:Mason` - Open Mason package manager for LSP servers, formatters, and linters

### Code Formatting and Linting
- `<leader>=` - Format current buffer (using conform.nvim)
- Formatting on save is configured for Lua files using stylua
- Linting is configured via nvim-lint (currently disabled for most filetypes)

### Important Keymaps
- `<space>` - Leader key
- `<leader>q` - Open diagnostic quickfix list
- `<leader>e` - Show floating diagnostic message
- `<leader>s` - Search commands (files, grep, help, etc.)
- `<leader>c` - ChatGPT commands
- `<leader>/` - Toggle comment

### Kubernetes Development
- `:K8SSchemasGenerate` - Generate Kubernetes CRD schemas (when nvim-k8s-crd plugin is loaded)
- The configuration includes Kubernetes CRD support for YAML files

## Architecture

The repository structure follows kickstart.nvim conventions:
- `init.lua` - Main configuration file containing all core settings, keymaps, and plugin specifications
- `lua/kickstart/` - Optional kickstart plugins (debug, lint, gitsigns, etc.)
  - `plugins/` - Individual plugin configurations that can be enabled/disabled
  - `health.lua` - Health check module
- `lua/custom/` - Custom user plugins and configurations
  - `plugins/` - User-specific plugin configurations
- `lazy-lock.json` - Plugin version lock file for reproducible installations
- `.stylua.toml` - Lua formatter configuration (160 column width, 2 spaces indent)

## Development Workflow

### Adding New Plugins
1. Add plugin specifications to the lazy.nvim setup in `init.lua` or create new files in `lua/custom/plugins/`
2. Restart Neovim or run `:Lazy` to install new plugins

### Configuring Language Servers
1. LSP servers are managed through Mason and configured in the LSP section of `init.lua`
2. Use `:Mason` to install additional language servers
3. Formatters are configured in the conform.nvim section (formatters_by_ft table)

### Code Style
- Lua files use stylua formatter with 160 character line width
- Auto-format on save is enabled for Lua files
- Additional formatters can be added to the formatters_by_ft table in init.lua

## Important Notes

1. This configuration is designed to be self-documenting - extensive comments throughout init.lua explain each section
2. The configuration uses lazy loading for better startup performance
3. Kubernetes CRD plugin is configured to lazy load only when needed
4. ChatGPT integration is available with various code assistance commands under `<leader>c`