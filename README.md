local url = (https://raw.https://github.com/toporik242/hap/new/main.lua)"

local repo = "https://raw.githubusercontent.com/toporik242/hap/new/main/"
local Library = loadstring(game:HttpGet(repo .. "Library.lua"))()
local ThemeManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/toporik242/pasta.land/refs/heads/main/ThemeManager"))()
local SaveManager = loadstring(game:HttpGet(repo .. "addons/SaveManager.lua"))()

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService        = game:GetService("RunService")
local Players           = game:GetService("Players")
local UIS               = game:GetService("UserInputService")
local Debris            = game:GetService("Debris")
local Lighting          = game:GetService("Lighting")
local Player            = Players.LocalPlayer
local Options           = Library.Options
local cam               = workspace.CurrentCamera

local Window = Library:CreateWindow({
    Title            = "Goose",
    Footer           = "TridentSurvivalV5",
    NotifySide       = "Right",
    ShowCustomCursor = true
})

local Tabs = {
    COMBAT          = Window:AddTab("COMBAT", "swords"),
    VISUAL          = Window:AddTab("VISUALS", "scan-eye"),
    ["UI Settings"] = Window:AddTab("UI SETTINGS", "settings")
}

-- -----------------------------------------------

local Aimbot = {
    UI = Tabs.COMBAT:AddRightGroupbox("AIMBOT", "crosshair"),
    Settings = {
        enabled      = false,
        prediction   = false,
        fovEnabled   = false,
        sleeperCheck = false,
        aiCheck      = false,
        teamCheck    = false,
        fovSize      = 150,
        fovColor     = Color3.fromRGB(0, 120, 255),
        smoothness   = 2,
        aimRadius    = 1000,
        predUpBase   = 0.5,
        locked       = nil,
        lockedPart   = nil,
        predX        = 0,
        predY        = 0,
        weapon       = {side = 0, up = 0},
        hasWeapon    = false,
        players      = {},
    },
    WeaponSets = {
        Bow              = {side = 1.1,  up = 3.0  },
        CrossBow         = {side = 0.55, up = 1.55 },
        AR15             = {side = 0.10, up = 0.025},
        M4A1             = {side = 0.20, up = 0    },
        SCAR             = {side = 0.25, up = 0.40 },
        SVD              = {side = 0.30, up = 0.1  },
        C9               = {side = 0.35, up = 1.0  },
        UZI              = {side = 0.35, up = 0    },
        USP9             = {side = 0.35, up = 1.0  },
        Blunderbuss      = {side = 1.0,  up = 0    },
        PumpShotgun      = {side = 1.0,  up = 0    },
        EnergyRifle      = {side = 0.5,  up = 0.05 },
        GaussRifle       = {side = 0.25, up = 0.1  },
        HMAR             = {side = 0.35, up = 2.0  },
        LeverActionRifle = {side = 0.35, up = 0.1  },
        PipePistol       = {side = 1.0,  up = 1.0  },
        PipeSMG          = {side = 5.0,  up = 0.50 },
        Magnum           = {side = 0.65, up = 1.0  },
    },
    PartSets = {
        Bow              = {"Arrow","Fabric","Meshes/Bow"},
        CrossBow         = {"Arrow","BackMetal","Body","FrontNails","Release","SpringSteel","String","Wheel","Slide"},
        AR15             = {"AnimSaves","Barrel","Body","Bolt","ChargingHandle","Decor","Grip","Mag","Rails","Stock","Muzzle"},
        M4A1             = {"DefaultSight","Body","Bolt","ChargeHandle","Grip","Mag","Metal","mbrk","Muzzle"},
        SCAR             = {"DefaultSight","Barrel","Body","ChargingHandle","Decals","Mag","Rails","ShoulderPad","Stock"},
        SVD              = {"DefaultSight","Body","Bolt","Magazine","Magazine2","Metal2","Wood"},
        C9               = {"Barrel","Body","Bolt","Decor","Grip","LowerSlide","Mag","Sight1","Sight2","UpperSlide","Muzzle"},
        UZI              = {"DefaultSight","Body","Body2","Bolt","ChargingHandle","Decor","Grip","Mag","Stock","Muzzle"},
        USP9             = {"Body","Mag","Slide","Muzzle"},
        Blunderbuss      = {"Body","Tube","thing","Muzzle"},
        PumpShotgun      = {"Barrel","Body","MainMetal","RearSight","Shell","Slider","Muzzle"},
        EnergyRifle      = {"DefaultSight","FrontCover","Glowing","Grip","Mag","Metal","Metal2","RearCover","RearDecor","Screws","Tubes"},
        GaussRifle       = {"DefaultSight","Barrel","Body","CoilHolders","Coils","Decals1","Decals2","Grip","Housing","Mag","StockBack"},
        HMAR             = {"DefaultSight","Body","Bolt","Bolts","Cover","Mag","Rails","Spring","Stock","Wood","Muzzle"},
        LeverActionRifle = {"9mm","DefaultSight","Body","Brass","Hammer","Lever","Metal","Thing","Wood","Muzzle"},
        PipePistol       = {"DefaultSight","Body","Bolt","Mag","Muzzle"},
        PipeSMG          = {"DefaultSight","Barrel","Body","Bolt","Flap","Grip","Mag","Stock","Muzzle"},
          Magnum           = {"Cylinder","Decor","EjectRod","EjectRodDecal","Frame","Grip"},
    },
    Cache = {
        sleep     = setmetatable({}, {__mode = "k"}),
        bot       = setmetatable({}, {__mode = "k"}),
        lastPos   = setmetatable({}, {__mode = "k"}),
        smoothVel = setmetatable({}, {__mode = "k"}),
    },
    FOV = Drawing.new("Circle"),
}

Aimbot.FOV.Thickness    = 1
Aimbot.FOV.NumSides     = 64
Aimbot.FOV.Radius       = Aimbot.Settings.fovSize
Aimbot.FOV.Filled       = false
Aimbot.FOV.Visible      = false
Aimbot.FOV.Transparency = 1
Aimbot.FOV.Color        = Aimbot.Settings.fovColor
Aimbot.FOV.ZIndex       = 999

local function updateFovPos()
    local s = cam.ViewportSize
    Aimbot.FOV.Position = Vector2.new(s.X / 2, s.Y / 2)
end
updateFovPos()
cam:GetPropertyChangedSignal("ViewportSize"):Connect(updateFovPos)

local function AB_IsTeam(model)
    if not model then return false end
    local head = model:FindFirstChild("Head")
    if not head then return false end
    local dot = head:FindFirstChild("Dot")
    return dot and dot.Enabled == true or false
end

local function AB_IsSleeper(model)
    if not model then return false end
    local cached = Aimbot.Cache.sleep[model]
    if cached and tick() - cached.time < 1 then return cached.value end
    local lt = model:FindFirstChild("LowerTorso")
    local value = false
    if lt then
        local rr = lt:FindFirstChild("RootRig")
        if rr then
            local ok, angle = pcall(function() return rr.CurrentAngle end)
            value = ok and type(angle) == "number" and angle ~= 0 or false
        end
    end
    Aimbot.Cache.sleep[model] = {value = value, time = tick()}
    return value
end

local function AB_IsNPC(model)
    if not model then return false end
    local cached = Aimbot.Cache.bot[model]
    if cached and tick() - cached.time < 2 then return cached.value end
    local value = false
    local torso = model:FindFirstChild("Torso")
    if torso then
        value = not torso:FindFirstChild("LeftBooster") and true or false
    else
        local hrp = model:FindFirstChild("HumanoidRootPart")
        value = hrp and hrp.CollisionGroup == "NPC" or false
    end
    Aimbot.Cache.bot[model] = {value = value, time = tick()}
    return value
end

local function AB_GetPath(root, p)
    local cur = root
    for n in p:gmatch("[^/]+") do
        cur = cur:FindFirstChild(n)
        if not cur then return nil end
    end
    return cur
end

local function AB_GetWeapon()
    local r = workspace
    for _, n in ipairs({"Const", "Ignore", "FPSArms"}) do
        r = r:FindFirstChild(n)
        if not r then Aimbot.Settings.hasWeapon = false; return nil end
    end
    local hand = r:FindFirstChild("HandModel")
    if not hand then Aimbot.Settings.hasWeapon = false; return nil end
    for w, parts in next, Aimbot.PartSets do
        local found = 0
        for _, p in ipairs(parts) do
            local part = p:find("/") and AB_GetPath(hand, p) or hand:FindFirstChild(p, true)
            if part then found = found + 1 end
        end
        if found >= #parts * 0.6 then
            Aimbot.Settings.hasWeapon = true
            local ws = Aimbot.WeaponSets[w]
            if ws then
                Aimbot.Settings.weapon.side = ws.side
                Aimbot.Settings.weapon.up   = ws.up
            end
            return w
        end
    end
    Aimbot.Settings.hasWeapon   = false
    Aimbot.Settings.weapon.side = 0
    Aimbot.Settings.weapon.up   = 0
    return nil
end

task.spawn(function()
    while true do task.wait(0.2); AB_GetWeapon() end
end)

local function AB_GetPart(m)
    return m and (
        m:FindFirstChild("Head") or
        m:FindFirstChild("UpperTorso") or
        m:FindFirstChild("LowerTorso") or
        m:FindFirstChild("HumanoidRootPart")
    )
end

local function AB_GetVel(m, dt)
    if not m then return Vector3.new() end
    local p = AB_GetPart(m)
    if not p then return Vector3.new() end
    local t    = tick()
    local last = Aimbot.Cache.lastPos[m]
    if not last then
        Aimbot.Cache.lastPos[m]   = {pos = p.Position, time = t}
        Aimbot.Cache.smoothVel[m] = Vector3.new()
        return Vector3.new()
    end
    local elapsed = t - last.time
    if elapsed <= 0 then return Aimbot.Cache.smoothVel[m] or Vector3.new() end
    local rawVel  = (p.Position - last.pos) / elapsed
    Aimbot.Cache.lastPos[m] = {pos = p.Position, time = t}
    local prev   = Aimbot.Cache.smoothVel[m] or Vector3.new()
    local lerpXZ = math.clamp(dt * 8, 0, 1)
    local lerpY  = math.clamp(dt * 2, 0, 1)
    local smooth = Vector3.new(
        prev.X + (rawVel.X - prev.X) * lerpXZ,
        prev.Y + (rawVel.Y - prev.Y) * lerpY,
        prev.Z + (rawVel.Z - prev.Z) * lerpXZ
    )
    Aimbot.Cache.smoothVel[m] = smooth
    return smooth
end

local function AB_AddPlayer(o)
    if o:IsA("Model") and o.Name ~= Player.Name then
        if o:FindFirstChild("HumanoidRootPart") or o:FindFirstChild("LowerTorso") then
            table.insert(Aimbot.Settings.players, o)
        end
    end
end

for _, v in next, workspace:GetChildren() do AB_AddPlayer(v) end
workspace.ChildAdded:Connect(AB_AddPlayer)
workspace.ChildRemoved:Connect(function(o)
    for i = #Aimbot.Settings.players, 1, -1 do
        if Aimbot.Settings.players[i] == o then
            table.remove(Aimbot.Settings.players, i)
            Aimbot.Cache.lastPos[o]   = nil
            Aimbot.Cache.sleep[o]     = nil
            Aimbot.Cache.bot[o]       = nil
            Aimbot.Cache.smoothVel[o] = nil
            break
        end
    end
end)

RunService.RenderStepped:Connect(function(dt)
    local S = Aimbot.Settings
    if not S.enabled then
        S.locked = nil; S.lockedPart = nil
        S.predX = 0; S.predY = 0
        Aimbot.FOV.Visible = false
        return
    end
    Aimbot.FOV.Visible = S.fovEnabled
    local camPos = cam.CFrame.Position
    local vp     = cam.ViewportSize
    local center = Vector2.new(vp.X / 2, vp.Y / 2)

    if S.locked and S.locked.Parent then
        local part = AB_GetPart(S.locked)
        if part then
            local sp, on = cam:WorldToViewportPoint(part.Position)
            if on then
                local d2c = (Vector2.new(sp.X, sp.Y) - center).Magnitude
                if d2c > S.fovSize * 2 then S.locked = nil; S.lockedPart = nil end
            else
                S.locked = nil; S.lockedPart = nil
            end
        else
            S.locked = nil; S.lockedPart = nil
        end
    else
        S.locked = nil; S.lockedPart = nil
    end

    if not S.locked then
        local bestDist = math.huge
        local bestD2C  = math.huge
        for _, pl in ipairs(S.players) do
            if not pl or not pl.Parent then continue end
            if S.sleeperCheck and AB_IsSleeper(pl) then continue end
            if S.aiCheck      and AB_IsNPC(pl)     then continue end
            if S.teamCheck    and AB_IsTeam(pl)    then continue end
            local part = AB_GetPart(pl)
            if not part then continue end
            local dist = (camPos - part.Position).Magnitude
            if dist > S.aimRadius then continue end
            local sp, on = cam:WorldToViewportPoint(part.Position)
            if not on then continue end
            local d2c = (Vector2.new(sp.X, sp.Y) - center).Magnitude
            if d2c > S.fovSize then continue end
            if dist < bestDist or (dist == bestDist and d2c < bestD2C) then
                bestDist = dist; bestD2C = d2c
                S.locked = pl; S.lockedPart = part
            end
        end
    end

    if not S.locked then S.predX = 0; S.predY = 0; return end

    local tPart = AB_GetPart(S.locked)
    if not tPart then S.locked = nil; S.predX = 0; S.predY = 0; return end

    local tDist  = (camPos - tPart.Position).Magnitude
    local tVel   = AB_GetVel(S.locked, dt)
    local sp, on = cam:WorldToViewportPoint(tPart.Position)
    if not on then S.predX = 0; S.predY = 0; return end

    if not UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        S.predX = 0; S.predY = 0; return
    end

    local dx = sp.X - center.X
    local dy = sp.Y - center.Y
    local mx = dx / S.smoothness
    local my = dy / S.smoothness

    if S.prediction and S.hasWeapon and tPart then
        local vm    = tVel.Magnitude
        local upOff = 0
        if S.weapon.up > 0 then
            upOff = (tDist / 12) * S.weapon.up * S.predUpBase
        end
        local sox = 0
        if S.weapon.side > 0 and vm > 2 then
            local velDir = tVel / vm
            local sm = math.clamp((vm - 2) / 8, 0, 1)
            local mf = (tDist / 20) * S.weapon.side * 0.6 * sm
            local pp, ons = cam:WorldToViewportPoint(tPart.Position + velDir * mf)
            if ons then sox = math.clamp(pp.X - sp.X, -35, 35) end
        end
        local sf = math.min(dt * 12, 0.3)
        S.predX = S.predX + (sox    - S.predX) * sf
        S.predY = S.predY + (-upOff - S.predY) * sf
        mousemoverel(mx + S.predX, my + S.predY)
    else
        S.predX = 0; S.predY = 0
        mousemoverel(mx, my)
    end
end)

Aimbot.UI:AddToggle("Enable", {
    Text = "Enable Aimbot", Default = false,
    Callback = function(v) Aimbot.Settings.enabled = v; if not v then Aimbot.FOV.Visible = false end end
})
Aimbot.UI:AddToggle("Prediction", {
    Text = "Enable Prediction", Default = false,
    Callback = function(v) Aimbot.Settings.prediction = v end
})
Aimbot.UI:AddToggle("SleeperCheck", {
    Text = "Sleeper Check", Default = false,
    Callback = function(v) Aimbot.Settings.sleeperCheck = v end
})
Aimbot.UI:AddToggle("AICheck", {
    Text = "AI Check", Default = false,
    Callback = function(v) Aimbot.Settings.aiCheck = v end
})
Aimbot.UI:AddToggle("TeamCheck", {
    Text = "Team Check", Default = false,
    Callback = function(v) Aimbot.Settings.teamCheck = v end
})
local fovToggle = Aimbot.UI:AddToggle("FovEnable", {
    Text = "Show FOV Circle", Default = false,
    Callback = function(v)
        Aimbot.Settings.fovEnabled = v
        Aimbot.FOV.Visible = v and Aimbot.Settings.enabled
    end
})
fovToggle:AddColorPicker("FovColor", {
    Title = "FOV Color", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v) Aimbot.Settings.fovColor = v; Aimbot.FOV.Color = v end
})
Aimbot.UI:AddSlider("FovSize", {
    Text = "FOV Size", Default = 150, Min = 50, Max = 500, Rounding = 0, Suffix = "px",
    Callback = function(v) Aimbot.Settings.fovSize = v; Aimbot.FOV.Radius = v end
})
Aimbot.UI:AddSlider("Smoothness", {
    Text = "Smoothness", Default = 2, Min = 1, Max = 10, Rounding = 1,
    Callback = function(v) Aimbot.Settings.smoothness = v end
})
Aimbot.UI:AddSlider("AimRadius", {
    Text = "Aimbot Radius", Default = 1000, Min = 100, Max = 1000, Rounding = 0, Suffix = "st",
    Callback = function(v) Aimbot.Settings.aimRadius = v end
})

