--[[
    Ale - VV Ultimatum Script
    Versão Estável e Otimizada
]]

-- Proteção contra crash
local success, errorMsg = pcall(function()

-- Aguardar o jogo carregar completamente
wait(3)

-- Serviços
local Services = {
    Players = game:GetService("Players"),
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    TweenService = game:GetService("TweenService"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    Workspace = game:GetService("Workspace")
}

local Player = Services.Players.LocalPlayer
if not Player then
    warn("Jogador não encontrado!")
    return
end

local PlayerGui = Player:WaitForChild("PlayerGui", 10)
if not PlayerGui then
    warn("PlayerGui não encontrado!")
    return
end

-- Verificar se já existe uma UI
if PlayerGui:FindFirstChild("AleUI") then
    PlayerGui:FindFirstChild("AleUI"):Destroy()
    wait(0.5)
end

-- Configurações
local Config = {
    AutoParry = {
        Enabled = false,
        Range = 20,
        Cooldown = 0.2,
        LastParry = 0
    },
    HitboxESP = {
        Enabled = false,
        Color = Color3.fromRGB(255, 60, 60),
        Transparency = 0.5
    },
    BossFarm = {
        Enabled = false,
        TargetBoss = nil,
        Position = "Frente",
        Distance = 15,
        AutoAttack = false
    },
    Minimized = false
}

-- Funções de segurança
local function safeFind(instance, name)
    local success, result = pcall(function()
        return instance:FindFirstChild(name)
    end)
    return success and result or nil
end

local function safeGetChildren(instance)
    local success, result = pcall(function()
        return instance:GetChildren()
    end)
    return success and result or {}
end

local function safeGetDescendants(instance)
    local success, result = pcall(function()
        return instance:GetDescendants()
    end)
    return success and result or {}
end

-- Função segura para encontrar mobs
local function GetMobs()
    local mobs = {}
    
    pcall(function()
        local descendants = safeGetDescendants(Services.Workspace)
        
        for _, obj in ipairs(descendants) do
            if obj:IsA("Model") then
                local humanoid = safeFind(obj, "Humanoid")
                local rootPart = safeFind(obj, "HumanoidRootPart")
                
                if humanoid and rootPart and humanoid.Health > 0 then
                    local isPlayer = false
                    pcall(function()
                        isPlayer = Services.Players:GetPlayerFromCharacter(obj) ~= nil
                    end)
                    
                    if not isPlayer then
                        local name = obj.Name:lower()
                        local isMob = name:find("hollow") or 
                                     name:find("menos") or 
                                     name:find("arrancar") or
                                     name:find("vasto") or
                                     name:find("monster") or
                                     name:find("mob") or
                                     name:find("enemy")
                        
                        if isMob then
                            table.insert(mobs, {
                                Model = obj,
                                Humanoid = humanoid,
                                RootPart = rootPart,
                                Name = obj.Name,
                                Position = rootPart.Position
                            })
                        end
                    end
                end
            end
        end
    end)
    
    return mobs
end

-- Função segura para encontrar bosses
local function GetBosses()
    local bosses = {}
    
    pcall(function()
        local descendants = safeGetDescendants(Services.Workspace)
        
        for _, obj in ipairs(descendants) do
            if obj:IsA("Model") then
                local humanoid = safeFind(obj, "Humanoid")
                local rootPart = safeFind(obj, "HumanoidRootPart")
                
                if humanoid and rootPart and humanoid.Health > 0 then
                    local name = obj.Name:lower()
                    local isBoss = name:find("giant") and name:find("dragonfly") or
                                  name:find("boss") or
                                  (name:find("vasto") and name:find("lorde"))
                    
                    if isBoss then
                        table.insert(bosses, {
                            Model = obj,
                            Humanoid = humanoid,
                            RootPart = rootPart,
                            Name = obj.Name,
                            Health = math.floor(humanoid.Health),
                            MaxHealth = math.floor(humanoid.MaxHealth)
                        })
                    end
                end
            end
        end
    end)
    
    return bosses
end

-- Auto Parry seguro
local function AutoParry()
    if not Config.AutoParry.Enabled then return end
    
    pcall(function()
        local character = Player.Character
        if not character then return end
        
        local now = tick()
        if now - Config.AutoParry.LastParry < Config.AutoParry.Cooldown then return end
        
        local mobs = GetMobs()
        local charPos = character:GetPivot().Position
        
        for _, mob in ipairs(mobs) do
            if (mob.Position - charPos).Magnitude <= Config.AutoParry.Range then
                local animator = safeFind(mob.Humanoid, "Animator")
                if animator then
                    local tracks = safeGetChildren(animator)
                    for _, track in ipairs(tracks) do
                        if track:IsA("AnimationTrack") and track.IsPlaying then
                            local animId = track.Animation.AnimationId:lower()
                            if animId:find("attack") or animId:find("m1") or animId:find("hit") then
                                Config.AutoParry.LastParry = now
                                pcall(function()
                                    local remotes = safeFind(Services.ReplicatedStorage, "Remotes")
                                    if remotes then
                                        local combat = safeFind(remotes, "Combat") or safeFind(remotes, "Parry")
                                        if combat then
                                            combat:FireServer("Parry", true)
                                        end
                                    end
                                end)
                                break
                            end
                        end
                    end
                end
            end
        end
    end)
end

-- Hitbox ESP seguro
local function UpdateHitboxESP()
    pcall(function()
        -- Limpar todas as hitboxes
        for _, obj in ipairs(safeGetDescendants(Services.Workspace)) do
            if obj.Name == "Ale_ESP" then
                pcall(function() obj:Destroy() end)
            end
        end
        
        if not Config.HitboxESP.Enabled then return end
        
        local mobs = GetMobs()
        
        for _, mob in ipairs(mobs) do
            pcall(function()
                local hitbox = Instance.new("Part")
                hitbox.Name = "Ale_ESP"
                hitbox.Size = Vector3.new(4, 5, 4)
                hitbox.Color = Config.HitboxESP.Color
                hitbox.Transparency = Config.HitboxESP.Transparency
                hitbox.Material = Enum.Material.Neon
                hitbox.Anchored = true
                hitbox.CanCollide = false
                hitbox.CanQuery = false
                hitbox.CanTouch = false
                hitbox.Position = mob.RootPart.Position
                hitbox.Parent = Services.Workspace
                
                -- Auto-destruir após 0.5 segundos (será recriado no próximo frame)
                game:GetService("Debris"):AddItem(hitbox, 0.5)
            end)
        end
    end)
end

-- Auto Farm seguro
local function AutoFarmBoss()
    if not Config.BossFarm.Enabled then return end
    if not Config.BossFarm.TargetBoss then return end
    
    pcall(function()
        local character = Player.Character
        if not character then return end
        
        local humanoidRootPart = safeFind(character, "HumanoidRootPart")
        if not humanoidRootPart then return end
        
        local boss = Config.BossFarm.TargetBoss
        local bossModel = boss.Model
        local bossRootPart = safeFind(bossModel, "HumanoidRootPart")
        
        if not bossRootPart then return end
        
        local bossPos = bossRootPart.Position
        local targetPos = bossPos
        
        if Config.BossFarm.Position == "Frente" then
            targetPos = bossPos + (bossRootPart.CFrame.LookVector * Config.BossFarm.Distance)
        elseif Config.BossFarm.Position == "Atrás" then
            targetPos = bossPos + (-bossRootPart.CFrame.LookVector * Config.BossFarm.Distance)
        elseif Config.BossFarm.Position == "Acima" then
            targetPos = bossPos + Vector3.new(0, Config.BossFarm.Distance, 0)
        elseif Config.BossFarm.Position == "Abaixo" then
            targetPos = bossPos + Vector3.new(0, -Config.BossFarm.Distance, 0)
        end
        
        humanoidRootPart.CFrame = CFrame.new(targetPos)
        
        -- Auto ataque
        if Config.BossFarm.AutoAttack then
            local bossHumanoid = safeFind(bossModel, "Humanoid")
            if bossHumanoid and bossHumanoid.Health > 0 then
                pcall(function()
                    local remotes = safeFind(Services.ReplicatedStorage, "Remotes")
                    if remotes then
                        local combat = safeFind(remotes, "Combat") or safeFind(remotes, "Attack")
                        if combat then
                            combat:FireServer("M1")
                        end
                    end
                end)
            end
        end
    end)
end

-- ===================== INTERFACE =====================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AleUI"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 380, 0, 520)
MainFrame.Position = UDim2.new(0.5, -190, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = TitleBar

local TitlePatch = Instance.new("Frame")
TitlePatch.Size = UDim2.new(1, 0, 0, 8)
TitlePatch.Position = UDim2.new(0, 0, 1, -8)
TitlePatch.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
TitlePatch.BorderSizePixel = 0
TitlePatch.Parent = TitleBar

-- Título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -80, 1, 0)
TitleLabel.Position = UDim2.new(0, 12, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "Ale - VV Ultimatum"
TitleLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
TitleLabel.TextSize = 14
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Botão Minimizar
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0, 28, 0, 28)
MinimizeButton.Position = UDim2.new(1, -65, 0, 4)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
MinimizeButton.Text = "−"
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.TextSize = 20
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.BorderSizePixel = 0
MinimizeButton.AutoButtonColor = false
MinimizeButton.Parent = TitleBar

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 14)
MinCorner.Parent = MinimizeButton

-- Botão Fechar
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 28, 0, 28)
CloseButton.Position = UDim2.new(1, -33, 0, 4)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
CloseButton.Text = "✕"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 14
CloseButton.Font = Enum.Font.GothamBold
CloseButton.BorderSizePixel = 0
CloseButton.AutoButtonColor = false
CloseButton.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 14)
CloseCorner.Parent = CloseButton

