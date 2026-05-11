local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local spawnRemote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("FireServer")

local SPAWN_LOCATION_OPTIONS = {
	"Antarctica",
	"Skull Island",
	"Hong Kong",
	"Desert",
}

local enabled = false
local respawnDebounce = false
local selectedSpawnLocation = SPAWN_LOCATION_OPTIONS[1]
local locationButtons = {}

local THEME_BLACK = Color3.fromRGB(10, 10, 12)
local THEME_PANEL = Color3.fromRGB(18, 18, 22)
local THEME_PANEL_ALT = Color3.fromRGB(26, 26, 31)
local THEME_RED = Color3.fromRGB(210, 34, 42)
local THEME_RED_DARK = Color3.fromRGB(120, 18, 24)
local THEME_WHITE = Color3.fromRGB(245, 245, 245)
local THEME_MUTED = Color3.fromRGB(170, 170, 176)
local THEME_OUTLINE = Color3.fromRGB(95, 20, 26)
local SIDE_DROPDOWN_WIDTH = 180

local dragging = false
local dragStart
local startPos

local function getCurrentKaiju()
	local leaderstats = localPlayer:FindFirstChild("leaderstats")
	local kaijuValue = leaderstats and leaderstats:FindFirstChild("Kaiju")
	return kaijuValue and kaijuValue.Value or nil
end

local gui = Instance.new("ScreenGui")
gui.Name = "AutoSpawnerGui"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local shadow = Instance.new("Frame")
shadow.Size = UDim2.new(0, 320, 0, 250)
shadow.Position = UDim2.new(0, 25, 0, 25)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BackgroundTransparency = 0.35
shadow.BorderSizePixel = 0
shadow.Parent = gui

local shadowCorner = Instance.new("UICorner")
shadowCorner.CornerRadius = UDim.new(0, 22)
shadowCorner.Parent = shadow

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 320, 0, 250)
frame.Position = UDim2.new(0, 20, 0, 20)
frame.BackgroundColor3 = THEME_BLACK
frame.BorderSizePixel = 0
frame.Parent = gui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 20)
frameCorner.Parent = frame

local frameStroke = Instance.new("UIStroke")
frameStroke.Color = THEME_OUTLINE
frameStroke.Thickness = 2
frameStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
frameStroke.Parent = frame

local header = Instance.new("Frame")
header.Size = UDim2.new(1, -24, 0, 58)
header.Position = UDim2.new(0, 12, 0, 10)
header.BackgroundColor3 = THEME_PANEL
header.BorderSizePixel = 0
header.Parent = frame

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 16)
headerCorner.Parent = header

local headerStroke = Instance.new("UIStroke")
headerStroke.Color = THEME_OUTLINE
headerStroke.Thickness = 1.5
headerStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
headerStroke.Parent = header

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -18, 0, 26)
title.Position = UDim2.new(0, 10, 0, 7)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextXAlignment = Enum.TextXAlignment.Left
title.Text = "Kaiju Alpha"
title.TextColor3 = THEME_WHITE
title.Parent = header

local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(1, -18, 0, 18)
creditLabel.Position = UDim2.new(0, 10, 0, 32)
creditLabel.BackgroundTransparency = 1
creditLabel.Font = Enum.Font.GothamSemibold
creditLabel.TextSize = 11
creditLabel.TextXAlignment = Enum.TextXAlignment.Left
creditLabel.TextColor3 = THEME_MUTED
creditLabel.Text = "auto spawner"
creditLabel.Parent = header

local page = Instance.new("Frame")
page.Size = UDim2.new(1, -24, 1, -86)
page.Position = UDim2.new(0, 12, 0, 78)
page.BackgroundTransparency = 1
page.Parent = frame

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(1, 0, 0, 40)
toggleButton.Position = UDim2.new(0, 0, 0, 0)
toggleButton.BackgroundColor3 = THEME_RED_DARK
toggleButton.TextColor3 = THEME_WHITE
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 16
toggleButton.Text = "Auto Spawn: OFF"
toggleButton.Parent = page

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 12)
toggleCorner.Parent = toggleButton

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Color = THEME_RED
toggleStroke.Thickness = 1.5
toggleStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
toggleStroke.Parent = toggleButton

local locationButton = Instance.new("TextButton")
locationButton.Size = UDim2.new(1, 0, 0, 36)
locationButton.Position = UDim2.new(0, 0, 0, 52)
locationButton.BackgroundColor3 = THEME_PANEL_ALT
locationButton.TextColor3 = THEME_WHITE
locationButton.Font = Enum.Font.GothamSemibold
locationButton.TextSize = 15
locationButton.Text = "Spawn Location: " .. selectedSpawnLocation
locationButton.Parent = page

local locationCorner = Instance.new("UICorner")
locationCorner.CornerRadius = UDim.new(0, 12)
locationCorner.Parent = locationButton

local locationStroke = Instance.new("UIStroke")
locationStroke.Color = THEME_OUTLINE
locationStroke.Thickness = 1.5
locationStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
locationStroke.Parent = locationButton