-- -----------------------------------------------

local Hitbox = {
    UI = Tabs.COMBAT:AddLeftGroupbox("HITBOX", "box"),
    Settings = {
        enabled = false, sizeX = 4, sizeY = 4, sizeZ = 4,
        transparency = 0.50, color = Color3.fromRGB(0, 120, 255),
        sleepCheck = false, aiCheck = false, teamCheck = false,
        target = "Head", mixSpeed = 1, canCollide = false,
    },
    Original = {}, MixToken = 0,
}

local function HB_IsTeam(model)
    if not model then return false end
    local h = model:FindFirstChild("Head")
    return h and h:FindFirstChild("Dot") and h.Dot.Enabled == true or false
end
local function HB_IsSleeper(model)
    if not model then return false end
    local lt = model:FindFirstChild("LowerTorso")
    if lt then
        local rr = lt:FindFirstChild("RootRig")
        if rr and typeof(rr.CurrentAngle) == "number" and rr.CurrentAngle ~= 0 then return true end
    end
    return false
end
local function HB_IsNPC(model)
    local torso = model:FindFirstChild("Torso")
    if torso then
        if torso.CollisionGroup == "NPC" then return true end
        if torso:FindFirstChild("LeftBooster") then return false end
        return true
    end
    local hrp = model:FindFirstChild("HumanoidRootPart")
    return hrp and hrp.CollisionGroup == "NPC" or false
end

Hitbox.ShouldSkip = function(model)
    if not model or not model.Parent then return true end
    local s = Hitbox.Settings
    if s.aiCheck    and HB_IsNPC(model)     then return true end
    if s.sleepCheck and HB_IsSleeper(model) then return true end
    if s.teamCheck  and HB_IsTeam(model)    then return true end
    return false
end

Hitbox.Save = function(part)
    if not Hitbox.Original[part] then
        Hitbox.Original[part] = {
            Size = part.Size, Transparency = part.Transparency,
            Color = part.Color, Material = part.Material, CanCollide = part.CanCollide
        }
    end
end
Hitbox.Restore = function(part)
    if not part or not part.Parent then return end
    local o = Hitbox.Original[part]
    if not o then return end
    part.Size = o.Size; part.Transparency = o.Transparency
    part.Color = o.Color; part.Material = o.Material; part.CanCollide = o.CanCollide
end
Hitbox.ApplyToPart = function(part)
    if not part or not part:IsA("BasePart") then return end
    Hitbox.Save(part)
    local s = Hitbox.Settings
    part.Size        = Vector3.new(s.sizeX, s.sizeY, s.sizeZ)
    part.Transparency = s.transparency
    part.Color       = s.color
    part.CanCollide  = not s.canCollide
end

Hitbox.RestoreAll = function()
    for part, o in pairs(Hitbox.Original) do
        if part and part.Parent then
            part.Size = o.Size; part.Transparency = o.Transparency
            part.Color = o.Color; part.Material = o.Material; part.CanCollide = o.CanCollide
        end
    end
    Hitbox.Original = {}
end

Hitbox.IsCandidate = function(m)
    return m:IsA("Model") and m.Name ~= Player.Name
        and (m:FindFirstChild("Head") or m:FindFirstChild("HumanoidRootPart"))
end

Hitbox.ApplyToModel = function(model)
    if not model or not model.Parent then return end
    local s = Hitbox.Settings
    local head  = model:FindFirstChild("Head")
    local torso = model:FindFirstChild("Torso") or model:FindFirstChild("HumanoidRootPart")

    if not head or not torso then return end

    if Hitbox.ShouldSkip(model) or not s.enabled then
        Hitbox.Restore(head)
        Hitbox.Restore(torso)
        return
    end

    if s.target == "Head" then
        Hitbox.ApplyToPart(head)
        Hitbox.Restore(torso)
    elseif s.target == "Torso" then
        Hitbox.ApplyToPart(torso)
        Hitbox.Restore(head)
    end

end

Hitbox.UpdateAll = function()
    local s = Hitbox.Settings
    if s.target == "Mix" then return end 
    for _, model in pairs(workspace:GetChildren()) do
        if Hitbox.IsCandidate(model) then
            Hitbox.ApplyToModel(model)
        end
    end
end

Hitbox.Refresh = function()
    local s = Hitbox.Settings
    if not s.enabled or s.target == "Mix" then return end
    for _, model in pairs(workspace:GetChildren()) do
        if Hitbox.IsCandidate(model) then
            local head  = model:FindFirstChild("Head")
            local torso = model:FindFirstChild("Torso") or model:FindFirstChild("HumanoidRootPart")
            -- Проверяем оба
            if head and torso then
                local part = s.target == "Head" and head or torso
                Hitbox.ApplyToPart(part)
            end
        end
    end
end

Hitbox.ApplyCanCollide = function()
    local s = Hitbox.Settings
    for part in pairs(Hitbox.Original) do
        if part and part.Parent then part.CanCollide = not s.canCollide end
    end
end

local function StartMixLoop()
    Hitbox.MixToken = Hitbox.MixToken + 1
    local myToken = Hitbox.MixToken
    task.spawn(function()
        local phase = 0
        local s = Hitbox.Settings
        while Hitbox.MixToken == myToken and s.enabled and s.target == "Mix" do
            for _, model in pairs(workspace:GetChildren()) do
                if Hitbox.IsCandidate(model) and model.Parent then
                    local head  = model:FindFirstChild("Head")
                    local torso = model:FindFirstChild("Torso") or model:FindFirstChild("HumanoidRootPart")

                    if head and torso then
                        if not Hitbox.ShouldSkip(model) then
                            if phase == 0 then
                                Hitbox.ApplyToPart(head)
                                Hitbox.Restore(torso)
                            else
                                Hitbox.ApplyToPart(torso)
                                Hitbox.Restore(head)
                            end
                        else
                            Hitbox.Restore(head)
                            Hitbox.Restore(torso)
                        end
                    end
                end
            end
            phase = 1 - phase
            task.wait(s.mixSpeed)
        end

        for _, model in pairs(workspace:GetChildren()) do
            if Hitbox.IsCandidate(model) then
                local head  = model:FindFirstChild("Head")
                local torso = model:FindFirstChild("Torso") or model:FindFirstChild("HumanoidRootPart")
                if head  then Hitbox.Restore(head)  end
                if torso then Hitbox.Restore(torso) end
            end
        end
    end)
end


task.spawn(function()
    while true do
        task.wait(0.5)
        if Hitbox.Settings.enabled and Hitbox.Settings.target ~= "Mix" then
            Hitbox.UpdateAll()
        end
    end
end)

workspace.ChildAdded:Connect(function(child)
    if not Hitbox.Settings.enabled or not Hitbox.IsCandidate(child) then return end
    task.wait(0.1)
    local s = Hitbox.Settings
    if s.target ~= "Mix" then
        Hitbox.ApplyToModel(child)
    end

end)

workspace.ChildRemoved:Connect(function(child)
    if child:IsA("Model") then
        local head  = child:FindFirstChild("Head")
        local torso = child:FindFirstChild("Torso") or child:FindFirstChild("HumanoidRootPart")
        if head  then Hitbox.Original[head]  = nil end
        if torso then Hitbox.Original[torso] = nil end
    end
end)

task.spawn(function()
    task.wait(1)
    local s = Hitbox.Settings
    if s.target == "Mix" then StartMixLoop() else Hitbox.UpdateAll() end
end)

