# SCRIPT-FISHZAR-


debugX = true

--// LOAD FRAMEWORK
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

--// CORE OBJECT
local HUB = {
    Version = "3.0",
    Modules = {},
    Permissions = {},
    Events = {},
    Logs = {}
}

--// SAFE LOGGER
function HUB:Log(message)
    table.insert(self.Logs, "[REV] "..message)
    if debugX then
        print("[REV-HUB]", message)
    end
end

--// EVENT BUS
function HUB:On(eventName, callback)
    self.Events[eventName] = self.Events[eventName] or {}
    table.insert(self.Events[eventName], callback)
end

function HUB:Emit(eventName, ...)
    if self.Events[eventName] then
        for _,callback in ipairs(self.Events[eventName]) do
            callback(...)
        end
    end
end

--// PERMISSION SYSTEM
function HUB:SetPermission(userId, level)
    self.Permissions[userId] = level
end

function HUB:CheckPermission(userId, required)
    return (self.Permissions[userId] or 0) >= required
end

--// MODULE REGISTRY
function HUB:RegisterModule(name, permissionLevel, callback)
    self.Modules[name] = {
        Level = permissionLevel,
        Execute = function(...)
            local player = game.Players.LocalPlayer
            if HUB:CheckPermission(player.UserId, permissionLevel) then
                HUB:Log("Executing module: "..name)
                local success,err = pcall(callback,...)
                if not success then
                    warn("Module Error:", err)
                end
            else
                Rayfield:Notify({
                    Title = "Access Denied",
                    Content = "Insufficient permission level.",
                    Duration = 3,
                    Image = "shield"
                })
            end
        end
    }
    HUB:Log("Module Registered: "..name)
end

--// PERFORMANCE MONITOR
local RunService = game:GetService("RunService")
local fps = 0
local last = tick()

RunService.RenderStepped:Connect(function()
    local now = tick()
    fps = math.floor(1 / (now - last))
    last = now
end)

--// WINDOW
local Window = Rayfield:CreateWindow({
   Name = "REV-HUB",
   Icon = "cpu",
   LoadingTitle = "REV-HUB v"..HUB.Version,
   LoadingSubtitle = "Enterprise Core System",
   Theme = "DarkBlue",
   DisableRayfieldPrompts = true,
   DisableBuildWarnings = true,
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "REV-HUB",
      FileName = "REV_Config"
   }
})

-- ============= [ AUTO SELL SYSTEM ] =============
-- ============= [ VARIABEL GLOBAL ] =============

local player = game:GetService("Players").LocalPlayer
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ============= [ FUNGSI FIND REMOTE ] =============

local function findFishingSystem()
    local possibleNames = {
        "FishingSystem", "Fishing", "FishSystem", "FishingService", 
        "Fish", "Fisher", "FishingGame", "FishingRemote"
    }
    
    for _, name in ipairs(possibleNames) do
        local found = ReplicatedStorage:FindFirstChild(name)
        if found then
            HUB:Log("✅ FishingSystem ditemukan dengan nama: " .. name)
            return found
        end
    end
    
    for _, child in ipairs(ReplicatedStorage:GetChildren()) do
        if child.Name:lower():find("fish") or child.Name:lower():find("fishing") then
            HUB:Log("🔍 FishingSystem mirip: " .. child.Name)
            return child
        end
    end
    
    return nil
end

local function findSellRemote(fishingSystem)
    if not fishingSystem then return nil end
    
    local possibleNames = {
        "SellFish", "Sell", "FishSell", "SellRemote",
        "SellFishRemote", "FishSold", "SellItem", "SellTool"
    }
    
    for _, name in ipairs(possibleNames) do
        local found = fishingSystem:FindFirstChild(name)
        if found then
            HUB:Log("✅ Sell remote ditemukan dengan nama: " .. name)
            return found
        end
    end
    
    for _, child in ipairs(fishingSystem:GetChildren()) do
        if child.Name:lower():find("sell") or child.Name:lower():find("jual") then
            HUB:Log("🔍 Sell remote mirip: " .. child.Name)
            return child
        end
    end
    
    return nil
end

