local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

local Window = OrionLib:MakeWindow({
    Name = "Blobman Kick V2",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "BlobmanKick"
})

local function GetPlayerByName(name)
    name = name:lower()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Name:lower():sub(1, #name) == name then
            return p
        end
    end
    return nil
end

local function GetPlayerNames()
    local names = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            table.insert(names, p.Name)
        end
    end
    return names
end

local MainTab = Window:MakeTab({
    Name = "Kick",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local selectedPlayer = nil
local playerDropdown = MainTab:AddDropdown({
    Name = "Target",
    Default = "Select Player",
    Options = GetPlayerNames(),
    Callback = function(value)
        selectedPlayer = GetPlayerByName(value)
        if selectedPlayer then
            print("Target set: " .. selectedPlayer.Name)
        end
    end
})

local function UpdatePlayerList()
    local names = GetPlayerNames()
    if #names > 0 then
        playerDropdown:Refresh(names, true)
    else
        playerDropdown:Refresh({"No Players"}, true)
    end
end

Players.PlayerAdded:Connect(function()
    task.wait(1)
    UpdatePlayerList()
end)

Players.PlayerRemoving:Connect(function()
    task.wait(0.5)
    UpdatePlayerList()
end)

local kickLoopActive = false
local kickConnection = nil

local function KickBlobman(target)
    if not target then return false end
    
    local GE = ReplicatedStorage:FindFirstChild("GrabEvents")
    if not GE then
        warn("GrabEvents not found!")
        return false
    end
    
    local remoteEvent = GE:FindFirstChild("Grab") or GE:FindFirstChild("Kick") or GE:FindFirstChild("BlobmanKick")
    if not remoteEvent then
        warn("Kick remote not found!")
        for _, child in ipairs(GE:GetChildren()) do
            print("Child: " .. child.Name)
        end
        return false
    end
    
    local success, err = pcall(function()
        remoteEvent:FireServer(target)
    end)
    
    if success then
        print("Kicked: " .. target.Name)
        return true
    else
        warn("Kick failed: " .. tostring(err))
        return false
    end
end

local function StartKickLoop(target)
    if kickLoopActive then
        StopKickLoop()
    end
    
    if not target then
        OrionLib:MakeNotification({
            Name = "Error",
            Content = "No target selected!",
            Image = "rbxassetid://4483345998",
            Time = 3
        })
        return
    end
    
    kickLoopActive = true
    
    kickConnection = game:GetService("RunService").Heartbeat:Connect(function()
        if not kickLoopActive then return end
        if not target or not target.Parent then
            StopKickLoop()
            OrionLib:MakeNotification({
                Name = "Stopped",
                Content = "Target left",
                Image = "rbxassetid://4483345998",
                Time = 2
            })
            return
        end
        KickBlobman(target)
    end)
    
    OrionLib:MakeNotification({
        Name = "Started",
        Content = "Kicking: " .. target.Name,
        Image = "rbxassetid://4483345998",
        Time = 2
    })
end

local function StopKickLoop()
    kickLoopActive = false
    if kickConnection then
        kickConnection:Disconnect()
        kickConnection = nil
    end
    OrionLib:MakeNotification({
        Name = "Stopped",
        Content = "Kick loop stopped",
        Image = "rbxassetid://4483345998",
        Time = 2
    })
end

MainTab:AddButton({
    Name = "Start Kick Loop",
    Callback = function()
        StartKickLoop(selectedPlayer)
    end
})

MainTab:AddButton({
    Name = "Stop Kick Loop",
    Callback = function()
        StopKickLoop()
    end
})

MainTab:AddButton({
    Name = "Kick Once",
    Callback = function()
        if selectedPlayer then
            if KickBlobman(selectedPlayer) then
                OrionLib:MakeNotification({
                    Name = "Success",
                    Content = "Kicked " .. selectedPlayer.Name,
                    Image = "rbxassetid://4483345998",
                    Time = 2
                })
            end
        else
            OrionLib:MakeNotification({
                Name = "Error",
                Content = "Select a target first",
                Image = "rbxassetid://4483345998",
                Time = 2
            })
        end
    end
})

local StatusTab = Window:MakeTab({
    Name = "Status",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

StatusTab:AddParagraph({
    Name = "Current Status",
    Content = "Waiting...\nTarget: " .. (selectedPlayer and selectedPlayer.Name or "None")
})

local SettingsTab = Window:MakeTab({
    Name = "Settings",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local kickInterval = 0.1
SettingsTab:AddSlider({
    Name = "Kick Interval (sec)",
    Min = 0.01,
    Max = 1,
    Default = 0.1,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 0.01,
    ValueName = "s",
    Callback = function(value)
        kickInterval = value
        if kickLoopActive then
            local currentTarget = selectedPlayer
            StopKickLoop()
            task.wait(0.1)
            StartKickLoop(currentTarget)
        end
        print("Interval updated: " .. kickInterval)
    end
})

OrionLib:Init()