-- Container com scroll
local ScrollingFrame = Instance.new("ScrollingFrame")
ScrollingFrame.Size = UDim2.new(1, 0, 1, -40)
ScrollingFrame.Position = UDim2.new(0, 0, 0, 40)
ScrollingFrame.BackgroundTransparency = 1
ScrollingFrame.BorderSizePixel = 0
ScrollingFrame.ScrollBarThickness = 3
ScrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 800)
ScrollingFrame.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 8)
UIListLayout.Parent = ScrollingFrame

-- Funções da UI
local function CreateSection(title, height)
    local Section = Instance.new("Frame")
    Section.Size = UDim2.new(1, -16, 0, height)
    Section.Position = UDim2.new(0, 8, 0, 0)
    Section.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    Section.BorderSizePixel = 0
    Section.Parent = ScrollingFrame
    
    local SectionCorner = Instance.new("UICorner")
    SectionCorner.CornerRadius = UDim.new(0, 6)
    SectionCorner.Parent = Section
    
    local SectionTitle = Instance.new("TextLabel")
    SectionTitle.Size = UDim2.new(1, -16, 0, 25)
    SectionTitle.Position = UDim2.new(0, 8, 0, 5)
    SectionTitle.BackgroundTransparency = 1
    SectionTitle.Text = title
    SectionTitle.TextColor3 = Color3.fromRGB(0, 200, 255)
    SectionTitle.TextSize = 13
    SectionTitle.Font = Enum.Font.GothamBold
    SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
    SectionTitle.Parent = Section
    
    return Section