local function findInventoryEvents(fishingSystem)
    if not fishingSystem then return nil end
    
    local possibleNames = {
        "InventoryEvents", "Inventory", "InvEvents",
        "InventorySystem", "PlayerInventory", "InventoryService"
    }
    
    for _, name in ipairs(possibleNames) do
        local found = fishingSystem:FindFirstChild(name)
        if found then
            HUB:Log("✅ InventoryEvents ditemukan dengan nama: " .. name)
            return found
        end
    end
    
    for _, child in ipairs(fishingSystem:GetChildren()) do
        if child.Name:lower():find("inventory") or child.Name:lower():find("inv") then
            HUB:Log("🔍 Inventory mirip: " .. child.Name)
            return child
        end
    end
    
    return nil
end

local function findEquipRemote(inventoryEvents)
    if not inventoryEvents then return nil end
    
    local possibleNames = {
        "Inventory_EquipFish", "EquipFish", "Equip",
        "EquipRemote", "FishEquip", "Inventory_Equip"
    }
    
    for _, name in ipairs(possibleNames) do
        local found = inventoryEvents:FindFirstChild(name)
        if found then
            HUB:Log("✅ Equip remote ditemukan dengan nama: " .. name)
            return found
        end
    end
    
    for _, child in ipairs(inventoryEvents:GetChildren()) do
        if child.Name:lower():find("equip") or child.Name:lower():find("pakai") then
            HUB:Log("🔍 Equip remote mirip: " .. child.Name)
            return child
        end
    end
    
    return nil
end

-- ============= [ INIT REMOTE ] =============

HUB:Log("🔍 Mencari remotes dengan Find First Child...")

local FishingSystem = findFishingSystem()
local SellFish = FishingSystem and findSellRemote(FishingSystem)
local InventoryEvents = FishingSystem and findInventoryEvents(FishingSystem)
local EquipFish = InventoryEvents and findEquipRemote(InventoryEvents)

-- ============= [ SETTINGS ] =============

local Settings = {
    AutoSellEnabled = false,
    AutoSellDelay = 0.5,
    MinRarity = "Common",
    EquipAfterSell = true,
    FishingSystem = FishingSystem,
    SellFish = SellFish,
    EquipFish = EquipFish,
    -- Pilih format yang mau dipake
    SellFormat = 1 -- 1,2,3,4
}

local Stats = {
    Sold = 0,
    TotalValue = 0
}

-- ============= [ FUNGSI GET FISH ID ] =============

local function getFishIdFromTool(tool)
    if not tool then return nil end
    
    if tool:FindFirstChild("fishId") then
        return tool.fishId.Value
    elseif tool:FindFirstChild("FishId") then
        return tool.FishId.Value
    elseif tool:FindFirstChild("id") then
        return tool.id.Value
    elseif tool:FindFirstChild("ID") then
        return tool.ID.Value
    end
    
    if tool:GetAttribute("fishId") then
        return tool:GetAttribute("fishId")
    elseif tool:GetAttribute("id") then
        return tool:GetAttribute("id")
    end
    
    return nil
end

local function getCurrentEquippedFish()
    local success, result = pcall(function()
        local char = player.Character
        if char then
            local tool = char:FindFirstChildOfClass("Tool")
            if tool then
                return getFishIdFromTool(tool)
            end
        end
    end)
    return success and result or nil
end

-- ============= [ FUNGSI GET INVENTORY ] =============

local function getInventoryFish()
    local fishList = {}
    pcall(function()
        local backpack = player:FindFirstChild("Backpack")
        if backpack then
            for _, child in ipairs(backpack:GetChildren()) do
                if child:IsA("Tool") then
                    local fishId = getFishIdFromTool(child)
                    if fishId then
                        table.insert(fishList, {
                            fishId = fishId,
                            rarity = child:GetAttribute("rarity") or "Common",
                            tool = child
                        })
                    end
                end
            end
        end
    end)
    return fishList
end

-- ============= [ FUNGSI EQUIP FISH ] =============

local function equipFish(fishId)
    if not EquipFish then return end
    
    pcall(function()
        local args = { fishId }
        EquipFish:FireServer(unpack(args))
    end)
end

-- ============= [ FUNGSI SELL FISH - MULTI FORMAT ] =============

