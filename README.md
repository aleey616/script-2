--[[
    VV Ultimatum - Script Completo v4.0
    Correções: Auto Parry, Hitbox, TP com chão, Auto Farm, Coleta de Baús
]]

local Services = {
    Players = game:GetService("Players"),
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    TweenService = game:GetService("TweenService"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    Workspace = game:GetService("Workspace"),
    TeleportService = game:GetService("TeleportService")
}

local Player = Services.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local Character = Player.Character or Player.CharacterAdded:Wait()

-- Configurações
local Config = {
    AutoParry = {
        Enabled = false,
        Range = 30,
        Cooldown = 0.1,
        LastParry = 0,
        Keybind = "F"
    },
    ESP = {
        Enabled = false,
        BossESP = false,
        MobColor = Color3.fromRGB(255, 50, 50),
        BossColor = Color3.fromRGB(255, 200, 0),
        Transparency = 0.5
    },
    AutoFarm = {
        Enabled = false,
        TargetBoss = nil,
        Position = "Em Cima", -- Em Cima, Frente, Atrás, Abaixo
        Distance = 15,
        AutoAttack = false,
        CollectItems = true,
        CreatePlatform = true
    },
    Positions = {"Em Cima", "Frente", "Atrás", "Abaixo"}
}

-- Variáveis do sistema
local Platform = nil
local Hitboxes = {}
local CollectedItems = {}

-- Função para criar plataforma embaixo do boss
local function CreatePlatformUnderBoss(boss)
    if not boss or not boss.RootPart then return end
    
    -- Remover plataforma antiga
    if Platform and Platform.Parent then
        Platform:Destroy()
    end
    
    -- Criar nova plataforma
    Platform = Instance.new("Part")
    Platform.Name = "FarmPlatform"
    Platform.Size = Vector3.new(20, 1, 20)
    Platform.Position = boss.RootPart.Position - Vector3.new(0, 15, 0)
    Platform.Anchored = true
    Platform.CanCollide = true
    Platform.Transparency = 0.5
    Platform.Color = Color3.fromRGB(100, 100, 100)
    Platform.Material = Enum.Material.SmoothPlastic
    Platform.Parent = Services.Workspace
    
    print("[Plataforma] Criada embaixo do boss")
end

-- Função para remover plataforma
local function RemovePlatform()
    if Platform and Platform.Parent then
        Platform:Destroy()
        Platform = nil
    end
end

-- Função para detectar mobs e bosses
local function GetMobsAndBosses()
    local mobs = {}
    local bosses = {}
    
    for _, obj in ipairs(Services.Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
            local humanoid = obj.Humanoid
            local rootPart = obj.HumanoidRootPart
            local name = obj.Name:lower()
            
            if humanoid.Health > 0 and not Services.Players:GetPlayerFromCharacter(obj) then
                local entity = {
                    Model = obj,
                    Humanoid = humanoid,
                    RootPart = rootPart,
                    Name = obj.Name,
                    Health = humanoid.Health,
                    MaxHealth = humanoid.MaxHealth,
                    Position = rootPart.Position
                }
                
                -- Verificar se é boss
                if name:find("boss") or 
                   name:find("giant") or 
                   name:find("dragonfly") or
                   name:find("vasto") and name:find("lorde") or
                   obj:FindFirstChild("Boss") or
                   rootPart:GetAttribute("IsBoss") then
                    table.insert(bosses, entity)
                end
                
                -- Verificar se é mob
                if name:find("hollow") or 
                   name:find("menos") or 
                   name:find("arrancar") or
                   name:find("monster") or
                   name:find("enemy") or
                   rootPart:GetAttribute("IsMob") or
                   rootPart:GetAttribute("IsEnemy") then
                    table.insert(mobs, entity)
                end
            end
        end
    end
    
    return mobs, bosses
end

-- Sistema de Auto Parry corrigido
local function AutoParrySystem()
    if not Config.AutoParry.Enabled then return end
    
    local character = Player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return end
    
    local now = tick()
    if now - Config.AutoParry.LastParry < Config.AutoParry.Cooldown then return end
    
    local mobs, bosses = GetMobsAndBosses()
    local allEnemies = {}
    for _, mob in ipairs(mobs) do table.insert(allEnemies, mob) end
    for _, boss in ipairs(bosses) do table.insert(allEnemies, boss) end
    
    local charPos = character:GetPivot().Position
    
    for _, enemy in ipairs(allEnemies) do
        local distance = (enemy.Position - charPos).Magnitude
        if distance <= Config.AutoParry.Range then
            -- Verificar se inimigo está atacando
            local attacking = false
            
            -- Método 1: Verificar animações
            local animator = enemy.Humanoid:FindFirstChild("Animator")
            if animator then
                for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                    local animId = track.Animation.AnimationId:lower()
                    if animId:find("attack") or animId:find("m1") or animId:find("hit") or 
                       animId:find("punch") or animId:find("strike") or animId:find("slash") then
                        attacking = true
                        break
                    end
                end
            end
            
            -- Método 2: Verificar se tem tool de ataque equipada
            if not attacking then
                for _, tool in ipairs(enemy.Model:GetChildren()) do
                    if tool:IsA("Tool") and tool:FindFirstChild("Handle") then
                        local handle = tool.Handle
                        if handle.Velocity.Magnitude > 10 then
                            attacking = true
                            break
                        end
                    end
                end
            end
            
            -- Método 3: Verificar mudança brusca de velocidade
            if not attacking and enemy.RootPart.Velocity.Magnitude > 30 then
                attacking = true
            end
            
            if attacking then
                -- Executar parry/defesa
                local success = false
                
                -- Tentar múltiplos métodos de defesa
                pcall(function()
                    -- Método 1: Remote de combate
                    local remotes = Services.ReplicatedStorage:FindFirstChild("Remotes")
                    if remotes then
                        local combatRemote = remotes:FindFirstChild("Combat") or 
                                            remotes:FindFirstChild("Parry") or
                                            remotes:FindFirstChild("Block")
                        if combatRemote then
                            combatRemote:FireServer("Parry")
                            combatRemote:FireServer("Block")
                            combatRemote:FireServer("Defend")
                            success = true
                        end
                    end
                end)
                
                if not success then
                    pcall(function()
                        -- Método 2: Evento de tecla
                        local vim = game:GetService("VirtualInputManager")
                        vim:SendKeyEvent(true, Config.AutoParry.Keybind, false, game)
                        vim:SendKeyEvent(false, Config.AutoParry.Keybind, false, game)
                        success = true
                    end)
                end
                
                if success then
                    Config.AutoParry.LastParry = now
                    print("[Auto Parry] ✓ Defendeu ataque de " .. enemy.Name)
                    return
                end
            end
        end
    end
end

-- Sistema de ESP (Hitbox) corrigido
local function UpdateESP()
    -- Limpar se desativado
    if not Config.ESP.Enabled then
        for _, hitbox in ipairs(Hitboxes) do
            if hitbox and hitbox.Parent then
                hitbox:Destroy()
            end
        end
        Hitboxes = {}
        return
    end
    
    local mobs, bosses = GetMobsAndBosses()
    local activeHitboxes = {}
    
    -- Função para criar/atualizar hitbox
    local function ProcessEntity(entity, color)
        local hitbox = entity.Model:FindFirstChild("ESP_Hitbox")
        
        if not hitbox then
            hitbox = Instance.new("Part")
            hitbox.Name = "ESP_Hitbox"
            hitbox.Size = Vector3.new(6, 7, 6)
            hitbox.Color = color
            hitbox.Transparency = Config.ESP.Transparency
            hitbox.Material = Enum.Material.ForceField
            hitbox.Anchored = true
            hitbox.CanCollide = false
            hitbox.CanQuery = false
            hitbox.CanTouch = false
            hitbox.Parent = entity.Model
            
            -- Adicionar Highlight para melhor visualização
            local highlight = Instance.new("Highlight")
            highlight.FillColor = color
            highlight.FillTransparency = Config.ESP.Transparency
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.OutlineTransparency = 0.2
            highlight.Parent = hitbox
        end
        
        hitbox.Position = entity.RootPart.Position
        table.insert(activeHitboxes, hitbox)
    end
    
    -- Processar mobs
    for _, mob in ipairs(mobs) do
        ProcessEntity(mob, Config.ESP.MobColor)
    end
    
    -- Processar bosses (se ativado)
    if Config.ESP.BossESP then
        for _, boss in ipairs(bosses) do
            ProcessEntity(boss, Config.ESP.BossColor)
        end
    end
    
    -- Remover hitboxes antigas
    for _, hitbox in ipairs(Hitboxes) do
        if hitbox and hitbox.Parent and not table.find(activeHitboxes, hitbox) then
            hitbox:Destroy()
        end
    end
    
    Hitboxes = activeHitboxes
end

-- Função para coletar itens (baús e drops)
local function CollectNearbyItems(position)
    if not Config.AutoFarm.CollectItems then return end
    
    local collectRange = 30
    
    for _, obj in ipairs(Services.Workspace:GetDescendants()) do
        if obj:IsA("BasePart") or obj:IsA("Model") then
            local objPos = obj:IsA("Model") and obj:GetPivot().Position or obj.Position
            local distance = (objPos - position).Magnitude
            
            if distance <= collectRange then
                local name = obj.Name:lower()
                
                -- Verificar se é item coletável
                if name:find("chest") or 
                   name:find("bau") or 
                   name:find("drop") or
                   name:find("item") or
                   name:find("loot") or
                   name:find("reward") or
                   name:find("gold") or
                   name:find("coin") or
                   obj:GetAttribute("IsLoot") or
                   obj:GetAttribute("IsItem") then
                    
                    -- Verificar se já foi coletado
                    if not CollectedItems[obj] then
                        -- Tentar coletar
                        pcall(function()
                            -- Método 1: Tocar no item
                            local character = Player.Character
                            if character then
                                local hrp = character:FindFirstChild("HumanoidRootPart")
                                if hrp then
                                    -- Teleportar até o item
                                    local oldPos = hrp.Position
                                    hrp.CFrame = CFrame.new(objPos)
                                    wait(0.1)
                                    hrp.CFrame = CFrame.new(oldPos)
                                end
                            end
                            
                            -- Método 2: Fire no remoto
                            local remotes = Services.ReplicatedStorage:FindFirstChild("Remotes")
                            if remotes then
                                local collectRemote = remotes:FindFirstChild("Collect") or 
                                                     remotes:FindFirstChild("PickUp") or
                                                     remotes:FindFirstChild("Loot")
                                if collectRemote then
                                    collectRemote:FireServer(obj)
                                end
                            end
                            
                            CollectedItems[obj] = true
                            print("[Coleta] ✓ Item coletado: " .. obj.Name)
                        end)
                    end
                end
            end
        end
    end
    
    -- Limpar cache de itens coletados periodicamente
    if #CollectedItems > 100 then
        CollectedItems = {}
    end
end

-- Sistema de Auto Farm corrigido
local function AutoFarmSystem()
    if not Config.AutoFarm.Enabled then return end
    if not Config.AutoFarm.TargetBoss then return end
    
    local character = Player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp or humanoid.Health <= 0 then return end
    
    local boss = Config.AutoFarm.TargetBoss
    if not boss.Model:FindFirstChild("HumanoidRootPart") then 
        print("[Auto Farm] Boss não encontrado")
        return 
    end
    
    local bossRoot = boss.Model.HumanoidRootPart
    local bossPos = bossRoot.Position
    
    -- Criar plataforma se configurado
    if Config.AutoFarm.CreatePlatform then
        CreatePlatformUnderBoss(boss)
    end
    
    -- Calcular posição alvo baseada na configuração
    local targetPos = bossPos
    local distance = Config.AutoFarm.Distance
    
    if Config.AutoFarm.Position == "Em Cima" then
        targetPos = bossPos + Vector3.new(0, distance, 0)
    elseif Config.AutoFarm.Position == "Frente" then
        targetPos = bossPos + (bossRoot.CFrame.LookVector * distance)
    elseif Config.AutoFarm.Position == "Atrás" then
        targetPos = bossPos + (-bossRoot.CFrame.LookVector * distance)
    elseif Config.AutoFarm.Position == "Abaixo" then
        targetPos = bossPos + Vector3.new(0, -distance, 0)
    end
    
    -- Ajustar para garantir que está no chão/plataforma
    local rayOrigin = targetPos
    local rayDirection = Vector3.new(0, -100, 0)
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Include
    rayParams.FilterDescendantsInstances = {Platform} or {}
    
    local rayResult = Services.Workspace:Raycast(rayOrigin, rayDirection, rayParams)
    if rayResult then
        targetPos = rayResult.Position + Vector3.new(0, 3, 0)
    end
    
    -- Teleporte suave
    local currentDist = (hrp.Position - targetPos).Magnitude
    if currentDist > 3 then
        hrp.CFrame = CFrame.new(targetPos)
    end
    
    -- Coletar itens próximos
    CollectNearbyItems(hrp.Position)
    
    -- Auto ataque
    if Config.AutoFarm.AutoAttack and boss.Humanoid.Health > 0 then
        local attackDist = (hrp.Position - bossPos).Magnitude
        
        if attackDist <= Config.AutoFarm.Distance + 10 then
            -- Tentar múltiplos métodos de ataque
            pcall(function()
                -- Método 1: Remote de combate
                local remotes = Services.ReplicatedStorage:FindFirstChild("Remotes")
                if remotes then
                    local attackRemote = remotes:FindFirstChild("Combat") or 
                                        remotes:FindFirstChild("Attack") or
                                        remotes:FindFirstChild("M1")
                    if attackRemote then
                        attackRemote:FireServer("M1")
                        attackRemote:FireServer("Attack")
                        print("[Auto Farm] ⚔️ Atacando " .. boss.Name)
                    end
                end
            end)
            
            -- Método 2: Simular clique
            pcall(function()
                local vim = game:GetService("VirtualInputManager")
                vim:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                wait(0.05)
                vim:SendMouseButtonEvent(0, 0, 0, false, game, 0)
            end)
        end
    end
end

-- ==================== INTERFACE GRÁFICA ====================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VVUltimatumUI"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 420, 0, 650)
MainFrame.Position = UDim2.new(0.5, -210, 0.15, 0)
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
TitleLabel.Text = "⚔️ VV Ultimatum Pro v4"
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
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 1200)
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
    Label.TextSize = 12
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

-- ===================== SEÇÕES DA UI =====================

-- Seção 1: Auto Parry
local ParrySection = CreateSection("🛡️ Auto Parry", 80)
CreateToggle(ParrySection, "Defesa Automática", 45, function(enabled)
    Config.AutoParry.Enabled = enabled
    print("[Auto Parry] " .. (enabled and "ATIVADO" or "DESATIVADO"))
end)

-- Seção 2: ESP Hitboxes
local ESPSection = CreateSection("👁️ ESP Hitboxes", 120)
CreateToggle(ESPSection, "Hitbox dos Mobs", 45, function(enabled)
    Config.ESP.Enabled = enabled
    if not enabled then
        UpdateESP()
    end
end)

CreateToggle(ESPSection, "Hitbox dos Bosses", 85, function(enabled)
    Config.ESP.BossESP = enabled
end)

-- Seção 3: Lista de Bosses
local BossSection = CreateSection("👑 Lista de Bosses", 280)

local BossListFrame = Instance.new("ScrollingFrame")
BossListFrame.Size = UDim2.new(1, -20, 0, 150)
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

local SelectedBossLabel = Instance.new("TextLabel")
SelectedBossLabel.Size = UDim2.new(1, -20, 0, 20)
SelectedBossLabel.Position = UDim2.new(0, 10, 0, 200)
SelectedBossLabel.BackgroundTransparency = 1
SelectedBossLabel.Text = "🎯 Nenhum boss selecionado"
SelectedBossLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
SelectedBossLabel.TextSize = 11
SelectedBossLabel.Font = Enum.Font.Gotham
SelectedBossLabel.TextXAlignment = Enum.TextXAlignment.Left
SelectedBossLabel.Parent = BossSection

CreateButton(BossSection, "🔄 Atualizar Lista", 225, function()
    UpdateBossList()
end)

-- Seção 4: Auto Farm
local FarmSection = CreateSection("⚔️ Auto Farm Boss", 200)

CreateToggle(FarmSection, "Auto Farm", 45, function(enabled)
    Config.AutoFarm.Enabled = enabled
    if not enabled then
        RemovePlatform()
    end
end)

CreateToggle(FarmSection, "Auto Atacar", 85, function(enabled)
    Config.AutoFarm.AutoAttack = enabled
end)

CreateToggle(FarmSection, "Criar Plataforma", 125, function(enabled)
    Config.AutoFarm.CreatePlatform = enabled
    if not enabled then
        RemovePlatform()
    end
end)

CreateDropdown(FarmSection, Config.Positions, 165, function(selected)
    Config.AutoFarm.Position = selected
    print("[Posição] " .. selected)
end)

-- ===================== FUNÇÕES DO SISTEMA =====================

function UpdateBossList()
    -- Limpar lista
    for _, child in ipairs(BossListFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    local _, bosses = GetMobsAndBosses()
    
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
                Config.AutoFarm.TargetBoss = boss
                SelectedBossLabel.Text = "🎯 Selecionado: " .. boss.Name
                
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
    RemovePlatform()
    ScreenGui:Destroy()
end)

-- Loop principal
Services.RunService.Heartbeat:Connect(function()
    pcall(function()
        AutoParrySystem()
        UpdateESP()
        AutoFarmSystem()
    end)
end)

-- Atualização da lista
