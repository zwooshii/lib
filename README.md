# fzzr UI

A Drawing-API menu library for the **Matcha LuaVM** (Roblox). Sidebar pages, boxes, switches, sliders, dropdowns,
keybinds, color pickers, toasts, themes, search, config saving, tooltips and more. It uses no Matcha menu API
(`UI.AddTab`, `Section`, `Toggle`, …), so it never clashes with Matcha's own menu.

* Lightweight: every drawing object is created once and only updated when something changes.
* Built-in **Common tab** (Settings → Configs / Display / Notifications, and Players) that is always added last.
* Everything is saved and loaded with configs, including colors and keybinds.

---

## 1. Install

1. Create a GitHub repo and upload `fzzr.lua` to it.
2. In your script, load it with `loadstring`. Matcha drops the return values of `loadstring`, so the library is exported as `_G.Fzzr`:

```lua
loadstring(httpget("https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPO/main/fzzr.lua"))()
local Fzzr = _G.Fzzr
```

> Use `httpget`, not `game:HttpGet`. Matcha does not hook `game:HttpGet`.

## 2. Quick start

```lua
loadstring(httpget("https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPO/main/fzzr.lua"))()
local Fzzr = _G.Fzzr

local Window = Fzzr:CreateWindow({ Title = "My Hub", Version = "1.0.0" })

local Combat = Window:AddPage("Combat", { category = "Weapon", icon = "crosshair" })
local Aim    = Combat:AddBox("Aimbot", "Left")

Aim:AddSwitch("aim_on", "Enable Aimbot", false, {
    bold = true,
    tip  = "Pulls your aim toward the nearest player.",
    binds = { { key = 0x02, mode = "Hold", show = true } },   -- right mouse, hold
    callback = function(on) print("aimbot", on) end,
})
Aim:AddSlider("aim_fov", "FOV", 1, 180, 40, { suffix = "°" })
Aim:AddChoice("aim_bone", "Hitbox", { "Head", "Torso", "Nearest" }, 3)

Window:Init()   -- always call this LAST

-- reading values later (for example inside your own loop)
if Window:IsActive("aim_on") then
    local fov  = Window:Get("aim_fov")
    local bone = Window:Get("aim_bone")     -- "Head" / "Torso" / "Nearest"
end
```

`example.lua` in this repo is a full script that uses most of the library.

---

## 3. Window

### `Fzzr:CreateWindow(options)` → `Window`

| Option | Type | Default | What it does |
|---|---|---|---|
| `Title` | string | `"fzzr"` | Name used in toasts and the watermark. |
| `Version` | string | `"1.0.0"` | Version text under the logo. |
| `Position` | `{x, y}` | `{60, 40}` | Where the window starts. |
| `LogoUrl` | string | none | Direct link to a PNG logo. Without it the library looks for `fzzr.png` in the icon folder. |
| `IconBase` | string | `zwooshii/lib/assets/flaticons/` | Folder URL (ending in `/`) that holds the sidebar icon PNGs. |
| `TagStyle` | table | see below | Look of the header pills. |
| `Tags` | list | none | Header pills: `{ { "BETA", Color3 }, ... }`. |

### Window methods

