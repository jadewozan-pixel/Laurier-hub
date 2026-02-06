if not game:IsLoaded() then game.Loaded:Wait() end

local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()

local Window = OrionLib:MakeWindow({
    Name = "🌿 Laurier Hub | Blox Fruits",
    HidePremium = true,
    SaveConfig = true,
    ConfigFolder = "LaurierHub"
})

getgenv().Settings = {
    AutoAttack = false,
    AutoTrial = false,
    AutoDraco = false,
    AutoSeaEvents = false,
    SafeMode = true,
    FastFlag = false,
    FastFlagValue = 0.12,
}

getgenv().AttackDelay = getgenv().Settings.FastFlagValue

local function applyFastFlag(val)
    if type(val) ~= "number" then return end
    local minVal = 0.06
    if getgenv().Settings.SafeMode then
        minVal = 0.12
    end
    if val < minVal then val = minVal end
    getgenv().AttackDelay = val
end

applyFastFlag(getgenv().Settings.FastFlagValue)

function AutoAttack()
    task.spawn(function()
        while getgenv().Settings.AutoAttack do
            local success, err = pcall(function()
                local player = game.Players.LocalPlayer
                local humanoid = player.Character and player.Character:FindFirstChild("Humanoid")
                if humanoid and humanoid.Health > 0 then
                    pcall(function()
                        game:GetService("VirtualUser"):ClickButton1(Vector2.new())
                    end)
                end
            end)
            local delay = tonumber(getgenv().AttackDelay) or 0.2
            task.wait(delay)
        end
    end)
end

function AutoTrial()
    task.spawn(function()
        while getgenv().Settings.AutoTrial do
            pcall(function()
                local plr = game.Players.LocalPlayer
                if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and workspace:FindFirstChild("Trials") then
                    local ok, cf = pcall(function() return workspace.Trials.CFrame end)
                    if ok and cf then
                        plr.Character.HumanoidRootPart.CFrame = cf + Vector3.new(0,5,0)
                    end
                end
            end)
            task.wait(getgenv().AttackDelay or 1)
        end
    end)
end

function AutoDraco()
    task.spawn(function()
        while getgenv().Settings.AutoDraco do
            pcall(function()
                local plr = game.Players.LocalPlayer
                if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and workspace:FindFirstChild("DracoTrial") then
                    local ok, cf = pcall(function() return workspace.DracoTrial.CFrame end)
                    if ok and cf then
                        plr.Character.HumanoidRootPart.CFrame = cf + Vector3.new(0,5,0)
                    end
                end
            end)
            task.wait(getgenv().AttackDelay or 1)
        end
    end)
end

function AutoSeaEvents()
    task.spawn(function()
        while getgenv().Settings.AutoSeaEvents do
            pcall(function()
                local plr = game.Players.LocalPlayer
                if not (plr and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")) then return end
                for _, v in pairs(workspace:GetChildren()) do
                    if v:IsA("Model") and v:FindFirstChild("PrimaryPart") then
                        local nm = v.Name:lower()
                        if nm:find("mirage") or nm:find("terror") or nm:find("shark") or nm:find("sea") then
                            local ok, part = pcall(function() return v.PrimaryPart end)
                            if ok and part then
                                pcall(function()
                                    plr.Character.HumanoidRootPart.CFrame = part.CFrame + Vector3.new(0,20,0)
                                end)
                                task.wait(getgenv().AttackDelay or 2)
                            end
                        end
                    end
                end
            end)
            task.wait(1)
        end
    end)
end

function SafeModeLoop()
    task.spawn(function()
        while true do
            pcall(function()
                if getgenv().Settings.SafeMode then
                    local plr = game.Players.LocalPlayer
                    if plr and plr.Character and plr.Character:FindFirstChild("Humanoid") and plr.Character:FindFirstChild("HumanoidRootPart") then
                        local hum = plr.Character.Humanoid
                        if hum and hum.Health > 0 and hum.MaxHealth and hum.Health / hum.MaxHealth <= 0.30 then
                            pcall(function()
                                plr.Character.HumanoidRootPart.CFrame = CFrame.new(0,500,0)
                            end)
                            task.wait(4)
                        end
                    end
                end
            end)
            task.wait(1)
        end
    end)
end

SafeModeLoop()

local Tab = Window:MakeTab({Name = "Main", Icon = "rbxassetid://4483345998", PremiumOnly = false})

Tab:AddToggle({
    Name = "⚔️ Auto Attack",
    Default = getgenv().Settings.AutoAttack,
    Callback = function(Value)
        getgenv().Settings.AutoAttack = Value
        if Value then AutoAttack() end
    end
})

Tab:AddToggle({
    Name = "🌀 Auto Trial",
    Default = getgenv().Settings.AutoTrial,
    Callback = function(Value)
        getgenv().Settings.AutoTrial = Value
        if Value then AutoTrial() end
    end
})

Tab:AddToggle({
    Name = "🐉 Auto Draco Trial",
    Default = getgenv().Settings.AutoDraco,
    Callback = function(Value)
        getgenv().Settings.AutoDraco = Value
        if Value then AutoDraco() end
    end
})

Tab:AddToggle({
    Name = "🌊 Auto Sea Events",
    Default = getgenv().Settings.AutoSeaEvents,
    Callback = function(Value)
        getgenv().Settings.AutoSeaEvents = Value
        if Value then AutoSeaEvents() end
    end
})

Tab:AddToggle({
    Name = "🛡️ Safe Mode (Anti-Death)",
    Default = getgenv().Settings.SafeMode,
    Callback = function(Value)
        getgenv().Settings.SafeMode = Value
        applyFastFlag(tonumber(getgenv().Settings.FastFlagValue) or getgenv().AttackDelay)
    end
})

Tab:AddToggle({
    Name = "⚡ FastFlag (réduit délais d'action)",
    Default = getgenv().Settings.FastFlag,
    Callback = function(Value)
        getgenv().Settings.FastFlag = Value
        if Value then
            applyFastFlag(tonumber(getgenv().Settings.FastFlagValue) or 0.12)
        else
            local restore = 0.20
            if getgenv().Settings.SafeMode then restore = 0.25 end
            applyFastFlag(restore)
        end
    end
})

Tab:AddSlider({
    Name = "⏱️ FastFlag Delay (sec)",
    Min = 0.06,
    Max = 1,
    Default = getgenv().Settings.FastFlagValue,
    Increment = 0.01,
    Callback = function(Value)
        getgenv().Settings.FastFlagValue = Value
        if getgenv().Settings.FastFlag then
            applyFastFlag(Value)
        end
    end
})

OrionLib:Init()
