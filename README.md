--[[
    VV Ultimatum - Interface Avançada
    Versão: 1.0.0
    Desenvolvido para auxiliar jogadores com funções essenciais
]]

-- Configurações principais
local Settings = {
    Window = {
        Title = "VV Ultimatum - Interface Avançada",
        Width = 420,
        Height = 500,
        Position = UDim2.new(0.5, -210, 0.3, 0),
        Draggable = true,
        Color = Color3.fromRGB(25, 25, 35),
        AccentColor = Color3.fromRGB(0, 200, 255)
    },
    AutoDefense = {
        Enabled = false,
        ResponseTime = 0.1, -- Tempo de resposta em segundos
        Keybind = "F"
    },
    HitboxVisualizer = {
        Enabled = false,
        Color = Color3.fromRGB(255, 0, 0),
        Transparency = 0.5,
        Thickness = 0.1
    },
    BossManager = {
        BossList = {},
        SelectedBoss = nil,
        UpdateInterval = 2 -- Atualiza a cada 2 segundos
    },
    CombatSystem = {
        TargetBoss = nil,
        Speed = 50,
        Positions = {
            "Acima",
            "Abaixo", 
            "Atrás",
            "Frente"
        },
        CurrentPosition = "Acima"
    }
}

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

-- Variáveis
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Criar a interface principal
local function CreateUI()
    -- ScreenGui principal
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "VVUltimatumUI"
    ScreenGui.Parent = PlayerGui
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Container principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, Settings.Window.Width, 0, Settings.Window.Height)
    MainFrame.Position = Settings.Window.Position
    MainFrame.BackgroundColor3 = Settings.Window.Color
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Parent = ScreenGui
    
    -- Arredondar cantos
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 8)
    UICorner.Parent = MainFrame
    
    -- Barra de título
    local TitleBar = Instance.new("Frame")
    TitleBar.Name = "TitleBar"
    TitleBar.Size = UDim2.new(1, 0, 0, 35)
    TitleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame
    
    local TitleCorner = Instance.new("UICorner")
    TitleCorner.CornerRadius = UDim.new(0, 8)
    TitleCorner.Parent = TitleBar
    
    -- Corrigir cantos inferiores
    local TitleBottom = Instance.new("Frame")
    TitleBottom.Size = UDim2.new(1, 0, 0, 8)
    TitleBottom.Position = UDim2.new(0, 0, 1, -8)
    TitleBottom.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    TitleBottom.BorderSizePixel = 0
    TitleBottom.Parent = TitleBar
    
    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Size = UDim2.new(1, -60, 1, 0)
    TitleLabel.Position = UDim2.new(0, 15, 0, 0)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Text = Settings.Window.Title
    TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    TitleLabel.TextSize = 14
    TitleLabel.Font = Enum.Font.GothamBold
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    TitleLabel.Parent = TitleBar
    
    -- Botão fechar
    local CloseButton = Instance.new("TextButton")
    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -35, 0, 3)
    CloseButton.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
    CloseButton.Text = "✕"
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.TextSize = 16
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.BorderSizePixel = 0
    CloseButton.Parent = TitleBar
    
    local CloseCorner = Instance.new("UICorner")
    CloseCorner.CornerRadius = UDim.new(0, 6)
    CloseCorner.Parent = CloseButton
    
    -- Container de conteúdo com scroll
    local ScrollingFrame = Instance.new("ScrollingFrame")
    ScrollingFrame.Size = UDim2.new(1, -10, 1, -45)
    ScrollingFrame.Position = UDim2.new(0, 5, 0, 40)
    ScrollingFrame.BackgroundTransparency = 1
    ScrollingFrame.BorderSizePixel = 0
    ScrollingFrame.ScrollBarThickness = 5
    ScrollingFrame.ScrollBarImageColor3 = Settings.Window.AccentColor
    ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 950)
    ScrollingFrame.Parent = MainFrame
    
    local ContentList = Instance.new("UIListLayout")
    ContentList.Padding = UDim.new(0, 8)
    ContentList.Parent = ScrollingFrame
    
    -- Função para criar seções
    local function CreateSection(title, description, height)
        local Section = Instance.new("Frame")
        Section.Size = UDim2.new(1, -10, 0, height or 150)
        Section.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
        Section.BorderSizePixel = 0
        Section.Parent = ScrollingFrame
        
        local SectionCorner = Instance.new("UICorner")
        SectionCorner.CornerRadius = UDim.new(0, 6)
        SectionCorner.Parent = Section
        
        local SectionTitle = Instance.new("TextLabel")
        SectionTitle.Size = UDim2.new(1, -20, 0, 25)
        SectionTitle.Position = UDim2.new(0, 10, 0, 8)
        SectionTitle.BackgroundTransparency = 1
        SectionTitle.Text = title
        SectionTitle.TextColor3 = Settings.Window.AccentColor
        SectionTitle.TextSize = 15
        SectionTitle.Font = Enum.Font.GothamBold
        SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
        SectionTitle.Parent = Section
        
        local SectionDesc = Instance.new("TextLabel")
        SectionDesc.Size = UDim2.new(1, -20, 0, 15)
        SectionDesc.Position = UDim2.new(0, 10, 0, 30)
        SectionDesc.BackgroundTransparency = 1
        SectionDesc.Text = description
        SectionDesc.TextColor3 = Color3.fromRGB(150, 150, 150)
        SectionDesc.TextSize = 10
        SectionDesc.Font = Enum.Font.Gotham
        SectionDesc.TextXAlignment = Enum.TextXAlignment.Left
        SectionDesc.Parent = Section
        
        return Section
    end
    
    -- Função para criar botões toggle
    local function CreateToggle(parent, text, position, callback)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(1, -20, 0, 35)
        ToggleFrame.Position = UDim2.new(0, 10, 0, position)
        ToggleFrame.BackgroundTransparency = 1
        ToggleFrame.Parent = parent
        
        local ToggleLabel = Instance.new("TextLabel")
        ToggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
        ToggleLabel.BackgroundTransparency = 1
        ToggleLabel.Text = text
        ToggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        ToggleLabel.TextSize = 13
        ToggleLabel.Font = Enum.Font.Gotham
        ToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
        ToggleLabel.Parent = ToggleFrame
        
        local ToggleButton = Instance.new("TextButton")
        ToggleButton.Size = UDim2.new(0, 60, 0, 24)
        ToggleButton.Position = UDim2.new(1, -60, 0.5, -12)
        ToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
        ToggleButton.Text = "DESATIVADO"
        ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        ToggleButton.TextSize = 10
        ToggleButton.Font = Enum.Font.GothamBold
        ToggleButton.BorderSizePixel = 0
        ToggleButton.AutoButtonColor = false
        ToggleButton.Parent = ToggleFrame
        
        local ToggleCorner = Instance.new("UICorner")
        ToggleCorner.CornerRadius = UDim.new(0, 6)
        ToggleCorner.Parent = ToggleButton
        
        local enabled = false
        
        ToggleButton.MouseButton1Click:Connect(function()
            enabled = not enabled
            if enabled then
                ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
                ToggleButton.Text = "ATIVADO"
            else
                ToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
                ToggleButton.Text = "DESATIVADO"
            end
            callback(enabled)
        end)
        
        return ToggleButton
    end
    
    -- Função para criar botões normais
    local function CreateButton(parent, text, position, callback)
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(1, -20, 0, 35)
        Button.Position = UDim2.new(0, 10, 0, position)
        Button.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
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
            Button.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
        end)
        
        Button.MouseLeave:Connect(function()
            Button.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
        end)
        
        Button.MouseButton1Click:Connect(callback)
        
        return Button
    end
    
    -- Função para criar dropdown
    local function CreateDropdown(parent, options, position, callback)
        local DropdownFrame = Instance.new("Frame")
        DropdownFrame.Size = UDim2.new(1, -20, 0, 35)
        DropdownFrame.Position = UDim2.new(0, 10, 0, position)
        DropdownFrame.BackgroundTransparency = 1
        DropdownFrame.Parent = parent
        
        local Dropdown = Instance.new("TextButton")
        Dropdown.Size = UDim2.new(1, 0, 1, 0)
        Dropdown.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
        Dropdown.Text = options[1] or "Selecionar..."
        Dropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
        Dropdown.TextSize = 13
        Dropdown.Font = Enum.Font.Gotham
        Dropdown.BorderSizePixel = 0
        Dropdown.AutoButtonColor = false
        Dropdown.Parent = DropdownFrame
        
        local DropCorner = Instance.new("UICorner")
        DropCorner.CornerRadius = UDim.new(0, 6)
        DropCorner.Parent = Dropdown
        
        local DropList = Instance.new("Frame")
        DropList.Size = UDim2.new(1, 0, 0, #options * 30)
        DropList.Position = UDim2.new(0, 0, 1, 5)
        DropList.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
        DropList.BorderSizePixel = 0
        DropList.Visible = false
        DropList.ZIndex = 2
        DropList.Parent = DropdownFrame
        
        local ListCorner = Instance.new("UICorner")
        ListCorner.CornerRadius = UDim.new(0, 6)
        ListCorner.Parent = DropList
        
        local ListLayout = Instance.new("UIListLayout")
        ListLayout.Parent = DropList
        
        for i, option in ipairs(options) do
            local OptionButton = Instance.new("TextButton")
            OptionButton.Size = UDim2.new(1, 0, 0, 30)
            OptionButton.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
            OptionButton.Text = option
            OptionButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            OptionButton.TextSize = 12
            OptionButton.Font = Enum.Font.Gotham
            OptionButton.BorderSizePixel = 0
            OptionButton.ZIndex = 3
            OptionButton.Parent = DropList
            
            OptionButton.MouseEnter:Connect(function()
                OptionButton.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
            end)
            
            OptionButton.MouseLeave:Connect(function()
                OptionButton.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
            end)
            
            OptionButton.MouseButton1Click:Connect(function()
                Dropdown.Text = option
                DropList.Visible = false
                callback(option)
            end)
        end
        
        Dropdown.MouseButton1Click:Connect(function()
            DropList.Visible = not DropList.Visible
        end)
        
        return Dropdown
    end
    
    -- === SEÇÃO 1: AUTO DEFESA ===
    local DefenseSection = CreateSection("🛡️ Auto Defesa", "Sistema automático de defesa contra ataques M1", 110)
    CreateToggle(DefenseSection, "Defesa Automática", 55, function(enabled)
        Settings.AutoDefense.Enabled = enabled
        if enabled then
            StartAutoDefense()
        else
            StopAutoDefense()
        end
    end)
    
    -- === SEÇÃO 2: VISUALIZAÇÃO DE HITBOX ===
    local HitboxSection = CreateSection("📦 Visualização de Hitbox", "Exibe as hitboxes dos ataques dos Hollows", 110)
    CreateToggle(HitboxSection, "Mostrar Hitboxes", 55, function(enabled)
        Settings.HitboxVisualizer.Enabled = enabled
        if enabled then
            StartHitboxVisualizer()
        else
            StopHitboxVisualizer()
        end
    end)
    
    -- === SEÇÃO 3: GERENCIADOR DE BOSSES ===
    local BossManagerSection = CreateSection("👑 Gerenciador de Bosses", "Localiza e lista todos os bosses no servidor", 200)
    
    local BossListFrame = Instance.new("Frame")
    BossListFrame.Size = UDim2.new(1, -20, 0, 80)
    BossListFrame.Position = UDim2.new(0, 10, 0, 55)
    BossListFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    BossListFrame.BorderSizePixel = 0
    BossListFrame.Parent = BossManagerSection
    
    local BossListCorner = Instance.new("UICorner")
    BossListCorner.CornerRadius = UDim.new(0, 4)
    BossListCorner.Parent = BossListFrame
    
    local BossListLabel = Instance.new("TextLabel")
    BossListLabel.Size = UDim2.new(1, -10, 0, 20)
    BossListLabel.Position = UDim2.new(0, 5, 0, 5)
    BossListLabel.BackgroundTransparency = 1
    BossListLabel.Text = "Bosses Encontrados:"
    BossListLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    BossListLabel.TextSize = 11
    BossListLabel.Font = Enum.Font.GothamBold
    BossListLabel.TextXAlignment = Enum.TextXAlignment.Left
    BossListLabel.Parent = BossListFrame
    
    local BossListScrolling = Instance.new("ScrollingFrame")
    BossListScrolling.Size = UDim2.new(1, -10, 1, -30)
    BossListScrolling.Position = UDim2.new(0, 5, 0, 25)
    BossListScrolling.BackgroundTransparency = 1
    BossListScrolling.BorderSizePixel = 0
    BossListScrolling.ScrollBarThickness = 3
    BossListScrolling.CanvasSize = UDim2.new(0, 0, 0, 0)
    BossListScrolling.Parent = BossListFrame
    
    local BossListLayout = Instance.new("UIListLayout")
    BossListLayout.Padding = UDim.new(0, 3)
    BossListLayout.Parent = BossListScrolling
    
    CreateButton(BossManagerSection, "Atualizar Lista", 145, function()
        UpdateBossList(BossListScrolling, BossListLayout)
    end)
    
    -- === SEÇÃO 4: SISTEMA DE COMBATE ===
    local CombatSection = CreateSection("⚔️ Sistema de Combate", "Combate automático contra bosses", 130)
    
    CreateButton(CombatSection, "Ir até Boss Selecionado", 55, function()
        if Settings.BossManager.SelectedBoss then
            MoveToBoss(Settings.BossManager.SelectedBoss)
        end
    end)
    
    -- === SEÇÃO 5: POSICIONAMENTO ===
    local PositionSection = CreateSection("📍 Posicionamento", "Escolha sua posição durante o combate", 100)
    
    local PositionDropdown = CreateDropdown(PositionSection, Settings.CombatSystem.Positions, 55, function(position)
        Settings.CombatSystem.CurrentPosition = position
        UpdateCombatPosition()
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
    
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)
    
    return ScreenGui
end

-- Funções de Auto Defesa
function StartAutoDefense()
    print("Auto Defesa ativada")
    -- Sistema de detecção e defesa
    spawn(function()
        while Settings.AutoDefense.Enabled do
            local character = Player.Character
            if character then
                local humanoid = character:FindFirstChild("Humanoid")
                if humanoid then
                    -- Lógica de defesa automática
                    local enemies = workspace:GetChildren()
                    for _, enemy in ipairs(enemies) do
                        if enemy:IsA("Model") and enemy:FindFirstChild("Humanoid") then
                            local enemyHumanoid = enemy.Humanoid
                            if enemyHumanoid.Health > 0 then
                                -- Verificar animação de ataque
                                local animator = enemyHumanoid:FindFirstChild("Animator")
                                if animator then
                                    local attackTrack = animator:FindFirstChild("Attack")
                                    if attackTrack and attackTrack.IsPlaying then
                                        -- Executar defesa
                                        pcall(function()
                                            local args = {
                                                [1] = "Defend",
                                                [2] = true
                                            }
                                            game:GetService("ReplicatedStorage"):FindFirstChild("Remotes"):FindFirstChild("Combat"):FireServer(unpack(args))
                                        end)
                                    end
                                end
                            end
                        end
                    end
                end
            end
            wait(Settings.AutoDefense.ResponseTime)
        end
    end)
end

function StopAutoDefense()
    print("Auto Defesa desativada")
end

-- Funções de Hitbox Visualizer
function StartHitboxVisualizer()
    print("Visualização de Hitbox ativada")
    spawn(function()
        while Settings.HitboxVisualizer.Enabled do
            -- Criar/atualizar visualização de hitboxes
            local hollows = workspace:GetChildren()
            for _, hollow in ipairs(hollows) do
                if hollow.Name:find("Hollow") or hollow.Name:find("Arrancar") then
                    local humanoid = hollow:FindFirstChild("Humanoid")
                    if humanoid and humanoid.Health > 0 then
                        -- Criar hitbox visual
                        local hitbox = hollow:FindFirstChild("VisualHitbox")
                        if not hitbox then
                            hitbox = Instance.new("Part")
                            hitbox.Name = "VisualHitbox"
                            hitbox.Size = Vector3.new(4, 5, 4)
                            hitbox.Transparency = Settings.HitboxVisualizer.Transparency
                            hitbox.Color = Settings.HitboxVisualizer.Color
                            hitbox.Material = Enum.Material.Neon
                            hitbox.Anchored = false
                            hitbox.CanCollide = false
                            hitbox.Parent = hollow
                        end
                        
                        -- Atualizar posição da hitbox
                        local rootPart = hollow:FindFirstChild("HumanoidRootPart")
                        if rootPart then
                            hitbox.Position = rootPart.Position
                        end
                    end
                end
            end
            wait(0.1)
        end
    end)
end

function StopHitboxVisualizer()
    print("Visualização de Hitbox desativada")
    -- Remover hitboxes visuais
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj.Name == "VisualHitbox" then
            obj:Destroy()
        end
    end
end

-- Funções do Gerenciador de Bosses
function UpdateBossList(scrollingFrame, layout)
    -- Limpar lista atual
    for _, child in ipairs(scrollingFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    Settings.BossManager.BossList = {}
    
    -- Procurar bosses
    local bosses = workspace:GetChildren()
    for _, boss in ipairs(bosses) do
        if boss:IsA("Model") and boss:FindFirstChild("Humanoid") then
            local humanoid = boss.Humanoid
            if humanoid.Health > 0 and (boss.Name:find("Boss") or boss.Name:find("Menos") or boss.Name:find("Vasto")) then
                table.insert(Settings.BossManager.BossList, boss)
                
                -- Criar botão para o boss
                local BossButton = Instance.new("TextButton")
                BossButton.Size = UDim2.new(1, 0, 0, 25)
                BossButton.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
                BossButton.Text = boss.Name .. " [❤️ " .. math.floor(humanoid.Health) .. "]"
                BossButton.TextColor3 = Color3.fromRGB(255, 255, 255)
                BossButton.TextSize = 10
                BossButton.Font = Enum.Font.Gotham
                BossButton.BorderSizePixel = 0
                BossButton.Parent = scrollingFrame
                
                BossButton.MouseButton1Click:Connect(function()
                    Settings.BossManager.SelectedBoss = boss
                    -- Destacar botão selecionado
                    for _, btn in ipairs(scrollingFrame:GetChildren()) do
                        if btn:IsA("TextButton") then
                            btn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
                        end
                    end
                    BossButton.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
                end)
            end
        end
    end
    
    -- Atualizar tamanho do canvas
    scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, #Settings.BossManager.BossList * 28)
end

-- Funções de Combate
function MoveToBoss(boss)
    if not boss or not boss:FindFirstChild("HumanoidRootPart") then return end
    
    local character = Player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local targetPosition = boss.HumanoidRootPart.Position
    local startPosition = character.HumanoidRootPart.Position
    
    -- Movimento suave usando Tween
    local distance = (targetPosition - startPosition).Magnitude
    local duration = distance / Settings.CombatSystem.Speed
    
    local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(character.HumanoidRootPart, tweenInfo, {CFrame = CFrame.new(targetPosition)})
    tween:Play()
    
    Settings.CombatSystem.TargetBoss = boss
end

function UpdateCombatPosition()
    local boss = Settings.CombatSystem.TargetBoss
    if not boss or not boss:FindFirstChild("HumanoidRootPart") then return end
    
    local character = Player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local bossPosition = boss.HumanoidRootPart.Position
    local offset = Vector3.new(0, 0, 0)
    local distance = 10
    
    if Settings.CombatSystem.CurrentPosition == "Acima" then
        offset = Vector3.new(0, distance, 0)
    elseif Settings.CombatSystem.CurrentPosition == "Abaixo" then
        offset = Vector3.new(0, -distance, 0)
    elseif Settings.CombatSystem.CurrentPosition == "Atrás" then
        offset = Vector3.new(0, 0, distance)
    elseif Settings.CombatSystem.CurrentPosition == "Frente" then
        offset = Vector3.new(0, 0, -distance)
    end
    
    local targetPosition = bossPosition + offset
    
    -- Manter posição automaticamente
    spawn(function()
        while Settings.CombatSystem.TargetBoss == boss do
            character.HumanoidRootPart.CFrame = CFrame.new(targetPosition)
            wait(0.1)
        end
    end)
end

-- Inicialização
local function Initialize()
    -- Esperar o personagem carregar
    if not Player.Character then
        Player.CharacterAdded:Wait()
    end
    
    -- Criar interface
    local UI = CreateUI()
    
    -- Iniciar atualização automática da lista de bosses
    spawn(function()
        while wait(Settings.BossManager.UpdateInterval) do
            local bossListFrame = UI.MainFrame:FindFirstChild("ScrollingFrame")
            if bossListFrame then
                local bossSection = bossListFrame:FindFirstChildWhichIsA("Frame")
                if bossSection then
                    local scrollingFrame = bossSection:FindFirstChildWhichIsA("ScrollingFrame")
                    local layout = scrollingFrame and scrollingFrame:FindFirstChildWhichIsA("UIListLayout")
                    if scrollingFrame and layout then
                        UpdateBossList(scrollingFrame, layout)
                    end
                end
            end
        end
    end)
    
    print("VV Ultimatum Interface carregada com sucesso!")
end

-- Iniciar script
Initialize()
