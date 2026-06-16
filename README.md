--[[
    Movement GUI para "Roube um Brainrot"
    Noclip + Speed Hack - Versão Otimizada
]]

-- Aguarda o jogo carregar
repeat task.wait() until game:IsLoaded()

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Jogador Local
local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then return end

-- Variáveis
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Configurações
local Settings = {
    Noclip = false,
    SpeedBoost = false,
    SpeedValue = 16,
    DefaultSpeed = 16
}

-- Conexões
local NoclipConnection = nil

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MovementGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

-- Frame Principal
local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Parent = ScreenGui
Main.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
Main.BorderSizePixel = 0
Main.Position = UDim2.new(0.5, -140, 0.3, 0)
Main.Size = UDim2.new(0, 280, 0, 310)
Main.Active = true

-- Cantos arredondados
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = Main

-- Barra de Título
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Parent = Main
TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.Active = true

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = TitleBar

local TitlePatch = Instance.new("Frame")
TitlePatch.Parent = TitleBar
TitlePatch.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
TitlePatch.BorderSizePixel = 0
TitlePatch.Position = UDim2.new(0, 0, 0.5, 0)
TitlePatch.Size = UDim2.new(1, 0, 0.5, 0)

-- Título
local Title = Instance.new("TextLabel")
Title.Parent = TitleBar
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Size = UDim2.new(0.6, 0, 1, 0)
Title.Font = Enum.Font.GothamBold
Title.Text = "Movement GUI"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 15
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Minimizar
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.Parent = TitleBar
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(55, 55, 60)
MinimizeBtn.BorderSizePixel = 0
MinimizeBtn.Position = UDim2.new(0.75, 0, 0.5, -10)
MinimizeBtn.Size = UDim2.new(0, 22, 0, 22)
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
CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 40, 40)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(0.88, 0, 0.5, -10)
CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 12
CloseBtn.Font = Enum.Font.GothamBold

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 5)
CloseCorner.Parent = CloseBtn

-- Conteúdo Principal
local Content = Instance.new("Frame")
Content.Name = "Content"
Content.Parent = Main
Content.BackgroundTransparency = 1
Content.BorderSizePixel = 0
Content.Position = UDim2.new(0, 0, 0, 40)
Content.Size = UDim2.new(1, 0, 1, -45)

-- ===== SEÇÃO NOCLIP =====
local NoclipFrame = Instance.new("Frame")
NoclipFrame.Name = "NoclipFrame"
NoclipFrame.Parent = Content
NoclipFrame.BackgroundColor3 = Color3.fromRGB(38, 38, 43)
NoclipFrame.BorderSizePixel = 0
NoclipFrame.Position = UDim2.new(0, 10, 0, 5)
NoclipFrame.Size = UDim2.new(1, -20, 0, 85)

local NoclipCorner = Instance.new("UICorner")
NoclipCorner.CornerRadius = UDim.new(0, 6)
NoclipCorner.Parent = NoclipFrame

local NoclipHeader = Instance.new("TextLabel")
NoclipHeader.Parent = NoclipFrame
NoclipHeader.BackgroundTransparency = 1
NoclipHeader.Position = UDim2.new(0, 12, 0, 8)
NoclipHeader.Size = UDim2.new(0.7, 0, 0, 20)
NoclipHeader.Font = Enum.Font.GothamBold
NoclipHeader.Text = "🚀 Noclip"
NoclipHeader.TextColor3 = Color3.fromRGB(255, 255, 255)
NoclipHeader.TextSize = 14
NoclipHeader.TextXAlignment = Enum.TextXAlignment.Left

local NoclipStatus = Instance.new("TextLabel")
NoclipStatus.Name = "NoclipStatus"
NoclipStatus.Parent = NoclipFrame
NoclipStatus.BackgroundTransparency = 1
NoclipStatus.Position = UDim2.new(0, 12, 0, 30)
NoclipStatus.Size = UDim2.new(0.5, 0, 0, 16)
NoclipStatus.Font = Enum.Font.Gotham
NoclipStatus.Text = "Atravessar paredes"
NoclipStatus.TextColor3 = Color3.fromRGB(160, 160, 160)
NoclipStatus.TextSize = 11
NoclipStatus.TextXAlignment = Enum.TextXAlignment.Left

