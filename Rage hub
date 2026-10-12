--// Servicios de Roblox
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local HttpService = game:GetService("HttpService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// Definición de Temas Base
local Themes = {
    RedMatte = {
        Name = "Red Matte",
        MainColor = Color3.fromRGB(18, 18, 18),
        ContainerColor = Color3.fromRGB(26, 26, 26),
        AccentColor = Color3.fromRGB(220, 35, 35),
        Font = Enum.Font.GothamBold
    },
    Cyberpunk = {
        Name = "Cyberpunk",
        MainColor = Color3.fromRGB(15, 10, 25),
        ContainerColor = Color3.fromRGB(25, 18, 40),
        AccentColor = Color3.fromRGB(180, 0, 255),
        Font = Enum.Font.FredokaOne
    },
    Midnight = {
        Name = "Midnight",
        MainColor = Color3.fromRGB(10, 15, 25),
        ContainerColor = Color3.fromRGB(18, 25, 38),
        AccentColor = Color3.fromRGB(0, 150, 255),
        Font = Enum.Font.SourceSansBold
    },
    Emerald = {
        Name = "Emerald",
        MainColor = Color3.fromRGB(10, 22, 16),
        ContainerColor = Color3.fromRGB(16, 32, 24),
        AccentColor = Color3.fromRGB(0, 230, 120),
        Font = Enum.Font.Arcade
    }
}

--// Variables de Estado Global
local CurrentTheme = Themes.RedMatte

local DefaultState = {
    -- Modo de Dispositivo
    DeviceMode = "PC",
    
    -- Combate
    AimbotEnabled = false,
    SilentAimbot = false,
    SilentHitchance = 100,
    AimbotTargetPart = "Head",
    AimbotSmoothing = 0.15,
    AimbotPrediction = false,
    TriggerbotEnabled = false,
    TriggerbotDelay = 0.05,
    KillAllEnabled = false,
    KillAllMelee = false,
    KillAllDelay = 0.1,
    AuraPistola = false,
    AuraPistolaRange = 50,
    Wallbang = false,
    HitboxExtender = false,
    HitboxSize = 5,
    AimbotFOV = false,
    FOVSize = 150,
    
    -- Visuales
    ESPBoxes = false,
    ESPTracers = false,
    ESPHealthBar = false,
    ESPInfo = false,
    ESPChams = false,
    DynamicColors = true,
    ColorVisible = Color3.fromRGB(0, 255, 120),
    ColorHidden = Color3.fromRGB(255, 40, 40),
    ESPRainbow = false,
    SkyboxRainbow = false,
    AtmosphereRainbow = false,
    RainbowCharacter = false,
    RainbowSpeed = 1,
    
    CustomBoxColor = Color3.fromRGB(0, 255, 255),
    CustomTracerColor = Color3.fromRGB(255, 255, 0),
    CustomChamsColor = Color3.fromRGB(255, 0, 255),
    CustomFOVColor = Color3.fromRGB(0, 255, 120),
    CustomInfoColor = Color3.fromRGB(255, 255, 255),
    
    -- Movimiento & Cámara
    ThirdPerson = false,
    ThirdPersonDistance = 12,
    Fly = false,
    FlySpeed = 50,
    SpeedHack = false,
    WalkSpeedValue = 100,
    InfiniteJump = false,
    Noclip = false,
    Spinbot = false,
    SpinSpeed = 30,
    CameraFOV = 70,
    Freecam = false,
    FreecamSpeed = 50,
    
    -- Utilidades
    AntiAFK = false,
    ShowWatermark = false,
    FPSBoost = false,
    FPSUnlocker = false,
    
    -- Panel Dimensions
    PanelWidth = 370,
    PanelHeight = 420,
    PanelTransparency = 0,
    SelectedFont = Enum.Font.GothamBold
}

local State = {}
for k, v in pairs(DefaultState) do State[k] = v end

-- Helper para asegurar que SelectedFont siempre sea un Enum.Font válido
local function GetSafeFont()
    if typeof(State.SelectedFont) == "EnumItem" then
        return State.SelectedFont
    elseif type(State.SelectedFont) == "string" then
        local fontName = State.SelectedFont:gsub("Enum.Font.", "")
        return Enum.Font[fontName] or Enum.Font.GothamBold
    end
    return Enum.Font.GothamBold
end

-- Mapeo de Atajos Tecla -> Función
local Keybinds = {}
local FunctionCallbacks = {}

-- Tabla para almacenar funciones de actualización visual de la UI
local UIUpdaters = {}

local ESPCache = {}
local OriginalHitboxSizes = {}
local ConfigFileName = "RageHubV9_Config.json"

--------------------------------------------------------------------------------
-- CONFIGURACIÓN DE TAMAÑOS DE INTERFAZ SEGÚN MODO DE DISPOSITIVO
--------------------------------------------------------------------------------
local DEVICE_CONFIG = {
    PC = {
        PanelSize = UDim2.new(0, 370, 0, 420),
        WMSize = UDim2.new(0, 240, 0, 28),
        WMPos = UDim2.new(0, 15, 0, 15),
        WMTextSize = 10,
        NotifHolderSize = UDim2.new(0, 260, 1, -20),
        NotifHolderPos = UDim2.new(1, -270, 0, 10),
        NotifItemSize = UDim2.new(1, 0, 0, 44),
        NotifTitleSize = 11,
        NotifMsgSize = 10,
        TabWidth = 95,
        TitleSize = 12
    },
    Mobile = {
        PanelSize = UDim2.new(0, 280, 0, 290),
        WMSize = UDim2.new(0, 150, 0, 20),
        WMPos = UDim2.new(0, 8, 0, 8),
        WMTextSize = 8,
        NotifHolderSize = UDim2.new(0, 180, 1, -10),
        NotifHolderPos = UDim2.new(1, -190, 0, 5),
        NotifItemSize = UDim2.new(1, 0, 0, 32),
        NotifTitleSize = 9,
        NotifMsgSize = 8,
        TabWidth = 75,
        TitleSize = 10
    }
}

--// Guardado y Carga JSON
local function ColorToTable(color)
    return {R = color.R, G = color.G, B = color.B}
end

local function TableToColor(tbl)
    if tbl and tbl.R and tbl.G and tbl.B then
        return Color3.new(tbl.R, tbl.G, tbl.B)
    end
    return Color3.fromRGB(255, 255, 255)
end

local function SaveConfig()
    pcall(function()
        local dataToSave = {
            State = {},
            Keybinds = Keybinds
        }
        for k, v in pairs(State) do
            if typeof(v) == "Color3" then
                dataToSave.State[k] = {__type = "Color3", value = ColorToTable(v)}
            elseif typeof(v) == "EnumItem" then
                dataToSave.State[k] = {__type = "Enum", value = tostring(v)}
            else
                dataToSave.State[k] = v
            end
        end
        
        local jsonString = HttpService:JSONEncode(dataToSave)
        if writefile then
            writefile(ConfigFileName, jsonString)
        end
    end)
end

local function UpdateAllVisualUI()
    for _, updateFn in ipairs(UIUpdaters) do
        pcall(updateFn)
    end
end

local function LoadConfig()
    pcall(function()
        if readfile and isfile and isfile(ConfigFileName) then
            local jsonString = readfile(ConfigFileName)
            local loadedData = HttpService:JSONEncode(jsonString)
            
            if loadedData and loadedData.State then
                for k, v in pairs(loadedData.State) do
                    if type(v) == "table" and v.__type == "Color3" then
                        State[k] = TableToColor(v.value)
                    elseif type(v) == "table" and v.__type == "Enum" then
                        local fontName = tostring(v.value):gsub("Enum.Font.", "")
                        State[k] = Enum.Font[fontName] or Enum.Font.GothamBold
                    else
                        State[k] = v
                    end
                end
            end
            if loadedData and loadedData.Keybinds then
                Keybinds = loadedData.Keybinds
            end
            
            UpdateAllVisualUI()
        end
    end)
end

local function ResetConfig()
    pcall(function()
        for k, v in pairs(DefaultState) do
            State[k] = v
        end
        Keybinds = {}
        if writefile and isfile and isfile(ConfigFileName) then
            delfile(ConfigFileName)
        end
        UpdateAllVisualUI()
    end)
end

--// Limpieza de ESP
local function RemoveESPForPlayer(plrName)
    if ESPCache[plrName] then
        pcall(function()
            if ESPCache[plrName].Box then ESPCache[plrName].Box:Remove() end
            if ESPCache[plrName].Tracer then ESPCache[plrName].Tracer:Remove() end
            if ESPCache[plrName].Health then ESPCache[plrName].Health:Remove() end
            if ESPCache[plrName].Info then ESPCache[plrName].Info:Remove() end
        end)
        ESPCache[plrName] = nil
    end
end

Players.PlayerRemoving:Connect(function(plr)
    RemoveESPForPlayer(plr.Name)
end)

--// FOV Circle
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.NumSides = 40
FOVCircle.Radius = State.FOVSize
FOVCircle.Filled = false
FOVCircle.Visible = false
FOVCircle.Color = CurrentTheme.AccentColor

local function GetEffectiveColor(category, defaultVisibleColor)
    if State.ESPRainbow then
        local hue = (tick() % 2) / 2
        return Color3.fromHSV(hue, 1, 1)
    end
    if State.DynamicColors and defaultVisibleColor then
        return defaultVisibleColor
    end
    if category == "Box" then return State.CustomBoxColor end
    if category == "Tracer" then return State.CustomTracerColor end
    if category == "Chams" then return State.CustomChamsColor end
    if category == "FOV" then return State.CustomFOVColor end
    if category == "Info" then return State.CustomInfoColor end
    return CurrentTheme.AccentColor
end

--// UI Base
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RageHubV9"
ScreenGui.ResetOnSpawn = false

local playerGui = LocalPlayer:WaitForChild("PlayerGui", 10)
if playerGui then
    ScreenGui.Parent = playerGui
else
    ScreenGui.Parent = game:GetService("CoreGui")
end

-- Watermark / Indicador FPS & Ping
local WatermarkFrame = Instance.new("Frame")
WatermarkFrame.Size = DEVICE_CONFIG.PC.WMSize
WatermarkFrame.Position = DEVICE_CONFIG.PC.WMPos
WatermarkFrame.BackgroundColor3 = CurrentTheme.MainColor
WatermarkFrame.Visible = State.ShowWatermark
WatermarkFrame.Parent = ScreenGui

local WMCorner = Instance.new("UICorner")
WMCorner.CornerRadius = UDim.new(0, 6)
WMCorner.Parent = WatermarkFrame

local WMStroke = Instance.new("UIStroke")
WMStroke.Thickness = 1.5
WMStroke.Color = CurrentTheme.AccentColor
WMStroke.Parent = WatermarkFrame

local WMLabel = Instance.new("TextLabel")
WMLabel.Size = UDim2.new(1, 0, 1, 0)
WMLabel.BackgroundTransparency = 1
WMLabel.Text = "RAGE HUB V9 | FPS: -- | PING: -- ms"
WMLabel.TextColor3 = Color3.fromRGB(240, 240, 240)
WMLabel.TextSize = DEVICE_CONFIG.PC.WMTextSize
WMLabel.Font = GetSafeFont()
WMLabel.Parent = WatermarkFrame

local lastFPSUpdate = tick()
local frameCount = 0
RunService.RenderStepped:Connect(function()
    frameCount = frameCount + 1
    local now = tick()
    if now - lastFPSUpdate >= 1 then
        local fps = frameCount
        frameCount = 0
        lastFPSUpdate = now
        local stats = game:GetService("Stats")
        local ping = 0
        if stats and stats.Network and stats.Network.ServerStatsItem and stats.Network.ServerStatsItem["Data Ping"] then
            ping = math.floor(stats.Network.ServerStatsItem["Data Ping"]:GetValue())
        end
        WMLabel.Text = string.format("RAGE HUB V9 | FPS: %d | PING: %d ms", fps, ping)
    end
end)

-- Contenedor de Notificaciones
local NotificationHolder = Instance.new("Frame")
NotificationHolder.Size = DEVICE_CONFIG.PC.NotifHolderSize
NotificationHolder.Position = DEVICE_CONFIG.PC.NotifHolderPos
NotificationHolder.BackgroundTransparency = 1
NotificationHolder.Parent = ScreenGui

local NotifLayout = Instance.new("UIListLayout")
NotifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
NotifLayout.Padding = UDim.new(0, 5)
NotifLayout.Parent = NotificationHolder

local function SendNotification(title, message)
    local cfg = DEVICE_CONFIG[State.DeviceMode] or DEVICE_CONFIG.PC
    local Frame = Instance.new("Frame")
    Frame.Size = cfg.NotifItemSize
    Frame.BackgroundColor3 = CurrentTheme.MainColor
    Frame.Parent = NotificationHolder

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = Frame

    local Bar = Instance.new("Frame")
    Bar.Size = UDim2.new(1, 0, 0, 2)
    Bar.Position = UDim2.new(0, 0, 1, -2)
    Bar.BackgroundColor3 = CurrentTheme.AccentColor
    Bar.BorderSizePixel = 0
    Bar.Parent = Frame

    local LblTitle = Instance.new("TextLabel")
    LblTitle.Size = UDim2.new(1, -12, 0, 14)
    LblTitle.Position = UDim2.new(0, 6, 0, 2)
    LblTitle.BackgroundTransparency = 1
    LblTitle.Text = title
    LblTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    LblTitle.TextSize = cfg.NotifTitleSize
    LblTitle.Font = GetSafeFont()
    LblTitle.TextXAlignment = Enum.TextXAlignment.Left
    LblTitle.Parent = Frame

    local LblMsg = Instance.new("TextLabel")
    LblMsg.Size = UDim2.new(1, -12, 0, 14)
    LblMsg.Position = UDim2.new(0, 6, 0, 16)
    LblMsg.BackgroundTransparency = 1
    LblMsg.Text = message
    LblMsg.TextColor3 = Color3.fromRGB(170, 170, 170)
    LblMsg.TextSize = cfg.NotifMsgSize
    LblMsg.Font = GetSafeFont()
    LblMsg.TextXAlignment = Enum.TextXAlignment.Left
    LblMsg.Parent = Frame

    task.delay(2.5, function() Frame:Destroy() end)
end

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, State.PanelWidth, 0, State.PanelHeight)
MainFrame.Position = UDim2.new(0.5, -State.PanelWidth/2, 0.5, -State.PanelHeight/2)
MainFrame.BackgroundColor3 = CurrentTheme.MainColor
MainFrame.BackgroundTransparency = State.PanelTransparency
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Thickness = 2
MainStroke.Color = CurrentTheme.AccentColor
MainStroke.Parent = MainFrame