-- UI
local enableToggle = Hitbox.UI:AddToggle("EnableHitbox", {
    Text = "Enable Hitbox", Default = false,
    Callback = function(state)
        Hitbox.Settings.enabled = state
        if not state then
            Hitbox.MixToken = Hitbox.MixToken + 1 
            Hitbox.RestoreAll()
        else
            local s = Hitbox.Settings
            if s.target == "Mix" then StartMixLoop() else Hitbox.UpdateAll() end
        end
    end
})
enableToggle:AddColorPicker("HitboxColor", {
    Default = Color3.fromRGB(0, 120, 255), Title = "Hitbox Color",
    Callback = function(color) Hitbox.Settings.color = color; Hitbox.Refresh() end
})
Hitbox.UI:AddDropdown("HitboxTarget", {
    Text = "Target", Values = {"Head", "Torso", "Mix"}, Default = "Head", Multi = false,
    Callback = function(value)
        local s = Hitbox.Settings
        local prev = s.target
        s.target = value
        if not s.enabled then return end
        if value == "Mix" then
            StartMixLoop()
        else

            if prev == "Mix" then Hitbox.MixToken = Hitbox.MixToken + 1 end
            task.wait(0.05) 
            Hitbox.UpdateAll()
        end
    end
})
Hitbox.UI:AddToggle("CanCollideToggle", {
    Text = "CanCollide", Default = false,
    Callback = function(state)
        Hitbox.Settings.canCollide = state
        if Hitbox.Settings.enabled then
            Hitbox.ApplyCanCollide()
        end
    end
})
Hitbox.UI:AddToggle("SleepCheck", {
    Text = "Sleep Check", Default = false,
    Callback = function(state)
        Hitbox.Settings.sleepCheck = state
        if Hitbox.Settings.enabled then Hitbox.UpdateAll() end
    end
})
Hitbox.UI:AddToggle("AICheck", {
    Text = "AI Check", Default = false,
    Callback = function(state)
        Hitbox.Settings.aiCheck = state
        if Hitbox.Settings.enabled then Hitbox.UpdateAll() end
    end
})
Hitbox.UI:AddToggle("TeamCheck", {
    Text = "Team Check", Default = false,
    Callback = function(state)
        Hitbox.Settings.teamCheck = state
        if Hitbox.Settings.enabled then Hitbox.UpdateAll() end
    end
})
Hitbox.UI:AddSlider("MixSpeed", {
    Text = "Mix Speed", Min = 0.5, Max = 3, Default = 1, Rounding = 1, Suffix = "s",
    Callback = function(value) Hitbox.Settings.mixSpeed = value end
})
Hitbox.UI:AddSlider("HitboxSizeX", {
    Text = "Size X", Min = 1, Max = 7, Default = 4, Rounding = 1, Suffix = "st",
    Callback = function(value) Hitbox.Settings.sizeX = value; Hitbox.Refresh() end
})
Hitbox.UI:AddSlider("HitboxSizeY", {
    Text = "Size Y", Min = 1, Max = 7, Default = 4, Rounding = 1, Suffix = "st",
    Callback = function(value) Hitbox.Settings.sizeY = value; Hitbox.Refresh() end
})
Hitbox.UI:AddSlider("HitboxSizeZ", {
    Text = "Size Z", Min = 1, Max = 7, Default = 4, Rounding = 1, Suffix = "st",
    Callback = function(value) Hitbox.Settings.sizeZ = value; Hitbox.Refresh() end
})
Hitbox.UI:AddSlider("HitboxTransparency", {
    Text = "Transparency", Min = 0, Max = 1, Default = 0.5, Rounding = 2, Suffix = "%",
    Callback = function(value) Hitbox.Settings.transparency = value; Hitbox.Refresh() end
})

-- -----------------------------------------------

local Character = { Jumpshoot = false, JsPart = nil }

Character.Js = function()
    local p = Instance.new("Part", workspace)
    p.Name = "!.!"; p.Size = Vector3.new(4, 0.2, 4)
    p.Anchored = true; p.Color = Color3.fromRGB(0, 120, 255)
    p.Transparency = 1; p.Material = Enum.Material.Neon
    local mesh = Instance.new("SpecialMesh")
    mesh.MeshType = Enum.MeshType.FileMesh
    mesh.MeshId   = "rbxassetid://20329976"
    mesh.Parent   = p
    return p
end

Character.Update = function()
    while Character.Jumpshoot do
        local ok, mid = pcall(function()
            return workspace.Const.Ignore.LocalCharacter.Middle.Position
        end)
        if ok and Character.JsPart and Character.JsPart.Parent then
            Character.JsPart.Position = mid - Vector3.new(0, 3.5, 0)
        end
        RunService.Heartbeat:Wait()
    end
end

local LN = { enabled = false, strength = 5 }

local function getNeck()
    local char = workspace.Const and workspace.Const.Ignore and workspace.Const.Ignore.LocalCharacter
    if not char then return nil end
    local top = char:FindFirstChild("Top")
    return top and top:FindFirstChild("Prism1")
end

local neck = getNeck()
print("rapid.hook")
local origNeckCF = neck and neck.CFrame

local function applyNeck(state)
    if not neck then neck = getNeck(); origNeckCF = neck and neck.CFrame end
    if not neck or not origNeckCF then return end
    neck.CFrame = state
        and CFrame.new(origNeckCF.Position - Vector3.new(0, LN.strength, 0)) * (origNeckCF - origNeckCF.Position)
        or origNeckCF
end

local OTH1 = Tabs.COMBAT:AddRightGroupbox("OTHER", "user-cog")

local jsToggle = OTH1:AddToggle("js", {
    Text = "Jump Shoot", Default = false,
    Callback = function(state)
        Character.Jumpshoot = state
        if state then
            if not Character.JsPart or not Character.JsPart.Parent then
                Character.JsPart = Character.Js()
            end
            task.spawn(function() Character.Update() end)
        else
            if Character.JsPart then Character.JsPart:Destroy(); Character.JsPart = nil end
        end
    end
})
jsToggle:AddKeyPicker("JsKey", {
    Default = "None", Text = "JumpShoot", Mode = "Toggle",
    Callback = function(state) jsToggle:SetValue(state) end
})

local lnToggle = OTH1:AddToggle("LongNeckToggle", {
    Text = "LongNeck", Default = false,
    Callback = function(v) LN.enabled = v; applyNeck(v) end
})
lnToggle:AddKeyPicker("LongNeckKey", {
    Text = "LongNeck", Default = "None", Mode = "Toggle",
    Callback = function(v) LN.enabled = v; applyNeck(v); lnToggle:SetValue(v) end
})
OTH1:AddSlider("LongNeckStrength", {
    Text = "Neck Height", Default = 5, Min = 3, Max = 6.5, Rounding = 1, Suffix = "st",
    Callback = function(v) LN.strength = v; if LN.enabled then applyNeck(true) end end
})

-- -----------------------------------------------

local ArmorCheck = { labels = {}, bg = {}, outline = {}, enabled = false, players = {} }
local AC_FOV       = 100
local AC_PADX      = 14
local AC_PADY      = 10
local AC_LINEH     = 26
local AC_FSIZE     = 18
local AC_RADIUS    = 8
local AC_RADIUS_3D = 1000
local AC_POS       = Vector2.new(20, cam.ViewportSize.Y / 2 - 60)
local dragging     = false
local dragOffset   = Vector2.new(0, 0)
local lastTotalW   = 0
local lastTotalH   = 0

local function AC_ClearShapes(t)
    if not t then return end
    for _, s in ipairs(t) do pcall(function() s.Visible = false; s:Remove() end) end
end
local function AC_Clear()
    for _, l in ipairs(ArmorCheck.labels) do pcall(function() l.Visible = false; l:Remove() end) end
    ArmorCheck.labels = {}
    AC_ClearShapes(ArmorCheck.bg);      ArmorCheck.bg      = {}
    AC_ClearShapes(ArmorCheck.outline); ArmorCheck.outline = {}
end
local function AC_MakeRounded(x, y, w, h, r, color, zindex)
    local shapes = {}
    local function sq(px, py, pw, ph)
        local s = Drawing.new("Square")
        s.Position = Vector2.new(px, py); s.Size = Vector2.new(pw, ph)
        s.Color = color; s.Filled = true; s.Thickness = 1
        s.Visible = true; s.ZIndex = zindex
        table.insert(shapes, s)
    end
    local function ci(px, py)
        local c = Drawing.new("Circle")
        c.Position = Vector2.new(px, py); c.Radius = r
        c.Color = color; c.Filled = true; c.Thickness = 1
        c.Visible = true; c.ZIndex = zindex
        table.insert(shapes, c)
    end
    sq(x, y+r, w, h-r*2); sq(x+r, y, w-r*2, r); sq(x+r, y+h-r, w-r*2, r)
    ci(x+r, y+r); ci(x+w-r, y+r); ci(x+r, y+h-r); ci(x+w-r, y+h-r)
    return shapes
end
local function AC_GetPart(m)
    return m and (m:FindFirstChild("Head") or m:FindFirstChild("UpperTorso") or m:FindFirstChild("LowerTorso") or m:FindFirstChild("HumanoidRootPart"))
end
local function AC_AddModel(o)
    if o:IsA("Model") and o ~= Player.Character then
        if o:FindFirstChild("HumanoidRootPart") or o:FindFirstChild("LowerTorso") or o:FindFirstChild("Head") then
            table.insert(ArmorCheck.players, o)
        end
    end
end
for _, v in ipairs(workspace:GetChildren()) do AC_AddModel(v) end
workspace.ChildAdded:Connect(function(o) task.wait(0.1); AC_AddModel(o) end)
workspace.ChildRemoved:Connect(function(o)
    for i = #ArmorCheck.players, 1, -1 do
        if ArmorCheck.players[i] == o then table.remove(ArmorCheck.players, i); break end
    end
end)

local acModel = nil
local acDrawn = nil
local acScan  = 0

local function AC_Draw(model)
    AC_Clear()
    if not model then return end
    local folder = model:FindFirstChild("Armor")
    local items  = folder and folder:GetChildren() or {}
    if #items == 0 then return end
    local x, y = AC_POS.X, AC_POS.Y
    local maxW  = 0
    for _, item in ipairs(items) do
        local w = #item.Name * (AC_FSIZE * 0.58) + AC_PADX * 2
        if w > maxW then maxW = w end
    end
    local totalH = #items * AC_LINEH + AC_PADY * 2
    lastTotalW = maxW; lastTotalH = totalH
    ArmorCheck.outline = AC_MakeRounded(x-2, y-2, maxW+4, totalH+4, AC_RADIUS+1, Color3.fromRGB(40,40,40), 8)
    ArmorCheck.bg      = AC_MakeRounded(x,   y,   maxW,   totalH,   AC_RADIUS,   Color3.fromRGB(10,10,10), 9)
    local yOff = y + AC_PADY
    for _, item in ipairs(items) do
        local l = Drawing.new("Text")
        l.Text = item.Name; l.Size = AC_FSIZE
        l.Color = Color3.fromRGB(255,255,255); l.Outline = true
        l.OutlineColor = Color3.fromRGB(0,0,0)
        l.Position = Vector2.new(x + AC_PADX, yOff)
        l.Visible = true; l.ZIndex = 10
        table.insert(ArmorCheck.labels, l)
        yOff = yOff + AC_LINEH
    end
end

UIS.InputBegan:Connect(function(input)
    if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
    local mp = Vector2.new(input.Position.X, input.Position.Y)
    if mp.X >= AC_POS.X-2 and mp.X <= AC_POS.X+lastTotalW+2 and
       mp.Y >= AC_POS.Y-2 and mp.Y <= AC_POS.Y+lastTotalH+2 then
        dragging = true; dragOffset = mp - AC_POS
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)
UIS.InputChanged:Connect(function(input)
    if not dragging or input.UserInputType ~= Enum.UserInputType.MouseMovement then return end
    AC_POS = Vector2.new(input.Position.X, input.Position.Y) - dragOffset
    if acModel then AC_Draw(acModel) end
end)

RunService.RenderStepped:Connect(function()
    if not ArmorCheck.enabled then return end
    local now    = tick()
    local vp     = cam.ViewportSize
    local center = Vector2.new(vp.X / 2, vp.Y / 2)
    local camPos = cam.CFrame.Position
    if acModel and acModel.Parent then
        local p = AC_GetPart(acModel)
        if p then
            local sp, on = cam:WorldToViewportPoint(p.Position)
            if on then
                local d2c  = (Vector2.new(sp.X, sp.Y) - center).Magnitude
                local dist = (camPos - p.Position).Magnitude
                if d2c < AC_FOV * 1.5 and dist <= AC_RADIUS_3D then
                    if acDrawn ~= acModel then AC_Draw(acModel); acDrawn = acModel end
                    return
                end
            end
        end
        acModel = nil; acDrawn = nil; AC_Clear()
    end
    if now - acScan < 0.1 then return end
    acScan = now
    local bestD = math.huge
    local bestM = nil
    for _, obj in ipairs(ArmorCheck.players) do
        if not obj or not obj.Parent then continue end
        local p = AC_GetPart(obj)
        if not p then continue end
        local dist = (camPos - p.Position).Magnitude
        if dist > AC_RADIUS_3D then continue end
        local sp, on = cam:WorldToViewportPoint(p.Position)
        if not on then continue end
        local d = (Vector2.new(sp.X, sp.Y) - center).Magnitude
        if d < AC_FOV and d < bestD then bestD = d; bestM = obj end
    end
    if bestM then
        if acModel ~= bestM then acModel = bestM; acDrawn = nil end
    elseif acModel then
        acModel = nil; acDrawn = nil; AC_Clear()
    end
end)
if type(messagebox) == "function" then
local rs = messagebox("this ai slop got cracked by 101 and u can skid ts shit without credits. GOOD LUCK MY LITTLE SKID <333","GOOSE X 101", 4 + 32)
if rs == 6 then
    print("10010100110101")
else
    Players.LocalPlayer:Kick("")
end
end
OTH1:AddToggle("ArmorCheckEnable", {
    Text = "Armor Check", Default = false,
    Callback = function(v)
        ArmorCheck.enabled = v
        if not v then acModel = nil; acDrawn = nil; AC_Clear() end
    end
})
OTH1:AddSlider("ArmorCheckRadius", {
    Text = "Armor Radius", Default = 1000, Min = 100, Max = 1000, Rounding = 0, Suffix = "st",
    Callback = function(v) AC_RADIUS_3D = v end
})

-- -----------------------------------------------

local Viewmodel = ReplicatedStorage:FindFirstChild("HandModels")
local Visuals = {
    Chams = {
        WeaponChams = false, WeaponChams_Color = Color3.fromRGB(0, 120, 255), WeaponChams_Material = "ForceField",
        ArmChams    = false, ArmChams_Color    = Color3.fromRGB(0, 120, 255), ArmChams_Material    = "ForceField",
        Section     = Tabs.VISUAL:AddRightGroupbox("OTHER", "package-open"),
    },
    Cache = { WeaponMaterial = {}, WeaponColor = {}, ArmMaterial = {}, ArmColor = {} },
}

