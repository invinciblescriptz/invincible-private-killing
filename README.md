-- Load UI Library
local ScriptLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/p4020854-hub/Lb/refs/heads/main/X", true))()

local Window = ScriptLib:AddWindow("Queen KKrxzy || Private || Hello " .. game.Players.LocalPlayer.DisplayName, {
    min_size = Vector2.new(660, 700),
    can_resize = true,
    main_color = Color3.fromRGB(255, 192, 203)
})

-- ============================
-- ====== Kill Tab ===========
-- ============================
local KillTab = Window:AddTab("Kill")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Variables
local playerWhitelist = {}
local targetPlayerNames = {}
local autoGoodKarma = false
local autoBadKarma = false
local autoKill = false
local killTarget = false
local spying = false
local autoEquipPunch = false
local autoPunchNoAnim = false

-- --- Auto Karma (Good) ---
KillTab:AddSwitch("Auto Good Karma", function(active)
    autoGoodKarma = active
    if active then
        task.spawn(function()
            while autoGoodKarma do
                local chr = LocalPlayer.Character
                local rightHand = chr and chr:FindFirstChild("RightHand")
                local leftHand = chr and chr:FindFirstChild("LeftHand")
                if chr and rightHand and leftHand then
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target ~= LocalPlayer then
                            local evilKarma = target:FindFirstChild("evilKarma")
                            local goodKarma = target:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if evilKarma.Value > goodKarma.Value then
                                    local root = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                                    if root then
                                        firetouchinterest(rightHand, root, 1)
                                        firetouchinterest(leftHand, root, 1)
                                        firetouchinterest(rightHand, root, 0)
                                        firetouchinterest(leftHand, root, 0)
                                    end
                                end
                            end
                        end
                    end
                end
                task.wait(0.01)
            end
        end)
    end
end)

-- --- Auto Karma (Bad) ---
KillTab:AddSwitch("Auto Bad Karma", function(active)
    autoBadKarma = active
    if active then
        task.spawn(function()
            while autoBadKarma do
                local chr = LocalPlayer.Character
                local rightHand = chr and chr:FindFirstChild("RightHand")
                local leftHand = chr and chr:FindFirstChild("LeftHand")
                if chr and rightHand and leftHand then
                    for _, target in ipairs(Players:GetPlayers()) do
                        if target ~= LocalPlayer then
                            local evilKarma = target:FindFirstChild("evilKarma")
                            local goodKarma = target:FindFirstChild("goodKarma")
                            if evilKarma and goodKarma and evilKarma:IsA("IntValue") and goodKarma:IsA("IntValue") then
                                if goodKarma.Value > evilKarma.Value then
                                    local root = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
                                    if root then
                                        firetouchinterest(rightHand, root, 1)
                                        firetouchinterest(leftHand, root, 1)
                                        firetouchinterest(rightHand, root, 0)
                                        firetouchinterest(leftHand, root, 0)
                                    end
                                end
                            end
                        end
                    end
                end
                task.wait(0.01)
            end
        end)
    end
end)

-- --- Whitelist Friends ---
local friendWhitelist = false
KillTab:AddSwitch("Auto Whitelist Friends", function(state)
    friendWhitelist=state
    if state then
        for _,player in ipairs(Players:GetPlayers()) do
            if player~=LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name]=true
            end
        end
        Players.PlayerAdded:Connect(function(player)
            if friendWhitelist and player~=LocalPlayer and LocalPlayer:IsFriendsWith(player.UserId) then
                playerWhitelist[player.Name]=true
            end
        end)
    else
        for name in pairs(playerWhitelist) do
            local p=Players:FindFirstChild(name)
            if p and LocalPlayer:IsFriendsWith(p.UserId) then
                playerWhitelist[name]=nil
            end
        end
    end
end)

KillTab:AddTextBox("Whitelist", function(text)
    local p=Players:FindFirstChild(text)
    if p then
        playerWhitelist[p.Name]=true
    end
end)

KillTab:AddTextBox("UnWhitelist", function(text)
    local p=Players:FindFirstChild(text)
    if p then
        playerWhitelist[p.Name]=nil
    end
end)

