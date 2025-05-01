-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local AIM_RADIUS = 200
local aimbotEnabled = false

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "AimbotUI"

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 160, 0, 40)
button.Position = UDim2.new(0, 20, 1, -60)
button.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 18
button.Text = "Ativar Aimbot [E]"
button.Parent = gui

button.MouseButton1Click:Connect(function()
	aimbotEnabled = not aimbotEnabled
	button.Text = aimbotEnabled and "Desativar Aimbot [E]" or "Ativar Aimbot [E]"
	button.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(200, 60, 60) or Color3.fromRGB(0, 150, 255)
end)

-- Ativar/desativar com tecla "E"
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if not gameProcessed and input.KeyCode == Enum.KeyCode.E then
		aimbotEnabled = not aimbotEnabled
		button.Text = aimbotEnabled and "Desativar Aimbot [E]" or "Ativar Aimbot [E]"
		button.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(200, 60, 60) or Color3.fromRGB(0, 150, 255)
	end
end)

-- Adiciona ESP (nome e caixa)
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

		-- Caixa ao redor (box esp)
		local box = Instance.new("BoxHandleAdornment")
		box.Name = "BoxESP"
		box.Adornee = enemy
		box.Size = Vector3.new(3, 5, 2)
		box.Color3 = Color3.new(1, 0, 0)
		box.AlwaysOnTop = true
		box.ZIndex = 5
		box.Transparency = 0.5
		box.Parent = enemy
	end
end

-- Busca inimigo mais próximo (exclui aliados)
local function getClosestEnemy()
	local enemiesFolder = workspace:FindFirstChild("Enemies")
	if not enemiesFolder then return nil end

	local closest = nil
	local shortestDistance = AIM_RADIUS
	local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

	for _, enemy in pairs(enemiesFolder:GetChildren()) do
		if enemy:IsA("Model") and enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
			if enemy.Humanoid.Health > 0 and enemy:FindFirstChild("Team") then
				if enemy.Team.Value ~= LocalPlayer:FindFirstChild("Team")?.Value then
					addESP(enemy)
					local pos, visible = Camera:WorldToViewportPoint(enemy.HumanoidRootPart.Position)
					if visible then
						local dist = (Vector2.new(pos.X, pos.Y) - screenCenter).Magnitude
						if dist < shortestDistance then
							shortestDistance = dist
							closest = enemy
						end
					end
				end
			end
		end
	end

	return closest
end

-- Mira automática
RunService.RenderStepped:Connect(function()
	if aimbotEnabled then
		local target = getClosestEnemy()
		if target then
			local targetPos = target.HumanoidRootPart.Position
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
		end
	end
end)
