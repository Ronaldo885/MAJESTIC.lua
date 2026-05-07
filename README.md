-- MAJESTIC - ULTIMATE VERSION
-- BLACK PANEL | PURE PURPLE STYLE (NO BLUE)
-- ROUNDED BORDERS

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- ==============================================
-- CONFIGURATIONS
-- ==============================================
local settings = {
    aimbotEnabled = false,
    aimbotMode = "Aimbot",
    aimbotSmoothness = 8,
    aimbotFOV = 150,
    aimbotPart = "Head",
    teamCheck = false,
    visibleOnly = false,
    showFOV = true,
    espEnabled = false,
    espBox = true,
    espSkeleton = true,
    espHealthBar = true,
    espName = true,
    espDistance = true,
    espLines = true,
}

local isAiming = false
local fovCircle = nil
local fovGui = nil
local mainGui = nil

-- PURE PURPLE COLORS (NO BLUE/CYAN)
local DARK_PURPLE = Color3.fromRGB(60, 0, 80)
local MEDIUM_PURPLE = Color3.fromRGB(100, 0, 120)
local LIGHT_PURPLE = Color3.fromRGB(140, 0, 160)

-- ==============================================
-- AIMBOT FUNCTIONS
-- ==============================================

local function IsTargetVisible(targetPart)
    if not targetPart then return false end
    
    local character = LocalPlayer.Character
    if not character then return false end
    
    local origin = Camera.CFrame.Position
    local targetPos = targetPart.Position
    local direction = (targetPos - origin).Unit
    local distance = (targetPos - origin).Magnitude
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {character, targetPart.Parent}
    
    local raycastResult = Workspace:Raycast(origin, direction * distance, raycastParams)
    
    if raycastResult then
        local hit = raycastResult.Instance
        return hit:IsDescendantOf(targetPart.Parent)
    end
    return true
end

local function IsSameTeam(player)
    local localTeam = LocalPlayer.Team
    local targetTeam = player.Team
    
    if localTeam and targetTeam and localTeam == targetTeam then
        return true
    end
    return false
end

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then isAiming = true end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then isAiming = false end
end)

function getBestTarget()
    local best = nil
    local bestDist = settings.aimbotFOV + 1
    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
    
    for _, other in ipairs(Players:GetPlayers()) do
        if other == LocalPlayer then continue end
        
        if settings.teamCheck and IsSameTeam(other) then
            continue
        end
        
        local character = other.Character
        local hum = character and character:FindFirstChild("Humanoid")
        if not hum or hum.Health <= 0 then continue end
        
        local part = character:FindFirstChild(settings.aimbotPart) or character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Head")
        if not part then continue end
        
        if settings.visibleOnly and not IsTargetVisible(part) then
            continue
        end
        
        local screenPos = Camera:WorldToScreenPoint(part.Position)
        if screenPos then
            local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
            if dist <= settings.aimbotFOV and dist < bestDist then
                bestDist = dist
                best = other
            end
        end
    end
    return best
end

function doAimbot()
    if not settings.aimbotEnabled or not isAiming then return end
    local target = getBestTarget()
    if not target or not target.Character then return end
    
    local part = target.Character:FindFirstChild(settings.aimbotPart) or target.Character:FindFirstChild("HumanoidRootPart") or target.Character:FindFirstChild("Head")
    if not part then return end
    
    local screenPos = Camera:WorldToScreenPoint(part.Position)
    if not screenPos then return end
    
    local diff = Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(Mouse.X, Mouse.Y)
    if diff.Magnitude < 0.5 then return end
    
    if settings.aimbotMode == "Aimlock" then
        mousemoverel(diff.X, diff.Y)
    else
        mousemoverel(diff.X / settings.aimbotSmoothness, diff.Y / settings.aimbotSmoothness)
    end
end

-- ==============================================
-- AIMBOT FOV CIRCLE (FIXED ON MOUSE)
-- ==============================================
function CreateFOVCircle()
    if fovCircle then 
        if fovCircle.Parent then fovCircle.Parent:Destroy() end
        fovCircle = nil
    end
    
    if not settings.showFOV then return end
    
    fovGui = Instance.new("ScreenGui")
    fovGui.Name = "FOVGui"
    fovGui.Parent = CoreGui
    
    fovCircle = Instance.new("Frame")
    fovCircle.Size = UDim2.new(0, settings.aimbotFOV * 2, 0, settings.aimbotFOV * 2)
    fovCircle.BackgroundTransparency = 1
    fovCircle.Parent = fovGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = fovCircle
    
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = MEDIUM_PURPLE
    stroke.Transparency = 0.5
    stroke.Parent = fovCircle
    
    fovGui.Visible = settings.aimbotEnabled
    UpdateFOVPosition()
