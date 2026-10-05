--[[
    TestToolkit v85
    - Fling K1LAS1K interno (9e9 + BodyVelocity P=20000 + 60 iterações)
    - Botão pra carregar o K1LAS1K original externamente
    - Touch Fling com o mesmo método
    - Silent Aim via flick
    - Wall check via câmera
    - ESP apenas Highlight
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")
local VirtualInputManager = game:GetService("VirtualInputManager")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
Workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
	if Workspace.CurrentCamera then Camera = Workspace.CurrentCamera end
end)

local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled

--------------------------------------------------------------------
-- SETTINGS
--------------------------------------------------------------------
local Settings = {
	ESP = true, Rainbow = true, ShowNames = true, ShowHealth = true,
	Wallhack = true, ESPMaxDist = 600, ESPMaxTargets = 12,
	ESPHighlight = true, ESPColor = Color3.fromRGB(255, 80, 80), ESPFillTrans = 0.65,
	ESPBox = 1, Tracers = false, TracerOrigin = 1,

	TargetMode = 3,
	AimEnabled = true, UseLegitAim = true, UseSilentAim = false,
	AimMode = 2, AimPart = 3, AimWallCheck = true,
	Smoothness = 0.15, SnapAngle = 6, AutoPredict = true, PredictScale = 0.6,
	AimAtCursor = true, Prediction = 0,
	ShotType = 1, FireLock = true, Priority = 1,
	AimKey = Enum.KeyCode.Q,
	SwitchKey = Enum.KeyCode.E,
	FOVEnabled = true, FOVRadius = 300, AimMaxDist = 0,
	SilentAimFOV = 300, SilentAimVisible = true,

	TriggerBot = false, TriggerBotAlways = false,
	TriggerFOV = 100, TriggerVisible = true, TriggerDelay = 0.01,

	HitboxExpander = false, HitboxSize = 6, HitboxRange = 200,
	HitboxInvisible = false,
	ExpandHead = true, ExpandTorso = true, ExpandUpperTorso = true, ExpandLowerTorso = false,

	WalkSpeedOn = false, WalkSpeed = 32,
	JumpOn = false, JumpPower = 100,
	Noclip = false, NoclipKey = Enum.KeyCode.N,
	Fly = false, FlyKey = Enum.KeyCode.F, FlySpeed = 60,
	AntiVoid = false,
	AntiVoidY = math.max(Workspace.FallenPartsDestroyHeight + 100, -400),
	AntiFling = false, AntiFlingSpeed = 160,
	InfJump = false, FullBright = false, NoFog = false,
	CamFOVOn = false, CamFOV = 90, AntiAFK = true,

	SeatInvisible = false, SeatInvisibleKey = Enum.KeyCode.I,
	SeatInvisibleX = -25.95, SeatInvisibleY = 84, SeatInvisibleZ = 3537.55,

	Fling = false, FlingKey = Enum.KeyCode.G,
	FlingRange = 12, FlingPower = 3000,
	FlingRepeat = 0.1,
	FlingPlayerTarget = "Nenhum",
	FlingPlayerDuration = 1,
	FlingPlayerKey = Enum.KeyCode.B,

	TouchFling = false, TouchFlingKey = Enum.KeyCode.T,
	TouchFlingRange = 8,

	Spectating = false, SpectateTarget = "Nenhum",

	MENU_KEY = Enum.KeyCode.RightControl,

	ShowHUD = true, HUDX = 10, HUDY = 10, HUDEdit = false,
	Notifications = true, MenuAlpha = 0.1,

	MobileBtnSize = 56, MobileBtnAlpha = 0.15,
	ShowBtnAim = true, ShowBtnFly = true, ShowBtnNoclip = true,
	ShowBtnEsp = true, ShowBtnFling = true,
}

--------------------------------------------------------------------
-- BOOT
--------------------------------------------------------------------
local bootGui, bootLabel = nil, nil
local function bootShow(text, color, hideAfter)
	pcall(function()
		if not bootGui then
			bootGui = Instance.new("ScreenGui")
			bootGui.ResetOnSpawn = false
			bootGui.IgnoreGuiInset = true
			bootGui.DisplayOrder = 1000
			bootLabel = Instance.new("TextLabel")
			bootLabel.AnchorPoint = Vector2.new(0.5, 0)
			bootLabel.Position = UDim2.new(0.5, 0, 0, 8)
			bootLabel.Size = UDim2.new(0.9, 0, 0, 50)
			bootLabel.BackgroundColor3 = Color3.fromRGB(16, 16, 24)
			bootLabel.BackgroundTransparency = 0.15
			bootLabel.BorderSizePixel = 0
			bootLabel.Font = Enum.Font.GothamMedium
			bootLabel.TextSize = 14
			bootLabel.TextWrapped = true
			bootLabel.TextColor3 = Color3.new(1, 1, 1)
			bootLabel.Parent = bootGui
			bootGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
		end
		bootLabel.Text = tostring(text or "")
		bootLabel.TextColor3 = color or Color3.new(1, 1, 1)
		bootGui.Enabled = true
		if hideAfter then
			task.delay(hideAfter, function()
				if bootGui then bootGui.Enabled = false end
			end)
		end
	end)
end
bootShow("TestToolkit v85 carregando...")

--------------------------------------------------------------------
-- TEAM
--------------------------------------------------------------------
local function isTeammate(model, plr)
	if plr and plr.Team and LocalPlayer.Team and plr.Team == LocalPlayer.Team then
		return true
	end
	return false
end

--------------------------------------------------------------------
-- TARGETS
--------------------------------------------------------------------
local humanoids = {}
local rootOf = setmetatable({}, { __mode = "k" })
local entries = setmetatable({}, { __mode = "k" })

local function registerHumanoid(d)
	if d.ClassName == "Humanoid" then
		local m = d.Parent
		if m and m:IsA("Model") then humanoids[m] = d end
	end
end
Workspace.DescendantAdded:Connect(registerHumanoid)
Workspace.DescendantRemoving:Connect(function(d)
	if d.ClassName == "Humanoid" and humanoids[d.Parent] == d then
		humanoids[d.Parent] = nil
	end
end)
task.spawn(function()
	local n = 0
	for _, d in ipairs(Workspace:GetDescendants()) do
		registerHumanoid(d)
		n += 1
		if n % 300 == 0 then task.wait() end
	end
end)

local function getRoot(model)
	local r = rootOf[model]
	if r and r.Parent then return r end
	r = model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart
	rootOf[model] = r
	return r
end

local function getBestPart(model)
	return model:FindFirstChild("Head")
		or model:FindFirstChild("UpperTorso")
		or model:FindFirstChild("Torso")
		or model:FindFirstChild("HumanoidRootPart")
		or model.PrimaryPart
end

local targetCache = {}
local CACHE_TTL = 0.15

local function isAlive(hum)
	if not hum or hum.Health <= 0 then return false end
	local state = hum:GetState()
	if state == Enum.HumanoidStateType.Dead
		or state == Enum.HumanoidStateType.Physics
		or state == Enum.HumanoidStateType.Ragdoll then
		return false
	end
	return true
end

local function getTargets(purpose)
	local now = os.clock()
	local c = targetCache[purpose]
	if not c then c = { t = -1, list = {} }; targetCache[purpose] = c end
	if now - c.t < CACHE_TTL then return c.list end
	c.t = now
	local list = c.list
	table.clear(list)
	local n = 0
	local myChar = LocalPlayer.Character
	local mode = Settings.TargetMode
	for model, hum in pairs(humanoids) do
		if model ~= myChar and model.Parent and hum.Parent and isAlive(hum) then
			local plr = Players:GetPlayerFromCharacter(model)
			local modeOk
			if plr then modeOk = plr ~= LocalPlayer and mode ~= 2
			else modeOk = mode ~= 1 end
			if modeOk then
				local mate = isTeammate(model, plr)
				if purpose == "esp" or not mate or (not Settings.TeamCheck) then
					local root = getRoot(model)
					if root then
						local e = entries[model]
						if not e then e = {}; entries[model] = e end
						e.Model = model; e.Humanoid = hum; e.Root = root
						e.Player = plr; e.Teammate = mate
						n += 1
						list[n] = e
					end
				end
			end
		end
	end
	return list
end

_G.TT_getTargets = getTargets
_G.TT_getRoot = getRoot
_G.TT_humanoids = humanoids

--------------------------------------------------------------------
-- WALL CHECK
--------------------------------------------------------------------
local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
rayParams.IgnoreWater = true
local filterList = {}

local function isIgnorableBlocker(hit)
	if hit.Transparency >= 0.95 and not hit.CanCollide then return true end
	if hit:FindFirstAncestorOfClass("Accessory") then return true end
	return false
end

local function hasLineOfSight(part, model)
	if not Settings.AimWallCheck then return true end
	if not part or not part.Parent then return false end
	if not LocalPlayer.Character then return false end

	local origin = Camera.CFrame.Position

	table.clear(filterList)
	filterList[1] = LocalPlayer.Character
	filterList[2] = Camera
	rayParams.FilterDescendantsInstances = filterList

	local dir = part.Position - origin

	for _ = 1, 8 do
		local result = Workspace:Raycast(origin, dir, rayParams)
		if not result then return true end
		local hit = result.Instance
		if not hit then return true end

		if hit:IsDescendantOf(model) then return true end

		if isIgnorableBlocker(hit) then
			filterList[#filterList + 1] = hit
			rayParams.FilterDescendantsInstances = filterList
		else
			return false
		end
	end
	return false
end
_G.TT_hasLOS = hasLineOfSight

local function getAimOrigin()
	if isMobile then
		local vp = Camera.ViewportSize
		return Vector2.new(vp.X / 2, vp.Y / 2)
	end
	return UserInputService:GetMouseLocation()
end

--------------------------------------------------------------------
-- GUI / TEMA
--------------------------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Name = "TestToolkitUniversal"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 100
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
_G.TT_gui = gui

local Theme = {
	Bg = Color3.fromRGB(16, 16, 24),
	Panel = Color3.fromRGB(27, 27, 40),
	PanelHover = Color3.fromRGB(38, 38, 56),
	Accent = Color3.fromRGB(125, 95, 255),
	Accent2 = Color3.fromRGB(255, 95, 190),
	Off = Color3.fromRGB(60, 60, 78),
	Text = Color3.fromRGB(235, 235, 245),
	SubText = Color3.fromRGB(150, 150, 172),
	Good = Color3.fromRGB(80, 255, 130),
	Bad = Color3.fromRGB(255, 90, 90),
}
_G.TT_Theme = Theme

local function tween(obj, props, t, style, dir)
	local tw = TweenService:Create(obj, TweenInfo.new(t or 0.18, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out), props)
	tw:Play()
	return tw
end
local function corner(obj, r)
	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, r)
	c.Parent = obj
	return c
end
local function stroke(obj, color, transparency, thickness)
	local s = Instance.new("UIStroke")
	s.Color = color
	s.Transparency = transparency or 0.6
	s.Thickness = thickness or 1
	s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
	s.Parent = obj
	return s
end

local fovCircle = Instance.new("Frame")
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.BackgroundTransparency = 1
fovCircle.Parent = gui
corner(fovCircle, 9999)
stroke(fovCircle, Color3.new(1, 1, 1), 0.15, 1.5)

local silentFovCircle = Instance.new("Frame")
silentFovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
silentFovCircle.BackgroundTransparency = 1
silentFovCircle.ZIndex = 10
silentFovCircle.Parent = gui
corner(silentFovCircle, 9999)
stroke(silentFovCircle, Color3.fromRGB(80, 255, 200), 0, 2)

local triggerFovCircle = Instance.new("Frame")
triggerFovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
triggerFovCircle.BackgroundTransparency = 1
triggerFovCircle.Parent = gui
corner(triggerFovCircle, 9999)
stroke(triggerFovCircle, Color3.fromRGB(255, 100, 100), 0.4, 1.5)

local lockMarker = Instance.new("Frame")
lockMarker.AnchorPoint = Vector2.new(0.5, 0.5)
lockMarker.Size = UDim2.fromOffset(24, 24)
lockMarker.BackgroundTransparency = 1
lockMarker.Visible = false
lockMarker.Parent = gui
corner(lockMarker, 9999)
stroke(lockMarker, Theme.Bad, 0, 2)

-- Toasts
local toastHolder = Instance.new("Frame")
toastHolder.AnchorPoint = Vector2.new(1, 1)
toastHolder.Position = UDim2.new(1, -16, 1, -16)
toastHolder.Size = UDim2.fromOffset(280, 320)
toastHolder.BackgroundTransparency = 1
toastHolder.ZIndex = 60
toastHolder.Parent = gui
local toastLayout = Instance.new("UIListLayout")
toastLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
toastLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
toastLayout.SortOrder = Enum.SortOrder.LayoutOrder
toastLayout.Padding = UDim.new(0, 6)
toastLayout.Parent = toastHolder
local toastOrder = 0

local function notify(text, kind)
	if not Settings.Notifications then return end
	text = tostring(text or "")
	local count = 0
	for _, c in ipairs(toastHolder:GetChildren()) do
		if c:IsA("Frame") then count += 1 end
	end
	if count >= 5 then return end
	toastOrder += 1
	local color = kind == "on" and Theme.Good or kind == "off" and Theme.Bad or Theme.Accent
	local row = Instance.new("Frame")
	row.LayoutOrder = toastOrder
	row.Size = UDim2.new(1, 0, 0, 34)
	row.BackgroundTransparency = 1
	row.ZIndex = 60
	row.Parent = toastHolder
	local card = Instance.new("Frame")
	card.Size = UDim2.fromScale(1, 1)
	card.Position = UDim2.fromOffset(320, 0)
	card.BackgroundColor3 = Theme.Bg
	card.BackgroundTransparency = 0.08
	card.BorderSizePixel = 0
	card.ZIndex = 60
	card.Parent = row
	corner(card, 8)
	stroke(card, color, 0.55, 1)
	local bar = Instance.new("Frame")
	bar.Size = UDim2.new(0, 4, 1, -12)
	bar.Position = UDim2.fromOffset(7, 6)
	bar.BackgroundColor3 = color
	bar.BorderSizePixel = 0
	bar.ZIndex = 60
	bar.Parent = card
	corner(bar, 2)
	local lbl = Instance.new("TextLabel")
	lbl.BackgroundTransparency = 1
	lbl.Position = UDim2.fromOffset(20, 0)
	lbl.Size = UDim2.new(1, -28, 1, 0)
	lbl.Font = Enum.Font.GothamMedium
	lbl.TextSize = 13
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	lbl.TextColor3 = Theme.Text
	lbl.TextTruncate = Enum.TextTruncate.AtEnd
	lbl.Text = text
	lbl.ZIndex = 60
	lbl.Parent = card
	tween(card, { Position = UDim2.fromOffset(0, 0) }, 0.32, Enum.EasingStyle.Back)
	task.delay(2.4, function()
		if not card.Parent then return end
		tween(card, { Position = UDim2.fromOffset(320, 0) }, 0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
		task.wait(0.25)
		row:Destroy()
	end)
end
_G.TT_notify = notify

--------------------------------------------------------------------
-- LOCAL PLAYER
--------------------------------------------------------------------
local cChar, cHum, cRoot = nil, nil, nil

local function refreshLocal()
	local char = LocalPlayer.Character
	if char ~= cChar or (char and (not cHum or cHum.Parent ~= char or not cRoot or cRoot.Parent ~= char)) then
		cChar = char
		cHum = char and char:FindFirstChildOfClass("Humanoid")
		cRoot = char and char:FindFirstChild("HumanoidRootPart")
	end
end
local function getLocalHumanoid() refreshLocal(); return cHum end
local function getLocalRoot() refreshLocal(); return cRoot end

local myParts = {}
local myPartsConns = {}
local function trackMyCharacter(char)
	for _, c in ipairs(myPartsConns) do c:Disconnect() end
	table.clear(myPartsConns)
	table.clear(myParts)
	for _, d in ipairs(char:GetDescendants()) do
		if d:IsA("BasePart") then myParts[d] = true end
	end
	myPartsConns[1] = char.DescendantAdded:Connect(function(d)
		if d:IsA("BasePart") then myParts[d] = true end
	end)
	myPartsConns[2] = char.DescendantRemoving:Connect(function(d)
		myParts[d] = nil
	end)
end
if LocalPlayer.Character then trackMyCharacter(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(trackMyCharacter)

-- NOCLIP
local noclipTouched = setmetatable({}, { __mode = "k" })
local noclipWasOn = false
RunService.Stepped:Connect(function()
	if Settings.Noclip then
		noclipWasOn = true
		for part in pairs(myParts) do
			if part.CanCollide then
				part.CanCollide = false
				noclipTouched[part] = true
			end
		end
	elseif noclipWasOn then
		noclipWasOn = false
		for part in pairs(noclipTouched) do
			if part.Parent then part.CanCollide = true end
			noclipTouched[part] = nil
		end
	end
end)

--------------------------------------------------------------------
-- AIM STATE
--------------------------------------------------------------------
local aimActive = false
local silentTarget = nil
local silentActive = false
local firing = false
local aimDebugFound = 0
local aimDebugTarget = "nenhum"
local aimDebugMoved = false

UserInputService.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then firing = true end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then firing = false end
end)

local function isAiming() return Settings.AimEnabled and (aimActive or Settings.AimMode == 3) end
local function isLegit() return Settings.UseLegitAim end
local function isSilent() return Settings.UseSilentAim end

--------------------------------------------------------------------
-- SILENT AIM - FLICK INSTANTÂNEO
--------------------------------------------------------------------
local mousemoverelFn = nil
if type(mousemoverel) == "function" then
	mousemoverelFn = mousemoverel
end

local flickInProgress = false

local function performSilentFlick(targetPart)
	if not targetPart or not targetPart.Parent then return false end
	if not mousemoverelFn then return false end
	if flickInProgress then return false end
	flickInProgress = true

	local vp = Camera.ViewportSize
	local cx, cy = vp.X / 2, vp.Y / 2

	local sp, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
	if not onScreen then
		flickInProgress = false
		return false
	end

	local dx = sp.X - cx
	local dy = sp.Y - cy

	pcall(mousemoverelFn, dx, dy)
	task.wait()

	pcall(function()
		local mouse = LocalPlayer:GetMouse()
		if mouse then
			if mouse1click then mouse1click() end
			if mouse1down and mouse1up then
				mouse1down()
				task.wait(0.01)
				mouse1up()
			end
		end
	end)

	task.wait()

	pcall(mousemoverelFn, -dx, -dy)

	flickInProgress = false
	return true
end

local function updateSilentTarget()
	if not isSilent() or not isAiming() then
		silentActive = false
		silentTarget = nil
		return
	end
	local origin = getAimOrigin()
	local camPos = Camera.CFrame.Position
	local maxDist = Settings.SilentAimFOV
	local best, bestD = nil, math.huge
	local needWall = Settings.SilentAimVisible

	for _, t in ipairs(getTargets("aim")) do
		local part = getBestPart(t.Model)
		if part then
			local rsp, on = Camera:WorldToViewportPoint(part.Position)
			if on and rsp.Z > 0 then
				local d = (Vector2.new(rsp.X, rsp.Y) - origin).Magnitude
				if d <= maxDist then
					if not needWall or hasLineOfSight(part, t.Model) then
						local wd = (part.Position - camPos).Magnitude
						if wd < bestD then
							best, bestD = t, wd
							best.Part = part
						end
					end
				end
			end
		end
	end

	if best and best.Model.Parent and best.Part then
		silentTarget = best
		silentActive = true

		if firing then
			task.spawn(function()
				performSilentFlick(best.Part)
			end)
		end
		return
	end

	silentTarget = nil
	silentActive = false
end

--------------------------------------------------------------------
-- TRIGGER BOT
--------------------------------------------------------------------
local triggerBotActive = false
local lastTriggerShot = 0
local triggerTargetName = "nenhum"

local function getTriggerTarget()
	if not Settings.TriggerBot then return nil end
	if not isAiming() and not Settings.TriggerBotAlways then return nil end
	local origin = getAimOrigin()
	local maxFov = Settings.TriggerFOV
	local best, bestD = nil, math.huge
	local needWall = Settings.TriggerVisible

	for _, t in ipairs(getTargets("aim")) do
		local parts = {
			t.Model:FindFirstChild("Head"),
			t.Model:FindFirstChild("UpperTorso"),
			t.Model:FindFirstChild("Torso"),
			t.Root,
		}
		for _, part in ipairs(parts) do
			if part and part:IsA("BasePart") then
				local sp, on = Camera:WorldToViewportPoint(part.Position)
				if on and sp.Z > 0 then
					local d = (Vector2.new(sp.X, sp.Y) - origin).Magnitude
					if d <= maxFov and d < bestD then
						if not needWall or hasLineOfSight(part, t.Model) then
							best, bestD = t, d
							break
						end
					end
				end
			end
		end
	end
	return best
end

local function simulateClick()
	local mouse = LocalPlayer:GetMouse()
	if mouse then
		pcall(function()
			if mouse1click then mouse1click() return end
		end)
		pcall(function()
			if mouse1down and mouse1up then
				mouse1down()
				task.wait(0.01)
				mouse1up()
				return
			end
		end)
		pcall(function()
			VirtualInputManager:SendMouseButtonEvent(mouse.X, mouse.Y, 0, true, game, 1)
			task.wait(0.005)
			VirtualInputManager:SendMouseButtonEvent(mouse.X, mouse.Y, 0, false, game, 1)
		end)
	end
end

local function updateTriggerBot()
	if not Settings.TriggerBot then
		triggerBotActive = false
		triggerTargetName = "nenhum"
		return
	end
	local now = os.clock()
	local minDelay = math.max(Settings.TriggerDelay, 0.016)
	if now - lastTriggerShot < minDelay then return end
	local target = getTriggerTarget()
	if not target then
		triggerBotActive = false
		triggerTargetName = "nenhum"
		return
	end
	triggerBotActive = true
	triggerTargetName = target.Player and target.Player.DisplayName or target.Model.Name
	lastTriggerShot = now
	simulateClick()
end

RunService.RenderStepped:Connect(function()
	pcall(updateSilentTarget)
	pcall(updateTriggerBot)
end)

--------------------------------------------------------------------
-- LEGIT AIM
--------------------------------------------------------------------
RunService.RenderStepped:Connect(function(dt)
	if not (isLegit() and isAiming()) then
		aimDebugFound = 0
		aimDebugTarget = "nenhum"
		aimDebugMoved = false
		fovCircle.Visible = false
		lockMarker.Visible = false
		return
	end

	local aimOrigin
	if isMobile then
		local vp = Camera.ViewportSize
		aimOrigin = Vector2.new(vp.X / 2, vp.Y / 2)
	else
		aimOrigin = UserInputService:GetMouseLocation()
	end

	if Settings.FOVEnabled then
		fovCircle.Visible = true
		fovCircle.Position = UDim2.fromOffset(aimOrigin.X, aimOrigin.Y)
		fovCircle.Size = UDim2.fromOffset(Settings.FOVRadius * 2, Settings.FOVRadius * 2)
	else
		fovCircle.Visible = false
	end

	local fovLimit = Settings.FOVEnabled and Settings.FOVRadius or math.huge
	local best, bestDist = nil, fovLimit
	local found = 0
	local needWall = Settings.AimWallCheck

	for _, t in ipairs(getTargets("aim")) do
		local part = getBestPart(t.Model)
		if part then
			local sp, onScreen = Camera:WorldToViewportPoint(part.Position)
			if onScreen and sp.Z > 0 then
				local d = (Vector2.new(sp.X, sp.Y) - aimOrigin).Magnitude
				if d <= bestDist then
					if not needWall or hasLineOfSight(part, t.Model) then
						found = found + 1
						best, bestDist = t, d
						best.Part = part
					end
				end
			end
		end
	end

	aimDebugFound = found
	aimDebugTarget = best and (best.Player and best.Player.DisplayName or best.Model.Name) or "nenhum"

	if not best or not best.Part then
		aimDebugMoved = false
		lockMarker.Visible = false
		return
	end

	local sp, onScreen = Camera:WorldToViewportPoint(best.Part.Position)
	if not onScreen then
		aimDebugMoved = false
		lockMarker.Visible = false
		return
	end

	lockMarker.Visible = true
	lockMarker.Position = UDim2.fromOffset(sp.X, sp.Y)
	lockMarker.BackgroundColor3 = Color3.fromRGB(80, 255, 130)
	lockMarker.Size = UDim2.fromOffset(20, 20)

	if isMobile then return end

	local vp = Camera.ViewportSize
	local cx, cy = vp.X / 2, vp.Y / 2
	local dx = sp.X - cx
	local dy = sp.Y - cy

	local distToCenter = math.sqrt(dx * dx + dy * dy)
	if distToCenter > Settings.FOVRadius * 1.5 then
		aimDebugMoved = false
		return
	end

	local smooth = math.clamp(Settings.Smoothness, 0.01, 0.95)
	local factor = 1 - smooth
	if firing and Settings.FireLock then factor = 1 end

	local moveX = dx * factor
	local moveY = dy * factor

	local maxMove = 30
	if math.abs(moveX) > maxMove then moveX = maxMove * (moveX > 0 and 1 or -1) end
	if math.abs(moveY) > maxMove then moveY = maxMove * (moveY > 0 and 1 or -1) end

	local moved = false
	if mousemoverelFn then
		local ok = pcall(mousemoverelFn, moveX, moveY)
		if ok then moved = true end
	end

	aimDebugMoved = moved
end)

-- FOV circles
RunService.RenderStepped:Connect(function()
	local showSilentFov = Settings.AimEnabled and Settings.UseSilentAim and isAiming()
	silentFovCircle.Visible = showSilentFov
	if showSilentFov then
		local o = getAimOrigin()
		silentFovCircle.Position = UDim2.fromOffset(o.X, o.Y)
		silentFovCircle.Size = UDim2.fromOffset(Settings.SilentAimFOV * 2, Settings.SilentAimFOV * 2)
		silentFovCircle.ZIndex = 10
	end
	local showTriggerFov = Settings.TriggerBot
	triggerFovCircle.Visible = showTriggerFov
	if showTriggerFov then
		local o = getAimOrigin()
		triggerFovCircle.Position = UDim2.fromOffset(o.X, o.Y)
		triggerFovCircle.Size = UDim2.fromOffset(math.max(Settings.TriggerFOV * 2, 4), math.max(Settings.TriggerFOV * 2, 4))
	end
end)

--------------------------------------------------------------------
-- HITBOX
--------------------------------------------------------------------
local HitboxExpander = {}
do
	local savedProps = setmetatable({}, { __mode = "k" })

	local function savePart(part)
		if savedProps[part] then return end
		savedProps[part] = {
			Size = part.Size,
			CanCollide = part.CanCollide,
			Massless = part.Massless,
			Transparency = part.Transparency,
		}
	end

	local function restorePart(part)
		local props = savedProps[part]
		if not props then return end
		pcall(function()
			part.Size = props.Size
			part.CanCollide = props.CanCollide
			part.Massless = props.Massless
			part.Transparency = props.Transparency
		end)
		savedProps[part] = nil
	end

	local PART_KEYS = {
		{ "Head", "ExpandHead" },
		{ "UpperTorso", "ExpandUpperTorso" },
		{ "Torso", "ExpandTorso" },
		{ "LowerTorso", "ExpandLowerTorso" },
	}

	local function expandPlayer(player)
		local char = player.Character
		if not char then return end
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hum or hum.Health <= 0 then return end
		for _, entry in ipairs(PART_KEYS) do
			if Settings[entry[2]] then
				local part = char:FindFirstChild(entry[1])
				if part and part:IsA("BasePart") then
					savePart(part)
					local size = Settings.HitboxSize
					pcall(function()
						part.Size = Vector3.new(size, size, size)
						part.CanCollide = false
						part.Massless = true
						if Settings.HitboxInvisible then
							part.Transparency = 1
						else
							part.Transparency = 0.5
						end
					end)
				end
			end
		end
	end

	local function restorePlayer(player)
		local char = player.Character
		if not char then return end
		for _, entry in ipairs(PART_KEYS) do
			local part = char:FindFirstChild(entry[1])
			if part and part:IsA("BasePart") then restorePart(part) end
		end
	end

	local function shouldExpand(player)
		local char = player.Character
		if not char then return false end
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hum or hum.Health <= 0 then return false end
		local myRoot = getLocalRoot()
		local theirRoot = getRoot(char)
		if not myRoot or not theirRoot then return false end
		local dist = (theirRoot.Position - myRoot.Position).Magnitude
		if dist > math.max(10, Settings.HitboxRange or 200) then return false end
		return true
	end

	local lastApply = 0
	local function step()
		if not Settings.HitboxExpander then
			for _, plr in ipairs(Players:GetPlayers()) do
				if plr ~= LocalPlayer then restorePlayer(plr) end
			end
			return
		end
		local now = os.clock()
		if now - lastApply < 0.1 then return end
		lastApply = now
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer then
				if shouldExpand(plr) then expandPlayer(plr) else restorePlayer(plr) end
			end
		end
	end

	HitboxExpander.step = step
	HitboxExpander.restoreAll = function()
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer then restorePlayer(plr) end
		end
	end
end

task.spawn(function()
	while task.wait(0.1) do pcall(HitboxExpander.step) end
end)

--------------------------------------------------------------------
-- FLY
--------------------------------------------------------------------
local flyBV = nil
local flyBG = nil
local flyPlat = false
local flyCur = 0
local FLY_RAMP = 500

local function destroyFly()
	flyCur = 0
	if flyBV then pcall(function() flyBV:Destroy() end); flyBV = nil end
	if flyBG then pcall(function() flyBG:Destroy() end); flyBG = nil end
	if flyPlat then
		local hum = getLocalHumanoid()
		if hum then
			hum.PlatformStand = false
			local root = getLocalRoot()
			if root then
				root.AssemblyLinearVelocity = Vector3.zero
				root.AssemblyAngularVelocity = Vector3.zero
			end
		end
		flyPlat = false
	end
end

local function ensureFly(root)
	if flyBV and flyBV.Parent == root then return end
	destroyFly()
	local bv = Instance.new("BodyVelocity")
	bv.MaxForce = Vector3.one * math.huge
	bv.P = 1250
	bv.Velocity = Vector3.zero
	bv.Parent = root
	flyBV = bv
	local bg = Instance.new("BodyGyro")
	bg.MaxTorque = Vector3.one * math.huge
	bg.P = 10000
	bg.D = 500
	bg.CFrame = root.CFrame
	bg.Parent = root
	flyBG = bg
end

local function updateFly(hum, root, dt)
	ensureFly(root)
	if not flyBV then return end
	if not hum.PlatformStand then hum.PlatformStand = true end
	flyPlat = true
	if flyBG then
		local look = Camera.CFrame.LookVector
		local flatLook = Vector3.new(look.X, 0, look.Z)
		if flatLook.Magnitude < 0.01 then flatLook = Vector3.new(0, 0, -1) end
		flyBG.CFrame = CFrame.lookAt(root.Position, root.Position + flatLook.Unit)
	end
	local cam = Camera
	local look = cam.CFrame.LookVector
	local flatLook = Vector3.new(look.X, 0, look.Z)
	if flatLook.Magnitude < 0.01 then flatLook = Vector3.new(0, 0, -1) end
	flatLook = flatLook.Unit
	local flatRight = Vector3.new(cam.CFrame.RightVector.X, 0, cam.CFrame.RightVector.Z)
	if flatRight.Magnitude > 0.01 then flatRight = flatRight.Unit end
	local move = hum.MoveDirection
	local fwd = move:Dot(flatLook)
	local side = move:Dot(flatRight)
	local dir = cam.CFrame.LookVector * fwd + cam.CFrame.RightVector * side
	local typing = UserInputService:GetFocusedTextBox() ~= nil
	local up = 0
	if not typing then
		if UserInputService:IsKeyDown(Enum.KeyCode.Space) or hum.Jump then up += 1 end
		if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then up -= 1 end
	end
	dir += Vector3.new(0, up, 0)
	if dir.Magnitude > 1 then dir = dir.Unit end
	local goal = Settings.FlySpeed
	if flyCur < goal then
		flyCur = math.min(goal, flyCur + math.max(FLY_RAMP, goal * 1.5) * dt)
	else
		flyCur = goal
	end
	flyBV.Velocity = dir * flyCur
end

--------------------------------------------------------------------
-- ANTI FLING / VOID
--------------------------------------------------------------------
RunService.Heartbeat:Connect(function()
	pcall(function()
		if not Settings.AntiFling then return end
		local hum = getLocalHumanoid()
		local root = getLocalRoot()
		if not hum or not root or hum.Health <= 0 then return end
		local v = root.AssemblyLinearVelocity
		local horizontal = Vector3.new(v.X, 0, v.Z).Magnitude
		if horizontal > Settings.AntiFlingSpeed or v.Y > 80 or root.AssemblyAngularVelocity.Magnitude > 40 then
			root.AssemblyLinearVelocity = Vector3.zero
			root.AssemblyAngularVelocity = Vector3.zero
			local state = hum:GetState()
			if state == Enum.HumanoidStateType.FallingDown
				or state == Enum.HumanoidStateType.Ragdoll
				or state == Enum.HumanoidStateType.Physics then
				hum:ChangeState(Enum.HumanoidStateType.GettingUp)
			end
			hum.PlatformStand = false
		end
	end)
end)

local lastSafe = nil
RunService.Heartbeat:Connect(function()
	pcall(function()
		local hum = getLocalHumanoid()
		local root = getLocalRoot()
		if not hum or not root or hum.Health <= 0 then return end
		if root.Position.Y < Settings.AntiVoidY then
			if Settings.AntiVoid then
				local target = lastSafe
				if not target then
					local spawn = Workspace:FindFirstChildWhichIsA("SpawnLocation", true)
					target = spawn and (spawn.CFrame + Vector3.new(0, 5, 0)) or CFrame.new(0, 50, 0)
				end
				LocalPlayer.Character:PivotTo(target)
				root.AssemblyLinearVelocity = Vector3.zero
				root.AssemblyAngularVelocity = Vector3.zero
				notify("Anti Void: trazido de volta", "info")
			end
		elseif hum.FloorMaterial ~= Enum.Material.Air then
			lastSafe = root.CFrame + Vector3.new(0, 3, 0)
		end
	end)
end)

--------------------------------------------------------------------
-- FLING K1LAS1K (9e9 + BodyVelocity P=20000 + 60 iterações)
--------------------------------------------------------------------
local Fling = {}
do
	local lastFling = 0

	-- Fling bot (NPCs próximos)
	local function doFling(targetRoot, myRoot, power)
		if not targetRoot or not targetRoot.Parent then return end
		if not myRoot or not myRoot.Parent then return end
		local hum = targetRoot.Parent:FindFirstChildOfClass("Humanoid")
		if hum and hum.Health <= 0 then return end
		local bv = Instance.new("BodyVelocity")
		bv.MaxForce = Vector3.one * math.huge
		bv.P = 20000
		bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
		bv.Parent = targetRoot
		local bav = Instance.new("BodyAngularVelocity")
		bav.MaxTorque = Vector3.one * math.huge
		bav.P = 20000
		bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
		bav.Parent = targetRoot
		local endTime = os.clock() + 0.6
		while os.clock() < endTime do
			if not targetRoot.Parent then break end
			RunService.Heartbeat:Wait()
		end
		if bv.Parent then bv:Destroy() end
		if bav.Parent then bav:Destroy() end
	end

	-- FLING PLAYER K1LAS1K EXATO
	local function doFlingPlayer(target)
		if not target or target == LocalPlayer then return false end

		local Character = LocalPlayer.Character
		if not Character then return false end
		local Humanoid = Character:FindFirstChildOfClass("Humanoid")
		if not Humanoid then return false end
		local RootPart = Character:FindFirstChild("HumanoidRootPart")
		if not RootPart then return false end

		local TCharacter = target.Character
		if not TCharacter then return false end
		local THumanoid = TCharacter:FindFirstChildOfClass("Humanoid")
		if not THumanoid then return false end
		local TRootPart = TCharacter:FindFirstChild("HumanoidRootPart")
		if not TRootPart then return false end

		if THumanoid.Health <= 0 then return false end

		local SavedCameraSubject = Camera.CameraSubject
		local SavedCameraType = Camera.CameraType

		pcall(function()
			Camera.CameraSubject = THumanoid
			Camera.CameraType = Enum.CameraType.Custom
		end)

		task.wait()

		-- BodyVelocity + BodyAngularVelocity no alvo (P = 20000)
		local bv = Instance.new("BodyVelocity")
		bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
		bv.P = 20000
		bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
		bv.Parent = TRootPart

		local bav = Instance.new("BodyAngularVelocity")
		bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
		bav.P = 20000
		bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
		bav.Parent = TRootPart

		-- 60 iterações
		for i = 1, 60 do
			if not Character or not Character.Parent then break end
			if not RootPart or not RootPart.Parent then break end
			if not TCharacter or not TCharacter.Parent then break end
			if not TRootPart or not TRootPart.Parent then break end
			if Humanoid.Health <= 0 then break end
			if THumanoid.Health <= 0 then break end

			RootPart.CFrame = TRootPart.CFrame

			RootPart.Velocity = Vector3.new(9e9, 9e9, 9e9)
			RootPart.RotVelocity = Vector3.new(9e9, 9e9, 9e9)

			TRootPart.Velocity = Vector3.new(9e9, 9e9, 9e9)
			TRootPart.RotVelocity = Vector3.new(9e9, 9e9, 9e9)

			pcall(function()
				bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
				bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
			end)

			RunService.Heartbeat:Wait()
		end

		pcall(function()
			if bv.Parent then bv:Destroy() end
			if bav.Parent then bav:Destroy() end
		end)

		pcall(function()
			Camera.CameraSubject = SavedCameraSubject or Humanoid
			Camera.CameraType = SavedCameraType or Enum.CameraType.Custom
		end)

		return true
	end

	Fling.doFlingPlayer = doFlingPlayer

	Fling.flingSelected = function()
		local name = Settings.FlingPlayerTarget
		if not name or name == "" or name == "Nenhum" then
			notify("Fling: nenhum player selecionado", "off")
			return
		end
		local target = nil
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr.Name == name or plr.DisplayName == name then
				target = plr
				break
			end
		end
		if not target or target == LocalPlayer then
			notify("Fling: player inválido", "off")
			return
		end
		task.spawn(function()
			local ok = doFlingPlayer(target)
			notify(ok and ("Fling: " .. tostring(target.DisplayName)) or "Fling: falhou", ok and "on" or "off")
		end)
	end

	Fling.step = function()
		if not Settings.Fling then return end
		local now = os.clock()
		if now - lastFling < Settings.FlingRepeat then return end
		lastFling = now
		local myHum, myRoot = getLocalHumanoid(), getLocalRoot()
		if not myHum or myHum.Health <= 0 then return end
		task.spawn(function()
			local myPos = myRoot.Position
			local range = Settings.FlingRange
			for model, hum in pairs(humanoids) do
				if model ~= LocalPlayer.Character and model.Parent and hum.Parent and hum.Health > 0 then
					local plr = Players:GetPlayerFromCharacter(model)
					if not plr then
						local r = getRoot(model)
						if r and (r.Position - myPos).Magnitude <= range then
							doFling(r, myRoot, Settings.FlingPower)
						end
					end
				end
			end
		end)
	end
end

--------------------------------------------------------------------
-- TOUCH FLING (K1LAS1K)
--------------------------------------------------------------------
local TouchFling = {}
do
	local cooldown = {}

	local function canFling(model)
		if not model or not model.Parent then return false end
		if not model:IsA("Model") then return false end
		local hum = model:FindFirstChildOfClass("Humanoid")
		if not hum or hum.Health <= 0 then return false end
		if model == LocalPlayer.Character then return false end
		return true
	end

	local function flingTarget(targetRoot, myRoot)
		if not targetRoot or not targetRoot.Parent then return false end
		if not myRoot or not myRoot.Parent then return false end

		local TCharacter = targetRoot.Parent
		local THumanoid = TCharacter:FindFirstChildOfClass("Humanoid")
		if not THumanoid or THumanoid.Health <= 0 then return false end

		local now = os.clock()
		if cooldown[targetRoot] and now - cooldown[targetRoot] < 0.8 then
			return false
		end
		cooldown[targetRoot] = now

		local Character = LocalPlayer.Character
		if not Character then return false end
		local Humanoid = Character:FindFirstChildOfClass("Humanoid")
		if not Humanoid then return false end
		local RootPart = Character:FindFirstChild("HumanoidRootPart")
		if not RootPart then return false end

		local SavedCameraSubject = Camera.CameraSubject
		local SavedCameraType = Camera.CameraType

		pcall(function()
			Camera.CameraSubject = THumanoid
			Camera.CameraType = Enum.CameraType.Custom
		end)

		task.spawn(function()
			task.wait()

			local bv = Instance.new("BodyVelocity")
			bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
			bv.P = 20000
			bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
			bv.Parent = targetRoot

			local bav = Instance.new("BodyAngularVelocity")
			bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
			bav.P = 20000
			bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
			bav.Parent = targetRoot

			for i = 1, 60 do
				if not Character or not Character.Parent then break end
				if not RootPart or not RootPart.Parent then break end
				if not TCharacter or not TCharacter.Parent then break end
				if not targetRoot or not targetRoot.Parent then break end
				if Humanoid.Health <= 0 then break end
				if THumanoid.Health <= 0 then break end

				RootPart.CFrame = targetRoot.CFrame
				RootPart.Velocity = Vector3.new(9e9, 9e9, 9e9)
				RootPart.RotVelocity = Vector3.new(9e9, 9e9, 9e9)

				targetRoot.Velocity = Vector3.new(9e9, 9e9, 9e9)
				targetRoot.RotVelocity = Vector3.new(9e9, 9e9, 9e9)

				pcall(function()
					bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
					bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
				end)

				RunService.Heartbeat:Wait()
			end

			pcall(function()
				if bv.Parent then bv:Destroy() end
				if bav.Parent then bav:Destroy() end
			end)

			pcall(function()
				Camera.CameraSubject = SavedCameraSubject or Humanoid
				Camera.CameraType = SavedCameraType or Enum.CameraType.Custom
			end)
		end)

		return true
	end

	TouchFling.step = function()
		if not Settings.TouchFling then return end
		local myRoot = getLocalRoot()
		if not myRoot or not myRoot.Parent then return end

		local myPos = myRoot.Position
		local range = Settings.TouchFlingRange

		for model, hum in pairs(humanoids) do
			if canFling(model) and hum.Health > 0 then
				local theirRoot = getRoot(model)
				if theirRoot and theirRoot.Parent then
					local dist = (theirRoot.Position - myPos).Magnitude
					if dist <= range then
						flingTarget(theirRoot, myRoot)
					end
				end
			end
		end
	end

	TouchFling.flingModel = function(model)
		local myRoot = getLocalRoot()
		if not myRoot then return end
		local theirRoot = getRoot(model)
		if not theirRoot then return end
		flingTarget(theirRoot, myRoot)
	end
end

_G.TT_TouchFling = TouchFling

--------------------------------------------------------------------
-- SPECTATE PLAYER
--------------------------------------------------------------------
local Spectate = {}
do
	local originalSubject = nil
	local originalType = nil
	local currentTarget = nil
	local savedState = false

	Spectate.start = function(plr)
		if not plr or plr == LocalPlayer then return false end

		if not savedState then
			originalSubject = Camera.CameraSubject
			originalType = Camera.CameraType
			savedState = true
		end

		local char = plr.Character
		if not char then return false end
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hum then return false end

		Camera.CameraType = Enum.CameraType.Custom
		Camera.CameraSubject = hum
		currentTarget = plr
		Settings.Spectating = true
		Settings.SpectateTarget = plr.DisplayName or plr.Name
		return true
	end

	Spectate.stop = function()
		if not savedState then return end
		Camera.CameraType = originalType or Enum.CameraType.Custom
		Camera.CameraSubject = originalSubject
		currentTarget = nil
		savedState = false
		Settings.Spectating = false
		Settings.SpectateTarget = "Nenhum"
	end

	Spectate.isActive = function()
		return currentTarget ~= nil and currentTarget.Character ~= nil
	end

	Spectate.getTarget = function()
		return currentTarget
	end

	task.spawn(function()
		while task.wait(0.3) do
			if Settings.Spectating and currentTarget then
				if not currentTarget.Parent then
					Spectate.stop()
					notify("Spectate: alvo saiu do jogo", "off")
					if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
				else
					local char = currentTarget.Character
					local hum = char and char:FindFirstChildOfClass("Humanoid")
					if char and hum and hum.Health > 0 then
						if Camera.CameraSubject ~= hum then
							Camera.CameraSubject = hum
							Camera.CameraType = Enum.CameraType.Custom
						end
					end
				end
			end
		end
	end)
end

_G.TT_Spectate = Spectate

--------------------------------------------------------------------
-- HEARTBEAT (movimento + touchfling)
--------------------------------------------------------------------
local defaultWalkSpeed = 16
local defaultJumpPower, defaultUseJumpPower = 50, true
local wsActive, jpActive = false, false

LocalPlayer.CharacterAdded:Connect(function()
	wsActive, jpActive = false, false
	lastSafe = nil
	if Settings.SeatInvisible then
		Settings.SeatInvisible = false
		if _G.TT_SeatInvisible then pcall(_G.TT_SeatInvisible.toggle) end
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	end
end)

RunService.Heartbeat:Connect(function(dt)
	pcall(function()
		local hum = getLocalHumanoid()
		if not hum then return end
		local root = getLocalRoot()

		if Settings.WalkSpeedOn then
			if not wsActive then
				defaultWalkSpeed = hum.WalkSpeed
				wsActive = true
			end
			if hum.WalkSpeed ~= Settings.WalkSpeed then hum.WalkSpeed = Settings.WalkSpeed end
		elseif wsActive then
			hum.WalkSpeed = defaultWalkSpeed
			wsActive = false
		end

		if Settings.JumpOn then
			if not jpActive then
				defaultUseJumpPower = hum.UseJumpPower
				defaultJumpPower = hum.JumpPower
				jpActive = true
			end
			if not hum.UseJumpPower then hum.UseJumpPower = true end
			if hum.JumpPower ~= Settings.JumpPower then hum.JumpPower = Settings.JumpPower end
		elseif jpActive then
			hum.UseJumpPower = defaultUseJumpPower
			hum.JumpPower = defaultJumpPower
			jpActive = false
		end

		if Settings.Fly and root and hum.Health > 0 then
			updateFly(hum, root, dt)
		elseif flyBV or flyPlat then
			destroyFly()
		end

		Fling.step()
		TouchFling.step()
	end)
end)

--------------------------------------------------------------------
-- EXTRAS
--------------------------------------------------------------------
UserInputService.JumpRequest:Connect(function()
	if Settings.InfJump and not Settings.Fly then
		local hum = getLocalHumanoid()
		if hum and hum.Health > 0 then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
	end
end)

LocalPlayer.Idled:Connect(function()
	if not Settings.AntiAFK then return end
	pcall(function()
		VirtualUser:CaptureController()
		VirtualUser:ClickButton2(Vector2.new())
	end)
end)

local Lit = Lighting
local litSaved = {}
local litAcc = 0

local function litApply()
	if Settings.FullBright then
		if not litSaved.FullBright then
			litSaved.FullBright = { Brightness = Lit.Brightness, ClockTime = Lit.ClockTime, GlobalShadows = Lit.GlobalShadows }
		end
		Lit.Brightness = 2
		Lit.ClockTime = 14
		Lit.GlobalShadows = false
	elseif litSaved.FullBright then
		for p, v in pairs(litSaved.FullBright) do pcall(function() Lit[p] = v end) end
		litSaved.FullBright = nil
	end
	if Settings.NoFog then
		if not litSaved.NoFog then
			litSaved.NoFog = { FogStart = Lit.FogStart, FogEnd = Lit.FogEnd }
		end
		Lit.FogStart = 1e9
		Lit.FogEnd = 1e9
	elseif litSaved.NoFog then
		for p, v in pairs(litSaved.NoFog) do pcall(function() Lit[p] = v end) end
		litSaved.NoFog = nil
	end
end

RunService.Heartbeat:Connect(function(dt)
	pcall(function()
		litAcc += dt
		if litAcc < 0.5 then return end
		litAcc = 0
		if Settings.FullBright or Settings.NoFog or litSaved.FullBright or litSaved.NoFog then
			litApply()
		end
	end)
end)

local camFovDefault = nil
RunService.RenderStepped:Connect(function()
	pcall(function()
		if Settings.CamFOVOn then
			if not camFovDefault then camFovDefault = Camera.FieldOfView end
			if Camera.FieldOfView ~= Settings.CamFOV then Camera.FieldOfView = Settings.CamFOV end
		elseif camFovDefault then
			Camera.FieldOfView = camFovDefault
			camFovDefault = nil
		end
	end)
end)

--------------------------------------------------------------------
-- ESP (somente Highlight + nome + hp + tracer)
--------------------------------------------------------------------
local espRoot = Instance.new("Frame")
espRoot.Name = "ESP"
espRoot.Size = UDim2.fromScale(1, 1)
espRoot.BackgroundTransparency = 1
espRoot.ZIndex = 1
espRoot.Parent = gui

local ALLY_COLOR = Color3.fromRGB(80, 190, 255)
local espObjs = {}
local espSel = {}

local function mkFrame(parent, round)
	local f = Instance.new("Frame")
	f.BorderSizePixel = 0
	f.Visible = false
	f.ZIndex = 1
	f.Parent = parent
	if round then corner(f, 9999) end
	return f
end

local function mkText(parent, size)
	local l = Instance.new("TextLabel")
	l.BackgroundTransparency = 1
	l.Font = Enum.Font.GothamBold
	l.TextSize = size
	l.TextColor3 = Color3.new(1, 1, 1)
	l.TextStrokeTransparency = 0.35
	l.Size = UDim2.fromOffset(220, 14)
	l.Visible = false
	l.ZIndex = 5
	l.Parent = parent
	return l
end

local function newEsp(model)
	local o = { Model = model, Seen = 0, H = 5.5, HAt = -1 }
	local hl = Instance.new("Highlight")
	hl.Adornee = model
	hl.Enabled = false
	hl.FillTransparency = 0.65
	hl.OutlineTransparency = 0
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Parent = gui
	o.HL = hl

	local holder = Instance.new("Frame")
	holder.BackgroundTransparency = 1
	holder.Size = UDim2.fromScale(1, 1)
	holder.Visible = false
	holder.ZIndex = 1
	holder.Parent = espRoot
	o.Holder = holder

	o.HpBg = mkFrame(holder)
	o.HpBg.BackgroundColor3 = Color3.new(0, 0, 0)
	o.HpBg.BackgroundTransparency = 0.4
	o.HpFill = Instance.new("Frame")
	o.HpFill.AnchorPoint = Vector2.new(0, 1)
	o.HpFill.Position = UDim2.fromScale(0, 1)
	o.HpFill.BorderSizePixel = 0
	o.HpFill.ZIndex = 1
	o.HpFill.Parent = o.HpBg
	o.Name = mkText(holder, 13)
	o.Name.AnchorPoint = Vector2.new(0.5, 1)
	o.Tracer = mkFrame(holder)
	return o
end

local function hideEsp(o)
	o.Holder.Visible = false
	o.HL.Enabled = false
end

local function destroyEsp(model, o)
	pcall(function() o.HL:Destroy() end)
	pcall(function() o.Holder:Destroy() end)
	espObjs[model] = nil
end

local function px(v) return math.floor(v + 0.5) end

local function drawEsp(o, t, dist, now, vp)
	local model = t.Model
	local color = t.Teammate and ALLY_COLOR
		or (Settings.Rainbow and Color3.fromHSV((now * 0.25) % 1, 0.85, 1) or Settings.ESPColor)

	if Settings.ESPHighlight then
		local hl = o.HL
		hl.Enabled = true
		hl.FillColor = color
		hl.OutlineColor = color
		hl.FillTransparency = Settings.ESPFillTrans
		hl.OutlineTransparency = 0
		hl.DepthMode = Settings.Wallhack and Enum.HighlightDepthMode.AlwaysOnTop
			or Enum.HighlightDepthMode.Occluded
	else
		o.HL.Enabled = false
	end

	local rootPos = t.Root.Position
	if now - o.HAt > 0.5 then
		o.HAt = now
		local ok, sz = pcall(function() return model:GetExtentsSize() end)
		if ok and sz.Y > 1 then o.H = sz.Y end
	end
	local h = o.H
	local center = Camera:WorldToViewportPoint(rootPos)
	if center.Z <= 0 then
		o.Holder.Visible = false
		return
	end
	local top = Camera:WorldToViewportPoint(rootPos + Vector3.new(0, h * 0.45, 0))
	local bot = Camera:WorldToViewportPoint(rootPos - Vector3.new(0, h * 0.55, 0))
	local boxH = math.max(bot.Y - top.Y, 10)
	local x, y = px(center.X), px(top.Y)

	o.Holder.Visible = true

	if Settings.ShowHealth then
		local hum = t.Humanoid
		local frac = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
		o.HpBg.Visible = true
		o.HpBg.Position = UDim2.fromOffset(x + 10, y)
		o.HpBg.Size = UDim2.fromOffset(4, boxH)
		o.HpFill.Size = UDim2.new(1, 0, frac, 0)
		o.HpFill.BackgroundColor3 = Color3.fromHSV(frac * 0.33, 0.9, 1)
	else
		o.HpBg.Visible = false
	end

	if Settings.ShowNames then
		local plr = t.Player
		o.Name.Visible = true
		local displayName = (plr and plr.DisplayName) or model.Name or "?"
		o.Name.Text = tostring(displayName) .. "  [" .. tostring(math.floor(dist or 0)) .. "m]"
		o.Name.TextColor3 = color
		o.Name.Position = UDim2.fromOffset(x, y - 2)
	else
		o.Name.Visible = false
	end

	if Settings.Tracers and center.Z > 0 then
		local p1
		local origin = Settings.TracerOrigin or 1
		if origin == 1 then p1 = Vector2.new(vp.X / 2, vp.Y)
		elseif origin == 2 then p1 = vp / 2
		else p1 = getAimOrigin() end
		local p2 = Vector2.new(center.X, center.Y)
		local d = p2 - p1
		local len = d.Magnitude
		local maxLen = math.max(vp.X, vp.Y)
		if len > 2 and len < maxLen then
			o.Tracer.Visible = true
			o.Tracer.BackgroundColor3 = color
			o.Tracer.Size = UDim2.fromOffset(len, 1.5)
			o.Tracer.Position = UDim2.fromOffset((p1.X + p2.X) / 2, (p1.Y + p2.Y) / 2)
			o.Tracer.Rotation = math.deg(math.atan2(d.Y, d.X))
		else
			o.Tracer.Visible = false
		end
	else
		o.Tracer.Visible = false
	end
end

local espPool = {}
local espSelAt = -1
local espWasOn = false
local function espLess(a, b) return a.D < b.D end

local function updateEsp(now)
	if not Settings.ESP then
		if espWasOn then
			espWasOn = false
			for _, o in pairs(espObjs) do hideEsp(o) end
		end
		return
	end
	espWasOn = true
	local camPos = Camera.CFrame.Position
	if now - espSelAt >= 0.1 then
		espSelAt = now
		local maxD = Settings.ESPMaxDist
		table.clear(espSel)
		local n = 0
		for _, t in ipairs(getTargets("esp")) do
			local d = (t.Root.Position - camPos).Magnitude
			if maxD == 0 or d <= maxD then
				n += 1
				local e = espPool[n]
				if not e then e = {}; espPool[n] = e end
				e.T, e.D = t, d
				espSel[n] = e
			end
		end
		table.sort(espSel, espLess)
	end
	local vp = Camera.ViewportSize
	for i = 1, math.min(#espSel, Settings.ESPMaxTargets) do
		local t = espSel[i].T
		if t.Model.Parent and t.Root.Parent then
			local o = espObjs[t.Model]
			if not o then
				o = newEsp(t.Model)
				espObjs[t.Model] = o
			end
			o.Seen = now
			drawEsp(o, t, (t.Root.Position - camPos).Magnitude, now, vp)
		end
	end
	for model, o in pairs(espObjs) do
		if o.Seen ~= now then
			hideEsp(o)
			if now - o.Seen > 3 or not model.Parent then destroyEsp(model, o) end
		end
	end
end

RunService.RenderStepped:Connect(function()
	pcall(function() updateEsp(os.clock()) end)
end)

--------------------------------------------------------------------
-- SEAT INVISIBILITY
--------------------------------------------------------------------
local SeatInvisible = {}
do
	local mySeat = nil
	local active = false

	local function cleanupSeat()
		local e = workspace:FindFirstChild("invischair")
		if e then pcall(function() e:Destroy() end) end
		mySeat = nil
	end

	local function activate()
		local char = LocalPlayer.Character
		if not char then return end
		local hrp = char:FindFirstChild("HumanoidRootPart")
		if not hrp then return end
		cleanupSeat()
		local sp = hrp.CFrame
		local tp = Vector3.new(Settings.SeatInvisibleX, Settings.SeatInvisibleY, Settings.SeatInvisibleZ)
		char:MoveTo(tp)
		task.wait(0.15)
		local st = Instance.new("Seat")
		st.Name = "invischair"
		st.Anchored = false
		st.CanCollide = false
		st.Transparency = 1
		st.Position = tp
		st.Parent = workspace
		mySeat = st
		local wl = Instance.new("Weld")
		wl.Part0 = st
		wl.Part1 = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
		wl.Parent = st
		task.wait()
		st.CFrame = sp
		for _, d in ipairs(char:GetDescendants()) do
			if d:IsA("BasePart") or d:IsA("Decal") then d.Transparency = 0.5 end
		end
	end

	local function deactivate()
		cleanupSeat()
		if LocalPlayer.Character then
			for _, d in ipairs(LocalPlayer.Character:GetDescendants()) do
				if d:IsA("BasePart") or d:IsA("Decal") then d.Transparency = 0 end
			end
		end
	end

	SeatInvisible.toggle = function()
		active = not active
		Settings.SeatInvisible = active
		if active then activate() else deactivate() end
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	end
	_G.TT_SeatInvisible = SeatInvisible

	task.spawn(function()
		while task.wait(0.5) do
			if active and (not mySeat or not mySeat.Parent) then
				active = false
				Settings.SeatInvisible = false
				deactivate()
				if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
			end
		end
	end)

	LocalPlayer.CharacterAdded:Connect(function()
		active = false
		Settings.SeatInvisible = false
		cleanupSeat()
	end)
end

--------------------------------------------------------------------
-- INTERFACE
--------------------------------------------------------------------
local menuOpen = false
local refreshers = {}
local uiRefresh = function() end

local function buildUI()

local TAB_TOTAL = isMobile and 8 or 7

local menu = Instance.new("CanvasGroup")
menu.AnchorPoint = Vector2.new(0.5, 0.5)
menu.Size = UDim2.fromOffset(440, 560)
menu.Position = UDim2.fromScale(0.5, 0.5)
menu.BackgroundColor3 = Theme.Bg
menu.BorderSizePixel = 0
menu.GroupTransparency = 1
menu.Visible = false
menu.ZIndex = 50
menu.Parent = gui
corner(menu, 14)
stroke(menu, Theme.Accent, 0.5, 1.5)

local menuScale = Instance.new("UIScale")
menuScale.Scale = 0.94
menuScale.Parent = menu

local accentLine = Instance.new("Frame")
accentLine.Size = UDim2.new(1, 0, 0, 3)
accentLine.BackgroundColor3 = Color3.new(1, 1, 1)
accentLine.BorderSizePixel = 0
accentLine.Parent = menu
local accentGradient = Instance.new("UIGradient")
accentGradient.Color = ColorSequence.new({
	ColorSequenceKeypoint.new(0, Theme.Accent),
	ColorSequenceKeypoint.new(0.5, Theme.Accent2),
	ColorSequenceKeypoint.new(1, Theme.Accent),
})
accentGradient.Parent = accentLine
TweenService:Create(accentGradient, TweenInfo.new(2.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), { Offset = Vector2.new(0.5, 0) }):Play()

local titleBar = Instance.new("Frame")
titleBar.Position = UDim2.fromOffset(0, 3)
titleBar.Size = UDim2.new(1, 0, 0, 46)
titleBar.BackgroundTransparency = 1
titleBar.Parent = menu

local title = Instance.new("TextLabel")
title.BackgroundTransparency = 1
title.Position = UDim2.fromOffset(16, 6)
title.Size = UDim2.new(1, -32, 0, 22)
title.Font = Enum.Font.GothamBold
title.TextSize = 17
title.TextXAlignment = Enum.TextXAlignment.Left
title.TextColor3 = Theme.Text
title.Text = "Test Toolkit v85"
title.Parent = titleBar

local subtitle = Instance.new("TextLabel")
subtitle.BackgroundTransparency = 1
subtitle.Position = UDim2.fromOffset(16, 26)
subtitle.Size = UDim2.new(1, -60, 0, 16)
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 12
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.TextColor3 = Theme.SubText
subtitle.Text = "Ctrl direito abre/fecha"
subtitle.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.AnchorPoint = Vector2.new(1, 0)
closeBtn.Position = UDim2.new(1, -10, 0, 6)
closeBtn.Size = UDim2.fromOffset(32, 32)
closeBtn.BackgroundColor3 = Theme.Panel
closeBtn.AutoButtonColor = false
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 15
closeBtn.TextColor3 = Theme.SubText
closeBtn.Text = "X"
closeBtn.Parent = titleBar
corner(closeBtn, 8)

local tabHolder = Instance.new("Frame")
tabHolder.Position = UDim2.fromOffset(8, 52)
tabHolder.Size = UDim2.new(1, -16, 0, 34)
tabHolder.BackgroundTransparency = 1
tabHolder.Parent = menu

local tabIndicator = Instance.new("Frame")
tabIndicator.Size = UDim2.new(1 / TAB_TOTAL, -4, 0, 3)
tabIndicator.Position = UDim2.new(0, 2, 1, -3)
tabIndicator.BackgroundColor3 = Theme.Accent
tabIndicator.BorderSizePixel = 0
tabIndicator.ZIndex = 2
tabIndicator.Parent = tabHolder
corner(tabIndicator, 2)

local pagesHolder = Instance.new("Frame")
pagesHolder.Position = UDim2.fromOffset(8, 92)
pagesHolder.Size = UDim2.new(1, -16, 1, -100)
pagesHolder.BackgroundTransparency = 1
pagesHolder.ClipsDescendants = true
pagesHolder.Parent = menu

do
	local dragging, dragStart, startPos = false, nil, nil
	titleBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = menu.Position
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
			or input.UserInputType == Enum.UserInputType.Touch) then
			local d = input.Position - dragStart
			menu.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X,
				startPos.Y.Scale, startPos.Y.Offset + d.Y)
		end
	end)
	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
		end
	end)
