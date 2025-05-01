-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local AIM_RADIUS = 200
local aimbotEnabled = false

-- Interface mobile
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "AimbotMobileUI"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 120)
frame.Position = UDim2.new(0.5, -110, 1, -150)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
frame.BorderSizePixel = 0
frame.AnchorPoint = Vector2.new(0.5, 0)

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "Aimbot Mobile"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextScaled = true
title.BackgroundTransparency = 1

-- Status
local statusLabel = Instance.new("TextLabel", frame)
statusLabel.Position = UDim2.new(0, 0, 0, 35)
statusLabel.Size = UDim2.new(1, 0, 0, 25)
statusLabel.Text = "Status: Desativado"
statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextScaled = true
statusLabel.BackgroundTransparency = 1

-- Botão Touch
local toggleButton = Instance.new("TextButton", frame)
toggleButton.Position = UDim2.new(0.1, 0, 0, 70)
toggleButton.Size = UDim2.new(0.8, 0, 0, 40)
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
toggleButton.Text = "Ativar Aimbot"
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextScaled = true

local function updateUI()
	toggleButton.Text = aimbotEnabled and "Desativar Aimbot" or "Ativar Aimbot"
	statusLabel.Text = aimbotEnabled and "Status: Ativado" or "Status: Desativado"
	statusLabel.TextColor3 = aimbotEnabled and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 80, 80)
	toggleButton.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(200, 60, 60) or Color3.fromRGB(0, 150, 255)
end

toggleButton.MouseButton1Click:Connect(function()
	aimbotEnabled = not aimbotEnabled
	updateUI()
end)

-- ESP + Mira
local function addESP(enemy)
	if enemy:FindFirstChild("HumanoidRootPart") and not enemy:FindFirstChild("ESP") then
		local esp = Instance.new("BillboardGui")
		esp.Name = "ESP"
		esp.Adornee = enemy.HumanoidRootPart
		esp.Size = UDim2.new(0, 100, 0, 30)
		esp.StudsOffset = Vector3.new(0, 3, 0)
		esp.AlwaysOnTop = true
		esp.Parent = enemy

		local nameLabel = Instance.new("TextLabel")
		nameLabel.Size = UDim2.new(1, 0, 1, 0)
		nameLabel.BackgroundTransparency = 1
		nameLabel.Text = enemy.Name
		nameLabel.TextColor3 = Color3.new(1, 0, 0)
		nameLabel.TextScaled = true
		nameLabel.Font = Enum.Font.GothamBold
		nameLabel.Parent = esp
	end
end

local function getClosestEnemy()
	local enemiesFolder = workspace:FindFirstChild("Enemies")
	if not enemiesFolder then return nil end

	local closest, closestDist
	local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

	for _, enemy in pairs(enemiesFolder:GetChildren()) do
		if enemy:IsA("Model") and enemy:FindFirstChild("Humanoid") and enemy:FindFirstChild("HumanoidRootPart") then
			if enemy.Humanoid.Health > 0 then
				addESP(enemy)
				local pos, visible = Camera:WorldToViewportPoint(enemy.HumanoidRootPart.Position)
				if visible then
					local dist = (Vector2.new(pos.X, pos.Y) - screenCenter).Magnitude
					if not closestDist or dist < closestDist then
						closest = enemy
						closestDist = dist
					end
				end
			end
		end
	end

	return closest
end

RunService.RenderStepped:Connect(function()
	if aimbotEnabled then
		local target = getClosestEnemy()
		if target then
			local camPos = Camera.CFrame.Position
			local targetPos = target.HumanoidRootPart.Position
			Camera.CFrame = CFrame.new(camPos, targetPos)
		end
	end
end)

updateUI()
