
local Players = game:GetService("Players")

local pastaMoedas = workspace:WaitForChild("Moedas")
local DISTANCIA = 8
local INTERVALO = 0.25

local ativo = {}

local function prepararJogador(player)
    ativo[player] = true

    local stats = player:FindFirstChild("leaderstats")
    if not stats then
        stats = Instance.new("Folder")
        stats.Name = "leaderstats"
        stats.Parent = player
    end

    local moedas = stats:FindFirstChild("Moedas")
    if not moedas then
        moedas = Instance.new("IntValue")
        moedas.Name = "Moedas"
        moedas.Value = 0
        moedas.Parent = stats
    end
end

Players.PlayerAdded:Connect(prepararJogador)

for _, player in Players:GetPlayers() do
    prepararJogador(player)
end

while true do
    for _, player in Players:GetPlayers() do
        if ativo[player] then
            local character = player.Character
            local root = character
                and character:FindFirstChild("HumanoidRootPart")

            if root then
                for _, moeda in pastaMoedas:GetChildren() do
                    if moeda:IsA("BasePart")
                        and not moeda:GetAttribute("Coletada")
                        and (root.Position - moeda.Position).Magnitude
                            <= DISTANCIA then

                        moeda:SetAttribute("Coletada", true)

                        local stats = player:FindFirstChild("leaderstats")
                        local contador = stats
                            and stats:FindFirstChild("Moedas")

                        if contador then
                            contador.Value += 1
                            print(player.Name .. " coletou uma moeda!")
                            moeda:Destroy()
                        end
                    end
                end
            end
        end
    end

    task.wait(INTERVALO)
end

Players.PlayerRemoving:Connect(function(player)
    ativo[player] = nil
end)
