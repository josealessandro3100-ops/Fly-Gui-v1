--[[ Roblox Fly GUI Script Author: (Your Name) Description: Creates a simple on‑screen menu with a toggle button that lets the local player fly. Includes: - Fly Toggle Button - Speed TextBox (number) - Optional keybind (F) to toggle fly - Safe (non exploit) usage intended for YOUR own games.

HOW IT WORKS
  * When flying, the script reads movement input (W/A/S/D, Space = up, LeftControl = down)
    relative to the camera's look vector, then sets the HumanoidRootPart velocity.
  * Physics is restored when flight ends.

SETUP
  1. Place this LocalScript inside StarterPlayer > StarterPlayerScripts OR
     insert it under StarterGui (as a LocalScript) and remove the auto-GUI creation
     part if you build the UI manually.
  2. Play test. Click the Fly button or press F to toggle.

NOTES / BEST PRACTICES
  * Do NOT use this to circumvent other creators' games or anti‑cheat systems.
  * Adjust MAX_SPEED_LIMIT if you want to clamp speed harder.
  * This script is intentionally simple; refine for production (permissions, roles, etc.).

--]]

-- CONFIGURABLE CONSTANTS local DEFAULT_SPEED = 60          -- Starting fly speed (studs per second) local MAX_SPEED_LIMIT = 300       -- Hard cap for user-entered speed local TOGGLE_KEY = Enum.KeyCode.F -- Keyboard toggle key

-- SERVICES local Players = game:GetService("Players") local UserInputService = game:GetService("UserInputService") local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer local character = localPlayer.Character or localPlayer.CharacterAdded:Wait() local humanoid = character:WaitForChild("Humanoid") local rootPart = character:WaitForChild("HumanoidRootPart")

-- STATE local flying = false local flySpeed = DEFAULT_SPEED local lastCharacterConn

-- GUI CREATION --------------------------------------------------------------- local screenGui = Instance.new("ScreenGui") screenGui.Name = "FlyMenuGui" screenGui.ResetOnSpawn = false screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame") frame.Name = "MenuFrame" frame.Size = UDim2.new(0, 200, 0, 110) frame.Position = UDim2.new(0, 20, 0, 120) frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25) frame.BorderSizePixel = 0 frame.BackgroundTransparency = 0.1 frame.Parent = screenGui

local uiCorner = Instance.new("UICorner") uiCorner.CornerRadius = UDim.new(0, 8) uiCorner.Parent = frame

local title = Instance.new("TextLabel") title.Name = "Title" title.Size = UDim2.new(1, -10, 0, 24) title.Position = UDim2.new(0, 5, 0, 5) title.BackgroundTransparency = 1 title.Text = "Fly Menu" title.TextColor3 = Color3.fromRGB(255,255,255) title.Font = Enum.Font.GothamBold title.TextSize = 18 title.Parent = frame

local flyButton = Instance.new("TextButton") flyButton.Name = "FlyButton" flyButton.Size = UDim2.new(1, -20, 0, 30) flyButton.Position = UDim2.new(0, 10, 0, 35) flyButton.BackgroundColor3 = Color3.fromRGB(60, 120, 255) flyButton.TextColor3 = Color3.fromRGB(255,255,255) flyButton.Font = Enum.Font.GothamSemibold flyButton.TextSize = 18 flyButton.Text = "Enable Fly" flyButton.AutoButtonColor = true flyButton.Parent = frame

local buttonCorner = Instance.new("UICorner") buttonCorner.CornerRadius = UDim.new(0,6) buttonCorner.Parent = flyButton

local speedLabel = Instance.new("TextLabel") speedLabel.Name = "SpeedLabel" speedLabel.Size = UDim2.new(0.45, -15, 0, 24) speedLabel.Position = UDim2.new(0, 10, 0, 70) speedLabel.BackgroundTransparency = 1 speedLabel.Text = "Speed:" speedLabel.TextColor3 = Color3.fromRGB(255,255,255) speedLabel.Font = Enum.Font.Gotham speedLabel.TextSize = 16 speedLabel.TextXAlignment = Enum.TextXAlignment.Left speedLabel.Parent = frame

