-- MAJESTIC - ULTIMATE VERSION
-- COM SISTEMA DE LOGIN LOCAL (SEM GITHUB)
-- BLACK PANEL | PURE PURPLE STYLE (NO BLUE)
-- ROUNDED BORDERS
-- FIXED: Aimbot NÃO troca de alvo aleatoriamente

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local TweenService = game:GetService("TweenService")

-- ==============================================
-- CORES ROXAS (SEM AZUL)
-- ==============================================
local DARK_PURPLE = Color3.fromRGB(60, 0, 80)
local MEDIUM_PURPLE = Color3.fromRGB(100, 0, 120)
local LIGHT_PURPLE = Color3.fromRGB(140, 0, 160)
local WHITE = Color3.fromRGB(255, 255, 255)
local BLACK = Color3.fromRGB(0, 0, 0)
local GREEN = Color3.fromRGB(0, 255, 0)
local RED = Color3.fromRGB(255, 0, 0)

-- ==============================================
-- SISTEMA DE AUTENTICAÇÃO LOCAL
-- ==============================================

-- DADOS DOS USUÁRIOS (VOCÊ PODE ADICIONAR MAIS)
local users = {
    ["admin"] = {
        password = "admin123",
        keys = {
            {type = "private", duration = "Lifetime", expires = "lifetime"},
            {type = "daily", duration = "1 dia", expires = os.time() + (86400)}
        }
    },
    ["teste"] = {
        password = "123",
        keys = {
            {type = "free", duration = "Trial", expires = "lifetime"}
        }
    }
}

-- FUNÇÃO PARA VERIFICAR LOGIN
local function authenticateUser(username, password)
    if users[username] and users[username].password == password then
        return {success = true, username = username, keys = users[username].keys}
    end
    return {success = false, message = "Usuário ou senha incorretos"}
end

-- FUNÇÃO PARA ATIVAR KEY
local function activateKey(username, keyCode)
    -- Keys pré-definidas que podem ser ativadas
    local availableKeys = {
        ["PRIVATE-ULTIMATE-2024"] = {type = "private", duration = "Lifetime", expires = "lifetime"},
        ["DAILY-FREE-2024"] = {type = "daily", duration = "1 dia", expires = os.time() + 86400},
        ["WEEKLY-GOLD-2024"] = {type = "weekly", duration = "7 dias", expires = os.time() + (86400 * 7)},
        ["MONTHLY-PRO-2024"] = {type = "monthly", duration = "30 dias", expires = os.time() + (86400 * 30)}
    }
    
    if availableKeys[keyCode] then
        -- Verifica se o usuário já tem essa key
        for _, key in pairs(users[username].keys) do
            if key.type == availableKeys[keyCode].type then
                return {success = false, message = "Você já possui esta key ativa!"}
            end
        end
        
        -- Adiciona a key ao usuário
        table.insert(users[username].keys, availableKeys[keyCode])
        return {success = true, message = "Key ativada com sucesso!", keyData = availableKeys[keyCode]}
    end
    
    return {success = false, message = "Key inválida!"}
end

-- ==============================================
-- VARIÁVEIS GLOBAIS
-- ==============================================
local currentUser = nil
local userKeys = {}
local isLoggedIn = false

-- CONFIGURAÇÕES DO AIMBOT/ESP
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
local currentTarget = nil
local targetLockTime = 0
local LOCK_DURATION = 0.8

-- ==============================================
-- FUNÇÃO PARA VERIFICAR KEY ATIVA POR TIPO
-- ==============================================
local function hasKeyType(keyType)
    for _, key in pairs(userKeys) do
        if key.type == keyType then
            -- Verifica expiração
            if key.expires ~= "lifetime" and key.expires < os.time() then
                return false
            end
            return true
        end
    end
    return false
end

local function getKeyData(keyType)
    for _, key in pairs(userKeys) do
        if key.type == keyType then
            return key
        end
    end
    return nil
end

local function formatTimeRemaining(expires)
    if expires == "lifetime" then
        return "∞ Lifetime"
    end
    local remaining = expires - os.time()
    if remaining <= 0 then
        return "Expirado"
    end
    local days = math.floor(remaining / 86400)
    local hours = math.floor((remaining % 86400) / 3600)
    if days > 0 then
        return days .. "d " .. hours .. "h restantes"
    else
        return hours .. " horas restantes"
    end
end

