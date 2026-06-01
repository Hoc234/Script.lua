--// ================================================================ //--
--// Beta HUB - PHIÊN BẢN BANANA (V5 - ĐÃ SỬA LỖI & ĐẦY ĐỦ)    --//
--// ================================================================ //--

local CONFIG = {
    HubName = "Beta HUB",
    Creator = "Beta Dev",
    LoadingText = "Đang tải Beta Hub",
    
    -- ID ROBLOX - HÃY THAY THẾ CÁC ID NÀY ĐỂ HUB ĐẸP HƠN
    LogoID = "rbxassetid://120488231660846", -- Ảnh Logo hình tròn (có thể thay bằng logo quả chuối)
    BackgroundID = "rbxassetid://120488231660846", -- Ảnh nền Menu (có thể thay bằng tông vàng)
    MusicID = "rbxassetid://1837879082", -- Nhạc nền (có thể đổi thành nhạc vui nhộn)
    
    TotalMembers = "978",
    OnlineMembers = "342",
    Description = "🍌 Chào mừng đến với Beta Hub! 🍌\nPhiên bản Banana siêu mượt mà và ngọt ngào.\nNơi cung cấp Script chất lượng, an toàn và ổn định.\nCập nhật liên tục để mang đến trải nghiệm tốt nhất.",

    -- Màu sắc theo phong cách Chuối / Banana
    MainColor = Color3.fromRGB(255, 225, 0),      -- Vàng chuối đậm (Chuối chín)
    GradientColor = Color3.fromRGB(255, 240, 100), -- Vàng nhạt hơn cho gradient
    SecondaryColor = Color3.fromRGB(30, 25, 0),   -- Nền tối pha vàng đất (thay vì đen tuyền)
    
    FontMain = Enum.Font.Ubuntu,
    FontBold = Enum.Font.GothamBold,
    ToggleKey = Enum.KeyCode.RightControl
}

--// ================================================================ //--

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")

local UI_NAME = "BetaHub_Banana"
local playerGui = Players.LocalPlayer:WaitForChild("PlayerGui")
if playerGui:FindFirstChild(UI_NAME) then playerGui[UI_NAME]:Destroy() end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = UI_NAME
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = playerGui

--// SETUP ÂM THANH
local function CreateSound(id, volume, loop)
    local snd = Instance.new("Sound", ScreenGui)
    snd.SoundId = id
    snd.Volume = volume
    snd.Looped = loop or false
    return snd
end
local BgMusic = CreateSound(CONFIG.MusicID, 0.4, true)
local HoverSound = CreateSound("rbxassetid://6895086153", 0.5)
local ClickSound = CreateSound("rbxassetid://6895079853", 0.5)

--// MODULE: KÉO THẢ MƯỢT
local function MakeDraggable(gui)
    local dragging, dragInput, dragStart, startPos
    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position
        end
    end)
    gui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            TweenService:Create(gui, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            }):Play()
        end
    end)
    gui.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            dragInput = nil
        end
    end)
end

--// MODULE: THÔNG BÁO
local function SendNotification(title, text, duration)
    local NotifFrame = Instance.new("Frame", ScreenGui)
    NotifFrame.Size = UDim2.new(0, 280, 0, 70)
    NotifFrame.Position = UDim2.new(1, 10, 0.85, 0)
    NotifFrame.BackgroundColor3 = CONFIG.SecondaryColor
    NotifFrame.ZIndex = 10
    
    Instance.new("UICorner", NotifFrame).CornerRadius = UDim.new(0, 8)
    
    local Stroke = Instance.new("UIStroke", NotifFrame)
    Stroke.Color = CONFIG.MainColor
    Stroke.Thickness = 1.5

    local TitleLabel = Instance.new("TextLabel", NotifFrame)
    TitleLabel.Size = UDim2.new(1, -20, 0, 30)
    TitleLabel.Position = UDim2.new(0, 10, 0, 5)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Text = "🍌 " .. title
    TitleLabel.TextColor3 = CONFIG.MainColor
    TitleLabel.Font = CONFIG.FontBold
    TitleLabel.TextSize = 16
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left

    local Msg = Instance.new("TextLabel", NotifFrame)
    Msg.Size = UDim2.new(1, -20, 0, 30)
    Msg.Position = UDim2.new(0, 10, 0, 30)
    Msg.BackgroundTransparency = 1
    Msg.Text = text
    Msg.TextColor3 = Color3.new(0.9, 0.9, 0.8)
    Msg.Font = CONFIG.FontMain
    Msg.TextSize = 14
    Msg.TextXAlignment = Enum.TextXAlignment.Left
    Msg.TextWrapped = true

    NotifFrame:TweenPosition(UDim2.new(1, -290, 0.85, 0), "Out", "Quad", 0.5, true)
    task.delay(duration or 3, function()
        NotifFrame:TweenPosition(UDim2.new(1, 10, 0.85, 0), "In", "Quad", 0.5, true)
        task.wait(0.5)
        NotifFrame:Destroy()
    end)
