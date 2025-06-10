local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESPEnabled = false
local AimbotEnabled = false
local ESPParts = {}

-- // Função de UI
local function createUI()
    local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
    gui.Name = "EUP_HUB"
    gui.ResetOnSpawn = false

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 200, 0, 140)
    frame.Position = UDim2.new(0, 20, 0, 20)
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    frame.Active = true
    frame.Draggable = true
    Instance.new("UICorner", frame)
    Instance.new("UIStroke", frame).Color = Color3.fromRGB(85, 170, 255)

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.Text = "🌟 EUP HUB"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 16
    title.BackgroundTransparency = 1

    local espBtn = Instance.new("TextButton", frame)
    espBtn.Size = UDim2.new(1, -20, 0, 35)
    espBtn.Position = UDim2.new(0, 10, 0, 40)
    espBtn.Text = "Ativar ESP"
    espBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    espBtn.TextColor3 = Color3.new(1, 1, 1)
    espBtn.Font = Enum.Font.Gotham
    espBtn.TextSize = 14
    Instance.new("UICorner", espBtn)

    local aimBtn = Instance.new("TextButton", frame)
    aimBtn.Size = UDim2.new(1, -20, 0, 35)
    aimBtn.Position = UDim2.new(0, 10, 0, 85)
    aimBtn.Text = "Ativar Aimbot"
    aimBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    aimBtn.TextColor3 = Color3.new(1, 1, 1)
    aimBtn.Font = Enum.Font.Gotham
    aimBtn.TextSize = 14
    Instance.new("UICorner", aimBtn)

    espBtn.MouseButton1Click:Connect(function()
        ESPEnabled = not ESPEnabled
        espBtn.Text = ESPEnabled and "Desativar ESP" or "Ativar ESP"
        if not ESPEnabled then
            for _, bb in pairs(ESPParts) do if bb then bb:Destroy() end end
            ESPParts = {}
        else
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= LocalPlayer then
                    createESP(p)
                end
            end
        end
    end)

    aimBtn.MouseButton1Click:Connect(function()
        AimbotEnabled = not AimbotEnabled
        aimBtn.Text = AimbotEnabled and "Desativar Aimbot" or "Ativar Aimbot"
    end)
end

-- // Função ESP
function createESP(player)
    if player == LocalPlayer or player.Team == LocalPlayer.Team then return end
    if not player.Character or not player.Character:FindFirstChild("Head") then return end

    local bb = Instance.new("BillboardGui")
    bb.Name = "ESP_BB"
    bb.Adornee = player.Character.Head
    bb.Size = UDim2.new(0, 100, 0, 30)
    bb.StudsOffset = Vector3.new(0, 2, 0)
    bb.AlwaysOnTop = true

    local label = Instance.new("TextLabel", bb)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = player.Name
    label.TextColor3 = Color3.fromRGB(255, 80, 80)
    label.TextStrokeTransparency = 0
    label.Font = Enum.Font.GothamBold
    label.TextSize = 14

    bb.Parent = player.Character
    ESPParts[player] = bb
end

-- // Monitorar novos jogadores
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function()
        wait(1)
        if ESPEnabled then
            createESP(p)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(p)
    if ESPParts[p] then
        ESPParts[p]:Destroy()
        ESPParts[p] = nil
    end
end)

-- // Aimbot: pega o inimigo mais perto do mouse
function getClosestPlayer()
    local closest, shortest = nil, math.huge
    local mousePos = UserInputService:GetMouseLocation()

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Team ~= LocalPlayer.Team and p.Character and p.Character:FindFirstChild("Head") then
            local headPos, onScreen = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(headPos.X, headPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = p
                end
            end
        end
    end

    return closest
end

-- // Loop do Aimbot
RunService.RenderStepped:Connect(function()
    if AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head.Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, head)
        end
    end
end)

-- Inicializa
createUI()
