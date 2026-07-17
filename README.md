--[[
    VENON MENU [EXTERNAL]
    Atualização: Aba Jogador Removida, Novo Sistema de Hitbox Modular na MAIN, Notificações Únicas.
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ==========================================
-- CORES E SISTEMA DE CONFIGURAÇÃO VENON
-- ==========================================
local Theme = {
    Background = Color3.fromRGB(28, 28, 30),    
    SidebarBg = Color3.fromRGB(22, 22, 24),     
    CardBg = Color3.fromRGB(38, 38, 40),        
    Accent = Color3.fromRGB(150, 150, 150),     
    Neon = Color3.fromRGB(210, 210, 210),       
    TextLight = Color3.fromRGB(230, 230, 230),  
    TextMuted = Color3.fromRGB(130, 130, 135),  
}

local DefaultGray = Color3.fromRGB(160, 160, 160)

local Config = {
    Aimbot = { Enabled = false, TargetPart = "Head", Smoothness = 5 },
    RageAimbot = { Enabled = false },
    Hitbox = { Enabled = false, Size = 2, Part = "Cabeça" }, -- NOVO SISTEMA HITBOX
    Player = { Wallbang = false },
    FOV = { Visible = false, Size = 100, Thickness = 2, RGB = false },
    Visuals = {
        Box = false, Line = false, Health = false, Name = false,
        Distance = false, Chams = false, Skeleton = false,
        MaxDistance = 600, SkeletonMaxDistance = 500,
    },
    Misc = { 
        SpinbotEnabled = false, SpinbotSpeed = 30
    },
    SelectedPlayer = nil,
    SelectedVehicle = nil,
    Colors = {
        Box = DefaultGray,
        Skeleton = DefaultGray,
        Line = DefaultGray,
        Name = Color3.fromRGB(255, 255, 255),
        Distance = Color3.fromRGB(200, 200, 200),
        Health = Color3.fromRGB(0, 255, 102),
        Chams = DefaultGray,
        FOV = DefaultGray,
    }
}

local CustomColorPresets = {
    {Name = "Cinza Fosco", Color = Color3.fromRGB(160, 160, 160)},
    {Name = "Prata", Color = Color3.fromRGB(210, 210, 210)},
    {Name = "Chumbo", Color = Color3.fromRGB(80, 80, 80)},
    {Name = "Branco", Color = Color3.fromRGB(255, 255, 255)},
    {Name = "Preto", Color = Color3.fromRGB(0, 0, 0)},
    {Name = "Vermelho", Color = Color3.fromRGB(255, 60, 60)},
    {Name = "Azul", Color = Color3.fromRGB(60, 150, 255)},
    {Name = "Verde", Color = Color3.fromRGB(60, 255, 100)},
    {Name = "Roxo", Color = Color3.fromRGB(150, 80, 255)},
    {Name = "Amarelo", Color = Color3.fromRGB(255, 220, 60)}
}

-- ==========================================
-- CRIAÇÃO DA TELA
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "Venon_MasterEngine"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Global
ScreenGui.IgnoreGuiInset = true

local success = pcall(function() ScreenGui.Parent = CoreGui end)
if not success then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

-- ==========================================
-- SISTEMA DE NOTIFICAÇÃO
-- ==========================================
local NotifyContainer = Instance.new("Frame", ScreenGui)
NotifyContainer.Name = "Venon_Notifications"
NotifyContainer.Size = UDim2.new(0, 300, 1, -20)
NotifyContainer.Position = UDim2.new(1, -310, 0, 10)
NotifyContainer.BackgroundTransparency = 1
NotifyContainer.ZIndex = 100

local NotifyLayout = Instance.new("UIListLayout", NotifyContainer)
NotifyLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
NotifyLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
NotifyLayout.Padding = UDim.new(0, 10)

local function ShowNotification(textMsg)
    -- Remove notificações antigas para aparecerem individualmente
    for _, child in ipairs(NotifyContainer:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end

    local Notif = Instance.new("Frame", NotifyContainer)
    Notif.Size = UDim2.new(1, 0, 0, 45)
    Notif.Position = UDim2.new(1, 350, 0, 0)
    Notif.BackgroundColor3 = Theme.CardBg
    Notif.ClipsDescendants = true
    Instance.new("UICorner", Notif).CornerRadius = UDim.new(0, 6)
    
    local Stroke = Instance.new("UIStroke", Notif)
    Stroke.Color = Theme.Accent
    Stroke.Thickness = 1
    
    local AccentLine = Instance.new("Frame", Notif)
    AccentLine.Size = UDim2.new(0, 4, 1, 0)
    AccentLine.BackgroundColor3 = Theme.Neon
    AccentLine.BorderSizePixel = 0
    Instance.new("UICorner", AccentLine).CornerRadius = UDim.new(0, 6)
    
    local Lbl = Instance.new("TextLabel", Notif)
    Lbl.Size = UDim2.new(1, -20, 1, 0)
    Lbl.Position = UDim2.new(0, 15, 0, 0)
    Lbl.BackgroundTransparency = 1
    Lbl.Text = textMsg
    Lbl.TextColor3 = Theme.TextLight
    Lbl.Font = Enum.Font.GothamMedium
    Lbl.TextSize = 13
    Lbl.TextXAlignment = Enum.TextXAlignment.Left
    
    TweenService:Create(Notif, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0, 0, 0, 0)}):Play()
    
    task.delay(3, function()
        if Notif.Parent then
            local out = TweenService:Create(Notif, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Position = UDim2.new(1, 350, 0, 0)})
            out:Play()
            out.Completed:Connect(function() Notif:Destroy() end)
        end
    end)