if Viewmodel then
    for _, Child in pairs(Viewmodel:GetChildren()) do
        if Child:IsA("Model") then
            for _, Part in pairs(Child:GetDescendants()) do
                if Part:IsA("BasePart") then
                    Visuals.Cache.WeaponMaterial[Part] = Part.Material
                    Visuals.Cache.WeaponColor[Part]    = Part.Color
                end
            end
        end
    end
end

local armPartNames = {
    {"Const","Ignore","FPSArms","RightHand"},
    {"Const","Ignore","FPSArms","RightLowerArm"},
    {"Const","Ignore","FPSArms","LeftLowerArm"},
    {"Const","Ignore","FPSArms","LeftHand"},
    {"Const","Ignore","FPSArms","Fake","c_RightLowerArm"},
    {"Const","Ignore","FPSArms","Fake","c_LeftLowerArm"},
}

local function findPath(root, paths)
    local cur = root
    for _, p in ipairs(paths) do
        cur = cur:FindFirstChild(p)
        if not cur then return nil end
    end
    return cur
end

local function AC_RecacheArms()
    Visuals.Cache.ArmMaterial = {}; Visuals.Cache.ArmColor = {}
    for _, paths in ipairs(armPartNames) do
        local p = findPath(workspace, paths)
        if p and p:IsA("BasePart") then
            Visuals.Cache.ArmMaterial[p] = p.Material
            Visuals.Cache.ArmColor[p]    = p.Color
        end
    end
end

local function AC_GetArmParts()
    local parts = {}
    for _, paths in ipairs(armPartNames) do
        local p = findPath(workspace, paths)
        if p and p:IsA("BasePart") then table.insert(parts, p) end
    end
    return parts
end

AC_RecacheArms()

local fpsArms = findPath(workspace, {"Const","Ignore","FPSArms"})
if fpsArms then
    fpsArms.ChildAdded:Connect(function()
        task.wait(0.2); AC_RecacheArms()
        if Visuals.Chams.ArmChams then Visuals.Chams.Update() end
    end)
    fpsArms.ChildRemoved:Connect(function() task.wait(0.1); AC_RecacheArms() end)
end

Visuals.Chams.Update = function()
    if Viewmodel then
        for _, Child in pairs(Viewmodel:GetChildren()) do
            if Child:IsA("Model") then
                for _, Part in pairs(Child:GetDescendants()) do
                    if Part:IsA("BasePart") then
                        if Visuals.Chams.WeaponChams then
                            Part.Material = Enum.Material[Visuals.Chams.WeaponChams_Material]
                            Part.Color    = Visuals.Chams.WeaponChams_Color
                        else
                            local origMat = Visuals.Cache.WeaponMaterial[Part]
                            local origCol = Visuals.Cache.WeaponColor[Part]
                            if origMat then Part.Material = origMat end
                            if origCol then Part.Color    = origCol end
                        end
                    end
                end
            end
        end
    end
    local armParts = AC_GetArmParts()
    for _, part in ipairs(armParts) do
        if part and part.Parent then
            if Visuals.Chams.ArmChams then
                part.Material = Enum.Material[Visuals.Chams.ArmChams_Material]
                part.Color    = Visuals.Chams.ArmChams_Color
            else
                local origMat = Visuals.Cache.ArmMaterial[part]
                local origCol = Visuals.Cache.ArmColor[part]
                if origMat then part.Material = origMat end
                if origCol then part.Color    = origCol end
            end
        end
    end
end

RunService.Heartbeat:Connect(function()
    if not Visuals.Chams.ArmChams then return end
    local armParts = AC_GetArmParts()
    for _, part in ipairs(armParts) do
        if part and part.Parent then
            part.Material = Enum.Material[Visuals.Chams.ArmChams_Material]
            part.Color    = Visuals.Chams.ArmChams_Color
        end
    end
end)

local WeaponChams = Visuals.Chams.Section:AddToggle("WeaponChams", {
    Text = "Weapon Chams", Default = false,
    Callback = function(value) Visuals.Chams.WeaponChams = value; Visuals.Chams.Update() end
})
WeaponChams:AddColorPicker("WeaponChamsColor", {
    Default = Visuals.Chams.WeaponChams_Color, Title = "Weapon Color",
    Callback = function(value) Visuals.Chams.WeaponChams_Color = value; if Visuals.Chams.WeaponChams then Visuals.Chams.Update() end end
})
Visuals.Chams.Section:AddDropdown("WeaponMaterial", {
    Text = "Weapon Material", Default = "ForceField", Values = {"Neon","ForceField"},
    Callback = function(value) Visuals.Chams.WeaponChams_Material = value; if Visuals.Chams.WeaponChams then Visuals.Chams.Update() end end
})
local ArmChams = Visuals.Chams.Section:AddToggle("ArmChams", {
    Text = "Arm Chams", Default = false,
    Callback = function(value) Visuals.Chams.ArmChams = value; Visuals.Chams.Update() end
})
ArmChams:AddColorPicker("ArmChamsColor", {
    Default = Visuals.Chams.ArmChams_Color, Title = "Arm Color",
    Callback = function(value) Visuals.Chams.ArmChams_Color = value; if Visuals.Chams.ArmChams then Visuals.Chams.Update() end end
})
Visuals.Chams.Section:AddDropdown("ArmMaterial", {
    Text = "Arm Material", Default = "ForceField", Values = {"Neon","ForceField"},
    Callback = function(value) Visuals.Chams.ArmChams_Material = value; if Visuals.Chams.ArmChams then Visuals.Chams.Update() end end
})

Visuals.Chams.Section:AddDivider()

-- -----------------------------------------------

local BT = { enabled = false, color = Color3.fromRGB(0, 120, 255), thickness = 0.1, lifetime = 1.5 }
local bulletNames = {"Bullet","BlueBullet","Projectile","Rocket","Missile","Grenade"}

local function isLocalArrow(obj)
    local ok, res = pcall(function()
        local fps = workspace:FindFirstChild("Const")
            and workspace.Const:FindFirstChild("Ignore")
            and workspace.Const.Ignore:FindFirstChild("FPSArms")
        if fps then
            local hm = fps:FindFirstChild("HandModel")
            return hm and hm:FindFirstChild("Arrow") == obj
        end
        return false
    end)
    return ok and res
end

local function createTracer(proj)
    if proj.Name == "Arrow" and isLocalArrow(proj) then return end
    local lastPos = proj.Position or proj.CFrame.Position
    local conn
    conn = RunService.RenderStepped:Connect(function()
        if not BT.enabled or not proj or not proj.Parent then conn:Disconnect(); return end
        if proj.Name == "Arrow" and isLocalArrow(proj) then conn:Disconnect(); return end
        local currentPos = proj.Position or proj.CFrame.Position
        local distance   = (currentPos - lastPos).Magnitude
        if distance > 0.5 and distance < 500 then
            local tracer = Instance.new("Part")
            tracer.Anchored = true; tracer.CanCollide = false
            tracer.CanQuery = false; tracer.CanTouch = false
            tracer.Size     = Vector3.new(BT.thickness, BT.thickness, distance)
            tracer.CFrame   = CFrame.new(lastPos, currentPos) * CFrame.new(0, 0, -distance/2)
            tracer.Color    = BT.color
            tracer.Material     = Enum.Material.Neon
            tracer.Transparency = 0
            tracer.Parent       = workspace
            Debris:AddItem(tracer, BT.lifetime)
        end
        lastPos = currentPos
    end)
end

workspace.DescendantAdded:Connect(function(obj)
    if obj:IsDescendantOf(ReplicatedStorage) then return end
    task.wait()
    if not BT.enabled then return end
    for _, name in ipairs(bulletNames) do
        if obj.Name == name then createTracer(obj); return end
    end
    if obj.Name == "Arrow" and not isLocalArrow(obj) then createTracer(obj) end
end)

local tracerToggle = Visuals.Chams.Section:AddToggle("BulletTracerToggle", {
    Text = "Bullet Tracers", Default = false,
    Callback = function(v) BT.enabled = v end
})
tracerToggle:AddColorPicker("BulletTracerColor", {
    Title = "Tracer Color", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v) BT.color = v end
})
Visuals.Chams.Section:AddSlider("BulletTracerThickness", {
    Text = "Thickness", Default = 0.1, Min = 0.05, Max = 0.5, Rounding = 2, Suffix = "st",
    Callback = function(v) BT.thickness = v end
})
Visuals.Chams.Section:AddSlider("BulletTracerLifetime", {
    Text = "Lifetime", Default = 2, Min = 0.5, Max = 5, Rounding = 1, Suffix = "s",
    Callback = function(v) BT.lifetime = v end
})

Visuals.Chams.Section:AddDivider()

-- -----------------------------------------------

local ThirdPerson = { enabled = false, back = 15, up = 3 }
local hrpCam = workspace.Const and workspace.Const.Ignore
    and workspace.Const.Ignore.FPSArms
    and workspace.Const.Ignore.FPSArms:FindFirstChild("HumanoidRootPart")

local camToggle = Visuals.Chams.Section:AddToggle("CamToggle", {
    Text = "3rd Person", Default = false,
    Callback = function(v) ThirdPerson.enabled = v end
})
camToggle:AddKeyPicker("CamKey", {
    Default = "None", SyncToggleState = true, Mode = "Toggle", Text = "3rd Person Key",
    Callback = function(v) ThirdPerson.enabled = v end
})
Visuals.Chams.Section:AddSlider("CamUp", {
    Text = "Camera Up", Default = 3, Min = 1, Max = 15, Rounding = 1,
    Callback = function(v) ThirdPerson.up = v end
})
Visuals.Chams.Section:AddSlider("CamBack", {
    Text = "Camera Back", Default = 15, Min = 2.5, Max = 30, Rounding = 1,
    Callback = function(v) ThirdPerson.back = v end
})

RunService.RenderStepped:Connect(function()
    if ThirdPerson.enabled and hrpCam then
        local p = hrpCam.Position - hrpCam.CFrame.LookVector * ThirdPerson.back + Vector3.new(0, ThirdPerson.up, 0)
        cam.CFrame = CFrame.new(p, p + hrpCam.CFrame.LookVector)
    end
end)

Visuals.Chams.Section:AddDivider()

local Freecam = { enabled = false, speed = 150, pos = cam.CFrame.Position }

local freecamToggle = Visuals.Chams.Section:AddToggle("FreecamToggle", {
    Text = "Freecam", Default = false,
    Callback = function(v) Freecam.enabled = v; if v then Freecam.pos = cam.CFrame.Position end end
})
freecamToggle:AddKeyPicker("FreecamKey", {
    Default = "None", SyncToggleState = true, Mode = "Toggle", Text = "Freecam Key",
    Callback = function(v) Freecam.enabled = v; if v then Freecam.pos = cam.CFrame.Position end end
})
Visuals.Chams.Section:AddSlider("FreecamSpeed", {
    Text = "Freecam Speed", Default = 150, Min = 10, Max = 250, Rounding = 0,
    Callback = function(v) Freecam.speed = v end
})
Visuals.Chams.Section:AddLabel("Move  ↑ ← ↓ →")

RunService.RenderStepped:Connect(function(dt)
    if not Freecam.enabled then return end
    local move = Vector3.zero
    if UIS:IsKeyDown(Enum.KeyCode.Up)    then move = move + cam.CFrame.LookVector  end
    if UIS:IsKeyDown(Enum.KeyCode.Down)  then move = move - cam.CFrame.LookVector  end
    if UIS:IsKeyDown(Enum.KeyCode.Left)  then move = move - cam.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.Right) then move = move + cam.CFrame.RightVector end
    if move.Magnitude > 0 then
        Freecam.pos = Freecam.pos + move.Unit * Freecam.speed * dt
    end
    cam.CFrame = CFrame.new(Freecam.pos, Freecam.pos + cam.CFrame.LookVector)
end)

-- -----------------------------------------------

local PLRESP = Tabs.VISUAL:AddLeftGroupbox("PLAYERS ESP", "eye")

