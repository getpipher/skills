# @getpipher/browser

Browser automation skill for [pi](https://github.com/earendil-works/pi-coding-agent) — drive your real Chrome via the `browser-use` CDP CLI: navigate, screenshot, coordinate-click, extract text/forms, manage tabs. Penetrates closed shadow-DOM and cross-origin iframes via compositor-level coordinate clicks where js selectors fail.

## Skill

| Skill | Purpose |
|---|---|
| `browser` | Drive a real browser via the browser-use CDP CLI — navigate, screenshot, coordinate-click, extract, tabs |

## Install

```bash
pi install npm:@getpipher/browser
```

Requires:
- [pi](https://github.com/earendil-works/pi-coding-agent)
- the `browser-use` CLI: `uv tool install browser-use`
- Chrome remote-debugging enabled: open `chrome://inspect/#remote-debugging` and tick "Allow remote debugging for this browser instance"

## License

MIT