-- ==============================================
-- LOADER / SPLASH SCREEN
-- ==============================================
local function showLoader()
    local loaderGui = Instance.new("ScreenGui")
    loaderGui.Name = "MajesticLoader"
    loaderGui.Parent = CoreGui
    loaderGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local splashFrame = Instance.new("Frame")
    splashFrame.Size = UDim2.new(1, 0, 1, 0)
    splashFrame.BackgroundColor3 = Color3.fromRGB(20, 0, 30)
    splashFrame.Parent = loaderGui
    
    local logoFrame = Instance.new("Frame")
    logoFrame.Size = UDim2.new(0, 350, 0, 450)
    logoFrame.Position = UDim2.new(0.5, -175, 0.5, -225)
    logoFrame.BackgroundColor3 = Color3.fromRGB(40, 0, 60)
    logoFrame.BackgroundTransparency = 0.5
    logoFrame.BorderSizePixel = 3
    logoFrame.BorderColor3 = LIGHT_PURPLE
    logoFrame.Parent = splashFrame
    
    local logoCorner = Instance.new("UICorner")
    logoCorner.CornerRadius = UDim.new(0, 20)
    logoCorner.Parent = logoFrame
    
    local logoText = Instance.new("TextLabel")
    logoText.Size = UDim2.new(1, 0, 0.4, 0)
    logoText.Position = UDim2.new(0, 0, 0.25, 0)
    logoText.BackgroundTransparency = 1
    logoText.Text = "MAJESTIC"
    logoText.TextColor3 = LIGHT_PURPLE
    logoText.TextSize = 36
    logoText.Font = Enum.Font.GothamBold
    logoText.TextScaled = true
    logoText.Parent = logoFrame
    
    local subtitleText = Instance.new("TextLabel")
    subtitleText.Size = UDim2.new(1, 0, 0.15, 0)
    subtitleText.Position = UDim2.new(0, 0, 0.55, 0)
    subtitleText.BackgroundTransparency = 1
    subtitleText.Text = "Carregando..."
    subtitleText.TextColor3 = WHITE
    subtitleText.TextSize = 14
    subtitleText.Font = Enum.Font.Gotham
    subtitleText.Parent = logoFrame
    
    local progressBar = Instance.new("Frame")
    progressBar.Size = UDim2.new(0.8, 0, 0, 8)
    progressBar.Position = UDim2.new(0.1, 0, 0.75, 0)
    progressBar.BackgroundColor3 = DARK_PURPLE
    progressBar.BorderSizePixel = 0
    progressBar.Parent = logoFrame
    
    local progressBarCorner = Instance.new("UICorner")
    progressBarCorner.CornerRadius = UDim.new(1, 0)
    progressBarCorner.Parent = progressBar
    
    local progressFill = Instance.new("Frame")
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.BackgroundColor3 = LIGHT_PURPLE
    progressFill.BorderSizePixel = 0
    progressFill.Parent = progressBar
    
    local progressFillCorner = Instance.new("UICorner")
    progressFillCorner.CornerRadius = UDim.new(1, 0)
    progressFillCorner.Parent = progressFill
    
    local steps = {
        {text = "Inicializando Majestic...", progress = 0.1},
        {text = "Carregando módulos...", progress = 0.25},
        {text = "Preparando interface...", progress = 0.4},
        {text = "Carregando recursos...", progress = 0.55},
        {text = "Quase lá...", progress = 0.75},
        {text = "Pronto!", progress = 1.0}
    }
    
    local stepIndex = 1
    local function updateLoader()
        if stepIndex <= #steps then
            local step = steps[stepIndex]
            subtitleText.Text = step.text
            local tween = TweenService:Create(progressFill, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {Size = UDim2.new(step.progress, 0, 1, 0)})
            tween:Play()
            tween.Completed:Wait()
            stepIndex = stepIndex + 1
            task.wait(0.2)
            updateLoader()
        else
            task.wait(0.5)
            loaderGui:Destroy()
            showLoginScreen()
        end
    end
    
    updateLoader()
end

