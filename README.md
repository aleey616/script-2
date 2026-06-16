--[[
    VV Ultimatum - Interface Avançada v2.0
    Script corrigido e funcional
]]

-- Proteção anti-erro
local success, errorMsg = pcall(function()

-- Serviços do Roblox
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

-- Variáveis principais
local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local Character = Player.Character or Player.CharacterAdded:Wait()

-- Configurações
local Config = {
    AutoDefense = false,
    ShowHitboxes = false,
    HitboxColor = Color3.fromRGB(255, 0, 0),
    HitboxTransparency = 0.5,
    TargetBoss = nil,
    CombatPosition = "Acima",
    AttackDistance = 15,
    UpdateInterval = 1
}

-- Criar ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VVUltimatumUI"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 400, 0, 550)
MainFrame.Position = UDim2.new(0.5, -200, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Borda arredondada
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = TitleBar

local TitlePatch = Instance.new("Frame")
TitlePatch.Size = UDim2.new(1, 0, 0, 8)
TitlePatch.Position = UDim2.new(0, 0, 1, -8)
TitlePatch.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
TitlePatch.BorderSizePixel = 0
TitlePatch.Parent = TitleBar

-- Título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -50, 1, 0)
TitleLabel.Position = UDim2.new(0, 15, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "⚔️ VV Ultimatum - Advanced"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 16
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Botão fechar
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
CloseButton.Text = "✕"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 18
CloseButton.Font = Enum.Font.GothamBold
CloseButton.BorderSizePixel = 0
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
UIListLayout.Padding = UDim.new(0, 10)
UIListLayout.Parent = ScrollingFrame

-- Função para criar seções
function CreateSection(title, height)
    local Section = Instance.new("Frame")
    Section.Size = UDim2.new(1, -20, 0, height)
    Section.Position = UDim2.new(0, 10, 0, 0)
    Section.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    Section.BorderSizePixel = 0
    Section.Parent = ScrollingFrame
    
    local SectionCorner = Instance.new("UICorner")
    SectionCorner.CornerRadius = UDim.new(0, 6)
    SectionCorner.Parent = Section
    
    local SectionTitle = Instance.new("TextLabel")
    SectionTitle.Size = UDim2.new(1, -20, 0, 30)
    SectionTitle.Position = UDim2.new(0, 10, 0, 5)
    SectionTitle.BackgroundTransparency = 1
    SectionTitle.Text = title
    SectionTitle.TextColor3 = Color3.fromRGB(0, 200, 255)
    SectionTitle.TextSize = 14
    SectionTitle.Font = Enum.Font.GothamBold
    SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
    SectionTitle.Parent = Section
    
    return Section
end

-- Função para criar Toggle
function CreateToggle(parent, text, yPosition, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Size = UDim2.new(1, -20, 0, 40)
    ToggleFrame.Position = UDim2.new(0, 10, 0, yPosition)
    ToggleFrame.BackgroundTransparency = 1
    ToggleFrame.Parent = parent
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(255, 255, 255)
    Label.TextSize = 13
    Label.Font = Enum.Font.Gotham
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = ToggleFrame
    
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0, 70, 0, 30)
    Button.Position = UDim2.new(1, -70, 0.5, -15)
    Button.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
    Button.Text = "OFF"
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextSize = 12
    Button.Font = Enum.Font.GothamBold
    Button.BorderSizePixel = 0
    Button.AutoButtonColor = false
    Button.Parent = ToggleFrame
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 6)
    ButtonCorner.Parent = Button
    
    local enabled = false
    
    Button.MouseButton1Click:Connect(function()
        enabled = not enabled
        if enabled then
            Button.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
            Button.Text = "ON"
        else
            Button.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
            Button.Text = "OFF"
        end
        callback(enabled)
    end)
    
    return Button
end

-- Função para criar botão normal
function CreateButton(parent, text, yPosition, callback)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(1, -20, 0, 40)
    Button.Position = UDim2.new(0, 10, 0, yPosition)
    Button.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    Button.Text = text
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextSize = 13
    Button.Font = Enum.Font.Gotham
    Button.BorderSizePixel = 0
    Button.AutoButtonColor = false
    Button.Parent = parent
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 6)
    ButtonCorner.Parent = Button
    
    Button.MouseEnter:Connect(function()
        Button.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
    end)
    
    Button.MouseLeave:Connect(function()
        Button.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    end)
    
    Button.MouseButton1Click:Connect(callback)
    
    return Button
end

