local Player = game:GetService("Players").LocalPlayer
local Char = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Char:WaitForChild("Humanoid")
local Root = Char:WaitForChild("HumanoidRootPart")

-- GUI básica
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 400, 0, 450) -- Aumentei a altura para acomodar novas funções
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -225)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)

local Title = Instance.new("TextLabel", MainFrame)
Title.Text = "STAFF DEVELOPER"
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

local MinimizeButton = Instance.new("TextButton", MainFrame)
MinimizeButton.Text = "─"
MinimizeButton.Size = UDim2.new(0, 35, 0, 35)
MinimizeButton.Position = UDim2.new(1, -35, 0, 0)

local MinimizedFrame = Instance.new("Frame", ScreenGui)
MinimizedFrame.Size = UDim2.new(0, 400, 0, 35)
MinimizedFrame.Position = MainFrame.Position
MinimizedFrame.Visible = false

local MinimizedTitle = Instance.new("TextLabel", MinimizedFrame)
MinimizedTitle.Text = "STAFF DEVELOPER"
MinimizedTitle.Size = UDim2.new(1, 0, 1, 0)
MinimizedTitle.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

local MaximizeButton = Instance.new("TextButton", MinimizedFrame)
MaximizeButton.Text = "+"
MaximizeButton.Size = UDim2.new(0, 35, 0, 35)
MaximizeButton.Position = UDim2.new(1, -35, 0, 0)

-- Tabs
local Tabs = {"MAIN", "FUNÇÕES"}
local TabButtons = {}
local TabsFrame = Instance.new("Frame", MainFrame)
TabsFrame.Size = UDim2.new(1, 0, 0, 30)
TabsFrame.Position = UDim2.new(0, 0, 0, 35)
TabsFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

for i, name in pairs(Tabs) do
    local TabBtn = Instance.new("TextButton", TabsFrame)
    TabBtn.Text = name
    TabBtn.Size = UDim2.new(0.5, 0, 1, 0)
    TabBtn.Position = UDim2.new((i-1)/2, 0, 0, 0)
    TabBtn.BackgroundColor3 = name == "MAIN" and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(50, 50, 50)
    TabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    TabBtn.Font = Enum.Font.Gotham
    TabButtons[name] = TabBtn
end

local ContentFrame = Instance.new("Frame", MainFrame)
ContentFrame.Size = UDim2.new(1, -20, 1, -90)
ContentFrame.Position = UDim2.new(0, 10, 0, 75)
ContentFrame.BackgroundTransparency = 1

-- Funções auxiliares
function CreateOptionRow(parent, y, labelText, defaultValue, hasValueBox)
    local Row = Instance.new("Frame", parent)
    Row.Size = UDim2.new(1, 0, 0, 30)
    Row.Position = UDim2.new(0, 0, 0, y)
    Row.BackgroundTransparency = 1
    
    local nameWidth = hasValueBox and 0.6 or 0.8
    local NameLabel = Instance.new("TextLabel", Row)
    NameLabel.Text = labelText
    NameLabel.Size = UDim2.new(nameWidth, 0, 1, 0)
    NameLabel.BackgroundTransparency = 1
    NameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    NameLabel.Font = Enum.Font.Gotham
    NameLabel.TextXAlignment = Enum.TextXAlignment.Left
    
    local ToggleBtn = Instance.new("TextButton", Row)
    ToggleBtn.Text = "OFF"
    ToggleBtn.Size = UDim2.new(0.2, -5, 1, 0)
    ToggleBtn.Position = UDim2.new(nameWidth, 0, 0, 0)
    ToggleBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    ToggleBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
    ToggleBtn.Font = Enum.Font.GothamBold
    
    if hasValueBox then
        local ValueBox = Instance.new("TextBox", Row)
        ValueBox.PlaceholderText = defaultValue
        ValueBox.Text = defaultValue
        ValueBox.Size = UDim2.new(0.2, 5, 1, 0)
        ValueBox.Position = UDim2.new(0.8, 0, 0, 0)
        ValueBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        ValueBox.TextColor3 = Color3.fromRGB(255, 255, 255)
        ValueBox.Font = Enum.Font.Gotham
        return ToggleBtn, ValueBox
    end
    return ToggleBtn
end

function UpdateToggleButton(button, active)
    button.Text = active and "ON" or "OFF"
    button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 255)
    button.TextColor3 = active and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(0, 0, 0)
end