-- ==============================================
-- TELA DE LOGIN
-- ==============================================
local function showLoginScreen()
    local loginGui = Instance.new("ScreenGui")
    loginGui.Name = "MajesticLogin"
    loginGui.Parent = CoreGui
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 400, 0, 350)
    mainFrame.Position = UDim2.new(0.5, -200, 0.5, -175)
    mainFrame.BackgroundColor3 = BLACK
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.BorderSizePixel = 2
    mainFrame.BorderColor3 = MEDIUM_PURPLE
    mainFrame.Parent = loginGui
    mainFrame.Active = true
    mainFrame.Draggable = true
    
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 16)
    mainCorner.Parent = mainFrame
    
    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 0, 60)
    titleLabel.Position = UDim2.new(0, 0, 0, 0)
    titleLabel.BackgroundColor3 = BLACK
    titleLabel.BackgroundTransparency = 0.3
    titleLabel.Text = "MAJESTIC LOGIN"
    titleLabel.TextColor3 = LIGHT_PURPLE
    titleLabel.TextSize = 24
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Parent = mainFrame
    
    -- Campos de login
    local usernameBox = Instance.new("TextBox")
    usernameBox.Size = UDim2.new(0.8, 0, 0, 40)
    usernameBox.Position = UDim2.new(0.1, 0, 0.25, 0)
    usernameBox.PlaceholderText = "Usuário"
    usernameBox.Text = ""
    usernameBox.TextColor3 = WHITE
    usernameBox.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    usernameBox.BorderSizePixel = 0
    usernameBox.Font = Enum.Font.Gotham
    usernameBox.TextSize = 14
    usernameBox.Parent = mainFrame
    
    local userCorner = Instance.new("UICorner")
    userCorner.CornerRadius = UDim.new(0, 8)
    userCorner.Parent = usernameBox
    
    local passwordBox = Instance.new("TextBox")
    passwordBox.Size = UDim2.new(0.8, 0, 0, 40)
    passwordBox.Position = UDim2.new(0.1, 0, 0.4, 0)
    passwordBox.PlaceholderText = "Senha"
    passwordBox.Text = ""
    passwordBox.TextColor3 = WHITE
    passwordBox.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    passwordBox.BorderSizePixel = 0
    passwordBox.Font = Enum.Font.Gotham
    passwordBox.TextSize = 14
    passwordBox.Parent = mainFrame
    passwordBox.ClearTextOnFocus = false
    
    local passCorner = Instance.new("UICorner")
    passCorner.CornerRadius = UDim.new(0, 8)
    passCorner.Parent = passwordBox
    
    -- Botão Login
    local loginBtn = Instance.new("TextButton")
    loginBtn.Size = UDim2.new(0.4, 0, 0, 45)
    loginBtn.Position = UDim2.new(0.3, 0, 0.55, 0)
    loginBtn.Text = "ENTRAR"
    loginBtn.TextColor3 = WHITE
    loginBtn.BackgroundColor3 = MEDIUM_PURPLE
    loginBtn.BorderSizePixel = 0
    loginBtn.Font = Enum.Font.GothamBold
    loginBtn.TextSize = 16
    loginBtn.Parent = mainFrame
    
    local loginCorner = Instance.new("UICorner")
    loginCorner.CornerRadius = UDim.new(0, 8)
    loginCorner.Parent = loginBtn
    
    -- Mensagem de status
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(0.9, 0, 0, 30)
    statusLabel.Position = UDim2.new(0.05, 0, 0.7, 0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "Use admin / admin123 para teste"
    statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    statusLabel.TextSize = 11
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.TextXAlignment = Enum.TextXAlignment.Center
    statusLabel.Parent = mainFrame
    
    loginBtn.MouseButton1Click:Connect(function()
        local username = usernameBox.Text
        local password = passwordBox.Text
        
        if username == "" or password == "" then
            statusLabel.Text = "Preencha todos os campos!"
            statusLabel.TextColor3 = RED
            return
        end
        
        statusLabel.Text = "Autenticando..."
        statusLabel.TextColor3 = WHITE
        
        local result = authenticateUser(username, password)
        if result.success then
            currentUser = result.username
            userKeys = result.keys
            statusLabel.Text = "Bem-vindo " .. username .. "!"
            statusLabel.TextColor3 = GREEN
            task.wait(1)
            loginGui:Destroy()
            showCheatMenu()
        else
            statusLabel.Text = result.message
            statusLabel.TextColor3 = RED
        end
    end)
end

-- ==============================================
-- TELA DE MENU DE CHEATS
-- ==============================================
local function showCheatMenu()
    local cheatMenuGui = Instance.new("ScreenGui")
    cheatMenuGui.Name = "MajesticCheatMenu"
    cheatMenuGui.Parent = CoreGui
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 500, 0, 550)
    mainFrame.Position = UDim2.new(0.5, -250, 0.5, -275)
    mainFrame.BackgroundColor3 = BLACK
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.BorderSizePixel = 2
    mainFrame.BorderColor3 = MEDIUM_PURPLE
    mainFrame.Parent = cheatMenuGui
    mainFrame.Active = true
    mainFrame.Draggable = true
    
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 16)
    mainCorner.Parent = mainFrame
    
    -- Header
    local headerFrame = Instance.new("Frame")
    headerFrame.Size = UDim2.new(1, 0, 0, 70)
    headerFrame.Position = UDim2.new(0, 0, 0, 0)
    headerFrame.BackgroundColor3 = BLACK
    headerFrame.BackgroundTransparency = 0.3
    headerFrame.Parent = mainFrame
    
    local headerCorner = Instance.new("UICorner")
    headerCorner.CornerRadius = UDim.new(0, 12)
    headerCorner.Parent = headerFrame
    
    local userLabel = Instance.new("TextLabel")
    userLabel.Size = UDim2.new(1, -20, 0, 25)
    userLabel.Position = UDim2.new(0, 15, 0, 10)
    userLabel.BackgroundTransparency = 1
    userLabel.Text = "👤 " .. currentUser
    userLabel.TextColor3 = WHITE
    userLabel.TextSize = 14
    userLabel.Font = Enum.Font.GothamBold
    userLabel.TextXAlignment = Enum.TextXAlignment.Left
    userLabel.Parent = headerFrame
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 0, 30)
    titleLabel.Position = UDim2.new(0, 0, 0, 35)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "MAJESTIC ULTIMATE"
    titleLabel.TextColor3 = LIGHT_PURPLE
    titleLabel.TextSize = 20
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Center
    titleLabel.Parent = headerFrame
    
    -- Container rolável
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Size = UDim2.new(1, -20, 1, -140)
    scrollFrame.Position = UDim2.new(0, 10, 0, 80)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    scrollFrame.ScrollBarThickness = 6
    scrollFrame.ScrollBarImageColor3 = MEDIUM_PURPLE
    scrollFrame.Parent = mainFrame
    
    local scrollLayout = Instance.new("UIListLayout")
    scrollLayout.Padding = UDim.new(0, 10)
    scrollLayout.Parent = scrollFrame
    
    -- Criar cards de cheat
    local function createCheatCard(cheatId, cheatName, cheatIcon, description, features, keyType)
        local hasKey = hasKeyType(keyType)
        local keyData = getKeyData(keyType)
        local isAvailable = (cheatId == "free") or hasKey
        
        local cardFrame = Instance.new("Frame")
        cardFrame.Size = UDim2.new(1, 0, 0, 130)
        cardFrame.BackgroundColor3 = BLACK
        cardFrame.BackgroundTransparency = 0.4
        cardFrame.BorderSizePixel = 1
        cardFrame.BorderColor3 = isAvailable and MEDIUM_PURPLE or RED
        cardFrame.Parent = scrollFrame
        
        local cardCorner = Instance.new("UICorner")
        cardCorner.CornerRadius = UDim.new(0, 10)
        cardCorner.Parent = cardFrame
        
        -- Nome e ícone
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(0.6, 0, 0, 30)
        nameLabel.Position = UDim2.new(0, 15, 0, 5)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = cheatIcon .. " " .. cheatName
        nameLabel.TextColor3 = isAvailable and LIGHT_PURPLE or Color3.fromRGB(100, 100, 100)
        nameLabel.TextSize = 16
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.TextXAlignment = Enum.TextXAlignment.Left
        nameLabel.Parent = cardFrame
        
        -- Descrição
        local descLabel = Instance.new("TextLabel")
        descLabel.Size = UDim2.new(0.6, 0, 0, 20)
        descLabel.Position = UDim2.new(0, 15, 0, 35)
        descLabel.BackgroundTransparency = 1
        descLabel.Text = description
        descLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
        descLabel.TextSize = 11
        descLabel.Font = Enum.Font.Gotham
        descLabel.TextXAlignment = Enum.TextXAlignment.Left
        descLabel.Parent = cardFrame
        
        -- Features
        local featuresLabel = Instance.new("TextLabel")
        featuresLabel.Size = UDim2.new(0.6, 0, 0, 50)
        featuresLabel.Position = UDim2.new(0, 15, 0, 55)
        featuresLabel.BackgroundTransparency = 1
        featuresLabel.Text = "🔧 " .. features
        featuresLabel.TextColor3 = WHITE
        featuresLabel.TextSize = 10
        featuresLabel.Font = Enum.Font.Gotham
        featuresLabel.TextXAlignment = Enum.TextXAlignment.Left
        featuresLabel.TextWrapped = true
        featuresLabel.Parent = cardFrame
        
        -- Status
        local statusLabel = Instance.new("TextLabel")
        statusLabel.Size = UDim2.new(0.35, 0, 0, 20)
        statusLabel.Position = UDim2.new(0.63, 0, 0, 10)
        statusLabel.BackgroundTransparency = 1
        statusLabel.Text = isAvailable and "✓ ATIVO" or "🔒 BLOQUEADO"
        statusLabel.TextColor3 = isAvailable and GREEN or RED
        statusLabel.TextSize = 11
        statusLabel.Font = Enum.Font.GothamBold
        statusLabel.TextXAlignment = Enum.TextXAlignment.Center
        statusLabel.Parent = cardFrame
        
        -- Tempo restante
        local timeLabel = Instance.new("TextLabel")
        timeLabel.Size = UDim2.new(0.35, 0, 0, 20)
        timeLabel.Position = UDim2.new(0.63, 0, 0, 35)
        timeLabel.BackgroundTransparency = 1
        timeLabel.Text = isAvailable and ("⏰ " .. formatTimeRemaining(keyData and keyData.expires or "lifetime")) or "Adquira uma key"
        timeLabel.TextColor3 = isAvailable and GREEN or RED
        timeLabel.TextSize = 10
        timeLabel.Font = Enum.Font.Gotham
        timeLabel.TextXAlignment = Enum.TextXAlignment.Center
        timeLabel.Parent = cardFrame
        
        -- Botão
        local actionBtn = Instance.new("TextButton")
        actionBtn.Size = UDim2.new(0.3, 0, 0, 35)
        actionBtn.Position = UDim2.new(0.67, 0, 0, 70)
        actionBtn.Text = isAvailable and "INICIAR" or "COMPRAR"
        actionBtn.TextColor3 = WHITE
        actionBtn.BackgroundColor3 = isAvailable and DARK_PURPLE or Color3.fromRGB(80, 0, 0)
        actionBtn.BorderSizePixel = 0
        actionBtn.Font = Enum.Font.GothamBold
        actionBtn.TextSize = 12
        actionBtn.Parent = cardFrame
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
        btnCorner.Parent = actionBtn
        
        actionBtn.MouseButton1Click:Connect(function()
            if isAvailable then
                cheatMenuGui:Destroy()
                if cheatId == "free" then
                    createFreeCheatGUI()
                else
                    createFullCheatGUI()
                end
            else
                statusLabel.Text = "Compre uma key!"
                statusLabel.TextColor3 = RED
                task.wait(1)
                statusLabel.Text = "🔒 BLOQUEADO"
                statusLabel.TextColor3 = RED
            end
        end)
    end
    
    -- Criar cards
    createCheatCard("free", "Majestic Free", "🎮", "Versão gratuita - Recursos básicos", "Aimbot básico, ESP simples, FOV Circle", "free")
    
    if hasKeyType("private") then
        createCheatCard("private", "Majestic Private", "👑", "Versão completa - Todos os recursos", "Aimbot avançado, ESP completo, Silent Aim, Triggerbot, Anti-Aim", "private")
    end
    
    if hasKeyType("daily") then
        createCheatCard("daily", "Majestic Daily", "⭐", "Acesso diário - 24 horas", "Aimbot completo, ESP completo, Silent Aim", "daily")
    end
    
    if hasKeyType("weekly") then
        createCheatCard("weekly", "Majestic Weekly", "🌟", "Acesso semanal - 7 dias", "Aimbot completo, ESP completo, Silent Aim, Triggerbot", "weekly")
    end
    
    if hasKeyType("monthly") then
        createCheatCard("monthly", "Majestic Monthly", "💎", "Acesso mensal - 30 dias", "Aimbot completo, ESP completo, Silent Aim, Triggerbot, Anti-Aim", "monthly")
    end
    
    -- Botão Adicionar Key
    local addKeyBtn = Instance.new("TextButton")
    addKeyBtn.Size = UDim2.new(0.9, 0, 0, 45)
    addKeyBtn.Position = UDim2.new(0.05, 0, 0.85, 0)
    addKeyBtn.Text = "🔑 ADICIONAR KEY"
    addKeyBtn.TextColor3 = WHITE
    addKeyBtn.BackgroundColor3 = MEDIUM_PURPLE
    addKeyBtn.BorderSizePixel = 0
    addKeyBtn.Font = Enum.Font.GothamBold
    addKeyBtn.TextSize = 14
    addKeyBtn.Parent = mainFrame
    
    local addKeyCorner = Instance.new("UICorner")
    addKeyCorner.CornerRadius = UDim.new(0, 8)
    addKeyCorner.Parent = addKeyBtn
    
    -- Popup para adicionar key
    local keyPopup = nil
    addKeyBtn.MouseButton1Click:Connect(function()
        if keyPopup then return end
        
        keyPopup = Instance.new("Frame")
        keyPopup.Size = UDim2.new(0, 350, 0, 200)
        keyPopup.Position = UDim2.new(0.5, -175, 0.5, -100)
        keyPopup.BackgroundColor3 = BLACK
        keyPopup.BackgroundTransparency = 0.1
        keyPopup.BorderSizePixel = 2
        keyPopup.BorderColor3 = MEDIUM_PURPLE
        keyPopup.Parent = cheatMenuGui
        
        local popupCorner = Instance.new("UICorner")
        popupCorner.CornerRadius = UDim.new(0, 12)
        popupCorner.Parent = keyPopup
        
        local popupTitle = Instance.new("TextLabel")
        popupTitle.Size = UDim2.new(1, 0, 0, 40)
        popupTitle.Position = UDim2.new(0, 0, 0, 0)
        popupTitle.BackgroundColor3 = BLACK
        popupTitle.BackgroundTransparency = 0.3
        popupTitle.Text = "ATIVAR KEY"
        popupTitle.TextColor3 = LIGHT_PURPLE
        popupTitle.TextSize = 18
        popupTitle.Font = Enum.Font.GothamBold
        popupTitle.Parent = keyPopup
        
        local keyBox = Instance.new("TextBox")
        keyBox.Size = UDim2.new(0.8, 0, 0, 35)
        keyBox.Position = UDim2.new(0.1, 0, 0.25, 0)
        keyBox.PlaceholderText = "Digite sua key aqui"
        keyBox.Text = ""
        keyBox.TextColor3 = WHITE
        keyBox.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
        keyBox.BorderSizePixel = 0
        keyBox.Font = Enum.Font.Gotham
        keyBox.TextSize = 12
        keyBox.Parent = keyPopup
        
        local keyCorner = Instance.new("UICorner")
        keyCorner.CornerRadius = UDim.new(0, 6)
        keyCorner.Parent = keyBox
        
        local infoLabel = Instance.new("TextLabel")
        infoLabel.Size = UDim2.new(0.9, 0, 0, 40)
        infoLabel.Position = UDim2.new(0.05, 0, 0.45, 0)
        infoLabel.BackgroundTransparency = 1
        infoLabel.Text = "Keys disponíveis:\nPRIVATE-ULTIMATE-2024 | DAILY-FREE-2024 | WEEKLY-GOLD-2024 | MONTHLY-PRO-2024"
        infoLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
        infoLabel.TextSize = 9
        infoLabel.Font = Enum.Font.Gotham
        infoLabel.TextXAlignment = Enum.TextXAlignment.Center
        infoLabel.TextWrapped = true
        infoLabel.Parent = keyPopup
        
        local confirmBtn = Instance.new("TextButton")
        confirmBtn.Size = UDim2.new(0.35, 0, 0, 30)
        confirmBtn.Position = UDim2.new(0.15, 0, 0.75, 0)
        confirmBtn.Text = "ATIVAR"
        confirmBtn.TextColor3 = WHITE
        confirmBtn.BackgroundColor3 = DARK_PURPLE
        confirmBtn.BorderSizePixel = 0
        confirmBtn.Font = Enum.Font.GothamBold
        confirmBtn.TextSize = 12
        confirmBtn.Parent = keyPopup
        
        local cancelBtn = Instance.new("TextButton")
        cancelBtn.Size = UDim2.new(0.35, 0, 0, 30)
        cancelBtn.Position = UDim2.new(0.55, 0, 0.75, 0)
        cancelBtn.Text = "CANCELAR"
        cancelBtn.TextColor3 = WHITE
        cancelBtn.BackgroundColor3 = Color3.fromRGB(80, 0, 0)
        cancelBtn.BorderSizePixel = 0
        cancelBtn.Font = Enum.Font.GothamBold
        cancelBtn.TextSize = 12
        cancelBtn.Parent = keyPopup
        
        local msgLabel = Instance.new("TextLabel")
        msgLabel.Size = UDim2.new(0.9, 0, 0, 20)
        msgLabel.Position = UDim2.new(0.05, 0, 0.65, 0)
        msgLabel.BackgroundTransparency = 1
        msgLabel.Text = ""
        msgLabel.TextColor3 = GREEN
        msgLabel.TextSize = 10
        msgLabel.Font = Enum.Font.Gotham
        msgLabel.TextXAlignment = Enum.TextXAlignment.Center
        msgLabel.Parent = keyPopup
        
        confirmBtn.MouseButton1Click:Connect(function()
            local key = string.upper(keyBox.Text)
            if key == "" then
                msgLabel.Text = "Digite uma key!"
                msgLabel.TextColor3 = RED
                return
            end
            
            msgLabel.Text = "Verificando..."
            msgLabel.TextColor3 = WHITE
            
            local result = activateKey(currentUser, key)
            if result.success then
                msgLabel.Text = result.message
                msgLabel.TextColor3 = GREEN
                userKeys = users[currentUser].keys
                task.wait(1)
                keyPopup:Destroy()
                keyPopup = nil
                cheatMenuGui:Destroy()
                showCheatMenu()
            else
                msgLabel.Text = result.message
                msgLabel.TextColor3 = RED
            end
        end)
        
        cancelBtn.MouseButton1Click:Connect(function()
            keyPopup:Destroy()
            keyPopup = nil
        end)
    end)
    
    -- Atualizar canvas
    local function updateCanvas()
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, scrollLayout.AbsoluteContentSize.Y + 20)
    end
    scrollLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updateCanvas)
    task.wait(0.1)
    updateCanvas()