local locationFrame = Instance.new("ScrollingFrame")
locationFrame.Size = UDim2.new(0, 0, 0, 120)
locationFrame.Position = UDim2.new(0, -176, 0, 52)
locationFrame.BackgroundColor3 = THEME_PANEL
locationFrame.BorderSizePixel = 0
locationFrame.ScrollBarThickness = 4
locationFrame.ScrollBarImageColor3 = THEME_RED
locationFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
locationFrame.Visible = false
locationFrame.Parent = page

local locationFrameCorner = Instance.new("UICorner")
locationFrameCorner.CornerRadius = UDim.new(0, 12)
locationFrameCorner.Parent = locationFrame

local locationFrameStroke = Instance.new("UIStroke")
locationFrameStroke.Color = THEME_OUTLINE
locationFrameStroke.Thickness = 1.5
locationFrameStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
locationFrameStroke.Parent = locationFrame

local locationListLayout = Instance.new("UIListLayout")
locationListLayout.Padding = UDim.new(0, 4)
locationListLayout.Parent = locationFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 0, 20)
statusLabel.Position = UDim2.new(0, 0, 1, -20)
statusLabel.BackgroundTransparency = 1
statusLabel.TextColor3 = THEME_RED
statusLabel.Font = Enum.Font.GothamSemibold
statusLabel.TextSize = 14
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Text = "Status: idle"
statusLabel.Parent = page

local function refreshStatus(textOverride)
	if textOverride then
		statusLabel.Text = textOverride
	elseif enabled then
		statusLabel.Text = "Status: waiting for death"
	else
		statusLabel.Text = "Status: idle"
	end
end

local function rebuildLocationDropdown()
	for _, button in ipairs(locationButtons) do
		button:Destroy()
	end
	table.clear(locationButtons)

	local ySize = 0

	for _, option in ipairs(SPAWN_LOCATION_OPTIONS) do
		local button = Instance.new("TextButton")
		button.Size = UDim2.new(1, -8, 0, 28)
		button.BackgroundColor3 = THEME_PANEL_ALT
		button.TextColor3 = THEME_WHITE
		button.Font = Enum.Font.GothamSemibold
		button.TextSize = 14
		button.Text = option
		button.Parent = locationFrame

		local buttonCorner = Instance.new("UICorner")
		buttonCorner.CornerRadius = UDim.new(0, 10)
		buttonCorner.Parent = button

		local buttonStroke = Instance.new("UIStroke")
		buttonStroke.Color = THEME_OUTLINE
		buttonStroke.Thickness = 1.2
		buttonStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
		buttonStroke.Parent = button

		button.MouseButton1Click:Connect(function()
			selectedSpawnLocation = option
			locationButton.Text = "Spawn Location: " .. option
			locationFrame.Visible = false
			locationFrame.Size = UDim2.new(0, 0, 0, 120)
			refreshStatus()
		end)

		table.insert(locationButtons, button)
		ySize += 32
	end

	locationFrame.CanvasSize = UDim2.new(0, 0, 0, ySize)
	if locationFrame.Visible then
		locationFrame.Size = UDim2.new(0, SIDE_DROPDOWN_WIDTH, 0, math.min(ySize, 120))
	end
end

local function spawnBackIn()
	if respawnDebounce or not enabled then
		return
	end

	respawnDebounce = true
	refreshStatus("Status: respawning in 5s")

	task.delay(5, function()
		if not enabled then
			respawnDebounce = false
			refreshStatus()
			return
		end

		local currentKaiju = getCurrentKaiju()
		if currentKaiju then
			pcall(function()
				spawnRemote:FireServer("Spawn", currentKaiju, selectedSpawnLocation)
			end)
		end

		respawnDebounce = false
		refreshStatus("Status: spawned at " .. selectedSpawnLocation)
		task.delay(1.5, function()
			if not respawnDebounce then
				refreshStatus()
			end
		end)
	end)
end

local function hookCharacter(character)
	local humanoid = character:WaitForChild("Humanoid", 10)
	if humanoid then
		humanoid.Died:Connect(spawnBackIn)
	end
end

header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		shadow.Position = UDim2.new(frame.Position.X.Scale, frame.Position.X.Offset + 5, frame.Position.Y.Scale, frame.Position.Y.Offset + 5)
	end
end)

toggleButton.MouseButton1Click:Connect(function()
	enabled = not enabled
	toggleButton.Text = enabled and "Auto Spawn: ON" or "Auto Spawn: OFF"
	toggleButton.BackgroundColor3 = enabled and THEME_RED or THEME_RED_DARK
	refreshStatus()
end)

locationButton.MouseButton1Click:Connect(function()
	locationFrame.Visible = not locationFrame.Visible
	if locationFrame.Visible then
		rebuildLocationDropdown()
		local canvasHeight = locationFrame.CanvasSize.Y.Offset
		locationFrame.Size = UDim2.new(0, SIDE_DROPDOWN_WIDTH, 0, math.min(canvasHeight, 120))
	else
		locationFrame.Size = UDim2.new(0, 0, 0, 120)
	end
end)

if localPlayer.Character then
	hookCharacter(localPlayer.Character)
end

localPlayer.CharacterAdded:Connect(function(character)
	hookCharacter(character)
	refreshStatus()
end)

refreshStatus()
