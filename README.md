--[[ VV Ultimatum - Ultra Otimizado v4.0 ]]
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

-- Configurações simplificadas
local Config = {
    AutoParry = false,
    HitboxESP = false,
    AutoFarm = false,
    AutoAttack = false,
    TargetBoss = nil,
    Position = "Frente",
    SafeDistance = 20, -- Distância segura do boss
    AttackDistance = 15, -- Distância para atacar
    LastUpdate = 0,
    Hitboxes = {}
}

-- Função super otimizada para encontrar mobs/bosses
local function FindTargets(nameFilter)
    local targets = {}
    for _, obj in ipairs(Services.Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
            local hum = obj.Humanoid
            if hum.Health > 0 and not Services.Players:GetPlayerFromCharacter(obj) then
                local n = obj.Name:lower()
                if nameFilter == "mobs" then
                    if n:find("hollow") or n:find("menos") or n:find("arrancar") or n:find("vasto") or n:find("mob") then
                        table.insert(targets, obj)
                    end
                elseif nameFilter == "boss" then
                    if (n:find("giant") and n:find("dragonfly")) or n:find("boss") or obj:GetAttribute("IsBoss") then
                        table.insert(targets, {Model = obj, Humanoid = hum, RootPart = obj.HumanoidRootPart, Name = obj.Name, Health = hum.Health, MaxHealth = hum.MaxHealth})
                    end
                end
            end
        end
    end
    return targets
end

-- Auto Parry otimizado
local function AutoParry()
    if not Config.AutoParry then return end
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    local now = tick()
    if now - Config.LastUpdate < 0.2 then return end
    Config.LastUpdate = now
    
    local charPos = char.HumanoidRootPart.Position
    local mobs = FindTargets("mobs")
    
    for _, mob in ipairs(mobs) do
        local root = mob:FindFirstChild("HumanoidRootPart")
        if root and (root.Position - charPos).Magnitude < 20 then
            local anim = mob:FindFirstChild("Humanoid"):FindFirstChild("Animator")
            if anim then
                for _, track in ipairs(anim:GetPlayingAnimationTracks()) do
                    if track.Animation.AnimationId:lower():find("attack") then
                        pcall(function()
                            local r = Services.ReplicatedStorage:FindFirstChild("Remotes")
                            if r then
                                local c = r:FindFirstChild("Combat") or r:FindFirstChild("Parry")
                                if c then c:FireServer("Parry") end
                            end
                        end)
                        return
                    end
                end
            end
        end
    end
end

-- Hitbox ESP otimizado (apenas cria, não fica recriando)
local function UpdateESP()
    if not Config.HitboxESP then
        for _, hb in ipairs(Config.Hitboxes) do
            if hb and hb.Parent then hb:Destroy() end
        end
        Config.Hitboxes = {}
        return
    end
    
    local mobs = FindTargets("mobs")
    
    -- Remover hitboxes de mobs mortos
    for i = #Config.Hitboxes, 1, -1 do
        local hb = Config.Hitboxes[i]
        if hb and hb.Parent then
            local found = false
            for _, mob in ipairs(mobs) do
                if mob == hb.Parent then found = true; break end
            end
            if not found then
                hb:Destroy()
                table.remove(Config.Hitboxes, i)
            end
        else
            table.remove(Config.Hitboxes, i)
        end
    end
    
    -- Criar/atualizar hitboxes
    for _, mob in ipairs(mobs) do
        local hb = mob:FindFirstChild("ESPHitbox")
        if not hb then
            hb = Instance.new("Part")
            hb.Name = "ESPHitbox"
            hb.Size = Vector3.new(4, 5, 4)
            hb.Color = Color3.fromRGB(255, 50, 50)
            hb.Transparency = 0.5
            hb.Material = Enum.Material.Neon
            hb.Anchored = true
            hb.CanCollide = false
            hb.CanQuery = false
            hb.Parent = mob
            table.insert(Config.Hitboxes, hb)
        end
        local root = mob:FindFirstChild("HumanoidRootPart")
        if root then hb.Position = root.Position end
    end
end

-- Auto Farm Boss ultra otimizado
local function AutoFarm()
    if not Config.AutoFarm or not Config.TargetBoss then return end
    
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    local boss = Config.TargetBoss
    local bossRoot = boss.Model:FindFirstChild("HumanoidRootPart")
    if not bossRoot or boss.Humanoid.Health <= 0 then return end
    
    local hrp = char.HumanoidRootPart
    local bossPos = bossRoot.Position
    
    -- Calcular posição segura
    local offset = Vector3.new(0, 0, 0)
    local dist = Config.SafeDistance
    
    if Config.Position == "Frente" then
        offset = bossRoot.CFrame.LookVector * dist
    elseif Config.Position == "Atrás" then
        offset = -bossRoot.CFrame.LookVector * dist
    elseif Config.Position == "Acima" then
        offset = Vector3.new(0, dist, 0)
    elseif Config.Position == "Abaixo" then
        offset = Vector3.new(0, -dist, 0)
    end
    
    local targetPos = bossPos + offset
    
    -- Teleporte apenas se muito longe
    if (hrp.Position - targetPos).Magnitude > 5 then
        hrp.CFrame = CFrame.new(targetPos)
    end
    
    -- Auto ataque
    if Config.AutoAttack then
        local distToBoss = (hrp.Position - bossPos).Magnitude
        if distToBoss <= Config.AttackDistance then
            pcall(function()
                local r = Services.ReplicatedStorage:FindFirstChild("Remotes")
                if r then
                    local c = r:FindFirstChild("Combat") or r:FindFirstChild("Attack")
                    if c then c:FireServer("M1") end
                end
            end)
        end
    end
end

-- =========== INTERFACE SUPER SIMPLES ===========
local SG = Instance.new("ScreenGui")
SG.Name = "VVUI"
SG.Parent = PlayerGui

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 380, 0, 480)
Main.Position = UDim2.new(0.5, -190, 0.2, 0)
Main.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
Main.BorderSizePixel = 0
Main.Parent = SG

Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)

