# ncs-menu

A standalone FiveM client-side menu resource by **NexCodeST / Guerrero Studios**.  
fully independent, no dependency on any other menu resource.

frontend developer: https://github.com/GuerreeroDev

---

## Installation

1. Place the `ncs-menu` folder inside your `resources` directory (or a subfolder like `[NexCode]`).
2. Add `ensure ncs-menu` to your `server.cfg`.
3. Start the server — no additional dependencies required.

```cfg
ensure ncs-menu
```

---

## Basic Usage

```lua
local Menu = exports['ncs-menu']

-- Using colon syntax (recommended)
Menu:CreateNewMenu(...)
Menu:CreateNewDialog(...)
Menu:CloseMenu()
Menu:CloseDialog()
Menu:IsMenuOpen()
```

---

## CreateNewMenu

Opens a new menu with a title, a list of items, and a callback.  
If a menu is already open, it is closed first before the new one opens.

```lua
Menu:CreateNewMenu(title, items, callback)
```

| Parameter  | Type     | Description                          |
|------------|----------|--------------------------------------|
| `title`    | `string` | Title displayed in the menu header   |
| `items`    | `table`  | Array of item tables (see Items)     |
| `callback` | `function` | Called when an item is selected    |

### Example

```lua
local Menu = exports['ncs-menu']

local items = {
    {
        title = 'Item 1',
        description = 'This is the first item',
        icon = 'fas fa-car',
        value = 'item_1',
        persoliazedData = {
            example = 'example'
        }
    },
    {
        title = 'Item 2',
        description = 'This is the second item',
        value = 'item_2'
    }
}

Menu:CreateNewMenu('Example Menu', items, function(data)
    if data == 'close' then
        -- User closed the menu (right click or close item)
        return
    end

    if data.value == 'item_1' then
        print(data.persoliazedData.example)
    elseif data.value == 'item_2' then
        print('Item 2 selected')
    end
end)
```

---

## Items

Each item in the `items` table supports the following fields:

| Field             | Type     | Required | Description                                                   |
|-------------------|----------|----------|---------------------------------------------------------------|
| `title`           | `string` | ✅        | The label shown in the menu                                   |
| `value`           | `string` | ✅        | Identifier returned in the callback. Use `'close'` to close   |
| `description`     | `string` | ❌        | Secondary text shown below the title                          |
| `icon`            | `string` | ❌        | Font Awesome class, e.g. `'fas fa-car'`                       |
| `persoliazedData` | `table`  | ❌        | Arbitrary custom data, preserved and returned in the callback |

> **Note**: The `persoliazedData` property name is intentional (matches the original API). Do not rename it.

### Item examples

```lua
-- Full item
{
    title = 'Vehicle',
    description = 'Spawn a vehicle',
    icon = 'fas fa-car',
    value = 'vehicle',
    persoliazedData = {
        model = 'adder',
        price = 50000
    }
}

-- Minimal item (no icon, no description)
{
    title = 'Close',
    value = 'close'
}

-- Item without icon
{
    title = 'Profile',
    description = 'View your profile',
    value = 'profile'
}
```

---

## Callbacks

The callback receives the full item table that was selected.

```lua
Menu:CreateNewMenu('Title', items, function(data)
    -- data is the selected item table
    -- data.value       → item value
    -- data.title       → item title
    -- data.description → item description (if any)
    -- data.icon        → item icon class (if any)
    -- data.persoliazedData → custom data table (if any)

    -- Special case: menu closed via right-click or value = 'close'
    if data == 'close' then
        print('Menu was closed')
        return
    end

    print(data.value)
    print(data.persoliazedData and data.persoliazedData.model)
end)
```

### Callback guarantees

- The callback fires **once** per item selection.
- Old callbacks are **cleared** when a new menu is opened.
- The callback is **never called** after the menu has been closed.
- No duplicate callbacks.

---

## CreateNewDialog

Opens an input dialog on top of the currently open menu.  
The menu must be open for the dialog to appear.

```lua
Menu:CreateNewDialog(title, type, callback)
```

| Parameter  | Type     | Description                                        |
|------------|----------|----------------------------------------------------|
| `title`    | `string` | Placeholder text shown in the input field          |
| `type`     | `string` | Input type: `'text'`, `'string'`, or `'number'`   |
| `callback` | `function` | Called with the entered value when submitted     |

### Supported types

| Type       | Description                                 | Returned value type |
|------------|---------------------------------------------|---------------------|
| `'text'`   | Free text input                             | `string`            |
| `'string'` | Free text input (alias for `'text'`)        | `string`            |
| `'number'` | Numeric input, converted via `tonumber()`   | `number` or `nil`   |

### Examples

```lua
-- Text dialog
Menu:CreateNewDialog('Enter your name', 'text', function(value)
    print('Name:', value)
    print('Type:', type(value))  -- string
end)

-- Number dialog
Menu:CreateNewDialog('Enter amount', 'number', function(value)
    print('Amount:', value)
    print('Type:', type(value))  -- number
end)
```

