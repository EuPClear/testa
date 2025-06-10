-- ESP funcional no Studio com Team Check, UI arrastável e Box
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Interface
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "ESP_UI"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 180, 0, 90)
frame.Position = UDim2.new(0, 20, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
frame.Active = true
frame.Draggable = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local toggleButton = Instance.new("TextButton", frame)
toggleButton.Size = UDim2.new(1, -20, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Ativar ESP"
toggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.Gotham
toggleButton.TextSize = 14
Instance.new("UICorner", toggleButton)

local ESPEnabled = false
local ESPParts = {}

-- Função para criar BillboardGui
local function CreateESP(player)
    if player == LocalPlayer then return end
    if player.Team == LocalPlayer.Team then return end
    if not player.Character then return end

    local head = player.Character:FindFirstChild("Head")
    if not head then return end

    local bb = Instance.new("BillboardGui")
    bb.Name = "ESP_BB"
    bb.Adornee = head
    bb.Size = UDim2.new(0, 100, 0, 40)
    bb.StudsOffset = Vector3.new(0, 2, 0)
    bb.AlwaysOnTop = true

    local text = Instance.new("TextLabel", bb)
    text.Size = UDim2.new(1, 0, 1, 0)
    text.BackgroundTransparency = 1
    text.Text = player.Name
    text.TextColor3 = Color3.new(1, 0, 0)
    text.TextStrokeTransparency = 0
    text.Font = Enum.Font.GothamBold
    text.TextSize = 14

    bb.Parent = player.Character
    ESPParts[player] = bb
end

-- Função para remover ESP
local function RemoveESP(player)
    if ESPParts[player] and ESPParts[player].Parent then
        ESPParts[player]:Destroy()
        ESPParts[player] = nil
    end
end

-- Toggle
toggleButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    toggleButton.Text = ESPEnabled and "Desativar ESP" or "Ativar ESP"

    if ESPEnabled then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                CreateESP(p)
            end
        end
    else
        for _, bb in pairs(ESPParts) do
            if bb then bb:Destroy() end
        end
        ESPParts = {}
    end
end)

-- Atualização ao novos jogadores
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        wait(1)
        if ESPEnabled then
            CreateESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
end)
