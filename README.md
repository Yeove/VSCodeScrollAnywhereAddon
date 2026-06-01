# Scroll Anywhere for VSCode

Middle mouse **grab-and-drag** scrolling with **momentum** for VSCode
<br>Flick the editor to scroll and let it coast to a stop, the same way you scroll on a phone or tablet
<br>Works in the editor, the sidebar, panels, and other scrollable lists
<br>
<br>Inspired by the [ScrollAnywhere](https://addons.mozilla.org/firefox/addon/scroll_anywhere/) Firefox extension

## Requirements

- [VSCode](https://code.visualstudio.com/) - or VSCodium / other Code-OSS builds
- VSCode [Custom CSS and JS Loader](https://marketplace.visualstudio.com/items?itemName=be5invis.vscode-custom-css) extension

## Installation

1. **Install the loader in VSCode.**
   <br>In VSCode, open Extensions and install
   *Custom CSS and JS Loader* (`be5invis.vscode-custom-css`)

2. **Save the script** in your VSCode home folder:
   - macOS / Linux: `~/.vscode/scroll-anywhere-vscode.js`
   - Windows: `C:\Users\<you>\.vscode\extensions\scroll-anywhere-vscode.js`

3. **Point the loader at it.** Open your `settings.json`
   <br>**Command Palette `Ctrl + Shift + P` → Preferences: Open User Settings (JSON)** and add these lines of code:

   ```jsonc
   // macOS / Linux
   "vscode_custom_css.imports": [
     "file:///home/you/.vscode/scroll-anywhere-vscode.js"
   ]
   ```

   ```jsonc
   // Windows - note the forward slashes, since VSCode loads stuff as an HTML path.
   "vscode_custom_css.imports": [
     "file:///C:/Users/you/.vscode/scroll-anywhere-vscode.js"
   ]
   ```

4. **Enable addon.**
   <br>**Command Palette `Ctrl + Shift + P` → Reload Custom CSS and JS → Restart Visual Studio Code**

## Recommended VSCode settings

Set these two settings in `settings.json`:

```jsonc
"editor.smoothScrolling": false,       
"editor.mouseWheelScrollSensitivity": 1
```

## Configuration

All options live in the `CONFIG` block at the top of the script
<br>To edit them: Save, then run **Command Palette `Ctrl + Shift + P` → Reload Custom CSS and JS → Restart Visual Studio Code** to apply

| Option | Default | What it does |
| --- | --- | --- |
| `dragButton` | `1` | Mouse button to drag with: `0` left, `1` middle, `2` right |
| `dragMultiplier` | `1.0` | Drag speed. Higher number scrolls faster than the hand moves |
| `flickMultiplier` | `1.0` | Scales **momentum/flick** speed only, independent of the active drag speed |
| `dragThreshold` | `3` | Pixels of movement before a press counts as a drag vs. a click (Do not adjust this unless you have a super high DPI monitor) |
| `momentumMultiplier` | `900` | Glide length: ms of coast per unit of flick speed. Higher = longer coast |
| `minMomentumSpeed` | `0.02` | px/ms. Flicks slower than this just stop with no momentum coast |
| `flickWindowMs` | `50` | Time window used to measure release velocity. Shorter = snappier and more responsive to a late flick; longer = steadier |
| `useCoalesced` | `true` | Use high-frequency sub-frame pointer samples (`getCoalescedEvents`) for a cleaner velocity estimate on high-Hz mice. Set `false` if it misbehaves.|
| `onlyInScrollables` | `true` | Restrict dragging to `.monaco-scrollable-element` panes |

### Tuning tips

- **Want faster flicks but keep 1:1 grab?** Leave `dragMultiplier: 1.0`, raise
  `flickMultiplier`
- **Coast too long / too short?** Adjust `momentumMultiplier`. Note, coast distance
  grows faster when adjusting flick speed, since glide duration scales non-linerally
  with speed
- **Flick feels twitchy or imprecise?** Adjust `flickWindowMs` - ~30 ms is snappier,
  ~80 ms is smoother. This plugin has high-Hz mouse sampling via `getCoalescedEvents` which helps responsivness at shorter windows

## How it works

VS Code's editor (Monaco) is **virtualized**: line elements are recycled as they
scroll out of view, and setting `scrollTop` on the editor does nothing. So the
script:

1. On middle-press, resolves the **stable scroll container**
   (`.monaco-scrollable-element`) under the cursor and targets it
2. Scrolls by dispatching synthetic **`wheel` events**
   Since this works for any monaco element, this means it also works in the sidebar, panels, and lists
3. Measures release velocity over a **fixed time window** (not a fixed sample
   count), optionally fed by `getCoalescedEvents()` for full mouse resolution, so
   the flick should feel identical at any refresh rate
4. Applies **uniformly decelerated** momentum - `x(t) = v₀·t − ½·a·t²`, with
   velocity falling linearly to exactly zero

## ⚠️ This is not a VSCode Marketplace extension since it works by injecting code

VS Code's extension API doesn't expose editor mouse events or pixel-level
scrolling, so this behavior is impossible to ship as a normal `.vsix` extension
<br>Instead, this is a **userscript** that runs inside the VSCode window via the
**Custom CSS and JS Loader** extension, which injects JS into the workbench

That has real consequences you should understand before installing:

- It works by patching VSCode's core files, which is **unsupported by Microsoft**
- VSCode will show a **"Your Code installation is corrupt"** warning after you
  enable it. This is expected, dismissable, and does not mean anything is broken.
- You must **re-enable it after every VSCode update**, because updates restore
  the patched files

## Troubleshooting

- **"Installation is corrupt" warning** - expected after enabling; dismiss it
- **Script broke after a VSCode update** - Command Palette ``Ctrl + Shift + P`` →  Reload Custom CSS and JS → Restart Visual Studio Code 
- **Middle-click paste is suppressed** in the editor, since the middle button is
  the drag trigger. Change `dragButton` if you rely on middle-click paste

---


## Uninstalling

1. Command Palette → *Disable Custom CSS and JS*.
2. Remove the entry from `vscode_custom_css.imports` in `settings.json`.
3. Optionally uninstall the Custom CSS and JS Loader extension.