### Dialog controls

| Key / Action         | Behavior                              |
|----------------------|---------------------------------------|
| `Enter`              | Submit the value and close dialog     |
| `Escape`             | Cancel and close dialog               |
| Right-click          | Cancel and close dialog               |
| Click confirm icon   | Submit the value and close dialog     |

After the dialog closes, the menu behind it continues working normally.

---

## CloseMenu

Closes the active menu, clears all callbacks, releases NUI focus, and resets the internal state.

```lua
Menu:CloseMenu()
```

### What it clears

- Menu open state (`menu.open = false`)
- Current item callback
- Dialog state (if dialog was open)
- NUI focus / keep input
- All internal state

---

## CloseDialog

Closes the dialog without closing the menu behind it.  
The menu continues to work normally after the dialog is closed.

```lua
Menu:CloseDialog()
```

---

## IsMenuOpen

Returns `true` if a menu is currently open, `false` otherwise.

```lua
local isOpen = Menu:IsMenuOpen()

if Menu:IsMenuOpen() then
    print('Menu is open')
end
```

---

## Complete Example

```lua
local Menu = exports['ncs-menu']

-- Full workflow: menu → item with dialog → print result

local function openVehicleMenu()
    local vehicles = {
        {
            title = 'Adder',
            description = 'Bugatti Veyron — $50,000',
            icon = 'fas fa-car',
            value = 'adder',
            persoliazedData = { model = 'adder', price = 50000 }
        },
        {
            title = 'Zentorno',
            description = 'Lamborghini — $75,000',
            icon = 'fas fa-car-side',
            value = 'zentorno',
            persoliazedData = { model = 'zentorno', price = 75000 }
        },
        {
            title = 'Close',
            value = 'close'
        }
    }

    Menu:CreateNewMenu('Vehicle Shop', vehicles, function(data)
        if data == 'close' then
            print('Menu closed')
            return
        end

        -- Ask for a custom plate
        Menu:CreateNewDialog('Enter plate (e.g. ABC123)', 'text', function(plate)
            local model = data.persoliazedData.model
            local price = data.persoliazedData.price

            print(('Spawning %s with plate %s — $%d'):format(model, plate, price))

            -- Do your spawn logic here
            Menu:CloseMenu()
        end)
    end)
end

RegisterCommand('vmenu', function()
    openVehicleMenu()
end, false)
```

---

## API Reference

| Function                                | Description                                              |
|-----------------------------------------|----------------------------------------------------------|
| `Menu:CreateNewMenu(title, items, cb)`  | Open a menu with items and a selection callback          |
| `Menu:CreateNewDialog(title, type, cb)` | Open a text/number dialog on top of the current menu     |
| `Menu:CloseMenu()`                      | Close the menu and clean up all state                    |
| `Menu:CloseDialog()`                    | Close the dialog, leaving the menu open                  |
| `Menu:IsMenuOpen()`                     | Returns `true` if the menu is currently open             |

All exports also support dot syntax via `exports['ncs-menu'].FunctionName(...)`.

---

## Controls (In-menu)

| Control              | Action                                       |
|----------------------|----------------------------------------------|
| Mouse scroll / ↑ ↓   | Navigate items                               |
| Left click / Enter   | Select highlighted item                      |
| Right click          | Close menu (sends `value = 'close'` callback)|
| Right click (dialog) | Cancel dialog                                |
| Enter (dialog)       | Submit dialog value                          |
| Escape (dialog)      | Cancel dialog                                |

While the menu is open (and no dialog is active), weapon controls and action controls are automatically disabled.

---

## Troubleshooting

### Menu doesn't open / no NUI appears

- Make sure `ensure ncs-menu` is in `server.cfg` and the resource has started.
- Check the F8 console for Lua errors.
- Verify the `ui_page` path in `fxmanifest.lua` points to `UI/index.html`.

### Dialog doesn't appear

- `CreateNewDialog` only works while a menu is already open. Call `CreateNewMenu` first.
- If you call `CreateNewDialog` and the dialog is already open, it will be closed first. Call it again after.

### Callback fires twice / fires after close

- This should not happen with the current implementation. If it does, make sure you are not calling `CreateNewMenu` while keeping old references to `exports['ncs-menu']` that could re-register.

### `persoliazedData` is nil in callback

- Make sure the item was sent with the exact key `persoliazedData` (not `personalizedData` or any other spelling).
- The NUI sends the entire item object back to Lua, so all fields including `persoliazedData` are preserved.

### Controls still active while menu is open

- The control-disabling thread runs at `Wait(0)` when the menu is open and dialog is closed.
- If a dialog is open, controls remain active intentionally (so the player can type).

### Sounds not playing

- Sound assets (`scroll.mp3`, `select.wav`) must be present in `UI/assets/sounds/`.