end

-- ==============================================
-- ==============================================
-- AIMBOT FUNCTIONS (CÓDIGO ORIGINAL COMPLETO)
-- ==============================================
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
    if localTeam and targetTeam and localTeam == targetTeam then return true end
    return false
end

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then 
        isAiming = true 
        currentTarget = nil
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then 
        isAiming = false 
        currentTarget = nil
    end
end)

function getBestTarget()
    if currentTarget and tick() - targetLockTime < LOCK_DURATION then
        local character = currentTarget.Character
        local hum = character and character:FindFirstChild("Humanoid")
        if hum and hum.Health > 0 then
            local part = character:FindFirstChild(settings.aimbotPart) or 
                        character:FindFirstChild("HumanoidRootPart") or 
                        character:FindFirstChild("Head")
            if part then
                local screenPos = Camera:WorldToScreenPoint(part.Position)
                if screenPos then
                    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if dist <= settings.aimbotFOV then
                        if settings.visibleOnly and not IsTargetVisible(part) then
                            currentTarget = nil
                        else
                            return currentTarget
                        end
                    else
                        currentTarget = nil
                    end
                else
                    currentTarget = nil
                end
            else
                currentTarget = nil
            end
        else
            currentTarget = nil
        end
    end
    
    local best = nil
    local bestDist = settings.aimbotFOV + 1
    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
    
    for _, other in ipairs(Players:GetPlayers()) do
        if other == LocalPlayer then continue end
        if settings.teamCheck and IsSameTeam(other) then continue end
        local character = other.Character
        local hum = character and character:FindFirstChild("Humanoid")
        if not hum or hum.Health <= 0 then continue end
        local part = character:FindFirstChild(settings.aimbotPart) or 
                    character:FindFirstChild("HumanoidRootPart") or 
                    character:FindFirstChild("Head")
        if not part then continue end
        if settings.visibleOnly and not IsTargetVisible(part) then continue end
        local screenPos = Camera:WorldToScreenPoint(part.Position)
        if screenPos then
            local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
            if dist <= settings.aimbotFOV and dist < bestDist then
                bestDist = dist
                best = other
            end
        end
    end
    
    if best then
        currentTarget = best
        targetLockTime = tick()
    end
    return best