-- Variáveis de controle
local NoclipActive, WalkSpeedActive, FlyActive, InfJumpActive, FarmActive, AntiAFKActive = false, false, false, false, false, false
local FlingActive, GrudarActive, AimbotActive, VisualizarActive, ESPActive, TrazerActive = false, false, false, false, false, false
local OriginalWalkSpeed = 16
local GrudarTarget, GrudarConnection, AimbotConnection, VisualizarConnection, ESPBillboards = {}, TrazerConnection

-- NOVA FUNÇÃO: TRAZER
function ToggleTrazer(active, targetName)
    TrazerActive = active
    
    if TrazerActive then
        local target = game:GetService("Players"):FindFirstChild(targetName)
        if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then
            TrazerActive = false
            UpdateToggleButton(TrazerToggle, false)
            return
        end
        
        TrazerConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not TrazerActive then return end
            
            local target = game:GetService("Players"):FindFirstChild(targetName)
            if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then
                ToggleTrazer(false)
                return
            end
            
            local targetRoot = target.Character.HumanoidRootPart
            local yourPosition = Root.Position
            
            -- Teleporta a pessoa para sua posição atual
            targetRoot.CFrame = CFrame.new(yourPosition + Vector3.new(0, 3, 0))
            
            -- Impede que a pessoa se afaste muito
            local distance = (targetRoot.Position - yourPosition).Magnitude
            if distance > 10 then
                targetRoot.CFrame = CFrame.new(yourPosition + Vector3.new(0, 3, 0))
            end
        end)
    else
        if TrazerConnection then 
            TrazerConnection:Disconnect() 
            TrazerConnection = nil
        end
    end
end

-- NOVA FUNÇÃO: ESP
function ToggleESP(active)
    ESPActive = active
    
    if ESPActive then
        -- Limpa billboards antigos
        for _, billboard in pairs(ESPBillboards) do
            billboard:Destroy()
        end
        ESPBillboards = {}
        
        -- Cria ESP para todos os jogadores
        for _, player in pairs(game:GetService("Players"):GetPlayers()) do
            if player ~= Player and player.Character then
                local char = player.Character
                local head = char:FindFirstChild("Head")
                
                if head then
                    -- Cria o billboard (texto com nome)
                    local billboard = Instance.new("BillboardGui")
                    billboard.Name = "ESPBillboard"
                    billboard.Adornee = head
                    billboard.Size = UDim2.new(0, 200, 0, 50)
                    billboard.StudsOffset = Vector3.new(0, 3, 0)
                    billboard.AlwaysOnTop = true
                    billboard.Parent = head
                    
                    local textLabel = Instance.new("TextLabel", billboard)
                    textLabel.Size = UDim2.new(1, 0, 1, 0)
                    textLabel.Text = player.Name
                    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                    textLabel.BackgroundTransparency = 1
                    textLabel.TextStrokeTransparency = 0
                    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                    textLabel.Font = Enum.Font.GothamBold
                    textLabel.TextScaled = true
                    
                    table.insert(ESPBillboards, billboard)
                    
                    -- Cria highlight (contorno branco)
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "ESPHighlight"
                    highlight.FillColor = Color3.fromRGB(255, 255, 255)
                    highlight.FillTransparency = 0.8
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.OutlineTransparency = 0
                    highlight.Parent = char
                end
            end
        end
        
        -- Conecta para novos jogadores
        local playerAddedConnection = game:GetService("Players").PlayerAdded:Connect(function(player)
            if ESPActive then
                player.CharacterAdded:Connect(function(char)
                    wait(1)
                    if ESPActive and char:FindFirstChild("Head") then
                        local head = char:FindFirstChild("Head")
                        
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = "ESPBillboard"
                        billboard.Adornee = head
                        billboard.Size = UDim2.new(0, 200, 0, 50)
                        billboard.StudsOffset = Vector3.new(0, 3, 0)
                        billboard.AlwaysOnTop = true
                        billboard.Parent = head
                        
                        local textLabel = Instance.new("TextLabel", billboard)
                        textLabel.Size = UDim2.new(1, 0, 1, 0)
                        textLabel.Text = player.Name
                        textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                        textLabel.BackgroundTransparency = 1
                        textLabel.TextStrokeTransparency = 0
                        textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                        textLabel.Font = Enum.Font.GothamBold
                        textLabel.TextScaled = true
                        
                        table.insert(ESPBillboards, billboard)
                        
                        local highlight = Instance.new("Highlight")
                        highlight.Name = "ESPHighlight"
                        highlight.FillColor = Color3.fromRGB(255, 255, 255)
                        highlight.FillTransparency = 0.8
                        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                        highlight.OutlineTransparency = 0
                        highlight.Parent = char
                    end
                end)
            end
        end)
        
        -- Mantém a conexão enquanto ESP estiver ativo
        spawn(function()
            while ESPActive do wait() end
            playerAddedConnection:Disconnect()
        end)
    else
        -- Remove todos os ESPs
        for _, billboard in pairs(ESPBillboards) do
            billboard:Destroy()
        end
        ESPBillboards = {}
        
        -- Remove todos os highlights
        for _, player in pairs(game:GetService("Players"):GetPlayers()) do
            if player.Character then
                local highlight = player.Character:FindFirstChild("ESPHighlight")
                if highlight then
                    highlight:Destroy()
                end
            end
        end
    end