-- Botón Flotante para Móvil
local MobileToggleBtn = Instance.new("TextButton")
MobileToggleBtn.Name = "MobileToggleBtn"
MobileToggleBtn.Size = UDim2.new(0, 42, 0, 42)
MobileToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
MobileToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MobileToggleBtn.Text = "HUB"
MobileToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MobileToggleBtn.Font = Enum.Font.GothamBold
MobileToggleBtn.TextSize = 10
MobileToggleBtn.Active = true
MobileToggleBtn.Draggable = true
MobileToggleBtn.Visible = (State.DeviceMode == "Mobile")
MobileToggleBtn.Parent = ScreenGui

local MobileCorner = Instance.new("UICorner")
MobileCorner.CornerRadius = UDim.new(1, 0)
MobileCorner.Parent = MobileToggleBtn

local MobileStroke = Instance.new("UIStroke")
MobileStroke.Thickness = 2
MobileStroke.Color = CurrentTheme.AccentColor
MobileStroke.Parent = MobileToggleBtn

task.spawn(function()
    while true do
        local hue = (tick() % 3) / 3
        MobileStroke.Color = Color3.fromHSV(hue, 0.9, 1)
        task.wait(0.03)
    end
end)

MobileToggleBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Topbar
local Topbar = Instance.new("Frame")
Topbar.Size = UDim2.new(1, 0, 0, 32)
Topbar.BackgroundColor3 = CurrentTheme.ContainerColor
Topbar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 110, 1, 0)
Title.Position = UDim2.new(0, 8, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "RAGE HUB V9"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = DEVICE_CONFIG.PC.TitleSize
Title.Font = GetSafeFont()
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Topbar

local ModeSwitchBtn = Instance.new("TextButton")
ModeSwitchBtn.Size = UDim2.new(0, 75, 0, 22)
ModeSwitchBtn.Position = UDim2.new(1, -135, 0.5, -11)
ModeSwitchBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ModeSwitchBtn.Text = State.DeviceMode == "PC" and "💻 Modo: PC" or "📱 Modo: Movil"
ModeSwitchBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ModeSwitchBtn.Font = GetSafeFont()
ModeSwitchBtn.TextSize = 8
ModeSwitchBtn.Parent = Topbar

local ModeCorner = Instance.new("UICorner")
ModeCorner.CornerRadius = UDim.new(0, 4)
ModeCorner.Parent = ModeSwitchBtn

-- Barra de Búsqueda
local SearchFrame = Instance.new("Frame")
SearchFrame.Size = UDim2.new(1, -16, 0, 24)
SearchFrame.Position = UDim2.new(0, 8, 0, 36)
SearchFrame.BackgroundColor3 = CurrentTheme.ContainerColor
SearchFrame.Parent = MainFrame

local SearchCorner = Instance.new("UICorner")
SearchCorner.CornerRadius = UDim.new(0, 5)
SearchCorner.Parent = SearchFrame

local SearchInput = Instance.new("TextBox")
SearchInput.Size = UDim2.new(1, -12, 1, 0)
SearchInput.Position = UDim2.new(0, 6, 0, 0)
SearchInput.BackgroundTransparency = 1
SearchInput.PlaceholderText = "Buscar función..."
SearchInput.PlaceholderColor3 = Color3.fromRGB(120, 120, 120)
SearchInput.Text = ""
SearchInput.TextColor3 = Color3.fromRGB(255, 255, 255)
SearchInput.Font = GetSafeFont()
SearchInput.TextSize = 9
SearchInput.TextXAlignment = Enum.TextXAlignment.Left
SearchInput.Parent = SearchFrame

-- Pestañas y Contenedores
local TabButtonsFrame = Instance.new("Frame")
TabButtonsFrame.Size = UDim2.new(0, DEVICE_CONFIG.PC.TabWidth, 1, -100)
TabButtonsFrame.Position = UDim2.new(0, 8, 0, 64)
TabButtonsFrame.BackgroundColor3 = CurrentTheme.ContainerColor
TabButtonsFrame.Parent = MainFrame

local TabButtonsCorner = Instance.new("UICorner")
TabButtonsCorner.CornerRadius = UDim.new(0, 5)
TabButtonsCorner.Parent = TabButtonsFrame

local TabListLayout = Instance.new("UIListLayout")
TabListLayout.Padding = UDim.new(0, 3)
TabListLayout.Parent = TabButtonsFrame

local TabContainersFrame = Instance.new("Frame")
TabContainersFrame.Size = UDim2.new(1, -(DEVICE_CONFIG.PC.TabWidth + 22), 1, -100)
TabContainersFrame.Position = UDim2.new(0, DEVICE_CONFIG.PC.TabWidth + 14, 0, 64)
TabContainersFrame.BackgroundTransparency = 1
TabContainersFrame.Parent = MainFrame

-- Barra Inferior
local BottomSaveFrame = Instance.new("Frame")
BottomSaveFrame.Size = UDim2.new(1, -16, 0, 24)
BottomSaveFrame.Position = UDim2.new(0, 8, 1, -28)
BottomSaveFrame.BackgroundColor3 = CurrentTheme.ContainerColor
BottomSaveFrame.Parent = MainFrame

local BottomCorner = Instance.new("UICorner")
BottomCorner.CornerRadius = UDim.new(0, 5)
BottomCorner.Parent = BottomSaveFrame

local SaveBtnBottom = Instance.new("TextButton")
SaveBtnBottom.Size = UDim2.new(1, 0, 1, 0)
SaveBtnBottom.BackgroundTransparency = 1
SaveBtnBottom.Text = "💾 Guardar Configuración + Keybinds (JSON)"
SaveBtnBottom.TextColor3 = Color3.fromRGB(255, 255, 255)
SaveBtnBottom.Font = GetSafeFont()
SaveBtnBottom.TextSize = 9
SaveBtnBottom.Parent = BottomSaveFrame

SaveBtnBottom.MouseButton1Click:Connect(function()
    SaveConfig()
    SendNotification("Guardado", "Configuración y atajos guardados en JSON.")
end)

--------------------------------------------------------------------------------
-- LÓGICA DE ACTIVACIÓN DE OPTIMIZACIÓN DE RENDIMIENTO (BOOST & FPS UNLOCKER)
--------------------------------------------------------------------------------
local function ApplyPerformanceBoost(enable)
    pcall(function()
        if enable then
            if setfpscap then setfpscap(240) end
            Lighting.GlobalShadows = false
            Lighting.Technology = Enum.Technology.Compatibility
            
            for _, effect in ipairs(Lighting:GetChildren()) do
                if effect:IsA("PostEffect") then
                    effect.Enabled = false
                end
            end
            
            if workspace.Terrain then
                workspace.Terrain.WaterWaveSize = 0
                workspace.Terrain.WaterWaveSpeed = 0
                workspace.Terrain.WaterReflectance = 0
                workspace.Terrain.WaterTransparency = 0
            end
        else
            if setfpscap then setfpscap(60) end
            Lighting.GlobalShadows = true
            Lighting.Technology = Enum.Technology.ShadowMap
            
            for _, effect in ipairs(Lighting:GetChildren()) do
                if effect:IsA("PostEffect") then
                    effect.Enabled = true
                end
            end
        end
    end)
end

--------------------------------------------------------------------------------
-- ACTUALIZACIÓN DINÁMICA DEL MODO DE DISPOSITIVO (REESCALADO DE UI & OCULTAR KEYBINDS)
--------------------------------------------------------------------------------
local Tabs = {}
local CurrentTab = nil

local function UpdateDeviceModeUI()
    local isMobile = (State.DeviceMode == "Mobile")
    local cfg = isMobile and DEVICE_CONFIG.Mobile or DEVICE_CONFIG.PC

    ModeSwitchBtn.Text = isMobile and "📱 Modo: Movil" or "💻 Modo: PC"
    MobileToggleBtn.Visible = isMobile

    -- Reescalado del panel principal y de la interfaz
    MainFrame.Size = cfg.PanelSize
    WatermarkFrame.Size = cfg.WMSize
    WatermarkFrame.Position = cfg.WMPos
    WMLabel.TextSize = cfg.WMTextSize

    NotificationHolder.Size = cfg.NotifHolderSize
    NotificationHolder.Position = cfg.NotifHolderPos

    Title.TextSize = cfg.TitleSize
    TabButtonsFrame.Size = UDim2.new(0, cfg.TabWidth, 1, -100)
    TabContainersFrame.Size = UDim2.new(1, -(cfg.TabWidth + 22), 1, -100)
    TabContainersFrame.Position = UDim2.new(0, cfg.TabWidth + 14, 0, 64)

    -- Aplicar FPS Boost y FPS Unlocker automáticamente al entrar en Modo Móvil
    State.FPSBoost = isMobile
    State.FPSUnlocker = isMobile
    ApplyPerformanceBoost(isMobile)

    -- Ocultar / Mostrar apartado de Keybinds según el modo
    if Tabs["Keybinds"] then
        Tabs["Keybinds"].Btn.Visible = not isMobile
        
        if isMobile then
            Tabs["Keybinds"].Container.Visible = false
            if CurrentTab == "Keybinds" and Tabs["Combate"] then
                CurrentTab = "Combate"
                Tabs["Combate"].Container.Visible = true
                Tabs["Combate"].Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
                Tabs["Combate"].Btn.BackgroundColor3 = CurrentTheme.AccentColor
            end
        end
    end
end

table.insert(UIUpdaters, UpdateDeviceModeUI)

ModeSwitchBtn.MouseButton1Click:Connect(function()
    if State.DeviceMode == "PC" then
        State.DeviceMode = "Mobile"
        SendNotification("Modo Dispositivo", "Modo Móvil Activado (FPS Boost & Unlocker ON)")
    else
        State.DeviceMode = "PC"
        SendNotification("Modo Dispositivo", "Modo PC Activado")
    end
    UpdateDeviceModeUI()
end)

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Size = UDim2.new(0, 22, 0, 22)
MinimizeBtn.Position = UDim2.new(1, -54, 0.5, -11)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
MinimizeBtn.Text = "-"
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.Font = GetSafeFont()
MinimizeBtn.TextSize = 13
MinimizeBtn.Parent = Topbar

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 4)
MinCorner.Parent = MinimizeBtn

