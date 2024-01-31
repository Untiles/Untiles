-- Obter o serviço de jogadores
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Obter o jogador local
local localPlayer = Players.LocalPlayer

-- Definir a nova distância máxima
local distanciaMaxima = 16

-- Função para manipular a aparência do jogador
local function customizePlayer(player)
    local function updateRootPart(character, humanoidRootPart)
        if character and humanoidRootPart then
            local rootPart = character:FindFirstChild("CustomRootPart") or Instance.new("Part")
            if not rootPart:IsA("Part") then
                return
            end

            if not rootPart.Parent then
                rootPart.Name = "CustomRootPart"
                rootPart.Size = Vector3.new(5, 5, 5)
                rootPart.Anchored = true -- Ancorar a parte ao mundo
                rootPart.CanCollide = false
                rootPart.Transparency = 0.5
                rootPart.Color = Color3.fromRGB(58, 12, 163) -- #3A0CA3
                rootPart.Parent = character

                -- Adicionar uma malha de bloco para torná-lo quadrado
                local blockMesh = rootPart:FindFirstChildOfClass("BlockMesh") or Instance.new("BlockMesh")
                blockMesh.Scale = Vector3.new(1, 1, 1)
                blockMesh.Parent = rootPart
            end

            -- Atualizar a posição da raiz
            rootPart.CFrame = humanoidRootPart.CFrame
        end
    end

    -- Conectar a posição da raiz à raiz do jogador
    local connection
    connection = RunService.RenderStepped:Connect(function()
        local character = player.Character
        if character then
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
            updateRootPart(character, humanoidRootPart)
        end
    end)

    -- Lidar com a desconexão quando o jogador deixa o jogo ou renasce
    local function playerRemoving()
        if connection then
            connection:Disconnect()
        end
    end
    player.CharacterRemoving:Connect(playerRemoving)
    player.CharacterAdded:Connect(function(character)
        -- Reconectar a personalização após o renascimento
        playerRemoving()
        customizePlayer(player)
    end)

    -- Atualizar a raiz ao iniciar
    updateRootPart(player.Character, player.Character and player.Character:FindFirstChild("HumanoidRootPart"))
end

-- Personalizar jogadores que entrarem no jogo
Players.PlayerAdded:Connect(function(player)
    if player ~= localPlayer then
        customizePlayer(player)
    end
end)

-- Personalizar jogadores que já estão no jogo
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= localPlayer then
        customizePlayer(player)
    end
end

-- Função para verificar a distância entre o jogador local e outros jogadores
local function verificarDistancia()
    -- Verificar se o jogador está equipado com uma ferramenta
    local character = localPlayer.Character
    if character and character:FindFirstChildOfClass("Tool") then
        local tool = character:FindFirstChildOfClass("Tool")
        -- Verificar se há outros jogadores no jogo
        local players = Players:GetPlayers()
        if #players > 1 then
            local localPosition = character.HumanoidRootPart.Position
            -- Verificar a distância entre o jogador local e outros jogadores
            for _, player in pairs(players) do
                if player ~= localPlayer then
                    local playerCharacter = player.Character
                    if playerCharacter and playerCharacter:FindFirstChild("HumanoidRootPart") then
                        local playerPosition = playerCharacter.HumanoidRootPart.Position
                        local distance = (localPosition - playerPosition).magnitude
                        -- Verificar se a distância é menor ou igual à distância máxima
                        if distance <= distanciaMaxima then
                            -- Usar a ferramenta atualmente equipada
                            tool:Activate()
                            break
                        end
                    end
                end
            end
        end
    end
end

-- Loop infinito para verificar a distância e usar a ferramenta rapidamente
while true do
    verificarDistancia()
    wait(0.1) -- Reduzir a velocidade de verificação para evitar sobrecarga desnecessária
end