end

function UpdateFOVPosition()
    if not fovCircle or not fovCircle.Parent then return end
    fovCircle.Position = UDim2.new(0, Mouse.X - settings.aimbotFOV, 0, Mouse.Y - settings.aimbotFOV)
end

function UpdateFOVSize()
    if fovCircle and fovCircle.Parent then
        fovCircle.Size = UDim2.new(0, settings.aimbotFOV * 2, 0, settings.aimbotFOV * 2)
        UpdateFOVPosition()
    end
end

RunService.RenderStepped:Connect(function()
    if settings.aimbotEnabled and settings.showFOV and fovCircle and fovCircle.Parent then
        UpdateFOVPosition()
    end
end)

-- ==============================================
-- ESP DRAWINGS
-- ==============================================
local SkeletonDrawings = {}
local ESPDrawings = {}
local LineDrawings = {}

local WHITE_COLOR = Color3.fromRGB(255, 255, 255)
local RED_COLOR = Color3.fromRGB(255, 0, 0)
local YELLOW_COLOR = Color3.fromRGB(255, 255, 0)
local GREEN_COLOR = Color3.fromRGB(0, 255, 0)

local bodyConnections = {
    R15 = {
        {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"}, {"LowerTorso", "LeftUpperLeg"},
        {"LowerTorso", "RightUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
        {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}, {"UpperTorso", "LeftUpperArm"},
        {"UpperTorso", "RightUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
        {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
    },
    R6 = {
        {"Head", "Torso"}, {"Torso", "Left Arm"}, {"Torso", "Right Arm"}, {"Torso", "Left Leg"}, {"Torso", "Right Leg"},
    }
}

local function getBodyPart(character, partName)
    local part = character:FindFirstChild(partName)
    if part then return part end
    
    local r6Map = {
        UpperTorso = "Torso", LowerTorso = "Torso", LeftUpperArm = "Left Arm", RightUpperArm = "Right Arm",
        LeftLowerArm = "Left Arm", RightLowerArm = "Right Arm", LeftHand = "Left Arm", RightHand = "Right Arm",
        LeftUpperLeg = "Left Leg", RightUpperLeg = "Right Leg", LeftLowerLeg = "Left Leg", RightLowerLeg = "Right Leg",
        LeftFoot = "Left Leg", RightFoot = "Right Leg",
    }
    
    return character:FindFirstChild(r6Map[partName] or partName)
end

-- ESP LINES
local function createLine()
    local line = Drawing.new("Line")
    line.Thickness = 1.5
    line.Transparency = 1
    line.Color = MEDIUM_PURPLE
    line.Visible = false
    return line
end

function UpdateLines()
    if not settings.espEnabled or not settings.espLines then
        for player, line in pairs(LineDrawings) do
            if line then pcall(function() line:Remove() end) end
        end
        LineDrawings = {}
        return
    end
    
    local localChar = LocalPlayer.Character
    local localRoot = localChar and (localChar:FindFirstChild("HumanoidRootPart") or localChar:FindFirstChild("Torso") or localChar:FindFirstChild("Head"))
    
    if not localRoot then return end
    
    local origin = Camera:WorldToViewportPoint(localRoot.Position)
    local originOnScreen = origin.Z > 0
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        
        if settings.teamCheck and IsSameTeam(player) then
            if LineDrawings[player] then
                pcall(function() LineDrawings[player]:Remove() end)
                LineDrawings[player] = nil
            end
            continue
        end
        
        local character = player.Character
        local humanoid = character and character:FindFirstChild("Humanoid")
        local isAlive = character and humanoid and humanoid.Health > 0
        
        if not isAlive then
            if LineDrawings[player] then
                pcall(function() LineDrawings[player]:Remove() end)
                LineDrawings[player] = nil
            end
            continue
        end
        
        local targetPart = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso") or character:FindFirstChild("Head")
        if not targetPart then continue end
        
        local targetPos = Camera:WorldToViewportPoint(targetPart.Position)
        local targetOnScreen = targetPos.Z > 0
        
        if originOnScreen and targetOnScreen then
            if not LineDrawings[player] then
                LineDrawings[player] = createLine()
            end
            LineDrawings[player].From = Vector2.new(origin.X, origin.Y)
            LineDrawings[player].To = Vector2.new(targetPos.X, targetPos.Y)
            LineDrawings[player].Visible = true
        else
            if LineDrawings[player] then
                LineDrawings[player].Visible = false
            end
        end
    end
end

-- SKELETON
local function createSkeletonLine()
    local line = Drawing.new("Line")
    line.Thickness = 2
    line.Transparency = 1
    line.Color = MEDIUM_PURPLE
    line.Visible = false
    return line
end

local function createJointPoint()
    local circle = Drawing.new("Circle")
    circle.Radius = 2
    circle.Thickness = 1
    circle.Transparency = 1
    circle.Color = MEDIUM_PURPLE
    circle.Filled = true
    circle.Visible = false
    return circle
end

local function ClearPlayerSkeleton(player)
    local skeleton = SkeletonDrawings[player]
    if skeleton then
        for _, line in pairs(skeleton.lines) do
            if line then pcall(function() line:Remove() end) end
        end
        for _, joint in pairs(skeleton.joints) do
            if joint then pcall(function() joint:Remove() end) end
        end
        SkeletonDrawings[player] = nil
    end
end

function UpdateSkeletonESP()
    if not settings.espEnabled or not settings.espSkeleton then
        for player, _ in pairs(SkeletonDrawings) do
            ClearPlayerSkeleton(player)
        end
        return 
    end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        
        if settings.teamCheck and IsSameTeam(player) then
            if SkeletonDrawings[player] then ClearPlayerSkeleton(player) end
            continue
        end
        
        local character = player.Character
        local humanoid = character and character:FindFirstChild("Humanoid")
        local isAlive = character and humanoid and humanoid.Health > 0
        
        if not isAlive then
            if SkeletonDrawings[player] then ClearPlayerSkeleton(player) end
            continue
        end
        
        local rigType = humanoid and humanoid.RigType.Name or "R6"
        local connections = bodyConnections[rigType] or bodyConnections.R6
        
        if not SkeletonDrawings[player] then
            SkeletonDrawings[player] = { lines = {}, joints = {} }
            for _, conn in pairs(connections) do
                SkeletonDrawings[player].lines[conn[1] .. "_" .. conn[2]] = createSkeletonLine()
            end
            local jointsSet = {}
            for _, conn in pairs(connections) do 
                jointsSet[conn[1]] = true
                jointsSet[conn[2]] = true 
            end
            for jointName in pairs(jointsSet) do
                SkeletonDrawings[player].joints[jointName] = createJointPoint()
            end
        end
        
        local skeleton = SkeletonDrawings[player]
        
        for _, conn in pairs(connections) do
            local partA = getBodyPart(character, conn[1])
            local partB = getBodyPart(character, conn[2])
            local line = skeleton.lines[conn[1] .. "_" .. conn[2]]
            
            if partA and partB and line then
                local posA, onScreenA = Camera:WorldToViewportPoint(partA.Position)
                local posB, onScreenB = Camera:WorldToViewportPoint(partB.Position)
                if onScreenA and onScreenB then
                    line.From = Vector2.new(posA.X, posA.Y)
                    line.To = Vector2.new(posB.X, posB.Y)
                    line.Visible = true
                else
                    line.Visible = false
                end
            end
        end
        
        for jointName, joint in pairs(skeleton.joints) do
            local part = getBodyPart(character, jointName)
            if part then
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    joint.Position = Vector2.new(pos.X, pos.Y)
                    joint.Visible = true
                else
                    joint.Visible = false
                end
            end
        end
    end
end

-- BOX AND HEALTH
local function createColoredBox()
    local box = {
        Outline = Drawing.new("Square"),
        TopLine = Drawing.new("Line"),
        BottomLine = Drawing.new("Line"),
        LeftLine = Drawing.new("Line"),
        RightLine = Drawing.new("Line")
    }
    for _, obj in pairs(box) do
        obj.Thickness = 2
        obj.Transparency = 1
        obj.Color = MEDIUM_PURPLE
        obj.Visible = false
    end
    box.Outline.Filled = false
    return box
end

local function createHealthBar()
    return { 
        Background = Drawing.new("Square"), 
        Fill = Drawing.new("Square")
    }
end

local function ClearPlayerESP(player)
    local esp = ESPDrawings[player]
    if esp then
        if esp.Box then
            for _, obj in pairs(esp.Box) do 
                if obj then pcall(function() obj:Remove() end) end
            end
        end
        if esp.Health then
            if esp.Health.Background then pcall(function() esp.Health.Background:Remove() end) end
            if esp.Health.Fill then pcall(function() esp.Health.Fill:Remove() end) end
        end
        if esp.Text then pcall(function() esp.Text:Remove() end) end
        ESPDrawings[player] = nil
    end
end

function UpdateESP()
    if not settings.espEnabled then
        for player, _ in pairs(ESPDrawings) do
            ClearPlayerESP(player)
        end
        return
    end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        
        if settings.teamCheck and IsSameTeam(player) then
            if ESPDrawings[player] then ClearPlayerESP(player) end
            continue
        end
        
        local character = player.Character
        local humanoid = character and character:FindFirstChild("Humanoid")
        local isAlive = character and humanoid and humanoid.Health > 0
        
        if not isAlive then
            if ESPDrawings[player] then ClearPlayerESP(player) end
            continue
        end
        
        local head = character:FindFirstChild("Head")
        local rootPart = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso")
        
        if head and rootPart then
            if not ESPDrawings[player] then
                ESPDrawings[player] = { Box = createColoredBox(), Health = createHealthBar() }
            end
            
            local esp = ESPDrawings[player]
            local rootPos, rootOnScreen = Camera:WorldToViewportPoint(rootPart.Position)
            
            if rootOnScreen then
                local boxHeight = 110
                local boxWidth = 70
                local boxY = rootPos.Y - boxHeight / 2
                local boxX = rootPos.X - boxWidth / 2
                local dist = (Camera.CFrame.Position - (head or rootPart).Position).Magnitude
                
                if settings.espBox and esp.Box then
                    esp.Box.Outline.Size = Vector2.new(boxWidth, boxHeight)
                    esp.Box.Outline.Position = Vector2.new(boxX, boxY)
                    esp.Box.Outline.Visible = true
                    
                    esp.Box.TopLine.From = Vector2.new(boxX, boxY)
                    esp.Box.TopLine.To = Vector2.new(boxX + boxWidth, boxY)
                    esp.Box.TopLine.Visible = true
                    
                    esp.Box.BottomLine.From = Vector2.new(boxX, boxY + boxHeight)
                    esp.Box.BottomLine.To = Vector2.new(boxX + boxWidth, boxY + boxHeight)
                    esp.Box.BottomLine.Visible = true
                    
                    esp.Box.LeftLine.From = Vector2.new(boxX, boxY)
                    esp.Box.LeftLine.To = Vector2.new(boxX, boxY + boxHeight)
                    esp.Box.LeftLine.Visible = true
                    
                    esp.Box.RightLine.From = Vector2.new(boxX + boxWidth, boxY)
                    esp.Box.RightLine.To = Vector2.new(boxX + boxWidth, boxY + boxHeight)
                    esp.Box.RightLine.Visible = true
                elseif esp.Box then
                    for _, obj in pairs(esp.Box) do 
                        if obj then obj.Visible = false end 
                    end
                end
                
                if settings.espHealthBar and esp.Health then
                    local healthPercent = humanoid.Health / humanoid.MaxHealth
                    local barWidth = 4
                    local barHeight = boxHeight
                    local barX = boxX - barWidth - 3
                    
                    local healthColor = healthPercent < 0.3 and RED_COLOR or 
                                    (healthPercent < 0.7 and YELLOW_COLOR or GREEN_COLOR)
                    
                    esp.Health.Background.Size = Vector2.new(barWidth, barHeight)
                    esp.Health.Background.Position = Vector2.new(barX, boxY)
                    esp.Health.Background.Color = Color3.fromRGB(30, 30, 30)
                    esp.Health.Background.Filled = true
                    esp.Health.Background.Transparency = 0.5
                    esp.Health.Background.Visible = true
                    
                    local fillHeight = barHeight * healthPercent
                    esp.Health.Fill.Size = Vector2.new(barWidth, fillHeight)
                    esp.Health.Fill.Position = Vector2.new(barX, boxY + barHeight - fillHeight)
                    esp.Health.Fill.Color = healthColor
                    esp.Health.Fill.Filled = true
                    esp.Health.Fill.Transparency = 0
                    esp.Health.Fill.Visible = true
                elseif esp.Health then
                    if esp.Health.Background then esp.Health.Background.Visible = false end
                    if esp.Health.Fill then esp.Health.Fill.Visible = false end
                end
                
                if settings.espName or settings.espDistance then
                    if not esp.Text then
                        esp.Text = Drawing.new("Text")
                        esp.Text.Size = 12
                        esp.Text.Color = WHITE_COLOR
                        esp.Text.Center = true
                        esp.Text.Outline = true
                        esp.Text.OutlineColor = Color3.fromRGB(0, 0, 0)
                        esp.Text.Transparency = 1
                        esp.Text.Visible = false
                    end
                    
                    local displayText = ""
                    if settings.espName then
                        displayText = player.Name
                    end
                    if settings.espDistance then
                        displayText = displayText .. (displayText ~= "" and " " or "") .. "[" .. math.floor(dist) .. "m]"
                    end
                    
                    esp.Text.Text = displayText
                    esp.Text.Position = Vector2.new(rootPos.X, boxY + boxHeight + 12)
                    esp.Text.Visible = true
                elseif esp.Text then
                    esp.Text.Visible = false
                end
            else
                if esp.Box then 
                    for _, obj in pairs(esp.Box) do 
                        if obj then obj.Visible = false end 
                    end 
                end
                if esp.Health then
                    if esp.Health.Background then esp.Health.Background.Visible = false end
                    if esp.Health.Fill then esp.Health.Fill.Visible = false end
                end
                if esp.Text then esp.Text.Visible = false end
            end
        end
    end
end

-- ==============================================
-- GET PLAYER THUMBNAIL FUNCTION
-- ==============================================
local function getPlayerThumbnail()
    local userId = LocalPlayer.UserId
    local content, isReady = Players:GetUserThumbnailAsync(userId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420)
    return content
end

-- ==============================================
-- PANEL FUNCTIONS
-- ==============================================
function createSection(parent, title)
    local section = Instance.new("Frame")
    section.Size = UDim2.new(1, 0, 0, 28)
    section.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    section.BackgroundTransparency = 0.3
    section.Parent = parent
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 6)
    c.Parent = section
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.Text = title
    titleLabel.TextColor3 = LIGHT_PURPLE
    titleLabel.TextSize = 11
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.BackgroundTransparency = 1
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Parent = section
    
    return section
end

function createToggle(parent, text, var, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 32)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.3
    frame.Parent = parent
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 6)
    c.Parent = frame
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.65, -10, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.Gotham
    label.Parent = frame
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 60, 0, 24)
    btn.Position = UDim2.new(1, -70, 0.5, -12)
    btn.Text = settings[var] and "ON" or "OFF"
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BackgroundColor3 = settings[var] and DARK_PURPLE or Color3.fromRGB(40, 0, 40)
    btn.BackgroundTransparency = 0.3
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    btn.Parent = frame
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn
    
    btn.MouseButton1Click:Connect(function()
        settings[var] = not settings[var]
        btn.Text = settings[var] and "ON" or "OFF"
        btn.BackgroundColor3 = settings[var] and DARK_PURPLE or Color3.fromRGB(40, 0, 40)
        if callback then callback(settings[var]) end
    end)
    return frame
end

function createSlider(parent, text, var, min, max, format, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 52)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.3
    frame.Parent = parent
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 6)
    c.Parent = frame
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -30, 0, 18)
    label.Position = UDim2.new(0, 10, 0, 4)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.Gotham
    label.Parent = frame
    
    local valLabel = Instance.new("TextLabel")
    valLabel.Size = UDim2.new(0, 50, 0, 18)
    valLabel.Position = UDim2.new(1, -60, 0, 4)
    valLabel.Text = string.format(format, settings[var])
    valLabel.TextColor3 = MEDIUM_PURPLE
    valLabel.TextSize = 11
    valLabel.TextXAlignment = Enum.TextXAlignment.Right
    valLabel.BackgroundTransparency = 1
    valLabel.Font = Enum.Font.GothamBold
    valLabel.Parent = frame
    
    local slider = Instance.new("Frame")
    slider.Size = UDim2.new(1, -30, 0, 3)
    slider.Position = UDim2.new(0, 10, 0, 32)
    slider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    slider.Parent = frame
    
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((settings[var] - min) / (max - min), 0, 1, 0)
    fill.BackgroundColor3 = MEDIUM_PURPLE
    fill.Parent = slider
    
    local knob = Instance.new("TextButton")
    knob.Size = UDim2.new(0, 10, 0, 10)
    knob.Position = UDim2.new((settings[var] - min) / (max - min), -5, 0.5, -5)
    knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    knob.Text = ""
    knob.Parent = slider
    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius = UDim.new(1, 0)
    knobCorner.Parent = knob
    
    local dragging = false
    knob.MouseButton1Down:Connect(function() dragging = true end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    
    RunService.RenderStepped:Connect(function()
        if dragging then
            local percent = math.clamp((Mouse.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X, 0, 1)
            local value = math.floor(min + (max - min) * percent)
            settings[var] = value
            fill.Size = UDim2.new(percent, 0, 1, 0)
            knob.Position = UDim2.new(percent, -5, 0.5, -5)
            valLabel.Text = string.format(format, value)
            if callback then callback(value) end
        end
    end)
    return frame
end

function createDropdown(parent, text, var, options, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 32)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.3
    frame.Parent = parent
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 6)
    c.Parent = frame
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.5, -10, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.Gotham
    label.Parent = frame
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.4, -10, 0, 24)
    btn.Position = UDim2.new(0.6, 0, 0.5, -12)
    btn.Text = settings[var]
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    btn.BackgroundTransparency = 0.5
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 11
    btn.Parent = frame
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn
    
    local menuOpen = false
    local menu = nil
    btn.MouseButton1Click:Connect(function()
        if menuOpen then
            if menu then menu:Destroy() end
            menuOpen = false
            return
        end
        
        menu = Instance.new("Frame")
        menu.Size = UDim2.new(0, 130, 0, #options * 24)
        menu.Position = UDim2.new(0.6, 0, 0, -((#options * 24) + 5))
        menu.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        menu.BackgroundTransparency = 0.1
        menu.BorderSizePixel = 1
        menu.BorderColor3 = MEDIUM_PURPLE
        menu.Parent = frame
        local menuCorner = Instance.new("UICorner")
        menuCorner.CornerRadius = UDim.new(0, 6)
        menuCorner.Parent = menu
        local menuLayout = Instance.new("UIListLayout")
        menuLayout.Padding = UDim.new(0, 1)
        menuLayout.Parent = menu
        
        for _, opt in ipairs(options) do
            local optBtn = Instance.new("TextButton")
            optBtn.Size = UDim2.new(1, 0, 0, 23)
            optBtn.Text = opt
            optBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
            optBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
            optBtn.Font = Enum.Font.Gotham
            optBtn.TextSize = 10
            optBtn.Parent = menu
            local optCorner = Instance.new("UICorner")
            optCorner.CornerRadius = UDim.new(0, 4)
            optCorner.Parent = optBtn
            
            optBtn.MouseButton1Click:Connect(function()
                settings[var] = opt
                btn.Text = opt
                if callback then callback(opt) end
                menu:Destroy()
                menuOpen = false
            end)
        end
        menuOpen = true
    end)
    return frame
end

-- ==============================================
-- CREATE PANEL
-- ==============================================
for _, g in pairs(CoreGui:GetChildren()) do
    if g.Name == "Majestic" then g:Destroy() end
end

mainGui = Instance.new("ScreenGui")
mainGui.Name = "Majestic"
mainGui.Parent = CoreGui

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 700, 0, 500)
panel.Position = UDim2.new(0.5, -350, 0.5, -250)
panel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
panel.BackgroundTransparency = 0.1
panel.BorderSizePixel = 2
panel.BorderColor3 = MEDIUM_PURPLE
panel.Parent = mainGui
panel.Active = true
panel.Draggable = true

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 16)
panelCorner.Parent = panel

-- ==============================================
-- TITLE BAR
-- ==============================================
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 45)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
titleBar.BackgroundTransparency = 0.3
titleBar.Parent = panel

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 12)
titleBarCorner.Parent = titleBar

