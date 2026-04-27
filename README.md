-- Banana Cat Hub - Blox Fruits Script v9.0 FINAL
-- Silent Kill Auto-ON + Race V2/V3 + Haki 7 Màu + Moon Gaze Blue Gear + Clean UI
-- Copy toàn bộ vào executor

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local VIM = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local TeleportService = game:GetService("TeleportService")
local Lighting = game:GetService("Lighting")

-- ==================== SILENT KILL (Always ON) ====================
local SilentKillRange = 45
local SilentKillDamage = 20
local HitboxSize = 35
local FarmHeight = 35

local function SilentKillAlways()
    spawn(function()
        while true do
            task.wait(0.015)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                local hum = c:FindFirstChildOfClass("Humanoid")
                if not root or not hum or hum.Health <= 0 then return end
                
                local weapon = nil
                for _, t in ipairs(c:GetChildren()) do
                    if t:IsA("Tool") and t:FindFirstChild("Handle") then
                        weapon = t
                        break
                    end
                end
                if not weapon then
                    for _, t in ipairs(LP.Backpack:GetChildren()) do
                        if t:IsA("Tool") and t:FindFirstChild("Handle") then
                            hum:EquipTool(t)
                            weapon = t
                            break
                        end
                    end
                end
                
                if weapon and weapon:FindFirstChild("Handle") then
                    local h = weapon.Handle
                    if h:IsA("BasePart") then
                        h.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
                        h.Transparency = 0.95
                        h.CanCollide = false
                    end
                end
                
                local nearest = nil
                local nearestDist = SilentKillRange
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                        local eh = obj:FindFirstChildOfClass("Humanoid")
                        local er = obj:FindFirstChild("HumanoidRootPart")
                        if eh and er and eh.Health > 0 and obj ~= c and not Players:GetPlayerFromCharacter(obj) then
                            local dist = (root.Position - er.Position).Magnitude
                            if dist < nearestDist then
                                nearestDist = dist
                                nearest = obj
                            end
                        end
                    end
                end
                
                if nearest then
                    local er = nearest:FindFirstChild("HumanoidRootPart")
                    local eh = nearest:FindFirstChildOfClass("Humanoid")
                    if er and eh and eh.Health > 0 then
                        root.CFrame = er.CFrame * CFrame.new(math.random(-5,5), FarmHeight, math.random(-5,5))
                        VirtualUser:CaptureController()
                        VirtualUser:ClickButton1(Vector2.new())
                        if eh.Health > 50 then
                            eh.Health = math.max(0, eh.Health - SilentKillDamage)
                        end
                    end
                end
            end)
        end
    end)
end

-- ==================== SEA & RACE DETECTION ====================
local CurrentSea = "Unknown"
function DetectSea()
    pcall(function()
        local placeId = game.PlaceId
        if placeId == 2753915549 then CurrentSea = "Sea 1"
        elseif placeId == 4442272183 then CurrentSea = "Sea 2"
        elseif placeId == 7449423635 then CurrentSea = "Sea 3"
        elseif Workspace:FindFirstChild("HydraIsland") or Workspace:FindFirstChild("GreatTree") then CurrentSea = "Sea 3"
        elseif Workspace:FindFirstChild("KingdomOfRose") or Workspace:FindFirstChild("GreenZone") then CurrentSea = "Sea 2"
        elseif Workspace:FindFirstChild("StartIsland") then CurrentSea = "Sea 1"
        end
    end)
    return CurrentSea
end

local PlayerRace = "Unknown"
local PlayerRaceVersion = "Unknown"
function CheckCurrentRace()
    pcall(function()
        local c = LP.Character
        if c then
            if c:FindFirstChild("CyborgParts") then PlayerRace = "Cyborg"
            elseif c:FindFirstChild("GhoulMask") then PlayerRace = "Ghoul"
            elseif c:FindFirstChild("Wings") then PlayerRace = "Angel"
            elseif c:FindFirstChild("Tail") then PlayerRace = "Shark"
            elseif c:FindFirstChild("BunnyEars") then PlayerRace = "Rabbit"
            else PlayerRace = "Human" end
        end
        if RaceV3Completed then PlayerRaceVersion = "V3"
        elseif RaceV2Completed then PlayerRaceVersion = "V2"
        else PlayerRaceVersion = "V1" end
    end)
    return PlayerRace, PlayerRaceVersion
end

-- ==================== AUTO FARM ====================
local AutoFarmEnabled = false
local FastHitEnabled = false
local AutoBossEnabled = false
local AutoSkillsEnabled = false

local BossList = {
    ["Sea 1"] = {"The Gorilla King", "Bobby", "Yeti", "Mob Leader", "Vice Admiral", "Warden", "Chief Warden", "Swan", "Magma Admiral", "Fishman Lord"},
    ["Sea 2"] = {"Diamond", "Jeremy", "Fajita", "Don Swan", "Smoke Admiral", "Awakened Ice Admiral", "Tide Keeper", "Darkbeard", "Cursed Captain"},
    ["Sea 3"] = {"Stone", "Island Empress", "Kilo Admiral", "Captain Elephant", "Beautiful Pirate", "Longma", "Soul Reaper", "Cake Queen", "Rip Indra"}
}

local SkillsList = {"Z", "X", "C", "V", "F", "T"}
local SkillCooldowns = {Z = 5, X = 8, C = 10, V = 15, F = 12, T = 30}
local LastSkillUse = {}

local function FindBestWeapon()
    for _, t in ipairs(LP.Backpack:GetChildren()) do
        if t:IsA("Tool") and t:FindFirstChild("Handle") then return t end
    end
    return nil
end

local function HasWeapon(character)
    for _, t in ipairs(character:GetChildren()) do
        if t:IsA("Tool") and t:FindFirstChild("Handle") then return t end
    end
    return nil
end

local function AutoFarm()
    spawn(function()
        while AutoFarmEnabled do
            task.wait(FastHitEnabled and 0.01 or 0.1)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                local hum = c:FindFirstChildOfClass("Humanoid")
                if not root or not hum or hum.Health <= 0 then return end
                
                if not HasWeapon(c) then
                    local w = FindBestWeapon()
                    if w then hum:EquipTool(w) end
                end
                
                local nearest = nil
                local nearestDist = 200
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                        local eh = obj:FindFirstChildOfClass("Humanoid")
                        local er = obj:FindFirstChild("HumanoidRootPart")
                        if eh and er and eh.Health > 0 and obj ~= c and not Players:GetPlayerFromCharacter(obj) then
                            local isBoss = false
                            for _, bossName in ipairs(BossList[DetectSea()] or {}) do
                                if obj.Name == bossName then isBoss = true; break end
                            end
                            if not AutoBossEnabled or isBoss then
                                local dist = (root.Position - er.Position).Magnitude
                                if dist < nearestDist then
                                    nearestDist = dist
                                    nearest = obj
                                end
                            end
                        end
                    end
                end
                
                if nearest then
                    local er = nearest:FindFirstChild("HumanoidRootPart")
                    local eh = nearest:FindFirstChildOfClass("Humanoid")
                    if er and eh and eh.Health > 0 then
                        root.CFrame = er.CFrame * CFrame.new(0, FarmHeight, 0)
                        VirtualUser:CaptureController()
                        VirtualUser:ClickButton1(Vector2.new())
                        if FastHitEnabled and eh.Health > 50 then
                            eh.Health = math.max(0, eh.Health - 15)
                        end
                    end
                end
            end)
        end
    end)
