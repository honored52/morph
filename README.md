local TARGET_SOUND_ID = "96182964301191"
local Enabled = true

local UIS = game:GetService("UserInputService")

-- ================= ROOT =================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BetterSoundUI"
ScreenGui.ResetOnSpawn = false
pcall(function()
    ScreenGui.Parent = game:GetService("CoreGui")
end)

-- ================= DRAG SYSTEM =================
local dragging = false
local dragInput, dragStart, startPos

local function makeDraggable(frame)
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

-- ================= MAIN WINDOW =================
local Main = Instance.new("Frame")
Main.Position = UDim2.new(0.5, -225, 0.5, -140)
Main.BorderSizePixel = 0
Main.Parent = ScreenGui
Main.Size = UDim2.fromOffset(560, 340)
Main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)

local Stroke = Instance.new("UIStroke")
Stroke.Color = Color3.fromRGB(55,55,55)
Stroke.Thickness = 1
Stroke.Parent = Main

local Gradient = Instance.new("UIGradient")
Gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(22,22,22)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(16,16,16))
}
Gradient.Parent = Main
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

makeDraggable(Main)

-- ================= TOP BAR =================

local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 32)
TopBar.BorderSizePixel = 0
TopBar.Parent = Main
TopBar.BackgroundColor3 = Color3.fromRGB(16,16,16)

local TopStroke = Instance.new("UIStroke")
TopStroke.Color = Color3.fromRGB(40,40,40)
TopStroke.Parent = TopBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -70, 1, 0)
Title.Position = UDim2.fromOffset(10, 0)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar
Title.Text = "CLC HUB"
Title.TextSize = 18
Title.Font = Enum.Font.GothamBlack

local Version = Instance.new("TextLabel")
Version.Size = UDim2.fromOffset(60, 32)
Version.Position = UDim2.new(1, -65, 0, 0)
Version.BackgroundTransparency = 1
Version.Text = "TEST"
Version.TextColor3 = Color3.fromRGB(0,170,255)
Version.TextXAlignment = Enum.TextXAlignment.Right
Version.Font = Enum.Font.GothamBold
Version.TextSize = 15
Version.Parent = TopBar

-- ================= SIDEBAR =================
local Side = Instance.new("Frame")
Side.Position = UDim2.fromOffset(0, 32) -- под TopBar
Side.Size = UDim2.new(0, 110, 1, -32)
Side.BorderSizePixel = 0
Side.Parent = Main
Side.BackgroundColor3 = Color3.fromRGB(15,15,15)

local SideStroke = Instance.new("UIStroke")
SideStroke.Color = Color3.fromRGB(35,35,35)
SideStroke.Parent = Side

Instance.new("UICorner", Side).CornerRadius = UDim.new(0, 10)

-- ================= PAGES =================
local Pages = Instance.new("Frame")
Pages.Size = UDim2.new(1, -120, 1, -50)
Pages.Position = UDim2.fromOffset(120, 40)
Pages.BackgroundTransparency = 1
Pages.Parent = Main

local function newPage(name)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 1, 0)
    f.BackgroundTransparency = 1
    f.Visible = false
    f.Name = name
    f.Parent = Pages
    return f
end

local MainPage = newPage("Main")
local LogsPage = newPage("Logs")
local ThemesPage = newPage("Themes")

MainPage.Visible = true

-- ================= TABS =================
local function tab(text, y, page)
    local b = Instance.new("TextButton")
    b.Size = UDim2.fromOffset(100, 35)
    b.Position = UDim2.fromOffset(5, y)
    b.Text = text
    b.BackgroundColor3 = Color3.fromRGB(25,25,25)
    b.AutoButtonColor = false
    b.TextColor3 = Color3.new(1,1,1)
    b.Parent = Side
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(40,40,40)
stroke.Parent = b

b.MouseEnter:Connect(function()
    b.BackgroundColor3 = Color3.fromRGB(35,35,35)
end)

b.MouseLeave:Connect(function()
    b.BackgroundColor3 = Color3.fromRGB(25,25,25)
end)

    b.MouseButton1Click:Connect(function()
        for _, p in pairs(Pages:GetChildren()) do
            p.Visible = false
        end
        page.Visible = true
    end)
end

tab("Main", 10, MainPage)
tab("Logs", 55, LogsPage)
tab("Themes", 100, ThemesPage)
-- ================= INDICATOR =================
local indicator = Instance.new("Frame")
indicator.Size = UDim2.fromOffset(12, 12)
indicator.Position = UDim2.new(0, 8, 1, -20)
indicator.BackgroundColor3 = Color3.fromRGB(30, 70, 30)
indicator.BorderSizePixel = 0
indicator.Parent = ScreenGui
Instance.new("UICorner", indicator).CornerRadius = UDim.new(1,0)

local function flash()
    indicator.BackgroundColor3 = Color3.fromRGB(0,255,0)
    task.delay(0.2, function()
        indicator.BackgroundColor3 = Color3.fromRGB(30,70,30)
    end)
end

