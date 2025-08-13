--// Mensagem \--
--[[
The original script wasnt made by me [doge#7202]
I just forked the script and added a keybind to change the range.
The original script creator is epzi
Make sure to tweak the settings to your likings
Have a great day!
[Forked version of script made by doge#7202 on discord]
]]

--// Settings \--
local range = 55 -- Alcance padrão do script
local rangeAddkeybind = Enum.KeyCode.E -- Tecla para aumentar o alcance
local rangeSubtractkeybind = Enum.KeyCode.Q -- Tecla para diminuir o alcance
local TogglePreciseRange = Enum.KeyCode.BackSlash -- Tecla para alternar precisão do alcance
local DoNotDisturb = false -- Se false, envia notificações; se true, não envia
local PreciseRange = false -- Se true, muda o alcance em 0.01; se false, muda em 1
local ScriptEnabled = true -- Controla se o script está ativo ou não

-- NOVO: limites do alcance
local MIN_RANGE = 1
local MAX_RANGE = 1000

--// Variáveis \--
local player = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")
local RunService = game:GetService("RunService")

-- Referências de UI para atualizar visualmente
local UIRefs = {
    Button = nil,
    RangeLabel = nil,
    RangeTextBox = nil,
    SliderBar = nil,
    SliderFill = nil,
    SliderKnob = nil
}

-- Função utilitária para notificar
local function notify(title, text)
    if DoNotDisturb == false then
        pcall(function()
            StarterGui:SetCore("SendNotification", { Title = title, Text = text })
        end)
    end
end

-- Clamp seguro (com arredondamento quando não está preciso)
local function clampRange(value)
    local v = value
    if not PreciseRange then
        v = math.floor(v + 0.5)
    end
    if v < MIN_RANGE then v = MIN_RANGE end
    if v > MAX_RANGE then v = MAX_RANGE end
    return v
end

-- Atualiza visual do slider conforme o range atual
local function updateSliderFromRange()
    if not (UIRefs.SliderBar and UIRefs.SliderFill and UIRefs.SliderKnob) then return end
    local bar = UIRefs.SliderBar
    local fill = UIRefs.SliderFill
    local knob = UIRefs.SliderKnob

    local barSizeX = bar.AbsoluteSize.X
    if barSizeX <= 0 then
        task.defer(updateSliderFromRange)
        return
    end

    local t = (range - MIN_RANGE) / (MAX_RANGE - MIN_RANGE)
    t = math.clamp(t, 0, 1)

    fill.Size = UDim2.new(t, 0, 1, 0)
    knob.Position = UDim2.new(t, -8, 0.5, -8)
end

-- Atualiza textos
local function updateRangeText()
    if UIRefs.RangeLabel then
        UIRefs.RangeLabel.Text = ("Alcance: %s"):format(PreciseRange and string.format("%.2f", range) or tostring(range))
    end
    if UIRefs.RangeTextBox and not UIRefs.RangeTextBox:IsFocused() then
        UIRefs.RangeTextBox.Text = PreciseRange and string.format("%.2f", range) or tostring(range)
    end
end

-- Define o range de forma centralizada
local function setRange(newValue, source)
    local newRange = clampRange(newValue)
    if newRange == range then
        updateRangeText()
        updateSliderFromRange()
        return
    end
    range = newRange
    updateRangeText()
    updateSliderFromRange()
    notify("Notification", "o range foi definido para " .. tostring(range))
end

--// GUI de Toggle + Controle de Range \--
local function createToggleGui()
    local playerGui = player:WaitForChild("PlayerGui")

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "RangeToggleGui"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui

    local frame = Instance.new("Frame")
    frame.Name = "Container"
    frame.Size = UDim2.new(0, 220, 0, 160) -- Aumentado para comportar o slider e o input
    frame.Position = UDim2.new(0, 20, 0, 200)
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    frame.BorderSizePixel = 0
    frame.Active = true
    frame.Draggable = true
    frame.Parent = screenGui

    local uicorner1 = Instance.new("UICorner")
    uicorner1.CornerRadius = UDim.new(0, 6)
    uicorner1.Parent = frame

    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, -10, 0, 20)
    title.Position = UDim2.new(0, 5, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "Auto Range"
    title.TextColor3 = Color3.fromRGB(230, 230, 230)
    title.TextSize = 14
    title.Font = Enum.Font.Gotham
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = frame

    local button = Instance.new("TextButton")
    button.Name = "ToggleButton"
    button.Size = UDim2.new(1, -10, 0, 35)
    button.Position = UDim2.new(0, 5, 0, 30)
    button.BackgroundColor3 = Color3.fromRGB(0, 170, 127)
    button.BorderSizePixel = 0
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 14
    button.Font = Enum.Font.GothamMedium
    button.Parent = frame
    UIRefs.Button = button

    local uicorner2 = Instance.new("UICorner")
    uicorner2.CornerRadius = UDim.new(0, 6)
    uicorner2.Parent = button

    local function updateToggleUI()
        if ScriptEnabled then
            button.Text = "Desativar (Ligado)"
            button.BackgroundColor3 = Color3.fromRGB(0, 170, 127)
        else
            button.Text = "Ativar (Desligado)"
            button.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        end
    end

    button.MouseButton1Click:Connect(function()
        ScriptEnabled = not ScriptEnabled
        updateToggleUI()
        notify("Auto Range", ScriptEnabled and "Script ativado" or "Script desativado")
    end)

    -- NOVO: Label do alcance
    local rangeLabel = Instance.new("TextLabel")
    rangeLabel.Name = "RangeLabel"
    rangeLabel.Size = UDim2.new(1, -10, 0, 18)
    rangeLabel.Position = UDim2.new(0, 5, 0, 70)
    rangeLabel.BackgroundTransparency = 1
    rangeLabel.Text = "Alcance: " .. tostring(range)
    rangeLabel.TextColor3 = Color3.fromRGB(230, 230, 230)
    rangeLabel.TextSize = 14
    rangeLabel.Font = Enum.Font.Gotham
    rangeLabel.TextXAlignment = Enum.TextXAlignment.Left
    rangeLabel.Parent = frame
    UIRefs.RangeLabel = rangeLabel

    -- NOVO: Slider container
    local sliderContainer = Instance.new("Frame")
    sliderContainer.Name = "SliderContainer"
    sliderContainer.Size = UDim2.new(1, -10, 0, 28)
    sliderContainer.Position = UDim2.new(0, 5, 0, 92)
    sliderContainer.BackgroundTransparency = 1
    sliderContainer.Parent = frame

    -- Barra do slider
    local sliderBar = Instance.new("Frame")
    sliderBar.Name = "SliderBar"
    sliderBar.Size = UDim2.new(0, 150, 0, 6)
    sliderBar.Position = UDim2.new(0, 0, 0.5, -3)
    sliderBar.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    sliderBar.BorderSizePixel = 0
    sliderBar.Parent = sliderContainer
    UIRefs.SliderBar = sliderBar

    local barCorner = Instance.new("UICorner")
    barCorner.CornerRadius = UDim.new(0, 3)
    barCorner.Parent = sliderBar

    -- Preenchimento
    local sliderFill = Instance.new("Frame")
    sliderFill.Name = "SliderFill"
    sliderFill.Size = UDim2.new(0, 0, 1, 0)
    sliderFill.Position = UDim2.new(0, 0, 0, 0)
    sliderFill.BackgroundColor3 = Color3.fromRGB(0, 170, 127)
    sliderFill.BorderSizePixel = 0
    sliderFill.Parent = sliderBar
    UIRefs.SliderFill = sliderFill

    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 3)
    fillCorner.Parent = sliderFill

    -- Knob
    local sliderKnob = Instance.new("Frame")
    sliderKnob.Name = "SliderKnob"
    sliderKnob.Size = UDim2.new(0, 16, 0, 16)
    sliderKnob.Position = UDim2.new(0, -8, 0.5, -8)
    sliderKnob.BackgroundColor3 = Color3.fromRGB(230, 230, 230)
    sliderKnob.BorderSizePixel = 0
    sliderKnob.Parent = sliderContainer
    UIRefs.SliderKnob = sliderKnob

    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius = UDim.new(1, 0)
    knobCorner.Parent = sliderKnob

    -- NOVO: TextBox para digitar valor
    local rangeBox = Instance.new("TextBox")
    rangeBox.Name = "RangeTextBox"
    rangeBox.Size = UDim2.new(0, 50, 0, 22)
    rangeBox.Position = UDim2.new(1, -55, 0.5, -11)
    rangeBox.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    rangeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    rangeBox.PlaceholderText = "1-1000"
    rangeBox.Text = tostring(range)
    rangeBox.TextSize = 14
    rangeBox.Font = Enum.Font.Gotham
    rangeBox.ClearTextOnFocus = false
    rangeBox.Parent = sliderContainer
    UIRefs.RangeTextBox = rangeBox

    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 4)
    boxCorner.Parent = rangeBox

    -- Lógica de arraste do slider
    local dragging = false

    local function positionToRange(x)
        local rel = (x - sliderBar.AbsolutePosition.X) / math.max(sliderBar.AbsoluteSize.X, 1)
        rel = math.clamp(rel, 0, 1)
        local raw = MIN_RANGE + rel * (MAX_RANGE - MIN_RANGE)
        if PreciseRange then
            return clampRange(raw) -- mantém decimais com clamp
        else
            return clampRange(math.floor(raw + 0.5))
        end
    end

    sliderKnob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)

    sliderKnob.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    sliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            setRange(positionToRange(input.Position.X), "slider")
            dragging = true
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            setRange(positionToRange(input.Position.X), "slider")
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    -- Digitação direta no TextBox
    rangeBox.FocusLost:Connect(function(enterPressed)
        local v = tonumber(rangeBox.Text)
        if v then
            setRange(v, "textbox")
        else
            -- Restaura texto válido se inválido
            rangeBox.Text = PreciseRange and string.format("%.2f", range) or tostring(range)
        end
    end)

    updateToggleUI()
    task.defer(function()
        updateRangeText()
        updateSliderFromRange()
    end)