end

-- ==========================================
-- MENU PRINCIPAL
-- ==========================================
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 750, 0, 480)
MainFrame.Position = UDim2.new(0.5, 0, 0.2, 0)
MainFrame.AnchorPoint = Vector2.new(0.5, 0)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Visible = false 
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 1.5; MainStroke.Color = Theme.Accent

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundTransparency = 1 
TopBar.BorderSizePixel = 0

local HeaderGlow = Instance.new("Frame", TopBar)
HeaderGlow.Size = UDim2.new(1, 0, 0, 3); HeaderGlow.BackgroundColor3 = Theme.Neon; HeaderGlow.BorderSizePixel = 0

local TopTitle = Instance.new("TextLabel", TopBar)
TopTitle.Size = UDim2.new(1, -60, 1, 0); TopTitle.Position = UDim2.new(0, 15, 0, 0)
TopTitle.BackgroundTransparency = 1; TopTitle.Text = "VENON MENU"
TopTitle.TextColor3 = Theme.TextLight; TopTitle.Font = Enum.Font.GothamBold; TopTitle.TextSize = 14
TopTitle.TextXAlignment = Enum.TextXAlignment.Left

local MinimizeBtn = Instance.new("TextButton", TopBar)
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30); MinimizeBtn.Position = UDim2.new(1, -40, 0.5, -15)
MinimizeBtn.BackgroundTransparency = 1
MinimizeBtn.Text = "—"; MinimizeBtn.TextColor3 = Theme.Neon; MinimizeBtn.Font = Enum.Font.GothamBold; MinimizeBtn.TextSize = 16

local ContentGroup = Instance.new("CanvasGroup", MainFrame)
ContentGroup.Size = UDim2.new(1, 0, 1, -40)
ContentGroup.Position = UDim2.new(0, 0, 0, 40)
ContentGroup.BackgroundTransparency = 1
ContentGroup.BorderSizePixel = 0

-- ==========================================
-- SISTEMA DE KEY
-- ==========================================
local ValidKeys = { ["DELTA-123"] = true, ["DELTA-443"] = true, ["MANUS-67"] = true }