end

function doAimbot()
    if not settings.aimbotEnabled or not isAiming then return end
    local target = getBestTarget()
    if not target or not target.Character then return end
    local part = target.Character:FindFirstChild(settings.aimbotPart) or 
                target.Character:FindFirstChild("HumanoidRootPart") or 
                target.Character:FindFirstChild("Head")
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

function CreateFOVCircle()
    if fovCircle then 
        if fovCircle.Parent then fovCircle.Parent:Destroy() end
        fovCircle = nil
    end
    if not settings.showFOV or not settings.aimbotEnabled then return end
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

-- ==============================================
-- ESP DRAWINGS
-- ==============================================
local SkeletonDrawings = {}
local ESPDrawings = {}
local LineDrawings = {}

local bodyConnections = {
    R15 = {
        {"Head","UpperTorso"},{"UpperTorso","LowerTorso"},{"LowerTorso","LeftUpperLeg"},
        {"LowerTorso","RightUpperLeg"},{"LeftUpperLeg","LeftLowerLeg"},{"LeftLowerLeg","LeftFoot"},
        {"RightUpperLeg","RightLowerLeg"},{"RightLowerLeg","RightFoot"},{"UpperTorso","LeftUpperArm"},
        {"UpperTorso","RightUpperArm"},{"LeftUpperArm","LeftLowerArm"},{"LeftLowerArm","LeftHand"},
        {"RightUpperArm","RightLowerArm"},{"RightLowerArm","RightHand"},
    },
    R6 = {{"Head","Torso"},{"Torso","Left Arm"},{"Torso","Right Arm"},{"Torso","Left Leg"},{"Torso","Right Leg"}}
}

