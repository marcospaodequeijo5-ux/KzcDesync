local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local pgui = player:WaitForChild("PlayerGui")

-- Limpa execuções anteriores
if pgui:FindFirstChild("KZC_FinalMenu") then
    pgui.KZC_FinalMenu:Destroy()
end

-- 1. ESTRUTURA DA GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "KZC_FinalMenu"
screenGui.ResetOnSpawn = false
screenGui.Parent = pgui

-- BOLINHA DE ATALHO (KZC)
local toggleCircle = Instance.new("TextButton")
toggleCircle.Name = "KZCBall"
toggleCircle.Size = UDim2.new(0, 45, 0, 45)
-- Posicionada um pouco mais para a direita para não sobrepor o botão de Desync
toggleCircle.Position = UDim2.new(0, 160, 0, 10) 
toggleCircle.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
toggleCircle.Text = "KZC"
toggleCircle.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleCircle.Font = Enum.Font.GothamBold
toggleCircle.TextSize = 12
toggleCircle.Parent = screenGui

local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(1, 0)
circleCorner.Parent = toggleCircle

-- BOTÃO DESYNC (NA POSIÇÃO DO CÍRCULO VERMELHO DA FOTO)
local desyncBtn = Instance.new("TextButton")
desyncBtn.Name = "DesyncButton"
desyncBtn.Size = UDim2.new(0, 130, 0, 40)
-- POSIÇÃO EXATA: Canto superior esquerdo, abaixo do ícone do Roblox
desyncBtn.Position = UDim2.new(0, 15, 0, 60) 
desyncBtn.BackgroundColor3 = Color3.fromRGB(20, 25, 35)
desyncBtn.Text = "Desync"
desyncBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
desyncBtn.Font = Enum.Font.GothamBold
desyncBtn.TextSize = 16
desyncBtn.Parent = screenGui

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = desyncBtn

local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 2.5
uiStroke.Color = Color3.fromRGB(0, 255, 255)
uiStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
uiStroke.Parent = desyncBtn

-- 2. FUNÇÃO DRAGGABLE
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    desyncBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

desyncBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = desyncBtn.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

desyncBtn.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then update(input) end
end)

-- 3. CONTROLE DE VISIBILIDADE
toggleCircle.MouseButton1Click:Connect(function()
    desyncBtn.Visible = not desyncBtn.Visible
end)

-- 4. LÓGICA DO DESYNC
local FFlags = {
    GameNetPVHeaderRotationalVelocityZeroCutoffExponent = -5000,
    LargeReplicatorWrite5 = true,
    LargeReplicatorEnabled9 = true,
    AngularVelociryLimit = 360,
    TimestepArbiterVelocityCriteriaThresholdTwoDt = 2147483646,
    S2PhysicsSenderRate = 15000,
    DisableDPIScale = true,
    MaxDataPacketPerSend = 2147483647,
    PhysicsSenderMaxBandwidthBps = 20000,
    TimestepArbiterHumanoidLinearVelThreshold = 21,
    MaxMissedWorldStepsRemembered = -2147483648,
    PlayerHumanoidPropertyUpdateRestrict = true,
    SimDefaultHumanoidTimestepMultiplier = 0,
    StreamJobNOUVolumeLengthCap = 2147483647,
    DebugSendDistInSteps = -2147483648,
    GameNetDontSendRedundantNumTimes = 1,
    CheckPVLinearVelocityIntegrateVsDeltaPositionThresholdPercent = 1,
    CheckPVDifferencesForInterpolationMinVelThresholdStudsPerSecHundredth = 1,
    LargeReplicatorSerializeRead3 = true,
    ReplicationFocusNouExtentsSizeCutoffForPauseStuds = 2147483647,
    CheckPVCachedVelThresholdPercent = 10,
    CheckPVDifferencesForInterpolationMinRotVelThresholdRadsPerSecHundredth = 1,
    GameNetDontSendRedundantDeltaPositionMillionth = 1,
    InterpolationFrameVelocityThresholdMillionth = 5,
    StreamJobNOUVolumeCap = 2147483647,
    InterpolationFrameRotVelocityThresholdMillionth = 5,
    CheckPVCachedRotVelThresholdPercent = 10,
    WorldStepMax = 30,
    InterpolationFramePositionThresholdMillionth = 5,
    TimestepArbiterHumanoidTurningVelThreshold = 1,
    SimOwnedNOUCountThresholdMillionth = 2147483647,
    GameNetPVHeaderLinearVelocityZeroCutoffExponent = -5000,
    NextGenReplicatorEnabledWrite4 = true,
    TimestepArbiterOmegaThou = 1073741823,
    MaxAcceptableUpdateDelay = 1,
    LargeReplicatorSerializeWrite4 = true
}

