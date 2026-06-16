--[[
    Advanced Movement GUI - Roblox Script
    Features: Noclip, Speed Hack, Modern UI
    Author: System Generated
]]

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- Player Variables
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Script Settings
local Settings = {
    Noclip = false,
    SpeedBoost = false,
    SpeedValue = 16,
    DefaultSpeed = 16,
    Minimized = false,
    Dragging = false,
    DragStart = nil,
    StartPos = nil
}

-- Connection Variables
local Connections = {}
local NoclipConnection = nil
local SpeedConnection = nil

-- UI Creation
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MovementGUI"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BackgroundTransparency = 0.05
MainFrame.BorderSizePixel = 0
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -150)
MainFrame.Size = UDim2.new(0, 300, 0, 350)
MainFrame.ClipsDescendants = true

-- Round corners
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Shadow effect
local Shadow = Instance.new("ImageLabel")
Shadow.Name = "Shadow"
Shadow.Parent = MainFrame
Shadow.BackgroundTransparency = 1
Shadow.Position = UDim2.new(0, -5, 0, -5)
Shadow.Size = UDim2.new(1, 10, 1, 10)
Shadow.Image = "rbxassetid://6014261993"
Shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
Shadow.ImageTransparency = 0.5
Shadow.ZIndex = -1

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Parent = MainFrame
TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
TitleBar.BorderSizePixel = 0
TitleBar.Size = UDim2.new(1, 0, 0, 40)

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = TitleBar

