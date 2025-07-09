-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local player = Players.LocalPlayer

-- GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Variáveis
local chaveCorreta = "Luizhub"
local acessoLiberado = false
local altura = 150
local plataformaAerea
local humanoid
local speedEnabled = false
local speedValue = 40
local voando = false

-- Notificação
local function showNotification(text)
	local note = Instance.new("TextLabel")
	note.Size = UDim2.new(0, 200, 0, 35)
	note.Position = UDim2.new(1, -210, 1, -60)
	note.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	note.BackgroundTransparency = 0.3
	note.Text = text
	note.TextColor3 = Color3.new(1, 1, 1)
	note.TextScaled = true
	note.Font = Enum.Font.GothamBold
	note.ZIndex = 20
	note.Parent = screenGui
	Instance.new("UICorner", note).CornerRadius = UDim.new(0, 8)
	Debris:AddItem(note, 2)
end

-- Drag
local function enableDrag(frame)
	local dragging, startPos, startInputPos
	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			startInputPos = input.Position
			startPos = frame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	frame.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local delta = input.Position - startInputPos
			frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)
end

-- Painel principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 210)
frame.Position = UDim2.new(0, 20, 0.5, -105)
frame.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
frame.Visible = false
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

-- Botão Toggle
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 40, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
toggleButton.Text = "≡"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 24
toggleButton.Visible = false
toggleButton.Parent = screenGui
Instance.new("UICorner", toggleButton).CornerRadius = UDim.new(1, 0)
enableDrag(toggleButton)
toggleButton.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

-- Créditos (modificado)
local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(1, 0, 0, 20)
creditLabel.Position = UDim2.new(0, 0, 1, -20)
creditLabel.BackgroundTransparency = 1
creditLabel.Text = "VIP Permanente"
creditLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
creditLabel.Font = Enum.Font.Gotham
creditLabel.TextSize = 14
creditLabel.TextXAlignment = Enum.TextXAlignment.Center
creditLabel.Parent = frame

-- Botões do Painel
local function criarBotao(pai, texto, posY, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, posY)
	btn.Text = texto
	btn.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 16
	btn.Parent = pai
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	btn.MouseButton1Click:Connect(callback)
end

-- Utilidade
local function getRootPart()
	return player.Character and player.Character:FindFirstChild("HumanoidRootPart") or nil
end

local function atualizarVelocidade()
	if humanoid then
		humanoid.WalkSpeed = speedEnabled and speedValue or 16
	end
end

local function onCharacterAdded(character)
	humanoid = character:WaitForChild("Humanoid")
	atualizarVelocidade()
	humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(atualizarVelocidade)
end

player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then onCharacterAdded(player.Character) end

-- Botão Velocidade
criarBotao(frame, "⚡ Velocidade", 20, function()
	speedEnabled = not speedEnabled
	atualizarVelocidade()
	showNotification(speedEnabled and "Velocidade ativada!" or "Velocidade desativada!")
end)

-- Botões soltos
local function criarBotaoSolto(nome, pos, texto, acao)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 90, 0, 30)
	btn.Position = pos
	btn.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
	btn.Text = texto
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 16
	btn.Visible = false
	btn.Parent = screenGui
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	enableDrag(btn)
	btn.MouseButton1Click:Connect(acao)
	return btn
end

local frameSubir = criarBotaoSolto("Subir", UDim2.new(1, -200, 1, -100), "↑ Subir", function()
	local rootPart = getRootPart()
	if rootPart then
		local destino = rootPart.Position + Vector3.new(0, altura, 0)
		if plataformaAerea then plataformaAerea:Destroy() end
		plataformaAerea = Instance.new("Part")
		plataformaAerea.Size = Vector3.new(20, 4, 20)
		plataformaAerea.Position = destino - Vector3.new(0, 2, 0)
		plataformaAerea.Anchored = true
		plataformaAerea.Transparency = 1
		plataformaAerea.CanCollide = true
		plataformaAerea.Parent = workspace
		rootPart.CFrame = CFrame.new(destino, destino + rootPart.CFrame.LookVector)
		showNotification("Teleportado para cima!")
	end
end)

local frameDescer = criarBotaoSolto("Descer", UDim2.new(1, -100, 1, -100), "↓ Descer", function()
	local rootPart = getRootPart()
	if rootPart then
		rootPart.CFrame = rootPart.CFrame - Vector3.new(0, altura, 0)
		if plataformaAerea then plataformaAerea:Destroy() plataformaAerea = nil end
		showNotification("Você desceu!")
	end
end)

local sairBaseBtn = criarBotaoSolto("Sair", UDim2.new(1, -300, 1, -220), "➡️ Sair da Base", function()
	local rootPart = getRootPart()
	if not rootPart then return end
	voando = true
	local duration = 3
	local speed = 40
	if rootPart:FindFirstChild("FlightVelocity") then
		rootPart.FlightVelocity:Destroy()
	end
	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.Name = "FlightVelocity"
	bodyVelocity.Velocity = (rootPart.CFrame.RightVector + Vector3.new(0, 0.2, 0)).Unit * speed
	bodyVelocity.MaxForce = Vector3.new(1, 1, 1) * 1e5
	bodyVelocity.Parent = rootPart
	showNotification("Voando para a direita...")
	task.delay(duration, function()
		if bodyVelocity and bodyVelocity.Parent then
			bodyVelocity:Destroy()
			showNotification("Você saiu da base!")
			voando = false
		end
	end)
end)

