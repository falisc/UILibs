# ColaUI Documentation

A Drawing-based Roblox UI library with tabs, sections, controls, a built-in Settings page, configs, theme editing, notifications, watermark support, active-keybind overlay, and optional logo/background images.

This guide documents the current implementation in opencodelibslop.lua.

## Quick start

The source is a loadstring-ready module and returns the ColaUI table directly. Create windows from your own script after loading it.

~~~
local ColaUI = loadstring(game:HttpGet("https://raw.githubusercontent.com/falisc/UILibs/refs/heads/main/ColaLib"))()

local Library = ColaUI

local Window = Library:CreateWindow({
    title = "My UI",
    game = "My Game",
    version = "v1.0.0",
    build = "PRIVATE BUILD",
    Keybind = "RightAlt",
    CloseKeybind = "Delete",
    Width = 600,
    Height = 750,
    Logo = "https://example.com/logo.png",
    Background = "https://example.com/background.png",
    ConfigFolder = "MyUI_configs",
    ConfigName = "MyConfig",
})

Window:AddTab("Home")
local Main = Window:AddLeftSection(1, "Main")
Main:Label("Welcome to my UI")
~~~

New windows center from Roblox's live viewport, so they center correctly on 1080p, 1440p, and other resolutions.

## Library API

### CreateWindow(config)

Creates, centers, and starts rendering a new window.

| Field | Default | Meaning |
|---|---|---|
| title | Window | Main title text. |
| game | Game | Game label in title bar. |
| version | 1.0.0 | Version label in title bar. |
| build | RELEASE | Build label in title bar. |
| Width | 600 | Window width. |
| Height | 750 | Content height; footer height is added automatically. |
| Keybind | p | Show/hide key. |
| CloseKeybind | delete | Permanently destroys the window. |
| Logo | none | Optional image URL for title bar. |
| Background | none | Optional background image URL. Default config starts with it disabled. |
| BackgroundOpacity | 0.24 | Background opacity when enabled. |
| ConfigFolder / configFolder | ColaUI_configs | Config folder. |
| ConfigName / configName | HavocConfig | Base preference key. |

### CreateWatermark(config)

~~~
local Watermark = Library:CreateWatermark({
    text = "FAL.LOL Baseplate v1.0.0",
    x = 20,
    y = 20,
})
~~~

The watermark is draggable. Use Watermark:Destroy() to remove it.

### Notify(config)

~~~
Library:Notify({
    Title = "Saved",
    Content = "Your configuration was saved.",
    Duration = 4,
})

Library:Notify("Saved", "Your configuration was saved.", 4)
~~~

Notify accepts Title/title, Content/content, Message/message, or Text/text. Duration defaults to four seconds and cannot be below 0.5 seconds. The returned notification supports Close().

### DestroyAll()

Library:DestroyAll() removes all windows, overlays, notifications, and pooled drawings created by the library.

## Window API

~~~
Window:Center()
Window:SetPosition(300, 180)
Window:SetSize(650, 760)
Window:SetTitle("Updated title")
Window:Show()
Window:Hide()
Window:Destroy()
~~~

Hide keeps the UI alive and lets its menu key reopen it. Destroy removes it permanently.

### Tabs and sections

Create tabs before their sections.

~~~
Window:AddTab("Home")
Window:AddTab("Visuals")

local Left = Window:AddLeftSection(1, "Settings")
local Right = Window:AddRightSection(1, "Preview")
Window:SelectTab(2)
~~~

Sections are collapsible. Content fades and clips at scroll boundaries.

## Section controls

### Labels

Labels support multi-line text directly.

~~~
Section:Label("One clean line")
Section:Label("First line\nSecond line\nThird line")
Section:Label("Plain text", { Background = false })
~~~

There is no need for separate AddLine or SetLine methods.

### Buttons and action rows

~~~
Section:AddButton("Reset", function()
    print("Reset clicked")
end)

Section:AddActionButtons({
    { Text = "Save", Callback = function() print("save") end },
    { Text = "Load", Callback = function() print("load") end },
    { Text = "Delete", Callback = function() print("delete") end },
})
~~~

### Checkboxes and attached keybinds

~~~
local Toggle = Section:AddCheckbox("Enable Feature", function(enabled, mode)
    print("Feature callback:", enabled, mode)
end, {
    Keybind = true,
    Default = "LeftControl",
    KeyMode = "Hold", -- Hold, Toggle, Always, or Click
})
~~~

A checkbox is enabled only by clicking it. Pressing its keybind never changes its checked state.

When checked:

- Hold runs its callback while the key is held.
- Toggle flips an active state on each press and runs while active.
- Always runs continuously while checked.
- Click runs once per press.

Left-click the small key box to capture a new key. Right-click it to select Hold, Toggle, Always, or Click. Keybind actions are blocked while typing in a textbox or dropdown search.