-- --- Auto Kill ---
local autoKill = false
KillTab:AddSwitch("Auto Kill", function(state)
    autoKill=state
    if state then
        task.spawn(function()
            while autoKill do
                local chr=LocalPlayer.Character
                local rightHand=chr and chr:FindFirstChild("RightHand")
                local leftHand=chr and chr:FindFirstChild("LeftHand")
                local punch=LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not chr:FindFirstChild("Punch") then
                    punch.Parent=chr
                end
                for _,target in ipairs(Players:GetPlayers()) do
                    if target~=LocalPlayer and not playerWhitelist[target.Name] then
                        local tchar=target.Character
                        local root=tchar and tchar:FindFirstChild("HumanoidRootPart")
                        local hum=tchar and tchar:FindFirstChild("Humanoid")
                        if root and hum and hum.Health>0 then
                            pcall(function()
                                firetouchinterest(rightHand,root,1)
                                firetouchinterest(leftHand,root,1)
                                firetouchinterest(rightHand,root,0)
                                firetouchinterest(leftHand,root,0)
                            end)
                        end
                    end
                end
                task.wait(0.05)
            end
        end)
    end
end)

-- --- Target List Dropdown ---
local targetPlayerNames={}
local selectedTarget=nil
local targetDropdown=KillTab:AddDropdown("Select Target",function(name)
    for _,player in ipairs(Players:GetPlayers()) do
        if player.DisplayName==name then
            selectedTarget=player
            break
        end
    end
end)

-- populate initial players
for _,player in ipairs(Players:GetPlayers()) do
    if player~=LocalPlayer then
        table.insert(targetPlayerNames,player.Name)
        targetDropdown:Add(player.DisplayName)
    end
end

-- update target list on join/leave
Players.PlayerAdded:Connect(function(player)
    if player~=LocalPlayer then
        table.insert(targetPlayerNames,player.Name)
        updateTargetDropdown()
    end
end)

Players.PlayerRemoving:Connect(function(player)
    for i=#targetPlayerNames,1,-1 do
        if targetPlayerNames[i]==player.Name then
            table.remove(targetPlayerNames,i)
        end
    end
    updateTargetDropdown()
    if selectedTarget and player==selectedTarget then
        selectedTarget=nil
    end
end)

local function updateTargetDropdown()
    targetDropdown:Clear()
    for _,name in ipairs(targetPlayerNames) do
        local p=Players:FindFirstChild(name)
        if p then
            targetDropdown:Add(p.DisplayName)
        end
    end
end

-- --- Start Kill with list ---
local killTargetActive=false
KillTab:AddSwitch("Start Kill Target",function(state)
    killTargetActive=state
    if state then
        task.spawn(function()
            while killTargetActive do
                local chr=LocalPlayer.Character
                local rightHand=chr and chr:FindFirstChild("RightHand")
                local leftHand=chr and chr:FindFirstChild("LeftHand")
                local punch=LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch and not chr:FindFirstChild("Punch") then
                    punch.Parent=chr
                end
                for _,name in ipairs(targetPlayerNames) do
                    local target=Players:FindFirstChild(name)
                    if target and target~=LocalPlayer and target.Character then
                        local root=target.Character:FindFirstChild("HumanoidRootPart")
                        local hum=target.Character:FindFirstChild("Humanoid")
                        if root and hum and hum.Health>0 then
                            pcall(function()
                                firetouchinterest(rightHand,root,1)
                                firetouchinterest(leftHand,root,1)
                                firetouchinterest(rightHand,root,0)
                                firetouchinterest(leftHand,root,0)
                            end)
                        end
                    end
                end
                task.wait(0.05)
            end
        end)
    end
end)

-- --- View Player (Spy) ---
local spyTargetName=nil
local spyDropdown=KillTab:AddDropdown("View Player",function(name)
    for _,player in ipairs(Players:GetPlayers()) do
        if player.DisplayName==name then
            spyTargetName=player.Name
            break
        end
    end
end)

for _,player in ipairs(Players:GetPlayers()) do
    if player~=LocalPlayer then
        spyDropdown:Add(player.DisplayName)
    end
end

local spying=false
KillTab:AddSwitch("View Player",function(state)
    spying=state
    if not spying then
        workspace.CurrentCamera.CameraSubject=LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") or nil
        return
    end
    task.spawn(function()
        while spying do
            local target=Players:FindFirstChild(spyTargetName)
            if target and target~=LocalPlayer then
                local humanoid=target.Character and target.Character:FindFirstChild("Humanoid")
                if humanoid then
                    workspace.CurrentCamera.CameraSubject=humanoid
                end
            end
            task.wait(0.1)
        end
    end)
end)