local function respawnar(plr)
    local char = plr.Character
    if char then
        local hum = char:FindFirstChildWhichIsA('Humanoid')
        if hum then hum:ChangeState(Enum.HumanoidStateType.Dead) end
        char:ClearAllChildren()
        local newChar = Instance.new('Model', workspace)
        plr.Character = newChar
        task.wait()
        plr.Character = char
        newChar:Destroy()
    end
end

desyncBtn.MouseButton1Click:Connect(function()
    task.spawn(function()
        for name, value in pairs(FFlags) do
            pcall(function() setfflag(tostring(name), tostring(value)) end)
        end
    end)
    respawnar(player)
    
    desyncBtn.Text = "ativando..."
    uiStroke.Color = Color3.fromRGB(255, 165, 0)
    
    task.wait(4.79)
    
    desyncBtn.Text = "Desync Ativo!"
    uiStroke.Color = Color3.fromRGB(0, 255, 0)
    
    task.wait(5)
    desyncBtn.Text = "Desync"
    uiStroke.Color = Color3.fromRGB(0, 255, 255)
end)
    StreamJobNOUVolumeCap = 2147483647,
    InterpolationFrameRotVelocityThresholdMillionth = 5,
    CheckPVCachedRotVelThresholdPercent = 10,
    WorldStepMax = 30,
    InterpolationFramePositionThresholdMillionth = 5,
    TimestepArbiterHumanoidTurningVelThreshold = 1,
    SimOwnedNOUCountThresholdMillionth = 2147483647,
    GameNetPVHeaderLinearVelocityZeroCutoffExponent = -5000,
    NextGenReplicatorEnabledWrite4 = true,
    TimestepArbiterOmegaThou = 1073741823,
    MaxAcceptableUpdateDelay = 1,
    LargeReplicatorSerializeWrite4 = true
}

local function respawnar(plr)
    local char = plr.Character
    if char then
        local hum = char:FindFirstChildWhichIsA('Humanoid')
        if hum then hum:ChangeState(Enum.HumanoidStateType.Dead) end
        char:ClearAllChildren()
        local newChar = Instance.new('Model', workspace)
        plr.Character = newChar
        task.wait()
        plr.Character = char
        newChar:Destroy()
    end
end

button.MouseButton1Click:Connect(function()
    -- ATIVAÇÃO IMEDIATA (FFLAG + RESPAWN)
    task.spawn(function()
        for name, value in pairs(FFlags) do
            pcall(function() setfflag(tostring(name), tostring(value)) end)
        end
    end)
    respawnar(player)
    
    -- FEEDBACK VISUAL
    button.Text = "ativando..."
    uiStroke.Color = Color3.fromRGB(255, 165, 0)
    
    task.wait(4.79) -- Voltou para 4.79s
    
    button.Text = "Desync Ativo!"
    uiStroke.Color = Color3.fromRGB(0, 255, 0)
    
    -- RESET APÓS 5 SEGUNDOS
    task.wait(5)
    button.Text = "Desync"
    uiStroke.Color = Color3.fromRGB(0, 255, 255)
end)
Instance.new("UICorner", toggleCircle).CornerRadius = UDim.new(1, 0)

-- BOTÃO DESYNC
local button = Instance.new("TextButton")
button.Name = "DesyncButton"
button.Size = UDim2.new(0, 140, 0, 45)
button.Position = UDim2.new(0.5, -70, 0.8, 0)
button.BackgroundColor3 = Color3.fromRGB(20, 25, 35)
button.Text = "Desync"
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.Font = Enum.Font.GothamBold
button.TextSize = 18
button.Parent = screenGui

local uiCorner = Instance.new("UICorner", button)
uiCorner.CornerRadius = UDim.new(0, 8)