end

local function AutoSkills()
    spawn(function()
        while AutoSkillsEnabled do
            task.wait(0.3)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local hum = c:FindFirstChildOfClass("Humanoid")
                if not hum or hum.Health <= 0 then return end
                
                for _, key in ipairs(SkillsList) do
                    if not AutoSkillsEnabled then break end
                    local ct = tick()
                    if ct - (LastSkillUse[key] or 0) >= (SkillCooldowns[key] or 5) then
                        local kc = Enum.KeyCode[key]
                        if kc then
                            VIM:SendKeyEvent(true, kc, false, game)
                            task.wait(0.1)
                            VIM:SendKeyEvent(false, kc, false, game)
                            LastSkillUse[key] = ct
                        end
                    end
                end
            end)
        end
    end)
end

-- ==================== AUTO RAID & DUNGEON ====================
local AutoRaidEnabled = false
local AutoDungeonEnabled = false

local function AutoRaid()
    spawn(function()
        while AutoRaidEnabled do
            task.wait(10)
            pcall(function()
                local c = LP.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    local root = c:FindFirstChild("HumanoidRootPart")
                    
                    if CurrentSea == "Sea 1" then root.CFrame = CFrame.new(-4900, 50, -400)
                    elseif CurrentSea == "Sea 2" then root.CFrame = CFrame.new(-5900, 10, -1800)
                    else root.CFrame = CFrame.new(-5000, 200, -500) end
                    task.wait(2)
                    
                    for _, npc in ipairs(Workspace:GetDescendants()) do
                        if npc:IsA("Model") and npc.Name:lower():find("scientist") then
                            local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                            if head then root.CFrame = head.CFrame * CFrame.new(0, 3, 4) end
                            task.wait(1)
                            local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                            if prompt then fireproximityprompt(prompt) end
                            break
                        end
                    end
                end
            end)
            task.wait(30)
        end
    end)
end

local function AutoDungeon()
    spawn(function()
        while AutoDungeonEnabled do
            task.wait(10)
            pcall(function()
                local c = LP.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    local root = c:FindFirstChild("HumanoidRootPart")
                    
                    if CurrentSea == "Sea 1" then root.CFrame = CFrame.new(-1800, 150, -5200)
                    elseif CurrentSea == "Sea 2" then root.CFrame = CFrame.new(-6100, 20, -1900)
                    else root.CFrame = CFrame.new(-4000, 350, -400) end
                    task.wait(2)
                    
                    for _, obj in ipairs(Workspace:GetDescendants()) do
                        if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") and obj:FindFirstChild("HumanoidRootPart") and obj ~= c and not Players:GetPlayerFromCharacter(obj) then
                            local eh = obj:FindFirstChildOfClass("Humanoid")
                            local er = obj:FindFirstChild("HumanoidRootPart")
                            if eh and er and eh.Health > 0 then
                                root.CFrame = er.CFrame * CFrame.new(0, FarmHeight, 0)
                                eh.Health = math.max(0, eh.Health - 80)
                                task.wait(0.08)
                            end
                        end
                    end
                end
            end)
            task.wait(30)
        end
    end)
end

-- ==================== FIND NEAREST ENEMY HELPER ====================
local function FindNearestEnemyForRace(pos, range)
    local nearest, nearestDist = nil, range
    pcall(function()
        local c = LP.Character
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                local hum = obj:FindFirstChildOfClass("Humanoid")
                local objRoot = obj:FindFirstChild("HumanoidRootPart")
                if hum and objRoot and hum.Health > 0 and obj ~= c and not Players:GetPlayerFromCharacter(obj) then
                    local dist = (pos - objRoot.Position).Magnitude
                    if dist < nearestDist then
                        nearestDist = dist
                        nearest = obj
                    end
                end
            end
        end
    end)
    return nearest
end

-- ==================== AUTO RACE V2 (ALCHEMIST) ====================
local AutoRaceV2Enabled = false
local BlueFlowerCollected = false
local RedFlowerCollected = false
local YellowFlowerCollected = false
local RaceV2Completed = false

local AlchemistPos = CFrame.new(-1000, 50, -6500)

local FlowerPositions = {
    ["Blue"] = CFrame.new(-2800, 50, -5600),
    ["Red"] = CFrame.new(-1200, 50, -6200),
    ["Yellow"] = CFrame.new(-1500, 50, -6000)
}

local function GetCurrentTime()
    local currentTime = Lighting:GetMinutesAfterMidnight()
    local hour = math.floor(currentTime / 60)
    return hour >= 6 and hour < 18 and "Day" or "Night"
end

local function CollectBlueFlower()
    if BlueFlowerCollected then return true end
    if GetCurrentTime() ~= "Night" then return false end
    
    pcall(function()
        local c = LP.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        root.CFrame = FlowerPositions["Blue"]
        task.wait(3)
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Tool") and obj.Name:lower():find("blue") and obj.Name:lower():find("flower") then
                local handle = obj:FindFirstChild("Handle")
                if handle then
                    root.CFrame = handle.CFrame * CFrame.new(0, 5, 0)
                    firetouchinterest(root, handle, 0)
                    task.wait(0.1)
                    firetouchinterest(root, handle, 1)
                    BlueFlowerCollected = true
                    print("🔵 Đã nhặt Hoa Xanh!")
                    return true
                end
            end
        end
    end)
    return false
end

local function CollectRedFlower()
    if RedFlowerCollected then return true end
    if GetCurrentTime() ~= "Day" then return false end
    
    pcall(function()
        local c = LP.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        root.CFrame = FlowerPositions["Red"]
        task.wait(3)
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Tool") and obj.Name:lower():find("red") and obj.Name:lower():find("flower") then
                local handle = obj:FindFirstChild("Handle")
                if handle then
                    root.CFrame = handle.CFrame * CFrame.new(0, 5, 0)
                    firetouchinterest(root, handle, 0)
                    task.wait(0.1)
                    firetouchinterest(root, handle, 1)
                    RedFlowerCollected = true
                    print("🔴 Đã nhặt Hoa Đỏ!")
                    return true
                end
            end
        end
    end)
    return false
end