-- MAJESTIC TITLE WITH GRADIENT
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(0, 200, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "MAJESTIC"
titleLabel.TextColor3 = LIGHT_PURPLE
titleLabel.TextSize = 28
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.TextYAlignment = Enum.TextYAlignment.Center
titleLabel.Parent = titleBar

-- PURPLE GRADIENT
local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(128, 0, 128)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(160, 0, 180)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(128, 0, 128))
})
gradient.Rotation = 0
gradient.Parent = titleLabel

task.spawn(function()
    local angle = 0
    while titleLabel and titleLabel.Parent do
        angle = angle + 0.5
        gradient.Rotation = math.sin(angle * 0.01) * 90
        task.wait(0.05)
    end
end)

-- PLAYER NAME
local playerName = LocalPlayer.Name
local playerNameLabel = Instance.new("TextLabel")
playerNameLabel.Size = UDim2.new(0, 120, 1, 0)
playerNameLabel.Position = UDim2.new(1, -145, 0, 0)
playerNameLabel.BackgroundTransparency = 1
playerNameLabel.Text = playerName
playerNameLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
playerNameLabel.TextSize = 12
playerNameLabel.Font = Enum.Font.GothamBold
playerNameLabel.TextXAlignment = Enum.TextXAlignment.Right
playerNameLabel.TextYAlignment = Enum.TextYAlignment.Center
playerNameLabel.Parent = titleBar

