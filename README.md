--// Xinread Premium GUI Compacta (Speed toggle e fundo de carregamento menor)
--// LocalScript - by ChatGPT

local function create(class, props)
    local obj = Instance.new(class)
    for k, v in pairs(props) do
        obj[k] = v
    end
    return obj
end

local player = game.Players.LocalPlayer
local gui = create("ScreenGui", {
    Name = "XinreadPremium",
    Parent = player:WaitForChild("PlayerGui"),
    ResetOnSpawn = false
})

-- Fundo de carregamento menor (compacto)
local loadingBackground = create("Frame", {
    Parent = gui,
    BackgroundColor3 = Color3.fromRGB(255,0,0),
    Size = UDim2.new(0,200,0,70), -- fundo menor
    AnchorPoint = Vector2.new(0.5,0.5),
    Position = UDim2.new(0.5,0,0.5,0),
    BorderSizePixel = 0,
})
create("UICorner", {Parent = loadingBackground, CornerRadius = UDim.new(0,8)})

-- Texto de carregamento
local loading = create("TextLabel", {
    Parent = loadingBackground,
    Text = "Carregando... 0%",
    Font = Enum.Font.GothamBold,
    TextSize = 20,
    BackgroundTransparency = 1,
    TextColor3 = Color3.new(1,1,1),
    AnchorPoint = Vector2.new(0.5,0),
    Position = UDim2.new(0.5,0,0,10)
})

-- Botão SKIP
local skipButton = create("TextButton", {
    Parent = loadingBackground,
    Text = "SKIP",
    Font = Enum.Font.GothamBold,
    TextSize = 18,
    BackgroundColor3 = Color3.fromRGB(0,0,0),
    TextColor3 = Color3.new(1,1,1),
    AnchorPoint = Vector2.new(0.5,0),
    Position = UDim2.new(0.5,0,0,40),
    Size = UDim2.new(0,70,0,25),
})
create("UICorner", {Parent = skipButton, CornerRadius = UDim.new(0,6)})

-- Carregamento animado
local loadingDone = false
task.spawn(function()
    for i = 0,100 do
        if loadingDone then break end
        loading.Text = "Carregando... "..i.."%"
        task.wait(0.05)
    end
    loadingDone = true
end)

skipButton.MouseButton1Click:Connect(function()
    loadingDone = true
end)

repeat task.wait() until loadingDone
loadingBackground:Destroy() -- remove o fundo após o carregamento

-- MENU PRINCIPAL COMPACTO
local main = create("Frame", {
    Parent = gui,
    BackgroundTransparency = 0.3,
    BackgroundColor3 = Color3.fromRGB(25,25,25),
    BorderSizePixel = 0,
    AnchorPoint = Vector2.new(0.5,0.5),
    Position = UDim2.new(0.5,0,0.5,0),
    Size = UDim2.new(0,160,0,140)
})
create("UICorner", {Parent = main, CornerRadius = UDim.new(0,10)})

local top = create("TextLabel", {
    Parent = main,
    Text = "Xinread",
    Font = Enum.Font.GothamBold,
    TextSize = 20,
    TextColor3 = Color3.fromRGB(255,0,0),
    BackgroundTransparency = 1,
    Size = UDim2.new(1,0,0,28)
})

-- RGB animado no título
task.spawn(function()
    local hue = 0
    while top.Parent do
        hue = (hue + 0.01) % 1
        top.TextColor3 = Color3.fromHSV(hue,1,1)
        task.wait(0.05)
    end
end)

-- Botões compactos
local function createButton(text, yPos)
    local btn = create("TextButton", {
        Parent = main,
        Text = text,
        Font = Enum.Font.GothamBold,
        TextSize = 14,
        BackgroundColor3 = Color3.fromRGB(40,40,40),
        TextColor3 = Color3.new(1,1,1),
        Position = UDim2.new(0.5,-60,0.5,yPos),
        Size = UDim2.new(0,120,0,25)
    })
    create("UICorner", {Parent=btn, CornerRadius=UDim.new(0,6)})
    return btn
end

local savePos = createButton("Salvar Posição",-40)
local steal = createButton("Instante Steal",0)
local speedButton = createButton("Speed 200",40)

-- Funções
local savedCFrame = nil
local defaultSpeed = 16
local speedActive = false

local function getChar()
    return player.Character or player.CharacterAdded:Wait()
end

savePos.MouseButton1Click:Connect(function()
    local char = getChar()
    if char and char:FindFirstChild("HumanoidRootPart") then
        savedCFrame = char.HumanoidRootPart.CFrame
        savePos.Text = "Posição Salva!"
        task.wait(1)
        savePos.Text = "Salvar Posição"
    end
end)

steal.MouseButton1Click:Connect(function()
    local char = getChar()
    if savedCFrame and char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = savedCFrame + Vector3.new(0,3,0)
    end
end)

-- Speed toggle
speedButton.MouseButton1Click:Connect(function()
    local char = getChar()
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        if not speedActive then
            hum.WalkSpeed = 200
            speedButton.Text = "Speed Ativado!"
            speedButton.BackgroundColor3 = Color3.fromRGB(255,0,0)
            speedActive = true
        else
            hum.WalkSpeed = defaultSpeed
            speedButton.Text = "Speed 200"
            speedButton.BackgroundColor3 = Color3.fromRGB(40,40,40)
            speedActive = false
        end
    end
end)

player.CharacterAdded:Connect(function(char)
    local hum = char:WaitForChild("Humanoid")
    if speedActive then
        hum.WalkSpeed = 200
    else
        hum.WalkSpeed = defaultSpeed
    end
end)

-- Arrastar menu
local UIS = game:GetService("UserInputService")
local dragging = false
local dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    local viewport = workspace.CurrentCamera.ViewportSize
    main.Position = UDim2.new(
        0, math.clamp(startPos.X.Offset + delta.X,0,viewport.X-main.AbsoluteSize.X),
        0, math.clamp(startPos.Y.Offset + delta.Y,0,viewport.Y-main.AbsoluteSize.Y)
    )
end

local function startDrag(input)
    dragging = true
    dragStart = input.Position
    startPos = main.Position

    local moveConnection
    moveConnection = UIS.InputChanged:Connect(function(input2)
        if input2.UserInputType == Enum.UserInputType.MouseMovement or input2.UserInputType == Enum.UserInputType.Touch then
            if dragging then update(input2) end
        end
    end)

    input.Changed:Connect(function()
        if input.UserInputState == Enum.UserInputState.End then
            dragging = false
            moveConnection:Disconnect()
        end
    end)
end

main.InputBegan:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
        startDrag(input)
    end
end)
