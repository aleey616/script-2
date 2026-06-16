--[[
	Advanced God Mode System with Modern UI
	Created for Roblox - Universal Compatibility
	Features: Smooth animations, responsive design, expandable framework
--]]

-- Services
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Constants
local TWEEN_SPEED = 0.3
local CORNER_RADIUS = UDim.new(0, 8)
local PADDING = 12

-- Module Table
local GodModeUI = {}
GodModeUI.Enabled = false
GodModeUI.Connections = {}

-- Create ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GodModeSystem"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Only create if it doesn't exist
if LocalPlayer:FindFirstChild("PlayerGui") then
	local existingGui = LocalPlayer.PlayerGui:FindFirstChild("GodModeSystem")
	if existingGui then
		existingGui:Destroy()
	end
	ScreenGui.Parent = LocalPlayer.PlayerGui
end

-- Colors Configuration (Easy to customize)
local Colors = {
	Background = Color3.fromRGB(25, 25, 35),
	Container = Color3.fromRGB(35, 35, 50),
	Accent = Color3.fromRGB(100, 150, 255),
	AccentHover = Color3.fromRGB(120, 170, 255),
	AccentActive = Color3.fromRGB(0, 255, 100),
	Text = Color3.fromRGB(255, 255, 255),
	TextSecondary = Color3.fromRGB(180, 180, 190),
	Border = Color3.fromRGB(60, 60, 80),
	Shadow = Color3.fromRGB(0, 0, 0),
	Disabled = Color3.fromRGB(80, 80, 90),
	Success = Color3.fromRGB(0, 255, 100),
	Warning = Color3.fromRGB(255, 200, 0),
	Danger = Color3.fromRGB(255, 80, 80),
}

-- Main Container
local MainContainer = Instance.new("Frame")
MainContainer.Name = "MainContainer"
MainContainer.Size = UDim2.new(0, 280, 0, 200)
MainContainer.Position = UDim2.new(0, 20, 0.5, -100)
MainContainer.BackgroundColor3 = Colors.Container
MainContainer.BackgroundTransparency = 0.1
MainContainer.BorderSizePixel = 0
MainContainer.Active = true
MainContainer.Draggable = true
MainContainer.Parent = ScreenGui

-- Add corner radius
local ContainerCorner = Instance.new("UICorner")
ContainerCorner.CornerRadius = CORNER_RADIUS
ContainerCorner.Parent = MainContainer

-- Add shadow effect
local Shadow = Instance.new("ImageLabel")
Shadow.Name = "Shadow"
Shadow.Size = UDim2.new(1, 20, 1, 20)
Shadow.Position = UDim2.new(0, -10, 0, -10)
Shadow.BackgroundTransparency = 1
Shadow.Image = "rbxassetid://6014261993"
Shadow.ImageColor3 = Colors.Shadow
Shadow.ImageTransparency = 0.7
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(49, 49, 49, 49)
Shadow.ZIndex = 0
Shadow.Parent = MainContainer

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Colors.Background
TitleBar.BackgroundTransparency = 0.5
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainContainer

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = CORNER_RADIUS
TitleCorner.Parent = TitleBar

-- Title Text
local TitleText = Instance.new("TextLabel")
TitleText.Name = "TitleText"
TitleText.Size = UDim2.new(1, -PADDING * 2, 1, 0)
TitleText.Position = UDim2.new(0, PADDING, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "🛡️ God Mode System"
TitleText.TextColor3 = Colors.Text
TitleText.TextSize = 16
TitleText.Font = Enum.Font.GothamBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.Parent = TitleBar

-- Close Button
local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 5)
CloseButton.BackgroundTransparency = 1
CloseButton.Text = "✕"
CloseButton.TextColor3 = Colors.TextSecondary
CloseButton.TextSize = 16
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Parent = TitleBar

-- Content Area
local ContentFrame = Instance.new("Frame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, 0, 1, -50)
ContentFrame.Position = UDim2.new(0, 0, 0, 50)
ContentFrame.BackgroundTransparency = 1
ContentFrame.Parent = MainContainer

-- UIListLayout for organization
local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 8)
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayout.VerticalAlignment = Enum.VerticalAlignment.Top
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Parent = ContentFrame

-- Padding for content
local ContentPadding = Instance.new("UIPadding")
ContentPadding.PaddingLeft = UDim.new(0, PADDING)
ContentPadding.PaddingRight = UDim.new(0, PADDING)
ContentPadding.PaddingTop = UDim.new(0, 8)
ContentPadding.Parent = ContentFrame

