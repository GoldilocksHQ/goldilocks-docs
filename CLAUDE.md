# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Goldilocks AI Docs is a Mintlify-powered documentation site for the Goldilocks AI Developer API. It provides comprehensive API reference documentation, quickstart guides, and integration examples for developers building applications with the Goldilocks AI natural language people search and enrichment platform.

**Key Stack**: Mintlify, MDX, OpenAPI 3.1, YAML configuration

**Live Documentation**: Deployed automatically via Mintlify GitHub integration

## Directory Structure

```
goldilocks-docs/
├── docs.json              # Main Mintlify configuration (navigation, theme, API settings)
├── openapi.yaml           # OpenAPI 3.1 specification for API reference
├── index.mdx              # Documentation homepage
├── quickstart.mdx         # Getting started guide
├── development.mdx        # Local development setup
├── api-reference/         # API documentation
│   ├── introduction.mdx   # API overview
│   ├── quickstart.mdx     # API quickstart (authentication, rate limiting)
│   ├── enrichment.mdx     # Profile enrichment endpoint
│   ├── jobs.mdx           # Job streaming endpoint
│   ├── schema/            # JSON schemas (if any)
│   └── searches/          # Search endpoints
│       ├── query.mdx      # POST /searches/query
│       ├── profiles.mdx   # GET /searches/{search_id}/profiles
│       └── more-profiles.mdx  # POST /searches/{search_id}/more-profiles
├── essentials/            # Writing content guides
│   ├── settings.mdx       # Global settings reference
│   ├── navigation.mdx     # Navigation configuration
│   ├── markdown.mdx       # Markdown syntax guide
│   ├── code.mdx           # Code blocks guide
│   ├── images.mdx         # Images and embeds guide
│   └── reusable-snippets.mdx  # Snippets guide
├── ai-tools/              # AI tool integration guides
│   ├── cursor.mdx         # Cursor setup guide
│   ├── claude-code.mdx    # Claude Code setup guide
│   └── windsurf.mdx       # Windsurf setup guide
├── snippets/              # Reusable MDX snippets
│   └── snippet-intro.mdx  # Example snippet
├── images/                # Static images
├── logo/                  # Logo assets
│   ├── goldilocks_logo.png
│   ├── light.svg
│   └── dark.svg
├── favicon.ico            # Favicon
├── favicon.svg            # SVG favicon
├── STYLING_GUIDE.md       # Design system reference
└── package.json           # Dependencies (Mintlify CLI)
```

## Development Commands

### Running Locally

```bash
# Install Mintlify CLI (first time only)
npm i -g mint

# Start local preview server (hot-reload enabled)
mint dev

# Run on custom port
mint dev --port 3333

# Validate broken links
mint broken-links

# Update CLI to latest version
npm mint update
```

**Local Preview**: http://localhost:3000

### Deployment

Deployment is automatic via Mintlify GitHub integration:
- Push to default branch → Auto-deploy to production
- No manual deployment steps required

## Configuration Reference

### docs.json Structure

The main configuration file controls all aspects of the documentation site:

```json
{
  "name": "Goldilocks",              // Site name
  "theme": "willow",                 // Mintlify theme
  "colors": {
    "primary": "#B09A32",            // Gold (light mode accents)
    "light": "#B09A32",              // Gold (dark mode accents)
    "dark": "#173652"                // Dark blue (buttons)
  },
  "api": {
    "playground": { "display": "interactive" }
  },
  "navigation": { ... },             // Tab and page structure
  "logo": { ... },                   // Logo configuration
  "navbar": { ... },                 // Top navigation
  "footer": { ... }                  // Footer configuration
}
```

### OpenAPI Specification

The `openapi.yaml` file defines the complete API schema:
- **Version**: 3.1.0
- **Base URL**: `https://api.goldilocksai.app`
- **Authentication**: API key via `X-API-Key` header
- **Endpoints**: Searches, Enrichment, Jobs

API endpoint pages reference the OpenAPI spec via frontmatter:
```yaml
---
title: 'Query'
openapi: 'POST /searches/query'
---
```

## Content Guidelines

### Working Relationship
- Push back on ideas when appropriate - cite sources and explain reasoning
- ALWAYS ask for clarification rather than making assumptions
- NEVER lie, guess, or make up information

### Page Structure

Every MDX page MUST start with YAML frontmatter:

```yaml
---
title: "Clear, specific title"
description: "Concise description for SEO and navigation"
icon: "icon-name"  # Optional: Font Awesome icon
---
```

### Writing Standards

- **Voice**: Use second person ("you") for instructions
- **Tense**: Present tense for current states, future for outcomes
- **Style**: Active voice, clear and direct language
- **Prerequisites**: Always list at start of procedural content
- **Code examples**: Test all examples before publishing
- **Language tags**: Required on all code blocks
- **Internal links**: Use relative paths (e.g., `/api-reference/quickstart`)
- **Images**: Include descriptive alt text

### Content Strategy

- Document just enough for user success - not too much, not too little
- Prioritize accuracy and usability
- Make content evergreen when possible
- Search for existing information before adding new content (avoid duplication)
- Check existing patterns for consistency
- Start by making the smallest reasonable changes

