-- Interface GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui
ScreenGui.Name = "HackDebugGUI"

-- Criação dos botões
local function criarBotao(nome, posY, callback)
	local botao = Instance.new("TextButton")
	botao.Parent = ScreenGui
	botao.Size = UDim2.new(0, 160, 0, 40)
	botao.Position = UDim2.new(0, 20, 0, posY)
	botao.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	botao.TextColor3 = Color3.fromRGB(255, 255, 255)
	botao.Text = nome
	botao.TextScaled = true
	botao.MouseButton1Click:Connect(callback)
	return botao
end

-- Referência ao jogador
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- HACK 1: Speed Hack
local normalWalkSpeed = 16
local fastSpeed = 100
local speedAtivo = false

local function toggleSpeed()
	if not player.Character or not player.Character:FindFirstChild("Humanoid") then return end
	speedAtivo = not speedAtivo
	player.Character.Humanoid.WalkSpeed = speedAtivo and fastSpeed or normalWalkSpeed
end

-- HACK 2: Jump Hack
local normalJump = 50
local jumpBoost = 150
local jumpAtivo = false

local function toggleJump()
	if not player.Character or not player.Character:FindFirstChild("Humanoid") then return end
	jumpAtivo = not jumpAtivo
	player.Character.Humanoid.JumpPower = jumpAtivo and jumpBoost or normalJump
end

-- HACK 3: Auto Win
local function darWin()
	local remote = game:GetService("ReplicatedStorage"):FindFirstChild("AddWin")
	if remote and remote:IsA("RemoteEvent") then
		remote:FireServer()
	else
		local stats = player:FindFirstChild("leaderstats")
		if stats and stats:FindFirstChild("Wins") then
			stats.Wins.Value = stats.Wins.Value + 1
		end
	end
end

-- HACK 4: Fly
local flying = false
local flySpeed = 3
local bodyGyro
local bodyVelocity
local keys = {W = false, A = false, S = false, D = false}

local function toggleFly()
	flying = not flying
	local char = player.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end

	if flying then
		local hrp = char.HumanoidRootPart

		bodyGyro = Instance.new("BodyGyro")
		bodyGyro.P = 9e4
		bodyGyro.maxTorque = Vector3.new(9e9, 9e9, 9e9)
		bodyGyro.cframe = hrp.CFrame
		bodyGyro.Parent = hrp

		bodyVelocity = Instance.new("BodyVelocity")
		bodyVelocity.velocity = Vector3.new(0, 0, 0)
		bodyVelocity.maxForce = Vector3.new(9e9, 9e9, 9e9)
		bodyVelocity.Parent = hrp

		RunService.RenderStepped:Connect(function()
			if flying and char:FindFirstChild("HumanoidRootPart") then
				local cam = workspace.CurrentCamera
				local moveDir = Vector3.new()

				if keys.W then moveDir = moveDir + cam.CFrame.LookVector end
				if keys.S then moveDir = moveDir - cam.CFrame.LookVector end
				if keys.A then moveDir = moveDir - cam.CFrame.RightVector end
				if keys.D then moveDir = moveDir + cam.CFrame.RightVector end

				bodyVelocity.velocity = moveDir.Unit * flySpeed * 50
				bodyGyro.CFrame = cam.CFrame
			end
		end)
	else
		if bodyGyro then bodyGyro:Destroy() end
		if bodyVelocity then bodyVelocity:Destroy() end
	end
end

UserInputService.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Keyboard then
		if keys[input.KeyCode.Name] ~= nil then
			keys[input.KeyCode.Name] = true
		end
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Keyboard then
		if keys[input.KeyCode.Name] ~= nil then
			keys[input.KeyCode.Name] = false
		end
	end
end)

-- HACK 5: Reset
local function resetar()
	speedAtivo = false
	jumpAtivo = false
	flying = false
	if player.Character and player.Character:FindFirstChild("Humanoid") then
		player.Character.Humanoid.WalkSpeed = normalWalkSpeed
		player.Character.Humanoid.JumpPower = normalJump
	end
	if bodyGyro then bodyGyro:Destroy() end
	if bodyVelocity then bodyVelocity:Destroy() end
end

-- Criar botões
criarBotao("⚡ Speed Hack", 200, toggleSpeed)
criarBotao("💨 Jump Hack", 250, toggleJump)
criarBotao("🏆 Auto Win", 300, darWin)
criarBotao("🕊️ Fly", 350, toggleFly)
criarBotao("❌ Reset", 400, resetar)
