-- Configurações iniciais
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Variáveis do exploit
local hitboxEnabled = false
local hitboxColor = Color3.fromRGB(255, 0, 0)
local hitboxSize = 5
local hitboxTransparency = 0.5
local hitboxVisible = true
local menuVisible = true

-- Criação da GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ExploitMenu"
ScreenGui.Parent = game:GetService("CoreGui")

-- Frame principal do menu
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0.25, 0, 0.4, 0)
MainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

-- Título do menu
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0.1, 0)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "EB Exploit v1.0"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.Parent = MainFrame

-- Botão de fechar (X)
local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0.1, 0, 0.1, 0)
CloseButton.Position = UDim2.new(0.9, 0, 0, 0)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 14
CloseButton.Parent = MainFrame

-- Toggle para hitbox
local HitboxToggle = Instance.new("TextButton")
HitboxToggle.Size = UDim2.new(0.9, 0, 0.1, 0)
HitboxToggle.Position = UDim2.new(0.05, 0, 0.15, 0)
HitboxToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
HitboxToggle.Text = "Hitbox: DESLIGADO"
HitboxToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
HitboxToggle.Font = Enum.Font.Gotham
HitboxToggle.TextSize = 12
HitboxToggle.Parent = MainFrame

-- Controle de cor da hitbox
local ColorLabel = Instance.new("TextLabel")
ColorLabel.Size = UDim2.new(0.9, 0, 0.08, 0)
ColorLabel.Position = UDim2.new(0.05, 0, 0.3, 0)
ColorLabel.BackgroundTransparency = 1
ColorLabel.Text = "Cor da Hitbox:"
ColorLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
ColorLabel.Font = Enum.Font.Gotham
ColorLabel.TextSize = 12
ColorLabel.TextXAlignment = Enum.TextXAlignment.Left
ColorLabel.Parent = MainFrame

local ColorBox = Instance.new("TextBox")
ColorBox.Size = UDim2.new(0.3, 0, 0.08, 0)
ColorBox.Position = UDim2.new(0.6, 0, 0.3, 0)
ColorBox.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
ColorBox.Text = "255,0,0"
ColorBox.TextColor3 = Color3.fromRGB(255, 255, 255)
ColorBox.Font = Enum.Font.Gotham
ColorBox.TextSize = 12
ColorBox.Parent = MainFrame

-- Controle de tamanho da hitbox
local SizeLabel = Instance.new("TextLabel")
SizeLabel.Size = UDim2.new(0.9, 0, 0.08, 0)
SizeLabel.Position = UDim2.new(0.05, 0, 0.4, 0)
SizeLabel.BackgroundTransparency = 1
SizeLabel.Text = "Tamanho da Hitbox: 5"
SizeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
SizeLabel.Font = Enum.Font.Gotham
SizeLabel.TextSize = 12
SizeLabel.TextXAlignment = Enum.TextXAlignment.Left
SizeLabel.Parent = MainFrame

local SizeSlider = Instance.new("TextButton")
SizeSlider.Size = UDim2.new(0.8, 0, 0.08, 0)
SizeSlider.Position = UDim2.new(0.1, 0, 0.5, 0)
SizeSlider.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
SizeSlider.Text = "Ajustar Tamanho (5)"
SizeSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
SizeSlider.Font = Enum.Font.Gotham
SizeSlider.TextSize = 12
SizeSlider.Parent = MainFrame

-- Controle de transparência
local TransparencyLabel = Instance.new("TextLabel")
TransparencyLabel.Size = UDim2.new(0.9, 0, 0.08, 0)
TransparencyLabel.Position = UDim2.new(0.05, 0, 0.6, 0)
TransparencyLabel.BackgroundTransparency = 1
TransparencyLabel.Text = "Transparência: 0.5"
TransparencyLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TransparencyLabel.Font = Enum.Font.Gotham
TransparencyLabel.TextSize = 12
TransparencyLabel.TextXAlignment = Enum.TextXAlignment.Left
TransparencyLabel.Parent = MainFrame

