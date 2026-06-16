--[[
    Movement GUI - Versão Corrigida
    Script compatível com a maioria dos executores
]]

-- Aguarda o jogo carregar completamente
repeat task.wait() until game:IsLoaded()

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")

-- Variáveis do jogador
local LocalPlayer = Players.LocalPlayer

-- Verifica se o jogador existe
if not LocalPlayer then
    warn("Jogador não encontrado!")
    return
end

-- Aguarda o personagem spawnar
local Character = LocalPlayer.Character
if not Character then
    Character = LocalPlayer.CharacterAdded:Wait()
end

local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Configurações
local Config = {
    NoclipEnabled = false,
    SpeedEnabled = false,
    SpeedValue = 16,
    DefaultSpeed = 16,
    MinSpeed = 1,
    MaxSpeed = 200
}

-- Criar GUI
local function CreateGUI()
    -- Remover GUI existente
    if game.CoreGui:FindFirstChild("MovementGUI") then
        game.CoreGui:FindFirstChild("MovementGUI"):Destroy()
    end
    
    -- Criar ScreenGui
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "MovementGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Tentar diferentes parents para compatibilidade
    local success = pcall(function()
        ScreenGui.Parent = game.CoreGui
    end)
    
    if not success then
        pcall(function()
            ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
        end)
    end
    
    -- Frame Principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Parent = ScreenGui
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Position = UDim2.new(0.5, -150, 0.3, 0)
    MainFrame.Size = UDim2.new(0, 300, 0, 350)
    MainFrame.Active = true
    
    -- Borda arredondada
    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 10)
    Corner.Parent = MainFrame
    
    -- Título
    local TitleBar = Instance.new("Frame")
    TitleBar.Name = "TitleBar"
    TitleBar.Parent = MainFrame
    TitleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
    TitleBar.BorderSizePixel = 0
    TitleBar.Size = UDim2.new(1, 0, 0, 35)
    TitleBar.Active = true
    
    local TitleCorner = Instance.new("UICorner")
    TitleCorner.CornerRadius = UDim.new(0, 10)
    TitleCorner.Parent = TitleBar
    
    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Parent = TitleBar
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Position = UDim2.new(0, 15, 0, 0)
    TitleLabel.Size = UDim2.new(0.6, 0, 1, 0)
    TitleLabel.Font = Enum.Font.GothamBold
    TitleLabel.Text = "Movement GUI"
    TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    TitleLabel.TextSize = 16
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Botão Minimizar
    local MinimizeBtn = Instance.new("TextButton")
    MinimizeBtn.Name = "MinimizeBtn"
    MinimizeBtn.Parent = TitleBar
    MinimizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
    MinimizeBtn.BorderSizePixel = 0
    MinimizeBtn.Position = UDim2.new(0.75, 0, 0.5, -10)
    MinimizeBtn.Size = UDim2.new(0, 20, 0, 20)
    MinimizeBtn.Text = "—"
    MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinimizeBtn.TextSize = 14
    MinimizeBtn.Font = Enum.Font.GothamBold
    
    local MinCorner = Instance.new("UICorner")
    MinCorner.CornerRadius = UDim.new(0, 5)
    MinCorner.Parent = MinimizeBtn
    
    -- Botão Fechar
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Name = "CloseBtn"
    CloseBtn.Parent = TitleBar
    CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    CloseBtn.BorderSizePixel = 0
    CloseBtn.Position = UDim2.new(0.88, 0, 0.5, -10)
    CloseBtn.Size = UDim2.new(0, 20, 0, 20)
    CloseBtn.Text = "×"
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.TextSize = 16
    CloseBtn.Font = Enum.Font.GothamBold
    
    local CloseCorner = Instance.new("UICorner")
    CloseCorner.CornerRadius = UDim.new(0, 5)
    CloseCorner.Parent = CloseBtn
    
    -- Container de conteúdo
    local Content = Instance.new("Frame")
    Content.Name = "Content"
    Content.Parent = MainFrame
    Content.BackgroundTransparency = 1
    Content.BorderSizePixel = 0
    Content.Position = UDim2.new(0, 0, 0, 40)
    Content.Size = UDim2.new(1, 0, 1, -45)
    
    -- Seção Noclip
    local NoclipSection = Instance.new("Frame")
    NoclipSection.Name = "NoclipSection"
    NoclipSection.Parent = Content
    NoclipSection.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
    NoclipSection.BorderSizePixel = 0
    NoclipSection.Position = UDim2.new(0, 10, 0, 10)
    NoclipSection.Size = UDim2.new(1, -20, 0, 100)
    
    local NoclipCorner = Instance.new("UICorner")
    NoclipCorner.CornerRadius = UDim.new(0, 8)
    NoclipCorner.Parent = NoclipSection
    
    local NoclipTitle = Instance.new("TextLabel")
    NoclipTitle.Parent = NoclipSection
    NoclipTitle.BackgroundTransparency = 1
    NoclipTitle.Position = UDim2.new(0, 12, 0, 10)
    NoclipTitle.Size = UDim2.new(0.8, 0, 0, 22)
    NoclipTitle.Font = Enum.Font.GothamBold
    NoclipTitle.Text = "🚀 Noclip"
    NoclipTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    NoclipTitle.TextSize = 14
    NoclipTitle.TextXAlignment = Enum.TextXAlignment.Left
    
    local NoclipDesc = Instance.new("TextLabel")
    NoclipDesc.Parent = NoclipSection
    NoclipDesc.BackgroundTransparency = 1
    NoclipDesc.Position = UDim2.new(0, 12, 0, 32)
    NoclipDesc.Size = UDim2.new(0.5, 0, 0, 18)
    NoclipDesc.Font = Enum.Font.Gotham
    NoclipDesc.Text = "Atravessar paredes"
    NoclipDesc.TextColor3 = Color3.fromRGB(150, 150, 150)
    NoclipDesc.TextSize = 11
    NoclipDesc.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Toggle Noclip
    local NoclipToggle = Instance.new("TextButton")
    NoclipToggle.Name = "NoclipToggle"
    NoclipToggle.Parent = NoclipSection
    NoclipToggle.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    NoclipToggle.BorderSizePixel = 0
    NoclipToggle.Position = UDim2.new(0.55, 0, 0, 10)
    NoclipToggle.Size = UDim2.new(0, 65, 0, 28)
    NoclipToggle.Text = "OFF"
    NoclipToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    NoclipToggle.TextSize = 13
    NoclipToggle.Font = Enum.Font.GothamBold
    NoclipToggle.AutoButtonColor = false
    
    local NoclipToggleCorner = Instance.new("UICorner")
    NoclipToggleCorner.CornerRadius = UDim.new(0, 14)
    NoclipToggleCorner.Parent = NoclipToggle
    
    -- Linha separadora
    local Divider1 = Instance.new("Frame")
    Divider1.Parent = NoclipSection
    Divider1.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    Divider1.BorderSizePixel = 0
    Divider1.Position = UDim2.new(0, 12, 0, 55)
    Divider1.Size = UDim2.new(1, -24, 0, 1)
    
    -- Status Noclip
    local NoclipStatus = Instance.new("TextLabel")
    NoclipStatus.Name = "NoclipStatus"
    NoclipStatus.Parent = NoclipSection
    NoclipStatus.BackgroundTransparency = 1
    NoclipStatus.Position = UDim2.new(0, 12, 0, 65)
    NoclipStatus.Size = UDim2.new(1, -24, 0, 20)
    NoclipStatus.Font = Enum.Font.Gotham
    NoclipStatus.Text = "Status: Desativado"
    NoclipStatus.TextColor3 = Color3.fromRGB(180, 180, 180)
    NoclipStatus.TextSize = 11
    NoclipStatus.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Seção Speed
    local SpeedSection = Instance.new("Frame")
    SpeedSection.Name = "SpeedSection"
    SpeedSection.Parent = Content
    SpeedSection.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
    SpeedSection.BorderSizePixel = 0
    SpeedSection.Position = UDim2.new(0, 10, 0, 120)
    SpeedSection.Size = UDim2.new(1, -20, 0, 180)
    
    local SpeedCorner = Instance.new("UICorner")
    SpeedCorner.CornerRadius = UDim.new(0, 8)
    SpeedCorner.Parent = SpeedSection
    
    local SpeedTitle = Instance.new("TextLabel")
    SpeedTitle.Parent = SpeedSection
    SpeedTitle.BackgroundTransparency = 1
    SpeedTitle.Position = UDim2.new(0, 12, 0, 10)
    SpeedTitle.Size = UDim2.new(0.8, 0, 0, 22)
    SpeedTitle.Font = Enum.Font.GothamBold
    SpeedTitle.Text = "⚡ Speed Hack"
    SpeedTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    SpeedTitle.TextSize = 14
    SpeedTitle.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Slider de velocidade
    local SliderBg = Instance.new("Frame")
    SliderBg.Name = "SliderBg"
    SliderBg.Parent = SpeedSection
    SliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
    SliderBg.BorderSizePixel = 0
    SliderBg.Position = UDim2.new(0, 12, 0, 50)
    SliderBg.Size = UDim2.new(0.65, 0, 0, 24)
    SliderBg.Active = true
    
    local SliderBgCorner = Instance.new("UICorner")
    SliderBgCorner.CornerRadius = UDim.new(0, 12)
    SliderBgCorner.Parent = SliderBg
    
    local SliderFill = Instance.new("Frame")
    SliderFill.Name = "SliderFill"
    SliderFill.Parent = SliderBg
    SliderFill.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    SliderFill.BorderSizePixel = 0
    SliderFill.Size = UDim2.new(0.1, 0, 1, 0)
    
    local SliderFillCorner = Instance.new("UICorner")
    SliderFillCorner.CornerRadius = UDim.new(0, 12)
    SliderFillCorner.Parent = SliderFill
    
    local SliderKnob = Instance.new("Frame")
    SliderKnob.Name = "SliderKnob"
    SliderKnob.Parent = SliderFill
    SliderKnob.AnchorPoint = Vector2.new(0.5, 0.5)
    SliderKnob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    SliderKnob.BorderSizePixel = 0
    SliderKnob.Position = UDim2.new(1, 0, 0.5, 0)
    SliderKnob.Size = UDim2.new(0, 18, 0, 18)
    
    local KnobCorner = Instance.new("UICorner")
    KnobCorner.CornerRadius = UDim.new(1, 0)
    KnobCorner.Parent = SliderKnob
    
    -- Valor do slider
    local SpeedValueDisplay = Instance.new("TextLabel")
    SpeedValueDisplay.Name = "SpeedValueDisplay"
    SpeedValueDisplay.Parent = SpeedSection
    SpeedValueDisplay.BackgroundTransparency = 1
    SpeedValueDisplay.Position = UDim2.new(0.75, 0, 0, 50)
    SpeedValueDisplay.Size = UDim2.new(0, 50, 0, 24)
    SpeedValueDisplay.Font = Enum.Font.GothamBold
    SpeedValueDisplay.Text = "16"
    SpeedValueDisplay.TextColor3 = Color3.fromRGB(0, 170, 255)
    SpeedValueDisplay.TextSize = 16
    SpeedValueDisplay.TextXAlignment = Enum.TextXAlignment.Center
    
    -- Toggle Speed
    local SpeedToggle = Instance.new("TextButton")
    SpeedToggle.Name = "SpeedToggle"
    SpeedToggle.Parent = SpeedSection
    SpeedToggle.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
    SpeedToggle.BorderSizePixel = 0
    SpeedToggle.Position = UDim2.new(0, 12, 0, 90)
    SpeedToggle.Size = UDim2.new(0, 65, 0, 28)
    SpeedToggle.Text = "OFF"
    SpeedToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    SpeedToggle.TextSize = 13
    SpeedToggle.Font = Enum.Font.GothamBold
    SpeedToggle.AutoButtonColor = false
    
    local SpeedToggleCorner = Instance.new("UICorner")
    SpeedToggleCorner.CornerRadius = UDim.new(0, 14)
    SpeedToggleCorner.Parent = SpeedToggle
    
    local SpeedToggleLabel = Instance.new("TextLabel")
    SpeedToggleLabel.Parent = SpeedSection
    SpeedToggleLabel.BackgroundTransparency = 1
    SpeedToggleLabel.Position = UDim2.new(0, 85, 0, 90)
    SpeedToggleLabel.Size = UDim2.new(0.5, 0, 0, 28)
    SpeedToggleLabel.Font = Enum.Font.Gotham
    SpeedToggleLabel.Text = "Ativar Boost"
    SpeedToggleLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    SpeedToggleLabel.TextSize = 12
    SpeedToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Linha separadora
    local Divider2 = Instance.new("Frame")
    Divider2.Parent = SpeedSection
    Divider2.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    Divider2.BorderSizePixel = 0
    Divider2.Position = UDim2.new(0, 12, 0, 130)
    Divider2.Size = UDim2.new(1, -24, 0, 1)
    
    -- Velocidade atual
    local CurrentSpeedLabel = Instance.new("TextLabel")
    CurrentSpeedLabel.Name = "CurrentSpeedLabel"
    CurrentSpeedLabel.Parent = SpeedSection
    CurrentSpeedLabel.BackgroundTransparency = 1
    CurrentSpeedLabel.Position = UDim2.new(0, 12, 0, 140)
    CurrentSpeedLabel.Size = UDim2.new(1, -24, 0, 20)
    CurrentSpeedLabel.Font = Enum.Font.GothamBold
    CurrentSpeedLabel.Text = "🏃 Velocidade Atual: 16"
    CurrentSpeedLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    CurrentSpeedLabel.TextSize = 12
    CurrentSpeedLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Variáveis de arrasto
    local dragging = false
    local dragStart = nil
    local startPos = nil
    
    -- Funções
    local function UpdateNoclipUI()
        if Config.NoclipEnabled then
            NoclipToggle.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
            NoclipToggle.Text = "ON"
            NoclipStatus.Text = "✅ Status: Ativado"
            NoclipStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
        else
            NoclipToggle.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
            NoclipToggle.Text = "OFF"
            NoclipStatus.Text = "❌ Status: Desativado"
            NoclipStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
        end
    end
    
    local function UpdateSpeedUI()
        if Config.SpeedEnabled then
            SpeedToggle.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
            SpeedToggle.Text = "ON"
        else
            SpeedToggle.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
            SpeedToggle.Text = "OFF"
        end
    end
    
    local function UpdateSlider()
        local percentage = (Config.SpeedValue - Config.MinSpeed) / (Config.MaxSpeed - Config.MinSpeed)
        SliderFill.Size = UDim2.new(percentage, 0, 1, 0)
        SpeedValueDisplay.Text = tostring(math.floor(Config.SpeedValue))
    end
    
    local function ApplySpeed()
        if Character and Humanoid and Humanoid.Parent then
            if Config.SpeedEnabled then
                Humanoid.WalkSpeed = Config.SpeedValue
                CurrentSpeedLabel.Text = "🏃 Velocidade Atual: " .. tostring(math.floor(Config.SpeedValue)) .. " (Boost)"
            else
                Humanoid.WalkSpeed = Config.DefaultSpeed
                CurrentSpeedLabel.Text = "🏃 Velocidade Atual: " .. tostring(Config.DefaultSpeed)
            end
        end
    end
    
    local noclipConnection = nil
    
    local function ToggleNoclip()
        Config.NoclipEnabled = not Config.NoclipEnabled
        UpdateNoclipUI()
        
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        
        if Config.NoclipEnabled then
            noclipConnection = RunService.Stepped:Connect(function()
                if Config.NoclipEnabled then
                    local currentChar = LocalPlayer.Character
                    if currentChar then
                        for _, part in ipairs(currentChar:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.CanCollide = false
                            end
                        end
                    end
                end
            end)
        else
            local currentChar = LocalPlayer.Character
            if currentChar then
                for _, part in ipairs(currentChar:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end
            end
        end
    end
    
    local function ToggleSpeed()
        Config.SpeedEnabled = not Config.SpeedEnabled
        UpdateSpeedUI()
        ApplySpeed()
    end
    
    -- Eventos dos botões
    NoclipToggle.MouseButton1Click:Connect(ToggleNoclip)
    SpeedToggle.MouseButton1Click:Connect(ToggleSpeed)
    
    -- Eventos do slider
    local function UpdateSliderFromInput(input)
        local relativePos = math.clamp((input.Position.X - SliderBg.AbsolutePosition.X) / SliderBg.AbsoluteSize.X, 0, 1)
        Config.SpeedValue = Config.MinSpeed + (relativePos * (Config.MaxSpeed - Config.MinSpeed))
        UpdateSlider()
        ApplySpeed()
    end
    
    SliderBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or 
           input.UserInputType == Enum.UserInputType.Touch then
            UpdateSliderFromInput(input)
            
            local connection
            connection = input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.Change then
                    UpdateSliderFromInput(input)
                elseif input.UserInputState == Enum.UserInputState.End then
                    connection:Disconnect()
                end
            end)
        end
    end)
    
    -- Drag da janela
    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or 
           input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
            
            local connection
            connection = input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                    connection:Disconnect()
                end
            end)
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or 
                        input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
    
    -- Minimizar
    local minimized = false
    MinimizeBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            Content.Visible = false
            MainFrame.Size = UDim2.new(0, 300, 0, 35)
        else
            Content.Visible = true
            MainFrame.Size = UDim2.new(0, 300, 0, 350)
        end
    end)
    
    -- Fechar
    CloseBtn.MouseButton1Click:Connect(function()
        if noclipConnection then
            noclipConnection:Disconnect()
        end
        ScreenGui:Destroy()
    end)
    
    -- Inicializar UI
    UpdateNoclipUI()
    UpdateSpeedUI()
    UpdateSlider()
    CurrentSpeedLabel.Text = "🏃 Velocidade Atual: " .. tostring(Config.DefaultSpeed)
    
    return ScreenGui