local function sellFish(fishData)
    if not SellFish then return end
    
    pcall(function()
        -- WEIGHT GEDE (number biasa kayak contoh)
        local bigWeight = 999999999999999999999999999999
        
        -- Coba berbagai format
        local success = false
        local lastError = ""
        
        -- FORMAT 1: Persis kayak contoh lo
        local format1 = {
            "SellSingle",
            {
                fishId = fishData.fishId,
                rarity = fishData.rarity or "Legendary",
                weight = bigWeight
            }
        }
        
        -- FORMAT 2: Pake string untuk "SellSingle"
        local format2 = {
            [1] = "SellSingle",
            [2] = {
                fishId = fishData.fishId,
                rarity = fishData.rarity or "Legendary",
                weight = bigWeight
            }
        }
        
        -- FORMAT 3: Tanpa "SellSingle"
        local format3 = {
            fishId = fishData.fishId,
            rarity = fishData.rarity or "Legendary",
            weight = bigWeight
        }
        
        -- FORMAT 4: Pake "Sell" instead of "SellSingle"
        local format4 = {
            "Sell",
            {
                fishId = fishData.fishId,
                rarity = fishData.rarity or "Legendary",
                weight = bigWeight
            }
        }
        
        -- Cek format yang dipilih user
        if Settings.SellFormat == 1 or Settings.SellFormat == 0 then
            pcall(function()
                SellFish:FireServer(unpack(format1))
                success = true
                HUB:Log("✅ Format 1 work: " .. fishData.fishId)
            end)
        end
        
        if (Settings.SellFormat == 2 or Settings.SellFormat == 0) and not success then
            pcall(function()
                SellFish:FireServer(unpack(format2))
                success = true
                HUB:Log("✅ Format 2 work: " .. fishData.fishId)
                if Settings.SellFormat == 0 then Settings.SellFormat = 2 end
            end)
        end
        
        if (Settings.SellFormat == 3 or Settings.SellFormat == 0) and not success then
            pcall(function()
                SellFish:FireServer(unpack(format3))
                success = true
                HUB:Log("✅ Format 3 work: " .. fishData.fishId)
                if Settings.SellFormat == 0 then Settings.SellFormat = 3 end
            end)
        end
        
        if (Settings.SellFormat == 4 or Settings.SellFormat == 0) and not success then
            pcall(function()
                SellFish:FireServer(unpack(format4))
                success = true
                HUB:Log("✅ Format 4 work: " .. fishData.fishId)
                if Settings.SellFormat == 0 then Settings.SellFormat = 4 end
            end)
        end
        
        if success then
            Stats.Sold = Stats.Sold + 1
            
            Rayfield:Notify({
                Title = "SOLD 💰",
                Content = "Ikan ke-" .. Stats.Sold .. " terjual! (Format " .. Settings.SellFormat .. ")",
                Duration = 1,
                Image = "dollar-sign"
            })
            
            HUB:Emit("FishSold", fishData)
        else
            HUB:Log("❌ Gagal jual ikan: " .. fishData.fishId)
            Rayfield:Notify({
                Title = "GAGAL ❌",
                Content = "Ikan gak kejual! Cek format",
                Duration = 2,
                Image = "alert-circle"
            })
        end
    end)
end

-- ============= [ RARITY CHECK ] =============

local rarityLevel = {
    ["Common"] = 1, ["Uncommon"] = 2, ["Rare"] = 3,
    ["Epic"] = 4, ["Legendary"] = 5, ["Mythic"] = 6
}

local function shouldSell(fishData)
    local minLevel = rarityLevel[Settings.MinRarity] or 1
    local fishLevel = rarityLevel[fishData.rarity] or 0
    return fishLevel >= minLevel
end

-- ============= [ MAIN LOOP ] =============

task.spawn(function()
    while true do
        if Settings.AutoSellEnabled then
            pcall(function()
                local equippedId = getCurrentEquippedFish()
                local inventory = getInventoryFish()
                local soldCount = 0
                
                for _, fishData in ipairs(inventory) do
                    if fishData.fishId ~= equippedId and shouldSell(fishData) then
                        sellFish(fishData)
                        soldCount = soldCount + 1
                        task.wait(Settings.AutoSellDelay)
                    end
                end
                
                if soldCount > 0 then
                    HUB:Log("🔄 Sesi jual: " .. soldCount .. " ikan")
                end
            end)
        end
        task.wait(2)
    end
end)