local isMinimized = false
MinimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    for _, child in pairs(MainFrame:GetChildren()) do
        if child ~= Topbar and not child:IsA("UICorner") and not child:IsA("UIStroke") then
            child.Visible = not isMinimized
        end
    end
    local cfg = DEVICE_CONFIG[State.DeviceMode] or DEVICE_CONFIG.PC
    MainFrame.Size = isMinimized and UDim2.new(0, cfg.PanelSize.X.Offset, 0, 32) or cfg.PanelSize
end)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Position = UDim2.new(1, -28, 0.5, -11)
CloseBtn.BackgroundColor3 = CurrentTheme.AccentColor
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = GetSafeFont()
CloseBtn.TextSize = 10
CloseBtn.Parent = Topbar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 4)
CloseCorner.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    pcall(function() FOVCircle:Remove() end)
end)

local function CreateTab(name)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 0, 24)
    Btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    Btn.Text = name
    Btn.TextColor3 = Color3.fromRGB(180, 180, 180)
    Btn.Font = GetSafeFont()
    Btn.TextSize = 9
    Btn.Parent = TabButtonsFrame

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 4)
    BtnCorner.Parent = Btn

    local Container = Instance.new("ScrollingFrame")
    Container.Size = UDim2.new(1, 0, 1, 0)
    Container.BackgroundTransparency = 1
    Container.CanvasSize = UDim2.new(0, 0, 0, 0)
    Container.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Container.ScrollBarThickness = 2
    Container.ScrollBarImageColor3 = CurrentTheme.AccentColor
    Container.Visible = false
    Container.Parent = TabContainersFrame

    local Layout = Instance.new("UIListLayout")
    Layout.Padding = UDim.new(0, 4)
    Layout.Parent = Container

    Btn.MouseButton1Click:Connect(function()
        for _, t in pairs(Tabs) do
            t.Container.Visible = false
            t.Btn.TextColor3 = Color3.fromRGB(180, 180, 180)
            t.Btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        end
        Container.Visible = true
        Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        Btn.BackgroundColor3 = CurrentTheme.AccentColor
        CurrentTab = name
    end)

    Tabs[name] = {Btn = Btn, Container = Container}
    if not CurrentTab then
        CurrentTab = name
        Container.Visible = true
        Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        Btn.BackgroundColor3 = CurrentTheme.AccentColor
    end

    return Container