-- ============================
-- === Animation Block & Tool Override ===
-- ============================
local function setupAnimationBlock()
    local chr=LocalPlayer.Character
    if not chr or not chr:FindFirstChild("Humanoid") then return end
    local humanoid=chr.Humanoid

    -- Stop specific animations
    for _,track in pairs(humanoid:GetPlayingAnimationTracks()) do
        if track.Animation then
            local id=track.Animation.AnimationId
            local name=track.Name:lower()
            if id=="rbxassetid://3638729053" or id=="rbxassetid://3638767427" or name:match("punch") or name:match("attack") or name:match("right") then
                track:Stop()
            end
        end
    end

    -- Connect to block future animations
    if not _G.AnimBlockConnection then
        _G.AnimBlockConnection=humanoid.AnimationPlayed:Connect(function(track)
            if track.Animation then
                local id=track.Animation.AnimationId
                local name=track.Name:lower()
                if id=="rbxassetid://3638729053" or id=="rbxassetid://3638767427" or name:match("punch") or name:match("attack") or name:match("right") then
                    track:Stop()
                end
            end
        end)
    end
end

local function overrideToolActivation()
    local function processTool(tool)
        if tool and (tool.Name=="Punch" or tool.Name:match("Attack") or tool.Name:match("Right")) then
            if not tool:GetAttribute("ActivatedOverride") then
                tool:SetAttribute("ActivatedOverride",true)
                local conn=tool.Activated:Connect(function()
                    task.wait(0.05)
                    local chr=LocalPlayer.Character
                    if chr and chr:FindFirstChild("Humanoid") then
                        for _,track in pairs(chr.Humanoid:GetPlayingAnimationTracks()) do
                            if track.Animation then
                                local id=track.Animation.AnimationId
                                local name=track.Name:lower()
                                if id=="rbxassetid://3638729053" or id=="rbxassetid://3638767427" or name:match("punch") or name:match("attack") or name:match("right") then
                                    track:Stop()
                                end
                            end
                        end
                    end
                end)
                if not _G.ToolConnections then _G.ToolConnections={} end
                _G.ToolConnections[tool]=conn
            end
        end
    end
    for _,tool in pairs(LocalPlayer.Backpack:GetChildren()) do
        processTool(tool)
    end
    local chr=LocalPlayer.Character
    if chr then
        for _,tool in pairs(chr:GetChildren()) do
            if tool:IsA("Tool") then
                processTool(tool)
            end
        end
    end
    if not _G.BackpackAddedConnection then
        _G.BackpackAddedConnection=LocalPlayer.Backpack.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                task.wait(0.1)
                processTool(child)
            end
        end)
    end
    if not _G.CharacterToolAddedConnection then
        _G.CharacterToolAddedConnection=chr.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                task.wait(0.1)
                processTool(child)
            end
        end)
    end
end

local function recoverPunchAnimations()
    if _G.AnimBlockConnection then _G.AnimBlockConnection:Disconnect() _G.AnimBlockConnection=nil end
    if _G.ToolConnections then
        for _,conn in pairs(_G.ToolConnections) do
            if conn then conn:Disconnect() end
        end
        _G.ToolConnections=nil
    end
    if _G.BackpackAddedConnection then _G.BackpackAddedConnection:Disconnect() _G.BackpackAddedConnection=nil end
    if _G.CharacterToolAddedConnection then _G.CharacterToolAddedConnection:Disconnect() _G.CharacterToolAddedConnection=nil end
end

-- Button to recover animations
KillTab:AddButton("Recover Punch Anim", function()
    recoverPunchAnimations()
end)

-- Auto Equip Punch
KillTab:AddSwitch("Auto Equip Punch", function(state)
    autoEquipPunch=state
    if state then
        task.spawn(function()
            while autoEquipPunch do
                local punch=LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent=LocalPlayer.Character
                end
                task.wait(0.1)
            end
        end)
    end
end)

-- Auto Punch without animation
KillTab:AddSwitch("Auto Punch [no animation]",function(state)
    autoPunchNoAnim=state
    if state then
        task.spawn(function()
            while autoPunchNoAnim do
                local punch=LocalPlayer.Backpack:FindFirstChild("Punch") or LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Punch")
                if punch then
                    if punch.Parent~=LocalPlayer.Character then
                        punch.Parent=LocalPlayer.Character
                    end
                    LocalPlayer.muscleEvent:FireServer("punch","rightHand")
                    LocalPlayer.muscleEvent:FireServer("punch","leftHand")
                else
                    autoPunchNoAnim=false
                end
                task.wait(0.01)
            end
        end)
    end
end)