-- PLAYER AVATAR
local playerAvatarUrl = getPlayerThumbnail()
local avatarImage = Instance.new("ImageLabel")
avatarImage.Size = UDim2.new(0, 28, 0, 28)
avatarImage.Position = UDim2.new(1, -170, 0.5, -14)
avatarImage.BackgroundTransparency = 1
avatarImage.Image = playerAvatarUrl
avatarImage.Parent = titleBar
local avatarCorner = Instance.new("UICorner")
avatarCorner.CornerRadius = UDim.new(1, 0)
avatarCorner.Parent = avatarImage

-- ==============================================
-- TABS
-- ==============================================
local tabContainer = Instance.new("Frame")
tabContainer.Size = UDim2.new(1, 0, 0, 35)
tabContainer.Position = UDim2.new(0, 0, 0, 45)
tabContainer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
tabContainer.BackgroundTransparency = 0.3
tabContainer.Parent = panel

local tabCorner = Instance.new("UICorner")
tabCorner.CornerRadius = UDim.new(0, 8)
tabCorner.Parent = tabContainer

local aimbotTab = Instance.new("TextButton")
aimbotTab.Size = UDim2.new(0.5, -2, 1, -2)
aimbotTab.Position = UDim2.new(0, 2, 0, 2)
aimbotTab.Text = "AIMBOT"
aimbotTab.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotTab.BackgroundColor3 = DARK_PURPLE
aimbotTab.BackgroundTransparency = 0.5
aimbotTab.Font = Enum.Font.GothamBold
aimbotTab.TextSize = 14
aimbotTab.Parent = tabContainer
local aimCorner = Instance.new("UICorner")
aimCorner.CornerRadius = UDim.new(0, 8)
aimCorner.Parent = aimbotTab