--// TABS
local CoreTab = Window:CreateTab("Core", "cpu")
local AutoSellTab = Window:CreateTab("💰 AUTO SELL", "dollar-sign")
local StatusTab = Window:CreateTab("📊 STATUS", "activity")
local ModuleTab = Window:CreateTab("Modules", "boxes")
local SystemTab = Window:CreateTab("System", "settings")

--// CORE TAB
CoreTab:CreateSection("System Boot")

--// BOOT SEQUENCE
task.spawn(function()
    local bootSteps = {
        "Initializing Kernel...",
        "Validating Modules...",
        "Binding Event Bus...",
        "Starting Runtime Monitor...",
        "REV-HUB Online."
    }

    for _,msg in ipairs(bootSteps) do
        HUB:Log(msg)
        Rayfield:Notify({
            Title = "REV Boot",
            Content = msg,
            Duration = 1.2,
            Image = "terminal"
        })
        task.wait(0.8)
    end

    HUB:Emit("BootComplete")
end)

--// AUTO SELL TAB
AutoSellTab:CreateSection("⚙️ PENGATURAN AUTO SELL")

AutoSellTab:CreateToggle({
    Name = "🔴 Aktifkan Auto Sell",
    CurrentValue = false,
    Flag = "AutoSellToggle",
    Callback = function(v) 
        Settings.AutoSellEnabled = v 
        HUB:Log("Auto Sell: " .. (v and "ON" or "OFF"))
        Rayfield:Notify({
            Title = "Auto Sell",
            Content = v and "Diaktifkan" or "Dimatikan",
            Duration = 2,
            Image = v and "power" or "power-off"
        })
    end
})

AutoSellTab:CreateDropdown({
    Name = "⭐ Minimal Rarity",
    Options = {"Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic"},
    CurrentOption = "Common",
    Flag = "RarityDropdown",
    Callback = function(o) 
        Settings.MinRarity = o
        HUB:Log("Min Rarity: " .. o)
    end
})

AutoSellTab:CreateSlider({
    Name = "⏱️ Delay Antar Jual",
    Range = {0.1, 2},
    Increment = 0.1,
    Suffix = "detik",
    CurrentValue = 0.5,
    Flag = "DelaySlider",
    Callback = function(v) 
        Settings.AutoSellDelay = v
    end
})

AutoSellTab:CreateDropdown({
    Name = "📁 Pilih Format Remote",
    Options = {"Auto Detect", "Format 1", "Format 2", "Format 3", "Format 4"},
    CurrentOption = "Auto Detect",
    Flag = "FormatDropdown",
    Callback = function(o)
        if o == "Auto Detect" then
            Settings.SellFormat = 0
        elseif o == "Format 1" then
            Settings.SellFormat = 1
        elseif o == "Format 2" then
            Settings.SellFormat = 2
        elseif o == "Format 3" then
            Settings.SellFormat = 3
        elseif o == "Format 4" then
            Settings.SellFormat = 4
        end
        HUB:Log("Format remote: " .. o)
    end
})

AutoSellTab:CreateSection("📋 INFORMASI REMOTE")

local RemoteStatus1 = AutoSellTab:CreateLabel(FishingSystem and "✅ FishingSystem: DITEMUKAN" or "❌ FishingSystem: TIDAK DITEMUKAN")
local RemoteStatus2 = AutoSellTab:CreateLabel(SellFish and "✅ Sell Remote: DITEMUKAN" or "❌ Sell Remote: TIDAK DITEMUKAN")
local RemoteStatus3 = AutoSellTab:CreateLabel(EquipFish and "✅ Equip Remote: DITEMUKAN" or "❌ Equip Remote: TIDAK DITEMUKAN")

--// STATUS TAB
StatusTab:CreateSection("📊 STATISTIK PENJUALAN")

local SoldCountLabel = StatusTab:CreateLabel("Ikan Terjual: 0")
local FormatActiveLabel = StatusTab:CreateLabel("Format Aktif: Auto Detect")
local FPSLabel = StatusTab:CreateLabel("FPS: 0")