local speedBox = Instance.new("TextBox") speedBox.Name = "SpeedBox" speedBox.Size = UDim2.new(0.55, -15, 0, 24) speedBox.Position = UDim2.new(0.45, 5, 0, 70) speedBox.BackgroundColor3 = Color3.fromRGB(50,50,50) speedBox.TextColor3 = Color3.fromRGB(255,255,255) speedBox.Font = Enum.Font.Gotham speedBox.TextSize = 16 speedBox.Text = tostring(flySpeed) speedBox.ClearTextOnFocus = false speedBox.Parent = frame

local speedCorner = Instance.new("UICorner") speedCorner.CornerRadius = UDim.new(0,6) speedCorner.Parent = speedBox

-- DRAG SUPPORT (optional) local dragging = false local dragOffset frame.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true dragOffset = input.Position - frame.AbsolutePosition end end) UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end) UserInputService.InputChanged:Connect(function(input) if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then frame.Position = UDim2.fromOffset(input.Position.X - dragOffset.X, input.Position.Y - dragOffset.Y) end end)

-- FUNCTIONS ----------------------------------------------------------------- local function clampSpeed(value) if value < 0 then return 0 elseif value > MAX_SPEED_LIMIT then return MAX_SPEED_LIMIT end return value end

local function updateButtonVisual() if flying then flyButton.Text = "Disable Fly" flyButton.BackgroundColor3 = Color3.fromRGB(255, 100, 100) else flyButton.Text = "Enable Fly" flyButton.BackgroundColor3 = Color3.fromRGB(60, 120, 255) end end

local function ensureCharacter() character = localPlayer.Character or localPlayer.CharacterAdded:Wait() humanoid = character:WaitForChild("Humanoid") rootPart = character:WaitForChild("HumanoidRootPart") end

local function setFlying(state) if state == flying then return end flying = state if flying then -- Optionally: humanoid:ChangeState(Enum.HumanoidStateType.Physics) -- We avoid PlatformStand so animations can still play (though physics overrides velocity anyway). else -- Restore default movement (no special state needed if we didn't force one) end updateButtonVisual() end

local function getMovementInput() local moveVector = Vector3.zero -- WASD movement from Humanoid (works with default controls) moveVector = humanoid.MoveDirection -- Already camera-relative for default controls

-- Vertical control: Space (up), LeftControl (down)
if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
    moveVector += Vector3.new(0,1,0)
end
if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.C) then
    moveVector += Vector3.new(0,-1,0)
end

if moveVector.Magnitude > 0 then
    moveVector = moveVector.Unit
end
return moveVector

end

-- FLIGHT LOOP ---------------------------------------------------------------- RunService.Heartbeat:Connect(function(dt) if flying and rootPart and humanoid then local desiredDir = getMovementInput() local desiredVelocity = desiredDir * flySpeed

-- Preserve some vertical velocity if no input? We'll just override fully for simplicity.
    rootPart.AssemblyLinearVelocity = Vector3.new(
        desiredVelocity.X,
        desiredVelocity.Y,
        desiredVelocity.Z
    )
end

end)

-- INPUT HANDLERS ------------------------------------------------------------- flyButton.MouseButton1Click:Connect(function() setFlying(not flying) end)

speedBox.FocusLost:Connect(function(enterPressed) local num = tonumber(speedBox.Text) if num then flySpeed = clampSpeed(num) else speedBox.Text = tostring(flySpeed) end end)

UserInputService.InputBegan:Connect(function(input, processed) if processed then return end if input.KeyCode == TOGGLE_KEY then setFlying(not flying) end end)

-- CHARACTER RESPAWN HANDLING ------------------------------------------------- localPlayer.CharacterAdded:Connect(function(newChar) character = newChar humanoid = character:WaitForChild("Humanoid") rootPart = character:WaitForChild("HumanoidRootPart") flying = false updateButtonVisual() end)

