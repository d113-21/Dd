local Remote = Instance.new("RemoteEvent")
Remote.Name = "ServerLagControl"
Remote.Parent = game:GetService("ReplicatedStorage")

local isLooping = false
local loopConnection = nil
local lagCount = 10000
local lagMethod = "Print"

local methods = {
    Print = function()
        for i = 1, lagCount do
            print("ServerLag: " .. i)
        end
    end,
    Part = function()
        for i = 1, lagCount do
            local p = Instance.new("Part")
            p.Size = Vector3.new(1, 1, 1)
            p.Anchored = true
            p.CFrame = CFrame.new(
                math.random(-1000, 1000),
                math.random(-1000, 1000),
                math.random(-1000, 1000)
            )
            p.Parent = workspace
            game:GetService("Debris"):AddItem(p, 0.1)
        end
    end,
    Math = function()
        for i = 1, lagCount do
            local x = math.random(1, 1000000)
            local y = math.sqrt(x) * math.pi
            local z = math.sin(y) / math.cos(x + 1)
        end
    end
}

local function startLoop()
    if loopConnection then return end
    isLooping = true
    loopConnection = game:GetService("RunService").Heartbeat:Connect(function()
        if not isLooping then return end
        if methods[lagMethod] then
            methods[lagMethod]()
        end
    end)
end

local function stopLoop()
    isLooping = false
    if loopConnection then
        loopConnection:Disconnect()
        loopConnection = nil
    end
end

Remote.OnServerEvent:Connect(function(player, action, value)
    if action == "Toggle" then
        if value then startLoop() else stopLoop() end
    elseif action == "SetCount" then
        lagCount = tonumber(value) or 10000
        if isLooping then
            stopLoop()
            startLoop()
        end
    elseif action == "SetMethod" then
        lagMethod = tostring(value) or "Print"
        if isLooping then
            stopLoop()
            startLoop()
        end
    elseif action == "EmergencyStop" then
        stopLoop()
    end
end)