local NoclipBtn = Instance.new("TextButton")
NoclipBtn.Name = "NoclipBtn"
NoclipBtn.Parent = NoclipFrame
NoclipBtn.BackgroundColor3 = Color3.fromRGB(220, 40, 40)
NoclipBtn.BorderSizePixel = 0
NoclipBtn.Position = UDim2.new(0.6, 0, 0, 12)
NoclipBtn.Size = UDim2.new(0, 70, 0, 30)
NoclipBtn.Text = "OFF"
NoclipBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
NoclipBtn.TextSize = 13
NoclipBtn.Font = Enum.Font.GothamBold
NoclipBtn.AutoButtonColor = false

local NoclipBtnCorner = Instance.new("UICorner")
NoclipBtnCorner.CornerRadius = UDim.new(0, 15)
NoclipBtnCorner.Parent = NoclipBtn

local NoclipInfo = Instance.new("TextLabel")
NoclipInfo.Parent = NoclipFrame
NoclipInfo.BackgroundTransparency = 1
NoclipInfo.Position = UDim2.new(0, 12, 0, 55)
NoclipInfo.Size = UDim2.new(1, -24, 0, 18)
NoclipInfo.Font = Enum.Font.Gotham
NoclipInfo.Text = "Status: Desativado"
NoclipInfo.TextColor3 = Color3.fromRGB(255, 100, 100)
NoclipInfo.TextSize = 11
NoclipInfo.TextXAlignment = Enum.TextXAlignment.Left

-- ===== SEÇÃO SPEED =====
local SpeedFrame = Instance.new("Frame")
SpeedFrame.Name = "SpeedFrame"
SpeedFrame.Parent = Content
SpeedFrame.BackgroundColor3 = Color3.fromRGB(38, 38, 43)
SpeedFrame.BorderSizePixel = 0
SpeedFrame.Position = UDim2.new(0, 10, 0, 100)
SpeedFrame.Size = UDim2.new(1, -20, 0, 160)

local SpeedCorner = Instance.new("UICorner")
SpeedCorner.CornerRadius = UDim.new(0, 6)
SpeedCorner.Parent = SpeedFrame

local SpeedHeader = Instance.new("TextLabel")
SpeedHeader.Parent = SpeedFrame
SpeedHeader.BackgroundTransparency = 1
SpeedHeader.Position = UDim2.new(0, 12, 0, 8)
SpeedHeader.Size = UDim2.new(0.7, 0, 0, 20)
SpeedHeader.Font = Enum.Font.GothamBold
SpeedHeader.Text = "⚡ Speed Hack"
SpeedHeader.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedHeader.TextSize = 14
SpeedHeader.TextXAlignment = Enum.TextXAlignment.Left

-- Slider Background
local SliderBg = Instance.new("Frame")
SliderBg.Name = "SliderBg"
SliderBg.Parent = SpeedFrame
SliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
SliderBg.BorderSizePixel = 0
SliderBg.Position = UDim2.new(0, 12, 0, 40)
SliderBg.Size = UDim2.new(0.6, 0, 0, 22)
SliderBg.Active = true

local SliderBgCorner = Instance.new("UICorner")
SliderBgCorner.CornerRadius = UDim.new(0, 11)
SliderBgCorner.Parent = SliderBg

-- Slider Fill
local SliderFill = Instance.new("Frame")
SliderFill.Name = "SliderFill"
SliderFill.Parent = SliderBg
SliderFill.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
SliderFill.BorderSizePixel = 0
SliderFill.Size = UDim2.new(0.1, 0, 1, 0)

local SliderFillCorner = Instance.new("UICorner")
SliderFillCorner.CornerRadius = UDim.new(0, 11)
SliderFillCorner.Parent = SliderFill

-- Valor do Slider
local SpeedValue = Instance.new("TextLabel")
SpeedValue.Name = "SpeedValue"
SpeedValue.Parent = SpeedFrame
SpeedValue.BackgroundTransparency = 1
SpeedValue.Position = UDim2.new(0.68, 0, 0, 40)
SpeedValue.Size = UDim2.new(0, 45, 0, 22)
SpeedValue.Font = Enum.Font.GothamBold
SpeedValue.Text = "16"
SpeedValue.TextColor3 = Color3.fromRGB(0, 150, 255)
SpeedValue.TextSize = 15
SpeedValue.TextXAlignment = Enum.TextXAlignment.Center