end

local CombatTab = CreateTab("Combate")
local VisualsTab = CreateTab("Visuales")
local MovementTab = CreateTab("Movimiento")
local KeybindsTab = CreateTab("Keybinds")
local UtilityTab = CreateTab("Utilidades")
local SettingsTab = CreateTab("Ajustes")

SearchInput:GetPropertyChangedSignal("Text"):Connect(function()
    local text = string.lower(SearchInput.Text)
    for _, tab in pairs(Tabs) do
        for _, child in pairs(tab.Container:GetChildren()) do
            if child:IsA("Frame") then
                local label = child:FindFirstChild("ItemLabel")
                if label then
                    child.Visible = (text == "" or string.find(string.lower(label.Text), text, 1, true) ~= nil)
                end
            end
        end
    end
end)

-- Creadores de Controles UI
local RegisteredContainers = {}
local RegisteredSwitches = {}

local function RegisterThemeElement(element, property, role)
    table.insert(RegisteredContainers, {Element = element, Property = property, Role = role})
end

local function CreateToggle(parent, name, internalStateKey, callback)
    local Frame = Instance.new("Frame")
    Frame.Name = name
    Frame.Size = UDim2.new(1, -4, 0, 28)
    Frame.BackgroundColor3 = CurrentTheme.ContainerColor
    Frame.Parent = parent
    RegisterThemeElement(Frame, "BackgroundColor3", "ContainerColor")

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 5)
    Corner.Parent = Frame

    local Label = Instance.new("TextLabel")
    Label.Name = "ItemLabel"
    Label.Size = UDim2.new(1, -40, 1, 0)
    Label.Position = UDim2.new(0, 6, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = name
    Label.TextColor3 = Color3.fromRGB(180, 180, 180)
    Label.Font = GetSafeFont()
    Label.TextSize = 9
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local SwitchTrack = Instance.new("Frame")
    SwitchTrack.Size = UDim2.new(0, 26, 0, 14)
    SwitchTrack.Position = UDim2.new(1, -30, 0.5, -7)
    SwitchTrack.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    SwitchTrack.Parent = Frame

    local TrackCorner = Instance.new("UICorner")
    TrackCorner.CornerRadius = UDim.new(1, 0)
    TrackCorner.Parent = SwitchTrack

    local SwitchBall = Instance.new("Frame")
    SwitchBall.Size = UDim2.new(0, 10, 0, 10)
    SwitchBall.Position = UDim2.new(0, 2, 0.5, -5)
    SwitchBall.BackgroundColor3 = Color3.fromRGB(220, 220, 220)
    SwitchBall.Parent = SwitchTrack

    local BallCorner = Instance.new("UICorner")
    BallCorner.CornerRadius = UDim.new(1, 0)
    BallCorner.Parent = SwitchBall

    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 1, 0)
    Btn.BackgroundTransparency = 1
    Btn.Text = ""
    Btn.Parent = Frame

    local function ApplyVisualState(val)
        SwitchBall.Position = val and UDim2.new(1, -12, 0.5, -5) or UDim2.new(0, 2, 0.5, -5)
        SwitchTrack.BackgroundColor3 = val and CurrentTheme.AccentColor or Color3.fromRGB(45, 45, 45)
        Label.TextColor3 = val and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(180, 180, 180)
    end

    local function SetToggleState(val)
        State[internalStateKey] = val
        ApplyVisualState(val)
        callback(val)
    end

    Btn.MouseButton1Click:Connect(function()
        local newState = not State[internalStateKey]
        SetToggleState(newState)
        SendNotification(name, newState and "Activado" or "Desactivado")
    end)

    FunctionCallbacks[name] = function()
        local newState = not State[internalStateKey]
        SetToggleState(newState)
        SendNotification(name, newState and "Activado (Keybind)" or "Desactivado (Keybind)")
    end

    table.insert(UIUpdaters, function()
        local currentVal = State[internalStateKey]
        ApplyVisualState(currentVal)
        callback(currentVal)
    end)

    table.insert(RegisteredSwitches, {Track = SwitchTrack, GetState = function() return State[internalStateKey] end})
end

local function CreateButton(parent, name, callback)
    local Frame = Instance.new("Frame")
    Frame.Name = name
    Frame.Size = UDim2.new(1, -4, 0, 28)
    Frame.BackgroundColor3 = CurrentTheme.ContainerColor
    Frame.Parent = parent
    RegisterThemeElement(Frame, "BackgroundColor3", "ContainerColor")

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 5)
    Corner.Parent = Frame

    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 1, 0)
    Btn.BackgroundTransparency = 1
    Btn.Text = name
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.Font = GetSafeFont()
    Btn.TextSize = 9
    Btn.Parent = Frame

    Btn.MouseButton1Click:Connect(callback)