end

-- NOVA FUNÇÃO: VISUALIZAR
function ToggleVisualizar(active, targetName)
    VisualizarActive = active
    
    if VisualizarActive then
        local target = game:GetService("Players"):FindFirstChild(targetName)
        if not target or not target.Character then
            VisualizarActive = false
            UpdateToggleButton(VisualizarToggle, false)
            return
        end
        
        -- Salva a câmera original
        local originalCamera = workspace.CurrentCamera
        
        VisualizarConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not VisualizarActive then return end
            
            local target = game:GetService("Players"):FindFirstChild(targetName)
            if not target or not target.Character then
                ToggleVisualizar(false)
                return
            end
            
            local targetChar = target.Character
            local head = targetChar:FindFirstChild("Head")
            
            if head then
                -- Anexa a câmera à cabeça do alvo
                workspace.CurrentCamera.CameraSubject = head
                workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
                
                -- Posiciona a câmera atrás da cabeça (como se estivesse vendo pelos olhos)
                local offset = CFrame.new(0, 0, 0.5) -- Pequeno ajuste para visualização
                workspace.CurrentCamera.CFrame = head.CFrame * offset
            end
        end)
    else
        if VisualizarConnection then 
            VisualizarConnection:Disconnect() 
            VisualizarConnection = nil
        end
        
        -- Restaura a câmera original
        workspace.CurrentCamera.CameraSubject = Char.Humanoid
        workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
    end
end

-- AIMBOT ENCOSTADO NA CABEÇA
function ToggleAimbot(active, targetName)
    AimbotActive = active
    
    if AimbotActive then
        local targetPlayer = game:GetService("Players"):FindFirstChild(targetName)
        if not targetPlayer or not targetPlayer.Character or not targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            AimbotActive = false
            UpdateToggleButton(AimbotToggle, false)
            return
        end
        
        AimbotConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not AimbotActive then return end
            
            local targetPlayer = game:GetService("Players"):FindFirstChild(targetName)
            if not targetPlayer or not targetPlayer.Character then
                ToggleAimbot(false)
                return
            end
            
            local targetChar = targetPlayer.Character
            local head = targetChar:FindFirstChild("Head")
            local targetRoot = targetChar.HumanoidRootPart
            
            if head then
                local positionOnHead = head.Position + Vector3.new(0, 0.2, 0)
                local yourHeight = 5
                local footPosition = positionOnHead - Vector3.new(0, yourHeight - 0.5, 0)
                Root.CFrame = CFrame.new(footPosition, head.Position)
                Root.CFrame = Root.CFrame * CFrame.Angles(math.rad(-10), 0, 0)
                
                if not NoclipActive then
                    for _, part in pairs(Char:GetDescendants()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                end
            else
                local positionVeryClose = targetRoot.Position + Vector3.new(0, 0.5, 0)
                Root.CFrame = CFrame.new(positionVeryClose, targetRoot.Position)
            end
        end)
    else
        if AimbotConnection then 
            AimbotConnection:Disconnect() 
            AimbotConnection = nil
        end
        
        if not NoclipActive then
            wait(0.2)
            for _, part in pairs(Char:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide = true end
            end
        end
    end
end

-- GRUDAR
function ToggleGrudar(active, targetName)
    GrudarActive = active
    
    if GrudarActive then
        local target = game:GetService("Players"):FindFirstChild(targetName)
        if not target then return end
        
        GrudarTarget = target
        local TargetRoot = target.Character.HumanoidRootPart
        
        Humanoid:ChangeState(Enum.HumanoidStateType.Flying)
        local BV = Instance.new("BodyVelocity", Root)
        BV.MaxForce = Vector3.new(100000, 100000, 100000)
        BV.P = 10000
        
        Humanoid.AutoRotate = false
        
        GrudarConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not GrudarActive or not GrudarTarget then
                ToggleGrudar(false)
                return
            end
            
            local TargetPos = GrudarTarget.Character.HumanoidRootPart.Position
            local CurrentPos = Root.Position
            local Distance = (TargetPos - CurrentPos).Magnitude
            
            if Distance > 5 then
                local Direction = (TargetPos - CurrentPos).Unit
                BV.Velocity = Direction * 100
            else
                BV.Velocity = Vector3.new(0, 0, 0)
                Root.CFrame = CFrame.new(CurrentPos, TargetPos)
            end
        end)
    else
        if GrudarConnection then GrudarConnection:Disconnect() end
        GrudarTarget = nil
        Humanoid:ChangeState(Enum.HumanoidStateType.Running)
        Humanoid.AutoRotate = true
        
        for _, part in pairs(Char:GetDescendants()) do
            if part:IsA("BodyVelocity") or part:IsA("BodyGyro") then
                part:Destroy()
            end
        end
    end