#### Attached color picker

~~~
Section:AddCheckbox("Outline", function(enabled)
    print("Outline:", enabled)
end, {
    Keybind = true,
    Default = "Z",
    ColorPicker = {
        Default = Color3.fromRGB(40, 170, 255),
        Callback = function(color)
            print("Outline color:", color)
        end,
    },
})
~~~

### Standalone keybind

~~~
local Bind = Section:AddKeybind("Action Bind", "F7", function(key)
    print("Pressed:", key)
end)

Bind:SetKeybind("F8")
~~~

Standalone binds call their callback on press. Left-click their key box to capture another key.

### Sliders

~~~
local Slider = Section:AddSlider("FOV", 60, 120, 90, function(value)
    print("FOV:", value)
end)

Slider:SetValue(100)
~~~

### Textboxes

~~~
local Box = Section:AddTextbox(
    "Name",
    "",
    function(value)
        print("Name:", value)
    end,
    "Type a name"
)

print(Box:GetValue())
Box:SetValue("Example")
Box:Blur()
~~~

Textboxes support typing, arrow navigation, Backspace/Delete, Enter/Escape to leave the field, double-click clear, and a 250-character limit. Long text is clipped instead of drawing outside its box.

While a textbox is focused, checkbox and standalone keybind actions are paused, the menu key is consumed, and Roblox input stays blocked.

Clipboard paste is deliberately not included: Matcha documents clipboard writing but does not expose a reliable clipboard-read API.

### Dropdowns

~~~
Section:AddDropdown("Weapon", { "Rifle", "SMG", "Pistol" }, "Single", function(value)
    print("Selected:", value)
end, "Rifle")

Section:AddDropdown("Tools", { "Ruler", "Protractor", "Grid" }, "Multi", function(selected)
    for option, enabled in pairs(selected) do
        print(option, enabled)
    end
end, { "Ruler" })
~~~

Use Single for a single selection. Any other type creates a multi-select dropdown. Multi-select dropdowns include search, Select All, and Unselect All.

### Standalone color pickers

~~~
local Picker = Section:AddColorPicker("Accent", Color3.fromRGB(0, 170, 255), function(color)
    print(color)
end)

Picker:SetValue(Color3.fromRGB(255, 80, 80))
local Value = Picker:GetValue()
Picker:SetHSV(0.55, 0.8, 1)
~~~

Color pickers use an HSV square popup. An open picker blocks clicks from reaching controls behind it.

### Dividers

~~~
Section:AddDivider("ADVANCED")
~~~

## Built-in Settings page

Every window has a built-in Settings page opened from the gear icon. It is created automatically; users of the library do not add it manually.

It includes:

- UI Keybind: changes the window show/hide key.
- Configs: name field, Save, Load, Delete, config dropdown, Auto-save, and Auto-load.
- Overlays: Watermark and Keybind Overlay controls.
- Appearance: Background Image and Logo checkboxes plus URL fields.
- Visual: color pickers for UI colors, slider outlines, borders, tabs, scrollbars, and every gradient stop.
- Reset Defaults: restores the default theme and gradient.

Appearance URLs apply after leaving their textbox, not on every keystroke.

## Config API

Default is an in-memory protected config. It cannot be written over or deleted.

~~~
Window:CaptureDefaultConfig()

Window:SaveConfig("Legit")
Window:LoadConfig("Legit")
Window:LoadConfig("Default")

Window:SetAutoSave(true)
Window:SetAutoLoad(true, "Legit")

for _, name in ipairs(Window:ListConfigs()) do
    print(name)
end

Window:DeleteConfig("Legit")
~~~

Configs save checkbox values, checkbox keybinds and modes, attached picker colors, sliders, dropdowns, textboxes, standalone keybinds, standalone pickers, theme colors, gradient colors, overlays, and appearance settings.

Auto-save pauses during text entry and briefly after loading. Selecting a config from the dropdown does not change the active auto-save target until it is loaded.

## Overlays and input

The Keybind Overlay is enabled by default in the protected Default config. It lists the UI key once, enabled checkbox keybinds, and standalone keybinds. Active toggle entries use accent text.

The watermark and keybind overlay are independently draggable. Each captures its drag input, so the window or other floating element cannot drag underneath it.

Input capture is click-based:

- Click any UI surface to block Roblox input.
- Click outside all UI surfaces to restore Roblox input.
- Text fields, modal controls, and active drags keep Roblox input blocked until interaction ends.

## Notes

- Logo and background URLs are not domain restricted. Use a directly downloadable image URL.
- Images are loaded through game:HttpGet and Drawing Image.Data.
- The footer resolves player name, UserId, and avatar asynchronously with retries, fallback lookup, cache, and alternate thumbnail endpoints.
- In the example at the source bottom, call Window:CaptureDefaultConfig() only after all user controls are added, then call Window:RestoreAutoLoad().