| Method | Description |
|---|---|
| `Window:AddPage(name, opts)` | Adds a sidebar page. See [Pages](#4-pages). |
| `Window:Init()` | Builds the Common tab, loads configs and opens the menu. **Call it last.** |
| `Window:Get(id)` | Value of any widget (see the table in [Widgets](#6-widgets)). |
| `Window:Set(id, value)` | Sets a widget; its `callback` runs. |
| `Window:GetColor(id)` | `Color3, alpha` of a switch created with `color = ...`. |
| `Window:IsActive(id)` | `true` while a switch is on **and** its bind (if any) is active. Use this in your feature loops. |
| `Window:Select(page)` | Opens a page. |
| `Window:Toast(opts)` | Shows a notification. See [Toasts](#7-toasts). |
| `Window:Copy(text, title)` | Copies to the clipboard and shows a toast. |
| `Window:AddTag(text, color)` | Adds a header pill (color: `Color3`). |
| `Window:Destroy()` | Removes the menu and all its drawings. |
| `Window.dirty` | `true` while there are unsaved config changes. |
| `Window.cfgName` | Name of the loaded config, or `nil`. |

Config helpers (they back the Configs page, so you rarely need them): `CfgSave(name)`, `CfgLoad(name)`,
`CfgDelete(name)`, `CfgSetAutoload(name)`, `CfgSelected()`, `CfgNames()`. Save/load/delete return `ok, err`.

### Header pills (build tags)

```lua
local TAG_TYPES = {
    Beta = { text = "BETA", color = Color3.fromRGB(255, 159, 64) },
    Dev  = { text = "DEV",  color = Color3.fromRGB(255, 120, 170) },
    Priv = { text = "PRIV", color = Color3.fromRGB(255, 77, 77) },
}
local BUILD_TAGS = { "Dev" }                  -- which ones this build shows
for _, n in ipairs(BUILD_TAGS) do Window:AddTag(TAG_TYPES[n].text, TAG_TYPES[n].color) end
```

`TagStyle` keys (all optional): `font` (`true` = bold), `faux` (draw twice for extra weight), `size`, `widthScale`,
`padX`, `dotGap`, `height`, `textDX`, `textDY` (nudge the text in pixels). FPS, ping and the config chip use the same style.

---

## 4. Pages

```lua
local page = Window:AddPage("Name", { category = "Weapon", icon = "crosshair" })
local sub  = Window:AddPage("Child", { parent = "Name", icon = "grid" })
```

| Option | Description |
|---|---|
| `category` | Sidebar heading. Pages with the same category are grouped. Categories appear in creation order, then **Common**. |
| `parent` | Name of an existing page. The page becomes a child shown when the parent is opened. |
| `landing` | `true` on a child to make it the one opened when the parent is clicked (default: first child). |
| `icon` | `crosshair`, `brush`, `mountain`, `blade`, `bolt`, `gear`, `user`, `globe`, `file`, `monitor`, `bell`, `grid`, `search`, `info`, `check`, `plus`, `x`, `trash`, `download`. |

Page names must be unique. The names `Settings`, `Configs`, `Display`, `Notifications` and `Players` (top level) belong to the Common tab.

### Boxes

```lua
local box = page:AddBox("Title", "Left")      -- "Left" or "Right" column
```

Boxes stack top to bottom inside their column.

---

## 5. Common options for widgets

Every widget takes `id` (unique string; used for `Get`, `Set`, config saving and search), `label`, and an `opts` table.
`opts.tip` shows a tooltip after hovering the row for half a second. `opts.callback` runs when the value changes.

## 6. Widgets

| Widget | `Window:Get(id)` returns | Callback receives |
|---|---|---|
| `AddSwitch` | `true` / `false` | the new boolean |
| `AddSlider` | number | the new number |
| `AddChoice` | the selected **text** | `index, text` |
| `AddMulti` | table `{ [name] = true, ... }` | that table |
| `AddKeyPill` | virtual-key code | the key code |
| `AddEntry` | the typed string | – |
| `AddTabs` | selected index | `index, text` |
| `AddChips` / chip | boolean per chip id | boolean |
| `AddPriority` | list of names in order | the list |
| `AddThemeList` | theme name | theme name |

### `box:AddSwitch(id, label, default, opts)`

| Option | Description |
|---|---|
| `bold` | Bold label (use for main switches). |
| `dim` | Dimmed label (secondary options). |
| `bind = true` | Adds a keybind button with the default settings. |
| `binds = {...}` | Preset binds: `{ { key = 0x02, mode = "Hold", show = true } }`. `mode` is `"Toggle"`, `"Hold"` or `"Off"`; `show = false` hides it from the bind list. Key codes: [virtual-key codes](https://learn.microsoft.com/en-us/windows/win32/inputdev/virtual-key-codes). |
| `color = Color3` | Adds a color swatch (alpha, hex, rainbow and pulse animation). Read with `Window:GetColor(id)`. |
| `more = { title, width }` | Adds a "…" button that opens a popup. Fill it with `switch.sub:AddSlider(...)` etc. |

```lua
local esp = box:AddSwitch("esp_box", "Box", true, { color = Color3.fromRGB(255, 168, 192), more = { title = "Box", width = 220 } })
esp.sub:AddChoice("esp_box_style", "Style", { "Full", "Corner" }, 1)
```

### `box:AddSlider(id, label, min, max, default, opts)`

`suffix` (text after the number), `dec` (decimals), `step`, `zeroText` (text shown at 0, for example `"Off"`),
`dot` (small dot next to the label), `noTrack` (hide the bar, value box only), `trackX` (bar start in pixels, use
a bigger number for long labels).

### `box:AddChoice(id, label, items, default, opts)` · alias `AddDropdown`

`items` is a list of strings, `default` an index, `opts.width` the box width. Change the list later by editing
`Window.items[id].items` in place.

### `box:AddMulti(id, label, items, defaults, opts)`

`defaults` is a list of selected names. `Get` returns a set, so test with `Window:Get("id")["Head"]`.

### Other widgets

| Call | Description |
|---|---|
| `box:AddKeyPill(id, label, defaultVk, { dim, callback })` | A key picker. Click it, then press a key. |
| `box:AddEntry(id, label, placeholder)` | A text field. |
| `box:AddButtons({ { "Label", fn, full = true }, ... })` | Buttons, two per row (`full = true` makes it full width). |
| `box:AddPopover(label, title, width)` | A row that opens a popup. **Returns the popup's box**: `pop:AddSlider(...)`. |
| `box:AddCaption(text)` | Wrapped dim text. Change it later: `cap.text = "new"; cap.wrapW = nil`. |
| `box:AddTabs(id, { "A", "B" }, default, callback)` | A segmented selector. |
| `box:AddChips({ { id, label, default, callback, width }, ... })` | Toggle chips that each open a popup (`chip.sub`). |
| `box:AddPriority(id, label, { "Head", "Chest" }, callback)` | A reorderable list. |
| `box:AddThemeList(id, callback)` | The built-in theme picker (used on the Display page). |
| `box:AddPreview(height)` | The ESP preview. It reads the ids `esp_box`, `esp_name`, `esp_hpbar`, `esp_hptext`, `esp_weapon`, `esp_flags`, `esp_glow`, `esp_highlight`, `esp_vischams`, `esp_arrows`, `pv_speed`, `pv_rotate`. |
| `box:AddNotice()` | The "Unsaved changes" status line (used on the Configs page). |

Short names without `Add` (`box:Switch`, `window:Page`, …) work too.

---

## 7. Toasts

```lua
Window:Toast({ Title = "Saved", Text = "Config saved", Icon = "check", Duration = 3 })

local t = Window:Toast({ Title = "Loading", Icon = "download", Total = 10, Current = 0, Pos = "Bottom Left" })
t:Set(5)      -- progress bar
t:Close()
```

`Title`, `Text`, `Icon`, `Duration` (seconds, ignored when `Total` is set), `Total` / `Current` (progress bar), `Pos`
(`"Center Bottom"`, `"Bottom Right"`, `"Bottom Left"`, `"Top Right"`, `"Top Left"`; default is the Toast Position setting).

---

## 8. What the Common tab gives you (always included)

* **Settings → Configs**: create, save, load, delete, autoload; auto save; the orange **!** on the sidebar and the
  "Unsaved changes" label appear while the loaded config differs from what is saved. Needs `writefile` / `readfile` in your executor.
* **Settings → Display**: Performance Mode, menu transparency and blur, show-on-load, menu key, username display,
  header stats, watermark, bind list, toast position and the theme picker.
* **Settings → Notifications**: toast demo.
* **Players**: player list with Friend / Priority marks (`Window.rel[name]`), copy user ID, player info options.

Item ids used by the library: `perf_mode trans blur blurpx show_on_load menu_key user_disp hdr_stats wm bl bl_active toast_pos theme auto_save cfg_name cfg_sel pl_sel pl_info pl_fields pl_skip_friends pl_priority_first`. Do not reuse them.

## 9. Using the menu

* Search box: filters every option by name and highlights the match.
* Scrolling: Matcha has no mouse wheel event, so use **Up / Down**, **PgUp / PgDn**, **Home / End** while the pointer
  is over a page, hold the **middle mouse button** and drag, or drag the scrollbar.
* Drag the header to move the window.

## 10. Tips and limits

* Lua allows 200 local variables per function. In a big script wrap your menu code in a function, or reuse variable names.
* Config files are stored in the executor workspace as `fzzr_cfg_<name>.txt`, `fzzr_index.txt`, `fzzr_autoload.txt`.
* The library does **not** run any features. It only draws the menu; your script reads values with `Get` / `IsActive`.
* Icons are PNG files named `Crosshair`, `Sword`, `World`, `Slash`, `Bolt`, `Cog` and `Profile` (plus the logo `fzzr.png`) inside the icon folder.
* Icons and the logo are downloaded once on load; shapes are drawn until they arrive.
* Matcha cannot play sounds and cannot create instances, so the library does neither.