end

createToggleGui()

--// Funções de Tecla \--
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == rangeSubtractkeybind then
        local step = PreciseRange and 0.01 or 1
        setRange(range - step, "key")
    end
end)

UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == rangeAddkeybind then
        local step = PreciseRange and 0.01 or 1
        setRange(range + step, "key")
    end
end)

UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == TogglePreciseRange then
        PreciseRange = not PreciseRange
        notify("Notification", "Precise range foi definido para " .. tostring(PreciseRange))
        -- Ao alternar precisão, atualiza exibição para refletir arredondamento
        setRange(range, "toggle-precision")
    end
end)

--// Script principal \--
game:GetService("RunService").RenderStepped:Connect(function()
    if not ScriptEnabled then return end

    local p = game.Players:GetPlayers()
    for i = 2, #p do
        local char = p[i].Character
        if char and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 and char:FindFirstChild("HumanoidRootPart") and player:DistanceFromCharacter(char.HumanoidRootPart.Position) <= range then
            local tool = player.Character and player.Character:FindFirstChildOfClass("Tool")
            if tool and tool:FindFirstChild("Handle") then
                tool:Activate()
                for _, part in next, char:GetChildren() do
                    if part:IsA("BasePart") then
                        firetouchinterest(tool.Handle, part, 0)
                        firetouchinterest(tool.Handle, part, 1)
                    end
                end
            end
        end
    end
end)

notify("Script Load", "o script foi carregado com sucesso!")
