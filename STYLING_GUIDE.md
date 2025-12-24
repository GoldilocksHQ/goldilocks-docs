# Goldilocks AI Design System & Mintlify Styling Guide

This document translates the Goldilocks AI design system from the frontend project into Mintlify-compatible configurations and patterns for the documentation site.

## Mintlify Architecture Overview

Mintlify uses a different styling approach than traditional CSS frameworks:
- **Configuration-based**: Colors and branding configured in `docs.json`
- **Component-based**: Uses built-in MDX components (Card, CodeGroup, Steps, etc.)
- **Theme system**: Built-in themes with limited customization
- **Custom CSS/JS**: Limited support for custom styling via `customCss` and `customJs` in `docs.json`

## Color System for Mintlify

### Brand Colors

**Primary Colors:**
- **Gold (Primary)**: `#B09A32` - Main brand color
- **Dark Blue**: `#173652` - Text/neutral color

**Supporting Colors:**
- **Success Green**: `#15803D` - For success states
- **Error Red**: `#B91C1C` - For error/destructive states
- **Medium Gold**: `#B09A32` - Same as primary (for consistency)

### Mintlify Color Configuration

In `docs.json`, configure colors as follows:

```json
{
  "colors": {
    "primary": "#B09A32",
    "light": "#B09A32",
    "dark": "#173652"
  }
}
```

**Color Property Meanings:**
- `primary`: Used for highlighted content, section headers, accents in **light mode**
- `light`: Used for highlighted content, section headers, accents in **dark mode**
- `dark`: Used for important buttons (applies to both modes)

**Optional Background Colors:**
```json
{
  "colors": {
    "primary": "#B09A32",
    "light": "#B09A32",
    "dark": "#173652",
    "background": {
      "light": "#FCFCFD",
      "dark": "#111827"
    }
  }
}
```

## Typography in Mintlify

### Mintlify's Typography System

Mintlify handles typography automatically through its theme system. You cannot directly customize fonts in `docs.json`, but you can:

1. **Use Markdown formatting** for emphasis:
   - `**bold**` for bold text
   - `_italic_` for italic text
   - Headings (`##`, `###`) for hierarchy

2. **Custom CSS** (if needed):
   - Add custom fonts via `customCss` in `docs.json`
   - Reference: Mintlify allows custom CSS files

### Typography Best Practices

**Headings:**
```mdx
## Main Section Heading
### Subsection Heading
```

**Emphasis:**
```mdx
**Important text** - Bold
_Emphasized text_ - Italic
```

**Code:**
```mdx
`inline code` - Inline code
```language
code block
``` - Code blocks
```

## Mintlify Component Patterns

### Cards

Mintlify's `<Card>` component:

```mdx
<Card
  title="Card Title"
  icon="rocket"
  href="/link"
>
  Card content description
</Card>
```

**Horizontal Card:**
```mdx
<Card
  title="Card Title"
  icon="rocket"
  href="/link"
  horizontal
>
  Card content
</Card>
```

**Card Groups:**
```mdx
<CardGroup cols={2}>
  <Card title="Card 1" icon="icon1" href="/link1">
    Content 1
  </Card>
  <Card title="Card 2" icon="icon2" href="/link2">
    Content 2
  </Card>
</CardGroup>
```

### Callout Components

**Note:**
```mdx
<Note>
  Additional helpful information
</Note>
```

**Tip:**
```mdx
<Tip>
  Best practices and pro tips
</Tip>
```

**Warning:**
```mdx
<Warning>
  Important cautions or breaking changes
</Warning>
```

**Info:**
```mdx
<Info>
  Neutral contextual information
</Info>
```

**Check:**
```mdx
<Check>
  Success confirmations
</Check>
```

### Code Components

**Single Code Block:**
```mdx
```javascript
const code = "example";
```
```

**Code Group (Multiple Languages):**
```mdx
<CodeGroup>
```javascript Node.js
const example = "Node.js";
```

```python Python
example = "Python"
```
</CodeGroup>

**Request/Response Examples:**
```mdx
<RequestExample>
```bash cURL
curl -X GET 'https://api.example.com/endpoint'
```
</RequestExample>

