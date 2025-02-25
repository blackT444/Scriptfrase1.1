local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera
local AimbotEnabled = false
local UIS = game:GetService("UserInputService")
local Mouse = LocalPlayer:GetMouse()
local SelectedTarget = nil

-- Nome da arma permitida
local ArmaPermitida = "Arisaka 7.7mm"

-- Criar botão para ativar/desativar o Aimbot
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local ToggleButton = Instance.new("TextButton", ScreenGui)

ToggleButton.Size = UDim2.new(0, 100, 0, 50)
ToggleButton.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleButton.Text = "Aimbot OFF"
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.BackgroundTransparency = 0.2  

ToggleButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    if AimbotEnabled then
        ToggleButton.Text = "Aimbot ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    else
        ToggleButton.Text = "Aimbot OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        SelectedTarget = nil -- Resetar alvo se desligar
    end
end)

-- Verifica se o jogador está segurando a arma certa
local function TemArmaCerta()
    local Character = LocalPlayer.Character
    if Character and Character:FindFirstChildOfClass("Tool") then
        local Arma = Character:FindFirstChildOfClass("Tool")
        return Arma.Name == ArmaPermitida
    end
    return false
end

-- Encontrar o inimigo mais próximo
local function GetClosestEnemy()
    local MelhorAlvo = nil
    local MenorDistancia = math.huge

    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") then
            if Player.TeamColor == BrickColor.new("Bright red") then
                local Head = Player.Character.Head
                local Distancia = (Head.Position - LocalPlayer.Character.Head.Position).Magnitude

                if Distancia < MenorDistancia then
                    MenorDistancia = Distancia
                    MelhorAlvo = Head
                end
            end
        end
    end
    return MelhorAlvo
end

-- Função para atirar e dar dano de 200 no inimigo vermelho
Mouse.Button1Down:Connect(function()
    if AimbotEnabled and TemArmaCerta() then
        SelectedTarget = GetClosestEnemy() -- Prioriza o inimigo mais próximo
        if SelectedTarget then
            -- Calcular a direção da bala mágica
            local HeadPos = SelectedTarget.Position
            local StartPos = LocalPlayer.Character.Head.Position
            local Direction = (HeadPos - StartPos).unit

            -- Criar um Ray
            local Ray = Ray.new(StartPos, Direction * 1000) -- Alvo a 1000 studs de distância
            local HitPart, HitPosition = workspace:FindPartOnRay(Ray, LocalPlayer.Character)

            -- Se acertar o inimigo
            if HitPart and HitPart.Parent and Players:GetPlayerFromCharacter(HitPart.Parent) then
                local humanoid = HitPart.Parent:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid:TakeDamage(200) -- Dá 200 de dano no inimigo vermelho
                end
            end

            -- Ajustar a mira para o alvo
            local TargetPos = SelectedTarget.Position
            local CurrentCFrame = Camera.CFrame
            local NewDirection = (TargetPos - CurrentCFrame.Position).unit
            
            -- Suaviza a transição para a mira
            local SmoothedCFrame = CFrame.new(CurrentCFrame.Position, CurrentCFrame.Position + NewDirection)
            Camera.CFrame = SmoothedCFrame
        end
    end
end)

-- Aimbot principal
local function Aimbot()
    while true do
        if AimbotEnabled and TemArmaCerta() then
            SelectedTarget = GetClosestEnemy() -- Atualiza o alvo para o mais próximo
        else
            SelectedTarget = nil -- Resetar se não tiver arma
        end
        task.wait(0.01)
    end
end

-- Iniciar Aimbot
task.spawn(Aimbot)