-- Function to create toggle buttons
local function CreateToggleButton(name, description, icon, order)
	local buttonFrame = Instance.new("Frame")
	buttonFrame.Name = name .. "Toggle"
	buttonFrame.Size = UDim2.new(1, 0, 0, 50)
	buttonFrame.BackgroundColor3 = Colors.Background
	buttonFrame.BackgroundTransparency = 0.7
	buttonFrame.BorderSizePixel = 0
	buttonFrame.LayoutOrder = order
	
	local buttonCorner = Instance.new("UICorner")
	buttonCorner.CornerRadius = UDim.new(0, 6)
	buttonCorner.Parent = buttonFrame
	
	-- Icon
	local iconLabel = Instance.new("TextLabel")
	iconLabel.Name = "Icon"
	iconLabel.Size = UDim2.new(0, 30, 0, 30)
	iconLabel.Position = UDim2.new(0, 10, 0.5, -15)
	iconLabel.BackgroundTransparency = 1
	iconLabel.Text = icon
	iconLabel.TextSize = 20
	iconLabel.Parent = buttonFrame
	
	-- Name
	local nameLabel = Instance.new("TextLabel")
	nameLabel.Name = "Name"
	nameLabel.Size = UDim2.new(1, -120, 0, 20)
	nameLabel.Position = UDim2.new(0, 50, 0, 6)
	nameLabel.BackgroundTransparency = 1
	nameLabel.Text = name
	nameLabel.TextColor3 = Colors.Text
	nameLabel.TextSize = 14
	nameLabel.Font = Enum.Font.GothamSemibold
	nameLabel.TextXAlignment = Enum.TextXAlignment.Left
	nameLabel.Parent = buttonFrame
	
	-- Description
	local descLabel = Instance.new("TextLabel")
	descLabel.Name = "Description"
	descLabel.Size = UDim2.new(1, -120, 0, 16)
	descLabel.Position = UDim2.new(0, 50, 0, 26)
	descLabel.BackgroundTransparency = 1
	descLabel.Text = description
	descLabel.TextColor3 = Colors.TextSecondary
	descLabel.TextSize = 11
	descLabel.Font = Enum.Font.Gotham
	descLabel.TextXAlignment = Enum.TextXAlignment.Left
	descLabel.Parent = buttonFrame
	
	-- Toggle Switch
	local switchFrame = Instance.new("Frame")
	switchFrame.Name = "SwitchFrame"
	switchFrame.Size = UDim2.new(0, 44, 0, 24)
	switchFrame.Position = UDim2.new(1, -54, 0.5, -12)
	switchFrame.BackgroundColor3 = Colors.Disabled
	switchFrame.BorderSizePixel = 0
	
	local switchCorner = Instance.new("UICorner")
	switchCorner.CornerRadius = UDim.new(1, 0)
	switchCorner.Parent = switchFrame
	
	local switchKnob = Instance.new("Frame")
	switchKnob.Name = "Knob"
	switchKnob.Size = UDim2.new(0, 18, 0, 18)
	switchKnob.Position = UDim2.new(0, 3, 0.5, -9)
	switchKnob.BackgroundColor3 = Colors.Text
	switchKnob.BorderSizePixel = 0
	
	local knobCorner = Instance.new("UICorner")
	knobCorner.CornerRadius = UDim.new(1, 0)
	knobCorner.Parent = switchKnob
	
	switchKnob.Parent = switchFrame
	switchFrame.Parent = buttonFrame
	
	-- Status indicator
	local statusIndicator = Instance.new("Frame")
	statusIndicator.Name = "StatusIndicator"
	statusIndicator.Size = UDim2.new(0, 6, 0, 6)
	statusIndicator.Position = UDim2.new(0, 8, 0.5, -3)
	statusIndicator.BackgroundColor3 = Colors.Danger
	statusIndicator.BorderSizePixel = 0
	
	local indicatorCorner = Instance.new("UICorner")
	indicatorCorner.CornerRadius = UDim.new(1, 0)
	indicatorCorner.Parent = statusIndicator
	
	statusIndicator.Visible = false
	statusIndicator.Parent = buttonFrame
	
	-- Toggle Function
	local isEnabled = false
	local toggleConnection
	
	local function updateToggle(state)
		isEnabled = state
		if state then
			switchFrame.BackgroundColor3 = Colors.AccentActive
			switchKnob:TweenPosition(
				UDim2.new(1, -21, 0.5, -9),
				"Out",
				"Quad",
				TWEEN_SPEED,
				true
			)
			statusIndicator.BackgroundColor3 = Colors.Success
			statusIndicator.Visible = true
		else
			switchFrame.BackgroundColor3 = Colors.Disabled
			switchKnob:TweenPosition(
				UDim2.new(0, 3, 0.5, -9),
				"Out",
				"Quad",
				TWEEN_SPEED,
				true
			)
			statusIndicator.BackgroundColor3 = Colors.Danger
			statusIndicator.Visible = true
		end
	end
	
	-- Hover effects
	buttonFrame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			isEnabled = not isEnabled
			updateToggle(isEnabled)
			return isEnabled
		end
	end)
	
	buttonFrame.MouseEnter:Connect(function()
		buttonFrame.BackgroundTransparency = 0.5
	end)
	
	buttonFrame.MouseLeave:Connect(function()
		buttonFrame.BackgroundTransparency = 0.7
	end)
	
	return {
		Frame = buttonFrame,
		Update = updateToggle,
		GetState = function() return isEnabled end,
	}