-- Título
local Title = Instance.new("Frame")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
Title.BorderSizePixel = 0
Title.Parent = Main
Instance.new("UICorner", Title).CornerRadius = UDim.new(0, 8)

local TitleText = Instance.new("TextLabel")
TitleText.Size = UDim2.new(1, -40, 1, 0)
TitleText.Position = UDim2.new(0, 15, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "VV Ultimatum Lite"
TitleText.TextColor3 = Color3.fromRGB(0, 200, 255)
TitleText.TextSize = 14
TitleText.Font = Enum.Font.GothamBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.Parent = Title

local Close = Instance.new("TextButton")
Close.Size = UDim2.new(0, 25, 0, 25)
Close.Position = UDim2.new(1, -30, 0, 5)
Close.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
Close.Text = "X"
Close.TextColor3 = Color3.fromRGB(255, 255, 255)
Close.TextSize = 14
Close.Font = Enum.Font.GothamBold
Close.BorderSizePixel = 0
Close.Parent = Title
Instance.new("UICorner", Close).CornerRadius = UDim.new(0, 12)

-- Scroll
local Scroll = Instance.new("ScrollingFrame")
Scroll.Size = UDim2.new(1, -5, 1, -40)
Scroll.Position = UDim2.new(0, 3, 0, 40)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.ScrollBarThickness = 3
Scroll.CanvasSize = UDim2.new(0, 0, 0, 800)
Scroll.Parent = Main

local List = Instance.new("UIListLayout")
List.Padding = UDim.new(0, 5)
List.Parent = Scroll

-- Função para criar toggle simplificado
local function AddToggle(text, y, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, -10, 0, 35)
    Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 42)
    Frame.BorderSizePixel = 0
    Frame.Parent = Scroll
    Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 5)
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(255, 255, 255)
    Label.TextSize = 12
    Label.Font = Enum.Font.Gotham
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame
    
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 55, 0, 24)
    Btn.Position = UDim2.new(1, -60, 0.5, -12)
    Btn.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
    Btn.Text = "OFF"
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.TextSize = 11
    Btn.Font = Enum.Font.GothamBold
    Btn.BorderSizePixel = 0
    Btn.AutoButtonColor = false
    Btn.Parent = Frame
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 12)
    
    local on = false
    Btn.MouseButton1Click:Connect(function()
        on = not on
        Btn.BackgroundColor3 = on and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(80, 80, 90)
        Btn.Text = on and "ON" or "OFF"
        callback(on)
    end)