end

local function CreateToggle(parent, text, yPos, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Size = UDim2.new(1, -16, 0, 32)
    ToggleFrame.Position = UDim2.new(0, 8, 0, yPos)
    ToggleFrame.BackgroundTransparency = 1
    ToggleFrame.Parent = parent
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.65, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(255, 255, 255)
    Label.TextSize = 12
    Label.Font = Enum.Font.Gotham
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = ToggleFrame
    
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0, 60, 0, 26)
    Button.Position = UDim2.new(1, -60, 0.5, -13)
    Button.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
    Button.Text = "OFF"
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextSize = 10
    Button.Font = Enum.Font.GothamBold
    Button.BorderSizePixel = 0
    Button.AutoButtonColor = false
    Button.Parent = ToggleFrame
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 13)
    ButtonCorner.Parent = Button
    
    local enabled = false
    
    Button.MouseButton1Click:Connect(function()
        enabled = not enabled
        if enabled then
            Button.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
            Button.Text = "ON"
        else
            Button.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
            Button.Text = "OFF"
        end
        callback(enabled)
    end)
    
    return Button
end

local function CreateButton(parent, text, yPos, callback)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(1, -16, 0, 32)
    Button.Position = UDim2.new(0, 8, 0, yPos)
    Button.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    Button.Text = text
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextSize = 11
    Button.Font = Enum.Font.Gotham
    Button.BorderSizePixel = 0
    Button.AutoButtonColor = false
    Button.Parent = parent
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 5)
    ButtonCorner.Parent = Button
    
    Button.MouseEnter:Connect(function()
        Button.BackgroundColor3 = Color3.fromRGB(70, 70, 85)
    end)
    
    Button.MouseLeave:Connect(function()
        Button.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    end)
    
    Button.MouseButton1Click:Connect(callback)
    
    return Button