end

local pages = {}
local tabButtons = {}
local tabIndex = {}
local currentPage = nil
local activeTab = nil
local order = 0
local tabCount = 0
local tabOrder = {}
local tabSlot = {}
local tabTotalNow = TAB_TOTAL

local function selectTab(name, instant)
	if activeTab == name then return end
	local idx = tabSlot[name] or tabIndex[name]
	local prevIdx = activeTab and (tabSlot[activeTab] or tabIndex[activeTab]) or idx
	for n, page in pairs(pages) do
		if n ~= name then page.Visible = false end
	end
	local page = pages[name]
	page.Visible = true
	if instant then
		page.Position = UDim2.new()
	else
		local dir = idx >= prevIdx and 1 or -1
		page.Position = UDim2.fromOffset(30 * dir, 0)
		tween(page, { Position = UDim2.new() }, 0.24, Enum.EasingStyle.Quint)
	end
	activeTab = name
	for n, btn in pairs(tabButtons) do
		local active = (n == name)
		tween(btn, {
			BackgroundColor3 = active and Theme.PanelHover or Theme.Panel,
			TextColor3 = active and Color3.new(1, 1, 1) or Theme.SubText,
		}, 0.16)
	end
	local target = UDim2.new((idx - 1) / tabTotalNow, 2, 1, -3)
	if instant then tabIndicator.Position = target
	else tween(tabIndicator, { Position = target }, 0.26, Enum.EasingStyle.Quint) end
