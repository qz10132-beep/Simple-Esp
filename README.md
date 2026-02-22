local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local ESP_ENABLED = true

local LIGHT_PINK = Color3.fromRGB(255, 170, 200)
local GREEN = Color3.fromRGB(0, 255, 0)

local espObjects = {} -- [player] = {highlight, billboard, label, humanoid, root}

-- Create ESP
local function createESP(player, character)
	if player == localPlayer then return end

	local humanoid = character:WaitForChild("Humanoid")
	local head = character:WaitForChild("Head")
	local root = character:WaitForChild("HumanoidRootPart")

	-- Highlight
	local highlight = Instance.new("Highlight")
	highlight.FillTransparency = 1
	highlight.OutlineColor = LIGHT_PINK
	highlight.OutlineTransparency = 0
	highlight.Adornee = character
	highlight.Enabled = ESP_ENABLED
	highlight.Parent = character

	-- Billboard
	local billboard = Instance.new("BillboardGui")
	billboard.Size = UDim2.new(0, 140, 0, 35)
	billboard.StudsOffset = Vector3.new(0, 2.8, 0)
	billboard.AlwaysOnTop = true
	billboard.Adornee = head
	billboard.Enabled = ESP_ENABLED
	billboard.Parent = head

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.TextScaled = true
	label.Font = Enum.Font.GothamBold
	label.TextColor3 = LIGHT_PINK
	label.TextStrokeTransparency = 0
	label.Parent = billboard

	espObjects[player] = {
		highlight = highlight,
		billboard = billboard,
		label = label,
		humanoid = humanoid,
		root = root
	}
end

-- Remove ESP completely
local function removeESP(player)
	if espObjects[player] then
		if espObjects[player].highlight then
			espObjects[player].highlight:Destroy()
		end
		if espObjects[player].billboard then
			espObjects[player].billboard:Destroy()
		end
		espObjects[player] = nil
	end
end

-- Toggle ESP
local function toggleESP()
	ESP_ENABLED = not ESP_ENABLED

	for _, data in pairs(espObjects) do
		data.highlight.Enabled = ESP_ENABLED
		data.billboard.Enabled = ESP_ENABLED
	end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.T then
		toggleESP()
	end
end)

-- Update loop
RunService.RenderStepped:Connect(function()
	if not ESP_ENABLED then return end

	local localChar = localPlayer.Character
	local localRoot = localChar and localChar:FindFirstChild("HumanoidRootPart")
	if not localRoot then return end

	local nearestPlayer = nil
	local shortestDistance = math.huge

	-- Find nearest
	for player, data in pairs(espObjects) do
		if data.root and data.root.Parent then
			local dist = (data.root.Position - localRoot.Position).Magnitude
			if dist < shortestDistance then
				shortestDistance = dist
				nearestPlayer = player
			end
		end
	end

	-- Update visuals
	for player, data in pairs(espObjects) do
		if data.root and data.humanoid then
			local distance = (data.root.Position - localRoot.Position).Magnitude
			local hp = math.floor(data.humanoid.Health)

			local color = LIGHT_PINK
			if player == nearestPlayer then
				color = GREEN
			end

			data.highlight.OutlineColor = color
			data.label.TextColor3 = color
			data.label.Text = player.Name ..
				" | " .. math.floor(distance) .. "m" ..
				" | " .. hp .. " HP"
		end
	end
end)

-- Setup players
for _, player in ipairs(Players:GetPlayers()) do
	if player ~= localPlayer then
		player.CharacterAdded:Connect(function(char)
			createESP(player, char)
		end)
		if player.Character then
			createESP(player, player.Character)
		end
	end
end

Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function(char)
		createESP(player, char)
	end)
end)

Players.PlayerRemoving:Connect(removeESP)
