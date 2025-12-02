# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Domo Developer Portal documentation repository. The portal is hosted with Stoplight at https://developer.domo.com/ and contains documentation for Domo's APIs, SDKs, tutorials, and guides for developers building on the Domo platform.

## Key Directory Structure

- `docs/` - All documentation content (markdown and OpenAPI YAML files)
  - `Getting-Started/` - Overview and SDK documentation
  - `API-Reference/` - Three categories of API documentation:
    - `Domo-App-APIs/` - APIs available within Domo App context
    - `Domo-APIs/` - Platform OAuth APIs (OpenAPI YAML specs)
    - `Product-APIs/` - Product APIs for UI-level operations
  - `Apps/` - App development documentation
    - `App-Framework/` - Main app framework guides and tutorials
    - `DDX-Bricks/` - Domo Bricks (no-code) documentation
    - `Pro-Code-Editor/` - Pro-code editor tutorials
  - `Automate-Actions/` - Workflows and Code Engine documentation
  - `Forms/` - Forms Documentation
  - `Connectors/` - Custom connector and data connection guides
  - `Data-Science/` - AI and data science resources
  - `Embedded-Analytics/` - Embedding Domo content guides
  - `Governance/` - Security, PDP, and administration docs
  - `Partner-Developers/` - Appstore submission guidelines
  - `Installation-Guides/` - Various installation guides
- `assets/` - Image files and other visual assets
- `templates/` - Document templates
- `toc.json` - Defines the left navigation sidebar structure in Stoplight
- `.spectral.mjs` - Stoplight linting configuration

## Documentation Types

The portal has three main documentation types with specific style requirements:

1. **Guides/How-to Articles** - Shorter, focused articles on single topics
2. **Tutorials** - Step-by-step, complete solution walkthroughs
3. **API Reference** - Three categories:
   - App Framework APIs (markdown with structured endpoints)
   - Platform APIs (OpenAPI YAML specs, testable in docs)
   - Product APIs (markdown documentation for "undocumented" APIs)

## Common Development Commands

### Domo Apps CLI Commands

The Domo Apps CLI (`ryuu` npm package) is the primary tool for Domo app development:

```bash
# Install CLI
npm install -g ryuu

# Authenticate to Domo instance
domo login

# Create new app from template
domo init

# Start local development server with live reload and data proxy
domo dev

# Publish app to Domo instance
domo publish

# Manage design owners
domo owner [add|rm|ls] email@example.com

# Release a design version for Appstore submission
domo release
```

### Working with Documentation

This repository contains only documentation files - there are no build, test, or lint commands to run. All content is markdown (`.md`) or OpenAPI YAML (`.yaml`) files rendered by Stoplight.

## Documentation Style Guidelines

### General Markdown Standards

- Use Markdown syntax `#` not HTML `<h1>`
- h1 (`#`) for article titles, h2 (`##`) and h3 (`###`) for content organization
- Use bullet points or numbered lists for steps and key points
- Include code snippets in triple backticks with syntax highlighting
- Use inline code formatting for short code references
- Hyperlink with descriptive text
- Scrub all PII, credentials, cookies, or authentication info - use `<placeholders>`

### Stoplight-Flavored Markdown

This repository uses Stoplight-flavored Markdown which supports special features. See https://docs.stoplight.io/docs/platform/b591e6d161539-stoplight-flavored-markdown-smd for reference.

### Navigation Structure

The `toc.json` file defines the entire navigation structure for the Developer Portal. When adding new documentation:
1. Create the markdown file in the appropriate `docs/` subdirectory
2. Add an entry to `toc.json` in the correct location with:
   - `type`: "item" (for pages), "group" (for sections), or "divider" (for separators)
   - `title`: Display name in navigation
   - `uri`: Relative path to the file (e.g., "/docs/Apps/Overview.md")

### API Documentation Standards

**App Framework APIs** (markdown format):
- Overview of the service with links to guides/examples
- Each endpoint includes: title, description, code example, HTTP request, request body arguments table, request body example, HTTP response example

**Platform APIs** (OpenAPI YAML):
- Full OpenAPI spec files that are testable directly in the documentation
- OAuth 2.0 authentication pattern

**Product APIs** (markdown):
- Document all endpoints referenced in Global Code Engine packages
- Provide at minimum: endpoint URL, method, authentication requirements, request/response examples

## Branch Naming Convention

Use format: `<section>/<purpose>`
- Section corresponds to Developer Portal section (e.g., `apps`, `data-science`, `api-reference`)
- Purpose describes the change (e.g., `update-jupyter-examples`, `add-workflow-api`)

## Architecture Notes

### Stoplight Integration

The repository is connected to Stoplight which:
- Renders all markdown and OpenAPI specs as interactive documentation
- Provides API testing capabilities for OpenAPI specs
- Uses `toc.json` as the source of truth for navigation
- Hosts the final portal at https://developer.domo.com/

### Domo Apps Platform

Domo Apps are custom web applications that run within Domo instances:
- Apps use standard web technologies (HTML, CSS, JavaScript)
- `manifest.json` defines app metadata, sizing, data mappings
- `domo.js` library provides APIs for data access, AppDB, Files, etc.
- Apps are published to a Domo instance where they become shareable cards
- Development workflow: `domo init` → `domo dev` (local) → `domo publish` (deploy)

### Key Domo Development Concepts

- **Custom Apps**: Full web applications built with App Framework
- **Domo Bricks**: No-code/low-code card builder
- **Pro-Code Editor**: In-browser code editor for apps
- **Code Engine**: Serverless JavaScript/Python execution for workflows
- **Workflows**: Visual automation tool that can trigger Code Engine
- **AppDB**: NoSQL database available to apps for storing data
- **Manifest**: Configuration file defining app properties and data requirements