StatusTab:CreateSection("🧪 TEST MANUAL")

StatusTab:CreateButton({
    Name = "🎣 TEST FORMAT 1",
    Callback = function()
        local fishId = getCurrentEquippedFish()
        if fishId and SellFish then
            HUB:Log("🧪 Test format 1 dengan fish: " .. fishId)
            pcall(function()
                local args = {
                    "SellSingle",
                    {
                        fishId = fishId,
                        rarity = "Legendary",
                        weight = 999999999999999999999999999999
                    }
                }
                SellFish:FireServer(unpack(args))
                Rayfield:Notify({ 
                    Title = "TEST FORMAT 1", 
                    Content = "Remote dicoba! Cek console", 
                    Duration = 2,
                    Image = "test-tube"
                })
            end)
        else
            Rayfield:Notify({
                Title = "GAGAL",
                Content = "Tidak ada ikan di tangan atau remote tidak ditemukan",
                Duration = 3,
                Image = "alert-circle"
            })
        end
    end
})

StatusTab:CreateButton({
    Name = "🎣 TEST FORMAT 2",
    Callback = function()
        local fishId = getCurrentEquippedFish()
        if fishId and SellFish then
            HUB:Log("🧪 Test format 2 dengan fish: " .. fishId)
            pcall(function()
                local args = {
                    [1] = "SellSingle",
                    [2] = {
                        fishId = fishId,
                        rarity = "Legendary",
                        weight = 999999999999999999999999999999
                    }
                }
                SellFish:FireServer(unpack(args))
                Rayfield:Notify({ 
                    Title = "TEST FORMAT 2", 
                    Content = "Remote dicoba! Cek console", 
                    Duration = 2,
                    Image = "test-tube"
                })
            end)
        else
            Rayfield:Notify({
                Title = "GAGAL",
                Content = "Tidak ada ikan di tangan atau remote tidak ditemukan",
                Duration = 3,
                Image = "alert-circle"
            })
        end
    end
})

StatusTab:CreateButton({
    Name = "🎣 TEST FORMAT 3",
    Callback = function()
        local fishId = getCurrentEquippedFish()
        if fishId and SellFish then
            HUB:Log("🧪 Test format 3 dengan fish: " .. fishId)
            pcall(function()
                local args = {
                    fishId = fishId,
                    rarity = "Legendary",
                    weight = 999999999999999999999999999999
                }
                SellFish:FireServer(unpack(args))
                Rayfield:Notify({ 
                    Title = "TEST FORMAT 3", 
                    Content = "Remote dicoba! Cek console", 
                    Duration = 2,
                    Image = "test-tube"
                })
            end)
        else
            Rayfield:Notify({
                Title = "GAGAL",
                Content = "Tidak ada ikan di tangan atau remote tidak ditemukan",
                Duration = 3,
                Image = "alert-circle"
            })
        end
    end
})

StatusTab:CreateButton({
    Name = "🎣 TEST FORMAT 4",
    Callback = function()
        local fishId = getCurrentEquippedFish()
        if fishId and SellFish then
            HUB:Log("🧪 Test format 4 dengan fish: " .. fishId)
            pcall(function()
                local args = {
                    "Sell",
                    {
                        fishId = fishId,
                        rarity = "Legendary",
                        weight = 999999999999999999999999999999
                    }
                }
                SellFish:FireServer(unpack(args))
                Rayfield:Notify({ 
                    Title = "TEST FORMAT 4", 
                    Content = "Remote dicoba! Cek console", 
                    Duration = 2,
                    Image = "test-tube"
                })
            end)
        else
            Rayfield:Notify({
                Title = "GAGAL",
                Content = "Tidak ada ikan di tangan atau remote tidak ditemukan",
                Duration = 3,
                Image = "alert-circle"
            })
        end
    end
})

--// LIVE UPDATE STATUS
task.spawn(function()
    while true do
        if Settings.AutoSellEnabled then
            SoldCountLabel:Set("Ikan Terjual: " .. Stats.Sold)
            
            local formatText = "Auto Detect"
            if Settings.SellFormat == 1 then
                formatText = "Format 1"
            elseif Settings.SellFormat == 2 then
                formatText = "Format 2"
            elseif Settings.SellFormat == 3 then
                formatText = "Format 3"
            elseif Settings.SellFormat == 4 then
                formatText = "Format 4"
            end
            FormatActiveLabel:Set("Format Aktif: " .. formatText)
        end
        
        FPSLabel:Set("FPS: " .. fps)
        task.wait(1)
    end
end)