-- Auto Punch
KillTab:AddSwitch("Auto Punch", function(state)
    _G.fastHitActive=state
    if state then
        task.spawn(function()
            while _G.fastHitActive do
                local punch=LocalPlayer.Backpack:FindFirstChild("Punch")
                if punch then
                    punch.Parent=LocalPlayer.Character
                    if punch:FindFirstChild("attackTime") then
                        punch.attackTime.Value=0
                    end
                end
                task.wait(0.05)
                LocalPlayer.muscleEvent:FireServer("punch","rightHand")
                LocalPlayer.muscleEvent:FireServer("punch","leftHand")
                local chr=LocalPlayer.Character
                if chr then
                    local punchTool=chr:FindFirstChild("Punch")
                    if punchTool then
                        punchTool:Activate()
                    end
                end
                task.wait()
            end
        end)
    else
        local chr=LocalPlayer.Character
        local punch=chr and chr:FindFirstChild("Punch")
        if punch then punch.Parent=LocalPlayer.Backpack end
    end
end)

-- --- God Mode (Good Mode) ---
local godMode=false
KillTab:AddSwitch("Good Mode", function(state)
    godMode=state
    if state then
        task.spawn(function()
            while godMode do
                game:GetService("ReplicatedStorage").rEvents.brawlEvent:FireServer("joinBrawl")
                task.wait()
            end
        end)
    end
end)

-- ============================
-- ==== Teleport & Follow ====
-- ============================

local following=false
local followTarget=nil

local function followPlayer(target)
    local hrp=LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local targethrp=target.Character and target.Character:FindFirstChild("HumanoidRootPart")
    if hrp and targethrp then
        local pos=targethrp.Position - (targethrp.CFrame.LookVector*3)
        hrp.CFrame=CFrame.new(pos,targethrp.Position)
    end
end

local followDropdown=KillTab:AddDropdown("Teleport Player",function(name)
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr.DisplayName==name then
            followTarget=plr.Name
            following=true
            print("✅ Following: "..plr.Name)
            followPlayer(plr)
            break
        end
    end
end)

for _,player in ipairs(Players:GetPlayers()) do
    if player~=LocalPlayer then
        followDropdown:Add(player.DisplayName)
    end
end

Players.PlayerAdded:Connect(function(plr)
    if plr~=LocalPlayer then
        followDropdown:Add(plr.DisplayName)
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    if followTarget==plr.Name then
        followTarget=nil
        following=false
    end
    -- refresh list
    followDropdown:Clear()
    for _,p in ipairs(Players:GetPlayers()) do
        if p~=LocalPlayer then
            followDropdown:Add(p.DisplayName)
        end
    end
end)

local function followLoop()
    task.spawn(function()
        while true do
            if following and followTarget then
                local target=Players:FindFirstChild(followTarget)
                if target then
                    followPlayer(target)
                else
                    following=false
                end
            end
            task.wait(0.01)
        end
    end)
end

local function onRespawn()
    LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1)
        if following and followTarget then
            local target=Players:FindFirstChild(followTarget)
            if target then
                followPlayer(target)
            end
        end
    end)
end

KillTab:AddButton("Unfollow", function()
    following=false
    followTarget=nil
    print("⛔ Stopped following")
end)

followLoop()
onRespawn()

-- --- Auto Slam ---
local autoSlam=false
KillTab:AddSwitch("Auto Slams", function(state)
    autoSlam=state
    if state then
        task.spawn(function()
            while autoSlam do
                local slam=LocalPlayer.Backpack:FindFirstChild("Ground Slam") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Ground Slam"))
                if slam then
                    if slam.Parent==LocalPlayer.Backpack then
                        slam.Parent=LocalPlayer.Character
                    end
                    if slam:FindFirstChild("attackTime") then
                        slam.attackTime.Value=0
                    end
                    LocalPlayer.muscleEvent:FireServer("slam")
                    slam:Activate()
                end
                task.wait(0.1)
            end
        end)
    end
end)

-- --- Combo NaN Button ---
KillTab:AddButton("Combo NaN", function()
    local args={"changeSize",0/0}
    game:GetService("ReplicatedStorage"):WaitForChild("rEvents"):WaitForChild("changeSpeedSizeRemote"):InvokeServer(unpack(args))
end)

-- --- URL Scripts ---
local urls={
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack2",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack4",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack5",
    "https://raw.githubusercontent.com/SadOz8/Stuffs/refs/heads/main/Crack6"
}
KillTab:AddButton("Touch Me!",function()
    for _,url in ipairs(urls) do
        spawn(function()
            local success,response=pcall(function() return game:HttpGet(url) end)
            if success and response then
                local loadSuccess,err=pcall(function() loadstring(response)() end)
                if not loadSuccess then warn("[Error] executing raw:",url,err) end
            else
                warn("[Error] could not load:",url)
            end
        end)
    end
end)

