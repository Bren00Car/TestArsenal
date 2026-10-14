# TestArsenal
--[[
    ARSENAL HUB - PREMIUM UI
    Interface: Rayfield Library
    Funcionalidades: Aimbot, ESP Dinâmico, Team Check e Customização de Cores.
]]

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Variáveis de Controle (Estado do Script)
local State = {
    AimbotEnabled = false,
    ESPEnabled = false,
    TeamCheck = true,
    AimPart = "Head",
    ESPColor = Color3.fromRGB(255, 0, 0),
    AimKey = Enum.UserInputType.MouseButton2
}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local HoldingAim = false

-- [ LÓGICA DO AIMBOT ]
local function isEnemy(player)
    if State.TeamCheck and player.Team == LocalPlayer.Team then return false end
    return player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0
end

local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if isEnemy(player) then
            local targetPart = player.Character:FindFirstChild(State.AimPart)
            if targetPart then
                local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local mousePos = UserInputService:GetMouseLocation()
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if distance < shortestDistance then
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end
    end
    return closestPlayer
end

RunService.RenderStepped:Connect(function()
    if State.AimbotEnabled and HoldingAim then
        local target = getClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild(State.AimPart) then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character[State.AimPart].Position)
        end
    end
end)

-- [ LÓGICA DO ESP ]
local function applyESP(player)
    if player == LocalPlayer then return end
    
    local function addHighlight(char)
        -- Remove ESP antigo se existir
        if char:FindFirstChild("ESP_Highlight") then char.ESP_Highlight:Destroy() end
        
        if State.ESPEnabled then
            local highlight = Instance.new("Highlight")
            highlight.Name = "ESP_Highlight"
            highlight.FillColor = State.ESPColor
            highlight.OutlineColor = Color3.new(1, 1, 1)
            highlight.FillTransparency = 0.5
            highlight.Adornee = char
            highlight.Parent = char
        end
    end

    player.CharacterAdded:Connect(addHighlight)
    if player.Character then addHighlight(player.Character) end
end

-- Atualiza ESP de todos quando a cor ou o estado muda
local function refreshESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            local highlight = player.Character:FindFirstChild("ESP_Highlight")
            if highlight then
                if not State.ESPEnabled then
                    highlight:Destroy()
                else
                    highlight.FillColor = State.ESPColor
                end
            elseif State.ESPEnabled then
                applyESP(player)
            end
        end
    end
end

for _, player in pairs(Players:GetPlayers()) do applyESP(player) end
Players.PlayerAdded:Connect(applyESP)

-- [ INPUTS ]
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == State.AimKey then HoldingAim = true end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == State.AimKey then HoldingAim = false end
end)

-- [ INTERFACE RAYFIELD ]
local Window = Rayfield:CreateWindow({
    Name = "ARSENAL HUB | Premium",
    LoadingTitle = "Carregando Script...",
    LoadingSubtitle = "by SkyNet",
    ConfigurationSaveLocation = getfs().BasePath.."/ArsenalHubConfig",
    KeyDown = Enum.KeyCode.RightShift -- Tecla para abrir/fechar o menu
})

-- Aba de Combate
local CombatTab = Window:CreateTab("Combate", 4483362458) 
local CombatSection = CombatTab:CreateSection("Aimbot Settings")

CombatTab:CreateToggle({
    Name = "Ativar Aimbot",
    CurrentValue = false,
    Callback = function(Value)
        State.AimbotEnabled = Value
    end,
})

CombatTab:CreateDropdown({
    Name = "Alvo (Aim Part)",
    Options = {"Head", "UpperTorso", "HumanoidRootPart"},
    CurrentOption = {"Head"},
    MultipleOptions = false,
    Callback = function(Option)
        State.AimPart = Option[1]
    end,
})

CombatTab:CreateToggle({
    Name = "Checar Time (Team Check)",
    CurrentValue = true,
    Callback = function(Value)
        State.TeamCheck = Value
    end,
})

-- Aba de Visual
local VisualTab = Window:CreateTab("Visual", 4483362458)
local VisualSection = VisualTab:CreateSection("ESP Settings")

VisualTab:CreateToggle({
    Name = "Ativar ESP",
    CurrentValue = false,
    Callback = function(Value)
        State.ESPEnabled = Value
        refreshESP()
    end,
})

VisualTab:CreateColorPicker({
    Name = "Cor do ESP",
    Color = Color3.fromRGB(255, 0, 0),
    Callback = function(Value)
        State.ESPColor = Value
        refreshESP()
    end,
})

Rayfield:Notify({
    Title = "Sucesso!",
    Content = "Arsenal Hub carregado. Aperte RightShift para abrir o menu.",
    Duration = 5,
    Image = 4483362458,
})