--// MODULE TAB
ModuleTab:CreateSection("System Modules")

-- Register Auto Sell as a module
HUB:RegisterModule("AutoSell", 1, function()
    Rayfield:Notify({
        Title = "Auto Sell Module",
        Content = "Status: " .. (Settings.AutoSellEnabled and "Active" or "Inactive"),
        Duration = 3,
        Image = "dollar-sign"
    })
end)

ModuleTab:CreateButton({
   Name = "Run Diagnostics",
   Callback = function()
       HUB.Modules["Diagnostics"].Execute()
   end,
})

-- Register Diagnostics module
HUB:RegisterModule("Diagnostics", 1, function()
    Rayfield:Notify({
        Title = "Diagnostics",
        Content = "FPS: "..fps.." | Modules: "..tostring((function()
            local c=0 for _ in pairs(HUB.Modules) do c=c+1 end return c end)()) .. 
            " | Auto Sell: " .. (Settings.AutoSellEnabled and "ON" or "OFF"),
        Duration = 4,
        Image = "activity"
    })
end)

--// SYSTEM TAB
SystemTab:CreateSection("System Controls")

SystemTab:CreateToggle({
    Name = "Debug Mode",
    CurrentValue = true,
    Flag = "DebugMode",
    Callback = function(val)
        debugX = val
        HUB:Log("Debug Mode: "..tostring(val))
    end,
})

SystemTab:CreateButton({
    Name = "Show Logs",
    Callback = function()
        for _,log in ipairs(HUB.Logs) do
            print(log)
        end
        Rayfield:Notify({
            Title = "Logs",
            Content = "Check console (F9)",
            Duration = 3,
            Image = "terminal"
        })
    end,
})

SystemTab:CreateButton({
    Name = "System Info",
    Callback = function()
        Rayfield:Notify({
            Title = "REV-HUB System",
            Content = "Version "..HUB.Version.." | FPS "..fps,
            Duration = 4,
            Image = "info"
        })
    end,
})

SystemTab:CreateSection("Remote Status")

SystemTab:CreateLabel("FishingSystem: " .. (FishingSystem and "✅" or "❌"))
SystemTab:CreateLabel("Sell Remote: " .. (SellFish and "✅" or "❌"))
SystemTab:CreateLabel("Equip Remote: " .. (EquipFish and "✅" or "❌"))

SystemTab:CreateButton({
    Name = "Rescan Remotes",
    Callback = function()
        HUB:Log("Rescanning remotes...")
        local newFishing = findFishingSystem()
        if newFishing then
            FishingSystem = newFishing
            SellFish = findSellRemote(FishingSystem)
            InventoryEvents = findInventoryEvents(FishingSystem)
            EquipFish = InventoryEvents and findEquipRemote(InventoryEvents)
            
            Settings.FishingSystem = FishingSystem
            Settings.SellFish = SellFish
            Settings.EquipFish = EquipFish
            
            Rayfield:Notify({
                Title = "Rescan Complete",
                Content = "FishingSystem: " .. (FishingSystem and "✅" or "❌"),
                Duration = 3,
                Image = "refresh-cw"
            })
        end
    end
})

--// INITIAL NOTIFICATION
Rayfield:Notify({
    Title = "REV-HUB v" .. HUB.Version,
    Content = "Auto Sell Multi Format Loaded!",
    Duration = 4,
    Image = "cpu"
})

HUB:Log("🔥 MULTI FORMAT VERSION JALAN!")
HUB:Log("🔍 FishingSystem: " .. (FishingSystem and "✅" or "❌"))
HUB:Log("🔍 SellFish: " .. (SellFish and "✅" or "❌"))
HUB:Log("🔍 EquipFish: " .. (EquipFish and "✅" or "❌"))

--// SAMPLE PERMISSION
HUB:SetPermission(game.Players.LocalPlayer.UserId, 2)

Rayfield:LoadConfiguration()