local Esp = {
    settings = {
        enabled = false, boxEnabled = false, nameEnabled = false,
        distanceEnabled = false, weaponEnabled = false, chamsEnabled = false,
        boxType = "Corner", boxOutline = true, boxFill = false,
        fillTransparency = 0.75, renderDistance = 1000,
        boxColor         = Color3.fromRGB(0, 120, 255),
        outlineColor     = Color3.fromRGB(20, 20, 20),
        fillColor        = Color3.fromRGB(0, 120, 255),
        fillColor2       = Color3.fromRGB(20, 20, 20),
        nameColor        = Color3.fromRGB(0, 120, 255),
        nameOutline      = true,
        nameOutlineColor = Color3.fromRGB(20, 20, 20),
        distColor        = Color3.fromRGB(0, 120, 255),
        distOutline      = true,
        distOutlineColor = Color3.fromRGB(20, 20, 20),
        weapColor        = Color3.fromRGB(0, 120, 255),
        weapOutline      = true,
        weapOutlineColor = Color3.fromRGB(20, 20, 20),
        chamsColor       = Color3.fromRGB(0, 120, 255),
        sleepCheck = false, teamCheck = false, aiCheck = false,
    },
    cache = {
        boxes      = setmetatable({}, {__mode = "k"}),
        sleep      = setmetatable({}, {__mode = "k"}),
        player     = setmetatable({}, {__mode = "k"}),
        weapon     = setmetatable({}, {__mode = "k"}),
        weaponTime = setmetatable({}, {__mode = "k"}),
        chams      = setmetatable({}, {__mode = "k"}),
    },
    const = {
        V3_UP = Vector3.new(0, 2.8, 0),
        V3_DN = Vector3.new(0, 3.0, 0),
        ANCHORS = {
            LeftTop         = Vector2.new(0, 0),
            LeftSide        = Vector2.new(0, 0),
            RightTop        = Vector2.new(1, 0),
            RightSide       = Vector2.new(0, 0),
            BottomSide      = Vector2.new(0, 1),
            BottomDown      = Vector2.new(0, 1),
            BottomRightSide = Vector2.new(1, 1),
            BottomRightDown = Vector2.new(1, 1),
        },
    },
}

local gui = Instance.new("ScreenGui")
gui.Name           = "ESPHolder"
gui.ResetOnSpawn   = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.IgnoreGuiInset = true
gui.Parent         = Player:WaitForChild("PlayerGui")

Player.CharacterAdded:Connect(function()
    task.wait(0.5)
    if not gui or not gui.Parent then
        gui = Instance.new("ScreenGui")
        gui.Name           = "ESPHolder"
        gui.ResetOnSpawn   = false
        gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        gui.IgnoreGuiInset = true
        gui.Parent         = Player:WaitForChild("PlayerGui")
    end
    neck = getNeck()
    origNeckCF = neck and neck.CFrame
end)

local function ESP_IsTeam(m)
    if not m then return false end
    local h = m:FindFirstChild("Head")
    return h and h:FindFirstChild("Dot") and h.Dot.Enabled == true or false
end
local function ESP_IsSleeper(m)
    if not m then return false end
    local c = Esp.cache.sleep[m]
    if c and tick() - c.time < 1 then return c.value end
    local lt = m:FindFirstChild("LowerTorso"); local v = false
    if lt then
        local rr = lt:FindFirstChild("RootRig")
        if rr then
            local ok, a = pcall(function() return rr.CurrentAngle end)
            v = ok and type(a) == "number" and a ~= 0 or false
        end
    end
    Esp.cache.sleep[m] = {value = v, time = tick()}; return v
end
local function ESP_IsPlayer(m)
    local c = Esp.cache.player[m]
    if c and tick() - c.time < 2 then return c.value end
    local t = m:FindFirstChild("Torso")
    local v = t and t:FindFirstChild("LeftBooster") and true or false
    Esp.cache.player[m] = {value = v, time = tick()}; return v
end
local function ESP_IsAI(m)
    local t = m:FindFirstChild("Torso") or m:FindFirstChild("HumanoidRootPart")
    return t and t.CollisionGroup == "NPC" or false
end
local function shouldSkip(m)
    if not m or not m.Parent then return true end
    local s = Esp.settings
    if s.sleepCheck and ESP_IsSleeper(m) then return true end
    if s.teamCheck  and ESP_IsTeam(m)    then return true end
    if s.aiCheck    and ESP_IsAI(m)      then return true end
    return false
end

local weaponData = {
    Bow={"Arrow","Fabric","Handle","Meshes/Bow","ADS","Mover","AnimationController"},
    Ar15={"AnimSaves","Barrel","Body","Bolt","ChargingHandle","Decor","Grip","Handle","Mag","Rails","Stock","ADS","Muzzle","AnimationController"},
    Blunderbuss={"Body","Handle","Tube","thing","ADS","Muzzle","AnimationController"},
    C9={"Barrel","Body","Bolt","Decor","Grip","Handle","LowerSlide","Mag","Sight1","Sight2","UpperSlide","ADS","Muzzle","AnimationController"},
    CrossBow={"Arrow","BackMetal","Body","FrontNails","Handle","Release","SpringSteel","String","Wheel","ADS","Slide","AnimationController"},
    EnergyRifle={"DefaultSight","FrontCover","Glowing","Grip","Handle","Mag","Metal","Metal2","RearCover","RearDecor","Screws","Tubes","AnimationController"},
    GaussRifle={"DefaultSight","Barrel","Body","CoilHolders","Coils","Decals1","Decals2","Grip","Handle","Housing","Mag","StockBack","AnimationController"},
    Hmar={"DefaultSight","Body","Bolt","Bolts","Cover","Handle","Mag","Rails","Spring","Stock","Wood","Muzzle","AnimationController"},
    LeverActionRifle={"9mm","DefaultSight","Body","Brass","Hammer","Handle","Lever","Metal","Thing","Wood","Muzzle","AnimationController"},
    M4a1={"DefaultSight","Body","Bolt","ChargeHandle","Grip","Handle","Mag","Metal","mbrk","Muzzle","AnimationController"},
    PipePistol={"DefaultSight","Body","Bolt","Handle","Mag","Muzzle","AnimationController"},
    PipeSmg={"DefaultSight","Barrel","Body","Bolt","Flap","Grip","Handle","Mag","Stock","Muzzle","AnimationController"},
    PumpShotgun={"Barrel","Body","Handle","MainMetal","RearSight","Shell","Slider","ADS","Muzzle","AnimationController"},
    RPG={"RocketModel","Body","Body2","Caps","Fasteners","FireMech","Handle","Safety","Sight","Trigger","ADS","Muzzle","AnimationController"},
    Scar={"DefaultSight","Barrel","Body","ChargingHandle","Decals","Handle","Mag","Rails","ShoulderPad","Stock","Muzzle","AnimationController"},
    Svd={"DefaultSight","Body","Bolt","Handle","Magazine","Magazine2","Metal2","Wood","AnimationController"},
    Usp9={"Body","Handle","Mag","Slide","ADS","Muzzle","AnimationController"},
    Uzi={"DefaultSight","Body","Body2","Bolt","ChargingHandle","Decor","Grip","Handle","Mag","Stock","Muzzle","AnimationController"},
    Magnum={"Cylinder","Decor","EjectRod","EjectRodDecal","Frame","Grip"},
}

local function detectWeapon(m)
    local t  = tick()
    local lu = Esp.cache.weaponTime[m]
    if lu and t - lu < 2 then return Esp.cache.weapon[m] or "None" end
    local hand = m:FindFirstChild("HandModel")
    if not hand then
        Esp.cache.weapon[m] = "None"; Esp.cache.weaponTime[m] = t; return "None"
    end
    local best, bestN = "None", 0
    for wn, parts in next, weaponData do
        local cnt = 0
        for _, p in ipairs(parts) do if hand:FindFirstChild(p, true) then cnt = cnt + 1 end end
        if cnt > bestN then best = wn; bestN = cnt end
    end
    Esp.cache.weapon[m] = best; Esp.cache.weaponTime[m] = t
    return best
end

local function updateChams(m, enabled)
    if not Esp.settings.chamsEnabled or not enabled then
        local c = Esp.cache.chams[m]
        if c and c.Enabled then c.Enabled = false end
        return
    end
    local c = Esp.cache.chams[m]
    if not c or not c.Parent then
        if c then pcall(function() c:Destroy() end) end
        local hl = Instance.new("Highlight")
        hl.FillTransparency    = 0
        hl.OutlineTransparency = 1
        hl.DepthMode           = Enum.HighlightDepthMode.AlwaysOnTop
        hl.FillColor           = Esp.settings.chamsColor
        hl.Parent              = m
        Esp.cache.chams[m]     = hl
    else
        if c.FillColor ~= Esp.settings.chamsColor then
            c.FillColor = Esp.settings.chamsColor
        end
        if not c.Enabled then c.Enabled = true end
    end
end

local function newText()
    local t = Drawing.new("Text")
    t.Visible = false; t.Size = 13; t.Center = true
    t.Font = 2; t.Outline = true; t.OutlineColor = Color3.fromRGB(0,0,0)
    return t
end

local function newCornerFrame(anchor)
    local f = Instance.new("Frame")
    f.BackgroundColor3       = Esp.settings.boxColor
    f.BorderSizePixel        = 0
    f.BackgroundTransparency = 0
    f.Visible                = false
    f.AnchorPoint            = anchor
    f.ZIndex                 = 2
    f.Parent                 = gui
    local s = Instance.new("UIStroke")
    s.Color = Color3.fromRGB(0,0,0); s.Thickness = 1; s.Transparency = 0
    s.LineJoinMode = Enum.LineJoinMode.Miter
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = f
    return f, s
end

local function makeCorners()
    local cf = {}
    for name, anchor in next, Esp.const.ANCHORS do
        local f, s = newCornerFrame(anchor)
        cf[name] = {f = f, s = s}
    end
    return cf
end

local function makeFillFrame() setclipboard("https://discord.gg/nNZvNd4wjy")
    local f = Instance.new("Frame")
    f.BorderSizePixel = 0; f.BackgroundColor3 = Color3.fromRGB(255,255,255)
    f.BackgroundTransparency = 1; f.Visible = false; f.ZIndex = 0; f.Parent = gui
    local g = Instance.new("UIGradient")
    g.Rotation = 90
    g.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Esp.settings.fillColor),
        ColorSequenceKeypoint.new(1, Esp.settings.fillColor2),
    }
    g.Parent = f
    return f, g
end

local function makeDefaultBox()
    local fill, fillGrad = makeFillFrame()
    fill.ZIndex = 0
    local strokeOutline = Instance.new("UIStroke")
    strokeOutline.Color = Esp.settings.outlineColor; strokeOutline.Thickness = 3
    strokeOutline.Transparency = 0; strokeOutline.LineJoinMode = Enum.LineJoinMode.Miter
    strokeOutline.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; strokeOutline.Parent = fill
    local main = Instance.new("Frame")
    main.BorderSizePixel = 0; main.BackgroundColor3 = Color3.fromRGB(0,0,0)
    main.BackgroundTransparency = 1; main.Visible = false; main.ZIndex = 2; main.Parent = gui
    local stroke = Instance.new("UIStroke")
    stroke.Color = Esp.settings.boxColor; stroke.Thickness = 1; stroke.Transparency = 0
    stroke.LineJoinMode = Enum.LineJoinMode.Miter
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; stroke.Parent = main
    return {fill=fill, fillGrad=fillGrad, main=main, stroke=stroke, strokeOutline=strokeOutline}
end

local function applyFill(fill, grad, l, t, w, h)
    grad.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Esp.settings.fillColor),
        ColorSequenceKeypoint.new(1, Esp.settings.fillColor2),
    }
    fill.BackgroundTransparency = Esp.settings.boxFill and Esp.settings.fillTransparency or 1
    fill.Position = UDim2.new(0, l, 0, t); fill.Size = UDim2.new(0, w, 0, h)
    fill.Visible = true
end

local CORNER_POS = {}
local function buildCornerPos(l, r, t, b, cx, cy)
    CORNER_POS.LeftTop         = {l,   t,  cx,  1.5}
    CORNER_POS.LeftSide        = {l,   t,  1.5, cy}
    CORNER_POS.RightTop        = {r,   t,  cx,  1.5}
    CORNER_POS.RightSide       = {r-1, t,  1.5, cy}
    CORNER_POS.BottomSide      = {l,   b,  1.5, cy}
    CORNER_POS.BottomDown      = {l,   b,  cx,  1.5}
    CORNER_POS.BottomRightSide = {r,   b,  1.5, cy}
    CORNER_POS.BottomRightDown = {r,   b,  cx,  1.5}
end

local function updateCorners(cf, fill, grad, px, py, w, h)
    local cx, cy = w*0.22, h*0.22
    local l, r   = px - w*0.5, px + w*0.5
    local t, b   = py - h*0.5, py + h*0.5
    buildCornerPos(l, r, t, b, cx, cy)
    local bc  = Esp.settings.boxColor
    local oc  = Esp.settings.outlineColor
    local otr = Esp.settings.boxOutline and 0 or 1
    for name, seg in next, cf do
        local d = CORNER_POS[name]
        seg.f.Position = UDim2.new(0, d[1], 0, d[2]); seg.f.Size = UDim2.new(0, d[3], 0, d[4])
        seg.f.BackgroundColor3 = bc; seg.f.Visible = true
        seg.s.Color = oc; seg.s.Transparency = otr
    end
    applyFill(fill, grad, l, t, w, h)
end

local function updateDefaultBox(db, px, py, w, h)
    local l, t = px - w*0.5, py - h*0.5
    applyFill(db.fill, db.fillGrad, l, t, w, h)
    db.strokeOutline.Color = Esp.settings.outlineColor
    db.strokeOutline.Transparency = Esp.settings.boxOutline and 0 or 1
    db.main.Position = UDim2.new(0, l, 0, t); db.main.Size = UDim2.new(0, w, 0, h)
    db.main.Visible = true; db.stroke.Color = Esp.settings.boxColor; db.stroke.Transparency = 0
