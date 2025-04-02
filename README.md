
# TTT2 Item Guide (WIP)

## Prologue

I am just an addon developer that decided to dive into the code a little bit and could not find a proper guide on how to mod TTT2 Items, so I decided to write one (The code in TTT2 github to items is well documented, although there does not yet exist a guide).

Take these explanations with a grain of salt since i do not exactly know the reasons behind everything nor have the desire to analyze the whole codebase, corrections are welcome and encouraged. 
 
This information is obtained from the TTT2 Github https://github.com/TTT-2/TTT2/. 
And TTT2 has its own fileloader as stated on their official docs (https://docs.ttt2.neoxult.de/developers/basics/creating-an-addon/) and decoding of some existing addons.

## Directory Structure

Items should mimic the structure of the gamemode / base game (https://wiki.facepunch.com/gmod/Lua_Folder_Structure). What that means is the TTT2 Structure for Items looks like this:  lua/terrortown/entities/items/*.lua
So if you want add your own addon you would create a file structure like this: myAddonName/lua/terrortown/entities/items/myAddonName.lua.

Optionally you could do:
* myAddonName
    * lua
        * terrortown
            * entities
                * items
                    * myItemName
                        * cl_init.lua
                        * init.lua
                        * shared.lua

The custom fileloader of TTT2 recognizes that there is a subfolder inside items/ and automatically searches for cl_init, init and shared lua files (https://github.com/TTT-2/TTT2/blob/798de4c162e9d85749e34f15c82a7101c9a230a4/gamemodes/terrortown/gamemode/shared/sh_item_module.lua).

All files inside [myItemName] folder are automatically included in the code, so if you were to add a blabla.lua into the folder it would also get included and ran by client and server, although you would have to add your own AddCSLuaFile and or check for server and client. But init.lua, cl_init.lua and shared.lua are treated special (Based on the default behaviour of gmod itself https://wiki.facepunch.com/gmod/Understanding_AddCSLuaFile_and_include).

__init.lua__ is only run on the server.

__cl_init__ is only run on the client and automatically provided* to the client.

__shared.lua__ is automatically provided* and run on both client and server

*Generally, a subfolder inside /items is a cleaner approach since you don't need to check or provide the files manually*
 
*AddCSLuaFile is added by the fileloader and thus the addon creator does not need to add it manually

## Item Structure
TTT2 has a new* way of adding items to the gamemode, They made their own item loader, and it expects the following structure (this is directly taken out of the TTT2 github https://github.com/TTT-2/TTT2/blob/798de4c162e9d85749e34f15c82a7101c9a230a4/lua/terrortown/entities/items/item_base/shared.lua):

```
---
-- @class ITEM

ITEM.PrintName = "Scripted Item" -- displayed @{ITEM} name (Shown on HUD), language supported

ITEM.Author = "" -- author
ITEM.Contact = "" -- contact to the author
ITEM.Purpose = "" -- purpose of this @{ITEM}
ITEM.Instructions = "" -- some Instructions for this @{ITEM}

-- If CanBuy is a table that contains ROLE_TRAITOR and/or ROLE_DETECTIVE, those
-- players are allowed to purchase it and it will appear in their Equipment Menu
-- for that purpose. If CanBuy is nil this @{ITEM} cannot be bought.
--   Example: ITEM.CanBuy = { ROLE_TRAITOR }
-- (just setting to nil here to document its existence, don't make this buyable)
ITEM.CanBuy = nil -- this @{table} contains a list of subrole ids (@param number) which are able to access this @{ITEM} in the shop

-- per default items can be bought only once. However this variable can be set to false to allow multibuy
ITEM.limited = true

ITEM.Category = "TTT" -- the @{ITEM} Category

if CLIENT then
    -- If this is a buyable @{ITEM} (ie. CanBuy is not nil) EquipMenuData must be
    -- a table containing some information to show in the Equipment Menu. See
    -- default equipment items for real-world examples.
    ITEM.EquipMenuData = nil

    -- Example data:
    -- ITEM.EquipMenuData = {
    --
    --   Type tells players if it's a weapon or @{ITEM}
    --     type = "Weapon",
    --
    --   Desc is the description in the menu. Needs manual linebreaks (via \n).
    --     desc = "Text."
    -- }

    -- A short description for the sidebar
    ITEM.sidebarDescription = nil

    -- This sets the icon shown for the @{ITEM} in the DNA sampler, search window,
    -- equipment menu (if buyable), etc.
    ITEM.material = "vgui/ttt/icon_nades" -- most generic icon I guess

    -- set to false if item should not be shown in body search
    ITEM.populateSearch = true

    -- You can make your own @{ITEM} icon using the template in:
    --   /garrysmod/gamemodes/terrortown/template/

    -- Open one of TTT's icons with VTFEdit to see what kind of settings to use
    -- when exporting to VTF. Once you have a VTF and VMT, you can
    -- resource.AddFile("materials/vgui/...") them here. GIVE YOUR ICON A UNIQUE
    -- FILENAME, or it WILL be overwritten by other servers! Gmod does not check
    -- if the files are different, it only looks at the name. I recommend you
    -- create your own directory so that this does not happen,
    -- eg. /materials/vgui/ttt/mycoolserver/mygun.vmt

    ---
    -- Draw some information in a small box next to the icon in the hud if a @{Player} is owning this @{ITEM}
    -- @hook
    -- @realm client
    function ITEM:DrawInfo() end

    ---
    -- This hook can be used by item addons to populate the equipment settings page
    -- with custom convars. The parent is the submenu, where a new form has to
    -- be added.
    -- @param DPanel parent The parent panel which is the submenu
    -- @hook
    -- @realm client
    function ITEM:AddToSettingsMenu(parent) end
end

---
-- Called just before entity is deleted. This is used to reset data you set before
-- @param Player ply
-- @hook
-- @realm shared
function ITEM:Reset(ply) end

---
-- A player or NPC has picked the @{ITEM} up
-- @param Player ply
-- @hook
-- @realm shared
function ITEM:Equip(ply) end

---
-- A player or NPC has bought the @{ITEM}
-- @param Player ply
-- @hook
-- @realm shared
function ITEM:Bought(ply) end

---
-- Called when the @{ITEM} is created.
-- @realm shared
function ITEM:Initialize() end

---
-- Checks whether this items is a valid equipment item.
-- @note Useable, but do not modify or overwrite this!
-- @return boolean Return whether this @{ITEM} is an equipment
-- @realm shared
function ITEM:IsEquipment()
    return WEPS.IsEquipment(self)
end

---
-- Returns the classname of an item. This is often the name of the Lua file or folder containing the files for the item.
-- @return string The item's class name
-- @realm shared
function ITEM:GetClass()
    return self.ClassName
end

---
-- Checks if the item is valid. Returns true if valid.
-- @return boolean Returns true if item is valid
-- @realm shared
function ITEM:IsValid()
    -- this is probably always valid if there is no custom weird stuff done
    -- to the item's class
    if self:GetClass() and self:GetClass() ~= "" then
        return true
    end

    return false
end
```

As you can see the code is already well documented. Please note that the code above would be located in a shared.lua.

*I just assume they do, since I could not find the same structure for items in TTT. But I never really created a TTT addon, so I am not 100% sure, although the old way is still compatible as stated here: https://docs.ttt2.neoxult.de/developers/basics/creating-an-addon/#items.

## Menus and Configuration
So you now know where and how to create your new item, but what about the configuration of it? What if I want to modify some property of my item? Well that is where ConVars and Menus come in.

TTT2 also has a folder structure you need to adhere to if you want to add a menu or submenu to their F1 menu. The good thing is they define some basic input fields you my want to use like a slider, checkbox etc. (Full list here: ).

Those automatically change the ConVars for you except the binder, but the keybindings are handled differently, TTT2 uses its own system for that (https://github.com/TTT-2/TTT2/blob/798de4c162e9d85749e34f15c82a7101c9a230a4/lua/ttt2/libraries/bind.lua).

## Example Addon