-- Title Text
local TitleText = Instance.new("TextLabel")
TitleText.Name = "TitleText"
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Size = UDim2.new(0.7, 0, 1, 0)
TitleText.Position = UDim2.new(0, 15, 0, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "Movement GUI"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 18
TitleText.TextXAlignment = Enum.TextXAlignment.Left

-- Minimize Button
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Name = "MinimizeButton"
MinimizeButton.Parent = TitleBar
MinimizeButton.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
MinimizeButton.BorderSizePixel = 0
MinimizeButton.Position = UDim2.new(0.7, 5, 0.5, -12)
MinimizeButton.Size = UDim2.new(0, 24, 0, 24)
MinimizeButton.Text = "—"
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.TextSize = 18
MinimizeButton.Font = Enum.Font.GothamBold

local MinButtonCorner = Instance.new("UICorner")
MinButtonCorner.CornerRadius = UDim.new(0, 6)
MinButtonCorner.Parent = MinimizeButton

-- Close Button
local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Parent = TitleBar
CloseButton.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
CloseButton.BorderSizePixel = 0
CloseButton.Position = UDim2.new(0.85, 5, 0.5, -12)
CloseButton.Size = UDim2.new(0, 24, 0, 24)
CloseButton.Text = "✕"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 14
CloseButton.Font = Enum.Font.GothamBold

local CloseButtonCorner = Instance.new("UICorner")
CloseButtonCorner.CornerRadius = UDim.new(0, 6)
CloseButtonCorner.Parent = CloseButton

-- Content Frame
local ContentFrame = Instance.new("Frame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Parent = MainFrame
ContentFrame.BackgroundTransparency = 1
ContentFrame.BorderSizePixel = 0
ContentFrame.Position = UDim2.new(0, 0, 0, 45)
ContentFrame.Size = UDim2.new(1, 0, 1, -45)

-- Categories Container
local CategoriesFrame = Instance.new("Frame")
CategoriesFrame.Name = "CategoriesFrame"
CategoriesFrame.Parent = ContentFrame
CategoriesFrame.BackgroundTransparency = 1
CategoriesFrame.BorderSizePixel = 0
CategoriesFrame.Size = UDim2.new(1, -20, 1, -10)
CategoriesFrame.Position = UDim2.new(0, 10, 0, 5)

-- UIListLayout for scrolling
local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = CategoriesFrame
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 10)

-- Function to create category
local function CreateCategory(name, icon)
    local Category = Instance.new("Frame")
    Category.Name = name.."Category"
    Category.Parent = CategoriesFrame
    Category.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    Category.BackgroundTransparency = 0.3
    Category.BorderSizePixel = 0
    Category.Size = UDim2.new(1, 0, 0, 60)
    
    local CategoryCorner = Instance.new("UICorner")
    CategoryCorner.CornerRadius = UDim.new(0, 8)
    CategoryCorner.Parent = Category
    
    local CategoryTitle = Instance.new("TextLabel")
    CategoryTitle.Name = "CategoryTitle"
    CategoryTitle.Parent = Category
    CategoryTitle.BackgroundTransparency = 1
    CategoryTitle.Position = UDim2.new(0, 12, 0, 8)
    CategoryTitle.Size = UDim2.new(0.8, 0, 0, 20)
    CategoryTitle.Font = Enum.Font.GothamMedium
    CategoryTitle.Text = icon.." "..name
    CategoryTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
    CategoryTitle.TextSize = 14
    CategoryTitle.TextXAlignment = Enum.TextXAlignment.Left
    
    return Category
end

-- Noclip Category
local NoclipCategory = CreateCategory("Noclip", "🚀")
NoclipCategory.Size = UDim2.new(1, 0, 0, 70)

local NoclipToggle = Instance.new("TextButton")
NoclipToggle.Name = "NoclipToggle"
NoclipToggle.Parent = NoclipCategory
NoclipToggle.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
NoclipToggle.BorderSizePixel = 0
NoclipToggle.Position = UDim2.new(0, 12, 0, 35)
NoclipToggle.Size = UDim2.new(0, 50, 0, 24)
NoclipToggle.Text = "OFF"
NoclipToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
NoclipToggle.TextSize = 12
NoclipToggle.Font = Enum.Font.GothamBold
NoclipToggle.AutoButtonColor = false

local NoclipToggleCorner = Instance.new("UICorner")
NoclipToggleCorner.CornerRadius = UDim.new(0, 12)
NoclipToggleCorner.Parent = NoclipToggle

local NoclipLabel = Instance.new("TextLabel")
NoclipLabel.Name = "NoclipLabel"
NoclipLabel.Parent = NoclipCategory
NoclipLabel.BackgroundTransparency = 1
NoclipLabel.Position = UDim2.new(0, 70, 0, 35)
NoclipLabel.Size = UDim2.new(0.7, 0, 0, 24)
NoclipLabel.Font = Enum.Font.GothamMedium
NoclipLabel.Text = "Walk through walls"
NoclipLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
NoclipLabel.TextSize = 12
NoclipLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Speed Category
local SpeedCategory = CreateCategory("Movement", "⚡")
SpeedCategory.Size = UDim2.new(1, 0, 0, 140)

-- Speed Slider
local SpeedSliderFrame = Instance.new("Frame")
SpeedSliderFrame.Name = "SpeedSliderFrame"
SpeedSliderFrame.Parent = SpeedCategory
SpeedSliderFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
SpeedSliderFrame.BorderSizePixel = 0
SpeedSliderFrame.Position = UDim2.new(0, 12, 0, 35)
SpeedSliderFrame.Size = UDim2.new(0.6, 0, 0, 20)

local SliderCorner = Instance.new("UICorner")
SliderCorner.CornerRadius = UDim.new(0, 10)
SliderCorner.Parent = SpeedSliderFrame

local SpeedSliderFill = Instance.new("Frame")
SpeedSliderFill.Name = "SpeedSliderFill"
SpeedSliderFill.Parent = SpeedSliderFrame
SpeedSliderFill.BackgroundColor3 = Color3.fromRGB(52, 152, 219)
SpeedSliderFill.BorderSizePixel = 0
SpeedSliderFill.Size = UDim2.new(0.16, 0, 1, 0)

local FillCorner = Instance.new("UICorner")
FillCorner.CornerRadius = UDim.new(0, 10)
FillCorner.Parent = SpeedSliderFill

local SpeedValueLabel = Instance.new("TextLabel")
SpeedValueLabel.Name = "SpeedValueLabel"
SpeedValueLabel.Parent = SpeedCategory
SpeedValueLabel.BackgroundTransparency = 1
SpeedValueLabel.Position = UDim2.new(0.68, 0, 0, 35)
SpeedValueLabel.Size = UDim2.new(0.25, 0, 0, 20)
SpeedValueLabel.Font = Enum.Font.GothamBold
SpeedValueLabel.Text = "16"
SpeedValueLabel.TextColor3 = Color3.fromRGB(52, 152, 219)
SpeedValueLabel.TextSize = 14
SpeedValueLabel.TextXAlignment = Enum.TextXAlignment.Center

local SpeedMinLabel = Instance.new("TextLabel")
SpeedMinLabel.Name = "SpeedMinLabel"
SpeedMinLabel.Parent = SpeedCategory
SpeedMinLabel.BackgroundTransparency = 1
SpeedMinLabel.Position = UDim2.new(0, 12, 0, 55)
SpeedMinLabel.Size = UDim2.new(0, 30, 0, 15)
SpeedMinLabel.Font = Enum.Font.GothamMedium
SpeedMinLabel.Text = "1"
SpeedMinLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
SpeedMinLabel.TextSize = 10

local SpeedMaxLabel = Instance.new("TextLabel")
SpeedMaxLabel.Name = "SpeedMaxLabel"
SpeedMaxLabel.Parent = SpeedCategory
SpeedMaxLabel.BackgroundTransparency = 1
SpeedMaxLabel.Position = UDim2.new(0.57, 0, 0, 55)
SpeedMaxLabel.Size = UDim2.new(0, 30, 0, 15)
SpeedMaxLabel.Font = Enum.Font.GothamMedium
SpeedMaxLabel.Text = "200"
SpeedMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
SpeedMaxLabel.TextSize = 10

-- Speed Toggle
local SpeedToggle = Instance.new("TextButton")
SpeedToggle.Name = "SpeedToggle"
SpeedToggle.Parent = SpeedCategory
SpeedToggle.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
SpeedToggle.BorderSizePixel = 0
SpeedToggle.Position = UDim2.new(0, 12, 0, 75)
SpeedToggle.Size = UDim2.new(0, 50, 0, 24)
SpeedToggle.Text = "OFF"
SpeedToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedToggle.TextSize = 12
SpeedToggle.Font = Enum.Font.GothamBold
SpeedToggle.AutoButtonColor = false

local SpeedToggleCorner = Instance.new("UICorner")
SpeedToggleCorner.CornerRadius = UDim.new(0, 12)
SpeedToggleCorner.Parent = SpeedToggle

local SpeedBoostLabel = Instance.new("TextLabel")
SpeedBoostLabel.Name = "SpeedBoostLabel"
SpeedBoostLabel.Parent = SpeedCategory
SpeedBoostLabel.BackgroundTransparency = 1
SpeedBoostLabel.Position = UDim2.new(0, 70, 0, 75)
SpeedBoostLabel.Size = UDim2.new(0.7, 0, 0, 24)
SpeedBoostLabel.Font = Enum.Font.GothamMedium
SpeedBoostLabel.Text = "Speed Boost"
SpeedBoostLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
SpeedBoostLabel.TextSize = 12
SpeedBoostLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Current Speed Display
local CurrentSpeedLabel = Instance.new("TextLabel")
CurrentSpeedLabel.Name = "CurrentSpeedLabel"
CurrentSpeedLabel.Parent = SpeedCategory
CurrentSpeedLabel.BackgroundTransparency = 1
CurrentSpeedLabel.Position = UDim2.new(0, 12, 0, 105)
CurrentSpeedLabel.Size = UDim2.new(0.9, 0, 0, 20)
CurrentSpeedLabel.Font = Enum.Font.GothamMedium
CurrentSpeedLabel.Text = "Current: "..tostring(Settings.DefaultSpeed)
CurrentSpeedLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
CurrentSpeedLabel.TextSize = 11
CurrentSpeedLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Functions
local function UpdateNoclip()
    if Settings.Noclip then
        NoclipToggle.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        NoclipToggle.Text = "ON"
    else
        NoclipToggle.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
        NoclipToggle.Text = "OFF"
    end
end

local function UpdateSpeedToggle()
    if Settings.SpeedBoost then
        SpeedToggle.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
        SpeedToggle.Text = "ON"
    else
        SpeedToggle.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
        SpeedToggle.Text = "OFF"
    end
end

local function UpdateSpeedSlider()
    local percentage = (Settings.SpeedValue - 1) / (200 - 1)
    SpeedSliderFill.Size = UDim2.new(percentage, 0, 1, 0)
    SpeedValueLabel.Text = tostring(Settings.SpeedValue)
end

local function ToggleNoclip()
    Settings.Noclip = not Settings.Noclip
    UpdateNoclip()
    
    if NoclipConnection then
        NoclipConnection:Disconnect()
        NoclipConnection = nil
    end
    
    if Settings.Noclip then
        NoclipConnection = RunService.Stepped:Connect(function()
            if Settings.Noclip and Character and Character.Parent then
                local currentCharacter = LocalPlayer.Character
                if currentCharacter then
                    for _, part in pairs(currentCharacter:GetDescendants()) do
                        if part:IsA("BasePart") and part.CanCollide then
                            part.CanCollide = false
                        end
                    end
                end
            end
        end)
    else
        -- Restore collisions
        local currentCharacter = LocalPlayer.Character
        if currentCharacter then
            for _, part in pairs(currentCharacter:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
end

local function ApplySpeed()
    if Settings.SpeedBoost and Character and Character.Parent and Humanoid then
        Humanoid.WalkSpeed = Settings.SpeedValue
        CurrentSpeedLabel.Text = "Current: "..tostring(Settings.SpeedValue)
    elseif Character and Humanoid then
        Humanoid.WalkSpeed = Settings.DefaultSpeed
        CurrentSpeedLabel.Text = "Current: "..tostring(Settings.DefaultSpeed)
    end
end

local function ToggleSpeed()
    Settings.SpeedBoost = not Settings.SpeedBoost
    UpdateSpeedToggle()
    ApplySpeed()
end

local function UpdateSliderFromMouse(x)
    local relativeX = x - SpeedSliderFrame.AbsolutePosition.X
    local percentage = math.clamp(relativeX / SpeedSliderFrame.AbsoluteSize.X, 0, 1)
    Settings.SpeedValue = math.floor(1 + (percentage * (200 - 1)))
    UpdateSpeedSlider()
    ApplySpeed()
end

-- Button Animations
local function CreateButtonAnimation(button)
    local HoverAnim = Instance.new("Tween")
    button.MouseEnter:Connect(function()
        if button.BackgroundColor3 == Color3.fromRGB(231, 76, 60) or 
           button.BackgroundColor3 == Color3.fromRGB(46, 204, 113) then
            button.BackgroundTransparency = 0.1
        end
    end)
    
    button.MouseLeave:Connect(function()
        button.BackgroundTransparency = 0
    end)
end

CreateButtonAnimation(NoclipToggle)
CreateButtonAnimation(SpeedToggle)
CreateButtonAnimation(MinimizeButton)
CreateButtonAnimation(CloseButton)

-- Minimize/Close Functions
local Minimized = false
local OriginalSize = MainFrame.Size

MinimizeButton.MouseButton1Click:Connect(function()
    Minimized = not Minimized
    if Minimized then
        MainFrame:TweenSize(UDim2.new(0, 300, 0, 45), "Out", "Quad", 0.3)
    else
        MainFrame:TweenSize(OriginalSize, "Out", "Quad", 0.3)
    end
end)

CloseButton.MouseButton1Click:Connect(function()
    MainFrame:TweenSize(UDim2.new(0, 0, 0, 0), "Out", "Quad", 0.2, true)
    task.wait(0.2)
    ScreenGui:Destroy()
    if NoclipConnection then
        NoclipConnection:Disconnect()
    end
end)

-- Toggle Buttons
NoclipToggle.MouseButton1Click:Connect(ToggleNoclip)
SpeedToggle.MouseButton1Click:Connect(ToggleSpeed)

-- Slider Interaction
SpeedSliderFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        UpdateSliderFromMouse(input.Position.X)
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.Change then
                UpdateSliderFromMouse(input.Position.X)
            end
        end)
    end
end)

