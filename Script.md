--[[
    TestToolkit v71 - UNIVERSAL (fix menu)
    Ctrl direito abre/fecha
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")

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
	-- ESP
	ESP = true, Rainbow = true, ShowNames = true, ShowHealth = true,
	Wallhack = true, ESPMaxDist = 600, ESPMaxTargets = 12,
	ESPHighlight = true, ESPColor = Color3.fromRGB(255, 80, 80), ESPFillTrans = 0.65,
	ESPBox = 1, Tracers = false, TracerOrigin = 1,

	-- Aim
	TargetMode = 3,
	AimEnabled = true, UseLegitAim = true, UseSilentAim = false,
	AimMode = 1, AimPart = 3, AimWallCheck = true,
	Smoothness = 0.15, SnapAngle = 6, AutoPredict = true, PredictScale = 0.6,
	AimAtCursor = true, Prediction = 0, BulletSpeed = 0,
	ShotType = 1, FireLock = true, Priority = 1,
	AimKey = Enum.UserInputType.MouseButton2, SwitchKey = Enum.KeyCode.T,
	FOVEnabled = true, FOVRadius = 150, AimMaxDist = 0,
	SilentAimFOV = 300, SilentAimVisible = true,

	TriggerBot = false, TriggerBotAlways = false,
	TriggerFOV = 100, TriggerVisible = true, TriggerDelay = 0.05,

	-- Hitbox
	HitboxExpander = false, HitboxSize = 6, HitboxRange = 200,
	HitboxInvisible = false, HitboxIgnoreAllies = true,
	ExpandHead = true, ExpandTorso = true, ExpandUpperTorso = true, ExpandLowerTorso = false,

	-- Movimento
	WalkSpeedOn = false, WalkSpeed = 32, WalkAutoLimit = true,
	JumpOn = false, JumpPower = 100,
	Noclip = false, NoclipKey = Enum.KeyCode.V,
	Fly = false, FlyKey = Enum.KeyCode.F, FlySpeed = 60,
	AntiVoid = false,
	AntiVoidY = math.max(Workspace.FallenPartsDestroyHeight + 100, -400),
	AntiKill = false, AntiKillMargin = 200,
	AntiFling = false, AntiFlingRestore = true, AntiFlingSpeed = 160,
	InfJump = false, FullBright = false, NoFog = false,
	CamFOVOn = false, CamFOV = 90, AntiAFK = true,

	-- Invisibilidade
	SeatInvisible = false,
	SeatInvisibleX = -25.95, SeatInvisibleY = 84, SeatInvisibleZ = 3537.55,

	-- Fling
	Fling = false, FlingKey = Enum.KeyCode.G,
	FlingRange = 12, FlingPower = 3000, FlingMode2 = 2,
	FlingRepeat = 0.1,
	FlingPlayerTarget = "Nenhum",
	FlingPlayerDuration = 2.5,
	FlingPlayerSpin = 90000,
	FlingPlayerKey = Enum.KeyCode.B,

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
		bootLabel.Text = text
		bootLabel.TextColor3 = color or Color3.new(1, 1, 1)
		bootGui.Enabled = true
		if hideAfter then
			task.delay(hideAfter, function()
				if bootGui then bootGui.Enabled = false end
			end)
		end
	end)
end
bootShow("TestToolkit v71 carregando...")

--------------------------------------------------------------------
-- TEAM
--------------------------------------------------------------------
local manualAllies = setmetatable({}, { __mode = "k" })
local function isTeammate(model, plr)
	local k = plr or model
	if manualAllies[k] then return true end
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
	if hum.Health < 5 then return false end
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

local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
local filterList = {}

local function isIgnorableBlocker(hit)
	return hit.Transparency >= 0.9 or hit:FindFirstAncestorOfClass("Accessory") ~= nil
end

local function hasLineOfSight(part, model)
	if not Settings.AimWallCheck then return true end
	table.clear(filterList)
	filterList[1] = LocalPlayer.Character
	filterList[2] = Camera
	rayParams.FilterDescendantsInstances = filterList
	local origin = Camera.CFrame.Position
	local dir = part.Position - origin
	for _ = 1, 4 do
		local result = Workspace:Raycast(origin, dir, rayParams)
		if not result then return true end
		local hit = result.Instance
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

local sizeInfo = setmetatable({}, { __mode = "k" })
local function isSmall(model)
	local now = os.clock()
	local info = sizeInfo[model]
	if not info then info = { t = -1, small = false }; sizeInfo[model] = info end
	if now - info.t > 0.5 then
		local _, size = model:GetBoundingBox()
		info.small = size.Y < 4.2
		info.t = now
	end
	return info.small
end

local trackers = setmetatable({}, { __mode = "k" })
local orderBuf = {}

local function fillOrder(model, hum)
	local root = getRoot(model)
	local head = model:FindFirstChild("Head")
	local torso = model:FindFirstChild("UpperTorso") or model:FindFirstChild("Torso") or root
	local headFirst
	if Settings.AimPart == 1 then headFirst = true
	elseif Settings.AimPart == 2 then headFirst = false
	else
		local tr = trackers[model]
		local fast = tr ~= nil and tr.vel.Magnitude > 12
		headFirst = not (fast or isSmall(model) or hum.FloorMaterial == Enum.Material.Air)
	end
	local a, b, c
	if headFirst then a, b, c = head, torso, root
	else a, b, c = torso, root, head end
	table.clear(orderBuf)
	local n = 0
	if a and a:IsA("BasePart") then n += 1; orderBuf[n] = a end
	if b and b ~= a and b:IsA("BasePart") then n += 1; orderBuf[n] = b end
	if c and c ~= a and c ~= b and c:IsA("BasePart") then n += 1; orderBuf[n] = c end
	return n
end

local function getAimOrigin()
	if isMobile then
		local vp = Camera.ViewportSize
		return Vector2.new(vp.X / 2, vp.Y / 2)
	end
	return UserInputService:GetMouseLocation()
end

local function pickAim(model, hum)
	local n = fillOrder(model, hum)
	local origin = getAimOrigin()
	local best, bestD = nil, nil
	for i = 1, n do
		local p = orderBuf[i]
		if hasLineOfSight(p, model) then
			if Settings.AimPart ~= 3 then return p end
			local sp, on = Camera:WorldToViewportPoint(p.Position)
			local d = on and (Vector2.new(sp.X, sp.Y) - origin).Magnitude or math.huge
			if i == 1 then d *= 0.6 end
			if not bestD or d < bestD then best, bestD = p, d end
		end
	end
	return best
end

local candCache, candTime, candLock = {}, -1, false
local roughBuf = {}

local function candLess(a, b)
	if Settings.Priority == 2 then return a.WorldDist < b.WorldDist
	elseif Settings.Priority == 3 then return a.Health < b.Health end
	return a.Dist < b.Dist
end

local function getCandidates(force)
	local now = os.clock()
	if not force and now - candTime < 0.08 then return candCache end
	if candLock then return candCache end
	candLock = true
	candTime = now

	local origin = getAimOrigin()
	local camPos = Camera.CFrame.Position
	local fovOn = Settings.FOVEnabled
	local fovLimit = Settings.FOVRadius + 80
	local maxD = Settings.AimMaxDist

	local rough = roughBuf
	table.clear(rough)
	for _, t in ipairs(getTargets("aim")) do
		local rp = t.Root.Position
		local rsp, rOn = Camera:WorldToViewportPoint(rp)
		if rOn then
			local d2 = (Vector2.new(rsp.X, rsp.Y) - origin).Magnitude
			local wd = (rp - camPos).Magnitude
			if (not fovOn or d2 <= fovLimit) and (maxD == 0 or wd <= maxD) then
				rough[#rough + 1] = { T = t, Dist = d2, WorldDist = wd, Health = t.Humanoid.Health }
			end
		end
	end
	table.sort(rough, candLess)

	local out = {}
	for i = 1, #rough do
		if #out >= 6 then break end
		local r = rough[i]
		local t = r.T
		local part = pickAim(t.Model, t.Humanoid)
		if part then
			local sp, onScreen = Camera.WorldToViewportPoint(part.Position)
			if onScreen then
				local dist = (Vector2.new(sp.X, sp.Y) - origin).Magnitude
				if not fovOn or dist <= Settings.FOVRadius then
					out[#out + 1] = {
						Model = t.Model, Humanoid = t.Humanoid, Player = t.Player,
						Part = part, Dist = dist,
						WorldDist = (part.Position - camPos).Magnitude,
						Health = r.Health,
					}
				end
			end
		end
	end
	table.sort(out, candLess)
	candCache = out
	candLock = false
	return out
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
	local tw = TweenService:Create(
		obj,
		TweenInfo.new(t or 0.18, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out),
		props)
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
silentFovCircle.Parent = gui
corner(silentFovCircle, 9999)
stroke(silentFovCircle, Color3.fromRGB(255, 100, 255), 0.3, 1.5)

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
-- LOCAL PLAYER + NOCLIP
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
-- AIM
--------------------------------------------------------------------
local currentTarget = nil
local aimActive = false
local spectating = false
local lastLocked, lastLockedAt = nil, 0
local silentTarget = nil
local silentActive = false

local function switchTarget()
	local c = getCandidates(true)
	if #c == 0 then currentTarget = nil; return end
	table.sort(c, function(a, b) return a.WorldDist < b.WorldDist end)
	local idx = 0
	if currentTarget then
		for i, v in ipairs(c) do
			if v.Model == currentTarget.Model then idx = i; break end
		end
	end
	currentTarget = c[(idx % #c) + 1]
end

local function isAiming() return Settings.AimEnabled and (aimActive or Settings.AimMode == 3) end
local function isLegit() return Settings.UseLegitAim end
local function isSilent() return Settings.UseSilentAim end

local firing = false
UserInputService.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then firing = true end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then firing = false end
end)

local function getVelocity(model, root)
	local now = os.clock()
	local tr = trackers[model]
	local pos = root.Position
	if not tr then
		tr = { pos = pos, t = now, vel = root.AssemblyLinearVelocity, acc = Vector3.zero }
		trackers[model] = tr
		return tr.vel, tr.acc
	end
	local el = now - tr.t
	if el >= 0.05 then
		local raw = (pos - tr.pos) / el
		local old = tr.vel
		if raw.Magnitude > 400 then
			tr.vel = Vector3.zero
			tr.acc = Vector3.zero
		else
			tr.vel = old:Lerp(raw, 0.6)
			local a = (tr.vel - old) / el
			if a.Magnitude > 80 then a = a.Unit * 80 end
			tr.acc = tr.acc:Lerp(a, 0.4)
		end
		tr.pos = pos
		tr.t = now
	end
	local asm = root.AssemblyLinearVelocity
	if tr.vel.Magnitude < 1 and asm.Magnitude > 3 and asm.Magnitude < 400 then
		return asm, Vector3.zero
	end
	return tr.vel, tr.acc
end

local pingAt, pingVal = -1, 0.05
local function predictPosition(t, dt)
	local model, hum, part = t.Model, t.Humanoid, t.Part
	local root = getRoot(model)
	local pos = part.Position
	if not root then return pos end
	local vel, acc = getVelocity(model, root)
	local shot = Settings.ShotType
	local hv = Vector3.new(vel.X, 0, vel.Z)
	if hv.Magnitude < 1.5 then hv = Vector3.zero end
	local time = Settings.Prediction
	if os.clock() - pingAt > 0.5 then
		pingAt = os.clock()
		pingVal = LocalPlayer:GetNetworkPing()
	end
	if shot == 1 then
		if Settings.AutoPredict then time += pingVal * 0.5 * Settings.PredictScale end
	else
		if Settings.AutoPredict then time += (pingVal + 0.04) * Settings.PredictScale end
	end
	time = math.clamp(time, 0, 0.6)
	if time <= 0 then return pos end
	return pos + hv * time + Vector3.new(0, math.clamp(vel.Y, -500, 500) * time, 0)
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
	for _, t in ipairs(getTargets("aim")) do
		local rsp, on = Camera:WorldToViewportPoint(t.Root.Position)
		if on then
			local d = (Vector2.new(rsp.X, rsp.Y) - origin).Magnitude
			local wd = (t.Root.Position - camPos).Magnitude
			if d <= maxDist then
				if not Settings.SilentAimVisible or hasLineOfSight(t.Root, t.Model) then
					if wd < bestD then best, bestD = t, wd end
				end
			end
		end
	end
	if best and best.Model.Parent then
		local head = best.Model:FindFirstChild("Head")
		local torso = best.Model:FindFirstChild("UpperTorso") or best.Model:FindFirstChild("Torso")
		local part = head or torso or best.Model:FindFirstChild("HumanoidRootPart")
		if part then
			best.Part = part
			silentTarget = best
			silentActive = true
			return
		end
	end
	silentTarget = nil
	silentActive = false
end

local triggerBotActive = false
local lastTriggerShot = 0

local function getTriggerTarget()
	if not Settings.TriggerBot then return nil end
	if not isAiming() and not Settings.TriggerBotAlways then return nil end
	local origin = getAimOrigin()
	local camPos = Camera.CFrame.Position
	local maxFov = Settings.TriggerFOV
	local best, bestD = nil, math.huge
	for _, t in ipairs(getTargets("aim")) do
		local rsp, on = Camera:WorldToViewportPoint(t.Root.Position)
		if on then
			local d = (Vector2.new(rsp.X, rsp.Y) - origin).Magnitude
			local wd = (t.Root.Position - camPos).Magnitude
			if d <= maxFov then
				if not Settings.TriggerVisible or hasLineOfSight(t.Root, t.Model) then
					if wd < bestD then best, bestD = t, wd end
				end
			end
		end
	end
	return best
end

local function simulateClick()
	local mouse = LocalPlayer:GetMouse()
	if not mouse then return end
	pcall(function()
		if mouse1click then mouse1click(); return end
	end)
	pcall(function()
		local vim = game:GetService("VirtualInputManager")
		if vim then
			vim:SendMouseButtonEvent(mouse.X, mouse.Y, 0, true, game, 1)
			task.wait(0.02)
			vim:SendMouseButtonEvent(mouse.X, mouse.Y, 0, false, game, 1)
		end
	end)
end

local function updateTriggerBot()
	if not Settings.TriggerBot then triggerBotActive = false; return end
	local now = os.clock()
	if now - lastTriggerShot < Settings.TriggerDelay then return end
	local target = getTriggerTarget()
	if not target then triggerBotActive = false; return end
	triggerBotActive = true
	lastTriggerShot = now
	simulateClick()
end

RunService.RenderStepped:Connect(function()
	pcall(function()
		updateSilentTarget()
		updateTriggerBot()
	end)
end)

RunService:BindToRenderStep("TTUniversalAim", Enum.RenderPriority.Camera.Value + 1, function(dt)
	if spectating then currentTarget = nil; lockMarker.Visible = false; return end

	local now = os.clock()
	local myRoot = getLocalRoot()
	local isFirstPerson = myRoot and (Camera.CFrame.Position - myRoot.Position).Magnitude < 1.5
	local aimOrigin
	if isFirstPerson then
		aimOrigin = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
	elseif isMobile then
		local vp = Camera.ViewportSize
		aimOrigin = Vector2.new(vp.X / 2, vp.Y / 2)
	else
		aimOrigin = UserInputService:GetMouseLocation()
	end

	if isLegit() and isAiming() then
		if currentTarget then
			local m, hum = currentTarget.Model, currentTarget.Humanoid
			if not m.Parent or not isAlive(hum) then currentTarget = nil
			elseif Settings.TeamCheck and isTeammate(m, currentTarget.Player) then currentTarget = nil end
		end

		if not currentTarget then
			currentTarget = getCandidates()[1]
			if currentTarget then
				currentTarget.LastSeen = now
				trackers[currentTarget.Model] = nil
			end
		end

		if currentTarget then
			if now - (currentTarget.CheckAt or 0) >= 0.05 then
				currentTarget.CheckAt = now
				local m, hum = currentTarget.Model, currentTarget.Humanoid
				local cur = currentTarget.Part
				local p2
				if cur and cur.Parent and now - (currentTarget.PickAt or 0) < 0.4 and hasLineOfSight(cur, m) then
					p2 = cur
				else
					p2 = pickAim(m, hum)
					currentTarget.PickAt = now
				end
				if p2 then
					currentTarget.Part = p2
					currentTarget.LastSeen = now
				elseif now - (currentTarget.LastSeen or 0) > 0.25 then
					currentTarget = nil
				end
			end
		end

		if currentTarget then
			local part = currentTarget.Part
			if not part or not part.Parent then
				currentTarget = nil
			end
		end

		if currentTarget then
			lastLocked = currentTarget.Model
			lastLockedAt = now
			local goalPos = predictPosition(currentTarget, dt)
			local camCF = Camera.CFrame
			local camPos = camCF.Position
			local toGoal = goalPos - camPos
			if toGoal.Magnitude > 0.01 then
				local goalDir = toGoal.Unit
				local curDir = camCF.LookVector
				if not isFirstPerson and Settings.AimAtCursor then
					local vp = Camera.ViewportSize
					local mp = getAimOrigin()
					local tanY = math.tan(math.rad(Camera.FieldOfView) / 2)
					local lx = ((mp.X / vp.X) * 2 - 1) * tanY * (vp.X / vp.Y)
					local ly = (1 - (mp.Y / vp.Y) * 2) * tanY
					curDir = camCF:VectorToWorldSpace(Vector3.new(lx, ly, -1).Unit)
				end
				local angleRad = math.acos(math.clamp(curDir:Dot(goalDir), -1, 1))
				local angle = math.deg(angleRad)
				local alpha = 1 - math.pow(Settings.Smoothness, dt * 60)
				alpha += (1 - alpha) * math.clamp(angle / 25, 0, 1) * 0.5
				if angle <= Settings.SnapAngle or (firing and Settings.FireLock) then alpha = 1 end
				local axis = curDir:Cross(goalDir)
				if axis.Magnitude > 1e-4 then
					local rot = CFrame.fromAxisAngle(axis.Unit, angleRad * alpha)
					Camera.CFrame = CFrame.new(camPos) * rot * camCF.Rotation
				end
			end
			local sp, onScreen = Camera:WorldToViewportPoint(goalPos)
			lockMarker.Visible = onScreen
			if onScreen then
				lockMarker.Position = UDim2.fromOffset(sp.X, sp.Y)
				lockMarker.BackgroundColor3 = Color3.new(0, 0, 0)
				local sz = 24 + math.sin(os.clock() * 9) * 3
				lockMarker.Size = UDim2.fromOffset(sz, sz)
			end
		end
	end

	if isSilent() and silentTarget and silentTarget.Part and silentTarget.Part.Parent then
		local sp, onScreen = Camera.WorldToViewportPoint(silentTarget.Part.Position)
		if onScreen then
			lockMarker.Visible = true
			lockMarker.Position = UDim2.fromOffset(sp.X, sp.Y)
			lockMarker.BackgroundColor3 = Color3.fromRGB(255, 100, 255)
			local sz = 20 + math.sin(os.clock() * 12) * 2
			lockMarker.Size = UDim2.fromOffset(sz, sz)
		end
	end

	if triggerBotActive then
		lockMarker.Visible = true
		lockMarker.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
	end

	if not currentTarget
		and not (silentActive and silentTarget and silentTarget.Part and silentTarget.Part.Parent)
		and not triggerBotActive then
		lockMarker.Visible = false
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
			Size = part.Size, CanCollide = part.CanCollide, Massless = part.Massless,
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
						part.Transparency = Settings.HitboxInvisible and 1 or 0.5
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
		if Settings.HitboxIgnoreAllies and isTeammate(char, player) then return false end
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
local flyObjs = nil
local flyPlat = false
local flyCur = 0
local FLY_RAMP = 500
local flyLastPos = nil
local flyCooldown = 0
local antiKillHold = 0

local function destroyFly()
	flyCur, flyLastPos = 0, nil
	if flyObjs then
		for _, o in pairs(flyObjs) do
			if o and o.Parent then o:Destroy() end
		end
		flyObjs = nil
	end
	if flyPlat then
		local hum = getLocalHumanoid()
		if hum then hum.PlatformStand = false end
		flyPlat = false
	end
end

local function ensureFly(root)
	if flyObjs and flyObjs.Att and flyObjs.Att.Parent == root then return flyObjs end
	destroyFly()
	local att = Instance.new("Attachment")
	att.Parent = root
	local lv = Instance.new("LinearVelocity")
	lv.Attachment0 = att
	lv.RelativeTo = Enum.ActuatorRelativeTo.World
	lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
	lv.MaxForce = math.huge
	lv.VectorVelocity = Vector3.zero
	lv.Parent = root
	local ao = Instance.new("AlignOrientation")
	ao.Mode = Enum.OrientationAlignmentMode.OneAttachment
	ao.Attachment0 = att
	ao.RigidityEnabled = false
	ao.MaxTorque = math.huge
	ao.Responsiveness = 40
	ao.Parent = root
	flyObjs = { Att = att, LV = lv, AO = ao }
	return flyObjs
end

local function updateFly(hum, root, dt)
	local f = ensureFly(root)
	if os.clock() < antiKillHold then
		f.LV.VectorVelocity = Vector3.zero
		return
	end
	if not hum.PlatformStand then hum.PlatformStand = true end
	flyPlat = true
	local cam = Workspace.CurrentCamera
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
	f.LV.VectorVelocity = dir * flyCur
	f.AO.CFrame = CFrame.lookAt(Vector3.zero, flatLook)
end

--------------------------------------------------------------------
-- FLING
--------------------------------------------------------------------
local Fling = {}
do
	local lastFling = 0
	local savedCollide = setmetatable({}, { __mode = "k" })

	local function disableBotCollide(model)
		for _, part in ipairs(model:GetDescendants()) do
			if part:IsA("BasePart") then
				if savedCollide[part] == nil then savedCollide[part] = part.CanCollide end
				part.CanCollide = false
			end
		end
	end

	local function doFling(targetRoot, myRoot, power, mode)
		if not targetRoot or not targetRoot.Parent then return end
		if not myRoot or not myRoot.Parent then return end
		local hum = targetRoot.Parent:FindFirstChildOfClass("Humanoid")
		if hum and hum.Health <= 0 then return end
		disableBotCollide(targetRoot.Parent)
		local myPos = myRoot.Position
		local tPos = targetRoot.Position
		local dir = Vector3.new(tPos.X - myPos.X, 0, tPos.Z - myPos.Z)
		if dir.Magnitude < 0.05 then dir = Vector3.new(0, 0, 1)
		else dir = dir.Unit end
		local linVel, angVel
		if mode == 1 then
			linVel = dir * power * 50 + Vector3.new(0, power * 10, 0)
			angVel = Vector3.new(power * 50, power * 50, power * 50)
		else
			linVel = Vector3.new(9e7, 9e7, 9e7)
			angVel = Vector3.new(9e8, 9e8, 9e8)
		end
		pcall(function()
			targetRoot.AssemblyLinearVelocity = linVel
			targetRoot.AssemblyAngularVelocity = angVel
		end)
	end

	local function doFlingPlayer(target)
		if not target or target == LocalPlayer then return false end
		local targetRoot = getRoot(target.Character)
		local myRoot = getLocalRoot()
		if not targetRoot or not myRoot then return false end
		local home = myRoot.CFrame
		local spin = Instance.new("BodyAngularVelocity")
		spin.MaxTorque = Vector3.one * math.huge
		spin.AngularVelocity = Vector3.new(0, Settings.FlingPlayerSpin or 9e4, 0)
		spin.Parent = myRoot
		local savedNoclip = {}
		local char = LocalPlayer.Character
		if char then
			for _, part in ipairs(char:GetChildren()) do
				if part:IsA("BasePart") then
					savedNoclip[part] = part.CanCollide
					part.CanCollide = false
				end
			end
		end
		local duration = Settings.FlingPlayerDuration or 2.5
		local started = os.clock()
		while os.clock() - started < duration do
			local currentTarget = getRoot(target.Character)
			if not currentTarget or not myRoot.Parent then break end
			myRoot.CFrame = currentTarget.CFrame
			myRoot.AssemblyLinearVelocity = Vector3.zero
			RunService.Heartbeat:Wait()
		end
		spin:Destroy()
		if myRoot.Parent then
			myRoot.AssemblyAngularVelocity = Vector3.zero
			myRoot.AssemblyLinearVelocity = Vector3.zero
			myRoot.CFrame = home
		end
		for part, canCollide in pairs(savedNoclip) do
			if part.Parent then part.CanCollide = canCollide end
		end
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
			if plr.DisplayName == name or plr.Name == name then
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
			notify(ok and ("Fling: " .. target.DisplayName) or "Fling: falhou", ok and "on" or "off")
		end)
	end

	Fling.step = function()
		if not Settings.Fling then return end
		local now = os.clock()
		if now - lastFling < Settings.FlingRepeat then return end
		lastFling = now
		local myHum, myRoot = getLocalHumanoid(), getLocalRoot()
		if not myHum or myHum.Health <= 0 then return end
		local myPos = myRoot.Position
		local range = Settings.FlingRange
		for model, hum in pairs(humanoids) do
			if model ~= LocalPlayer.Character and model.Parent and hum.Parent and hum.Health > 0 then
				local plr = Players:GetPlayerFromCharacter(model)
				if not plr then
					local r = getRoot(model)
					if r and (r.Position - myPos).Magnitude <= range then
						doFling(r, myRoot, Settings.FlingPower, Settings.FlingMode2 or 2)
					end
				end
			end
		end
	end
end

--------------------------------------------------------------------
-- HEARTBEAT
--------------------------------------------------------------------
local defaultWalkSpeed = 16
local defaultJumpPower, defaultUseJumpPower = 50, true
local lastSafe = nil
local ANTIFLING_MAX_SPIN = 40
local lastFlingNotify = 0
local stableOld, stableNew = nil, nil
local stableTimer = 0
local wsActive, jpActive = false, false
local SPEED_RAMP_BASE = 240
local wsCurrent = 16
local walkLastPos = nil
local walkCooldown = 0
local walkUserGoal = nil
local walkPulledAt = 0
local walkProbeAt = 0

LocalPlayer.CharacterAdded:Connect(function()
	lastSafe = nil
	wsActive, jpActive = false, false
end)

local function walkGuard(hum, root, dt)
	local pos = root.Position
	local last = walkLastPos
	walkLastPos = pos
	if not Settings.WalkAutoLimit or not last or Settings.Fly then return end
	local now = os.clock()
	if now < antiKillHold + 0.5 then return end
	local move = hum.MoveDirection
	local flatMove = Vector3.new(move.X, 0, move.Z)
	if flatMove.Magnitude < 0.1 then return end
	flatMove = flatMove.Unit
	local disp = Vector3.new(pos.X - last.X, 0, pos.Z - last.Z)
	local back = -disp:Dot(flatMove)
	local limit = math.max(6, wsCurrent * dt * 2.5)
	if now >= walkCooldown and back > limit and wsCurrent > 20 then
		walkUserGoal = walkUserGoal or Settings.WalkSpeed
		walkCooldown = now + 1.2
		walkPulledAt = now
		local new = math.max(16, math.floor(wsCurrent * 0.7))
		wsCurrent = new
		Settings.WalkSpeed = math.max(16, new)
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	elseif walkUserGoal and now - walkPulledAt > 6 and now - walkProbeAt > 3
		and Settings.WalkSpeed < walkUserGoal then
		walkProbeAt = now
		Settings.WalkSpeed = math.min(walkUserGoal, math.ceil(Settings.WalkSpeed * 1.12))
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	end
end

RunService.Heartbeat:Connect(function(dt)
	pcall(function()
		local hum = getLocalHumanoid()
		if not hum then return end
		local root = getLocalRoot()

		if Settings.WalkSpeedOn then
			if not wsActive then
				defaultWalkSpeed = hum.WalkSpeed
				wsCurrent = hum.WalkSpeed
				wsActive = true
				walkLastPos = nil
				walkUserGoal = nil
			end
			if walkUserGoal and Settings.WalkSpeed > walkUserGoal then
				walkUserGoal = Settings.WalkSpeed
			end
			local goal = Settings.WalkSpeed
			if wsCurrent < goal then
				wsCurrent = math.min(goal, wsCurrent + math.max(SPEED_RAMP_BASE, goal * 2) * dt)
			else
				wsCurrent = goal
			end
			if hum.WalkSpeed ~= wsCurrent then hum.WalkSpeed = wsCurrent end
			if root and hum.Health > 0 then walkGuard(hum, root, dt) end
		elseif wsActive then
			hum.WalkSpeed = defaultWalkSpeed
			wsActive = false
			walkLastPos = nil
			walkUserGoal = nil
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
		elseif flyObjs or flyPlat then
			destroyFly()
		end

		if Settings.AntiFling and root and hum.Health > 0 then
			local v = root.AssemblyLinearVelocity
			local horizontal = Vector3.new(v.X, 0, v.Z).Magnitude
			local limit = Settings.AntiFlingSpeed
			if Settings.Fly then limit = math.max(limit, Settings.FlySpeed * 2.5) end
			if Settings.WalkSpeedOn then limit = math.max(limit, Settings.WalkSpeed * 3) end
			local fastOwn = Settings.WalkSpeedOn or Settings.Fly
			local spinLimit = fastOwn and 120 or ANTIFLING_MAX_SPIN
			local flung = horizontal > limit or v.Y > 80
				or root.AssemblyAngularVelocity.Magnitude > spinLimit
			if flung then
				if Settings.AntiFlingRestore and stableOld and not fastOwn then
					LocalPlayer.Character:PivotTo(stableOld)
				end
				root.AssemblyLinearVelocity = Vector3.zero
				root.AssemblyAngularVelocity = Vector3.zero
				local state = hum:GetState()
				if state == Enum.HumanoidStateType.FallingDown
					or state == Enum.HumanoidStateType.Ragdoll
					or state == Enum.HumanoidStateType.Physics then
					if not Settings.Fly then hum:ChangeState(Enum.HumanoidStateType.GettingUp) end
				end
				if not Settings.Fly then hum.PlatformStand = false end
				local now = os.clock()
				if now - lastFlingNotify > 2 then
					lastFlingNotify = now
					notify("Antifling: bloqueado", "info")
				end
			else
				stableTimer += dt
				if stableTimer >= 0.25 then
					stableTimer = 0
					stableOld = stableNew or root.CFrame
					stableNew = root.CFrame
				end
			end
		else
			stableOld, stableNew = nil, nil
			stableTimer = 0
		end

		if root and hum.Health > 0 then
			if root.Position.Y < Settings.AntiVoidY and Settings.AntiVoid then
				local target = lastSafe
				if not target then
					local spawn = Workspace:FindFirstChildWhichIsA("SpawnLocation", true)
					target = spawn and (spawn.CFrame + Vector3.new(0, 5, 0)) or CFrame.new(0, 50, 0)
				end
				LocalPlayer.Character:PivotTo(target)
				root.AssemblyLinearVelocity = Vector3.zero
				root.AssemblyAngularVelocity = Vector3.zero
			elseif hum.FloorMaterial ~= Enum.Material.Air then
				lastSafe = root.CFrame + Vector3.new(0, 3, 0)
			end
		end

		Fling.step()
	end)
end)

--------------------------------------------------------------------
-- ANTI KILL
--------------------------------------------------------------------
local AntiKill = {}
do
	local bounds = nil
	local scanning = false
	local history = {}
	local histTimer, lastRescue, notifyAt, nextFall, nextOut = 0, 0, 0, 0, 0
	local rpChar = nil
	local DOWN = Vector3.new(0, -5000, 0)
	local rp = RaycastParams.new()
	rp.FilterType = Enum.RaycastFilterType.Exclude
	rp.RespectCanCollide = true
	local rp2 = RaycastParams.new()
	rp2.FilterType = Enum.RaycastFilterType.Exclude
	local BAD_NAMES = { "kill", "lava", "death", "void", "acid", "damage", "lethal", "magma", "spike", "toxic" }

	local function isBadGround(inst)
		local n = string.lower(inst.Name)
		for _, w in ipairs(BAD_NAMES) do
			if string.find(n, w, 1, true) then return true end
		end
		return false
	end

	local function pct(arr, p)
		return arr[math.clamp(math.floor(#arr * p) + 1, 1, #arr)]
	end

	local function scanMap()
		if scanning then return end
		scanning = true
		task.spawn(function()
			local xs, zs, n = {}, {}, 0
			for _, d in ipairs(Workspace:GetDescendants()) do
				if d:IsA("BasePart") and d ~= Workspace.Terrain and d.Anchored and d.CanCollide
					and d.Size.Magnitude < 2000 and not isBadGround(d) then
					xs[#xs + 1] = d.Position.X
					zs[#zs + 1] = d.Position.Z
				end
				n += 1
				if n % 400 == 0 then task.wait() end
			end
			if #xs >= 10 then
				table.sort(xs); table.sort(zs)
				bounds = {
					minX = pct(xs, 0.01) - 60, maxX = pct(xs, 0.99) + 60,
					minZ = pct(zs, 0.01) - 60, maxZ = pct(zs, 0.99) + 60,
				}
			end
			scanning = false
		end)
	end

	AntiKill.recalc = function() bounds = nil; scanMap() end

	local function isInside(p, m)
		return p.X >= bounds.minX - m and p.X <= bounds.maxX + m
			and p.Z >= bounds.minZ - m and p.Z <= bounds.maxZ + m
	end

	local function noGround(p)
		return Workspace:Raycast(p, DOWN, rp) == nil and Workspace:Raycast(p, DOWN, rp2) == nil
	end

	local function safeTarget()
		local now = os.clock()
		local idx = nil
		for i = #history, 1, -1 do
			if now - history[i].t >= 1 then idx = i; break end
		end
		idx = idx or (#history > 0 and 1 or nil)
		if idx then
			local cf = history[idx].cf
			for i = #history, idx + 1, -1 do history[i] = nil end
			return cf
		end
		local spawn = Workspace:FindFirstChildWhichIsA("SpawnLocation", true)
		return spawn and (spawn.CFrame + Vector3.new(0, 5, 0)) or CFrame.new(0, 50, 0)
	end

	local function rescue(root)
		local now = os.clock()
		lastRescue = now
		antiKillHold = now + 0.5
		LocalPlayer.Character:PivotTo(safeTarget())
		root.AssemblyLinearVelocity = Vector3.zero
		root.AssemblyAngularVelocity = Vector3.zero
		if flyObjs and flyObjs.LV then flyObjs.LV.VectorVelocity = Vector3.zero end
		if now - notifyAt > 2 then
			notifyAt = now
			notify("Anti Kill: trazido de volta", "info")
		end
	end

	LocalPlayer.CharacterAdded:Connect(function()
		table.clear(history)
		rpChar = nil
	end)

	RunService.Heartbeat:Connect(function()
		pcall(function()
			if not Settings.AntiKill then
				if next(history) then table.clear(history) end
				return
			end
			local hum, root = getLocalHumanoid(), getLocalRoot()
			if not hum or not root or hum.Health <= 0 then return end
			local char = LocalPlayer.Character
			if rpChar ~= char then
				rpChar = char
				rp.FilterDescendantsInstances = { char }
				rp2.FilterDescendantsInstances = { char }
			end
			if not bounds and not scanning then scanMap() end
			local now = os.clock()
			local pos, vel = root.Position, root.AssemblyLinearVelocity
			local airborne = hum.FloorMaterial == Enum.Material.Air
			local danger, why = false, nil
			local destroyY = Workspace.FallenPartsDestroyHeight
			if pos.Y < destroyY + 80 then
				danger, why = true, "altura do vazio"
			elseif vel.Y < -40 and pos.Y + vel.Y * 0.4 < destroyY + 80 and noGround(pos) then
				danger, why = true, "queda no vazio"
			end
			if not danger and not Settings.Fly then
				if airborne and now >= nextFall and vel.Y < -40 then
					nextFall = now + 0.05
					if noGround(pos) then danger, why = true, "queda sem chão" end
				end
				local m = Settings.AntiKillMargin
				if not danger and bounds and m > 0 and now >= nextOut then
					nextOut = now + 0.1
					local fut = pos + Vector3.new(vel.X, 0, vel.Z) * 0.25
					if not isInside(fut, m) and noGround(fut) and noGround(pos) then
						danger, why = true, "borda do mapa"
					end
				end
			end
			if danger then
				if now - lastRescue >= 0.1 then rescue(root) end
				return
			end
			if not airborne and now - histTimer >= 0.4 and vel.Y > -30 then
				histTimer = now
				local hit = Workspace:Raycast(pos, Vector3.new(0, -12, 0), rp)
				if hit and not isBadGround(hit.Instance) and (not bounds or isInside(pos, 0)) then
					history[#history + 1] = { cf = root.CFrame + Vector3.new(0, 3, 0), t = now }
					if #history > 10 then table.remove(history, 1) end
				end
			end
		end)
	end)
end

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

--------------------------------------------------------------------
-- ESP
--------------------------------------------------------------------
local espRoot = Instance.new("Frame")
espRoot.Name = "ESP"
espRoot.Size = UDim2.fromScale(1, 1)
espRoot.BackgroundTransparency = 1
espRoot.ZIndex = 1
espRoot.Parent = gui
_G.TT_espRoot = espRoot

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

	o.Box = mkFrame(holder)
	o.Box.BackgroundTransparency = 1
	o.BoxStroke = stroke(o.Box, Color3.new(1, 1, 1), 0, 1.5)
	o.Corners = {}
	for i = 1, 8 do o.Corners[i] = mkFrame(holder) end
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
	local boxW = boxH * 0.55
	local x, y = px(center.X - boxW / 2), px(top.Y)
	boxW, boxH = px(boxW), px(boxH)

	o.Holder.Visible = true

	local boxMode = Settings.ESPBox
	o.Box.Visible = boxMode == 2
	if boxMode == 2 then
		o.Box.Position = UDim2.fromOffset(x, y)
		o.Box.Size = UDim2.fromOffset(boxW, boxH)
		o.BoxStroke.Color = color
	end
	if boxMode == 3 then
		local L, th = math.max(4, px(boxW * 0.28)), 2
		local r, b = x + boxW, y + boxH
		local specs = {
			{ x, y, L, th }, { x, y, th, L },
			{ r - L, y, L, th }, { r - th, y, th, L },
			{ x, b - th, L, th }, { x, b - L, th, L },
			{ r - L, b - th, L, th }, { r - th, b - L, th, L },
		}
		for i = 1, 8 do
			local f, s = o.Corners[i], specs[i]
			f.BackgroundColor3 = color
			f.Position = UDim2.fromOffset(s[1], s[2])
			f.Size = UDim2.fromOffset(s[3], s[4])
			f.Visible = true
		end
	else
		for i = 1, 8 do o.Corners[i].Visible = false end
	end

	if Settings.ShowHealth then
		local hum = t.Humanoid
		local frac = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
		o.HpBg.Visible = true
		o.HpBg.Position = UDim2.fromOffset(x - 7, y)
		o.HpBg.Size = UDim2.fromOffset(4, boxH)
		o.HpFill.Size = UDim2.new(1, 0, frac, 0)
		o.HpFill.BackgroundColor3 = Color3.fromHSV(frac * 0.33, 0.9, 1)
	else
		o.HpBg.Visible = false
	end

	if Settings.ShowNames then
		local plr = t.Player
		o.Name.Visible = true
		o.Name.Text = (plr and plr.DisplayName or model.Name) .. "  [" .. math.floor(dist) .. "m]"
		o.Name.TextColor3 = color
		o.Name.Position = UDim2.fromOffset(px(center.X), y - 2)
	else
		o.Name.Visible = false
	end

	if Settings.Tracers and center.Z > 0 then
		local p1
		if Settings.TracerOrigin == 1 then p1 = Vector2.new(vp.X / 2, vp.Y)
		elseif Settings.TracerOrigin == 2 then p1 = vp / 2
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
-- RENDER PRINCIPAL (FOV + ESP)
--------------------------------------------------------------------
RunService:BindToRenderStep("TTUniversalVisuals", Enum.RenderPriority.Camera.Value + 2, function(dt)
	pcall(function()
		local aimOrigin = getAimOrigin()
		local showFov = Settings.FOVEnabled and Settings.AimEnabled and isLegit()
		fovCircle.Visible = showFov
		if showFov then
			fovCircle.Position = UDim2.fromOffset(aimOrigin.X, aimOrigin.Y)
			local d = Settings.FOVRadius * 2
			fovCircle.Size = UDim2.fromOffset(d, d)
		end
		local showSilent = Settings.AimEnabled and isSilent()
		silentFovCircle.Visible = showSilent
		if showSilent then
			silentFovCircle.Position = UDim2.fromOffset(aimOrigin.X, aimOrigin.Y)
			local d = Settings.SilentAimFOV * 2
			silentFovCircle.Size = UDim2.fromOffset(d, d)
		end
		local showTrig = Settings.TriggerBot
		triggerFovCircle.Visible = showTrig
		if showTrig then
			triggerFovCircle.Position = UDim2.fromOffset(aimOrigin.X, aimOrigin.Y)
			local d = math.max(Settings.TriggerFOV * 2, 4)
			triggerFovCircle.Size = UDim2.fromOffset(d, d)
		end
		if Settings.CamFOVOn then
			if not camFovDefault then camFovDefault = Camera.FieldOfView end
			if Camera.FieldOfView ~= Settings.CamFOV then Camera.FieldOfView = Settings.CamFOV end
		elseif camFovDefault then
			Camera.FieldOfView = camFovDefault
			camFovDefault = nil
		end
		updateEsp(os.clock())
	end)
end)

--------------------------------------------------------------------
-- INTERFACE
--------------------------------------------------------------------
local menuOpen = false
local refreshers = {}
local uiRefresh

local function buildUI()

local TAB_TOTAL = 8

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
title.Text = "Test Toolkit v71"
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
	l.Text = text
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
		d.Text = desc
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
	l.Text = string.upper(text)
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
	l.Text = text
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
		notify(text .. (Settings[key] and ": ligado" or ": desligado"), Settings[key] and "on" or "off")
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
	local function render() l.Text = text .. ": " .. options[Settings[key]] end
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
	local function render() l.Text = text .. ": [" .. Settings[key].Name .. "]" end
	render()
	row.MouseButton1Click:Connect(function()
		l.Text = text .. ": pressione uma tecla..."
		rebinding = {
			mouse = allowMouse,
			apply = function(bind)
				if bind then
					Settings[key] = bind
					notify(text .. ": " .. bind.Name, "info")
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
			if plr ~= LocalPlayer then list[#list + 1] = plr.DisplayName end
		end
		return list
	end
	local function closeDropdown()
		if dropdownFrame then dropdownFrame:Destroy(); dropdownFrame = nil end
	end
	local function render()
		l.Text = text .. ": " .. (Settings[key] or "Nenhum")
	end
	render()
	local function openDropdown()
		if dropdownFrame then closeDropdown(); return end
		local list = getList()
		local h = math.min(#list * 28, 240)
		dropdownFrame = Instance.new("Frame")
		dropdownFrame.Size = UDim2.new(1, -24, 0, h)
		dropdownFrame.Position = UDim2.new(0, 12, 1, 4)
		dropdownFrame.BackgroundColor3 = Theme.Bg
		dropdownFrame.BorderSizePixel = 0
		dropdownFrame.ZIndex = 100
		dropdownFrame.Parent = row
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
			btn.Text = name
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
addToggle("Silent Aim", "UseSilentAim", "Redireciona o tiro")
addCycle("Modo", "AimMode", { "Segurar", "Alternar", "Automático" }, "Como ativa", function() aimActive = false end)
addCycle("Parte do corpo", "AimPart", { "Cabeça", "Corpo", "Automático" }, "Parte alvo")
addCycle("Prioridade", "Priority", { "Perto do cursor", "Perto de mim", "Menos vida" }, "Ordem")
addCycle("Alvos", "TargetMode", { "Jogadores", "Bots", "Ambos" }, "Quem mirar")
addToggle("Ignorar equipe", "TeamCheck", "Não mira aliados")
addToggle("Checar parede", "AimWallCheck", "Só mira se enxergar")
addSection("Legit Aim")
addSlider("Suavidade", "Smoothness", 0.01, 0.95, 0.01, 2, "Menor = mais rápida")
addSlider("Ângulo de trava", "SnapAngle", 0, 30, 1, 0, "Cola abaixo deste")
addToggle("Trava ao atirar", "FireLock", "Sem suavização ao atirar")
addToggle("Mirar no cursor", "AimAtCursor", "PC: alvo sob o cursor")
addSection("Silent Aim")
addSlider("FOV do Silent", "SilentAimFOV", 50, 800, 10, 0, "Raio")
addToggle("Só visíveis", "SilentAimVisible", "Só atira com LOS")
addSection("Trigger Bot")
addToggle("Trigger Bot", "TriggerBot", "Atira automaticamente")
addToggle("Sempre ativo", "TriggerBotAlways", "Sem precisar segurar")
addSlider("FOV do Trigger", "TriggerFOV", 5, 400, 1, 0, "5 = centro")
addSlider("Delay", "TriggerDelay", 0.01, 1, 0.01, 2, "Entre tiros")
addSection("Tiro e previsão")
addCycle("Tipo de tiro", "ShotType", { "Instantâneo", "Projétil", "Automático" }, "Método")
addToggle("Previsão automática", "AutoPredict", "Compensa ping")
addSlider("Força da previsão", "PredictScale", 0, 2, 0.05, 2, "Ajuste")
addSection("FOV do Legit")
addToggle("Limitar pelo FOV", "FOVEnabled", "Só dentro do círculo")
addSlider("Raio do FOV", "FOVRadius", 20, 600, 5, 0, "Pixels")
addSlider("Distância máxima", "AimMaxDist", 0, 2000, 50, 0, "0 = sem limite")
addSection("Teclas (PC)")
addKeybind("Tecla da mira", "AimKey", "Ativa a mira", true)
addKeybind("Trocar de alvo", "SwitchKey", "Próximo inimigo")

newPage("hitbox", "Hitbox")
addSection("Hitbox Expander")
addToggle("Hitbox Expander", "HitboxExpander", "Aumenta a hitbox", function()
	if not Settings.HitboxExpander then pcall(HitboxExpander.restoreAll) end
end)
addToggle("Invisível", "HitboxInvisible", "Hitbox transparente")
addToggle("Ignorar aliados", "HitboxIgnoreAllies", "Não expande aliados")
addSlider("Tamanho", "HitboxSize", 5, 30, 1, 0, "Studs")
addSlider("Alcance", "HitboxRange", 10, 1000, 10, 0, "Distância máxima")
addSection("Partes")
addToggle("Cabeça", "ExpandHead", "Expande a cabeça")
addToggle("Torso (R6)", "ExpandTorso", "")
addToggle("UpperTorso (R15)", "ExpandUpperTorso", "")
addToggle("LowerTorso (R15)", "ExpandLowerTorso", "")

newPage("esp", "ESP")
addSection("ESP")
addToggle("ESP", "ESP", "Mostra alvos")
addToggle("Ver através de paredes", "Wallhack", "Contorno atrás de objetos")
addToggle("Contorno colorido", "ESPHighlight", "Mais pesado")
addToggle("Cor arco-íris", "Rainbow", "Cor animada")
addCycle("Caixa", "ESPBox", { "Desligada", "Completa", "Cantos" }, "Moldura")
addToggle("Nomes e distância", "ShowNames", "Texto acima")
addToggle("Barra de vida", "ShowHealth", "Barra verde/vermelha")
addSection("Linhas (tracers)")
addToggle("Linhas até os alvos", "Tracers", "Linha da tela")
addCycle("Origem da linha", "TracerOrigin", { "Baixo", "Centro", "Cursor" }, "De onde sai")
addSection("Limites")
addSlider("Distância máxima", "ESPMaxDist", 0, 3000, 50, 0, "0 = sem limite")
addSlider("Máx. de alvos", "ESPMaxTargets", 1, 30, 1, 0, "Menos = mais FPS")
addSlider("Transparência", "ESPFillTrans", 0, 1, 0.05, 2, "0 = sólido")

newPage("player", "Jogador")
addSection("Movimento")
addToggle("Velocidade", "WalkSpeedOn", "Muda WalkSpeed")
addSlider("Valor da velocidade", "WalkSpeed", 16, 5000, 5, 0, "Padrão 16")
addToggle("Velocidade segura (auto)", "WalkAutoLimit", "Anti-puxão")
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
addCycle("Tipo", "FlingMode2", { "Empurrar", "Void" }, "Tipo de arremesso")
addSlider("Alcance (bots)", "FlingRange", 3, 30, 1, 0, "Studs")
addSlider("Força", "FlingPower", 500, 5000, 100, 0, "Força")
addSlider("Intervalo", "FlingRepeat", 0.05, 1, 0.05, 2, "Entre aplicações")
addSection("Fling Player")
addInfo("Escolha o player e clique em Fling Player.", 30)
addDropdown("FlingPlayerTarget", "Escolher Player", "Alvo do fling")
addButton("Fling Player", "Arremessa o player selecionado (ou tecla B)", function()
	Fling.flingSelected()
end)
addSlider("Duração (segundos)", "FlingPlayerDuration", 0.5, 10, 0.5, 1, "Tempo preso")
addSlider("Força de giro", "FlingPlayerSpin", 1000, 500000, 1000, 0, "BodyAngularVelocity")
addSection("Teclas (PC)")
addKeybind("Tecla do Fling Bots", "FlingKey", "Liga/desliga Fling")
addKeybind("Tecla do Fling Player", "FlingPlayerKey", "Arremessa o player selecionado")

newPage("security", "Segurança")
addSection("Proteção contra morte")
addToggle("Anti Void", "AntiVoid", "Volta se cair no vazio")
addToggle("Anti Kill", "AntiKill", "Não deixa morrer no vazio")
addSlider("Folga do Anti Kill", "AntiKillMargin", 0, 1000, 25, 0, "Distância")
addButton("Recalcular limites", "Use após o mapa carregar", function() AntiKill.recalc() end)
addSection("Proteção contra arremesso")
addToggle("Anti Fling", "AntiFling", "Bloqueia arremessos")
addToggle("Voltar ao ponto estável", "AntiFlingRestore", "Retorna a posição")
addSlider("Limite", "AntiFlingSpeed", 50, 500, 10, 0, "Velocidade")
addSection("Proteção contra kick")
addToggle("Anti AFK", "AntiAFK", "Evita kick")

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
	b.Text = label
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
	if Settings.AimMode == 1 then aimActive = true
	else aimActive = not aimActive end
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

-- KEYBIND (dentro do buildUI, único lugar onde MENU_KEY é tratado)
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
-- BUILD + HUD
--------------------------------------------------------------------
local okUI, errUI = pcall(buildUI)
if okUI then
	local fpsFrames, fpsAcc, hudAcc = 0, 0, 0
	local lastFps = 60
	local hud = Instance.new("TextLabel")
	hud.Name = "HUD"
	hud.Position = UDim2.fromOffset(Settings.HUDX, Settings.HUDY)
	hud.Size = UDim2.fromOffset(700, 20)
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

	local hudDrag, hudDragStart, hudDragFrom = false, nil, nil
	hud.InputBegan:Connect(function(input)
		if Settings.HUDEdit and (input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch) then
			hudDrag = true
			hudDragStart = input.Position
			hudDragFrom = Vector2.new(Settings.HUDX, Settings.HUDY)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if hudDrag and (input.UserInputType == Enum.UserInputType.MouseMovement
			or input.UserInputType == Enum.UserInputType.Touch) then
			local d = input.Position - hudDragStart
			local vp = Camera.ViewportSize
			Settings.HUDX = math.clamp(hudDragFrom.X + d.X, 0, math.max(vp.X - hud.AbsoluteSize.X, 0))
			Settings.HUDY = math.clamp(hudDragFrom.Y + d.Y, 0, math.max(vp.Y - hud.AbsoluteSize.Y, 0))
		end
	end)
	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then
			hudDrag = false
		end
	end)

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
				local parts = { math.floor(lastFps) .. " FPS", ping .. " ms" }
				if Settings.Fly then parts[#parts + 1] = "Voo" end
				if Settings.Noclip then parts[#parts + 1] = "Noclip" end
				if Settings.Fling then parts[#parts + 1] = "Fling" end
				if Settings.HitboxExpander then parts[#parts + 1] = "Hitbox" end
				hud.Text = table.concat(parts, "  •  ")
			end
		end)
	end)

	bootShow("TestToolkit v71 carregado  •  "
		.. (isMobile and "botão TT abre o menu" or "Ctrl direito abre o menu"),
		Color3.fromRGB(80, 255, 130), 5)
	print("[TestToolkit] v71 carregado com sucesso!")
else
	warn("[TestToolkit] erro na interface: " .. tostring(errUI))
	bootShow("TestToolkit: erro: " .. tostring(errUI), Color3.fromRGB(255, 90, 90))
end

--------------------------------------------------------------------
-- TECLAS (fora do buildUI, só as teclas de jogo)
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
		if Settings.AimMode == 1 then aimActive = true
		elseif Settings.AimMode == 2 then
			aimActive = not aimActive
			notify(aimActive and "Mira ativada" or "Mira desativada", aimActive and "on" or "off")
		end
	elseif matchesBind(input, Settings.SwitchKey) then
		if isAiming() then switchTarget() end
	elseif matchesBind(input, Settings.NoclipKey) then
		Settings.Noclip = not Settings.Noclip
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
		notify("Noclip: " .. (Settings.Noclip and "ON" or "OFF"), Settings.Noclip and "on" or "off")
	elseif matchesBind(input, Settings.FlyKey) then
		Settings.Fly = not Settings.Fly
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	elseif input.KeyCode == Settings.FlingKey then
		Settings.Fling = not Settings.Fling
		if _G.TT_uiRefresh then pcall(_G.TT_uiRefresh) end
	elseif input.KeyCode == Settings.FlingPlayerKey then
		task.spawn(function() Fling.flingSelected() end)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if Settings.AimMode == 1 and matchesBind(input, Settings.AimKey) then
		aimActive = false
	end
end)