end

local function CreateDropdown(parent, options, yPos, callback)
    local DropdownFrame = Instance.new("Frame")
    DropdownFrame.Size = UDim2.new(1, -16, 0, 32)
    DropdownFrame.Position = UDim2.new(0, 8, 0, yPos)
    DropdownFrame.BackgroundTransparency = 1
    DropdownFrame.Parent = parent
    
    local MainButton = Instance.new("TextButton")
    MainButton.Size = UDim2.new(1, 0, 1, 0)
    MainButton.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    MainButton.Text = options[1] or "Selecionar"
    MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MainButton.TextSize = 11
    MainButton.Font = Enum.Font.Gotham
    MainButton.BorderSizePixel = 0
    MainButton.AutoButtonColor = false
    MainButton.Parent = DropdownFrame
    
    local MainCorner = Instance.new("UICorner")
    MainCorner.CornerRadius = UDim.new(0, 5)
    MainCorner.Parent = MainButton
    
    local DropList = Instance.new("Frame")
    DropList.Size = UDim2.new(1, 0, 0, #options * 32)
    DropList.Position = UDim2.new(0, 0, 1, 3)
    DropList.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    DropList.BorderSizePixel = 0
    DropList.Visible = false
    DropList.ZIndex = 10
    DropList.Parent = DropdownFrame
    
    local DropCorner = Instance.new("UICorner")
    DropCorner.CornerRadius = UDim.new(0, 5)
    DropCorner.Parent = DropList
    
    for i, option in ipairs(options) do
        local OptionButton = Instance.new("TextButton")
        OptionButton.Size = UDim2.new(1, 0, 0, 32)
        OptionButton.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
        OptionButton.Text = option
        OptionButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        OptionButton.TextSize = 11
        OptionButton.Font = Enum.Font.Gotham
        OptionButton.BorderSizePixel = 0
        OptionButton.ZIndex = 11
        OptionButton.Parent = DropList
        
        OptionButton.MouseButton1Click:Connect(function()
            MainButton.Text = option
            DropList.Visible = false
            callback(option)
        end)
    end
    
    MainButton.MouseButton1Click:Connect(function()
        DropList.Visible = not DropList.Visible
    end)
    
    return MainButton
end

-- ===================== SEÇÕES =====================

-- Auto Parry
local ParrySection = CreateSection("🛡️ Auto Parry", 75)
CreateToggle(ParrySection, "Defesa Automática", 38, function(enabled)
    Config.AutoParry.Enabled = enabled
end)

-- ESP Hitboxes
local ESPSection = CreateSection("👁️ ESP Hitboxes", 75)
CreateToggle(ESPSection, "Hitboxes dos Mobs", 38, function(enabled)
    Config.HitboxESP.Enabled = enabled
end)

-- Bosses
local BossSection = CreateSection("👑 Bosses", 220)

local BossListFrame = Instance.new("ScrollingFrame")
BossListFrame.Size = UDim2.new(1, -16, 0, 100)
BossListFrame.Position = UDim2.new(0, 8, 0, 38)
BossListFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
BossListFrame.BorderSizePixel = 0
BossListFrame.ScrollBarThickness = 2
BossListFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
BossListFrame.Parent = BossSection

local BossListCorner = Instance.new("UICorner")
BossListCorner.CornerRadius = UDim.new(0, 4)
BossListCorner.Parent = BossListFrame

local BossListLayout = Instance.new("UIListLayout")
BossListLayout.Padding = UDim.new(0, 2)
BossListLayout.Parent = BossListFrame

local SelectedBossText = Instance.new("TextLabel")
SelectedBossText.Size = UDim2.new(1, -16, 0, 20)
SelectedBossText.Position = UDim2.new(0, 8, 0, 145)
SelectedBossText.BackgroundTransparency = 1
SelectedBossText.Text = "🎯 Nenhum boss selecionado"
SelectedBossText.TextColor3 = Color3.fromRGB(255, 200, 0)
SelectedBossText.TextSize = 10
SelectedBossText.Font = Enum.Font.Gotham
SelectedBossText.TextXAlignment = Enum.TextXAlignment.Left
SelectedBossText.Parent = BossSection

CreateButton(BossSection, "🔄 Atualizar Lista", 170, function()
    UpdateBossList()
end)

-- Auto Farm
local FarmSection = CreateSection("⚔️ Auto Farm Boss", 140)

CreateToggle(FarmSection, "Auto Farm Ativo", 38, function(enabled)
    Config.BossFarm.Enabled = enabled
end)

CreateToggle(FarmSection, "Auto Atacar", 75, function(enabled)
    Config.BossFarm.AutoAttack = enabled
end)

CreateDropdown(FarmSection, {"Frente", "Atrás", "Acima", "Abaixo"}, 112, function(selected)
    Config.BossFarm.Position = selected
end)

-- Função para atualizar lista de bosses
function UpdateBossList()
    for _, child in ipairs(safeGetChildren(BossListFrame)) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    local bosses = GetBosses()
    
    if #bosses == 0 then
        local NoBossLabel = Instance.new("TextLabel")
        NoBossLabel.Size = UDim2.new(1, 0, 0, 25)
        NoBossLabel.BackgroundTransparency = 1
        NoBossLabel.Text = "Nenhum boss encontrado"
        NoBossLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
        NoBossLabel.TextSize = 11
        NoBossLabel.Font = Enum.Font.Gotham
        NoBossLabel.Parent = BossListFrame
    else
        for _, boss in ipairs(bosses) do
            local BossButton = Instance.new("TextButton")
            BossButton.Size = UDim2.new(1, -4, 0, 26)
            BossButton.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
            BossButton.Text = string.format("%s [%d/%d]", boss.Name, boss.Health, boss.MaxHealth)
            BossButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            BossButton.TextSize = 9
            BossButton.Font = Enum.Font.Gotham
            BossButton.BorderSizePixel = 0
            BossButton.AutoButtonColor = false
            BossButton.Parent = BossListFrame
            
            BossButton.MouseButton1Click:Connect(function()
                Config.BossFarm.TargetBoss = boss
                SelectedBossText.Text = "🎯 " .. boss.Name
                
                for _, btn in ipairs(safeGetChildren(BossListFrame)) do
                    if btn:IsA("TextButton") then
                        btn.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
                    end
                end
                BossButton.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
            end)
        end
    end
    
    BossListFrame.CanvasSize = UDim2.new(0, 0, 0, #bosses * 28)
end

-- Sistema de arraste
local dragging = false
local dragStart = nil
local startPos = nil

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)

Services.UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

Services.UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Botão Minimizar
MinimizeButton.MouseButton1Click:Connect(function()
    Config.Minimized = true
    MainFrame.Visible = false
end)

-- Botão Fechar
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Restaurar com Shift Esquerdo
Services.UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.LeftShift and Config.Minimized then
        Config.Minimized = false
        MainFrame.Visible = true
    end
end)

-- Loop principal seguro
local function MainLoop()
    pcall(function()
        AutoParry()
    end)
    
    pcall(function()
        UpdateHitboxESP()
    end)
    
    pcall(function()
        AutoFarmBoss()
    end)
end

-- Iniciar loop
local heartbeatConnection
heartbeatConnection = Services.RunService.Heartbeat:Connect(function()
    pcall(MainLoop)
end)

-- Atualização periódica da lista de bosses
spawn(function()
    while wait(2) do
        pcall(UpdateBossList)
    end
end)

-- Reconectar personagem
Player.CharacterAdded:Connect(function()
    wait(2)
end)

print("✅ Ale Script carregado!")
print("📌 Shift Esquerdo para mostrar/esconder")
print("🎯 Procure pelo Giant Dragonfly na lista!")

end)

-- Tratamento de erro final
if not success then
    warn("❌ Erro: " .. tostring(errorMsg))
    wait(2)
end
