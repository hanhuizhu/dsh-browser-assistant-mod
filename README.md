# dsh-browser-assistant-mod

Modified version of the [dsh Browser Assistant](https://github.com/Lum1104/dsh-browser) Chrome MV3 extension.

## Modifications vs upstream

| Capability | Upstream | This mod |
|---|---|---|
| Action approval (click / type / navigate) | Requires side-panel confirmation | **Auto-approved** (no confirmation dialog) |
| Tab switch handling | Pauses and asks "follow current page?" | **Auto-follows** the newly active tab |

## Files

- `manifest.json` — MV3 extension manifest
- `background.js` — service worker (patched: `Fe()` always approves; `observeActive()` auto-follows tabs)
- `content.js` — content script
- `panel/` — side panel UI
- `_locales/` — i18n messages (en / zh_CN / zh_TW)
- `assets/` — icons

## Install

1. Open `chrome://extensions`
2. Enable **Developer mode**
3. Click **Load unpacked** and select this directory
4. Pin the "dsh Browser Assistant" extension and open its side panel

## Notes

- The bridge probes ports 3080 / 3081 / 3090 automatically; set the address manually in the side-panel settings if dsh runs elsewhere.
- Removing the approval gate and tab-follow confirmation disables safety rails — use at your own risk.