-- ================= LOG SYSTEM (IMPROVED CONSOLE) =================

local logList = Instance.new("ScrollingFrame")
logList.Size = UDim2.new(1, 0, 1, 0)
logList.CanvasSize = UDim2.new(0,0,0,0)
logList.ScrollBarThickness = 6
logList.BackgroundTransparency = 1
logList.Parent = LogsPage

logList.BackgroundColor3 = Color3.fromRGB(18,18,18)

local logCorner = Instance.new("UICorner")
logCorner.Parent = logList

local logStroke = Instance.new("UIStroke")
logStroke.Color = Color3.fromRGB(40,40,40)
logStroke.Parent = logList

local layout = Instance.new("UIListLayout")
layout.Parent = logList
layout.SortOrder = Enum.SortOrder.LayoutOrder

-- лимит чтобы не лагало
local MAX_LOGS = 120
local logs = {}

local function addLog(text, color)
    color = color or Color3.fromRGB(200,200,200)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 18)
    label.BackgroundTransparency = 1
    label.TextColor3 = color
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.Gotham
    label.TextSize = 13
    label.Text = text
    label.Parent = logList

    table.insert(logs, label)

    -- удаление старых логов
    if #logs > MAX_LOGS then
        logs[1]:Destroy()
        table.remove(logs, 1)
    end

    task.defer(function()
        logList.CanvasSize = UDim2.new(0,0,0,layout.AbsoluteContentSize.Y)
        logList.CanvasPosition = Vector2.new(0, math.huge)
    end)
end
-- ================= MAIN UI =================
local status = Instance.new("TextLabel")
status.Size = UDim2.new(1,0,0,40)
status.BackgroundTransparency = 1
status.TextColor3 = Color3.new(1,1,1)
status.Text = "Waiting..."
status.TextScaled = true
status.Parent = MainPage

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.fromOffset(200, 40)
toggle.Position = UDim2.new(0,10,0,60)
toggle.Text = "Script: ON"
toggle.BackgroundColor3 = Color3.fromRGB(50,120,50)
toggle.TextColor3 = Color3.new(1,1,1)
toggle.Parent = MainPage
Instance.new("UICorner", toggle).CornerRadius = UDim.new(0,6)

toggle.MouseButton1Click:Connect(function()
    Enabled = not Enabled
    toggle.Text = Enabled and "Script: ON" or "Script: OFF"
end)

local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.fromOffset(200, 40)
espBtn.Position = UDim2.new(0, 10, 0, 110)
espBtn.Text = "ESP: ON"
espBtn.BackgroundColor3 = Color3.fromRGB(40,80,160)
espBtn.TextColor3 = Color3.new(1,1,1)
espBtn.Parent = MainPage

Instance.new("UICorner", espBtn)

espBtn.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    espBtn.Text = ESPEnabled and "ESP: ON" or "ESP: OFF"

    for char, data in pairs(ESPObjects) do
        if data.hl then data.hl.Enabled = ESPEnabled end
        if data.gui then data.gui.Enabled = ESPEnabled end
    end
end)

-- ================= SOUND LOGIC (OPTIMIZED) =================
local connected = {}

local function normalize(id)
    return tostring(id):match("%d+") or ""
end

local function hookSound(sound)
    if connected[sound] then
        return
    end

    connected[sound] = true

    local soundId = normalize(sound.SoundId)

    addLog(
        "➕ DETECTED: " .. sound.Name .. " | ID: " .. (soundId ~= "" and soundId or "None"),
        Color3.fromRGB(180,180,180)
    )

    sound.Played:Connect(function()
        if not Enabled then
            return
        end

        local id = normalize(sound.SoundId)

        addLog(
            "🔊 PLAYED: " .. sound.Name .. " | ID: " .. (id ~= "" and id or "None"),
            Color3.fromRGB(0,170,255)
        )

        if id == TARGET_SOUND_ID then
            flash()
            status.Text = "TRIGGERED"

            addLog(
                "🎯 TARGET HIT: " .. id,
                Color3.fromRGB(0,255,120)
            )
        end
    end)
end

for _, v in ipairs(game:GetDescendants()) do
    if v:IsA("Sound") then
        hookSound(v)
    end
end

game.DescendantAdded:Connect(function(v)
    if v:IsA("Sound") then
        hookSound(v)
    end
end)

-- ================= INSERT TOGGLE =================
UIS.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Insert then
        Main.Visible = not Main.Visible
    end
end)
-- ================= THEMES =================
local Themes = {
    Dark = {
        Main = Color3.fromRGB(18,18,18),
        Side = Color3.fromRGB(28,28,28),
        Button = Color3.fromRGB(45,45,45),
        Text = Color3.fromRGB(255,255,255)
    },

    Light = {
        Main = Color3.fromRGB(235,235,235),
        Side = Color3.fromRGB(215,215,215),
        Button = Color3.fromRGB(190,190,190),
        Text = Color3.fromRGB(0,0,0)
    },

    Green = {
        Main = Color3.fromRGB(15,30,15),
        Side = Color3.fromRGB(25,50,25),
        Button = Color3.fromRGB(45,90,45),
        Text = Color3.fromRGB(220,255,220)
    },

    Purple = {
        Main = Color3.fromRGB(30,20,40),
        Side = Color3.fromRGB(50,30,70),
        Button = Color3.fromRGB(90,50,140),
        Text = Color3.fromRGB(255,255,255)
    }
}