local pararVooBtn = criarBotaoSolto("Parar", UDim2.new(1, -300, 1, -180), "🛑 Parar Voo", function()
	if voando then
		voando = false
		local rootPart = getRootPart()
		if rootPart and rootPart:FindFirstChild("FlightVelocity") then
			rootPart.FlightVelocity:Destroy()
			showNotification("Voo cancelado!")
		end
	end
end)

-- ESP LockTime
local plotName
for _, plot in ipairs(workspace.Plots:GetChildren()) do
	if plot:FindFirstChild("YourBase", true) and plot:FindFirstChild("YourBase", true).Enabled then
		plotName = plot.Name
		break
	end
end

local function atualizarLockESP()
	for _, plot in pairs(workspace.Plots:GetChildren()) do
		local timeLabel = plot:FindFirstChild("Purchases", true)
			and plot.Purchases:FindFirstChild("PlotBlock", true)
			and plot.Purchases.PlotBlock.Main:FindFirstChild("BillboardGui", true)
			and plot.Purchases.PlotBlock.Main.BillboardGui:FindFirstChild("RemainingTime", true)

		if timeLabel and timeLabel:IsA("TextLabel") then
			local espName = "LockTimeESP_" .. plot.Name
			local existing = plot:FindFirstChild(espName)
			local isUnlocked = timeLabel.Text == "0s"
			local displayText = isUnlocked and "Unlocked" or ("Lock: " .. timeLabel.Text)
			local textColor = (plot.Name == plotName)
				and Color3.fromRGB(0, 255, 0)
				or (isUnlocked and Color3.fromRGB(220, 20, 60) or Color3.fromRGB(255, 255, 0))

			if not existing then
				local billboard = Instance.new("BillboardGui")
				billboard.Name = espName
				billboard.Size = UDim2.new(0, 200, 0, 30)
				billboard.StudsOffset = Vector3.new(0, 5, 0)
				billboard.AlwaysOnTop = true
				billboard.Adornee = plot.Purchases.PlotBlock.Main

				local label = Instance.new("TextLabel")
				label.Text = displayText
				label.Size = UDim2.new(1, 0, 1, 0)
				label.BackgroundTransparency = 1
				label.TextScaled = true
				label.TextColor3 = textColor
				label.TextStrokeColor3 = Color3.new(0, 0, 0)
				label.TextStrokeTransparency = 0
				label.Font = Enum.Font.SourceSansBold
				label.Parent = billboard

				billboard.Parent = plot
			else
				existing.TextLabel.Text = displayText
				existing.TextLabel.TextColor3 = textColor
			end
		end
	end
end

local function ativarESPLock()
	task.spawn(function()
		while acessoLiberado do
			atualizarLockESP()
			task.wait(0.25)
		end
	end)
end

-- Tela da Key
local keyFrame = Instance.new("Frame")
keyFrame.Size = UDim2.new(0, 300, 0, 150)
keyFrame.Position = UDim2.new(0.5, -150, 0.5, -75)
keyFrame.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
keyFrame.Parent = screenGui
Instance.new("UICorner", keyFrame).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.new(0, 0, 0, 10)
title.BackgroundTransparency = 1
title.Text = "🔑 Digite a chave de acesso"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.Parent = keyFrame

local input = Instance.new("TextBox")
input.PlaceholderText = "Digite a chave"
input.Size = UDim2.new(0.8, 0, 0, 30)
input.Position = UDim2.new(0.1, 0, 0.4, 0)
input.Font = Enum.Font.Gotham
input.TextSize = 16
input.TextColor3 = Color3.fromRGB(255, 255, 255)
input.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
input.BorderSizePixel = 0
input.Parent = keyFrame
Instance.new("UICorner", input).CornerRadius = UDim.new(0, 6)

local verificarBtn = Instance.new("TextButton")
verificarBtn.Text = "✅ Confirmar"
verificarBtn.Size = UDim2.new(0.8, 0, 0, 30)
verificarBtn.Position = UDim2.new(0.1, 0, 0.7, 0)
verificarBtn.Font = Enum.Font.GothamBold
verificarBtn.TextSize = 16
verificarBtn.BackgroundColor3 = Color3.fromRGB(0, 85, 170)
verificarBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
verificarBtn.ZIndex = 11
verificarBtn.Parent = keyFrame
Instance.new("UICorner", verificarBtn).CornerRadius = UDim.new(0, 6)

verificarBtn.MouseButton1Click:Connect(function()
	if input.Text == chaveCorreta then
		keyFrame:Destroy()
		acessoLiberado = true
		frame.Visible = true
		toggleButton.Visible = true
		frameSubir.Visible = true
		frameDescer.Visible = true
		sairBaseBtn.Visible = true
		pararVooBtn.Visible = true
		showNotification("✅ Acesso Liberado!")
		ativarESPLock()
	else
		showNotification("❌ Chave incorreta.")
	end
end)