local function getBodyPart(character, partName)
    local part = character:FindFirstChild(partName)
    if part then return part end
    local r6Map = {UpperTorso="Torso",LowerTorso="Torso",LeftUpperArm="Left Arm",RightUpperArm="Right Arm",
        LeftLowerArm="Left Arm",RightLowerArm="Right Arm",LeftHand="Left Arm",RightHand="Right Arm",
        LeftUpperLeg="Left Leg",RightUpperLeg="Right Leg",LeftLowerLeg="Left Leg",RightLowerLeg="Right Leg",
        LeftFoot="Left Leg",RightFoot="Right Leg"}
    return character:FindFirstChild(r6Map[partName] or partName)
end

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
                    
                    local healthColor = healthPercent < 0.3 and RED or 
                                    (healthPercent < 0.7 and Color3.fromRGB(255,255,0) or GREEN)
                    
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
                        esp.Text.Color = WHITE
                        esp.Text.Center = true
                        esp.Text.Outline = true
                        esp.Text.OutlineColor = BLACK
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
-- GUI DO CHEAT COMPLETO
-- ==============================================

local function createFullCheatGUI()
    for _, g in pairs(CoreGui:GetChildren()) do
        if g.Name == "Majestic" then g:Destroy() end
    end
    
    mainGui = Instance.new("ScreenGui")
    mainGui.Name = "Majestic"
    mainGui.Parent = CoreGui
    
    local panel = Instance.new("Frame")
    panel.Size = UDim2.new(0, 700, 0, 500)
    panel.Position = UDim2.new(0.5, -350, 0.5, -250)
    panel.BackgroundColor3 = BLACK
    panel.BackgroundTransparency = 0.1
    panel.BorderSizePixel = 2
    panel.BorderColor3 = MEDIUM_PURPLE
    panel.Parent = mainGui
    panel.Active = true
    panel.Draggable = true
    
    local panelCorner = Instance.new("UICorner")
    panelCorner.CornerRadius = UDim.new(0, 16)
    panelCorner.Parent = panel
    
    -- Title bar
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 45)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = BLACK
    titleBar.BackgroundTransparency = 0.3
    titleBar.Parent = panel
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0, 250, 1, 0)
    titleLabel.Position = UDim2.new(0, 15, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "MAJESTIC ULTIMATE"
    titleLabel.TextColor3 = LIGHT_PURPLE
    titleLabel.TextSize = 24
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.TextYAlignment = Enum.TextYAlignment.Center
    titleLabel.Parent = titleBar
    
    local playerNameLabel = Instance.new("TextLabel")
    playerNameLabel.Size = UDim2.new(0, 150, 1, 0)
    playerNameLabel.Position = UDim2.new(1, -160, 0, 0)
    playerNameLabel.BackgroundTransparency = 1
    playerNameLabel.Text = currentUser
    playerNameLabel.TextColor3 = WHITE
    playerNameLabel.TextSize = 12
    playerNameLabel.Font = Enum.Font.GothamBold
    playerNameLabel.TextXAlignment = Enum.TextXAlignment.Right
    playerNameLabel.TextYAlignment = Enum.TextYAlignment.Center
    playerNameLabel.Parent = titleBar
    
    -- Tabs
    local tabContainer = Instance.new("Frame")
    tabContainer.Size = UDim2.new(1, 0, 0, 35)
    tabContainer.Position = UDim2.new(0, 0, 0, 45)
    tabContainer.BackgroundColor3 = BLACK
    tabContainer.BackgroundTransparency = 0.3
    tabContainer.Parent = panel
    
    local aimbotTab = Instance.new("TextButton")
    aimbotTab.Size = UDim2.new(0.5, -2, 1, -2)
    aimbotTab.Position = UDim2.new(0, 2, 0, 2)
    aimbotTab.Text = "AIMBOT"
    aimbotTab.TextColor3 = WHITE
    aimbotTab.BackgroundColor3 = DARK_PURPLE
    aimbotTab.BorderSizePixel = 0
    aimbotTab.Font = Enum.Font.GothamBold
    aimbotTab.TextSize = 14
    aimbotTab.Parent = tabContainer
    
    local espTab = Instance.new("TextButton")
    espTab.Size = UDim2.new(0.5, -2, 1, -2)
    espTab.Position = UDim2.new(0.5, 2, 0, 2)
    espTab.Text = "ESP"
    espTab.TextColor3 = WHITE
    espTab.BackgroundColor3 = BLACK
    espTab.BorderSizePixel = 0
    espTab.Font = Enum.Font.GothamBold
    espTab.TextSize = 14
    espTab.Parent = tabContainer
    
    local contentContainer = Instance.new("Frame")
    contentContainer.Size = UDim2.new(1, -20, 1, -100)
    contentContainer.Position = UDim2.new(0, 10, 0, 85)
    contentContainer.BackgroundTransparency = 1
    contentContainer.Parent = panel
    
    -- Aimbot content
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
    
    -- ESP content
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
    
    -- Funções UI
    local function createSection(parent, title)
        local section = Instance.new("Frame")
        section.Size = UDim2.new(1, 0, 0, 28)
        section.BackgroundColor3 = BLACK
        section.BackgroundTransparency = 0.3
        section.BorderSizePixel = 0
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
    
    local function createToggle(parent, text, var, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 32)
        frame.BackgroundColor3 = BLACK
        frame.BackgroundTransparency = 0.3
        frame.BorderSizePixel = 0
        frame.Parent = parent
        
        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, 6)
        c.Parent = frame
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.65, -10, 1, 0)
        label.Position = UDim2.new(0, 10, 0, 0)
        label.Text = text
        label.TextColor3 = WHITE
        label.TextSize = 12
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.BackgroundTransparency = 1
        label.Font = Enum.Font.Gotham
        label.Parent = frame
        
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 60, 0, 24)
        btn.Position = UDim2.new(1, -70, 0.5, -12)
        btn.Text = settings[var] and "ON" or "OFF"
        btn.TextColor3 = WHITE
        btn.BackgroundColor3 = settings[var] and DARK_PURPLE or Color3.fromRGB(40,0,40)
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 11
        btn.Parent = frame
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 6)
        btnCorner.Parent = btn
        
        btn.MouseButton1Click:Connect(function()
            settings[var] = not settings[var]
            btn.Text = settings[var] and "ON" or "OFF"
            btn.BackgroundColor3 = settings[var] and DARK_PURPLE or Color3.fromRGB(40,0,40)
            if callback then callback(settings[var]) end
        end)
        
        return frame
    end
    
    local function createSlider(parent, text, var, minVal, maxVal, format, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 52)
        frame.BackgroundColor3 = BLACK
        frame.BackgroundTransparency = 0.3
        frame.BorderSizePixel = 0
        frame.Parent = parent
        
        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, 6)
        c.Parent = frame
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, -30, 0, 18)
        label.Position = UDim2.new(0, 10, 0, 4)
        label.Text = text
        label.TextColor3 = WHITE
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
        slider.BackgroundColor3 = Color3.fromRGB(40,40,50)
        slider.BorderSizePixel = 0
        slider.Parent = frame
        
        local fill = Instance.new("Frame")
        local percent = (settings[var]-minVal)/(maxVal-minVal)
        fill.Size = UDim2.new(percent, 0, 1, 0)
        fill.BackgroundColor3 = MEDIUM_PURPLE
        fill.BorderSizePixel = 0
        fill.Parent = slider
        
        local knob = Instance.new("TextButton")
        knob.Size = UDim2.new(0, 10, 0, 10)
        knob.Position = UDim2.new(percent, -5, 0.5, -5)
        knob.BackgroundColor3 = WHITE
        knob.BorderSizePixel = 0
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
                local value = math.floor(minVal + (maxVal - minVal) * percent)
                settings[var] = value
                fill.Size = UDim2.new(percent, 0, 1, 0)
                knob.Position = UDim2.new(percent, -5, 0.5, -5)
                valLabel.Text = tostring(value)
                if callback then callback(value) end
            end
        end)
        
        return frame
    end
    
    local function createDropdown(parent, text, var, options, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 32)
        frame.BackgroundColor3 = BLACK
        frame.BackgroundTransparency = 0.3
        frame.BorderSizePixel = 0
        frame.Parent = parent
        
        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, 6)
        c.Parent = frame
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.5, -10, 1, 0)
        label.Position = UDim2.new(0, 10, 0, 0)
        label.Text = text
        label.TextColor3 = WHITE
        label.TextSize = 11
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.BackgroundTransparency = 1
        label.Font = Enum.Font.Gotham
        label.Parent = frame
        
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.4, -10, 0, 24)
        btn.Position = UDim2.new(0.6, 0, 0.5, -12)
        btn.Text = settings[var]
        btn.TextColor3 = WHITE
        btn.BackgroundColor3 = Color3.fromRGB(30,30,40)
        btn.BorderSizePixel = 0
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
            menu.BackgroundColor3 = BLACK
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
                optBtn.TextColor3 = WHITE
                optBtn.BackgroundColor3 = Color3.fromRGB(20,20,30)
                optBtn.BorderSizePixel = 0
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
    
    -- Construir Aimbot Tab
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
    
    -- Construir ESP Tab
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
    
    -- Switch tabs
    aimbotTab.MouseButton1Click:Connect(function()
        aimbotScroll.Visible = true
        espScroll.Visible = false
        aimbotTab.BackgroundColor3 = DARK_PURPLE
        espTab.BackgroundColor3 = BLACK
    end)
    
    espTab.MouseButton1Click:Connect(function()
        aimbotScroll.Visible = false
        espScroll.Visible = true
        aimbotTab.BackgroundColor3 = BLACK
        espTab.BackgroundColor3 = DARK_PURPLE
    end)
    
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.Home then
            mainGui.Enabled = not mainGui.Enabled
        end
    end)
    
    -- Main Loop
    RunService.RenderStepped:Connect(function()
        UpdateLines()
        UpdateSkeletonESP()
        UpdateESP()
        doAimbot()
        UpdateFOVPosition()
    end)
    
    print("========================================")
    print("MAJESTIC ULTIMATE - CARREGADO!")
    print("Usuário: " .. currentUser)
    print("Press HOME to open/close")
    print("========================================")