end

local function layoutTabs()
	local vis = {}
	for _, n in ipairs(tabOrder) do
		vis[#vis + 1] = n
	end
	local total = math.max(#vis, 1)
	for _, btn in pairs(tabButtons) do btn.Visible = false end
	table.clear(tabSlot)
	for i, n in ipairs(vis) do
		local btn = tabButtons[n]
		btn.Visible = true
		btn.Position = UDim2.new((i - 1) / total, 2, 0, 0)
		btn.Size = UDim2.new(1 / total, -4, 1, -6)
		tabSlot[n] = i
	end
	tabTotalNow = total
	tabIndicator.Size = UDim2.new(1 / total, -4, 0, 3)
	if activeTab and tabSlot[activeTab] then
		tabIndicator.Position = UDim2.new((tabSlot[activeTab] - 1) / total, 2, 1, -3)
	end
end

local function newPage(name, label)
	local page = Instance.new("ScrollingFrame")
	page.Name = name
	page.Size = UDim2.fromScale(1, 1)
	page.BackgroundTransparency = 1
	page.BorderSizePixel = 0
	page.ScrollBarThickness = 3
	page.ScrollBarImageColor3 = Theme.Accent
	page.AutomaticCanvasSize = Enum.AutomaticSize.Y
	page.CanvasSize = UDim2.new()
	page.Visible = false
	page.Parent = pagesHolder
	local layout = Instance.new("UIListLayout")
	layout.Padding = UDim.new(0, 6)
	layout.SortOrder = Enum.SortOrder.LayoutOrder
	layout.Parent = page
	local pad = Instance.new("UIPadding")
	pad.PaddingRight = UDim.new(0, 6)
	pad.PaddingBottom = UDim.new(0, 6)
	pad.Parent = page

	tabCount += 1
	tabOrder[#tabOrder + 1] = name
	local btn = Instance.new("TextButton")
	btn.Position = UDim2.new((tabCount - 1) / TAB_TOTAL, 2, 0, 0)
	btn.Size = UDim2.new(1 / TAB_TOTAL, -4, 1, -6)
	btn.BackgroundColor3 = Theme.Panel
	btn.BorderSizePixel = 0
	btn.AutoButtonColor = false
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 11
	btn.TextColor3 = Theme.SubText
	btn.Text = label
	btn.Parent = tabHolder
	corner(btn, 8)
	btn.MouseButton1Click:Connect(function() selectTab(name) end)

	pages[name] = page
	tabButtons[name] = btn
	tabIndex[name] = tabCount
	currentPage = page
	return page
end

local function newRow(height, className)
	order += 1
	local r = Instance.new(className or "Frame")
	r.Size = UDim2.new(1, 0, 0, height)
	r.BackgroundColor3 = Theme.Panel
	r.BorderSizePixel = 0
	r.LayoutOrder = order
	if r:IsA("TextButton") then
		r.AutoButtonColor = false
		r.Text = ""
	end
	corner(r, 8)
	r.Parent = currentPage
	return r
end

local function rowLabel(parent, text, desc, rightPad)
	local l = Instance.new("TextLabel")
	l.BackgroundTransparency = 1
	l.Position = UDim2.fromOffset(12, desc and 4 or 0)
	l.Size = UDim2.new(1, -(rightPad or 66), desc and 0 or 1, desc and 18 or 0)
	l.Font = Enum.Font.Gotham
	l.TextSize = 13
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.TextColor3 = Theme.Text
	l.TextTruncate = Enum.TextTruncate.AtEnd
	l.Text = tostring(text or "")
	l.Parent = parent
	if desc then
		local d = Instance.new("TextLabel")
		d.BackgroundTransparency = 1
		d.Position = UDim2.fromOffset(12, 22)
		d.Size = UDim2.new(1, -24, 0, 13)
		d.Font = Enum.Font.Gotham
		d.TextSize = 10
		d.TextXAlignment = Enum.TextXAlignment.Left
		d.TextColor3 = Theme.SubText
		d.TextTruncate = Enum.TextTruncate.AtEnd
		d.Text = tostring(desc or "")
		d.Parent = parent
	end
	return l
end

local function hover(row)
	local scale = Instance.new("UIScale")
	scale.Parent = row
	row.MouseEnter:Connect(function() tween(row, { BackgroundColor3 = Theme.PanelHover }, 0.12) end)
	row.MouseLeave:Connect(function()
		tween(row, { BackgroundColor3 = Theme.Panel }, 0.12)
		tween(scale, { Scale = 1 }, 0.1)
	end)
end

local function addSection(text)
	order += 1
	local holder = Instance.new("Frame")
	holder.BackgroundTransparency = 1
	holder.Size = UDim2.new(1, 0, 0, 26)
	holder.LayoutOrder = order
	holder.Parent = currentPage
	local l = Instance.new("TextLabel")
	l.BackgroundTransparency = 1
	l.Position = UDim2.fromOffset(4, 6)
	l.Size = UDim2.new(1, -4, 0, 16)
	l.Font = Enum.Font.GothamBold
	l.TextSize = 12
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.TextColor3 = Theme.Accent
	l.Text = string.upper(tostring(text or ""))
	l.Parent = holder
	local line = Instance.new("Frame")
	line.AnchorPoint = Vector2.new(0, 1)
	line.Position = UDim2.new(0, 4, 1, 0)
	line.Size = UDim2.new(1, -4, 0, 1)
	line.BackgroundColor3 = Theme.Accent
	line.BackgroundTransparency = 0.7
	line.BorderSizePixel = 0
	line.Parent = holder
	return holder
end

local function addInfo(text, height)
	order += 1
	local l = Instance.new("TextLabel")
	l.BackgroundTransparency = 1
	l.Size = UDim2.new(1, 0, 0, height or 32)
	l.LayoutOrder = order
	l.Font = Enum.Font.Gotham
	l.TextSize = 12
	l.TextWrapped = true
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.TextYAlignment = Enum.TextYAlignment.Top
	l.TextColor3 = Theme.SubText
	l.Text = tostring(text or "")
	l.Parent = currentPage
	return l
end

local function addToggle(text, key, desc, onChange)
	local row = newRow(desc and 42 or 36, "TextButton")
	rowLabel(row, text, desc)
	hover(row)
	local pill = Instance.new("Frame")
	pill.AnchorPoint = Vector2.new(1, 0.5)
	pill.Position = UDim2.new(1, -12, 0.5, 0)
	pill.Size = UDim2.fromOffset(40, 20)
	pill.BorderSizePixel = 0
	pill.Parent = row
	corner(pill, 10)
	local knob = Instance.new("Frame")
	knob.AnchorPoint = Vector2.new(0, 0.5)
	knob.Size = UDim2.fromOffset(14, 14)
	knob.BackgroundColor3 = Color3.new(1, 1, 1)
	knob.BorderSizePixel = 0
	knob.Parent = pill
	corner(knob, 7)
	local function render(animate)
		local on = Settings[key]
		local pillColor = on and Theme.Accent or Theme.Off
		local knobPos = on and UDim2.new(1, -17, 0.5, 0) or UDim2.new(0, 3, 0.5, 0)
		if animate then
			tween(pill, { BackgroundColor3 = pillColor })
			tween(knob, { Position = knobPos }, 0.22, Enum.EasingStyle.Back)
		else
			pill.BackgroundColor3 = pillColor
			knob.Position = knobPos
		end
	end
	render(false)
	table.insert(refreshers, function() render(true) end)
	row.MouseButton1Click:Connect(function()
		Settings[key] = not Settings[key]
		for _, refresh in ipairs(refreshers) do refresh() end
		notify(tostring(text) .. (Settings[key] and ": ligado" or ": desligado"), Settings[key] and "on" or "off")
		if onChange then onChange() end
	end)
end

local function addCycle(text, key, options, desc, onChange)
	local row = newRow(desc and 42 or 36, "TextButton")
	hover(row)
	local l = rowLabel(row, "", desc, 30)
	local arrow = Instance.new("TextLabel")
	arrow.BackgroundTransparency = 1
	arrow.AnchorPoint = Vector2.new(1, 0.5)
	arrow.Position = UDim2.new(1, -12, 0.5, 0)
	arrow.Size = UDim2.fromOffset(16, 16)
	arrow.Font = Enum.Font.GothamBold
	arrow.TextSize = 14
	arrow.TextColor3 = Theme.Accent
	arrow.Text = ">"
	arrow.Parent = row
	local function render()
		local opt = options[Settings[key]] or "?"
		l.Text = tostring(text) .. ": " .. tostring(opt)
	end
	render()
	row.MouseButton1Click:Connect(function()
		Settings[key] = (Settings[key] % #options) + 1
		render()
		if onChange then onChange() end
	end)
end

local sliderDrag = nil
local sliderPage = nil

local function addSlider(text, key, min, max, step, decimals, desc, onChange)
	local page = currentPage
	local row = newRow(desc and 58 or 50, "Frame")
	local name = rowLabel(row, text, desc, 0)
	name.Size = UDim2.new(0.65, -12, 0, desc and 18 or 28)
	local valueLabel = Instance.new("TextLabel")
	valueLabel.BackgroundTransparency = 1
	valueLabel.AnchorPoint = Vector2.new(1, 0)
	valueLabel.Position = UDim2.new(1, -12, 0, desc and 4 or 0)
	valueLabel.Size = UDim2.new(0.35, -12, 0, desc and 18 or 28)
	valueLabel.Font = Enum.Font.GothamBold
	valueLabel.TextSize = 13
	valueLabel.TextXAlignment = Enum.TextXAlignment.Right
	valueLabel.TextColor3 = Theme.SubText
	valueLabel.Parent = row
	local hit = Instance.new("TextButton")
	hit.BackgroundTransparency = 1
	hit.Text = ""
	hit.Position = UDim2.new(0, 12, 0, desc and 40 or 28)
	hit.Size = UDim2.new(1, -24, 0, 16)
	hit.Parent = row
	local bar = Instance.new("Frame")
	bar.AnchorPoint = Vector2.new(0, 0.5)
	bar.Position = UDim2.new(0, 0, 0.5, 0)
	bar.Size = UDim2.new(1, 0, 0, 6)
	bar.BackgroundColor3 = Theme.Off
	bar.BorderSizePixel = 0
	bar.Parent = hit
	corner(bar, 3)
	local fill = Instance.new("Frame")
	fill.Size = UDim2.new(0, 0, 1, 0)
	fill.BackgroundColor3 = Theme.Accent
	fill.BorderSizePixel = 0
	fill.Parent = bar
	corner(fill, 3)
	local handle = Instance.new("Frame")
	handle.AnchorPoint = Vector2.new(0.5, 0.5)
	handle.Position = UDim2.new(1, 0, 0.5, 0)
	handle.Size = UDim2.fromOffset(12, 12)
	handle.BackgroundColor3 = Color3.new(1, 1, 1)
	handle.BorderSizePixel = 0
	handle.Parent = fill
	corner(handle, 6)

	local fmt = "%." .. (decimals or 0) .. "f"
	local function render()
		local rel = math.clamp((Settings[key] - min) / (max - min), 0, 1)
		valueLabel.Text = string.format(fmt, Settings[key])
		fill.Size = UDim2.new(rel, 0, 1, 0)
	end
	render()
	table.insert(refreshers, render)

	local function setFromX(x)
		local rel = math.clamp((x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
		local v = min + rel * (max - min)
		v = math.floor(v / step + 0.5) * step
		Settings[key] = math.clamp(v, min, max)
		render()
		if onChange then onChange() end
	end

	hit.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then
			setFromX(input.Position.X)
			sliderDrag = setFromX
			page.ScrollingEnabled = false
			sliderPage = page
		end
	end)

	valueLabel.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			local box = Instance.new("TextBox")
			box.Size = UDim2.new(1, 0, 1, 0)
			box.BackgroundTransparency = 0.5
			box.BackgroundColor3 = Theme.Bg
			box.Text = tostring(Settings[key])
			box.Font = Enum.Font.GothamBold
			box.TextSize = 13
			box.TextColor3 = Theme.Text
			box.ClearTextOnFocus = false
			box.Parent = valueLabel
			corner(box, 4)
			box:CaptureFocus()
			box.FocusLost:Connect(function()
				local num = tonumber(box.Text)
				if num then
					Settings[key] = math.clamp(num, min, max)
					render()
					if onChange then onChange() end
				end
				box:Destroy()
			end)
		end
	end)
end

UserInputService.InputChanged:Connect(function(input)
	if sliderDrag and (input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch) then
		sliderDrag(input.Position.X)
	end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
		sliderDrag = nil
		if sliderPage then
			sliderPage.ScrollingEnabled = true
			sliderPage = nil
		end
	end
end)

local rebinding = nil

local function addKeybind(text, key, desc, allowMouse)
	local row = newRow(desc and 42 or 36, "TextButton")
	row.Visible = not isMobile
	hover(row)
	local l = rowLabel(row, "", desc, 30)
	local function render()
		local kb = Settings[key]
		l.Text = tostring(text) .. ": [" .. tostring(kb and kb.Name or "?") .. "]"
	end
	render()
	row.MouseButton1Click:Connect(function()
		l.Text = tostring(text) .. ": pressione uma tecla..."
		rebinding = {
			mouse = allowMouse,
			apply = function(bind)
				if bind then
					Settings[key] = bind
					notify(tostring(text) .. ": " .. tostring(bind.Name), "info")
				end
				render()
			end,
		}
	end)
end

local function addButton(text, desc, onClick)
	local row = newRow(desc and 42 or 36, "TextButton")
	hover(row)
	local l = rowLabel(row, text, desc, 30)
	l.TextColor3 = Theme.Accent
	row.MouseButton1Click:Connect(function()
		if onClick then onClick() end
	end)
end

local function addDropdown(text, key, desc, onChange)
	local row = newRow(desc and 42 or 36, "TextButton")
	hover(row)
	local l = rowLabel(row, "", desc, 30)
	local arrow = Instance.new("TextLabel")
	arrow.BackgroundTransparency = 1
	arrow.AnchorPoint = Vector2.new(1, 0.5)
	arrow.Position = UDim2.new(1, -12, 0.5, 0)
	arrow.Size = UDim2.fromOffset(16, 16)
	arrow.Font = Enum.Font.GothamBold
	arrow.TextSize = 14
	arrow.TextColor3 = Theme.Accent
	arrow.Text = "▼"
	arrow.Parent = row

	local dropdownFrame = nil
	local function getList()
		local list = { "Nenhum" }
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer then
				local display = plr.DisplayName or plr.Name
				list[#list + 1] = tostring(display)
			end
		end
		return list
	end
	local function closeDropdown()
		if dropdownFrame then dropdownFrame:Destroy(); dropdownFrame = nil end
	end
	local function render()
		l.Text = tostring(text) .. ": " .. tostring(Settings[key] or "Nenhum")
	end
	render()
	local function openDropdown()
		if dropdownFrame then closeDropdown(); return end
		local list = getList()
		local h = math.min(#list * 28, 240)
		dropdownFrame = Instance.new("Frame")
		dropdownFrame.Position = UDim2.fromOffset(
			row.AbsolutePosition.X + 12,
			row.AbsolutePosition.Y + row.AbsoluteSize.Y + 4
		)
		dropdownFrame.Size = UDim2.new(0, row.AbsoluteSize.X - 24, 0, h)
		dropdownFrame.BackgroundColor3 = Theme.Bg
		dropdownFrame.BorderSizePixel = 0
		dropdownFrame.ZIndex = 500
		dropdownFrame.Parent = gui
		corner(dropdownFrame, 8)
		stroke(dropdownFrame, Theme.Accent, 0.5, 1)
		local scroll = Instance.new("ScrollingFrame")
		scroll.Size = UDim2.fromScale(1, 1)
		scroll.BackgroundTransparency = 1
		scroll.BorderSizePixel = 0
		scroll.ScrollBarThickness = 3
		scroll.ScrollBarImageColor3 = Theme.Accent
		scroll.CanvasSize = UDim2.new(0, 0, 0, #list * 28)
		scroll.Parent = dropdownFrame
		local layout = Instance.new("UIListLayout")
		layout.Padding = UDim.new(0, 0)
		layout.Parent = scroll
		for i, name in ipairs(list) do
			local btn = Instance.new("TextButton")
			btn.Size = UDim2.new(1, -8, 0, 26)
			btn.Position = UDim2.new(0, 4, 0, (i - 1) * 28 + 2)
			btn.BackgroundColor3 = Theme.Panel
			btn.BorderSizePixel = 0
			btn.AutoButtonColor = false
			btn.Font = Enum.Font.Gotham
			btn.TextSize = 12
			btn.TextColor3 = (Settings[key] == name) and Theme.Accent or Theme.Text
			btn.Text = tostring(name)
			btn.TextXAlignment = Enum.TextXAlignment.Left
			btn.Parent = scroll
			corner(btn, 6)
			local pad = Instance.new("UIPadding")
			pad.PaddingLeft = UDim.new(0, 8)
			pad.Parent = btn
			btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Theme.PanelHover end)
			btn.MouseLeave:Connect(function() btn.BackgroundColor3 = Theme.Panel end)
			btn.MouseButton1Click:Connect(function()
				Settings[key] = name
				render()
				closeDropdown()
				if onChange then onChange(name) end
			end)
		end
	end
	row.MouseButton1Click:Connect(openDropdown)
	table.insert(refreshers, render)
end

--------------------------------------------------------------------
-- ABAS
--------------------------------------------------------------------
newPage("aim", "Mira")
addSection("Modo de mira")
addToggle("Aimbot", "AimEnabled", "Ativa a mira automática")
addToggle("Legit Aim", "UseLegitAim", "Move a câmera suavemente")
addToggle("Silent Aim", "UseSilentAim", "Flick instantâneo no alvo")
addCycle("Modo", "AimMode", { "Segurar", "Alternar", "Automático" }, "Como ativa", function() aimActive = false end)
addCycle("Alvos", "TargetMode", { "Jogadores", "Bots", "Ambos" }, "Quem mirar")
addToggle("Ignorar equipe", "TeamCheck", "Não mira aliados")
addToggle("Checar parede", "AimWallCheck", "Só mira se enxergar")
addSection("Legit Aim")
addSlider("Suavidade", "Smoothness", 0.01, 0.95, 0.01, 2, "Menor = mais rápida")
addToggle("Trava ao atirar", "FireLock", "Sem suavização ao atirar")
addSection("FOV do Legit")
addToggle("Limitar pelo FOV", "FOVEnabled", "Só dentro do círculo")
addSlider("Raio do FOV", "FOVRadius", 20, 600, 5, 0, "Pixels")
addSection("Silent Aim")
addSlider("FOV do Silent", "SilentAimFOV", 50, 800, 10, 0, "Raio")
addToggle("Só visíveis", "SilentAimVisible", "Só atira com LOS")
addSection("Trigger Bot")
addToggle("Trigger Bot", "TriggerBot", "Atira automaticamente")
addToggle("Sempre ativo", "TriggerBotAlways", "Sem precisar segurar")
addToggle("Só visíveis", "TriggerVisible", "Só atira com LOS")
addSlider("FOV do Trigger", "TriggerFOV", 5, 400, 1, 0, "5 = centro")
addSlider("Delay", "TriggerDelay", 0.01, 1, 0.01, 2, "Entre tiros")
addSection("Teclas (PC)")
addKeybind("Tecla da mira", "AimKey", "Ativa/desativa a mira")
addKeybind("Trocar de alvo", "SwitchKey", "Próximo inimigo")

newPage("hitbox", "Hitbox")
addSection("Hitbox Expander")
addToggle("Hitbox Expander", "HitboxExpander", "Aumenta a hitbox")
addToggle("Invisível", "HitboxInvisible", "Hitbox transparente")
addSlider("Tamanho", "HitboxSize", 5, 30, 1, 0, "Studs")
addSlider("Alcance", "HitboxRange", 10, 1000, 10, 0, "Distância máxima")
addSection("Partes")
addToggle("Cabeça", "ExpandHead", "")
addToggle("Torso (R6)", "ExpandTorso", "")
addToggle("UpperTorso (R15)", "ExpandUpperTorso", "")
addToggle("LowerTorso (R15)", "ExpandLowerTorso", "")

newPage("esp", "ESP")
addSection("ESP")
addToggle("ESP", "ESP", "Mostra alvos")
addToggle("Ver através de paredes", "Wallhack", "Contorno atrás de objetos")
addToggle("Contorno colorido", "ESPHighlight", "Mais pesado")
addToggle("Cor arco-íris", "Rainbow", "Cor animada")
addToggle("Nomes e distância", "ShowNames", "Texto acima")
addToggle("Barra de vida", "ShowHealth", "Barra verde/vermelha")
addToggle("Linhas", "Tracers", "Linha da tela")
addCycle("Origem da linha", "TracerOrigin", { "Baixo", "Centro", "Cursor" }, "De onde sai")
addSection("Limites")
addSlider("Distância máxima", "ESPMaxDist", 0, 3000, 50, 0, "0 = sem limite")
addSlider("Máx. de alvos", "ESPMaxTargets", 1, 30, 1, 0, "Menos = mais FPS")
addSlider("Transparência", "ESPFillTrans", 0, 1, 0.05, 2, "0 = sólido")

newPage("player", "Jogador")
addSection("Movimento")
addToggle("Velocidade", "WalkSpeedOn", "Muda WalkSpeed")
addSlider("Valor da velocidade", "WalkSpeed", 16, 5000, 5, 0, "Padrão 16")
addToggle("Pulo", "JumpOn", "Muda JumpPower")
addSlider("Valor do pulo", "JumpPower", 50, 900, 5, 0, "Padrão 50")
addToggle("Pulo infinito", "InfJump", "Pula no ar")
addToggle("Voo", "Fly", "Espaço sobe, Ctrl desce")
addSlider("Velocidade do voo", "FlySpeed", 10, 5000, 5, 0, "Studs/s")
addToggle("Noclip", "Noclip", "Atravessa paredes")
addSection("Teclas (PC)")
addKeybind("Tecla do voo", "FlyKey", "Liga/desliga voo")
addKeybind("Tecla do noclip", "NoclipKey", "Liga/desliga noclip")
addSection("Invisibilidade (Seat Bug)")
addToggle("Invisível (Seat Bug)", "SeatInvisible", "Ativa a invisibilidade", function()
	if _G.TT_SeatInvisible then pcall(_G.TT_SeatInvisible.toggle) end
end)
addKeybind("Tecla da Invisibilidade", "SeatInvisibleKey", "Liga/desliga invisibilidade")

newPage("misc", "Extras")
addSection("Visão")
addToggle("Luz total", "FullBright", "Remove escuridão")
addToggle("Sem neblina", "NoFog", "Enxerga longe")
addToggle("FOV da câmera", "CamFOVOn", "Muda FOV")
addSlider("Valor do FOV", "CamFOV", 40, 120, 1, 0, "Padrão 70")
addToggle("Anti-AFK", "AntiAFK", "Evita kick")
addSection("Interface")
addToggle("HUD", "ShowHUD", "FPS e estados")
addToggle("Mover HUD", "HUDEdit", "Arraste o HUD")
addToggle("Notificações", "Notifications", "Avisos")
addSlider("Transparência do menu", "MenuAlpha", 0, 0.6, 0.05, 2, "0 = sólido")

newPage("fling", "Fling")
addSection("Fling Bots")
addToggle("Fling Bots", "Fling", "Arremessa bots")
addSlider("Alcance (bots)", "FlingRange", 3, 30, 1, 0, "Studs")
addSlider("Força", "FlingPower", 500, 5000, 100, 0, "Força")
addSlider("Intervalo", "FlingRepeat", 0.05, 1, 0.05, 2, "Entre aplicações")
addSection("Fling Player (métodos)")
addInfo("Escolha o player e clique em um dos botões.", 30)
addDropdown("Escolher Player", "FlingPlayerTarget", "Alvo do fling")
addButton("FLING PLAYER (K1LAS1K interno)", "Usa o método 9e9 + BodyVelocity P=20000", function()
	local name = Settings.FlingPlayerTarget
	if not name or name == "Nenhum" then
		notify("Fling: selecione um player", "off")
		return
	end
	for _, plr in ipairs(Players:GetPlayers()) do
		if (plr.Name == name or plr.DisplayName == name) and plr ~= LocalPlayer then
			task.spawn(function()
				local ok = Fling.doFlingPlayer(plr)
				notify(ok and ("Fling: " .. plr.DisplayName) or "Fling: falhou", ok and "on" or "off")
			end)
			return
		end
	end
end)
addButton("ABRIR MENU K1LAS1K ORIGINAL", "Carrega a GUI completa do K1LAS1K", function()
	task.spawn(function()
		notify("Carregando K1LAS1K...", "info")
		local ok, err = pcall(function()
			loadstring(game:HttpGet("https://raw.githubusercontent.com/K1LAS1K/Ultimate-Fling-GUI/main/flingscript.lua"))()
		end)
		if not ok then
			notify("Falha: " .. tostring(err), "off")
		else
			notify("K1LAS1K carregado!", "on")
		end
	end)
end)
addButton("FLING TODOS OS PLAYERS", "Arremessa TODOS os players do servidor", function()
	local count = 0
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character then
			task.spawn(function()
				Fling.doFlingPlayer(plr)
			end)
			count = count + 1
			task.wait(0.1)
		end
	end
	notify("Fling: " .. count .. " players", "on")
end)
addSlider("Duração (segundos)", "FlingPlayerDuration", 0.5, 5, 0.5, 1, "Tempo preso")
addSection("Touch Fling")
addToggle("Touch Fling", "TouchFling", "Arremessa quem chegar perto de você")
addSlider("Alcance do Touch", "TouchFlingRange", 3, 30, 1, 0, "Studs")
addKeybind("Tecla do Touch Fling", "TouchFlingKey", "Liga/desliga Touch Fling")
addSection("Spectate")
addDropdown("Spectate Player", "SpectateTarget", "Seguir um player")
addButton("Spectar", "Começa a spectar o player selecionado", function()
	local name = Settings.SpectateTarget
	if not name or name == "Nenhum" then
		notify("Spectate: selecione um player", "off")
		return
	end
	for _, plr in ipairs(Players:GetPlayers()) do
		if (plr.DisplayName == name or plr.Name == name) and plr ~= LocalPlayer then
			local ok = Spectate.start(plr)
			notify(ok and ("Spectando: " .. plr.DisplayName) or "Spectate falhou", ok and "on" or "off")
			return
		end
	end
end)
addButton("Parar Spectate", "Volta a câmera pro seu personagem", function()
	Spectate.stop()
	notify("Spectate parado", "info")
end)
addSection("Teclas (PC)")
addKeybind("Tecla do Fling Bots", "FlingKey", "Liga/desliga Fling")
addKeybind("Tecla do Fling Player", "FlingPlayerKey", "Arremessa o player selecionado")

newPage("security", "Segurança")
addSection("Proteção contra morte")
addToggle("Anti Void", "AntiVoid", "Volta se cair no vazio")
addSection("Proteção contra arremesso")
addToggle("Anti Fling", "AntiFling", "Bloqueia arremessos")
addSlider("Limite", "AntiFlingSpeed", 50, 500, 10, 0, "Velocidade")
addSection("Proteção contra kick")
addToggle("Anti AFK", "AntiAFK", "Evita kick")

if isMobile then
	newPage("buttons", "Botões")
	addSection("Botões na tela")
	addToggle("Botão MIRA", "ShowBtnAim", "")
	addToggle("Botão VOO", "ShowBtnFly", "")
	addToggle("Botão NOCLIP", "ShowBtnNoclip", "")
	addToggle("Botão ESP", "ShowBtnEsp", "")
	addToggle("Botão FLING", "ShowBtnFling", "")
	addSection("Aparência")
	addSlider("Tamanho dos botões", "MobileBtnSize", 40, 90, 1, 0, "Pixels")
	addSlider("Transparência", "MobileBtnAlpha", 0, 0.9, 0.05, 2, "0 = sólido")
end

--------------------------------------------------------------------
-- BOTÕES MOBILE
--------------------------------------------------------------------
local mobileBtns = {}

local function makeMobileBtn(label, pos, showKey, stateFn, onDown, onUp)
	local b = Instance.new("TextButton")
	b.AnchorPoint = Vector2.new(0.5, 0.5)
	b.Position = pos
	b.Size = UDim2.fromOffset(56, 56)
	b.BackgroundColor3 = Theme.Bg
	b.BorderSizePixel = 0
	b.AutoButtonColor = false
	b.Font = Enum.Font.GothamBold
	b.TextSize = 11
	b.TextColor3 = Theme.Text
	b.Text = tostring(label or "")
	b.Visible = false
	b.ZIndex = 40
	b.Parent = gui
	corner(b, 9999)
	stroke(b, Theme.Accent, 0.2, 1.5)
	b.InputBegan:Connect(function(input)
		if input.UserInputType ~= Enum.UserInputType.Touch
			and input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
		if onDown then onDown() end
	end)
	b.InputEnded:Connect(function(input)
		if (input.UserInputType == Enum.UserInputType.Touch
			or input.UserInputType == Enum.UserInputType.MouseButton1)
			and onUp then
			onUp()
		end
	end)
	mobileBtns[#mobileBtns + 1] = { Btn = b, ShowKey = showKey, State = stateFn }
end

makeMobileBtn("TT", UDim2.fromScale(0.07, 0.2), nil, function() return menuOpen end, function()
	if _G.TT_setMenu then _G.TT_setMenu(not _G.TT_isMenuOpen()) end
end)
makeMobileBtn("MIRA", UDim2.fromScale(0.9, 0.42), "ShowBtnAim", function() return aimActive end, function()
	aimActive = true
end, function()
	if Settings.AimMode == 1 then aimActive = false end
end)
makeMobileBtn("VOO", UDim2.fromScale(0.9, 0.54), "ShowBtnFly", function() return Settings.Fly end,
	function() Settings.Fly = not Settings.Fly end)
makeMobileBtn("NOCLIP", UDim2.fromScale(0.9, 0.66), "ShowBtnNoclip", function() return Settings.Noclip end,
	function() Settings.Noclip = not Settings.Noclip end)
makeMobileBtn("ESP", UDim2.fromScale(0.8, 0.66), "ShowBtnEsp", function() return Settings.ESP end,
	function() Settings.ESP = not Settings.ESP end)
makeMobileBtn("FLING", UDim2.fromScale(0.8, 0.78), "ShowBtnFling", function() return Settings.Fling end,
	function() Settings.Fling = not Settings.Fling end)

local function updateMobileBtns()
	if not isMobile then
		for _, m in ipairs(mobileBtns) do m.Btn.Visible = false end
		return
	end
	for _, m in ipairs(mobileBtns) do
		local show = m.ShowKey == nil or Settings[m.ShowKey]
		m.Btn.Visible = show
		if show then
			local sz = Settings.MobileBtnSize
			if m.Btn.Size.X.Offset ~= sz then m.Btn.Size = UDim2.fromOffset(sz, sz) end
			m.Btn.BackgroundTransparency = Settings.MobileBtnAlpha
			m.Btn.BackgroundColor3 = m.State() and Theme.Accent or Theme.Bg
		end
	end
end

layoutTabs()
selectTab("aim", true)
updateMobileBtns()

--------------------------------------------------------------------
-- MENU OPEN/CLOSE
--------------------------------------------------------------------
local function refreshAll()
	for _, refresh in ipairs(refreshers) do refresh() end
end
uiRefresh = refreshAll
_G.TT_uiRefresh = refreshAll

local function baseScale()
	local vp = Camera.ViewportSize
	return math.clamp(math.min(vp.X / 470, vp.Y / 590), 0.5, 1)
end

local function setMenu(open)
	menuOpen = open
	local s = baseScale()
	if open then
		menu.Visible = true
		menuScale.Scale = s * 0.94
		tween(menu, { GroupTransparency = Settings.MenuAlpha }, 0.2)
		tween(menuScale, { Scale = s }, 0.26, Enum.EasingStyle.Back)
	else
		tween(menu, { GroupTransparency = 1 }, 0.16)
		tween(menuScale, { Scale = s * 0.94 }, 0.16)
		task.delay(0.18, function()
			if not menuOpen then menu.Visible = false end
		end)
	end
end
_G.TT_setMenu = setMenu
_G.TT_isMenuOpen = function() return menuOpen end

closeBtn.MouseButton1Click:Connect(function() setMenu(false) end)

refreshAll()

-- KEYBIND
UserInputService.InputBegan:Connect(function(input)
	if rebinding then
		local r = rebinding
		if input.KeyCode == Enum.KeyCode.Escape then
			rebinding = nil
			r.apply(nil)
			return
		end
		if input.KeyCode ~= Enum.KeyCode.Unknown then
			rebinding = nil
			r.apply(input.KeyCode)
		end
		return
	end
	if UserInputService:GetFocusedTextBox() then return end
	if input.KeyCode == Settings.MENU_KEY then
		setMenu(not menuOpen)
	end
end)

end -- fim buildUI

--------------------------------------------------------------------
-- BUILD
--------------------------------------------------------------------
local okUI, errUI = pcall(buildUI)
if okUI then
	bootShow("TestToolkit v85 carregado  •  " .. (isMobile and "botão TT abre o menu" or "Ctrl direito abre o menu"),
		Color3.fromRGB(80, 255, 130), 5)
	print("[TestToolkit] v85 carregado com sucesso!")
else
	warn("[TestToolkit] erro na interface: " .. tostring(errUI))
	bootShow("TestToolkit: erro: " .. tostring(errUI), Color3.fromRGB(255, 90, 90))
end

--------------------------------------------------------------------
-- TECLAS DE JOGO
--------------------------------------------------------------------
local function matchesBind(input, bind)
	if typeof(bind) ~= "EnumItem" then return false end
	if bind.EnumType == Enum.KeyCode then return input.KeyCode == bind end
	return input.UserInputType == bind
end

UserInputService.InputBegan:Connect(function(input)
	if UserInputService:GetFocusedTextBox() then return end
	if rebinding then return end

	if matchesBind(input, Settings.AimKey) then
		if Settings.AimMode == 1 then
			aimActive = true
		elseif Settings.AimMode == 2 then
			aimActive = not aimActive
			notify("Mira " .. (aimActive and "ativada" or "desativada"), aimActive and "on" or "off")
		end
	elseif matchesBind(input, Settings.NoclipKey) then
		Settings.Noclip = not Settings.Noclip
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	elseif matchesBind(input, Settings.FlyKey) then
		Settings.Fly = not Settings.Fly
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	elseif matchesBind(input, Settings.SeatInvisibleKey) then
		if _G.TT_SeatInvisible then pcall(_G.TT_SeatInvisible.toggle) end
	elseif input.KeyCode == Settings.FlingKey then
		Settings.Fling = not Settings.Fling
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	elseif input.KeyCode == Settings.FlingPlayerKey then
		task.spawn(function() Fling.flingSelected() end)
	elseif input.KeyCode == Settings.TouchFlingKey then
		Settings.TouchFling = not Settings.TouchFling
		notify("Touch Fling " .. (Settings.TouchFling and "ligado" or "desligado"), Settings.TouchFling and "on" or "off")
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if Settings.AimMode == 1 and matchesBind(input, Settings.AimKey) then
		aimActive = false
	end
end)

--------------------------------------------------------------------
-- HUD
--------------------------------------------------------------------
local hud = Instance.new("TextLabel")
hud.Name = "HUD"
hud.Position = UDim2.fromOffset(Settings.HUDX, Settings.HUDY)
hud.Size = UDim2.fromOffset(1000, 20)
hud.BackgroundColor3 = Theme.Bg
hud.BackgroundTransparency = 0.25
hud.BorderSizePixel = 0
hud.Font = Enum.Font.GothamMedium
hud.TextSize = 12
hud.TextColor3 = Theme.Text
hud.TextXAlignment = Enum.TextXAlignment.Left
hud.ZIndex = 30
hud.Parent = gui
corner(hud, 6)
local hudPad = Instance.new("UIPadding")
hudPad.PaddingLeft = UDim.new(0, 8)
hudPad.Parent = hud

local fpsFrames, fpsAcc, hudAcc = 0, 0, 0
local lastFps = 60

RunService.RenderStepped:Connect(function(dt)
	pcall(function()
		fpsFrames += 1
		fpsAcc += dt
		if fpsAcc >= 0.5 then
			lastFps = fpsFrames / fpsAcc
			fpsFrames, fpsAcc = 0, 0
		end
		hud.Visible = Settings.ShowHUD or Settings.HUDEdit
		hud.Position = UDim2.fromOffset(Settings.HUDX, Settings.HUDY)
		hud.BackgroundColor3 = Settings.HUDEdit and Theme.Accent or Theme.Bg
		hudAcc += dt
		if hudAcc >= 0.25 then
			hudAcc = 0
			local okp, pv = pcall(LocalPlayer.GetNetworkPing, LocalPlayer)
			local ping = math.floor((okp and pv or 0) * 1000)
			local parts = {
				tostring(math.floor(lastFps or 0)) .. " FPS",
				tostring(ping) .. " ms",
			}
			if Settings.Fly then parts[#parts + 1] = "Voo" end
			if Settings.Noclip then parts[#parts + 1] = "Noclip" end
			if Settings.Fling then parts[#parts + 1] = "Fling" end
			if Settings.HitboxExpander then parts[#parts + 1] = "Hitbox" end
			if Settings.SeatInvisible then parts[#parts + 1] = "Invisível" end
			if Settings.AimEnabled and (aimActive or Settings.AimMode == 3) then
				parts[#parts + 1] = "🎯 " .. tostring(aimDebugFound or 0) .. " alvos"
				parts[#parts + 1] = "alvo: " .. tostring(aimDebugTarget or "?")
				parts[#parts + 1] = aimDebugMoved and "✔ moveu" or "✘ não moveu"
				parts[#parts + 1] = Settings.AimWallCheck and "🧱 parede ON" or "🧱 parede OFF"
			end
			if Settings.TriggerBot and triggerBotActive then
				parts[#parts + 1] = "🔫 " .. tostring(triggerTargetName or "?")
			end
			if Settings.TouchFling then parts[#parts + 1] = "💥 TouchFling" end
			if Settings.Spectating then parts[#parts + 1] = "👁 " .. tostring(Settings.SpectateTarget) end
			hud.Text = table.concat(parts, "  •  ")
		end
	end)
end)
