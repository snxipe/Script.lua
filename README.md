-- Roblox Mobile Toggle Script: Flight, Speed, and Jump Boost
local Players = game:Service("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:Service("RunService")
local UserInputService = game:Service("UserInputService")

local flying = false
local speedValue = 100        -- Change this to go faster/slower
local jumpValue = 150         -- Change this to jump higher
local flySpeed = 50          -- Flight speed

-- Create Mobile Screen Button
local ScreenGui = Instance.new("ScreenGui")
local ToggleButton = Instance.new("TextButton")

ScreenGui.Parent = game:CoreGui
ScreenGui.Name = "ModToggleGui"

ToggleButton.Parent = ScreenGui
ToggleButton.Size = UDim2.new(0, 120, 0, 50)
ToggleButton.Position = UDim2.new(0.1, 0, 0.4, 0) -- Adjust position on screen
ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 18
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.Text = "MODS: OFF"
ToggleButton.BorderSizePixel = 2

-- Flight Mechanics
local bv, bg
local function startFlying(char)
    local root = char:WaitForChild("HumanoidRootPart")
    local hum = char:WaitForChild("Humanoid")
    
    bg = Instance.new("BodyGyro")
    bg.maxTorque = Vector3.new(4e4, 4e4, 4e4)
    bg.d = 10
    bg.p = 10000
    bg.cframe = root.CFrame
    bg.Parent = root
    
    bv = Instance.new("BodyVelocity")
    bv.maxForce = Vector3.new(4e4, 4e4, 4e4)
    bv.velocity = Vector3.new(0, 0.1, 0)
    bv.Parent = root
    
    hum.PlatformStand = true
end

local function stopFlying(char)
    if bv then bv:Destroy() end
    if bg then bg:Destroy() end
    local hum = char:FindFirstChild("Humanoid")
    if hum then hum.PlatformStand = false end
end

-- Toggle Logic
ToggleButton.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    
    flying = not flying
    
    if flying then
        -- Turn EVERYTHING On
        ToggleButton.Text = "MODS: ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
        
        hum.WalkSpeed = speedValue
        hum.JumpPower = jumpValue
        hum.UseJumpPower = true
        startFlying(char)
    else
        -- Reset to normal Roblox settings
        ToggleButton.Text = "MODS: OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        
        hum.WalkSpeed = 16
        hum.JumpPower = 50
        stopFlying(char)
    end
end)

-- Continuous Flight Direction Update (Mobile Touch/Camera Friendly)
RunService.RenderStepped:Connect(function()
    if flying and LocalPlayer.Character and bv and bg then
        local root = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local camera = workspace.CurrentCamera
        if root and camera then
            bg.cframe = camera.CFrame
            
            -- Direction based on movement vector
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.MoveDirection.Magnitude > 0 then
                bv.velocity = hum.MoveDirection * flySpeed
            else
                bv.velocity = Vector3.new(0, 0.1, 0) -- Hover in place
            end
        end
    end
end)