end

-- Create God Mode Toggle
local godModeToggle = CreateToggleButton(
	"God Mode",
	"Torna o jogador invencível",
	"🛡️",
	1
)
godModeToggle.Frame.Parent = ContentFrame

-- Create Auto Heal Toggle (Example expandable feature)
local autoHealToggle = CreateToggleButton(
	"Auto Heal",
	"Regenera vida automaticamente",
	"💚",
	2
)
autoHealToggle.Frame.Parent = ContentFrame

-- Status Bar
local statusBar = Instance.new("Frame")
statusBar.Name = "StatusBar"
statusBar.Size = UDim2.new(1, 0, 0, 2)
statusBar.Position = UDim2.new(0, 0, 0, 48)
statusBar.BackgroundColor3 = Colors.AccentActive
statusBar.BackgroundTransparency = 0.5
statusBar.BorderSizePixel = 0
statusBar.Visible = false
statusBar.Parent = ContentFrame

local statusBarCorner = Instance.new("UICorner")
statusBarCorner.CornerRadius = UDim.new(1, 0)
statusBarCorner.Parent = statusBar

-- God Mode Core System
local characterConnections = {}

local function protectCharacter(character)
	if not character then return end
	
	-- Clear existing connections for this character
	if characterConnections[character] then
		for _, conn in ipairs(characterConnections[character]) do
			conn:Disconnect()
		end
	end
	
	characterConnections[character] = {}
	
	-- Find Humanoid
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		-- Store original max health
		local originalMaxHealth = humanoid.MaxHealth
		
		-- Set max health to infinite
		humanoid.MaxHealth = math.huge
		humanoid.Health = math.huge
		
		-- Connection to maintain health
		local healthConnection = humanoid.HealthChanged:Connect(function()
			if GodModeUI.Enabled and humanoid.Health < math.huge then
				humanoid.Health = math.huge
			end
		end)
		
		table.insert(characterConnections[character], healthConnection)
		
		-- Connection for when god mode is disabled
		local disableConnection
		disableConnection = GodModeUI.OnDisabled:Connect(function()
			if humanoid and humanoid.Parent then
				humanoid.MaxHealth = originalMaxHealth
				humanoid.Health = originalMaxHealth
			end
			disableConnection:Disconnect()
		end)
		
		table.insert(characterConnections[character], disableConnection)
	end
	
	-- Protect against damage from all sources
	local descendantAddedConnection
	descendantAddedConnection = character.DescendantAdded:Connect(function(descendant)
		if GodModeUI.Enabled and descendant:IsA("Humanoid") then
			descendant.MaxHealth = math.huge
			descendant.Health = math.huge
		end
	end)
	
	table.insert(characterConnections[character], descendantAddedConnection)
end

local function unprotectCharacter(character)
	if not character then return end
	
	-- Find Humanoid
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.MaxHealth = 100
		humanoid.Health = 100
	end
	
	-- Clear connections
	if characterConnections[character] then
		for _, conn in ipairs(characterConnections[character]) do
			conn:Disconnect()
		end
		characterConnections[character] = nil
	end
end