<ResponseExample>
```json Success
{
  "status": "success"
}
```
</ResponseExample>
```

### Structural Components

**Steps:**
```mdx
<Steps>
<Step title="Step 1">
  First step instructions
</Step>
<Step title="Step 2">
  Second step instructions
</Step>
</Steps>
```

**Tabs:**
```mdx
<Tabs>
<Tab title="macOS">
  macOS instructions
</Tab>
<Tab title="Windows">
  Windows instructions
</Tab>
</Tabs>
```

**Accordions:**
```mdx
<AccordionGroup>
<Accordion title="Section 1">
  Content for section 1
</Accordion>
<Accordion title="Section 2">
  Content for section 2
</Accordion>
</AccordionGroup>
```

**Columns:**
```mdx
<Columns cols={2}>
  <div>Column 1 content</div>
  <div>Column 2 content</div>
</Columns>
```

### API Documentation Components

**Parameter Fields:**
```mdx
<ParamField path="user_id" type="string" required>
  Unique identifier for the user
</ParamField>

<ParamField query="limit" type="integer" default="10">
  Maximum number of results
</ParamField>
```

**Response Fields:**
```mdx
<ResponseField name="user_id" type="string" required>
  Unique identifier assigned to the user
</ResponseField>

<ResponseField name="created_at" type="timestamp">
  ISO 8601 formatted timestamp
</ResponseField>
```

**Expandable Nested Fields:**
```mdx
<ResponseField name="user" type="object">
  Complete user object
  
  <Expandable title="User properties">
    <ResponseField name="profile" type="object">
      User profile information
    </ResponseField>
  </Expandable>
</ResponseField>
```

## Responsive Design in Mintlify

Mintlify handles responsiveness automatically. Components adapt to screen size:

- **Cards**: Automatically stack on mobile
- **Code blocks**: Scroll horizontally on small screens
- **Tables**: Scroll horizontally when needed
- **Navigation**: Collapses to mobile menu

**Best Practices:**
- Use `<Columns>` component for responsive layouts
- Test on mobile using `mint dev`
- Keep content concise for mobile readability

## Dark Mode in Mintlify

Mintlify automatically supports dark mode:

1. **Automatic detection**: Uses system preference by default
2. **Manual toggle**: Users can toggle via theme switcher
3. **Configuration**: Control via `modeToggle` in `docs.json`

**Mode Toggle Configuration:**
```json
{
  "modeToggle": {
    "default": "light",
    "isHidden": false
  }
}
```

**Force single mode:**
```json
{
  "modeToggle": {
    "default": "dark",
    "isHidden": true
  }
}
```

Colors configured in `docs.json` automatically adapt to light/dark mode.

## Assets Configuration

### Favicon

Configure in `docs.json`:
```json
{
  "favicon": "/favicon.ico"
}
```

**Available files:**
- `favicon.ico` - Copied from frontend project
- `favicon.svg` - SVG version (if preferred)

### Logos

Configure in `docs.json`:
```json
{
  "logo": {
    "light": "/logo/light.svg",
    "dark": "/logo/dark.svg",
    "href": "/"
  }
}
```

**Available logo files:**
- `logo/light.svg` - Light mode logo (SVG, recommended)
- `logo/dark.svg` - Dark mode logo (SVG, recommended)
- `logo/light.png` - Light mode logo (PNG fallback)
- `logo/dark.png` - Dark mode logo (PNG fallback)
- `logo/goldilocks_logo.png` - Main logo (1920x1080, for reference)

**Recommendation:** Use SVG logos for better scalability and performance.

## Complete Mintlify Configuration Example

Based on the Goldilocks AI design system, here's a recommended `docs.json` configuration:

```json
{
  "$schema": "https://mintlify.com/docs.json",
  "theme": "mint",
  "name": "Goldilocks AI",
  "colors": {
    "primary": "#B09A32",
    "light": "#B09A32",
    "dark": "#173652",
    "background": {
      "light": "#FCFCFD",
      "dark": "#111827"
    }
  },
  "favicon": "/favicon.ico",
  "logo": {
    "light": "/logo/light.svg",
    "dark": "/logo/dark.svg",
    "href": "/"
  },
  "modeToggle": {
    "default": "light",
    "isHidden": false
  }
}
```

## Custom Styling (Advanced)

### Custom CSS

Mintlify supports custom CSS files for advanced styling:

```json
{
  "customCss": "/custom.css"
}
```

**Limitations:**
- Custom CSS is limited and may not override all Mintlify styles
- Use sparingly and test thoroughly
- Prefer Mintlify components over custom styling

### Custom JavaScript

For interactive features:

```json
{
  "customJs": "/custom.js"
}
```

## Key Design Principles for Mintlify

1. **Use Mintlify Components**: Always prefer built-in components (`<Card>`, `<CodeGroup>`, etc.) over custom HTML
2. **Configure Colors in docs.json**: Use the `colors` object, not CSS
3. **Mobile-First Content**: Write content that works well on mobile (Mintlify handles layout)
4. **Dark Mode Support**: Colors automatically adapt - ensure content is readable in both modes
5. **Gold Accent**: Primary gold color (`#B09A32`) is the brand accent color
6. **Dark Blue Base**: Dark blue (`#173652`) is the primary text/neutral color
7. **Consistent Component Usage**: Use the same components consistently across pages
8. **Accessible Content**: Use semantic HTML and proper heading hierarchy