end

-- NO CLIP
function ToggleNoClip(active)
    NoclipActive = active
    spawn(function()
        while NoclipActive do
            wait(0.1)
            for _, part in pairs(Char:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end
        wait(0.2)
        for _, part in pairs(Char:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = true end
        end
    end)
end

-- WALK SPEED
function ToggleWalkSpeed(active, speed)
    WalkSpeedActive = active
    if WalkSpeedActive then
        OriginalWalkSpeed = Humanoid.WalkSpeed
        local newSpeed = math.clamp(tonumber(speed) or 500, 1, 1000)
        spawn(function()
            while WalkSpeedActive do
                if Humanoid then Humanoid.WalkSpeed = newSpeed end
                wait(0.1)
            end
            if Humanoid then Humanoid.WalkSpeed = OriginalWalkSpeed end
        end)
    elseif Humanoid then
        Humanoid.WalkSpeed = OriginalWalkSpeed
    end
end

-- INFINITY JUMP
function ToggleInfinityJump(active, height)
    InfJumpActive = active
    if InfJumpActive then
        local jumpPower = math.clamp(tonumber(height) or 100, 0, 500)
        local jumpConnection = game:GetService("UserInputService").JumpRequest:Connect(function()
            if InfJumpActive then
                Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                Root.Velocity = Vector3.new(Root.Velocity.X, jumpPower, Root.Velocity.Z)
            end
        end)
        spawn(function()
            while InfJumpActive do wait() end
            if jumpConnection then jumpConnection:Disconnect() end
        end)
    end
end

-- FLY
function ToggleFly(active, speed)
    FlyActive = active
    if FlyActive then
        local BV = Instance.new("BodyVelocity", Root)
        local BG = Instance.new("BodyGyro", Root)
        BV.MaxForce = Vector3.new(100000, 100000, 100000)
        BG.MaxTorque = Vector3.new(100000, 100000, 100000)
        
        local flyConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not FlyActive then
                if BV then BV:Destroy() end
                if BG then BG:Destroy() end
                return
            end
        end)
    else
        for _, part in pairs(Char:GetDescendants()) do
            if part:IsA("BodyVelocity") or part:IsA("BodyGyro") then
                part:Destroy()
            end
        end
    end
end

-- FLING
function ToggleFling(active, speed)
    FlingActive = active
    if FlingActive then
        local flingSpeed = math.clamp(tonumber(speed) or 50, 1, 100)
        local BA = Instance.new("BodyAngularVelocity", Root)
        BA.MaxTorque = Vector3.new(100000, 100000, 100000)
        BA.AngularVelocity = Vector3.new(0, flingSpeed * 10, 0)
        
        spawn(function()
            while FlingActive do wait() end
            BA:Destroy()
        end)
    else
        for _, part in pairs(Char:GetDescendants()) do
            if part:IsA("BodyAngularVelocity") then
                part:Destroy()
            end
        end
    end
end

-- Conteúdo MAIN
local MainContent = Instance.new("ScrollingFrame", ContentFrame)
MainContent.Size = UDim2.new(1, 0, 1, 0)
MainContent.BackgroundTransparency = 1
MainContent.ScrollBarThickness = 5
MainContent.Visible = true

-- Opções MAIN (incluindo novas funções)
local GrudarToggle, GrudarValue = CreateOptionRow(MainContent, 10, "Grudar", "Nome", true)
GrudarValue.PlaceholderText = "Nome do Jogador"
GrudarToggle.MouseButton1Click:Connect(function()
    GrudarActive = not GrudarActive
    UpdateToggleButton(GrudarToggle, GrudarActive)
    ToggleGrudar(GrudarActive, GrudarValue.Text)
end)

-- NOVA FUNÇÃO: TRAZER
local TrazerToggle, TrazerValue = CreateOptionRow(MainContent, 50, "Trazer", "Nome", true)
TrazerValue.PlaceholderText = "Nome do Jogador"
TrazerToggle.MouseButton1Click:Connect(function()
    TrazerActive = not TrazerActive
    UpdateToggleButton(TrazerToggle, TrazerActive)
    ToggleTrazer(TrazerActive, TrazerValue.Text)
end)

local FarmToggle = CreateOptionRow(MainContent, 90, "Farmar AFK", "", false)
FarmToggle.MouseButton1Click:Connect(function()
    FarmActive = not FarmActive
    UpdateToggleButton(FarmToggle, FarmActive)
    if FarmActive then
        spawn(function()
            local targetPos = Root.Position + Vector3.new(math.random(-50, 50), 20, math.random(-50, 50))
            Root.CFrame = CFrame.new(targetPos)
            local BV = Instance.new("BodyVelocity", Root)
            BV.MaxForce = Vector3.new(0, 10000, 0)
            while FarmActive do wait(1) end
            BV:Destroy()
        end)
    end
end)

local TeleportToggle, TeleportValue = CreateOptionRow(MainContent, 130, "Teleport Player", "Nome", true)
TeleportToggle.MouseButton1Click:Connect(function()
    local target = game:GetService("Players"):FindFirstChild(TeleportValue.Text)
    if target then
        Root.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 5)
    end
end)

