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
Main.Size = UDim2.fromOffset(450, 280)
Main.Position = UDim2.new(0.5, -225, 0.5, -140)
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
Main.BorderSizePixel = 0
Main.Parent = ScreenGui

Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

makeDraggable(Main)

-- ================= SIDEBAR =================
local Side = Instance.new("Frame")
Side.Size = UDim2.fromOffset(110, 280)
Side.BackgroundColor3 = Color3.fromRGB(28, 28, 28)
Side.BorderSizePixel = 0
Side.Parent = Main

Instance.new("UICorner", Side).CornerRadius = UDim.new(0, 10)

-- ================= PAGES =================
local Pages = Instance.new("Frame")
Pages.Size = UDim2.new(1, -120, 1, -20)
Pages.Position = UDim2.fromOffset(120, 10)
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
local SnakePage = newPage("Snake")

MainPage.Visible = true

-- ================= TABS =================
local function tab(text, y, page)
    local b = Instance.new("TextButton")
    b.Size = UDim2.fromOffset(100, 35)
    b.Position = UDim2.fromOffset(5, y)
    b.Text = text
    b.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    b.TextColor3 = Color3.new(1,1,1)
    b.Parent = Side
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)

    b.MouseButton1Click:Connect(function()
        for _, p in pairs(Pages:GetChildren()) do
            p.Visible = false
        end
        page.Visible = true
    end)
end

tab("Main", 10, MainPage)
tab("Logs", 55, LogsPage)
tab("Snake", 100, SnakePage)

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

-- ================= SNAKE (IMPROVED WAVE) =================
local snake = {}

for i = 1, 18 do
    local p = Instance.new("Frame")
    p.Size = UDim2.fromOffset(8,8)
    p.Position = UDim2.fromOffset(20 + i*10, 120)
    p.BackgroundColor3 = Color3.fromRGB(0,255,0)
    p.BorderSizePixel = 0
    p.Parent = SnakePage
    Instance.new("UICorner", p).CornerRadius = UDim.new(1,0)
    table.insert(snake, p)
end

task.spawn(function()
    local t = 0
    while true do
        t += 0.08
        for i, p in ipairs(snake) do
            local wave = math.sin(t + i * 0.35) * 18
            local wave2 = math.cos(t * 0.6 + i * 0.2) * 8
            p.Position = UDim2.fromOffset(20 + i*10 + wave2, 120 + wave)
        end
        task.wait(0.02)
    end
end)

-- ================= SOUND LOGIC (OPTIMIZED) =================
local connected = {}

local function normalize(id)
    return tostring(id):match("%d+") or ""
end

local function hookSound(sound)
    if connected[sound] then return end
    connected[sound] = true
            addLog("➕ DETECTED: " .. sound.Name, Color3.fromRGB(180,180,180))
    sound.Played:Connect(function()
        if not Enabled then return end

        local id = normalize(sound.SoundId)

        if id == TARGET_SOUND_ID then
            flash()
            status.Text = "TRIGGERED"

            addLog("🔊 SOUND: " .. sound.Name .. " | " .. id, Color3.fromRGB(0,255,120))
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