local function CollectYellowFlower()
    if YellowFlowerCollected then return true end
    
    pcall(function()
        local c = LP.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local attempts = 0
        while attempts < 50 do
            local enemy = FindNearestEnemyForRace(root.Position, 200)
            if enemy then
                local er = enemy:FindFirstChild("HumanoidRootPart")
                local eh = enemy:FindFirstChildOfClass("Humanoid")
                if er and eh and eh.Health > 0 then
                    root.CFrame = er.CFrame * CFrame.new(0, 35, 0)
                    VirtualUser:CaptureController()
                    VirtualUser:ClickButton1(Vector2.new())
                    eh.Health = 0
                    task.wait(0.1)
                end
            end
            
            for _, obj in ipairs(Workspace:GetDescendants()) do
                if obj:IsA("Tool") and obj.Name:lower():find("yellow") and obj.Name:lower():find("flower") then
                    local handle = obj:FindFirstChild("Handle")
                    if handle and (root.Position - handle.Position).Magnitude < 20 then
                        firetouchinterest(root, handle, 0)
                        task.wait(0.1)
                        firetouchinterest(root, handle, 1)
                        YellowFlowerCollected = true
                        print("🟡 Đã nhặt Hoa Vàng!")
                        return true
                    end
                end
            end
            attempts = attempts + 1
            task.wait(0.5)
        end
    end)
    return false
end

local function RaceV2Upgrade()
    spawn(function()
        while AutoRaceV2Enabled and not RaceV2Completed do
            task.wait(5)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                if not root then return end
                
                CheckCurrentRace()
                
                if PlayerRaceVersion ~= "V1" then
                    AutoRaceV2Enabled = false
                    RaceV2Completed = true
                    return
                end
                
                root.CFrame = AlchemistPos
                task.wait(3)
                
                local alchemistFound = false
                for _, npc in ipairs(Workspace:GetDescendants()) do
                    if npc:IsA("Model") and npc.Name:lower():find("alchemist") then
                        local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                        if head then
                            root.CFrame = head.CFrame * CFrame.new(0, 5, 3)
                            task.wait(1)
                            local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                            if prompt then
                                fireproximityprompt(prompt)
                                task.wait(2)
                                alchemistFound = true
                            end
                        end
                        break
                    end
                end
                
                if not alchemistFound then return end
                
                while not BlueFlowerCollected do
                    CollectBlueFlower()
                    if not BlueFlowerCollected then task.wait(10) end
                end
                
                while not RedFlowerCollected do
                    CollectRedFlower()
                    if not RedFlowerCollected then task.wait(10) end
                end
                
                while not YellowFlowerCollected do
                    CollectYellowFlower()
                    task.wait(3)
                end
                
                root.CFrame = AlchemistPos
                task.wait(3)
                
                for _, npc in ipairs(Workspace:GetDescendants()) do
                    if npc:IsA("Model") and npc.Name:lower():find("alchemist") then
                        local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                        if head then
                            root.CFrame = head.CFrame * CFrame.new(0, 5, 3)
                            task.wait(1)
                            local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                            if prompt then
                                fireproximityprompt(prompt)
                                task.wait(2)
                                RaceV2Completed = true
                                PlayerRaceVersion = "V2"
                                AutoRaceV2Enabled = false
                                print("🎉 ĐÃ NÂNG CẤP LÊN V2!")
                            end
                        end
                        break
                    end
                end
            end)
            task.wait(30)
        end
    end)
end

-- ==================== AUTO RACE V3 (AROWE) ====================
local AutoRaceV3Enabled = false
local RaceV3QuestStarted = false
local RaceV3Completed = false

local ArowePos = CFrame.new(-2600, 30, -4500)

local HumanV3Bosses = {
    {name = "Diamond", pos = CFrame.new(-3000, 50, -3500)},
    {name = "Jeremy", pos = CFrame.new(-4000, 50, -3000)},
    {name = "Fajita", pos = CFrame.new(-3500, 50, -4000)}
}

local function KillBossForV3(bossName, bossPos)
    pcall(function()
        local c = LP.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        root.CFrame = bossPos
        task.wait(2)
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Model") and obj.Name == bossName and obj:FindFirstChildOfClass("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                local eh = obj:FindFirstChildOfClass("Humanoid")
                local er = obj:FindFirstChild("HumanoidRootPart")
                if eh and er and eh.Health > 0 then
                    while eh.Health > 0 do
                        root.CFrame = er.CFrame * CFrame.new(0, 10, 0)
                        VirtualUser:CaptureController()
                        VirtualUser:ClickButton1(Vector2.new())
                        eh.Health = math.max(0, eh.Health - 30)
                        task.wait(0.05)
                    end
                    return true
                end
            end
        end
    end)
    return false
end

local function RaceV3Upgrade()
    spawn(function()
        while AutoRaceV3Enabled and not RaceV3Completed do
            task.wait(5)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                if not root then return end
                
                CheckCurrentRace()
                
                if PlayerRaceVersion == "V3" then
                    AutoRaceV3Enabled = false
                    RaceV3Completed = true
                    return
                end
                
                root.CFrame = ArowePos
                task.wait(3)
                
                local aroweNPC = nil
                for _, npc in ipairs(Workspace:GetDescendants()) do
                    if npc:IsA("Model") and (npc.Name:lower():find("arowe") or npc.Name:lower():find("aaro")) then
                        aroweNPC = npc
                        local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                        if head then
                            root.CFrame = head.CFrame * CFrame.new(0, 5, 3)
                            task.wait(1)
                            local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                            if prompt then
                                fireproximityprompt(prompt)
                                task.wait(2)
                                RaceV3QuestStarted = true
                            end
                        end
                        break
                    end
                end
                
                if not aroweNPC then return end
                
                -- Quest theo tộc
                if PlayerRace == "Human" then
                    for _, boss in ipairs(HumanV3Bosses) do
                        KillBossForV3(boss.name, boss.pos)
                        task.wait(2)
                    end
                elseif PlayerRace == "Angel" then
                    for _, player in ipairs(Players:GetPlayers()) do
                        if player ~= LP and player.Character and player.Character:FindFirstChild("Wings") then
                            local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
                            if targetRoot then
                                root.CFrame = targetRoot.CFrame * CFrame.new(0, 5, 0)
                                VirtualUser:CaptureController()
                                VirtualUser:ClickButton1(Vector2.new())
                                task.wait(0.5)
                                break
                            end
                        end
                    end
                elseif PlayerRace == "Rabbit" then
                    for i = 1, 30 do
                        for _, chest in ipairs(Workspace:GetDescendants()) do
                            if chest:IsA("Model") and chest.Name:lower():find("chest") then
                                local chestPart = chest:FindFirstChildWhichIsA("BasePart")
                                if chestPart then
                                    root.CFrame = chestPart.CFrame * CFrame.new(0, 3, 0)
                                    local prompt = chest:FindFirstChildOfClass("ProximityPrompt")
                                    if prompt then fireproximityprompt(prompt) end
                                    task.wait(0.1)
                                    break
                                end
                            end
                        end
                        task.wait(0.5)
                    end
                elseif PlayerRace == "Shark" then
                    root.CFrame = CFrame.new(-5000, 10, 5000)
                    task.wait(5)
                    for _, obj in ipairs(Workspace:GetDescendants()) do
                        if obj:IsA("Model") and obj.Name:lower():find("sea") and obj.Name:lower():find("beast") and obj:FindFirstChildOfClass("Humanoid") then
                            local lr = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
                            if lr then
                                root.CFrame = lr.CFrame * CFrame.new(0, 40, 0)
                                obj:FindFirstChildOfClass("Humanoid").Health = 0
                                break
                            end
                        end
                    end
                elseif PlayerRace == "Ghoul" then
                    local killCount = 0
                    for _, player in ipairs(Players:GetPlayers()) do
                        if player ~= LP and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and killCount < 5 then
                            local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
                            root.CFrame = targetRoot.CFrame * CFrame.new(0, 5, 0)
                            VirtualUser:CaptureController()
                            VirtualUser:ClickButton1(Vector2.new())
                            killCount = killCount + 1
                            task.wait(1)
                        end
                    end
                elseif PlayerRace == "Cyborg" then
                    for _, obj in ipairs(Workspace:GetDescendants()) do
                        if obj:IsA("Tool") and obj.Name:lower():find("fruit") then
                            local handle = obj:FindFirstChild("Handle")
                            if handle then
                                root.CFrame = handle.CFrame * CFrame.new(0, 3, 0)
                                firetouchinterest(root, handle, 0)
                                task.wait(0.1)
                                firetouchinterest(root, handle, 1)
                                break
                            end
                        end
                    end
                end
                
                root.CFrame = ArowePos
                task.wait(3)
                
                if aroweNPC then
                    local head = aroweNPC:FindFirstChild("Head") or aroweNPC:FindFirstChild("HumanoidRootPart")
                    if head then
                        root.CFrame = head.CFrame * CFrame.new(0, 5, 3)
                        task.wait(1)
                        local prompt = aroweNPC:FindFirstChildOfClass("ProximityPrompt")
                        if prompt then
                            fireproximityprompt(prompt)
                            task.wait(2)
                            RaceV3Completed = true
                            PlayerRaceVersion = "V3"
                            AutoRaceV3Enabled = false
                            print("🎉 ĐÃ NÂNG CẤP LÊN V3!")
                        end
                    end
                end
            end)
            task.wait(30)
        end
    end)