end

-- Slider optimizado con detección del máximo y etiquetas dinámicas
local function CreateSlider(parent, name, minVal, maxVal, defaultVal, callback, internalStateKey)
    local Frame = Instance.new("Frame")
    Frame.Name = name
    Frame.Size = UDim2.new(1, -4, 0, 34)
    Frame.BackgroundColor3 = CurrentTheme.ContainerColor
    Frame.Parent = parent
    RegisterThemeElement(Frame, "BackgroundColor3", "ContainerColor")

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 5)
    Corner.Parent = Frame

    local Label = Instance.new("TextLabel")
    Label.Name = "ItemLabel"
    Label.Size = UDim2.new(1, -12, 0, 14)
    Label.Position = UDim2.new(0, 6, 0, 2)
    Label.BackgroundTransparency = 1
    Label.TextColor3 = Color3.fromRGB(180, 180, 180)
    Label.Font = GetSafeFont()
    Label.TextSize = 9
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local SliderBar = Instance.new("Frame")
    SliderBar.Size = UDim2.new(1, -12, 0, 5)
    SliderBar.Position = UDim2.new(0, 6, 0, 22)
    SliderBar.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    SliderBar.Parent = Frame

    local SliderFill = Instance.new("Frame")
    SliderFill.Name = "SliderFill"
    SliderFill.Size = UDim2.new((defaultVal - minVal)/(maxVal - minVal), 0, 1, 0)
    SliderFill.BackgroundColor3 = CurrentTheme.AccentColor
    SliderFill.BorderSizePixel = 0
    SliderFill.Parent = SliderBar
    RegisterThemeElement(SliderFill, "BackgroundColor3", "AccentColor")

    local SliderHitbox = Instance.new("TextButton")
    SliderHitbox.Size = UDim2.new(1, 10, 1, 12)
    SliderHitbox.Position = UDim2.new(0, -5, 0, -6)
    SliderHitbox.BackgroundTransparency = 1
    SliderHitbox.Text = ""
    SliderHitbox.Parent = SliderBar

    local function FormatText(val)
        if val >= maxVal then
            return name .. ": MÁXIMO"
        else
            return name .. ": " .. tostring(val)
        end
    end

    Label.Text = FormatText(defaultVal)

    local dragging = false
    local function UpdateVal(input)
        local relPos = input.Position.X - SliderBar.AbsolutePosition.X
        local pct = math.clamp(relPos / SliderBar.AbsoluteSize.X, 0, 1)
        local val = math.floor(minVal + (maxVal - minVal) * pct)
        SliderFill.Size = UDim2.new(pct, 0, 1, 0)
        Label.Text = FormatText(val)
        callback(val)
    end

    SliderHitbox.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            MainFrame.Draggable = false
            UpdateVal(input)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                dragging = false
                MainFrame.Draggable = true
            end
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            UpdateVal(input)
        end
    end)

    if internalStateKey then
        table.insert(UIUpdaters, function()
            local rawVal = State[internalStateKey]
            if rawVal then
                local val = math.floor(rawVal)
                local pct = math.clamp((val - minVal) / (maxVal - minVal), 0, 1)
                SliderFill.Size = UDim2.new(pct, 0, 1, 0)
                Label.Text = FormatText(val)
            end
        end)
    end
end

local function CreateDropdown(parent, name, options, callback)
    local Frame = Instance.new("Frame")
    Frame.Name = name
    Frame.Size = UDim2.new(1, -4, 0, 28)
    Frame.BackgroundColor3 = CurrentTheme.ContainerColor
    Frame.Parent = parent
    RegisterThemeElement(Frame, "BackgroundColor3", "ContainerColor")

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 5)
    Corner.Parent = Frame

    local Label = Instance.new("TextLabel")
    Label.Name = "ItemLabel"
    Label.Size = UDim2.new(0, 80, 1, 0)
    Label.Position = UDim2.new(0, 6, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = name .. ":"
    Label.TextColor3 = Color3.fromRGB(180, 180, 180)
    Label.Font = GetSafeFont()
    Label.TextSize = 9
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 85, 0, 18)
    Btn.Position = UDim2.new(1, -90, 0.5, -9)
    Btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    Btn.Text = options[1]
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.Font = GetSafeFont()
    Btn.TextSize = 8
    Btn.Parent = Frame

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 4)
    BtnCorner.Parent = Btn

    local idx = 1
    Btn.MouseButton1Click:Connect(function()
        idx = idx + 1
        if idx > #options then idx = 1 end
        Btn.Text = options[idx]
        callback(options[idx])
    end)
end

--// Keybinds Manager UI
local AvailableKeys = {
    "A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M",
    "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z",
    "Space", "LeftShift", "LeftControl"
}

local BindableFunctions = {
    "Ninguno",
    "Aimbot",
    "Silent Aimbot",
    "Triggerbot",
    "Kill All (Pistola)",
    "Kill All (Cuchillo/Melee)",
    "Aura Pistola",
    "Fly (Volar)",
    "Speed Hack",
    "Spinbot",
    "Noclip",
    "Salto Infinito",
    "Cajas ESP",
    "Líneas ESP"
}

local function RenderKeybindsList()
    for _, child in pairs(KeybindsTab:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end

    for _, keyName in ipairs(AvailableKeys) do
        local Frame = Instance.new("Frame")
        Frame.Name = "Bind_" .. keyName
        Frame.Size = UDim2.new(1, -4, 0, 28)
        Frame.BackgroundColor3 = CurrentTheme.ContainerColor
        Frame.Parent = KeybindsTab
        RegisterThemeElement(Frame, "BackgroundColor3", "ContainerColor")

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 5)
        Corner.Parent = Frame

        local Label = Instance.new("TextLabel")
        Label.Name = "ItemLabel"
        Label.Size = UDim2.new(0, 60, 1, 0)
        Label.Position = UDim2.new(0, 6, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = "Tecla [" .. keyName .. "]:"
        Label.TextColor3 = Color3.fromRGB(200, 200, 200)
        Label.Font = GetSafeFont()
        Label.TextSize = 9
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Frame

        local DropdownBtn = Instance.new("TextButton")
        DropdownBtn.Size = UDim2.new(0, 110, 0, 18)
        DropdownBtn.Position = UDim2.new(1, -115, 0.5, -9)
        DropdownBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        
        local currentFunc = Keybinds[keyName] or "Ninguno"
        DropdownBtn.Text = currentFunc
        DropdownBtn.TextColor3 = currentFunc ~= "Ninguno" and CurrentTheme.AccentColor or Color3.fromRGB(150, 150, 150)
        DropdownBtn.Font = GetSafeFont()
        DropdownBtn.TextSize = 8
        DropdownBtn.Parent = Frame

        local DropCorner = Instance.new("UICorner")
        DropCorner.CornerRadius = UDim.new(0, 4)
        DropCorner.Parent = DropdownBtn

        local currentIdx = 1
        for i, f in ipairs(BindableFunctions) do
            if f == currentFunc then currentIdx = i break end
        end

        DropdownBtn.MouseButton1Click:Connect(function()
            currentIdx = currentIdx + 1
            if currentIdx > #BindableFunctions then currentIdx = 1 end
            
            local selectedFunc = BindableFunctions[currentIdx]
            Keybinds[keyName] = selectedFunc ~= "Ninguno" and selectedFunc or nil
            DropdownBtn.Text = selectedFunc
            DropdownBtn.TextColor3 = selectedFunc ~= "Ninguno" and CurrentTheme.AccentColor or Color3.fromRGB(150, 150, 150)
            
            SendNotification("Atajo " .. keyName, "Asignado a: " .. selectedFunc)
        end)
    end
end

table.insert(UIUpdaters, RenderKeybindsList)
RenderKeybindsList()

-- Escuchador de Teclas en PC
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    
    if input.KeyCode == Enum.KeyCode.RightControl or input.KeyCode == Enum.KeyCode.Insert then
        MainFrame.Visible = not MainFrame.Visible
        return
    end

    if State.DeviceMode == "PC" then
        local keyString = tostring(input.KeyCode.Name)
        local mappedFunction = Keybinds[keyString]
        
        if mappedFunction and FunctionCallbacks[mappedFunction] then
            FunctionCallbacks[mappedFunction]()
        end
    end
end)

