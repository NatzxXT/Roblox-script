--[[
    TestToolkit v59.5 - LocalScript (100% cliente)
    Abrir/fechar: CTRL DIREITO (PC) ou botão TT (mobile)

    v59.5:
      - Corrigido: ESP e Aimbot não pegam corpos caídos (verificação tripla)
      - Seat Invisibility mantido no original
      - NOVO: ESP Murder Mystery 2 (Inocente=Verde, Murder=Vermelho, Sheriff=Azul)
      - NOVO: Destaque da arma dropada em amarelo
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local PhysicsService = game:GetService("PhysicsService")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
Workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function()
	if Workspace.CurrentCamera then Camera = Workspace.CurrentCamera end
end)

--------------------------------------------------------------------
-- BOOT
--------------------------------------------------------------------
local bootGui, bootLabel, bootToken = nil, nil, 0
local function bootShow(text, color, hideAfter)
	pcall(function()
		if not bootGui then
			bootGui = Instance.new("ScreenGui")
			bootGui.Name = "TestToolkitBoot"
			bootGui.ResetOnSpawn = false
			bootGui.IgnoreGuiInset = true
			bootGui.DisplayOrder = 1000
			bootLabel = Instance.new("TextLabel")
			bootLabel.AnchorPoint = Vector2.new(0.5, 0)
			bootLabel.Position = UDim2.new(0.5, 0, 0, 8)
			bootLabel.Size = UDim2.new(0.9, 0, 0, 60)
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
		bootToken += 1
		local token = bootToken
		bootLabel.Text = text
		bootLabel.TextColor3 = color or Color3.new(1, 1, 1)
		bootGui.Enabled = true
		if hideAfter then
			task.delay(hideAfter, function()
				if token == bootToken and bootGui then bootGui.Enabled = false end
			end)
		end
	end)
end
bootShow("TestToolkit v59.5 carregando...")

--------------------------------------------------------------------
-- SETTINGS
--------------------------------------------------------------------
local Settings = {
	ESP = true, Rainbow = true, ShowNames = true, ShowHealth = true,
	Wallhack = true, ESPTeammates = true, ESPMaxDist = 600, ESPMaxTargets = 12,
	ESPHighlight = true, ESPColor = Color3.fromRGB(255, 80, 80), ESPFillTrans = 0.6,
	ESPBox = 1, ESPHeadDot = false, ESPTool = false, TracerOrigin = 1,

	TeamCheck = true, TargetMode = 3,

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

	HitboxExpander = false,
	HitboxSize = 6,
	HitboxRange = 200,
	HitboxInvisible = false,
	HitboxWallCheck = false,
	HitboxIgnoreAllies = true,
	ExpandHead = true,
	ExpandTorso = true,
	ExpandUpperTorso = true,
	ExpandLowerTorso = false,

	WalkSpeedOn = false, WalkSpeed = 32, WalkAutoLimit = true,
	JumpOn = false, JumpPower = 100,
	Noclip = false, NoclipKey = Enum.KeyCode.V,
	Fly = false, FlyKey = Enum.KeyCode.F, FlySpeed = 60, FlyAutoLimit = true,
	AntiVoid = false,
	AntiVoidY = math.max(Workspace.FallenPartsDestroyHeight + 100, -400),
	AntiKill = false, AntiKillMargin = 200,
	AntiFling = false, AntiFlingRestore = true, AntiFlingSpeed = 160,
	InfJump = false, FullBright = false, NoFog = false,
	CamFOVOn = false, CamFOV = 90, AntiAFK = true, Tracers = false,

	-- MM2 ESP
	MM2Mode = false,
	MM2InnocentColor = Color3.fromRGB(0, 255, 0),
	MM2MurderColor = Color3.fromRGB(255, 0, 0),
	MM2SheriffColor = Color3.fromRGB(0, 100, 255),
	MM2ShowDroppedGun = true,
	MM2DroppedGunColor = Color3.fromRGB(255, 255, 0),

	SeatInvisible = false,
	SeatInvisibleX = -25.95,
	SeatInvisibleY = 84,
	SeatInvisibleZ = 3537.55,
	SeatInvisibleDuration = 0,
	SeatInvisibleReturn = true,

	Fling = false, FlingKey = Enum.KeyCode.G,
	FlingRange = 12, FlingPower = 3000, FlingMode = 1, FlingMode2 = 2,
	FlingBots = true, FlingPlayers = false, FlingRepeat = 0.1,

	ShowHUD = true, HUDX = 10, HUDY = 10, HUDEdit = false,
	Notifications = true, MenuAlpha = 0.1, DeviceMode = 1,

	PerfMode = false, FPSUnlock = false, FPSCap = 240,

	MobileEdit = false, MobileBtnSize = 56, MobileBtnAlpha = 0.15,

	ShowBtnFly = true, ShowBtnNoclip = true, ShowBtnDown = true,
	ShowBtnEsp = true, ShowBtnTrace = true, ShowBtnInf = true,
	ShowBtnAim = true, ShowBtnFling = true,
}

local MENU_KEY = Enum.KeyCode.RightControl

--------------------------------------------------------------------
-- CONFIG
--------------------------------------------------------------------
local CONFIG_FILE = "TestToolkit_config.json"
local CONFIG_SKIP = {
	DeviceMode = true, PerfMode = true, FPSUnlock = true, MobileEdit = true,
	HUDEdit = true, Fly = true, Noclip = true, Fling = true, HitboxExpander = true,
	SeatInvisible = true, MM2Mode = true,
}

local function serializeSettings()
	local data = {}
	for k, v in pairs(Settings) do
		if not CONFIG_SKIP[k] then
			local t = typeof(v)
			if t == "boolean" or t == "number" then
				data[k] = v
			elseif t == "EnumItem" then
				data[k] = { tostring(v.EnumType), v.Name }
			elseif t == "Color3" then
				data[k] = { "Color3", v.R, v.G, v.B }
			end
		end
	end
	return data
end

local function applySavedConfig(data)
	if type(data) ~= "table" then return end
	for k, v in pairs(data) do
		local cur = Settings[k]
		if cur ~= nil and not CONFIG_SKIP[k] then
			local t = typeof(cur)
			if (t == "boolean" or t == "number") and typeof(v) == t then
				Settings[k] = v
			elseif t == "EnumItem" and type(v) == "table" and #v == 2 then
				local ok, item = pcall(function() return Enum[v[1]][v[2]] end)
				if ok and typeof(item) == "EnumItem" and item.EnumType == cur.EnumType then
					Settings[k] = item
				end
			elseif t == "Color3" and type(v) == "table" and v[1] == "Color3" then
				Settings[k] = Color3.new(v[2], v[3], v[4])
			end
		end
	end
end

pcall(function()
	if isfile and readfile and isfile(CONFIG_FILE) then
		applySavedConfig(HttpService:JSONDecode(readfile(CONFIG_FILE)))
	end
end)

--------------------------------------------------------------------
-- DEVICE
--------------------------------------------------------------------
local function detectMobile()
	if Settings.DeviceMode == 2 then return false end
	if Settings.DeviceMode == 3 then return true end
	return UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
end

local isMobile = detectMobile()
local deviceListeners = {}
local mobileFlyDown = false

local function refreshDevice()
	local m = detectMobile()
	if m ~= isMobile then
		isMobile = m
		for _, fn in ipairs(deviceListeners) do pcall(fn) end
	end
end
UserInputService:GetPropertyChangedSignal("KeyboardEnabled"):Connect(refreshDevice)
UserInputService:GetPropertyChangedSignal("TouchEnabled"):Connect(refreshDevice)

local function getAimOrigin()
	if isMobile then
		local vp = Camera.ViewportSize
		return Vector2.new(vp.X / 2, vp.Y / 2)
	end
	return UserInputService:GetMouseLocation()
end

--------------------------------------------------------------------
-- TEAM
--------------------------------------------------------------------
local manualAllies = setmetatable({}, { __mode = "k" })
local TEAM_KEYS = { "Team", "TeamName", "TeamColor", "team", "teamName", "Side", "Faction" }

local function toKey(v)
	local t = typeof(v)
	if t == "string" then return v ~= "" and ("s:" .. v) or nil
	elseif t == "Instance" then return v.Name ~= "" and ("s:" .. v.Name) or nil
	elseif t == "BrickColor" then return "c:" .. v.Name
	elseif t == "Color3" then return "c:" .. v:ToHex()
	elseif t == "number" then return "n:" .. v end
	return nil
end

local function fromContainer(inst)
	for _, n in ipairs(TEAM_KEYS) do
		local k = toKey(inst:GetAttribute(n))
		if k then return k end
	end
	for _, n in ipairs(TEAM_KEYS) do
		local v = inst:FindFirstChild(n)
		if v and v:IsA("ValueBase") then
			local k = toKey(v.Value)
			if k then return k end
		end
	end
	return nil
end

local function getTeamKey(model, plr)
	if plr then
		if plr.Team then return "s:" .. plr.Team.Name end
		local k = fromContainer(plr)
		if k then return k end
		if not plr.Neutral and plr.TeamColor then return "c:" .. plr.TeamColor.Name end
	end
	if model then return fromContainer(model) end
	return nil
end

local teamCache = setmetatable({}, { __mode = "k" })
local myTeamKey, myTeamAt = nil, -1

local function isTeammate(model, plr)
	local k = plr or model
	if manualAllies[k] then return true end
	local now = os.clock()
	if now - myTeamAt > 0.5 then
		myTeamKey = getTeamKey(LocalPlayer.Character, LocalPlayer)
		myTeamAt = now
	end
	if myTeamKey == nil then return false end
	local c = teamCache[k]
	if c and now - c.t < 0.5 then return c.v end
	if not c then c = {}; teamCache[k] = c end
	local theirs = getTeamKey(model, plr)
	c.v = theirs ~= nil and theirs == myTeamKey
	c.t = now
	return c.v
end

--------------------------------------------------------------------
-- MM2 DETECÇÃO DE FUNÇÃO
--------------------------------------------------------------------
local MM2 = {}
do
	local roleCache = setmetatable({}, { __mode = "k" })
	local ROLE_KEYS = {
		"Role", "role", "Team", "team", "Class", "class",
		"Rank", "rank", "Type", "type", "Murderer", "murderer",
	}

	local function toRole(v)
		if type(v) ~= "string" then return nil end
		v = string.lower(v)
		if string.find(v, "murder") then return "murder" end
		if string.find(v, "sheriff") or string.find(v, "cop") then return "sheriff" end
		if string.find(v, "inno") or string.find(v, "hero") or string.find(v, "civil") then return "innocent" end
		return nil
	end

	local function checkTags(inst)
		for _, tag in ipairs(inst:GetTags()) do
			local r = toRole(tag)
			if r then return r end
		end
		return nil
	end

	local function checkAttrs(inst)
		for _, key in ipairs(ROLE_KEYS) do
			local v = inst:GetAttribute(key)
			local r = toRole(v)
			if r then return r end
		end
		return nil
	end

	local function checkValues(inst)
		for _, key in ipairs(ROLE_KEYS) do
			local child = inst:FindFirstChild(key)
			if child and child:IsA("ValueBase") then
				local r = toRole(child.Value)
				if r then return r end
			end
			if child and child:IsA("StringValue") then
				local r = toRole(child.Value)
				if r then return r end
			end
		end
		return nil
	end

	local function scanDescendants(inst)
		local n = 0
		for _, d in ipairs(inst:GetDescendants()) do
			local r = checkTags(d) or checkAttrs(d) or checkValues(d)
			if r then return r end
			n += 1
			if n > 100 then break end
		end
		return nil
	end

	function MM2.getRole(model, plr)
		if not model and not plr then return nil end
		local key = plr or model
		local c = roleCache[key]
		local now = os.clock()
		if c and now - c.t < 0.5 then return c.v end
		if not c then c = { v = nil, t = 0 }; roleCache[key] = c end

		local role = nil
		if plr then
			role = checkTags(plr) or checkAttrs(plr) or checkValues(plr) or scanDescendants(plr)
		end
		if not role and model then
			role = checkTags(model) or checkAttrs(model) or checkValues(model)
				or scanDescendants(model)
		end
		-- Fallback: verifica ferramentas no personagem
		if not role and model then
			for _, tool in ipairs(model:GetChildren()) do
				if tool:IsA("Tool") then
					local n = string.lower(tool.Name)
					if string.find(n, "knife") or string.find(n, "murder") then
						role = "murder"; break
					elseif string.find(n, "gun") or string.find(n, "revolver")
						or string.find(n, "sheriff") then
						role = "sheriff"; break
					end
				end
			end
		end

		c.v = role
		c.t = now
		return role
	end

	function MM2.clearCache()
		table.clear(roleCache)
	end

	-- Cache de armas dropadas
	local dropCache = setmetatable({}, { __mode = "k" })
	local dropAt = 0
	local dropped = {}

	function MM2.getDroppedGuns()
		local now = os.clock()
		if now - dropAt < 0.4 then return dropped end
		dropAt = now
		table.clear(dropped)
		table.clear(dropCache)
		for _, d in ipairs(Workspace:GetChildren()) do
			if d:IsA("Tool") then
				local n = string.lower(d.Name)
				if string.find(n, "gun") or string.find(n, "revolver")
					or string.find(n, "knife") then
					local handle = d:FindFirstChild("Handle")
					if handle and handle:IsA("BasePart") then
						dropped[#dropped + 1] = d
						dropCache[d] = handle
					end
				end
			end
		end
		return dropped
	end
end

--------------------------------------------------------------------
-- TARGETS (VERIFICAÇÃO DE CORPO CAÍDO APRIMORADA)
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

-- VERIFICAÇÃO AGRESSIVA: Retorna true apenas se o alvo estiver VIVO e EM PÉ
local function isAlive(hum)
	if not hum or hum.Health <= 0 then return false end

	local state = hum:GetState()
	if state == Enum.HumanoidStateType.Dead
		or state == Enum.HumanoidStateType.Physics
		or state == Enum.HumanoidStateType.Ragdoll
		or state == Enum.HumanoidStateType.FallingDown then
		return false
	end

	if hum.Health < 5 then return false end

	local root = hum.Parent:FindFirstChild("HumanoidRootPart")
	if root then
		local upVector = root.CFrame.UpVector
		local worldUp = Vector3.new(0, 1, 0)
		local dot = upVector:Dot(worldUp)
		if dot < 0.5 then return false end

		if hum.FloorMaterial == Enum.Material.Air then
			local vel = root.AssemblyLinearVelocity
			if vel.Y < -50 then return false end
		end
	end

	local head = hum.Parent:FindFirstChild("Head")
	if head and root then
		local heightDiff = head.Position.Y - root.Position.Y
		if heightDiff < 0.5 then return false end
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
				local include
				if purpose == "esp" then
					include = (not mate) or Settings.ESPTeammates
				else
					include = (not mate) or (not Settings.TeamCheck)
				end
				if include then
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

local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
local filterList = {}
local LOS_GRACE = 0.25

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
		if isAlive(t.Humanoid) then
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
	end
	table.sort(rough, candLess)

	local out = {}
	for i = 1, #rough do
		if #out >= 6 then break end
		local r = rough[i]
		local t = r.T
		local part = pickAim(t.Model, t.Humanoid)
		if part then
			local sp, onScreen = Camera:WorldToViewportPoint(part.Position)
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
-- GUI + TEMA
--------------------------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Name = "TestToolkit"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 100
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

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

local function tween(obj, props, t, style, dir)
	local tw = TweenService:Create(
		obj,
		TweenInfo.new(t or 0.18, style or Style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out),
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
fovCircle.Name = "FOVCircle"
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.BackgroundTransparency = 1
fovCircle.BorderSizePixel = 0
fovCircle.Parent = gui
corner(fovCircle, 9999)
local fovStroke = stroke(fovCircle, Color3.new(1, 1, 1), 0.15, 1.5)

local silentFovCircle = Instance.new("Frame")
silentFovCircle.Name = "SilentFOVCircle"
silentFovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
silentFovCircle.BackgroundTransparency = 1
silentFovCircle.BorderSizePixel = 0
silentFovCircle.Parent = gui
corner(silentFovCircle, 9999)
local silentFovStroke = stroke(silentFovCircle, Color3.fromRGB(255, 100, 255), 0.3, 1.5)

local triggerFovCircle = Instance.new("Frame")
triggerFovCircle.Name = "TriggerFOVCircle"
triggerFovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
triggerFovCircle.BackgroundTransparency = 1
triggerFovCircle.BorderSizePixel = 0
triggerFovCircle.Parent = gui
corner(triggerFovCircle, 9999)
local triggerFovStroke = stroke(triggerFovCircle, Color3.fromRGB(255, 100, 100), 0.4, 1.5)

local lockMarker = Instance.new("Frame")
lockMarker.Name = "LockMarker"
lockMarker.AnchorPoint = Vector2.new(0.5, 0.5)
lockMarker.Size = UDim2.fromOffset(24, 24)
lockMarker.BackgroundTransparency = 1
lockMarker.Visible = false
lockMarker.Parent = gui
corner(lockMarker, 9999)
local lockStroke = stroke(lockMarker, Theme.Bad, 0, 2)

--------------------------------------------------------------------
-- TOASTS
--------------------------------------------------------------------
local toastHolder = Instance.new("Frame")
toastHolder.Name = "Toasts"
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

local otherChars = {}
local otherConns = {}

local function trackChar(char)
	local data = { Parts = {}, Root = char:FindFirstChild("HumanoidRootPart") }
	otherChars[char] = data
	for _, d in ipairs(char:GetDescendants()) do
		if d:IsA("BasePart") then data.Parts[d] = true end
	end
	local c1 = char.DescendantAdded:Connect(function(d)
		if d:IsA("BasePart") then
			data.Parts[d] = true
			if d.Name == "HumanoidRootPart" then data.Root = d end
		end
	end)
	local c2 = char.DescendantRemoving:Connect(function(d)
		data.Parts[d] = nil
	end)
	local c3
	c3 = char.AncestryChanged:Connect(function(_, parent)
		if not parent then
			c1:Disconnect(); c2:Disconnect(); c3:Disconnect()
			if otherChars[char] == data then otherChars[char] = nil end
		end
	end)
end

local function trackPlayer(plr)
	if plr == LocalPlayer then return end
	if plr.Character then trackChar(plr.Character) end
	otherConns[plr] = plr.CharacterAdded:Connect(trackChar)
end

for _, p in ipairs(Players:GetPlayers()) do trackPlayer(p) end
Players.PlayerAdded:Connect(trackPlayer)
Players.PlayerRemoving:Connect(function(plr)
	local conn = otherConns[plr]
	if conn then conn:Disconnect(); otherConns[plr] = nil end
	if plr.Character then otherChars[plr.Character] = nil end
end)

local flyObjs = nil
local antiKillHold = 0
local flyCur, flyLastPos, flyCooldown = 0, nil, 0
local FLY_RAMP = 500
local uiRefresh = function() end
local flyPlatformSet = false
local stableOld, stableNew = nil, nil
local stableTimer = 0

local wsActive, jpActive = false, false
local SPEED_RAMP_BASE = 240
local wsCurrent = 16
local walkLastPos, walkCooldown = nil, 0
local walkUserGoal = nil
local walkPulledAt = 0
local walkProbeAt = 0

if LocalPlayer.Character then trackMyCharacter(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(function(char)
	flyObjs = nil
	flyPlatformSet = false
	stableOld, stableNew = nil, nil
	walkLastPos = nil
	trackMyCharacter(char)
end)

--------------------------------------------------------------------
-- AIM-LOCK / SILENT / TRIGGER
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

local function isAiming()
	return Settings.AimEnabled and (aimActive or Settings.AimMode == 3)
end
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
local groundParams = RaycastParams.new()
groundParams.FilterType = Enum.RaycastFilterType.Exclude

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
		local speed = Settings.BulletSpeed
		if speed > 0 then
			local camPos = Camera.CFrame.Position
			local flight = (pos - camPos).Magnitude / speed
			for _ = 1, 3 do
				local future = pos + hv * (time + flight)
				flight = (future - camPos).Magnitude / speed
			end
			time += flight
		end
	end
	time = math.clamp(time, 0, 0.6)
	if time <= 0 then return pos end

	local horizontal = hv * time
	if shot ~= 1 and hv.Magnitude > 1.5 then
		horizontal += Vector3.new(acc.X, 0, acc.Z) * (0.25 * time * time)
	end
	local vertical = 0
	if Settings.AutoPredict and shot ~= 1 then
		if hum.FloorMaterial == Enum.Material.Air then
			local vy = math.clamp(vel.Y, -500, 500)
			vertical = vy * time - 0.5 * Workspace.Gravity * time * time
			groundParams.FilterDescendantsInstances = { LocalPlayer.Character, model }
			local hit = Workspace:Raycast(pos, Vector3.new(0, -300, 0), groundParams)
			if hit then
				local minY = hit.Position.Y + (pos.Y - root.Position.Y) + hum.HipHeight + root.Size.Y / 2
				vertical = math.max(vertical, math.min(0, minY - pos.Y))
			end
		end
	else
		vertical = math.clamp(vel.Y, -500, 500) * time
	end
	local result = pos + horizontal + Vector3.new(0, vertical, 0)
	if result.X ~= result.X or result.Y ~= result.Y or result.Z ~= result.Z then
		return pos
	end
	return result
end

local hookInstalled = false
local function tryInstallHooks()
	if hookInstalled then return end
	if not hookfunction then return end
	pcall(function()
		local original = Camera.WorldToViewportPoint
		Camera.WorldToViewportPoint = function(self, pos)
			if silentActive and silentTarget and silentTarget.Part and silentTarget.Part.Parent then
				if typeof(pos) == "Vector3" then
					local d = (pos - silentTarget.Part.Position).Magnitude
					if d < 2 then return original(self, silentTarget.Part.Position) end
				end
			end
			return original(self, pos)
		end
	end)
	pcall(function()
		local original = Camera.WorldToScreenPoint
		Camera.WorldToScreenPoint = function(self, pos)
			if silentActive and silentTarget and silentTarget.Part and silentTarget.Part.Parent then
				if typeof(pos) == "Vector3" then
					local d = (pos - silentTarget.Part.Position).Magnitude
					if d < 2 then return original(self, silentTarget.Part.Position) end
				end
			end
			return original(self, pos)
		end
	end)
	hookInstalled = true
end

local mouseHookInstalled = false
local function tryInstallMouseHook()
	if mouseHookInstalled then return end
	local mouse = LocalPlayer:GetMouse()
	if not mouse then return end
	if not getrawmetatable or not setreadonly then return end
	pcall(function()
		local mt = getrawmetatable(mouse)
		if not mt or getreadonly(mt) then return end
		setreadonly(mt, false)
		local oldIndex = mt.__index
		mt.__index = function(self, key)
			if silentActive and silentTarget and silentTarget.Part and silentTarget.Part.Parent then
				if key == "Hit" then
					return silentTarget.Part.CFrame
				elseif key == "Target" then
					return silentTarget.Part
				elseif key == "UnitRay" then
					local myRoot = getLocalRoot()
					if myRoot then
						local origin = Camera.CFrame.Position
						local dir = (silentTarget.Part.Position - origin).Unit
						return Ray.new(origin, dir * 1000)
					end
				end
			end
			if type(oldIndex) == "function" then return oldIndex(self, key) end
			return mouse[key]
		end
		setreadonly(mt, true)
		mouseHookInstalled = true
	end)
end

task.spawn(function()
	task.wait(0.5)
	pcall(tryInstallHooks)
	pcall(tryInstallMouseHook)
end)

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
		if isAlive(t.Humanoid) and t.Root and t.Root.Parent then
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
	end
	if best and isAlive(best.Humanoid) and best.Model.Parent then
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
		if isAlive(t.Humanoid) and t.Root and t.Root.Parent then
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
	if not Settings.TriggerBot then
		triggerBotActive = false
		return
	end
	local now = os.clock()
	local jitter = (math.random() - 0.5) * 0.02
	if now - lastTriggerShot < Settings.TriggerDelay + jitter then return end
	local target = getTriggerTarget()
	if not target then
		triggerBotActive = false
		return
	end
	triggerBotActive = true
	if Settings.TriggerVisible and not hasLineOfSight(target.Root, target.Model) then return end
	lastTriggerShot = now
	simulateClick()
end

RunService.RenderStepped:Connect(function()
	pcall(function()
		if not Settings.AimEnabled and not Settings.TriggerBot then
			silentActive = false
			silentTarget = nil
			return
		end
		updateSilentTarget()
		updateTriggerBot()
	end)
end)

RunService:BindToRenderStep("TestToolkitAim", Enum.RenderPriority.Camera.Value + 1, function(dt)
	if spectating then
		currentTarget = nil
		lockMarker.Visible = false
		return
	end

	local now = os.clock()
	local myRoot = getLocalRoot()
	local isFirstPerson = myRoot and (Camera.CFrame.Position - myRoot.Position).Magnitude < 1.5
	local aimOrigin = isFirstPerson
		and Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
		or getAimOrigin()

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
				elseif now - (currentTarget.LastSeen or 0) > LOS_GRACE then
					currentTarget = nil
				end
			end
		end

		if currentTarget then
			local part = currentTarget.Part
			if not part or not part.Parent then
				currentTarget = nil
			else
				local sp, on = Camera:WorldToViewportPoint(part.Position)
				local tooFar = Settings.AimMaxDist > 0
					and (part.Position - Camera.CFrame.Position).Magnitude > Settings.AimMaxDist * 1.1
				local outFov = Settings.FOVEnabled
					and (not on or (Vector2.new(sp.X, sp.Y) - aimOrigin).Magnitude > Settings.FOVRadius * 1.35)
				if tooFar or outFov then currentTarget = nil end
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
-- HITBOX EXPANDER
--------------------------------------------------------------------
local HitboxExpander = {}
do
	local savedProps = setmetatable({}, { __mode = "k" })
	local savedGroups = setmetatable({}, { __mode = "k" })

	local HB_GROUP = "TTHitbox"
	local PLAYER_GROUP = "TTPlayer"
	local groupsReady = false

	local function setupGroups()
		local ok = pcall(function()
			PhysicsService:RegisterCollisionGroup(HB_GROUP)
		end)
		pcall(function()
			PhysicsService:RegisterCollisionGroup(PLAYER_GROUP)
		end)
		pcall(function()
			PhysicsService:CollisionGroupSetCollidable(HB_GROUP, HB_GROUP, false)
		end)
		pcall(function()
			PhysicsService:CollisionGroupSetCollidable(HB_GROUP, PLAYER_GROUP, false)
		end)
		pcall(function()
			PhysicsService:CollisionGroupSetCollidable(PLAYER_GROUP, PLAYER_GROUP, false)
		end)
		pcall(function()
			PhysicsService:CollisionGroupSetCollidable(PLAYER_GROUP, "Default", true)
		end)
		groupsReady = true
		return ok
	end

	task.spawn(function()
		pcall(setupGroups)
	end)

	local function markPlayerPart(part)
		if not part or not part:IsA("BasePart") then return end
		if groupsReady then
			savedGroups[part] = savedGroups[part] or part.CollisionGroup
			pcall(function() part.CollisionGroup = PLAYER_GROUP end)
		end
	end

	local function bindMyChar(char)
		for _, d in ipairs(char:GetDescendants()) do
			if d:IsA("BasePart") then markPlayerPart(d) end
		end
		char.DescendantAdded:Connect(function(d)
			if d:IsA("BasePart") then
				task.defer(markPlayerPart, d)
			end
		end)
	end

	if LocalPlayer.Character then bindMyChar(LocalPlayer.Character) end
	LocalPlayer.CharacterAdded:Connect(function(char)
		task.wait(0.1)
		bindMyChar(char)
	end)

	local PART_KEYS = {
		{ "Head", "ExpandHead" },
		{ "UpperTorso", "ExpandUpperTorso" },
		{ "Torso", "ExpandTorso" },
		{ "LowerTorso", "ExpandLowerTorso" },
	}

	local function getActiveParts()
		local list = {}
		for _, entry in ipairs(PART_KEYS) do
			if Settings[entry[2]] then
				list[#list + 1] = entry[1]
			end
		end
		return list
	end

	local function savePart(part)
		if savedProps[part] then return end
		savedProps[part] = {
			Size = part.Size,
			CanCollide = part.CanCollide,
			Massless = part.Massless,
			Transparency = part.Transparency,
			Color = part.Color,
			Material = part.Material,
			CollisionGroup = part.CollisionGroup,
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
			part.Color = props.Color
			part.Material = props.Material
			if groupsReady then part.CollisionGroup = props.CollisionGroup end
		end)
		savedProps[part] = nil
	end

	local function expandPlayer(player)
		local char = player.Character
		if not char then return end
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hum or hum.Health <= 0 then return end

		local activeParts = getActiveParts()
		for _, name in ipairs(activeParts) do
			local part = char:FindFirstChild(name)
			if part and part:IsA("BasePart") then
				savePart(part)
				local size = Settings.HitboxSize
				pcall(function()
					part.Size = Vector3.new(size, size, size)
					part.CanCollide = false
					part.Massless = true
					if groupsReady then part.CollisionGroup = HB_GROUP end
					if Settings.HitboxInvisible then
						part.Transparency = 1
					else
						part.Transparency = 0.5
					end
				end)
			end
		end
	end

	local function restorePlayer(player)
		local char = player.Character
		if not char then return end
		for _, entry in ipairs(PART_KEYS) do
			local part = char:FindFirstChild(entry[1])
			if part and part:IsA("BasePart") then
				restorePart(part)
			end
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

		if Settings.HitboxIgnoreAllies then
			if isTeammate(char, player) then return false end
		end

		if Settings.HitboxWallCheck then
			local head = char:FindFirstChild("Head")
			if head and not hasLineOfSight(head, char) then
				if not hasLineOfSight(theirRoot, char) then
					return false
				end
			end
		end

		return true
	end

	local lastApply = 0
	local APPLY_INTERVAL = 0.1

	local function step()
		if not Settings.HitboxExpander then
			for _, plr in ipairs(Players:GetPlayers()) do
				if plr ~= LocalPlayer then
					restorePlayer(plr)
				end
			end
			return
		end

		local now = os.clock()
		if now - lastApply < APPLY_INTERVAL then return end
		lastApply = now

		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer then
				if shouldExpand(plr) then
					expandPlayer(plr)
				else
					restorePlayer(plr)
				end
			end
		end
	end

	HitboxExpander.step = step
	HitboxExpander.restoreAll = function()
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr ~= LocalPlayer then
				restorePlayer(plr)
			end
		end
	end
end

task.spawn(function()
	while task.wait(0.1) do
		pcall(HitboxExpander.step)
	end
end)

LocalPlayer.CharacterAdded:Connect(function()
	task.wait(0.5)
	pcall(HitboxExpander.restoreAll)
end)

--------------------------------------------------------------------
-- INVISIBILIDADE VIA SEAT (ORIGINAL - v59)
--------------------------------------------------------------------
local SeatInvisible = {}
do
	local mySeat = nil
	local active = false

	local function cleanupSeat()
		local existing = workspace:FindFirstChild("invischair")
		if existing then
			pcall(function() existing:Destroy() end)
		end
		mySeat = nil
	end

	local function findSafeCoords()
		local bestPos = Vector3.new(-25.95, 84, 3537.55)
		local myRoot = getLocalRoot()
		if not myRoot then return bestPos end

		pcall(function()
			local xs, ys, zs = {}, {}, {}
			local count = 0
			for _, d in ipairs(Workspace:GetDescendants()) do
				if d:IsA("BasePart") and d.Anchored then
					local p = d.Position
					xs[#xs + 1] = p.X
					ys[#ys + 1] = p.Y
					zs[#zs + 1] = p.Z
					count += 1
					if count >= 500 then break end
				end
			end
			if #xs >= 10 then
				table.sort(xs); table.sort(zs)
				local maxX = xs[#xs]
				local minZ = zs[1]
				bestPos = Vector3.new(maxX + 5000, 500, minZ - 5000)
			end
		end)
		return bestPos
	end

	local function activate()
		local char = LocalPlayer.Character
		if not char then return end
		local hrp = char:FindFirstChild("HumanoidRootPart")
		if not hrp then return end

		cleanupSeat()

		local savedPosition = hrp.CFrame
		local targetPos = Vector3.new(
			Settings.SeatInvisibleX,
			Settings.SeatInvisibleY,
			Settings.SeatInvisibleZ
		)

		char:MoveTo(targetPos)
		task.wait(0.15)

		local seat = Instance.new("Seat")
		seat.Name = "invischair"
		seat.Anchored = false
		seat.CanCollide = false
		seat.Transparency = 1
		seat.Position = targetPos
		seat.Parent = workspace
		mySeat = seat

		local weld = Instance.new("Weld")
		weld.Part0 = seat
		weld.Part1 = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
		weld.Parent = seat

		task.wait()

		seat.CFrame = savedPosition

		for _, descendant in ipairs(char:GetDescendants()) do
			if descendant:IsA("BasePart") or descendant:IsA("Decal") then
				descendant.Transparency = 0.5
			end
		end
	end

	local function deactivate()
		cleanupSeat()
		if LocalPlayer.Character then
			for _, descendant in ipairs(LocalPlayer.Character:GetDescendants()) do
				if descendant:IsA("BasePart") or descendant:IsA("Decal") then
					descendant.Transparency = 0
				end
			end
		end
	end

	local function toggleInvisibility()
		active = not active
		Settings.SeatInvisible = active

		if active then
			activate()
		else
			deactivate()
		end

		if uiRefresh then pcall(uiRefresh) end
	end

	SeatInvisible.toggle = toggleInvisibility
	SeatInvisible.isActive = function() return active end
	SeatInvisible.findSafeCoords = findSafeCoords
	SeatInvisible.forceOff = function()
		if active then
			active = false
			Settings.SeatInvisible = false			deactivate()
			if uiRefresh then pcall(uiRefresh) end
		end
	end

	task.spawn(function()
		while task.wait(0.5) do
			if active and (not mySeat or not mySeat.Parent) then
				active = false
				Settings.SeatInvisible = false
				deactivate()
				if uiRefresh then pcall(uiRefresh) end
			end
		end
	end)

	task.spawn(function()
		while task.wait(0.3) do
			if active then
				local char = LocalPlayer.Character
				if char then
					for _, descendant in ipairs(char:GetDescendants()) do
						if descendant:IsA("BasePart") or descendant:IsA("Decal") then
							if descendant.Transparency ~= 0.5 and descendant.Transparency ~= 1 then
								descendant.Transparency = 0.5
							end
						end
					end
				end
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
-- FLY
--------------------------------------------------------------------
local function destroyFly()
	flyCur, flyLastPos = 0, nil
	if flyObjs then
		for _, o in pairs(flyObjs) do
			if o and o.Parent then o:Destroy() end
		end
		flyObjs = nil
	end
	if flyPlatformSet then
		local hum = getLocalHumanoid()
		if hum then hum.PlatformStand = false end
		flyPlatformSet = false
	end
end

local function ensureFly(root)
	if flyObjs and flyObjs.Att and flyObjs.Att.Parent == root then return flyObjs end
	destroyFly()
	local att = Instance.new("Attachment")
	att.Name = "TTFlyAttachment"
	att.Parent = root
	local lv = Instance.new("LinearVelocity")
	lv.Name = "TTFlyVelocity"
	lv.Attachment0 = att
	lv.RelativeTo = Enum.ActuatorRelativeTo.World
	lv.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
	lv.MaxForce = math.huge
	lv.VectorVelocity = Vector3.zero
	lv.Parent = root
	local ao = Instance.new("AlignOrientation")
	ao.Name = "TTFlyAlign"
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
	flyPlatformSet = true
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
		if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or mobileFlyDown then up -= 1 end
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

local function flyGuard(root, dt)
	local pos = root.Position
	local last = flyLastPos
	flyLastPos = pos
	if not Settings.FlyAutoLimit or not last or not flyObjs then return end
	local now = os.clock()
	if now < flyCooldown or now < antiKillHold + 0.3 then return end
	local intended = flyObjs.LV.VectorVelocity
	if intended.Magnitude < 5 then return end
	local back = -(pos - last):Dot(intended.Unit)
	if back > math.max(20, intended.Magnitude * dt * 2) then
		flyCooldown = now + 1.5
		local new = math.max(20, math.floor(Settings.FlySpeed * 0.65))
		if new < Settings.FlySpeed then
			Settings.FlySpeed = new
			flyCur = math.min(flyCur, new)
			uiRefresh()
			notify("Voo: puxão detectado, velocidade reduzida para " .. new, "off")
		end
	end
end

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
		uiRefresh()
		notify("Velocidade: puxão detectado, reduzida para " .. new, "off")
	elseif walkUserGoal and now - walkPulledAt > 6 and now - walkProbeAt > 3
		and Settings.WalkSpeed < walkUserGoal then
		walkProbeAt = now
		Settings.WalkSpeed = math.min(walkUserGoal, math.ceil(Settings.WalkSpeed * 1.12))
		uiRefresh()
	end
end

--------------------------------------------------------------------
-- FLING
--------------------------------------------------------------------
local Fling = {}
do
	local lastFling = 0
	local lastNotify = 0
	local flingActive = false
	local savedCollide = {}

	local function getTargetParts(model)
		local hum = model:FindFirstChildOfClass("Humanoid")
		if not hum then return nil end
		if hum.RigType == Enum.HumanoidRigType.R6 then
			return {
				Root = model:FindFirstChild("HumanoidRootPart"),
				Torso = model:FindFirstChild("Torso"),
				Head = model:FindFirstChild("Head"),
			}
		end
		return {
			Root = model:FindFirstChild("HumanoidRootPart"),
			Torso = model:FindFirstChild("UpperTorso"),
			Head = model:FindFirstChild("Head"),
		}
	end

	local function disableBotCollide(model)
		for _, part in ipairs(model:GetDescendants()) do
			if part:IsA("BasePart") then
				if savedCollide[part] == nil then savedCollide[part] = part.CanCollide end
				part.CanCollide = false
			end
		end
	end

	local function restoreAllCollide()
		for part, val in pairs(savedCollide) do
			if part.Parent then part.CanCollide = val end
		end
		table.clear(savedCollide)
	end

	local function doFling(targetRoot, myRoot, power, mode)
		if not targetRoot or not targetRoot.Parent then return false end
		if not myRoot or not myRoot.Parent then return false end
		local hum = targetRoot.Parent:FindFirstChildOfClass("Humanoid")
		if hum and hum.Health <= 0 then return false end
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
			targetRoot.Velocity = linVel
			targetRoot.RotVelocity = angVel
		end)
		pcall(function()
			for _, part in ipairs(targetRoot.Parent:GetDescendants()) do
				if part:IsA("BasePart") and part ~= targetRoot then
					part.AssemblyLinearVelocity = linVel
					part.AssemblyAngularVelocity = angVel
				end
			end
		end)
		return true
	end

	local function pickFlingTargets()
		local myRoot = getLocalRoot()
		if not myRoot then return {} end
		local myPos = myRoot.Position
		local range = Settings.FlingRange
		local list = {}
		if Settings.FlingMode == 2 and currentTarget and currentTarget.Model then
			local m = currentTarget.Model
			if m.Parent and not Players:GetPlayerFromCharacter(m) then
				local parts = getTargetParts(m)
				if parts and parts.Root then
					list[1] = { Model = m, Parts = parts, D = 0 }
				end
			end
			return list
		end
		for model, hum in pairs(humanoids) do
			if model ~= LocalPlayer.Character and model.Parent and hum.Parent and hum.Health > 0 then
				local plr = Players:GetPlayerFromCharacter(model)
				if not plr and Settings.FlingBots then
					local parts = getTargetParts(model)
					if parts and parts.Root then
						local d = (parts.Root.Position - myPos).Magnitude
						if d <= range then
							list[#list + 1] = { Model = model, Parts = parts, D = d }
						end
					end
				end
			end
		end
		table.sort(list, function(a, b) return a.D < b.D end)
		if Settings.FlingMode == 3 and #list > 1 then list = { list[1] } end
		return list
	end

	Fling.step = function()
		if not Settings.Fling then
			if flingActive then
				flingActive = false
				restoreAllCollide()
			end
			return
		end
		local now = os.clock()
		local jitter = (math.random() - 0.5) * 0.04
		if now - lastFling < Settings.FlingRepeat + jitter then return end
		lastFling = now
		local myHum, myRoot = getLocalHumanoid(), getLocalRoot()
		if not myHum or myHum.Health <= 0 then return end
		flingActive = true
		local targets = pickFlingTargets()
		if #targets == 0 then
			if now - lastNotify > 4 then
				lastNotify = now
				notify("Fling: nenhum BOT no alcance (" .. Settings.FlingRange .. " studs)", "off")
			end
			return
		end
		local power = Settings.FlingPower
		local mode = Settings.FlingMode2 or 2
		for i = 1, #targets do
			doFling(targets[i].Parts.Root, myRoot, power, mode)
		end
	end

	Fling.isActive = function() return flingActive end
end

--------------------------------------------------------------------
-- HEARTBEAT
--------------------------------------------------------------------
local defaultWalkSpeed = 16
local defaultJumpPower, defaultUseJumpPower = 50, true
local lastSafe = nil
local ANTIFLING_MAX_SPIN = 40
local lastFlingNotify = 0

LocalPlayer.CharacterAdded:Connect(function()
	lastSafe = nil
	wsActive, jpActive = false, false
end)

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
			flyGuard(root, dt)
		elseif flyObjs or flyPlatformSet then
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
			local yLimit = math.max(limit, Settings.JumpOn and Settings.JumpPower * 3 or 0)
			local flung = horizontal > limit or v.Y > yLimit
				or root.AssemblyAngularVelocity.Magnitude > spinLimit
			if flung then
				if Settings.AntiFlingRestore and stableOld and not fastOwn then
					LocalPlayer.Character:PivotTo(stableOld)
				end
				root.AssemblyLinearVelocity = Vector3.zero
				root.AssemblyAngularVelocity = Vector3.zero
				for part in pairs(myParts) do
					if part:IsA("BasePart") and part ~= root then
						part.AssemblyLinearVelocity = Vector3.zero
						part.AssemblyAngularVelocity = Vector3.zero
					end
				end
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
					notify("Antifling: arremesso bloqueado", "info")
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
				end
			elseif hum.FloorMaterial ~= Enum.Material.Air then
				lastSafe = root.CFrame + Vector3.new(0, 3, 0)
			end
		end

		Fling.step()
	end)
end)

local ANTIFLING_RADIUS = 60
local nearAcc, nearList = 0, {}
local noclipTouched = setmetatable({}, { __mode = "k" })
local noclipWasOn = false
local antiTouched = setmetatable({}, { __mode = "k" })
local antiWasOn = false

RunService.Stepped:Connect(function(_, dt)
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

	if Settings.AntiFling then
		antiWasOn = true
		nearAcc += dt
		if nearAcc >= 0.1 then
			nearAcc = 0
			table.clear(nearList)
			local myRoot = getLocalRoot()
			if myRoot then
				local myPos = myRoot.Position
				for _, data in pairs(otherChars) do
					local r = data.Root
					if r and r.Parent and data.Parts and (r.Position - myPos).Magnitude <= ANTIFLING_RADIUS then
						nearList[#nearList + 1] = data
					end
				end
			end
		end
		for i = 1, #nearList do
			if nearList[i].Parts then
				for part in pairs(nearList[i].Parts) do
					if part.CanCollide then
						part.CanCollide = false
						antiTouched[part] = true
					end
				end
			end
		end
	elseif antiWasOn then
		antiWasOn = false
		table.clear(nearList)
		for part in pairs(antiTouched) do
			if part.Parent then part.CanCollide = true end
			antiTouched[part] = nil
		end
	end
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
					local p = d.Position
					xs[#xs + 1] = p.X
					zs[#zs + 1] = p.Z
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
				notify("Anti Kill: limites do mapa calculados", "info")
			end
			scanning = false
		end)
	end

	AntiKill.recalc = function()
		bounds = nil
		scanMap()
	end

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

	local function rescue(root, why)
		local now = os.clock()
		lastRescue = now
		antiKillHold = now + 0.5
		LocalPlayer.Character:PivotTo(safeTarget())
		root.AssemblyLinearVelocity = Vector3.zero
		root.AssemblyAngularVelocity = Vector3.zero
		for part in pairs(myParts) do
			part.AssemblyLinearVelocity = Vector3.zero
			part.AssemblyAngularVelocity = Vector3.zero
		end
		if flyObjs and flyObjs.LV then flyObjs.LV.VectorVelocity = Vector3.zero end
		if now - notifyAt > 2 then
			notifyAt = now
			notify("Anti Kill: trazido de volta (" .. why .. ")", "info")
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
				if airborne and now >= nextFall
					and (vel.Y < -40 or (Settings.Noclip and vel.Y < -12)) then
					nextFall = now + 0.05
					if noGround(pos) and noGround(pos + Vector3.new(vel.X, 0, vel.Z) * 0.3) then
						danger, why = true, "queda sem chão"
					end
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
				if now - lastRescue >= 0.1 then rescue(root, why) end
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
		local vu = game:GetService("VirtualUser")
		vu:CaptureController()
		vu:ClickButton2(Vector2.new())
	end)
end)

LocalPlayer.CharacterAdded:Connect(function() spectating = false end)

local Lit = Lighting
local LIT_GROUPS = {
	FullBright = { "Brightness", "ClockTime", "GlobalShadows", "Ambient", "OutdoorAmbient" },
	NoFog = { "FogStart", "FogEnd" },
}
local litSaved = {}

local function litApply()
	for key, props in pairs(LIT_GROUPS) do
		if Settings[key] then
			if not litSaved[key] then
				local saved = {}
				for _, p in ipairs(props) do saved[p] = Lit[p] end
				litSaved[key] = saved
			end
		elseif litSaved[key] then
			for p, v in pairs(litSaved[key]) do
				pcall(function() Lit[p] = v end)
			end
			litSaved[key] = nil
		end
	end
	if Settings.FullBright then
		Lit.Brightness = 2
		Lit.ClockTime = 14
		Lit.GlobalShadows = false
		Lit.Ambient = Color3.fromRGB(178, 178, 178)
		Lit.OutdoorAmbient = Color3.fromRGB(178, 178, 178)
	end
	if Settings.NoFog then
		Lit.FogStart = 1000000
		Lit.FogEnd = 1000000
		local atm = Lit:FindFirstChildOfClass("Atmosphere")
		if atm and atm.Density ~= 0 then atm.Density = 0 end
	end
end

local litAcc = 0
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
-- INTERFACE
--------------------------------------------------------------------
local function buildUI()

local TAB_TOTAL = 8
local secStatus = nil

local menu = Instance.new("CanvasGroup")
menu.Name = "Menu"
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
local gradientTween = TweenService:Create(
	accentGradient,
	TweenInfo.new(2.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
	{ Offset = Vector2.new(0.5, 0) })
gradientTween:Play()

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
title.Text = "Test Toolkit v59.5"
title.Parent = titleBar

local subtitle = Instance.new("TextLabel")
subtitle.BackgroundTransparency = 1
subtitle.Position = UDim2.fromOffset(16, 26)
subtitle.Size = UDim2.new(1, -60, 0, 16)
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 12
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.TextColor3 = Theme.SubText
subtitle.Text = "Ctrl direito abre/fecha  •  arraste aqui para mover"
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
		if n ~= "buttons" or isMobile then vis[#vis + 1] = n end
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

local refreshers = {}

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
	end	corner(r, 8)
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
	row.MouseButton1Down:Connect(function() tween(scale, { Scale = 0.97 }, 0.07) end)
	row.MouseButton1Up:Connect(function() tween(scale, { Scale = 1 }, 0.16, Enum.EasingStyle.Back) end)
end

local UPPER_ACCENTS = {
	["á"] = "Á", ["à"] = "À", ["â"] = "Â", ["ã"] = "Ã", ["é"] = "É", ["ê"] = "Ê",
	["í"] = "Í", ["ó"] = "Ó", ["ô"] = "Ô", ["õ"] = "Õ", ["ú"] = "Ú", ["ç"] = "Ç",
}
local function upperPT(t)
	return (string.gsub(t, utf8.charpattern, function(c)
		return UPPER_ACCENTS[c] or string.upper(c)
	end))
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
	l.Text = upperPT(text)
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
		currentTarget = nil
		table.clear(targetCache)
		candTime = -1
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

	hit.MouseEnter:Connect(function()
		tween(handle, { Size = UDim2.fromOffset(16, 16) }, 0.12, Enum.EasingStyle.Back)
		tween(valueLabel, { TextColor3 = Theme.Text }, 0.12)
	end)
	hit.MouseLeave:Connect(function()
		if sliderDrag ~= setFromX then
			tween(handle, { Size = UDim2.fromOffset(12, 12) }, 0.12)
			tween(valueLabel, { TextColor3 = Theme.SubText }, 0.12)
		end
	end)
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
local keybindRows = {}

local function addKeybind(text, key, desc, allowMouse)
	local row = newRow(desc and 42 or 36, "TextButton")
	row.Visible = not isMobile
	table.insert(keybindRows, row)
	hover(row)
	local l = rowLabel(row, "", desc, 30)
	local function render() l.Text = text .. ": [" .. Settings[key].Name .. "]" end
	render()
	row.MouseButton1Click:Connect(function()
		l.Text = text .. ": pressione uma tecla... (Esc cancela)"
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

local function addColorPicker(text, key, desc)
	local row = newRow(desc and 66 or 50, "TextButton")
	hover(row)
	rowLabel(row, text, desc, 60)
	local preview = Instance.new("Frame")
	preview.AnchorPoint = Vector2.new(1, 0.5)
	preview.Position = UDim2.new(1, -12, 0.5, 0)
	preview.Size = UDim2.fromOffset(44, 26)
	preview.BackgroundColor3 = Settings[key]
	preview.BorderSizePixel = 0
	preview.Parent = row
	corner(preview, 6)
	stroke(preview, Color3.new(1, 1, 1), 0.4, 1)

	local hue = Instance.new("TextLabel")
	hue.BackgroundTransparency = 1
	hue.AnchorPoint = Vector2.new(1, 1)
	hue.Position = UDim2.new(1, -12, 0.5, -18)
	hue.Size = UDim2.fromOffset(100, 12)
	hue.Font = Enum.Font.Gotham
	hue.TextSize = 10
	hue.TextColor3 = Theme.SubText
	hue.TextXAlignment = Enum.TextXAlignment.Right
	hue.Text = string.format("R:%.0f G:%.0f B:%.0f",
		Settings[key].R * 255, Settings[key].G * 255, Settings[key].B * 255)
	hue.Parent = row

	local function update()
		preview.BackgroundColor3 = Settings[key]
		hue.Text = string.format("R:%.0f G:%.0f B:%.0f",
			Settings[key].R * 255, Settings[key].G * 255, Settings[key].B * 255)
	end

	row.MouseButton1Click:Connect(function()
		-- Cicla por cores pré-definidas
		local presets = {
			Color3.fromRGB(0, 255, 0),
			Color3.fromRGB(0, 200, 0),
			Color3.fromRGB(0, 150, 0),
			Color3.fromRGB(50, 255, 50),
			Color3.fromRGB(255, 0, 0),
			Color3.fromRGB(200, 0, 0),
			Color3.fromRGB(255, 50, 50),
			Color3.fromRGB(0, 100, 255),
			Color3.fromRGB(0, 150, 255),
			Color3.fromRGB(100, 150, 255),
			Color3.fromRGB(255, 255, 0),
			Color3.fromRGB(255, 200, 0),
			Color3.fromRGB(255, 255, 255),
		}
		local cur = Settings[key]
		local idx = 1
		for i, c in ipairs(presets) do
			if math.abs(c.R - cur.R) < 0.01 and math.abs(c.G - cur.G) < 0.01
				and math.abs(c.B - cur.B) < 0.01 then
				idx = (i % #presets) + 1
				break
			end
		end
		Settings[key] = presets[idx]
		update()
		notify(text .. " alterado", "info")
	end)

	table.insert(refreshers, update)
end

--------------------------------------------------------------------
-- PERFORMANCE + FPS
--------------------------------------------------------------------
local Terrain = Workspace.Terrain
local perfSaved = setmetatable({}, { __mode = "k" })
local perfToken = 0
local perfConns = {}
local perfQuality = nil

local function perfSet(inst, prop, value)
	pcall(function()
		local s = perfSaved[inst]
		if not s then s = {}; perfSaved[inst] = s end
		if s[prop] == nil then s[prop] = inst[prop] end
		inst[prop] = value
	end)
end

local function perfInstance(d)
	if d == Terrain then return end
	if d:IsA("BasePart") then
		perfSet(d, "Material", Enum.Material.SmoothPlastic)
		perfSet(d, "MaterialVariant", "")
		perfSet(d, "CastShadow", false)
		perfSet(d, "Reflectance", 0)
	elseif d:IsA("Light") then
		perfSet(d, "Enabled", false)
	elseif d:IsA("PostEffect") or d:IsA("Clouds") then
		perfSet(d, "Enabled", false)
	elseif d:IsA("Atmosphere") then
		perfSet(d, "Density", 0)
		perfSet(d, "Haze", 0)
	end
end

local function perfEnable()
	perfSet(Terrain, "Decoration", false)
	perfSet(Terrain, "WaterWaveSize", 0)
	perfSet(Terrain, "WaterWaveSpeed", 0)
	perfSet(Terrain, "WaterReflectance", 0)
	pcall(function()
		local r = settings().Rendering
		perfQuality = perfQuality or r.QualityLevel
		r.QualityLevel = Enum.QualityLevel.Level01
	end)
	perfSet(Lit, "GlobalShadows", false)
	perfSet(Lit, "Brightness", 0)
	perfSet(Lit, "Ambient", Color3.fromRGB(190, 190, 190))
	perfSet(Lit, "OutdoorAmbient", Color3.fromRGB(190, 190, 190))
	perfSet(Lit, "ShadowSoftness", 0)
	perfSet(Lit, "EnvironmentDiffuseScale", 0)
	perfSet(Lit, "EnvironmentSpecularScale", 0)
	for _, d in ipairs(Lit:GetDescendants()) do perfInstance(d) end
	perfConns[1] = Lit.DescendantAdded:Connect(perfInstance)
	perfConns[2] = Workspace.DescendantAdded:Connect(perfInstance)
	local token = perfToken
	task.spawn(function()
		local n = 0
		for _, d in ipairs(Workspace:GetDescendants()) do
			if token ~= perfToken then return end
			perfInstance(d)
			n += 1
			if n % 400 == 0 then task.wait() end
		end
	end)
end

local function perfDisable()
	for _, c in ipairs(perfConns) do c:Disconnect() end
	table.clear(perfConns)
	for inst, props in pairs(perfSaved) do
		if inst.Parent then
			for prop, value in pairs(props) do
				pcall(function() inst[prop] = value end)
			end
		end
		perfSaved[inst] = nil
	end
	if perfQuality then
		pcall(function() settings().Rendering.QualityLevel = perfQuality end)
		perfQuality = nil
	end
end

local function applyPerformance()
	perfToken += 1
	if Settings.PerfMode then perfEnable() else perfDisable() end
end

local lastFps = 60

local function lookupGlobal(name)
	local sources = {
		function() return getgenv()[name] end,
		function() return _G[name] end,
	}
	for _, f in ipairs(sources) do
		local ok, v = pcall(f)
		if ok and type(v) == "function" then return v end
	end
	return nil
end

local function fpsMethodList()
	local list = {}
	for _, n in ipairs({ "setfpscap", "set_fps_cap", "setfps" }) do
		local fn = lookupGlobal(n)
		if fn then list[#list + 1] = { name = n, fn = fn } end
	end
	local okSyn, syn = pcall(function() return getgenv().syn end)
	if okSyn and type(syn) == "table" and type(syn.set_fps_cap) == "function" then
		list[#list + 1] = { name = "syn.set_fps_cap", fn = syn.set_fps_cap }
	end
	local sf = lookupGlobal("setfflag")
	if sf then
		list[#list + 1] = { name = "setfflag", fn = function(c)
			sf("DFIntTaskSchedulerTargetFps", tostring(c))
		end }
	end
	list[#list + 1] = { name = "FramerateCap", fn = function(c)
		UserSettings():GetService("UserGameSettings").FramerateCap = c
	end }
	return list
end

local fpsMethod = nil
local fpsBusy = false

local function fpsValue(cap)
	if Settings.FPSUnlock and fpsMethod and fpsMethod.zero then return 0 end
	return cap
end

local function applyFps(explicit)
	if not Settings.FPSUnlock and not explicit then return end
	local cap = Settings.FPSUnlock and Settings.FPSCap or 60
	if fpsMethod then
		pcall(fpsMethod.fn, fpsValue(cap))
		if explicit then
			notify("Limite de FPS: " .. cap .. " (" .. fpsMethod.name .. ")", "info")
		end
		return
	end
	if fpsBusy then return end
	local list = fpsMethodList()
	if #list == 0 then
		if Settings.FPSUnlock then
			Settings.FPSUnlock = false
			for _, refresh in ipairs(refreshers) do refresh() end
			notify("FPS: este executor não tem função de limite", "off")
		end
		return
	end
	fpsBusy = true
	task.spawn(function()
		local fallback = nil
		for _, m in ipairs(list) do
			for _, zero in ipairs({ false, true }) do
				local value = (zero and Settings.FPSUnlock) and 0 or cap
				if pcall(m.fn, value) then
					fallback = fallback or { name = m.name, fn = m.fn, zero = false }
					if cap <= 60 or not Settings.FPSUnlock then
						fpsMethod = fallback
						fpsBusy = false
						if explicit then
							notify("Limite de FPS: " .. cap .. " (" .. m.name .. ")", "info")
						end
						return
					end
					task.wait(1.5)
					if lastFps > 65 then
						fpsMethod = { name = m.name, fn = m.fn, zero = zero }
						fpsBusy = false
						notify("FPS desbloqueado via " .. m.name .. (zero and " (sem limite)" or ""), "on")
						return
					end
				end
			end
		end
		fpsBusy = false
		if fallback then
			fpsMethod = fallback
			notify("Limite aplicado, mas FPS não passou de " .. math.floor(lastFps), "off")
		end
	end)
end

task.spawn(function()
	while true do
		task.wait(2)
		if Settings.FPSUnlock then applyFps() end
	end
end)

--------------------------------------------------------------------
-- EQUIPE AUX
--------------------------------------------------------------------
local function resetAimCaches()
	currentTarget = nil
	table.clear(targetCache)
	table.clear(teamCache)
	myTeamAt = -1
	candTime = -1
	MM2.clearCache()
end

local function markCurrentAsAlly()
	local model = currentTarget and currentTarget.Model
	if not model and lastLocked and lastLocked.Parent and os.clock() - lastLockedAt < 15 then
		model = lastLocked
	end
	if not model then
		local myRoot = getLocalRoot()
		local bd = math.huge
		if myRoot then
			for _, t in ipairs(getTargets("esp")) do
				local d = (t.Root.Position - myRoot.Position).Magnitude
				if d < bd and not manualAllies[t.Player or t.Model] then
					model, bd = t.Model, d
				end
			end
		end
	end
	if not model then
		notify("Nenhum alvo para marcar", "off")
		return
	end
	local plr = Players:GetPlayerFromCharacter(model)
	manualAllies[plr or model] = true
	resetAimCaches()
	notify("Aliado marcado: " .. (plr and plr.DisplayName or model.Name), "on")
end

local function clearManualAllies()
	for k in pairs(manualAllies) do manualAllies[k] = nil end
	resetAimCaches()
	notify("Aliados manuais limpos", "info")
end

local function teamDiagnostic()
	local mine = getTeamKey(LocalPlayer.Character, LocalPlayer)
	notify("Eu: " .. (mine and mine:sub(3) or "sem equipe detectada"), "info")
	local myRoot = getLocalRoot()
	local best, bd = nil, math.huge
	if myRoot then
		for model, hum in pairs(humanoids) do
			if model ~= LocalPlayer.Character and hum.Health > 0 then
				local r = getRoot(model)
				if r then
					local d = (r.Position - myRoot.Position).Magnitude
					if d < bd then best, bd = model, d end
				end
			end
		end
	end
	if best then
		local plr = Players:GetPlayerFromCharacter(best)
		local k = getTeamKey(best, plr)
		notify(best.Name .. ": " .. (k and k:sub(3) or "sem equipe")
			.. (isTeammate(best, plr) and " (aliado)" or " (inimigo)"), "info")
	else
		notify("Nenhum jogador/bot por perto", "off")
	end
end

--------------------------------------------------------------------
-- CONFIG + SPECTATE
--------------------------------------------------------------------
local function saveConfig()
	local ok = pcall(function()
		if not writefile then error("sem writefile") end
		writefile(CONFIG_FILE, HttpService:JSONEncode(serializeSettings()))
	end)
	notify(ok and "Configuração salva" or "Salvar indisponível", ok and "on" or "off")
end

local function deleteConfig()
	local ok = pcall(function()
		if delfile and isfile and isfile(CONFIG_FILE) then delfile(CONFIG_FILE)
		else error("nada para apagar") end
	end)
	notify(ok and "Configuração apagada" or "Nada para apagar", ok and "info" or "off")
end

local spectateIdx = 0

local function stopSpectate()
	if not spectating then return end
	spectating = false
	local hum = getLocalHumanoid()
	if hum then Camera.CameraSubject = hum end
	notify("Observação parada", "off")
end

local function spectateNext()
	local list = {}
	for model, hum in pairs(humanoids) do
		if model ~= LocalPlayer.Character and hum.Health > 0 and model.Parent then
			list[#list + 1] = model
		end
	end
	table.sort(list, function(a, b) return a.Name < b.Name end)
	if #list == 0 then notify("Ninguém para observar", "off"); return end
	spectateIdx = (spectateIdx % #list) + 1
	local m = list[spectateIdx]
	spectating = true
	currentTarget = nil
	Camera.CameraSubject = humanoids[m]
	local plr = Players:GetPlayerFromCharacter(m)
	notify("Observando: " .. (plr and plr.DisplayName or m.Name), "info")
end

--------------------------------------------------------------------
-- MENU OPEN/CLOSE
--------------------------------------------------------------------
local menuOpen = false

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

local function toggleMenu() setMenu(not menuOpen) end

closeBtn.MouseButton1Click:Connect(function() setMenu(false) end)

local function refreshAll()
	for _, refresh in ipairs(refreshers) do refresh() end
	if secStatus and secStatus.Parent then
		local parts = {}
		if Settings.AntiVoid then parts[#parts + 1] = "AntiVoid" end
		if Settings.AntiKill then parts[#parts + 1] = "AntiKill" end
		if Settings.AntiFling then parts[#parts + 1] = "AntiFling" end
		if Settings.AntiAFK then parts[#parts + 1] = "AntiAFK" end
		secStatus.Text = #parts == 0
			and "Nenhuma proteção ativa — use os toggles abaixo"
			or ("Ativas: " .. table.concat(parts, ", "))
	end
end

uiRefresh = refreshAll

local function toggleSetting(key, label)
	Settings[key] = not Settings[key]
	refreshAll()
	notify(label .. (Settings[key] and ": ligado" or ": desligado"), Settings[key] and "on" or "off")
end

--------------------------------------------------------------------
-- ABA: MIRA
--------------------------------------------------------------------
newPage("aim", "Mira")

addSection("Modo de mira")
addToggle("Aimbot", "AimEnabled", "Ativa a mira automática")
addToggle("Legit Aim", "UseLegitAim", "Move a câmera suavemente para o alvo", resetAimCaches)
addToggle("Silent Aim", "UseSilentAim", "Redireciona o tiro pro alvo sem mexer na câmera")
addCycle("Modo", "AimMode", { "Segurar", "Alternar", "Automático" },
	"Segurar/alternar pela tecla da mira; Automático = sempre", function() aimActive = false end)
addCycle("Parte do corpo", "AimPart", { "Cabeça", "Corpo", "Automático" },
	"Automático mira a parte visível mais perto do cursor")
addCycle("Prioridade", "Priority", { "Perto do cursor", "Perto de mim", "Menos vida" },
	"Qual inimigo é escolhido primeiro")
addCycle("Alvos", "TargetMode", { "Jogadores", "Bots", "Ambos" },
	"Quem a mira e o ESP consideram alvo")
addToggle("Ignorar equipe", "TeamCheck", "Não mira em aliados", resetAimCaches)
addToggle("Checar parede", "AimWallCheck", "Só mira em quem você enxerga")

addSection("Legit Aim")
addSlider("Suavidade", "Smoothness", 0.01, 0.95, 0.01, 2, "Menor = mira mais rápida e firme")
addSlider("Ângulo de trava", "SnapAngle", 0, 30, 1, 0, "Abaixo deste ângulo cola no alvo")
addToggle("Trava firme ao atirar", "FireLock", "Sem suavização enquanto atira")
addToggle("Mirar no cursor", "AimAtCursor", "PC: alvo sob o cursor")

addSection("Silent Aim")
addSlider("FOV do Silent", "SilentAimFOV", 50, 800, 10, 0, "Raio do Silent Aim")
addToggle("Só alvos visíveis", "SilentAimVisible", "Só atira em alvos com linha de visão")

addSection("Trigger Bot")
addToggle("Trigger Bot", "TriggerBot", "Atira automaticamente quando o alvo está sob a mira")
addToggle("Sempre ativo", "TriggerBotAlways", "Atira mesmo sem segurar a tecla da mira")
addToggle("Só alvos visíveis", "TriggerVisible", "Só atira se tiver linha de visão")
addSlider("FOV do Trigger", "TriggerFOV", 5, 400, 1, 0, "5 = bolinha no centro, 400 = círculo grande")
addSlider("Delay entre tiros", "TriggerDelay", 0.01, 1, 0.01, 2, "Tempo mínimo entre cada tiro")

addSection("Tiro e previsão")
addCycle("Tipo de tiro", "ShotType", { "Instantâneo", "Projétil", "Automático" },
	"Instantâneo = metade do ping; Projétil = prevê voo")
addToggle("Previsão automática", "AutoPredict", "Compensa ping e movimento")
addSlider("Força da previsão", "PredictScale", 0, 2, 0.05, 2, "Ajuste fino")
addSlider("Compensação manual", "Prediction", 0, 0.5, 0.01, 2, "Segundos à frente")
addSlider("Velocidade da bala", "BulletSpeed", 0, 1000, 10, 0, "0 = sem previsão de voo")

addSection("FOV do Legit")
addToggle("Limitar pelo FOV", "FOVEnabled", "Só mira dentro do círculo")
addSlider("Raio do FOV", "FOVRadius", 20, 600, 5, 0, "Pixels")
addSlider("Distância máxima", "AimMaxDist", 0, 2000, 50, 0, "0 = sem limite")

table.insert(keybindRows, addSection("Teclas (PC)"))
addKeybind("Tecla da mira", "AimKey", "Botão que ativa a mira", true)
addKeybind("Trocar de alvo", "SwitchKey", "Próximo inimigo")

addSection("Equipe e observação")
addButton("Marcar alvo como aliado", "Mira e ESP ignoram", markCurrentAsAlly)
addButton("Limpar aliados marcados", "Remove marcações", clearManualAllies)
addButton("Diagnóstico de equipe", "Mostra equipes", teamDiagnostic)
addButton("Observar próximo", "Câmera segue outro", spectateNext)
addButton("Parar de observar", "Volta câmera", stopSpectate)

addSection("Aviso")
addInfo("Legit Aim = seguro. Silent = detectável. Hitbox = muito detectável. "
	.. "Use só em servidor privado seu.", 60)

--------------------------------------------------------------------
-- ABA: HITBOX
--------------------------------------------------------------------
newPage("hitbox", "Hitbox")

addSection("Hitbox Expander")
addToggle("Hitbox Expander", "HitboxExpander", "Aumenta a hitbox dos players próximos", function()
	if not Settings.HitboxExpander then
		pcall(HitboxExpander.restoreAll)
	end
end)
addToggle("Invisível (não muda visual)", "HitboxInvisible", "Hitbox fica transparente")
addToggle("Checar parede", "HitboxWallCheck", "Só expande alvos visíveis")
addToggle("Ignorar aliados", "HitboxIgnoreAllies", "Não expande aliados")
addSlider("Tamanho da hitbox", "HitboxSize", 5, 30, 1, 0, "Tamanho em studs")
addSlider("Alcance da hitbox", "HitboxRange", 10, 1000, 10, 0, "Distância máxima para expandir")

addSection("Partes para expandir")
addToggle("Cabeça (Head)", "ExpandHead", "Expande a cabeça")
addToggle("Torso (R6)", "ExpandTorso", "Expande o Torso (R6)")
addToggle("UpperTorso (R15)", "ExpandUpperTorso", "Expande o UpperTorso (R15)")
addToggle("LowerTorso (R15)", "ExpandLowerTorso", "Expande o LowerTorso (R15)")
addInfo("HumanoidRootPart nunca é expandido — evita puxão de física", 20)

addSection("Aviso")
addInfo("Hitbox é MUITO detectável. Use somente em servidor privado seu.", 40)

--------------------------------------------------------------------
-- ABA: ESP
--------------------------------------------------------------------
newPage("esp", "ESP")

addSection("ESP")
addToggle("ESP", "ESP", "Mostra alvos")
addToggle("Ver através de paredes", "Wallhack", "Contorno atrás de objetos")
addToggle("Contorno colorido", "ESPHighlight", "Mais pesado; desligue se cair FPS")
addToggle("Cor arco-íris", "Rainbow", "Cor animada")
addToggle("Mostrar aliados", "ESPTeammates", "Aliados em azul")
addCycle("Caixa", "ESPBox", { "Desligada", "Completa", "Cantos" }, "Moldura")
addToggle("Nomes e distância", "ShowNames", "Texto acima")
addToggle("Barra de vida", "ShowHealth", "Barra verde/vermelha")
addToggle("Ponto na cabeça", "ESPHeadDot", "Marca cabeça")
addToggle("Item na mão", "ESPTool", "Ferramenta do alvo")

addSection("Linhas (tracers)")
addToggle("Linhas até os alvos", "Tracers", "Linha da tela")
addCycle("Origem da linha", "TracerOrigin", { "Baixo", "Centro", "Cursor" }, "De onde sai")

addSection("Limites")
addSlider("Distância máxima", "ESPMaxDist", 0, 3000, 50, 0, "0 = sem limite")
addSlider("Máx. de alvos", "ESPMaxTargets", 1, 30, 1, 0, "Menos = mais FPS")
addSlider("Transparência do preenchimento", "ESPFillTrans", 0, 1, 0.05, 2, "0 = sólido")

addSection("Murder Mystery 2")
addInfo("Mostra a função de cada jogador com cor específica. "
	.. "Inocente = Verde, Murder = Vermelho, Sheriff = Azul. "
	.. "Arma dropada = Amarelo.", 54)
addToggle("Ativar modo MM2", "MM2Mode", "Detecta e colore por função")
addColorPicker("Cor do Inocente", "MM2InnocentColor", "Verde por padrão")
addColorPicker("Cor do Murder", "MM2MurderColor", "Vermelho por padrão")
addColorPicker("Cor do Sheriff", "MM2SheriffColor", "Azul por padrão")
addToggle("Destacar arma dropada", "MM2ShowDroppedGun", "Mostra a arma no chão")
addColorPicker("Cor da arma dropada", "MM2DroppedGunColor", "Amarelo por padrão")

--------------------------------------------------------------------
-- ABA: JOGADOR
--------------------------------------------------------------------
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
addToggle("Voo seguro (auto)", "FlyAutoLimit", "Anti-puxão do voo")
addToggle("Noclip", "Noclip", "Atravessa paredes")

table.insert(keybindRows, addSection("Teclas (PC)"))
addKeybind("Tecla do voo", "FlyKey", "Liga/desliga voo")
addKeybind("Tecla do noclip", "NoclipKey", "Liga/desliga noclip")

addSection("Invisibilidade (Seat Bug)")
addInfo("⚠ Usa bug do Seat pra te tirar da sincronia do servidor. "
	.. "Funciona em jogos sem validação de posição server-side. "
	.. "Se for kickado/revertido, o jogo tem anti-cheat server-side.", 60)
addToggle("Invisível (Seat Bug)", "SeatInvisible",
	"Ativa a invisibilidade via bug do Seat (réplica do script Ziaa)", function()
		pcall(SeatInvisible.toggle)
	end)
addButton("Auto-detectar coordenadas", "Procura um lugar seguro no mapa", function()
	local pos = SeatInvisible.findSafeCoords()
	Settings.SeatInvisibleX = math.floor(pos.X)
	Settings.SeatInvisibleY = math.floor(pos.Y)
	Settings.SeatInvisibleZ = math.floor(pos.Z)
	if uiRefresh then uiRefresh() end
	notify("Coordenadas: " .. math.floor(pos.X) .. ", "
		.. math.floor(pos.Y) .. ", " .. math.floor(pos.Z), "on")
end)
addSlider("Coord X", "SeatInvisibleX", -10000, 10000, 1, 0, "Posição X")
addSlider("Coord Y", "SeatInvisibleY", -500, 5000, 1, 0, "Posição Y")
addSlider("Coord Z", "SeatInvisibleZ", -10000, 10000, 1, 0, "Posição Z")
addSlider("Duração (0 = infinito)", "SeatInvisibleDuration", 0, 300, 1, 0, "Segundos")
addToggle("Voltar ao desligar", "SeatInvisibleReturn", "Retorna pra posição original")

addSection("Proteção")
addInfo("Todas as proteções também estão na aba SEGURANÇA. Use-a para controle rápido.", 32)

--------------------------------------------------------------------
-- ABA: EXTRAS
--------------------------------------------------------------------
newPage("misc", "Extras")

addSection("Visão")
addToggle("Luz total", "FullBright", "Remove escuridão")
addToggle("Sem neblina", "NoFog", "Enxerga longe")
addToggle("FOV da câmera", "CamFOVOn", "Muda FOV")
addSlider("Valor do FOV", "CamFOV", 40, 120, 1, 0, "Padrão 70")
addToggle("Anti-AFK", "AntiAFK", "Evita kick por inatividade")

addSection("Interface")
addToggle("HUD", "ShowHUD", "FPS, ping e estado")
addToggle("Mover HUD", "HUDEdit", "Arraste o HUD")
addButton("Resetar posição do HUD", "Volta pro canto", function()
	Settings.HUDX, Settings.HUDY = 10, 10
	notify("HUD voltou pro canto", "info")
end)
addToggle("Notificações", "Notifications", "Avisos ao ligar/desligar")
addCycle("Dispositivo", "DeviceMode", { "Automático", "PC", "Mobile" }, "Layout", refreshDevice)
addSlider("Transparência do menu", "MenuAlpha", 0, 0.6, 0.05, 2, "0 = sólido", function()
	if menuOpen then menu.GroupTransparency = Settings.MenuAlpha end
end)

addSection("Desempenho")
addToggle("Modo desempenho", "PerfMode", "Gráficos mínimos", applyPerformance)
addToggle("Desbloquear FPS", "FPSUnlock", "Tira limite 60", function() applyFps(true) end)
addSlider("Limite de FPS", "FPSCap", 60, 10000, 10, 0, "Com desbloqueio ligado", function() applyFps() end)

addSection("Configuração")
addButton("Salvar configuração", "Guarda as opções", saveConfig)
addButton("Apagar configuração", "Volta ao padrão", deleteConfig)

--------------------------------------------------------------------
-- ABA: FLING
--------------------------------------------------------------------
newPage("fling", "Fling")

addSection("Fling")
addToggle("Fling ativo", "Fling", "Arremessa BOTS (players não funciona em jogos modernos)")
addCycle("Modo de alvo", "FlingMode", { "Todos perto", "Alvo da mira", "Mais próximo" },
	"Qual bot é arremessado")
addCycle("Tipo de arremesso", "FlingMode2", { "Empurrar (leve)", "Mandar pro void" },
	"Empurrar: voa e pode sobreviver. Void: morre ao cair")

addSection("Ajuste")
addSlider("Alcance", "FlingRange", 3, 30, 1, 0, "Studs")
addSlider("Força", "FlingPower", 500, 5000, 100, 0, "Empurrar: distância. Void: ignora")
addSlider("Intervalo", "FlingRepeat", 0.05, 1, 0.05, 2, "Tempo entre aplicações")

table.insert(keybindRows, addSection("Teclas (PC)"))
addKeybind("Tecla do Fling", "FlingKey", "Liga/desliga Fling (padrão G)")

addSection("Dicas")
addInfo("Fling SÓ funciona em BOTS. Em players é impossível sem arremessar você junto. "
	.. "Desligue o AntiFling ao usar.", 70)

--------------------------------------------------------------------
-- ABA: SEGURANÇA
--------------------------------------------------------------------
newPage("security", "Segurança")

addSection("Status")
secStatus = addInfo("Carregando estado...", 44)

addSection("Proteção contra morte")
addToggle("Anti Void", "AntiVoid", "Volta à última posição segura se cair no vazio")
addToggle("Anti Kill", "AntiKill", "Não deixa morrer ao sair do mapa / cair no vazio")
addSlider("Folga do Anti Kill", "AntiKillMargin", 0, 1000, 25, 0,
	"Distância além da borda do mapa; 0 = só vazio")
addButton("Recalcular limites do mapa", "Use depois que o mapa mudar", function()
	AntiKill.recalc()
end)

addSection("Proteção contra arremesso")
addToggle("Anti Fling", "AntiFling", "Bloqueia arremessos e colisão de outros")
addToggle("Voltar ao ponto estável", "AntiFlingRestore", "Após um arremesso, retorna onde estava")
addSlider("Limite do Anti Fling", "AntiFlingSpeed", 50, 500, 10, 0,
	"Velocidade que conta como arremesso")

addSection("Proteção contra kick")
addToggle("Anti AFK", "AntiAFK", "Evita ser expulso por inatividade")

addSection("Movimento seguro")
addToggle("Velocidade segura (auto)", "WalkAutoLimit",
	"Se o jogo te puxar de volta, baixa a velocidade")
addToggle("Voo seguro (auto)", "FlyAutoLimit",
	"Se o jogo te puxar de volta, baixa a velocidade do voo")

addSection("Recomendado")
addInfo("Para fling em bots: desligue 'Anti Fling'. "
	.. "Para uso normal: ligue tudo. Anti Kill + Anti Void protegem contra morte por queda.", 70)

--------------------------------------------------------------------
-- ABA: BOTÕES (mobile)
--------------------------------------------------------------------
newPage("buttons", "Botões")

addSection("Botões na tela")
addToggle("Botão MIRA", "ShowBtnAim", "Ativa/segura mira")
addToggle("Botão VOO", "ShowBtnFly", "Liga/desliga voo")
addToggle("Botão NOCLIP", "ShowBtnNoclip", "Liga/desliga noclip")
addToggle("Botão DESCER", "ShowBtnDown", "Desce voando")
addToggle("Botão ESP", "ShowBtnEsp", "Liga/desliga ESP")
addToggle("Botão LINHA", "ShowBtnTrace", "Liga/desliga tracers")
addToggle("Botão PULO", "ShowBtnInf", "Liga/desliga InfJump")
addToggle("Botão FLING", "ShowBtnFling", "Liga/desliga Fling")

addSection("Aparência")
addToggle("Editar posições", "MobileEdit", "Arraste os botões")
addSlider("Tamanho dos botões", "MobileBtnSize", 40, 90, 1, 0, "Pixels")
addSlider("Transparência", "MobileBtnAlpha", 0, 0.9, 0.05, 2, "0 = sólido")

--------------------------------------------------------------------
-- BOTÕES MOBILE
--------------------------------------------------------------------
local mobileBtns = {}
local dragBtn, dragStart, dragFrom = nil, nil, nil

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
		if Settings.MobileEdit then
			dragBtn = b
			dragStart = input.Position
			dragFrom = b.AbsolutePosition + b.AbsoluteSize / 2
		elseif onDown then
			onDown()
		end
	end)
	b.InputEnded:Connect(function(input)
		if (input.UserInputType == Enum.UserInputType.Touch
			or input.UserInputType == Enum.UserInputType.MouseButton1)
			and onUp and not Settings.MobileEdit then
			onUp()
		end
	end)

	mobileBtns[#mobileBtns + 1] = { Btn = b, ShowKey = showKey, State = stateFn }
end

UserInputService.InputChanged:Connect(function(input)
	if dragBtn and (input.UserInputType == Enum.UserInputType.Touch
		or input.UserInputType == Enum.UserInputType.MouseMovement) then
		local d = input.Position - dragStart
		dragBtn.Position = UDim2.fromOffset(dragFrom.X + d.X, dragFrom.Y + d.Y)
	end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch
		or input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragBtn = nil
	end
end)

makeMobileBtn("TT", UDim2.fromScale(0.07, 0.2), nil, function() return menuOpen end, toggleMenu)
makeMobileBtn("MIRA", UDim2.fromScale(0.9, 0.42), "ShowBtnAim", function() return aimActive end, function()
	if Settings.AimMode == 1 then aimActive = true
	else aimActive = not aimActive end
end, function()
	if Settings.AimMode == 1 then aimActive = false end
end)
makeMobileBtn("VOO", UDim2.fromScale(0.9, 0.54), "ShowBtnFly", function() return Settings.Fly end,
	function() toggleSetting("Fly", "Voo") end)
makeMobileBtn("NOCLIP", UDim2.fromScale(0.9, 0.66), "ShowBtnNoclip", function() return Settings.Noclip end,
	function() toggleSetting("Noclip", "Noclip") end)
makeMobileBtn("DESCER", UDim2.fromScale(0.8, 0.54), "ShowBtnDown", function() return mobileFlyDown end,
	function() mobileFlyDown = true end, function() mobileFlyDown = false end)
makeMobileBtn("ESP", UDim2.fromScale(0.8, 0.66), "ShowBtnEsp", function() return Settings.ESP end,
	function() toggleSetting("ESP", "ESP") end)
makeMobileBtn("LINHA", UDim2.fromScale(0.8, 0.42), "ShowBtnTrace", function() return Settings.Tracers end,
	function() toggleSetting("Tracers", "Linhas") end)
makeMobileBtn("PULO", UDim2.fromScale(0.9, 0.78), "ShowBtnInf", function() return Settings.InfJump end,
	function() toggleSetting("InfJump", "Pulo infinito") end)
makeMobileBtn("FLING", UDim2.fromScale(0.8, 0.78), "ShowBtnFling", function() return Settings.Fling end,
	function() toggleSetting("Fling", "Fling") end)

local mobileShown = false
local function updateMobileBtns()
	if not isMobile then
		if mobileShown then
			mobileShown = false
			for _, m in ipairs(mobileBtns) do m.Btn.Visible = false end
		end
		return
	end
	mobileShown = true
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

local function applyDeviceLayout()
	for _, r in ipairs(keybindRows) do r.Visible = not isMobile end
	subtitle.Text = isMobile and "Botão TT abre/fecha  •  arraste aqui para mover"
		or "Ctrl direito abre/fecha  •  arraste aqui para mover"
	layoutTabs()
	if not isMobile and activeTab == "buttons" then selectTab("aim") end
	updateMobileBtns()
end
table.insert(deviceListeners, applyDeviceLayout)

layoutTabs()
selectTab("aim", true)
applyDeviceLayout()
refreshAll()

--------------------------------------------------------------------
-- TECLAS
--------------------------------------------------------------------
local function matchesBind(input, bind)
	if typeof(bind) ~= "EnumItem" then return false end
	if bind.EnumType == Enum.KeyCode then return input.KeyCode == bind end
	return input.UserInputType == bind
end

UserInputService.InputBegan:Connect(function(input)
	if rebinding then
		local r = rebinding
		if input.KeyCode == Enum.KeyCode.Escape then
			rebinding = nil
			r.apply(nil)
			return
		end
		local bind = nil
		if input.KeyCode ~= Enum.KeyCode.Unknown then
			bind = input.KeyCode
		elseif r.mouse and (input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.MouseButton2
			or input.UserInputType == Enum.UserInputType.MouseButton3) then
			bind = input.UserInputType
		end
		if bind then
			rebinding = nil
			r.apply(bind)
		end
		return
	end
	if UserInputService:GetFocusedTextBox() then return end

	if input.KeyCode == MENU_KEY then toggleMenu(); return end

	if matchesBind(input, Settings.AimKey) then
		if Settings.AimMode == 1 then aimActive = true
		elseif Settings.AimMode == 2 then
			aimActive = not aimActive
			notify(aimActive and "Mira ativada" or "Mira desativada", aimActive and "on" or "off")
		end
	elseif matchesBind(input, Settings.SwitchKey) then
		if isAiming() then switchTarget() end
	elseif matchesBind(input, Settings.NoclipKey) then
		toggleSetting("Noclip", "Noclip")
	elseif matchesBind(input, Settings.FlyKey) then
		toggleSetting("Fly", "Voo")
	elseif input.KeyCode == Settings.FlingKey then
		toggleSetting("Fling", "Fling")
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if Settings.AimMode == 1 and matchesBind(input, Settings.AimKey) then
		aimActive = false
	end
end)

--------------------------------------------------------------------
-- ESP
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
	l.ZIndex = 1
	l.Parent = parent
	return l
end

local function newEsp(model)
	local o = { Model = model, Seen = 0, H = 5.5, HAt = -1, Tool = "", ToolAt = -1 }
	local hl = Instance.new("Highlight")
	hl.Name = "TTHighlight"
	hl.Adornee = model
	hl.Enabled = false
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
	o.Name = mkText(holder, 12)
	o.Name.AnchorPoint = Vector2.new(0.5, 1)
	o.ToolLbl = mkText(holder, 11)
	o.ToolLbl.AnchorPoint = Vector2.new(0.5, 0)
	o.Dot = mkFrame(holder, true)
	o.Dot.AnchorPoint = Vector2.new(0.5, 0.5)
	o.Dot.Size = UDim2.fromOffset(6, 6)
	o.Tracer = mkFrame(holder)
	o.Tracer.AnchorPoint = Vector2.new(0.5, 0.5)
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

-- ESP para arma dropada
local droppedEsp = {}
local droppedAt = 0

local function makeDropEsp(tool)
	local hl = Instance.new("Highlight")
	hl.Name = "TTDropHL"
	hl.Adornee = tool
	hl.FillColor = Settings.MM2DroppedGunColor
	hl.OutlineColor = Settings.MM2DroppedGunColor
	hl.FillTransparency = 0.3
	hl.OutlineTransparency = 0
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Parent = gui

	local name = Instance.new("TextLabel")
	name.BackgroundTransparency = 1
	name.Font = Enum.Font.GothamBold
	name.TextSize = 14
	name.TextColor3 = Settings.MM2DroppedGunColor
	name.TextStrokeTransparency = 0.3
	name.Size = UDim2.fromOffset(160, 18)
	name.AnchorPoint = Vector2.new(0.5, 1)
	name.Text = "🔫 ARMA DROPADA"
	name.Visible = true
	name.ZIndex = 2
	name.Parent = espRoot

	return { HL = hl, Name = name, Tool = tool }
end

local function updateDroppedGuns(now)
	if not Settings.MM2ShowDroppedGun or not Settings.MM2Mode then
		for tool, o in pairs(droppedEsp) do
			pcall(function() o.HL:Destroy() end)
			pcall(function() o.Name:Destroy() end)
			droppedEsp[tool] = nil
		end
		return
	end
	if now - droppedAt < 0.3 then
		-- Apenas atualiza posição do texto
		for tool, o in pairs(droppedEsp) do
			if tool.Parent and o.HL and o.HL.Parent then
				local handle = tool:FindFirstChild("Handle")
				if handle and handle:IsA("BasePart") then
					local sp, on = Camera:WorldToViewportPoint(handle.Position + Vector3.new(0, 2, 0))
					o.Name.Visible = on
					if on then
						o.Name.Position = UDim2.fromOffset(sp.X, sp.Y)
					end
					o.HL.FillColor = Settings.MM2DroppedGunColor
					o.HL.OutlineColor = Settings.MM2DroppedGunColor
					o.Name.TextColor3 = Settings.MM2DroppedGunColor
				end
			end
		end
		return
	end
	droppedAt = now
	local guns = MM2.getDroppedGuns()
	local seen = {}
	for _, tool in ipairs(guns) do
		seen[tool] = true
		local o = droppedEsp[tool]
		if not o then
			o = makeDropEsp(tool)
			droppedEsp[tool] = o
		end
		local handle = tool:FindFirstChild("Handle")
		if handle and handle:IsA("BasePart") then
			local sp, on = Camera:WorldToViewportPoint(handle.Position + Vector3.new(0, 2, 0))
			o.Name.Visible = on
			if on then
				o.Name.Position = UDim2.fromOffset(sp.X, sp.Y)
			end
		end
	end
	for tool, o in pairs(droppedEsp) do
		if not seen[tool] then
			pcall(function() o.HL:Destroy() end)
			pcall(function() o.Name:Destroy() end)
			droppedEsp[tool] = nil
		end
	end
end

local function px(v) return math.floor(v + 0.5) end

local function drawEsp(o, t, dist, now, vp)
	local model = t.Model
	local color = t.Teammate and ALLY_COLOR
		or (Settings.Rainbow and Color3.fromHSV((now * 0.25) % 1, 0.85, 1) or Settings.ESPColor)
	local mm2Role = nil

	-- MM2 Mode: sobrescreve a cor pela função
	if Settings.MM2Mode then
		mm2Role = MM2.getRole(model, t.Player)
		if mm2Role == "innocent" then color = Settings.MM2InnocentColor
		elseif mm2Role == "murder" then color = Settings.MM2MurderColor
		elseif mm2Role == "sheriff" then color = Settings.MM2SheriffColor
		end
	end

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
	local boxH = math.max(bot.Y - top.Y, 6)
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
		o.HpBg.Position = UDim2.fromOffset(x - 6, y)
		o.HpBg.Size = UDim2.fromOffset(3, boxH)
		o.HpFill.Size = UDim2.new(1, 0, frac, 0)
		o.HpFill.BackgroundColor3 = Color3.fromHSV(frac * 0.33, 0.9, 1)
	else
		o.HpBg.Visible = false
	end

	if Settings.ShowNames then
		local plr = t.Player
		local roleTag = ""
		if Settings.MM2Mode and mm2Role then
			if mm2Role == "innocent" then roleTag = " [INOCENTE]"
			elseif mm2Role == "murder" then roleTag = " [MURDER]"
			elseif mm2Role == "sheriff" then roleTag = " [SHERIFF]" end
		end
		o.Name.Visible = true
		o.Name.Text = (plr and plr.DisplayName or model.Name) .. roleTag
			.. "  [" .. math.floor(dist) .. "m]"
		o.Name.TextColor3 = color
		o.Name.Position = UDim2.fromOffset(px(center.X), y - 2)
	else
		o.Name.Visible = false
	end

	if Settings.ESPTool then
		if now - o.ToolAt > 0.5 then
			o.ToolAt = now
			local tool = model:FindFirstChildOfClass("Tool")
			o.Tool = tool and tool.Name or ""
		end
		o.ToolLbl.Visible = o.Tool ~= ""
		o.ToolLbl.Text = o.Tool
		o.ToolLbl.Position = UDim2.fromOffset(px(center.X), y + boxH + 2)
	else
		o.ToolLbl.Visible = false
	end

	if Settings.ESPHeadDot then
		local head = model:FindFirstChild("Head")
		if head and head:IsA("BasePart") then
			local hp, on = Camera:WorldToViewportPoint(head.Position)
			o.Dot.Visible = on
			o.Dot.BackgroundColor3 = color
			o.Dot.Position = UDim2.fromOffset(px(hp.X), px(hp.Y))
		else
			o.Dot.Visible = false
		end
	else
		o.Dot.Visible = false
	end

	if Settings.Tracers then
		local p1
		if Settings.TracerOrigin == 1 then p1 = Vector2.new(vp.X / 2, vp.Y)
		elseif Settings.TracerOrigin == 2 then p1 = vp / 2
		else p1 = getAimOrigin() end
		local p2 = Vector2.new(center.X, center.Y)
		local d = p2 - p1
		local len = d.Magnitude
		if len > 2 then
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
	-- Arma dropada
	updateDroppedGuns(now)
end

--------------------------------------------------------------------
-- HUD
--------------------------------------------------------------------
local hud = Instance.new("TextLabel")
hud.Name = "HUD"
hud.Position = UDim2.fromOffset(10, 10)
hud.Size = UDim2.fromOffset(600, 20)
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
hud.Active = true

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

local fpsFrames, fpsAcc, hudAcc, btnAcc = 0, 0, 0, 0

RunService:BindToRenderStep("TestToolkitVisuals", Enum.RenderPriority.Camera.Value + 2, function(dt)
	local now = os.clock()

	fpsFrames += 1
	fpsAcc += dt
	if fpsAcc >= 0.5 then
		lastFps = fpsFrames / fpsAcc
		fpsFrames, fpsAcc = 0, 0
	end

	local showFov = Settings.FOVEnabled and Settings.AimEnabled and not spectating and isLegit()
	fovCircle.Visible = showFov
	if showFov then
		local o = getAimOrigin()
		fovCircle.Position = UDim2.fromOffset(o.X, o.Y)
		local d = Settings.FOVRadius * 2
		fovCircle.Size = UDim2.fromOffset(d, d)
	end

	local showSilentFov = Settings.AimEnabled and not spectating and isSilent()
	silentFovCircle.Visible = showSilentFov
	if showSilentFov then
		local o = getAimOrigin()
		silentFovCircle.Position = UDim2.fromOffset(o.X, o.Y)
		local d = Settings.SilentAimFOV * 2
		silentFovCircle.Size = UDim2.fromOffset(d, d)
	end

	local showTriggerFov = Settings.TriggerBot and not spectating
	triggerFovCircle.Visible = showTriggerFov
	if showTriggerFov then
		local o = getAimOrigin()
		triggerFovCircle.Position = UDim2.fromOffset(o.X, o.Y)
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

	updateEsp(now)

	btnAcc += dt
	if btnAcc >= 0.1 then
		btnAcc = 0
		updateMobileBtns()
	end

	hud.Visible = Settings.ShowHUD or Settings.HUDEdit
	hud.Position = UDim2.fromOffset(Settings.HUDX, Settings.HUDY)
	hud.BackgroundColor3 = Settings.HUDEdit and Theme.Accent or Theme.Bg
	hudAcc += dt
	if (Settings.ShowHUD or Settings.HUDEdit) and hudAcc >= 0.25 then
		hudAcc = 0
		local okp, pv = pcall(LocalPlayer.GetNetworkPing, LocalPlayer)
		local ping = math.floor((okp and pv or pingVal) * 1000)
		local parts = { math.floor(lastFps) .. " FPS", ping .. " ms" }
		if Settings.AimEnabled then
			local aimType
			if Settings.UseLegitAim and Settings.UseSilentAim then aimType = "Legit+Silent"
			elseif Settings.UseLegitAim then aimType = "Legit"
			elseif Settings.UseSilentAim then aimType = "Silent"
			else aimType = "OFF" end
			parts[#parts + 1] = "Mira: " .. aimType
		end
		if Settings.MM2Mode then parts[#parts + 1] = "MM2" end
		if Settings.Fly then parts[#parts + 1] = "Voo" end
		if Settings.Noclip then parts[#parts + 1] = "Noclip" end
		if Settings.Fling then parts[#parts + 1] = "Fling" end
		if Settings.HitboxExpander then parts[#parts + 1] = "Hitbox" end
		if Settings.SeatInvisible then parts[#parts + 1] = "SeatInvis" end
		if Settings.TriggerBot and triggerBotActive then parts[#parts + 1] = "Trigger" end
		if Settings.AntiKill then parts[#parts + 1] = "AntiKill" end
		if Settings.AntiVoid then parts[#parts + 1] = "AntiVoid" end
		hud.Text = table.concat(parts, "  •  ")
	end
end)

end -- fim buildUI

--------------------------------------------------------------------
-- INICIAR
--------------------------------------------------------------------
local okUI, errUI = pcall(buildUI)
if okUI then
	bootShow("TestToolkit v59.5 carregado  •  "
		.. (isMobile and "botão TT abre o menu" or "Ctrl direito abre o menu"),
		Color3.fromRGB(80, 255, 130), 5)
else
	warn("[TestToolkit] erro na interface: " .. tostring(errUI))
	bootShow("TestToolkit: erro na interface: " .. tostring(errUI), Color3.fromRGB(255, 90, 90))
end