-- Função para criar Dropdown
function CreateDropdown(parent, options, yPosition, callback)
    local DropdownFrame = Instance.new("Frame")
    DropdownFrame.Size = UDim2.new(1, -20, 0, 40)
    DropdownFrame.Position = UDim2.new(0, 10, 0, yPosition)
    DropdownFrame.BackgroundTransparency = 1
    DropdownFrame.Parent = parent
    
    local MainButton = Instance.new("TextButton")
    MainButton.Size = UDim2.new(1, 0, 1, 0)
    MainButton.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    MainButton.Text = options[1] or "Selecionar..."
    MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MainButton.TextSize = 13
    MainButton.Font = Enum.Font.Gotham
    MainButton.BorderSizePixel = 0
    MainButton.AutoButtonColor = false
    MainButton.Parent = DropdownFrame
    
    local MainCorner = Instance.new("UICorner")
    MainCorner.CornerRadius = UDim.new(0, 6)
    MainCorner.Parent = MainButton
    
    local DropList = Instance.new("Frame")
    DropList.Size = UDim2.new(1, 0, 0, #options * 35)
    DropList.Position = UDim2.new(0, 0, 1, 5)
    DropList.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    DropList.BorderSizePixel = 0
    DropList.Visible = false
    DropList.ZIndex = 5
    DropList.Parent = DropdownFrame
    
    local DropCorner = Instance.new("UICorner")
    DropCorner.CornerRadius = UDim.new(0, 6)
    DropCorner.Parent = DropList
    
    local DropLayout = Instance.new("UIListLayout")
    DropLayout.Parent = DropList
    
    for i, option in ipairs(options) do
        local OptionButton = Instance.new("TextButton")
        OptionButton.Size = UDim2.new(1, 0, 0, 35)
        OptionButton.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
        OptionButton.Text = option
        OptionButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        OptionButton.TextSize = 12
        OptionButton.Font = Enum.Font.Gotham
        OptionButton.BorderSizePixel = 0
        OptionButton.ZIndex = 6
        OptionButton.Parent = DropList
        
        OptionButton.MouseEnter:Connect(function()
            OptionButton.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
        end)
        
        OptionButton.MouseLeave:Connect(function()
            OptionButton.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
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

-- ===================== SEÇÃO 1: AUTO DEFESA =====================
local DefenseSection = CreateSection("🛡️ Auto Defesa", 100)

CreateToggle(DefenseSection, "Defesa Automática M1", 45, function(enabled)
    Config.AutoDefense = enabled
    if enabled then
        print("[Auto Defesa] Ativada")
    else
        print("[Auto Defesa] Desativada")
    end
end)

-- ===================== SEÇÃO 2: VISUALIZAÇÃO DE HITBOX =====================
local HitboxSection = CreateSection("📦 Visualização de Hitbox", 100)

CreateToggle(HitboxSection, "Mostrar Hitboxes", 45, function(enabled)
    Config.ShowHitboxes = enabled
    if enabled then
        print("[Hitbox] Visualização ativada")
    else
        print("[Hitbox] Visualização desativada")
        -- Remover todas as hitboxes visuais
        for _, part in ipairs(Workspace:GetDescendants()) do
            if part.Name == "HitboxVisual" then
                part:Destroy()
            end
        end
    end
end)

-- ===================== SEÇÃO 3: GERENCIADOR DE BOSSES =====================
local BossSection = CreateSection("👑 Gerenciador de Bosses", 220)

local BossListLabel = Instance.new("TextLabel")
BossListLabel.Size = UDim2.new(1, -20, 0, 20)
BossListLabel.Position = UDim2.new(0, 10, 0, 45)
BossListLabel.BackgroundTransparency = 1
BossListLabel.Text = "Bosses encontrados:"
BossListLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
BossListLabel.TextSize = 12
BossListLabel.Font = Enum.Font.GothamBold
BossListLabel.TextXAlignment = Enum.TextXAlignment.Left
BossListLabel.Parent = BossSection

local BossListFrame = Instance.new("ScrollingFrame")
BossListFrame.Size = UDim2.new(1, -20, 0, 80)
BossListFrame.Position = UDim2.new(0, 10, 0, 70)
BossListFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
BossListFrame.BorderSizePixel = 0
BossListFrame.ScrollBarThickness = 3
BossListFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
BossListFrame.Parent = BossSection

local BossListCorner = Instance.new("UICorner")
BossListCorner.CornerRadius = UDim.new(0, 4)
BossListCorner.Parent = BossListFrame

local BossListLayout = Instance.new("UIListLayout")
BossListLayout.Padding = UDim.new(0, 2)
BossListLayout.Parent = BossListFrame

local SelectedBossLabel = Instance.new("TextLabel")
SelectedBossLabel.Size = UDim2.new(1, -20, 0, 20)
SelectedBossLabel.Position = UDim2.new(0, 10, 0, 160)
SelectedBossLabel.BackgroundTransparency = 1
SelectedBossLabel.Text = "Nenhum boss selecionado"
SelectedBossLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
SelectedBossLabel.TextSize = 11
SelectedBossLabel.Font = Enum.Font.Gotham
SelectedBossLabel.TextXAlignment = Enum.TextXAlignment.Left
SelectedBossLabel.Parent = BossSection

CreateButton(BossSection, "🔄 Atualizar Lista", 190, function()
    UpdateBossList()
end)

-- ===================== SEÇÃO 4: COMBATE =====================
local CombatSection = CreateSection("⚔️ Sistema de Combate", 140)

CreateButton(CombatSection, "🎯 Ir até Boss Selecionado", 45, function()
    MoveToTargetBoss()
end)

-- ===================== SEÇÃO 5: POSICIONAMENTO =====================
local PositionSection = CreateSection("📍 Posicionamento Estratégico", 100)

local positions = {"Acima", "Abaixo", "Atrás", "Frente"}
CreateDropdown(PositionSection, positions, 45, function(selected)
    Config.CombatPosition = selected
    print("[Posicionamento] Definido para: " .. selected)
end)

-- ===================== FUNÇÕES DO SISTEMA =====================

-- Função para encontrar bosses
function FindBosses()
    local bosses = {}
    
    -- Procurar em todo o workspace
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") then
            local humanoid = obj.Humanoid
            if humanoid.Health > 0 then
                -- Verificar se é um boss por nome ou características
                local name = obj.Name:lower()
                if name:find("boss") or name:find("menos") or name:find("vasto") or 
                   name:find("arrancar") or name:find("hollow") and obj:FindFirstChild("Boss") then
                    table.insert(bosses, obj)
                end
            end
        end
    end
    
    -- Também procurar em pastas específicas
    local folders = {"Bosses", "Enemies", "Monsters", "NPCs"}
    for _, folderName in ipairs(folders) do
        local folder = Workspace:FindFirstChild(folderName)
        if folder then
            for _, obj in ipairs(folder:GetChildren()) do
                if obj:IsA("Model") and obj:FindFirstChild("Humanoid") then
                    local humanoid = obj.Humanoid
                    if humanoid.Health > 0 then
                        table.insert(bosses, obj)
                    end
                end
            end
        end
    end
    
    return bosses
end

-- Atualizar lista de bosses
function UpdateBossList()
    -- Limpar lista atual
    for _, child in ipairs(BossListFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    local bosses = FindBosses()
    
    if #bosses == 0 then
        local NoBossLabel = Instance.new("TextLabel")
        NoBossLabel.Size = UDim2.new(1, 0, 0, 25)
        NoBossLabel.BackgroundTransparency = 1
        NoBossLabel.Text = "Nenhum boss encontrado"
        NoBossLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
        NoBossLabel.TextSize = 12
        NoBossLabel.Font = Enum.Font.Gotham
        NoBossLabel.Parent = BossListFrame
    else
        for _, boss in ipairs(bosses) do
            local humanoid = boss:FindFirstChild("Humanoid")
            local health = humanoid and math.floor(humanoid.Health) or 0
            local maxHealth = humanoid and math.floor(humanoid.MaxHealth) or 0
            
            local BossButton = Instance.new("TextButton")
            BossButton.Size = UDim2.new(1, -4, 0, 25)
            BossButton.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
            BossButton.Text = boss.Name .. " [❤️ " .. health .. "/" .. maxHealth .. "]"
            BossButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            BossButton.TextSize = 10
            BossButton.Font = Enum.Font.Gotham
            BossButton.BorderSizePixel = 0
            BossButton.AutoButtonColor = false
            BossButton.Parent = BossListFrame
            
            BossButton.MouseButton1Click:Connect(function()
                Config.TargetBoss = boss
                SelectedBossLabel.Text = "Selecionado: " .. boss.Name
                
                -- Destacar botão selecionado
                for _, btn in ipairs(BossListFrame:GetChildren()) do
                    if btn:IsA("TextButton") then
                        btn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
                    end
                end
                BossButton.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
            end)
        end
    end
    
    -- Atualizar tamanho do canvas
    BossListFrame.CanvasSize = UDim2.new(0, 0, 0, #bosses * 27)
end

-- Função para mover até o boss
function MoveToTargetBoss()
    if not Config.TargetBoss then
        print("[Erro] Nenhum boss selecionado!")
        return
    end
    
    local character = Player.Character
    if not character then
        print("[Erro] Personagem não encontrado!")
        return
    end
    
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then
        print("[Erro] HumanoidRootPart não encontrada!")
        return
    end
    
    local bossRootPart = Config.TargetBoss:FindFirstChild("HumanoidRootPart")
    if not bossRootPart then
        print("[Erro] Boss não tem HumanoidRootPart!")
        return
    end
    
    -- Calcular posição baseada na configuração
    local bossPosition = bossRootPart.Position
    local offset = Vector3.new(0, 0, 0)
    local distance = Config.AttackDistance
    
    if Config.CombatPosition == "Acima" then
        offset = Vector3.new(0, distance, 0)
    elseif Config.CombatPosition == "Abaixo" then
        offset = Vector3.new(0, -distance, 0)
    elseif Config.CombatPosition == "Atrás" then
        local lookVector = bossRootPart.CFrame.LookVector
        offset = -lookVector * distance
    elseif Config.CombatPosition == "Frente" then
        local lookVector = bossRootPart.CFrame.LookVector
        offset = lookVector * distance
    end
    
    local targetPosition = bossPosition + offset
    
    -- Movimento suave
    local tweenInfo = TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(humanoidRootPart, tweenInfo, {CFrame = CFrame.new(targetPosition)})
    tween:Play()
    
    print("[Movimento] Indo para " .. Config.CombatPosition .. " do boss " .. Config.TargetBoss.Name)
end

-- Função para criar/atualizar hitboxes visuais
function UpdateHitboxVisuals()
    local enemies = {}
    
    -- Encontrar todos os inimigos
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") then
            local humanoid = obj.Humanoid
            if humanoid.Health > 0 and obj ~= Character then
                table.insert(enemies, obj)
            end
        end
    end
    
    -- Criar/atualizar hitboxes
    for _, enemy in ipairs(enemies) do
        local humanoidRootPart = enemy:FindFirstChild("HumanoidRootPart")
        if humanoidRootPart then
            local hitbox = enemy:FindFirstChild("HitboxVisual")
            
            if Config.ShowHitboxes then
                if not hitbox then
                    -- Criar hitbox visual
                    hitbox = Instance.new("Part")
                    hitbox.Name = "HitboxVisual"
                    hitbox.Size = Vector3.new(5, 6, 5)
                    hitbox.Color = Config.HitboxColor
                    hitbox.Transparency = Config.HitboxTransparency
                    hitbox.Material = Enum.Material.Neon
                    hitbox.Anchored = true
                    hitbox.CanCollide = false
                    hitbox.CanQuery = false
                    hitbox.Parent = enemy
                end
                
                -- Atualizar posição
                hitbox.Position = humanoidRootPart.Position
            else
                -- Remover hitbox se existir
                if hitbox then
                    hitbox:Destroy()
                end
            end
        end
    end
end

-- Sistema de Auto Defesa
function AutoDefenseSystem()
    if not Config.AutoDefense then return end
    
    local character = Player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    -- Verificar inimigos próximos
    local enemies = {}
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj ~= character then
            local enemyHumanoid = obj.Humanoid
            if enemyHumanoid.Health > 0 then
                local distance = (character:GetPivot().Position - obj:GetPivot().Position).Magnitude
                if distance < 20 then -- Raio de detecção
                    table.insert(enemies, obj)
                end
            end
        end
    end
    
    -- Tentar defender se houver ataque próximo
    for _, enemy in ipairs(enemies) do
        local enemyHumanoid = enemy:FindFirstChild("Humanoid")
        if enemyHumanoid then
            -- Verificar se inimigo está atacando
            local animator = enemyHumanoid:FindFirstChild("Animator")
            if animator then
                for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                    local animName = track.Animation.AnimationId:lower()
                    if animName:find("attack") or animName:find("m1") or animName:find("hit") then
                        -- Executar defesa
                        pcall(function()
                            local combatRemote = ReplicatedStorage:FindFirstChild("Remotes")
                            if combatRemote then
                                local combat = combatRemote:FindFirstChild("Combat")
                                if combat then
                                    combat:FireServer("Defend", true)
                                    print("[Auto Defesa] Defesa executada!")
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

-- Sistema de arraste da interface
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

UserInputService.InputChanged:Connect(function(input)
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

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Fechar interface
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Loop principal de atualização
spawn(function()
    while wait(Config.UpdateInterval) do
        pcall(function()
            -- Atualizar lista de bosses
            UpdateBossList()
            
            -- Atualizar hitboxes visuais
            UpdateHitboxVisuals()
            
            -- Executar auto defesa
            AutoDefenseSystem()
        end)
    end
end)

-- Atualizar quando personagem mudar
Player.CharacterAdded:Connect(function(newCharacter)
    Character = newCharacter
    print("[Sistema] Novo personagem detectado")
end)

print("✅ VV Ultimatum UI carregada com sucesso!")
print("📋 Comandos disponíveis:")
print("   - Auto Defesa: Proteção automática contra M1")
print("   - Hitbox Visual: Mostra área de ataque dos inimigos")
print("   - Gerenciador de Bosses: Lista bosses do servidor")
print("   - Sistema de Combate: Teleporte até bosses")
print("   - Posicionamento: Escolha onde ficar na luta")

end)

-- Tratamento de erro
if not success then
    warn("❌ Erro ao carregar script: " .. tostring(errorMsg))
end
