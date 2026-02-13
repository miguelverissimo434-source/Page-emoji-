-- =========================================================
--            PAINEL PREMIUM ROSTO - INTEGRADO
-- =========================================================

local player = game.Players.LocalPlayer
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Criando a GUI Principal
local gui = Instance.new("ScreenGui")
gui.Parent = player:WaitForChild("PlayerGui")
gui.Name = "PainelPremiumRosto"
gui.ResetOnSpawn = false

-- Configurações Internas
local HitboxSettings = {
    Size = 20,
    Enabled = false
}
_G.ESP_Enabled = false

-- ================= PAINEL (O ROSTO) =================

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0,420,0,420)
panel.Position = UDim2.new(0.5,-210,0.5,-210)
panel.BackgroundColor3 = Color3.fromRGB(255,255,0)
panel.Visible = false
panel.Parent = gui
Instance.new("UICorner", panel).CornerRadius = UDim.new(1,0)

-- Criar os Olhos do Emoji
local function criarOlho(x)
    local eye = Instance.new("Frame")
    eye.Size = UDim2.new(0,130,0,130)
    eye.Position = UDim2.new(x,-65,0.32,-65)
    eye.BackgroundColor3 = Color3.fromRGB(255,255,255)
    eye.BorderSizePixel = 0
    eye.Parent = panel
    Instance.new("UICorner", eye).CornerRadius = UDim.new(1,0)

    local pupil = Instance.new("Frame")
    pupil.Size = UDim2.new(0,80,0,80)
    pupil.Position = UDim2.new(0.55,-40,0.45,-40)
    pupil.BackgroundColor3 = Color3.fromRGB(0,0,0)
    pupil.BorderSizePixel = 0
    pupil.Parent = eye
    Instance.new("UICorner", pupil).CornerRadius = UDim.new(1,0)
end

criarOlho(0.3)
criarOlho(0.7)

-- Criar a Boca (Triângulo/Semicírculo)
local mouth = Instance.new("Frame")
mouth.Size = UDim2.new(0,220,0,160)
mouth.Position = UDim2.new(0.5,-110,0.6,-20)
mouth.BackgroundColor3 = Color3.fromRGB(0,0,0)
mouth.BorderSizePixel = 0
mouth.Rotation = 180
mouth.Parent = panel

local gradient = Instance.new("UIGradient")
gradient.Rotation = 90
gradient.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0,1),
    NumberSequenceKeypoint.new(0.49,1),
    NumberSequenceKeypoint.new(0.5,0),
    NumberSequenceKeypoint.new(1,0)
})
gradient.Parent = mouth

-- Criar a Língua
local tongue = Instance.new("Frame")
tongue.Size = UDim2.new(0,110,0,50)
tongue.Position = UDim2.new(0.5,-55,0.73,-10)
tongue.BackgroundColor3 = Color3.fromRGB(200,0,0)
tongue.BorderSizePixel = 0
tongue.Parent = panel
Instance.new("UICorner", tongue).CornerRadius = UDim.new(1,0)

-- ================= BOTÕES DE FUNÇÃO =================

-- Botão Hitbox (Na parte de cima)
local hitboxBtn = Instance.new("TextButton")
hitboxBtn.Size = UDim2.new(0,240,0,80)
hitboxBtn.Position = UDim2.new(0.5,-120,0.13,0)
hitboxBtn.Text = "Hitbox: OFF"
hitboxBtn.Font = Enum.Font.SourceSansBold
hitboxBtn.TextSize = 20
hitboxBtn.BackgroundColor3 = Color3.fromRGB(240,240,170)
hitboxBtn.ZIndex = 5
hitboxBtn.Parent = panel
Instance.new("UICorner", hitboxBtn).CornerRadius = UDim.new(1,0)

-- Botão ESP (Na parte de baixo)
local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.new(0,220,0,80)
espBtn.Position = UDim2.new(0.5,-110,0.8,-80)
espBtn.Text = "ESP: OFF"
espBtn.Font = Enum.Font.SourceSansBold
espBtn.TextSize = 20
espBtn.BackgroundColor3 = Color3.fromRGB(240,240,170)
espBtn.ZIndex = 5
espBtn.Parent = panel
Instance.new("UICorner", espBtn).CornerRadius = UDim.new(1,0)

-- ================= LÓGICA HITBOX PREMIUM =================

