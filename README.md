-- Fly Script com botão para celular - por @joicyscripteira (com voo automático até o spawn)

local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local flying = false
local speed = 50
local fixedHeight = 60
local flyConnection = nil

-- Ponto de spawn do personagem
local function getSpawnPoint()
	local spawnLocation = Workspace:FindFirstChildWhichIsA("SpawnLocation")
	if spawnLocation then
		return spawnLocation.Position
	else
		-- fallback: posição inicial do personagem
		return player.Character:GetPivot().Position
	end
end

-- Parar voo
local function stopFlying()
	flying = false
	if flyConnection then flyConnection:Disconnect() end
	if humanoidRootPart:FindFirstChild("BodyGyro") then
		humanoidRootPart.BodyGyro:Destroy()
	end
	if humanoidRootPart:FindFirstChild("BodyPosition") then
		humanoidRootPart.BodyPosition:Destroy()
	end
	toggleButton.Text = "Fly to Spawn"
	toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
end

-- Função para voar até o spawn
local function flyToSpawn()
	if flying then return end
	flying = true

	local targetPosition = getSpawnPoint()

	local bodyGyro = Instance.new("BodyGyro")
	bodyGyro.P = 9e4
	bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
	bodyGyro.CFrame = humanoidRootPart.CFrame
	bodyGyro.Parent = humanoidRootPart

	local bodyPosition = Instance.new("BodyPosition")
	bodyPosition.MaxForce = Vector3.new(9e9, 9e9, 9e9)
	bodyPosition.D = 1000
	bodyPosition.P = 10000
	bodyPosition.Position = humanoidRootPart.Position
	bodyPosition.Parent = humanoidRootPart

	flyConnection = RunService.RenderStepped:Connect(function()
		if not flying then return end

		-- Detectar o chão abaixo
		local rayOrigin = humanoidRootPart.Position
		local rayDirection = Vector3.new(0, -200, 0)
		local raycastParams = RaycastParams.new()
		raycastParams.FilterDescendantsInstances = {character}
		raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

		local raycastResult = Workspace:Raycast(rayOrigin, rayDirection, raycastParams)
		local groundY = raycastResult and raycastResult.Position.Y or 0
		local desiredY = groundY + fixedHeight

		-- Direção para o ponto de spawn
		local direction = (targetPosition - humanoidRootPart.Position)
		local flatDistance = (Vector3.new(direction.X, 0, direction.Z)).Magnitude

		-- Verificar se já chegou no ponto de spawn (tolerância de 5 studs)
		if flatDistance <= 5 then
			stopFlying()
			return
		end

		local moveDirection = Vector3.new(direction.X, 0, direction.Z).Unit * speed

		bodyPosition.Position = Vector3.new(
			humanoidRootPart.Position.X + moveDirection.X,
			desiredY,
			humanoidRootPart.Position.Z + moveDirection.Z
		)

		bodyGyro.CFrame = CFrame.new(humanoidRootPart.Position, Vector3.new(targetPosition.X, humanoidRootPart.Position.Y, targetPosition.Z))
	end)

	player.CharacterRemoving:Connect(function()
		stopFlying()
	end)
end

-- Interface botão lateral direita
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(1, -110, 0.85, 0)
toggleButton.AnchorPoint = Vector2.new(0, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
toggleButton.Text = "Fly to Spawn"
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Parent = screenGui

toggleButton.MouseButton1Click:Connect(function()
	if flying then
		stopFlying()
	else
		flyToSpawn()
		toggleButton.Text = "Flying..."
		toggleButton.BackgroundColor3 = Color3.fromRGB(30, 200, 30)
	end
end)