-- INITIAL updateButtonVisual() print("[FlyMenu] Loaded. Press F or click button to toggle flight.") --[[ Roblox Fly GUI Script Author: (Your Name) Description: Creates a simple on‑screen menu with a toggle button that lets the local player fly. Includes: - Fly Toggle Button - Speed TextBox (number) - Optional keybind (F) to toggle fly - Safe (non exploit) usage intended for YOUR own games.

HOW IT WORKS
  * When flying, the script reads movement input (W/A/S/D, Space = up, LeftControl = down) relative to the camera.
  * It sets the HumanoidRootPart.AssemblyLinearVelocity toward the desired direction at the chosen speed.
  * This is meant for Studio / your own place. Do NOT use to exploit other creators' games.

INSTALL
  1. In Studio, StarterPlayer > StarterPlayerScripts: Insert a LocalScript named "FlyClient".
  2. Paste this entire code.
  3. Play test.

NOTES
  * Uses modern APIs (UserInputService, RunService) rather than deprecated BodyMovers.
  * Adjust MAX_SPEED_LIMIT or defaultSpeed as you wish.
  * Network ownership: As the local player controls their own character, directly setting AssemblyLinearVelocity client-side is fine for a local, custom game mechanic.
  * Add server‑side checks in production games if you need anti‑cheat.

--]]

--// SERVICES local Players = game:GetService("Players") local UserInputService = game:GetService("UserInputService") local RunService = game:GetService("RunService") local ContextActionService = game:GetService("ContextActionService")

--// CONSTANTS local TOGGLE_KEY = Enum.KeyCode.F      -- Keybind to toggle fly local UP_KEY     = Enum.KeyCode.Space local DOWN_KEY   = Enum.KeyCode.LeftControl local MAX_SPEED_LIMIT = 300            -- Hard cap for safety local defaultSpeed = 60                -- Default speed when script starts

--// STATE local localPlayer = Players.LocalPlayer local character = localPlayer.Character or localPlayer.CharacterAdded:Wait() local humanoid = character:WaitForChild("Humanoid") local rootPart = character:WaitForChild("HumanoidRootPart")

local flying = false local flySpeed = defaultSpeed local moveVector = Vector3.zero local connection -- heartbeat connection

--// UI CREATION local screenGui = Instance.new("ScreenGui") screenGui.Name = "FlyMenu" screenGui.ResetOnSpawn = false screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame") frame.Name = "MenuFrame" frame.Size = UDim2.new(0, 180, 0, 120) frame.Position = UDim2.new(0, 20, 0, 200) frame.BackgroundColor3 = Color3.fromRGB(30, 30, 35) frame.BackgroundTransparency = 0.1 frame.BorderSizePixel = 0 frame.Parent = screenGui

local uiCorner = Instance.new("UICorner") uiCorner.CornerRadius = UDim.new(0, 8) uiCorner.Parent = frame

local uiStroke = Instance.new("UIStroke") uiStroke.Thickness = 1 uiStroke.Color = Color3.fromRGB(90, 90, 120) uiStroke.Parent = frame

local title = Instance.new("TextLabel") title.Size = UDim2.new(1, -10, 0, 24) title.Position = UDim2.new(0, 5, 0, 5) title.BackgroundTransparency = 1 title.Text = "Fly Menu" title.Font = Enum.Font.GothamBold title.TextSize = 18 title.TextColor3 = Color3.fromRGB(255,255,255) title.TextXAlignment = Enum.TextXAlignment.Left title.Parent = frame

local flyButton = Instance.new("TextButton") flyButton.Name = "FlyButton" flyButton.Size = UDim2.new(1, -20, 0, 32) flyButton.Position = UDim2.new(0, 10, 0, 34) flyButton.Text = "Enable Fly" flyButton.Font = Enum.Font.GothamBold flyButton.TextSize = 16 flyButton.TextColor3 = Color3.fromRGB(255,255,255) flyButton.BackgroundColor3 = Color3.fromRGB(50, 120, 200) flyButton.AutoButtonColor = true flyButton.Parent = frame

local flyCorner = Instance.new("UICorner") flyCorner.CornerRadius = UDim.new(0,6) flyCorner.Parent = flyButton

local speedLabel = Instance.new("TextLabel") speedLabel.Size = UDim2.new(0.55, -15, 0, 26) speedLabel.Position = UDim2.new(0, 10, 0, 74) speedLabel.BackgroundTransparency = 1 speedLabel.Text = "Speed:" .. tostring(flySpeed) speedLabel.Font = Enum.Font.Gotham yy =  speedLabel.TextSize or 14 speedLabel.TextSize = 14 speedLabel.TextColor3 = Color3.fromRGB(255,255,255) speedLabel.TextXAlignment = Enum.TextXAlignment.Left speedLabel.Parent = frame

