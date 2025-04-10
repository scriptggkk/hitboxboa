-- Configurações iniciais
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Configurações ajustáveis
local settings = {
    enabled = false,
    color = Color3.fromRGB(255, 0, 0),
    size = 5,
    transparency = 0.5,
    refreshRate = 0.2, -- Taxa de atualização (segundos)
    damage = 25 -- Dano aplicado
}

-- Cache para melhor performance
local cache = {
    hitboxes = {},
    connections = {},
    gui = nil
}

-- Função para criar a interface
local function createGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "AdvancedHitboxGUI"
    ScreenGui.Parent = game:GetService("CoreGui")

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0.25, 0, 0.5, 0)
    MainFrame.Position = UDim2.new(0.05, 0, 0.25, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui

    -- Elementos da UI
    local elements = {
        Title = {
            Type = "TextLabel",
            Props = {
                Size = UDim2.new(1, 0, 0.08, 0),
                Text = "CONTROLE DE HITBOX",
                BackgroundColor3 = Color3.fromRGB(0, 100, 255),
                Font = Enum.Font.GothamBold,
                TextSize = 14
            }
        },
        CloseButton = {
            Type = "TextButton",
            Props = {
                Size = UDim2.new(0.1, 0, 0.08, 0),
                Position = UDim2.new(0.9, 0, 0, 0),
                Text = "X",
                BackgroundColor3 = Color3.fromRGB(255, 50, 50),
                TextColor3 = Color3.new(1, 1, 1)
            }
        },
        Toggle = {
            Type = "TextButton",
            Props = {
                Size = UDim2.new(0.9, 0, 0.1, 0),
                Position = UDim2.new(0.05, 0, 0.1, 0),
                Text = "HITBOX: OFF",
                BackgroundColor3 = Color3.fromRGB(60, 60, 70)
            }
        },
        ColorLabel = {
            Type = "TextLabel",
            Props = {
                Size = UDim2.new(0.45, 0, 0.07, 0),
                Position = UDim2.new(0.05, 0, 0.22, 0),
                Text = "Cor (R,G,B):",
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left
            }
        },
        ColorInput = {
            Type = "TextBox",
            Props = {
                Size = UDim2.new(0.45, 0, 0.07, 0),
                Position = UDim2.new(0.5, 0, 0.22, 0),
                Text = "255,0,0",
                BackgroundColor3 = Color3.fromRGB(50, 50, 60)
            }
        },
        SizeLabel = {
            Type = "TextLabel",
            Props = {
                Size = UDim2.new(0.45, 0, 0.07, 0),
                Position = UDim2.new(0.05, 0, 0.32, 0),
                Text = "Tamanho: 5",
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left
            }
        },
        SizeSlider = {
            Type = "TextButton",
            Props = {
                Size = UDim2.new(0.45, 0, 0.07, 0),
                Position = UDim2.new(0.5, 0, 0.32, 0),
                Text = "Ajustar",
                BackgroundColor3 = Color3.fromRGB(50, 50, 60)
            }
        },
        TransparencyLabel = {
            Type = "TextLabel",
            Props = {
                Size = UDim2.new(0.45, 0, 0.07, 0),
                Position = UDim2.new(0.05, 0, 0.42, 0),
                Text = "Transparência: 0.5",
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left
            }
        },
        TransparencySlider = {
            Type = "TextButton",
            Props = {
                Size = UDim2.new(0.45, 0, 0.07, 0),
                Position = UDim2.new(0.5, 0, 0.42, 0),
                Text = "Ajustar",
                BackgroundColor3 = Color3.fromRGB(50, 50, 60)
            }
        },
        HitCounter = {
            Type = "TextLabel",
            Props = {
                Size = UDim2.new(0.9, 0, 0.07, 0),
                Position = UDim2.new(0.05, 0, 0.52, 0),
                Text = "Acertos: 0",
                BackgroundColor3 = Color3.fromRGB(40, 40, 50)
            }
        }
    }

    -- Cria os elementos da UI
    for name, element in pairs(elements) do
        local instance = Instance.new(element.Type)
        for prop, value in pairs(element.Props) do
            instance[prop] = value
        end
        instance.Parent = MainFrame
        cache[name] = instance
    end

    return ScreenGui
end

-- Função para criar/atualizar hitbox
local function updateHitbox(character)
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end

    -- Remove hitbox existente se houver
    if cache.hitboxes[character] then
        cache.hitboxes[character]:Destroy()
    end

    local hitbox = Instance.new("Part")
    hitbox.Name = "CustomHitbox"
    hitbox.Size = Vector3.new(settings.size, settings.size, settings.size)
    hitbox.Transparency = settings.transparency
    hitbox.Color = settings.color
    hitbox.Material = Enum.Material.Neon
    hitbox.Anchored = false
    hitbox.CanCollide = false
    hitbox.CFrame = character.HumanoidRootPart.CFrame

    -- Conecta a hitbox ao jogador
    local weld = Instance.new("WeldConstraint")
    weld.Part0 = character.HumanoidRootPart
    weld.Part1 = hitbox
    weld.Parent = hitbox

    hitbox.Parent = character
    cache.hitboxes[character] = hitbox
end

-- Sistema de detecção de acertos
local function setupHitDetection()
    -- Verifica se o que foi atingido é uma hitbox
    local function onHit(hit)
        if hit.Name == "CustomHitbox" and hit.Parent then
            local humanoid = hit.Parent:FindFirstChildOfClass("Humanoid")
            if humanoid then
                -- Aplica dano ao jogador
                humanoid:TakeDamage(settings.damage)
                
                -- Atualiza contador
                local current = tonumber(cache.HitCounter.Text:match("%d+")) or 0
                cache.HitCounter.Text = "Acertos: " .. (current + 1)
                
                return true -- Indica que acertou
            end
        end
        return false
    end

    -- Monitora projéteis (ajuste conforme o jogo)
    workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("BasePart") and (descendant.Name == "Bullet" or descendant.Name:find("Projectile")) then
            descendant.Touched:Connect(function(hit)
                if onHit(hit) then
                    descendant:Destroy() -- Remove o projétil se acertou
                end
            end)
        end
    end)

    -- Conexão para tiros do jogador local
    mouse.Button1Down:Connect(function()
        if settings.enabled then
            local target = mouse.Target
            if target and onHit(target) then
                -- Feedback visual opcional
            end
        end
    end)
