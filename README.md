local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Interface Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BrainrotAntiBugGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Janela do Menu Principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 530)
mainFrame.Position = UDim2.new(0.05, 0, 0.2, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 14)
menuCorner.Parent = mainFrame

local menuStroke = Instance.new("UIStroke")
menuStroke.Thickness = 2
menuStroke.Color = Color3.fromRGB(160, 32, 240)
menuStroke.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "☠️ROUBE UM BRAINROT☠️"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 16
titleLabel.Font = Enum.Font.FredokaOne
titleLabel.Parent = mainFrame

local function createTextBox(placeholder, defaultText, pos, color)
	local box = Instance.new("TextBox")
	box.Size = UDim2.new(0, 190, 0, 35)
	box.Position = pos
	box.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	box.TextColor3 = color
	box.TextSize = 14
	box.Font = Enum.Font.SourceSansBold
	box.Text = defaultText
	box.PlaceholderText = placeholder
	box.ClearTextOnFocus = false
	box.Parent = mainFrame
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = box
	local stroke = Instance.new("UIStroke")
	stroke.Thickness = 1
	stroke.Color = color
	stroke.Parent = box
	return box
end

local speedInput = createTextBox("Velocidade (ex: 50)", "50", UDim2.new(0, 15, 0, 45), Color3.fromRGB(0, 255, 150))
local jumpInput = createTextBox("Pulo (ex: 100)", "100", UDim2.new(0, 15, 0, 90), Color3.fromRGB(255, 180, 0))

local states = {
	infJump = false,
	speedActive = false,
	noclipActive = false,
	fastSteal = false
}

local savedCFrame = nil
local lastValidCFrame = nil
local valSpeed = 50
local valJump = 100

local function clearPhysicsForces(rootPart)
	if rootPart then
		rootPart.Velocity = Vector3.new(0, 0, 0)
		rootPart.RotVelocity = Vector3.new(0, 0, 0)
	end
end

local function updateSpeedValue()
	local num = tonumber(speedInput.Text)
	if num then valSpeed = math.clamp(num, 0, 500) else valSpeed = 50 end
	if states.speedActive then
		local char = player.Character
		local hum = char and char:FindFirstChildOfClass("Humanoid")
		local root = char and char:FindFirstChild("HumanoidRootPart")
		if hum then 
			hum.WalkSpeed = valSpeed 
			clearPhysicsForces(root)
		end
	end
end

local function updateJumpValue()
	local num = tonumber(jumpInput.Text)
	if num then valJump = math.clamp(num, 0, 500) else valJump = 100 end
end

speedInput.FocusLost:Connect(updateSpeedValue)
jumpInput.FocusLost:Connect(updateJumpValue)

local function createMenuButton(text, pos, color)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 190, 0, 45)
	btn.Position = pos
	btn.BackgroundColor3 = color
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.TextSize = 14
	btn.Font = Enum.Font.FredokaOne
	btn.Text = text
	btn.Parent = mainFrame
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 10)
	corner.Parent = btn
	return btn
end

local btnBase = createMenuButton("🗿 MARCAR BASE", UDim2.new(0, 15, 0, 145), Color3.fromRGB(45, 45, 45))
local btnTp = createMenuButton("🚽 ROUBAR", UDim2.new(0, 15, 0, 200), Color3.fromRGB(110, 40, 180))
local btnJump = createMenuButton("🦘 INF JUMP: OFF", UDim2.new(0, 15, 0, 255), Color3.fromRGB(180, 40, 40))
local btnSpeed = createMenuButton("⚡ VELOCIDADE: OFF", UDim2.new(0, 15, 0, 310), Color3.fromRGB(180, 40, 40))
local btnNoclip = createMenuButton("🧱 NOCLIP: OFF", UDim2.new(0, 15, 0, 365), Color3.fromRGB(180, 40, 40))
local btnFastSteal = createMenuButton("🧲 FAST STEAL: OFF", UDim2.new(0, 15, 0, 420), Color3.fromRGB(180, 40, 40))

-- Sistema Arrastável
local dragging, dragInput, dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		if UserInputService:GetFocusedTextBox() then return end
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then dragging = false end
		end)
	end
end)
mainFrame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInput = input
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