end

-- Função para criar botão
local function AddButton(text, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -10, 0, 35)
    Btn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    Btn.Text = text
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.TextSize = 12
    Btn.Font = Enum.Font.Gotham
    Btn.BorderSizePixel = 0
    Btn.AutoButtonColor = false
    Btn.Parent = Scroll
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 5)
    
    Btn.MouseEnter:Connect(function() Btn.BackgroundColor3 = Color3.fromRGB(60, 60, 75) end)
    Btn.MouseLeave:Connect(function() Btn.BackgroundColor3 = Color3.fromRGB(40, 40, 55) end)
    Btn.MouseButton1Click:Connect(callback)
end

-- Seção de título
local function AddSection(text)
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -10, 0, 25)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(0, 200, 255)
    Label.TextSize = 13
    Label.Font = Enum.Font.GothamBold
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Scroll
end

-- Construir UI
AddSection("⚔️ COMBATE")
AddToggle("Auto Parry (Defesa M1)", 0, function(v) Config.AutoParry = v end)
AddToggle("ESP Hitboxes (Mobs)", 0, function(v) Config.HitboxESP = v end)

AddSection("👑 BOSS FARM")
AddToggle("Auto Farm Boss", 0, function(v) Config.AutoFarm = v end)
AddToggle("Auto Atacar Boss", 0, function(v) Config.AutoAttack = v end)

-- Dropdown posição
local PosFrame = Instance.new("Frame")
PosFrame.Size = UDim2.new(1, -10, 0, 35)
PosFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 42)
PosFrame.BorderSizePixel = 0
PosFrame.Parent = Scroll
Instance.new("UICorner", PosFrame).CornerRadius = UDim.new(0, 5)

local PosLabel = Instance.new("TextLabel")
PosLabel.Size = UDim2.new(0.5, 0, 1, 0)
PosLabel.Position = UDim2.new(0, 10, 0, 0)
PosLabel.BackgroundTransparency = 1
PosLabel.Text = "Posição:"
PosLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
PosLabel.TextSize = 12
PosLabel.Font = Enum.Font.Gotham
PosLabel.TextXAlignment = Enum.TextXAlignment.Left
PosLabel.Parent = PosFrame

local PosBtn = Instance.new("TextButton")
PosBtn.Size = UDim2.new(0, 100, 0, 26)
PosBtn.Position = UDim2.new(1, -105, 0.5, -13)
PosBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
PosBtn.Text = "Frente"
PosBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
PosBtn.TextSize = 11
PosBtn.Font = Enum.Font.Gotham
PosBtn.BorderSizePixel = 0
PosBtn.AutoButtonColor = false
PosBtn.Parent = PosFrame
Instance.new("UICorner", PosBtn).CornerRadius = UDim.new(0, 5)

local PosList = Instance.new("Frame")
PosList.Size = UDim2.new(1, 0, 0, 140)
PosList.Position = UDim2.new(0, 0, 1, 3)
PosList.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
PosList.BorderSizePixel = 0
PosList.Visible = false
PosList.ZIndex = 10
PosList.Parent = PosFrame
Instance.new("UICorner", PosList).CornerRadius = UDim.new(0, 5)