## Mintlify Component Reference

### Callout Components

```mdx
<Note>Supplementary helpful information</Note>
<Tip>Best practices and expert advice</Tip>
<Warning>Important cautions or breaking changes</Warning>
<Info>Neutral contextual information</Info>
<Check>Success confirmations</Check>
```

### Code Components

```mdx
{/* Single code block with language and filename */}
```python example.py
code here
```

{/* Multi-language code group */}
<CodeGroup>
```python Python
code
```
```javascript JavaScript
code
```
</CodeGroup>

{/* API request/response examples */}
<RequestExample>
```bash cURL
curl -X POST ...
```
</RequestExample>

<ResponseExample>
```json Success
{ "status": "success" }
```
</ResponseExample>
```

### Structural Components

```mdx
{/* Step-by-step procedures */}
<Steps>
<Step title="Step 1">Instructions</Step>
<Step title="Step 2">Instructions</Step>
</Steps>

{/* Tabbed content */}
<Tabs>
<Tab title="macOS">macOS content</Tab>
<Tab title="Windows">Windows content</Tab>
</Tabs>

{/* Collapsible sections */}
<AccordionGroup>
<Accordion title="Section 1">Content</Accordion>
<Accordion title="Section 2">Content</Accordion>
</AccordionGroup>

{/* Cards for navigation */}
<Card title="Title" icon="icon" href="/link">Description</Card>

<CardGroup cols={2}>
<Card title="Card 1" icon="icon1" href="/link1">Content</Card>
<Card title="Card 2" icon="icon2" href="/link2">Content</Card>
</CardGroup>
```

### API Documentation Components

```mdx
{/* Parameter documentation */}
<ParamField path="user_id" type="string" required>
Description of the parameter
</ParamField>

<ParamField query="limit" type="integer" default="10">
Optional parameter with default
</ParamField>

{/* Response field documentation */}
<ResponseField name="data" type="object" required>
Description of the response field
</ResponseField>

{/* Nested fields */}
<ResponseField name="user" type="object">
User object
<Expandable title="User properties">
  <ResponseField name="id" type="string">User ID</ResponseField>
  <ResponseField name="name" type="string">User name</ResponseField>
</Expandable>
</ResponseField>
```

## API Documentation Guidelines

### Endpoint Pages

For OpenAPI-backed endpoints, use minimal frontmatter:

```yaml
---
title: 'Endpoint Name'
openapi: 'METHOD /path'
---
```

Mintlify auto-generates the rest from `openapi.yaml`.

### Manual API Documentation

When documenting APIs manually:
1. Document ALL parameters (required and optional)
2. Show both success AND error response examples
3. Include authentication examples
4. Specify rate limiting information
5. Cover complete request/response cycles

### Error Response Format

All API errors follow this structure:
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {}
  },
  "request_id": "uuid",
  "processing_time_ms": 0
}
```

## Git Workflow

### Branch Strategy

- **main**: Production documentation (auto-deployed)
- **feature branches**: For significant changes

### Commit Guidelines

- NEVER use `--no-verify` when committing
- Create clear, descriptive commit messages
- Test locally with `mint dev` before pushing

### Adding New Pages

1. Create the `.mdx` file in the appropriate directory
2. Add the page path to `docs.json` navigation
3. Test locally to verify navigation works
4. Push to trigger deployment

## Key Files to Reference

| File | Purpose |
|------|---------|
| `docs.json` | Main configuration (navigation, theme, API settings) |
| `openapi.yaml` | Complete API specification |
| `api-reference/quickstart.mdx` | Authentication and rate limiting documentation |
| `api-reference/introduction.mdx` | API overview and capabilities |
| `STYLING_GUIDE.md` | Design system and color reference |

## Brand Colors

| Color | Hex | Usage |
|-------|-----|-------|
| Gold (Primary) | `#B09A32` | Accents, highlights, section headers |
| Dark Blue | `#173652` | Important buttons, text |
| Light Background | `#FCFCFD` | Light mode background |
| Dark Background | `#111827` | Dark mode background |

## Common Tasks

### Adding a New API Endpoint

1. Add the endpoint to `openapi.yaml`
2. Create a new `.mdx` file in `api-reference/`
3. Use OpenAPI frontmatter reference
4. Add to `docs.json` navigation
5. Test locally

### Updating the OpenAPI Spec

1. Edit `openapi.yaml`
2. Validate YAML syntax
3. Test with `mint dev` to verify API playground works
4. Commit changes

### Adding a New Guide Page

1. Create `.mdx` file with proper frontmatter
2. Write content using Mintlify components
3. Add to appropriate navigation group in `docs.json`
4. Test locally

## Do Not

- Skip frontmatter on any MDX file
- Use absolute URLs for internal links (use relative paths)
- Include untested code examples
- Make assumptions without clarification
- Add pages without updating `docs.json` navigation
- Commit directly to main without testing locally

## Resources

- [Mintlify Documentation](https://mintlify.com/docs)
- [Mintlify Components](https://mintlify.com/docs/components)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Font Awesome Icons](https://fontawesome.com/search)