end

local function hideCorners(cf, fill)
    for _, seg in next, cf do
        if seg.f.Visible then seg.f.Visible = false end
    end
    if fill.Visible then fill.Visible = false end
end
local function hideDefault(db)
    if db.fill.Visible then db.fill.Visible = false end
    if db.main.Visible then db.main.Visible = false end
end

local function destroyEntry(m, d)
    pcall(function() d.cacheConn:Disconnect() end)
    for _, seg in next, d.corners do pcall(function() seg.f:Destroy() end) end
    pcall(function() d.cFill:Destroy() end)
    pcall(function() d.default.fill:Destroy() end)
    pcall(function() d.default.main:Destroy() end)
    pcall(function() d.nameText:Remove() end)
    pcall(function() d.distText:Remove() end)
    pcall(function() d.weapText:Remove() end)
    Esp.cache.boxes[m] = nil
end

local function createEntry(m)
    if Esp.cache.boxes[m] then return end
    local cFill, cGrad = makeFillFrame()
    local entry = {
        corners   = makeCorners(),
        cFill     = cFill,
        cGrad     = cGrad,
        default   = makeDefaultBox(),
        nameText  = newText(),
        distText  = newText(),
        weapText  = newText(),
        hrp       = m:FindFirstChild("HumanoidRootPart"),
        lastCache = 0,
        isPlayer  = false,
        isSleeper = false,
    }
    entry.cacheConn = m.ChildAdded:Connect(function()
        entry.hrp = m:FindFirstChild("HumanoidRootPart")
    end)
    Esp.cache.boxes[m] = entry
end

local ESP_INTERVAL = 1 / 90
local espLastTick  = 0

RunService.Heartbeat:Connect(function()
    local now = tick()
    if now - espLastTick < ESP_INTERVAL then return end
    espLastTick = now

    local camPos = cam.CFrame.Position
    local s      = Esp.settings

    for m, d in next, Esp.cache.boxes do
        local function hide()
            hideCorners(d.corners, d.cFill)
            hideDefault(d.default)
            if d.nameText.Visible then d.nameText.Visible = false end
            if d.distText.Visible then d.distText.Visible = false end
            if d.weapText.Visible then d.weapText.Visible = false end
        end

        if not m or not m.Parent then
            hide(); destroyEntry(m, d); continue
        end
        if not s.enabled then hide(); updateChams(m, false); continue end

        local hrp = d.hrp
        if not hrp or not hrp.Parent then
            d.hrp = m:FindFirstChild("HumanoidRootPart")
            hide(); continue
        end

        local hrpPos = hrp.Position
        local dx = hrpPos.X - camPos.X
        local dy = hrpPos.Y - camPos.Y
        local dz = hrpPos.Z - camPos.Z
        local distSq = dx*dx + dy*dy + dz*dz
        local rd = s.renderDistance
        if distSq > rd*rd then
            hide(); updateChams(m, false); continue
        end

        if shouldSkip(m) then hide(); updateChams(m, false); continue end

        if now - d.lastCache > 5 then
            d.isPlayer  = ESP_IsPlayer(m)
            d.isSleeper = ESP_IsSleeper(m)
            d.lastCache = now
        end

        local topPos, topVis = cam:WorldToViewportPoint(hrpPos + Esp.const.V3_UP)
        local botPos         = cam:WorldToViewportPoint(hrpPos - Esp.const.V3_DN)

        if not topVis then
            hide(); updateChams(m, true); continue
        end

        local h = botPos.Y - topPos.Y
        if h < 5 then hide(); continue end

        local w  = h * 0.65
        local px = topPos.X
        local py = topPos.Y + h * 0.5
        local dist = math.floor(math.sqrt(distSq))

        if s.boxEnabled then
            if s.boxType == "Corner" then
                hideDefault(d.default)
                updateCorners(d.corners, d.cFill, d.cGrad, px, py, w, h)
            else
                hideCorners(d.corners, d.cFill)
                updateDefaultBox(d.default, px, py, w, h)
            end
        else
            hideCorners(d.corners, d.cFill)
            hideDefault(d.default)
        end

        local dtype = d.isSleeper and "Sleeper" or (d.isPlayer and "Player" or "Bot")

        if s.nameEnabled then
            d.nameText.Text         = dtype
            d.nameText.Position     = Vector2.new(px, py - h*0.5 - 16)
            d.nameText.Color        = s.nameColor
            d.nameText.Outline      = s.nameOutline
            d.nameText.OutlineColor = s.nameOutlineColor
            d.nameText.Visible      = true
        else
            if d.nameText.Visible then d.nameText.Visible = false end
        end

        if s.distanceEnabled then
            d.distText.Text         = "[" .. dist .. "m]"
            d.distText.Position     = Vector2.new(px, py + h*0.5 + 4)
            d.distText.Color        = s.distColor
            d.distText.Outline      = s.distOutline
            d.distText.OutlineColor = s.distOutlineColor
            d.distText.Visible      = true
        else
            if d.distText.Visible then d.distText.Visible = false end
        end

        if s.weaponEnabled then
            d.weapText.Text         = detectWeapon(m)
            d.weapText.Position     = Vector2.new(px, py + h*0.5 + (s.distanceEnabled and 18 or 4))
            d.weapText.Color        = s.weapColor
            d.weapText.Outline      = s.weapOutline
            d.weapText.OutlineColor = s.weapOutlineColor
            d.weapText.Visible      = true
        else
            if d.weapText.Visible then d.weapText.Visible = false end
        end

        updateChams(m, true)
    end
end)

workspace.DescendantAdded:Connect(function(obj)
    if obj:IsA("Model") then
        task.wait(0.1)
        if obj:FindFirstChild("HumanoidRootPart") and not Esp.cache.boxes[obj] then
            createEntry(obj)
        end
    elseif obj.Name == "HandModel" and obj.Parent then
        Esp.cache.weapon[obj.Parent]     = nil
        Esp.cache.weaponTime[obj.Parent] = nil
    end
end)

workspace.DescendantRemoving:Connect(function(obj)
    if obj:IsA("Model") then
        local d = Esp.cache.boxes[obj]
        if d then destroyEntry(obj, d) end
        local ch = Esp.cache.chams[obj]
        if ch then
            pcall(function() ch:Destroy() end)
            Esp.cache.chams[obj] = nil
        end
        Esp.cache.weapon[obj]     = nil
        Esp.cache.weaponTime[obj] = nil
        Esp.cache.player[obj]     = nil
        Esp.cache.sleep[obj]      = nil
    elseif obj.Name == "HandModel" and obj.Parent then
        Esp.cache.weapon[obj.Parent]     = nil
        Esp.cache.weaponTime[obj.Parent] = nil
    end
end)

task.spawn(function()
    while true do
        task.wait(15)
        for m in next, Esp.cache.weapon do
            if not m or not m.Parent then
                Esp.cache.weapon[m] = nil
                Esp.cache.weaponTime[m] = nil
            end
        end
        for m in next, Esp.cache.sleep do
            if not m or not m.Parent then Esp.cache.sleep[m] = nil end
        end
        for m in next, Esp.cache.player do
            if not m or not m.Parent then Esp.cache.player[m] = nil end
        end
        for m, d in next, Esp.cache.boxes do
            if not m or not m.Parent then destroyEntry(m, d) end
        end
        for m, c in next, Esp.cache.chams do
            if not m or not m.Parent then
                pcall(function() c:Destroy() end)
                Esp.cache.chams[m] = nil
            end
        end
    end
end)

for _, m in next, workspace:GetChildren() do
    if m:IsA("Model") then
        task.spawn(function()
            task.wait(0.1)
            if m:FindFirstChild("HumanoidRootPart") then createEntry(m) end
        end)
    end
end

PLRESP:AddToggle("EnableEsp", {
    Text = "Enable ESP", Default = false,
    Callback = function(v) Esp.settings.enabled = v end
})
PLRESP:AddToggle("SleepCheck", {
    Text = "Sleep Check", Default = false,
    Callback = function(v) Esp.settings.sleepCheck = v end
})
PLRESP:AddToggle("AICheck", {
    Text = "AI Check", Default = false,
    Callback = function(v) Esp.settings.aiCheck = v end
})
PLRESP:AddToggle("TeamCheck", {
    Text = "Team Check", Default = false,
    Callback = function(v) Esp.settings.teamCheck = v end
})
PLRESP:AddSlider("RenderDistance", {
    Text = "Render Distance", Default = 1000, Min = 500, Max = 1500, Rounding = 0, Suffix = "st",
    Callback = function(v) Esp.settings.renderDistance = v end
})
local boxToggle = PLRESP:AddToggle("EnableBox", {
    Text = "Enable Box", Default = false,
    Callback = function(v) Esp.settings.boxEnabled = v end
})
boxToggle:AddColorPicker("BoxColor", {
    Title = "Box Color", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v) Esp.settings.boxColor = v end
})
PLRESP:AddDropdown("BoxType", {
    Values = {"Corner","Default"}, Default = 1, Multi = false, Text = "Box Type",
    Callback = function(v) Esp.settings.boxType = v end
})
local outlineToggle = PLRESP:AddToggle("BoxOutline", {
    Text = "Box Outline", Default = true,
    Callback = function(v) Esp.settings.boxOutline = v end
})
outlineToggle:AddColorPicker("OutlineColor", {
    Title = "Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.outlineColor = v end
})
local fillToggle = PLRESP:AddToggle("BoxFill", {
    Text = "Box Fill", Default = false,
    Callback = function(v) Esp.settings.boxFill = v end
})
fillToggle:AddColorPicker("FillColor1", {
    Title = "Fill Color 1", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v) Esp.settings.fillColor = v end
})
fillToggle:AddColorPicker("FillColor2", {
    Title = "Fill Color 2", Default = Color3.fromRGB(0, 30, 80),
    Callback = function(v) Esp.settings.fillColor2 = v end
})
PLRESP:AddSlider("FillTransparency", {
    Text = "Fill Transparency", Default = 0.75, Min = 0, Max = 1, Rounding = 2, Suffix = "%",
    Callback = function(v) Esp.settings.fillTransparency = v end
})
local nameToggle = PLRESP:AddToggle("EnableName", {
    Text = "Name", Default = false,
    Callback = function(v) Esp.settings.nameEnabled = v end
})
nameToggle:AddColorPicker("NameColor", {
    Title = "Name Color", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v) Esp.settings.nameColor = v end
})
local nameOutlineToggle = PLRESP:AddToggle("NameOutline", {
    Text = "Name Outline", Default = true,
    Callback = function(v) Esp.settings.nameOutline = v end
})
nameOutlineToggle:AddColorPicker("NameOutlineColor", {
    Title = "Name Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.nameOutlineColor = v end
})
local distToggle = PLRESP:AddToggle("EnableDist", {
    Text = "Distance", Default = false,
    Callback = function(v) Esp.settings.distanceEnabled = v end
})
distToggle:AddColorPicker("DistColor", {
    Title = "Distance Color", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v) Esp.settings.distColor = v end
})
local distOutlineToggle = PLRESP:AddToggle("DistOutline", {
    Text = "Distance Outline", Default = true,
    Callback = function(v) Esp.settings.distOutline = v end
})
distOutlineToggle:AddColorPicker("DistOutlineColor", {
    Title = "Distance Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.distOutlineColor = v end
})
local weapToggle = PLRESP:AddToggle("EnableWeapon", {
    Text = "Weapon", Default = false,
    Callback = function(v) Esp.settings.weaponEnabled = v end
})
weapToggle:AddColorPicker("WeapColor", {
    Title = "Weapon Color", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v) Esp.settings.weapColor = v end
})
local weapOutlineToggle = PLRESP:AddToggle("WeapOutline", {
    Text = "Weapon Outline", Default = true,
    Callback = function(v) Esp.settings.weapOutline = v end
})
weapOutlineToggle:AddColorPicker("WeapOutlineColor", {
    Title = "Weapon Outline Color", Default = Color3.fromRGB(20, 20, 20),
    Callback = function(v) Esp.settings.weapOutlineColor = v end
})
local chamsToggle = PLRESP:AddToggle("EnableChams", {
    Text = "Chams", Default = false,
    Callback = function(v)
        Esp.settings.chamsEnabled = v
        if not v then
            for _, c in next, Esp.cache.chams do
                if c.Enabled then c.Enabled = false end
            end
        end
    end
})
chamsToggle:AddColorPicker("ChamsColor", {
    Title = "Chams Color", Default = Color3.fromRGB(0, 120, 255),
    Callback = function(v)
        Esp.settings.chamsColor = v
        for _, c in next, Esp.cache.chams do
            if c.Enabled then c.FillColor = v end
        end
    end
})

-- -----------------------------------------------

local WRLD = Tabs.VISUAL:AddLeftGroupbox("WORLD", "earth")

local fakeGammaEnabled = false
local gammaValue = 0.5

local function applyFakeGamma()
    local effect = Lighting:FindFirstChild("StimEffect")
    if not effect then
        effect = Instance.new("ColorCorrectionEffect")
        effect.Name = "StimEffect"; effect.Parent = Lighting
    end
    effect.Enabled    = fakeGammaEnabled
    effect.Brightness = gammaValue
    effect.Contrast   = 1
    effect.Saturation = 1
    effect.TintColor  = Color3.fromRGB(255, 255, 255)
