
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

local normalSpeed = 16
local boostedSpeed = 32
local speedEnabled = false

local gui = Instance.new("ScreenGui")
gui.Name = "TrainingPanel"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 240, 0, 260)
frame.Position = UDim2.new(0, 20, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(28, 28, 38)
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 42)
title.BackgroundTransparency = 1
title.Text = "PAINEL DE TREINO"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 18
title.Font = Enum.Font.GothamBold
title.Parent = frame

local function createButton(text, position, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -20, 0, 42)
    button.Position = UDim2.new(0, 10, 0, position)
    button.BackgroundColor3 = Color3.fromRGB(75, 65, 160)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Text = text
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.Parent = frame

    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 7)
    buttonCorner.Parent = button

    button.Activated:Connect(callback)
    return button
end

local function getCharacter()
    return player.Character
        or player.CharacterAdded:Wait()
end

createButton("Velocidade: alternar", 50, function()
    speedEnabled = not speedEnabled

    local humanoid = getCharacter():WaitForChild("Humanoid")
    humanoid.WalkSpeed = speedEnabled and boostedSpeed or normalSpeed
end)

createButton("Pulo alto: alternar", 100, function()
    local humanoid = getCharacter():WaitForChild("Humanoid")
    humanoid.UseJumpPower = true
    humanoid.JumpPower =
        humanoid.JumpPower == 50 and 100 or 50
end)

createButton("Voltar ao normal", 150, function()
    speedEnabled = false

    local humanoid = getCharacter():WaitForChild("Humanoid")
    humanoid.WalkSpeed = normalSpeed
    humanoid.JumpPower = 50
end)

createButton("Ocultar painel (H)", 200, function()
    frame.Visible = false
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end

    if input.KeyCode == Enum.KeyCode.H then
        frame.Visible = not frame.Visible
    end
end)

print("Painel de treino carregado!")
