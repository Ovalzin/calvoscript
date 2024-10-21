local player = game.Players.LocalPlayer
local userInputService = game:GetService("UserInputService")
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Criando a GUI
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 400, 0, 300)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
titleLabel.Text = "👴 Calvinho Hub 👴"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 24

local statusLabel = Instance.new("TextLabel", mainFrame)
statusLabel.Size = UDim2.new(1, 0, 0, 40)
statusLabel.Position = UDim2.new(0, 0, 0, 40)
statusLabel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
statusLabel.Text = "Status: Parado"
statusLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 20

local startButton = Instance.new("TextButton", mainFrame)
startButton.Size = UDim2.new(0.45, 0, 0, 50)
startButton.Position = UDim2.new(0.05, 0, 0, 100)
startButton.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
startButton.Text = "Iniciar"
startButton.TextColor3 = Color3.fromRGB(255, 255, 255)
startButton.Font = Enum.Font.Gotham
startButton.TextSize = 20

local stopButton = Instance.new("TextButton", mainFrame)
stopButton.Size = UDim2.new(0.45, 0, 0, 50)
stopButton.Position = UDim2.new(0.5, 0, 0, 100)
stopButton.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
stopButton.Text = "Parar"
stopButton.TextColor3 = Color3.fromRGB(255, 255, 255)
stopButton.Font = Enum.Font.Gotham
stopButton.TextSize = 20

local farming = false

-- Função principal do farming
local function kaitunFarm()
    farming = true
    statusLabel.Text = "Status: Farmando..."

    while farming do
        -- Buscar o NPC de missão
        local questGiver = workspace:FindFirstChild("QuestGiver")  -- Nome do NPC de missões (ajuste conforme o jogo)
        if questGiver then
            -- Chegar perto do NPC de missão
            humanoid:MoveTo(questGiver.Position)
            humanoid.MoveToFinished:Wait()

            -- Simula aceitar a missão
            local proximityPrompt = questGiver:FindFirstChildOfClass("ProximityPrompt")
            if proximityPrompt then
                fireproximityprompt(proximityPrompt)
            end

            -- Procurar NPCs inimigos para atacar
            local nearestEnemy
            local closestDist = math.huge

            for _, npc in pairs(workspace.Enemies:GetChildren()) do  -- Ajuste o caminho até os NPCs inimigos
                local dist = (npc.HumanoidRootPart.Position - character.HumanoidRootPart.Position).Magnitude
                if dist < closestDist then
                    nearestEnemy = npc
                    closestDist = dist
                end
            end

            if nearestEnemy then
                -- Movendo-se para o inimigo
                humanoid:MoveTo(nearestEnemy.HumanoidRootPart.Position)
                humanoid.MoveToFinished:Wait()

                -- Auto click para atacar
                while nearestEnemy.Humanoid.Health > 0 and farming do
                    game:GetService("VirtualUser"):ClickButton1(Vector2.new())
                    wait(0.1)
                end
            end
        else
            print("NPC de missão não encontrado")
        end
        wait(1)
    end

    statusLabel.Text = "Status: Parado"
end

-- Conectando os botões
startButton.MouseButton1Click:Connect(function()
    if not farming then
        kaitunFarm()
    end
end)

stopButton.MouseButton1Click:Connect(function()
    farming = false
end)

-- Função para mover a GUI
local dragging
local dragInput
local dragStart
local startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

userInputService.InputChanged:Connect(function()
    if dragging and dragInput then
        local delta = dragInput.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

print("Calvinho Hub iniciado!")