local KeyGroup = Instance.new("CanvasGroup", ScreenGui)
KeyGroup.Size = UDim2.new(0, 300, 0, 170) 
KeyGroup.Position = UDim2.new(0.5, 0, 0.5, 0)
KeyGroup.AnchorPoint = Vector2.new(0.5, 0.5)
KeyGroup.BackgroundColor3 = Theme.Background
KeyGroup.GroupTransparency = 1 
Instance.new("UICorner", KeyGroup).CornerRadius = UDim.new(0, 12)
local KeyStroke = Instance.new("UIStroke", KeyGroup)
KeyStroke.Color = Theme.Accent
KeyStroke.Thickness = 1.5

local KeyTitle = Instance.new("TextLabel", KeyGroup)
KeyTitle.Size = UDim2.new(1, 0, 0, 40); KeyTitle.Position = UDim2.new(0, 0, 0, 15)
KeyTitle.BackgroundTransparency = 1; KeyTitle.Text = "VENON SECURITY"
KeyTitle.TextColor3 = Theme.Neon; KeyTitle.Font = Enum.Font.GothamBold; KeyTitle.TextSize = 18

local KeySub = Instance.new("TextLabel", KeyGroup)
KeySub.Size = UDim2.new(1, 0, 0, 20); KeySub.Position = UDim2.new(0, 0, 0, 45)
KeySub.BackgroundTransparency = 1; KeySub.Text = "Insira sua Key para autenticar"
KeySub.TextColor3 = Theme.TextMuted; KeySub.Font = Enum.Font.GothamMedium; KeySub.TextSize = 12

local KeyInputBase = Instance.new("Frame", KeyGroup)
KeyInputBase.Size = UDim2.new(0.8, 0, 0, 40); KeyInputBase.Position = UDim2.new(0.1, 0, 0.45, 0)
KeyInputBase.BackgroundColor3 = Theme.CardBg
Instance.new("UICorner", KeyInputBase).CornerRadius = UDim.new(0, 8)
local KeyInputStroke = Instance.new("UIStroke", KeyInputBase)
KeyInputStroke.Color = Color3.fromRGB(60, 60, 65)

local KeyInput = Instance.new("TextBox", KeyInputBase)
KeyInput.Size = UDim2.new(1, -20, 1, 0); KeyInput.Position = UDim2.new(0, 10, 0, 0)
KeyInput.BackgroundTransparency = 1; KeyInput.Text = ""
KeyInput.PlaceholderText = "Cole sua Key aqui..."
KeyInput.TextColor3 = Theme.TextLight; KeyInput.PlaceholderColor3 = Theme.TextMuted
KeyInput.Font = Enum.Font.GothamMedium; KeyInput.TextSize = 12
KeyInput.TextXAlignment = Enum.TextXAlignment.Left

local VerifyBtn = Instance.new("TextButton", KeyGroup)
VerifyBtn.Size = UDim2.new(0.8, 0, 0, 35); VerifyBtn.Position = UDim2.new(0.1, 0, 0.75, 0)
VerifyBtn.BackgroundColor3 = Theme.Accent; VerifyBtn.Text = "VERIFICAR KEY"
VerifyBtn.TextColor3 = Theme.Background; VerifyBtn.Font = Enum.Font.GothamBold; VerifyBtn.TextSize = 12
VerifyBtn.AutoButtonColor = false
Instance.new("UICorner", VerifyBtn).CornerRadius = UDim.new(0, 6)

VerifyBtn.MouseEnter:Connect(function() TweenService:Create(VerifyBtn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.Neon}):Play() end)
VerifyBtn.MouseLeave:Connect(function() TweenService:Create(VerifyBtn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.Accent}):Play() end)