-- Raycast Helper
local raycastParams = RaycastParams.new()
raycastParams.FilterType = Enum.RaycastFilterType.Exclude

local function IsVisible(part, character)
    local origin = Camera.CFrame.Position
    local direction = part.Position - origin
    local ignore = {Camera}
    if LocalPlayer.Character then table.insert(ignore, LocalPlayer.Character) end
    if character then table.insert(ignore, character) end
    raycastParams.FilterDescendantsInstances = ignore

    local result = workspace:Raycast(origin, direction, raycastParams)
    return result == nil
end

local function GetTargetColor(visible)
    if State.DynamicColors then
        return visible and State.ColorVisible or State.ColorHidden
    end
    return CurrentTheme.AccentColor
end

local function GetClosestPlayerToCenter()
    local closestPlr = nil
    local shortestDist = State.FOVSize
    local centerPos = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local targetPart = plr.Character:FindFirstChild(State.AimbotTargetPart) or plr.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)

            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - centerPos).Magnitude
                if dist < shortestDist then
                    closestPlr = plr
                    shortestDist = dist
                end
            end
        end
    end
    return closestPlr
end

-- Hook / Lógica de Silent Aimbot
local meta = getrawmetatable or function() return nil end
if meta and setreadonly then
    local gmt = meta(game)
    local oldIndex = gmt.__index
    local oldNamecall = gmt.__namecall
    setreadonly(gmt, false)

    gmt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if not checkcaller() and State.SilentAimbot and (method == "Raycast" or method == "FindPartOnRay" or method == "FindPartOnRayWithIgnoreList") then
            if math.random(1, 100) <= State.SilentHitchance then
                local target = GetClosestPlayerToCenter()
                if target and target.Character then
                    local targetPart = target.Character:FindFirstChild(State.AimbotTargetPart) or target.Character.HumanoidRootPart
                    if targetPart then
                        local args = {...}
                        if method == "Raycast" and args[1] and args[2] then
                            args[2] = (targetPart.Position - args[1]).Unit * args[2].Magnitude
                            return oldNamecall(self, unpack(args))
                        end
                    end
                end
            end
        end
        return oldNamecall(self, ...)
    end)
    setreadonly(gmt, true)
end

--------------------------------------------------------------------------------
-- TRIGGERBOT HÍBRIDO (SINCRONIZADO PC/MÓVIL)
--------------------------------------------------------------------------------
task.spawn(function()
    while true do
        task.wait(State.TriggerbotDelay)
        if State.TriggerbotEnabled and LocalPlayer.Character then
            local mousePos = UserInputService:GetMouseLocation()
            local ray = Camera:ViewportPointToRay(mousePos.X, mousePos.Y)
            raycastParams.FilterDescendantsInstances = {Camera, LocalPlayer.Character}
            local result = workspace:Raycast(ray.Origin, ray.Direction * 1000, raycastParams)

            if result and result.Instance then
                local hitChar = result.Instance:FindFirstAncestorOfClass("Model")
                if hitChar then
                    local plr = Players:GetPlayerFromCharacter(hitChar)
                    if plr and plr ~= LocalPlayer then
                        local hum = hitChar:FindFirstChildOfClass("Humanoid")
                        if hum and hum.Health > 0 then
                            local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
                            if tool then
                                if State.DeviceMode == "PC" then
                                    VirtualInputManager:SendMouseButtonEvent(mousePos.X, mousePos.Y, 0, true, game, 0)
                                    task.wait(0.05)
                                    VirtualInputManager:SendMouseButtonEvent(mousePos.X, mousePos.Y, 0, false, game, 0)
                                else
                                    tool:Activate()
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end)

-- Vuelo y Vista en Tercera Persona
local flyVector = Vector3.zero
RunService.RenderStepped:Connect(function(dt)
    -- Lógica de Tercera Persona
    if State.ThirdPerson and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.CameraMaxZoomDistance = State.ThirdPersonDistance
        LocalPlayer.CameraMinZoomDistance = State.ThirdPersonDistance
    else
        LocalPlayer.CameraMaxZoomDistance = 128
        LocalPlayer.CameraMinZoomDistance = 0.5
    end

    -- Lógica de Vuelo
    if State.Fly and LocalPlayer.Character then
        local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hrp then
            if hum then hum.PlatformStand = true end

            local moveDir = Vector3.zero
            local camCF = Camera.CFrame

            if State.DeviceMode == "Mobile" then
                if hum.MoveDirection.Magnitude > 0 then
                    moveDir = camCF.LookVector
                end
                if hum.Jump then
                    moveDir = moveDir + Vector3.new(0, 1, 0)
                end
            else
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + camCF.LookVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - camCF.LookVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - camCF.RightVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + camCF.RightVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0, 1, 0) end
            end

            if moveDir.Magnitude > 0 then
                flyVector = flyVector:Lerp(moveDir.Unit * State.FlySpeed, dt * 10)
            else
                flyVector = flyVector:Lerp(Vector3.zero, dt * 10)
            end

            hrp.AssemblyLinearVelocity = flyVector
        end
    else
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.PlatformStand then
                hum.PlatformStand = false
            end
        end
    end
end)

-- Teleport Tool
local function GiveTeleportTool()
    local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
    if not backpack then return end

    if backpack:FindFirstChild("Click Teleport Tool") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Click Teleport Tool")) then
        SendNotification("Teleport Tool", "Ya tienes la herramienta en tu inventario.")
        return
    end

    local tool = Instance.new("Tool")
    tool.Name = "Click Teleport Tool"
    tool.RequiresHandle = false

    tool.Activated:Connect(function()
        local mouse = LocalPlayer:GetMouse()
        if mouse and mouse.Hit and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0, 3, 0))
            SendNotification("Teleport Tool", "Teletransportado al punto seleccionado.")
        end
    end)

    tool.Parent = backpack
    SendNotification("Teleport Tool", "Herramienta añadida a tu inventario.")
end

-- Helper Función para Obtener/Equipar Herramienta
local function GetEquippedTool()
    if not LocalPlayer.Character then return nil end
    local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
    if not tool then
        local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
        if backpack then
            tool = backpack:FindFirstChildOfClass("Tool")
            if tool then
                tool.Parent = LocalPlayer.Character
            end
        end
    end
    return tool
end

-- Kill All Loop
task.spawn(function()
    while true do
        task.wait(State.KillAllDelay)
        
        -- Kill All Cuchillo / Melee
        if State.KillAllMelee and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local tool = GetEquippedTool()
            if tool then
                for _, plr in pairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                        local enemyHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if enemyHum and enemyHum.Health > 0 then
                            local enemyHRP = plr.Character.HumanoidRootPart
                            LocalPlayer.Character.HumanoidRootPart.CFrame = enemyHRP.CFrame * CFrame.new(0, 0, 1.5)
                            tool:Activate()
                            task.wait(0.05)
                        end
                    end
                end
            end
        end

        -- Kill All Pistola / Distancia
        if State.KillAllEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local tool = GetEquippedTool()
            if tool then
                for _, plr in pairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                        local enemyHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if enemyHum and enemyHum.Health > 0 then
                            local targetPart = plr.Character:FindFirstChild(State.AimbotTargetPart) or plr.Character.HumanoidRootPart
                            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPart.Position)
                            
                            tool:Activate()
                            if State.DeviceMode == "PC" then
                                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                                task.wait(0.02)
                                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                            end
                            task.wait(0.03)
                        end
                    end
                end
            end
        end
    end
end)