local espTab = Instance.new("TextButton")
espTab.Size = UDim2.new(0.5, -2, 1, -2)
espTab.Position = UDim2.new(0.5, 2, 0, 2)
espTab.Text = "ESP"
espTab.TextColor3 = Color3.fromRGB(255, 255, 255)
espTab.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
espTab.BackgroundTransparency = 0.5
espTab.Font = Enum.Font.GothamBold
espTab.TextSize = 14
espTab.Parent = tabContainer
local espCorner = Instance.new("UICorner")
espCorner.CornerRadius = UDim.new(0, 8)
espCorner.Parent = espTab

local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, -20, 1, -100)
contentContainer.Position = UDim2.new(0, 10, 0, 85)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = panel

-- AIMBOT SCROLL
local aimbotScroll = Instance.new("ScrollingFrame")
aimbotScroll.Size = UDim2.new(1, 0, 1, 0)
aimbotScroll.BackgroundTransparency = 1
aimbotScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
aimbotScroll.ScrollBarThickness = 6
aimbotScroll.ScrollBarImageColor3 = MEDIUM_PURPLE
aimbotScroll.Visible = true
aimbotScroll.Parent = contentContainer

local aimbotLayout = Instance.new("UIListLayout")
aimbotLayout.Padding = UDim.new(0, 5)
aimbotLayout.Parent = aimbotScroll

