local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local Lighting = game:GetService("Lighting")
local Stats = game:GetService("Stats")
local TeleportService = game:GetService("TeleportService")
local MarketplaceService = game:GetService("MarketplaceService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local genv = (getgenv and getgenv()) or _G

local VERSION = "v1.23.1"

for _, env in ipairs({genv, _G}) do
    local old = env.StrawberryUnload
    if old then
        env.StrawberryUnload = nil
        pcall(old)
    end
end

local running = true

-- runs one feature; if it fails (for example a function your executor does not have), the rest still loads
local function safeRun(f, ...)
    local ok, err = pcall(f, ...)
    if not ok then warn("[Strawberry Hub] a feature could not load: " .. tostring(err)) end
end
local loaded = false
local connections = {}
local H: {[string]: any} = {}

-- the script owner's name is never shown to other users: everywhere in the script they are "Owner"
H.OWNER_ID = 2038953612
H.nameOf = function(plr)
    if not plr then return "?" end
    if plr.UserId == H.OWNER_ID and player.UserId ~= H.OWNER_ID then return "\u{1F451} Owner" end
    return plr.DisplayName
end

local function connect(signal, fn)
    local c = signal:Connect(fn)
    connections[#connections + 1] = c
    return c
end

local S: {[string]: any} = {
    autoEat = false, autoDrink = false, autoZone = false, eatPlants = false, eatMeat = false,
    eatRate = 25, eatPromptOnly = false, autoFishing = false, fishGoTo = false,
    zoneUnder = true, zoneAutoLower = true, zoneHeight = 8, zoneDepth = 15, zoneNoclip = false, zoneWait = 20,
    zoneEnemyRadius = 0, zoneDefend = false,
    autoMeat = false, eatTp = false, eatBelow = 95, returnAfter = true, eatRange = 40,
    autoFish = false, fishNoQuest = false, tripBusy = false, instantEat = false,
    fishFarm = false, autoMinigame = false, fishRealSpeed = false, fishDelay = 1,

    safeEnabled = false, hpThreshold = 10, lowHpActive = false, safeCFrame = nil,

    thirdPerson = false, fovEnabled = false, fovValue = 70,
    spectate = nil,

    speedEnabled = false, speedValue = 50, savedWalkSpeed = 16,
    jumpEnabled = false, jumpValue = 75, savedJump = 50,
    flyEnabled = false, flySpeed = 60,
    noclip = false, infJump = false, clickTp = false, antiAfk = false,

    espOn = false, espNames = false, espDist = false, espBoxes = false, espHealth = false, espChams = false,
    espMax = 2000, espTextSize = 13, espBoxThick = 1, chamsAlpha = 0.5,
    cName = Color3.fromRGB(255, 255, 255), cDist = Color3.fromRGB(190, 190, 190),
    cBox = Color3.fromRGB(255, 255, 255), cHealth = Color3.fromRGB(80, 255, 110),
    cChamsFill = Color3.fromRGB(170, 90, 255), cChamsOut = Color3.fromRGB(255, 255, 255),

    zoneEsp = false, zoneDist = false, zoneStatus = false,
    zOwned = Color3.fromRGB(80, 255, 120), zEnemy = Color3.fromRGB(255, 70, 70),

    mapOn = false, mapAlpha = 0.7, fullbright = false, noFog = false,

    watermark = false,
}

local Options = {}

local WHITE, BLACK = Color3.new(1, 1, 1), Color3.new(0, 0, 0)
local FONT = Enum.Font.Code
local function rgb(r, g, b) return Color3.fromRGB(r, g, b) end

local Presets = {
    {"Default",     rgb(60, 230, 215),  rgb(26, 26, 26),  rgb(48, 48, 48),  WHITE},
    {"Strawberry",  rgb(255, 70, 110),  rgb(28, 22, 24),  rgb(60, 40, 46),  WHITE},
    {"Ocean",       rgb(45, 140, 255),  rgb(20, 26, 36),  rgb(38, 52, 72),  WHITE},
    {"Amethyst",    rgb(170, 90, 255),  rgb(24, 21, 32),  rgb(52, 44, 72),  WHITE},
    {"Mint",        rgb(90, 255, 160),  rgb(22, 28, 25),  rgb(40, 58, 48),  WHITE},
    {"Sunset",      rgb(255, 140, 60),  rgb(30, 24, 22),  rgb(64, 48, 40),  WHITE},
    {"Crimson",     rgb(220, 40, 50),   rgb(22, 20, 20),  rgb(58, 36, 36),  WHITE},
    {"Midnight",    rgb(88, 101, 242),  rgb(14, 15, 25),  rgb(30, 32, 54),  WHITE},
    {"Tokyo Night", rgb(122, 162, 247), rgb(26, 27, 38),  rgb(41, 46, 66),  rgb(192, 202, 245)},
    {"Dracula",     rgb(189, 147, 249), rgb(40, 42, 54),  rgb(68, 71, 90),  rgb(248, 248, 242)},
    {"Gruvbox",     rgb(250, 189, 47),  rgb(40, 40, 40),  rgb(80, 73, 69),  rgb(235, 219, 178)},
    {"Nord",        rgb(136, 192, 208), rgb(46, 52, 64),  rgb(67, 76, 94),  rgb(216, 222, 233)},
    {"Rose Pine",   rgb(235, 111, 146), rgb(25, 23, 36),  rgb(38, 35, 58),  rgb(224, 222, 244)},
    {"Monochrome",  rgb(225, 225, 225), rgb(24, 24, 24),  rgb(52, 52, 52),  WHITE},
}

local Base = {Accent = Presets[2][2], Background = Presets[2][3], Outline = Presets[2][4], Text = Presets[2][5]}
local Theme = {Black = BLACK}
local binds = {}

local function recompute()
    local bg = Base.Background
    Theme.Accent = Base.Accent
    Theme.Window = bg
    Theme.Inner = bg:Lerp(BLACK, 0.22)
    Theme.Group = bg:Lerp(BLACK, 0.12)
    Theme.Border = Base.Outline
    Theme.Control = bg:Lerp(WHITE, 0.08)
    Theme.Hover = bg:Lerp(WHITE, 0.17)
    Theme.Text = Base.Text
    Theme.Dim = Base.Text:Lerp(bg, 0.45)
end
recompute()

local function bind(fn)
    binds[#binds + 1] = fn
    fn()
end

local function applyTheme()
    recompute()
    for _, f in ipairs(binds) do pcall(f) end
end

local function skin(inst, prop, key)
    bind(function() inst[prop] = Theme[key] end)
end

local function contrast(c)
    local lum = 0.299 * c.R + 0.587 * c.G + 0.114 * c.B
    return lum > 0.55 and rgb(10, 10, 10) or WHITE
end

local function addStroke(inst, key, thickness)
    local s = Instance.new("UIStroke")
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Thickness = thickness or 1
    s.Parent = inst
    if key then skin(s, "Color", key) end
    return s
end

local function tween(inst, time, props, style, dir)
    local t = TweenService:Create(inst,
        TweenInfo.new(time, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out), props)
    t:Play()
    return t
end

local function isPointerInput(input)
    return input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch
end

local function pointerPos(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        local i = GuiService:GetGuiInset()
        return Vector2.new(input.Position.X + i.X, input.Position.Y + i.Y)
    end
    return UIS:GetMouseLocation()
end

local function inside(gui, p)
    local a, s = gui.AbsolutePosition, gui.AbsoluteSize
    return p.X >= a.X and p.X <= a.X + s.X and p.Y >= a.Y and p.Y <= a.Y + s.Y
end

local function setOrder(inst, parent)
    local n = (parent:GetAttribute("OrderCount") or 0) + 1
    parent:SetAttribute("OrderCount", n)
    inst.LayoutOrder = n
end

local function getRoot(plr)
    local char = plr and plr.Character
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function getHumanoid(plr)
    local char = plr and plr.Character
    return char and char:FindFirstChildOfClass("Humanoid")
end

local function trim(s)
    return (string.match(tostring(s or ""), "^%s*(.-)%s*$"))
end

-- anonymous / streamer mode helpers
local ANON_TEXT = ".gg/strawberryhub"
S.anon = false

local function shownName()
    return S.anon and ANON_TEXT or player.DisplayName
end

local function shownUser()
    return S.anon and ANON_TEXT or ("@" .. player.Name)
end

local function fmtTime(sec)
    sec = math.max(0, math.floor(sec + 0.5))
    local h = math.floor(sec / 3600)
    local m = math.floor((sec % 3600) / 60)
    local s = sec % 60
    if h > 0 then return string.format("%dh %02dm %02ds", h, m, s) end
    if m > 0 then return string.format("%dm %02ds", m, s) end
    return s .. "s"
end

-- Roblox's own UI (ESC menu, player list, chat) lives in CoreGui.
-- While Anonymous Mode is on, your name / @username / headshot there are replaced.
local savedNameDist = nil

do
    local CoreGui = game:GetService("CoreGui")
    local origText = setmetatable({}, {__mode = "k"})
    local lastSet = setmetatable({}, {__mode = "k"})
    local origImg = setmetatable({}, {__mode = "k"})
    local hooks = {}
    local addedConn = nil
    local scanId = 0

    local function esc(str) return (string.gsub(str, "%W", "%%%0")) end

    local function anonymize(text)
        local names, seen = {}, {}
        for _, n in ipairs({player.Name, player.DisplayName}) do
            if n ~= "" and not seen[n] then
                seen[n] = true
                names[#names + 1] = n
            end
        end
        table.sort(names, function(a, b) return #a > #b end)
        local out = text
        for _, n in ipairs(names) do
            local e = esc(n)
            out = string.gsub(out, "@" .. e .. "%f[^%w_]", ANON_TEXT)
            out = string.gsub(out, "%f[%w_]" .. e .. "%f[^%w_]", ANON_TEXT)
        end
        return out
    end

    local function handleText(lbl)
        local ok, cur = pcall(function() return lbl.Text end)
        if not ok or type(cur) ~= "string" then return end
        local new = anonymize(cur)
        if new ~= cur then
            origText[lbl] = cur
            lastSet[lbl] = new
            lbl.Text = new
        elseif origText[lbl] and cur ~= lastSet[lbl] then
            -- Roblox reused this label for something else, do not restore stale text later
            origText[lbl] = nil
        end
    end

    local function handleImage(img)
        local ok, src = pcall(function() return img.Image end)
        if not ok or type(src) ~= "string" or src == "" then return end
        local low = string.lower(src)
        if string.find(src, tostring(player.UserId), 1, true)
            and (string.find(low, "rbxthumb", 1, true) or string.find(low, "thumbnail", 1, true)) then
            if origImg[img] == nil then origImg[img] = img.ImageTransparency end
            img.ImageTransparency = 1
        end
    end

    local function hookInst(inst)
        if hooks[inst] then return end
        if inst:IsA("TextLabel") or inst:IsA("TextButton") then
            hooks[inst] = inst:GetPropertyChangedSignal("Text"):Connect(function() handleText(inst) end)
            handleText(inst)
        elseif inst:IsA("ImageLabel") or inst:IsA("ImageButton") then
            hooks[inst] = inst:GetPropertyChangedSignal("Image"):Connect(function() handleImage(inst) end)
            handleImage(inst)
        end
    end

    H.coreAnon = function(on)
        scanId = scanId + 1
        local id = scanId
        if addedConn then addedConn:Disconnect(); addedConn = nil end
        for inst, c in pairs(hooks) do
            pcall(function() c:Disconnect() end)
            hooks[inst] = nil
        end
        if on then
            task.spawn(function()
                local ok, list = pcall(function() return CoreGui:GetDescendants() end)
                if not ok then return end
                local n = 0
                for _, inst in ipairs(list) do
                    if scanId ~= id or not running then return end
                    pcall(hookInst, inst)
                    n = n + 1
                    if n % 1500 == 0 then task.wait() end
                end
            end)
            pcall(function()
                addedConn = CoreGui.DescendantAdded:Connect(function(inst)
                    task.defer(function()
                        if S.anon and running then pcall(hookInst, inst) end
                    end)
                end)
            end)
        else
            for lbl, t in pairs(origText) do
                pcall(function() lbl.Text = t end)
                origText[lbl] = nil
            end
            for img, t in pairs(origImg) do
                pcall(function() img.ImageTransparency = t end)
                origImg[img] = nil
            end
        end
    end

    -- restores your own overhead name tag
    H.anonRestore = function()
        local hum = getHumanoid(player)
        if hum and savedNameDist then
            pcall(function() hum.DisplayDistanceType = savedNameDist end)
        end
        savedNameDist = nil
    end
end

for _, name in ipairs({"PrimevalHubGUI", "StrawberryHubGUI", "StrawberryLoader", "StrawberryFX",
    "StrawberryCooldowns", "StrawberryPicker", "StrawberryESP"}) do
    local old = playerGui:FindFirstChild(name)
    if old then old:Destroy() end
end

connect(player.Idled, function()
    if S.antiAfk then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)

local loader = Instance.new("ScreenGui")
loader.Name = "StrawberryLoader"
loader.ResetOnSpawn = false
loader.IgnoreGuiInset = true
loader.DisplayOrder = 100
loader.Parent = playerGui

local dim = Instance.new("Frame")
dim.Size = UDim2.new(1, 0, 1, 0)
dim.BackgroundColor3 = BLACK
dim.BackgroundTransparency = 1
dim.BorderSizePixel = 0
dim.Parent = loader

local card = Instance.new("Frame")
card.Size = UDim2.fromOffset(340, 96)
card.AnchorPoint = Vector2.new(0.5, 0.5)
card.Position = UDim2.new(0.5, 0, 0.5, 0)
card.BackgroundColor3 = Theme.Window
card.BorderSizePixel = 0
card.Parent = dim
do
    local st = Instance.new("UIStroke"); st.Color = BLACK; st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; st.Parent = card
end

local loaderScale = Instance.new("UIScale")
loaderScale.Scale = 0
loaderScale.Parent = card

local loaderInner = Instance.new("Frame")
loaderInner.Size = UDim2.new(1, -8, 1, -8)
loaderInner.Position = UDim2.fromOffset(4, 4)
loaderInner.BackgroundColor3 = Theme.Inner
loaderInner.BorderSizePixel = 0
loaderInner.Parent = card
do
    local st = Instance.new("UIStroke"); st.Color = Theme.Border; st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; st.Parent = loaderInner
end

local loaderAccent = Instance.new("Frame")
loaderAccent.Size = UDim2.new(1, 0, 0, 2)
loaderAccent.BackgroundColor3 = Theme.Accent
loaderAccent.BorderSizePixel = 0
loaderAccent.Parent = loaderInner

local function loaderLabel(text, y, size, color)
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, -16, 0, 16)
    l.Position = UDim2.fromOffset(8, y)
    l.BackgroundTransparency = 1
    l.Text = text
    l.TextColor3 = color
    l.TextSize = size
    l.Font = FONT
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = loaderInner
    return l
end
loaderLabel("Strawberry Hub " .. VERSION, 8, 15, Theme.Text)
local loaderStatus = loaderLabel("Planting seeds...", 28, 13, Theme.Dim)

local loaderTrack = Instance.new("Frame")
loaderTrack.Size = UDim2.new(1, -16, 0, 16)
loaderTrack.Position = UDim2.fromOffset(8, 52)
loaderTrack.BackgroundColor3 = Theme.Control
loaderTrack.BorderSizePixel = 0
loaderTrack.Parent = loaderInner

local loaderFill = Instance.new("Frame")
loaderFill.Size = UDim2.new(0.01, 0, 1, 0)
loaderFill.BackgroundColor3 = Theme.Accent
loaderFill.BorderSizePixel = 0
loaderFill.Parent = loaderTrack

local loaderPercent = Instance.new("TextLabel")
loaderPercent.Size = UDim2.new(1, 0, 1, 0)
loaderPercent.BackgroundTransparency = 1
loaderPercent.Text = "1%"
loaderPercent.TextColor3 = WHITE
loaderPercent.TextStrokeTransparency = 0.5
loaderPercent.TextSize = 12
loaderPercent.Font = FONT
loaderPercent.ZIndex = 2
loaderPercent.Parent = loaderTrack

local gui = Instance.new("ScreenGui")
gui.Name = "StrawberryHubGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = playerGui

local WIN_W, WIN_H = 600, 500
local COL_W = 274

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(WIN_W, WIN_H)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.Position = UDim2.new(0.5, 0, 0.5, 0)
main.BorderSizePixel = 0
main.Active = true
main.Visible = false
main.Parent = gui
skin(main, "BackgroundColor3", "Window")
addStroke(main, "Black", 1)

local uiScale = Instance.new("UIScale")
uiScale.Scale = 0
uiScale.Parent = main

local topLine = Instance.new("Frame")
topLine.Size = UDim2.new(1, 0, 0, 2)
topLine.BorderSizePixel = 0
topLine.Parent = main
skin(topLine, "BackgroundColor3", "Accent")

local top = Instance.new("Frame")
top.Size = UDim2.new(1, 0, 0, 28)
top.BackgroundTransparency = 1
top.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -50, 1, 0)
title.Position = UDim2.fromOffset(10, 1)
title.BackgroundTransparency = 1
title.Text = "Strawberry Hub " .. VERSION
title.TextSize = 15
title.Font = FONT
title.TextXAlignment = Enum.TextXAlignment.Left
title.TextTruncate = Enum.TextTruncate.AtEnd
title.Parent = top
skin(title, "TextColor3", "Text")

task.spawn(function()
    local ok, info = pcall(function() return MarketplaceService:GetProductInfo(game.PlaceId) end)
    if ok and info and info.Name then
        title.Text = "Strawberry Hub " .. VERSION .. "  |  " .. info.Name
    end
end)

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.fromOffset(22, 18)
closeButton.AnchorPoint = Vector2.new(1, 0.5)
closeButton.Position = UDim2.new(1, -8, 0.5, 1)
closeButton.BorderSizePixel = 0
closeButton.Text = "X"
closeButton.TextSize = 13
closeButton.Font = FONT
closeButton.AutoButtonColor = false
closeButton.Parent = top
skin(closeButton, "BackgroundColor3", "Control")
skin(closeButton, "TextColor3", "Text")
addStroke(closeButton, "Border", 1)
closeButton.MouseEnter:Connect(function() tween(closeButton, 0.12, {BackgroundColor3 = Color3.fromRGB(170, 30, 50)}) end)
closeButton.MouseLeave:Connect(function() tween(closeButton, 0.12, {BackgroundColor3 = Theme.Control}) end)

local inner = Instance.new("Frame")
inner.Size = UDim2.new(1, -16, 1, -61)
inner.Position = UDim2.fromOffset(8, 53)
inner.BorderSizePixel = 0
inner.Parent = main
skin(inner, "BackgroundColor3", "Inner")
addStroke(inner, "Border", 1)

local pages, tabRefresh = {}, {}
local currentPage = nil

local function createPage(name)
    local sf = Instance.new("ScrollingFrame")
    sf.Name = name
    sf.Size = UDim2.new(1, -12, 1, -12)
    sf.Position = UDim2.fromOffset(6, 6)
    sf.BackgroundTransparency = 1
    sf.BorderSizePixel = 0
    sf.ScrollBarThickness = 3
    sf.CanvasSize = UDim2.new(0, 0, 0, 0)
    sf.AutomaticCanvasSize = Enum.AutomaticSize.Y
    sf.ScrollingDirection = Enum.ScrollingDirection.Y
    sf.Visible = false
    sf.Parent = inner
    skin(sf, "ScrollBarImageColor3", "Accent")

    local function column(x)
        local f = Instance.new("Frame")
        f.BackgroundTransparency = 1
        f.Size = UDim2.new(0, COL_W, 0, 0)
        f.Position = UDim2.fromOffset(x, 0)
        f.AutomaticSize = Enum.AutomaticSize.Y
        f.Parent = sf
        local l = Instance.new("UIListLayout")
        l.Padding = UDim.new(0, 8)
        l.SortOrder = Enum.SortOrder.LayoutOrder
        l.Parent = f
        return f
    end

    local page = {Frame = sf, Left = column(0), Right = column(COL_W + 8)}
    pages[name] = page
    return page
end

local function switchPage(name)
    currentPage = name
    for pageName, page in pairs(pages) do
        local show = pageName == name
        page.Frame.Visible = show
        if show then
            -- small slide-in so switching tabs feels smooth
            page.Frame.Position = UDim2.fromOffset(6, 16)
            tween(page.Frame, 0.16, {Position = UDim2.fromOffset(6, 6)})
        end
    end
    for _, f in pairs(tabRefresh) do f() end
end

local TAB_NAMES = {"Main", "Autofarm", "Visuals", "Players", "Miscellaneous", "Settings"}

local function createTab(name, index, total)
    local gap = 2
    local width = (WIN_W - 16 - (total - 1) * gap) / total
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.fromOffset(width, 24)
    btn.Position = UDim2.fromOffset(8 + (index - 1) * (width + gap), 29)
    btn.BorderSizePixel = 0
    btn.Text = name
    btn.TextSize = 14
    btn.Font = FONT
    btn.AutoButtonColor = false
    btn.Parent = main
    addStroke(btn, "Border", 1)

    local hovered = false
    local function refresh()
        local a = currentPage == name
        btn.BackgroundColor3 = a and Theme.Inner or Theme.Control
        btn.TextColor3 = (a or hovered) and Theme.Text or Theme.Dim
    end
    tabRefresh[name] = refresh
    bind(refresh)

    btn.MouseEnter:Connect(function() hovered = true; refresh() end)
    btn.MouseLeave:Connect(function() hovered = false; refresh() end)
    btn.Activated:Connect(function()
        if currentPage ~= name then switchPage(name) end
    end)
end

local Pages = {}
for i, name in ipairs(TAB_NAMES) do
    Pages[name] = createPage(name)
    createTab(name, i, #TAB_NAMES)
end

local function createGroupbox(column, titleText)
    local box = Instance.new("Frame")
    box.Size = UDim2.new(1, 0, 0, 0)
    box.AutomaticSize = Enum.AutomaticSize.Y
    box.BorderSizePixel = 0
    box.Parent = column
    setOrder(box, column)
    skin(box, "BackgroundColor3", "Group")
    addStroke(box, "Border", 1)

    local line = Instance.new("Frame")
    line.Size = UDim2.new(1, 0, 0, 2)
    line.BorderSizePixel = 0
    line.Parent = box
    skin(line, "BackgroundColor3", "Accent")

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -16, 0, 18)
    label.Position = UDim2.fromOffset(8, 5)
    label.BackgroundTransparency = 1
    label.Text = titleText
    label.TextSize = 14
    label.Font = FONT
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = box
    skin(label, "TextColor3", "Text")

    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, 0, 0, 0)
    content.Position = UDim2.fromOffset(0, 26)
    content.AutomaticSize = Enum.AutomaticSize.Y
    content.BackgroundTransparency = 1
    content.Parent = box

    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 6)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = content

    local pad = Instance.new("UIPadding")
    pad.PaddingLeft = UDim.new(0, 8)
    pad.PaddingRight = UDim.new(0, 8)
    pad.PaddingBottom = UDim.new(0, 8)
    pad.Parent = content

    return content
end

local function createTabbox(column, names)
    local box = Instance.new("Frame")
    box.Size = UDim2.new(1, 0, 0, 0)
    box.AutomaticSize = Enum.AutomaticSize.Y
    box.BorderSizePixel = 0
    box.Parent = column
    setOrder(box, column)
    skin(box, "BackgroundColor3", "Group")
    addStroke(box, "Border", 1)

    local line = Instance.new("Frame")
    line.Size = UDim2.new(1, 0, 0, 2)
    line.BorderSizePixel = 0
    line.ZIndex = 3
    line.Parent = box
    skin(line, "BackgroundColor3", "Accent")

    local n = #names
    local tabs, contents = {}, {}
    local active = 1

    local function refresh()
        for i = 1, n do
            local on = i == active
            tabs[i].BackgroundColor3 = on and Theme.Group or Theme.Control
            tabs[i].TextColor3 = on and Theme.Text or Theme.Dim
            contents[i].Visible = on
            if on then
                contents[i].AutomaticSize = Enum.AutomaticSize.Y
            else
                contents[i].AutomaticSize = Enum.AutomaticSize.None
                contents[i].Size = UDim2.new(1, 0, 0, 0)
            end
        end
    end

    for i, name in ipairs(names) do
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(1 / n, 0, 0, 22)
        b.Position = UDim2.new((i - 1) / n, 0, 0, 2)
        b.BorderSizePixel = 0
        b.Text = name
        b.TextSize = 14
        b.Font = FONT
        b.AutoButtonColor = false
        b.Parent = box
        addStroke(b, "Border", 1)
        tabs[i] = b

        local c = Instance.new("Frame")
        c.Size = UDim2.new(1, 0, 0, 0)
        c.Position = UDim2.fromOffset(0, 28)
        c.AutomaticSize = Enum.AutomaticSize.Y
        c.BackgroundTransparency = 1
        c.Parent = box
        local l = Instance.new("UIListLayout")
        l.Padding = UDim.new(0, 6)
        l.SortOrder = Enum.SortOrder.LayoutOrder
        l.Parent = c
        local p = Instance.new("UIPadding")
        p.PaddingLeft = UDim.new(0, 8)
        p.PaddingRight = UDim.new(0, 8)
        p.PaddingTop = UDim.new(0, 4)
        p.PaddingBottom = UDim.new(0, 8)
        p.Parent = c
        contents[i] = c

        b.Activated:Connect(function()
            active = i
            refresh()
        end)
    end

    bind(refresh)
    return contents
end

local function createLabel(parent, text, height, bright)
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, 0, 0, height or 16)
    l.BackgroundTransparency = 1
    l.Text = text
    l.TextSize = 13
    l.Font = FONT
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.TextYAlignment = Enum.TextYAlignment.Top
    l.TextWrapped = true
    l.Parent = parent
    setOrder(l, parent)
    skin(l, "TextColor3", bright and "Text" or "Dim")
    return l
end

local pickerGui = Instance.new("ScreenGui")
pickerGui.Name = "StrawberryPicker"
pickerGui.ResetOnSpawn = false
pickerGui.IgnoreGuiInset = true
pickerGui.DisplayOrder = 150
pickerGui.Parent = playerGui

local popup = Instance.new("Frame")
popup.Size = UDim2.fromOffset(186, 152)
popup.BorderSizePixel = 0
popup.Visible = false
popup.Active = true
popup.Parent = pickerGui
skin(popup, "BackgroundColor3", "Window")
addStroke(popup, "Black", 1)
do
    local pl = Instance.new("Frame")
    pl.Size = UDim2.new(1, 0, 0, 2)
    pl.BorderSizePixel = 0
    pl.ZIndex = 5
    pl.Parent = popup
    skin(pl, "BackgroundColor3", "Accent")
end

local svBox = Instance.new("Frame")
svBox.Size = UDim2.fromOffset(150, 110)
svBox.Position = UDim2.fromOffset(8, 8)
svBox.BorderSizePixel = 0
svBox.Parent = popup
addStroke(svBox, "Black", 1)

local whiteOv = Instance.new("Frame")
whiteOv.Size = UDim2.new(1, 0, 1, 0)
whiteOv.BackgroundColor3 = WHITE
whiteOv.BorderSizePixel = 0
whiteOv.Parent = svBox
do
    local g = Instance.new("UIGradient")
    g.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(1, 1)})
    g.Parent = whiteOv
end

local blackOv = Instance.new("Frame")
blackOv.Size = UDim2.new(1, 0, 1, 0)
blackOv.BackgroundColor3 = BLACK
blackOv.BorderSizePixel = 0
blackOv.Parent = svBox
do
    local g = Instance.new("UIGradient")
    g.Rotation = 90
    g.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(1, 0)})
    g.Parent = blackOv
end

local svCursor = Instance.new("Frame")
svCursor.Size = UDim2.fromOffset(6, 6)
svCursor.BackgroundColor3 = WHITE
svCursor.BorderSizePixel = 0
svCursor.ZIndex = 4
svCursor.Parent = svBox
addStroke(svCursor, "Black", 1)

local hueBar = Instance.new("Frame")
hueBar.Size = UDim2.fromOffset(16, 110)
hueBar.Position = UDim2.fromOffset(164, 8)
hueBar.BackgroundColor3 = WHITE
hueBar.BorderSizePixel = 0
hueBar.Parent = popup
addStroke(hueBar, "Black", 1)
do
    local g = Instance.new("UIGradient")
    g.Rotation = 90
    g.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromHSV(0, 1, 1)),
        ColorSequenceKeypoint.new(1 / 6, Color3.fromHSV(1 / 6, 1, 1)),
        ColorSequenceKeypoint.new(2 / 6, Color3.fromHSV(2 / 6, 1, 1)),
        ColorSequenceKeypoint.new(3 / 6, Color3.fromHSV(3 / 6, 1, 1)),
        ColorSequenceKeypoint.new(4 / 6, Color3.fromHSV(4 / 6, 1, 1)),
        ColorSequenceKeypoint.new(5 / 6, Color3.fromHSV(5 / 6, 1, 1)),
        ColorSequenceKeypoint.new(1, Color3.fromHSV(0, 1, 1)),
    })
    g.Parent = hueBar
end

local hueCursor = Instance.new("Frame")
hueCursor.Size = UDim2.new(1, 4, 0, 2)
hueCursor.BackgroundColor3 = WHITE
hueCursor.BorderSizePixel = 0
hueCursor.ZIndex = 4
hueCursor.Parent = hueBar
addStroke(hueCursor, "Black", 1)

local hexBox = Instance.new("TextBox")
hexBox.Size = UDim2.fromOffset(172, 20)
hexBox.Position = UDim2.fromOffset(8, 124)
hexBox.BorderSizePixel = 0
hexBox.TextSize = 13
hexBox.Font = FONT
hexBox.ClearTextOnFocus = false
hexBox.Text = "FFFFFF"
hexBox.Parent = popup
skin(hexBox, "BackgroundColor3", "Control")
skin(hexBox, "TextColor3", "Text")
addStroke(hexBox, "Black", 1)

local pk = {h = 0, s = 0, v = 1, cb = nil, swatch = nil}
local pkDrag = nil

local function pkColor() return Color3.fromHSV(pk.h, pk.s, pk.v) end

local function pkRefresh()
    svBox.BackgroundColor3 = Color3.fromHSV(pk.h, 1, 1)
    svCursor.Position = UDim2.new(pk.s, -3, 1 - pk.v, -3)
    hueCursor.Position = UDim2.new(0, -2, pk.h, -1)
    hexBox.Text = string.upper(pkColor():ToHex())
end

local function pkCommit()
    local c = pkColor()
    if pk.swatch then pk.swatch.BackgroundColor3 = c end
    if pk.cb then pk.cb(c) end
end

local function pkUpdate(p)
    if pkDrag == "sv" then
        pk.s = math.clamp((p.X - svBox.AbsolutePosition.X) / svBox.AbsoluteSize.X, 0, 1)
        pk.v = 1 - math.clamp((p.Y - svBox.AbsolutePosition.Y) / svBox.AbsoluteSize.Y, 0, 1)
    elseif pkDrag == "hue" then
        pk.h = math.clamp((p.Y - hueBar.AbsolutePosition.Y) / hueBar.AbsoluteSize.Y, 0, 0.999)
    else
        return
    end
    pkRefresh()
    pkCommit()
end

svBox.InputBegan:Connect(function(input)
    if isPointerInput(input) then pkDrag = "sv"; pkUpdate(pointerPos(input)) end
end)
hueBar.InputBegan:Connect(function(input)
    if isPointerInput(input) then pkDrag = "hue"; pkUpdate(pointerPos(input)) end
end)
connect(UIS.InputChanged, function(input)
    if pkDrag and (input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch) then
        pkUpdate(pointerPos(input))
    end
end)
connect(UIS.InputEnded, function(input)
    if isPointerInput(input) then pkDrag = nil end
end)
hexBox.FocusLost:Connect(function()
    local txt = string.gsub(trim(hexBox.Text), "#", "")
    local ok, c = pcall(function() return Color3.fromHex(txt) end)
    if ok and c then
        pk.h, pk.s, pk.v = c:ToHSV()
        pkRefresh()
        pkCommit()
    else
        pkRefresh()
    end
end)

local function closePicker()
    popup.Visible = false
    pk.swatch, pk.cb = nil, nil
end

local function openPicker(swatch, color, cb)
    if popup.Visible and pk.swatch == swatch then closePicker(); return end
    pk.swatch, pk.cb = swatch, cb
    pk.h, pk.s, pk.v = color:ToHSV()
    pkRefresh()
    local vp = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
    local x = math.clamp(swatch.AbsolutePosition.X + swatch.AbsoluteSize.X - 186, 4, vp.X - 190)
    local y = math.clamp(swatch.AbsolutePosition.Y + swatch.AbsoluteSize.Y + 4, 4, vp.Y - 156)
    popup.Position = UDim2.fromOffset(x, y)
    popup.Visible = true
end

local function attachColor(row, default, onChange, key, slot)
    slot = slot or 1
    local current = default

    local sw = Instance.new("TextButton")
    sw.Size = UDim2.fromOffset(26, 12)
    sw.AnchorPoint = Vector2.new(1, 0.5)
    sw.Position = UDim2.new(1, -((slot - 1) * 32), 0.5, 0)
    sw.BackgroundColor3 = default
    sw.BorderSizePixel = 0
    sw.Text = ""
    sw.AutoButtonColor = false
    sw.ZIndex = 3
    sw.Parent = row
    addStroke(sw, "Black", 1)

    local function setColor(c)
        if type(c) == "table" then
            c = Color3.fromRGB(c[1] or 255, c[2] or 255, c[3] or 255)
        end
        if typeof(c) ~= "Color3" then return end
        current = c
        sw.BackgroundColor3 = c
        onChange(c)
    end

    sw.Activated:Connect(function()
        openPicker(sw, current, function(c)
            current = c
            onChange(c)
        end)
    end)

    if key then
        Options[key] = {
            Default = default,
            Get = function()
                return {math.floor(current.R * 255 + 0.5), math.floor(current.G * 255 + 0.5), math.floor(current.B * 255 + 0.5)}
            end,
            Set = setColor,
        }
    end
    return {Set = setColor, Swatch = sw}
end

local function createColorRow(parent, text, default, onChange, key)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 18)
    row.BackgroundTransparency = 1
    row.Parent = parent
    setOrder(row, parent)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -40, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextSize = 14
    label.Font = FONT
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = row
    skin(label, "TextColor3", "Text")

    return attachColor(row, default, onChange, key, 1)
end

local function createCheckbox(parent, text, initial, onToggle, key)
    local row = Instance.new("TextButton")
    row.Size = UDim2.new(1, 0, 0, 18)
    row.BackgroundTransparency = 1
    row.BorderSizePixel = 0
    row.Text = ""
    row.AutoButtonColor = false
    row.Parent = parent
    setOrder(row, parent)

    local state = initial and true or false

    local box = Instance.new("Frame")
    box.Size = UDim2.fromOffset(14, 14)
    box.Position = UDim2.new(0, 0, 0.5, -7)
    box.BorderSizePixel = 0
    box.Parent = row
    local boxStroke = addStroke(box, "Black", 1)
    bind(function() box.BackgroundColor3 = state and Theme.Accent or Theme.Control end)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -24, 1, 0)
    label.Position = UDim2.fromOffset(22, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextSize = 14
    label.Font = FONT
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = row
    skin(label, "TextColor3", "Text")

    row.MouseEnter:Connect(function() boxStroke.Color = Theme.Accent end)
    row.MouseLeave:Connect(function() boxStroke.Color = Theme.Black end)

    local function setState(v)
        state = v and true or false
        tween(box, 0.12, {BackgroundColor3 = state and Theme.Accent or Theme.Control})
        onToggle(state)
    end

    row.Activated:Connect(function() setState(not state) end)

    if key then
        Options[key] = {Default = initial and true or false, Get = function() return state end, Set = setState}
    end
    if state then task.defer(onToggle, true) end

    return {Row = row, Label = label, Set = setState, Get = function() return state end}
end

local function createSlider(parent, text, minV, maxV, initial, suffix, onChange, key)
    suffix = suffix or ""

    local holder = Instance.new("Frame")
    holder.Size = UDim2.new(1, 0, 0, 36)
    holder.BackgroundTransparency = 1
    holder.Parent = parent
    setOrder(holder, parent)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 16)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextSize = 14
    label.Font = FONT
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = holder
    skin(label, "TextColor3", "Text")

    local track = Instance.new("Frame")
    track.Size = UDim2.new(1, 0, 0, 16)
    track.Position = UDim2.fromOffset(0, 19)
    track.BorderSizePixel = 0
    track.Parent = holder
    skin(track, "BackgroundColor3", "Control")
    addStroke(track, "Black", 1)

    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((initial - minV) / (maxV - minV), 0, 1, 0)
    fill.BorderSizePixel = 0
    fill.Parent = track
    skin(fill, "BackgroundColor3", "Accent")

    local valueLabel = Instance.new("TextLabel")
    valueLabel.Size = UDim2.new(1, 0, 1, 0)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Text = initial .. suffix .. "/" .. maxV .. suffix
    valueLabel.TextColor3 = WHITE
    valueLabel.TextStrokeTransparency = 0.5
    valueLabel.TextSize = 13
    valueLabel.Font = FONT
    valueLabel.ZIndex = 2
    valueLabel.Parent = track

    local sliding = false
    local current = initial

    local function setValue(value)
        value = math.clamp(math.floor((tonumber(value) or initial) + 0.5), minV, maxV)
        current = value
        fill.Size = UDim2.new((value - minV) / (maxV - minV), 0, 1, 0)
        valueLabel.Text = value .. suffix .. "/" .. maxV .. suffix
        onChange(value)
    end

    local function update(x)
        local ratio = math.clamp((x - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        setValue(minV + ratio * (maxV - minV))
    end

    track.InputBegan:Connect(function(input)
        if isPointerInput(input) then
            sliding = true
            update(input.Position.X)
        end
    end)
    connect(UIS.InputChanged, function(input)
        if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch) then
            update(input.Position.X)
        end
    end)
    connect(UIS.InputEnded, function(input)
        if isPointerInput(input) then sliding = false end
    end)

    if key then
        Options[key] = {Default = initial, Get = function() return current end, Set = setValue}
    end
    return {Set = setValue}
end

local function refreshButton(b)
    local a = b:GetAttribute("Active")
    b.BackgroundColor3 = a and Theme.Accent or Theme.Control
    b.TextColor3 = a and contrast(Theme.Accent) or Theme.Text
end

local function styleButton(b)
    b.BorderSizePixel = 0
    b.TextSize = 14
    b.Font = FONT
    b.AutoButtonColor = false
    addStroke(b, "Black", 1)
    bind(function() refreshButton(b) end)

    b.MouseEnter:Connect(function()
        if not b:GetAttribute("Active") then tween(b, 0.1, {BackgroundColor3 = Theme.Hover}) end
    end)
    b.MouseLeave:Connect(function()
        if not b:GetAttribute("Active") then tween(b, 0.1, {BackgroundColor3 = Theme.Control}) end
    end)
end

local function setButtonActive(b, active)
    b:SetAttribute("Active", active)
    refreshButton(b)
end

local function createButton(parent, text)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 22)
    b.Text = text
    b.Parent = parent
    styleButton(b)
    setOrder(b, parent)
    return b
end

local function createTextbox(parent, text, placeholder, default, onChanged, key)
    local holder = Instance.new("Frame")
    holder.Size = UDim2.new(1, 0, 0, 38)
    holder.BackgroundTransparency = 1
    holder.Parent = parent
    setOrder(holder, parent)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 16)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextSize = 14
    label.Font = FONT
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = holder
    skin(label, "TextColor3", "Text")

    local box = Instance.new("TextBox")
    box.Size = UDim2.new(1, 0, 0, 18)
    box.Position = UDim2.fromOffset(0, 19)
    box.BorderSizePixel = 0
    box.Text = default or ""
    box.PlaceholderText = placeholder or ""
    box.TextSize = 13
    box.Font = FONT
    box.TextXAlignment = Enum.TextXAlignment.Left
    box.ClearTextOnFocus = false
    box.ClipsDescendants = true
    box.Parent = holder
    skin(box, "BackgroundColor3", "Control")
    skin(box, "TextColor3", "Text")
    skin(box, "PlaceholderColor3", "Dim")
    local boxStroke = addStroke(box, "Black", 1)

    local pad = Instance.new("UIPadding")
    pad.PaddingLeft = UDim.new(0, 5)
    pad.PaddingRight = UDim.new(0, 5)
    pad.Parent = box

    box.Focused:Connect(function() boxStroke.Color = Theme.Accent end)
    box.FocusLost:Connect(function()
        boxStroke.Color = Theme.Black
        onChanged(box.Text)
    end)

    local function setText(v)
        box.Text = tostring(v or "")
        onChanged(box.Text)
    end

    if key then
        Options[key] = {Default = default or "", Get = function() return box.Text end, Set = setText}
    end
    return {Box = box, Set = setText}
end

local Keybinds = {}
local listeningKb = nil

local function createKeybind(parent, text, defaultName, action, key)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 20)
    row.BackgroundTransparency = 1
    row.Parent = parent
    setOrder(row, parent)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -90, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextSize = 14
    label.Font = FONT
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = row
    skin(label, "TextColor3", "Text")

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.fromOffset(82, 18)
    btn.AnchorPoint = Vector2.new(1, 0.5)
    btn.Position = UDim2.new(1, 0, 0.5, 0)
    btn.Text = defaultName or "None"
    btn.Parent = row
    styleButton(btn)

    local kb = {Key = nil, Action = action, Btn = btn}

    local function setKey(name)
        local kc = nil
        if name and name ~= "None" then
            local ok, v = pcall(function() return Enum.KeyCode[name] end)
            if ok then kc = v end
        end
        kb.Key = kc
        btn.Text = kc and kc.Name or "None"
        setButtonActive(btn, false)
    end
    kb.SetKey = setKey
    setKey(defaultName)

    btn.Activated:Connect(function()
        listeningKb = kb
        btn.Text = "..."
        setButtonActive(btn, true)
    end)

    Keybinds[#Keybinds + 1] = kb
    if key then
        Options[key] = {Default = defaultName or "None", Get = function() return kb.Key and kb.Key.Name or "None" end, Set = setKey}
    end
    return kb
end

connect(UIS.InputBegan, function(input, gp)

    if popup.Visible and isPointerInput(input) then
        local p = pointerPos(input)
        if not inside(popup, p) and not (pk.swatch and inside(pk.swatch, p)) then
            closePicker()
        end
    end

    if listeningKb then
        if input.UserInputType == Enum.UserInputType.Keyboard then
            local kb = listeningKb
            listeningKb = nil
            if input.KeyCode == Enum.KeyCode.Escape then
                kb.SetKey(kb.Key and kb.Key.Name or "None")
            elseif input.KeyCode == Enum.KeyCode.Backspace then
                kb.SetKey("None")
            else
                kb.SetKey(input.KeyCode.Name)
            end
        end
        return
    end

    if gp or not loaded then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        for _, kb in ipairs(Keybinds) do
            if kb.Key and kb.Key == input.KeyCode then
                pcall(kb.Action)
            end
        end
    end
end)

local ZonesList = {
    { Position = Vector3.new(-144.54, -102.15, 1032.19), Teleport = CFrame.new(-144.54, -87.15, 1032.19) },
    { Position = Vector3.new(-617.92, 143.39, -1130.44), Teleport = CFrame.new(-617.92, 158.39, -1130.44) },
    { Position = Vector3.new(1449.99, 77.16, -677.59), Teleport = CFrame.new(1449.99, 92.16, -677.59) },
    { Position = Vector3.new(276.03, 75.50, -1029.02), Teleport = CFrame.new(276.03, 90.50, -1029.02) },
    { Position = Vector3.new(-513.23, 75.73, -430.79), Teleport = CFrame.new(-513.23, 90.73, -430.79) },
    { Position = Vector3.new(1090.65, 66.03, 467.14), Teleport = CFrame.new(1090.65, 81.03, 467.14) },
    { Position = Vector3.new(209.25, -32.79, -359.19), Teleport = CFrame.new(209.25, -17.79, -359.19) },
    { Position = Vector3.new(-170.20, 109.98, 388.42), Teleport = CFrame.new(-170.20, 124.98, 388.42) },
    { Position = Vector3.new(-69.11, -61.16, -1493.54), Teleport = CFrame.new(-69.11, -46.16, -1493.54) }
}

local function IsThisZoneGreen(targetPos)
    local isGreen = false
    pcall(function()
        local folder = (workspace:FindFirstChild("MapResources") :: any).Zones
        for _, zoneObj in pairs(folder:GetChildren()) do
            local core = zoneObj:FindFirstChild("LightCore")
            if core and (core.Position - targetPos).Magnitude < 15 then
                local color = core.Color
                if color.R == 0 and color.G > 0.9 and color.B == 0 then
                    isGreen = true
                end
                break
            end
        end
    end)
    return isGreen
end

local wm = Instance.new("Frame")
wm.AutomaticSize = Enum.AutomaticSize.X
wm.Size = UDim2.new(0, 0, 0, 22)
wm.Position = UDim2.fromOffset(10, 10)
wm.BorderSizePixel = 0
wm.Visible = false
wm.Parent = gui
skin(wm, "BackgroundColor3", "Window")
addStroke(wm, "Border", 1)

local wmLine = Instance.new("Frame")
wmLine.Size = UDim2.new(1, 0, 0, 2)
wmLine.BorderSizePixel = 0
wmLine.Parent = wm
skin(wmLine, "BackgroundColor3", "Accent")

local wmText = Instance.new("TextLabel")
wmText.AutomaticSize = Enum.AutomaticSize.X
wmText.Size = UDim2.new(0, 0, 1, 0)
wmText.BackgroundTransparency = 1
wmText.Text = "Strawberry Hub"
wmText.TextSize = 13
wmText.Font = FONT
wmText.Parent = wm
skin(wmText, "TextColor3", "Text")
do
    local p = Instance.new("UIPadding")
    p.PaddingLeft = UDim.new(0, 8)
    p.PaddingRight = UDim.new(0, 8)
    p.PaddingTop = UDim.new(0, 2)
    p.Parent = wmText
end

task.spawn(function()
    local frames, last = 0, tick()
    connect(RunService.RenderStepped, function() frames = frames + 1 end)
    while running do
        task.wait(0.5)
        local now = tick()
        local fps = math.floor(frames / math.max(now - last, 0.01) + 0.5)
        frames, last = 0, now
        local ping = 0
        pcall(function() ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue() + 0.5) end)
        wmText.Text = string.format("Strawberry Hub %s  |  %s  |  %d fps  |  %d ms", VERSION, shownName(), fps, ping)
    end
end)

do
    wm.Active = true
    local dragging, dragStart, startPos = false, nil, nil

    wm.InputBegan:Connect(function(input)
        if isPointerInput(input) then
            dragging = true
            dragStart = input.Position
            startPos = wm.Position
        end
    end)
    connect(UIS.InputEnded, function(input)
        if isPointerInput(input) then dragging = false end
    end)
    connect(UIS.InputChanged, function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            local vp = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
            local x = math.clamp(startPos.X.Offset + d.X, 0, math.max(vp.X - wm.AbsoluteSize.X, 0))
            local y = math.clamp(startPos.Y.Offset + d.Y, 0, math.max(vp.Y - wm.AbsoluteSize.Y, 0))
            wm.Position = UDim2.fromOffset(x, y)
        end
    end)

    Options.wmPos = {
        Default = {10, 10},
        Get = function() return {wm.Position.X.Offset, wm.Position.Y.Offset} end,
        Set = function(v)
            if type(v) == "table" and tonumber(v[1]) and tonumber(v[2]) then
                wm.Position = UDim2.fromOffset(v[1], v[2])
            end
        end,
    }
end

----------------------------------------------------------------------
-- MAIN + AUTOFARM PAGES
----------------------------------------------------------------------
local zoneRow

-- Main page, dino growth, fossils and Autofarm live in their own block
-- (keeps the main script under Roblox's 200 local variable limit)
do
    local sessionStart = os.clock()

    -- progress bar that follows the theme
    local function createBar(parent, height)
        local track = Instance.new("Frame")
        track.Size = UDim2.new(1, 0, 0, height or 16)
        track.BorderSizePixel = 0
        track.Parent = parent
        setOrder(track, parent)
        skin(track, "BackgroundColor3", "Control")
        addStroke(track, "Black", 1)

        local fill = Instance.new("Frame")
        fill.Size = UDim2.new(0, 0, 1, 0)
        fill.BorderSizePixel = 0
        fill.Parent = track
        skin(fill, "BackgroundColor3", "Accent")

        local txt = Instance.new("TextLabel")
        txt.Size = UDim2.new(1, 0, 1, 0)
        txt.BackgroundTransparency = 1
        txt.TextColor3 = WHITE
        txt.TextStrokeTransparency = 0.5
        txt.TextSize = 12
        txt.Font = FONT
        txt.ZIndex = 3
        txt.Parent = track

        return {
            Track = track,
            Set = function(frac, text)
                fill.Size = UDim2.new(math.clamp(frac, 0, 1), 0, 1, 0)
                txt.Text = text or ""
            end,
        }
    end

    ----------------------------------------------------------------------
    -- DINOSAUR GROWTH CONFIG
    -- I do not know which attribute / value this game uses for growth, so the script
    -- scans your Player and Character for anything that looks like growth.
    -- If it picks the wrong one (or none), press "Scan Growth Values" on the Main page,
    -- then put the exact name in ForceName below.
    ----------------------------------------------------------------------
    local DINO = {
        ForceName = nil,        -- e.g. "Growth"  (exact name of the Attribute / Value object)
        MaxGrowth = nil,        -- e.g. 100 or 1; nil = auto (0-1 or 0-100)
        -- {stage name, growth fraction where the stage starts}. Edit to match the game.
        Stages = {
            {"Baby", 0},
            {"Juvenile", 0.25},
            {"Adult", 0.5},
            {"Elder", 0.9},
        },
    }

    local dinoState = {src = nil, lastSrc = nil, samples = {}, lastFrac = nil, char = nil, trend = {},
        hist = {}, eta = {}, lastIdx = nil, since = os.clock()}

    -- values we never treat as growth when guessing by trend
    local SKIP_NAMES = {"health", "hp", "hunger", "food", "thirst", "water", "stamina", "oxygen", "energy",
        "mana", "armor", "shield", "speed", "ping", "fps", "damage", "kills", "coins", "cash", "money",
        "gold", "points", "score", "ordercount", "cooldown", "time", "fossil", "bone", "gem", "token", "credit"}

    local function isSkipped(name)
        local l = string.lower(name)
        for _, k in ipairs(SKIP_NAMES) do
            if string.find(l, k, 1, true) then return true end
        end
        return false
    end

    local function matchesGrowth(name)
        local l = string.lower(name)
        if DINO.ForceName then return name == DINO.ForceName end
        if l == "age" then return true end
        for _, k in ipairs({"grow", "stage", "matur", "lifestage", "evol"}) do
            if string.find(l, k, 1, true) then return true end
        end
        if string.sub(l, -3) == "age" then
            for _, bad in ipairs({"damage", "image", "message", "package", "storage", "mileage", "percentage", "voltage", "usage", "language"}) do
                if string.sub(l, -#bad) == bad then return false end
            end
            return true
        end
        return false
    end

    -- collects every number/string attribute or Value object that passes pred(name, value)
    -- looks at: your Player (not PlayerGui), your Character, and workspace / ReplicatedStorage entries named after you
    local function gather(pred)
        local found = {}

        local function consider(path, name, value, inst, reader)
            if (type(value) == "number" or type(value) == "string") and pred(name, value) then
                found[#found + 1] = {path = path, name = name, value = value, inst = inst, read = reader}
            end
        end

        local function scanOne(inst, path)
            for n, v in pairs(inst:GetAttributes()) do
                consider(path .. "@" .. n, n, v, inst, function() return inst:GetAttribute(n) end)
            end
            if inst:IsA("NumberValue") or inst:IsA("IntValue") or inst:IsA("StringValue") then
                consider(path, inst.Name, inst.Value, inst, function() return inst.Value end)
            end
        end

        local function scanTree(root, label)
            scanOne(root, label)
            for _, d in ipairs(root:GetDescendants()) do
                scanOne(d, label .. "/" .. d.Name)
            end
        end

        scanOne(player, "Player")
        for _, c in ipairs(player:GetChildren()) do
            if c.Name ~= "PlayerGui" and c.Name ~= "PlayerScripts" then
                scanTree(c, "Player/" .. c.Name)
            end
        end

        local char = player.Character
        if char then scanTree(char, "Character") end

        local uid = tostring(player.UserId)
        for _, c in ipairs(workspace:GetChildren()) do
            if c ~= char and not c:IsA("Terrain") and not c:IsA("Camera") then
                if c.Name == player.Name or c.Name == uid then
                    scanTree(c, "Workspace/" .. c.Name)
                else
                    local x = c:FindFirstChild(player.Name) or c:FindFirstChild(uid)
                    if x and x ~= char then scanTree(x, "Workspace/" .. c.Name .. "/" .. x.Name) end
                end
            end
        end

        pcall(function()
            for _, c in ipairs(game:GetService("ReplicatedStorage"):GetChildren()) do
                local x = (c.Name == player.Name or c.Name == uid) and c
                    or c:FindFirstChild(player.Name) or c:FindFirstChild(uid)
                if x then scanTree(x, "ReplicatedStorage/" .. c.Name) end
            end
        end)

        return found
    end

    local function collectCandidates()
        return gather(function(n) return matchesGrowth(n) end)
    end

    -- best guess at which dinosaur you are playing
    local function getSpecies()
        local char = player.Character
        if not char then return nil end
        local function isSpeciesName(n)
            n = string.lower(n)
            return string.find(n, "dino", 1, true) or string.find(n, "species", 1, true)
                or string.find(n, "creature", 1, true) or string.find(n, "animal", 1, true)
        end
        for _, holder in ipairs({char, player}) do
            for n, v in pairs(holder:GetAttributes()) do
                if type(v) == "string" and v ~= "" and isSpeciesName(n) then return v end
            end
            for _, d in ipairs(holder:GetChildren()) do
                if d:IsA("StringValue") and d.Value ~= "" and isSpeciesName(d.Name) then return d.Value end
            end
        end
        if char.Name ~= player.Name then return char.Name end
        return nil
    end
    H.getSpecies = getSpecies

    ----------------------------------------------------------------------
    -- FOSSILS (exact amount, with thousands separators)
    ----------------------------------------------------------------------
    local function formatNumber(n)
        local str = string.format("%.0f", n)
        local sign, digits = string.match(str, "^(-?)(%d+)$")
        if not digits then return str end
        local rev = string.reverse(digits)
        rev = string.gsub(rev, "(%d%d%d)", "%1,")
        rev = string.gsub(rev, ",$", "")
        return sign .. string.reverse(rev)
    end

    -- returns number, approximate?  (approximate = the game only stores an abbreviated text like "1.14M")
    local function parseFossil(v)
        if type(v) == "number" then return v, false end
        if type(v) == "string" then
            local clean = string.gsub(v, ",", "")
            local n = tonumber(clean)
            if n then return n, false end
            local num, suf = string.match(clean, "^%s*([%d%.]+)%s*([KkMmBbTt])%s*$")
            if num and tonumber(num) then
                local mult = ({k = 1e3, m = 1e6, b = 1e9, t = 1e12})[string.lower(suf)]
                return tonumber(num) * mult, true
            end
        end
        return nil
    end

    local fossilState = {pick = nil, nextScan = 0}

    local function findFossils()
        local p = fossilState.pick
        if p then
            local ok, v = pcall(p.read)
            if ok and v ~= nil and p.inst and p.inst.Parent then
                local n, approx = parseFossil(v)
                if n then return n, approx end
            end
            fossilState.pick = nil
        end
        local now = os.clock()
        if now < fossilState.nextScan then return nil end
        fossilState.nextScan = now + 3

        local cands = gather(function(name, v)
            return string.find(string.lower(name), "fossil", 1, true) ~= nil and parseFossil(v) ~= nil
        end)
        if #cands == 0 then return nil end

        local best = cands[1]
        for _, c in ipairs(cands) do
            local l = string.lower(c.name)
            if l == "fossils" or l == "fossil" then best = c; break end
        end
        fossilState.pick = best
        return parseFossil(best.value)
    end

    ----------------------------------------------------------------------
    -- STAGE TEXT (what the game itself shows on your screen)
    ----------------------------------------------------------------------
    local STAGE_ALIASES = {
        {"baby", "hatchling", "hatchlings"},
        {"juvenile", "juv", "teen", "teenager"},
        {"adult"},
        {"elder", "ancient"},
    }

    local function normWords(text)
        local t = string.gsub(tostring(text), "<[^>]+>", "")
        t = string.lower(t)
        t = string.gsub(t, "[^%a]+", " ")
        return " " .. trim(t) .. " "
    end

    local function exactStage(w)
        if w == " sub adult " or w == " subadult " then return 2 end
        for idx, list in ipairs(STAGE_ALIASES) do
            for _, a in ipairs(list) do
                if w == " " .. a .. " " then return idx end
            end
        end
        return nil
    end

    local function stageFromWords(w)
        if string.find(w, " sub adult ", 1, true) or string.find(w, " subadult ", 1, true) then return 2 end
        for idx, list in ipairs(STAGE_ALIASES) do
            for _, a in ipairs(list) do
                if string.find(w, " " .. a .. " ", 1, true) then return idx end
            end
        end
        return nil
    end

    local function stageFromString(v)
        if type(v) ~= "string" then return nil end
        local w = normWords(v)
        return exactStage(w) or stageFromWords(w)
    end

    local function parseDuration(text)
        local h, m, sec = string.match(text, "(%d+):(%d+):(%d+)")
        if h then return (tonumber(h) or 0) * 3600 + (tonumber(m) or 0) * 60 + (tonumber(sec) or 0) end
        local m2, s2 = string.match(text, "(%d+):(%d+)")
        if m2 then return (tonumber(m2) or 0) * 60 + (tonumber(s2) or 0) end
        local total, hit = 0, false
        for n, u in string.gmatch(string.lower(text), "(%d+)%s*([hms])%f[%A]") do
            hit = true
            total = total + (tonumber(n) or 0) * (u == "h" and 3600 or (u == "m" and 60 or 1))
        end
        if hit then return total end
        return nil
    end

    local OWN_GUIS = {StrawberryHubGUI = true, StrawberryLoader = true, StrawberryPicker = true, StrawberryESP = true,
        StrawberryFX = true, StrawberryCooldowns = true, PrimevalHubGUI = true}
    local GROW_KEYS = {"grow", "stage", "matur", "evol", "life"}
    local hud = {labels = {}, nextScan = 0}

    local function rescanHud()
        local list = {}
        for _, g in ipairs(playerGui:GetChildren()) do
            if not OWN_GUIS[g.Name] then
                for _, d in ipairs(g:GetDescendants()) do
                    if d:IsA("TextLabel") or d:IsA("TextButton") then list[#list + 1] = d end
                end
            end
        end
        hud.labels = list
    end

    local function isShown(lbl)
        local o = lbl
        while o and o ~= playerGui do
            if o:IsA("GuiObject") and not o.Visible then return false end
            if o:IsA("LayerCollector") and not o.Enabled then return false end
            o = o.Parent
        end
        return o == playerGui
    end

    -- reads the stage word / growth timer straight off the game's HUD
    -- returns stageIdx, seconds, stageLabelPath, timeLabelPath
    local function hudRead()
        local now = os.clock()
        if now >= hud.nextScan then
            hud.nextScan = now + 5
            rescanHud()
        end
        local stages, count = {}, 0
        local stageDesc, secs, timeDesc = nil, nil, nil
        for _, lbl in ipairs(hud.labels) do
            local _ok = pcall(function()
                if lbl.Parent and #lbl.Text > 0 and #lbl.Text <= 60 and isShown(lbl) then
                    local text = string.gsub(lbl.Text, "<[^>]+>", "")
                    local ctx = string.lower(text .. " " .. lbl.Name .. " " .. (lbl.Parent and lbl.Parent.Name or ""))
                    local hasCtx = false
                    for _, k in ipairs(GROW_KEYS) do
                        if string.find(ctx, k, 1, true) then hasCtx = true; break end
                    end
                    local w = normWords(text)
                    local idx = exactStage(w) or (hasCtx and stageFromWords(w) or nil)
                    if idx and not stages[idx] then
                        stages[idx] = lbl
                        count = count + 1
                        stageDesc = lbl:GetFullName()
                    end
                    if hasCtx and not secs then
                        local d = parseDuration(text)
                        if d then secs = d; timeDesc = lbl:GetFullName() end
                    end
                end
            end)
        end
        local stageIdx = nil
        if count == 1 then
            for idx in pairs(stages) do stageIdx = idx end
        else
            stageDesc = nil
        end
        return stageIdx, secs, stageDesc, timeDesc
    end

    ----------------------------------------------------------------------
    -- NUMERIC GROWTH VALUES
    ----------------------------------------------------------------------
    -- plain 0-1 / 0-100 value -> fraction (nil if it cannot be a fraction)
    local function toFraction(c)
        local v = c.value
        if type(v) ~= "number" then return nil end
        local max = DINO.MaxGrowth
        if not max then
            if v <= 1 then max = 1 elseif v <= 100 then max = 100 else return nil end
        end
        return math.clamp(v / max, 0, 1)
    end

    local function _stageFromFraction(f)
        local idx = 1
        for i, st in ipairs(DINO.Stages) do
            if f >= st[2] then idx = i end
        end
        return idx
    end

    -- fallback: watch every number on you; a value that only ever goes up is probably growth
    local function trendPick()
        local now = os.clock()
        local vals = gather(function(name, value)
            return type(value) == "number" and not isSkipped(name)
        end)
        local seen = {}
        for _, c in ipairs(vals) do
            seen[c.path] = true
            local t = dinoState.trend[c.path]
            if not t then
                dinoState.trend[c.path] = {first = c.value, last = c.value, rises = 0, falls = 0, t0 = now, name = c.name}
            else
                if c.value > t.last then t.rises = t.rises + 1
                elseif c.value < t.last then t.falls = t.falls + 1 end
                t.last = c.value
            end
        end
        for path in pairs(dinoState.trend) do
            if not seen[path] then dinoState.trend[path] = nil end
        end

        local best, bestRises = nil, 1
        for path, t in pairs(dinoState.trend) do
            if t.falls == 0 and t.rises > bestRises and t.last > t.first then
                local cand = {path = path .. " (auto)", name = t.name, value = t.last}
                if toFraction(cand) then
                    best, bestRises = cand, t.rises
                end
            end
        end
        return best
    end

    -- turns the chosen value into (fraction, stageIndex, countdownSeconds). Any of them can be nil.
    -- It deliberately returns nothing when it cannot tell, instead of guessing "Elder".
    local function interpret(pick)
        local v, name = pick.value, string.lower(pick.name)

        if type(v) == "string" then
            local idx = stageFromString(v)
            if idx then return DINO.Stages[idx][2], idx, nil end
            return nil
        end

        local h = dinoState.hist[pick.path]
        if not h then
            h = {first = v, last = v, min = v, max = v, rises = 0, falls = 0, nonint = false}
            dinoState.hist[pick.path] = h
        else
            if v > h.last then h.rises = h.rises + 1 elseif v < h.last then h.falls = h.falls + 1 end
            h.last = v
            h.min = math.min(h.min, v)
            h.max = math.max(h.max, v)
        end
        if v % 1 ~= 0 then h.nonint = true end

        -- a stage / tier / phase number (1-4 or 0-3)
        local stagey = string.find(name, "stage", 1, true) or string.find(name, "tier", 1, true)
            or string.find(name, "phase", 1, true)
        if stagey and v % 1 == 0 then
            local idx = (h.min == 0) and (v + 1) or v
            if idx >= 1 and idx <= #DINO.Stages then return DINO.Stages[idx][2], idx, nil end
        end

        -- a value that counts DOWN is a "time left" style value
        if h.falls >= 2 and h.rises == 0 and h.max > 0 then
            local timey = string.find(name, "time", 1, true) or string.find(name, "left", 1, true)
                or string.find(name, "remain", 1, true) or string.find(name, "timer", 1, true)
                or string.find(name, "count", 1, true) or string.find(name, "until", 1, true)
            return math.clamp(1 - v / h.max, 0, 1), nil, (timey and v or nil)
        end

        -- exactly 1 with no history could be "stage 1" or "100%", so do not guess
        if v == 1 and not h.nonint and h.max <= 1 and h.rises == 0 then
            return nil
        end

        return toFraction(pick), nil, nil
    end

    -- keeps one fixed target time so the countdown does not jump around
    local function _stableEta(key, now, secs, tol)
        local e = dinoState.eta[key]
        local target = now + secs
        if not e or math.abs(target - e) > math.max(tol or 8, secs * 0.12) then
            dinoState.eta[key] = target
            e = target
        end
        return e
    end

    local function _fmtClock(sec)
        sec = math.max(0, math.ceil(sec))
        local h = math.floor(sec / 3600)
        local m = math.floor((sec % 3600) / 60)
        local sc = sec % 60
        if h > 0 then return string.format("%d:%02d:%02d", h, m, sc) end
        return string.format("%02d:%02d", m, sc)
    end

    -- Detection is automatic and restarts for every new dinosaur you spawn.
    -- returns an info table or nil
    local function _readGrowth()
        local char = player.Character
        if char ~= dinoState.char then
            dinoState.char = char
            dinoState.src, dinoState.lastSrc = nil, nil
            dinoState.samples, dinoState.lastFrac = {}, nil
            dinoState.trend, dinoState.hist = {}, {}
            dinoState.eta, dinoState.lastIdx = {}, nil
            dinoState.since = os.clock()
            hud.nextScan = 0
        end
        if not char then return nil end

        local hudStage, hudSecs, hudStageDesc, hudTimeDesc = hudRead()

        local usable = {}
        for _, c in ipairs(collectCandidates()) do
            if type(c.value) == "number" or stageFromString(c.value) then usable[#usable + 1] = c end
        end
        local pick = nil
        if #usable > 0 then
            pick = usable[1]
            if dinoState.src then
                for _, c in ipairs(usable) do
                    if c.path == dinoState.src then pick = c; break end
                end
            end
        else
            pick = trendPick()
        end

        if not pick and not hudStage then return nil end

        local info = {hudStage = hudStage, hudSecs = hudSecs}

        if pick then
            dinoState.src = pick.path
            if dinoState.lastSrc ~= pick.path then
                dinoState.lastSrc = pick.path
                dinoState.samples, dinoState.lastFrac = {}, nil
            end
            info.frac, info.idx, info.countdown = interpret(pick)
            info.source = pick.path
            info.raw = pick.name .. " = " .. tostring(pick.value)
        else
            info.source = "HUD " .. tostring(hudStageDesc)
            info.raw = "shown on screen"
        end
        if hudSecs then info.source = info.source .. " | timer: " .. tostring(hudTimeDesc) end
        return info
    end

    do
        ------------------------------------------------------------------
        -- MAIN PAGE: PROFILE
        ------------------------------------------------------------------
        local prof = createGroupbox(Pages.Main.Left, "Roblox Profile")

        local profCard = Instance.new("Frame")
        profCard.Size = UDim2.new(1, 0, 0, 64)
        profCard.BackgroundTransparency = 1
        profCard.Parent = prof
        setOrder(profCard, prof)

        local avatar = Instance.new("ImageLabel")
        avatar.Size = UDim2.fromOffset(60, 60)
        avatar.BorderSizePixel = 0
        avatar.Image = "rbxthumb://type=AvatarHeadShot&id=" .. player.UserId .. "&w=150&h=150"
        avatar.Parent = profCard
        skin(avatar, "BackgroundColor3", "Control")
        addStroke(avatar, "Accent", 1)

        local anonMark = Instance.new("TextLabel")
        anonMark.Size = UDim2.new(1, 0, 1, 0)
        anonMark.BackgroundTransparency = 1
        anonMark.Text = "gg/"
        anonMark.TextSize = 20
        anonMark.Font = FONT
        anonMark.Visible = false
        anonMark.Parent = avatar
        skin(anonMark, "TextColor3", "Accent")

        local function cardLabel(y, size, key)
            local l = Instance.new("TextLabel")
            l.Size = UDim2.new(1, -70, 0, 18)
            l.Position = UDim2.fromOffset(68, y)
            l.BackgroundTransparency = 1
            l.TextSize = size
            l.Font = FONT
            l.TextXAlignment = Enum.TextXAlignment.Left
            l.TextTruncate = Enum.TextTruncate.AtEnd
            l.Parent = profCard
            skin(l, "TextColor3", key)
            return l
        end
        local nameL = cardLabel(4, 16, "Text")
        local userL = cardLabel(24, 13, "Dim")
        local ageL = cardLabel(42, 13, "Dim")
        H.rankL = createLabel(prof, "Rank: -", 18, true)

        createCheckbox(prof, "Anonymous Mode (streaming)", false, function(s)
            S.anon = s
            if H.coreAnon then H.coreAnon(s) end
            if not s and H.anonRestore then H.anonRestore() end
        end, "anonMode")
        createLabel(prof, "Also hides your name in Roblox's menu, player list and chat.", 30)

        local hpBar = createBar(prof, 16)
        local fossilL = createLabel(prof, "Fossils: -", 18, true)
        fossilL.TextSize = 15
        local infoLines = {}
        for i = 1, 7 do infoLines[i] = createLabel(prof, "", 16, true) end

        ------------------------------------------------------------------
        -- MAIN PAGE: INFO
        ------------------------------------------------------------------
        local info = createGroupbox(Pages.Main.Left, "Info")
        createLabel(info, "Open and close the menu with the menu key (default K, change it in Settings). Drag the title bar to move the window.", 62)

        ------------------------------------------------------------------
        -- AUTOFARM PAGE
        ------------------------------------------------------------------
        local zoneBox = createGroupbox(Pages.Autofarm.Left, "Territory Autofarm")
        zoneRow = createCheckbox(zoneBox, "Auto Zone", false, function(s)
            S.autoZone = s
            if not s and zoneRow then zoneRow.Label.Text = "Auto Zone" end
        end, "autoZone")
        createLabel(zoneBox, "Farms every territory from under the map (nobody can see or hit you). If a capture does not start it moves you up by itself. Use the slider to go higher or lower.", 58)
        createSlider(zoneBox, "Undermap Depth (move up / down)", 3, 120, 15, "", function(v) S.zoneDepth = v end, "zoneDepth")
        createSlider(zoneBox, "Max Wait Per Territory (s)", 20, 300, 20, "", function(v) S.zoneWait = v end, "zoneWait")
        createSlider(zoneBox, "Enemy Check Radius (0 = auto)", 0, 200, 0, "", function(v) S.zoneEnemyRadius = v end, "zoneEnemyRadius")
        createCheckbox(zoneBox, "Defend My Territories", false, function(s) S.zoneDefend = s end, "zoneDefend")
        H.zoneState = createLabel(zoneBox, "Territory: idle", 30, true)
        local zoneCount = createLabel(zoneBox, "Territories owned: -", 16, true)
        local zoneBar = createBar(zoneBox, 16)

        local survive = createGroupbox(Pages.Autofarm.Left, "Survival")
        createCheckbox(survive, "Auto Eat", false, function(s) S.autoEat = s end, "autoEat")
        createLabel(survive, "Presses E really fast while you are hungry and eats the closest food in range, even when the Eat prompt is not showing. Works for herbivores and carnivores.", 58)
        createSlider(survive, "Eat Speed (E presses / sec)", 5, 40, 25, "", function(v) S.eatRate = v end, "eatRate")
        createSlider(survive, "Eat Range (studs)", 10, 150, 40, "", function(v) S.eatRange = v end, "eatRange")
        createSlider(survive, "Eat When Hunger Below", 5, 100, S.eatBelow, "%", function(v) S.eatBelow = v end, "eatBelow")
        createCheckbox(survive, "Only When An E Prompt Shows", false, function(s) S.eatPromptOnly = s end, "eatPromptOnly")
        createCheckbox(survive, "Auto Drink", false, function(s) S.autoDrink = s end, "autoDrink")
        H.foodStatus = createLabel(survive, "Food: idle", 30, true)

        local fishBox = createGroupbox(Pages.Autofarm.Right, "Fishing")
        createCheckbox(fishBox, "Auto Fishing (Start Fishing)", false, function(s) S.autoFishing = s end, "autoFishing")
        createCheckbox(fishBox, "Go To Fishing Spot If Far", false, function(s) S.fishGoTo = s end, "fishGoTo")
        createLabel(fishBox, "Presses Start Fishing for you, plays the fishing minigame, then starts again.", 30)
        createCheckbox(fishBox, "Auto Catch Fish (quests)", false, function(s) S.autoFish = s end, "autoFish")
        createCheckbox(fishBox, "Farm Fish Continuously", false, function(s) S.fishFarm = s end, "fishFarm")
        createCheckbox(fishBox, "Auto Complete Minigame", false, function(s) S.autoMinigame = s end, "autoMinigame")
        createCheckbox(fishBox, "Replay At Real Speed", false, function(s) S.fishRealSpeed = s end, "fishRealSpeed")
        createSlider(fishBox, "Delay Between Catches (x0.1s)", 0, 100, 10, "", function(v) S.fishDelay = v / 10 end, "fishDelay")
        H.fishRecBtn = createButton(fishBox, "Record One Catch")
        H.fishClearBtn = createButton(fishBox, "Clear Recording")
        H.fishRecL = createLabel(fishBox, "Recording: none. Press Record One Catch, catch one fish by hand, then press Stop.", 44)
        H.questStatus = createLabel(fishBox, "Fish quest: -", 30, true)
        H.fishStatus = createLabel(fishBox, "Fish: idle", 30, true)

        local safeBox = createGroupbox(Pages.Autofarm.Right, "Low HP Safezone")
        createCheckbox(safeBox, "Enable Safezone", false, function(s)
            S.safeEnabled = s
            if not s then S.lowHpActive = false end
        end, "safeEnabled")
        createSlider(safeBox, "Teleport At HP", 1, 100, S.hpThreshold, "%", function(v) S.hpThreshold = v end, "hpThreshold")
        H.setSafeButton = createButton(safeBox, "Set Safezone Here")
        H.tpSafeButton = createButton(safeBox, "Teleport To Safezone")

        ------------------------------------------------------------------
        -- LIVE UPDATE LOOPS
        ------------------------------------------------------------------
        local fps = 60
        connect(RunService.RenderStepped, function(dt)
            fps = fps + (1 / math.max(dt, 1e-3) - fps) * 0.1
        end)

        task.spawn(function()
            local placeName = "..."
            pcall(function() placeName = MarketplaceService:GetProductInfo(game.PlaceId).Name end)

            while running do
                task.wait(0.5)
                pcall(function()
                    -- profile card
                    local anon = S.anon
                    avatar.ImageTransparency = anon and 1 or 0
                    anonMark.Visible = anon
                    nameL.Text = shownName()
                    userL.Text = shownUser()
                    ageL.Text = anon and "" or ("Account age: " .. player.AccountAge .. " days")

                    local hum, root = getHumanoid(player), getRoot(player)
                    if hum then
                        if anon then
                            if savedNameDist == nil then savedNameDist = hum.DisplayDistanceType end
                            hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
                        elseif savedNameDist ~= nil then
                            hum.DisplayDistanceType = savedNameDist
                            savedNameDist = nil
                        end
                        local hp, mh = hum.Health, math.max(hum.MaxHealth, 1)
                        if H.myHealth then
                            local h2, m2 = H.myHealth()
                            if h2 then hp, mh = h2, m2 end
                        end
                        hpBar.Set(hp / mh, string.format("HP %d / %d", math.floor(hp + 0.5), math.floor(mh + 0.5)))
                    else
                        hpBar.Set(0, "No character")
                    end

                    local ping = 0
                    pcall(function() ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue() + 0.5) end)

                    infoLines[1].Text = "User ID: " .. (anon and "hidden" or tostring(player.UserId))
                    infoLines[2].Text = "Game: " .. placeName
                    infoLines[3].Text = string.format("Players: %d / %d", #Players:GetPlayers(), Players.MaxPlayers)
                    infoLines[4].Text = string.format("FPS: %d   Ping: %d ms", math.floor(fps + 0.5), ping)
                    infoLines[5].Text = root and string.format("Position: %d, %d, %d",
                        math.floor(root.Position.X), math.floor(root.Position.Y), math.floor(root.Position.Z)) or "Position: -"
                    infoLines[6].Text = "Session: " .. fmtTime(os.clock() - sessionStart)
                    infoLines[7].Text = "Server: " .. (anon and "hidden" or string.sub(game.JobId, 1, 8))

                    if H.fossilDisplay then
                        fossilL.Text = "Fossils: " .. H.fossilDisplay
                    else
                        local fv, fapprox = findFossils()
                        if fv then
                            fossilL.Text = "Fossils: " .. (fapprox and "~" or "") .. formatNumber(fv)
                        elseif not fossilState.pick then
                            fossilL.Text = "Fossils: searching..."
                        end
                    end

                end)
            end
        end)

        -- territory counter
        task.spawn(function()
            while running do
                task.wait(1)
                pcall(function()
                    local owned = 0
                    for _, z in ipairs(ZonesList) do
                        if IsThisZoneGreen(z.Position) then owned = owned + 1 end
                    end
                    zoneCount.Text = string.format("Territories owned: %d / %d", owned, #ZonesList)
                    zoneBar.Set(owned / #ZonesList, owned .. " / " .. #ZonesList)
                end)
            end
        end)
    end
end

local espFolder = Instance.new("Folder")
espFolder.Name = "StrawberryESP"
espFolder.Parent = playerGui

do
    local vis = Pages.Visuals

    local espTabs = createTabbox(vis.Left, {"Player ESP", "ESP Settings"})
    local pe, ps = espTabs[1], espTabs[2]

    createCheckbox(pe, "Enable Player ESP", false, function(s) S.espOn = s end, "espOn")
    local r

    r = createCheckbox(pe, "Nametags", false, function(s) S.espNames = s end, "espNames")
    attachColor(r.Row, S.cName, function(c) S.cName = c end, "cName", 1)

    r = createCheckbox(pe, "Display Distance", false, function(s) S.espDist = s end, "espDist")
    attachColor(r.Row, S.cDist, function(c) S.cDist = c end, "cDist", 1)


    r = createCheckbox(pe, "Health Text", false, function(s) S.espHealth = s end, "espHealth")
    attachColor(r.Row, S.cHealth, function(c) S.cHealth = c end, "cHealth", 1)

    r = createCheckbox(pe, "Chams", false, function(s) S.espChams = s end, "espChams")
    attachColor(r.Row, S.cChamsOut, function(c) S.cChamsOut = c end, "cChamsOut", 1)
    attachColor(r.Row, S.cChamsFill, function(c) S.cChamsFill = c end, "cChamsFill", 2)

    createSlider(pe, "Chams Transparency", 0, 100, 50, "%", function(v) S.chamsAlpha = v / 100 end, "chamsAlpha")

    createSlider(ps, "Max Distance", 100, 5000, S.espMax, "", function(v) S.espMax = v end, "espMax")
    createSlider(ps, "Text Size", 10, 20, S.espTextSize, "px", function(v) S.espTextSize = v end, "espTextSize")

    local zb = createGroupbox(vis.Left, "Zone ESP")
    createCheckbox(zb, "Enable Zone ESP", false, function(s) S.zoneEsp = s end, "zoneEsp")
    createCheckbox(zb, "Show Distance", false, function(s) S.zoneDist = s end, "zoneDist")
    createCheckbox(zb, "Show Capture Status", false, function(s) S.zoneStatus = s end, "zoneStatus")
    createColorRow(zb, "Your Zone Color", S.zOwned, function(c) S.zOwned = c end, "zOwned")
    createColorRow(zb, "Enemy Zone Color", S.zEnemy, function(c) S.zEnemy = c end, "zEnemy")

    local wTabs = createTabbox(vis.Right, {"World", "Camera"})
    local wt, ct = wTabs[1], wTabs[2]

    H.mapToggle = createCheckbox(wt, "Transparent Map", false, function(s)
        S.mapOn = s
        if H.mapSet then H.mapSet(s) end
    end, "mapOn")
    createSlider(wt, "Map Transparency", 10, 95, 70, "%", function(v)
        S.mapAlpha = v / 100
        if H.mapRefresh then H.mapRefresh() end
    end, "mapAlpha")
    createCheckbox(wt, "Fullbright", false, function(s)
        S.fullbright = s
        if H.lightRefresh then H.lightRefresh() end
    end, "fullbright")
    createCheckbox(wt, "No Fog", false, function(s)
        S.noFog = s
        if H.lightRefresh then H.lightRefresh() end
    end, "noFog")

    local savedMin, savedMax, savedMode = player.CameraMinZoomDistance, player.CameraMaxZoomDistance, player.CameraMode
    createCheckbox(ct, "Third Person View", false, function(s)
        S.thirdPerson = s
        if s then
            savedMin, savedMax, savedMode = player.CameraMinZoomDistance, player.CameraMaxZoomDistance, player.CameraMode
            player.CameraMode = Enum.CameraMode.Classic
            player.CameraMaxZoomDistance = math.max(savedMax, 30)
            player.CameraMinZoomDistance = 12
        else
            player.CameraMode = savedMode
            player.CameraMinZoomDistance = savedMin
            player.CameraMaxZoomDistance = savedMax
        end
    end, "thirdPerson")
    createLabel(ct, "Scroll the mouse wheel to zoom while third person is on.", 30)
    createCheckbox(ct, "FOV Changer", false, function(s)
        S.fovEnabled = s
        if not s then
            local cam = workspace.CurrentCamera
            if cam then cam.FieldOfView = 70 end
        end
    end, "fovEnabled")
    createSlider(ct, "Field Of View", 30, 120, S.fovValue, "", function(v) S.fovValue = v end, "fovValue")

    H.camRestore = function()
        if S.thirdPerson then
            player.CameraMode = savedMode
            player.CameraMinZoomDistance = savedMin
            player.CameraMaxZoomDistance = savedMax
        end
    end
end

connect(RunService.RenderStepped, function()
    local cam = workspace.CurrentCamera
    if cam and S.fovEnabled then cam.FieldOfView = S.fovValue end

    if S.spectate then
        local hum = getHumanoid(S.spectate)
        if cam and hum and cam.CameraSubject ~= hum then cam.CameraSubject = hum end
    end
end)

task.spawn(function()
    while running do
        task.wait(0.5)
        if S.thirdPerson then
            pcall(function()
                if player.CameraMode ~= Enum.CameraMode.Classic then player.CameraMode = Enum.CameraMode.Classic end
                if player.CameraMinZoomDistance < 12 then player.CameraMinZoomDistance = 12 end
                if player.CameraMaxZoomDistance < 30 then player.CameraMaxZoomDistance = 30 end
            end)
        end
    end
end)

----------------------------------------------------------------------
-- PLAYER ESP
-- ESP objects are made once per player and re-pointed at that player's current
-- dino on every update, so respawns, dino swaps, map streaming and other features
-- can not break them. Each player updates on its own, so one error never hides the rest.
----------------------------------------------------------------------
do
    local espObjs = {}
    local MAX_CHAMS = 30   -- Roblox only draws 31 highlights at once

    local function espParent()
        if not espFolder or not espFolder.Parent then
            espFolder = Instance.new("Folder")
            espFolder.Name = "StrawberryESP"
            espFolder.Parent = playerGui
        end
        return espFolder
    end

    local function mk(class, props, parent)
        local i = Instance.new(class)
        for k, v in pairs(props) do i[k] = v end
        i.Parent = parent
        return i
    end

    local function destroyEsp(plr)
        local o = espObjs[plr]
        if o then
            for _, i in ipairs(o.insts) do pcall(function() i:Destroy() end) end
            espObjs[plr] = nil
        end
    end

    local function buildEsp(plr)
        local parent = espParent()
        local o = {plr = plr}
        o.tagGui = mk("BillboardGui", {Name = "Tag", AlwaysOnTop = true, Size = UDim2.fromOffset(240, 54),
            ResetOnSpawn = false, ClipsDescendants = false, Enabled = false, LightInfluence = 0}, parent)
        o.nameL = mk("TextLabel", {Size = UDim2.new(1, 0, 0, 16), BackgroundTransparency = 1, Font = FONT,
            TextStrokeTransparency = 0, Text = H.nameOf(plr)}, o.tagGui)
        o.hpL = mk("TextLabel", {Size = UDim2.new(1, 0, 0, 16), BackgroundTransparency = 1, Font = FONT,
            TextStrokeTransparency = 0, Text = "", RichText = true}, o.tagGui)
        o.distL = mk("TextLabel", {Size = UDim2.new(1, 0, 0, 14), BackgroundTransparency = 1, Font = FONT,
            TextStrokeTransparency = 0, Text = ""}, o.tagGui)
        o.hl = mk("Highlight", {DepthMode = Enum.HighlightDepthMode.AlwaysOnTop, OutlineTransparency = 0,
            Enabled = false}, parent)
        o.insts = {o.tagGui, o.hl}
        espObjs[plr] = o
        return o
    end

    local function intact(o)
        for _, i in ipairs(o.insts) do
            if not i.Parent then return false end
        end
        return o.nameL.Parent == o.tagGui and o.hpL.Parent == o.tagGui and o.distL.Parent == o.tagGui
    end

    -- finds the part to stick the tag on, works for dinos with or without a Humanoid
    local function resolveRoot(o, char)
        local r = o.root
        if r and r.Parent and r:IsDescendantOf(char) then return r end
        r = char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart or char:FindFirstChild("Torso")
            or char:FindFirstChild("Head") or char:FindFirstChildWhichIsA("BasePart", true)
        o.root = r
        o.bbT = 0
        return r
    end

    local HP_NAMES = {"Health", "HP", "CurrentHealth", "DinoHealth"}
    local MAX_NAMES = {"MaxHealth", "MaxHP", "HealthMax", "MaximumHealth"}

    local function readNum(holder, names)
        for _, n in ipairs(names) do
            local a = holder:GetAttribute(n)
            if type(a) == "number" then return a end
            local v = holder:FindFirstChild(n)
            if v and (v:IsA("NumberValue") or v:IsA("IntValue")) then return v.Value end
        end
        return nil
    end

    -- current, max (the game's own health value first, then the Humanoid). nil if unknown.
    local function getHealth(char, root, hum, plr)
        for _, holder in ipairs({char, root, plr}) do
            if holder then
                local cur = readNum(holder, HP_NAMES)
                if cur then
                    local mx = readNum(holder, MAX_NAMES) or readNum(char, MAX_NAMES)
                    if not mx or mx <= 0 then mx = math.max(cur, 1) end
                    return cur, mx
                end
            end
        end
        if hum then return hum.Health, math.max(hum.MaxHealth, 1) end
        return nil, nil
    end

    local RED, YELLOW = Color3.fromRGB(255, 60, 60), Color3.fromRGB(255, 210, 60)
    local function hpColor(pct)
        if pct >= 0.5 then return YELLOW:Lerp(S.cHealth, (pct - 0.5) * 2) end
        return RED:Lerp(YELLOW, pct * 2)
    end

    local function shortNum(n)
        n = math.floor(n + 0.5)
        if n >= 1e6 then return string.format("%.1fM", n / 1e6) end
        if n >= 1e4 then return string.format("%.1fK", n / 1e3) end
        return tostring(n)
    end

    local function hide(o)
        if o.tagGui.Enabled then o.tagGui.Enabled = false end
        if o.hl.Enabled then o.hl.Enabled = false end
    end

    connect(Players.PlayerRemoving, function(plr) destroyEsp(plr) end)

    local function updateOne(plr, myRoot, chamList)
        local o = espObjs[plr]
        if o and not intact(o) then destroyEsp(plr); o = nil end
        if not o then o = buildEsp(plr) end

        local char = plr.Character
        if not char or not char.Parent then hide(o); return end
        local root = resolveRoot(o, char)
        if not root then hide(o); return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hp, maxHp = getHealth(char, root, hum, plr)
        if hp and hp <= 0 then hide(o); return end

        local dist = myRoot and (myRoot.Position - root.Position).Magnitude or 0
        if dist > S.espMax then hide(o); return end

        -- bounding box is slow, refresh it once a second
        local now = os.clock()
        if not o.bbT or now - o.bbT > 1 then
            o.bbT = now
            local ok, cf, size = pcall(function() return char:GetBoundingBox() end)
            if ok and cf then
                o.top = (cf.Position - root.Position) + Vector3.new(0, size.Y / 2 + 1.4, 0)
            else
                o.top = Vector3.new(0, 4, 0)
            end
        end

        if o.tagGui.Adornee ~= root then o.tagGui.Adornee = root end
        o.tagGui.StudsOffsetWorldSpace = o.top or Vector3.new(0, 4, 0)

        local ts = S.espTextSize
        local y = 0
        o.nameL.Visible = S.espNames
        if S.espNames then
            o.nameL.Text = H.nameOf(plr)
            o.nameL.TextColor3 = S.cName
            o.nameL.TextSize = ts
            o.nameL.Size = UDim2.new(1, 0, 0, ts + 3)
            o.nameL.Position = UDim2.fromOffset(0, y)
            y = y + ts + 3
        end

        o.hpL.Visible = S.espHealth
        if S.espHealth then
            if hp then
                local pct = math.clamp(hp / math.max(maxHp, 1), 0, 1)
                o.hpL.TextColor3 = hpColor(pct)
                o.hpL.Text = string.format("HP %s / %s  <font transparency=\"0.25\">(%d%%)</font>",
                    shortNum(hp), shortNum(maxHp), math.floor(pct * 100 + 0.5))
            else
                o.hpL.TextColor3 = S.cHealth
                o.hpL.Text = "HP ?"
            end
            o.hpL.TextSize = ts
            o.hpL.Size = UDim2.new(1, 0, 0, ts + 3)
            o.hpL.Position = UDim2.fromOffset(0, y)
            y = y + ts + 3
        end

        o.distL.Visible = S.espDist
        if S.espDist then
            o.distL.TextColor3 = S.cDist
            o.distL.TextSize = ts - 1
            o.distL.Size = UDim2.new(1, 0, 0, ts + 2)
            o.distL.Position = UDim2.fromOffset(0, y)
            o.distL.Text = "[" .. math.floor(dist) .. " studs]"
            y = y + ts + 2
        end
        o.tagGui.Size = UDim2.fromOffset(240, math.max(y, 1))
        o.tagGui.Enabled = y > 0

        if S.espChams then
            chamList[#chamList + 1] = {o = o, d = dist, char = char}
        elseif o.hl.Enabled then
            o.hl.Enabled = false
        end
    end

    task.spawn(function()
        while running do
            task.wait(0.1)
            if not S.espOn then
                for _, o in pairs(espObjs) do pcall(hide, o) end
                task.wait(0.25)
                continue
            end
            local myRoot = getRoot(player)
            local chamList = {}
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= player then
                    local ok = pcall(updateOne, plr, myRoot, chamList)
                    if not ok then
                        local o = espObjs[plr]
                        if o then pcall(hide, o) end
                    end
                end
            end
            -- chams: only the closest players get a highlight (Roblox limit)
            pcall(function()
                table.sort(chamList, function(x, y) return x.d < y.d end)
                for i, e in ipairs(chamList) do
                    local o = e.o
                    if i <= MAX_CHAMS - ((S.meatEsp and S.meatChams) or 0) then
                        if o.hl.Adornee ~= e.char then o.hl.Adornee = e.char end
                        o.hl.FillColor = S.cChamsFill
                        o.hl.OutlineColor = S.cChamsOut
                        o.hl.FillTransparency = S.chamsAlpha
                        o.hl.Enabled = true
                    elseif o.hl.Enabled then
                        o.hl.Enabled = false
                    end
                end
            end)
        end
    end)

    H.espCleanup = function()
        for plr in pairs(espObjs) do destroyEsp(plr) end
    end
end

do
    local zoneObjs = {}

    for i, z in ipairs(ZonesList) do
        local part = Instance.new("Part")
        part.Name = "StrawberryZoneAnchor"
        part.Anchored = true
        part.CanCollide = false
        part.CanTouch = false
        part.CanQuery = false
        part.Transparency = 1
        part.Size = Vector3.new(1, 1, 1)
        part.Position = z.Teleport.Position
        part.Parent = workspace

        local bb = Instance.new("BillboardGui")
        bb.AlwaysOnTop = true
        bb.Size = UDim2.fromOffset(180, 34)
        bb.ResetOnSpawn = false
        bb.Enabled = false
        bb.Adornee = part
        bb.Parent = espFolder

        local function lbl(y, h)
            local l = Instance.new("TextLabel")
            l.Size = UDim2.new(1, 0, 0, h)
            l.Position = UDim2.fromOffset(0, y)
            l.BackgroundTransparency = 1
            l.Font = FONT
            l.TextSize = 14
            l.TextStrokeTransparency = 0
            l.Parent = bb
            return l
        end
        local nameL = lbl(0, 18)
        nameL.Text = "Zone " .. i
        local subL = lbl(18, 14)
        subL.TextSize = 12

        zoneObjs[i] = {part = part, bb = bb, nameL = nameL, subL = subL}
    end

    task.spawn(function()
        while running do
            task.wait(0.25)
            pcall(function()
                local myRoot = getRoot(player)
                for i, o in ipairs(zoneObjs) do
                    o.bb.Enabled = S.zoneEsp
                    if S.zoneEsp then
                        local owned = IsThisZoneGreen(ZonesList[i].Position)
                        local col = owned and S.zOwned or S.zEnemy
                        o.nameL.TextColor3 = col
                        o.subL.TextColor3 = col
                        local parts = {}
                        if S.zoneStatus then parts[#parts + 1] = owned and "Yours" or "Not captured" end
                        if S.zoneDist and myRoot then
                            parts[#parts + 1] = math.floor((myRoot.Position - ZonesList[i].Position).Magnitude) .. " studs"
                        end
                        o.subL.Text = table.concat(parts, "  |  ")
                    end
                end
            end)
        end
    end)

    H.zoneCleanup = function()
        for _, o in ipairs(zoneObjs) do
            pcall(function() o.part:Destroy() end)
        end
    end
end

do
    local mapOrig = setmetatable({}, {__mode = "k"})
    local mapSetTo = setmetatable({}, {__mode = "k"})
    local addedConn = nil
    local scanId = 0
    local waterSaved = nil

    local function isCharPart(inst)
        local p = inst.Parent
        while p and p ~= workspace do
            if p:IsA("Model") and p:FindFirstChildOfClass("Humanoid") then return true end
            p = p.Parent
        end
        return false
    end

    local function handle(inst)
        if inst.Name == "StrawberryZoneAnchor" or inst.Name == "StrawberryHitboxVis" then return end
        if not (inst:IsA("BasePart") or inst:IsA("Decal")) then return end
        if inst:IsA("Terrain") then return end
        if isCharPart(inst) then return end
        local cur = inst.Transparency
        if mapOrig[inst] == nil or mapSetTo[inst] ~= cur then
            mapOrig[inst] = cur
        end
        local v = math.max(mapOrig[inst], S.mapAlpha)
        if cur ~= v then inst.Transparency = v end
        mapSetTo[inst] = v
    end

    local function applyWater()
        local t = workspace:FindFirstChildOfClass("Terrain")
        if not t then return end
        if waterSaved == nil then waterSaved = t.WaterTransparency end
        local v = math.max(waterSaved, S.mapAlpha)
        if t.WaterTransparency ~= v then t.WaterTransparency = v end
    end

    local function restore()
        for inst, orig in pairs(mapOrig) do
            pcall(function() inst.Transparency = orig end)
            mapOrig[inst] = nil
            mapSetTo[inst] = nil
        end
        if waterSaved ~= nil then
            pcall(function() workspace.Terrain.WaterTransparency = waterSaved end)
            waterSaved = nil
        end
    end

    local function sweep(id)
        local n = 0
        for inst in pairs(mapOrig) do
            if scanId ~= id or not running then return end
            if inst.Parent then pcall(handle, inst) end
            n = n + 1
            if n % 2000 == 0 then task.wait() end
        end
        pcall(applyWater)
    end

    H.mapSet = function(on)
        scanId = scanId + 1
        local id = scanId
        if addedConn then addedConn:Disconnect(); addedConn = nil end
        if on then
            task.spawn(function()
                local n = 0
                for _, inst in ipairs(workspace:GetDescendants()) do
                    if scanId ~= id or not running then return end
                    pcall(handle, inst)
                    n = n + 1
                    if n % 1500 == 0 then task.wait() end
                end
                pcall(applyWater)
                while scanId == id and running do
                    task.wait(1)
                    sweep(id)
                end
            end)
            addedConn = workspace.DescendantAdded:Connect(function(inst)
                task.defer(function()
                    if S.mapOn then pcall(handle, inst) end
                end)
            end)
        else
            restore()
        end
    end

    H.mapRefresh = function()
        if not S.mapOn then return end
        local id = scanId
        task.spawn(sweep, id)
    end

    H.mapCleanup = function()
        scanId = scanId + 1
        if addedConn then addedConn:Disconnect(); addedConn = nil end
        restore()
    end

    local lightSaved = nil
    local FOG_END, FOG_START = 1e9, 9e8
    local fogGame = {FogEnd = Lighting.FogEnd, FogStart = Lighting.FogStart}
    local atmSaved = setmetatable({}, {__mode = "k"})
    local atmHooked = setmetatable({}, {__mode = "k"})

    local function saveLight()
        if lightSaved then return end
        lightSaved = {
            Brightness = Lighting.Brightness, ClockTime = Lighting.ClockTime,
            GlobalShadows = Lighting.GlobalShadows,
            Ambient = Lighting.Ambient, OutdoorAmbient = Lighting.OutdoorAmbient,
        }
    end

    local function restoreLight()
        if not lightSaved then return end
        for k, v in pairs(lightSaved) do
            pcall(function() Lighting[k] = v end)
        end
        lightSaved = nil
    end

    local function onFogChanged()
        if S.noFog then
            if Lighting.FogEnd ~= FOG_END then
                fogGame.FogEnd = Lighting.FogEnd
                Lighting.FogEnd = FOG_END
            end
            if Lighting.FogStart ~= FOG_START then
                fogGame.FogStart = Lighting.FogStart
                Lighting.FogStart = FOG_START
            end
        else
            fogGame.FogEnd = Lighting.FogEnd
            fogGame.FogStart = Lighting.FogStart
        end
    end

    local function onAtmChanged(atm)
        local d = atmSaved[atm]
        if not d then d = {}; atmSaved[atm] = d end
        if S.noFog then
            if atm.Density ~= 0 then d.Density = atm.Density; atm.Density = 0 end
            if atm.Haze ~= 0 then d.Haze = atm.Haze; atm.Haze = 0 end
        else
            d.Density, d.Haze = atm.Density, atm.Haze
        end
    end

    local function hookAtm(atm)
        if not atm:IsA("Atmosphere") or atmHooked[atm] then return end
        atmHooked[atm] = true
        atmSaved[atm] = {Density = atm.Density, Haze = atm.Haze}
        connect(atm:GetPropertyChangedSignal("Density"), function() onAtmChanged(atm) end)
        connect(atm:GetPropertyChangedSignal("Haze"), function() onAtmChanged(atm) end)
        if S.noFog then onAtmChanged(atm) end
    end

    for _, d in ipairs(Lighting:GetDescendants()) do hookAtm(d) end
    connect(Lighting.DescendantAdded, hookAtm)
    connect(Lighting:GetPropertyChangedSignal("FogEnd"), onFogChanged)
    connect(Lighting:GetPropertyChangedSignal("FogStart"), onFogChanged)

    local function restoreFog()
        pcall(function()
            Lighting.FogEnd = fogGame.FogEnd
            Lighting.FogStart = fogGame.FogStart
        end)
        for atm, d in pairs(atmSaved) do
            pcall(function()
                if d.Density then atm.Density = d.Density end
                if d.Haze then atm.Haze = d.Haze end
            end)
        end
    end

    local function enforce()
        if S.fullbright then
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.GlobalShadows = false
            Lighting.Ambient = Color3.fromRGB(178, 178, 178)
            Lighting.OutdoorAmbient = Color3.fromRGB(178, 178, 178)
        end
        if S.noFog then
            onFogChanged()
            for atm in pairs(atmHooked) do
                if atm.Parent then onAtmChanged(atm) end
            end
        end
    end

    H.lightRefresh = function()
        if S.fullbright then saveLight() else restoreLight() end
        if S.noFog then enforce() else restoreFog() end
    end

    H.lightCleanup = function()
        S.fullbright, S.noFog = false, false
        restoreLight()
        restoreFog()
    end

    connect(RunService.RenderStepped, enforce)
    connect(RunService.Heartbeat, enforce)
    connect(RunService.Stepped, enforce)
end

do
    local page = Pages.Players
    page.Left.Size = UDim2.new(1, -8, 0, 0)
    local playersBox = createGroupbox(page.Left, "Player List")

    local spectateInfo = createLabel(playersBox, "Players in server: 0", 16, true)
    local stopSpectateButton = createButton(playersBox, "Stop Watching")

    local listFrame = Instance.new("ScrollingFrame")
    listFrame.Size = UDim2.new(1, 0, 0, 250)
    listFrame.BorderSizePixel = 0
    listFrame.ScrollBarThickness = 4
    listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    listFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    listFrame.Parent = playersBox
    setOrder(listFrame, playersBox)
    skin(listFrame, "BackgroundColor3", "Inner")
    skin(listFrame, "ScrollBarImageColor3", "Accent")
    addStroke(listFrame, "Border", 1)

    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 4)
    listLayout.SortOrder = Enum.SortOrder.Name
    listLayout.Parent = listFrame

    local listPadding = Instance.new("UIPadding")
    listPadding.PaddingTop = UDim.new(0, 4)
    listPadding.PaddingBottom = UDim.new(0, 4)
    listPadding.PaddingLeft = UDim.new(0, 4)
    listPadding.PaddingRight = UDim.new(0, 8)
    listPadding.Parent = listFrame

    local playerRows = {}

    local function refreshCount()
        if S.spectate then
            spectateInfo.Text = "Watching: " .. H.nameOf(S.spectate)
        else
            spectateInfo.Text = "Players in server: " .. #Players:GetPlayers()
        end
    end

    local function stopSpectating()
        S.spectate = nil
        local cam = workspace.CurrentCamera
        local hum = getHumanoid(player)
        if cam and hum then cam.CameraSubject = hum end
        for _, d in pairs(playerRows) do
            d.Watch.Text = "Watch"
            setButtonActive(d.Watch, false)
        end
        refreshCount()
    end
    H.stopSpectating = stopSpectating

    local function startSpectating(plr)
        local hum = getHumanoid(plr)
        local cam = workspace.CurrentCamera
        if not (hum and cam) then return end
        S.spectate = plr
        cam.CameraSubject = hum
        for p, d in pairs(playerRows) do
            local a = p == plr
            d.Watch.Text = a and "Stop" or "Watch"
            setButtonActive(d.Watch, a)
        end
        refreshCount()
    end

    stopSpectateButton.Activated:Connect(stopSpectating)

    local function addRow(plr)
        if plr == player or playerRows[plr] then return end

        local rowFrame = Instance.new("Frame")
        rowFrame.Name = string.lower(plr.Name)
        rowFrame.Size = UDim2.new(1, 0, 0, 28)
        rowFrame.BorderSizePixel = 0
        rowFrame.Parent = listFrame
        skin(rowFrame, "BackgroundColor3", "Group")
        addStroke(rowFrame, "Border", 1)

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, -130, 1, 0)
        nameLabel.Position = UDim2.fromOffset(8, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = (plr.UserId == H.OWNER_ID and player.UserId ~= H.OWNER_ID) and H.nameOf(plr)
            or (plr.DisplayName .. "  (@" .. plr.Name .. ")")
        nameLabel.TextSize = 13
        nameLabel.Font = FONT
        nameLabel.TextXAlignment = Enum.TextXAlignment.Left
        nameLabel.TextTruncate = Enum.TextTruncate.AtEnd
        nameLabel.Parent = rowFrame
        skin(nameLabel, "TextColor3", "Text")

        local tp = Instance.new("TextButton")
        tp.Size = UDim2.fromOffset(46, 20)
        tp.Position = UDim2.new(1, -112, 0.5, -10)
        tp.Text = "TP"
        tp.Parent = rowFrame
        styleButton(tp)

        local watch = Instance.new("TextButton")
        watch.Size = UDim2.fromOffset(56, 20)
        watch.Position = UDim2.new(1, -62, 0.5, -10)
        watch.Text = "Watch"
        watch.Parent = rowFrame
        styleButton(watch)

        tp.Activated:Connect(function()
            local mine, theirs = getRoot(player), getRoot(plr)
            if mine and theirs then
                mine.CFrame = theirs.CFrame * CFrame.new(0, 0, 3)
                mine.AssemblyLinearVelocity = Vector3.new()
            end
        end)
        watch.Activated:Connect(function()
            if S.spectate == plr then stopSpectating() else startSpectating(plr) end
        end)

        playerRows[plr] = {Frame = rowFrame, Watch = watch}
        refreshCount()
    end

    local function removeRow(plr)
        if S.spectate == plr then stopSpectating() end
        local d = playerRows[plr]
        if d then d.Frame:Destroy(); playerRows[plr] = nil end
        refreshCount()
    end

    for _, plr in ipairs(Players:GetPlayers()) do addRow(plr) end
    refreshCount()
    connect(Players.PlayerAdded, addRow)
    connect(Players.PlayerRemoving, removeRow)
end

local miscToggles = {}
do
    local misc = Pages.Miscellaneous

    local moveBox = createGroupbox(misc.Left, "Movement")

    miscToggles.speed = createCheckbox(moveBox, "Speed Changer", false, function(s)
        S.speedEnabled = s
        local hum = getHumanoid(player)
        if hum then
            if s then S.savedWalkSpeed = hum.WalkSpeed else hum.WalkSpeed = S.savedWalkSpeed end
        end
    end, "speedEnabled")
    createSlider(moveBox, "Walk Speed", 16, 300, S.speedValue, "", function(v) S.speedValue = v end, "speedValue")

    miscToggles.fly = createCheckbox(moveBox, "Fly", false, function(s) S.flyEnabled = s end, "flyEnabled")
    createSlider(moveBox, "Fly Speed", 10, 300, S.flySpeed, "", function(v) S.flySpeed = v end, "flySpeed")
    createLabel(moveBox, "W A S D to move toward where the camera looks. SPACE = up, LEFT CTRL or Q = down.", 44)

    miscToggles.noclip = createCheckbox(moveBox, "Noclip", false, function(s) S.noclip = s end, "noclip")
    createCheckbox(moveBox, "Infinite Jump", false, function(s) S.infJump = s end, "infJump")

    createCheckbox(moveBox, "Jump Power Changer", false, function(s)
        S.jumpEnabled = s
        local hum = getHumanoid(player)
        if hum then
            if s then S.savedJump = hum.JumpPower else hum.JumpPower = S.savedJump end
        end
    end, "jumpEnabled")
    createSlider(moveBox, "Jump Power", 50, 300, S.jumpValue, "", function(v) S.jumpValue = v end, "jumpValue")

    local utilBox = createGroupbox(misc.Right, "Utility")
    createCheckbox(utilBox, "Anti AFK", false, function(s) S.antiAfk = s end, "antiAfk")
    createCheckbox(utilBox, "Ctrl + Click Teleport", false, function(s) S.clickTp = s end, "clickTp")

    local resetBtn = createButton(utilBox, "Reset Character")
    resetBtn.Activated:Connect(function()
        local hum = getHumanoid(player)
        if hum then hum.Health = 0 end
    end)

    local rejoinBtn = createButton(utilBox, "Rejoin Server")
    rejoinBtn.Activated:Connect(function()
        pcall(function()
            if #Players:GetPlayers() <= 1 then
                TeleportService:Teleport(game.PlaceId, player)
            else
                TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, player)
            end
        end)
    end)

    local copyBtn = createButton(utilBox, "Copy Place ID")
    copyBtn.Activated:Connect(function()
        if setclipboard then
            pcall(setclipboard, tostring(game.PlaceId))
            copyBtn.Text = "Copied!"
            task.delay(1.2, function() copyBtn.Text = "Copy Place ID" end)
        end
    end)
end

local flyBV, flyBG
local function stopFly()
    if flyBV then flyBV:Destroy(); flyBV = nil end
    if flyBG then flyBG:Destroy(); flyBG = nil end
    local hum = getHumanoid(player)
    if hum then hum.PlatformStand = false end
end

connect(RunService.Heartbeat, function()
    local hum = getHumanoid(player)
    local root = getRoot(player)

    if S.speedEnabled and hum then hum.WalkSpeed = S.speedValue end
    if S.jumpEnabled and hum then
        hum.UseJumpPower = true
        hum.JumpPower = S.jumpValue
    end

    if S.flyEnabled and hum and root and hum.Health > 0 then
        local camera = workspace.CurrentCamera

        if not flyBV or flyBV.Parent ~= root then
            if flyBV then flyBV:Destroy() end
            flyBV = Instance.new("BodyVelocity")
            flyBV.MaxForce = Vector3.new(9e9, 9e9, 9e9)
            flyBV.Velocity = Vector3.new()
            flyBV.Parent = root
        end
        if not flyBG or flyBG.Parent ~= root then
            if flyBG then flyBG:Destroy() end
            flyBG = Instance.new("BodyGyro")
            flyBG.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
            flyBG.P = 9e4
            flyBG.Parent = root
        end

        hum.PlatformStand = true

        local cf = camera.CFrame
        local look, right = cf.LookVector, cf.RightVector
        local flatLook = Vector3.new(look.X, 0, look.Z)
        local flatRight = Vector3.new(right.X, 0, right.Z)
        local move = hum.MoveDirection
        local vel = Vector3.new()

        if move.Magnitude > 0 and flatLook.Magnitude > 0 and flatRight.Magnitude > 0 then
            vel = vel + look * move:Dot(flatLook.Unit) + right * move:Dot(flatRight.Unit)
        end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then vel = vel + Vector3.new(0, 1, 0) end
        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) or UIS:IsKeyDown(Enum.KeyCode.Q) then
            vel = vel - Vector3.new(0, 1, 0)
        end

        flyBV.Velocity = vel * S.flySpeed
        flyBG.CFrame = cf
    elseif flyBV or flyBG then
        stopFly()
    end
end)

connect(RunService.Stepped, function()
    if S.noclip or S.zoneNoclip then
        local char = player.Character
        if char then
            for _, p in ipairs(char:GetDescendants()) do
                if p:IsA("BasePart") and p.CanCollide then p.CanCollide = false end
            end
        end
    end
end)

connect(UIS.JumpRequest, function()
    if S.infJump then
        local hum = getHumanoid(player)
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

do
    local mouse = player:GetMouse()
    connect(mouse.Button1Down, function()
        if S.clickTp and UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
            local root = getRoot(player)
            if root and mouse.Hit then
                root.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0, 3, 0))
            end
        end
    end)
end

local hasFS = type(writefile) == "function" and type(readfile) == "function"
    and type(isfile) == "function" and type(isfolder) == "function" and type(makefolder) == "function"

local DIR = "StrawberryHub"
local CFG_DIR = DIR .. "/configs"
local THEME_DIR = DIR .. "/themes"
local AUTOLOAD_FILE = DIR .. "/autoload.txt"
local THEME_DEFAULT_FILE = DIR .. "/default_theme.txt"

local function ensureFolders()
    if not hasFS then return end
    pcall(function()
        for _, d in ipairs({DIR, CFG_DIR, THEME_DIR}) do
            if not isfolder(d) then makefolder(d) end
        end
    end)
end

local function cleanName(name)
    name = tostring(name or ""):gsub("[^%w%-_ ]", "")
    return trim(name)
end

local function listJson(dir)
    local out = {}
    if not hasFS or type(listfiles) ~= "function" then return out end
    ensureFolders()
    local ok, files = pcall(listfiles, dir)
    if ok and type(files) == "table" then
        for _, path in ipairs(files) do
            local n = string.match(path, "([^/\\]+)%.json$")
            if n then out[#out + 1] = n end
        end
    end
    table.sort(out, function(a, b) return string.lower(a) < string.lower(b) end)
    return out
end

local function readText(path)
    if not hasFS then return nil end
    local ok, v = pcall(function() if isfile(path) then return readfile(path) end; return nil end)
    if ok and v and v ~= "" then return cleanName(v) end
    return nil
end

local function removeFile(path, fallbackBlank)
    pcall(function()
        if type(delfile) == "function" and isfile(path) then
            delfile(path)
        elseif fallbackBlank then
            writefile(path, "")
        end
    end)
end

do
    local settings = Pages.Settings

    local menuBox = createGroupbox(settings.Left, "Menu")

    createKeybind(menuBox, "Menu Toggle", "K", function() H.toggleGui() end, "kbMenu")
    createKeybind(menuBox, "Fly Toggle", "None", function() if miscToggles.fly then miscToggles.fly.Set(not miscToggles.fly.Get()) end end, "kbFly")
    createKeybind(menuBox, "Noclip Toggle", "None", function() if miscToggles.noclip then miscToggles.noclip.Set(not miscToggles.noclip.Get()) end end, "kbNoclip")
    createKeybind(menuBox, "Speed Toggle", "None", function() if miscToggles.speed then miscToggles.speed.Set(not miscToggles.speed.Get()) end end, "kbSpeed")
    createLabel(menuBox, "Click a keybind, then press a key. Backspace clears it, Escape cancels.", 30)

    createCheckbox(menuBox, "Show Watermark", false, function(s)
        S.watermark = s
        if loaded then wm.Visible = s end
    end, "watermark")

    local unloadBtn = createButton(menuBox, "Unload Script")
    unloadBtn.Activated:Connect(function() H.unload() end)

    local themeBox = createGroupbox(settings.Right, "Theme")
    local pAccent = createColorRow(themeBox, "Accent Color", Base.Accent, function(c) Base.Accent = c; applyTheme() end, "guiAccent")
    local pBg = createColorRow(themeBox, "Background Color", Base.Background, function(c) Base.Background = c; applyTheme() end, "guiBackground")
    local pOutline = createColorRow(themeBox, "Outline Color", Base.Outline, function(c) Base.Outline = c; applyTheme() end, "guiOutline")
    local pText = createColorRow(themeBox, "Font Color", Base.Text, function(c) Base.Text = c; applyTheme() end, "guiText")

    local presetBox = createGroupbox(settings.Right, "Themes")

    local themeList = Instance.new("ScrollingFrame")
    themeList.Size = UDim2.new(1, 0, 0, 130)
    themeList.BorderSizePixel = 0
    themeList.ScrollBarThickness = 4
    themeList.CanvasSize = UDim2.new(0, 0, 0, 0)
    themeList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    themeList.Parent = presetBox
    setOrder(themeList, presetBox)
    skin(themeList, "BackgroundColor3", "Inner")
    skin(themeList, "ScrollBarImageColor3", "Accent")
    addStroke(themeList, "Border", 1)
    do
        local l = Instance.new("UIListLayout")
        l.Padding = UDim.new(0, 3)
        l.SortOrder = Enum.SortOrder.LayoutOrder
        l.Parent = themeList
        local p = Instance.new("UIPadding")
        p.PaddingTop = UDim.new(0, 3); p.PaddingBottom = UDim.new(0, 3)
        p.PaddingLeft = UDim.new(0, 3); p.PaddingRight = UDim.new(0, 7)
        p.Parent = themeList
    end

    local themeNameBox = createTextbox(presetBox, "Theme Name", "name for your custom theme", "", function() end)
    local saveThemeBtn = createButton(presetBox, "Save Custom Theme")
    local deleteThemeBtn = createButton(presetBox, "Delete Selected Custom Theme")
    local defaultThemeBtn = createButton(presetBox, "Set Selected As Default Theme")
    local themeStatus = createLabel(presetBox, hasFS and "Pick a theme to apply it." or "No file functions, custom themes are unavailable.", 30)

    local selectedTheme = "Strawberry"

    local function c3t(c) return {math.floor(c.R * 255 + 0.5), math.floor(c.G * 255 + 0.5), math.floor(c.B * 255 + 0.5)} end
    local function t2c(t) return Color3.fromRGB(t[1] or 0, t[2] or 0, t[3] or 0) end

    local function allThemes()
        local out = {}
        for _, p in ipairs(Presets) do
            out[#out + 1] = {name = p[1], Accent = p[2], Background = p[3], Outline = p[4], Text = p[5], custom = false}
        end
        for _, n in ipairs(listJson(THEME_DIR)) do
            local ok, data = pcall(function() return HttpService:JSONDecode(readfile(THEME_DIR .. "/" .. n .. ".json")) end)
            if ok and type(data) == "table" and data.Accent and data.Background and data.Outline and data.Text then
                out[#out + 1] = {name = n, Accent = t2c(data.Accent), Background = t2c(data.Background),
                    Outline = t2c(data.Outline), Text = t2c(data.Text), custom = true}
            end
        end
        return out
    end

    local function applyThemeEntry(e)
        pAccent.Set(e.Accent)
        pBg.Set(e.Background)
        pOutline.Set(e.Outline)
        pText.Set(e.Text)
        selectedTheme = e.name
    end

    local refreshThemes
    refreshThemes = function()
        for _, c in ipairs(themeList:GetChildren()) do
            if c:IsA("TextButton") then c:Destroy() end
        end
        local defName = readText(THEME_DEFAULT_FILE)
        for i, e in ipairs(allThemes()) do
            local b = Instance.new("TextButton")
            b.Size = UDim2.new(1, 0, 0, 20)
            b.LayoutOrder = i
            b.Text = e.name .. (e.custom and "  [custom]" or "") .. ((e.name == defName) and "  [default]" or "")
            b.Parent = themeList
            styleButton(b)
            setButtonActive(b, e.name == selectedTheme)

            local sw = Instance.new("Frame")
            sw.Size = UDim2.fromOffset(10, 10)
            sw.AnchorPoint = Vector2.new(1, 0.5)
            sw.Position = UDim2.new(1, -6, 0.5, 0)
            sw.BackgroundColor3 = e.Accent
            sw.BorderSizePixel = 0
            sw.Parent = b
            addStroke(sw, "Black", 1)

            b.Activated:Connect(function()
                applyThemeEntry(e)
                themeNameBox.Set(e.custom and e.name or "")
                themeStatus.Text = "Applied '" .. e.name .. "'."
                refreshThemes()
            end)
        end
    end
    refreshThemes()

    saveThemeBtn.Activated:Connect(function()
        if not hasFS then themeStatus.Text = "File functions not available."; return end
        local name = cleanName(themeNameBox.Box.Text)
        if name == "" then themeStatus.Text = "Type a theme name first."; return end
        for _, p in ipairs(Presets) do
            if string.lower(p[1]) == string.lower(name) then
                themeStatus.Text = "That name is used by a built-in theme."
                return
            end
        end
        ensureFolders()
        local ok = pcall(function()
            writefile(THEME_DIR .. "/" .. name .. ".json", HttpService:JSONEncode({
                Accent = c3t(Base.Accent), Background = c3t(Base.Background),
                Outline = c3t(Base.Outline), Text = c3t(Base.Text),
            }))
        end)
        if ok then
            selectedTheme = name
            themeStatus.Text = "Saved theme '" .. name .. "'."
            refreshThemes()
        else
            themeStatus.Text = "Could not save the theme."
        end
    end)

    deleteThemeBtn.Activated:Connect(function()
        if not hasFS then return end
        for _, p in ipairs(Presets) do
            if p[1] == selectedTheme then themeStatus.Text = "Built-in themes cannot be deleted."; return end
        end
        removeFile(THEME_DIR .. "/" .. selectedTheme .. ".json", false)
        if readText(THEME_DEFAULT_FILE) == selectedTheme then removeFile(THEME_DEFAULT_FILE, true) end
        themeStatus.Text = "Deleted '" .. selectedTheme .. "'."
        selectedTheme = "Strawberry"
        refreshThemes()
    end)

    defaultThemeBtn.Activated:Connect(function()
        if not hasFS then themeStatus.Text = "File functions not available."; return end
        ensureFolders()
        local ok = pcall(function() writefile(THEME_DEFAULT_FILE, selectedTheme) end)
        themeStatus.Text = ok and ("'" .. selectedTheme .. "' loads on start.") or "Could not save."
        refreshThemes()
    end)

    task.spawn(function()
        task.wait(0.5)
        local n = readText(THEME_DEFAULT_FILE)
        if n then
            for _, e in ipairs(allThemes()) do
                if e.name == n then
                    applyThemeEntry(e)
                    break
                end
            end
            pcall(refreshThemes)
        end
    end)

    local cfgBox = createGroupbox(settings.Left, "Configs")
    local selected = nil

    local nameBox = createTextbox(cfgBox, "Config Name", "type a name", "", function() end)

    local cfgList = Instance.new("ScrollingFrame")
    cfgList.Size = UDim2.new(1, 0, 0, 100)
    cfgList.BorderSizePixel = 0
    cfgList.ScrollBarThickness = 4
    cfgList.CanvasSize = UDim2.new(0, 0, 0, 0)
    cfgList.AutomaticCanvasSize = Enum.AutomaticSize.Y
    cfgList.Parent = cfgBox
    setOrder(cfgList, cfgBox)
    skin(cfgList, "BackgroundColor3", "Inner")
    skin(cfgList, "ScrollBarImageColor3", "Accent")
    addStroke(cfgList, "Border", 1)
    do
        local l = Instance.new("UIListLayout")
        l.Padding = UDim.new(0, 3)
        l.SortOrder = Enum.SortOrder.Name
        l.Parent = cfgList
        local p = Instance.new("UIPadding")
        p.PaddingTop = UDim.new(0, 3); p.PaddingBottom = UDim.new(0, 3)
        p.PaddingLeft = UDim.new(0, 3); p.PaddingRight = UDim.new(0, 7)
        p.Parent = cfgList
    end

    local createBtn = createButton(cfgBox, "Create / Save Config")
    local loadBtn = createButton(cfgBox, "Load Selected")
    local overwriteBtn = createButton(cfgBox, "Overwrite Selected")
    local deleteBtn = createButton(cfgBox, "Delete Selected")
    local refreshBtn = createButton(cfgBox, "Refresh List")
    local setAutoBtn = createButton(cfgBox, "Set Selected As Autoload")
    local resetAutoBtn = createButton(cfgBox, "Reset Autoload")
    local resetAllBtn = createButton(cfgBox, "Reset All Settings")
    local autoLabel = createLabel(cfgBox, "Autoload: none", 16, true)
    local statusLabel = createLabel(cfgBox, hasFS and "Ready." or "Your executor has no file functions, configs are unavailable.", 30, true)

    local function setStatus(t) statusLabel.Text = t end

    local function cfgPath(n) return CFG_DIR .. "/" .. n .. ".json" end

    local function collect()
        local data = {}
        for key, opt in pairs(Options) do
            local ok, v = pcall(opt.Get)
            if ok then data[key] = v end
        end
        if S.safeCFrame then data._safezone = {S.safeCFrame:GetComponents()} end
        return data
    end

    local function applyData(data)
        for key, opt in pairs(Options) do
            if data[key] ~= nil then pcall(opt.Set, data[key]) end
        end
        if type(data._safezone) == "table" and #data._safezone == 12 then
            S.safeCFrame = CFrame.new(table.unpack(data._safezone))
        end
    end

    local refreshList
    refreshList = function()
        for _, c in ipairs(cfgList:GetChildren()) do
            if c:IsA("TextButton") then c:Destroy() end
        end
        local autoName = readText(AUTOLOAD_FILE)
        for _, name in ipairs(listJson(CFG_DIR)) do
            local b = Instance.new("TextButton")
            b.Name = string.lower(name)
            b.Size = UDim2.new(1, 0, 0, 20)
            b.Text = name .. ((name == autoName) and "  [autoload]" or "")
            b.Parent = cfgList
            styleButton(b)
            setButtonActive(b, name == selected)
            b.Activated:Connect(function()
                selected = name
                nameBox.Set(name)
                refreshList()
            end)
        end
        autoLabel.Text = "Autoload: " .. (autoName or "none")
    end

    local function saveConfig(name)
        if not hasFS then setStatus("File functions not available."); return end
        name = cleanName(name)
        if name == "" then setStatus("Type a config name first."); return end
        ensureFolders()
        local ok, err = pcall(function() writefile(cfgPath(name), HttpService:JSONEncode(collect())) end)
        if ok then
            selected = name
            setStatus("Saved '" .. name .. "'.")
            refreshList()
        else
            setStatus("Save failed: " .. tostring(err))
        end
    end

    local function loadConfig(name)
        if not hasFS then setStatus("File functions not available."); return end
        name = cleanName(name)
        if name == "" then setStatus("Select a config first."); return end
        local ok, data = pcall(function()
            local p = cfgPath(name)
            if not isfile(p) then return nil end
            return HttpService:JSONDecode(readfile(p))
        end)
        if ok and type(data) == "table" then
            applyData(data)
            pcall(refreshThemes)
            setStatus("Loaded '" .. name .. "'.")
        else
            setStatus("Could not load '" .. name .. "'.")
        end
    end

    createBtn.Activated:Connect(function() saveConfig(nameBox.Box.Text) end)
    loadBtn.Activated:Connect(function() loadConfig(selected or nameBox.Box.Text) end)
    overwriteBtn.Activated:Connect(function()
        if not selected then setStatus("Select a config first."); return end
        saveConfig(selected)
    end)
    deleteBtn.Activated:Connect(function()
        if not hasFS then setStatus("File functions not available."); return end
        if not selected then setStatus("Select a config first."); return end
        local name = selected
        removeFile(cfgPath(name), false)
        if readText(AUTOLOAD_FILE) == name then removeFile(AUTOLOAD_FILE, true) end
        selected = nil
        setStatus("Deleted '" .. name .. "'.")
        refreshList()
    end)
    refreshBtn.Activated:Connect(function() refreshList(); setStatus("List refreshed.") end)
    setAutoBtn.Activated:Connect(function()
        if not hasFS then setStatus("File functions not available."); return end
        if not selected then setStatus("Select a config first."); return end
        ensureFolders()
        local ok = pcall(function() writefile(AUTOLOAD_FILE, selected) end)
        setStatus(ok and ("'" .. selected .. "' will load on start.") or "Could not save autoload.")
        refreshList()
    end)
    resetAutoBtn.Activated:Connect(function()
        if not hasFS then setStatus("File functions not available."); return end
        removeFile(AUTOLOAD_FILE, true)
        setStatus("Autoload reset.")
        refreshList()
    end)
    resetAllBtn.Activated:Connect(function()
        for _, opt in pairs(Options) do
            if not opt.NoReset then pcall(opt.Set, opt.Default) end
        end
        S.safeCFrame = nil
        selectedTheme = "Strawberry"
        pcall(refreshThemes)
        setStatus("All settings reset to default.")
    end)

    refreshList()

    task.spawn(function()
        task.wait(1.2)
        local name = readText(AUTOLOAD_FILE)
        if name then
            loadConfig(name)
            selected = name
            pcall(refreshList)
        end
    end)
end

local guiOpen = false

-- how big the menu can be on this screen (1 on PC, smaller on phones)
local function fitScale()
    local cam = workspace.CurrentCamera
    local vp = cam and cam.ViewportSize or Vector2.new(1280, 720)
    return math.clamp(math.min((vp.X - 24) / WIN_W, (vp.Y - 60) / WIN_H, 1), 0.35, 1)
end

local function setGuiOpen(open)
    guiOpen = open
    if open then
        main.Visible = true
        -- keep it on screen
        main.Position = UDim2.new(0.5, 0, 0.5, 0)
        tween(uiScale, 0.2, {Scale = fitScale()})
    else
        closePicker()
        local t = tween(uiScale, 0.15, {Scale = 0}, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
        t.Completed:Connect(function(state)
            if not guiOpen and state == Enum.PlaybackState.Completed then main.Visible = false end
        end)
    end
end

H.toggleGui = function()
    if loaded then setGuiOpen(not guiOpen) end
end

-- screen rotated / resized: resize the open menu
task.spawn(function()
    local cam = workspace.CurrentCamera
    if not cam then return end
    connect(cam:GetPropertyChangedSignal("ViewportSize"), function()
        if guiOpen then uiScale.Scale = fitScale() end
    end)
end)

-- ---------- STRAWBERRY BUTTON (phones / tablets only) ----------
-- tap = open / close the menu, hold and move = drag it anywhere (it remembers where you put it)
local menuBtn = Instance.new("TextButton")
menuBtn.Name = "StrawberryMenuButton"
menuBtn.Size = UDim2.fromOffset(58, 58)
menuBtn.AnchorPoint = Vector2.new(0.5, 0.5)
menuBtn.Position = UDim2.new(0, 46, 0.5, 0)
menuBtn.Text = ""
menuBtn.AutoButtonColor = false
menuBtn.BorderSizePixel = 0
menuBtn.BackgroundColor3 = Color3.fromRGB(255, 70, 110)
menuBtn.Visible = false
menuBtn.ZIndex = 50
menuBtn.Parent = gui
do
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = menuBtn
    local grad = Instance.new("UIGradient")
    grad.Rotation = 90
    grad.Color = ColorSequence.new(Color3.fromRGB(255, 120, 150), Color3.fromRGB(200, 30, 70))
    grad.Parent = menuBtn
    local ring = Instance.new("UIStroke")
    ring.Color = WHITE
    ring.Thickness = 2
    ring.Transparency = 0.15
    ring.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    ring.Parent = menuBtn

    local berry = Instance.new("TextLabel")
    berry.Size = UDim2.new(1, 0, 1, -10)
    berry.BackgroundTransparency = 1
    berry.Text = "\u{1F353}"
    berry.TextScaled = true
    berry.Font = Enum.Font.GothamBold
    berry.ZIndex = 51
    berry.Parent = menuBtn
    local pad = Instance.new("UIPadding")
    pad.PaddingTop = UDim.new(0, 6)
    pad.PaddingLeft = UDim.new(0, 8)
    pad.PaddingRight = UDim.new(0, 8)
    pad.Parent = berry

    local tag = Instance.new("TextLabel")
    tag.Size = UDim2.new(1, 0, 0, 12)
    tag.Position = UDim2.new(0, 0, 1, -14)
    tag.BackgroundTransparency = 1
    tag.Text = "MENU"
    tag.TextSize = 10
    tag.Font = Enum.Font.GothamBold
    tag.TextColor3 = WHITE
    tag.ZIndex = 51
    tag.Parent = menuBtn

    local btnScale = Instance.new("UIScale")
    btnScale.Parent = menuBtn

    local pressing, moved, startPos, startInput = false, false, nil, nil
    menuBtn.InputBegan:Connect(function(input)
        if isPointerInput(input) then
            pressing, moved = true, false
            startPos, startInput = menuBtn.Position, input.Position
            tween(btnScale, 0.08, {Scale = 0.88})
        end
    end)
    connect(UIS.InputChanged, function(input)
        if pressing and (input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - startInput
            if d.Magnitude > 8 then moved = true end
            if moved then
                menuBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
            end
        end
    end)
    connect(UIS.InputEnded, function(input)
        if pressing and isPointerInput(input) then
            pressing = false
            tween(btnScale, 0.12, {Scale = 1}, Enum.EasingStyle.Back)
            if not moved then
                H.toggleGui()
                -- little spin so you see it worked
                berry.Rotation = 0
                tween(berry, 0.25, {Rotation = 360})
            end
        end
    end)
    -- remembers where you put it (saved with your config)
    Options.menuBtnPos = {
        Default = {0, 46, 0.5, 0},
        Get = function()
            local pz = menuBtn.Position
            return {pz.X.Scale, pz.X.Offset, pz.Y.Scale, pz.Y.Offset}
        end,
        Set = function(v)
            if type(v) == "table" and #v == 4 then menuBtn.Position = UDim2.new(v[1], v[2], v[3], v[4]) end
        end,
    }
end

-- phones and tablets only: a touch screen without a mouse (or a phone OS)
local isTouch = UIS.TouchEnabled and (not UIS.MouseEnabled or not UIS.KeyboardEnabled)
pcall(function()
    local pf = UIS:GetPlatform()
    if pf == Enum.Platform.Android or pf == Enum.Platform.IOS then isTouch = true end
end)
gui.DisplayOrder = math.max(gui.DisplayOrder, 10)

closeButton.Activated:Connect(function()
    if loaded then setGuiOpen(false) end
end)

switchPage("Main")

task.spawn(function()
    tween(dim, 0.3, {BackgroundTransparency = 0.45})
    tween(loaderScale, 0.25, {Scale = 1})
    task.wait(0.4)

    local statuses = {
        {1, "Planting seeds..."}, {20, "Watering the vines..."}, {45, "Picking strawberries..."},
        {70, "Washing the berries..."}, {90, "Almost ripe..."},
    }

    for p = 1, 100 do
        if not running then return end
        loaderFill.Size = UDim2.new(p / 100, 0, 1, 0)
        loaderPercent.Text = p .. "%"
        for _, s in ipairs(statuses) do
            if p >= s[1] then loaderStatus.Text = s[2] end
        end
        task.wait(0.01 + math.random() * 0.03)
    end

    loaderStatus.Text = "Ripe and ready!"
    task.wait(0.4)

    tween(loaderScale, 0.2, {Scale = 0}, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
    tween(dim, 0.25, {BackgroundTransparency = 1})
    task.wait(0.3)
    loader:Destroy()

    loaded = true
    wm.Visible = S.watermark
    menuBtn.Visible = isTouch
    setGuiOpen(true)
end)

do
    local dragging, dragStart, startPos = false, nil, nil

    top.InputBegan:Connect(function(input)
        if isPointerInput(input) then
            dragging = true
            dragStart = input.Position
            startPos = main.Position
            closePicker()
        end
    end)
    connect(UIS.InputEnded, function(input)
        if isPointerInput(input) then dragging = false end
    end)
    connect(UIS.InputChanged, function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
end

local function getSafeCFrame()
    if S.safeCFrame then return S.safeCFrame end
    local meteor = workspace:FindFirstChild("Meteor", true)
    if meteor then
        if meteor:IsA("BasePart") then
            return meteor.CFrame + Vector3.new(0, 10, 0)
        elseif meteor:IsA("Model") then
            return meteor:GetPivot() + Vector3.new(0, 10, 0)
        end
    end
    return nil
end

H.setSafeButton.Activated:Connect(function()
    local hrp = getRoot(player)
    if hrp then
        S.safeCFrame = hrp.CFrame
        H.setSafeButton.Text = "Safezone Saved!"
        task.delay(1.5, function() H.setSafeButton.Text = "Set Safezone Here" end)
    end
end)

H.tpSafeButton.Activated:Connect(function()
    local hrp = getRoot(player)
    local target = getSafeCFrame()
    if hrp and target then
        hrp.CFrame = target
        hrp.AssemblyLinearVelocity = Vector3.new()
    else
        H.tpSafeButton.Text = "No Safezone - Set One!"
        task.delay(1.5, function() H.tpSafeButton.Text = "Teleport To Safezone" end)
    end
end)

task.spawn(function()
    while running do
        task.wait(0.1)
        if S.safeEnabled then
            pcall(function()
                local hrp = getRoot(player)
                local hp, maxHp = nil, nil
                if H.myHealth then hp, maxHp = H.myHealth() end
                if hrp and hp and maxHp and maxHp > 0 then
                    local pct = (hp / maxHp) * 100
                    if hp > 0 and pct <= S.hpThreshold then
                        if not S.lowHpActive then
                            local target = getSafeCFrame()
                            if target then
                                S.lowHpActive = true
                                hrp.CFrame = target
                                hrp.AssemblyLinearVelocity = Vector3.new()
                            else
                                H.setSafeButton.Text = "No Safezone - Set One!"
                            end
                        end
                    elseif pct > S.hpThreshold then
                        S.lowHpActive = false
                    end
                end
            end)
        else
            S.lowHpActive = false
        end
    end
end)

;safeRun(function()
    local DEFAULT_RETURN = CFrame.new(1449.99, 82.16, -677.59)
    -- where you wait once every territory is yours: under the map at -618, -1131
    -- (the ground there is about Y 143, you wait "Undermap Depth" studs below it, min 10)
    local ALL_DONE_X, ALL_DONE_Z, ALL_DONE_GROUND = -618, -1131, 143.4
    local returnSpot = nil
    local learned = {}           -- territory index -> height offset that started a capture
    local holdCF, holdConn = nil, nil
    local defending = false

    local function status(t) pcall(function() H.zoneState.Text = t end) end

    if H.zoneReturnBtn then
        H.zoneReturnBtn.Activated:Connect(function()
            local hrp = getRoot(player)
            if hrp then
                returnSpot = hrp.CFrame
                H.zoneReturnBtn.Text = "Return Spot Saved!"
                task.delay(1.5, function() H.zoneReturnBtn.Text = "Set Return Spot Here" end)
            end
        end)
    end
    Options.zoneReturn = {
        Default = false,
        Get = function() return returnSpot and {returnSpot:GetComponents()} or false end,
        Set = function(v)
            if type(v) == "table" and #v == 12 then returnSpot = CFrame.new(table.unpack(v)) else returnSpot = nil end
        end,
    }

    -- holds you exactly in place every frame (you can't fall or be pushed, even under the map)
    local function startHold()
        if holdConn then return end
        holdConn = RunService.Heartbeat:Connect(function()
            -- low HP safezone / turning it off always wins over the hold
            if S.lowHpActive or not (S.autoZone or defending) or not running then
                if holdConn then holdConn:Disconnect(); holdConn = nil end
                holdCF = nil
                S.zoneNoclip = false
                return
            end
            local hrp = getRoot(player)
            if hrp and holdCF then
                hrp.CFrame = holdCF
                hrp.AssemblyLinearVelocity = Vector3.new()
                hrp.AssemblyAngularVelocity = Vector3.new()
            end
        end)
    end
    local function stopHold()
        if holdConn then holdConn:Disconnect(); holdConn = nil end
        holdCF = nil
        S.zoneNoclip = false
    end
    H.zoneStop = stopHold

    -- the capture / cooldown timer the game shows on screen
    local CAP_WORDS = {"captur", "claim", "conquer", "territor", "zone", "control", "occupy"}
    local CD_WORDS = {"cooldown", "wait", "again in", "recharg", "protected"}
    local labels, nextScan = {}, 0
    local function shown(lbl)
        local o = lbl
        while o and o ~= playerGui do
            if o:IsA("GuiObject") and not o.Visible then return false end
            if o:IsA("LayerCollector") and not o.Enabled then return false end
            o = o.Parent
        end
        return o == playerGui
    end
    local function zoneUi()
        local now = os.clock()
        if now >= nextScan then
            nextScan = now + 1
            labels = {}
            for _, g in ipairs(playerGui:GetChildren()) do
                if g:IsA("LayerCollector") and not string.find(g.Name, "Strawberry", 1, true) then
                    for _, d in ipairs(g:GetDescendants()) do
                        if d:IsA("TextLabel") then labels[#labels + 1] = d end
                    end
                end
            end
        end
        for _, lbl in ipairs(labels) do
            local ok, kind, txt = pcall(function()
                if not lbl.Parent or #lbl.Text == 0 or #lbl.Text > 80 or not shown(lbl) then return nil end
                local plain = string.gsub(lbl.Text, "<[^>]+>", "")
                local l = string.lower(plain .. " " .. lbl.Name .. " " .. (lbl.Parent and lbl.Parent.Name or ""))
                local timer = string.find(plain, "%d+:%d%d") or string.find(plain, "%d+%.?%d*%s*s%f[%A]")
                    or string.find(plain, "%d+%s*%%")
                if not timer then return nil end
                for _, w in ipairs(CD_WORDS) do
                    if string.find(l, w, 1, true) then return "cooldown", plain end
                end
                for _, w in ipairs(CAP_WORDS) do
                    if string.find(l, w, 1, true) then return "capturing", plain end
                end
                return nil
            end)
            if ok and kind then return kind, txt end
        end
        return nil
    end

    local function coreColor(pos)
        local col = nil
        pcall(function()
            for _, z in ipairs((workspace:FindFirstChild("MapResources") :: any).Zones:GetChildren()) do
                local core = z:FindFirstChild("LightCore")
                if core and (core.Position - pos).Magnitude < 15 then col = core.Color; break end
            end
        end)
        return col
    end

    local function spotFor(base, off, jiggle)
        local cf = CFrame.new(base + Vector3.new(0, off, 0))
        if jiggle then
            cf = cf * CFrame.new(math.sin(os.clock() * 8) * 0.35, 0, math.cos(os.clock() * 8) * 0.35)
        end
        return cf
    end

    -- ---------- other players standing in a territory ----------
    local radiusCache = {}
    local function zoneRadius(i, zone)
        if S.zoneEnemyRadius and S.zoneEnemyRadius > 0 then return S.zoneEnemyRadius end
        if radiusCache[i] then return radiusCache[i] end
        local r = 60
        pcall(function()
            local mr = workspace:FindFirstChild("MapResources") :: any
            for _, z in ipairs(mr.Zones:GetChildren()) do
                local core = z:FindFirstChild("LightCore")
                if core and (core.Position - zone.Position).Magnitude < 15 then
                    if z:IsA("Model") then
                        local _, size = z:GetBoundingBox()
                        r = math.clamp(math.max(size.X, size.Z) / 2, 20, 200)
                    end
                    break
                end
            end
        end)
        radiusCache[i] = r
        return r
    end

    -- the first other player (not on your team) inside territory i, or nil
    local function enemyIn(i, zone)
        local r = zoneRadius(i, zone)
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= player and not (player.Team ~= nil and plr.Team == player.Team) then
                local c = plr.Character
                local root = c and (c:FindFirstChild("HumanoidRootPart") or c.PrimaryPart) :: any
                if root then
                    local d = root.Position - zone.Position
                    if Vector3.new(d.X, 0, d.Z).Magnitude <= r and math.abs(d.Y) <= 80 then return plr end
                end
            end
        end
        return nil
    end

    -- ---------- defend: someone is inside one of YOUR territories ----------
    local function defendTarget()
        if not S.zoneDefend or S.lowHpActive then return nil end
        for i, z in ipairs(ZonesList) do
            if IsThisZoneGreen(z.Position) then
                local e = enemyIn(i, z)
                if e then return i, z end
            end
        end
        return nil
    end

    -- sit just under the territory (close enough to count as being there) until they leave
    local function defend(i, zone)
        defending = true
        S.zoneNoclip = true
        startHold()
        local off = math.clamp(learned[i] or -3, -8, -2)
        local t0 = os.clock()
        while running and S.zoneDefend and not S.lowHpActive and os.clock() - t0 < 300 do
            local e = enemyIn(i, zone)
            if not e then break end
            holdCF = CFrame.new(zone.Position + Vector3.new(0, off, 0))
            status(string.format("Territory %d: DEFENDING, %s is inside your territory", i, H.nameOf(e)))
            pcall(function() zoneRow.Label.Text = "Defending zone " .. i end)
            task.wait(0.1)
        end
        defending = false
        status(string.format("Territory %d: they left, defense done", i))
    end

    -- the height you set with the sliders, read live every tick
    local function userOffset()
        return S.zoneUnder and -S.zoneDepth or S.zoneHeight
    end

    local function farmZone(i, zone)
        local base = zone.Position
        local lastUser = userOffset()
        local lastUnder = S.zoneUnder
        local off = learned[i] or lastUser
        local startColor = coreColor(base)
        local t0, lastAdjust, capStart, lastSeen = os.clock(), os.clock(), nil, nil
        S.zoneNoclip = S.zoneUnder
        startHold()

        while S.autoZone and not S.lowHpActive and running do
            -- you moved a slider or flipped Undermap: use the new position right away
            local u = userOffset()
            if u ~= lastUser or S.zoneUnder ~= lastUnder then
                lastUser, lastUnder = u, S.zoneUnder
                off = u
                learned[i] = nil
                lastAdjust = os.clock()
                S.zoneNoclip = S.zoneUnder
            end
            local under = S.zoneUnder
            if defendTarget() then
                return false
            end
            local intruder = enemyIn(i, zone)
            if intruder then
                status(string.format("Territory %d: %s is inside, waiting for them to leave", i, H.nameOf(intruder)))
                return false
            end
            if IsThisZoneGreen(base) then
                status(string.format("Territory %d captured!", i))
                return true
            end
            local now = os.clock()
            local kind, txt = zoneUi()
            local col = coreColor(base)
            local colorMoved = startColor and col and col ~= startColor
            local active = kind ~= nil or colorMoved

            if active then
                capStart = capStart or now
                lastSeen = now
                learned[i] = off
                -- the timer is running: stand completely still until it is done
                holdCF = spotFor(base, off, false)
                status(string.format("Territory %d: %s%s", i, kind == "cooldown" and "cooldown " or "capturing ",
                    txt and ("(" .. txt .. ")") or ""))
                if now - capStart > S.zoneWait then
                    status(string.format("Territory %d: waited too long, next", i))
                    return false
                end
            elseif capStart then
                -- the timer went away: finished, go to the next territory
                if now - (lastSeen or now) > 1.5 then
                    status(string.format("Territory %d: timer finished", i))
                    return true
                end
            else
                holdCF = spotFor(base, off, true)
                -- nothing started yet: move a little lower (or, under the map, a little closer to the surface)
                if S.zoneAutoLower and now - lastAdjust > 2.5 then
                    lastAdjust = now
                    if under then
                        off = math.min(off + 3, -2)
                    else
                        off = math.max(off - 3, -15)
                    end
                end
                status(string.format("Territory %d: waiting for capture to start (height %d)", i, math.floor(off)))
                if now - t0 > (S.zoneAutoLower and 25 or 15) then
                    status(string.format("Territory %d: capture never started, next", i))
                    return false
                end
            end
            task.wait(0.05)
        end
        return false
    end

    task.spawn(function()
        while running do
            if S.autoZone and not S.lowHpActive and not S.tripBusy and getRoot(player) then
                local ok = pcall(function()
                    local lastBase = nil
                    for i, zone in ipairs(ZonesList) do
                        if not S.autoZone or S.lowHpActive or not running then break end
                        local di, dz = defendTarget()
                        if di then defend(di, dz) end
                        if IsThisZoneGreen(zone.Position) then
                            zoneRow.Label.Text = "Zone " .. i .. " Yours, Skip"
                        elseif enemyIn(i, zone) then
                            zoneRow.Label.Text = "Zone " .. i .. " player inside, later"
                        else
                            zoneRow.Label.Text = "Zone " .. i .. (S.zoneUnder and " (undermap)" or "")
                            farmZone(i, zone)
                            lastBase = zone.Position
                        end
                    end
                    if not S.autoZone or S.lowHpActive then return end

                    -- territories still to take (nobody inside), and the ones blocked by a player
                    local function pending()
                        local free, blocked = 0, {}
                        for i, z in ipairs(ZonesList) do
                            if not IsThisZoneGreen(z.Position) then
                                local e = enemyIn(i, z)
                                if e then
                                    blocked[#blocked + 1] = i .. " (" .. H.nameOf(e) .. ")"
                                else
                                    free = free + 1
                                end
                            end
                        end
                        return free, blocked
                    end
                    local function allOwned()
                        local free = pending()
                        return free == 0
                    end

                    if allOwned() then
                        -- everything is yours: hide under the map at the waiting spot and stay frozen
                        -- there until one of them gets taken, then farm again straight away
                        S.zoneNoclip = true
                        startHold()
                        zoneRow.Label.Text = "All zones yours - hidden undermap"
                        while S.autoZone and not S.lowHpActive and running and allOwned() do
                            local di, dz = defendTarget()
                            if di then defend(di, dz) end
                            local depth = math.max(S.zoneDepth or 15, 10)
                            holdCF = CFrame.new(ALL_DONE_X, ALL_DONE_GROUND - depth, ALL_DONE_Z)
                            S.zoneNoclip = true
                            local _, blocked = pending()
                            if #blocked > 0 then
                                status(string.format("Territory: hidden %d studs under the map, waiting for players to leave: %s",
                                    depth, table.concat(blocked, ", ")))
                                zoneRow.Label.Text = "Waiting for players to leave"
                            else
                                status(string.format("Territory: all captured, hidden %d studs under the map until one is lost", depth))
                                zoneRow.Label.Text = "All zones yours - hidden undermap"
                            end
                            task.wait(0.25)
                        end
                        if S.autoZone and not S.lowHpActive then
                            status("Territory: a territory is free to take, farming again")
                        end
                        return
                    end

                    -- some territories did not finish: short break, then try again
                    if S.zoneUnder and lastBase then
                        holdCF = CFrame.new(lastBase - Vector3.new(0, S.zoneDepth, 0))
                    else
                        stopHold()
                        local hrp = getRoot(player)
                        if hrp then
                            hrp.CFrame = returnSpot or DEFAULT_RETURN
                            hrp.AssemblyLinearVelocity = Vector3.new()
                        end
                    end
                    zoneRow.Label.Text = "Waiting (5s)..."
                    status("Territory: round done, trying the rest again in 5s")
                    local w0 = os.clock()
                    while os.clock() - w0 < 5 and S.autoZone and running do
                        local di, dz = defendTarget()
                        if di then defend(di, dz) end
                        task.wait(0.25)
                    end
                end)
                if not ok then task.wait(0.5) end
            else
                local di, dz = nil, nil
                if not S.autoZone then di, dz = defendTarget() end
                if di then
                    local hrp = getRoot(player)
                    local back = hrp and hrp.CFrame
                    pcall(defend, di, dz)
                    stopHold()
                    hrp = getRoot(player)
                    if hrp and back and not S.lowHpActive then
                        hrp.CFrame = back
                        hrp.AssemblyLinearVelocity = Vector3.new()
                    end
                end
                if holdConn and not defending then stopHold() end
                if not S.autoZone then
                    pcall(function() if zoneRow.Label.Text ~= "Auto Zone" then zoneRow.Label.Text = "Auto Zone" end end)
                    if not S.zoneDefend then
                        status("Territory: idle")
                    elseif not di then
                        status("Territory: defending is on, watching your territories")
                    end
                end
                task.wait(S.zoneDefend and 0.25 or 0.1)
            end
        end
        stopHold()
    end)
end)

----------------------------------------------------------------------
-- ATTACK HITBOX + COOLDOWNS  (Miscellaneous tab)
-- Learns every dino you play. Exact hitboxes come from catching the game's own
-- hit checks (GetPartBoundsInBox / Radius / PartsInPart / Blockcast / Spherecast ...)
-- when your attack runs. Roars (keys 1 and 2 by default) are never counted.
-- Data is remembered per dino in StrawberryHub/attacks.json.
----------------------------------------------------------------------
;safeRun(function()
    S.hbShow, S.hbPreview, S.cdHud = false, false, false
    S.hbColor, S.hbAlpha, S.hbTime = Color3.fromRGB(255, 70, 110), 0.6, 1.5

    ------------------------------------------------------------------
    -- UI
    ------------------------------------------------------------------
    local ui = createGroupbox(Pages.Miscellaneous.Right, "Attack Hitbox + Cooldowns")
    createCheckbox(ui, "Show Hitbox On Attack", false, function(v) S.hbShow = v end, "hbShow")
    createCheckbox(ui, "Always Show Reach Preview", false, function(v) S.hbPreview = v end, "hbPreview")
    createCheckbox(ui, "Cooldown HUD", false, function(v) S.cdHud = v end, "cdHud")
    createColorRow(ui, "Hitbox Color", S.hbColor, function(c) S.hbColor = c end, "hbColor")
    createSlider(ui, "Hitbox Transparency", 0, 95, 60, "%", function(v) S.hbAlpha = v / 100 end, "hbAlpha")
    createSlider(ui, "Show Hitbox For (x0.1s)", 3, 50, 15, "", function(v) S.hbTime = v / 10 end, "hbTime")

    local ignoreSet = {}
    local DIGITS = {["0"] = "Zero", ["1"] = "One", ["2"] = "Two", ["3"] = "Three", ["4"] = "Four",
        ["5"] = "Five", ["6"] = "Six", ["7"] = "Seven", ["8"] = "Eight", ["9"] = "Nine"}
    local purgeIgnored
    local function applyIgnore(txt)
        ignoreSet = {}
        for part in string.gmatch(tostring(txt), "[^,%s]+") do
            local k = DIGITS[part] or (#part == 1 and string.upper(part)) or part
            ignoreSet[k] = true
        end
        if purgeIgnored then purgeIgnored() end
    end
    applyIgnore("1,2")
    createTextbox(ui, "Roar / Ignored Keys", "e.g. 1,2", "1,2", applyIgnore, "hbIgnoreKeys")

    local dinoL = createLabel(ui, "Dino: -", 16, true)
    local listL = createLabel(ui, "Turn a toggle on, then attack with your dino.", 30)
    local infoL = createLabel(ui, "", 44)
    local forgetBtn = createButton(ui, "Forget This Dino's Data")

    ------------------------------------------------------------------
    -- saved data per dino
    ------------------------------------------------------------------
    local DATA_FILE = DIR .. "/attacks.json"
    local data: {[any]: any} = {}
    if hasFS then
        pcall(function()
            if isfile(DATA_FILE) then data = HttpService:JSONDecode(readfile(DATA_FILE)) end
        end)
    end
    if type(data) ~= "table" then data = {} end
    local dirty = false
    if data._v ~= 2 then
        for _, dd in pairs(data) do
            if type(dd) == "table" then
                for _, a in pairs(dd) do
                    if type(a) == "table" then
                        a.size, a.off, a.shape, a.src, a.rank, a.cand = nil, nil, nil, nil, nil, nil
                    end
                end
            end
        end
        data._v = 2
        dirty = true
    end

    task.spawn(function()
        while running do
            task.wait(3)
            if dirty and hasFS then
                dirty = false
                pcall(ensureFolders)
                pcall(function() writefile(DATA_FILE, HttpService:JSONEncode(data)) end)
            end
        end
    end)

    local function species()
        local ok, sp = pcall(function() return H.getSpecies and H.getSpecies() end)
        if not ok or not sp or sp == "" then
            local c = player.Character
            sp = c and c.Name or "unknown"
        end
        return tostring(sp)
    end

    local function dinoData()
        local sp = species()
        if sp == "_v" then sp = "unknown" end
        if type(data[sp]) ~= "table" then data[sp] = {} end
        return data[sp], sp
    end

    forgetBtn.Activated:Connect(function()
        local sp = species()
        data[sp] = {}
        dirty = true
    end)

    local function v3t(v) return {v.X, v.Y, v.Z} end
    local function t2v3(t) return Vector3.new(t[1] or 1, t[2] or 1, t[3] or 1) end
    local function dims(v) return string.format("%.1f x %.1f x %.1f", v.X, v.Y, v.Z) end

    ------------------------------------------------------------------
    -- word lists
    ------------------------------------------------------------------
    local ATTACK_WORDS = {"attack", "bite", "claw", "slash", "strike", "swipe", "tail", "stomp", "headbutt", "ram",
        "peck", "sting", "whip", "pounce", "lunge", "chomp", "charge", "gore", "kick", "smash", "tackle", "maul",
        "rake", "snap", "combo", "special", "heavy", "light", "slam", "spit", "venom"}
    local ROAR_WORDS = {"roar", "call", "growl", "howl", "bellow", "screech", "chirp", "honk", "squawk", "trumpet",
        "grunt", "cry", "vocal", "shout", "scream", "hiss", "rumble", "sound", "voice", "emote"}
    local IGNORE_ANIMS = {"idle", "walk", "run", "sprint", "swim", "fly", "glide", "jump", "fall", "land", "sit",
        "rest", "sleep", "lay", "eat", "drink", "look", "breath", "turn", "dance", "pose", "crouch", "limp",
        "death", "die", "hurt", "stun", "flinch", "react", "sniff", "drown", "climb", "dodge", "spawn", "hatch"}
    local REMOTE_WORDS = {"attack", "bite", "hit", "damage", "combat", "ability", "skill", "swing", "strike",
        "claw", "slash", "melee"}
    local REMOTE_SKIP = {"eat", "drink", "quest", "chat", "ping", "zone", "teleport", "shop", "buy"}
    local MOVE_KEYS = {W = true, A = true, S = true, D = true, Space = true, LeftShift = true, RightShift = true,
        LeftControl = true, Tab = true, Slash = true, Escape = true, Up = true, Down = true, Left = true,
        Right = true, LeftAlt = true, K = true}

    local function hasWord(l, list)
        for _, w in ipairs(list) do
            if string.find(l, w, 1, true) then return w end
        end
        return nil
    end

    local function pretty(name)
        name = tostring(name or "")
        name = string.gsub(name, "[_%-]", " ")
        name = string.gsub(name, "(%l)(%u)", "%1 %2")
        name = trim(name)
        local l = string.lower(name)
        if name == "" or l == "animation" or l == "animationtrack" then return "Attack" end
        return name
    end

    -- removes anything learned from roars / ignored keys (also cleans data from older versions)
    purgeIgnored = function()
        for _, dd in pairs(data) do
            if type(dd) == "table" then
                for key, a in pairs(dd) do
                    local k = string.match(tostring(key), "^in:(.+)$")
                    local lbl = type(a) == "table" and string.lower(tostring(a.label or "")) or ""
                    if (k and ignoreSet[k]) or hasWord(lbl, ROAR_WORDS) or hasWord(string.lower(tostring(key)), ROAR_WORDS) then
                        dd[key] = nil
                        dirty = true
                    end
                end
            end
        end
    end

    purgeIgnored()

    local function inOtherChar(obj)
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character and obj:IsDescendantOf(plr.Character) then return true end
        end
        return false
    end

    -- numeric values on your dino whose lowercase name passes match()
    local function scanNumbers(match)
        local out = {}
        local char = player.Character
        for _, h in ipairs({player, char, getRoot(player), char and char:FindFirstChildOfClass("Humanoid")}) do
            if h then
                for n, v in pairs(h:GetAttributes()) do
                    if (type(v) == "number" or typeof(v) == "Vector3") and match(string.lower(n)) then
                        out[#out + 1] = {n, v}
                    end
                end
                for _, d in ipairs(h:GetChildren()) do
                    if (d:IsA("NumberValue") or d:IsA("IntValue") or d:IsA("Vector3Value")) and match(string.lower(d.Name)) then
                        out[#out + 1] = {d.Name, d.Value}
                    end
                end
            end
        end
        return out
    end

    local function pickFor(list, label)
        if #list == 0 then return nil end
        local l = string.lower(label)
        for _, e in ipairs(list) do
            local w = hasWord(string.lower(e[1]), ATTACK_WORDS)
            if w and string.find(l, w, 1, true) then return e end
        end
        if #list == 1 then return list[1] end
        return nil
    end

    local function cooldownAttr(label)
        local nums = {}
        for _, e in ipairs(scanNumbers(function(n)
            return (string.find(n, "cooldown", 1, true) or string.sub(n, -2) == "cd")
                and not string.find(n, "last", 1, true) and not hasWord(n, ROAR_WORDS)
        end)) do
            if type(e[2]) == "number" then
                local v = e[2]
                if v > 120 and v < 120000 then v = v / 1000 end
                if v >= 0.05 and v <= 120 then nums[#nums + 1] = {e[1], v} end
            end
        end
        local e = pickFor(nums, label)
        return e and e[2], e and e[1]
    end

    local function rangeAttr(label)
        local ok = {}
        for _, e in ipairs(scanNumbers(function(n)
            if string.find(n, "render", 1, true) or string.find(n, "view", 1, true) or string.find(n, "smell", 1, true)
                or hasWord(n, ROAR_WORDS) then return false end
            return string.find(n, "hitbox", 1, true) or string.find(n, "range", 1, true)
                or string.find(n, "reach", 1, true) or string.find(n, "radius", 1, true)
        end)) do
            if typeof(e[2]) == "Vector3" or (type(e[2]) == "number" and e[2] >= 1 and e[2] <= 200) then ok[#ok + 1] = e end
        end
        local e = pickFor(ok, label)
        return e and e[2], e and e[1]
    end

    local function _estimate()
        local char, root = player.Character, getRoot(player)
        if not (char and root) then return nil end
        local _, size = char:GetBoundingBox()
        local long = math.max(size.X, size.Z)
        local reach = long * 0.4 + 2
        local sz = Vector3.new(math.max(math.min(size.X, size.Z) * 0.9, 3), math.max(size.Y * 0.6, 3), reach)
        return sz, CFrame.new(0, 0, -(long / 2 + reach / 2) + 1)
    end

    ------------------------------------------------------------------
    -- drawing
    ------------------------------------------------------------------
    local visFolder = nil
    local function getFolder()
        if not visFolder or not visFolder.Parent then
            visFolder = Instance.new("Folder")
            visFolder.Name = "StrawberryHitboxVis"
            visFolder.Parent = workspace
        end
        return visFolder
    end

    -- glowing force-field box with a thin outline, no text on it
    local function makeVis(size, shape)
        local p = Instance.new("Part")
        p.Name = "StrawberryHitboxVis"
        p.Anchored, p.CanCollide, p.CanTouch, p.CanQuery, p.CastShadow = true, false, false, false, false
        p.Material = Enum.Material.ForceField
        p.Color = S.hbColor
        p.Transparency = 1
        if shape == "Ball" then
            p.Shape = Enum.PartType.Ball
            local d = math.max(size.X, size.Y, size.Z)
            size = Vector3.new(d, d, d)
        elseif shape == "Cylinder" then
            p.Shape = Enum.PartType.Cylinder
        end
        p.Size = size
        p.Parent = getFolder()

        local outline
        if shape == "Ball" then
            outline = Instance.new("SelectionSphere")
            outline.SurfaceTransparency = 0.92
            outline.SurfaceColor3 = S.hbColor
        else
            outline = Instance.new("SelectionBox")
            outline.LineThickness = 0.035
            outline.SurfaceTransparency = 0.92
            outline.SurfaceColor3 = S.hbColor
        end
        outline.Color3 = S.hbColor
        outline.Transparency = 1
        outline.Adornee = p
        outline.Parent = p
        return p, outline
    end

    local function fillAlpha() return math.clamp(S.hbAlpha * 0.7, 0, 0.95) end

    local function flash(rel, size, shape)
        if not S.hbShow then return end
        local root = getRoot(player)
        if not root then return end
        local ok, p, outline = pcall(makeVis, size, shape)
        if not ok or not p then return end
        p.CFrame = root.CFrame * rel
        pcall(function()
            tween(p, 0.08, {Transparency = fillAlpha()})
            tween(outline, 0.08, {Transparency = 0.1})
        end)
        local t0 = os.clock()
        task.spawn(function()
            while p.Parent and os.clock() - t0 < S.hbTime and running do
                local r = getRoot(player)
                if r then p.CFrame = r.CFrame * rel end
                RunService.RenderStepped:Wait()
            end
            pcall(function()
                tween(p, 0.25, {Transparency = 1})
                tween(outline, 0.25, {Transparency = 1})
            end)
            task.wait(0.27)
            pcall(function() p:Destroy() end)
        end)
    end

    ------------------------------------------------------------------
    -- detection state
    ------------------------------------------------------------------
    local trackRoot = nil         -- your root part, updated every frame (read inside the hook)
    local armUntil = 0            -- only record hit checks shortly after you press a key
    local qbuf, qn = {}, 0        -- ring buffer of the game's hit checks
    local lastInput = nil
    local lastUsed = {}
    local cur, lastDone = nil, nil
    local watchUntil = 0
    local hookOn = false
    local hookStatus = "hit-check hook: not supported by your executor"

    local QUERY = {
        GetPartBoundsInBox = 1, GetPartBoundsInRadius = 2, GetPartsInPart = 3, Blockcast = 4,
        Spherecast = 5, Shapecast = 6, Raycast = 7, FindPartsInRegion3 = 8,
        FindPartsInRegion3WithIgnoreList = 8, FindPartsInRegion3WithWhiteList = 8,
    }

    connect(RunService.Heartbeat, function()
        trackRoot = getRoot(player)
    end)

    -- turns a recorded hit check into a world CFrame + size + shape
    local function queryShape(q)
        local a, k = q.a, q.k
        local ok, cf, size, shape = pcall(function()
            if k == 1 then
                return a[1], a[2], "Block"
            elseif k == 2 then
                local r = a[2] * 2
                return CFrame.new(a[1]), Vector3.new(r, r, r), "Ball"
            elseif k == 3 then
                local part = a[1]
                return part.CFrame, part.Size, part:IsA("Part") and part.Shape.Name or "Block"
            elseif k == 4 or k == 5 or k == 6 then
                local base, dir
                if k == 4 then base, dir = a[2], a[3]
                elseif k == 5 then base, dir = Vector3.new(a[2] * 2, a[2] * 2, a[2] * 2), a[3]
                else base, dir = a[1].Size, a[2] end
                local startPos = (k == 4 and a[1].Position) or (k == 5 and a[1]) or a[1].Position
                local len = dir.Magnitude
                local center = startPos + dir / 2
                local look = len > 0.01 and CFrame.lookAt(center, center + dir) or CFrame.new(center)
                return look, Vector3.new(base.X, base.Y, base.Z + len), "Block"
            elseif k == 7 then
                return nil
            elseif k == 8 then
                return a[1].CFrame, a[1].Size, "Block"
            end
            return nil
        end)
        if not ok or typeof(cf) ~= "CFrame" or typeof(size) ~= "Vector3" then return nil end
        if math.max(size.X, size.Y, size.Z) > 80 or size.Magnitude < 0.5 then return nil end
        if q.rcf and (cf.Position - q.rcf.Position).Magnitude > 40 then return nil end
        return cf, size, shape
    end

    local RANK = {query = 6, hitpart = 5, value = 4, measured = 3}

    -- things that can be hit (other players and NPC dinos)
    local hitModels = setmetatable({}, {__mode = "k"})
    local function addHum(h)
        if h:IsA("Humanoid") and h.Parent and h.Parent:IsA("Model") then hitModels[h.Parent] = true end
    end
    task.spawn(function()
        local n = 0
        for _, d in ipairs(workspace:GetDescendants()) do
            if not running then return end
            if d:IsA("Humanoid") then addHum(d) end
            n = n + 1
            if n % 3000 == 0 then task.wait() end
        end
    end)
    connect(workspace.DescendantAdded, function(d)
        if d:IsA("Humanoid") then task.defer(function() pcall(addHum, d) end) end
    end)

    local function modelHp(m)
        local at = m:GetAttribute("Health")
        if type(at) == "number" then return at end
        local h = m:FindFirstChildOfClass("Humanoid")
        return h and h.Health or nil
    end

    local function snapshotTargets()
        local root = getRoot(player)
        if not root then return {} end
        local myChar = player.Character
        local out = {}
        for m in pairs(hitModels) do
            if m.Parent and m ~= myChar then
                local pp = m.PrimaryPart or m:FindFirstChild("HumanoidRootPart") or m:FindFirstChildWhichIsA("BasePart")
                if pp and (pp.Position - root.Position).Magnitude < 70 then
                    local hp = modelHp(m)
                    if hp and hp > 0 then out[#out + 1] = {m = m, hp = hp} end
                end
            end
        end
        return out
    end

    -- distance from the center of your dino to the closest point of the target's body
    local function surfaceDistance(rootCf, m)
        local ok, cf, size = pcall(function() return m:GetBoundingBox() end)
        if not ok or not cf then return nil end
        local lp = cf:PointToObjectSpace(rootCf.Position)
        local h = size / 2
        local cl = Vector3.new(math.clamp(lp.X, -h.X, h.X), math.clamp(lp.Y, -h.Y, h.Y), math.clamp(lp.Z, -h.Z, h.Z))
        return (lp - cl).Magnitude
    end

    -- how much a hit check looks like YOUR attack: 3 = ignores your dino (best), 2 = looks only at others, 1 = no filter, 0 = not yours
    local PARAM_INDEX = {[1] = 3, [2] = 3, [3] = 2, [4] = 4, [5] = 4, [6] = 3}
    local function paramScore(q)
        local idx = PARAM_INDEX[q.k]
        local prm = idx and q.a[idx]
        if prm == nil then return 1 end
        local ok, res = pcall(function()
            local t = typeof(prm)
            if t ~= "OverlapParams" and t ~= "RaycastParams" then return 1 end
            local myChar = player.Character
            local hasMe = false
            for _, inst in ipairs(prm.FilterDescendantsInstances) do
                if myChar and (inst == myChar or myChar:IsDescendantOf(inst) or inst:IsDescendantOf(myChar)) then
                    hasMe = true
                    break
                end
            end
            local ft = tostring(prm.FilterType)
            local exclude = string.find(ft, "Exclude", 1, true) or string.find(ft, "Blacklist", 1, true)
            if hasMe then return exclude and 3 or 0 end
            return exclude and 1 or 2
        end)
        return ok and res or 1
    end

    local function sameShape(c, size, rel)
        return (t2v3(c.size) - size).Magnitude <= math.max(0.6, size.Magnitude * 0.12)
            and (CFrame.new(table.unpack(c.off)).Position - rel.Position).Magnitude <= 2
    end

    -- how far in front of your dino the box reaches
    local function boxReach(sizeT, offT)
        local size, rel = t2v3(sizeT), CFrame.new(table.unpack(offT))
        local far = 0
        for _, sx in ipairs({-1, 1}) do
            for _, sy in ipairs({-1, 1}) do
                for _, sz in ipairs({-1, 1}) do
                    local corner = rel:PointToWorldSpace(Vector3.new(sx * size.X / 2, sy * size.Y / 2, sz * size.Z / 2))
                    far = math.max(far, Vector3.new(corner.X, 0, corner.Z).Magnitude)
                end
            end
        end
        return far
    end

    -- box drawn from measured reach (no exact hit check available)
    local function reachBox(reach)
        local char = player.Character
        if not char then return nil end
        local _, size = char:GetBoundingBox()
        local w = math.max(math.min(size.X, size.Z) * 0.9, 2)
        local h = math.max(size.Y * 0.7, 2)
        return Vector3.new(w, h, reach), CFrame.new(0, 0, -reach / 2)
    end

    local function measureHits(rec, a)
        if not rec.targets or not rec.rootCf then return end
        for _, t in ipairs(rec.targets) do
            if not t.counted and t.m.Parent then
                local hp = modelHp(t.m)
                if hp and hp < t.hp - 0.01 then
                    t.counted = true
                    local d = surfaceDistance(rec.rootCf, t.m)
                    if d and d < 60 then
                        a.reach = math.max(a.reach or 0, d)
                        a.hits = (a.hits or 0) + 1
                        dirty = true
                    end
                end
            end
        end
    end

    local function finalize(rec)
        if cur == rec then cur = nil end
        local dd = dinoData()
        local a = dd[rec.key]
        if type(a) ~= "table" then a = {} end
        a.label = rec.label

        if rec.anim then a.animSeen = true end
        if a.animSeen and not rec.anim then return end

        -- cooldown
        local last = lastUsed[rec.key]
        if last then
            local gap = rec.t0 - last
            if gap > 0.15 and gap < 60 then
                a.learned = a.learned and math.min(a.learned, gap) or gap
                a.samples = (a.samples or 0) + 1
            end
        end
        lastUsed[rec.key] = rec.t0
        local acd, aname = cooldownAttr(rec.label)
        if acd then a.attrCd, a.attrName = acd, aname end

        local found = nil

        -- 1) the game's own hit check for this attack
        local bestQ, bestScore = nil, 0
        for i = 1, #qbuf do
            local q = qbuf[i]
            if q and q.t >= rec.t0 - 0.15 and q.t <= rec.t0 + 0.45 and q.k ~= 7 then
                local sc = paramScore(q)
                if sc > 0 then
                    local cf, size, shape = queryShape(q)
                    if cf and q.rcf then
                        if sc > bestScore or (sc == bestScore and bestQ and q.t < bestQ.q.t) then
                            bestQ, bestScore = {q = q, cf = cf, size = size, shape = shape}, sc
                        end
                    end
                end
            end
        end
        if bestQ then
            found = {rel = bestQ.q.rcf:ToObjectSpace(bestQ.cf), size = bestQ.size, shape = bestQ.shape,
                rank = RANK.query, kind = "q" .. bestQ.q.k, strong = bestScore >= 3, src = "exact hit check"}
        end

        -- 2) a part literally named hitbox that appeared for this attack
        if not found and #rec.parts > 0 and rec.rootCf then
            table.sort(rec.parts, function(x, y) return x.score > y.score end)
            local hp = rec.parts[1]
            found = {rel = rec.rootCf:ToObjectSpace(hp.cf), size = hp.size, shape = hp.shape, rank = RANK.hitpart,
                kind = "part", strong = false, src = "hitbox part"}
        end

        -- 3) a range / hitbox value stored on your dino
        if not found then
            local r, rname = rangeAttr(rec.label)
            if typeof(r) == "Vector3" then
                found = {rel = CFrame.new(0, 0, -r.Z / 2), size = r, shape = "Block", rank = RANK.value,
                    kind = "value", strong = true, src = "game value " .. rname}
            elseif r then
                found = {rel = CFrame.new(0, 0, -r * 0.5), size = Vector3.new(r * 2, r * 2, r * 2), shape = "Ball",
                    rank = RANK.value, kind = "value", strong = true, src = "game value " .. rname}
            end
        end

        -- only trust a shape once it is clearly the attack: a strong match, or the same box twice in a row
        if found and found.rank >= (a.rank or 0) then
            if a.cand and a.cand.kind == found.kind and sameShape(a.cand, found.size, found.rel) then
                a.cand.n = a.cand.n + 1
            else
                a.cand = {size = v3t(found.size), off = {found.rel:GetComponents()}, shape = found.shape, kind = found.kind, n = 1}
            end
            if found.strong or a.cand.n >= 2 then
                a.size, a.off, a.shape, a.src, a.rank = a.cand.size, a.cand.off, a.cand.shape, found.src, found.rank
            end
        end

        -- 4) real hits: how far away the things you damaged were
        measureHits(rec, a)
        task.delay(0.6, function()
            pcall(function()
                local a2 = dinoData()[rec.key]
                if type(a2) == "table" then measureHits(rec, a2) end
            end)
        end)

        dd[rec.key] = a
        dirty = true
        lastDone = rec

        -- draw only what is known for real
        if a.size and a.off then
            flash(CFrame.new(table.unpack(a.off)), t2v3(a.size), a.shape)
        elseif a.reach then
            local size, rel = reachBox(a.reach)
            if size then flash(rel, size, "Block") end
        end
    end

    local function ignoredInput(now)
        return lastInput and now - lastInput.t < 0.6 and ignoreSet[lastInput.k]
    end

    -- every detector calls this; events within 0.3s are the same attack. Returns nil when ignored.
    local function attackEvent(kind, name, id)
        local now = os.clock()
        if hasWord(string.lower(tostring(name or "")), ROAR_WORDS) then return nil end
        if ignoredInput(now) then return nil end
        if cur and now - cur.t0 < 0.3 then
            if kind == "an" then
                cur.anim = true
                if cur.generic and name then
                    cur.label = pretty(name) .. (cur.input and (" [" .. cur.input .. "]") or "")
                    cur.generic = false
                end
            end
            return cur
        end
        local inp = (lastInput and now - lastInput.t < 0.5) and lastInput.k or nil
        local label = pretty(name)
        local rec = {
            key = inp and ("in:" .. inp) or (kind .. ":" .. tostring(id or name)),
            label = label .. (inp and (" [" .. inp .. "]") or ""),
            generic = label == "Attack",
            input = inp, t0 = now, parts = {}, anim = (kind == "an"),
        }
        cur = rec
        local r0 = getRoot(player)
        rec.rootCf = r0 and r0.CFrame
        rec.targets = snapshotTargets()
        watchUntil = now + 0.45
        armUntil = math.max(armUntil, now + 0.5)
        task.delay(0.45, function() pcall(finalize, rec) end)
        return rec
    end

    -- input: remembers the key, arms the hit-check recorder, and learns "pressed but refused" (on cooldown)
    connect(UIS.InputBegan, function(input, gp)
        if gp then return end
        local k = nil
        local t = input.UserInputType
        if t == Enum.UserInputType.MouseButton1 then k = "LMB"
        elseif t == Enum.UserInputType.MouseButton2 then k = "RMB"
        elseif t == Enum.UserInputType.Touch then k = "Tap"
        elseif t == Enum.UserInputType.Keyboard and not MOVE_KEYS[input.KeyCode.Name] then k = input.KeyCode.Name end
        if not k then return end
        local now = os.clock()
        lastInput = {k = k, t = now}
        if ignoreSet[k] then return end
        armUntil = now + 0.8
        local key = "in:" .. k
        task.delay(0.5, function()
            pcall(function()
                local last = lastUsed[key]
                if last and last < now and (not lastDone or lastDone.key ~= key or lastDone.t0 < now) and not (cur and cur.key == key) then
                    local dd = dinoData()
                    local a = dd[key]
                    if type(a) == "table" then
                        local gap = now - last
                        if gap < 60 and (not a.learned or gap < a.learned) then
                            a.lower = math.max(a.lower or 0, gap)
                            dirty = true
                        end
                    end
                end
            end)
        end)
    end)

    -- 1) animations
    local charConns, hookedAnimators, hookedChar = {}, setmetatable({}, {__mode = "k"}), nil
    local function hookAnimator(an)
        if hookedAnimators[an] then return end
        hookedAnimators[an] = true
        charConns[#charConns + 1] = an.AnimationPlayed:Connect(function(track)
            pcall(function()
                local anim = track.Animation
                local name = (track.Name ~= "" and track.Name ~= "Animation" and track.Name) or (anim and anim.Name) or ""
                local l = string.lower(name)
                if hasWord(l, ROAR_WORDS) then return end
                local id = anim and string.match(anim.AnimationId, "%d+") or name
                if not hasWord(l, ATTACK_WORDS) then
                    if hasWord(l, IGNORE_ANIMS) or track.Looped then return end
                    if not (lastInput and os.clock() - lastInput.t < 0.4) then return end
                end
                attackEvent("an", name, id)
            end)
        end)
    end

    -- 2) game cooldown flags (exact cooldown)
    local flagStart = {}
    local function hookFlags(holder)
        charConns[#charConns + 1] = holder.AttributeChanged:Connect(function(n)
            pcall(function()
                local v = holder:GetAttribute(n)
                if type(v) ~= "boolean" then return end
                local l = string.lower(n)
                if hasWord(l, ROAR_WORDS) then return end
                local cdName = string.find(l, "cooldown", 1, true) or string.sub(l, -2) == "cd"
                local readyName = string.find(l, "canattack", 1, true) or string.find(l, "ready", 1, true)
                if not (cdName or readyName) then return end
                local onCd = (cdName and v) or (readyName and not cdName and not v)
                local now = os.clock()
                if onCd then
                    local rec = cur or lastDone
                    if rec and now - rec.t0 < 0.8 then flagStart[n] = rec end
                else
                    local rec = flagStart[n]
                    flagStart[n] = nil
                    if rec then
                        local a = dinoData()[rec.key]
                        if type(a) == "table" then
                            a.gameCd, a.gameName = now - rec.t0, n
                            dirty = true
                        end
                    end
                end
            end)
        end)
    end

    local function hookCharacter(char)
        for _, c in ipairs(charConns) do pcall(function() c:Disconnect() end) end
        charConns, hookedAnimators = {}, setmetatable({}, {__mode = "k"})
        cur, lastDone, lastUsed = nil, nil, {}
        hookedChar = char
        if not char then return end
        for _, d in ipairs(char:GetDescendants()) do
            if d:IsA("Animator") then hookAnimator(d) end
        end
        charConns[#charConns + 1] = char.DescendantAdded:Connect(function(d)
            if d:IsA("Animator") then pcall(hookAnimator, d) end
        end)
        hookFlags(char)
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then hookFlags(root) end
    end
    hookFlags(player)

    -- watchdog: re-hooks whenever you spawn a new dino, so it never stops working
    task.spawn(function()
        while running do
            pcall(function()
                local char = player.Character
                if char ~= hookedChar then
                    hookCharacter(char)
                elseif char then
                    for _, d in ipairs(char:GetDescendants()) do
                        if d:IsA("Animator") and not hookedAnimators[d] then hookAnimator(d) end
                    end
                end
            end)
            task.wait(1)
        end
    end)

    -- 3) parts literally named hitbox that appear right after you attack
    connect(workspace.DescendantAdded, function(obj)
        if os.clock() > watchUntil or not cur then return end
        if not obj:IsA("BasePart") or obj.Name == "StrawberryHitboxVis" then return end
        local rec = cur
        task.defer(function()
            pcall(function()
                if not obj.Parent or (visFolder and obj:IsDescendantOf(visFolder)) or inOtherChar(obj) then return end
                local l = string.lower(obj.Name)
                if not (string.find(l, "hitbox", 1, true) or string.find(l, "attackbox", 1, true)
                    or string.find(l, "damagebox", 1, true)) or hasWord(l, ROAR_WORDS) then return end
                local root = getRoot(player)
                if not root then return end
                local d = (obj.Position - root.Position).Magnitude
                if d > 40 or obj.Size.Magnitude < 0.5 then return end
                local mine = player.Character and obj:IsDescendantOf(player.Character)
                rec.parts[#rec.parts + 1] = {size = obj.Size, cf = obj.CFrame, name = obj.Name,
                    score = (mine and 3 or 0) - d / 20, shape = obj:IsA("Part") and obj.Shape.Name or "Block"}
            end)
        end)
    end)

    -- 4) attack remotes
    local function onRemote(remote, args)
        local l = string.lower(remote.Name)
        if hasWord(l, REMOTE_SKIP) or hasWord(l, ROAR_WORDS) then return end
        local label = remote.Name
        for i = 1, args.n do
            if type(args[i]) == "string" and #args[i] < 30 then
                local al = string.lower(args[i])
                if hasWord(al, ROAR_WORDS) then return end
                if hasWord(al, ATTACK_WORDS) then label = args[i]; break end
            end
        end
        attackEvent("re", label, remote.Name .. ":" .. label)
    end

    -- the hook: records the game's hit checks right after you press a key, and its attack remotes
    if type(hookmetamethod) == "function" and type(getnamecallmethod) == "function" then
        local ok = pcall(function()
            local wrap = (type(newcclosure) == "function") and newcclosure or function(f) return f end
            local isOurs = (type(checkcaller) == "function") and checkcaller or function() return false end
            local old
            old = hookmetamethod(game, "__namecall", wrap(function(self, ...)
                if hookOn and not isOurs() then
                    local m = getnamecallmethod()
                    local qk = QUERY[m]
                    if qk then
                        if self == workspace and os.clock() < armUntil and trackRoot then
                            qn = qn % 40 + 1
                            qbuf[qn] = {t = os.clock(), k = qk, a = table.pack(...), rcf = trackRoot.CFrame}
                        end
                    elseif (m == "FireServer" or m == "InvokeServer") and typeof(self) == "Instance" then
                        local n = string.lower(self.Name)
                        for _, w in ipairs(REMOTE_WORDS) do
                            if string.find(n, w, 1, true) then
                                local args = table.pack(...)
                                task.defer(function() pcall(onRemote, self, args) end)
                                break
                            end
                        end
                    end
                end
                return old(self, ...)
            end))
        end)
        if ok then
            hookOn = true
            hookStatus = "hit-check hook: on (exact hitboxes when the game checks hits on your side)"
        end
    end

    ------------------------------------------------------------------
    -- cooldown HUD
    ------------------------------------------------------------------
    local hud = Instance.new("Frame")
    hud.Size = UDim2.fromOffset(240, 0)
    hud.AutomaticSize = Enum.AutomaticSize.Y
    hud.Position = UDim2.fromOffset(10, 40)
    hud.BorderSizePixel = 0
    hud.Visible = false
    hud.Active = true
    hud.Parent = gui
    skin(hud, "BackgroundColor3", "Window")
    addStroke(hud, "Border", 1)
    do
        local line = Instance.new("Frame")
        line.Size = UDim2.new(1, 0, 0, 2)
        line.BorderSizePixel = 0
        line.Parent = hud
        skin(line, "BackgroundColor3", "Accent")
        local lay = Instance.new("UIListLayout")
        lay.Padding = UDim.new(0, 3)
        lay.SortOrder = Enum.SortOrder.LayoutOrder
        lay.Parent = hud
        local pad = Instance.new("UIPadding")
        pad.PaddingLeft, pad.PaddingRight = UDim.new(0, 6), UDim.new(0, 6)
        pad.PaddingTop, pad.PaddingBottom = UDim.new(0, 4), UDim.new(0, 6)
        pad.Parent = hud
    end
    local hudTitle = Instance.new("TextLabel")
    hudTitle.Size = UDim2.new(1, 0, 0, 16)
    hudTitle.BackgroundTransparency = 1
    hudTitle.Font = FONT
    hudTitle.TextSize = 13
    hudTitle.TextXAlignment = Enum.TextXAlignment.Left
    hudTitle.LayoutOrder = 1
    hudTitle.Parent = hud
    skin(hudTitle, "TextColor3", "Text")

    local rows = {}
    local function getRow(i)
        if rows[i] then return rows[i] end
        local f = Instance.new("Frame")
        f.Size = UDim2.new(1, 0, 0, 20)
        f.BackgroundTransparency = 1
        f.LayoutOrder = 10 + i
        f.Parent = hud
        local name = Instance.new("TextLabel")
        name.Size = UDim2.new(1, -60, 0, 14)
        name.BackgroundTransparency = 1
        name.Font = FONT
        name.TextSize = 12
        name.TextXAlignment = Enum.TextXAlignment.Left
        name.TextTruncate = Enum.TextTruncate.AtEnd
        name.Parent = f
        skin(name, "TextColor3", "Text")
        local st = Instance.new("TextLabel")
        st.Size = UDim2.new(0, 58, 0, 14)
        st.Position = UDim2.new(1, -58, 0, 0)
        st.BackgroundTransparency = 1
        st.Font = FONT
        st.TextSize = 12
        st.TextXAlignment = Enum.TextXAlignment.Right
        st.Parent = f
        local track = Instance.new("Frame")
        track.Size = UDim2.new(1, 0, 0, 3)
        track.Position = UDim2.fromOffset(0, 16)
        track.BorderSizePixel = 0
        track.Parent = f
        skin(track, "BackgroundColor3", "Control")
        local fill = Instance.new("Frame")
        fill.Size = UDim2.new(1, 0, 1, 0)
        fill.BorderSizePixel = 0
        fill.Parent = track
        skin(fill, "BackgroundColor3", "Accent")
        rows[i] = {f = f, name = name, st = st, fill = fill}
        return rows[i]
    end

    do
        local dragging, ds, sp0 = false, nil, nil
        hud.InputBegan:Connect(function(input)
            if isPointerInput(input) then dragging, ds, sp0 = true, input.Position, hud.Position end
        end)
        connect(UIS.InputEnded, function(input) if isPointerInput(input) then dragging = false end end)
        connect(UIS.InputChanged, function(input)
            if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
                or input.UserInputType == Enum.UserInputType.Touch) then
                local d = input.Position - ds
                hud.Position = UDim2.fromOffset(sp0.X.Offset + d.X, sp0.Y.Offset + d.Y)
            end
        end)
        Options.cdPos = {
            Default = {10, 40},
            Get = function() return {hud.Position.X.Offset, hud.Position.Y.Offset} end,
            Set = function(v)
                if type(v) == "table" and tonumber(v[1]) and tonumber(v[2]) then hud.Position = UDim2.fromOffset(v[1], v[2]) end
            end,
        }
    end

    -- best cooldown + how sure we are
    local function bestCd(a)
        if a.gameCd then return a.gameCd, "exact" end
        if a.attrCd then return a.attrCd, "game value" end
        if a.learned then
            if a.lower and a.learned - a.lower <= 0.25 then return a.learned, "exact" end
            return a.learned, ((a.samples or 0) >= 3) and "learned" or "learning"
        end
        return nil, nil
    end

    local function sortedAttacks(dd)
        local list = {}
        for key, a in pairs(dd) do
            if type(a) == "table" and a.label then list[#list + 1] = {key = key, a = a} end
        end
        table.sort(list, function(x, y) return x.a.label < y.a.label end)
        return list
    end

    -- reach preview that follows you (only when the real hitbox / reach is known)
    local preview, previewOutline = nil, nil
    connect(RunService.RenderStepped, function()
        if not S.hbPreview and not preview then return end
        pcall(function()
            local root = getRoot(player)
            if not S.hbPreview or not root then
                if preview then preview:Destroy(); preview = nil end
                return
            end
            local dd = dinoData()
            local pick = lastDone and dd[lastDone.key]
            if type(pick) ~= "table" or not (pick.size or pick.reach) then
                pick = nil
                for _, e in ipairs(sortedAttacks(dd)) do
                    if e.a.size or e.a.reach then pick = e.a; break end
                end
            end
            if not pick then
                if preview then preview:Destroy(); preview = nil end
                return
            end
            local size, rel, shape
            if pick.size and pick.off then
                size, rel, shape = t2v3(pick.size), CFrame.new(table.unpack(pick.off)), pick.shape
            else
                size, rel = reachBox(pick.reach)
                shape = "Block"
            end
            if not size then return end
            local key = dims(size) .. tostring(shape)
            if not preview or not preview.Parent or preview:GetAttribute("K") ~= key then
                if preview then preview:Destroy() end
                preview, previewOutline = makeVis(size, shape)
                preview:SetAttribute("K", key)
                previewOutline.Transparency = 0.3
            end
            preview.Color = S.hbColor
            previewOutline.Color3 = S.hbColor
            preview.Transparency = math.min(fillAlpha() + 0.15, 0.97)
            preview.CFrame = root.CFrame * rel
        end)
    end)

    task.spawn(function()
        while running do
            task.wait(0.1)
            pcall(function()
                local dd, sp = dinoData()
                local list = sortedAttacks(dd)
                local now = os.clock()

                hud.Visible = S.cdHud and loaded and #list > 0
                hudTitle.Text = "Cooldowns  |  " .. sp
                for i, e in ipairs(list) do
                    local row = getRow(i)
                    row.f.Visible = true
                    row.name.Text = e.a.label
                    local cd = bestCd(e.a)
                    local last = lastUsed[e.key]
                    if cd and last and now - last < cd then
                        local left = cd - (now - last)
                        row.st.Text = string.format("%.1fs", left)
                        row.st.TextColor3 = Color3.fromRGB(255, 90, 90)
                        row.fill.Size = UDim2.new(1 - left / cd, 0, 1, 0)
                    else
                        row.st.Text = cd and "READY" or "?"
                        row.st.TextColor3 = Color3.fromRGB(90, 255, 140)
                        row.fill.Size = UDim2.new(1, 0, 1, 0)
                    end
                end
                for i = #list + 1, #rows do rows[i].f.Visible = false end

                dinoL.Text = "Dino: " .. sp .. "  (" .. #list .. " attack" .. (#list == 1 and "" or "s") .. " known)"
                if #list == 0 then
                    listL.Text = "Turn a toggle on, then attack with your dino."
                    listL.Size = UDim2.new(1, 0, 0, 30)
                else
                    local lines = {}
                    for _, e in ipairs(list) do
                        local cd, src = bestCd(e.a)
                        local cdTxt = cd and string.format("%s%.2fs (%s)", (src == "learning" or src == "learned") and "~" or "", cd, src)
                            or "cooldown ?"
                        local hb
                        if e.a.size and e.a.off then
                            hb = string.format("reach %.1f studs, box %s (%s)", boxReach(e.a.size, e.a.off), dims(t2v3(e.a.size)), tostring(e.a.src))
                        elseif e.a.reach then
                            hb = string.format("reach %.1f studs (measured from %d hit%s)", e.a.reach, e.a.hits or 0, (e.a.hits == 1) and "" or "s")
                        else
                            hb = "hitbox: hit something to measure it"
                        end
                        lines[#lines + 1] = e.a.label .. "\n  CD " .. cdTxt .. "\n  " .. hb
                    end
                    listL.Text = table.concat(lines, "\n")
                    listL.Size = UDim2.new(1, 0, 0, 16 * #lines * 3 + 4)
                end
                infoL.Text = hookStatus .. "\nSpam each attack a few times so it can lock in the exact cooldown."
            end)
        end
    end)

    H.attackCleanup = function()
        hookOn = false
        for _, c in ipairs(charConns) do pcall(function() c:Disconnect() end) end
        if preview then pcall(function() preview:Destroy() end) end
        if visFolder then pcall(function() visFolder:Destroy() end) end
    end
end)

----------------------------------------------------------------------
-- HP READER - your dino's real health (the game's own Health value first, then the Humanoid)
----------------------------------------------------------------------
;safeRun(function()
    local HP_NAMES = {"Health", "HP", "CurrentHealth", "DinoHealth", "Hp"}
    local MAX_NAMES = {"MaxHealth", "MaxHP", "HealthMax", "MaximumHealth", "MaxHp"}
    local maxSeen, maxChar = 0, nil

    local function num(h, names)
        if not h then return nil end
        for _, n in ipairs(names) do
            local a = h:GetAttribute(n)
            if type(a) == "number" then return a end
            local v = h:FindFirstChild(n)
            if v and (v:IsA("NumberValue") or v:IsA("IntValue")) then return v.Value end
        end
        return nil
    end

    -- returns current, max, where it came from
    H.myHealth = function()
        local char = player.Character
        if not char then return nil end
        if char ~= maxChar then maxChar, maxSeen = char, 0 end
        for _, h in ipairs({char, char:FindFirstChild("HumanoidRootPart"), char:FindFirstChildOfClass("Humanoid"), player}) do
            local cur = num(h, HP_NAMES)
            if cur then
                local mx = num(h, MAX_NAMES) or num(char, MAX_NAMES)
                if not mx or mx <= 0 then
                    maxSeen = math.max(maxSeen, cur)
                    mx = maxSeen
                end
                return cur, math.max(mx, 1), "game value"
            end
        end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then return hum.Health, math.max(hum.MaxHealth, 1), "Humanoid" end
        return nil
    end
end)

----------------------------------------------------------------------
-- DINO LIST (Primeval Earth) - shared by Growth and the rest of the script
-- finds which dino you are from its name, your character, or any value the game puts on you
----------------------------------------------------------------------
;safeRun(function()
    local LIST = {
        {"Acrocanthosaurus", "carnivore"}, {"Allosaurus", "carnivore"}, {"Cameroceras", "carnivore"},
        {"Carnotaurus", "carnivore"}, {"Cryolophosaurus", "carnivore"}, {"Deinosuchus", "carnivore"},
        {"Giganotosaurus", "carnivore"}, {"Irritator", "carnivore"}, {"Kelenken", "carnivore"},
        {"Microraptor", "carnivore"}, {"Monolophosaurus", "carnivore"}, {"Mosasaurus", "carnivore"},
        {"Otodus Megalodon", "carnivore"}, {"Megalodon", "carnivore"}, {"Pteranodon", "carnivore"},
        {"Pycnonemosaurus", "carnivore"}, {"Quetzalcoatlus", "carnivore"}, {"Spinosaurus", "carnivore"},
        {"Tarbosaurus", "carnivore"}, {"Ubirajara Jubatus", "carnivore"}, {"Ubirajara", "carnivore"},
        {"Tyrant Raptor", "carnivore"}, {"Albino Tyrant", "carnivore"}, {"Bauruspinus", "carnivore"},
        {"Yuqivenator", "carnivore"}, {"Kharnivore", "carnivore"},
        {"Apatosaurus", "herbivore"}, {"Edmontosaurus", "herbivore"}, {"Triceratops", "herbivore"},
        {"Parasaurolophus", "herbivore"}, {"Plateosaurus", "herbivore"}, {"Therizinosaurus", "herbivore"},
        {"Argentinosaurus", "herbivore"},
        {"Deinocheirus", "omnivore"}, {"Dodo", "omnivore"}, {"Gallimimus", "omnivore"},
    }
    local function norm(v) return (string.gsub(string.lower(tostring(v or "")), "[^%a]", "")) end
    local keys = {}
    for _, e in ipairs(LIST) do keys[#keys + 1] = {k = norm(e[1]), name = e[1], diet = e[2]} end
    table.sort(keys, function(a, b) return #a.k > #b.k end)   -- longest first: "Otodus Megalodon" before "Megalodon"

    H.matchDino = function(str)
        local n = norm(str)
        if n == "" then return nil end
        for _, e in ipairs(keys) do
            if n == e.k then return e.name, e.diet end
        end
        for _, e in ipairs(keys) do
            if string.find(n, e.k, 1, true) then return e.name, e.diet end
        end
        return nil
    end

    local cacheChar, cacheName, cacheDiet, cacheT = nil, nil, nil, 0
    H.findMyDino = function()
        local char = player.Character
        if not char then return nil end
        if char == cacheChar and cacheName and os.clock() - cacheT < 5 then return cacheName, cacheDiet end
        if char == cacheChar and not cacheName and os.clock() - cacheT < 1 then return nil end
        cacheChar, cacheT = char, os.clock()
        local tries = {char.Name}
        local ok, sp = pcall(function() return H.getSpecies and H.getSpecies() end)
        if ok and sp then table.insert(tries, 1, sp) end
        for _, h in ipairs({char, char:FindFirstChildOfClass("Humanoid"), player}) do
            if h then
                for _, v in pairs(h:GetAttributes()) do
                    if type(v) == "string" then tries[#tries + 1] = v end
                end
                for _, c in ipairs(h:GetChildren()) do
                    if c:IsA("StringValue") then tries[#tries + 1] = c.Value end
                end
            end
        end
        for _, t in ipairs(tries) do
            local name, diet = H.matchDino(t)
            if name then
                cacheName, cacheDiet = name, diet
                return name, diet
            end
        end
        cacheName, cacheDiet = nil, nil
        return nil
    end
end)

----------------------------------------------------------------------
-- GROWTH  (Main tab)
-- One bar per stage: Baby fills up -> Juvenile starts at 0 and fills -> Adult -> Elder.
-- Reads your stage and its progress straight from the game's own growth UI / values,
-- so it works for any dino and any skin.
----------------------------------------------------------------------
;safeRun(function()
    local STAGES = {"Baby", "Juvenile", "Adult", "Elder"}
    local ALIASES = {
        {"baby", "hatchling", "infant", "newborn"},
        {"juvenile", "juv", "teen", "teenager", "young", "subadult", "sub adult"},
        {"adult"},
        {"elder", "ancient"},
    }
    local CTX = {"grow", "stage", "age", "life", "matur", "evol", "elder"}

    -- ---------- UI ----------
    local box = createGroupbox(Pages.Main.Right, "Growth")
    local nameL = createLabel(box, "Dinosaur: -", 16, true)
    local chainL = createLabel(box, "", 16, true)
    chainL.RichText = true
    chainL.TextSize = 12
    chainL.TextWrapped = false
    chainL.TextTruncate = Enum.TextTruncate.AtEnd

    local track = Instance.new("Frame")
    track.Size = UDim2.new(1, 0, 0, 22)
    track.BorderSizePixel = 0
    track.Parent = box
    setOrder(track, box)
    skin(track, "BackgroundColor3", "Control")
    addStroke(track, "Black", 1)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BorderSizePixel = 0
    fill.Parent = track
    skin(fill, "BackgroundColor3", "Accent")
    local barText = Instance.new("TextLabel")
    barText.Size = UDim2.new(1, 0, 1, 0)
    barText.BackgroundTransparency = 1
    barText.TextColor3 = WHITE
    barText.TextStrokeTransparency = 0.4
    barText.TextSize = 13
    barText.Font = FONT
    barText.ZIndex = 3
    barText.Text = "detecting..."
    barText.Parent = track

    local nextL = createLabel(box, "Next stage: -", 16, true)
    local elderL = createLabel(box, "To Elder: -", 16, true)
    local srcL = createLabel(box, "", 30)
    local infoBtn = createButton(box, "Show Detection Info")
    local infoOut = createLabel(box, "", 16)

    -- if the game never shows your stage: tell it once, it follows the % bar from there
    local manualStage = {}   -- dino -> stage index set by you or learned
    local curSp = "-"
    local stageRow = Instance.new("Frame")
    stageRow.Size = UDim2.new(1, 0, 0, 20)
    stageRow.BackgroundTransparency = 1
    stageRow.Parent = box
    setOrder(stageRow, box)
    do
        local lay = Instance.new("UIListLayout")
        lay.FillDirection = Enum.FillDirection.Horizontal
        lay.Padding = UDim.new(0, 3)
        lay.Parent = stageRow
        for i, nm in ipairs({"Baby", "Juv", "Adult", "Elder"}) do
            local b = Instance.new("TextButton")
            b.Size = UDim2.new(0.25, -3, 1, 0)
            b.Text = nm
            b.Parent = stageRow
            styleButton(b)
            b.Activated:Connect(function()
                manualStage[curSp] = i
                infoOut.Text = "Stage set to " .. STAGES[i] .. " for " .. curSp .. ". It moves up by itself when the % bar restarts."
                infoOut.Size = UDim2.new(1, 0, 0, 30)
            end)
        end
    end
    createLabel(box, "Stage wrong or not detected? Press your stage once.", 16)

    local function setBar(frac, text)
        fill:TweenSize(UDim2.new(math.clamp(frac or 0, 0, 1), 0, 1, 0), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.25, true)
        barText.Text = text
    end

    -- ---------- helpers ----------
    local function norm(t)
        local a = string.gsub(tostring(t), "<[^>]+>", "")
        local b = string.gsub(string.lower(a), "[^%a]+", " ")
        return " " .. trim(b) .. " "
    end
    local function stageOfWords(w, exact)
        for idx, list in ipairs(ALIASES) do
            for _, a in ipairs(list) do
                if exact then
                    if w == " " .. a .. " " then return idx end
                elseif string.find(w, " " .. a .. " ", 1, true) then
                    return idx
                end
            end
        end
        return nil
    end
    local function hasCtx(s2)
        local l = string.lower(s2)
        for _, k in ipairs(CTX) do
            if string.find(l, k, 1, true) then return true end
        end
        return false
    end
    local function shown(g)
        local o = g
        while o and o ~= playerGui do
            if o:IsA("GuiObject") and not o.Visible then return false end
            if o:IsA("LayerCollector") and not o.Enabled then return false end
            o = o.Parent
        end
        return o == playerGui
    end

    local labels, nextScan = {}, 0
    local function hudLabels()
        local now = os.clock()
        if now >= nextScan then
            nextScan = now + 2
            labels = {}
            for _, g in ipairs(playerGui:GetChildren()) do
                if g:IsA("LayerCollector") and not string.find(g.Name, "Strawberry", 1, true) then
                    for _, d in ipairs(g:GetDescendants()) do
                        if d:IsA("TextLabel") or d:IsA("TextButton") then labels[#labels + 1] = d end
                    end
                end
            end
        end
        return labels
    end

    -- the progress bar / percent that sits next to the stage text on the game's HUD
    local function progressNear(lbl)
        local best, bestScore, how = nil, -1, nil
        local roots = {lbl.Parent, lbl.Parent and lbl.Parent.Parent}
        for _, r in ipairs(roots) do
            if r and r:IsA("GuiObject") then
                for _, d in ipairs(r:GetDescendants()) do
                    if d ~= lbl and d.Visible then
                        if (d:IsA("Frame") or d:IsA("ImageLabel")) and d.Parent and d.Parent:IsA("GuiObject") then
                            local l = string.lower(d.Name)
                            local named = string.find(l, "fill", 1, true) or string.find(l, "bar", 1, true)
                                or string.find(l, "progress", 1, true) or string.find(l, "grow", 1, true)
                                or string.find(l, "xp", 1, true) or string.find(l, "inner", 1, true)
                            local pw = d.Parent.AbsoluteSize.X
                            if named and pw > 30 and d.AbsoluteSize.X <= pw + 1 and d.Parent.AbsoluteSize.X > d.Parent.AbsoluteSize.Y then
                                local f = math.clamp(d.AbsoluteSize.X / pw, 0, 1)
                                local score = (string.find(l, "fill", 1, true) and 3 or 1) + (hasCtx(d:GetFullName()) and 2 or 0)
                                if score > bestScore then best, bestScore, how = f, score, "HUD bar '" .. d.Name .. "'" end
                            end
                        elseif (d:IsA("TextLabel")) and bestScore < 5 then
                            local pct = string.match(d.Text, "(%d+%.?%d*)%s*%%")
                            if pct then best, bestScore, how = math.clamp((tonumber(pct) or 0) / 100, 0, 1), 5, "HUD text '" .. d.Text .. "'" end
                        end
                    end
                end
            end
        end
        local pct = string.match(lbl.Text, "(%d+%.?%d*)%s*%%")
        if pct then return math.clamp((tonumber(pct) or 0) / 100, 0, 1), "HUD text '" .. lbl.Text .. "'" end
        return best, how
    end

    -- stage from the HUD: exact stage word wins, ties go to labels that look like growth UI
    local function fromHud()
        local found, foundLbl, score = nil, nil, -1
        local distinct = {}
        for _, lbl in ipairs(hudLabels()) do
            local ok, idx, sc = pcall(function()
                if not lbl.Parent or #lbl.Text == 0 or #lbl.Text > 40 then return nil end
                local w = norm(lbl.Text)
                -- quick text check first, the slow checks only for labels that mention a stage
                if not stageOfWords(w, false) then return nil end
                if not shown(lbl) then return nil end
                local exact = stageOfWords(w, true)
                local ctx = hasCtx(lbl:GetFullName())
                local idx2 = exact or (ctx and stageOfWords(w, false)) or nil
                if not idx2 then return nil end
                return idx2, (exact and 2 or 0) + (ctx and 3 or 0)
            end)
            if ok and idx then
                distinct[idx] = true
                if sc > score then found, foundLbl, score = idx, lbl, sc end
            end
        end
        -- a visible list of ALL stages (a legend) is not your stage, unless one label looks like the growth UI
        local n = 0
        for _ in pairs(distinct) do n = n + 1 end
        if n > 1 and score < 3 then return nil end
        return found, foundLbl
    end

    -- stage / progress from values on your dino
    -- your dino model (or a part of it) named after the stage, e.g. "Argentinosaurus_Juvenile" / a child called "Adult"
    local function fromModel()
        local char = player.Character
        if not char then return nil end
        local idx = stageOfWords(norm((string.gsub(char.Name, "_", " "))), false)
        if idx then return idx, "model name '" .. char.Name .. "'" end
        for _, c in ipairs(char:GetChildren()) do
            if c:IsA("Model") or c:IsA("Folder") or c:IsA("StringValue") then
                local w = norm(c:IsA("StringValue") and c.Value or c.Name)
                local i2 = stageOfWords(w, true)
                if i2 then return i2, "'" .. c.Name .. "' in your dino" end
            end
        end
        return nil
    end

    local function fromValues()
        local char = player.Character
        local stage, frac, how = nil, nil, nil
        local function check(h, path)
            if not h then return end
            local attrs = h:GetAttributes()
            for _, d in ipairs(h:GetChildren()) do
                if d:IsA("ValueBase") then attrs[d.Name] = d.Value end
            end
            for n, v in pairs(attrs) do
                local l = string.lower(n)
                if type(v) == "string" and not stage and (hasCtx(l) or string.find(l, "stage", 1, true)) then
                    local idx = stageOfWords(norm(v), false)
                    if idx then stage, how = idx, path .. "." .. n end
                elseif type(v) == "number" then
                    if (string.find(l, "stage", 1, true) or string.find(l, "lifestage", 1, true)) and v % 1 == 0 and not stage then
                        if v >= 0 and v <= 4 then
                            stage = (attrs[n] == 0 or v == 0) and 1 or v
                            if stage > 4 then stage = 4 end
                            how = path .. "." .. n
                        end
                    elseif (string.find(l, "grow", 1, true) or string.find(l, "matur", 1, true) or string.find(l, "progress", 1, true)
                        or l == "xp" or l == "exp") and not string.find(l, "max", 1, true) and not string.find(l, "rate", 1, true) and not frac then
                        local mx = attrs["Max" .. n] or attrs[n .. "Max"]
                        if type(mx) == "number" and mx > 0 then
                            frac = math.clamp(v / mx, 0, 1)
                        elseif v <= 1 then
                            frac = math.clamp(v, 0, 1)
                        elseif v <= 100 then
                            frac = v / 100
                        end
                        if frac then how = (how and (how .. " + ") or "") .. path .. "." .. n end
                    end
                end
            end
        end
        check(char, "Character")
        check(char and char:FindFirstChildOfClass("Humanoid"), "Humanoid")
        check(player, "Player")
        for _, c in ipairs(player:GetChildren()) do
            if c:IsA("Folder") or c:IsA("Configuration") then check(c, "Player." .. c.Name) end
        end
        return stage, frac, how
    end

    -- ---------- timing ----------
    local samples, curKey, etaT = {}, nil, {}
    local stageStart, stageLen = {}, {}   -- species -> stage -> seconds it took

    local function fmtClock(sec)
        sec = math.max(0, math.ceil(sec))
        local h = math.floor(sec / 3600)
        local m = math.floor((sec % 3600) / 60)
        local sc = sec % 60
        if h > 0 then return string.format("%d:%02d:%02d", h, m, sc) end
        return string.format("%02d:%02d", m, sc)
    end
    local function stable(key, now, secs)
        local e = etaT[key]
        local target = now + secs
        if not e or math.abs(target - e) > math.max(8, secs * 0.12) then etaT[key] = target; e = target end
        return math.max(0, e - now)
    end

    -- the game's growth percentage ("64%"): a label that is only a percentage, best match wins
    local pctLbl, pctNext = nil, 0
    local GROW_WORDS = {"grow", "age", "xp", "exp", "stage", "progress", "level", "matur", "evol", "elder"}
    local function scorePct(d)
        if not d.Parent or not shown(d) then return -1 end
        if not string.match(d.Text, "^%s*%d+%.?%d*%s*%%%s*$") then return -1 end
        local sc = 0
        local full = string.lower(d:GetFullName())
        for _, w in ipairs(GROW_WORDS) do
            if string.find(full, w, 1, true) then sc = sc + 5; break end
        end
        local cam = workspace.CurrentCamera
        local vp = cam and cam.ViewportSize or Vector2.new(1280, 720)
        local holder = d.Parent :: any
        if holder and holder:IsA("GuiObject") and holder.AbsoluteSize.X > vp.X * 0.4 then sc = sc + 3 end
        if d.AbsolutePosition.Y > vp.Y * 0.75 then sc = sc + 2 end
        -- health / hunger / stamina bars are not growth
        for _, w in ipairs({"health", "hunger", "thirst", "stamina", "food", "water", "hp", "oxygen"}) do
            if string.find(full, w, 1, true) then return -1 end
        end
        return sc
    end
    local function growthPct()
        if pctLbl and scorePct(pctLbl) >= 0 then
            return math.clamp((tonumber(string.match(pctLbl.Text, "(%d+%.?%d*)")) or 0) / 100, 0, 1), pctLbl
        end
        if os.clock() < pctNext then return nil end
        pctNext = os.clock() + 2
        local best, bestSc = nil, -1
        for _, lbl in ipairs(hudLabels()) do
            local ok, sc = pcall(scorePct, lbl)
            if ok and sc > bestSc then best, bestSc = lbl, sc end
        end
        pctLbl = best
        if best then
            return math.clamp((tonumber(string.match(best.Text, "(%d+%.?%d*)")) or 0) / 100, 0, 1), best
        end
        return nil
    end

    local lastInfo = ""
    local lastFracBy = {}
    local function refresh()
            pcall(function()
                local char = player.Character
                local listed, diet = nil, nil
                if H.findMyDino then listed, diet = H.findMyDino() end
                local sp = listed
                if not sp then
                    local ok, gs = pcall(function() return H.getSpecies and H.getSpecies() end)
                    sp = (ok and gs) or (char and char.Name) or "-"
                end
                nameL.Text = "Dinosaur: " .. tostring(sp) .. (diet and ("  (" .. diet:sub(1, 1):upper() .. diet:sub(2) .. ")") or "")

                local stage, lbl = fromHud()
                local frac, fracHow = nil, nil
                local stageHow = lbl and ("HUD '" .. lbl.Text .. "'") or nil
                -- 1) the growth % the game shows (bottom bar), 2) a bar next to the stage text
                local pf, pl = growthPct()
                if pf then frac, fracHow = pf, "HUD growth '" .. pl.Text .. "' (" .. pl:GetFullName() .. ")" end
                if not frac and lbl then frac, fracHow = progressNear(lbl) end
                local vStage, vFrac, vHow = fromValues()
                if not stage and vStage then stage, stageHow = vStage, vHow end
                if not stage then
                    local mStage, mHow = fromModel()
                    if mStage then stage, stageHow = mStage, mHow end
                end

                -- remember the stage per dino and move it up when the % bar restarts (95% -> 3%)
                local spKey = tostring(sp)
                curSp = spKey
                if stage then
                    manualStage[spKey] = stage
                elseif manualStage[spKey] then
                    stage, stageHow = manualStage[spKey], "remembered / set by you"
                end
                if frac then
                    local prevF = lastFracBy[spKey]
                    if prevF and prevF >= 0.85 and frac <= 0.15 and manualStage[spKey] and not string.find(tostring(stageHow), "HUD", 1, true) then
                        manualStage[spKey] = math.min(manualStage[spKey] + 1, #STAGES)
                        stage = manualStage[spKey]
                    end
                    lastFracBy[spKey] = frac
                end
                if not frac and vFrac then frac, fracHow = vFrac, vHow end
                lastInfo = "Stage from: " .. tostring(stageHow or "nothing found") .. "\nProgress from: " .. tostring(fracHow or "nothing found")

                if not stage then
                    chainL.Text = "Baby > Juvenile > Adult > Elder"
                    if frac then
                        setBar(frac, string.format("Growth  %.1f%%  (press your stage below)", frac * 100))
                    else
                        setBar(0, "stage not detected")
                    end
                    nextL.Text = "Next stage: press your stage below"
                    elderL.Text = "To Elder: -"
                    srcL.Text = ""
                    return
                end

                -- stage chain: done = green, current = accent, upcoming = grey
                local parts = {}
                for i, name in ipairs(STAGES) do
                    if i < stage then
                        parts[#parts + 1] = '<font color="#5AFF8C">' .. name .. '</font>'
                    elseif i == stage then
                        parts[#parts + 1] = '<font color="#' .. Theme.Accent:ToHex() .. '"><b>' .. string.upper(name) .. '</b></font>'
                    else
                        parts[#parts + 1] = '<font color="#808080">' .. name .. '</font>'
                    end
                end
                chainL.Text = table.concat(parts, " > ")

                local now = os.clock()
                local key = tostring(sp) .. ":" .. stage
                if key ~= curKey then
                    -- stage changed: remember how long the last one took, reset the bar
                    local prev = curKey and stageStart[curKey]
                    if prev and curKey and string.sub(curKey, 1, #tostring(sp) + 1) == tostring(sp) .. ":" then
                        stageLen[curKey] = now - prev
                    end
                    curKey = key
                    stageStart[key] = stageStart[key] or now
                    samples, etaT = {}, {}
                end

                if stage >= #STAGES then
                    setBar(1, "ELDER - fully grown")
                    nextL.Text = "Next stage: none, you are an Elder"
                    elderL.Text = "To Elder: done"
                    srcL.Text = ""
                    return
                end

                if frac then
                    setBar(frac, string.format("%s  %.1f%%", STAGES[stage], frac * 100))
                else
                    setBar(0, STAGES[stage] .. "  (progress not shown by the game)")
                end

                -- speed of the CURRENT stage bar (averaged up to 5 minutes so the timer stays steady)
                local rate = nil
                if frac then
                    if samples[#samples] and frac < samples[#samples].f - 0.02 then samples = {} end
                    samples[#samples + 1] = {t = now, f = frac}
                    while #samples > 3 and now - samples[1].t > 300 do table.remove(samples, 1) end
                    local first, last = samples[1], samples[#samples]
                    if #samples >= 3 and last.t - first.t >= 10 and last.f > first.f then
                        rate = (last.f - first.f) / (last.t - first.t)
                    end
                end

                local nextName = STAGES[stage + 1]
                if rate then
                    local secsNext = (1 - frac) / rate
                    local left = stable("next", now, secsNext)
                    nextL.Text = string.format("To %s: %s  (at %s)", nextName, fmtClock(left), os.date("%H:%M", os.time() + math.floor(left)))
                    -- later stages: what they took for this dino before, otherwise the same speed as now
                    local total = secsNext
                    for st = stage + 1, #STAGES - 1 do
                        total = total + (stageLen[tostring(sp) .. ":" .. st] or (1 / rate))
                    end
                    local leftE = stable("elder", now, total)
                    elderL.Text = string.format("To Elder: ~%s  (at %s)", fmtClock(leftE), os.date("%H:%M", os.time() + math.floor(leftE)))
                else
                    nextL.Text = "To " .. nextName .. ": measuring..."
                    elderL.Text = "To Elder: measuring..."
                end
                srcL.Text = ""
            end)
    end

    -- instant: redraw the moment the game's growth % changes, plus a check every second
    local watchedPct, pctConn = nil, nil
    task.spawn(function()
        while running do
            refresh()
            if pctLbl ~= watchedPct then
                if pctConn then pctConn:Disconnect(); pctConn = nil end
                watchedPct = pctLbl
                if pctLbl then
                    pctConn = pctLbl:GetPropertyChangedSignal("Text"):Connect(refresh)
                    connections[#connections + 1] = pctConn
                end
            end
            task.wait(1)
        end
    end)

    infoBtn.Activated:Connect(function()
        infoOut.Text = lastInfo
        infoOut.Size = UDim2.new(1, 0, 0, 34)
        print("[Strawberry] growth detection:\n" .. lastInfo)
    end)
end)

----------------------------------------------------------------------
-- FOSSILS  (Main tab)
-- exact amount with commas. Looks in: your data values, the game's scripts (getgc),
-- and as a last resort the number shown on screen (may be rounded, marked with ~).
----------------------------------------------------------------------
;safeRun(function()
    local box = createGroupbox(Pages.Main.Right, "Fossils")
    local big = createLabel(box, "searching...", 30, true)
    big.TextSize = 24
    big.TextYAlignment = Enum.TextYAlignment.Center
    big.Font = Enum.Font.Code
    local gainL = createLabel(box, "Gained this session: +0", 16, true)
    local rateL = createLabel(box, "Per hour: -", 16, true)
    local avgL = createLabel(box, "Session average: -", 16)
    local lastL = createLabel(box, "Last gain: -", 16)
    local srcL = createLabel(box, "Source: -", 30)

    local function commas(n)
        local str = string.format("%.0f", n)
        local sign, digits = string.match(str, "^(-?)(%d+)$")
        if not digits then return str end
        local rev = string.reverse(digits)
        rev = string.gsub(rev, "(%d%d%d)", "%1,")
        rev = string.gsub(rev, ",$", "")
        return sign .. string.reverse(rev)
    end

    local function parse(v)
        if type(v) == "number" then return v, false end
        if type(v) ~= "string" then return nil end
        local clean = string.gsub(string.gsub(v, ",", ""), "%s", "")
        local n = tonumber(clean)
        if n then return n, false end
        local num, suf = string.match(clean, "^([%d%.]+)([KkMmBbTt])$")
        if num and tonumber(num) then
            return tonumber(num) * (({k = 1e3, m = 1e6, b = 1e9, t = 1e12})[string.lower(suf)]), true
        end
        return nil
    end

    -- names that are about fossils but are NOT your fossil count
    local NOT_COUNT = {"icon", "image", "price", "cost", "reward", "earn", "multi", "bonus", "boost", "last",
        "gain", "rate", "quest", "goal", "require", "need", "chance", "drop", "per", "spent", "bought", "max",
        "shop", "sell", "buy", "pack", "skin", "item"}
    local function isFossilName(n)
        local l = string.lower(n)
        if not string.find(l, "fossil", 1, true) then return false end
        for _, w in ipairs(NOT_COUNT) do
            if string.find(l, w, 1, true) then return false end
        end
        return true
    end

    -- 1) values / attributes
    local function fromInstances()
        local found = {}
        local function scanOne(inst, path)
            for n, v in pairs(inst:GetAttributes()) do
                if isFossilName(n) then
                    local num, approx = parse(v)
                    if num then found[#found + 1] = {n = num, approx = approx, src = path .. "@" .. n, name = n, inst = inst, attr = n} end
                end
            end
            if (inst:IsA("NumberValue") or inst:IsA("IntValue") or inst:IsA("StringValue")) and isFossilName(inst.Name) then
                local num, approx = parse(inst.Value)
                if num then found[#found + 1] = {n = num, approx = approx, src = path, name = inst.Name, inst = inst} end
            end
        end
        local function scanTree(root, label)
            scanOne(root, label)
            for _, d in ipairs(root:GetDescendants()) do scanOne(d, label .. "/" .. d.Name) end
        end
        scanOne(player, "Player")
        for _, c in ipairs(player:GetChildren()) do
            if c.Name ~= "PlayerGui" and c.Name ~= "PlayerScripts" and c.Name ~= "Backpack" then
                scanTree(c, "Player/" .. c.Name)
            end
        end
        local uid = tostring(player.UserId)
        pcall(function()
            for _, c in ipairs(game:GetService("ReplicatedStorage"):GetChildren()) do
                local x = (c.Name == player.Name or c.Name == uid) and c or c:FindFirstChild(player.Name) or c:FindFirstChild(uid)
                if x then scanTree(x, "ReplicatedStorage/" .. c.Name) end
            end
        end)
        -- prefer an exact, plainly named one
        table.sort(found, function(a, b)
            local la, lb = string.lower(a.name), string.lower(b.name)
            local pa = (la == "fossils" or la == "fossil") and 0 or 1
            local pb = (lb == "fossils" or lb == "fossil") and 0 or 1
            if pa ~= pb then return pa < pb end
            if a.approx ~= b.approx then return not a.approx end
            return a.n > b.n
        end)
        return found[1], found
    end

    -- 2) the game's own scripts (exact even when the screen shows 1.14M)
    local gcHit, gcNext = nil, 0
    local function dropGc() gcHit = nil; gcNext = 0 end
    local GC_KEYS = {"Fossils", "fossils", "Fossil", "fossil", "FossilCount", "TotalFossils"}
    local function fromGc()
        if gcHit then
            local v = rawget(gcHit.t, gcHit.k)
            if type(v) == "number" then return {n = v, approx = false, src = "game script data (" .. gcHit.k .. ")"} end
            gcHit = nil
        end
        if type(getgc) ~= "function" or os.clock() < gcNext then return nil end
        gcNext = os.clock() + 20
        task.spawn(function()
            local ok, list = pcall(getgc, true)
            if not ok or type(list) ~= "table" then return end
            local n = 0
            for _, t in ipairs(list) do
                n = n + 1
                if n % 5000 == 0 then task.wait() end
                if type(t) == "table" then
                    local ok2, hit = pcall(function()
                        for _, k in ipairs(GC_KEYS) do
                            local v = rawget(t, k)
                            if type(v) == "number" and v >= 0 and v % 1 == 0 then return {t = t, k = k} end
                        end
                        return nil
                    end)
                    if ok2 and hit then gcHit = hit; return end
                end
            end
        end)
        return nil
    end

    -- 3) the number shown on your screen (also used to check the other sources are not stale)
    local scrLbl, scrNext = nil, 0
    local function readLabel(d)
        local txt = string.gsub(d.Text, "<[^>]+>", "")
        local raw = string.match(txt, "[%d][%d,%.]*%s*[KkMmBbTt]?") or ""
        local num, approx = parse(raw)
        if not num then return nil end
        local tol = 0
        if approx then
            local dec = string.match(raw, "%.(%d+)")
            local unit = ({k = 1e3, m = 1e6, b = 1e9, t = 1e12})[string.lower(string.match(raw, "[KkMmBbTt]") or "k")]
            tol = 0.5 * (10 ^ -(dec and #dec or 0)) * unit + 1
        end
        return {n = num, approx = approx, tol = tol, src = "screen (" .. d.Name .. ")", lbl = d}
    end
    local function fromScreen()
        if scrLbl and scrLbl.Parent and scrLbl.Visible then
            local hit = readLabel(scrLbl)
            if hit then return hit end
        end
        if os.clock() < scrNext then return nil end
        scrNext = os.clock() + 2
        scrLbl = nil
        for _, g in ipairs(playerGui:GetChildren()) do
            if g:IsA("LayerCollector") and g.Enabled and not string.find(g.Name, "Strawberry", 1, true) then
                for _, d in ipairs(g:GetDescendants()) do
                    if d:IsA("TextLabel") and d.Visible then
                        local ctx = d.Name .. " " .. (d.Parent and d.Parent.Name or "")
                        if isFossilName(ctx) or (d.Parent and d.Parent:FindFirstChild("FossilIcon")) then
                            local hit = readLabel(d)
                            if hit then scrLbl = d; return hit end
                        end
                    end
                end
            end
        end
        return nil
    end

    -- "gained" only counts real increases while you play: the first reading, a switch of
    -- source, data still loading (0 -> your total) and spending never count
    local gained, firstT = 0, nil
    local exactLast, approxLast, lastExactT = nil, nil, -100
    local gainedApprox = false
    local gains = {}            -- {t, amount} for the live per-hour rate
    local lastGainAmt, lastGainT = nil, nil
    local WINDOW = 600          -- live rate looks at the last 10 minutes

    local function fmtAgo(sec)
        sec = math.floor(sec)
        if sec < 60 then return sec .. "s ago" end
        if sec < 3600 then return math.floor(sec / 60) .. "m " .. (sec % 60) .. "s ago" end
        return math.floor(sec / 3600) .. "h " .. math.floor((sec % 3600) / 60) .. "m ago"
    end

    local function refreshRates()
        local now = os.clock()
        gainL.Text = "Gained this session: +" .. (gainedApprox and "~" or "") .. commas(gained)
        H.fossilGained = gained
        H.fossilGainedTxt = "+" .. (gainedApprox and "~" or "") .. commas(gained)
        while gains[1] and now - gains[1].t > WINDOW do table.remove(gains, 1) end
        local recent = 0
        for _, g in ipairs(gains) do recent = recent + g.a end
        local span = firstT and math.min(WINDOW, now - firstT) or 0
        if recent > 0 and span >= 30 then
            rateL.Text = "Per hour: " .. commas(recent / span * 3600) .. "  (last " .. math.max(1, math.floor(span / 60)) .. " min)"
            H.fossilRateTxt = commas(recent / span * 3600) .. " / hour"
        elseif recent > 0 then
            rateL.Text = "Per hour: measuring..."
            H.fossilRateTxt = "measuring..."
        else
            rateL.Text = "Per hour: 0 (no gains in the last 10 min)"
            H.fossilRateTxt = "0 / hour"
        end
        local hrs = firstT and (now - firstT) / 3600 or 0
        avgL.Text = "Session average: " .. ((hrs > 0.01 and gained > 0) and (commas(gained / hrs) .. "/h") or "-")
        lastL.Text = lastGainT and string.format("Last gain: +%s (%s)", commas(lastGainAmt), fmtAgo(now - lastGainT)) or "Last gain: -"
    end

    local function update(hit)
        if not hit then
            big.Text = "not found"
            srcL.Text = "Source: no fossil value found yet"
            H.fossilDisplay = nil
            return
        end
        local shown = (hit.approx and "~" or "") .. commas(hit.n)
        big.Text = shown
        H.fossilDisplay = shown
        srcL.Text = "Source: " .. hit.src .. (hit.approx and "\n(rounded by the game, exact value not found)" or "")
        -- follow the NUMBER, whichever source it came from: any rise is a gain,
        -- a drop (spending) just moves the baseline, 0 -> total (data loading) never counts
        local function addGain(amt, approx)
            gained = gained + amt
            if approx then gainedApprox = true end
            gains[#gains + 1] = {t = os.clock(), a = amt}
            lastGainAmt, lastGainT = amt, os.clock()
            pcall(function()
                big.TextColor3 = Color3.fromRGB(90, 255, 140)
                task.delay(0.6, function() pcall(function() big.TextColor3 = Theme.Text end) end)
            end)
        end
        local now = os.clock()
        if hit.approx then
            -- rounded number (1.14M): only used for gains when no exact number is available
            if now - lastExactT > 10 then
                if approxLast and approxLast > 0 and hit.n > approxLast then addGain(hit.n - approxLast, true) end
                approxLast = hit.n
            end
        else
            lastExactT = now
            if exactLast and exactLast > 0 and hit.n > exactLast then addGain(hit.n - exactLast, false) end
            exactLast = hit.n
            approxLast = hit.n
        end
        if not firstT then firstT = now end
        refreshRates()
    end

    -- does a value match the fossil number on your screen (rounded numbers: within the rounding)
    local function agrees(hit, scr)
        if not hit or not scr then return true end
        return math.abs(hit.n - scr.n) <= (scr.approx and scr.tol or 0.5)
    end

    -- the number on your screen is the truth: a hidden value is only used when it MATCHES it
    -- (so it can be exact when the screen says 1.14M, and updates instantly), never a random other value
    local function read()
        local scr = fromScreen()
        local inst, all = fromInstances()
        if scr then
            for _, c in ipairs(all or {}) do
                if not c.approx and agrees(c, scr) then return c end
            end
            if not scr.approx then return scr end
            local gc = fromGc()
            if gc and agrees(gc, scr) then return gc end
            if gc then dropGc() end   -- old / wrong script data: look again later
            return scr
        end
        -- no fossil number on screen: best value we can find
        return inst or fromGc()
    end

    -- instant updates: listen to the fossil value itself so a gain shows the moment it happens
    local scrConn, scrWatched = nil, nil
    local function watchScreen()
        if scrLbl == scrWatched then return end
        if scrConn then scrConn:Disconnect(); scrConn = nil end
        scrWatched = scrLbl
        if scrLbl then
            scrConn = scrLbl:GetPropertyChangedSignal("Text"):Connect(function()
                pcall(function() update(read()) end)
            end)
            connections[#connections + 1] = scrConn
        end
    end

    local watchConn, watchKey = nil, nil
    local function watch(hit)
        local key = hit and hit.inst and (tostring(hit.inst) .. "@" .. tostring(hit.attr)) or nil
        if key == watchKey then return end
        if watchConn then watchConn:Disconnect(); watchConn = nil end
        watchKey = key
        if not key then return end
        local sig = hit.attr and hit.inst:GetAttributeChangedSignal(hit.attr) or hit.inst:GetPropertyChangedSignal("Value")
        watchConn = sig:Connect(function()
            pcall(function()
                local v = hit.attr and hit.inst:GetAttribute(hit.attr) or hit.inst.Value
                local num, approx = parse(v)
                if num then update({n = num, approx = approx, src = hit.src, inst = hit.inst, attr = hit.attr}) end
            end)
        end)
        connections[#connections + 1] = watchConn
    end

    task.spawn(function()
        local lastFull = 0
        while running do
            pcall(function()
                local now = os.clock()
                -- full search every 2s (finds the source), fast re-read of game script data in between
                if now - lastFull >= (watchKey and 4 or 1) then
                    lastFull = now
                    local hit = read()
                    watch(hit)
                    watchScreen()
                    update(hit)
                elseif not watchKey and (gcHit or scrLbl) then
                    -- script data / screen label are cheap to re-read, so check them fast
                    local scr = fromScreen()
                    local gc = gcHit and fromGc() or nil
                    if gc and not agrees(gc, scr) then dropGc(); gc = nil end
                    local hit = (scr and not scr.approx and scr) or gc or scr
                    if hit then update(hit) end
                else
                    refreshRates()
                end
            end)
            task.wait(0.25)
        end
        if watchConn then watchConn:Disconnect() end
    end)
end)

----------------------------------------------------------------------
-- AUTO RESPAWN  (Miscellaneous tab)
-- when you die it presses the respawn / play button the game shows you
----------------------------------------------------------------------
;safeRun(function()
    S.autoRespawn = false
    local box = createGroupbox(Pages.Miscellaneous.Left, "Respawn")
    createCheckbox(box, "Auto Respawn", false, function(v) S.autoRespawn = v end, "autoRespawn")
    local stL = createLabel(box, "Respawn: off", 30, true)

    local WORDS = {"respawn", "revive", "spawn", "play", "continue", "deploy", "try again", "retry", "hatch", "start"}
    local AVOID = {"buy", "purchase", "robux", "shop", "store", "close", "cancel", "quit", "leave", "skin", "gamepass",
        "premium", "sell", "trade", "delete", "reset", "settings", "code", "gift", "spectate"}
    local VIMr = nil
    pcall(function() VIMr = game:GetService("VirtualInputManager") end)

    local function isDead()
        local char = player.Character
        if not char or not char.Parent then return true end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health <= 0 then return true end
        local hp = char:GetAttribute("Health")
        if type(hp) == "number" and hp <= 0 then return true end
        if H.myHealth then
            local h2 = H.myHealth()
            if h2 and h2 <= 0 then return true end
        end
        return false
    end

    local function visibleNow(g)
        local o = g
        while o and o ~= playerGui do
            if o:IsA("GuiObject") and not o.Visible then return false end
            if o:IsA("LayerCollector") and not o.Enabled then return false end
            o = o.Parent
        end
        return o == playerGui
    end

    local function buttonText(b)
        local t = b.Name
        if b:IsA("TextButton") then t = t .. " " .. b.Text end
        for _, c in ipairs(b:GetDescendants()) do
            if c:IsA("TextLabel") then t = t .. " " .. c.Text end
        end
        return string.lower(t)
    end

    local function findButton()
        local best, bestRank = nil, math.huge
        for _, g in ipairs(playerGui:GetChildren()) do
            if g:IsA("LayerCollector") and g.Enabled and not string.find(g.Name, "Strawberry", 1, true) then
                for _, b in ipairs(g:GetDescendants()) do
                    if b:IsA("GuiButton") and b.Visible and b.AbsoluteSize.X > 4 and visibleNow(b) then
                        local t = buttonText(b)
                        local bad = false
                        for _, w in ipairs(AVOID) do
                            if string.find(t, w, 1, true) then bad = true; break end
                        end
                        if not bad then
                            for rank, w in ipairs(WORDS) do
                                if string.find(t, w, 1, true) and rank < bestRank then best, bestRank = b, rank; break end
                            end
                        end
                    end
                end
            end
        end
        return best
    end

    local function click(b, useMouse)
        local fired = false
        if not useMouse and type(getconnections) == "function" then
            for _, sig in ipairs({"MouseButton1Click", "Activated", "MouseButton1Down", "MouseButton1Up"}) do
                pcall(function()
                    for _, c in ipairs(getconnections(b[sig])) do
                        pcall(function() c:Fire() end)
                        fired = true
                    end
                end)
            end
        end
        if not fired and type(firesignal) == "function" and not useMouse then
            pcall(firesignal, b.MouseButton1Click)
            pcall(firesignal, b.Activated)
            fired = true
        end
        if (useMouse or not fired) and VIMr then
            local inset = GuiService:GetGuiInset()
            local sg = b:FindFirstAncestorWhichIsA("ScreenGui")
            local c = b.AbsolutePosition + b.AbsoluteSize / 2
            if sg and not sg.IgnoreGuiInset then c = c + inset end
            pcall(function()
                VIMr:SendMouseButtonEvent(c.X, c.Y, 0, true, game, 0)
                task.wait(0.05)
                VIMr:SendMouseButtonEvent(c.X, c.Y, 0, false, game, 0)
            end)
        end
    end

    task.spawn(function()
        local tries, deadSince = 0, nil
        while running do
            pcall(function()
                if not S.autoRespawn then
                    stL.Text = "Respawn: off"
                    tries, deadSince = 0, nil
                    return
                end
                if not isDead() then
                    if deadSince then stL.Text = "Respawn: back in game" end
                    if not deadSince then stL.Text = "Respawn: ready (alive)" end
                    tries, deadSince = 0, nil
                    return
                end
                deadSince = deadSince or os.clock()
                if os.clock() - deadSince < 0.6 then return end   -- let the death screen open
                local b = findButton()
                if b then
                    tries = tries + 1
                    -- every other try uses a real mouse click, in case the button ignores the direct press
                    click(b, tries % 2 == 0)
                    stL.Text = "Respawn: pressed '" .. (b:IsA("TextButton") and b.Text ~= "" and b.Text or b.Name) .. "' (" .. tries .. ")"
                else
                    stL.Text = "Respawn: dead, waiting for the respawn button..."
                end
            end)
            task.wait(0.7)
        end
    end)
end)

----------------------------------------------------------------------
-- WEBHOOK  (Autofarm tab) - Discord reports of your fossils
----------------------------------------------------------------------
;safeRun(function()
    S.webhookOn, S.webhookMins, S.webhookOnGain = false, 10, false
    local url = ""

    local box = createGroupbox(Pages.Autofarm.Right, "Webhook")
    createTextbox(box, "Discord Webhook URL", "https://discord.com/api/webhooks/...", "", function(t) url = trim(t) end, "webhookUrl")
    createCheckbox(box, "Send Fossil Reports", false, function(v) S.webhookOn = v end, "webhookOn")
    createSlider(box, "Report Every (minutes)", 1, 60, 10, "", function(v) S.webhookMins = v end, "webhookMins")
    createCheckbox(box, "Also Report When Fossils Go Up", false, function(v) S.webhookOnGain = v end, "webhookOnGain")
    local testBtn = createButton(box, "Send Test Report")
    local stL = createLabel(box, "Webhook: off", 30, true)

    local function getRequest(): any
        local g: any = (getgenv and getgenv()) or _G
        local syn = rawget(g, "syn")
        local http = rawget(g, "http")
        return (type(syn) == "table" and syn.request) or (type(http) == "table" and http.request)
            or rawget(g, "http_request") or rawget(g, "request")
    end

    local function validUrl(u)
        return string.match(u, "^https://[%w%.]*discord%.com/api/webhooks/") ~= nil
            or string.match(u, "^https://[%w%.]*discordapp%.com/api/webhooks/") ~= nil
    end

    local function territories()
        local owned = 0
        for _, z in ipairs(ZonesList) do
            if IsThisZoneGreen(z.Position) then owned = owned + 1 end
        end
        return owned .. " / " .. #ZonesList
    end

    local sending = false
    local msgT = 0
    local function send(reason)
        msgT = os.clock() + 4
        if sending then return end
        if not validUrl(url) then stL.Text = "Webhook: paste a Discord webhook URL first"; return end
        local req = getRequest()
        if type(req) ~= "function" then stL.Text = "Webhook: your executor has no request function"; return end
        sending = true
        task.spawn(function()
            local dino = nil
            if H.findMyDino then dino = H.findMyDino() end
            local fields = {
                {name = "Fossils", value = tostring(H.fossilDisplay or "not found"), inline = true},
                {name = "Gained this session", value = tostring(H.fossilGainedTxt or "+0"), inline = true},
                {name = "Making", value = tostring(H.fossilRateTxt or "measuring..."), inline = true},
                {name = "Territories", value = territories(), inline = true},
                {name = "Dino", value = tostring(dino or "-"), inline = true},
                {name = "Player", value = S.anon and "hidden" or shownName(), inline = true},
            }
            local payload = {
                username = "Strawberry Hub",
                embeds = {{
                    title = "Fossil report",
                    description = reason,
                    color = 0xFF466E,
                    fields = fields,
                    footer = {text = "Strawberry Hub " .. VERSION},
                    timestamp = DateTime.now():ToIsoDate(),
                }},
            }
            local ok, res = pcall(req, {
                Url = url, Method = "POST",
                Headers = {["Content-Type"] = "application/json"},
                Body = HttpService:JSONEncode(payload),
            })
            local code = ok and type(res) == "table" and (res.StatusCode or res.Status) or nil
            if ok and (code == nil or (tonumber(code) or 0) < 300) then
                stL.Text = "Webhook: sent (" .. reason .. ") at " .. os.date("%H:%M")
            else
                stL.Text = "Webhook: failed" .. (code and (" (" .. tostring(code) .. ")") or "")
            end
            sending = false
        end)
    end

    testBtn.Activated:Connect(function() send("Test report") end)

    task.spawn(function()
        local lastSend, lastGained, lastGainSend = os.clock(), nil, 0
        while running do
            task.wait(1)
            pcall(function()
                if not S.webhookOn then
                    if os.clock() - msgT > 8 then stL.Text = "Webhook: off" end
                    lastSend = os.clock()
                    return
                end
                local now = os.clock()
                if now - lastSend >= (S.webhookMins or 10) * 60 then
                    lastSend = now
                    send("Every " .. (S.webhookMins or 10) .. " min report")
                end
                local g = H.fossilGained
                if S.webhookOnGain and g and lastGained and g > lastGained and now - lastGainSend >= 20 then
                    lastGainSend = now
                    send("Fossils went up by " .. tostring(g - lastGained))
                    lastGained = g
                elseif g and (not lastGained or not S.webhookOnGain) then
                    lastGained = g
                end
            end)
        end
    end)
end)

----------------------------------------------------------------------
-- OWNER + USER TAGS (always on)
-- * Roblox ID 2038953612 is the owner: Owner badge in the menu, crown + "Owner" above their head
-- * everyone running Strawberry Hub plays a tiny invisible marker animation on their own dino
--   (animations are shared with the server), so other script users can see "Free User" above them
-- * you never see your own tag, only other people's
----------------------------------------------------------------------
;safeRun(function()
    local OWNER_ID = 2038953612
    local MARKER_ID = "507770453"   -- a Roblox-made animation, played at almost zero weight and zero speed
    local isOwner = player.UserId == OWNER_ID
    H.isOwner = isOwner

    -- ---------- for everyone except the owner: notification + the owner's identity is hidden ----------
    if not isOwner then
        local StarterGui = game:GetService("StarterGui")
        local function notify(title, text)
            task.spawn(function()
                for _ = 1, 10 do
                    local ok = pcall(function()
                        StarterGui:SetCore("SendNotification", {Title = title, Text = text, Duration = 8})
                    end)
                    if ok then return end
                    task.wait(0.5)
                end
            end)
        end

        -- a random profile picture for the owner (a random existing Roblox account, same for the whole session)
        local fakeThumb = "rbxthumb://type=AvatarHeadShot&id=1&w=150&h=150"
        task.spawn(function()
            for _ = 1, 6 do
                local id = math.random(100000, 2500000000)
                local ok = pcall(function() return Players:GetNameFromUserIdAsync(id) end)
                if ok then
                    fakeThumb = "rbxthumb://type=AvatarHeadShot&id=" .. id .. "&w=150&h=150"
                    return
                end
                task.wait(0.3)
            end
        end)

        local CoreGui = game:GetService("CoreGui")
        local ownerPlr = nil
        local hooked = setmetatable({}, {__mode = "k"})
        local function esc(x) return (string.gsub(x, "%W", "%%%0")) end

        local function cloakText(lbl)
            if not ownerPlr then return end
            local ok, cur = pcall(function() return lbl.Text end)
            if not ok or type(cur) ~= "string" or cur == "" then return end
            local new = cur
            for _, n in ipairs({ownerPlr.Name, ownerPlr.DisplayName}) do
                if n ~= "" then
                    new = string.gsub(new, "@" .. esc(n) .. "%f[^%w_]", "@Owner")
                    new = string.gsub(new, "%f[%w_]" .. esc(n) .. "%f[^%w_]", "Owner")
                end
            end
            if new ~= cur then pcall(function() lbl.Text = new end) end
        end

        local function cloakImage(img)
            if not ownerPlr then return end
            local ok, src = pcall(function() return img.Image end)
            if ok and type(src) == "string" and string.find(src, tostring(OWNER_ID), 1, true)
                and (string.find(src, "rbxthumb", 1, true) or string.find(string.lower(src), "thumbnail", 1, true)) then
                pcall(function() img.Image = fakeThumb end)
            end
        end

        local function hook(inst)
            if hooked[inst] then return end
            if inst:IsA("TextLabel") or inst:IsA("TextButton") or inst:IsA("TextBox") then
                hooked[inst] = inst:GetPropertyChangedSignal("Text"):Connect(function() cloakText(inst) end)
                cloakText(inst)
            elseif inst:IsA("ImageLabel") or inst:IsA("ImageButton") then
                hooked[inst] = inst:GetPropertyChangedSignal("Image"):Connect(function() cloakImage(inst) end)
                cloakImage(inst)
            end
        end

        local roots, addConns = {}, {}
        local function startCloak(plr)
            if ownerPlr then return end
            ownerPlr = plr
            roots = {CoreGui, playerGui}
            for _, root in ipairs(roots) do
                pcall(function()
                    addConns[#addConns + 1] = root.DescendantAdded:Connect(function(d)
                        if ownerPlr and not string.find(d:GetFullName(), "Strawberry", 1, true) then
                            task.defer(function() pcall(hook, d) end)
                        end
                    end)
                    task.spawn(function()
                        local n = 0
                        for _, d in ipairs(root:GetDescendants()) do
                            if not ownerPlr then return end
                            pcall(hook, d)
                            n = n + 1
                            if n % 2000 == 0 then task.wait() end
                        end
                    end)
                end)
            end
            -- name above the owner's head and name tags on their dino
            task.spawn(function()
                while running and ownerPlr == plr and plr.Parent do
                    pcall(function()
                        local c = plr.Character
                        if c then
                            local hum = c:FindFirstChildOfClass("Humanoid")
                            if hum and hum.DisplayDistanceType ~= Enum.HumanoidDisplayDistanceType.None then
                                hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
                            end
                            for _, d in ipairs(c:GetDescendants()) do
                                if d:IsA("TextLabel") then hook(d) end
                            end
                        end
                    end)
                    task.wait(2)
                end
            end)
        end
        local function stopCloak()
            ownerPlr = nil
            for _, cn in ipairs(addConns) do pcall(function() cn:Disconnect() end) end
            addConns = {}
            for inst, cn in pairs(hooked) do
                pcall(function() cn:Disconnect() end)
                hooked[inst] = nil
            end
        end

        local function ownerJoined(plr)
            if plr.UserId ~= OWNER_ID then return end
            startCloak(plr)
            notify("Strawberry Hub", "\u{1F451} The Strawberry Hub owner is in your server!")
        end
        for _, plr in ipairs(Players:GetPlayers()) do ownerJoined(plr) end
        connect(Players.PlayerAdded, ownerJoined)
        connect(Players.PlayerRemoving, function(plr)
            if plr.UserId == OWNER_ID then stopCloak() end
        end)
        H.ownerCloakStop = stopCloak
    end
    local GOLD = Color3.fromRGB(255, 200, 60)

    -- ---------- menu badge ----------
    local badge = Instance.new("TextLabel")
    badge.AnchorPoint = Vector2.new(1, 0.5)
    badge.Position = UDim2.new(1, -36, 0.5, 1)
    badge.AutomaticSize = Enum.AutomaticSize.X
    badge.Size = UDim2.fromOffset(0, 18)
    badge.BorderSizePixel = 0
    badge.Font = Enum.Font.GothamBold
    badge.TextSize = 12
    badge.Text = isOwner and "  \u{1F451} OWNER  " or "  FREE USER  "
    badge.Parent = top
    do
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 4)
        corner.Parent = badge
    end
    if isOwner then
        badge.BackgroundColor3 = GOLD
        badge.TextColor3 = Color3.fromRGB(35, 25, 0)
    else
        skin(badge, "BackgroundColor3", "Control")
        skin(badge, "TextColor3", "Text")
        addStroke(badge, "Border", 1)
    end
    title.Size = UDim2.new(1, -170, 1, 0)
    if H.rankL then
        H.rankL.Text = isOwner and "Rank: \u{1F451} Owner" or "Rank: Free User"
        if isOwner then H.rankL.TextColor3 = GOLD end
    end

    -- ---------- marker on your own dino ----------
    local markerTrack, markerChar = nil, nil
    local function ensureMarker()
        local char = player.Character
        if not char then return end
        if markerChar == char and markerTrack and markerTrack.IsPlaying then return end
        local animator = char:FindFirstChildWhichIsA("Animator", true)
        if not animator then return end
        local anim = Instance.new("Animation")
        anim.AnimationId = "rbxassetid://" .. MARKER_ID
        local ok, tr = pcall(function() return animator:LoadAnimation(anim) end)
        if ok and tr then
            tr.Priority = Enum.AnimationPriority.Core
            tr.Looped = true
            tr:Play(0, 0.001, 0)
            markerTrack, markerChar = tr, char
        end
    end

    -- ---------- spotting other users ----------
    local animCache = setmetatable({}, {__mode = "k"})   -- character -> its animators
    local function usesScript(plr)
        local c = plr.Character
        if not c then return false end
        local list = animCache[c]
        if not list or #list == 0 then
            list = {}
            for _, d in ipairs(c:GetDescendants()) do
                if d:IsA("Animator") then list[#list + 1] = d end
            end
            animCache[c] = list
        end
        for _, an in ipairs(list) do
            if an.Parent then
                for _, tr in ipairs(an:GetPlayingAnimationTracks()) do
                    local a = tr.Animation
                    if a and string.find(a.AnimationId, MARKER_ID, 1, true) then return true end
                end
            end
        end
        return false
    end

    -- ---------- tags above heads ----------
    local folder = Instance.new("Folder")
    folder.Name = "StrawberryTags"
    folder.Parent = playerGui
    local tags = {}

    local function makeTag(owner)
        local bb = Instance.new("BillboardGui")
        bb.AlwaysOnTop = true
        bb.LightInfluence = 0
        bb.ResetOnSpawn = false
        bb.Size = owner and UDim2.fromOffset(160, 52) or UDim2.fromOffset(140, 22)
        bb.MaxDistance = owner and 100000 or 600
        if owner then
            local crown = Instance.new("TextLabel")
            crown.Size = UDim2.new(1, 0, 0, 30)
            crown.BackgroundTransparency = 1
            crown.Text = "\u{1F451}"
            crown.TextSize = 28
            crown.Font = Enum.Font.GothamBold
            crown.Parent = bb
            local lbl = Instance.new("TextLabel")
            lbl.Size = UDim2.new(1, 0, 0, 20)
            lbl.Position = UDim2.fromOffset(0, 30)
            lbl.BackgroundTransparency = 1
            lbl.Text = "Owner"
            lbl.TextSize = 17
            lbl.Font = Enum.Font.GothamBold
            lbl.TextColor3 = GOLD
            lbl.TextStrokeTransparency = 0.2
            lbl.Parent = bb
        else
            local lbl = Instance.new("TextLabel")
            lbl.Size = UDim2.new(1, 0, 1, 0)
            lbl.BackgroundTransparency = 1
            lbl.Text = "Free User"
            lbl.TextSize = 14
            lbl.Font = Enum.Font.GothamBold
            lbl.TextColor3 = Color3.fromRGB(255, 120, 150)
            lbl.TextStrokeTransparency = 0.3
            lbl.Parent = bb
        end
        bb.Parent = folder
        return bb
    end

    local function dropTag(plr)
        local t = tags[plr]
        if t then
            pcall(function() t.bb:Destroy() end)
            tags[plr] = nil
        end
    end
    connect(Players.PlayerRemoving, dropTag)

    task.spawn(function()
        while running do
            pcall(ensureMarker)
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= player then   -- never your own tag
                    pcall(function()
                        local owner = plr.UserId == OWNER_ID
                        local show = owner or usesScript(plr)
                        local c = plr.Character
                        local root = c and (c:FindFirstChild("Head") or c:FindFirstChild("HumanoidRootPart") or c.PrimaryPart) :: any
                        if not show or not root then
                            dropTag(plr)
                            return
                        end
                        local t = tags[plr]
                        if not t or t.owner ~= owner or not t.bb.Parent then
                            dropTag(plr)
                            t = {bb = makeTag(owner), owner = owner}
                            tags[plr] = t
                        end
                        if t.bb.Adornee ~= root or t.char ~= c then
                            t.char = c
                            t.bb.Adornee = root
                            -- sit above the top of the dino (and above the ESP name tag)
                            local ok, cf, size = pcall(function() return c:GetBoundingBox() end)
                            local top = (ok and cf) and ((cf.Position.Y + size.Y / 2) - root.Position.Y) or 3
                            t.bb.StudsOffsetWorldSpace = Vector3.new(0, top + (owner and 5 or 4), 0)
                        end
                    end)
                end
            end
            task.wait(2)
        end
    end)

    H.tagCleanup = function()
        if markerTrack then pcall(function() markerTrack:Stop(0) end) end
        pcall(function() folder:Destroy() end)
    end
end)

----------------------------------------------------------------------
-- FPS BOOSTER  (Miscellaneous tab)
-- every option is off by default and puts everything back exactly when turned off
----------------------------------------------------------------------
;safeRun(function()
    local box = createGroupbox(Pages.Miscellaneous.Right, "FPS Booster")
    local stL

    -- ---------- helpers: remember originals, restore later ----------
    local saved = {}   -- feature -> {inst -> {prop -> value}}
    local function remember(feat, inst, prop)
        local f = saved[feat]
        if not f then f = setmetatable({}, {__mode = "k"}); saved[feat] = f end
        local e = f[inst]
        if not e then e = {}; f[inst] = e end
        if e[prop] == nil then e[prop] = inst[prop] end
    end
    local function set(feat, inst, prop, value)
        pcall(function()
            if inst[prop] ~= value then
                remember(feat, inst, prop)
                inst[prop] = value
            end
        end)
    end
    local function restore(feat)
        local f = saved[feat]
        if not f then return end
        for inst, props in pairs(f) do
            for prop, v in pairs(props) do
                pcall(function() inst[prop] = v end)
            end
        end
        saved[feat] = nil
    end

    -- ---------- the features ----------
    local PARTICLES = {"ParticleEmitter", "Trail", "Beam", "Smoke", "Fire", "Sparkles"}
    local function isParticle(d)
        for _, c in ipairs(PARTICLES) do
            if d:IsA(c) then return true end
        end
        return false
    end
    local function isMine(d)
        -- never touch Strawberry Hub's own parts (hitbox boxes, anchors)
        return string.find(d.Name, "Strawberry", 1, true) ~= nil
    end

    local function applyTo(d)
        if isMine(d) then return end
        if S.fpsTextures then
            if d:IsA("Decal") or d:IsA("Texture") then
                set("textures", d, "Transparency", 1)
            elseif d:IsA("BasePart") and not d:IsA("Terrain") then
                if d.Material ~= Enum.Material.SmoothPlastic then set("textures", d, "Material", Enum.Material.SmoothPlastic) end
                if d.Reflectance ~= 0 then set("textures", d, "Reflectance", 0) end
                if d:IsA("MeshPart") then set("textures", d, "TextureID", "") end
            elseif d:IsA("SurfaceAppearance") then
                set("textures", d, "Parent", nil)
            end
        end
        if S.fpsParticles and isParticle(d) then
            set("particles", d, "Enabled", false)
        end
        if S.fpsShadows and d:IsA("BasePart") and d.CastShadow then
            set("shadows", d, "CastShadow", false)
        end
    end

    -- go through the map a little at a time (never freezes the game)
    local sweepId = 0
    local function sweep()
        sweepId = sweepId + 1
        local id = sweepId
        task.spawn(function()
            local list = workspace:GetDescendants()
            for i, d in ipairs(list) do
                if id ~= sweepId or not running then return end
                pcall(applyTo, d)
                if i % 1500 == 0 then task.wait() end
            end
        end)
    end

    local addedConn = nil
    local function watchNew()
        local need = S.fpsTextures or S.fpsParticles or S.fpsShadows
        if need and not addedConn then
            addedConn = workspace.DescendantAdded:Connect(function(d)
                task.defer(function() pcall(applyTo, d) end)
            end)
            connections[#connections + 1] = addedConn
        elseif not need and addedConn then
            addedConn:Disconnect()
            addedConn = nil
        end
    end

    local function lightingEffects(on)
        if on then
            set("lighting", Lighting, "GlobalShadows", false)
            for _, d in ipairs(Lighting:GetDescendants()) do
                if d:IsA("PostEffect") then set("lighting", d, "Enabled", false) end
            end
            pcall(function()
                local atm = Lighting:FindFirstChildOfClass("Atmosphere")
                if atm then
                    set("lighting", atm, "Density", 0)
                    set("lighting", atm, "Haze", 0)
                    set("lighting", atm, "Glare", 0)
                end
            end)
        else
            restore("lighting")
        end
    end

    local function water(on)
        local t = workspace:FindFirstChildOfClass("Terrain")
        if not t then return end
        if on then
            set("water", t, "WaterWaveSize", 0)
            set("water", t, "WaterWaveSpeed", 0)
            set("water", t, "WaterReflectance", 0)
            set("water", t, "Decoration", false)
        else
            restore("water")
        end
    end

    local origQuality = nil
    local function quality(on)
        pcall(function()
            local r = (settings() :: any).Rendering
            if on then
                if origQuality == nil then origQuality = r.QualityLevel end
                r.QualityLevel = Enum.QualityLevel.Level01
            elseif origQuality ~= nil then
                r.QualityLevel = origQuality
                origQuality = nil
            end
        end)
    end

    local g: any = (getgenv and getgenv()) or _G
    local function fpsCap(n)
        local f = rawget(g, "setfpscap")
        if type(f) == "function" then pcall(f, n); return true end
        return false
    end

    -- ---------- UI ----------
    createCheckbox(box, "Low Graphics Quality", false, function(v) S.fpsQuality = v; quality(v) end, "fpsQuality")
    createCheckbox(box, "No Shadows & Light Effects", false, function(v)
        S.fpsShadows = v
        lightingEffects(v)
        if v then sweep() else restore("shadows") end
        watchNew()
    end, "fpsShadows")
    createCheckbox(box, "No Textures (smooth parts)", false, function(v)
        S.fpsTextures = v
        if v then sweep() else restore("textures") end
        watchNew()
    end, "fpsTextures")
    createCheckbox(box, "No Particles (smoke, fire, trails)", false, function(v)
        S.fpsParticles = v
        if v then sweep() else restore("particles") end
        watchNew()
    end, "fpsParticles")
    createCheckbox(box, "Simple Water", false, function(v) S.fpsWater = v; water(v) end, "fpsWater")
    createCheckbox(box, "Unlock FPS", false, function(v)
        S.fpsUnlock = v
        local ok = fpsCap(v and (S.fpsCap or 240) or 60)
        if stL then
            local msg = v and ("FPS cap: " .. (S.fpsCap or 240)) or "FPS cap: 60 (default)"
            stL.Text = ok and msg or "Your executor can not change the FPS cap"
        end
    end, "fpsUnlock")
    createSlider(box, "FPS Cap", 30, 360, 240, "", function(v)
        S.fpsCap = v
        if S.fpsUnlock then
            fpsCap(v)
            if stL then stL.Text = "FPS cap: " .. v end
        end
    end, "fpsCap")
    stL = createLabel(box, "Turn on what you need, everything comes back when you turn it off.", 30)

    H.fpsCleanup = function()
        sweepId = sweepId + 1
        if addedConn then addedConn:Disconnect(); addedConn = nil end
        for _, feat in ipairs({"textures", "particles", "shadows", "lighting", "water"}) do restore(feat) end
        quality(false)
        if S.fpsUnlock then fpsCap(60) end
    end
end)

----------------------------------------------------------------------
-- INF STAMINA  (Miscellaneous tab, one toggle)
-- * holds your stamina value at max (also stamina inside the game's scripts with getgc)
-- * never stops running: learns your dino's sprint speed + sprint key and forces the
--   sprint speed back when the server slows you down for having no stamina
-- * the game's stamina bar turns red and its number becomes an infinity sign
----------------------------------------------------------------------
;safeRun(function()
    S.infStamina = false
    local INF_RED = Color3.fromRGB(255, 40, 40)
    local INF_TEXT = "∞"

    local box = createGroupbox(Pages.Miscellaneous.Left, "Stamina")
    local restoreBar
    createCheckbox(box, "Inf Stamina", false, function(v)
        S.infStamina = v
        if not v and restoreBar then pcall(restoreBar) end
    end, "infStamina")
    local stL = createLabel(box, "Stamina: off", 44, true)

    ------------------------------------------------------------------
    -- stamina values
    ------------------------------------------------------------------
    local function isStam(n)
        local l = string.lower(n)
        if string.find(l, "max", 1, true) or string.find(l, "regen", 1, true) or string.find(l, "drain", 1, true)
            or string.find(l, "cost", 1, true) or string.find(l, "rate", 1, true) then return false end
        return string.find(l, "stamina", 1, true) ~= nil or string.find(l, "endurance", 1, true) ~= nil
    end

    local targets, gcTargets, seenMax = {}, {}, {}
    local lastScan, lastGc, scannedChar, gcChar = 0, -100, nil, false

    local function findMax(holder, name, kind)
        for _, mn in ipairs({"Max" .. name, name .. "Max", "Maximum" .. name, "Max" .. string.lower(name)}) do
            if kind == "attr" then
                local v = holder:GetAttribute(mn)
                if type(v) == "number" and v > 0 then return v end
            else
                local par = holder.Parent
                local v = par and par:FindFirstChild(mn)
                if v and (v:IsA("NumberValue") or v:IsA("IntValue")) and v.Value > 0 then return v.Value end
            end
        end
        return nil
    end

    local function scanInstances()
        targets = {}
        local char = player.Character
        local holders = {player, char}
        if char then
            holders[#holders + 1] = char:FindFirstChildOfClass("Humanoid")
            holders[#holders + 1] = char:FindFirstChild("HumanoidRootPart")
            for _, c in ipairs(char:GetChildren()) do
                if c:IsA("Folder") or c:IsA("Configuration") then holders[#holders + 1] = c end
            end
        end
        for _, c in ipairs(player:GetChildren()) do
            if c:IsA("Folder") or c:IsA("Configuration") then holders[#holders + 1] = c end
        end
        for _, h in ipairs(holders) do
            if h then
                for n, v in pairs(h:GetAttributes()) do
                    if type(v) == "number" and isStam(n) then
                        targets[#targets + 1] = {kind = "attr", holder = h, name = n, max = findMax(h, n, "attr")}
                    end
                end
                for _, d in ipairs(h:GetChildren()) do
                    if (d:IsA("NumberValue") or d:IsA("IntValue")) and isStam(d.Name) then
                        targets[#targets + 1] = {kind = "value", holder = d, name = d.Name, max = findMax(d, d.Name, "value")}
                    end
                end
            end
        end
    end

    local GC_KEYS = {"Stamina", "stamina", "CurrentStamina", "currentStamina", "StaminaValue"}
    local GC_MAX = {"MaxStamina", "maxStamina", "StaminaMax", "staminaMax", "MaximumStamina"}
    local function scanGc()
        if type(getgc) ~= "function" then return end
        task.spawn(function()
            local ok, list = pcall(getgc, true)
            if not ok or type(list) ~= "table" then return end
            local found, n = {}, 0
            for _, t in ipairs(list) do
                n = n + 1
                if n % 5000 == 0 then task.wait() end
                if type(t) == "table" then
                    pcall(function()
                        for _, k in ipairs(GC_KEYS) do
                            if type(rawget(t, k)) == "number" then
                                local mk = nil
                                for _, m in ipairs(GC_MAX) do
                                    if type(rawget(t, m)) == "number" then mk = m; break end
                                end
                                found[#found + 1] = {tbl = t, key = k, maxKey = mk}
                                break
                            end
                        end
                    end)
                    if #found >= 30 then break end
                end
            end
            gcTargets = found
        end)
    end

    local function maxFor(id, cur, known)
        if known and known > 0 then return known end
        seenMax[id] = math.max(seenMax[id] or 0, cur)
        local m = seenMax[id]
        if m <= 1 then return 1 end
        if m <= 100 then return 100 end
        return m
    end

    local function holdValues()
        for _, tg in ipairs(targets) do
            pcall(function()
                if tg.kind == "attr" then
                    local v = tg.holder:GetAttribute(tg.name)
                    if type(v) == "number" then
                        local m = maxFor(tg.holder:GetFullName() .. "@" .. tg.name, v, tg.max or findMax(tg.holder, tg.name, "attr"))
                        if v < m then tg.holder:SetAttribute(tg.name, m) end
                    end
                else
                    local v = tg.holder.Value
                    local m = maxFor(tg.holder:GetFullName(), v, tg.max)
                    if v < m then tg.holder.Value = m end
                end
            end)
        end
        for i, g in ipairs(gcTargets) do
            pcall(function()
                local v = rawget(g.tbl, g.key)
                if type(v) == "number" then
                    local known = g.maxKey and rawget(g.tbl, g.maxKey)
                    local m = maxFor("gc" .. i, v, type(known) == "number" and known or nil)
                    if v < m then rawset(g.tbl, g.key, m) end
                end
            end)
        end
    end

    -- exhaustion flags the game's own sprint script may read
    local function clearExhaustion()
        local char = player.Character
        for _, h in ipairs({player, char, char and char:FindFirstChildOfClass("Humanoid")}) do
            if h then
                for n, v in pairs(h:GetAttributes()) do
                    if type(v) == "boolean" then
                        local l = string.lower(n)
                        if (string.find(l, "exhaust", 1, true) or string.find(l, "tired", 1, true)
                            or string.find(l, "winded", 1, true) or string.find(l, "outofstamina", 1, true)
                            or string.find(l, "nostamina", 1, true)) and v then
                            h:SetAttribute(n, false)
                        elseif (string.find(l, "cansprint", 1, true) or string.find(l, "canrun", 1, true)) and not v then
                            h:SetAttribute(n, true)
                        end
                    end
                end
            end
        end
    end

    ------------------------------------------------------------------
    -- the game's stamina bar: red fill + infinity sign
    ------------------------------------------------------------------
    local bars = {}          -- {container, fills = {}, texts = {}, overlay}
    local origColor = setmetatable({}, {__mode = "k"})
    local lastBarScan = 0

    local FILL_WORDS = {"fill", "bar", "inner", "progress", "amount", "value", "meter", "current"}

    local function scanBars()
        local found = {}
        for _, g in ipairs(playerGui:GetChildren()) do
            if g:IsA("LayerCollector") and not string.find(g.Name, "Strawberry", 1, true) then
                for _, d in ipairs(g:GetDescendants()) do
                    if d:IsA("GuiObject") and string.find(string.lower(d.Name), "stamina", 1, true)
                        and not (d.Parent and string.find(string.lower(d.Parent.Name), "stamina", 1, true)) then
                        local e = {container = d, fills = {}, texts = {}}
                        local list = d:GetDescendants()
                        list[#list + 1] = d
                        for _, f in ipairs(list) do
                            if f:IsA("TextLabel") or f:IsA("TextBox") then
                                if f.Name ~= "StrawberryInf" then e.texts[#e.texts + 1] = f end
                            elseif (f:IsA("Frame") or f:IsA("ImageLabel")) and f ~= d then
                                local l = string.lower(f.Name)
                                for _, w in ipairs(FILL_WORDS) do
                                    if string.find(l, w, 1, true) then e.fills[#e.fills + 1] = f; break end
                                end
                            end
                        end
                        -- a bar with no named fill: its first child frame is the fill
                        if #e.fills == 0 then
                            for _, c in ipairs(d:GetChildren()) do
                                if c:IsA("Frame") or c:IsA("ImageLabel") then e.fills[#e.fills + 1] = c; break end
                            end
                        end
                        found[#found + 1] = e
                    end
                end
            end
        end
        -- keep overlays we already made, remove stray ones
        for _, old in ipairs(bars) do
            if old.overlay then
                local kept = false
                for _, e in ipairs(found) do
                    if e.container == old.container then e.overlay = old.overlay; kept = true end
                end
                if not kept then pcall(function() old.overlay:Destroy() end) end
            end
        end
        bars = found
    end

    local hiddenText = setmetatable({}, {__mode = "k"})   -- duplicate number labels we hid -> original transparency

    local function paintFills()
        for _, e in ipairs(bars) do
            pcall(function()
                if not e.container.Parent then return end
                for _, f in ipairs(e.fills) do
                    if f.Parent then
                        if not origColor[f] then
                            origColor[f] = {bg = f.BackgroundColor3, img = f:IsA("ImageLabel") and f.ImageColor3 or nil, size = f.Size}
                        end
                        if f.BackgroundTransparency < 1 and f.BackgroundColor3 ~= INF_RED then f.BackgroundColor3 = INF_RED end
                        if f:IsA("ImageLabel") and f.ImageColor3 ~= INF_RED then f.ImageColor3 = INF_RED end
                        local sz = f.Size
                        if sz.X.Scale > 0 and sz.X.Scale < 1 then
                            f.Size = UDim2.new(1, sz.X.Offset, sz.Y.Scale, sz.Y.Offset)
                        end
                        local grad = f:FindFirstChildOfClass("UIGradient")
                        if grad then grad.Enabled = false end
                    end
                end
            end)
        end
    end

    -- exactly ONE infinity sign: the front-most number label gets it, every other number label
    -- on the stamina UI (shadow copies, second bars) is hidden while Inf Stamina is on
    local function paintBars()
        paintFills()
        local labels = {}
        for _, e in ipairs(bars) do
            if e.container.Parent then
                for _, t in ipairs(e.texts) do
                    if t.Parent and (string.find(t.Text, "%d") or t.Text == INF_TEXT) then labels[#labels + 1] = t end
                end
            end
        end
        local primary = nil
        for _, t in ipairs(labels) do
            if not primary or t.ZIndex > primary.ZIndex
                or (t.ZIndex == primary.ZIndex and t.AbsoluteSize.Magnitude > primary.AbsoluteSize.Magnitude) then
                primary = t
            end
        end
        for _, t in ipairs(labels) do
            pcall(function()
                if t == primary then
                    if t.Text ~= INF_TEXT then t.Text = INF_TEXT end
                    if hiddenText[t] then
                        t.TextTransparency = hiddenText[t][1]
                        t.TextStrokeTransparency = hiddenText[t][2]
                        hiddenText[t] = nil
                    end
                else
                    if hiddenText[t] == nil then hiddenText[t] = {t.TextTransparency, t.TextStrokeTransparency} end
                    t.TextTransparency = 1
                    t.TextStrokeTransparency = 1
                end
            end)
        end

        -- the bar shows no number at all: add our own single infinity sign to the first bar
        for i, e in ipairs(bars) do
            local want = (primary == nil) and (i == 1) and e.container.Parent ~= nil
            if want then
                if not e.overlay or not e.overlay.Parent then
                    local o = Instance.new("TextLabel")
                    o.Name = "StrawberryInf"
                    o.Size = UDim2.new(1, 0, 1, 0)
                    o.BackgroundTransparency = 1
                    o.Text = INF_TEXT
                    o.TextScaled = true
                    o.Font = Enum.Font.GothamBold
                    o.TextColor3 = WHITE
                    o.TextStrokeTransparency = 0.4
                    o.ZIndex = 50
                    o.Parent = e.container
                    e.overlay = o
                end
            elseif e.overlay then
                pcall(function() e.overlay:Destroy() end)
                e.overlay = nil
            end
        end
    end

    restoreBar = function()
        for _, e in ipairs(bars) do
            if e.overlay then pcall(function() e.overlay:Destroy() end); e.overlay = nil end
            for _, f in ipairs(e.fills) do
                local o = origColor[f]
                if o then
                    pcall(function()
                        f.BackgroundColor3 = o.bg
                        if o.img then f.ImageColor3 = o.img end
                        local grad = f:FindFirstChildOfClass("UIGradient")
                        if grad then grad.Enabled = true end
                    end)
                    origColor[f] = nil
                end
            end
        end
        for t, tr in pairs(hiddenText) do
            pcall(function()
                t.TextTransparency = tr[1]
                t.TextStrokeTransparency = tr[2]
            end)
            hiddenText[t] = nil
        end
    end
    H.staminaCleanup = restoreBar

    ------------------------------------------------------------------
    -- never stop running
    ------------------------------------------------------------------
    local speeds = {}                      -- species -> {walk, sprint}
    local sprintKeys = {LeftShift = true}  -- learned from what you hold when the game speeds you up
    local MOVE = {W = true, A = true, S = true, D = true, Up = true, Down = true, Left = true, Right = true, Space = true}
    local forcing = false
    local sprintStatus = "sprint: -"

    local spCache, spChar, spT = "?", nil, 0
    local function speciesKey()
        local c = player.Character
        if c == spChar and os.clock() - spT < 3 then return spCache end
        spChar, spT = c, os.clock()
        local ok, sp = pcall(function() return H.getSpecies and H.getSpecies() end)
        if ok and sp and sp ~= "" then spCache = tostring(sp) else spCache = c and c.Name or "?" end
        return spCache
    end

    local function sprintHeld()
        for k in pairs(sprintKeys) do
            local ok, kc = pcall(function() return Enum.KeyCode[k] end)
            if ok and kc and UIS:IsKeyDown(kc) then return true end
        end
        return false
    end

    local function learnSprintKey()
        local held = {}
        local any = false
        for _, inp in ipairs(UIS:GetKeysPressed()) do
            local n = inp.KeyCode.Name
            if not MOVE[n] then held[n] = true; any = true end
        end
        if any then sprintKeys = held end
    end

    local lastWs = nil
    local function sprintStep()
        local hum = getHumanoid(player)
        if not hum or hum.Health <= 0 then forcing = false; return end
        local rec = speeds[speciesKey()]
        if not rec then rec = {}; speeds[speciesKey()] = rec end

        local moving = hum.MoveDirection.Magnitude > 0.1
        local ws = hum.WalkSpeed
        local otherFeature = S.speedEnabled or S.flyEnabled

        if moving and not forcing and not otherFeature and ws > 0 then
            if not rec.walk or ws < rec.walk then rec.walk = ws end
            if not rec.sprint or ws > rec.sprint then rec.sprint = ws end
            -- the game just sped you up: whatever you are holding is the sprint key
            if lastWs and rec.walk and ws > lastWs * 1.05 and ws > rec.walk * 1.05 then learnSprintKey() end
        end
        lastWs = ws

        if otherFeature then forcing = false; sprintStatus = "sprint: paused (Speed/Fly on)"; return end
        local target = (rec.sprint and rec.walk and rec.sprint > rec.walk * 1.05) and rec.sprint or nil
        if not target then
            forcing = false
            sprintStatus = "sprint: sprint once while you still have stamina"
            return
        end
        if moving and sprintHeld() then
            if ws < target then
                forcing = true
                hum.WalkSpeed = target
            end
            sprintStatus = string.format("sprint: holding %d (walk %d)", math.floor(target + 0.5), math.floor((rec.walk or 0) + 0.5))
        else
            if forcing and rec.walk and ws >= target - 0.01 then hum.WalkSpeed = rec.walk end
            forcing = false
            sprintStatus = string.format("sprint: ready (%d)", math.floor(target + 0.5))
        end
    end

    local wsConn, wsHum = nil, nil
    local function watchHumanoid()
        local hum = getHumanoid(player)
        if hum == wsHum then return end
        if wsConn then wsConn:Disconnect(); wsConn = nil end
        wsHum = hum
        if hum then
            wsConn = hum:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
                if forcing and S.infStamina then pcall(sprintStep) end
            end)
        end
    end

    ------------------------------------------------------------------
    -- loops
    ------------------------------------------------------------------
    local learnT = 0
    connect(RunService.Heartbeat, function()
        if not S.infStamina then
            forcing = false
            -- keep learning your walk / sprint speed a few times a second (cheap)
            local now = os.clock()
            if now - learnT > 0.25 then
                learnT = now
                pcall(watchHumanoid)
                pcall(sprintStep)
            end
            return
        end
        pcall(watchHumanoid)
        local now = os.clock()
        local char = player.Character
        if char ~= scannedChar or now - lastScan > 4 then
            scannedChar, lastScan = char, now
            pcall(scanInstances)
        end
        if char and (char ~= gcChar or (#gcTargets == 0 and #targets == 0 and now - lastGc > 20)) then
            gcChar, lastGc = char, now
            scanGc()
        end
        if now - lastBarScan > 3 then
            lastBarScan = now
            pcall(scanBars)
        end
        holdValues()
        pcall(clearExhaustion)
        pcall(sprintStep)
    end)
    connect(RunService.RenderStepped, function()
        if S.infStamina then
            holdValues()
            paintBars()
        end
    end)
    connect(RunService.Stepped, function()
        if S.infStamina and forcing then pcall(sprintStep) end
    end)

    task.spawn(function()
        while running do
            task.wait(0.5)
            pcall(function()
                if not S.infStamina then
                    stL.Text = "Stamina: off"
                    return
                end
                local vals = #targets + #gcTargets
                local barCount = 0
                for _, e in ipairs(bars) do if e.container.Parent then barCount = barCount + 1 end end
                stL.Text = string.format("Stamina: %s\nBar: %s\n%s",
                    vals > 0 and ("holding " .. vals .. " value(s) at max") or "value not found (sprint still works)",
                    barCount > 0 and "red + " .. INF_TEXT or "not found on screen",
                    sprintStatus)
            end)
        end
    end)
end)

----------------------------------------------------------------------
-- FOOD + FISH QUEST
-- Eats plants / meat at any distance (teleports there and back when needed)
-- and collects fish for fish quests.
----------------------------------------------------------------------
;safeRun(function()
    local RS = game:GetService("ReplicatedStorage")
    local ignoreUntil = setmetatable({}, {__mode = "k"})

    local function status(lbl, text)
        if lbl then pcall(function() lbl.Text = text end) end
    end

    local function eatRemote()
        local ok, r = pcall(function() return RS.Remotes.Character.Eat end)
        return ok and r or nil
    end

    local partCache = setmetatable({}, {__mode = "k"})
    local function partOf(obj)
        if obj:IsA("BasePart") then return obj end
        local c = partCache[obj]
        if c and c.Parent and c:IsDescendantOf(obj) then return c end
        if obj:IsA("Model") then
            c = obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart", true)
            partCache[obj] = c
            return c
        end
        return nil
    end

    local function inLiveCharacter(obj)
        for _, plr in ipairs(Players:GetPlayers()) do
            local c = plr.Character
            if c and (obj == c or obj:IsDescendantOf(c)) then return true end
        end
        return false
    end

    ------------------------------------------------------------------
    -- hunger (so it does not teleport around when you are already full)
    ------------------------------------------------------------------
    -- hunger as 0..1 (1 = full). Looks in the game's values first, then reads the hunger bar on your HUD
    local HUNGER_WORDS = {"hunger", "food", "appetite", "nutrition", "satiety", "stomach", "fullness", "meal"}
    local function hungerName(n)
        local l = string.lower(n)
        if string.find(l, "max", 1, true) or string.find(l, "rate", 1, true) or string.find(l, "type", 1, true)
            or string.find(l, "drain", 1, true) or string.find(l, "icon", 1, true) or string.find(l, "image", 1, true) then
            return false
        end
        for _, w in ipairs(HUNGER_WORDS) do
            if string.find(l, w, 1, true) then return true end
        end
        return false
    end

    local hungerSrc = "?"
    local function fromValues()
        local best = nil
        local function check(holder)
            if best or not holder then return end
            for n, v in pairs(holder:GetAttributes()) do
                if type(v) == "number" and hungerName(n) then
                    best = {v, holder:GetAttribute("Max" .. n) or holder:GetAttribute(n .. "Max"), holder.Name .. "@" .. n}
                    return
                end
            end
            for _, d in ipairs(holder:GetChildren()) do
                if (d:IsA("NumberValue") or d:IsA("IntValue")) and hungerName(d.Name) then
                    local m = holder:FindFirstChild("Max" .. d.Name) :: any
                    best = {d.Value, m and m.Value or nil, holder.Name .. "/" .. d.Name}
                    return
                end
            end
        end
        local char = player.Character
        check(char)
        check(char and char:FindFirstChildOfClass("Humanoid"))
        check(player)
        -- data folders under you, two levels deep (Stats, Data, Status ...)
        for _, c in ipairs(player:GetChildren()) do
            if c:IsA("Folder") or c:IsA("Configuration") then
                check(c)
                for _, c2 in ipairs(c:GetChildren()) do
                    if c2:IsA("Folder") or c2:IsA("Configuration") then check(c2) end
                end
            end
        end
        if char then
            for _, c in ipairs(char:GetChildren()) do
                if c:IsA("Folder") or c:IsA("Configuration") then check(c) end
            end
        end
        if not best then return nil end
        local v, mx = best[1], best[2]
        if not mx or mx <= 0 then mx = (v <= 1) and 1 or 100 end
        hungerSrc = best[3]
        return math.clamp(v / mx, 0, 1)
    end

    -- the hunger bar on screen: a frame named like hunger / food, read its fill or its number
    local hudBar, hudNext = nil, 0
    local findOrangeBar
    local function readBar(c)
        if not c.Parent or not c.Visible then return nil end
        -- "75/100" or "75"
        for _, d in ipairs(c:GetDescendants()) do
            if d:IsA("TextLabel") and d.Visible then
                local cur, mx = string.match(d.Text, "(%d+%.?%d*)%s*/%s*(%d+%.?%d*)")
                if cur then return math.clamp((tonumber(cur) or 0) / math.max(tonumber(mx) or 100, 1), 0, 1) end
                local pct = string.match(d.Text, "^%s*(%d+%.?%d*)%s*%%?%s*$")
                if pct then return math.clamp((tonumber(pct) or 0) / 100, 0, 1) end
            end
        end
        -- no number: use the fill bar width
        for _, d in ipairs(c:GetDescendants()) do
            if (d:IsA("Frame") or d:IsA("ImageLabel")) and d.Visible then
                local l = string.lower(d.Name)
                local par = d.Parent :: any
                if (string.find(l, "fill", 1, true) or string.find(l, "bar", 1, true) or string.find(l, "inner", 1, true)
                    or string.find(l, "progress", 1, true)) and par and par:IsA("GuiObject") and par.AbsoluteSize.X > 10 then
                    return math.clamp(d.AbsoluteSize.X / par.AbsoluteSize.X, 0, 1)
                end
            end
        end
        return nil
    end
    local orangeBar = nil
    local function isOrange(c)
        local h, sat, v = c:ToHSV()
        return h >= 0.03 and h <= 0.11 and sat >= 0.5 and v >= 0.6
    end
    local function orangeFrac(f)
        local par = f.Parent :: any
        if par and par:IsA("GuiObject") and par.AbsoluteSize.X > f.AbsoluteSize.X + 2 then
            return math.clamp(f.AbsoluteSize.X / par.AbsoluteSize.X, 0, 1)
        end
        return readBar(par or f) or 1
    end
    findOrangeBar = function()
        if orangeBar and orangeBar.Parent and orangeBar.Visible then return orangeBar end
        orangeBar = nil
        local best, bestW = nil, 0
        for _, g in ipairs(playerGui:GetChildren()) do
            if g:IsA("LayerCollector") and g.Enabled and not string.find(g.Name, "Strawberry", 1, true) then
                for _, d in ipairs(g:GetDescendants()) do
                    if (d:IsA("Frame") and d.BackgroundTransparency < 0.9 and isOrange(d.BackgroundColor3))
                        or (d:IsA("ImageLabel") and d.ImageTransparency < 0.9 and isOrange(d.ImageColor3) and d.Image ~= "") then
                        local w = d.AbsoluteSize.X
                        -- a bar: wide and flat, and actually on screen
                        if w > 30 and w > d.AbsoluteSize.Y * 2 and d.Visible and w > bestW then best, bestW = d, w end
                    end
                end
            end
        end
        orangeBar = best
        return best
    end

    local function fromHud()
        if orangeBar and orangeBar.Parent and orangeBar.Visible and not hudBar then
            return orangeFrac(orangeBar)
        end
        if hudBar then
            local f = readBar(hudBar)
            if f then return f end
        end
        if os.clock() < hudNext then return nil end
        hudNext = os.clock() + 3
        hudBar = nil
        for _, g in ipairs(playerGui:GetChildren()) do
            if g:IsA("LayerCollector") and g.Enabled and not string.find(g.Name, "Strawberry", 1, true) then
                for _, d in ipairs(g:GetDescendants()) do
                    if d:IsA("GuiObject") and hungerName(d.Name) then
                        local f = readBar(d)
                        if f then
                            hudBar = d
                            hungerSrc = "HUD '" .. d.Name .. "'"
                            return f
                        end
                    end
                end
            end
        end
        -- bars with no useful names: the hunger bar is the ORANGE one (health green, water blue, stamina yellow)
        local orange = findOrangeBar()
        if orange then
            hungerSrc = "HUD orange bar '" .. orange.Name .. "'"
            return orangeFrac(orange)
        end
        return nil
    end

    local function hungerFrac()
        local v = fromValues()
        if v then return v end
        local h = fromHud()
        if h then return h end
        hungerSrc = "?"
        return nil
    end
    H.hungerSource = function() return hungerSrc end

    ------------------------------------------------------------------
    -- object index (meat / fish), kept up to date instead of rescanning the map
    ------------------------------------------------------------------
    local MEAT_KEYS = {"meat", "carcass", "corpse", "steak", "flesh", "ribs", "drumstick", "deadbody", "remains"}
    local FISH_KEYS = {"fish", "salmon", "trout", "koi", "carp", "tuna", "piranha", "eel", "catfish"}
    local SKIP_KEYS = {"npc", "giver", "quest", "sign", "board", "spawn", "region", "zone", "area", "shop",
        "statue", "icon", "ui", "gui", "sound", "anim", "script", "fishing"}

    local PLANT_KEYS = {"bush", "herb", "shrub", "berry", "berries", "fern", "plant", "flower", "leafy", "tree",
        "fruit", "mushroom", "cactus", "palm", "grass", "crop", "veggie", "vegetable", "seaweed", "kelp"}
    local index = {meat = {}, fish = {}, plant = {}}
    local indexed = false

    local function kindOf(name)
        local l = string.lower(name)
        for _, k in ipairs(SKIP_KEYS) do
            if string.find(l, k, 1, true) then return nil end
        end
        for _, k in ipairs(MEAT_KEYS) do
            if string.find(l, k, 1, true) then return "meat" end
        end
        for _, k in ipairs(FISH_KEYS) do
            if string.find(l, k, 1, true) then return "fish" end
        end
        for _, k in ipairs(PLANT_KEYS) do
            if string.find(l, k, 1, true) then return "plant" end
        end
        return nil
    end

    local function consider(obj)
        if not (obj:IsA("Model") or obj:IsA("BasePart")) then return end
        local kind = kindOf(obj.Name)
        if not kind then return end
        -- only keep the outermost match (skip the parts inside a matched model)
        local par = obj.Parent
        if par and par ~= workspace and (par:IsA("Model") or par:IsA("BasePart")) and kindOf(par.Name) == kind then return end
        index[kind][obj] = true
    end

    H.meatIndex = index.meat
    H.partOf = partOf
    local function buildIndex()
        if indexed then return end
        indexed = true
        connect(workspace.DescendantAdded, function(obj)
            task.defer(function() pcall(consider, obj) end)
        end)
        connect(workspace.DescendantRemoving, function(obj)
            index.meat[obj] = nil
            index.fish[obj] = nil
            index.plant[obj] = nil
        end)
        task.spawn(function()
            local n = 0
            for _, obj in ipairs(workspace:GetDescendants()) do
                if not running then return end
                pcall(consider, obj)
                n = n + 1
                if n % 2500 == 0 then task.wait() end
            end
        end)
    end

    H.buildIndex = buildIndex
    local function nearest(set)
        local hrp = getRoot(player)
        if not hrp then return nil end
        local now = os.clock()
        local best, bestPart, bestD = nil, nil, math.huge
        local hp = hrp.Position
        for obj in pairs(set) do
            if not obj.Parent then
                set[obj] = nil
            elseif (ignoreUntil[obj] or 0) < now then
                local p = partOf(obj)
                if p then
                    local d = (p.Position - hp).Magnitude
                    -- the expensive "is this inside a player" check only for a new best
                    if d < bestD and not inLiveCharacter(obj) then best, bestPart, bestD = obj, p, d end
                end
            end
        end
        return best, bestPart, bestD
    end

    local function nearestTree()
        local hrp = getRoot(player)
        if not hrp then return nil end
        local closest, cPart, shortest = nil, nil, math.huge
        pcall(function()
            local env = workspace:FindFirstChild("MapEnvironment") :: any
            if not env then return end
            local folders = {
                { F = env:FindFirstChild("RedWood") and env.RedWood:FindFirstChild("Trees"), N = "RedWoodTree" },
                { F = env:FindFirstChild("Pines") and env.Pines:FindFirstChild("Trees"), N = "PineTree" },
                { F = env:FindFirstChild("Desert") and env.Desert:FindFirstChild("Tress"), N = "DesertTree" },
                { F = env:FindFirstChild("Lobby") and env.Lobby:FindFirstChild("Trees"), N = "RedWoodTree" },
            }
            local now = os.clock()
            for _, conf in ipairs(folders) do
                if conf.F then
                    for _, tree in ipairs(conf.F:GetChildren()) do
                        if tree.Name == conf.N and (ignoreUntil[tree] or 0) < now then
                            local p = partOf(tree)
                            if p then
                                local d = (hrp.Position - p.Position).Magnitude
                                if d < shortest then shortest, closest, cPart = d, tree, p end
                            end
                        end
                    end
                end
            end
        end)
        return closest, cPart, shortest
    end

    ------------------------------------------------------------------
    -- interacting: prompt, click detector, touch and the game's eat remote
    ------------------------------------------------------------------
    local hasPrompt = type(fireproximityprompt) == "function"
    local hasClick = type(fireclickdetector) == "function"
    local hasTouch = type(firetouchinterest) == "function"
    -- typed as 'any': executors take 0 / 1 here, some editors expect true / false
    local touchFn: any = firetouchinterest

    local VIMe = nil
    pcall(function() VIMe = game:GetService("VirtualInputManager") end)
    local GRAB_WORDS = {"grab", "carry", "pick", "drag", "hold", "take", "lift"}
    local EAT_WORDS = {"eat", "consume", "feed", "devour", "bite"}

    -- the "eat" prompt (E) - never the "grab" one (G)
    local function isEatPrompt(pp)
        local txt = string.lower(pp.ActionText .. " " .. pp.ObjectText .. " " .. pp.Name)
        if pp.KeyboardKeyCode == Enum.KeyCode.G then return false end
        for _, w in ipairs(GRAB_WORDS) do
            if string.find(txt, w, 1, true) then return false end
        end
        if pp.KeyboardKeyCode == Enum.KeyCode.E then return true end
        for _, w in ipairs(EAT_WORDS) do
            if string.find(txt, w, 1, true) then return true end
        end
        return false
    end

    local function firePrompt(d)
        if hasPrompt then
            pcall(fireproximityprompt, d)
        else
            pcall(function()
                d:InputHoldBegin()
                task.delay(d.HoldDuration + 0.05, function() pcall(function() d:InputHoldEnd() end) end)
            end)
        end
    end

    -- food: only the eat prompt and the eat remote. If the food has no prompt we can use, press E.
    local function _eatOnly(target)
        local list = target:GetDescendants()
        list[#list + 1] = target
        local used = false
        for _, d in ipairs(list) do
            if d:IsA("ProximityPrompt") and d.Enabled and isEatPrompt(d) then
                firePrompt(d)
                used = true
            end
        end
        local r = eatRemote()
        if r then pcall(function() r:FireServer(target) end) end
        if not used and VIMe then
            local hasAnyPrompt = false
            for _, d in ipairs(list) do
                if d:IsA("ProximityPrompt") then hasAnyPrompt = true; break end
            end
            if hasAnyPrompt then
                pcall(function()
                    VIMe:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                    task.wait(0.05)
                    VIMe:SendKeyEvent(false, Enum.KeyCode.E, false, game)
                end)
            end
        end
    end

    local function interact(target, useEat)
        local hrp = getRoot(player)
        local list = target:GetDescendants()
        list[#list + 1] = target
        local touched = 0
        for _, d in ipairs(list) do
            if d:IsA("ProximityPrompt") and d.Enabled then
                if hasPrompt then
                    pcall(fireproximityprompt, d)
                else
                    pcall(function()
                        d:InputHoldBegin()
                        task.delay(d.HoldDuration + 0.05, function() pcall(function() d:InputHoldEnd() end) end)
                    end)
                end
            elseif d:IsA("ClickDetector") and hasClick then
                pcall(fireclickdetector, d)
            elseif d:IsA("BasePart") and hasTouch and hrp and touched < 6 then
                touched = touched + 1
                pcall(touchFn, hrp, d, 0)
                pcall(touchFn, hrp, d, 1)
            end
        end
        if useEat then
            local r = eatRemote()
            if r then pcall(function() r:FireServer(target) end) end
        end
    end

    -- teleport to the target, interact for a moment, then go back
    local function _trip(target, part, seconds, offsetY, useEat, lbl, what)
        local hrp = getRoot(player)
        if not hrp or S.lowHpActive then return end
        S.tripBusy = true
        local back = hrp.CFrame
        status(lbl, what .. ": teleporting...")
        local t0 = os.clock()
        pcall(function()
            while running and os.clock() - t0 < seconds do
                if not target.Parent or not part.Parent or S.lowHpActive then break end
                hrp.CFrame = CFrame.new(part.Position + Vector3.new(0, offsetY, 0))
                hrp.AssemblyLinearVelocity = Vector3.new()
                interact(target, useEat)
                task.wait(0.15)
            end
        end)
        local gone = not target.Parent
        if S.returnAfter and not S.lowHpActive and hrp.Parent then
            hrp.CFrame = back
            hrp.AssemblyLinearVelocity = Vector3.new()
        end
        S.tripBusy = false
        if not gone then ignoreUntil[target] = os.clock() + 15 end
        return gone
    end

    -- INSTANT EAT: pull loose food to you, otherwise blink to it for a few frames
    -- with the camera frozen so your screen never moves
    local function canMove(part)
        if part.Anchored then return false end
        if type(isnetworkowner) == "function" then
            local ok, own = pcall(isnetworkowner, part)
            return ok and own
        end
        return false
    end

    local function _instantBite(target, part)
        local hrp = getRoot(player)
        if not hrp or S.lowHpActive then return false end
        if canMove(part) then
            pcall(function()
                for _ = 1, 4 do
                    if not target.Parent then break end
                    part.CFrame = hrp.CFrame * CFrame.new(0, 0, -4)
                    part.AssemblyLinearVelocity = Vector3.new()
                    interact(target, true)
                    RunService.Heartbeat:Wait()
                end
            end)
            if not target.Parent then return true end
        end
        S.tripBusy = true
        local cam = workspace.CurrentCamera
        local camType, camCF = cam.CameraType, cam.CFrame
        local back = hrp.CFrame
        pcall(function()
            cam.CameraType = Enum.CameraType.Scriptable
            cam.CFrame = camCF
            for i = 1, 6 do
                if not target.Parent or not part.Parent then break end
                hrp.CFrame = CFrame.new(part.Position + Vector3.new(0, 3, 0))
                hrp.AssemblyLinearVelocity = Vector3.new()
                if i >= 3 then interact(target, true) end
                RunService.Heartbeat:Wait()
            end
        end)
        pcall(function()
            hrp.CFrame = back
            hrp.AssemblyLinearVelocity = Vector3.new()
        end)
        pcall(function()
            cam.CameraType = camType
            cam.CFrame = camCF
        end)
        S.tripBusy = false
        return not target.Parent
    end

    ------------------------------------------------------------------
    -- DIET DETECTION: herbivore / carnivore / omnivore, per dino
    ------------------------------------------------------------------
    local dietCache = {}
    local function classifyDiet(v)
        local l = string.lower(tostring(v))
        if string.find(l, "omni", 1, true) then return "omnivore" end
        if string.find(l, "carn", 1, true) or string.find(l, "pisc", 1, true) or string.find(l, "meat", 1, true)
            or string.find(l, "predator", 1, true) then return "carnivore" end
        if string.find(l, "herb", 1, true) or string.find(l, "plant", 1, true) or string.find(l, "veg", 1, true) then return "herbivore" end
        return nil
    end
    local DIET_NAMES = {"diet", "type", "class", "eats", "foodtype", "category", "role"}
    local function dietFrom(holder)
        if not holder then return nil end
        for n, v in pairs(holder:GetAttributes()) do
            local l = string.lower(n)
            if type(v) == "string" then
                for _, k in ipairs(DIET_NAMES) do
                    if string.find(l, k, 1, true) then
                        local d = classifyDiet(v)
                        if d then return d end
                    end
                end
            elseif v == true then
                local d = classifyDiet(n)   -- e.g. IsCarnivore = true
                if d then return d end
            end
        end
        for _, c in ipairs(holder:GetChildren()) do
            if c:IsA("StringValue") then
                local l = string.lower(c.Name)
                for _, k in ipairs(DIET_NAMES) do
                    if string.find(l, k, 1, true) then
                        local d = classifyDiet(c.Value)
                        if d then return d end
                    end
                end
            elseif c:IsA("BoolValue") and c.Value then
                local d = classifyDiet(c.Name)
                if d then return d end
            end
        end
        return nil
    end

    -- Primeval Earth dinos (hybrids eat like carnivores, omnivores eat everything)
    local DIET_LIST = {
        acrocanthosaurus = "carnivore", allosaurus = "carnivore", cameroceras = "carnivore",
        carnotaurus = "carnivore", cryolophosaurus = "carnivore", deinosuchus = "carnivore",
        giganotosaurus = "carnivore", irritator = "carnivore", kelenken = "carnivore",
        microraptor = "carnivore", monolophosaurus = "carnivore", mosasaurus = "carnivore",
        otodusmegalodon = "carnivore", megalodon = "carnivore", pteranodon = "carnivore",
        pycnonemosaurus = "carnivore", quetzalcoatlus = "carnivore", spinosaurus = "carnivore",
        tarbosaurus = "carnivore", ubirajarajubatus = "carnivore", ubirajara = "carnivore",
        tyrantraptor = "carnivore", albinotyrant = "carnivore", bauruspinus = "carnivore",
        yuqivenator = "carnivore", kharnivore = "carnivore",
        apatosaurus = "herbivore", edmontosaurus = "herbivore", triceratops = "herbivore",
        parasaurolophus = "herbivore", plateosaurus = "herbivore", therizinosaurus = "herbivore",
        argentinosaurus = "herbivore",
        deinocheirus = "omnivore", dodo = "omnivore", gallimimus = "omnivore",
    }

    local function norm(v) return (string.gsub(string.lower(tostring(v or "")), "[^%a]", "")) end

    -- exact name first, then the longest dino name hidden inside it (skins like "Albino Spinosaurus")
    local function dietFromList(name)
        local n = norm(name)
        if n == "" then return nil end
        if DIET_LIST[n] then return DIET_LIST[n], n end
        local best, bestKey = nil, ""
        for key, d in pairs(DIET_LIST) do
            if #key > #bestKey and string.find(n, key, 1, true) then best, bestKey = d, key end
        end
        return best, best and bestKey or nil
    end

    -- every string on your dino / player that could be its name (species values, model name, skin name)
    local function listDiet(char, sp)
        local d, key = dietFromList(sp)
        if d then return d, key end
        d, key = dietFromList(char.Name)
        if d then return d, key end
        for _, h in ipairs({char, char:FindFirstChildOfClass("Humanoid"), player}) do
            if h then
                for _, v in pairs(h:GetAttributes()) do
                    if type(v) == "string" then
                        d, key = dietFromList(v)
                        if d then return d, key end
                    end
                end
                for _, c in ipairs(h:GetChildren()) do
                    if c:IsA("StringValue") then
                        d, key = dietFromList(c.Value)
                        if d then return d, key end
                    end
                end
            end
        end
        return nil
    end

    local function _detectDiet()
        local char = player.Character
        if not char then return nil, "no dino" end
        local ok, sp = pcall(function() return H.getSpecies and H.getSpecies() end)
        sp = (ok and sp) or char.Name
        local c = dietCache[sp]
        if c and (c.d or os.clock() - c.t < 10) then return c.d, c.name or sp end
        local listed, key = listDiet(char, sp)
        if listed then
            local pretty = key and (key:sub(1, 1):upper() .. key:sub(2)) or sp
            dietCache[sp] = {d = listed, t = os.clock(), name = pretty}
            return listed, pretty
        end
        local d = dietFrom(char) or dietFrom(char:FindFirstChildOfClass("Humanoid")) or dietFrom(player)
        if not d then
            -- the game's dino list: something named like your dino that says its diet
            pcall(function()
                for _, x in ipairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
                    if x.Name == sp then
                        d = dietFrom(x)
                        if d then break end
                    end
                end
            end)
        end
        dietCache[sp] = {d = d, t = os.clock()}
        return d, sp
    end

    local function _nearestPlant()
        local t, p, d = nearestTree()
        local b, bp, bd = nearest(index.plant)
        if b and (not d or bd < d) then return b, bp, bd end
        return t, p, d
    end

    ------------------------------------------------------------------
    -- FISHING
    -- 1) Record One Catch: records what the game sends while you catch a fish by hand,
    --    then replays it instantly to farm (skips the minigame on the server side).
    -- 2) Minigame solver: clicks / holds for you when a fishing minigame is on screen.
    -- 3) Fish lying in the water get grabbed directly.
    -- 4) Fish-catching quests are tracked and stop the farm when complete.
    ------------------------------------------------------------------
    local FISH_FILE = DIR .. "/fishing.json"
    local VIM = nil
    pcall(function() VIM = game:GetService("VirtualInputManager") end)
    local caught = 0

    -- ---------- record / replay ----------
    local recording, recList, recStart = false, {}, 0
    local saved = nil
    if hasFS then
        pcall(function()
            if isfile(FISH_FILE) then
                local d = HttpService:JSONDecode(readfile(FISH_FILE))
                if type(d) == "table" and type(d.steps) == "table" and #d.steps > 0 then saved = d end
            end
        end)
    end

    local function ser(v, depth)
        local t = typeof(v)
        if t == "string" or t == "number" or t == "boolean" then return {t = "v", v = v} end
        if t == "Vector3" then return {t = "V3", v = {v.X, v.Y, v.Z}} end
        if t == "CFrame" then return {t = "CF", v = {v:GetComponents()}} end
        if t == "EnumItem" then return {t = "E", e = tostring(v.EnumType), n = v.Name} end
        if t == "Instance" then return {t = "I", p = v:GetFullName(), n = v.Name} end
        if t == "table" and (depth or 0) < 3 then
            local o = {}
            for k, x in pairs(v) do
                if type(k) == "string" or type(k) == "number" then o[#o + 1] = {k = k, x = ser(x, (depth or 0) + 1)} end
            end
            return {t = "T", v = o}
        end
        return {t = "nil"}
    end

    local function resolvePath(path)
        local cur = game
        for seg in string.gmatch(path, "[^%.]+") do
            local nxt = cur:FindFirstChild(seg)
            if not nxt and cur == game then pcall(function() nxt = game:GetService(seg) end) end
            if not nxt then return nil end
            cur = nxt
        end
        return cur
    end

    local function deser(x)
        if type(x) ~= "table" then return nil end
        if x.t == "v" then return x.v end
        if x.t == "V3" then return Vector3.new(table.unpack(x.v)) end
        if x.t == "CF" then return CFrame.new(table.unpack(x.v)) end
        if x.t == "E" then
            local ok, e = pcall(function() return Enum[x.e][x.n] end)
            return ok and e or nil
        end
        if x.t == "I" then
            local inst = resolvePath(x.p)
            if not inst and x.n then
                -- the fish from the recording is gone: use the nearest object with the same name
                local hrp = getRoot(player)
                local best, bd = nil, math.huge
                for obj in pairs(index.fish) do
                    if obj.Parent and obj.Name == x.n then
                        local pp = partOf(obj)
                        local d = (pp and hrp) and (pp.Position - hrp.Position).Magnitude or 0
                        if d < bd then best, bd = obj, d end
                    end
                end
                inst = best
            end
            return inst
        end
        if x.t == "T" then
            local o = {}
            for _, e in ipairs(x.v) do o[e.k] = deser(e.x) end
            return o
        end
        return nil
    end

    local NOISE = {"mouse", "camera", "look", "ping", "heartbeat", "analytic", "sound", "chat", "replicat",
        "sprint", "stamina", "attack", "bite", "claw", "damage", "cursor", "aim", "emote", "log"}

    local ring = {}   -- last few seconds of what the game sent (so the "start fishing" step is included)
    local bufferOn = false
    local autoRec = false

    local function recordCall(remote, method, args, at)
        local l = string.lower(remote.Name)
        for _, w in ipairs(NOISE) do
            if string.find(l, w, 1, true) then return end
        end
        local list = {}
        for i = 1, args.n do list[i] = ser(args[i], 0) end
        local e = {path = remote:GetFullName(), name = remote.Name, method = method, n = args.n, args = list, at = at}
        ring[#ring + 1] = e
        while #ring > 40 or (ring[1] and at - ring[1].at > 4) do table.remove(ring, 1) end
        if recording then
            e.t = at - recStart
            recList[#recList + 1] = e
        end
    end

    local function saveRecording()
        if #recList == 0 then return false end
        local t0 = recList[1].at or 0
        for _, e in ipairs(recList) do e.t = (e.at or t0) - t0 end
        saved = {steps = recList}
        if hasFS then
            pcall(ensureFolders)
            pcall(function() writefile(FISH_FILE, HttpService:JSONEncode(saved)) end)
        end
        return true
    end

    local hookOk = false
    if type(hookmetamethod) == "function" and type(getnamecallmethod) == "function" then
        hookOk = pcall(function()
            local wrap = (type(newcclosure) == "function") and newcclosure or function(f) return f end
            local isOurs = (type(checkcaller) == "function") and checkcaller or function() return false end
            local old
            old = hookmetamethod(game, "__namecall", wrap(function(self, ...)
                if (recording or bufferOn) and running and not isOurs() then
                    local m = getnamecallmethod()
                    if (m == "FireServer" or m == "InvokeServer") and typeof(self) == "Instance" then
                        local args = table.pack(...)
                        local at = os.clock()
                        task.defer(function() pcall(recordCall, self, m, args, at) end)
                    end
                end
                return old(self, ...)
            end))
        end)
    end

    local function describeSaved()
        if not saved then return "Recording: none. Press Record One Catch, catch one fish by hand, then press Stop." end
        local names, seen = {}, {}
        for _, st in ipairs(saved.steps) do
            if not seen[st.name] then
                seen[st.name] = true
                names[#names + 1] = st.name
            end
        end
        return string.format("Recording: %d step(s) [%s]", #saved.steps, table.concat(names, ", "))
    end
    status(H.fishRecL, describeSaved())

    if H.fishRecBtn then
        H.fishRecBtn.Activated:Connect(function()
            if not hookOk then
                status(H.fishRecL, "Your executor can not record (no hookmetamethod). The minigame solver still works.")
                return
            end
            if not recording then
                recording, recList, recStart = true, {}, os.clock()
                H.fishRecBtn.Text = "Stop Recording"
                status(H.fishRecL, "RECORDING: catch one fish by hand now, then press Stop.")
            else
                recording = false
                H.fishRecBtn.Text = "Record One Catch"
                task.wait(0.1)
                if not saveRecording() then
                    status(H.fishRecL, "Nothing was sent while recording. Try again and catch a fish fully.")
                    return
                end
                status(H.fishRecL, describeSaved())
            end
        end)
    end
    if H.fishClearBtn then
        H.fishClearBtn.Activated:Connect(function()
            saved = nil
            if hasFS then pcall(function() if isfile(FISH_FILE) then writefile(FISH_FILE, "{}") end end) end
            status(H.fishRecL, describeSaved())
        end)
    end

    local function replay()
        if not saved then return false end
        local lastT = 0
        for _, st in ipairs(saved.steps) do
            if not running then return false end
            local gap = math.max((st.t or 0) - lastT, 0)
            lastT = st.t or 0
            if S.fishRealSpeed then task.wait(gap) elseif gap > 0 then task.wait(math.min(gap, 0.05)) end
            local remote = resolvePath(st.path)
            if remote then
                local args = {}
                for i = 1, (st.n or #st.args) do args[i] = deser(st.args[i]) end
                if st.method == "InvokeServer" then
                    local done = false
                    task.spawn(function()
                        pcall(function() remote:InvokeServer(table.unpack(args, 1, st.n or #st.args)) end)
                        done = true
                    end)
                    local t0 = os.clock()
                    while not done and os.clock() - t0 < 3 do task.wait() end
                else
                    pcall(function() remote:FireServer(table.unpack(args, 1, st.n or #st.args)) end)
                end
            end
        end
        return true
    end

    -- ---------- minigame solver ----------
    local MG_WORDS = {"fish", "reel", "catch", "minigame", "hook", "bait", "rod", "tension", "struggle", "cast"}
    local AVOID = {"close", "exit", "cancel", "quit", "leave", "back", "sell", "shop", "buy", "inventory"}
    local MARKER = {"marker", "pointer", "needle", "cursor", "indicator", "arrow", "fishicon", "fish", "line", "slider"}
    local ZONE = {"zone", "target", "green", "area", "sweet", "safe", "bar", "catchbar", "player"}
    local mgNext, mgRoots = 0, {}
    local lastZonePos, holding = nil, false

    local function wordIn(l, list)
        for _, w in ipairs(list) do
            if string.find(l, w, 1, true) then return w end
        end
        return nil
    end

    local function visibleNow(g)
        local o = g
        while o and o ~= playerGui do
            if o:IsA("GuiObject") and not o.Visible then return false end
            if o:IsA("LayerCollector") and not o.Enabled then return false end
            o = o.Parent
        end
        return o == playerGui
    end

    local function findMinigame()
        local now = os.clock()
        if now >= mgNext then
            mgNext = now + 2
            mgRoots = {}
            for _, g in ipairs(playerGui:GetChildren()) do
                if g:IsA("LayerCollector") and not string.find(g.Name, "Strawberry", 1, true) then
                    if wordIn(string.lower(g.Name), MG_WORDS) then
                        mgRoots[#mgRoots + 1] = g
                    else
                        for _, d in ipairs(g:GetDescendants()) do
                            if d:IsA("Frame") and wordIn(string.lower(d.Name), MG_WORDS) then
                                mgRoots[#mgRoots + 1] = d
                            end
                        end
                    end
                end
            end
        end
        for _, r in ipairs(mgRoots) do
            if r.Parent and visibleNow(r) then
                -- needs something interactive or moving inside to count as a minigame
                for _, d in ipairs(r:GetDescendants()) do
                    if (d:IsA("GuiButton") or d:IsA("Frame") or d:IsA("ImageLabel")) and d.Visible then
                        local l = string.lower(d.Name)
                        if d:IsA("GuiButton") or wordIn(l, MARKER) or wordIn(l, ZONE) then return r end
                    end
                end
            end
        end
        return nil
    end

    local function screenPos(g)
        local inset = GuiService:GetGuiInset()
        local sg = g:FindFirstAncestorWhichIsA("ScreenGui")
        local c = g.AbsolutePosition + g.AbsoluteSize / 2
        if sg and not sg.IgnoreGuiInset then c = c + inset end
        return c
    end

    local function mouse(down, at)
        if not VIM then return end
        local pos = at or UIS:GetMouseLocation()
        pcall(function() VIM:SendMouseButtonEvent(pos.X, pos.Y, 0, down, game, 0) end)
    end

    local function pressButton(btn)
        local fired = false
        if type(getconnections) == "function" then
            for _, sig in ipairs({"MouseButton1Click", "Activated", "MouseButton1Down"}) do
                pcall(function()
                    for _, c in ipairs(getconnections(btn[sig])) do
                        pcall(function() c:Fire() end)
                        fired = true
                    end
                end)
            end
        elseif type(firesignal) == "function" then
            pcall(firesignal, btn.MouseButton1Click)
            pcall(firesignal, btn.Activated)
            fired = true
        end
        if not fired then
            local c = screenPos(btn)
            mouse(true, c)
            mouse(false, c)
        end
    end

    local function overlap(a, b, horizontal)
        local ap, as, bp, bs = a.AbsolutePosition, a.AbsoluteSize, b.AbsolutePosition, b.AbsoluteSize
        if horizontal then
            return ap.X + as.X / 2 >= bp.X and ap.X + as.X / 2 <= bp.X + bs.X
        end
        return ap.Y + as.Y / 2 >= bp.Y and ap.Y + as.Y / 2 <= bp.Y + bs.Y
    end

    local function solve(root)
        local markers, zones, buttons = {}, {}, {}
        for _, d in ipairs(root:GetDescendants()) do
            if d:IsA("GuiObject") and d.Visible and d.AbsoluteSize.X > 0 then
                local l = string.lower(d.Name)
                if d:IsA("GuiButton") then
                    local txt = d:IsA("TextButton") and string.lower(d.Text) or ""
                    if not wordIn(l, AVOID) and not wordIn(txt, AVOID) and txt ~= "x" then buttons[#buttons + 1] = d end
                elseif wordIn(l, ZONE) then
                    zones[#zones + 1] = d
                elseif wordIn(l, MARKER) then
                    markers[#markers + 1] = d
                end
            end
        end

        -- "keep the fish inside your bar" or "click when the marker is in the zone"
        local marker, zone = markers[1], zones[1]
        if marker and zone then
            local horizontal = zone.Parent and zone.Parent.AbsoluteSize.X >= zone.Parent.AbsoluteSize.Y
            local zp = zone.AbsolutePosition
            local zoneMoves = lastZonePos and (zp - lastZonePos).Magnitude > 0.5
            lastZonePos = zp
            if zoneMoves or holding then
                -- you steer the bar: hold to move it toward the fish, release to let it fall back
                local mc = marker.AbsolutePosition + marker.AbsoluteSize / 2
                local zc = zone.AbsolutePosition + zone.AbsoluteSize / 2
                local want = horizontal and (mc.X > zc.X) or (mc.Y < zc.Y)
                if want ~= holding then
                    holding = want
                    mouse(want)
                end
                return "steering"
            elseif overlap(marker, zone, horizontal) then
                if #buttons > 0 then pressButton(buttons[1]) else mouse(true); mouse(false) end
                pcall(function()
                    VIM:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
                    VIM:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
                end)
                return "timed click"
            end
            return "waiting for the zone"
        end

        -- plain "spam click to reel" minigames
        if #buttons > 0 then
            for _, b in ipairs(buttons) do pressButton(b) end
            return "clicking"
        end
        mouse(true)
        mouse(false)
        return "clicking"
    end

    -- ---------- fish quests ----------
    local CATCH_WORDS = {"catch", "caught", "collect", "hunt", "eat", "grab", "gather", "bring", "get", "kill"}
    local qCache, qNext = {}, 0

    local function readFishQuest()
        local now = os.clock()
        if now >= qNext then
            qNext = now + ((S.autoFish or S.fishFarm) and 1 or 4)
            qCache = {}
            for _, g in ipairs(playerGui:GetChildren()) do
                if not string.find(g.Name, "Strawberry", 1, true) then
                    for _, d in ipairs(g:GetDescendants()) do
                        if d:IsA("TextLabel") or d:IsA("TextButton") then qCache[#qCache + 1] = d end
                    end
                end
            end
        end
        local best, bestCur, bestMax = nil, nil, nil
        for _, lbl in ipairs(qCache) do
            local ok, txt = pcall(function()
                if not lbl.Parent or not visibleNow(lbl) then return nil end
                return lbl.Text
            end)
            if ok and txt and #txt > 0 and #txt < 140 then
                local plain = string.gsub(txt, "<[^>]+>", "")
                local l = string.lower(plain)
                if string.find(l, "fish", 1, true) and not string.find(l, "fishing rod", 1, true) then
                    local cur, mx = string.match(plain, "(%d+)%s*/%s*(%d+)")
                    if not cur and lbl.Parent then
                        for _, sib in ipairs(lbl.Parent:GetChildren()) do
                            if sib ~= lbl and (sib:IsA("TextLabel") or sib:IsA("TextButton")) then
                                local c2, m2 = string.match(sib.Text, "(%d+)%s*/%s*(%d+)")
                                if c2 then cur, mx = c2, m2; break end
                            end
                        end
                    end
                    if wordIn(l, CATCH_WORDS) or cur then
                        if cur and not bestCur then
                            best, bestCur, bestMax = plain, tonumber(cur), tonumber(mx)
                        elseif not best then
                            best = plain
                        end
                    end
                end
            end
        end
        return best, bestCur, bestMax
    end

    -- grabs fish objects lying in the water
    local function grabFishObjects(needed)
        local hrp = getRoot(player)
        if not hrp then return 0 end
        buildIndex()
        if not next(index.fish) then return 0 end
        S.tripBusy = true
        local back = hrp.CFrame
        local got = 0
        pcall(function()
            for _ = 1, math.clamp(needed or 5, 1, 30) do
                if not running or S.lowHpActive then break end
                local fish, fpart, fd = nearest(index.fish)
                if not fish then break end
                local t0 = os.clock()
                while running and os.clock() - t0 < 0.6 do
                    if not fish.Parent or not fpart.Parent then break end
                    if fd > 12 then
                        hrp.CFrame = CFrame.new(fpart.Position)
                        hrp.AssemblyLinearVelocity = Vector3.new()
                    end
                    interact(fish, true)
                    task.wait(0.05)
                end
                if fish.Parent then ignoreUntil[fish] = os.clock() + 15 else got = got + 1 end
                local _, cur, mx = readFishQuest()
                if cur and mx and cur >= mx then break end
            end
        end)
        if hrp.Parent and not S.lowHpActive then
            hrp.CFrame = back
            hrp.AssemblyLinearVelocity = Vector3.new()
        end
        S.tripBusy = false
        return got
    end

    -- ---------- loops ----------
    -- minigame watcher: auto-records the first catch, and solves it when asked
    local mgWasUp = false
    task.spawn(function()
        while running do
            local up = false
            local ok = pcall(function()
                local watching = S.autoMinigame or S.fishFarm or S.autoFishing or (S.autoEat and H.eatsMeatNow)
                -- only listen to what the game sends while there is still something to learn
                bufferOn = watching and not saved
                if not watching then return end
                local root = findMinigame()
                up = root ~= nil

                -- learn: the mini-quest just opened and nothing is recorded yet
                if up and not mgWasUp and not saved and not recording and hookOk then
                    autoRec, recording = true, true
                    recStart = os.clock() - 3
                    recList = {}
                    for _, e in ipairs(ring) do
                        if os.clock() - e.at <= 3 then
                            e.t = e.at - recStart
                            recList[#recList + 1] = e
                        end
                    end
                    status(H.fishRecL, "Learning: finish this fishing mini-quest once...")
                end
                -- it closed: keep what was sent as the recording
                if not up and mgWasUp and autoRec then
                    autoRec = false
                    task.wait(0.4)
                    recording = false
                    if saveRecording() then
                        status(H.fishRecL, "Learned! " .. describeSaved())
                    else
                        status(H.fishRecL, "Could not learn that one, finish the next mini-quest again.")
                    end
                end
                mgWasUp = up

                if up and (S.autoMinigame or S.autoEat or S.autoFishing) then
                    local what = solve(root)
                    status(H.fishStatus, "Fish: minigame - " .. what)
                elseif not up and holding then
                    holding = false
                    mouse(false)
                    lastZonePos = nil
                end
            end)
            -- fast only while a minigame is on screen
            task.wait((ok and up) and 0.03 or 0.3)
        end
        if holding then mouse(false) end
    end)

    -- ---------- AUTO FISHING: press "Start Fishing", the solver above does the minigame ----------
    local fishPrompts, promptsIndexed = setmetatable({}, {__mode = "k"}), false
    local function isFishPrompt(pp)
        local t = string.lower(pp.ActionText .. " " .. pp.ObjectText .. " " .. pp.Name)
        return string.find(t, "fish", 1, true) ~= nil and not string.find(t, "sell", 1, true)
    end
    local function indexPrompts()
        if promptsIndexed then return end
        promptsIndexed = true
        connect(workspace.DescendantAdded, function(d)
            if d:IsA("ProximityPrompt") then
                task.defer(function() if isFishPrompt(d) then fishPrompts[d] = true end end)
            end
        end)
        task.spawn(function()
            local n = 0
            for _, d in ipairs(workspace:GetDescendants()) do
                if d:IsA("ProximityPrompt") and isFishPrompt(d) then fishPrompts[d] = true end
                n = n + 1
                if n % 4000 == 0 then task.wait() end
            end
        end)
    end

    local function nearestFishPrompt()
        local hrp = getRoot(player)
        if not hrp then return nil end
        local best, bestPart, bestD = nil, nil, math.huge
        for pp in pairs(fishPrompts) do
            if pp.Parent and pp.Enabled then
                local holder = pp.Parent :: any
                local part = holder:IsA("BasePart") and holder or (holder:IsA("Attachment") and holder.Parent)
                    or holder:FindFirstChildWhichIsA("BasePart", true)
                if part and part:IsA("BasePart") then
                    local d = (part.Position - hrp.Position).Magnitude
                    if d < bestD then best, bestPart, bestD = pp, part, d end
                end
            else
                fishPrompts[pp] = nil
            end
        end
        return best, bestPart, bestD
    end

    -- a "Start Fishing" button the game shows on screen
    local startBtn, startNext = nil, 0
    local function findStartButton()
        if startBtn and startBtn.Parent and visibleNow(startBtn) then return startBtn end
        if os.clock() < startNext then return nil end
        startNext = os.clock() + 1
        startBtn = nil
        for _, g in ipairs(playerGui:GetChildren()) do
            if g:IsA("LayerCollector") and g.Enabled and not string.find(g.Name, "Strawberry", 1, true) then
                for _, b in ipairs(g:GetDescendants()) do
                    if b:IsA("GuiButton") and b.Visible then
                        local t = string.lower(b.Name .. " " .. (b:IsA("TextButton") and b.Text or ""))
                        for _, c in ipairs(b:GetChildren()) do
                            if c:IsA("TextLabel") then t = t .. " " .. string.lower(c.Text) end
                        end
                        if string.find(t, "fish", 1, true) and (string.find(t, "start", 1, true) or string.find(t, "cast", 1, true))
                            and visibleNow(b) then
                            startBtn = b
                            return b
                        end
                    end
                end
            end
        end
        return nil
    end

    task.spawn(function()
        local lastStart, starts = 0, 0
        while running do
            pcall(function()
                if not S.autoFishing or S.lowHpActive then return end
                indexPrompts()
                if findMinigame() then
                    status(H.fishStatus, "Fish: minigame on screen, playing it")
                    return
                end
                if os.clock() - lastStart < 2 then return end

                -- 1) a Start Fishing button on screen
                local btn = findStartButton()
                if btn then
                    pressButton(btn)
                    lastStart, starts = os.clock(), starts + 1
                    status(H.fishStatus, "Fish: pressed Start Fishing (" .. starts .. ")")
                    return
                end

                -- 2) a Start Fishing prompt in the world
                local pp, part, d = nearestFishPrompt()
                if not pp then
                    status(H.fishStatus, "Fish: no Start Fishing spot found yet")
                    return
                end
                local hrp = getRoot(player)
                if hrp and S.fishGoTo and d > pp.MaxActivationDistance - 1 then
                    hrp.CFrame = CFrame.new(part.Position + Vector3.new(0, 3, 0))
                    hrp.AssemblyLinearVelocity = Vector3.new()
                    task.wait(0.2)
                end
                firePrompt(pp)
                lastStart, starts = os.clock(), starts + 1
                status(H.fishStatus, string.format("Fish: Start Fishing (%dm away, %d)", math.floor(d), starts))
            end)
            task.wait(0.25)
        end
    end)

    -- farm / quest loop
    task.spawn(function()
        while running do
            local ok = pcall(function()
                if not S.autoFish and not S.fishFarm then
                    if not S.autoMinigame then status(H.fishStatus, "Fish: idle") end
                    status(H.questStatus, "Fish quest: -")
                    task.wait(1)
                    return
                end
                local qText, cur, mx = readFishQuest()
                if qText then
                    status(H.questStatus, "Fish quest: " .. qText .. ((cur and not string.find(qText, "/", 1, true)) and (" (" .. cur .. "/" .. mx .. ")") or ""))
                else
                    status(H.questStatus, "Fish quest: none active")
                end
                local questDone = cur and mx and cur >= mx
                local questActive = S.autoFish and qText and not questDone
                local farming = S.fishFarm or questActive

                if not farming then
                    if not S.autoMinigame then
                        status(H.fishStatus, S.autoFish and (questDone and "Fish: quest complete!" or "Fish: waiting for a fish catch quest") or "Fish: idle")
                    end
                    task.wait(0.4)
                    return
                end
                if S.lowHpActive or S.tripBusy then task.wait(0.2); return end

                local did = false
                if saved then
                    status(H.fishStatus, "Fish: replaying catch (" .. caught .. " so far)")
                    if replay() then
                        caught = caught + 1
                        did = true
                    end
                end
                local got = grabFishObjects(questActive and (mx and cur) and (mx - cur) or 5)
                if got > 0 then
                    caught = caught + got
                    did = true
                end

                if did then
                    status(H.fishStatus, "Fish: " .. caught .. " caught this session")
                    task.wait(math.max(S.fishDelay or 1, 0.05))
                else
                    status(H.fishStatus, saved and "Fish: replay sent, waiting..." or "Fish: record one catch first, or turn on Auto Complete Minigame and fish by hand")
                    task.wait(1)
                end
            end)
            if not ok then task.wait(0.5) end
        end
    end)

    ------------------------------------------------------------------
    -- AUTO EAT: only presses E, very fast. The game decides what you eat,
    -- so it works the same for herbivores and carnivores and costs almost nothing.
    ------------------------------------------------------------------
    H.eatsMeatNow = false
    local PPS = game:GetService("ProximityPromptService")
    local shownE = {}
    connect(PPS.PromptShown, function(pp)
        if pp.KeyboardKeyCode == Enum.KeyCode.E then shownE[pp] = true end
    end)
    connect(PPS.PromptHidden, function(pp) shownE[pp] = nil end)
    local function ePromptShown()
        for pp in pairs(shownE) do
            if pp.Parent and pp.Enabled then return true end
            shownE[pp] = nil
        end
        return false
    end

    -- every "eat" prompt (E) on the map, kept up to date
    local eatPrompts, eatIndexed = setmetatable({}, {__mode = "k"}), false
    local drinkPrompts = setmetatable({}, {__mode = "k"})
    local origPrompt = setmetatable({}, {__mode = "k"})
    local function isDrinkPrompt(pp)
        local t = string.lower(pp.ActionText .. " " .. pp.ObjectText .. " " .. pp.Name)
        return string.find(t, "drink", 1, true) ~= nil or string.find(t, "water", 1, true) ~= nil
    end
    local function indexEat()
        if eatIndexed then return end
        eatIndexed = true
        connect(workspace.DescendantAdded, function(d)
            if d:IsA("ProximityPrompt") then
                task.defer(function()
                    if isEatPrompt(d) then eatPrompts[d] = true end
                    if isDrinkPrompt(d) then drinkPrompts[d] = true end
                end)
            end
        end)
        task.spawn(function()
            local n = 0
            for _, d in ipairs(workspace:GetDescendants()) do
                if d:IsA("ProximityPrompt") then
                    if isEatPrompt(d) then eatPrompts[d] = true end
                    if isDrinkPrompt(d) then drinkPrompts[d] = true end
                end
                n = n + 1
                if n % 4000 == 0 then task.wait() end
            end
        end)
    end

    -- hide a prompt's bubble (it still works with E / directly), or give it back
    local origStyle = setmetatable({}, {__mode = "k"})
    local function silence(pp, on)
        if on then
            if origStyle[pp] == nil then origStyle[pp] = pp.Style end
            if pp.Style ~= Enum.ProximityPromptStyle.Custom then pp.Style = Enum.ProximityPromptStyle.Custom end
        elseif origStyle[pp] ~= nil then
            pp.Style = origStyle[pp]
            origStyle[pp] = nil
        end
    end

    -- let food prompts work from your Eat Range (on your side), or put them back
    local function stretchPrompts(on)
        for pp in pairs(eatPrompts) do
            if pp.Parent then
                if on then
                    if not origPrompt[pp] then origPrompt[pp] = {pp.MaxActivationDistance, pp.RequiresLineOfSight} end
                    local want = math.max(origPrompt[pp][1], S.eatRange or 40)
                    if pp.MaxActivationDistance ~= want then pp.MaxActivationDistance = want end
                    if pp.RequiresLineOfSight then pp.RequiresLineOfSight = false end
                    silence(pp, true)
                elseif origPrompt[pp] then
                    pp.MaxActivationDistance = origPrompt[pp][1]
                    pp.RequiresLineOfSight = origPrompt[pp][2]
                    origPrompt[pp] = nil
                    silence(pp, false)
                end
            else
                eatPrompts[pp] = nil
            end
        end
    end

    local function promptPos(pp)
        local h = pp.Parent :: any
        if not h then return nil end
        if h:IsA("BasePart") then return h.Position end
        if h:IsA("Attachment") then return h.WorldPosition end
        local part = h:FindFirstChildWhichIsA("BasePart", true)
        return part and part.Position or nil
    end

    -- the closest food prompt inside your Eat Range
    local function nearestEat()
        local hrp = getRoot(player)
        if not hrp then return nil end
        local best, bestD = nil, S.eatRange or 40
        for pp in pairs(eatPrompts) do
            if pp.Parent and pp.Enabled then
                local pos = promptPos(pp)
                if pos then
                    local d = (pos - hrp.Position).Magnitude
                    if d <= bestD then best, bestD = pp, d end
                end
            end
        end
        return best, bestD
    end

    H.promptRestore = function()
        pcall(stretchPrompts, false)
        for pp in pairs(drinkPrompts) do pcall(silence, pp, false) end
    end

    task.spawn(function()
        local wasDrink = false
        while running do
            pcall(function()
                if S.autoDrink then
                    indexEat()
                    wasDrink = true
                    for pp in pairs(drinkPrompts) do
                        if pp.Parent then silence(pp, true) else drinkPrompts[pp] = nil end
                    end
                elseif wasDrink then
                    wasDrink = false
                    for pp in pairs(drinkPrompts) do
                        if pp.Parent then silence(pp, false) end
                    end
                end
            end)
            task.wait(0.5)
        end
    end)

    local g: any = (getgenv and getgenv()) or _G
    local kp, kr = rawget(g, "keypress"), rawget(g, "keyrelease")
    local function keyE(down)
        if VIMe then
            pcall(function() VIMe:SendKeyEvent(down, Enum.KeyCode.E, false, game) end)
        elseif type(kp) == "function" and type(kr) == "function" then
            pcall(down and kp or kr, 0x45)
        end
    end

    task.spawn(function()
        local presses, hungerT, hungerV, statusT = 0, 0, nil, 0
        local wasOn = false
        local target, targetD, scanT = nil, nil, 0
        while running do
            if not S.autoEat or S.lowHpActive then
                if wasOn then
                    keyE(false)
                    wasOn = false
                    pcall(stretchPrompts, false)
                end
                if not S.autoEat then status(H.foodStatus, "Food: idle") end
                presses = 0
                task.wait(0.25)
                continue
            end
            wasOn = true
            local now = os.clock()
            if now - scanT > 0.25 then
                scanT = now
                indexEat()
                pcall(stretchPrompts, true)
                local ok, t, d = pcall(nearestEat)
                if ok then target, targetD = t, d else target = nil end
            end
            if now - hungerT > 0.5 then
                hungerT = now
                local ok, v = pcall(hungerFrac)
                hungerV = ok and v or nil
            end
            local hungerTxt = hungerV and string.format("%d%%", math.floor(hungerV * 100 + 0.5)) or "? (not found, eating anyway)"
            if hungerV and hungerV * 100 >= S.eatBelow then
                if now - statusT > 0.5 then statusT = now; status(H.foodStatus, "Food: full (hunger " .. hungerTxt .. ")") end
                task.wait(0.25)
            elseif UIS:GetFocusedTextBox() ~= nil then
                task.wait(0.1)   -- never type E into chat / a text box
            elseif S.eatPromptOnly and not ePromptShown() and not target then
                if now - statusT > 0.5 then statusT = now; status(H.foodStatus, "Food: waiting for an E prompt") end
                task.wait(0.1)
            else
                -- eat the closest food directly (works without the prompt on screen) + press E
                if target and target.Parent then firePrompt(target) end
                keyE(true)
                task.wait()
                keyE(false)
                presses = presses + 1
                if now - statusT > 0.5 then
                    statusT = now
                    if target then
                        status(H.foodStatus, string.format("Food: eating (%dm away, %d)  |  hunger %s", math.floor(targetD or 0), presses, hungerTxt))
                    else
                        status(H.foodStatus, string.format("Food: pressing E (%d), no food prompt in range  |  hunger %s", presses, hungerTxt))
                    end
                end
                task.wait(math.max(1 / (S.eatRate or 25) - 1 / 60, 0))
            end
        end
        keyE(false)
    end)
end)

----------------------------------------------------------------------
-- MEAT ESP  (Visuals tab)
-- chams on the closest meat, and a small tag with the distance on every meat in range
----------------------------------------------------------------------
;safeRun(function()
    S.meatEsp, S.meatColor, S.meatMax, S.meatChams = false, Color3.fromRGB(255, 60, 60), 1500, 10

    local box = createGroupbox(Pages.Visuals.Left, "Meat ESP")
    createCheckbox(box, "Meat ESP", false, function(v) S.meatEsp = v end, "meatEsp")
    createColorRow(box, "Meat Color", S.meatColor, function(c) S.meatColor = c end, "meatColor")
    createSlider(box, "Max Distance", 100, 5000, 1500, "", function(v) S.meatMax = v end, "meatMax")
    createSlider(box, "Chams On Closest", 1, 15, 10, "", function(v) S.meatChams = v end, "meatChams")
    local infoL = createLabel(box, "Meat on map: -", 16, true)

    local folder = nil
    local function getFolder()
        if not folder or not folder.Parent then
            folder = Instance.new("Folder")
            folder.Name = "StrawberryMeatESP"
            folder.Parent = playerGui
        end
        return folder
    end

    local objs = {}   -- meat -> {bb, lbl, hl}
    local MAX_TAGS = 60

    local function drop(m)
        local o = objs[m]
        if o then
            pcall(function() o.bb:Destroy() end)
            if o.hl then pcall(function() o.hl:Destroy() end) end
            objs[m] = nil
        end
    end

    local function ensure(m, part)
        local o = objs[m]
        if o and o.bb.Parent then return o end
        if o then drop(m) end
        local bb = Instance.new("BillboardGui")
        bb.AlwaysOnTop = true
        bb.Size = UDim2.fromOffset(110, 16)
        bb.StudsOffset = Vector3.new(0, 2.5, 0)
        bb.ResetOnSpawn = false
        bb.LightInfluence = 0
        bb.Parent = getFolder()
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, 0, 1, 0)
        lbl.BackgroundTransparency = 1
        lbl.Font = FONT
        lbl.TextSize = 12
        lbl.TextStrokeTransparency = 0
        lbl.Parent = bb
        o = {bb = bb, lbl = lbl}
        objs[m] = o
        return o
    end

    task.spawn(function()
        while running do
            task.wait(0.25)
            pcall(function()
                if not S.meatEsp or not H.meatIndex then
                    if next(objs) then for m in pairs(objs) do drop(m) end end
                    infoL.Text = "Meat on map: -"
                    return
                end
                if H.buildIndex then H.buildIndex() end
                local root = getRoot(player)
                local list = {}
                local total = 0
                for m in pairs(H.meatIndex) do
                    if m.Parent and m:IsDescendantOf(workspace) then
                        total = total + 1
                        local part = H.partOf(m)
                        if part then
                            local d = root and (part.Position - root.Position).Magnitude or 0
                            if d <= S.meatMax then list[#list + 1] = {m = m, p = part, d = d} end
                        end
                    end
                end
                table.sort(list, function(a, b) return a.d < b.d end)

                local keep = {}
                for i, e in ipairs(list) do
                    if i > MAX_TAGS then break end
                    keep[e.m] = true
                    local o = ensure(e.m, e.p)
                    if o.bb.Adornee ~= e.p then o.bb.Adornee = e.p end
                    o.lbl.Text = string.format("Meat [%dm]", math.floor(e.d))
                    o.lbl.TextColor3 = S.meatColor
                    if i <= S.meatChams then
                        if not o.hl or not o.hl.Parent then
                            o.hl = Instance.new("Highlight")
                            o.hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                            o.hl.Parent = getFolder()
                        end
                        if o.hl.Adornee ~= e.m then o.hl.Adornee = e.m end
                        o.hl.FillColor = S.meatColor
                        o.hl.OutlineColor = WHITE
                        o.hl.FillTransparency = 0.45
                        o.hl.OutlineTransparency = 0
                    elseif o.hl then
                        pcall(function() o.hl:Destroy() end)
                        o.hl = nil
                    end
                end
                for m in pairs(objs) do
                    if not keep[m] then drop(m) end
                end
                infoL.Text = string.format("Meat on map: %d  (in range: %d)", total, #list)
            end)
        end
        for m in pairs(objs) do drop(m) end
    end)

    H.meatCleanup = function()
        for m in pairs(objs) do drop(m) end
        if folder then pcall(function() folder:Destroy() end) end
    end
end)

task.spawn(function()
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    while running do
        if S.autoDrink then
            pcall(function()
                ReplicatedStorage.Remotes.Character.CharacterFunctions:InvokeServer("Drink")
            end)
            task.wait(0.05)
        else
            task.wait(0.1)
        end
    end
end)

H.unload = function()
    if not running then return end
    running = false

    S.flyEnabled, S.speedEnabled, S.jumpEnabled, S.noclip = false, false, false, false
    pcall(stopFly)
    local hum = getHumanoid(player)
    if hum then
        pcall(function()
            hum.WalkSpeed = S.savedWalkSpeed
            hum.JumpPower = S.savedJump
        end)
    end

    S.anon = false
    pcall(function() if H.coreAnon then H.coreAnon(false) end end)
    pcall(function() if H.anonRestore then H.anonRestore() end end)

    pcall(function() if H.stopSpectating then H.stopSpectating() end end)
    pcall(H.camRestore)
    pcall(function()
        local cam = workspace.CurrentCamera
        if cam and S.fovEnabled then cam.FieldOfView = 70 end
    end)
    S.fullbright, S.noFog = false, false
    pcall(H.lightCleanup)
    pcall(H.mapCleanup)
    pcall(H.espCleanup)
    pcall(H.zoneCleanup)
    pcall(function() if H.attackCleanup then H.attackCleanup() end end)
    pcall(function() if H.zoneStop then H.zoneStop() end end)
    pcall(function() if H.meatCleanup then H.meatCleanup() end end)
    pcall(function() if H.tagCleanup then H.tagCleanup() end end)
    pcall(function() if H.promptRestore then H.promptRestore() end end)
    pcall(function() if H.fpsCleanup then H.fpsCleanup() end end)
    pcall(function() if H.ownerCloakStop then H.ownerCloakStop() end end)
    pcall(function() if H.staminaCleanup then H.staminaCleanup() end end)

    for _, c in ipairs(connections) do pcall(function() c:Disconnect() end) end

    pcall(function() gui:Destroy() end)
    pcall(function() pickerGui:Destroy() end)
    pcall(function() espFolder:Destroy() end)
    pcall(function() loader:Destroy() end)

    for _, env in ipairs({genv, _G}) do
        if env.StrawberryUnload == H.unload then env.StrawberryUnload = nil end
    end
end

genv.StrawberryUnload = H.unload
genv.StrawberryVersion = VERSION
_G.StrawberryUnload = H.unload