local TransparencySlider = Instance.new("TextButton")
TransparencySlider.Size = UDim2.new(0.8, 0, 0.08, 0)
TransparencySlider.Position = UDim2.new(0.1, 0, 0.7, 0)
TransparencySlider.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
TransparencySlider.Text = "Ajustar Transparência (0.5)"
TransparencySlider.TextColor3 = Color3.fromRGB(255, 255, 255)
TransparencySlider.Font = Enum.Font.Gotham
TransparencySlider.TextSize = 12
TransparencySlider.Parent = MainFrame

-- Função para criar hitboxes
local function createHitbox(character)
    if not character:FindFirstChild("HumanoidRootPart") then return end
    
    local hitbox = Instance.new("Part")
    hitbox.Name = "CustomHitbox"
    hitbox.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
    hitbox.Transparency = hitboxTransparency
    hitbox.Color = hitboxColor
    hitbox.Material = Enum.Material.Neon
    hitbox.Anchored = false
    hitbox.CanCollide = false
    hitbox.CFrame = character.HumanoidRootPart.CFrame
    
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = character.HumanoidRootPart
    weld.Part1 = hitbox
    weld.Parent = hitbox
    
    hitbox.Parent = character
end

-- Função para remover hitboxes
local function removeHitbox(character)
    for _, v in pairs(character:GetChildren()) do
        if v.Name == "CustomHitbox" then
            v:Destroy()
        end
    end
end

-- Atualizar hitboxes existentes
local function updateHitboxes()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            removeHitbox(player.Character)
            if hitboxEnabled then
                createHitbox(player.Character)
            end
        end
    end
end

-- Verificar acertos na hitbox
local function checkHit(hit)
    if hit.Name == "CustomHitbox" and hit.Parent and hit.Parent:FindFirstChildOfClass("Humanoid") then
        local enemyPlayer = Players:GetPlayerFromCharacter(hit.Parent)
        if enemyPlayer then
            print("Acertou o jogador:", enemyPlayer.Name)
            -- Aqui você pode adicionar lógica para contar os acertos
        end
    end
end

-- Configurar eventos
mouse.Button1Down:Connect(function()
    if hitboxEnabled then
        local target = mouse.Target
        if target then
            checkHit(target)
        end
    end
end)

-- Atualizar hitboxes quando um jogador spawnar
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if hitboxEnabled then
            createHitbox(character)
        end
    end)
end)

-- Configurar eventos do menu
HitboxToggle.MouseButton1Click:Connect(function()
    hitboxEnabled = not hitboxEnabled
    HitboxToggle.Text = "Hitbox: " .. (hitboxEnabled and "LIGADO" or "DESLIGADO")
    HitboxToggle.BackgroundColor3 = hitboxEnabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    updateHitboxes()
end)

ColorBox.FocusLost:Connect(function()
    local r, g, b = ColorBox.Text:match("(%d+),(%d+),(%d+)")
    if r and g and b then
        hitboxColor = Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b))
        updateHitboxes()
    else
        ColorBox.Text = "255,0,0"
    end
end)

SizeSlider.MouseButton1Click:Connect(function()
    hitboxSize = hitboxSize + 1
    if hitboxSize > 10 then hitboxSize = 1 end
    SizeSlider.Text = "Ajustar Tamanho (" .. hitboxSize .. ")"
    SizeLabel.Text = "Tamanho da Hitbox: " .. hitboxSize
    updateHitboxes()
end)

TransparencySlider.MouseButton1Click:Connect(function()
    hitboxTransparency = hitboxTransparency + 0.1
    if hitboxTransparency > 1 then hitboxTransparency = 0 end
    TransparencySlider.Text = string.format("Ajustar Transparência (%.1f)", hitboxTransparency)
    TransparencyLabel.Text = string.format("Transparência: %.1f", hitboxTransparency)
    updateHitboxes()
end)

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    script:Destroy() -- Encerra o script
end)

-- Inicializar hitboxes para jogadores existentes
for _, player in pairs(Players:GetPlayers()) do
    if player ~= game.Players.LocalPlayer and player.Character then
        removeHitbox(player.Character)
        if hitboxEnabled then
            createHitbox(player.Character)
        end
    end
end

-- Loop para manter as hitboxes atualizadas
while wait(0.1) do
    if hitboxEnabled then
        updateHitboxes()
    end
end