end

WRLD:AddToggle("FakeGammaToggle", {
    Text = "Fake Gamma", Default = false,
    Callback = function(s) fakeGammaEnabled = s; applyFakeGamma() end
}):AddKeyPicker("FakeGammaKey", {
    Default = "None", SyncToggleState = true, Mode = "Toggle", Text = "Fake Gamma Key",
    Callback = function(s) fakeGammaEnabled = s; applyFakeGamma() end
})
WRLD:AddSlider("FakeGammaSlider", {
    Text = "Gamma Value", Default = 0.5, Min = 0.2, Max = 0.99, Rounding = 2, Suffix = "%",
    Callback = function(val) gammaValue = val; if fakeGammaEnabled then applyFakeGamma() end end
})

local XR = { enabled = false, transparency = 0.5 }
local xrayMaterials = {
    [Enum.Material.Cobblestone]  = true,
    [Enum.Material.WoodPlanks]   = true,
    [Enum.Material.Metal]        = true,
    [Enum.Material.CorrodedMetal]= true,
    [Enum.Material.Concrete]     = true,
    [Enum.Material.Brick]        = true,
    [Enum.Material.DiamondPlate] = true,
}
local xrayOriginal = {}
local xrayParts    = {}
local xraySet      = {}

local function isXrayMat(p) return p:IsA("BasePart") and xrayMaterials[p.Material] end
local function addXrayPart(p)
    if xraySet[p] then return end
    xraySet[p] = true; table.insert(xrayParts, p); xrayOriginal[p] = p.Transparency
end
local function collectXrayParts()
    xrayParts = {}; xrayOriginal = {}; xraySet = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        if isXrayMat(v) then addXrayPart(v) end
    end
end
local function applyXray(state)
    for _, p in ipairs(xrayParts) do
        if p and p.Parent then
            p.Transparency = state and XR.transparency or (xrayOriginal[p] or 0)
        end
    end
end
collectXrayParts()

local xrayPendingAdd = {}
local xrayPendingDel = {}
local xrayBatchRunning = false

local function startXrayBatch()
    if xrayBatchRunning then return end
    xrayBatchRunning = true
    task.spawn(function()
        while #xrayPendingAdd > 0 or #xrayPendingDel > 0 do
            task.wait(0.1)
            for _, p in ipairs(xrayPendingDel) do
                if xraySet[p] then
                    xraySet[p] = nil; xrayOriginal[p] = nil
                    for i = #xrayParts, 1, -1 do
                        if xrayParts[i] == p then table.remove(xrayParts, i); break end
                    end
                end
            end
            xrayPendingDel = {}
            local batch = xrayPendingAdd; xrayPendingAdd = {}
            for _, p in ipairs(batch) do
                if p and p.Parent and not xraySet[p] then
                    addXrayPart(p)
                    if XR.enabled then p.Transparency = XR.transparency end
                end
            end
        end
        xrayBatchRunning = false
    end)
end

workspace.DescendantAdded:Connect(function(p)
    if isXrayMat(p) then table.insert(xrayPendingAdd, p); startXrayBatch() end
end)
workspace.DescendantRemoving:Connect(function(p)
    if isXrayMat(p) then table.insert(xrayPendingDel, p); startXrayBatch() end
end)

local xrayToggle = WRLD:AddToggle("XrayToggle", {
    Text = "XRAY", Default = false,
    Callback = function(v) XR.enabled = v; applyXray(v) end
})
xrayToggle:AddKeyPicker("XrayKey", {
    Default = "None", SyncToggleState = true, Mode = "Toggle", Text = "Xray",
    Callback = function(v) XR.enabled = v; applyXray(v) end
})
WRLD:AddSlider("XrayTransparency", {
    Text = "XRAY Transparency", Min = 0, Max = 1, Default = 0.5, Rounding = 2, Suffix = "%",
    Callback = function(v) XR.transparency = v; if XR.enabled then applyXray(true) end end
})

-- -----------------------------------------------

local VisualTabbox = Tabs.VISUAL:AddRightTabbox()
local VTab1 = VisualTabbox:AddTab("CONTAINER ESP")
local VTab2 = VisualTabbox:AddTab("ORE ESP")

local LootESP = {
    enabled     = false,
    maxDistance = 750,
    cache       = setmetatable({}, {__mode = "k"}),
    data        = setmetatable({}, {__mode = "k"}),

    Bucket   = { textColor = Color3.fromRGB(255, 165, 0),   textSize = 10, label = "Bucket",       textEnabled = false },
    Box      = { textColor = Color3.fromRGB(230, 182, 0),   textSize = 10, label = "DefaultBox",   textEnabled = false },
    Chest    = { textColor = Color3.fromRGB(150, 150, 150), textSize = 10, label = "GrayBox",      textEnabled = false },
    Crafting = { textColor = Color3.fromRGB(255, 0, 207),   textSize = 10, label = "HealtMachine", textEnabled = false },
    Crate    = { textColor = Color3.fromRGB(44, 97, 0),     textSize = 10, label = "GreenCrate",   textEnabled = false },
    Vault    = { textColor = Color3.fromRGB(100, 100, 100), textSize = 10, label = "Safe",         textEnabled = false },
    Gas      = { textColor = Color3.fromRGB(200, 0, 0),     textSize = 10, label = "Gasoline",     textEnabled = false },
}

local lootTypes = {"Bucket","Box","Chest","Crafting","Crate","Vault","Gas"}

VTab1:AddToggle("LootTextMaster", {
    Text = "Enable Container ESP", Default = false,
    Callback = function(v)
        LootESP.enabled = v
        for _, d in pairs(LootESP.data) do
            if d and d.t then
                d.t.Visible = LootESP[d.kind].textEnabled and v
            end
        end
    end
})
VTab1:AddSlider("LootMaxDist", {
    Text = "Max Distance", Default = 750, Min = 250, Max = 1250, Rounding = 0,
    Callback = function(v) LootESP.maxDistance = v end
})
VTab1:AddSlider("LootTextSize", {
    Text = "Text Size", Default = 10, Min = 8, Max = 16, Rounding = 0,
    Callback = function(v)
        for _, s in ipairs(lootTypes) do LootESP[s].textSize = v end
        for _, d in pairs(LootESP.data) do if d.t then d.t.Size = v end end
    end
})
VTab1:AddDivider()

for _, key in ipairs(lootTypes) do
    VTab1:AddToggle("LootText"..key, {
        Text = LootESP[key].label, Default = false,
        Callback = function(v)
            LootESP[key].textEnabled = v
            for _, d in pairs(LootESP.data) do
                if d and d.kind == key and d.t then
                    d.t.Visible = v and LootESP.enabled
                end
            end
        end
    }):AddColorPicker("LootText"..key.."Color", {
        Title = LootESP[key].label, Default = LootESP[key].textColor,
        Callback = function(v)
            LootESP[key].textColor = v
            for _, d in pairs(LootESP.data) do
                if d and d.kind == key and d.t then d.t.Color = v end
            end
        end
    })
end

local function isSalvage(model)
    local cached = LootESP.cache[model]
    if cached ~= nil then return cached end
    if not model:IsA("Model") then LootESP.cache[model] = false; return false end

    if model:FindFirstChild("default") then
        local n = 0
        for _, c in ipairs(model:GetChildren()) do
            if c:IsA("BasePart") and c.Name == "Part" then n = n + 1 end
        end
        if n >= 10 then LootESP.cache[model] = "Bucket"; return "Bucket" end
    end

    local boxM  = model:FindFirstChild("box")
    local trash = model:FindFirstChild("trash")
    if boxM and boxM:IsA("MeshPart") and trash and trash:IsA("MeshPart") then
        LootESP.cache[model] = "Box"; return "Box"
    end

    local bodyM = model:FindFirstChild("Body")
    local defP  = model:FindFirstChild("default")
    if bodyM and bodyM:IsA("MeshPart") and defP and defP:IsA("BasePart") then
        LootESP.cache[model] = "Chest"; return "Chest"
    end

    if model:FindFirstChild("Dispenser") and model:FindFirstChild("Machine") and model:FindFirstChild("Sign") then
        LootESP.cache[model] = "Crafting"; return "Crafting"
    end

    if model:FindFirstChild("Bottom") and model:FindFirstChild("Handles") and model:FindFirstChild("Top") then
        LootESP.cache[model] = "Crate"; return "Crate"
    end

    if model:FindFirstChild("Body") and model:FindFirstChild("Bolts") and
       model:FindFirstChild("Dials") and model:FindFirstChild("Hinge") and
       model:FindFirstChild("Pins") and model:FindFirstChild("Wheel") then
        LootESP.cache[model] = "Vault"; return "Vault"
    end

    local prim = model:FindFirstChild("Prim")
    if prim and prim:FindFirstChildWhichIsA("SpecialMesh") then
        LootESP.cache[model] = "Gas"; return "Gas"
    end

    LootESP.cache[model] = false
    return false
end

local function createLootESP(model)
    if LootESP.data[model] then return end
    local kind = isSalvage(model)
    if not kind then return end
    local s = LootESP[kind]
    if not s then return end
    local anchor = model:FindFirstChildWhichIsA("BasePart")
    if not anchor then return end

    local t = Drawing.new("Text")
    t.Text = s.label; t.Size = s.textSize; t.Center = true; t.Font = 2
    t.Outline = true; t.OutlineColor = Color3.new(0,0,0)
    t.Color = s.textColor; t.Visible = false

    local conn = model.AncestryChanged:Connect(function()
        if not model.Parent then
            t:Remove()
            LootESP.data[model]  = nil
            LootESP.cache[model] = nil
        end
    end)

    LootESP.data[model] = {t = t, anchor = anchor, conn = conn, kind = kind}
end

local function removeLootESP(model)
    local d = LootESP.data[model]
    if not d then return end
    d.conn:Disconnect()
    if d.t then d.t:Remove() end
    LootESP.data[model]  = nil
    LootESP.cache[model] = nil
end

for _, m in ipairs(workspace:GetChildren()) do task.spawn(createLootESP, m) end
workspace.ChildAdded:Connect(function(m) task.spawn(createLootESP, m) end)
workspace.ChildRemoved:Connect(removeLootESP)

-- -----------------------------------------------

local OreESP = {
    enabled     = false,
    maxDistance = 750,
    textSize    = 10,
    Cache       = setmetatable({}, {__mode = "k"}),
    ESP         = {},
    Types = {
        Stone   = { enabled = false, color = Color3.fromRGB(120, 120, 120) },
        Iron    = { enabled = false, color = Color3.fromRGB(255, 215, 0)   },
        Nitrate = { enabled = false, color = Color3.fromRGB(200, 255, 200) },
    },
}

local STONE_C   = Color3.fromRGB(72,  72,  72)
local IRON_C1   = Color3.fromRGB(199, 172, 120)
local NITRATE_C = Color3.fromRGB(248, 248, 248)

local function detectOre(model)
    local cached = OreESP.Cache[model]
    if cached ~= nil then return cached.t, cached.p end
    local fp = model:FindFirstChildOfClass("MeshPart")
    if not fp then OreESP.Cache[model] = {t=nil,p=nil}; return nil end
    local parts = {fp}
    for _, c in ipairs(model:GetChildren()) do
        if c:IsA("MeshPart") and c ~= fp then
            table.insert(parts, c)
            if #parts >= 2 then break end
        end
    end
    local t, p
    if #parts == 1 and fp.Color == STONE_C then
        t, p = "Stone", fp
    elseif #parts == 2 then
        local c1, c2 = parts[1].Color, parts[2].Color
        if (c1 == STONE_C and c2 == IRON_C1) or (c2 == STONE_C and c1 == IRON_C1) then
            t, p = "Iron", parts[1]
        elseif (c1 == NITRATE_C and c2 == STONE_C) or (c2 == NITRATE_C and c1 == STONE_C) then
            t, p = "Nitrate", parts[1]
        end
    end
    OreESP.Cache[model] = {t=t, p=p}
    return t, p
end

local function addOre(m)
    if OreESP.ESP[m] then return end
    local ot, op = detectOre(m)
    if not ot then return end
    local tx = Drawing.new("Text")
    tx.Text = ot; tx.Size = OreESP.textSize; tx.Center = true
    tx.Outline = true; tx.OutlineColor = Color3.new(0,0,0)
    tx.Color = OreESP.Types[ot].color; tx.Visible = false
    OreESP.ESP[m] = {Text = tx, OreType = ot, Part = op}
end

local function removeOre(m)
    local d = OreESP.ESP[m]
    if not d then return end
    if d.Text then d.Text:Remove() end
    OreESP.ESP[m] = nil; OreESP.Cache[m] = nil
end

local function scanOres()
    for _, m in ipairs(workspace:GetChildren()) do
        if m:IsA("Model") and not OreESP.ESP[m] then addOre(m) end
    end
end

