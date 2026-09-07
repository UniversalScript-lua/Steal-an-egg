local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local ragdollSpeed = 80
local isRagdoll = false
local ragdollObjects = {}

local character
local humanoid
local rootPart

local function setupCharacter()
	character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
	humanoid = character:WaitForChild("Humanoid")
	rootPart = character:WaitForChild("HumanoidRootPart")
end

setupCharacter()

local function stopRagdoll()
	if not character then return end

	isRagdoll = false

	for _, object in ipairs(ragdollObjects) do
		if object and object.Parent then
			object:Destroy()
		end
	end

	table.clear(ragdollObjects)

	for _, object in ipairs(character:GetDescendants()) do
		if object:IsA("Motor6D") then
			object.Enabled = true
		end
	end

	if humanoid and humanoid.Parent then
		humanoid.PlatformStand = false
		humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
	end
end

local function activateRagdoll(direction)
	setupCharacter()

	if isRagdoll then
		stopRagdoll()
	end

	isRagdoll = true

	for _, motor in ipairs(character:GetDescendants()) do
		if motor:IsA("Motor6D") and motor.Part0 and motor.Part1 then
			
			if motor.Name ~= "RootJoint" then
				local attachment0 = Instance.new("Attachment")
				attachment0.CFrame = motor.C0
				attachment0.Parent = motor.Part0

				local attachment1 = Instance.new("Attachment")
				attachment1.CFrame = motor.C1
				attachment1.Parent = motor.Part1

				local constraint = Instance.new("BallSocketConstraint")
				constraint.Attachment0 = attachment0
				constraint.Attachment1 = attachment1
				constraint.LimitsEnabled = true
				constraint.TwistLimitsEnabled = true
				constraint.Parent = motor.Part0

				motor.Enabled = false

				table.insert(ragdollObjects, attachment0)
				table.insert(ragdollObjects, attachment1)
				table.insert(ragdollObjects, constraint)
			end
		end
	end

	humanoid.PlatformStand = true

	rootPart.AssemblyLinearVelocity = direction * ragdollSpeed
end

local function createButton(parent, text, position, color, callback)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1, -20, 0, 40)
	button.Position = UDim2.new(0, 10, 0, position)
	button.BackgroundColor3 = color
	button.BorderSizePixel = 0
	button.Text = text
	button.TextColor3 = Color3.new(1, 1, 1)
	button.TextSize = 16
	button.Font = Enum.Font.GothamBold
	button.Parent = parent

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = button

	button.MouseButton1Click:Connect(callback)

	return button
end

local oldGui = PlayerGui:FindFirstChild("RagdollPanel")
if oldGui then
	oldGui:Destroy()
end

local gui = Instance.new("ScreenGui")
gui.Name = "RagdollPanel"
gui.ResetOnSpawn = false
gui.Parent = PlayerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 160)
frame.Position = UDim2.new(0, 20, 0.5, -80)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

createButton(
	frame,
	"Frente",
	10,
	Color3.fromRGB(220, 60, 60),
	function()
		setupCharacter()
		activateRagdoll(rootPart.CFrame.LookVector)
	end
)

createButton(
	frame,
	"Trás",
	60,
	Color3.fromRGB(220, 180, 40),
	function()
		setupCharacter()
		activateRagdoll(-rootPart.CFrame.LookVector)
	end
)

createButton(
	frame,
	"Parar",
	110,
	Color3.fromRGB(60, 180, 80),
	function()
		stopRagdoll()
	end
)

LocalPlayer.CharacterAdded:Connect(function()
	task.wait(1)
	setupCharacter()
	isRagdoll = false
	table.clear(ragdollObjects)
end)

