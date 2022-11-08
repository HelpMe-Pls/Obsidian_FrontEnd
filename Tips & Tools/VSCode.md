## Shortkeys 
Ctrl + `[` or `]`: Code block indent

Alt + ↑ or ↓: Move the selected *line*/code block 

Alt + Shift + ⮜ or ⮞: Move the selected TEXT to left/right

Alt + Click: Multiplying cursors at clicked position

Ctrl + D: Select next same text of the current selected one, Ctrl + D + K to skip the current selected

Ctrl + Alt + ↑ or ↓: Multiline editing

Ctrl + / or Shift + Alt + A: comment the selected code

Shift + Alt + ↑ or ↓: Duplicate line of code based on current position

Ctrl + Shift + K: Delete whole current line 
OR 
Shift + Del (works like a shortcut of Ctrl + X for that line without having to select it & not saving it in the clipboard) 

clg + Tab: `console.log()`

`.` To open the online VSCode in a [GitHub](https://github.com) repo when you visit 

To clear DevTool's console history: Ctrl + Shift + P then type "clear" then choose it

#### Wrap a block of code inside a new scope:
- Select text (optional)
- Open command palette (usually Ctrl+Shift+P)
- Type in: Emmet: Wrap with Abbreviation then hit Enter
- Enter a tag name (or an abbreviation .className>p) then hit Enter

#### ToLower - ToUpper:
- Select text
- Ctrl + Shift + P
- Type in upper or lower on your preference
- Hit Enter

---
### Initial setups for a new project 
[1] Add this to `package.json` and you'll have to restart VSCode for it to work:
"prettier": {	
    "trailingComma": "es5",	// or "none" || "all"
    "tabWidth": 4,
    "useTabs": true,
    "semi": false,
    "singleQuote": true
 }

[2] IF decided to use yarn: `yarn set version berry`

[3] `.gitignore`:
node_modules/
.pnp.*
.yarn/*
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/sdks
!.yarn/versions