## Mintlify Component Reference

### Common Components

| Component | Usage | Example |
|-----------|-------|---------|
| `<Card>` | Feature cards, links | `<Card title="Guide" href="/guide">` |
| `<CardGroup>` | Multiple cards in grid | `<CardGroup cols={2}>` |
| `<Note>` | Supplementary info | `<Note>Helpful tip</Note>` |
| `<Tip>` | Best practices | `<Tip>Pro tip</Tip>` |
| `<Warning>` | Cautions | `<Warning>Breaking change</Warning>` |
| `<Info>` | Contextual info | `<Info>Background info</Info>` |
| `<Check>` | Success states | `<Check>Completed</Check>` |
| `<CodeGroup>` | Multi-language code | `<CodeGroup>` with multiple blocks |
| `<Steps>` | Procedures | `<Steps><Step title="...">` |
| `<Tabs>` | Alternative content | `<Tabs><Tab title="...">` |
| `<AccordionGroup>` | Collapsible sections | `<AccordionGroup><Accordion>` |
| `<Columns>` | Multi-column layout | `<Columns cols={2}>` |
| `<Frame>` | Image containers | `<Frame><img src="..." />` |

### API Documentation Components

| Component | Usage | Example |
|-----------|-------|---------|
| `<ParamField>` | API parameters | `<ParamField path="id" type="string">` |
| `<ResponseField>` | API responses | `<ResponseField name="data" type="object">` |
| `<Expandable>` | Nested fields | `<Expandable title="Properties">` |
| `<RequestExample>` | Request examples | `<RequestExample>` with code block |
| `<ResponseExample>` | Response examples | `<ResponseExample>` with code block |

## Migration Notes

### From Frontend to Mintlify

**What translates:**
- ✅ Color values (configure in `docs.json`)
- ✅ Logo assets (use SVG when possible)
- ✅ Favicon
- ✅ Content structure and hierarchy
- ✅ Design principles

**What doesn't translate:**
- ❌ Tailwind CSS classes (use Mintlify components instead)
- ❌ Custom CSS variables (limited support)
- ❌ Custom fonts (Mintlify uses its own typography system)
- ❌ Custom component styling (use Mintlify's built-in components)

**Best Approach:**
1. Configure colors in `docs.json`
2. Use Mintlify components for UI elements
3. Write content in MDX with proper structure
4. Test in both light and dark modes
5. Use custom CSS only when absolutely necessary

## Resources

- [Mintlify Documentation](https://mintlify.com/docs)
- [Mintlify Components Reference](https://mintlify.com/docs/components)
- [Mintlify Settings Guide](https://mintlify.com/docs/settings)