local function ShakeUI()
    local orig = KeyGroup.Position
    for i = 1, 4 do TweenService:Create(KeyGroup, TweenInfo.new(0.04), {Position = orig + UDim2.new(0, math.random(-8, 8), 0, 0)}):Play(); task.wait(0.04) end
    TweenService:Create(KeyGroup, TweenInfo.new(0.04), {Position = orig}):Play()
    KeyInputStroke.Color = Color3.fromRGB(255, 60, 60)
    task.wait(0.4)
    KeyInputStroke.Color = Color3.fromRGB(60, 60, 65)
end

VerifyBtn.MouseButton1Click:Connect(function()
    local inputtedKey = KeyInput.Text:match("^%s*(.-)%s*$")
    if ValidKeys[inputtedKey] then
        VerifyBtn.Text = "SUCESSO!"
        TweenService:Create(VerifyBtn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(50, 220, 100)}):Play()
        KeyInputStroke.Color = Color3.fromRGB(50, 220, 100)
        task.wait(0.3)
        local fadeOut = TweenService:Create(KeyGroup, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {GroupTransparency = 1, Size = UDim2.new(0, 300, 0, 170)})
        fadeOut:Play()
        fadeOut.Completed:Connect(function()
            KeyGroup:Destroy()
            MainFrame.Size = UDim2.new(0, 650, 0, 400) 
            MainFrame.Visible = true
            TweenService:Create(MainFrame, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 750, 0, 480)}):Play()
        end)
    else
        VerifyBtn.Text = "KEY INVÁLIDA"
        ShakeUI()
        task.wait(0.4)
        VerifyBtn.Text = "VERIFICAR KEY"
    end
end)

TweenService:Create(KeyGroup, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 350, 0, 210), GroupTransparency = 0}):Play()

-- ==========================================
-- LOGO E SIDEBAR
-- ==========================================
local LogoContainer = Instance.new("Frame", ContentGroup)
LogoContainer.Size = UDim2.new(0, 180, 0, 100); LogoContainer.Position = UDim2.new(1, -190, 0, 15); LogoContainer.BackgroundTransparency = 1

local LogoTextContainer = Instance.new("Frame", LogoContainer)
LogoTextContainer.Size = UDim2.new(1, 0, 1, 0); LogoTextContainer.BackgroundTransparency = 1

local LogoV = Instance.new("TextLabel", LogoTextContainer)
LogoV.Size = UDim2.new(1, 0, 0, 55); LogoV.BackgroundTransparency = 1; LogoV.Text = "V"
LogoV.TextColor3 = Theme.Neon; LogoV.Font = Enum.Font.GothamBlack; LogoV.TextSize = 65; LogoV.TextYAlignment = Enum.TextYAlignment.Bottom

local LogoVenon = Instance.new("TextLabel", LogoTextContainer)
LogoVenon.Size = UDim2.new(1, 0, 0, 25); LogoVenon.Position = UDim2.new(0, 0, 0, 60)
LogoVenon.BackgroundTransparency = 1; LogoVenon.Text = "VENON MENU"
LogoVenon.TextColor3 = Theme.Neon; LogoVenon.Font = Enum.Font.GothamBold; LogoVenon.TextSize = 18

local LogoComm = Instance.new("TextLabel", LogoTextContainer)
LogoComm.Size = UDim2.new(1, 0, 0, 15); LogoComm.Position = UDim2.new(0, 0, 0, 85)
LogoComm.BackgroundTransparency = 1; LogoComm.Text = "COMMUNITY UI"
LogoComm.TextColor3 = Theme.TextMuted; LogoComm.Font = Enum.Font.GothamBold; LogoComm.TextSize = 10

local Sidebar = Instance.new("Frame", ContentGroup)
Sidebar.Size = UDim2.new(0, 160, 1, 0); Sidebar.BackgroundColor3 = Theme.SidebarBg; Sidebar.BorderSizePixel = 0
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 12)