end

-- ==================== HAKI 7 MÀU SYSTEM ====================
local AutoHakiQuest = false
local HakiQuestStarted = false
local HakiQuestCompleted = false
local RainbowHakiUnlocked = false
local CurrentQuestBoss = 1
local BossKillCount = 0
local AllBossesKilled = false

local HakiBosses = {
    [1] = {name = "Stone", killed = false, position = CFrame.new(-3000, 200, -4000)},
    [2] = {name = "Hydra Leader", killed = false, position = CFrame.new(-4500, 300, -3500)},
    [3] = {name = "Kilo Admiral", killed = false, position = CFrame.new(-5500, 200, -2000)},
    [4] = {name = "Captain Elephant", killed = false, position = CFrame.new(-3500, 250, -5000)},
    [5] = {name = "Beautiful Pirate", killed = false, position = CFrame.new(-4000, 200, -3000)}
}

local function TalkToHornedMan()
    pcall(function()
        local c = LP.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        for _, npc in ipairs(Workspace:GetDescendants()) do
            if npc:IsA("Model") and (npc.Name:lower():find("horned") or npc.Name:lower():find("horn")) then
                local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                if head then
                    root.CFrame = head.CFrame * CFrame.new(0, 5, 3)
                    task.wait(1)
                    local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                    if prompt then
                        fireproximityprompt(prompt)
                        task.wait(2)
                        HakiQuestStarted = true
                        return true
                    end
                end
            end
        end
        
        root.CFrame = CFrame.new(-5000, 300, -6000)
        task.wait(3)
        
        for _, npc in ipairs(Workspace:GetDescendants()) do
            if npc:IsA("Model") and npc.Name:lower():find("horned") then
                local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                if head then
                    root.CFrame = head.CFrame * CFrame.new(0, 5, 3)
                    task.wait(1)
                    local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                    if prompt then
                        fireproximityprompt(prompt)
                        task.wait(2)
                        HakiQuestStarted = true
                        return true
                    end
                end
            end
        end
    end)
    return false
end

local function KillHakiBoss(bossNumber)
    local bossData = HakiBosses[bossNumber]
    if not bossData then return end
    
    pcall(function()
        local c = LP.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        root.CFrame = bossData.position
        task.wait(2)
        
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Model") and obj.Name == bossData.name and obj:FindFirstChildOfClass("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                local eh = obj:FindFirstChildOfClass("Humanoid")
                local er = obj:FindFirstChild("HumanoidRootPart")
                if eh and er and eh.Health > 0 then
                    while eh.Health > 0 do
                        root.CFrame = er.CFrame * CFrame.new(0, 10, 0)
                        VirtualUser:CaptureController()
                        VirtualUser:ClickButton1(Vector2.new())
                        eh.Health = math.max(0, eh.Health - 30)
                        task.wait(0.05)
                    end
                    HakiBosses[bossNumber].killed = true
                    BossKillCount = BossKillCount + 1
                    break
                end
            end
        end
    end)
end

local function TalkToBarista()
    pcall(function()
        local c = LP.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        for _, npc in ipairs(Workspace:GetDescendants()) do
            if npc:IsA("Model") and npc.Name:lower():find("barista") then
                local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                if head then
                    root.CFrame = head.CFrame * CFrame.new(0, 5, 3)
                    task.wait(1)
                    local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                    if prompt then
                        fireproximityprompt(prompt)
                        task.wait(2)
                        RainbowHakiUnlocked = true
                        HakiQuestCompleted = true
                        return true
                    end
                end
            end
        end
    end)
    return false
end

local function HakiQuestManager()
    spawn(function()
        while AutoHakiQuest and not HakiQuestCompleted do
            task.wait(5)
            pcall(function()
                if not HakiQuestStarted then
                    TalkToHornedMan()
                    return
                end
                
                if HakiQuestStarted and not AllBossesKilled then
                    for i = 1, 5 do
                        if not HakiBosses[i].killed then
                            CurrentQuestBoss = i
                            KillHakiBoss(i)
                            task.wait(3)
                        end
                    end
                    
                    AllBossesKilled = true
                    for i = 1, 5 do
                        if not HakiBosses[i].killed then
                            AllBossesKilled = false
                            break
                        end
                    end
                end
                
                if AllBossesKilled and not RainbowHakiUnlocked then
                    TalkToBarista()
                end
            end)
            task.wait(10)
        end
    end)
end

-- ==================== FARM BERRY ====================
local AutoFarmBerryEnabled = false
local BerryCount = 0