local AntiAFKToggle = CreateOptionRow(MainContent, 170, "Anti-AFK", "", false)
AntiAFKToggle.MouseButton1Click:Connect(function()
    AntiAFKActive = not AntiAFKActive
    UpdateToggleButton(AntiAFKToggle, AntiAFKActive)
    if AntiAFKActive then
        spawn(function()
            while AntiAFKActive do
                wait(1200)
                if AntiAFKActive then
                    game:GetService("VirtualUser"):CaptureController()
                    game:GetService("VirtualUser"):ClickButton2(Vector2.new())
                end
            end
        end)
    end
end)

MainContent.CanvasSize = UDim2.new(0, 0, 0, 210)

-- Conteúdo FUNÇÕES
local FunctionsContent = Instance.new("ScrollingFrame", ContentFrame)
FunctionsContent.Size = UDim2.new(1, 0, 1, 0)
FunctionsContent.BackgroundTransparency = 1
FunctionsContent.ScrollBarThickness = 5
FunctionsContent.Visible = false

-- NOVA FUNÇÃO: VISUALIZAR
local VisualizarToggle, VisualizarValue = CreateOptionRow(FunctionsContent, 10, "Visualizar", "Nome", true)
VisualizarValue.PlaceholderText = "Nome do Jogador"
VisualizarToggle.MouseButton1Click:Connect(function()
    VisualizarActive = not VisualizarActive
    UpdateToggleButton(VisualizarToggle, VisualizarActive)
    ToggleVisualizar(VisualizarActive, VisualizarValue.Text)
end)

VisualizarValue:GetPropertyChangedSignal("Text"):Connect(function()
    if VisualizarActive then
        ToggleVisualizar(false)
        wait(0.1)
        ToggleVisualizar(true, VisualizarValue.Text)
        UpdateToggleButton(VisualizarToggle, true)
    end
end)

-- NOVA FUNÇÃO: ESP
local ESPToggle = CreateOptionRow(FunctionsContent, 50, "ESP", "", false)
ESPToggle.MouseButton1Click:Connect(function()
    ESPActive = not ESPActive
    UpdateToggleButton(ESPToggle, ESPActive)
    ToggleESP(ESPActive)
end)

-- AIMBOT
local AimbotToggle, AimbotValue = CreateOptionRow(FunctionsContent, 90, "Aimbot Soco", "Nome", true)
AimbotValue.PlaceholderText = "Nome do Jogador"
AimbotToggle.MouseButton1Click:Connect(function()
    AimbotActive = not AimbotActive
    UpdateToggleButton(AimbotToggle, AimbotActive)
    ToggleAimbot(AimbotActive, AimbotValue.Text)
end)

