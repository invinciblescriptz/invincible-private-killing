-- Create a simple UI with ScreenGui
local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "InvincibleUI"

-- Create a main frame
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 600, 0, 400)
frame.Position = UDim2.new(0.5, -300, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(0, 10, 0)
frame.BorderSizePixel = 0

-- Title
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(0, 20, 0)
title.TextColor3 = Color3.white
title.Text = "Invincible Private Killing"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

-- Auto Punch Toggle
local autoPunchToggle = Instance.new("TextButton", frame)
autoPunchToggle.Size = UDim2.new(0, 200, 0, 40)
autoPunchToggle.Position = UDim2.new(0.5, -100, 0, 60)
autoPunchToggle.Text = "Auto Punch (Fast): OFF"
autoPunchToggle.BackgroundColor3 = Color3.fromRGB(0,50,0)
autoPunchToggle.TextColor3 = Color3.white

-- Auto Kill Toggle
local autoKillToggle = Instance.new("TextButton", frame)
autoKillToggle.Size = UDim2.new(0, 200, 0, 40)
autoKillToggle.Position = UDim2.new(0.5, -100, 0, 110)
autoKillToggle.Text = "Auto Kill: OFF"
autoKillToggle.BackgroundColor3 = Color3.fromRGB(0,50,0)
autoKillToggle.TextColor3 = Color3.white

-- Follow Player Button
local followButton = Instance.new("TextButton", frame)
followButton.Size = UDim2.new(0, 200, 0, 40)
followButton.Position = UDim2.new(0.5, -100, 0, 160)
followButton.Text = "Follow Player"
followButton.BackgroundColor3 = Color3.fromRGB(0,50,0)
followButton.TextColor3 = Color3.white

-- Stop Follow Button
local stopFollowButton = Instance.new("TextButton", frame)
stopFollowButton.Size = UDim2.new(0, 200, 0, 40)
stopFollowButton.Position = UDim2.new(0.5, -100, 0, 210)
stopFollowButton.Text = "Stop Following"
stopFollowButton.BackgroundColor3 = Color3.fromRGB(0,50,0)
stopFollowButton.TextColor3 = Color3.white

-- Variables for toggles
local autoPunch = false
local autoKill = false
local spying = false
local targetPlayerName = nil

local Players = game:GetService("Players")
local LP = Players.LocalPlayer

-- Helper to get HumanoidRootPart
local function getHRP(character)
    if character then
        return character:FindFirstChild("HumanoidRootPart")
    end
    return nil
end

-- Toggle Auto Punch
autoPunchToggle.MouseButton1Click:Connect(function()
    autoPunch = not autoPunch
    autoPunchToggle.Text = "Auto Punch (Fast): " .. (autoPunch and "ON" or "OFF")
end)

-- Toggle Auto Kill
autoKillToggle.MouseButton1Click:Connect(function()
    autoKill = not autoKill
    autoKillToggle.Text = "Auto Kill: " .. (autoKill and "ON" or "OFF")
end)

-- Follow Button
followButton.MouseButton1Click:Connect(function()
    -- Prompt user to select a player name
    local selectionGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
    local box = Instance.new("TextBox", selectionGui)
    box.Size = UDim2.new(0, 300, 0, 50)
    box.Position = UDim2.new(0.5, -150, 0.5, -25)
    box.PlaceholderText = "Enter Player Name"
    box.Text = ""
    box.FocusLost:Connect(function()
        targetPlayerName = box.Text
        selectionGui:Destroy()
        spying = true
    end)
end)

-- Stop Following
stopFollowButton.MouseButton1Click:Connect(function()
    spying = false
    workspace.CurrentCamera.CameraSubject = LP.Character and LP.Character:FindFirstChild("Humanoid") or LP
end)

-- Main loops
spawn(function()
    while true do
        wait(0.05)
        -- Auto Punch
        if autoPunch and getgenv().NexusRunning then
            pcall(function()
                if muscleEvent then
                    muscleEvent:FireServer("punch", "rightHand")
                    muscleEvent:FireServer("punch", "leftHand")
                end
                local char = LP.Character
                if char then
                    local punch = char:FindFirstChild("Punch") or LP.Backpack:FindFirstChild("Punch")
                    if punch and punch.Parent ~= char then
                        char.Humanoid:EquipTool(punch)
                    end
                    if punch and punch.Parent == char then
                        local atk = punch:FindFirstChild("attackTime")
                        if atk and atk:IsA("NumberValue") then atk.Value = 0.01 end
                        punch:Activate()
                    end
                end
            end)
        end

        -- Auto Kill
        if autoKill then
            local myChar = LP.Character
            if myChar then
                local rightHand = myChar:FindFirstChild("RightHand")
                local leftHand = myChar:FindFirstChild("LeftHand")
                local punch = LP.Backpack:FindFirstChild("Punch")
                if punch and not myChar:FindFirstChild("Punch") then
                    punch.Parent = myChar
                end
                if rightHand and leftHand then
                    for _, target in pairs(Players:GetPlayers()) do
                        if target ~= LP and not playerWhitelist[target.Name] then
                            local targetChar = target.Character
                            if targetChar then
                                local rootPart = targetChar:FindFirstChild("HumanoidRootPart")
                                local humanoid = targetChar:FindFirstChild("Humanoid")
                                if rootPart and humanoid and humanoid.Health > 0 then
                                    pcall(function()
                                        firetouchinterest(rightHand, rootPart, 1)
                                        firetouchinterest(leftHand, rootPart, 1)
                                        firetouchinterest(rightHand, rootPart, 0)
                                        firetouchinterest(leftHand, rootPart, 0)
                                    end)
                                end
                            end
                        end
                    end
                end
            end
        end

        -- Follow Player if active
        if spying and targetPlayerName then
            local targetPlayer = Players:FindFirstChild(targetPlayerName)
            if targetPlayer and targetPlayer.Character then
                local targetHRP = getHRP(targetPlayer.Character)
                local myChar = LP.Character
                local myHRP = getHRP(myChar)
                if targetHRP and myHRP then
                    local newCFrame = targetHRP.CFrame * CFrame.new(0, 0, -3)
                    myHRP.CFrame = newCFrame
                end
            end
        end
    end
end)

print("UI loaded. Use the buttons and toggles.")
