# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A two-page static website for **El Gran Azafrán Gourmet**, a Dominican restaurant. No build system, no dependencies to install — just open the HTML files in a browser or serve with any static file server (e.g. `python3 -m http.server`).

## Architecture

### Pages
- `index.html` — customer-facing daily menu builder
- `admin.html` — password-protected admin panel for updating the menu

### State sharing via localStorage
The admin panel writes menu configuration to `localStorage` under the key `azafran_menu_v1`. The customer page reads from the same key on load. **Both pages must be opened in the same browser** for admin changes to take effect; this is the intended deployment model (restaurant owner manages the menu on their own device, customers view the page in the same browser or a hosted server won't work without a backend).

The data shape stored/read:
```js
{
  mensaje: string,        // hero welcome message
  precioPlato: string,    // base plate price in RD$
  categorias: {
    bases: Item[],        // radio, customer picks 1
    caldos: Item[],       // radio, optional
    proteinas: Item[],    // radio, customer picks 1
    guarniciones: Item[], // checkbox, max 2 (oldest is bumped when limit hit)
    especiales: EspecialItem[], // radio, priced separately — selecting one bypasses regular menu pricing
  }
}
```

`DEFAULT_MENU` is duplicated verbatim in both files — keep them in sync when adding default items.

### Order flow
1. Customer selects items → `order` object updated in memory
2. "Enviar por WhatsApp" builds a formatted message string and opens `https://wa.me/18495641828?text=...`
3. Minimum selection to enable the send button: at least one of (base, proteina) OR an especial

### Admin auth
Password is hardcoded as `ADMIN_PASS = 'azafran2025'` in `admin.html:577`. Session is tracked via `sessionStorage` (clears on tab close). To change the password, edit that constant directly.

## Known bug

`safeIcons()` in both files is self-recursive and never actually calls Lucide:
```js
function safeIcons() {
  if (typeof lucide !== 'undefined') safeIcons(); // ← should be lucide.createIcons()
}
```
Icons render only because the Lucide `<script>` tag's `onload` triggers the call after the library is available. Any call to `safeIcons()` before the CDN script loads is a no-op (correct behavior by accident). Fix by replacing with `lucide.createIcons()`.

## Key constants to know
- WhatsApp number: `WA_NUMBER = '18495641828'` (`index.html:776`) — also hardcoded in several `href` attributes throughout both files
- localStorage key: `'azafran_menu_v1'`
- Guarniciones limit: `MAX_GUARN = 2` (`index.html:923`)
- Lucide icons CDN version: `0.383.0`