local speedBox = Instance.new("TextBox") speedBox.Size = UDim2.new(0.45, -15, 0, 26) speedBox.Position = UDim2.new(0.55, 5, 0, 74) speedBox.BackgroundColor3 = Color3.fromRGB(45,45,60) speedBox.Text = tostring(flySpeed) speedBox.ClearTextOnFocus = false speedBox.Font = Enum.Font.Gotham speedBox.TextSize = 14 speedBox.TextColor3 = Color3.fromRGB(255,255,255) speedBox.PlaceholderText = "Speed" speedBox.Parent = frame

local speedCorner = Instance.new("UICorner") speedCorner.CornerRadius = UDim.new(0,6) speedCorner.Parent = speedBox

local info = Instance.new("TextLabel") info.Size = UDim2.new(1, -10, 0, 18) info.Position = UDim2.new(0, 5, 1, -20) info.BackgroundTransparency = 1 info.Text = "Key: F" info.Font = Enum.Font.Gotham info.TextSize = 12 info.TextColor3 = Color3.fromRGB(180,180,200) info.TextXAlignment = Enum.TextXAlignment.Left info.Parent = frame

-- Draggable (simple) frame.Active = true frame.Draggable = true -- (For new UI system you may implement custom drag logic)

--// FUNCTIONS local function updateButton() flyButton.Text = flying and "Disable Fly" or "Enable Fly" flyButton.BackgroundColor3 = flying and Color3.fromRGB(200,70,70) or Color3.fromRGB(50,120,200) end

local function startFlying() if flying then return end flying = true updateButton() -- Optional: Prevent fall damage or ragdoll states while flying humanoid.PlatformStand = false

connection = RunService.Heartbeat:Connect(function(dt)
    -- Compute camera-relative movement
    local cam = workspace.CurrentCamera
    local forward = cam.CFrame.LookVector
    local right = cam.CFrame.RightVector

    -- Flatten forward and right on XZ for horizontal plane, we still allow vertical keys
    forward = Vector3.new(forward.X, 0, forward.Z).Unit
    right = Vector3.new(right.X, 0, right.Z).Unit

    if forward.Magnitude ~= forward.Magnitude then -- NaN guard
        forward = Vector3.new(0,0,-1)
    end
    if right.Magnitude ~= right.Magnitude then
        right = Vector3.new(1,0,0)
    end

    local desiredDir = Vector3.zero
    desiredDir += moveVector.Z * forward  -- moveVector Z used for forward/back
    desiredDir += moveVector.X * right    -- moveVector X used for left/right

    -- Vertical
    if UserInputService:IsKeyDown(UP_KEY) then
        desiredDir += Vector3.new(0,1,0)
    end
    if UserInputService:IsKeyDown(DOWN_KEY) then
        desiredDir += Vector3.new(0,-1,0)
    end

    if desiredDir.Magnitude > 0 then
        desiredDir = desiredDir.Unit
    end

    local targetVelocity = desiredDir * flySpeed
    -- Apply velocity smoothly
    rootPart.AssemblyLinearVelocity = targetVelocity
    -- Optional: keep character upright
    rootPart.CFrame = CFrame.new(rootPart.Position, rootPart.Position + forward)
end)

end

local function stopFlying() if not flying then return end flying = false updateButton() if connection then connection:Disconnect() connection = nil end -- Let physics resume; set downward velocity slight so player lands rootPart.AssemblyLinearVelocity = Vector3.new(0, -5, 0) end

local function toggleFly() if flying then stopFlying() else startFlying() end end

--// INPUT HANDLING local function onInputBegan(input, gp) if gp then return end if input.KeyCode == TOGGLE_KEY then toggleFly() end

-- Movement keys (WASD)
if input.KeyCode == Enum.KeyCode.W then
    moveVector = Vector3.new(moveVector.X, 0, 1)
elseif input.KeyCode == Enum.KeyCode.S then
    moveVector = Vector3.new(moveVector.X, 0, -1)
elseif input.KeyCode == Enum.KeyCode.A then
    moveVector = Vector3.new(-1, 0, moveVector.Z)
elseif input.KeyCode == Enum.KeyCode.D then
    moveVector = Vector3.new(1, 0, moveVector.Z)
end

end