local function FarmBerry()
    spawn(function()
        while AutoFarmBerryEnabled do
            task.wait(2)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                if not root then return end
                
                local berryIslands = {"HydraIsland", "GreatTree", "FloatingTurtle", "CastleOnTheSea", "PortTown"}
                
                for _, islandName in ipairs(berryIslands) do
                    local island = Workspace:FindFirstChild(islandName)
                    if island then
                        for _, obj in ipairs(island:GetDescendants()) do
                            if obj:IsA("Tool") and obj.Name:lower():find("berry") then
                                local handle = obj:FindFirstChild("Handle")
                                if handle then
                                    root.CFrame = handle.CFrame * CFrame.new(0, 3, 0)
                                    firetouchinterest(root, handle, 0)
                                    task.wait(0.1)
                                    firetouchinterest(root, handle, 1)
                                    BerryCount = BerryCount + 1
                                    task.wait(0.5)
                                end
                            end
                        end
                    end
                end
            end)
        end
    end)
end

-- ==================== AUTO ISLANDS ====================
local AutoMysteryIsland = false
local AutoKitsuneIsland = false
local AutoPrehistoricIsland = false
local AutoLeviathan = false

local function AutoIsland(islandType)
    spawn(function()
        local findFunc
        if islandType == "Mystery" then
            findFunc = function()
                local list = {}
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("Model") and (obj.Name:lower():find("mystery") or obj.Name:lower():find("haunted")) then
                        table.insert(list, obj)
                    end
                end
                return list
            end
        elseif islandType == "Kitsune" then
            findFunc = function()
                local list = {}
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj.Name:lower():find("kitsune") then
                        table.insert(list, obj)
                    end
                end
                return list
            end
        elseif islandType == "Prehistoric" then
            findFunc = function()
                local list = {}
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj.Name:lower():find("prehistoric") then
                        table.insert(list, obj)
                    end
                end
                return list
            end
        else return end
        
        while (islandType == "Mystery" and AutoMysteryIsland) or (islandType == "Kitsune" and AutoKitsuneIsland) or (islandType == "Prehistoric" and AutoPrehistoricIsland) do
            task.wait(10)
            pcall(function()
                local c = LP.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    local root = c:FindFirstChild("HumanoidRootPart")
                    local islands = findFunc()
                    if #islands > 0 then
                        local part = islands[1]:FindFirstChildWhichIsA("BasePart")
                        if part then root.CFrame = part.CFrame * CFrame.new(0, 15, 0) end
                    end
                end
            end)
        end
    end)
end

local function LeviathanFarm()
    spawn(function()
        while AutoLeviathan do
            task.wait(10)
            pcall(function()
                local c = LP.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    local root = c:FindFirstChild("HumanoidRootPart")
                    for _, obj in ipairs(Workspace:GetDescendants()) do
                        if obj:IsA("Model") and obj.Name:lower():find("levi") and obj:FindFirstChildOfClass("Humanoid") then
                            local lr = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
                            if lr then
                                root.CFrame = lr.CFrame * CFrame.new(0, 40, 0)
                                obj:FindFirstChildOfClass("Humanoid").Health = math.max(0, obj:FindFirstChildOfClass("Humanoid").Health - 100)
                            end
                            break
                        end
                    end
                end
            end)
        end
    end)
end

-- ==================== AUTO MOON GAZE + BLUE GEAR ====================
local AutoMoonGaze = false
local AutoBlueGear = false
local BlueGearCollected = false
local MirageFound = false
local MoonGazeCompleted = false

local MirageSpawnPositions = {
    CFrame.new(-5000, 10, 5000),
    CFrame.new(-3000, 10, 8000),
    CFrame.new(-7000, 10, 6000),
    CFrame.new(-4000, 10, 10000),
    CFrame.new(-6000, 10, 12000),
}

local function IsNightTime()
    local currentTime = Lighting:GetMinutesAfterMidnight()
    local hour = math.floor(currentTime / 60)
    return hour >= 18 or hour < 6
end

local function MoonGaze()
    spawn(function()
        while AutoMoonGaze do
            task.wait(5)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                if not root then return end
                
                if not IsNightTime() then
                    print("☀️ Chưa phải đêm! Đợi...")
                    task.wait(10)
                    return
                end
                
                local mirageIsland = nil
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("Model") and obj.Name:lower():find("mirage") then
                        mirageIsland = obj
                        MirageFound = true
                        break
                    end
                end
                
                if not mirageIsland then
                    local randomPos = MirageSpawnPositions[math.random(1, #MirageSpawnPositions)]
                    root.CFrame = randomPos
                    task.wait(5)
                end
                
                if MirageFound and mirageIsland then
                    local highestPart, highestY = nil, -999
                    for _, part in ipairs(mirageIsland:GetDescendants()) do
                        if part:IsA("BasePart") and part.Position.Y > highestY then
                            highestY = part.Position.Y
                            highestPart = part
                        end
                    end
                    
                    if highestPart then
                        root.CFrame = highestPart.CFrame * CFrame.new(0, 10, 0)
                        task.wait(2)
                        
                        VIM:SendKeyEvent(true, Enum.KeyCode.T, false, game)
                        task.wait(0.1)
                        VIM:SendKeyEvent(false, Enum.KeyCode.T, false, game)
                        
                        local gazeTime = 0
                        while gazeTime < 15 and AutoMoonGaze do
                            task.wait(1)
                            gazeTime = gazeTime + 1
                        end
                        
                        MoonGazeCompleted = true
                        if not AutoBlueGear then
                            AutoBlueGear = true
                            BlueGearCollector()
                        end
                    end
                end
            end)
            task.wait(30)
        end
    end)
end

local function BlueGearCollector()
    spawn(function()
        while AutoBlueGear do
            task.wait(3)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                if not root then return end
                
                if BlueGearCollected then
                    AutoBlueGear = false
                    return
                end
                
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if (obj:IsA("Tool") or obj:IsA("BasePart")) and obj.Name:lower():find("blue") and obj.Name:lower():find("gear") then
                        local handle = obj:IsA("Tool") and obj:FindFirstChild("Handle") or obj
                        root.CFrame = handle.CFrame * CFrame.new(0, 3, 0)
                        firetouchinterest(root, handle, 0)
                        task.wait(0.1)
                        firetouchinterest(root, handle, 1)
                        BlueGearCollected = true
                        print("✅ Đã nhặt Blue Gear!")
                        break
                    end
                end
            end)
        end
    end)
end

-- ==================== STACK FARMING ====================
local AutoStackFarm = false
local function StackFarm()
    spawn(function()
        while AutoStackFarm do
            task.wait(5)
            pcall(function()
                local c = LP.Character
                if not c then return end
                local root = c:FindFirstChild("HumanoidRootPart")
                if not root then return end
                
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LP and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
                        root.CFrame = targetRoot.CFrame * CFrame.new(0, FarmHeight, 0)
                        VirtualUser:CaptureController()
                        VirtualUser:ClickButton1(Vector2.new())
                        task.wait(0.05)
                    end
                end
            end)
        end
    end)
end

-- ==================== UI ====================
local isUIOpen = true