for _, pos in ipairs({"Frente", "Atrás", "Acima", "Abaixo"}) do
    local Opt = Instance.new("TextButton")
    Opt.Size = UDim2.new(1, 0, 0, 35)
    Opt.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
    Opt.Text = pos
    Opt.TextColor3 = Color3.fromRGB(255, 255, 255)
    Opt.TextSize = 12
    Opt.Font = Enum.Font.Gotham
    Opt.BorderSizePixel = 0
    Opt.ZIndex = 11
    Opt.Parent = PosList
    
    Opt.MouseEnter:Connect(function() Opt.BackgroundColor3 = Color3.fromRGB(70, 70, 85) end)
    Opt.MouseLeave:Connect(function() Opt.BackgroundColor3 = Color3.fromRGB(50, 50, 65) end)
    Opt.MouseButton1Click:Connect(function()
        Config.Position = pos
        PosBtn.Text = pos
        PosList.Visible = false
    end)
end

PosBtn.MouseButton1Click:Connect(function()
    PosList.Visible = not PosList.Visible
end)

-- Lista de bosses
AddSection("📋 BOSSES ENCONTRADOS")

local BossScroll = Instance.new("ScrollingFrame")
BossScroll.Size = UDim2.new(1, -10, 0, 120)
BossScroll.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
BossScroll.BorderSizePixel = 0
BossScroll.ScrollBarThickness = 2
BossScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
BossScroll.Parent = Scroll
Instance.new("UICorner", BossScroll).CornerRadius = UDim.new(0, 5)

local BossList = Instance.new("UIListLayout")
BossList.Padding = UDim.new(0, 2)
BossList.Parent = BossScroll

AddButton("🔄 Atualizar Lista de Bosses", function()
    for _, v in ipairs(BossScroll:GetChildren()) do
        if v:IsA("TextButton") then v:Destroy() end
    end
    
    local bosses = FindTargets("boss")
    for _, boss in ipairs(bosses) do
        local Btn = Instance.new("TextButton")
        Btn.Size = UDim2.new(1, -4, 0, 25)
        Btn.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
        Btn.Text = string.format("%s [%d%%]", boss.Name, math.floor(boss.Health/boss.MaxHealth*100))
        Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        Btn.TextSize = 10
        Btn.Font = Enum.Font.Gotham
        Btn.BorderSizePixel = 0
        Btn.AutoButtonColor = false
        Btn.Parent = BossScroll
        
        Btn.MouseButton1Click:Connect(function()
            Config.TargetBoss = boss
            for _, b in ipairs(BossScroll:GetChildren()) do
                if b:IsA("TextButton") then b.BackgroundColor3 = Color3.fromRGB(45, 45, 60) end
            end
            Btn.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
        end)
    end
    
    BossScroll.CanvasSize = UDim2.new(0, 0, 0, #bosses * 27)
end)

-- Sistema de arraste
local drag, start, pos = false, nil, nil
Title.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        drag, start, pos = true, i.Position, Main.Position
    end
end)

Services.UserInputService.InputChanged:Connect(function(i)
    if drag and i.UserInputType == Enum.UserInputType.MouseMovement then
        local d = i.Position - start
        Main.Position = UDim2.new(pos.X.Scale, pos.X.Offset + d.X, pos.Y.Scale, pos.Y.Offset + d.Y)
    end
end)

Services.UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
end)

Close.MouseButton1Click:Connect(function() SG:Destroy() end)

-- Loop principal ultra otimizado
Services.RunService.Heartbeat:Connect(function()
    pcall(AutoParry)
    pcall(UpdateESP)
    pcall(AutoFarm)
end)

print("✅ VV Ultimatum Lite - Pronto!")
print("📌 Dica: Fique a 20 studs do boss para não tomar dano!")
