--[[
    VV Ultimatum - Script Otimizado v4.0
    Categorizado e anti-travamento
]]

-- ============================================
-- CATEGORIA 1: SERVIÇOS E VARIÁVEIS
-- ============================================
local Services = {
    Players = game:GetService("Players"),
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    TweenService = game:GetService("TweenService"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    Workspace = game:GetService("Workspace")
}

local Player = Services.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local Character = Player.Character or Player.CharacterAdded:Wait()

-- Estados dos sistemas
local Systems = {
    AutoParry = false,
    HitboxESP = false,
    AutoFarm = false,
    AutoAttack = false,
    SelectedBoss = nil,
    Position = "Frente"
}

-- Controle de performance
local Performance = {
    LastParryTime = 0,
    ParryCooldown = 0.5,
    LastESPUpdate = 0,
    ESPUpdateInterval = 0.3,
    LastFarmUpdate = 0,
    FarmUpdateInterval = 0.2,
    LastBossUpdate = 0,
    BossUpdateInterval = 3
}

-- ============================================
-- CATEGORIA 2: FUNÇÕES DE DETECÇÃO
-- ============================================
local Detector = {}

-- Encontra apenas mobs (hollows, monstros)
function Detector.GetMobs()
    local mobs = {}
    local playerChar = Player.Character
    
    for _, obj in ipairs(Services.Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= playerChar then
            local humanoid = obj:FindFirstChild("Humanoid")
            local rootPart = obj:FindFirstChild("HumanoidRootPart")
            
            if humanoid and rootPart and humanoid.Health > 0 then
                -- Verificar se é um mob
                local isMob = false
                local name = obj.Name:lower()
                
                if name:find("hollow") or name:find("menos") or 
                   name:find("arrancar") or name:find("vasto") or
                   obj:FindFirstChild("Mob") or obj:GetAttribute("IsMob") then
                    isMob = true
                end
                
                if isMob then
                    table.insert(mobs, obj)
                end
            end
        end
    end
    
    return mobs
end

-- Encontra bosses (incluindo Giant Dragonfly)
function Detector.GetBosses()
    local bosses = {}
    
    for _, obj in ipairs(Services.Workspace:GetDescendants()) do
        if obj:IsA("Model") then
            local humanoid = obj:FindFirstChild("Humanoid")
            local rootPart = obj:FindFirstChild("HumanoidRootPart")
            
            if humanoid and rootPart and humanoid.Health > 0 then
                local name = obj.Name:lower()
                local isBoss = false
                
                -- Giant Dragonfly específico
                if name:find("giant") and name:find("dragonfly") then
                    isBoss = true
                -- Outros bosses
                elseif name:find("boss") or obj:FindFirstChild("Boss") then
                    isBoss = true
                end
                
                if isBoss then
                    table.insert(bosses, {
                        Object = obj,
                        Name = obj.Name,
                        Health = math.floor(humanoid.Health),
                        MaxHealth = math.floor(humanoid.MaxHealth),
                        RootPart = rootPart,
                        Humanoid = humanoid
                    })
                end
            end
        end
    end
    
    return bosses
end

-- Verifica se um mob está atacando
function Detector.IsMobAttacking(mob)
    local humanoid = mob:FindFirstChild("Humanoid")
    if not humanoid then return false end
    
    local animator = humanoid:FindFirstChild("Animator")
    if not animator then return false end
    
    for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
        local animId = track.Animation.AnimationId:lower()
        if animId:find("attack") or animId:find("m1") or animId:find("hit") then
            return true
        end
    end
    
    return false
end

-- ============================================
-- CATEGORIA 3: SISTEMA DE AUTO PARRY
-- ============================================
local AutoParrySystem = {}

function AutoParrySystem.Update()
    if not Systems.AutoParry then return end
    
    local currentTime = tick()
    if currentTime - Performance.LastParryTime < Performance.ParryCooldown then
        return
    end
    
    local character = Player.Character
    if not character then return end
    
    local charRoot = character:FindFirstChild("HumanoidRootPart")
    if not charRoot then return end
    
    local mobs = Detector.GetMobs()
    
    for _, mob in ipairs(mobs) do
        local mobRoot = mob:FindFirstChild("HumanoidRootPart")
        if mobRoot then
            local distance = (charRoot.Position - mobRoot.Position).Magnitude
            
            -- Só verifica mobs próximos
            if distance <= 20 then
                if Detector.IsMobAttacking(mob) then
                    -- Tentar defender
                    pcall(function()
                        local remotes = Services.ReplicatedStorage:FindFirstChild("Remotes")
                        if remotes then
                            local combat = remotes:FindFirstChild("Combat") or remotes:FindFirstChild("Parry") or remotes:FindFirstChild("Defense")
                            if combat then
                                combat:FireServer("Parry")
                                Performance.LastParryTime = currentTime
                                return
                            end
                        end
                    end)
                end
            end
        end
    end
end

-- ============================================
-- CATEGORIA 4: SISTEMA DE HITBOX ESP
-- ============================================
local HitboxESPSystem = {
    ActiveHitboxes = {}
}

function HitboxESPSystem.Update()
    local currentTime = tick()
    if currentTime - Performance.LastESPUpdate < Performance.ESPUpdateInterval then
        return
    end
    
    -- Limpar hitboxes se desativado
    if not Systems.HitboxESP then
        HitboxESPSystem.ClearAll()
        return
    end
    
    local mobs = Detector.GetMobs()
    local newHitboxes = {}
    
    for _, mob in ipairs(mobs) do
        local rootPart = mob:FindFirstChild("HumanoidRootPart")
        if rootPart then
            local hitbox = mob:FindFirstChild("ESPHitbox")
            
            if not hitbox then
                -- Criar nova hitbox
                hitbox = Instance.new("Part")
                hitbox.Name = "ESPHitbox"
                hitbox.Size = Vector3.new(3, 5, 3)
                hitbox.Color = Color3.fromRGB(255, 0, 0)
                hitbox.Transparency = 0.7
                hitbox.Material = Enum.Material.Neon
                hitbox.Anchored = true
                hitbox.CanCollide = false
                hitbox.CanQuery = false
                hitbox.CanTouch = false
                hitbox.Parent = mob
            end
            
            hitbox.Position = rootPart.Position
            table.insert(newHitboxes, hitbox)
        end
    end
    
    -- Remover hitboxes de mobs que morreram
    for _, oldHitbox in ipairs(HitboxESPSystem.ActiveHitboxes) do
        if oldHitbox and oldHitbox.Parent and not table.find(newHitboxes, oldHitbox) then
            oldHitbox:Destroy()
        end
    end
    
    HitboxESPSystem.ActiveHitboxes = newHitboxes
    Performance.LastESPUpdate = currentTime
end

function HitboxESPSystem.ClearAll()
    for _, hitbox in ipairs(HitboxESPSystem.ActiveHitboxes) do
        if hitbox and hitbox.Parent then
            hitbox:Destroy()
        end
    end
    HitboxESPSystem.ActiveHitboxes = {}
end

-- ============================================
-- CATEGORIA 5: SISTEMA DE AUTO FARM
-- ============================================
local AutoFarmSystem = {}

function AutoFarmSystem.Update()
    if not Systems.AutoFarm then return end
    if not Systems.SelectedBoss then return end
    
    local currentTime = tick()
    if currentTime - Performance.LastFarmUpdate < Performance.FarmUpdateInterval then
        return
    end
    
    local character = Player.Character
    if not character then return end
    
    local charRoot = character:FindFirstChild("HumanoidRootPart")
    if not charRoot then return end
    
    local boss = Systems.SelectedBoss
    if not boss.RootPart or not boss.RootPart.Parent then
        Systems.SelectedBoss = nil
        return
    end
    
    -- Calcular posição
    local bossPos = boss.RootPart.Position
    local targetPos = bossPos
    
    local distance = 10
    
    if Systems.Position == "Frente" then
        targetPos = bossPos + (boss.RootPart.CFrame.LookVector * distance)
    elseif Systems.Position == "Atrás" then
        targetPos = bossPos + (-boss.RootPart.CFrame.LookVector * distance)
    elseif Systems.Position == "Acima" then
        targetPos = bossPos + Vector3.new(0, distance, 0)
    elseif Systems.Position == "Abaixo" then
        targetPos = bossPos + Vector3.new(0, -distance, 0)
    end
    
    -- Teleporte
    if (charRoot.Position - targetPos).Magnitude > 2 then
        charRoot.CFrame = CFrame.new(targetPos)
    end
    
    -- Auto ataque
    if Systems.AutoAttack then
        local attackDistance = (charRoot.Position - bossPos).Magnitude
        if attackDistance <= 15 then
            pcall(function()
                local remotes = Services.ReplicatedStorage:FindFirstChild("Remotes")
                if remotes then
                    local combat = remotes:FindFirstChild("Combat") or remotes:FindFirstChild("Attack")
                    if combat then
                        combat:FireServer("Attack")
                    end
                end
            end)
        end
    end
    
    Performance.LastFarmUpdate = currentTime
end

-- ============================================
-- CATEGORIA 6: INTERFACE GRÁFICA
-- ============================================
local UI = {}

function UI.Create()
    -- ScreenGui
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "VVUltimatumUI"
    ScreenGui.Parent = PlayerGui
    ScreenGui.ResetOnSpawn = false
    
    -- Frame principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "Main"
    MainFrame.Size = UDim2.new(0, 380, 0, 500)
    MainFrame.Position = UDim2.new(0.5, -190, 0.3, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ScreenGui
    
    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 8)
    Corner.Parent = MainFrame
    
    -- Título
    local TitleBar = Instance.new("Frame")
    TitleBar.Size = UDim2.new(1, 0, 0, 35)
    TitleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame
    
    local TitleText = Instance.new("TextLabel")
    TitleText.Size = UDim2.new(1, -40, 1, 0)
    TitleText.Position = UDim2.new(0, 15, 0, 0)
    TitleText.BackgroundTransparency = 1
    TitleText.Text = "VV Ultimatum"
    TitleText.TextColor3 = Color3.fromRGB(0, 200, 255)
    TitleText.TextSize = 15
    TitleText.Font = Enum.Font.GothamBold
    TitleText.TextXAlignment = Enum.TextXAlignment.Left
    TitleText.Parent = TitleBar
    
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 25, 0, 25)
    CloseBtn.Position = UDim2.new(1, -30, 0, 5)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.TextSize = 14
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.BorderSizePixel = 0
    CloseBtn.Parent = TitleBar
    
    -- Container com scroll
    local Scroll = Instance.new("ScrollingFrame")
    Scroll.Size = UDim2.new(1, 0, 1, -40)
    Scroll.Position = UDim2.new(0, 0, 0, 40)
    Scroll.BackgroundTransparency = 1
    Scroll.BorderSizePixel = 0
    Scroll.ScrollBarThickness = 3
    Scroll.CanvasSize = UDim2.new(0, 0, 0, 800)
    Scroll.Parent = MainFrame
    
    local List = Instance.new("UIListLayout")
    List.Padding = UDim.new(0, 8)
    List.Parent = Scroll
    
    -- Função para criar seção
    function UI.CreateSection(title, height)
        local Section = Instance.new("Frame")
        Section.Size = UDim2.new(1, -16, 0, height)
        Section.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
        Section.BorderSizePixel = 0
        Section.Parent = Scroll
        
        local SectionCorner = Instance.new("UICorner")
        SectionCorner.CornerRadius = UDim.new(0, 5)
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
    
    -- Função para criar toggle
    function UI.CreateToggle(parent, text, yPos, callback)
        local Toggle = Instance.new("Frame")
        Toggle.Size = UDim2.new(1, -16, 0, 30)
        Toggle.Position = UDim2.new(0, 8, 0, yPos)
        Toggle.BackgroundTransparency = 1
        Toggle.Parent = parent
        
        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(0.6, 0, 1, 0)
        Label.BackgroundTransparency = 1
        Label.Text = text
        Label.TextColor3 = Color3.fromRGB(255, 255, 255)
        Label.TextSize = 12
        Label.Font = Enum.Font.Gotham
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Toggle
        
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(0, 55, 0, 24)
        Button.Position = UDim2.new(1, -55, 0.5, -12)
        Button.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
        Button.Text = "OFF"
        Button.TextColor3 = Color3.fromRGB(255, 255, 255)
        Button.TextSize = 10
        Button.Font = Enum.Font.GothamBold
        Button.BorderSizePixel = 0
        Button.AutoButtonColor = false
        Button.Parent = Toggle
        
        local BCorner = Instance.new("UICorner")
        BCorner.CornerRadius = UDim.new(0, 12)
        BCorner.Parent = Button
        
        local state = false
        Button.MouseButton1Click:Connect(function()
            state = not state
            if state then
                Button.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
                Button.Text = "ON"
            else
                Button.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
                Button.Text = "OFF"
            end
            callback(state)
        end)
    end
    
    -- Função para criar dropdown
    function UI.CreateDropdown(parent, options, yPos, callback)
        local DropFrame = Instance.new("Frame")
        DropFrame.Size = UDim2.new(1, -16, 0, 30)
        DropFrame.Position = UDim2.new(0, 8, 0, yPos)
        DropFrame.BackgroundTransparency = 1
        DropFrame.Parent = parent
        
        local MainBtn = Instance.new("TextButton")
        MainBtn.Size = UDim2.new(1, 0, 1, 0)
        MainBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
        MainBtn.Text = options[1] or "..."
        MainBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        MainBtn.TextSize = 12
        MainBtn.Font = Enum.Font.Gotham
        MainBtn.BorderSizePixel = 0
        MainBtn.Parent = DropFrame
        
        local MainCorner = Instance.new("UICorner")
        MainCorner.CornerRadius = UDim.new(0, 4)
        MainCorner.Parent = MainBtn
        
        local DropList = Instance.new("Frame")
        DropList.Size = UDim2.new(1, 0, 0, #options * 30)
        DropList.Position = UDim2.new(0, 0, 1, 2)
        DropList.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
        DropList.BorderSizePixel = 0
        DropList.Visible = false
        DropList.ZIndex = 5
        DropList.Parent = DropFrame
        
        for i, opt in ipairs(options) do
            local OptBtn = Instance.new("TextButton")
            OptBtn.Size = UDim2.new(1, 0, 0, 30)
            OptBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
            OptBtn.Text = opt
            OptBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            OptBtn.TextSize = 11
            OptBtn.Font = Enum.Font.Gotham
            OptBtn.BorderSizePixel = 0
            OptBtn.ZIndex = 6
            OptBtn.Parent = DropList
            
            OptBtn.MouseButton1Click:Connect(function()
                MainBtn.Text = opt
                DropList.Visible = false
                callback(opt)
            end)
        end
        
        MainBtn.MouseButton1Click:Connect(function()
            DropList.Visible = not DropList.Visible
        end)
    end
    
    -- ============ SEÇÕES DA UI ============
    
    -- Seção 1: Auto Parry
    local Sec1 = UI.CreateSection("🛡️ Auto Parry", 70)
    UI.CreateToggle(Sec1, "Defesa Automática", 40, function(state)
        Systems.AutoParry = state
    end)
    
    -- Seção 2: Hitbox ESP
    local Sec2 = UI.CreateSection("👁️ Hitbox ESP", 70)
    UI.CreateToggle(Sec2, "Mostrar Hitboxes", 40, function(state)
        Systems.HitboxESP = state
        if not state then
            HitboxESPSystem.ClearAll()
        end
    end)
    
    -- Seção 3: Bosses
    local Sec3 = UI.CreateSection("👑 Bosses", 200)
    
    local BossScroll = Instance.new("ScrollingFrame")
    BossScroll.Size = UDim2.new(1, -16, 0, 100)
    BossScroll.Position = UDim2.new(0, 8, 0, 35)
    BossScroll.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    BossScroll.BorderSizePixel = 0
    BossScroll.ScrollBarThickness = 2
    BossScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    BossScroll.Parent = Sec3
    
    local BossLayout = Instance.new("UIListLayout")
    BossLayout.Padding = UDim.new(0, 2)
    BossLayout.Parent = BossScroll
    
    local SelectedText = Instance.new("TextLabel")
    SelectedText.Size = UDim2.new(1, -16, 0, 20)
    SelectedText.Position = UDim2.new(0, 8, 0, 140)
    SelectedText.BackgroundTransparency = 1
    SelectedText.Text = "Nenhum boss selecionado"
    SelectedText.TextColor3 = Color3.fromRGB(255, 200, 0)
    SelectedText.TextSize = 11
    SelectedText.Font = Enum.Font.Gotham
    SelectedText.TextXAlignment = Enum.TextXAlignment.Left
    SelectedText.Parent = Sec3
    
    local RefreshBtn = Instance.new("TextButton")
    RefreshBtn.Size = UDim2.new(1, -16, 0, 30)
    RefreshBtn.Position = UDim2.new(0, 8, 0, 165)
    RefreshBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    RefreshBtn.Text = "🔄 Atualizar Lista"
    RefreshBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    RefreshBtn.TextSize = 11
    RefreshBtn.Font = Enum.Font.Gotham
    RefreshBtn.BorderSizePixel = 0
    RefreshBtn.Parent = Sec3
    
    local function UpdateBossUI()
        -- Limpar lista
        for _, child in ipairs(BossScroll:GetChildren()) do
            if child:IsA("TextButton") then
                child:Destroy()
            end
        end
        
        local bosses = Detector.GetBosses()
        
        if #bosses == 0 then
            local NoBoss = Instance.new("TextLabel")
            NoBoss.Size = UDim2.new(1, -8, 0, 25)
            NoBoss.Position = UDim2.new(0, 4, 0, 0)
            NoBoss.BackgroundTransparency = 1
            NoBoss.Text = "Nenhum boss encontrado"
            NoBoss.TextColor3 = Color3.fromRGB(150, 150, 150)
            NoBoss.TextSize = 11
            NoBoss.Font = Enum.Font.Gotham
            NoBoss.Parent = BossScroll
        else
            for _, boss in ipairs(bosses) do
                local BossBtn = Instance.new("TextButton")
                BossBtn.Size = UDim2.new(1, -4, 0, 25)
                BossBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
                BossBtn.Text = boss.Name .. " [❤️ " .. boss.Health .. "/" .. boss.MaxHealth .. "]"
                BossBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
                BossBtn.TextSize = 10
                BossBtn.Font = Enum.Font.Gotham
                BossBtn.BorderSizePixel = 0
                BossBtn.AutoButtonColor = false
                BossBtn.Parent = BossScroll
                
                BossBtn.MouseButton1Click:Connect(function()
                    Systems.SelectedBoss = boss
                    SelectedText.Text = "Selecionado: " .. boss.Name
                    
                    for _, btn in ipairs(BossScroll:GetChildren()) do
                        if btn:IsA("TextButton") then
                            btn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
                        end
                    end
                    BossBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
                end)
            end
        end
        
        BossScroll.CanvasSize = UDim2.new(0, 0, 0, #bosses * 27)
    end
    
    RefreshBtn.MouseButton1Click:Connect(UpdateBossUI)
    
    -- Seção 4: Auto Farm
    local Sec4 = UI.CreateSection("⚔️ Auto Farm", 130)
    
    UI.CreateToggle(Sec4, "Ativar Auto Farm", 40, function(state)
        Systems.AutoFarm = state
    end)
    
    UI.CreateToggle(Sec4, "Auto Atacar", 75, function(state)
        Systems.AutoAttack = state
    end)
    
    UI.CreateDropdown(Sec4, {"Frente", "Atrás", "Acima", "Abaixo"}, 110, function(opt)
        Systems.Position = opt
    end)
    
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
    
    CloseBtn.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)
    
    -- Retornar função de atualização
    return UpdateBossUI
end

-- ============================================
-- CATEGORIA 7: LOOP PRINCIPAL (OTIMIZADO)
-- ============================================
local UpdateBossUI = UI.Create()

-- Loop principal com Heartbeat (60 FPS)
Services.RunService.Heartbeat:Connect(function()
    pcall(function()
        AutoParrySystem.Update()
        HitboxESPSystem.Update()
        AutoFarmSystem.Update()
    end)
end)

-- Atualização lenta para bosses (a cada 3 segundos)
spawn(function()
    while wait(3) do
        pcall(function()
            UpdateBossUI()
        end)
    end
end)

-- Reconectar personagem
Player.CharacterAdded:Connect(function(char)
    Character = char
    Performance.LastParryTime = 0
    HitboxESPSystem.ClearAll()
end)

print("✅ Script carregado com sucesso!")
print("📋 Comandos disponíveis:")
print("   🛡️ Auto Parry - Defesa automática")
print("   👁️ Hitbox ESP - Visualização de hitboxes")
print("   👑 Bosses - Lista de bosses (Giant Dragonfly)")
print("   ⚔️ Auto Farm - Farm automático de bosses")