-- Labels min/max
local MinSpeed = Instance.new("TextLabel")
MinSpeed.Parent = SpeedFrame
MinSpeed.BackgroundTransparency = 1
MinSpeed.Position = UDim2.new(0, 12, 0, 62)
MinSpeed.Size = UDim2.new(0, 20, 0, 12)
MinSpeed.Font = Enum.Font.Gotham
MinSpeed.Text = "1"
MinSpeed.TextColor3 = Color3.fromRGB(140, 140, 140)
MinSpeed.TextSize = 9

local MaxSpeed = Instance.new("TextLabel")
MaxSpeed.Parent = SpeedFrame
MaxSpeed.BackgroundTransparency = 1
MaxSpeed.Position = UDim2.new(0.58, 0, 0, 62)
MaxSpeed.Size = UDim2.new(0, 30, 0, 12)
MaxSpeed.Font = Enum.Font.Gotham
MaxSpeed.Text = "200"
MaxSpeed.TextColor3 = Color3.fromRGB(140, 140, 140)
MaxSpeed.TextSize = 9

-- Botão Speed
local SpeedBtn = Instance.new("TextButton")
SpeedBtn.Name = "SpeedBtn"
SpeedBtn.Parent = SpeedFrame
SpeedBtn.BackgroundColor3 = Color3.fromRGB(220, 40, 40)
SpeedBtn.BorderSizePixel = 0
SpeedBtn.Position = UDim2.new(0, 12, 0, 85)
SpeedBtn.Size = UDim2.new(0, 70, 0, 30)
SpeedBtn.Text = "OFF"
SpeedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedBtn.TextSize = 13
SpeedBtn.Font = Enum.Font.GothamBold
SpeedBtn.AutoButtonColor = false

local SpeedBtnCorner = Instance.new("UICorner")
SpeedBtnCorner.CornerRadius = UDim.new(0, 15)
SpeedBtnCorner.Parent = SpeedBtn

local SpeedBtnLabel = Instance.new("TextLabel")
SpeedBtnLabel.Parent = SpeedFrame
SpeedBtnLabel.BackgroundTransparency = 1
SpeedBtnLabel.Position = UDim2.new(0, 90, 0, 85)
SpeedBtnLabel.Size = UDim2.new(0.5, 0, 0, 30)
SpeedBtnLabel.Font = Enum.Font.Gotham
SpeedBtnLabel.Text = "Ativar Speed Boost"
SpeedBtnLabel.TextColor3 = Color3.fromRGB(160, 160, 160)
SpeedBtnLabel.TextSize = 12
SpeedBtnLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Velocidade Atual
local CurrentSpeed = Instance.new("TextLabel")
CurrentSpeed.Name = "CurrentSpeed"
CurrentSpeed.Parent = SpeedFrame
CurrentSpeed.BackgroundTransparency = 1
CurrentSpeed.Position = UDim2.new(0, 12, 0, 128)
CurrentSpeed.Size = UDim2.new(1, -24, 0, 18)
CurrentSpeed.Font = Enum.Font.GothamBold
CurrentSpeed.Text = "🏃 Velocidade: 16"
CurrentSpeed.TextColor3 = Color3.fromRGB(200, 200, 200)
CurrentSpeed.TextSize = 12
CurrentSpeed.TextXAlignment = Enum.TextXAlignment.Left

-- ===== FUNÇÕES =====
local function UpdateNoclipUI()
    if Settings.Noclip then
        NoclipBtn.BackgroundColor3 = Color3.fromRGB(40, 200, 40)
        NoclipBtn.Text = "ON"
        NoclipInfo.Text = "✅ Status: Ativado"
        NoclipInfo.TextColor3 = Color3.fromRGB(100, 255, 100)
    else
        NoclipBtn.BackgroundColor3 = Color3.fromRGB(220, 40, 40)
        NoclipBtn.Text = "OFF"
        NoclipInfo.Text = "❌ Status: Desativado"
        NoclipInfo.TextColor3 = Color3.fromRGB(255, 100, 100)
    end
end