-- ============================
-- === Lighting Time Control ===
-- ============================
local timeOptions={"Morning","Noon","Afternoon","Sunset","Night","Midnight","Dawn","Early Morning"}
local timeDropdown=KillTab:AddDropdown("Change Time",function(selection)
    print("Selected:",selection)
    -- Reset defaults
    Lighting.Brightness=2
    Lighting.FogEnd=100000
    Lighting.Ambient=Color3.fromRGB(127,127,127)
    if selection=="Morning" then
        Lighting.ClockTime=6
        Lighting.Brightness=2
        Lighting.Ambient=Color3.fromRGB(200,200,255)
    elseif selection=="Noon" then
        Lighting.ClockTime=12
        Lighting.Brightness=3
        Lighting.Ambient=Color3.fromRGB(255,255,255)
    elseif selection=="Afternoon" then
        Lighting.ClockTime=16
        Lighting.Brightness=2.5
        Lighting.Ambient=Color3.fromRGB(255,220,180)
    elseif selection=="Sunset" then
        Lighting.ClockTime=18
        Lighting.Brightness=2
        Lighting.Ambient=Color3.fromRGB(255,150,100)
        Lighting.FogEnd=500
    elseif selection=="Night" then
        Lighting.ClockTime=20
        Lighting.Brightness=1.5
        Lighting.Ambient=Color3.fromRGB(100,100,150)
        Lighting.FogEnd=800
    elseif selection=="Midnight" then
        Lighting.ClockTime=0
        Lighting.Brightness=1
        Lighting.Ambient=Color3.fromRGB(50,50,100)
        Lighting.FogEnd=400
    elseif selection=="Dawn" then
        Lighting.ClockTime=4
        Lighting.Brightness=1.8
        Lighting.Ambient=Color3.fromRGB(180,180,220)
    elseif selection=="Early Morning" then
        Lighting.ClockTime=2
        Lighting.Brightness=1.2
        Lighting.Ambient=Color3.fromRGB(100,120,180)
    end
end)
for _,option in ipairs(timeOptions) do timeDropdown:Add(option) end

print("All features loaded!")

-- ============================
-- ======= Farm OP =========== 
-- ============================

local FarmTab=MainWindow:AddTab("Farm OP")
FarmTab:Show()

-- Auto farm toggle
getgenv()._AutoRepfarm=false
FarmTab:AddSwitch("Auto Farm (Use if ping < 250ms)", function(state)
    getgenv()._AutoRepfarm=state
end)

local function getPing()
    local success,ping=pcall(function() return Stats.Network.ServerStatsItem["Data Ping"]:GetValue() end)
    return success and ping or 999
end

local function updateCharacter()
    local character=LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hrp=character:WaitForChild("HumanoidRootPart")
    return hrp
end

local function equipPet()
    local petFolder=LocalPlayer:FindFirstChild("petsFolder")
    if petFolder and petFolder:FindFirstChild("Unique") then
        for _,pet in pairs(petFolder.Unique:GetChildren()) do
            if pet.Name=="Swift Samurai" then
                ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet",pet)
                break
            end
        end
    end
end

local lastEgg=0
local lastRock=0
local eggInterval=30*60 -- 30 minutes
local rockInterval=1
local maxPing=1100
local minPing=300

task.spawn(function()
    while true do
        if getgenv()._AutoRepfarm then
            local ping=getPing()
            if ping>maxPing then
                warn("[AutoFarm] High ping, pausing for 5s.")
                task.wait(5)
            else
                -- Repetitions
                for i=1,185 do
                    LocalPlayer.muscleEvent:FireServer("rep")
                end
                -- Egg
                if tick()-lastEgg>=eggInterval then
                    local egg=LocalPlayer.Backpack:FindFirstChild("ProteinEgg")
                    if egg then
                        egg.Parent=LocalPlayer.Character
                        egg:Activate()
                    end
                    lastEgg=tick()
                end
                -- Hit Rock
                if tick()-lastRock>=rockInterval then
                    local rock=workspace:FindFirstChild("Rock5M")
                    local hrp=updateCharacter()
                    if rock and hrp then
                        hrp.CFrame=rock.CFrame*CFrame.new(0,0,-5)
                        -- Fire hit event
                        ReplicatedStorage.rEvents.hitEvent:FireServer("hit",rock)
                    end
                    lastRock=tick()
                end
            end
        end
        task.wait(0.01)
    end
end)

-- Additional features like auto-eat eggs, auto-lag, etc., can be added following the same pattern.

-- -- Additional features: auto spin, jungle squat, rebirth, stats, calculator, etc.
-- Continue adding features similarly

-- END OF SCRIPT
