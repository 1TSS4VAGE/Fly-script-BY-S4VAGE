-- Roblox Fly Script for Mobile (Touch + Button)
-- Paste this code into any executor or save as raw .lua and load with:
-- loadstring(game:HttpGet("YOUR_RAW_URL_HERE"))()

local plr = game:GetService("Players").LocalPlayer
local uis = game:GetService("UserInputService")
local rs = game:GetService("RunService")
local cam = workspace.CurrentCamera

local flying = false
local speed = 60
local char, root, hum

-- GUI setup (button)
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = plr.PlayerGui
local flyBtn = Instance.new("TextButton")
flyBtn.Size = UDim2.new(0, 80, 0, 80)
flyBtn.Position = UDim2.new(0, 20, 0, 300)
flyBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
flyBtn.TextColor3 = Color3.new(1,1,1)
flyBtn.Text = "FLY"
flyBtn.Font = Enum.Font.GothamBold
flyBtn.TextSize = 18
flyBtn.Parent = screenGui
local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(1, 0)
btnCorner.Parent = flyBtn

-- Vertical control buttons (up/down)
local upBtn = Instance.new("TextButton")
upBtn.Size = UDim2.new(0, 60, 0, 60)
upBtn.Position = UDim2.new(0, 120, 0, 300)
upBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
upBtn.Text = "▲"
upBtn.Font = Enum.Font.GothamBold
upBtn.TextSize = 24
upBtn.Parent = screenGui
local upCorner = Instance.new("UICorner")
upCorner.CornerRadius = UDim.new(1,0)
upCorner.Parent = upBtn

local downBtn = Instance.new("TextButton")
downBtn.Size = UDim2.new(0, 60, 0, 60)
downBtn.Position = UDim2.new(0, 120, 0, 380)
downBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
downBtn.Text = "▼"
downBtn.Font = Enum.Font.GothamBold
downBtn.TextSize = 24
downBtn.Parent = screenGui
local downCorner = Instance.new("UICorner")
downCorner.CornerRadius = UDim.new(1,0)
downCorner.Parent = downBtn

local function getChar()
    char = plr.Character
    if not char then return false end
    root = char:FindFirstChild("HumanoidRootPart")
    hum = char:FindFirstChild("Humanoid")
    return root and hum
end

getChar() or plr.CharacterAdded:Wait() and getChar()

local conn
local function startFly()
    if conn then return end
    if not getChar() then return end
    flying = true
    flyBtn.Text = "LAND"
    flyBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    
    conn = rs.Heartbeat:Connect(function(dt)
        if not flying or not getChar() then
            return
        end
        
        -- Horizontal movement (from mobile joystick or WASD)
        local moveVec = Vector3.new(0,0,0)
        local fwd = cam.CFrame.LookVector
        local rgt = cam.CFrame.RightVector
        
        -- Use Humanoid.MoveDirection for mobile joystick
        local moveDir = hum.MoveDirection
        if moveDir.Magnitude > 0 then
            moveVec = moveDir.Unit
        else
            -- PC fallback (WASD)
            if uis:IsKeyDown(Enum.KeyCode.W) then moveVec = moveVec + fwd end
            if uis:IsKeyDown(Enum.KeyCode.S) then moveVec = moveVec - fwd end
            if uis:IsKeyDown(Enum.KeyCode.A) then moveVec = moveVec - rgt end
            if uis:IsKeyDown(Enum.KeyCode.D) then moveVec = moveVec + rgt end
        end
        
        -- Vertical: buttons pressed state
        local upPressed = false
        local downPressed = false
        -- Check if up/down buttons are being held (using mouse button down state)
        -- Since we can't easily track hold on mobile, we use simple toggle? Better: use touch begin/end events.
        -- For simplicity, we'll use global variables set by button events.
        if _G.__flyUp then upPressed = true end
        if _G.__flyDown then downPressed = true end
        -- Also keyboard Space/Shift
        if uis:IsKeyDown(Enum.KeyCode.Space) then upPressed = true end
        if uis:IsKeyDown(Enum.KeyCode.LeftShift) then downPressed = true end
        
        if upPressed then moveVec = moveVec + Vector3.new(0,1,0) end
        if downPressed then moveVec = moveVec - Vector3.new(0,1,0) end
        
        local dir = Vector3.new(0,0,0)
        if moveVec.Magnitude > 0 then
            dir = moveVec.Unit * speed
        end
        
        root.CFrame = root.CFrame + dir * dt
    end)
end

local function stopFly()
    flying = false
    flyBtn.Text = "FLY"
    flyBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    if conn then conn:Disconnect() conn = nil end
end

-- Button events (use Activated for mobile)
flyBtn.Activated:Connect(function()
    if flying then stopFly() else startFly() end
end)

-- Up/Down button hold events (using TouchBegin/TouchEnd or Mouse events)
upBtn.MouseButton1Down:Connect(function() _G.__flyUp = true end)
upBtn.MouseButton1Up:Connect(function() _G.__flyUp = false end)
upBtn.MouseLeave:Connect(function() _G.__flyUp = false end)

downBtn.MouseButton1Down:Connect(function() _G.__flyDown = true end)
downBtn.MouseButton1Up:Connect(function() _G.__flyDown = false end)
downBtn.MouseLeave:Connect(function() _G.__flyDown = false end)

-- For mobile touch hold, we need TouchBegin/TouchEnd
upBtn.TouchBegin:Connect(function() _G.__flyUp = true end)
upBtn.TouchEnd:Connect(function() _G.__flyUp = false end)
downBtn.TouchBegin:Connect(function() _G.__flyDown = true end)
downBtn.TouchEnd:Connect(function() _G.__flyDown = false end)

-- Keyboard toggle (F key) for PC
uis.InputBegan:Connect(function(inp, gp)
    if gp then return end
    if inp.KeyCode == Enum.KeyCode.F then
        if flying then stopFly() else startFly() end
    end
end)

-- Reset on character death
plr.CharacterAdded:Connect(function()
    getChar()
    if flying then stopFly() end
end)

print("Mobile Fly script loaded. Press FLY button to toggle. Use ▲/▼ for vertical.")