-- ESP SCROLL
local espScroll = Instance.new("ScrollingFrame")
espScroll.Size = UDim2.new(1, 0, 1, 0)
espScroll.BackgroundTransparency = 1
espScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
espScroll.ScrollBarThickness = 6
espScroll.ScrollBarImageColor3 = MEDIUM_PURPLE
espScroll.Visible = false
espScroll.Parent = contentContainer

local espLayout = Instance.new("UIListLayout")
espLayout.Padding = UDim.new(0, 5)
espLayout.Parent = espScroll

-- BUILD AIMBOT TAB (ALL IN ENGLISH)
createSection(aimbotScroll, "MAIN CONFIGURATIONS")
createToggle(aimbotScroll, "Enable Aimbot", "aimbotEnabled", function(v)
    if v and settings.showFOV then 
        CreateFOVCircle() 
    elseif not v then 
        if fovGui then 
            fovGui:Destroy() 
            fovGui = nil
            fovCircle = nil
        end 
    end
end)
createDropdown(aimbotScroll, "Aim Mode", "aimbotMode", {"Aimbot", "Aimlock"}, nil)
createSlider(aimbotScroll, "Smoothness", "aimbotSmoothness", 1, 20, "%d", nil)

createSection(aimbotScroll, "FOV CONFIGURATIONS")
createSlider(aimbotScroll, "FOV Radius", "aimbotFOV", 50, 600, "%dpx", function() 
    UpdateFOVSize() 
end)
createToggle(aimbotScroll, "Show FOV", "showFOV", nil)