local function onInputEnded(input, gp) if gp then return end if input.KeyCode == Enum.KeyCode.W and moveVector.Z == 1 then moveVector = Vector3.new(moveVector.X, 0, 0) elseif input.KeyCode == Enum.KeyCode.S and moveVector.Z == -1 then moveVector = Vector3.new(moveVector.X, 0, 0) elseif input.KeyCode == Enum.KeyCode.A and moveVector.X == -1 then moveVector = Vector3.new(0, 0, moveVector.Z) elseif input.KeyCode == Enum.KeyCode.D and moveVector.X == 1 then moveVector = Vector3.new(0, 0, moveVector.Z) end end

UserInputService.InputBegan:Connect(onInputBegan) UserInputService.InputEnded:Connect(onInputEnded)

-- Button click flyButton.MouseButton1Click:Connect(toggleFly)

-- Speed change speedBox.FocusLost:Connect(function(enterPressed) local value = tonumber(speedBox.Text) if value then value = math.clamp(value, 5, MAX_SPEED_LIMIT) flySpeed = value speedLabel.Text = "Speed:" .. tostring(flySpeed) speedBox.Text = tostring(flySpeed) else speedBox.Text = tostring(flySpeed) end end)

-- Character respawn handling localPlayer.CharacterAdded:Connect(function(newChar) character = newChar humanoid = character:WaitForChild("Humanoid") rootPart = character:WaitForChild("HumanoidRootPart") if flying then -- Re-init flight after respawn stopFlying() startFlying() end end)

updateButton() print("[FlyClient] Loaded. Press '" .. TOGGLE_KEY.Name .. "' or use the GUI button.") --[[ Roblox Fly GUI Script Author: (Your Name) Description: Creates a simple on‑screen menu with a toggle button that lets the local player fly. Includes: - Fly Toggle Button - Speed TextBox (number) - Optional keybind (F) to toggle fly - Safe (non exploit) usage intended for YOUR own games.

HOW IT WORKS
  * When flying, the script reads movement input (W/A/S/D, Space = up, LeftControl = down) and sets
    the player's HumanoidRootPart.AssemblyLinearVelocity each frame toward a desired velocity.
  * Uses camera forward/right directions so movement feels natural.

INSTALLATION
  1. Place this LocalScript in StarterPlayer > StarterPlayerScripts **or** StarterGui.
  2. Test in Play mode.
  3. Adjust defaults (DEFAULT_SPEED, MAX_SPEED) as desired.

IMPORTANT NOTES
  * This is for learning / your own game. Do NOT use scripts to exploit other creators' games.
  * Excessively high speeds can cause the server to correct your position.
  * You can adapt this to use VectorForce if you prefer forces over setting velocity directly.

--]]

-- // CONFIGURATION --------------------------------------------------------- local TOGGLE_KEY = Enum.KeyCode.F      -- Keyboard shortcut to toggle fly local UP_KEY     = Enum.KeyCode.Space local DOWN_KEY   = Enum.KeyCode.LeftControl local DEFAULT_SPEED = 70               -- studs per second local MAX_SPEED     = 300              -- safety clamp local UI_SIZE       = UDim2.fromOffset(160, 130)

-- // SERVICES -------------------------------------------------------------- local Players = game:GetService("Players") local UserInputService = game:GetService("UserInputService") local RunService = game:GetService("RunService") local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer local character = player.Character or player.CharacterAdded:Wait() local humanoidRoot = character:WaitForChild("HumanoidRootPart") local humanoid = character:WaitForChild("Humanoid")

-- Re-fetch character references on respawn player.CharacterAdded:Connect(function(char) character = char humanoidRoot = character:WaitForChild("HumanoidRootPart") humanoid = character:WaitForChild("Humanoid") end)

-- // STATE ----------------------------------------------------------------- local flying = false local desiredSpeed = DEFAULT_SPEED local moveDir = Vector3.zero

-- // GUI CREATION ---------------------------------------------------------- local screenGui = Instance.new("ScreenGui") screenGui.Name = "FlyMenu" screenGui.ResetOnSpawn = false screenGui.IgnoreGuiInset = true screenGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame") frame.Name = "Container" frame.Size = UI_SIZE frame.Position = UDim2.new(0, 20, 0, 100) frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25) frame.BackgroundTransparency = 0.15 frame.BorderSizePixel = 0 frame.Parent = screenGui

local corner = Instance.new("UICorner") corner.CornerRadius = UDim.new(0, 10) corner.Parent = frame

local uiList = Instance.new("UIListLayout") uiList.FillDirection