task.spawn(scanOres)
workspace.ChildAdded:Connect(function(m) if m:IsA("Model") then task.wait(); addOre(m) end end)
workspace.ChildRemoved:Connect(removeOre)
task.spawn(function()
    while true do
        task.wait(3)
        scanOres()
        for m in pairs(OreESP.Cache) do
            if not m or not m.Parent then OreESP.Cache[m] = nil end
        end
    end
end)

VTab2:AddToggle("OreESP_Master", {
    Text = "Enable Ore ESP", Default = false,
    Callback = function(v)
        OreESP.enabled = v
        if not v then for _, d in pairs(OreESP.ESP) do if d.Text then d.Text.Visible = false end end end
    end
})
VTab2:AddSlider("OreESP_MaxDist", {
    Text = "Max Distance", Default = 750, Min = 250, Max = 1250, Rounding = 0,
    Callback = function(v) OreESP.maxDistance = v end
})
VTab2:AddSlider("OreESP_TextSize", {
    Text = "Text Size", Default = 10, Min = 8, Max = 16, Rounding = 0,
    Callback = function(v)
        OreESP.textSize = v
        for _, d in pairs(OreESP.ESP) do if d.Text then d.Text.Size = v end end
    end
})
VTab2:AddDivider()

for _, oreName in ipairs({"Stone","Iron","Nitrate"}) do
    VTab2:AddToggle("OreESP_"..oreName, {
        Text = oreName, Default = false,
        Callback = function(v) OreESP.Types[oreName].enabled = v end
    }):AddColorPicker("OreESP_"..oreName.."Color", {
        Default = OreESP.Types[oreName].color, Title = oreName.." Color",
        Callback = function(c)
            OreESP.Types[oreName].color = c
            for _, d in pairs(OreESP.ESP) do
                if d.OreType == oreName and d.Text then d.Text.Color = c end
            end
        end
    })
end

-- -----------------------------------------------

local CorpseESP = {
    enabled     = false,
    maxDistance = 750,
    textSize    = 10,
    color       = Color3.fromRGB(0, 120, 255),
    ESP         = {},
    Cache       = setmetatable({}, {__mode = "k"}),
}

local function isCorpse(m)
    local cached = CorpseESP.Cache[m]
    if cached ~= nil then return cached end
    local count, mat1, mat2 = 0, nil, nil
    for _, c in ipairs(m:GetChildren()) do
        if c:IsA("BasePart") then
            count = count + 1
            if     count == 1 then mat1 = c.Material
            elseif count == 2 then mat2 = c.Material
            else CorpseESP.Cache[m] = false; return false end
        end
    end
    if count ~= 2 then CorpseESP.Cache[m] = false; return false end
    local v = (mat1 == Enum.Material.Fabric and mat2 == Enum.Material.Metal) or
              (mat1 == Enum.Material.Metal   and mat2 == Enum.Material.Fabric)
    CorpseESP.Cache[m] = v
    return v
end

local function addCorpse(m)
    if CorpseESP.ESP[m] then return end
    if not isCorpse(m) then return end
    local tx = Drawing.new("Text")
    tx.Text = "Corpse"; tx.Size = CorpseESP.textSize; tx.Center = true
    tx.Outline = true; tx.OutlineColor = Color3.new(0,0,0)
    tx.Color = CorpseESP.color; tx.Visible = false
    CorpseESP.ESP[m] = {Text = tx, Part = m.PrimaryPart or m:FindFirstChildWhichIsA("BasePart")}
end

local function removeCorpse(m)
    local d = CorpseESP.ESP[m]
    if not d then return end
    if d.Text then d.Text:Remove() end
    CorpseESP.ESP[m] = nil; CorpseESP.Cache[m] = nil
end

local function scanCorpses()
    for _, m in ipairs(workspace:GetChildren()) do
        if m:IsA("Model") and not CorpseESP.ESP[m] then addCorpse(m) end
    end
end

task.spawn(scanCorpses)
workspace.ChildAdded:Connect(function(m) if not m:IsA("Model") then return end; task.wait(); addCorpse(m) end)
workspace.ChildRemoved:Connect(removeCorpse)
task.spawn(function()
    while true do
        task.wait(3)
        scanCorpses()
        for m in pairs(CorpseESP.Cache) do
            if not m or not m.Parent then CorpseESP.Cache[m] = nil end
        end
    end
end)

local crosshairGroup = Tabs.VISUAL:AddRightGroupbox("OTHER ESP")

crosshairGroup:AddToggle("CorpseESP_Enable", {
    Text = "Corpse ESP", Default = false,
    Callback = function(v)
        CorpseESP.enabled = v
        if not v then for _, d in pairs(CorpseESP.ESP) do if d.Text then d.Text.Visible = false end end end
    end
}):AddColorPicker("CorpseESP_Color", {
    Title = "Corpse Color", Default = Color3.fromRGB(255, 0, 0),
    Callback = function(c)
        CorpseESP.color = c
        for _, d in pairs(CorpseESP.ESP) do if d.Text then d.Text.Color = c end end
    end
})
crosshairGroup:AddSlider("CorpseESP_Dist", {
    Text = "Max Distance", Default = 750, Min = 250, Max = 1250, Rounding = 0,
    Callback = function(v) CorpseESP.maxDistance = v end
})
crosshairGroup:AddSlider("CorpseESP_TextSize", {
    Text = "Text Size", Default = 10, Min = 8, Max = 16, Rounding = 0,
    Callback = function(v)
        CorpseESP.textSize = v
        for _, d in pairs(CorpseESP.ESP) do if d.Text then d.Text.Size = v end end
    end
})

-- -----------------------------------------------

local LOOT_INTERVAL = 1 / 60
local lootLastTick  = 0

RunService.Heartbeat:Connect(function()
    local now = tick()
    if now - lootLastTick < LOOT_INTERVAL then return end
    lootLastTick = now

    local camPos = cam.CFrame.Position
    local vp     = cam.ViewportSize

    if LootESP.enabled then
        local maxDSq = LootESP.maxDistance * LootESP.maxDistance
        for model, d in pairs(LootESP.data) do
            if not model or not model.Parent or not d.anchor or not d.anchor.Parent then
                removeLootESP(model); continue
            end
            local s = LootESP[d.kind]
            if not s then removeLootESP(model); continue end
            local anchorPos = d.kind == "Vault"
                and (d.anchor.CFrame * CFrame.new(2,0,0)).Position
                or d.anchor.Position
            local diff = anchorPos - camPos
            if d.t then
                if s.textEnabled and (diff.X*diff.X + diff.Y*diff.Y + diff.Z*diff.Z) <= maxDSq then
                    local sp, on = cam:WorldToViewportPoint(anchorPos)
                    if on then
                        d.t.Text     = s.label
                        d.t.Position = Vector2.new(
                            math.clamp(sp.X, 20, vp.X - 20),
                            math.clamp(sp.Y - 20, 20, vp.Y - 20)
                        )
                        d.t.Color   = s.textColor
                        d.t.Visible = true
                    else
                        d.t.Visible = false
                    end
                else
                    d.t.Visible = false
                end
            end
        end
    else
        for _, d in pairs(LootESP.data) do if d.t then d.t.Visible = false end end
    end

    if OreESP.enabled then
        local maxDSq = OreESP.maxDistance * OreESP.maxDistance
        for m, d in pairs(OreESP.ESP) do
            local p = d.Part
            if p and p.Parent and OreESP.Types[d.OreType].enabled then
                local diff = p.Position - camPos
                if diff.X*diff.X + diff.Y*diff.Y + diff.Z*diff.Z <= maxDSq then
                    local sp, on = cam:WorldToViewportPoint(p.Position)
                    if on then
                        d.Text.Position = Vector2.new(sp.X, sp.Y)
                        d.Text.Visible  = true
                    else
                        d.Text.Visible = false
                    end
                else
                    d.Text.Visible = false
                end
            else
                if d.Text then d.Text.Visible = false end
            end
        end
    else
        for _, d in pairs(OreESP.ESP) do if d.Text then d.Text.Visible = false end end
    end

    if CorpseESP.enabled then
        local maxDSq = CorpseESP.maxDistance * CorpseESP.maxDistance
        for m, d in pairs(CorpseESP.ESP) do
            local p = d.Part
            if p and p.Parent then
                local diff = p.Position - camPos
                if diff.X*diff.X + diff.Y*diff.Y + diff.Z*diff.Z <= maxDSq then
                    local sp, on = cam:WorldToViewportPoint(p.Position)
                    if on then
                        d.Text.Position = Vector2.new(sp.X, sp.Y - 20)
                        d.Text.Visible  = true
                    else
                        d.Text.Visible = false
                    end
                else
                    d.Text.Visible = false
                end
            else
                removeCorpse(m)
            end
        end
    else
        for _, d in pairs(CorpseESP.ESP) do if d.Text then d.Text.Visible = false end end
    end
end)

-- -----------------------------------------------

local ZONA = Tabs.VISUAL:AddLeftGroupbox("ZONE CHARMS")

local Zones = {
    SafeZone = {
        part = workspace:FindFirstChild("World") and
               workspace.World:FindFirstChild("Zones") and
               workspace.World.Zones:FindFirstChild("SafeZones") and
               workspace.World.Zones.SafeZones:FindFirstChild("SAFEZONE_Town"),
        color = Color3.fromRGB(0, 255, 0),
    },
    NoBuild = {
        parts = workspace:FindFirstChild("World") and
                workspace.World:FindFirstChild("Zones") and
                workspace.World.Zones:FindFirstChild("NoBuildZones"),
        color = Color3.fromRGB(255, 0, 0),
    }
}

local szToggle = ZONA:AddToggle("SafeZone", {
    Text = "SafeZone", Default = false,
    Callback = function(state)
        local p = Zones.SafeZone.part
        if not p then return end
        if state then
            p.Transparency = 0
            p.Material = Enum.Material.ForceField
            p.Color = Zones.SafeZone.color
        else
            p.Transparency = 1
            p.Material = Enum.Material.Neon
        end
    end
})
szToggle:AddColorPicker("SafeZoneColor", {
    Default = Zones.SafeZone.color,
    Title = "SafeZone Color",
    Transparency = 0,
    Callback = function(c)
        Zones.SafeZone.color = c
        if Toggles.SafeZone.Value and Zones.SafeZone.part then
            Zones.SafeZone.part.Color = c
        end
    end
})

local nbToggle = ZONA:AddToggle("BildZone", {
    Text = "NoBuild Zone", Default = false,
    Callback = function(state)
        if not Zones.NoBuild.parts then return end
        for _, p in ipairs(Zones.NoBuild.parts:GetDescendants()) do
            if p:IsA("BasePart") then
                if state then
                    p.Transparency = 0
                    p.Material = Enum.Material.ForceField
                    p.Color = Zones.NoBuild.color
                else
                    p.Transparency = 1
                    p.Material = Enum.Material.Neon
                end
            end
        end
    end
})
nbToggle:AddColorPicker("BildZoneColor", {
    Default = Zones.NoBuild.color,
    Title = "NoBuild Color",
    Transparency = 0,
    Callback = function(c)
        Zones.NoBuild.color = c
        if Toggles.BildZone.Value and Zones.NoBuild.parts then
            for _, p in ipairs(Zones.NoBuild.parts:GetDescendants()) do
                if p:IsA("BasePart") then p.Color = c end
            end
        end
    end
})

-- -----------------------------------------------

local UIGroup = Tabs["UI Settings"]:AddLeftGroupbox("UI Settings")

UIGroup:AddToggle("KeybindMenuOpen", {
    Default = Library.KeybindFrame.Visible, Text = "Show Keybind Menu",
    Callback = function(v) Library.KeybindFrame.Visible = v end
})
UIGroup:AddToggle("ShowCustomCursor", {
    Text = "Custom Cursor", Default = true,
    Callback = function(v) Library.ShowCustomCursor = v end
})
UIGroup:AddDropdown("NotificationSide", {
    Values = {"Left","Right"}, Default = "Right", Text = "Notification Side",
    Callback = function(v) Library:SetNotifySide(v) end
})
UIGroup:AddDropdown("DPIDropdown", {
    Values = {"50%","75%","100%","125%","150%","175%","200%"}, Default = "100%", Text = "DPI Scale",
    Callback = function(v) local dpi = tonumber(v:gsub("%%","")); Library:SetDPIScale(dpi) end
})
UIGroup:AddDivider()

local menuKeyLabel = UIGroup:AddLabel("Menu Keybind")
menuKeyLabel:AddKeyPicker("MenuKeybind", {
    Default = "RightShift", NoUI = false, Text = "Menu Keybind",
    Callback = function() end
})
UIGroup:AddButton("Unload", function() Library:Unload() end)

ThemeManager:SetLibrary(Library)
ThemeManager:SetFolder("AV-X")
ThemeManager:ApplyToTab(Tabs["UI Settings"])

SaveManager:SetLibrary(Library)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({"MenuKeybind"})
SaveManager:SetFolder("AV-X")
SaveManager:BuildConfigSection(Tabs["UI Settings"])

ThemeManager.Library:SetFont(Enum.Font.Jura)
ThemeManager.Library:UpdateColorsUsingRegistry()

SaveManager:LoadAutoloadConfig()

if Options and Options.MenuKeybind then
    Library.ToggleKeybind = Options.MenuKeybind
end
