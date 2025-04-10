-- Configurações iniciais otimizadas
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Variáveis de estado
local settings = {
    enabled = false,
    color = Color3.fromRGB(255, 0, 0),
    size = 5,
    transparency = 0.5,
    visible = true
}

-- Cache de objetos para melhor performance
local cache = {
    hitboxes = {},
    connections = {}
}

-- Criação da GUI otimizada
local function createGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "HitboxExploit"
    ScreenGui.Parent = game:GetService("CoreGui")

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0.25, 0, 0.35, 0)
    MainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui

    -- Elementos da UI (similar ao anterior, mas mais organizado)
    local elements = {
        Title = {
            Type = "TextLabel",
            Props = {
                Size = UDim2.new(1, 0, 0.1, 0),
                Text = "Hitbox Controller",
                BackgroundColor3 = Color3.fromRGB(0, 100, 255)
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
        ToggleButton = {
            Type = "TextButton",
            Props = {
                Size = UDim2.new(0.9, 0, 0.12, 0),
                Position = UDim2.new(0.05, 0, 0.15, 0),
                Text = "Hitbox: OFF",
                BackgroundColor3 = Color3.fromRGB(60, 60, 70)
            }
        }
        -- Adicione outros elementos conforme necessário
    }

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

-- Sistema de hitbox otimizado
local function createHitbox(character)
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

    local weld = Instance.new("WeldConstraint")
    weld.Part0 = character.HumanoidRootPart
    weld.Part1 = hitbox
    weld.Parent = hitbox

    hitbox.Parent = character
    cache.hitboxes[character] = hitbox
end

-- Sistema de detecção de acertos melhorado
local function setupHitDetection()
    -- Conexão para detectar balas/tiros
    local function onProjectileHit(hit)
        if hit.Name == "CustomHitbox" and hit.Parent then
            local humanoid = hit.Parent:FindFirstChildOfClass("Humanoid")
            if humanoid then
                -- Ajuste para garantir que o dano seja aplicado
                humanoid:TakeDamage(25) -- Valor de dano ajustável
                
                -- Atualiza a interface se necessário
                if cache.hitCounter then
                    cache.hitCounter.Text = "Acertos: " .. (tonumber(cache.hitCounter.Text:match("%d+") or 0) + 1)
                end
            end
        end
    end

    -- Monitora novos projéteis no workspace
    workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("BasePart") and descendant.Name == "Bullet" then -- Ajuste conforme o nome dos projéteis no jogo
            descendant.Touched:Connect(onProjectileHit)
        end
    end)
end

-- Controles do menu
local function setupUIEvents(gui)
    cache.ToggleButton.MouseButton1Click:Connect(function()
        settings.enabled = not settings.enabled
        cache.ToggleButton.Text = "Hitbox: " .. (settings.enabled and "ON" or "OFF")
        cache.ToggleButton.BackgroundColor3 = settings.enabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(60, 60, 70)
        
        if settings.enabled then
            setupHitDetection()
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= player and player.Character then
                    createHitbox(player.Character)
                end
            end
        else
            for character, hitbox in pairs(cache.hitboxes) do
                hitbox:Destroy()
            end
            cache.hitboxes = {}
        end
    end)

    cache.CloseButton.MouseButton1Click:Connect(function()
        gui:Destroy()
        for _, connection in pairs(cache.connections) do
            connection:Disconnect()
        end
    end)
end

-- Inicialização otimizada
local function initialize()
    local gui = createGUI()
    setupUIEvents(gui)
    
    -- Monitora novos jogadores
    table.insert(cache.connections, Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function(character)
            if settings.enabled then
                createHitbox(character)
            end
        end)
    end))

    -- Atualização eficiente das hitboxes
    table.insert(cache.connections, RunService.Heartbeat:Connect(function()
        if not settings.enabled then return end
        
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= player and player.Character then
                if not cache.hitboxes[player.Character] then
                    createHitbox(player.Character)
                end
            end
        end
    end))
end

-- Inicia o script
initialize()