end

local function createFreeCheatGUI()
    for _, g in pairs(CoreGui:GetChildren()) do
        if g.Name == "MajesticFree" then g:Destroy() end
    end
    
    local freeGui = Instance.new("ScreenGui")
    freeGui.Name = "MajesticFree"
    freeGui.Parent = CoreGui
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 400, 0, 300)
    mainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
    mainFrame.BackgroundColor3 = BLACK
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.BorderSizePixel = 2
    mainFrame.BorderColor3 = MEDIUM_PURPLE
    mainFrame.Parent = freeGui
    mainFrame.Active = true
    mainFrame.Draggable = true
    
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 16)
    mainCorner.Parent = mainFrame
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 0, 50)
    titleLabel.Position = UDim2.new(0, 0, 0, 20)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "MAJESTIC FREE"
    titleLabel.TextColor3 = LIGHT_PURPLE
    titleLabel.TextSize = 28
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Center
    titleLabel.Parent = mainFrame
    
    local infoLabel = Instance.new("TextLabel")
    infoLabel.Size = UDim2.new(0.9, 0, 0, 50)
    infoLabel.Position = UDim2.new(0.05, 0, 0.3, 0)
    infoLabel.BackgroundTransparency = 1
    infoLabel.Text = "Versão Gratuita\nRecursos: Aimbot básico, ESP simples"
    infoLabel.TextColor3 = WHITE
    infoLabel.TextSize = 12
    infoLabel.Font = Enum.Font.Gotham
    infoLabel.TextXAlignment = Enum.TextXAlignment.Center
    infoLabel.Parent = mainFrame
    
    local upgradeBtn = Instance.new("TextButton")
    upgradeBtn.Size = UDim2.new(0.6, 0, 0, 40)
    upgradeBtn.Position = UDim2.new(0.2, 0, 0.65, 0)
    upgradeBtn.Text = "COMPRAR VERSÃO COMPLETA"
    upgradeBtn.TextColor3 = WHITE
    upgradeBtn.BackgroundColor3 = MEDIUM_PURPLE
    upgradeBtn.BorderSizePixel = 0
    upgradeBtn.Font = Enum.Font.GothamBold
    upgradeBtn.TextSize = 12
    upgradeBtn.Parent = mainFrame
    
    local upgradeCorner = Instance.new("UICorner")
    upgradeCorner.CornerRadius = UDim.new(0, 8)
    upgradeCorner.Parent = upgradeBtn
    
    upgradeBtn.MouseButton1Click:Connect(function()
        upgradeBtn.Text = "CONTATE UM VENDEDOR"
        task.wait(2)
        upgradeBtn.Text = "COMPRAR VERSÃO COMPLETA"
    end)
    
    print("========================================")
    print("MAJESTIC FREE - CARREGADO!")
    print("Versão gratuita com recursos limitados")
    print("========================================")
end

-- ==============================================
-- INICIAR O SISTEMA
-- ==============================================
showLoader()