end

-- Configura os eventos da UI
local function setupUIEvents()
    -- Toggle hitbox
    cache.Toggle.MouseButton1Click:Connect(function()
        settings.enabled = not settings.enabled
        cache.Toggle.Text = "HITBOX: " .. (settings.enabled and "ON" or "OFF")
        cache.Toggle.BackgroundColor3 = settings.enabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(60, 60, 70)
        
        if settings.enabled then
            setupHitDetection()
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= player and player.Character then
                    updateHitbox(player.Character)
                end
            end
        else
            for character, hitbox in pairs(cache.hitboxes) do
                hitbox:Destroy()
            end
            cache.hitboxes = {}
        end
    end)

    -- Controle de cor
    cache.ColorInput.FocusLost:Connect(function()
        local r, g, b = cache.ColorInput.Text:match("(%d+),(%d+),(%d+)")
        if r and g and b then
            settings.color = Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b))
            if settings.enabled then
                for character, hitbox in pairs(cache.hitboxes) do
                    if hitbox and hitbox.Parent then
                        hitbox.Color = settings.color
                    end
                end
            end
        else
            cache.ColorInput.Text = "255,0,0"
        end
    end)

    -- Controle de tamanho
    cache.SizeSlider.MouseButton1Click:Connect(function()
        settings.size = settings.size < 10 and settings.size + 1 or 1
        cache.SizeLabel.Text = "Tamanho: " .. settings.size
        if settings.enabled then
            for character, hitbox in pairs(cache.hitboxes) do
                if hitbox and hitbox.Parent then
                    hitbox.Size = Vector3.new(settings.size, settings.size, settings.size)
                end
            end
        end
    end)

    -- Controle de transparência
    cache.TransparencySlider.MouseButton1Click:Connect(function()
        settings.transparency = settings.transparency >= 1 and 0 or settings.transparency + 0.1
        cache.TransparencyLabel.Text = string.format("Transparência: %.1f", settings.transparency)
        if settings.enabled then
            for character, hitbox in pairs(cache.hitboxes) do
                if hitbox and hitbox.Parent then
                    hitbox.Transparency = settings.transparency
                end
            end
        end
    end)

    -- Fechar menu
    cache.CloseButton.MouseButton1Click:Connect(function()
        cache.gui:Destroy()
        for _, conn in pairs(cache.connections) do
            conn:Disconnect()
        end
    end)
end

-- Monitora novos jogadores
local function setupPlayerTracking()
    table.insert(cache.connections, Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function(character)
            if settings.enabled then
                updateHitbox(character)
            end
        end)
    end))
end

-- Atualização otimizada das hitboxes
local function setupUpdateLoop()
    table.insert(cache.connections, RunService.Heartbeat:Connect(function()
        if not settings.enabled then return end
        
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= player and player.Character then
                if not cache.hitboxes[player.Character] then
                    updateHitbox(player.Character)
                end
            end
        end
    end))
end

-- Inicialização principal
local function init()
    cache.gui = createGUI()
    setupUIEvents()
    setupPlayerTracking()
    setupUpdateLoop()
    
    -- Atualiza hitboxes para jogadores existentes
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= player and player.Character then
            updateHitbox(player.Character)
        end
    end
end

-- Inicia o script
init()