AimbotValue:GetPropertyChangedSignal("Text"):Connect(function()
    if AimbotActive then
        ToggleAimbot(false)
        wait(0.1)
        ToggleAimbot(true, AimbotValue.Text)
        UpdateToggleButton(AimbotToggle, true)
    end
end)

local NoClipToggle = CreateOptionRow(FunctionsContent, 130, "No Clip", "", false)
NoClipToggle.MouseButton1Click:Connect(function()
    NoclipActive = not NoclipActive
    UpdateToggleButton(NoClipToggle, NoclipActive)
    ToggleNoClip(NoclipActive)
end)

local InfJumpToggle, InfJumpValue = CreateOptionRow(FunctionsContent, 170, "Infinity Jump", "100", true)
InfJumpToggle.MouseButton1Click:Connect(function()
    InfJumpActive = not InfJumpActive
    UpdateToggleButton(InfJumpToggle, InfJumpActive)
    ToggleInfinityJump(InfJumpActive, InfJumpValue.Text)
end)

local WalkSpeedToggle, WalkSpeedValue = CreateOptionRow(FunctionsContent, 210, "Walk Speed", "500", true)
WalkSpeedToggle.MouseButton1Click:Connect(function()
    WalkSpeedActive = not WalkSpeedActive
    UpdateToggleButton(WalkSpeedToggle, WalkSpeedActive)
    ToggleWalkSpeed(WalkSpeedActive, WalkSpeedValue.Text)
end)

local FlyToggle, FlyValue = CreateOptionRow(FunctionsContent, 250, "Fly", "500", true)
FlyToggle.MouseButton1Click:Connect(function()
    FlyActive = not FlyActive
    UpdateToggleButton(FlyToggle, FlyActive)
    ToggleFly(FlyActive, FlyValue.Text)
end)

local FlingToggle, FlingValue = CreateOptionRow(FunctionsContent, 290, "Fling", "50", true)
FlingToggle.MouseButton1Click:Connect(function()
    FlingActive = not FlingActive
    UpdateToggleButton(FlingToggle, FlingActive)
    ToggleFling(FlingActive, FlingValue.Text)
end)

FunctionsContent.CanvasSize = UDim2.new(0, 0, 0, 340)

-- Navegação entre abas
for name, btn in pairs(TabButtons) do
    btn.MouseButton1Click:Connect(function()
        for n, b in pairs(TabButtons) do
            b.BackgroundColor3 = n == name and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(50, 50, 50)
        end
        MainContent.Visible = name == "MAIN"
        FunctionsContent.Visible = name == "FUNÇÕES"
    end
end

-- Sistema de arrastar
local dragging, dragStart, startPos
Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)

MinimizedTitle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MinimizedFrame.Position
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local frame = MainFrame.Visible and MainFrame or MinimizedFrame
        frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + (input.Position.X - dragStart.X),
            startPos.Y.Scale, startPos.Y.Offset + (input.Position.Y - dragStart.Y)
        )
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- Minimizar/Maximizar
MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    MinimizedFrame.Visible = true
    MinimizedFrame.Position = MainFrame.Position
end)

MaximizeButton.MouseButton1Click:Connect(function()
    MinimizedFrame.Visible = false
    MainFrame.Visible = true
    MainFrame.Position = MinimizedFrame.Position
end)

-- Resetar ao trocar de personagem
Player.CharacterAdded:Connect(function(newChar)
    Char = newChar
    Humanoid = newChar:WaitForChild("Humanoid")
    Root = newChar:WaitForChild("HumanoidRootPart")
    wait(1)
    if GrudarActive then ToggleGrudar(true, GrudarValue.Text) end
    if AimbotActive then ToggleAimbot(true, AimbotValue.Text) end
    if NoclipActive then ToggleNoClip(true) end
    if WalkSpeedActive then ToggleWalkSpeed(true, WalkSpeedValue.Text) end
    if FlyActive then ToggleFly(true, FlyValue.Text) end
    if InfJumpActive then ToggleInfinityJump(true, InfJumpValue.Text) end
    if FlingActive then ToggleFling(true, FlingValue.Text) end
    if VisualizarActive then ToggleVisualizar(true, VisualizarValue.Text) end
    if ESPActive then ToggleESP(true) end
    if TrazerActive then ToggleTrazer(true, TrazerValue.Text) end
end)

print("Staff Developer Menu carregado! Novas funções: Visualizar, ESP e Trazer adicionadas!")