local function CreateUI()
    if CoreGui:FindFirstChild("BananaHub") then CoreGui:FindFirstChild("BananaHub"):Destroy() end
    
    local BananaGui = Instance.new("ScreenGui")
    BananaGui.Name = "BananaHub"
    BananaGui.Parent = CoreGui
    BananaGui.ResetOnSpawn = false
    
    local Main = Instance.new("Frame")
    Main.Name = "Main"
    Main.Parent = BananaGui
    Main.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
    Main.BorderSizePixel = 0
    Main.Position = UDim2.new(0.5, -310, 0.5, -250)
    Main.Size = UDim2.new(0, 620, 0, 500)
    Main.Active = true
    Main.Visible = true
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
    
    local Header = Instance.new("Frame")
    Header.Name = "Header"
    Header.Parent = Main
    Header.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
    Header.BorderSizePixel = 0
    Header.Size = UDim2.new(1, 0, 0, 50)
    Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 12)
    
    local Logo = Instance.new("Frame")
    Logo.Name = "Logo"
    Logo.Parent = Header
    Logo.BackgroundColor3 = Color3.fromRGB(255, 180, 30)
    Logo.BorderSizePixel = 0
    Logo.Size = UDim2.new(0, 34, 0, 34)
    Logo.Position = UDim2.new(0, 12, 0.5, -17)
    Instance.new("UICorner", Logo).CornerRadius = UDim.new(0, 8)
    
    local LogoText = Instance.new("TextLabel")
    LogoText.Parent = Logo
    LogoText.Text = "B"
    LogoText.TextColor3 = Color3.fromRGB(0, 0, 0)
    LogoText.BackgroundTransparency = 1
    LogoText.Size = UDim2.new(1, 0, 1, 0)
    LogoText.Font = Enum.Font.SourceSansBold
    LogoText.TextSize = 18
    
    local Title = Instance.new("TextLabel")
    Title.Parent = Header
    Title.Text = "Banana Cat Hub - Blox Fruit"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.BackgroundTransparency = 1
    Title.Size = UDim2.new(0, 250, 1, 0)
    Title.Position = UDim2.new(0, 54, 0, 0)
    Title.Font = Enum.Font.SourceSansBold
    Title.TextSize = 15
    Title.TextXAlignment = Enum.TextXAlignment.Left
    
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Parent = Header
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = Color3.fromRGB(180, 180, 190)
    CloseBtn.BackgroundTransparency = 1
    CloseBtn.Size = UDim2.new(0, 36, 0, 36)
    CloseBtn.Position = UDim2.new(1, -42, 0.5, -18)
    CloseBtn.Font = Enum.Font.SourceSansBold
    CloseBtn.TextSize = 16
    
    local SearchFrame = Instance.new("Frame")
    SearchFrame.Parent = Main
    SearchFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 34)
    SearchFrame.BorderSizePixel = 0
    SearchFrame.Position = UDim2.new(0, 10, 0, 58)
    SearchFrame.Size = UDim2.new(1, -20, 0, 36)
    Instance.new("UICorner", SearchFrame).CornerRadius = UDim.new(0, 8)
    
    local SearchBox = Instance.new("TextBox")
    SearchBox.Parent = SearchFrame
    SearchBox.PlaceholderText = "Search section or Fun"
    SearchBox.PlaceholderColor3 = Color3.fromRGB(100, 100, 110)
    SearchBox.Text = ""
    SearchBox.TextColor3 = Color3.fromRGB(200, 200, 210)
    SearchBox.BackgroundTransparency = 1
    SearchBox.Position = UDim2.new(0, 12, 0, 0)
    SearchBox.Size = UDim2.new(1, -24, 1, 0)
    SearchBox.Font = Enum.Font.SourceSans
    SearchBox.TextSize = 13
    SearchBox.TextXAlignment = Enum.TextXAlignment.Left
    
    local Sidebar = Instance.new("Frame")
    Sidebar.Parent = Main
    Sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 26)
    Sidebar.BorderSizePixel = 0
    Sidebar.Position = UDim2.new(0, 0, 0, 102)
    Sidebar.Size = UDim2.new(0, 180, 1, -102)
    
    local Content = Instance.new("ScrollingFrame")
    Content.Parent = Main
    Content.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
    Content.BorderSizePixel = 0
    Content.Position = UDim2.new(0, 181, 0, 102)
    Content.Size = UDim2.new(1, -181, 1, -102)
    Content.CanvasSize = UDim2.new(0, 0, 0, 800)
    Content.ScrollBarThickness = 4
    Content.ScrollBarImageColor3 = Color3.fromRGB(60, 60, 70)
    
    local sections = {
        "Shop",
        "Status And Server",
        "LocalPlayer",
        "Setting Farm",
        "Farming",
        "Race Upgrade",
        "Stack Farming",
        "Farming Other",
        "Fruit and Raid",
        "Haki 7 Mau (Rainbow)",
        "Race V4 (Blue Gear)",
        "Sea Event"
    }
    
    local selected = "Setting Farm"
    local sidebarBtns = {}
    
    for i, sec in ipairs(sections) do
        local btn = Instance.new("TextButton")
        btn.Parent = Sidebar
        btn.Text = ""
        btn.BackgroundColor3 = sec == selected and Color3.fromRGB(255, 180, 30) or Color3.fromRGB(20, 20, 26)
        btn.BorderSizePixel = 0
        btn.Size = UDim2.new(1, -6, 0, 36)
        btn.Position = UDim2.new(0, 3, 0, 8 + (i-1) * 42)
        btn.AutoButtonColor = false
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
        
        local lbl = Instance.new("TextLabel")
        lbl.Parent = btn
        lbl.Text = "  " .. sec
        lbl.TextColor3 = sec == selected and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(200, 200, 210)
        lbl.BackgroundTransparency = 1
        lbl.Size = UDim2.new(1, 0, 1, 0)
        lbl.Font = Enum.Font.SourceSans
        lbl.TextSize = 12
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        
        table.insert(sidebarBtns, {btn = btn, sec = sec, lbl = lbl})
        
        btn.MouseButton1Click:Connect(function()
            selected = sec
            for _, sb in ipairs(sidebarBtns) do
                local isSel = sb.sec == sec
                sb.btn.BackgroundColor3 = isSel and Color3.fromRGB(255, 180, 30) or Color3.fromRGB(20, 20, 26)
                sb.lbl.TextColor3 = isSel and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(200, 200, 210)
            end
            LoadContent(sec)
        end)
    end
    
    function LoadContent(secName)
        for _, child in ipairs(Content:GetChildren()) do child:Destroy() end
        local y = 15
        
        local function AddSectionTitle(text)
            local f = Instance.new("Frame")
            f.Parent = Content
            f.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
            f.BorderSizePixel = 0
            f.Position = UDim2.new(0, 10, 0, y)
            f.Size = UDim2.new(1, -20, 0, 28)
            Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = f
            lbl.Text = "  " .. text
            lbl.TextColor3 = Color3.fromRGB(255, 180, 30)
            lbl.BackgroundTransparency = 1
            lbl.Size = UDim2.new(1, 0, 1, 0)
            lbl.Font = Enum.Font.SourceSansBold
            lbl.TextSize = 12
            lbl.TextXAlignment = Enum.TextXAlignment.Left
            y = y + 34
        end
        
        local function AddToggle(text, default, callback)
            local f = Instance.new("Frame")
            f.Parent = Content
            f.BackgroundColor3 = Color3.fromRGB(28, 28, 34)
            f.BorderSizePixel = 0
            f.Position = UDim2.new(0, 10, 0, y)
            f.Size = UDim2.new(1, -20, 0, 36)
            Instance.new("UICorner", f).CornerRadius = UDim.new(0, 8)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = f
            lbl.Text = "  " .. text
            lbl.TextColor3 = Color3.fromRGB(200, 200, 210)
            lbl.BackgroundTransparency = 1
            lbl.Size = UDim2.new(0.68, 0, 1, 0)
            lbl.Font = Enum.Font.SourceSans
            lbl.TextSize = 11
            lbl.TextXAlignment = Enum.TextXAlignment.Left
            local tb = Instance.new("TextButton")
            tb.Parent = f
            tb.Text = default and "ON" or "OFF"
            tb.TextColor3 = Color3.fromRGB(255, 255, 255)
            tb.BackgroundColor3 = default and Color3.fromRGB(50, 180, 50) or Color3.fromRGB(180, 50, 50)
            tb.BorderSizePixel = 0
            tb.Size = UDim2.new(0, 44, 0, 24)
            tb.Position = UDim2.new(1, -54, 0.5, -12)
            tb.Font = Enum.Font.SourceSansBold
            tb.TextSize = 10
            tb.AutoButtonColor = false
            Instance.new("UICorner", tb).CornerRadius = UDim.new(0, 12)
            local en = default or false
            tb.MouseButton1Click:Connect(function()
                en = not en
                tb.Text = en and "ON" or "OFF"
                tb.BackgroundColor3 = en and Color3.fromRGB(50, 180, 50) or Color3.fromRGB(180, 50, 50)
                callback(en)
            end)
            y = y + 42
        end
        
        local function AddButton(text, callback)
            local btn = Instance.new("TextButton")
            btn.Parent = Content
            btn.Text = ""
            btn.BackgroundColor3 = Color3.fromRGB(28, 28, 34)
            btn.BorderSizePixel = 0
            btn.Position = UDim2.new(0, 10, 0, y)
            btn.Size = UDim2.new(1, -20, 0, 36)
            btn.AutoButtonColor = false
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
            local lbl = Instance.new("TextLabel")
            lbl.Parent = btn
            lbl.Text = "  " .. text
            lbl.TextColor3 = Color3.fromRGB(200, 200, 210)
            lbl.BackgroundTransparency = 1
            lbl.Size = UDim2.new(1, 0, 1, 0)
            lbl.Font = Enum.Font.SourceSans
            lbl.TextSize = 12
            lbl.TextXAlignment = Enum.TextXAlignment.Left
            btn.MouseButton1Click:Connect(callback)
            y = y + 42
        end
        
        -- ==================== RENDER CONTENT ====================
        if secName == "Shop" then
            AddSectionTitle("Shop & Teleport")
            AddButton("Misc Shop", function()
                pcall(function()
                    local c = LP.Character
                    if c and c:FindFirstChild("HumanoidRootPart") then
                        for _, npc in ipairs(Workspace:GetDescendants()) do
                            if npc:IsA("Model") and npc.Name:lower():find("shop") and npc:FindFirstChild("Head") then
                                c.HumanoidRootPart.CFrame = npc.Head.CFrame * CFrame.new(0, 3, 4)
                                break
                            end
                        end
                    end
                end)
            end)
            AddButton("Redeem Code", function() print("Redeem Code System") end)
            AddButton("Teleport Old World", function() TeleportService:Teleport(2753915549) end)
            AddButton("Teleport New World", function() TeleportService:Teleport(4442272183) end)
            AddButton("Teleport Third Sea", function() TeleportService:Teleport(7449423635) end)
            AddButton("Buy Dual Flintlock", function() print("Buy Dual Flintlock") end)
            AddButton("Reroll Race", function()
                local c = LP.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    for _, npc in ipairs(Workspace:GetDescendants()) do
                        if npc:IsA("Model") and npc.Name:lower():find("reroll") then
                            local head = npc:FindFirstChild("Head") or npc:FindFirstChild("HumanoidRootPart")
                            if head then
                                c.HumanoidRootPart.CFrame = head.CFrame * CFrame.new(0, 3, 4)
                                local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
                                if prompt then fireproximityprompt(prompt) end
                            end
                            break
                        end
                    end
                end
            end)
            
        elseif secName == "Status And Server" then
            DetectSea()
            CheckCurrentRace()
            AddButton("Sea: " .. CurrentSea, function() end)
            AddButton("Race: " .. PlayerRace .. " " .. PlayerRaceVersion, function() end)
            AddButton("Rejoin Server", function() TeleportService:Teleport(game.PlaceId) end)
            AddButton("Hop Server", function() TeleportService:Teleport(game.PlaceId) end)
            
        elseif secName == "LocalPlayer" then
            AddToggle("WalkSpeed Boost", false, function(v)
                local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
                if hum then hum.WalkSpeed = v and 100 or 16 end
            end)
            AddToggle("JumpPower Boost", false, function(v)
                local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
                if hum then hum.JumpPower = v and 100 or 50 end
            end)
            AddToggle("NoClip", false, function(v)
                spawn(function()
                    while v do
                        pcall(function()
                            local c = LP.Character
                            if c then
                                for _, part in ipairs(c:GetDescendants()) do
                                    if part:IsA("BasePart") then part.CanCollide = false end
                                end
                            end
                        end)
                        task.wait(0.5)
                    end
                end)
            end)
            
        elseif secName == "Setting Farm" then
            AddSectionTitle("Main Farming")
            AddToggle("Auto Farm", AutoFarmEnabled, function(v)
                AutoFarmEnabled = v
                if v then AutoFarm() end
            end)
            AddToggle("Fast Mode", FastHitEnabled, function(v)
                FastHitEnabled = v
                SilentKillDamage = v and 50 or 20
                HitboxSize = v and 50 or 35
            end)
            AddToggle("Auto Skills", AutoSkillsEnabled, function(v)
                AutoSkillsEnabled = v
                if v then AutoSkills() end
            end)
            AddToggle("Auto Boss", AutoBossEnabled, function(v)
                AutoBossEnabled = v
                if v and not AutoFarmEnabled then
                    AutoFarmEnabled = true
                    AutoFarm()
                end
            end)
            
        elseif secName == "Farming" then
            AddSectionTitle("Berry Farm")
            AddToggle("Auto Farm Berry", AutoFarmBerryEnabled, function(v)
                AutoFarmBerryEnabled = v
                if v then FarmBerry() end
            end)
            AddButton("Berry Count: " .. BerryCount, function() end)
            
        elseif secName == "Race Upgrade" then
            AddSectionTitle("👤 RACE UPGRADE V2 & V3")
            DetectSea()
            CheckCurrentRace()
            AddButton("Tộc: " .. PlayerRace, function()
                CheckCurrentRace()
                LoadContent("Race Upgrade")
            end)
            AddButton("Version: " .. PlayerRaceVersion, function()
                CheckCurrentRace()
                LoadContent("Race Upgrade")
            end)
            
            AddSectionTitle("V2 - Alchemist")
            AddToggle("Auto Race V2", AutoRaceV2Enabled, function(v)
                AutoRaceV2Enabled = v
                if v then RaceV2Upgrade() end
            end)
            AddButton("Blue: " .. (BlueFlowerCollected and "✅" or "❌"), function() end)
            AddButton("Red: " .. (RedFlowerCollected and "✅" or "❌"), function() end)
            AddButton("Yellow: " .. (YellowFlowerCollected and "✅" or "❌"), function() end)
            AddButton("V2: " .. (RaceV2Completed and "✅ Done" or "⏳ ..."), function() end)
            
            AddSectionTitle("V3 - Arowe")
            AddToggle("Auto Race V3", AutoRaceV3Enabled, function(v)
                AutoRaceV3Enabled = v
                if v then RaceV3Upgrade() end
            end)
            AddButton("V3: " .. (RaceV3Completed and "✅ Done" or "⏳ ..."), function() end)
            
        elseif secName == "Stack Farming" then
            AddSectionTitle("Stack Farming")
            AddToggle("Stack Farm Players", AutoStackFarm, function(v)
                AutoStackFarm = v
                if v then StackFarm() end
            end)
            
        elseif secName == "Farming Other" then
            AddSectionTitle("Islands & Bosses")
            AddToggle("Mystery Island", AutoMysteryIsland, function(v)
                AutoMysteryIsland = v
                if v then AutoIsland("Mystery") end
            end)
            AddToggle("Kitsune Island", AutoKitsuneIsland, function(v)
                AutoKitsuneIsland = v
                if v then AutoIsland("Kitsune") end
            end)
            AddToggle("Prehistoric Island", AutoPrehistoricIsland, function(v)
                AutoPrehistoricIsland = v
                if v then AutoIsland("Prehistoric") end
            end)
            AddToggle("Auto Leviathan", AutoLeviathan, function(v)
                AutoLeviathan = v
                if v then LeviathanFarm() end
            end)
            
        elseif secName == "Fruit and Raid" then
            AddSectionTitle("Raid & Dungeon")
            AddToggle("Auto Raid", AutoRaidEnabled, function(v)
                AutoRaidEnabled = v
                if v then AutoRaid() end
            end)
            AddToggle("Auto Dungeon", AutoDungeonEnabled, function(v)
                AutoDungeonEnabled = v
                if v then AutoDungeon() end
            end)
            
        elseif secName == "Haki 7 Mau (Rainbow)" then
            AddSectionTitle("🌈 HAKI 7 MAU SYSTEM")
            AddToggle("Auto Haki Quest", AutoHakiQuest, function(v)
                AutoHakiQuest = v
                if v then HakiQuestManager() end
            end)
            AddButton("Quest: " .. (HakiQuestCompleted and "✅ Done" or (HakiQuestStarted and "⏳ ..." or "❌")), function() end)
            AddSectionTitle("5 Boss Progress")
            for i = 1, 5 do
                AddButton(i .. ". " .. HakiBosses[i].name .. ": " .. (HakiBosses[i].killed and "✅" or "❌"), function() end)
            end
            AddButton("Rainbow: " .. (RainbowHakiUnlocked and "✅ UNLOCKED!" or "🔒"), function() end)
            
        elseif secName == "Race V4 (Blue Gear)" then
            AddSectionTitle("🌙 MOON GAZE + BLUE GEAR")
            AddToggle("Auto Nhìn Trăng", AutoMoonGaze, function(v)
                AutoMoonGaze = v
                if v then MoonGaze() end
            end)
            AddToggle("Auto Nhặt Blue Gear", AutoBlueGear, function(v)
                AutoBlueGear = v
                if v then BlueGearCollector() end
            end)
            AddButton("Blue Gear: " .. (BlueGearCollected and "✅ Đã nhặt" or "❌ Chưa có"), function() end)
            AddButton("Thời gian: " .. (IsNightTime() and "🌙 Đêm" or "☀️ Ngày"), function() end)
            AddSectionTitle("Hướng dẫn")
            AddButton("1. Đợi ban đêm", function() end)
            AddButton("2. Tìm Mirage Island", function() end)
            AddButton("3. Lên điểm cao nhất", function() end)
            AddButton("4. Bật V3 + Nhìn trăng 15s", function() end)
            AddButton("5. Nhặt Blue Gear", function() end)
            
        elseif secName == "Sea Event" then
            AddSectionTitle("Sea Events")
            AddButton("Sea Beast", function()
                local c = LP.Character
                if c and c:FindFirstChild("HumanoidRootPart") then
                    local root = c:FindFirstChild("HumanoidRootPart")
                    for _, obj in ipairs(Workspace:GetDescendants()) do
                        if obj:IsA("Model") and obj.Name:lower():find("sea") and obj.Name:lower():find("beast") and obj:FindFirstChildOfClass("Humanoid") then
                            local lr = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head")
                            if lr then
                                root.CFrame = lr.CFrame * CFrame.new(0, 40, 0)
                                obj:FindFirstChildOfClass("Humanoid").Health = 0
                            end
                            break
                        end
                    end
                end
            end)
        end
        
        Content.CanvasSize = UDim2.new(0, 0, 0, y + 20)
    end
    
    LoadContent(selected)
    
    -- Drag
    local dragging, dragStart, startPos = false, nil, nil
    Header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true; dragStart = input.Position; startPos = Main.Position
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    
    -- Toggle UI
    UIS.InputBegan:Connect(function(input, gpe)
        if gpe then return end
        if input.KeyCode == Enum.KeyCode.RightShift then
            isUIOpen = not isUIOpen
            Main.Visible = isUIOpen
        end
    end)
    
    CloseBtn.MouseButton1Click:Connect(function()
        Main.Visible = not Main.Visible
    end)
end

-- Anti AFK
LP.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- Initialize
DetectSea()
CheckCurrentRace()
CreateUI()
SilentKillAlways()

print("========================================")
print(" 🍌🐱 Banana Cat Hub - Blox Fruit v9.0")
print("========================================")
print(" 🔪 Silent Kill: AUTO ON (Luôn bật)")
print(" ⚔️ Auto Farm | 🎯 Auto Skills | 😈 Boss")
print(" 👤 Race V2: Alchemist + 3 Hoa")
print(" 👤 Race V3: Arowe + Quest riêng tộc")
print(" 🌈 Haki 7 Màu: Full Auto Quest")
print(" 🌙 Moon Gaze + 🔵 Blue Gear (V4 Prep)")
print(" 🍓 Berry | 🏃 Stack | 🏝️ Islands | 👾 Raid")
print("========================================")
print(" 🎮 RightShift: Mo/Tat UI")
print(" 📂 Tabs: Race Upgrade | Haki 7 Mau | Race V4")
print("========================================")
