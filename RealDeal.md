--noli real!!

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

local ANIM_IDS = {
    Idle = "rbxassetid://83465205704188",
    Walk = "rbxassetid://103292185212679",
    Run = "rbxassetid://103292185212679",
    M1_1 = "rbxassetid://109230267448394",
    M1_2 = "rbxassetid://109230267448394",
    Blocked = "rbxassetid://80277760801310" -- Edit this to whatever yu want this is Mafioso's M1 so it doesnt override with the other animations m1 and stuff
}

local animations = {}
for name, id in pairs(ANIM_IDS) do
    animations[name] = Instance.new("Animation")
    animations[name].AnimationId = id
end

local tracks = {
    Idle = humanoid:LoadAnimation(animations.Idle),
    Walk = humanoid:LoadAnimation(animations.Walk),
    Run = humanoid:LoadAnimation(animations.Run),
    M1_1 = humanoid:LoadAnimation(animations.M1_1),
    M1_2 = humanoid:LoadAnimation(animations.M1_2)
}

local isMoving = false
local isRunning = false
local canAttack = true
local nextIsAltAttack = false
local ATTACK_COOLDOWN = 2

local function setupAnimationBlocker(animator)
    animator.AnimationPlayed:Connect(function(track)
        if track.Animation.AnimationId == ANIM_IDS.Blocked then
            track:Stop()
        end
    end)
end

local animator = humanoid:FindFirstChildOfClass("Animator")
if animator then
    setupAnimationBlocker(animator)
else
    humanoid.ChildAdded:Connect(function(child)
        if child:IsA("Animator") then
            setupAnimationBlocker(child)
        end
    end)
end

local function updateMovementAnimations()
    if isMoving then
        if isRunning then
            if not tracks.Run.IsPlaying then
                tracks.Idle:Stop()
                tracks.Walk:Stop()
                tracks.Run:Play()
            end
        else
            if not tracks.Walk.IsPlaying then
                tracks.Idle:Stop()
                tracks.Run:Stop()
                tracks.Walk:Play()
            end
        end
    else
        if not tracks.Idle.IsPlaying then
            tracks.Walk:Stop()
            tracks.Run:Stop()
            tracks.Idle:Play()
        end
    end
end

local function handleAttack()
    if not canAttack then return end
    canAttack = false
    local attackTrack = nextIsAltAttack and tracks.M1_2 or tracks.M1_1
    attackTrack:Play()
    tracks.Idle:Stop()
    tracks.Walk:Stop()
    tracks.Run:Stop()
    nextIsAltAttack = not nextIsAltAttack
    task.wait(ATTACK_COOLDOWN)
    canAttack = true
end

game:GetService("RunService").Heartbeat:Connect(function()
    isMoving = humanoid.MoveDirection.Magnitude > 0
    isRunning = humanoid.WalkSpeed > 16
    updateMovementAnimations()
end)

game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.UserInputType == Enum.UserInputType.MouseButton1 then
        handleAttack()
    end
end)

humanoid.WalkSpeed = 16
updateMovementAnimations()