end

-- Criar a GUI
local GUI = CreateGUI()

-- Reconectar em caso de respawn
LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    Character = newCharacter
    Humanoid = Character:WaitForChild("Humanoid")
    RootPart = Character:WaitForChild("HumanoidRootPart")
    
    -- Reaplicar efeitos após um pequeno delay
    task.wait(0.5)
    
    if Config.NoclipEnabled then
        -- Reativar noclip
        Config.NoclipEnabled = false
        local noclipToggle = GUI:FindFirstChild("MainFrame"):FindFirstChild("Content"):FindFirstChild("NoclipSection"):FindFirstChild("NoclipToggle")
        if noclipToggle then
            pcall(function()
                local oldConnection = getconnections(noclipToggle.MouseButton1Click)[1]
                if oldConnection then
                    oldConnection.Function()
                end
            end)
        end
    end
    
    if Config.SpeedEnabled then
        -- Reaplicar velocidade
        task.wait(0.5)
        if Humanoid and Humanoid.Parent then
            Humanoid.WalkSpeed = Config.SpeedValue
        end
    end
end)

-- Tratamento de erros
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "Movement GUI",
        Text = "Script carregado com sucesso!",
        Duration = 3,
    })
end)

print("Movement GUI carregado com sucesso!")
print("Pressione F9 para abrir o console e verificar erros")