local TabsContainer = Instance.new("Frame", Sidebar)
TabsContainer.Size = UDim2.new(1, 0, 1, -80); TabsContainer.Position = UDim2.new(0, 0, 0, 40); TabsContainer.BackgroundTransparency = 1
local TabsLayout = Instance.new("UIListLayout", TabsContainer)
TabsLayout.Padding = UDim.new(0, 8); TabsLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

local PagesContainer = Instance.new("Frame", ContentGroup)
PagesContainer.Size = UDim2.new(1, -345, 1, -30); PagesContainer.Position = UDim2.new(0, 175, 0, 15); PagesContainer.BackgroundTransparency = 1

local Pages, TabButtons = {}, {}

-- ==========================================
-- SISTEMA DE ROLAGEM E ABAS
-- ==========================================
local function CreateTab(name, isFirst)
    local TabButton = Instance.new("TextButton", TabsContainer)
    TabButton.Size = UDim2.new(0.9, 0, 0, 36)
    TabButton.BackgroundColor3 = isFirst and Theme.CardBg or Color3.fromRGB(0,0,0)
    TabButton.BackgroundTransparency = isFirst and 0 or 1
    TabButton.Text = "  " .. name
    TabButton.TextColor3 = isFirst and Theme.Neon or Theme.TextMuted
    TabButton.Font = Enum.Font.GothamMedium; TabButton.TextSize = 12
    TabButton.TextXAlignment = Enum.TextXAlignment.Left; TabButton.AutoButtonColor = false
    Instance.new("UICorner", TabButton).CornerRadius = UDim.new(0, 6)

    local Indicator = Instance.new("Frame", TabButton)
    Indicator.Size = UDim2.new(0, 3, 0.5, 0); Indicator.Position = UDim2.new(0, 0, 0.25, 0)
    Indicator.BackgroundColor3 = Theme.Neon; Indicator.BorderSizePixel = 0; Indicator.Visible = isFirst

    local Page = Instance.new("ScrollingFrame", PagesContainer)
    Page.Size = UDim2.new(1, 0, 1, 0); Page.BackgroundTransparency = 1; Page.BorderSizePixel = 0
    Page.ScrollBarThickness = 3; Page.ScrollBarImageColor3 = Theme.Accent
    Page.Visible = isFirst; Page.ClipsDescendants = true
    Page.AutomaticCanvasSize = Enum.AutomaticSize.Y

    local PageLayout = Instance.new("UIListLayout", Page)
    PageLayout.Padding = UDim.new(0, 15); PageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

    Pages[name] = Page; TabButtons[name] = {Button = TabButton, Indicator = Indicator}

    TabButton.MouseButton1Click:Connect(function()
        for tName, data in pairs(TabButtons) do
            TweenService:Create(data.Button, TweenInfo.new(0.2), { BackgroundTransparency = 1, TextColor3 = Theme.TextMuted }):Play()
            data.Indicator.Visible = false
            Pages[tName].Visible = false
        end
        TweenService:Create(TabButton, TweenInfo.new(0.2), { BackgroundTransparency = 0, BackgroundColor3 = Theme.CardBg, TextColor3 = Theme.Neon }):Play()
        Indicator.Visible = true; Page.Visible = true
    end)
    return Page
end

local PageMain = CreateTab("MAIN", true)
local PageVisual = CreateTab("VISUAL", false)
local PageTeleport = CreateTab("TELEPORT", false)
local PageVehicle = CreateTab("VEHICLE", false)
local PageMisc = CreateTab("MISC", false)

