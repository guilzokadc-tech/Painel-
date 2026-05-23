-- Painel de Controle - Script UI Educacional
local Player = game.Players.LocalPlayer
local Mouse = Player:GetMouse()
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Criando a Interface Principal
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local Container = Instance.new("Frame")
local UIListLayout = Instance.new("UIListLayout")
local ToggleButton = Instance.new("TextButton") -- Botão para abrir/fechar

ScreenGui.Name = "ModMenu"
ScreenGui.Parent = game.CoreGui -- Tenta colocar no CoreGui (comum em executors)
ScreenGui.ResetOnSpawn = false

-- Configuração do Botão de Abrir/Fechar (Fixo na tela)
ToggleButton.Name = "ToggleButton"
ToggleButton.Parent = ScreenGui
ToggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ToggleButton.Position = UDim2.new(0, 10, 0.5, 0)
ToggleButton.Size = UDim2.new(0, 50, 0, 50)
ToggleButton.Text = "MENU"
ToggleButton.TextColor3 = Color3.new(1, 1, 1)

-- Configuração do Painel Principal
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.Position = UDim2.new(0.5, -100, 0.5, -150)
MainFrame.Size = UDim2.new(0, 200, 0, 300)
MainFrame.Active = true
MainFrame.Draggable = true -- Nota: Draggable é antigo, mas funciona para exemplos simples

Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "SISTEMA V1"
Title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Title.TextColor3 = Color3.new(1, 1, 1)

Container.Parent = MainFrame
Container.Position = UDim2.new(0, 0, 0, 40)
Container.Size = UDim2.new(1, 0, 1, -40)
Container.BackgroundTransparency = 1

UIListLayout.Parent = Container
UIListLayout.Padding = UDim.new(0, 5)
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Variáveis de Estado
local States = {
    ESP = false,
    Aimbot = false,
    Fly = false,
    Noclip = false
}

-- Função para Criar Botões de Opção
local function CreateToggle(name, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = Container
    btn.Size = UDim2.new(0, 180, 0, 35)
    btn.Text = name .. ": OFF"
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.TextColor3 = Color3.new(1, 1, 1)
    
    btn.MouseButton1Click:Connect(function()
        States[name] = not States[name]
        btn.Text = name .. ": " .. (States[name] and "ON" or "OFF")
        btn.BackgroundColor3 = States[name] and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(60, 60, 60)
        callback(States[name])
    end)
end

-- Lógica do Toggle do Menu
ToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-----------------------------------------------------------
-- FUNCIONALIDADES
-----------------------------------------------------------

-- 1. NOCLIP
RunService.Stepped:Connect(function()
    if States.Noclip then
        if Player.Character then
            for _, part in pairs(Player.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end
end)

-- 2. FLY
local flySpeed = 50
RunService.RenderStepped:Connect(function()
    if States.Fly and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = Player.Character.HumanoidRootPart
        local moveDir = Vector3.new(0,0,0)
        
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir += workspace.CurrentCamera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir -= workspace.CurrentCamera.CFrame.LookVector end
        
        hrp.Velocity = moveDir * flySpeed
    end
end)

-- 3. ESP (Highlights simples)
local function updateESP()
    for _, p in pairs(game.Players:GetPlayers()) do
        if p ~= Player and p.Character then
            local highlight = p.Character:FindFirstChild("ESPHighlight")
            if States.ESP then
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "ESPHighlight"
                    highlight.Parent = p.Character
                    highlight.FillColor = Color3.new(1, 0, 0)
                end
            else
                if highlight then highlight:Destroy() end
            end
        end
    end
end
spawn(function()
    while wait(1) do updateESP() end
end)

-- 4. AIMBOT (Básico para o jogador mais próximo)
RunService.RenderStepped:Connect(function()
    if States.Aimbot then
        local closest = nil
        local dist = math.huge
        
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= Player and p.Character and p.Character:FindFirstChild("Head") then
                local pos = workspace.CurrentCamera:WorldToViewportPoint(p.Character.Head.Position)
                local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                local magnitude = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
                
                if magnitude < dist and magnitude < 400 then -- 400 é o raio do FOV
                    dist = magnitude
                    closest = p.Character.Head
                end
            end
        end
        
        if closest then
            workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, closest.Position)
        end
    end
end)

-----------------------------------------------------------
-- INICIALIZAÇÃO DOS BOTÕES
-----------------------------------------------------------
CreateToggle("ESP", function() end)
CreateToggle("Aimbot", function() end)
CreateToggle("Fly", function() end)
CreateToggle("Noclip", function() end)

print("Painel Carregado com Sucesso!")
