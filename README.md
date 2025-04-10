-- Configurações iniciais
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Variáveis de configuração
local settings = {
    enabled = false,
    color = Color3.fromRGB(255, 0, 0),
    size = 5,
    transparency = 0.5,
    refreshRate = 0.2,  -- Taxa de atualização reduzida para melhor performance
    keybind = Enum.KeyCode.H  -- Tecla para alternar menu
}

-- Cache para otimização
local cache = {
    hitboxes = {},
    connections = {},
    gui = nil,
    lastUpdate = 0
}

-- Função para criar a interface otimizada
local function createOptimizedGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "HitboxV2"
    ScreenGui.Parent = game:GetService("CoreGui")

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0.25, 0, 0.35, 0)
    MainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui

    -- Elementos UI (versão simplificada e mais eficiente)
    local uiElements = {
        Title = {
            Type = "TextLabel",
            Props = {
                Size = UDim2.new(1, 0, 0.1, 0),
                Text = "HITBOX V2 (OTIMIZADO)",
                BackgroundColor3 = Color3.fromRGB(0, 100, 255),
                Font = Enum.Font.GothamBold
            }
        },
        CloseButton = {
            Type = "TextButton",
            Props = {
                Size = UDim2.new(0.1, 0, 0.1, 0),
                Position = UDim2.new(0.9, 0, 0, 0),
                Text = "X",
                BackgroundColor3 = Color3.fromRGB(255, 50, 50)
            }
        },
        Toggle = {
            Type = "TextButton",
            Props = {
                Size = UDim2.new(0.9, 0, 0.1, 0),
                Position = UDim2.new(0.05, 0, 0.15, 0),
                Text = "HITBOX: OFF",
                BackgroundColor3 = Color3.fromRGB(60, 60, 70)
            }
        }
        -- (Adicione outros elementos de controle aqui)
    }

    -- Criação otimizada dos elementos UI
    for name, element in pairs(uiElements) do
        local instance = Instance.new(element.Type)
        for prop, value in pairs(element.Props) do
            instance[prop] = value
        end
        instance.Parent = MainFrame
        cache[name] = instance
    end

    return ScreenGui
end

-- Sistema de hitbox otimizado
local function manageHitbox(character, create)
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end

    if create then
        -- Remove hitbox existente antes de criar nova
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

        local weld = Instance.new("WeldConstraint")
        weld.Part0 = character.HumanoidRootPart
        weld.Part1 = hitbox
        weld.Parent = hitbox

        hitbox.Parent = character
        cache.hitboxes[character] = hitbox
    elseif cache.hitboxes[character] then
        cache.hitboxes[character]:Destroy()
        cache.hitboxes[character] = nil
    end
end

-- Sistema de detecção de acertos melhorado
local function setupHitDetection()
    local function onHit(hit)
        if hit.Name == "CustomHitbox" and hit.Parent then
            local humanoid = hit.Parent:FindFirstChildOfClass("Humanoid")
            if humanoid then
                -- Aplica dano ao jogador
                humanoid:TakeDamage(25)
                return true
            end
        end
        return false
    end

    -- Conexão otimizada para detecção de tiros
    table.insert(cache.connections, workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("BasePart") and (descendant.Name == "Bullet" or descendant.Name:find("Projectile")) then
            descendant.Touched:Connect(function(hit)
                if onHit(hit) then
                    descendant:Destroy()
                end
            end)
        end
    end))
end

-- Atualização otimizada das hitboxes
local function updateHitboxes()
    local now = tick()
    if now - cache.lastUpdate < settings.refreshRate then return end
    cache.lastUpdate = now

    for _, targetPlayer in pairs(Players:GetPlayers()) do
        if targetPlayer ~= player and targetPlayer.Character then
            manageHitbox(targetPlayer.Character, settings.enabled)
        end
    end
end

-- Configuração dos eventos da interface
local function setupUIEvents()
    cache.Toggle.MouseButton1Click:Connect(function()
        settings.enabled = not settings.enabled
        cache.Toggle.Text = "HITBOX: " .. (settings.enabled and "ON" or "OFF")
        cache.Toggle.BackgroundColor3 = settings.enabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(60, 60, 70)
        
        if settings.enabled then
            setupHitDetection()
        else
            for character, _ in pairs(cache.hitboxes) do
                manageHitbox(character, false)
            end
        end
    end)

    cache.CloseButton.MouseButton1Click:Connect(function()
        cache.gui:Destroy()
        for _, conn in pairs(cache.connections) do
            conn:Disconnect()
        end
        script:Destroy()
    end)
end

-- Inicialização principal
local function init()
    cache.gui = createOptimizedGUI()
    setupUIEvents()

    -- Conexão para novos jogadores
    table.insert(cache.connections, Players.PlayerAdded:Connect(function(newPlayer)
        newPlayer.CharacterAdded:Connect(function(character)
            if settings.enabled then
                manageHitbox(character, true)
            end
        end)
    end))

    -- Loop de atualização otimizado
    table.insert(cache.connections, RunService.Heartbeat:Connect(function()
        if settings.enabled then
            updateHitboxes()
        end
    end))

    -- Inicializa para jogadores existentes
    for _, existingPlayer in pairs(Players:GetPlayers()) do
        if existingPlayer ~= player and existingPlayer.Character then
            manageHitbox(existingPlayer.Character, settings.enabled)
        end
    end
end

-- Inicia o script
init()