end

--// MODULE: TẢI GIAO DIỆN
local function StartLoading(callback)
    local Blur = Instance.new("BlurEffect", Lighting)
    Blur.Size = 0
    TweenService:Create(Blur, TweenInfo.new(1), {Size = 20}):Play()

    local LoadFrame = Instance.new("Frame", ScreenGui)
    LoadFrame.Size = UDim2.new(1, 0, 1, 0)
    LoadFrame.BackgroundColor3 = Color3.new(0, 0, 0)
    LoadFrame.BackgroundTransparency = 1
    LoadFrame.ZIndex = 10
    TweenService:Create(LoadFrame, TweenInfo.new(0.5), {BackgroundTransparency = 0.5}):Play()

    local Center = Instance.new("Frame", LoadFrame)
    Center.Size = UDim2.new(0, 400, 0, 200)
    Center.Position = UDim2.new(0.5, 0, 0.5, 0)
    Center.AnchorPoint = Vector2.new(0.5, 0.5)
    Center.BackgroundTransparency = 1

    local Title = Instance.new("TextLabel", Center)
    Title.Size = UDim2.new(1, 0, 0, 50)
    Title.Text = "🍌 " .. CONFIG.LoadingText .. " 🍌"
    Title.TextColor3 = Color3.new(1,1,1)
    Title.Font = CONFIG.FontBold
    Title.TextSize = 35
    Title.BackgroundTransparency = 1
    
    local UIGradient = Instance.new("UIGradient", Title)
    UIGradient.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, CONFIG.MainColor), ColorSequenceKeypoint.new(1, CONFIG.GradientColor)}

    local SubTitle = Instance.new("TextLabel", Center)
    SubTitle.Size = UDim2.new(1, 0, 0, 30)
    SubTitle.Position = UDim2.new(0, 0, 0, 45)
    SubTitle.Text = "Được tạo bởi " .. CONFIG.Creator
    SubTitle.TextColor3 = Color3.new(0.9, 0.9, 0.7)
    SubTitle.Font = CONFIG.FontMain
    SubTitle.TextSize = 18
    SubTitle.BackgroundTransparency = 1

    local BarBack = Instance.new("Frame", Center)
    BarBack.Size = UDim2.new(0.8, 0, 0, 6)
    BarBack.Position = UDim2.new(0.1, 0, 0.7, 0)
    BarBack.BackgroundColor3 = Color3.fromRGB(40, 40, 0)
    Instance.new("UICorner", BarBack).CornerRadius = UDim.new(1, 0)

    local BarFill = Instance.new("Frame", BarBack)
    BarFill.Size = UDim2.new(0, 0, 1, 0)
    BarFill.BackgroundColor3 = CONFIG.MainColor
    Instance.new("UICorner", BarFill).CornerRadius = UDim.new(1, 0)

    local PercentText = Instance.new("TextLabel", Center)
    PercentText.Size = UDim2.new(1, 0, 0, 30)
    PercentText.Position = UDim2.new(0, 0, 0.8, 0)
    PercentText.Text = "Đang tải... 0%"
    PercentText.TextColor3 = Color3.new(1, 1, 1)
    PercentText.Font = CONFIG.FontMain
    PercentText.TextSize = 14
    PercentText.BackgroundTransparency = 1

    for i = 0, 100, math.random(3, 8) do
        local percent = math.min(i, 100)
        TweenService:Create(BarFill, TweenInfo.new(0.1), {Size = UDim2.new(percent/100, 0, 1, 0)}):Play()
        PercentText.Text = "Đang tải dữ liệu... " .. percent .. "%"
        task.wait(math.random(1, 3)/25)
    end
    
    TweenService:Create(BarFill, TweenInfo.new(0.1), {Size = UDim2.new(1, 0, 1, 0)}):Play()
    PercentText.Text = "Tải thành công!!"
    task.wait(0.6)

    TweenService:Create(Blur, TweenInfo.new(0.5), {Size = 0}):Play()
    local closeTween = TweenService:Create(LoadFrame, TweenInfo.new(0.5), {BackgroundTransparency = 1})
    closeTween:Play()
    
    for _, v in pairs(Center:GetDescendants()) do
        if v:IsA("TextLabel") then
            TweenService:Create(v, TweenInfo.new(0.3), {TextTransparency = 1, BackgroundTransparency = 1}):Play()
        elseif v:IsA("Frame") then
            TweenService:Create(v, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
        end
    end
    
    closeTween.Completed:Wait()
    LoadFrame:Destroy()
    Blur:Destroy()
    callback()
end

--// MODULE: HIỆU ỨNG HẠT (Đổi thành hạt chuối bay lên)
local function CreateParticles(parentFrame)
    task.spawn(function()
        while parentFrame.Parent and parentFrame.Visible do
            local particle = Instance.new("Frame", parentFrame)
            particle.BackgroundColor3 = CONFIG.MainColor
            particle.Size = UDim2.new(0, math.random(2, 4), 0, math.random(2, 4))
            particle.Position = UDim2.new(math.random(), 0, 1, 0)
            particle.BackgroundTransparency = math.random(3, 7)/10
            particle.ZIndex = 1
            Instance.new("UICorner", particle).CornerRadius = UDim.new(1,0)
            
            local tween = TweenService:Create(particle, TweenInfo.new(math.random(3, 6), Enum.EasingStyle.Linear), {
                Position = UDim2.new(particle.Position.X.Scale + math.random(-10, 10)/100, 0, -0.1, 0),
                BackgroundTransparency = 1
            })
            tween:Play()
            tween.Completed:Connect(function() particle:Destroy() end)
            task.wait(math.random(1, 3)/10)
        end
    end)
end

--// MODULE: MENU CHÍNH (PHONG CÁCH BANANA - KHÔNG DISCORD)
local function CreateMainMenu()
    local isEffectsOn = true

    -- 1. NÚT DI ĐỘNG NỔI (Biểu tượng chuối)
    local MiniBtn = Instance.new("ImageButton", ScreenGui)
    MiniBtn.Size = UDim2.new(0, 60, 0, 60)
    MiniBtn.Position = UDim2.new(0.1, 0, 0.2, 0)
    MiniBtn.Image = CONFIG.LogoID
    MiniBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 100)
    MiniBtn.Visible = false
    MiniBtn.ZIndex = 10
    Instance.new("UICorner", MiniBtn).CornerRadius = UDim.new(1, 0)
    local MiniStroke = Instance.new("UIStroke", MiniBtn)
    MiniStroke.Color = CONFIG.MainColor
    MiniStroke.Thickness = 2
    MakeDraggable(MiniBtn)

    -- 2. KHUNG CHÍNH
    local Frame = Instance.new("Frame", ScreenGui)
    Frame.Size = UDim2.new(0, 0, 0, 0)
    Frame.Position = UDim2.new(0.5, 0, 0.5, 0)
    Frame.AnchorPoint = Vector2.new(0.5, 0.5)
    Frame.BackgroundColor3 = CONFIG.SecondaryColor
    Frame.ClipsDescendants = true
    Frame.ZIndex = 5
    Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 12)
    MakeDraggable(Frame)

    local FrameStroke = Instance.new("UIStroke", Frame)
    FrameStroke.Color = CONFIG.MainColor
    FrameStroke.Thickness = 2

    local BG = Instance.new("ImageLabel", Frame)
    BG.Size = UDim2.new(1, 0, 1, 0)
    BG.BackgroundTransparency = 1
    BG.BorderSizePixel = 0
    BG.Image = CONFIG.BackgroundID
    BG.ScaleType = Enum.ScaleType.Crop
    BG.ZIndex = 0
    
    local Overlay = Instance.new("Frame", Frame)
    Overlay.Size = UDim2.new(1, 0, 1, 0)
    Overlay.BackgroundColor3 = Color3.fromRGB(40, 35, 0) -- Lớp phủ vàng đậm
    Overlay.BackgroundTransparency = 0.4
    Overlay.ZIndex = 1

    -- THANH TRÊN CÙNG
    local TopBar = Instance.new("Frame", Frame)
    TopBar.Size = UDim2.new(1, 0, 0, 40)
    TopBar.BackgroundTransparency = 1
    TopBar.ZIndex = 2

    local Title = Instance.new("TextLabel", TopBar)
    Title.Size = UDim2.new(0.6, 0, 1, 0)
    Title.Position = UDim2.new(0, 15, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = "🍌 " .. CONFIG.HubName
    Title.TextColor3 = Color3.new(1,1,1)
    Title.Font = CONFIG.FontBold
    Title.TextSize = 18
    Title.TextXAlignment = Enum.TextXAlignment.Left
    
    local TitleGradient = Instance.new("UIGradient", Title)
    TitleGradient.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, CONFIG.MainColor), ColorSequenceKeypoint.new(1, CONFIG.GradientColor)}

    -- NÚT ĐIỀU KHIỂN
    local function CreateTopBtn(text, posX, color)
        local btn = Instance.new("TextButton", TopBar)
        btn.Size = UDim2.new(0, 30, 0, 30)
        btn.Position = UDim2.new(1, posX, 0.5, 0)
        btn.AnchorPoint = Vector2.new(0, 0.5)
        btn.BackgroundColor3 = color
        btn.Text = text
        btn.TextColor3 = Color3.new(1,1,1)
        btn.Font = CONFIG.FontBold
        btn.TextSize = 14
        btn.ZIndex = 3
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
        return btn
    end

    local CloseBtn = CreateTopBtn("❌", -40, Color3.fromRGB(200, 50, 50))
    local HideBtn = CreateTopBtn("➖", -75, Color3.fromRGB(80, 80, 10))
    local FXBtn = CreateTopBtn("🍌", -110, CONFIG.MainColor)

    -- LOGO & THỐNG KÊ
    local Logo = Instance.new("ImageLabel", Frame)
    Logo.Size = UDim2.new(0, 80, 0, 80)
    Logo.Position = UDim2.new(0, 20, 0, 50)
    Logo.Image = CONFIG.LogoID
    Logo.ZIndex = 2
    Instance.new("UICorner", Logo).CornerRadius = UDim.new(1, 0)
    local LogoStroke = Instance.new("UIStroke", Logo)
    LogoStroke.Color = CONFIG.MainColor
    LogoStroke.Thickness = 2

    local StatsContainer = Instance.new("Frame", Frame)
    StatsContainer.Size = UDim2.new(0, 150, 0, 60)
    StatsContainer.Position = UDim2.new(0, 115, 0, 60)
    StatsContainer.BackgroundTransparency = 1
    StatsContainer.ZIndex = 2

    local TotalMem = Instance.new("TextLabel", StatsContainer)
    TotalMem.Size = UDim2.new(1, 0, 0.5, 0)
    TotalMem.BackgroundTransparency = 1
    TotalMem.Text = "👥 Thành viên: <font color='rgb(255, 225, 0)'><b>" .. CONFIG.TotalMembers .. "</b></font>"
    TotalMem.RichText = true
    TotalMem.TextColor3 = Color3.fromRGB(230, 230, 210)
    TotalMem.Font = CONFIG.FontBold
    TotalMem.TextSize = 15
    TotalMem.TextXAlignment = Enum.TextXAlignment.Left

    local OnlineMem = Instance.new("TextLabel", StatsContainer)
    OnlineMem.Size = UDim2.new(1, 0, 0.5, 0)
    OnlineMem.Position = UDim2.new(0, 0, 0.5, 0)
    OnlineMem.BackgroundTransparency = 1
    OnlineMem.Text = "🟢 Trực tuyến: <font color='rgb(255, 240, 100)'><b>" .. CONFIG.OnlineMembers .. "</b></font>"
    OnlineMem.RichText = true
    OnlineMem.TextColor3 = Color3.fromRGB(230, 230, 210)
    OnlineMem.Font = CONFIG.FontBold
    OnlineMem.TextSize = 15
    OnlineMem.TextXAlignment = Enum.TextXAlignment.Left

    -- MÔ TẢ
    local Desc = Instance.new("TextLabel", Frame)
    Desc.Size = UDim2.new(1, -40, 0, 70)
    Desc.Position = UDim2.new(0, 20, 0, 140)
    Desc.BackgroundTransparency = 1
    Desc.Text = CONFIG.Description
    Desc.TextColor3 = Color3.fromRGB(220, 220, 180)
    Desc.Font = CONFIG.FontMain
    Desc.TextSize = 14
    Desc.TextWrapped = true
    Desc.TextXAlignment = Enum.TextXAlignment.Left
    Desc.TextYAlignment = Enum.TextYAlignment.Top
    Desc.ZIndex = 2

    -- NÚT CHỨC NĂNG (Thay thế nút Discord bằng nút Script)
    local ScriptBtn = Instance.new("TextButton", Frame)
    ScriptBtn.Size = UDim2.new(0.4, 0, 0, 40)
    ScriptBtn.Position = UDim2.new(0.1, 0, 1, -55)
    ScriptBtn.BackgroundColor3 = CONFIG.MainColor
    ScriptBtn.Text = "🍌 LOAD SCRIPT"
    ScriptBtn.TextColor3 = Color3.new(0, 0, 0) -- Chữ đen nổi trên nền vàng
    ScriptBtn.Font = CONFIG.FontBold
    ScriptBtn.TextSize = 16
    ScriptBtn.ZIndex = 3
    Instance.new("UICorner", ScriptBtn).CornerRadius = UDim.new(0, 8)

    local ExecuteBtn = Instance.new("TextButton", Frame)
    ExecuteBtn.Size = UDim2.new(0.4, 0, 0, 40)
    ExecuteBtn.Position = UDim2.new(0.5, 0, 1, -55)
    ExecuteBtn.BackgroundColor3 = CONFIG.GradientColor
    ExecuteBtn.Text = "⚡ EXECUTE"
    ExecuteBtn.TextColor3 = Color3.new(0, 0, 0) -- Chữ đen nổi trên nền vàng
    ExecuteBtn.Font = CONFIG.FontBold
    ExecuteBtn.TextSize = 16
    ExecuteBtn.ZIndex = 3
    Instance.new("UICorner", ExecuteBtn).CornerRadius = UDim.new(0, 8)

    -- LOGIC HIỆU ỨNG NÚT BẤM
    local function AddButtonEffects(btn)
        btn.MouseEnter:Connect(function()
            pcall(function() HoverSound:Play() end)
            TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0.2}):Play()
        end)
        btn.MouseLeave:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
        end)
    end
    AddButtonEffects(ScriptBtn)
    AddButtonEffects(ExecuteBtn)
    AddButtonEffects(CloseBtn)
    AddButtonEffects(HideBtn)
    AddButtonEffects(FXBtn)

    -- Hiệu ứng Ripple & Shake cho nút Script
    local function AddRippleEffect(btn)
        btn.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                if input.Target == btn then
                    pcall(function() ClickSound:Play() end)
                    local ripple = Instance.new("ImageLabel", btn)
                    ripple.BackgroundTransparency = 1
                    ripple.Image = "rbxassetid://2708891598"
                    ripple.ImageTransparency = 0.6
                    ripple.ImageColor3 = Color3.new(0,0,0) -- Ripple màu đen để tương phản
                    ripple.ZIndex = 4
                    
                    local x = input.Position.X - btn.AbsolutePosition.X
                    local y = input.Position.Y - btn.AbsolutePosition.Y
                    local size = math.max(btn.AbsoluteSize.X, btn.AbsoluteSize.Y) * 1.5
                    
                    ripple.Position = UDim2.new(0, x, 0, y)
                    ripple.Size = UDim2.new(0, 0, 0, 0)
                    
                    TweenService:Create(ripple, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                        Size = UDim2.new(0, size, 0, size),
                        Position = UDim2.new(0, x - size/2, 0, y - size/2),
                        ImageTransparency = 1
                    }):Play()
                    task.delay(0.5, function() ripple:Destroy() end)
                end
            end
        end)
    end
    AddRippleEffect(ScriptBtn)
    AddRippleEffect(ExecuteBtn)

    -- Chức năng nút Script (Load Script)
    ScriptBtn.MouseButton1Click:Connect(function()
        local origPos = ScriptBtn.Position
        for i = 1, 6 do
            ScriptBtn.Position = origPos + UDim2.new(0, math.random(-3, 3), 0, math.random(-3, 3))
            task.wait(0.02)
        end
        ScriptBtn.Position = origPos
        
        ScriptBtn.Text = " ✔ ĐÃ LOAD!"
        ScriptBtn.BackgroundColor3 = Color3.fromRGB(100, 200, 50)
        SendNotification("Thành công", "Script đã được load thành công!", 3)
        task.wait(2)
        ScriptBtn.Text = "🍌 LOAD SCRIPT"
        ScriptBtn.BackgroundColor3 = CONFIG.MainColor
    end)

    -- Chức năng nút Execute (Thực thi Script)
    ExecuteBtn.MouseButton1Click:Connect(function()
        local origPos = ExecuteBtn.Position
        for i = 1, 6 do
            ExecuteBtn.Position = origPos + UDim2.new(0, math.random(-3, 3), 0, math.random(-3, 3))
            task.wait(0.02)
        end
        ExecuteBtn.Position = origPos
        
        ExecuteBtn.Text = " ✔ ĐÃ EXECUTE!"
        ExecuteBtn.BackgroundColor3 = Color3.fromRGB(100, 200, 50)
        SendNotification("Thành công", "Script đã được thực thi thành công!", 3)
        task.wait(2)
        ExecuteBtn.Text = "⚡ EXECUTE"
        ExecuteBtn.BackgroundColor3 = CONFIG.GradientColor
    end)

    -- Nút Tắt/Bật Nhạc
    FXBtn.MouseButton1Click:Connect(function()
        pcall(function() ClickSound:Play() end)
        isEffectsOn = not isEffectsOn
        if isEffectsOn then
            FXBtn.BackgroundColor3 = CONFIG.MainColor
            pcall(function() BgMusic:Resume() end)
            FrameStroke.Color = CONFIG.MainColor
            FrameStroke.Thickness = 2
        else
            FXBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
            pcall(function() BgMusic:Pause() end)
            FrameStroke.Color = Color3.fromRGB(100, 100, 100)
        end
    end)

    -- Mở / Đóng giao diện
    local isVisible = true
    local function ToggleUI()
        isVisible = not isVisible
        if isVisible then
            pcall(function() ClickSound:Play() end)
            MiniBtn.Visible = false
            Frame.Visible = true
            TweenService:Create(Frame, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 480, 0, 280)}):Play()
            CreateParticles(Frame)
        else
            pcall(function() ClickSound:Play() end)
            local tween = TweenService:Create(Frame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)})
            tween:Play()
            tween.Completed:Wait()
            Frame.Visible = false
            MiniBtn.Visible = true
            SendNotification("Đã ẩn Menu", "Bấm vào Logo hoặc nhấn " .. tostring(CONFIG.ToggleKey) .. " để mở lại.", 3)
        end
    end

    HideBtn.MouseButton1Click:Connect(ToggleUI)
    MiniBtn.MouseButton1Click:Connect(ToggleUI)
    UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
        if not gameProcessedEvent and input.KeyCode == CONFIG.ToggleKey then ToggleUI() end
    end)

    -- Đóng hoàn toàn (Xóa Script)
    CloseBtn.MouseButton1Click:Connect(function()
        pcall(function() ClickSound:Play() end)
        pcall(function() BgMusic:Stop() end)
        local tween = TweenService:Create(Frame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)})
        tween:Play()
        tween.Completed:Wait()
        ScreenGui:Destroy()
    end)

    -- KHỞI CHẠY GIAO DIỆN
    pcall(function() BgMusic:Play() end)
    TweenService:Create(Frame, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 480, 0, 280)}):Play()
    CreateParticles(Frame)
    
    -- Vòng lặp điện giật
    task.spawn(function()
        while Frame.Parent do
            if isEffectsOn then
                TweenService:Create(FrameStroke, TweenInfo.new(0.1), {
                    Color = (math.random(1,2) == 1 and CONFIG.MainColor or CONFIG.GradientColor),
                    Thickness = math.random(1, 3)
                }):Play()
            end
            task.wait(0.15)
        end
    end)
end

--// ================== [ THỰC THI KỊCH BẢN ] =================== //--
StartLoading(function()
    SendNotification("Thành công", "🍌 " .. CONFIG.HubName .. " đã tải thành công! 🍌", 4)
    CreateMainMenu()
end)
