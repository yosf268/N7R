-- مكتبة Roblox
local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- إنشاء واجهة رئيسية
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 220, 0, 420)
frame.Position = UDim2.new(0.3, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.new(1,1,1)
frame.Active = true
frame.Draggable = true

local uiCorner = Instance.new("UICorner", frame)
uiCorner.CornerRadius = UDim.new(0,8)

-- أزرار عامة
local function createButton(text, y)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 200, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(200,200,200)
    local c = Instance.new("UICorner", btn)
    c.CornerRadius = UDim.new(0,6)
    return btn
end

-- سرعة وقفز
local walkSpeed = 16
local jumpPower = 50

local speedLabel = Instance.new("TextLabel", frame)
speedLabel.Size = UDim2.new(0,200,0,20)
speedLabel.Position = UDim2.new(0,10,0,10)
speedLabel.Text = "السرعة: "..walkSpeed
speedLabel.BackgroundTransparency = 1

local addSpeed = createButton("+", 40)
local subSpeed = createButton("-", 75)

addSpeed.MouseButton1Click:Connect(function()
    walkSpeed = walkSpeed + 1
    player.Character.Humanoid.WalkSpeed = walkSpeed
    speedLabel.Text = "السرعة: "..walkSpeed
end)

subSpeed.MouseButton1Click:Connect(function()
    walkSpeed = walkSpeed - 1
    player.Character.Humanoid.WalkSpeed = walkSpeed
    speedLabel.Text = "السرعة: "..walkSpeed
end)

local jumpLabel = Instance.new("TextLabel", frame)
jumpLabel.Size = UDim2.new(0,200,0,20)
jumpLabel.Position = UDim2.new(0,10,0,110)
jumpLabel.Text = "القفز: "..jumpPower
jumpLabel.BackgroundTransparency = 1

local addJump = createButton("+", 140)
local subJump = createButton("-", 175)

addJump.MouseButton1Click:Connect(function()
    jumpPower = jumpPower + 1
    player.Character.Humanoid.JumpPower = jumpPower
    jumpLabel.Text = "القفز: "..jumpPower
end)

subJump.MouseButton1Click:Connect(function()
    jumpPower = jumpPower - 1
    player.Character.Humanoid.JumpPower = jumpPower
    jumpLabel.Text = "القفز: "..jumpPower
end)

-- زر ريست
local resetBtn = createButton("إعادة ضبط", 210)
resetBtn.MouseButton1Click:Connect(function()
    walkSpeed = 16
    jumpPower = 50
    player.Character.Humanoid.WalkSpeed = walkSpeed
    player.Character.Humanoid.JumpPower = jumpPower
    speedLabel.Text = "السرعة: "..walkSpeed
    jumpLabel.Text = "القفز: "..jumpPower
end)

-- الطيران (يتأثر بالاتجاه)
local flyBtn = createButton("تشغيل الطيران", 245)
local flying = false
local flyConn

flyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    if flying then
        flyBtn.Text = "إيقاف الطيران"
        player.Character.Humanoid.PlatformStand = true

        flyConn = RunService.RenderStepped:Connect(function()
            local root = player.Character:WaitForChild("HumanoidRootPart")
            local cam = workspace.CurrentCamera
            local moveDir = player.Character.Humanoid.MoveDirection

            if moveDir.Magnitude > 0 then
                local direction = cam.CFrame.LookVector * 60
                root.Velocity = Vector3.new(direction.X, direction.Y, direction.Z)
            else
                root.Velocity = Vector3.new(0,0,0)
            end
        end)
    else
        flyBtn.Text = "تشغيل الطيران"
        player.Character.Humanoid.PlatformStand = false
        if flyConn then flyConn:Disconnect() end
    end
end)

-- اختراق الجدران (دائم)
local noclipBtn = createButton("اختراق الجدران", 280)
noclipBtn.MouseButton1Click:Connect(function()
    for _, part in pairs(player.Character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end)

-- ايم بوت
local aimbotBtn = createButton("Aimbot: Off", 315)
local aiming = false
local aimConn

aimbotBtn.MouseButton1Click:Connect(function()
    aiming = not aiming
    if aiming then
        aimbotBtn.Text = "Aimbot: On"
        aimConn = RunService.RenderStepped:Connect(function()
            local closestPlayer, shortestDist = nil, math.huge
            local cam = workspace.CurrentCamera
            for _, plr in pairs(game.Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local rootPart = plr.Character.HumanoidRootPart
                    local pos, visible = cam:WorldToViewportPoint(rootPart.Position)
                    if visible then
                        local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(cam.ViewportSize.X/2, cam.ViewportSize.Y/2)).Magnitude
                        if dist < shortestDist then
                            shortestDist = dist
                            closestPlayer = rootPart
                        end
                    end
                end
            end
            if closestPlayer then
                cam.CFrame = CFrame.new(cam.CFrame.Position, closestPlayer.Position)
            end
        end)
    else
        aimbotBtn.Text = "Aimbot: Off"
        if aimConn then aimConn:Disconnect() end
    end
end)

-- زر نسخ رابط ديسكورد + إشعار
local copyBtn = createButton("نسخ رابط ديسكورد", 350)
copyBtn.MouseButton1Click:Connect(function()
    setclipboard("https://discord.gg/rtxtop1")

    local notify = Instance.new("TextLabel", screenGui)
    notify.Size = UDim2.new(0,200,0,40)
    notify.Position = UDim2.new(0.5,-100,0.1,0)
    notify.Text = "✅ تم نسخ رابط الديسكورد"
    notify.BackgroundColor3 = Color3.fromRGB(50,200,50)
    notify.TextColor3 = Color3.new(1,1,1)
    notify.Font = Enum.Font.SourceSansBold
    notify.TextSize = 18
    local c = Instance.new("UICorner", notify)
    c.CornerRadius = UDim.new(0,8)

    task.delay(2, function()
        notify:Destroy()
    end)
end)

-- زر RTXTOP1 (إظهار/إخفاء القائمة)
local toggleBtn = Instance.new("TextButton", screenGui)
toggleBtn.Size = UDim2.new(0, 80, 0, 30)
toggleBtn.Position = UDim2.new(0.9, 0, 0.5, 0)
toggleBtn.Text = "RTXTOP1"
toggleBtn.BackgroundColor3 = Color3.new(1,1,1)
toggleBtn.Active = true
toggleBtn.Draggable = true

local togCorner = Instance.new("UICorner", toggleBtn)
togCorner.CornerRadius = UDim.new(0,6)

toggleBtn.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)