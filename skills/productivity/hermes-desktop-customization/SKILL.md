---
name: hermes-desktop-customization
description: "Use when customizing Hermes Desktop fonts or UI chrome."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes, desktop, customization, fonts, plugins, appearance]
    related_skills: [hermes-agent]
---

# Hermes Desktop Customization

Use this when the user asks to change Hermes Desktop UI appearance in a way the built-in skin/font pickers do not directly support, especially custom local fonts.

## Procedure: apply a custom Desktop UI font

1. **Load `hermes-agent` first for the current Hermes rules.** Treat it as authoritative for paths, profile handling, and desktop plugin constraints.
2. **Check whether the font is already built in.** Hermes Desktop has a curated dashboard font catalog; prefer the built-in picker/config path when the requested font is in that catalog.
3. **Find the local font files.** Prefer `.woff2` web font files when available; include regular plus useful weights such as light, medium, and bold.
4. **Create a standalone desktop runtime plugin** under the active Hermes home:
   ```text
   <HERMES_HOME>/desktop-plugins/<id>/plugin.js
   ```
   Keep the plugin `id` equal to the folder name.
5. **Embed custom font assets as data URLs inside `plugin.js`.** Runtime plugins are evaluated from a blob URL, so relative asset references such as `new URL('./fonts/font.woff2', import.meta.url)` can resolve against the blob instead of the plugin folder. Base64 `data:font/woff2` URLs are self-contained and survive hot reload.
6. **Inject a `<style>` tag** that defines `@font-face` and sets both:
   ```css
   :root {
     --theme-font-sans: 'FontName', system-ui, sans-serif !important;
     --theme-font-display: 'FontName', system-ui, sans-serif !important;
   }
   html, body, #root,
   body :not(code):not(pre):not(kbd):not(samp):not(.font-mono):not(.xterm):not(.xterm *) {
     font-family: 'FontName', system-ui, sans-serif !important;
   }
   ```
   Exclude code, terminal, and monospace selectors so code blocks and xterm stay readable.
7. **Verify before reporting success.** Run a JS syntax check on the written plugin:
   ```bash
   node --check <HERMES_HOME>/desktop-plugins/<id>/plugin.js
   ```
   Also verify the file exists and is non-empty. The desktop app should hot-reload the plugin; if it does not, tell the user to use Cmd/Ctrl+K → Reload desktop plugins.

## Plugin template for self-contained font injection

```js
const STYLE_ID = 'hermes-custom-font-style'

function fontData(base64) {
  return `data:font/woff2;base64,${base64}`
}

function injectCustomFont() {
  const existing = document.getElementById(STYLE_ID)
  if (existing) existing.remove()

  const regular = fontData('BASE64_WOFF2_HERE')

  const style = document.createElement('style')
  style.id = STYLE_ID
  style.textContent = `
@font-face {
  font-family: 'CustomFont';
  src: url('${regular}') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
:root {
  --theme-font-sans: 'CustomFont', system-ui, -apple-system, 'Segoe UI', sans-serif !important;
  --theme-font-display: 'CustomFont', system-ui, -apple-system, 'Segoe UI', sans-serif !important;
}
html, body, #root,
body :not(code):not(pre):not(kbd):not(samp):not(.font-mono):not(.xterm):not(.xterm *) {
  font-family: 'CustomFont', system-ui, -apple-system, 'Segoe UI', sans-serif !important;
}
`
  document.head.appendChild(style)
}

export default {
  id: 'custom-font',
  name: 'Custom Font',
  register() {
    injectCustomFont()
  },
}
```

## Pitfalls

- **Do not rely on plugin-relative font URLs in runtime plugins.** The runtime loader imports plugins from a blob URL, so relative URLs can resolve incorrectly; embed fonts as data URLs or use a verified reachable absolute URL.
- **Do not change terminal/code fonts unless asked.** The Desktop terminal uses monospace metrics and code readability depends on the monospace stack.
- **Do not hand-edit `config.yaml` for Hermes appearance.** Use supported `hermes config set` or a runtime plugin; malformed YAML can break the live gateway.
