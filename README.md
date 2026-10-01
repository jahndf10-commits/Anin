-- =============================================
-- 🌸 ANINHA HUB — Completo
-- Coloque em: StarterGui → LocalScript
-- =============================================

local Player = game:GetService("Players").LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local TweenInfo = TweenInfo.new(0.2)

-- 🌸 CRIAR TELA
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AninhaHub"
ScreenGui.Parent = PlayerGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 220, 0, 300)
MainFrame.Position = UDim2.new(0.02, 0, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
MainFrame.BorderSizePixel = 2
MainFrame.BorderColor3 = Color3.fromRGB(255, 105, 180)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(255, 105, 180)
Title.Text = "🌸 ANINHA HUB"
Title.TextColor3 = Color3.new(1,1,1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.Parent = MainFrame

-- FUNÇÃO DE BOTÃO
local function createButton(y, text)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.9, 0, 0, 38)
	btn.Position = UDim2.new(0.05, 0, 0, y)
	btn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
	btn.Text = text
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 12
	btn.Parent = MainFrame
	return btn
end

-- BOTÕES
local FlyBtn = createButton(45, "✈️ Fly: OFF")
local AntiLagBtn = createButton(90, "🧹 Anti-Lag: OFF")
local AutoClickBtn = createButton(135, "👆 Auto Click: OFF")
local ServerHopBtn = createButton(180, "🌐 Server Hop")
local FpsLabel = Instance.new("TextLabel")
FpsLabel.Size = UDim2.new(0.9, 0, 0, 30)
FpsLabel.Position = UDim2.new(0.05, 0, 0, 230)
FpsLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
FpsLabel.Text = "🎯 FPS: 60"
FpsLabel.TextColor3 = Color3.new(1,1,1)
FpsLabel.Font = Enum.Font.Gotham
FpsLabel.TextSize = 11
FpsLabel.Parent = MainFrame

-- =============================================
-- ✈️ FLY
-- =============================================
local FlyEnabled = false
local FlySpeed = 0.6
local FlyConnection

local function toggleFly()
	FlyEnabled = not FlyEnabled
	FlyBtn.Text = FlyEnabled and "✈️ Fly: ON" or "✈️ Fly: OFF"
	FlyBtn.BackgroundColor3 = FlyEnabled and Color3.fromRGB(80, 200, 120) or Color3.fromRGB(60, 60, 80)
	
	local Char = Player.Character
	if not Char then return end
	local HRP = Char:FindFirstChild("HumanoidRootPart")
	local Hum = Char:FindFirstChild("Humanoid")
	if not HRP or not Hum then return end
	
	if FlyEnabled then
		Hum.PlatformStand = true
		FlyConnection = RunService.RenderStepped:Connect(function()
			local camCF = workspace.CurrentCamera.CFrame
			local dir = Vector3.new()
			if UIS:IsKeyDown(Enum.KeyCode.W) then dir += camCF.LookVector end
			if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= camCF.LookVector end
			if UIS:IsKeyDown(Enum.KeyCode.A) then dir -= camCF.RightVector end
			if UIS:IsKeyDown(Enum.KeyCode.D) then dir += camCF.RightVector end
			if UIS:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
			if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
			if dir.Magnitude > 0 then
				HRP.CFrame += dir.Unit * FlySpeed
			end
		end)
	else
		if FlyConnection then FlyConnection:Disconnect() end
		Hum.PlatformStand = false
	end
end
FlyBtn.MouseButton1Click:Connect(toggleFly)

-- =============================================
-- 🧹 ANTI-LAG + LIMITE DE FPS (60)
-- =============================================
local AntiLagEnabled = false
local AntiLagConnection
local desiredFPS = 60

local function setFPSLimit(fps)
	if RunService:IsRunning() then
		-- Limita taxa de atualização
		RunService.RenderStepped:Connect(function()
			task.wait(1/fps)
		end)
	end
end

local function toggleAntiLag()
	AntiLagEnabled = not AntiLagEnabled
	AntiLagBtn.Text = AntiLagEnabled and "🧹 Anti-Lag: ON" or "🧹 Anti-Lag: OFF"
	AntiLagBtn.BackgroundColor3 = AntiLagEnabled and Color3.fromRGB(80, 200, 120) or Color3.fromRGB(60, 60, 80)
	
	if AntiLagEnabled then
		setFPSLimit(desiredFPS)
		AntiLagConnection = RunService.Heartbeat:Connect(function()
			local Char = Player.Character
			if not Char or not Char:FindFirstChild("HumanoidRootPart") then return end
			local myPos = Char.HumanoidRootPart.Position
			
			for _, e in pairs(workspace:GetDescendants()) do
				if e:IsA("ParticleEmitter") then
					local dist = (e.Parent.Position - myPos).Magnitude
					e.Enabled = dist < 120
				end
				if e:IsA("PointLight") or e:IsA("SpotLight") then
					local dist = (e.Parent.Position - myPos).Magnitude
					e.Enabled = dist < 100
				end
			end
		end)
	else
		if AntiLagConnection then AntiLagConnection:Disconnect() end
		for _, e in pairs(workspace:GetDescendants()) do
			if e:IsA("ParticleEmitter") then e.Enabled = true end
			if e:IsA("PointLight") or e:IsA("SpotLight") then e.Enabled = true end
		end
	end
end
AntiLagBtn.MouseButton1Click:Connect(toggleAntiLag)

-- =============================================
-- 👆 AUTO CLICKER
-- =============================================
local AutoClickEnabled = false
local AutoClickDelay = 0.1 -- segundos
local AutoClickConnection

local function toggleAutoClick()
	AutoClickEnabled = not AutoClickEnabled
	AutoClickBtn.Text = AutoClickEnabled and "👆 Auto Click: ON" or "👆 Auto Click: OFF"
	AutoClickBtn.BackgroundColor3 = AutoClickEnabled and Color3.fromRGB(80, 200, 120) or Color3.fromRGB(60, 60, 80)
	
	if AutoClickEnabled then
		AutoClickConnection = task.spawn(function()
			while AutoClickEnabled do
				task.wait(AutoClickDelay)
				UIS:SendMouseButtonEvent(1, Enum.UserInputState.Begin)
				task.wait(0.01)
				UIS:SendMouseButtonEvent(1, Enum.UserInputState.End)
			end
		end)
	else
		task.cancel(AutoClickConnection)
	end
end
AutoClickBtn.MouseButton1Click:Connect(toggleAutoClick)

-- =============================================
-- 🌐 SERVER HOP
-- =============================================
local function serverHop()
	ServerHopBtn.Text = "🔄 Trocando..."
	ServerHopBtn.BackgroundColor3 = Color3.fromRGB(255, 200, 80)
	task.spawn(function()
		local success, err = pcall(function()
			game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, "", Player)
		end)
		if not success then
			warn("[Server Hop] Erro: " .. tostring(err))
			ServerHopBtn.Text = "❌ Erro!"
			task.wait(2)
		end
		ServerHopBtn.Text = "🌐 Server Hop"
		ServerHopBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
	end)
end
ServerHopBtn.MouseButton1Click:Connect(serverHop)

print("[🌸 Aninha Hub] ✅ Carregado com sucesso!")
# Anin