-- Aura Pistola Loop
task.spawn(function()
    while true do
        task.wait(0.05)
        if State.AuraPistola and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local myHRP = LocalPlayer.Character.HumanoidRootPart
            local tool = GetEquippedTool()
            
            if tool then
                for _, plr in pairs(Players:GetPlayers()) do
                    if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                        local enemyHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if enemyHum and enemyHum.Health > 0 then
                            local enemyHRP = plr.Character.HumanoidRootPart
                            local distance = (myHRP.Position - enemyHRP.Position).Magnitude
                            
                            if distance <= State.AuraPistolaRange then
                                local targetPart = plr.Character:FindFirstChild("Head") or enemyHRP
                                Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPart.Position)
                                tool:Activate()
                                if State.DeviceMode == "PC" then
                                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                                    task.wait(0.01)
                                    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end)

-- Lógica de Movimiento
RunService.Stepped:Connect(function()
    if LocalPlayer.Character then
        if State.SpeedHack then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum.WalkSpeed = State.WalkSpeedValue end
        end
        if State.Noclip then
            for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                if not OriginalHitboxSizes[plr.Name] then
                    OriginalHitboxSizes[plr.Name] = hrp.Size
                end

                if State.HitboxExtender then
                    hrp.Size = Vector3.new(State.HitboxSize, State.HitboxSize, State.HitboxSize)
                    hrp.Transparency = 0.7
                    hrp.BrickColor = BrickColor.new("Really red")
                    hrp.Material = Enum.Material.Neon
                    hrp.CanCollide = false
                else
                    if OriginalHitboxSizes[plr.Name] then
                        hrp.Size = OriginalHitboxSizes[plr.Name]
                        hrp.Transparency = 1
                        hrp.CanCollide = true
                    end
                end
            end
        end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if State.InfiniteJump and LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

--------------------------------------------------------------------------------
-- MANEJO DE ATMÓSFERA Y CIELO RAINBOW
--------------------------------------------------------------------------------
local CustomSky = nil
local originalLightingState = {
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient,
    ColorShift_Top = Lighting.ColorShift_Top,
    ColorShift_Bottom = Lighting.ColorShift_Bottom,
    FogEnd = Lighting.FogEnd,
    FogStart = Lighting.FogStart
}

RunService.RenderStepped:Connect(function()
    local rainbowColor = Color3.fromHSV((tick() * State.RainbowSpeed * 0.1) % 1, 1, 1)

    -- Atmósfera Rainbow Neón
    if State.AtmosphereRainbow then
        Lighting.FogStart = 10000000
        Lighting.FogEnd = 10000000
        Lighting.Ambient = rainbowColor
        Lighting.OutdoorAmbient = rainbowColor
        Lighting.ColorShift_Top = rainbowColor
        Lighting.ColorShift_Bottom = rainbowColor
    end

    -- Cielo Rainbow Neón
    if State.SkyboxRainbow then
        if not CustomSky or CustomSky.Parent ~= Lighting then
            CustomSky = Lighting:FindFirstChild("RainbowSkybox")
            if not CustomSky then
                CustomSky = Instance.new("Sky")
                CustomSky.Name = "RainbowSkybox"
                CustomSky.Parent = Lighting
            end
        end
        
        CustomSky.SkyboxBk = "rbxassetid://159300201"
        CustomSky.SkyboxDn = "rbxassetid://159300201"
        CustomSky.SkyboxFt = "rbxassetid://159300201"
        CustomSky.SkyboxLf = "rbxassetid://159300201"
        CustomSky.SkyboxRt = "rbxassetid://159300201"
        CustomSky.SkyboxUp = "rbxassetid://159300201"
        CustomSky.SkyboxColor = rainbowColor
    else
        if CustomSky then
            CustomSky:Destroy()
            CustomSky = nil
        end
    end

    -- Personaje Rainbow
    if State.RainbowCharacter and LocalPlayer.Character then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Color = rainbowColor
            end
        end
    end
end)

-- Loop ESP & FOV
RunService.RenderStepped:Connect(function()
    Camera.FieldOfView = State.CameraFOV

    if State.AimbotEnabled then
        local target = GetClosestPlayerToCenter()
        if target and target.Character then
            local targetPart = target.Character:FindFirstChild(State.AimbotTargetPart) or target.Character.HumanoidRootPart
            local targetPos = targetPart.Position

            if State.AimbotPrediction and target.Character:FindFirstChild("HumanoidRootPart") then
                local vel = target.Character.HumanoidRootPart.Velocity
                targetPos = targetPos + (vel * 0.135)
            end

            local currentCFrame = Camera.CFrame
            local targetCFrame = CFrame.new(currentCFrame.Position, targetPos)
            Camera.CFrame = currentCFrame:Lerp(targetCFrame, State.AimbotSmoothing)
        end
    end

    if State.Spinbot and LocalPlayer.Character then
        local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(State.SpinSpeed), 0) end
    end

    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FOVCircle.Radius = State.FOVSize
    FOVCircle.Visible = State.AimbotFOV
    FOVCircle.Color = GetEffectiveColor("FOV", nil)

    local players = Players:GetPlayers()
    for i = 1, #players do
        local plr = players[i]
        if plr ~= LocalPlayer then
            local name = plr.Name
            local char = plr.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            local hrp = char and char:FindFirstChild("HumanoidRootPart")

            if char and hrp and hum and hum.Health > 0 then
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local visible = IsVisible(hrp, char)
                    local dynamicTargetColor = GetTargetColor(visible)

                    if not ESPCache[name] then
                        ESPCache[name] = {
                            Box = Drawing.new("Square"),
                            Tracer = Drawing.new("Line"),
                            Health = Drawing.new("Line"),
                            Info = Drawing.new("Text")
                        }
                        ESPCache[name].Info.Size = 11
                        ESPCache[name].Info.Center = true
                        ESPCache[name].Info.Outline = true
                    end

                    local cache = ESPCache[name]
                    local head = char:FindFirstChild("Head")
                    local topPos = head and (head.Position + Vector3.new(0, 1.2, 0)) or (hrp.Position + Vector3.new(0, 3, 0))
                    local bottomPos = hrp.Position - Vector3.new(0, 3.5, 0)
                    
                    local topScreen = Camera:WorldToViewportPoint(topPos)
                    local bottomScreen = Camera:WorldToViewportPoint(bottomPos)
                    local boxHeight = math.abs(bottomScreen.Y - topScreen.Y)
                    local boxWidth = boxHeight * 0.65

                    if State.ESPBoxes then
                        cache.Box.Size = Vector2.new(boxWidth, boxHeight)
                        cache.Box.Position = Vector2.new(pos.X - boxWidth / 2, topScreen.Y)
                        cache.Box.Color = GetEffectiveColor("Box", dynamicTargetColor)
                        cache.Box.Thickness = 1.5
                        cache.Box.Filled = false
                        cache.Box.Visible = true
                    else
                        cache.Box.Visible = false
                    end

                    if State.ESPTracers then
                        cache.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                        cache.Tracer.To = Vector2.new(pos.X, pos.Y)
                        cache.Tracer.Color = GetEffectiveColor("Tracer", dynamicTargetColor)
                        cache.Tracer.Thickness = 1.5
                        cache.Tracer.Visible = true
                    else
                        cache.Tracer.Visible = false
                    end

                    if State.ESPHealthBar then
                        local pct = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
                        cache.Health.From = Vector2.new(pos.X - boxWidth / 2 - 5, topScreen.Y + boxHeight)
                        cache.Health.To = Vector2.new(pos.X - boxWidth / 2 - 5, topScreen.Y + boxHeight - (boxHeight * pct))
                        cache.Health.Color = Color3.fromRGB(255 - (255 * pct), 255 * pct, 0)
                        cache.Health.Thickness = 2
                        cache.Health.Visible = true
                    else
                        cache.Health.Visible = false
                    end

                    if State.ESPInfo then
                        local dist = math.floor((hrp.Position - Camera.CFrame.Position).Magnitude)
                        cache.Info.Text = string.format("%s [%dm]", plr.Name, dist)
                        cache.Info.Position = Vector2.new(pos.X, topScreen.Y - 14)
                        cache.Info.Color = GetEffectiveColor("Info", Color3.fromRGB(255, 255, 255))
                        cache.Info.Visible = true
                    else
                        cache.Info.Visible = false
                    end
                else
                    if ESPCache[name] then
                        ESPCache[name].Box.Visible = false
                        ESPCache[name].Tracer.Visible = false
                        ESPCache[name].Health.Visible = false
                        ESPCache[name].Info.Visible = false
                    end
                end
            else
                RemoveESPForPlayer(name)
            end
        end
    end
end)