-- Anti-Morte
local function onCharacterAdded(char)
	local humanoid = char:WaitForChild("Humanoid")
	local rootPart = char:WaitForChild("HumanoidRootPart")
	task.wait(0.5)
	if states.speedActive then humanoid.WalkSpeed = valSpeed end
	task.spawn(function()
		while char:IsDescendantOf(workspace) do
			if rootPart and rootPart.Position.Y > -350 then
				lastValidCFrame = rootPart.CFrame
			elseif rootPart and rootPart.Position.Y <= -350 then
				clearPhysicsForces(rootPart)
				rootPart.CFrame = lastValidCFrame or CFrame.new(0, 10, 0)
			end
			task.wait(0.5)
		end
	end)
end

if player.Character then onCharacterAdded(player.Character) end
player.CharacterAdded:Connect(onCharacterAdded)

-- BOTÕES

btnBase.Activated:Connect(function()
	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")
	if root then
		savedCFrame = root.CFrame
		btnBase.Text = "🤫 MEWING INICIADO"
		task.wait(1)
		btnBase.Text = "🗿 MARCAR BASE"
	end
end)

btnTp.Activated:Connect(function()
	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")
	if root and savedCFrame then
		clearPhysicsForces(root)
		root.CFrame = savedCFrame
		task.wait(0.05)
		clearPhysicsForces(root)
	elseif not savedCFrame then
		btnTp.Text = "❌ SEM RIZZ"
		task.wait(1)
		btnTp.Text = "🚽 ROUBAR"
	end
end)

btnJump.Activated:Connect(function()
	states.infJump = not states.infJump
	if states.infJump then
		btnJump.Text = "🦘 INF JUMP: LIGADO"
		btnJump.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
	else
		btnJump.Text = "🦘 INF JUMP: OFF"
		btnJump.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
	end
end)

UserInputService.JumpRequest:Connect(function()
	if states.infJump then
		local char = player.Character
		local hum = char and char:FindFirstChildOfClass("Humanoid")
		local root = char and char:FindFirstChild("HumanoidRootPart")
		if hum then
			hum:ChangeState(Enum.HumanoidStateType.Jumping)
			updateJumpValue()
			hum.JumpPower = valJump
			if root and root.Velocity.Y > valJump then
				root.Velocity = Vector3.new(root.Velocity.X, valJump, root.Velocity.Z)
			end
		end
	end
end)

btnSpeed.Activated:Connect(function()
	states.speedActive = not states.speedActive
	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	local root = char and char:FindFirstChild("HumanoidRootPart")
	if hum then
		if states.speedActive then
			updateSpeedValue()
			btnSpeed.Text = "⚡ VELOCIDADE: " .. tostring(valSpeed)
			btnSpeed.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
		else
			clearPhysicsForces(root)
			hum.WalkSpeed = 16
			btnSpeed.Text = "⚡ VELOCIDADE: OFF"
			btnSpeed.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
		end
	end
end)

btnNoclip.Activated:Connect(function()
	states.noclipActive = not states.noclipActive
	if states.noclipActive then
		btnNoclip.Text = "🧱 NOCLIP: ATIVO"
		btnNoclip.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
	else
		btnNoclip.Text = "🧱 NOCLIP: OFF"
		btnNoclip.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
	end
end)

-- FAST STEAL
btnFastSteal.Activated:Connect(function()
	states.fastSteal = not states.fastSteal
	if states.fastSteal then
		btnFastSteal.Text = "🧲 FAST STEAL: ON"
		btnFastSteal.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
	else
		btnFastSteal.Text = "🧲 FAST STEAL: OFF"
		btnFastSteal.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
	end
end)

RunService.Stepped:Connect(function()
	if states.noclipActive then
		local char = player.Character
		if char then
			for _, part in pairs(char:GetDescendants()) do
				if part:IsA("BasePart") and part.CanCollide then
					part.CanCollide = false
				end
			end
		end
	end

	if states.fastSteal then
		for _, obj in pairs(workspace:GetDescendants()) do
			if obj:IsA("ProximityPrompt") then
				obj.HoldDuration = 0
				if obj.Enabled then
					fireproximityprompt(obj)
				end
			end
		end
	end
end)