local function ApplyTheme(theme)
    Main.BackgroundColor3 = theme.Main
    Side.BackgroundColor3 = theme.Side

    for _, v in ipairs(ScreenGui:GetDescendants()) do
        if v:IsA("TextLabel") then
            v.TextColor3 = theme.Text

        elseif v:IsA("TextButton") then
            v.BackgroundColor3 = theme.Button
            v.TextColor3 = theme.Text
        end
    end
end
local y = 10

for name, theme in pairs(Themes) do
    local btn = Instance.new("TextButton")

    btn.Size = UDim2.fromOffset(180, 35)
    btn.Position = UDim2.fromOffset(10, y)

    btn.Text = name
    btn.Parent = ThemesPage

    btn.BackgroundColor3 = theme.Button
    btn.TextColor3 = theme.Text

    Instance.new("UICorner", btn)

    btn.MouseButton1Click:Connect(function()
        ApplyTheme(theme)
    end)

    y += 45
end
-- ================= ESP =================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local ESPEnabled = false
local ESPObjects = {}

local function removeESP(char)
    local data = ESPObjects[char]
    if not data then return end

    if data.hl then data.hl:Destroy() end
    if data.gui then data.gui:Destroy() end

    ESPObjects[char] = nil
end

local function createESP(char)
    if not char then return end
    if ESPObjects[char] then return end

    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end

    -- HIGHLIGHT
    local hl = Instance.new("Highlight")
    hl.Name = "ESP_HL"
    hl.FillColor = Color3.fromRGB(0,170,255)
    hl.OutlineColor = Color3.fromRGB(255,255,255)
    hl.FillTransparency = 0.5
    hl.Enabled = ESPEnabled
    hl.Parent = char

    -- LEFT HP BAR GUI
    local gui = Instance.new("BillboardGui")
    gui.Name = "HP_BAR"
    gui.Size = UDim2.fromOffset(6, 50)
    gui.AlwaysOnTop = true
    gui.StudsOffset = Vector3.new(-2.5, 1, 0) -- 👈 СЛЕВА ОТ ПЕРСОНАЖА
    gui.Parent = char

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1,0,1,0)
    bg.BackgroundColor3 = Color3.fromRGB(40,40,40)
    bg.BorderSizePixel = 0
    bg.Parent = gui

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1,0,1,0)
    bar.BackgroundColor3 = Color3.fromRGB(0,255,0)
    bar.BorderSizePixel = 0
    bar.Parent = bg

    ESPObjects[char] = {
        hl = hl,
        gui = gui,
        bar = bar,
        hum = hum
    }
end

local function updateESP()
    for char, data in pairs(ESPObjects) do
        if not char or not char.Parent then
            removeESP(char)
            continue
        end

        local hum = data.hum
        if hum then
            local hp = math.clamp(hum.Health / hum.MaxHealth, 0, 1)

            data.bar.Size = UDim2.new(1, 0, hp, 0)
            data.bar.Position = UDim2.new(0, 0, 1 - hp, 0)

            -- цвет по HP
            if hp > 0.6 then
                data.bar.BackgroundColor3 = Color3.fromRGB(0,255,0)
            elseif hp > 0.3 then
                data.bar.BackgroundColor3 = Color3.fromRGB(255,170,0)
            else
                data.bar.BackgroundColor3 = Color3.fromRGB(255,0,0)
            end
        end

        data.hl.Enabled = ESPEnabled
        data.gui.Enabled = ESPEnabled
    end
end

task.spawn(function()
    while true do
        updateESP()
        task.wait(0.1)
    end
end)

local function updateESP()
    for char, data in pairs(ESPObjects) do
        if char and char.Parent and data.root and data.hum then
            local dist = (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart"))
                and (LocalPlayer.Character.HumanoidRootPart.Position - data.root.Position).Magnitude
                or 0

            data.text.Text = string.format(
                "%s\nHP: %d\n%.0fm",
                char.Name,
                data.hum.Health,
                dist
            )

            data.hl.Enabled = ESPEnabled
            data.bb.Enabled = ESPEnabled
        end
    end
end

task.spawn(function()
    while true do
        updateESP()
        task.wait(0.2)
    end
end)
-- ================= PLAYER ESP HOOK =================

local function hookPlayer(p)
    if p == LocalPlayer then return end

    p.CharacterAdded:Connect(function(c)
        task.wait(0.5)
        if ESPEnabled then
            createESP(c)
        end
    end)

    if p.Character and ESPEnabled then
        createESP(p.Character)
    end
end

for _, p in ipairs(Players:GetPlayers()) do
    hookPlayer(p)
end

Players.PlayerAdded:Connect(hookPlayer)
