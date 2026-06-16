--[[
    VV Ultimatum - Interface Avançada v3.0
    Script otimizado e funcional
    Foco: Giant Dragonfly, Auto Farm, Hitbox de Mobs
]]

local Services = {
    Players = game:GetService("Players"),
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    TweenService = game:GetService("TweenService"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    Workspace = game:GetService("Workspace"),
    HttpService = game:GetService("HttpService")
}

local Player = Services.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Configurações otimizadas
local Config = {
    AutoParry = {
        Enabled = false,
        Range = 25,
        Cooldown = 0.15,
        LastParry = 0
    },
    HitboxESP = {
        Enabled = false,
        Color = Color3.fromRGB(255, 50, 50),
        Transparency = 0.6,
        Size = Vector3.new(4, 5, 4),
        OnlyMobs = true -- Apenas monstros/hollows
    },
    BossFarm = {
        Enabled = false,
        TargetBoss = nil,
        BossName = "Giant Dragonfly",
        Position = "Frente",
        Distance = 12,
        AutoAttack = false,
        AttackRange = 15
    },
    Combat = {
        Positions = {"Frente", "Atrás", "Acima", "Abaixo"},
        CurrentPosition = "Frente"
    }
}

-- Cache de objetos para performance
local Cache = {
    Hitboxes = {},
    Bosses = {},
    LastUpdate = 0,
    UpdateInterval = 0.5
}

-- Função otimizada para encontrar mobs (apenas monstros/hollows)
local function GetMobs()
    local mobs = {}
    local now = tick()
    
    -- Cache para evitar processamento excessivo
    if now - Cache.LastUpdate < Cache.UpdateInterval then
        return Cache.CachedMobs or {}
    end
    
    for _, obj in ipairs(Services.Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
            local humanoid = obj.Humanoid
            local name = obj.Name:lower()
            
            -- Filtrar apenas mobs (não players, não NPCs amigáveis)
            if humanoid.Health > 0 and not Services.Players:GetPlayerFromCharacter(obj) then
                local isMob = false
                
                -- Verificar se é um mob/hollow
                if name:find("hollow") or 
                   name:find("menos") or 
                   name:find("arrancar") or
                   name:find("vasto") or
                   name:find("monster") or
                   name:find("mob") or
                   name:find("enemy") or
                   obj:GetAttribute("IsMob") or
                   obj:GetAttribute("IsEnemy") then
                    isMob = true
                end
                
                -- Verificar por tags de NPC inimigo
                local humanoidRootPart = obj.HumanoidRootPart
                if humanoidRootPart:GetAttribute("IsEnemy") or 
                   humanoidRootPart:GetAttribute("IsMob") then
                    isMob = true
                end
                
                if isMob then
                    table.insert(mobs, {
                        Model = obj,
                        Humanoid = humanoid,
                        RootPart = humanoidRootPart,
                        Name = obj.Name,
                        Health = humanoid.Health,
                        MaxHealth = humanoid.MaxHealth,
                        Position = humanoidRootPart.Position
                    })
                end
            end
        end
    end
    
    Cache.CachedMobs = mobs
    Cache.LastUpdate = now
    return mobs
end

-- Função otimizada para encontrar bosses específicos
local function GetBosses()
    local bosses = {}
    
    for _, obj in ipairs(Services.Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
            local humanoid = obj.Humanoid
            local name = obj.Name
            
            -- Verificar se é um boss
            local isBoss = false
            
            -- Boss específico: Giant Dragonfly
            if name:lower():find("giant") and name:lower():find("dragonfly") then
                isBoss = true
            end
            
            -- Outros bosses
            if name:lower():find("boss") or
               name:lower():find("vasto") and name:lower():find("lorde") or
               obj:GetAttribute("IsBoss") or
               obj:FindFirstChild("Boss") then
                isBoss = true
            end
            
            if isBoss and humanoid.Health > 0 then
                table.insert(bosses, {
                    Model = obj,
                    Humanoid = humanoid,
                    RootPart = obj.HumanoidRootPart,
                    Name = name,
                    Health = humanoid.Health,
                    MaxHealth = humanoid.MaxHealth,
                    Position = obj.HumanoidRootPart.Position
                })
            end
        end
    end
    
    return bosses
end

-- Sistema de Auto Parry otimizado
local function AutoParry()
    if not Config.AutoParry.Enabled then return end
    
    local character = Player.Character
    if not character then return end
    
    local now = tick()
    if now - Config.AutoParry.LastParry < Config.AutoParry.Cooldown then return end
    
    local mobs = GetMobs()
    local characterPosition = character:GetPivot().Position
    
    for _, mob in ipairs(mobs) do
        local distance = (mob.Position - characterPosition).Magnitude
        if distance <= Config.AutoParry.Range then
            -- Verificar se o mob está atacando
            local animator = mob.Humanoid:FindFirstChild("Animator")
            if animator then
                for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                    local animId = track.Animation.AnimationId:lower()
                    if animId:find("attack") or animId:find("m1") or animId:find("hit") or animId:find("punch") then
                        -- Executar parry
                        pcall(function()
                            local remotes = Services.ReplicatedStorage:FindFirstChild("Remotes")
                            if remotes then
                                local combatRemote = remotes:FindFirstChild("Combat") or remotes:FindFirstChild("Parry")
                                if combatRemote then
                                    combatRemote:FireServer("Parry", true)
                                    Config.AutoParry.LastParry = now
                                    print("[Auto Parry] ✓ Defendeu ataque de " .. mob.Name)
                                    return
                                end
                            end
                        end)
                    end
                end
            end
        end
    end
end

-- Sistema de ESP para Hitboxes de mobs
local function UpdateHitboxESP()
    -- Limpar hitboxes se desativado
    if not Config.HitboxESP.Enabled then
        for _, hitbox in ipairs(Cache.Hitboxes) do
            if hitbox and hitbox.Parent then
                hitbox:Destroy()
            end
        end
        Cache.Hitboxes = {}
        return
    end
    
    local mobs = GetMobs()
    local activeHitboxes = {}
    
    for _, mob in ipairs(mobs) do
        local hitbox = mob.Model:FindFirstChild("ESP_Hitbox")
        
        if not hitbox then
            -- Criar hitbox visual
            hitbox = Instance.new("Part")
            hitbox.Name = "ESP_Hitbox"
            hitbox.Size = Config.HitboxESP.Size
            hitbox.Color = Config.HitboxESP.Color
            hitbox.Transparency = Config.HitboxESP.Transparency
            hitbox.Material = Enum.Material.ForceField
            hitbox.Anchored = true
            hitbox.CanCollide = false
            hitbox.CanQuery = false
            hitbox.CanTouch = false
            hitbox.Parent = mob.Model
            
            -- Adicionar highlight
            local highlight = Instance.new("Highlight")
            highlight.FillColor = Config.HitboxESP.Color
            highlight.FillTransparency = Config.HitboxESP.Transparency
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.OutlineTransparency = 0.3
            highlight.Parent = hitbox
        end
        
        -- Atualizar posição
        hitbox.Position = mob.RootPart.Position
        table.insert(activeHitboxes, hitbox)
    end
    
    -- Remover hitboxes de mobs mortos
    for _, hitbox in ipairs(Cache.Hitboxes) do
        if hitbox and hitbox.Parent and not table.find(activeHitboxes, hitbox) then
            hitbox:Destroy()
        end
    end
    
    Cache.Hitboxes = activeHitboxes
end

-- Sistema de Auto Farm do Boss
local function AutoFarmBoss()
    if not Config.BossFarm.Enabled then return end
    if not Config.BossFarm.TargetBoss then return end
    
    local character = Player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local boss = Config.BossFarm.TargetBoss
    if not boss.Model:FindFirstChild("HumanoidRootPart") then return end
    
    local humanoidRootPart = character.HumanoidRootPart
    local bossRootPart = boss.Model.HumanoidRootPart
    local bossPosition = bossRootPart.Position
    
    -- Calcular posição ideal baseada na configuração
    local targetPosition = bossPosition
    local distance = Config.BossFarm.Distance
    
    if Config.BossFarm.Position == "Frente" then
        targetPosition = bossPosition + (bossRootPart.CFrame.LookVector * distance)
    elseif Config.BossFarm.Position == "Atrás" then
        targetPosition = bossPosition + (-bossRootPart.CFrame.LookVector * distance)
    elseif Config.BossFarm.Position == "Acima" then
        targetPosition = bossPosition + Vector3.new(0, distance, 0)
    elseif Config.BossFarm.Position == "Abaixo" then
        targetPosition = bossPosition + Vector3.new(0, -distance, 0)
    end
    
    -- Teleporte suave
    local currentDistance = (humanoidRootPart.Position - targetPosition).Magnitude
    if currentDistance > 3 then
        humanoidRootPart.CFrame = CFrame.new(targetPosition)
    end
    
    -- Auto ataque
    if Config.BossFarm.AutoAttack and boss.Humanoid.Health > 0 then
        local attackDistance = (humanoidRootPart.Position - bossPosition).Magnitude
        if attackDistance <= Config.BossFarm.AttackRange then
            pcall(function()
                local remotes = Services.ReplicatedStorage:FindFirstChild("Remotes")
                if remotes then
                    local combatRemote = remotes:FindFirstChild("Combat") or remotes:FindFirstChild("Attack")
                    if combatRemote then
                        combatRemote:FireServer("M1")
                    end
                end
            end)
        end
    end
end

-- =============== INTERFACE GRÁFICA ===============
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VVUltimatumUI"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 420, 0, 600)
MainFrame.Position = UDim2.new(0.5, -210, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = TitleBar

local TitlePatch = Instance.new("Frame")
TitlePatch.Size = UDim2.new(1, 0, 0, 10)
TitlePatch.Position = UDim2.new(0, 0, 1, -10)
TitlePatch.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
TitlePatch.BorderSizePixel = 0
TitlePatch.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -50, 1, 0)
TitleLabel.Position = UDim2.new(0, 15, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "⚔️ VV Ultimatum Pro"
TitleLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
TitleLabel.TextSize = 16
TitleLabel.Font = Enum.Font.GothamBlack
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
CloseButton.Text = "✕"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 18
CloseButton.Font = Enum.Font.GothamBold
CloseButton.BorderSizePixel = 0
CloseButton.AutoButtonColor = false
CloseButton.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 15)
CloseCorner.Parent = CloseButton

-- Container com scroll
local ScrollingFrame = Instance.new("ScrollingFrame")
ScrollingFrame.Size = UDim2.new(1, 0, 1, -45)
ScrollingFrame.Position = UDim2.new(0, 0, 0, 45)
ScrollingFrame.BackgroundTransparency = 1
ScrollingFrame.BorderSizePixel = 0
ScrollingFrame.ScrollBarThickness = 4
ScrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 1100)
ScrollingFrame.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 8)
UIListLayout.Parent = ScrollingFrame

-- Funções da UI
local function CreateSection(title, height)
    local Section = Instance.new("Frame")
    Section.Size = UDim2.new(1, -20, 0, height)
    Section.Position = UDim2.new(0, 10, 0, 0)
    Section.BackgroundColor3 = Color3.fromRGB(30, 30, 42)
    Section.BorderSizePixel = 0
    Section.Parent = ScrollingFrame
    
    local SectionCorner = Instance.new("UICorner")
    SectionCorner.CornerRadius = UDim.new(0, 8)
    SectionCorner.Parent = Section
    
    local SectionTitle = Instance.new("TextLabel")
    SectionTitle.Size = UDim2.new(1, -20, 0, 30)
    SectionTitle.Position = UDim2.new(0, 10, 0, 8)
    SectionTitle.BackgroundTransparency = 1
    SectionTitle.Text = title
    SectionTitle.TextColor3 = Color3.fromRGB(0, 200, 255)
    SectionTitle.TextSize = 14
    SectionTitle.Font = Enum.Font.GothamBold
    SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
    SectionTitle.Parent = Section
    
    return Section
end

local function CreateToggle(parent, text, yPos, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Size = UDim2.new(1, -20, 0, 35)
    ToggleFrame.Position = UDim2.new(0, 10, 0, yPos)
    ToggleFrame.BackgroundTransparency = 1
    ToggleFrame.Parent = parent
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.65, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(255, 255, 255)
    Label.TextSize = 13
    Label.Font = Enum.Font.Gotham
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = ToggleFrame
    
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0, 65, 0, 28)
    Button.Position = UDim2.new(1, -65, 0.5, -14)
    Button.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
    Button.Text = "OFF"
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextSize = 11
    Button.Font = Enum.Font.GothamBold
    Button.BorderSizePixel = 0
    Button.AutoButtonColor = false
    Button.Parent = ToggleFrame
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 14)
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
    Button.Size = UDim2.new(1, -20, 0, 35)
    Button.Position = UDim2.new(0, 10, 0, yPos)
    Button.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    Button.Text = text
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextSize = 12
    Button.Font = Enum.Font.Gotham
    Button.BorderSizePixel = 0
    Button.AutoButtonColor = false
    Button.Parent = parent
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 6)
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
    DropdownFrame.Size = UDim2.new(1, -20, 0, 35)
    DropdownFrame.Position = UDim2.new(0, 10, 0, yPos)
    DropdownFrame.BackgroundTransparency = 1
    DropdownFrame.Parent = parent
    
    local MainButton = Instance.new("TextButton")
    MainButton.Size = UDim2.new(1, 0, 1, 0)
    MainButton.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    MainButton.Text = options[1] or "Selecionar"
    MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MainButton.TextSize = 12
    MainButton.Font = Enum.Font.Gotham
    MainButton.BorderSizePixel = 0
    MainButton.AutoButtonColor = false
    MainButton.Parent = DropdownFrame
    
    local MainCorner = Instance.new("UICorner")
    MainCorner.CornerRadius = UDim.new(0, 6)
    MainCorner.Parent = MainButton
    
    local DropList = Instance.new("Frame")
    DropList.Size = UDim2.new(1, 0, 0, #options * 35)
    DropList.Position = UDim2.new(0, 0, 1, 3)
    DropList.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    DropList.BorderSizePixel = 0
    DropList.Visible = false
    DropList.ZIndex = 10
    DropList.Parent = DropdownFrame
    
    local DropCorner = Instance.new("UICorner")
    DropCorner.CornerRadius = UDim.new(0, 6)
    DropCorner.Parent = DropList
    
    for i, option in ipairs(options) do
        local OptionButton = Instance.new("TextButton")
        OptionButton.Size = UDim2.new(1, 0, 0, 35)
        OptionButton.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
        OptionButton.Text = option
        OptionButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        OptionButton.TextSize = 12
        OptionButton.Font = Enum.Font.Gotham
        OptionButton.BorderSizePixel = 0
        OptionButton.ZIndex = 11
        OptionButton.Parent = DropList
        
        OptionButton.MouseEnter:Connect(function()
            OptionButton.BackgroundColor3 = Color3.fromRGB(70, 70, 85)
        end)
        
        OptionButton.MouseLeave:Connect(function()
            OptionButton.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
        end)
        
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

-- Seção 1: Auto Parry
local ParrySection = CreateSection("🛡️ Auto Parry", 80)
CreateToggle(ParrySection, "Defesa Automática (M1)", 45, function(enabled)
    Config.AutoParry.Enabled = enabled
end)

-- Seção 2: ESP Hitboxes
local ESPSection = CreateSection("👁️ ESP de Hitboxes", 80)
CreateToggle(ESPSection, "Hitboxes dos Mobs", 45, function(enabled)
    Config.HitboxESP.Enabled = enabled
    if not enabled then
        UpdateHitboxESP() -- Limpar hitboxes
    end
end)

-- Seção 3: Bosses
local BossSection = CreateSection("👑 Bosses", 260)

local BossListFrame = Instance.new("ScrollingFrame")
BossListFrame.Size = UDim2.new(1, -20, 0, 130)
BossListFrame.Position = UDim2.new(0, 10, 0, 45)
BossListFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
BossListFrame.BorderSizePixel = 0
BossListFrame.ScrollBarThickness = 3
BossListFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
BossListFrame.Parent = BossSection

local BossListCorner = Instance.new("UICorner")
BossListCorner.CornerRadius = UDim.new(0, 5)
BossListCorner.Parent = BossListFrame

local BossListLayout = Instance.new("UIListLayout")
BossListLayout.Padding = UDim.new(0, 2)
BossListLayout.Parent = BossListFrame

local SelectedBossText = Instance.new("TextLabel")
SelectedBossText.Size = UDim2.new(1, -20, 0, 20)
SelectedBossText.Position = UDim2.new(0, 10, 0, 180)
SelectedBossText.BackgroundTransparency = 1
SelectedBossText.Text = "🎯 Nenhum boss selecionado"
SelectedBossText.TextColor3 = Color3.fromRGB(255, 200, 0)
SelectedBossText.TextSize = 11
SelectedBossText.Font = Enum.Font.Gotham
SelectedBossText.TextXAlignment = Enum.TextXAlignment.Left
SelectedBossText.Parent = BossSection

CreateButton(BossSection, "🔄 Atualizar Lista", 205, function()
    UpdateBossList()
end)

-- Seção 4: Auto Farm
local FarmSection = CreateSection("⚔️ Auto Farm Boss", 160)

CreateToggle(FarmSection, "Auto Farm Ativo", 45, function(enabled)
    Config.BossFarm.Enabled = enabled
    if enabled and not Config.BossFarm.TargetBoss then
        print("[Auto Farm] Selecione um boss primeiro!")
    end
end)

CreateToggle(FarmSection, "Auto Atacar", 85, function(enabled)
    Config.BossFarm.AutoAttack = enabled
end)

CreateDropdown(FarmSection, Config.Combat.Positions, 125, function(selected)
    Config.BossFarm.Position = selected
end)

-- ===================== FUNÇÕES DE ATUALIZAÇÃO =====================

function UpdateBossList()
    -- Limpar lista
    for _, child in ipairs(BossListFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    local bosses = GetBosses()
    
    if #bosses == 0 then
        local NoBossLabel = Instance.new("TextLabel")
        NoBossLabel.Size = UDim2.new(1, 0, 0, 30)
        NoBossLabel.BackgroundTransparency = 1
        NoBossLabel.Text = "Nenhum boss encontrado"
        NoBossLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
        NoBossLabel.TextSize = 12
        NoBossLabel.Font = Enum.Font.Gotham
        NoBossLabel.Parent = BossListFrame
    else
        for _, boss in ipairs(bosses) do
            local BossButton = Instance.new("TextButton")
            BossButton.Size = UDim2.new(1, -4, 0, 28)
            BossButton.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
            BossButton.Text = string.format("%s [❤️ %d/%d]", boss.Name, boss.Health, boss.MaxHealth)
            BossButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            BossButton.TextSize = 10
            BossButton.Font = Enum.Font.Gotham
            BossButton.BorderSizePixel = 0
            BossButton.AutoButtonColor = false
            BossButton.Parent = BossListFrame
            
            BossButton.MouseButton1Click:Connect(function()
                Config.BossFarm.TargetBoss = boss
                SelectedBossText.Text = "🎯 Selecionado: " .. boss.Name
                
                -- Highlight visual
                for _, btn in ipairs(BossListFrame:GetChildren()) do
                    if btn:IsA("TextButton") then
                        btn.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
                    end
                end
                BossButton.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
            end)
        end
    end
    
    BossListFrame.CanvasSize = UDim2.new(0, 0, 0, #bosses * 30)
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

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Loop principal otimizado
Services.RunService.Heartbeat:Connect(function()
    pcall(function()
        AutoParry()
        UpdateHitboxESP()
        AutoFarmBoss()
    end)
end)

-- Atualização menos frequente para a UI
spawn(function()
    while wait(1) do
        pcall(function()
            UpdateBossList()
        end)
    end
end)

-- Reconectar quando personagem mudar
Player.CharacterAdded:Connect(function(character)
    wait(1) -- Aguardar carregar
    Config.AutoParry.LastParry = 0
    print("[Sistema] Novo personagem conectado")
end)

print("✅ VV Ultimatum Pro carregado!")
print("🎯 Procure pelo Giant Dragonfly na lista de bosses")
print("⚔️ Selecione o boss e ative o Auto Farm!")

end)