-- Dragging Functionality
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        Settings.Dragging = true
        Settings.DragStart = input.Position
        Settings.StartPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                Settings.Dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if Settings.Dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - Settings.DragStart
        MainFrame.Position = UDim2.new(
            Settings.StartPos.X.Scale,
            Settings.StartPos.X.Offset + delta.X,
            Settings.StartPos.Y.Scale,
            Settings.StartPos.Y.Offset + delta.Y
        )
    end
end)

-- Character Refresh Functions
local function OnCharacterAdded(newCharacter)
    Character = newCharacter
    Humanoid = Character:WaitForChild("Humanoid")
    RootPart = Character:WaitForChild("HumanoidRootPart")
    
    -- Reapply effects with delay to ensure character is fully loaded
    task.wait(0.5)
    
    if Settings.Noclip then
        ToggleNoclip()
        ToggleNoclip()
    end
    
    if Settings.SpeedBoost then
        ApplySpeed()
    end
    
    -- Update current speed display
    CurrentSpeedLabel.Text = "Current: "..tostring(Humanoid.WalkSpeed)
end

local function OnCharacterRemoving()
    if NoclipConnection then
        NoclipConnection:Disconnect()
        NoclipConnection = nil
    end
end

-- Connect to character events
LocalPlayer.CharacterAdded:Connect(OnCharacterAdded)
if Character then
    Character:GetPropertyChangedSignal("Parent"):Connect(function()
        if not Character.Parent then
            OnCharacterRemoving()
        end
    end)
end

-- Initial Setup
UpdateNoclip()
UpdateSpeedToggle()
UpdateSpeedSlider()
CurrentSpeedLabel.Text = "Current: "..tostring(Settings.DefaultSpeed)

-- Error Handling
local function SafeCall(func, ...)
    local success, result = pcall(func, ...)
    if not success then
        warn("Movement GUI Error: "..tostring(result))
    end
end

-- Periodic check to ensure stability
task.spawn(function()
    while ScreenGui.Parent do
        SafeCall(function()
            if Settings.Noclip and (not Character or not Character.Parent) then
                -- Character was removed while noclip was active
                if NoclipConnection then
                    NoclipConnection:Disconnect()
                    NoclipConnection = nil
                end
            end
            
            if Settings.SpeedBoost and Character and Character.Parent and Humanoid then
                if Humanoid.WalkSpeed ~= Settings.SpeedValue then
                    ApplySpeed()
                end
            end
        end)
        
        task.wait(1)
    end
end)

-- Initial animation
MainFrame.Size = UDim2.new(0, 0, 0, 0)
MainFrame:TweenSize(UDim2.new(0, 300, 0, 350), "Out", "Back", 0.5)
