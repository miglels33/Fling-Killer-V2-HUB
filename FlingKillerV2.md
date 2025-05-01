-- Fling Killer V2 - Complete Edition
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Settings
local FLING_DURATION = 5
local BASE_DISTANCE = 1.5
local FLING_POWER = 500
local FLING_UP = 100
local SPEED_MULTIPLIER = 4.0

-- Couch Coordinates (CFrame)
local COUCH_CFRAME = CFrame.new(
    -82.6146545, 19.1627922, -129.998917,
    -0.987688661, -0.0271274857, -0.154069379,
    3.72731847e-05, 0.984809637, -0.173637524,
    0.156439334, -0.171505392, -0.972684145
)

-- Emergency Coordinates (CFrame)
local EMERGENCY_CFRAME = CFrame.new(
    9.76088238, 3.29197598, 19.2252636,
    0.18106319, 8.98323407e-08, 0.983471453,
    -7.28798213e-08, 1, -7.79244687e-08,
    -0.983471453, -5.75659733e-08, 0.18106319
)

-- Create GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlingKillerV2"
screenGui.Parent = player.PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 420)
mainFrame.Position = UDim2.new(0, 10, 0.5, -210)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BackgroundTransparency = 0.15
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

-- Title Bar
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
titleBar.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -70, 1, 0)
title.Text = "FLING KILLER V2"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.BackgroundTransparency = 1
title.Parent = titleBar

-- Close Button
local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -35, 0, 2)
closeButton.Text = "X"
closeButton.TextColor3 = Color3.new(1, 1, 1)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeButton.Parent = titleBar

-- Player List
local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(1, -10, 0, 180)
playerList.Position = UDim2.new(0, 5, 0, 40)
playerList.BackgroundTransparency = 1
playerList.ScrollBarThickness = 6
playerList.Parent = mainFrame

-- Main Buttons
local flingButton = Instance.new("TextButton")
flingButton.Size = UDim2.new(1, -10, 0, 40)
flingButton.Position = UDim2.new(0, 5, 0, 230)
flingButton.Text = "ACTIVATE FLING KILLER"
flingButton.TextColor3 = Color3.new(1, 1, 1)
flingButton.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
flingButton.Font = Enum.Font.GothamBold
flingButton.Parent = mainFrame

local couchButton = Instance.new("TextButton")
couchButton.Size = UDim2.new(1, -10, 0, 40)
couchButton.Position = UDim2.new(0, 5, 0, 280)
couchButton.Text = "GET COUCH (NEED)"
couchButton.TextColor3 = Color3.new(1, 1, 1)
couchButton.BackgroundColor3 = Color3.fromRGB(80, 120, 200)
couchButton.Font = Enum.Font.GothamBold
couchButton.Parent = mainFrame

local groundButton = Instance.new("TextButton")
groundButton.Size = UDim2.new(1, -10, 0, 40)
groundButton.Position = UDim2.new(0, 5, 0, 330)
groundButton.Text = "TELEPORT TO GROUND (FOR EMERGENCIES)"
groundButton.TextColor3 = Color3.new(1, 1, 1)
groundButton.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
groundButton.Font = Enum.Font.GothamBold
groundButton.TextScaled = true
groundButton.Parent = mainFrame

-- Variables
local selectedPlayer = nil
local isFlingActive = false
local originalPosition = nil

-- Refresh Player List
local function refreshPlayerList()
    for _, child in ipairs(playerList:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end
    
    local yPos = 0
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local playerButton = Instance.new("TextButton")
            playerButton.Size = UDim2.new(1, 0, 0, 35)
            playerButton.Position = UDim2.new(0, 0, 0, yPos)
            playerButton.Text = "  ▶ "..otherPlayer.Name
            playerButton.TextXAlignment = Enum.TextXAlignment.Left
            playerButton.TextColor3 = Color3.new(1, 1, 1)
            playerButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
            playerButton.Parent = playerList
            
            playerButton.MouseButton1Click:Connect(function()
                selectedPlayer = otherPlayer
                for _, btn in ipairs(playerList:GetChildren()) do
                    if btn:IsA("TextButton") then
                        btn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
                        btn.Text = btn.Text:gsub("✓", "▶")
                    end
                end
                playerButton.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
                playerButton.Text = playerButton.Text:gsub("▶", "✓")
            end)
            
            yPos = yPos + 38
        end
    end
    playerList.CanvasSize = UDim2.new(0, 0, 0, yPos)
end

-- Teleport to Couch (2 seconds only)
local function teleportToCouch()
    if originalPosition then return end
    
    originalPosition = humanoidRootPart.CFrame
    humanoidRootPart.CFrame = COUCH_CFRAME
    
    task.delay(2, function()
        if humanoidRootPart and originalPosition then
            humanoidRootPart.CFrame = originalPosition
        end
        originalPosition = nil
    end)
end

-- Emergency Teleport to Specific Coordinates
local function emergencyTeleport()
    humanoidRootPart.CFrame = EMERGENCY_CFRAME
end

-- Fling Killer Function
local function activateFlingKiller(target)
    if not target or not target.Character or isFlingActive then return end
    
    isFlingActive = true
    flingButton.Text = "FLINGING..."
    
    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
    if not targetRoot then
        isFlingActive = false
        flingButton.Text = "ACTIVATE FLING KILLER"
        return
    end
    
    local originalCFrame = humanoidRootPart.CFrame
    local originalCollision = humanoidRootPart.CanCollide
    humanoidRootPart.CanCollide = false
    
    local targetVelocity = Instance.new("BodyVelocity")
    targetVelocity.Velocity = Vector3.new(0, 0, 0)
    targetVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    targetVelocity.Parent = targetRoot
    
    local startTime = time()
    local flingLoop
    flingLoop = RunService.Heartbeat:Connect(function()
        if not targetRoot or (time() - startTime) > FLING_DURATION then
            flingLoop:Disconnect()
            return
        end
        
        local offset = targetRoot.CFrame.LookVector * (math.sin(time() * 10) * BASE_DISTANCE)
        humanoidRootPart.CFrame = CFrame.new(targetRoot.Position + offset, targetRoot.Position)
        targetVelocity.Velocity = (targetRoot.Position - humanoidRootPart.Position).Unit * FLING_POWER + Vector3.new(0, FLING_UP, 0)
    end)
    
    task.delay(FLING_DURATION, function()
        if targetVelocity then targetVelocity:Destroy() end
        if humanoidRootPart and originalCFrame then
            humanoidRootPart.CFrame = originalCFrame
            humanoidRootPart.CanCollide = originalCollision
        end
        isFlingActive = false
        flingButton.Text = "ACTIVATE FLING KILLER"
    end)
end

-- Connect buttons
flingButton.MouseButton1Click:Connect(function()
    if selectedPlayer and not isFlingActive then
        activateFlingKiller(selectedPlayer)
    end
end)

couchButton.MouseButton1Click:Connect(teleportToCouch)
groundButton.MouseButton1Click:Connect(emergencyTeleport)
closeButton.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Initialize
refreshPlayerList()
Players.PlayerAdded:Connect(refreshPlayerList)
Players.PlayerRemoving:Connect(refreshPlayerList)