local function UpdateSpeedUI()
    if Settings.SpeedBoost then
        SpeedBtn.BackgroundColor3 = Color3.fromRGB(40, 200, 40)
        SpeedBtn.Text = "ON"
    else
        SpeedBtn.BackgroundColor3 = Color3.fromRGB(220, 40, 40)
        SpeedBtn.Text = "OFF"
    end
end

local function UpdateSlider()
    local percent = (Settings.SpeedValue - 1) / (200 - 1)
    SliderFill.Size = UDim2.new(percent, 0, 1, 0)
    SpeedValue.Text = tostring(math.floor(Settings.SpeedValue))
end

local function ApplySpeed()
    if Character and Humanoid and Humanoid.Parent then
        if Settings.SpeedBoost then
            Humanoid.WalkSpeed = Settings.SpeedValue
            CurrentSpeed.Text = "🏃 Velocidade: " .. tostring(math.floor(Settings.SpeedValue)) .. " (Boost)"
        else
            Humanoid.WalkSpeed = Settings.DefaultSpeed
            CurrentSpeed.Text = "🏃 Velocidade: " .. tostring(Settings.DefaultSpeed)
        end
    end
end

-- Função principal do Noclip
local function ToggleNoclip()
    Settings.Noclip = not Settings.Noclip
    UpdateNoclipUI()
    
    if NoclipConnection then
        NoclipConnection:Disconnect()
        NoclipConnection = nil
    end
    
    if Settings.Noclip then
        NoclipConnection = RunService.Stepped:Connect(function()
            local char = LocalPlayer.Character
            if char and Settings.Noclip then
                for _, v in pairs(char:GetDescendants()) do
                    if v:IsA("BasePart") then
                        v.CanCollide = false
                    end
                end
            end
        end)
    else
        local char = LocalPlayer.Character
        if char then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then
                    v.CanCollide = true
                end
            end
        end
    end
end

local function ToggleSpeed()
    Settings.SpeedBoost = not Settings.SpeedBoost
    UpdateSpeedUI()
    ApplySpeed()
end

-- Eventos de clique
NoclipBtn.MouseButton1Click:Connect(ToggleNoclip)
SpeedBtn.MouseButton1Click:Connect(ToggleSpeed)

-- Eventos do Slider
local function UpdateSliderPosition(input)
    local pos = math.clamp((input.Position.X - SliderBg.AbsolutePosition.X) / SliderBg.AbsoluteSize.X, 0, 1)
    Settings.SpeedValue = 1 + (pos * (200 - 1))
    UpdateSlider()
    ApplySpeed()
end

SliderBg.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or 
       input.UserInputType == Enum.UserInputType.Touch then
        UpdateSliderPosition(input)
        
        local connection
        connection = input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.Change then
                UpdateSliderPosition(input)
            elseif input.UserInputState == Enum.UserInputState.End then
                connection:Disconnect()
            end
        end)
    end
end)

-- Arrastar janela
local dragging = false
local dragStart = Vector2.new()
local startPos = UDim2.new()

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or 
       input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = Main.Position
        
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
        Main.Position = UDim2.new(
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
        Main.Size = UDim2.new(0, 280, 0, 35)
    else
        Content.Visible = true
        Main.Size = UDim2.new(0, 280, 0, 310)
    end
end)

-- Fechar
CloseBtn.MouseButton1Click:Connect(function()
    if NoclipConnection then
        NoclipConnection:Disconnect()
    end
    
    -- Restaurar colisões
    if Character then
        for _, v in pairs(Character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = true
            end
        end
    end
    
    ScreenGui:Destroy()
end)

-- Respawn Handler
LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    Humanoid = Character:WaitForChild("Humanoid")
    RootPart = Character:WaitForChild("HumanoidRootPart")
    
    wait(0.5)
    
    -- Reaplicar efeitos
    if Settings.Noclip then
        ToggleNoclip()
        ToggleNoclip()
    end
    
    if Settings.SpeedBoost then
        ApplySpeed()
    end
end)

-- Inicialização
UpdateNoclipUI()
UpdateSpeedUI()
UpdateSlider()
CurrentSpeed.Text = "🏃 Velocidade: " .. tostring(Settings.DefaultSpeed)

-- Confirmação
print("✅ Movement GUI para 'Roube um Brainrot' carregado!")
print("📁 Localização: CoreGui")
print("🎮 Use os botões para controlar Noclip e Speed")