local uiStroke = Instance.new("UIStroke", button)
uiStroke.Thickness = 2.5
uiStroke.Color = Color3.fromRGB(0, 255, 255)
uiStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

-- 2. FUNÇÃO DRAGGABLE (MANTÉM A POSIÇÃO)
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

button.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = button.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

button.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then update(input) end
end)

-- ABRIR/FECHAR SEM RESETAR POSIÇÃO
toggleCircle.MouseButton1Click:Connect(function()
    button.Visible = not button.Visible
end)

-- 3. LOGICA DO DESYNC (CORRIGIDA)
local FFlags = {
    GameNetPVHeaderRotationalVelocityZeroCutoffExponent = -5000,
    LargeReplicatorWrite5 = true,
    LargeReplicatorEnabled9 = true,
    AngularVelociryLimit = 360,
    TimestepArbiterVelocityCriteriaThresholdTwoDt = 2147483646,
    S2PhysicsSenderRate = 15000,
    DisableDPIScale = true,
    MaxDataPacketPerSend = 2147483647,
    PhysicsSenderMaxBandwidthBps = 20000,
    TimestepArbiterHumanoidLinearVelThreshold = 21,
    MaxMissedWorldStepsRemembered = -2147483648,
    PlayerHumanoidPropertyUpdateRestrict = true,
    SimDefaultHumanoidTimestepMultiplier = 0,
    StreamJobNOUVolumeLengthCap = 2147483647,
    DebugSendDistInSteps = -2147483648,
    GameNetDontSendRedundantNumTimes = 1,
    CheckPVLinearVelocityIntegrateVsDeltaPositionThresholdPercent = 1,
    CheckPVDifferencesForInterpolationMinVelThresholdStudsPerSecHundredth = 1,
    LargeReplicatorSerializeRead3 = true,
    ReplicationFocusNouExtentsSizeCutoffForPauseStuds = 2147483647,
    CheckPVCachedVelThresholdPercent = 10,
    CheckPVDifferencesForInterpolationMinRotVelThresholdRadsPerSecHundredth = 1,
    GameNetDontSendRedundantDeltaPositionMillionth = 1,
    InterpolationFrameVelocityThresholdMillionth = 5,
    StreamJobNOUVolumeCap = 2147483647,
    InterpolationFrameRotVelocityThresholdMillionth = 5,
    CheckPVCachedRotVelThresholdPercent = 10,
    WorldStepMax = 30,
    InterpolationFramePositionThresholdMillionth = 5,
    TimestepArbiterHumanoidTurningVelThreshold = 1,
    SimOwnedNOUCountThresholdMillionth = 2147483647,
    GameNetPVHeaderLinearVelocityZeroCutoffExponent = -5000,
    NextGenReplicatorEnabledWrite4 = true,
    TimestepArbiterOmegaThou = 1073741823,
    MaxAcceptableUpdateDelay = 1,
    LargeReplicatorSerializeWrite4 = true
}

local function respawnar(plr)
    local char = plr.Character
    if char then
        local hum = char:FindFirstChildWhichIsA('Humanoid')
        if hum then hum:ChangeState(Enum.HumanoidStateType.Dead) end
        char:ClearAllChildren()
        local newChar = Instance.new('Model', workspace)
        plr.Character = newChar
        task.wait()
        plr.Character = char
        newChar:Destroy()
    end
end

button.MouseButton1Click:Connect(function()
    -- ATIVAÇÃO IMEDIATA (FFLAG + RESPAWN)
    task.spawn(function()
        for name, value in pairs(FFlags) do
            pcall(function() setfflag(tostring(name), tostring(value)) end)
        end
    end)
    respawnar(player) -- O respawn acontece agora, no momento do clique
    
    -- FEEDBACK VISUAL (CRONÔMETRO)
    button.Text = "ativando..."
    uiStroke.Color = Color3.fromRGB(255, 165, 0)
    
    task.wait(4.79) -- O tempo de espera é só para o texto mudar
    
    button.Text = "Desync Ativo!"
    uiStroke.Color = Color3.fromRGB(0, 255, 0)
    
    -- VOLTA AO NORMAL APÓS 5 SEGUNDOS
    task.wait(5)
    button.Text = "Desync"
    uiStroke.Color = Color3.fromRGB(0, 255, 255)
end)
