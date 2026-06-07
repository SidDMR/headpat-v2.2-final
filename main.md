# headpat-v2.2-final
simple headpat GUI that i created for roblox while bored, tested in 2 games and seemed to work fine.


local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local lp = Players.LocalPlayer
local playerGui = lp:WaitForChild("PlayerGui")

local existing = playerGui:FindFirstChild("HeadPatGui")
if existing then existing:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HeadPatGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 210, 0, 82)
main.Position = UDim2.new(0.5, -105, 0.05, 0)
main.BackgroundColor3 = Color3.fromRGB(26, 32, 26)
main.BorderSizePixel = 0
main.Parent = screenGui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 8)

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 28)
titleBar.BackgroundColor3 = Color3.fromRGB(18, 22, 18)
titleBar.BorderSizePixel = 0
titleBar.Parent = main
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 8)
local tfix = Instance.new("Frame")
tfix.Size = UDim2.new(1, 0, 0, 8)
tfix.Position = UDim2.new(0, 0, 1, -8)
tfix.BackgroundColor3 = Color3.fromRGB(18, 22, 18)
tfix.BorderSizePixel = 0
tfix.Parent = titleBar

local titleLbl = Instance.new("TextLabel")
titleLbl.Size = UDim2.new(1, -36, 1, 0)
titleLbl.Position = UDim2.new(0, 10, 0, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "Head Pat"
titleLbl.TextColor3 = Color3.fromRGB(200, 200, 200)
titleLbl.TextSize = 12
titleLbl.Font = Enum.Font.GothamBold
titleLbl.TextXAlignment = Enum.TextXAlignment.Left
titleLbl.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 24, 0, 18)
closeBtn.Position = UDim2.new(1, -28, 0, 5)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "x"
closeBtn.TextColor3 = Color3.fromRGB(180, 70, 70)
closeBtn.TextSize = 14
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = titleBar
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

local row = Instance.new("Frame")
row.Size = UDim2.new(1, -16, 0, 34)
row.Position = UDim2.new(0, 8, 0, 36)
row.BackgroundColor3 = Color3.fromRGB(36, 43, 36)
row.BorderSizePixel = 0
row.Parent = main
Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)

local rowLabel = Instance.new("TextLabel")
rowLabel.Size = UDim2.new(1, -52, 1, 0)
rowLabel.Position = UDim2.new(0, 10, 0, 0)
rowLabel.BackgroundTransparency = 1
rowLabel.Text = "Head Pat"
rowLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
rowLabel.TextSize = 12
rowLabel.Font = Enum.Font.Gotham
rowLabel.TextXAlignment = Enum.TextXAlignment.Left
rowLabel.Parent = row

local toggleBg = Instance.new("Frame")
toggleBg.Size = UDim2.new(0, 36, 0, 20)
toggleBg.Position = UDim2.new(1, -44, 0.5, -10)
toggleBg.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
toggleBg.BorderSizePixel = 0
toggleBg.Parent = row
Instance.new("UICorner", toggleBg).CornerRadius = UDim.new(1, 0)

local circle = Instance.new("Frame")
circle.Size = UDim2.new(0, 14, 0, 14)
circle.Position = UDim2.new(0, 3, 0.5, -7)
circle.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
circle.BorderSizePixel = 0
circle.Parent = toggleBg
Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)

local rowBtn = Instance.new("TextButton")
rowBtn.Size = UDim2.new(1, 0, 1, 0)
rowBtn.BackgroundTransparency = 1
rowBtn.Text = ""
rowBtn.Parent = row

-- Drag
local dragging, dragStart, startPos = false, nil, nil
titleBar.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = i.Position; startPos = main.Position
    end
end)
titleBar.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)
UserInputService.InputChanged:Connect(function(i)
    if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
        local d = i.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)

-- Pat logic
local patActive = false
local toggled = false
local motor, orig

local function getMotor()
    local char = lp.Character
    if not char then return nil end
    for _, p in ipairs(char:GetDescendants()) do
        if p:IsA("Motor6D") then
            local n = p.Name:lower()
            if n:find("right") and (n:find("shoulder") or n:find("arm")) then return p end
        end
    end
end

local function getNearestPlayer()
    local char = lp.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local nearest, minDist = nil, 15
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local r = p.Character:FindFirstChild("HumanoidRootPart")
            if r then
                local d = (root.Position - r.Position).Magnitude
                if d < minDist then minDist = d; nearest = p end
            end
        end
    end
    return nearest
end

local function tw(m, c0, t)
    TweenService:Create(m, TweenInfo.new(t, Enum.EasingStyle.Sine), {C0 = c0}):Play()
    task.wait(t)
end

local function doPat()
    local char = lp.Character
    if not char then return end

    local newMotor = getMotor()
    if not newMotor then return end
    if newMotor ~= motor then
        motor = newMotor
        orig = motor.C0
    end
    if not orig then orig = motor.C0 end

    local nearest = getNearestPlayer()
    if not nearest or not nearest.Character then return end
    local targetHead = nearest.Character:FindFirstChild("Head")
    if not targetHead then return end

    local root = char:FindFirstChild("HumanoidRootPart")
    local targetRoot = nearest.Character:FindFirstChild("HumanoidRootPart")
    if not root or not targetRoot then return end

    local lookCF = CFrame.lookAt(root.Position, Vector3.new(targetRoot.Position.X, root.Position.Y, targetRoot.Position.Z))
    TweenService:Create(root, TweenInfo.new(0.15, Enum.EasingStyle.Sine), {CFrame = lookCF}):Play()
    task.wait(0.15)

    local shoulderPos = (motor.Part0.CFrame * CFrame.new(orig.Position)).Position
    local headPos = targetHead.Position + Vector3.new(0, 0.6, 0)
    local localDiff = motor.Part0.CFrame:VectorToObjectSpace(headPos - shoulderPos)
    local horizontalDist = math.sqrt(localDiff.X ^ 2 + localDiff.Z ^ 2)
    local elevation = math.atan2(localDiff.Y, horizontalDist)

    local aimC0   = orig * CFrame.Angles(math.rad(90), 0, -elevation)
    local patUp   = orig * CFrame.Angles(math.rad(90), 0, -elevation - math.rad(8))
    local patDown = aimC0

    tw(motor, aimC0,   0.2)
    tw(motor, patDown, 0.09)
    tw(motor, patUp,   0.09)
    tw(motor, patDown, 0.09)
    tw(motor, patUp,   0.09)
    tw(motor, patDown, 0.09)
    tw(motor, patUp,   0.09)
    tw(motor, patDown, 0.09)
    tw(motor, patUp,   0.09)
    tw(motor, patDown, 0.09)
    tw(motor, patUp,   0.09)
end

local function restoreArm()
    if motor and orig then
        TweenService:Create(motor, TweenInfo.new(0.3, Enum.EasingStyle.Sine), {C0 = orig}):Play()
        task.wait(0.3)
        orig = nil
        motor = nil
    end
end

local function setToggle(state)
    toggled = state
    if state then
        toggleBg.BackgroundColor3 = Color3.fromRGB(38, 105, 38)
        TweenService:Create(circle, TweenInfo.new(0.15), {Position = UDim2.new(1, -17, 0.5, -7)}):Play()
        patActive = true
        task.spawn(function()
            while patActive do
                doPat()
                task.wait(0.2)
            end
            restoreArm()
        end)
    else
        toggleBg.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
        TweenService:Create(circle, TweenInfo.new(0.15), {Position = UDim2.new(0, 3, 0.5, -7)}):Play()
        patActive = false
    end
end

rowBtn.MouseButton1Click:Connect(function() setToggle(not toggled) end)