createSection(aimbotScroll, "TARGET SETTINGS")
createDropdown(aimbotScroll, "Body Part", "aimbotPart", {"Head", "HumanoidRootPart", "UpperTorso"}, nil)
createToggle(aimbotScroll, "Team Check", "teamCheck", nil)
createToggle(aimbotScroll, "Visible Only", "visibleOnly", nil)

-- BUILD ESP TAB (ALL IN ENGLISH)
createSection(espScroll, "MAIN CONFIGURATIONS")
createToggle(espScroll, "Enable ESP", "espEnabled", nil)

createSection(espScroll, "VISUAL SETTINGS")
createToggle(espScroll, "Box ESP", "espBox", nil)
createToggle(espScroll, "Skeleton ESP", "espSkeleton", nil)
createToggle(espScroll, "Health Bar", "espHealthBar", nil)
createToggle(espScroll, "Show Name", "espName", nil)
createToggle(espScroll, "Show Distance", "espDistance", nil)
createToggle(espScroll, "Lines ESP", "espLines", nil)

local function updateAimbotCanvas()
    aimbotScroll.CanvasSize = UDim2.new(0, 0, 0, aimbotLayout.AbsoluteContentSize.Y + 20)
end
local function updateEspCanvas()
    espScroll.CanvasSize = UDim2.new(0, 0, 0, espLayout.AbsoluteContentSize.Y + 20)
end

aimbotLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updateAimbotCanvas)
espLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updateEspCanvas)
task.wait(0.1)
updateAimbotCanvas()
updateEspCanvas()

-- SWITCH TABS
aimbotTab.MouseButton1Click:Connect(function()
    aimbotScroll.Visible = true
    espScroll.Visible = false
    aimbotTab.BackgroundColor3 = DARK_PURPLE
    espTab.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
end)

espTab.MouseButton1Click:Connect(function()
    aimbotScroll.Visible = false
    espScroll.Visible = true
    aimbotTab.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    espTab.BackgroundColor3 = DARK_PURPLE
end)

UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Home then
        mainGui.Enabled = not mainGui.Enabled
    end
end)

-- MAIN LOOP
RunService.RenderStepped:Connect(function()
    UpdateLines()
    UpdateSkeletonESP()
    UpdateESP()
    doAimbot()
end)

print("========================================")
print("MAJESTIC ULTIMATE LOADED!")
print("Press HOME to open/close")
print("Aimbot: FIXED to MOUSE")
print("FOV: PURPLE | Max: 600px")
print("========================================")