-- ==========================================
-- COMPONENTES DA UI
-- ==========================================
local function CreateCard(parent, title)
    local Card = Instance.new("Frame", parent)
    Card.Size = UDim2.new(0.98, 0, 0, 40); Card.BackgroundColor3 = Theme.CardBg; Card.BorderSizePixel = 0
    Instance.new("UICorner", Card).CornerRadius = UDim.new(0, 8)
    local Stroke = Instance.new("UIStroke", Card); Stroke.Color = Color3.fromRGB(60, 60, 65); Stroke.Thickness = 1

    local TitleLabel = Instance.new("TextLabel", Card)
    TitleLabel.Size = UDim2.new(1, -20, 0, 30); TitleLabel.Position = UDim2.new(0, 10, 0, 5)
    TitleLabel.Text = title:upper(); TitleLabel.TextColor3 = Theme.Neon
    TitleLabel.Font = Enum.Font.GothamBold; TitleLabel.TextSize = 12
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left; TitleLabel.BackgroundTransparency = 1

    local Content = Instance.new("Frame", Card)
    Content.Size = UDim2.new(1, -20, 1, -35); Content.Position = UDim2.new(0, 10, 0, 35)
    Content.BackgroundTransparency = 1
    local ContentLayout = Instance.new("UIListLayout", Content)
    ContentLayout.Padding = UDim.new(0, 12); ContentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    ContentLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        Card.Size = UDim2.new(0.98, 0, 0, ContentLayout.AbsoluteContentSize.Y + 45)
    end)
    return Content
end

local function CreateToggle(parent, text, callback)
    local Frame = Instance.new("Frame", parent)
    Frame.Size = UDim2.new(1, 0, 0, 30); Frame.BackgroundTransparency = 1

    local Label = Instance.new("TextLabel", Frame)
    Label.Size = UDim2.new(0.7, 0, 1, 0); Label.BackgroundTransparency = 1; Label.Text = text
    Label.TextColor3 = Theme.TextLight; Label.Font = Enum.Font.GothamSemibold; Label.TextSize = 12
    Label.TextXAlignment = Enum.TextXAlignment.Left

    local Btn = Instance.new("TextButton", Frame)
    Btn.Size = UDim2.new(0, 42, 0, 20); Btn.Position = UDim2.new(1, -42, 0.5, -10)
    Btn.BackgroundColor3 = Color3.fromRGB(55, 55, 60); Btn.Text = ""; Btn.AutoButtonColor = false
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(1, 0)

    local Indicator = Instance.new("Frame", Btn)
    Indicator.Size = UDim2.new(0, 14, 0, 14); Indicator.Position = UDim2.new(0, 3, 0.5, -7)
    Indicator.BackgroundColor3 = Theme.TextMuted; Indicator.BorderSizePixel = 0
    Instance.new("UICorner", Indicator).CornerRadius = UDim.new(1, 0)

    local active = false
    Btn.MouseButton1Click:Connect(function()
        active = not active
        TweenService:Create(Indicator, TweenInfo.new(0.2), {
            Position = active and UDim2.new(1, -17, 0.5, -7) or UDim2.new(0, 3, 0.5, -7),
            BackgroundColor3 = active and Theme.Neon or Theme.TextMuted
        }):Play()
        TweenService:Create(Btn, TweenInfo.new(0.2), { BackgroundColor3 = active and Color3.fromRGB(90, 90, 95) or Color3.fromRGB(55, 55, 60) }):Play()
        task.spawn(function() callback(active) end)
    end)
end

local function CreateSlider(parent, text, min, max, default, callback)
    local Frame = Instance.new("Frame", parent)
    Frame.Size = UDim2.new(1, 0, 0, 45); Frame.BackgroundTransparency = 1

    local Label = Instance.new("TextLabel", Frame)
    Label.Size = UDim2.new(0.7, 0, 0, 20); Label.BackgroundTransparency = 1; Label.Text = text
    Label.TextColor3 = Theme.TextLight; Label.Font = Enum.Font.GothamSemibold; Label.TextSize = 12; Label.TextXAlignment = Enum.TextXAlignment.Left

    local ValueLbl = Instance.new("TextLabel", Frame)
    ValueLbl.Size = UDim2.new(0.3, 0, 0, 20); ValueLbl.Position = UDim2.new(0.7, 0, 0, 0)
    ValueLbl.BackgroundTransparency = 1; ValueL