
-- SISTEMA DE TESTE: VELOCIDADE, MOEDAS E COMBATE
-- Coloque este Script em ServerScriptService.
-- Uso destinado ao seu próprio jogo no Roblox Studio.

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local NORMAL_SPEED = 16
local FAST_SPEED = 32
local COLLECT_DISTANCE = 6
local ATTACK_RANGE = 7
local ATTACK_DAMAGE = 12
local ATTACK_COOLDOWN = 0.8

-- Criar pasta de moedas
local coinsFolder = workspace:FindFirstChild("Moedas")
if not coinsFolder then
    coinsFolder = Instance.new("Folder")
    coinsFolder.Name = "Moedas"
    coinsFolder.Parent = workspace
end

-- Criar evento
local remote = ReplicatedStorage:FindFirstChild("GameAction")
if not remote then
    remote = Instance.new("RemoteEvent")
    remote.Name = "GameAction"
    remote.Parent = ReplicatedStorage
end

local lastAttack = {}
local fastMode = {}

local function setupPlayer(player)
    local stats = player:FindFirstChild("leaderstats")
    if not stats then
        stats = Instance.new("Folder")
        stats.Name = "leaderstats"
        stats.Parent = player
    end

    if not stats:FindFirstChild("Moedas") then
        local coins = Instance.new("IntValue")
        coins.Name = "Moedas"
        coins.Value = 0
        coins.Parent = stats
    end

    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        humanoid.WalkSpeed = fastMode[player]
            and FAST_SPEED or NORMAL_SPEED
    end)

    -- Controles em um único script do servidor
    local gui = Instance.new("ScreenGui")
    gui.Name = "GameControls"
    gui.ResetOnSpawn = false

    local function createButton(name, text, position, action)
        local button = Instance.new("TextButton")
        button.Name = name
        button.Text = text
        button.Size = UDim2.fromOffset(150, 42)
        button.Position = position
        button.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
        button.TextColor3 = Color3.new(1, 1, 1)
        button.TextScaled = true
        button.Parent = gui

        button.Activated:Connect(function()
            remote:FireServer(action)
        end)
    end

    createButton(
        "SpeedButton", "Velocidade: V",
        UDim2.new(0, 20, 0, 80), "Velocidade"
    )
    createButton(
        "CoinButton", "Coletar moedas",
        UDim2.new(0, 20, 0, 130), "Coletar"
    )
    createButton(
        "AttackButton", "Atacar",
        UDim2.new(0, 20, 0, 180), "Atacar"
    )

    local playerGui = player:WaitForChild("PlayerGui")
    gui.Parent = playerGui
end

Players.PlayerAdded:Connect(setupPlayer)

for _, player in Players:GetPlayers() do
    task.spawn(setupPlayer, player)
end

remote.OnServerEvent:Connect(function(player, action)
    if action ~= "Velocidade"
        and action ~= "Coletar"
        and action ~= "Atacar" then
        return
    end

    local character = player.Character
    local humanoid = character
        and character:FindFirstChildOfClass("Humanoid")
    local root = character
        and character:FindFirstChild("HumanoidRootPart")

    if not humanoid or humanoid.Health <= 0 or not root then
        return
    end

    if action == "Velocidade" then
        fastMode[player] = not fastMode[player]
        humanoid.WalkSpeed = fastMode[player]
            and FAST_SPEED or NORMAL_SPEED

    elseif action == "Coletar" then
        for _, coin in coinsFolder:GetChildren() do
            if not coin:IsA("BasePart")
                or coin:GetAttribute("Coletada") then
                continue
            end

            if (root.Position - coin.Position).Magnitude
                <= COLLECT_DISTANCE then

                coin:SetAttribute("Coletada", true)

                local stats = player:FindFirstChild("leaderstats")
                local balance = stats and stats:FindFirstChild("Moedas")

                if balance then
                    balance.Value += 1
                    coin:Destroy()
                end
            end
        end

    elseif action == "Atacar" then
        local now = os.clock()
        if now - (lastAttack[player] or 0) < ATTACK_COOLDOWN then
            return
        end
        lastAttack[player] = now

        -- Ataque de teste em outro jogador próximo.
        for _, target in Players:GetPlayers() do
            if target == player then
                continue
            end

            local targetCharacter = target.Character
            local targetHumanoid = targetCharacter
                and targetCharacter:FindFirstChildOfClass("Humanoid")
            local targetRoot = targetCharacter
                and targetCharacter:FindFirstChild("HumanoidRootPart")

            if targetHumanoid and targetRoot
                and targetHumanoid.Health > 0
                and (root.Position - targetRoot.Position).Magnitude
                    <= ATTACK_RANGE then

                targetHumanoid:TakeDamage(ATTACK_DAMAGE)
                break
            end
        end
    end
end)

Players.PlayerRemoving:Connect(function(player)
    lastAttack[player] = nil
    fastMode[player] = nil
end)

print("Sistema carregado: velocidade, moedas e combate!")