-- Registros de Controles con nuevos valores máximos
CreateToggle(CombatTab, "Aimbot", "AimbotEnabled", function(val) end)
CreateToggle(CombatTab, "Silent Aimbot", "SilentAimbot", function(val) end)
CreateSlider(CombatTab, "Probabilidad Silent (%)", 1, 100, 100, function(val) State.SilentHitchance = val end, "SilentHitchance")
CreateToggle(CombatTab, "Triggerbot (Disparo Aut.)", "TriggerbotEnabled", function(val) end)
CreateButton(CombatTab, "Obtener Teleport Tool", GiveTeleportTool)
CreateSlider(CombatTab, "Suavizado Aimbot", 1, 100, 15, function(val) State.AimbotSmoothing = val / 100 end)
CreateToggle(CombatTab, "Kill All (Pistola)", "KillAllEnabled", function(val) end)
CreateToggle(CombatTab, "Kill All (Cuchillo/Melee)", "KillAllMelee", function(val) end)
CreateToggle(CombatTab, "Aura Pistola (Free Fire)", "AuraPistola", function(val) end)
CreateSlider(CombatTab, "Distancia Aura Pistola", 10, 500, 50, function(val) State.AuraPistolaRange = val end, "AuraPistolaRange")
CreateToggle(CombatTab, "Predicción Aimbot", "AimbotPrediction", function(val) end)
CreateDropdown(CombatTab, "Parte Objetivo", {"Head", "HumanoidRootPart"}, function(val) State.AimbotTargetPart = val end)
CreateToggle(CombatTab, "Wallbang / Atraviesa Paredes", "Wallbang", function(val) end)
CreateToggle(CombatTab, "Aumentar Hitbox", "HitboxExtender", function(val) end)
CreateSlider(CombatTab, "Tamaño Hitbox", 2, 100, 5, function(val) State.HitboxSize = val end, "HitboxSize")
CreateToggle(CombatTab, "Ver Círculo FOV", "AimbotFOV", function(val) end)
CreateSlider(CombatTab, "Tamaño FOV Aimbot", 50, 800, 150, function(val) State.FOVSize = val end, "FOVSize")

CreateToggle(VisualsTab, "Atmósfera Rainbow Neón", "AtmosphereRainbow", function(val)
    if not val then
        Lighting.Ambient = originalLightingState.Ambient
        Lighting.OutdoorAmbient = originalLightingState.OutdoorAmbient
        Lighting.ColorShift_Top = originalLightingState.ColorShift_Top
        Lighting.ColorShift_Bottom = originalLightingState.ColorShift_Bottom
        Lighting.FogStart = originalLightingState.FogStart
        Lighting.FogEnd = originalLightingState.FogEnd
    end
end)
CreateToggle(VisualsTab, "Cielo Rainbow", "SkyboxRainbow", function(val) end)
CreateToggle(VisualsTab, "Personaje Skin Rainbow", "RainbowCharacter", function(val) end)
CreateSlider(VisualsTab, "Velocidad Rainbow", 1, 100, 1, function(val) State.RainbowSpeed = val / 2 end, "RainbowSpeed")
CreateToggle(VisualsTab, "Efecto Rainbow ESP", "ESPRainbow", function(val) end)
CreateToggle(VisualsTab, "Colores Dinámicos", "DynamicColors", function(val) end)
CreateToggle(VisualsTab, "Cajas ESP", "ESPBoxes", function(val) end)
CreateToggle(VisualsTab, "Líneas ESP", "ESPTracers", function(val) end)
CreateToggle(VisualsTab, "Barra de Vida", "ESPHealthBar", function(val) end)
CreateToggle(VisualsTab, "Información Detallada", "ESPInfo", function(val) end)

-- Opciones de Movimiento & Cámara
CreateToggle(MovementTab, "Tercera Persona", "ThirdPerson", function(val) end)
CreateSlider(MovementTab, "Distancia Cámara 3ra Persona", 5, 200, 12, function(val) State.ThirdPersonDistance = val end, "ThirdPersonDistance")
CreateToggle(MovementTab, "Fly (Volar)", "Fly", function(val) end)
CreateSlider(MovementTab, "Velocidad Fly", 10, 500, 50, function(val) State.FlySpeed = val end, "FlySpeed")
CreateToggle(MovementTab, "Noclip", "Noclip", function(val) end)
CreateToggle(MovementTab, "Speed Hack", "SpeedHack", function(val)
    if not val and LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = 16 end
    end
end)
CreateSlider(MovementTab, "Velocidad Caminar", 16, 500, 100, function(val) State.WalkSpeedValue = val end, "WalkSpeedValue")
CreateToggle(MovementTab, "Spinbot", "Spinbot", function(val) end)
CreateSlider(MovementTab, "Velocidad Spinbot", 5, 300, 30, function(val) State.SpinSpeed = val end, "SpinSpeed")
CreateSlider(MovementTab, "FOV Cámara", 30, 120, 70, function(val) State.CameraFOV = val end, "CameraFOV")
CreateToggle(MovementTab, "Salto Infinito", "InfiniteJump", function(val) end)

CreateToggle(UtilityTab, "Mostrar FPS & Ping", "ShowWatermark", function(val)
    WatermarkFrame.Visible = val
end)

CreateToggle(UtilityTab, "Boost FPS (Rendimiento)", "FPSBoost", function(val)
    ApplyPerformanceBoost(val)
end)

CreateToggle(UtilityTab, "Anti-AFK", "AntiAFK", function(val)
    if val then
        LocalPlayer.Idled:Connect(function()
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
            task.wait(0.2)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
        end)
    end
end)

CreateButton(SettingsTab, "Guardar Configuración (JSON)", function()
    SaveConfig()
    SendNotification("Guardado", "Configuración y atajos guardados.")
end)

CreateButton(SettingsTab, "Cargar Configuración (JSON)", function()
    LoadConfig()
    SendNotification("Cargado", "Sincronización visual completada.")
end)

CreateButton(SettingsTab, "🔄 Reset Parámetros (Default)", function()
    ResetConfig()
    SendNotification("Reset", "Parámetros restaurados a valores por defecto.")
end)

CreateDropdown(SettingsTab, "Seleccionar Tema", {"RedMatte", "Cyberpunk", "Midnight", "Emerald"}, function(val)
    CurrentTheme = Themes[val]
    MainFrame.BackgroundColor3 = CurrentTheme.MainColor
    Topbar.BackgroundColor3 = CurrentTheme.ContainerColor
    MainStroke.Color = CurrentTheme.AccentColor
    WMStroke.Color = CurrentTheme.AccentColor
    WatermarkFrame.BackgroundColor3 = CurrentTheme.MainColor
    CloseBtn.BackgroundColor3 = CurrentTheme.AccentColor
    
    for _, item in pairs(RegisteredContainers) do
        if item.Role == "ContainerColor" then
            item.Element[item.Property] = CurrentTheme.ContainerColor
        elseif item.Role == "AccentColor" then
            item.Element[item.Property] = CurrentTheme.AccentColor
        end
    end
    
    for _, item in pairs(RegisteredSwitches) do
        if item.GetState() then
            item.Track.BackgroundColor3 = CurrentTheme.AccentColor
        end
    end
    
    SendNotification("Tema Aplicado", CurrentTheme.Name)
end)

-- Carga inicial automática
LoadConfig()
UpdateDeviceModeUI()
SendNotification("RAGE HUB V9", "Menú inicializado correctamente.")