-- Signal-like events for God Mode state changes
GodModeUI.OnEnabled = Instance.new("BindableEvent")
GodModeUI.OnDisabled = Instance.new("BindableEvent")

-- Enable/Disable functions
function GodModeUI.Enable()
	if GodModeUI.Enabled then return end
	GodModeUI.Enabled = true
	
	-- Protect current character
	if LocalPlayer.Character then
		protectCharacter(LocalPlayer.Character)
	end
	
	-- Update UI
	godModeToggle.Update(true)
	statusBar.Visible = true
	
	-- Animate status bar
	TweenService:Create(
		statusBar,
		TweenInfo.new(1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out, -1),
		{BackgroundTransparency = 0}
	):Play()
	
	GodModeUI.OnEnabled:Fire()
	
	print("🛡️ God Mode Ativado")
end

function GodModeUI.Disable()
	if not GodModeUI.Enabled then return end
	GodModeUI.Enabled = false
	
	-- Unprotect current character
	if LocalPlayer.Character then
		unprotectCharacter(LocalPlayer.Character)
	end
	
	-- Update UI
	godModeToggle.Update(false)
	statusBar.Visible = false
	
	GodModeUI.OnDisabled:Fire()
	
	print("⚔️ God Mode Desativado")
end

-- Character handling
local function onCharacterAdded(character)
	if GodModeUI.Enabled then
		protectCharacter(character)
	end
end

local function onCharacterRemoving(character)
	unprotectCharacter(character)
end

-- Connect character events
if LocalPlayer.Character then
	onCharacterAdded(LocalPlayer.Character)
end

LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
LocalPlayer.CharacterRemoving:Connect(onCharacterRemoving)

-- Toggle button click handler
godModeToggle.Frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		if GodModeUI.Enabled then
			GodModeUI.Disable()
		else
			GodModeUI.Enable()
		end
	end
end)

-- Auto Heal System (Example expandable feature)
local autoHealEnabled = false
local healConnection

autoHealToggle.Frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		autoHealEnabled = not autoHealEnabled
		autoHealToggle.Update(autoHealEnabled)
		
		if autoHealEnabled then
			healConnection = RunService.Heartbeat:Connect(function()
				if LocalPlayer.Character then
					local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
					if humanoid and humanoid.Health < humanoid.MaxHealth then
						humanoid.Health = math.min(humanoid.Health + 2, humanoid.MaxHealth)
					end
				end
			end)
			print("💚 Auto Heal Ativado")
		else
			if healConnection then
				healConnection:Disconnect()
			end
			print("💚 Auto Heal Desativado")
		end
	end
end)

-- Close button functionality
CloseButton.MouseButton1Click:Connect(function()
	ScreenGui.Enabled = false
	print("📱 UI Minimizada - Use o comando /godmode para reabrir")
end)

-- Keyboard shortcut (Press RightShift to toggle)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.RightShift then
		if GodModeUI.Enabled then
			GodModeUI.Disable()
		else
			GodModeUI.Enable()
		end
	end
end)

-- Toggle UI visibility with LeftControl
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.LeftControl then
		ScreenGui.Enabled = not ScreenGui.Enabled
	end
end)

-- Command bar support
LocalPlayer.Chatted:Connect(function(message)
	if message:lower() == "/godmode" then
		ScreenGui.Enabled = true
		if GodModeUI.Enabled then
			GodModeUI.Disable()
		else
			GodModeUI.Enable()
		end
	elseif message:lower() == "/godmode ui" then
		ScreenGui.Enabled = not ScreenGui.Enabled
	end
end)

-- Initial animation
MainContainer.Size = UDim2.new(0, 0, 0, 200)
MainContainer.Position = UDim2.new(0, 20, 0.5, -100)
TweenService:Create(
	MainContainer,
	TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
	{Size = UDim2.new(0, 280, 0, 200)}
):Play()

-- Print instructions
print([[
╔══════════════════════════════════════╗
║     🛡️ GOD MODE SYSTEM LOADED      ║
╠══════════════════════════════════════╣
║ Comandos:                           ║
║  /godmode    - Toggle God Mode     ║
║  /godmode ui - Toggle UI           ║
║                                     ║
║ Atalhos:                            ║
║  RightShift  - Toggle God Mode     ║
║  LeftControl - Toggle UI           ║
║                                     ║
║ Arraste a janela para mover!        ║
╚══════════════════════════════════════╝
]])

return GodModeUI