hitboxBtn.MouseButton1Click:Connect(function()
    HitboxSettings.Enabled = not HitboxSettings.Enabled
    if HitboxSettings.Enabled then
        hitboxBtn.Text = "Hitbox: ON"
        hitboxBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    else
        hitboxBtn.Text = "Hitbox: OFF"
        hitboxBtn.BackgroundColor3 = Color3.fromRGB(240,240,170)
        -- Resetar tamanho ao desligar
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                p.Character.HumanoidRootPart.Size = Vector3.new(2, 2, 1)
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    if HitboxSettings.Enabled then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                pcall(function()
                    local root = p.Character.HumanoidRootPart
                    root.Size = Vector3.new(HitboxSettings.Size, HitboxSettings.Size, HitboxSettings.Size)
                    root.Transparency = 1
                    root.CanCollide = false
                end)
            end
        end
    end
end)

-- ================= LÓGICA ESP PREMIUM (SQUARES) =================

local function createESP(targetPlayer)
    local box = Drawing.new("Square")
    box.Visible = false
    box.Thickness = 2
    box.Filled = false
    box.Transparency = 1

    local nameTag = Drawing.new("Text")
    nameTag.Visible = false
    nameTag.Center = true
    nameTag.Outline = true
    nameTag.Size = 13

    local connection
    connection = RunService.RenderStepped:Connect(function()
        if _G.ESP_Enabled and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") and targetPlayer.Character:FindFirstChild("Humanoid") and targetPlayer.Character.Humanoid.Health > 0 then
            local hrp = targetPlayer.Character.HumanoidRootPart
            local vector, onScreen = workspace.CurrentCamera:WorldToViewportPoint(hrp.Position)

            if onScreen then
                -- Lógica de Time
                local boxColor = Color3.fromRGB(255, 255, 255)
                if targetPlayer.Team ~= nil then
                    boxColor = (targetPlayer.Team == player.Team) and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
                end

                local sizeX = 1000 / vector.Z
                local sizeY = 1500 / vector.Z
                box.Size = Vector2.new(sizeX, sizeY)
                box.Position = Vector2.new(vector.X - sizeX / 2, vector.Y - sizeY / 2)
                box.Color = boxColor
                box.Visible = true

                nameTag.Text = targetPlayer.Name .. " [" .. math.floor(targetPlayer.Character.Humanoid.Health) .. " HP]"
                nameTag.Position = Vector2.new(vector.X, vector.Y - (sizeY / 2) - 15)
                nameTag.Color = boxColor
                nameTag.Visible = true
            else
                box.Visible = false
                nameTag.Visible = false
            end
        else
            box.Visible = false
            nameTag.Visible = false
            if not targetPlayer.Parent then
                box:Remove()
                nameTag:Remove()
                connection:Disconnect()
            end
        end
    end)
end

espBtn.MouseButton1Click:Connect(function()
    _G.ESP_Enabled = not _G.ESP_Enabled
    if _G.ESP_Enabled then
        espBtn.Text = "ESP: ON"
        espBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= player then createESP(p) end
        end
    else
        espBtn.Text = "ESP: OFF"
        espBtn.BackgroundColor3 = Color3.fromRGB(240,240,170)
    end
end)

Players.PlayerAdded:Connect(function(p)
    if _G.ESP_Enabled then createESP(p) end
end)

-- ================= BOTÃO TOGGLE (ABRIR/FECHAR) =================

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.new(0,60,0,60)
toggle.BackgroundColor3 = Color3.fromRGB(255,255,0)
toggle.Text = ""
toggle.TextSize = 30
toggle.Parent = gui
toggle.Position = UDim2.new(0.5,-30,0.85,0)
Instance.new("UICorner", toggle).CornerRadius = UDim.new(1,0)

toggle.MouseButton1Click:Connect(function()
    panel.Visible = not panel.Visible
    if panel.Visible then
        toggle.Parent = panel
        toggle.Position = UDim2.new(0.5,-30,-0.12,0)
        toggle.Text = "X"
        toggle.TextSize = 20
    else
        toggle.Parent = gui
        toggle.Position = UDim2.new(0.5,-30,0.85,0)
        toggle.Text = ""
        toggle.TextSize = 30
    end
end)

-- Sistema Draggable (Opcional para mover o botão)
local dragging, dragInput, dragStart, startPos
toggle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = toggle.Position
    end
end)

toggle.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        toggle.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
