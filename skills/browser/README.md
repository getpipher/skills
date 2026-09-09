# @getpipher/browser

Browser automation skill — drive your real Chrome via the `browser-use` CDP CLI: navigate, screenshot, coordinate-click, extract text/forms, manage tabs. Penetrates closed shadow-DOM and cross-origin iframes via compositor-level coordinate clicks where js selectors fail. For any coding agent — install as a pi package or via npx skills add.

## Skill

| Skill | Purpose |
|---|---|
| `browser` | Drive a real browser via the browser-use CDP CLI — navigate, screenshot, coordinate-click, extract, tabs |

## Install

```bash
pi install npm:@getpipher/browser
```

Or, for any other coding agent: `npx skills add getpipher/skills`

Requires:
- the `browser-use` CLI: `uv tool install browser-use`
- Chrome remote-debugging enabled: open `chrome://inspect/#remote-debugging` and tick "Allow remote debugging for this browser instance"

## License

MIT