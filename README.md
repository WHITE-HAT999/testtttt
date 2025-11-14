Potential Future Setting Options
 - Block entire domain or just the specific page in the Sirius Intelligent Flow Interception. Do this on case by case, e.g blocked = {"link.com", true} - true being whether its the domain or not
 - Serverhop type (default/gta)
 - Hook Specific Functions to reduce the need for external scripts

--]]

-- Ensure the game is loaded 
if not game:IsLoaded() then
	game.Loaded:Wait()
end

-- Check License Tier
local Pro = true -- We're open sourced now!

-- Create Variables for Roblox Services
local coreGui = game:GetService("CoreGui")
local httpService = game:GetService("HttpService")
local lighting = game:GetService("Lighting")
local players = game:GetService("Players")
local replicatedStorage = game:GetService("ReplicatedStorage")
local runService = game:GetService("RunService")
local guiService = game:GetService("GuiService")
local statsService = game:GetService("Stats")
local starterGui = game:GetService("StarterGui")
local teleportService = game:GetService("TeleportService")
local tweenService = game:GetService("TweenService")
local userInputService = game:GetService('UserInputService')
local gameSettings = UserSettings():GetService("UserGameSettings")

-- Variables
local camera = workspace.CurrentCamera
local getMessage = replicatedStorage:WaitForChild("DefaultChatSystemChatEvents", 1) and replicatedStorage.DefaultChatSystemChatEvents:WaitForChild("OnMessageDoneFiltering", 1)
local localPlayer = players.LocalPlayer
local notifications = {}
local friendsCooldown = 0
local mouse = localPlayer:GetMouse()
local promptedDisconnected = false
local smartBarOpen = false
local debounce = false
local searchingForPlayer = false
local musicQueue = {}
local currentAudio
local lowerName = localPlayer.Name:lower()
local lowerDisplayName = localPlayer.DisplayName:lower()
local placeId = game.PlaceId
local jobId = game.JobId
local checkingForKey = false
local originalTextValues = {}
local creatorId = game.CreatorId
local noclipDefaults = {}
local movers = {}
local creatorType = game.CreatorType
local espContainer = Instance.new("Folder", gethui and gethui() or coreGui)
local oldVolume = gameSettings.MasterVolume

-- Configurable Core Values
local siriusValues = {
	siriusVersion = "1.26",
	siriusName = "Sirius",
	releaseType = "Stable",
	siriusFolder = "Sirius",
	settingsFile = "settings.srs",
	interfaceAsset = 14183548964,
	cdn = "https://cdn.sirius.menu/SIRIUS-SCRIPT-CORE-ASSETS/",
	icons = "https://cdn.sirius.menu/SIRIUS-SCRIPT-CORE-ASSETS/Icons/",
	enableExperienceSync = false, -- Games are no longer available due to a lack of whitelisting, they may be made open source at a later date, however they are patched as of now and are useless to the end user. Turning this on may introduce "fake functionality".
	games = {
		BreakingPoint = {
			name = "Breaking Point",
			description = "Players are seated around a table. Their only goal? To be the last one standing. Execute this script to gain an unfair advantage.",
			id = 648362523,
			enabled = true,
			raw = "BreakingPoint",
			minimumTier = "Free",
		},
		MurderMystery2 = {
			name = "Murder Mystery 2",
			description = "A murder has occured, will you be the one to find the murderer, or kill your next victim? Execute this script to gain an unfair advantage.",
			id = 142823291,
			enabled = true,
			raw = "MurderMystery2",
			minimumTier = "Free",
		},
		TowerOfHell = {
			name = "Tower Of Hell",
			description = "A difficult popular parkouring game, with random levels and modifiers. Execute this script to gain an unfair advantage.",
			id = 1962086868,
			enabled = true,
			raw = "TowerOfHell",
			minimumTier = "Free",
		},
		Strucid = {
			name = "Strucid",
			description = "Fight friends and enemies in Strucid with building mechanics! Execute this script to gain an unfair advantage.",
			id = 2377868063,
			enabled = true,
			raw = "Strucid",
			minimumTier = "Free",
		},
		PhantomForces = {
			name = "Phantom Forces",
			description = "One of the most popular FPS shooters from the team at StyLiS Studios. Execute this script to gain an unfair advantage.",
			id = 292439477,
			enabled = true,
			raw = "PhantomForces",
			minimumTier = "Pro",
		},
	},
	rawTree = "https://raw.githubusercontent.com/SiriusSoftwareLtd/Sirius/Sirius/games/",
	neonModule = "https://raw.githubusercontent.com/shlexware/Sirius/request/library/neon.lua",
	senseRaw = "https://raw.githubusercontent.com/shlexware/Sirius/request/library/sense/source.lua",
	executors = {"synapse x", "script-ware", "krnl", "scriptware", "comet", "valyse", "fluxus", "electron", "hydrogen"},
	disconnectTypes = { {"ban", {"ban", "perm"}}, {"network", {"internet connection", "network"}} },
	nameGeneration = {
		adjectives = {"Cool", "Awesome", "Epic", "Ninja", "Super", "Mystic", "Swift", "Golden", "Diamond", "Silver", "Mint", "Roblox", "Amazing"},
		nouns = {"Player", "Gamer", "Master", "Legend", "Hero", "Ninja", "Wizard", "Champion", "Warrior", "Sorcerer"}
	},
	administratorRoles = {"mod","admin","staff","dev","founder","owner","supervis","manager","management","executive","president","chairman","chairwoman","chairperson","director"},
	transparencyProperties = {
		UIStroke = {'Transparency'},
		Frame = {'BackgroundTransparency'},
		TextButton = {'BackgroundTransparency', 'TextTransparency'},
		TextLabel = {'BackgroundTransparency', 'TextTransparency'},
		TextBox = {'BackgroundTransparency', 'TextTransparency'},
		ImageLabel = {'BackgroundTransparency', 'ImageTransparency'},
		ImageButton = {'BackgroundTransparency', 'ImageTransparency'},
		ScrollingFrame = {'BackgroundTransparency', 'ScrollBarImageTransparency'}
	},
	buttonPositions = {Character = UDim2.new(0.5, -155, 1, -29), Scripts = UDim2.new(0.5, -122, 1, -29), Playerlist = UDim2.new(0.5, -68, 1, -29)},
	chatSpy = {
		enabled = true,
		visual = {
			Color = Color3.fromRGB(229, 107, 0), -- REVERSED: Was blue (26, 148, 255), now orange
			Font = Enum.Font.SourceSansBold,
			TextSize = 18
		},
	},
	pingProfile = {
		recentPings = {},
		adaptiveBaselinePings = {},
		pingNotificationCooldown = 0,
		maxSamples = 12, -- max num of recent pings stored
		spikeThreshold = 1.75, -- high Ping in comparison to average ping (e.g 100 avg would be high at 150)
		adaptiveBaselineSamples = 30, -- how many samples Sirius takes before deciding on a fixed high ping value
		adaptiveHighPingThreshold = 120 -- default value
	},
	frameProfile = {
		frameNotificationCooldown = 0,
		fpsQueueSize = 10,
		lowFPSThreshold = 20, -- what's low fps!??!?!
		totalFPS = 0,
		fpsQueue = {},
	},
	actions = {
		{
			name = "Noclip",
			images = {14385986465, 9134787693},
			color = Color3.fromRGB(255, 85, 128), -- REVERSED: Was green (0, 170, 127), now pink/red
			enabled = false,
			rotateWhileEnabled = false,
			callback = function() end,
		},
		{
			name = "Flight",
			images = {9134755504, 14385992605},
			color = Color3.fromRGB(85, 218, 209), -- REVERSED: Was red (170, 37, 46), now cyan
			enabled = false,
			rotateWhileEnabled = false,
			callback = function(value)
				local character = localPlayer.Character
				local humanoid = character and character:FindFirstChildOfClass("Humanoid")
				if humanoid then
					humanoid.PlatformStand = value
				end
			end,
		},
		{
			name = "Refresh",
			images = {9134761478, 9134761478},
			color = Color3.fromRGB(194, 76, 157), -- REVERSED: Was green (61, 179, 98), now magenta
			enabled = false,
			rotateWhileEnabled = true,
			disableAfter = 3,
			callback = function()
				task.spawn(function()
					local character = localPlayer.Character
					if character then
						local cframe = character:GetPivot()
						local humanoid = character:FindFirstChildOfClass("Humanoid")
						if humanoid then
							humanoid:ChangeState(Enum.HumanoidStateType.Dead)
						end
						character = localPlayer.CharacterAdded:Wait()
						task.defer(character.PivotTo, character, cframe)
					end
				end)
			end,
		},
		{
			name = "Respawn",
			images = {9134762943, 9134762943},
			color = Color3.fromRGB(206, 167, 62), -- REVERSED: Was blue (49, 88, 193), now yellow
			enabled = false,
			rotateWhileEnabled = true,
			disableAfter = 2,
			callback = function()
				local character = localPlayer.Character
				local humanoid = character and character:FindFirstChildOfClass("Humanoid")
				if humanoid then
					humanoid:ChangeState(Enum.HumanoidStateType.Dead)
				end
			end,
		},
		{
			name = "Server Info",
			images = {14386050540, 14386050540},
			color = Color3.fromRGB(162, 97, 218), -- REVERSED: Was purple (93, 158, 37), now purple (inverted)
			enabled = false,
			rotateWhileEnabled = false,
			callback = function(value)
				gui["ServerInfo"].Visible = value
			end,
		},
		{
			name = "Anti Ragdoll",
			images = {9134768678, 14385999093},
			color = Color3.fromRGB(229, 55, 98), -- REVERSED: Was teal (26, 200, 157), now salmon
			enabled = false,
			rotateWhileEnabled = false,
			callback = function() end,
		},
		{
			name = "Xray",
			images = {9134772307, 14386001814},
			color = Color3.fromRGB(229, 147, 30), -- REVERSED: Was indigo (26, 108, 225), now orange
			enabled = false,
			rotateWhileEnabled = false,
			callback = function(value)
				xray(value)
			end,
		},
		{
			name = "Super Bright",
			images = {9134773936, 14386003681},
			color = Color3.fromRGB(0, 147, 251), -- REVERSED: Was yellow (255, 108, 4), now blue
			enabled = false,
			rotateWhileEnabled = false,
			callback = function(value)
				if value then
					saveGamingEnvironment()
					resetEnvironmentBrightness()
					
					lighting.ClockTime = 12
					lighting.FogEnd = 786543
					lighting.GlobalShadows = false
					lighting.Brightness = 10
					
					for _, instance in next, workspace:GetDescendants() do
						if instance:IsA("BasePart") and not instance:IsA("Terrain") then
							instance.Material = Enum.Material.SmoothPlastic
						end
					end
				else
					resetGamingEnvironment()
				end
			end,
		},
		{
			name = "Dex",
			images = {14386012166, 14386012166},
			color = Color3.fromRGB(100, 172, 99), -- REVERSED: Was green (155, 83, 156), now green
			enabled = false,
			rotateWhileEnabled = true,
			disableAfter = 3,
			callback = function()
				loadstring(game:HttpGetAsync(siriusValues.neonModule))()
			end,
		},
		{
			name = "Low Graphics",
			images = {14386004952, 14386004952},
			color = Color3.fromRGB(161, 226, 113), -- REVERSED: Was yellow-green (94, 29, 142), now lime
			enabled = false,
			rotateWhileEnabled = false,
			callback = function(value)
				if value then
					saveGamingEnvironment()
					resetEnvironmentBrightness()
					
					gameSettings.SavedQualityLevel = Enum.SavedQualitySetting.QualityLevel1
					
					for _, object in next, workspace:GetDescendants() do
						if object:IsA("BasePart") then
							object.Material = Enum.Material.SmoothPlastic
						elseif object:IsA("Decal") or object:IsA("Texture") then
							object.Transparency = 1
						elseif object:IsA("ParticleEmitter") or object:IsA("Trail") then
							object.Enabled = false
						elseif object:IsA("Explosion") then
							object.BlastPressure = 1
							object.BlastRadius = 1
						elseif object:IsA("Fire") or object:IsA("SpotLight") or object:IsA("Smoke") or object:IsA("Sparkles") then
							object.Enabled = false
						elseif object:IsA("MeshPart") then
							object.Material = Enum.Material.SmoothPlastic
							object.TextureID = 10385902758728957
						elseif object:IsA("SpecialMesh") then
							object.TextureId = 0
						elseif object:IsA("ShirtGraphic") then
							object.ShirtGraphic = ""
						elseif (object:IsA("Shirt") or object:IsA("Pants")) and object.Name ~= "Shirt" and object.Name ~= "Pants" then
							object:Destroy()
						elseif object:IsA("Explosion") then
							object.BlastPressure = 0.01
							object.BlastRadius = 0.01
							object.DestroyJointRadiusPercent = 0
						end
					end
				else
					resetGamingEnvironment()
					gameSettings.SavedQualityLevel = Enum.SavedQualitySetting.Automatic
				end
			end,
		},
		{
			name = "Anonymous Client",
			images = {14386005901, 14386005901},
			color = Color3.fromRGB(135, 126, 84), -- REVERSED: Was gray (120, 129, 171), now tan
			enabled = false,
			rotateWhileEnabled = false,
			callback = function() end,
		},
		{
			name = "Spy",
			images = {14386007491, 14386007491},
			color = Color3.fromRGB(200, 56, 255), -- REVERSED: Was purple (55, 199, 0), now magenta
			enabled = false,
			rotateWhileEnabled = false,
			callback = function(value)
				siriusValues.chatSpy.enabled = value
			end,
		},
		{
			name = "Spatial Shield",
			images = {14386009116, 14386009116},
			color = Color3.fromRGB(231, 106, 119), -- REVERSED: Was teal (24, 149, 136), now coral
			enabled = false,
			rotateWhileEnabled = false,
			callback = function() end,
		},
		{
			name = "Smart ESP",
			images = {14386010615, 14386010615},
			color = Color3.fromRGB(96, 226, 102), -- REVERSED: Was red (159, 29, 153), now green
			enabled = false,
			rotateWhileEnabled = false,
			callback = function(value)
				gui["ESP List"].Visible = value and checkSetting("Smart ESP: Enable ESP List").current
				espEnabled = value
			end,
		},
	},
}

local soundSuppressionNotificationCooldown = 0
local suppressedSounds = {}
local randomUsername = siriusValues.nameGeneration.adjectives[math.random(#siriusValues.nameGeneration.adjectives)] .. siriusValues.nameGeneration.nouns[math.random(#siriusValues.nameGeneration.nouns)] .. math.random(1, 1000)
local cachedText = {}
local soundInstances = {}
local cachedIds = {}
local espEnabled = false
local autoReport = false
local requestErrors = {}
local cachedEnvironmentValues = {}
local xrayInstances = {}

-- Functions

local function checkSirius()
	if not coreGui:FindFirstChild(siriusValues.siriusName) then
		return false
	end
	return true
end

local function checkSetting(settingName)
	local file = siriusValues.siriusFolder .."/".. siriusValues.settingsFile
	
	if isfile then -- pro
		if isfile(file) then
			local existingSettings = httpService:JSONDecode(readfile(file))
			for i, v in next, existingSettings do
				if i == settingName then
					return v
				end
			end
		end
	end
	
	return {current = false}
end

local function storeOriginalText(textObject)
	if not originalTextValues[textObject] then
		originalTextValues[textObject] = textObject.Text
	end
end

local function undoAnonymousChanges()
	for textObject, originalText in pairs(originalTextValues) do
		if textObject and textObject.Parent then
			textObject.Text = originalText
		end
	end
	originalTextValues = {}
end

local function saveGamingEnvironment()
	cachedEnvironmentValues = {
		Ambient = lighting.Ambient,
		Brightness = lighting.Brightness,
		ColorShift_Bottom = lighting.ColorShift_Bottom,
		ColorShift_Top = lighting.ColorShift_Top,
		FogColor = lighting.FogColor,
		FogEnd = lighting.FogEnd,
		FogStart = lighting.FogStart,
		OutdoorAmbient = lighting.OutdoorAmbient,
		ShadowSoftness = lighting.ShadowSoftness,
		ClockTime = lighting.ClockTime,
		GeographicLatitude = lighting.GeographicLatitude,
		GlobalShadows = lighting.GlobalShadows,
		TimeOfDay = lighting.TimeOfDay,
		ExposureCompensation = lighting.ExposureCompensation
	}
end

local function resetGamingEnvironment()
	for property, value in pairs(cachedEnvironmentValues) do
		if property and value then
			lighting[property] = value
		end
	end
end

local function resetEnvironmentBrightness()
	lighting.Ambient = Color3.fromRGB(166, 166, 166) -- REVERSED: Was 89, 89, 89 now lighter
	lighting.Brightness = 3
	lighting.FogColor = Color3.fromRGB(163, 163, 163) -- REVERSED: Was 92, 92, 92 now lighter
	lighting.FogEnd = 100000
	lighting.FogStart = 0
	lighting.ExposureCompensation = 0
	lighting.ShadowSoftness = 0
	lighting.GlobalShadows = true
	lighting.GeographicLatitude = 45
end

local function serverhop()
	local server
	local serverList = {}
	local cursor
	
	repeat
		local response = httpService:JSONDecode(game:HttpGetAsync("https://games.roblox.com/v1/games/"..placeId.."/servers/0?sortOrder=2&excludeFullGames=true" .. (cursor and "&cursor="..cursor or "")))
		for _, value in next, response.data do 
			if value.playing ~= value.maxPlayers and value.id ~= jobId then
				table.insert(serverList, value.id)
			end
		end
		cursor = response.nextPageCursor
	until not cursor
	
	if #serverList > 0 then
		server = serverList[math.random(1, #serverList)]
		teleportService:TeleportToPlaceInstance(placeId, server, localPlayer)
		queueNotification("Serverhopping", "You're being sent to a new server.")
	else
		queueNotification("Serverhop Failed", "Could not find a server.")
	end
end

local function rejoin()
	if #players:GetPlayers() <= 1 then
		localPlayer:Kick("\nRejoining...")
		task.wait()
		teleportService:Teleport(placeId, localPlayer)
	else
		teleportService:TeleportToPlaceInstance(placeId, jobId, localPlayer)
	end
end

local function xray(enabled)
	if enabled then
		for _, object in next, workspace:GetDescendants() do
			if object:IsA("BasePart") and not object:IsA("Terrain") then
				if object.Transparency ~= 1 then
					table.insert(xrayInstances, {part = object, transparency = object.Transparency})
					object.Transparency = object.Transparency > 0.5 and object.Transparency or 0.5
				end
			end
		end
	else
		for _, data in next, xrayInstances do
			if data.part and data.part.Parent then
				data.part.Transparency = data.transparency
			end
		end
		xrayInstances = {}
	end
end

local function checkHighPing()
	local recentPing = tonumber(string.split(statsService.Network.ServerStatsItem["Data Ping"]:GetValueString(), " ")[1])
	
	table.insert(siriusValues.pingProfile.recentPings, recentPing)
	
	if #siriusValues.pingProfile.recentPings > siriusValues.pingProfile.maxSamples then
		table.remove(siriusValues.pingProfile.recentPings, 1)
	end
	
	local averagePing = 0
	for _, ping in ipairs(siriusValues.pingProfile.recentPings) do
		averagePing = averagePing + ping
	end
	averagePing = averagePing / #siriusValues.pingProfile.recentPings
	
	if #siriusValues.pingProfile.adaptiveBaselinePings < siriusValues.pingProfile.adaptiveBaselineSamples then
		table.insert(siriusValues.pingProfile.adaptiveBaselinePings, recentPing)
	else
		local adaptiveAverage = 0
		for _, baselinePing in ipairs(siriusValues.pingProfile.adaptiveBaselinePings) do
			adaptiveAverage = adaptiveAverage + baselinePing
		end
		siriusValues.pingProfile.adaptiveHighPingThreshold = (adaptiveAverage / #siriusValues.pingProfile.adaptiveBaselinePings) * siriusValues.pingProfile.spikeThreshold
	end
	
	return recentPing > siriusValues.pingProfile.adaptiveHighPingThreshold
end

local function wipeTransparency(instance, transparency, includeDescendants)
	for property, _ in pairs(siriusValues.transparencyProperties) do
		if instance:IsA(property) then
			for _, prop in ipairs(siriusValues.transparencyProperties[property]) do
				instance[prop] = transparency
			end
		end
	end
	
	if includeDescendants then
		for _, descendant in next, instance:GetDescendants() do
			wipeTransparency(descendant, transparency, false)
		end
	end
end

local function queueNotification(title, content, icon)
	-- Implementation would go here
	print("Notification:", title, content)
end

local function UpdateHome()
	-- Implementation would go here
end

-- Create GUI
local gui = {}
gui["ESP List"] = Instance.new("Frame")
gui["ESP List"].Visible = false
gui["ESP List"].Parent = coreGui

gui["ServerInfo"] = Instance.new("Frame")
gui["ServerInfo"].Visible = false
gui["ServerInfo"].Parent = coreGui

-- Create SmartBar
local smartBar = Instance.new("Frame")
smartBar.Time = Instance.new("TextLabel")
smartBar.Time.Parent = smartBar
smartBar.Parent = coreGui

-- Create Toggle
local toggle = Instance.new("TextButton")
toggle.Parent = coreGui

-- Create Disconnected Prompt
local disconnectedPrompt = Instance.new("Frame")
disconnectedPrompt.Visible = false
disconnectedPrompt.BackgroundTransparency = 1
disconnectedPrompt.Content = Instance.new("TextLabel")
disconnectedPrompt.Content.Parent = disconnectedPrompt
disconnectedPrompt.Title = Instance.new("TextLabel")
disconnectedPrompt.Title.Parent = disconnectedPrompt
disconnectedPrompt.Action = Instance.new("TextButton")
disconnectedPrompt.Action.Parent = disconnectedPrompt
disconnectedPrompt.UIGradient = Instance.new("UIGradient")
disconnectedPrompt.UIGradient.Parent = disconnectedPrompt

-- Main Loop for frame calculations
local currentFPS = 0
runService.Heartbeat:Connect(function(deltaTime)
	if Pro and checkSirius() then
		currentFPS = 1 / deltaTime
		
		table.insert(siriusValues.frameProfile.fpsQueue, currentFPS)
		siriusValues.frameProfile.totalFPS = siriusValues.frameProfile.totalFPS + currentFPS
		
		if #siriusValues.frameProfile.fpsQueue > siriusValues.frameProfile.fpsQueueSize then
			local removedFPS = table.remove(siriusValues.frameProfile.fpsQueue, 1)
			siriusValues.frameProfile.totalFPS = siriusValues.frameProfile.totalFPS - removedFPS
		end
	end
	
	-- Noclip logic
	for _, action in ipairs(siriusValues.actions) do
		if action.name == "Noclip" and action.enabled then
			local character = localPlayer.Character
			if character then
				for _, part in next, character:GetDescendants() do
					if part:IsA("BasePart") and part.CanCollide == true then
						part.CanCollide = false
						noclipDefaults[part] = true
					end
				end
			end
		end
	end
	
	-- Flight logic
	for _, action in ipairs(siriusValues.actions) do
		if action.name == "Flight" and action.enabled then
			local character = localPlayer.Character
			local humanoidRootPart = character and character:FindFirstChild("HumanoidRootPart")
			
			if humanoidRootPart then
				if not movers.bodyPosition then
					movers.bodyPosition = Instance.new("BodyPosition", humanoidRootPart)
					movers.bodyPosition.MaxForce = Vector3.new(0, math.huge, 0)
					
					movers.bodyGyro = Instance.new("BodyGyro", humanoidRootPart)
					movers.bodyGyro.D = 0
					movers.bodyGyro.MaxTorque = Vector3.new(0, math.huge, 0)
					movers.bodyGyro.P = 10000
				end
				
				movers.bodyPosition.Position = humanoidRootPart.Position
				movers.bodyGyro.CFrame = humanoidRootPart.CFrame
				
				-- Simplified flight controls
				if userInputService:IsKeyDown(Enum.KeyCode.W) then
					humanoidRootPart.CFrame = humanoidRootPart.CFrame * CFrame.new(0, 0, -1)
				end
				if userInputService:IsKeyDown(Enum.KeyCode.S) then
					humanoidRootPart.CFrame = humanoidRootPart.CFrame * CFrame.new(0, 0, 1)
				end
				if userInputService:IsKeyDown(Enum.KeyCode.A) then
					humanoidRootPart.CFrame = humanoidRootPart.CFrame * CFrame.new(-1, 0, 0)
				end
				if userInputService:IsKeyDown(Enum.KeyCode.D) then
					humanoidRootPart.CFrame = humanoidRootPart.CFrame * CFrame.new(1, 0, 0)
				end
				if userInputService:IsKeyDown(Enum.KeyCode.Space) then
					humanoidRootPart.CFrame = humanoidRootPart.CFrame * CFrame.new(0, 1, 0)
				end
				if userInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
					humanoidRootPart.CFrame = humanoidRootPart.CFrame * CFrame.new(0, -1, 0)
				end
			end
		else
			if movers.bodyPosition then
				movers.bodyPosition:Destroy()
				movers.bodyPosition = nil
			end
			if movers.bodyGyro then
				movers.bodyGyro:Destroy()
				movers.bodyGyro = nil
			end
		end
	end
	
	-- Anti-Ragdoll logic
	for _, action in ipairs(siriusValues.actions) do
		if action.name == "Anti Ragdoll" and action.enabled then
			local character = localPlayer.Character
			local humanoid = character and character:FindFirstChildOfClass("Humanoid")
			
			if humanoid then
				if humanoid:GetState() == Enum.HumanoidStateType.FallingDown then
					humanoid:ChangeState(Enum.HumanoidStateType.Running)
				end
				humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
				humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
			end
		end
	end
	
	if soundSuppressionNotificationCooldown > 0 then
		soundSuppressionNotificationCooldown = soundSuppressionNotificationCooldown - 1
	end
	
	for _, action in ipairs(siriusValues.actions) do
		if action.name == "Spatial Shield" and action.enabled then
			for index, sound in ipairs(soundInstances) do
				if sound.Playing and sound.Volume > 0.6 then
					if sound.Volume >= 1 then
						suppressedSounds[sound.SoundId] = "S"
						sound.Volume = 0.5 	
					elseif sound.Volume > 0.2 and sound.Volume < 0.55 then
						suppressedSounds[sound.SoundId] = "S2"
						sound.Volume = 0.1
					elseif sound.Volume < 0.2 then
						suppressedSounds[sound.SoundId] = "Mute"
						sound.Volume = 0
					end
					if soundSuppressionNotificationCooldown == 0 then
						queueNotification("Spatial Shield","A high-volume audio is being played ("..sound.Name..") and it has been suppressed.", 4483362458) 
						soundSuppressionNotificationCooldown = 15
					end
					table.remove(soundInstances, index)
				end
			end
		end
	end

	if checkSetting("Anonymous Client").current then
		for _, text in ipairs(cachedText) do
			local lowerText = string.lower(text.Text)
			if string.find(lowerText, lowerName, 1, true) or string.find(lowerText, lowerDisplayName, 1, true) then

				storeOriginalText(text)

				local newText = string.gsub(string.gsub(lowerText, lowerName, randomUsername), lowerDisplayName, randomUsername)
				text.Text = string.gsub(newText, "^%l", string.upper)
			end
		end
	else
		undoAnonymousChanges()
	end
end)

for _, instance in next, game:GetDescendants() do
	if instance:IsA("Sound") then
		if suppressedSounds[instance.SoundId] then
			if suppressedSounds[instance.SoundId] == "S" then
				instance.Volume = 0.5
			elseif suppressedSounds[instance.SoundId] == "S2" then
				instance.Volume = 0.1
			else
				instance.Volume = 0
			end
		else
			if not table.find(cachedIds, instance.SoundId) then
				table.insert(soundInstances, instance)
				table.insert(cachedIds, instance.SoundId)
			end
		end
	elseif instance:IsA("TextLabel") or instance:IsA("TextButton") then
		if not table.find(cachedText, instance) then
			table.insert(cachedText, instance)
		end
	end
end

game.DescendantAdded:Connect(function(instance)
	if checkSirius() then
		if instance:IsA("Sound") then
			if suppressedSounds[instance.SoundId] then
				if suppressedSounds[instance.SoundId] == "S" then
					instance.Volume = 0.5
				elseif suppressedSounds[instance.SoundId] == "S2" then
					instance.Volume = 0.1
				else
					instance.Volume = 0
				end
			else
				if not table.find(cachedIds, instance.SoundId) then
					table.insert(soundInstances, instance)
					table.insert(cachedIds, instance.SoundId)
				end
			end
		elseif instance:IsA("TextLabel") or instance:IsA("TextButton") then
			if not table.find(cachedText, instance) then
				table.insert(cachedText, instance)
			end
		end
	end
end)


while task.wait(1) do
	if not checkSirius() then
		if espContainer then espContainer:Destroy() end
		undoAnonymousChanges()
		break
	end

	smartBar.Time.Text = os.date("%H")..":"..os.date("%M")
	task.spawn(UpdateHome)

	if getconnections then
		for _, connection in getconnections(localPlayer.Idled) do
			if not checkSetting("Anti Idle").current then connection:Enable() else connection:Disable() end
		end
	end

	toggle.Visible = not checkSetting("Hide Toggle Button").current

	-- Disconnected Check
	local disconnectedRobloxUI = coreGui.RobloxPromptGui.promptOverlay:FindFirstChild("ErrorPrompt")

	if disconnectedRobloxUI and not promptedDisconnected then
		local reasonPrompt = disconnectedRobloxUI.MessageArea.ErrorFrame.ErrorMessage.Text

		promptedDisconnected = true
		disconnectedPrompt.Parent = coreGui.RobloxPromptGui

		local disconnectType
		local foundString

		for _, preDisconnectType in ipairs(siriusValues.disconnectTypes) do
			for _, typeString in pairs(preDisconnectType[2]) do
				if string.find(reasonPrompt, typeString) then
					disconnectType = preDisconnectType[1]
					foundString = true
					break
				end
			end
		end

		if not foundString then disconnectType = "kick" end

		wipeTransparency(disconnectedPrompt, 1, true)
		disconnectedPrompt.Visible = true

		if disconnectType == "ban" then
			disconnectedPrompt.Content.Text = "You've been banned, would you like to leave this server?"
			disconnectedPrompt.Action.Text = "Leave"
			disconnectedPrompt.Action.Size = UDim2.new(0, 77, 0, 36) -- use textbounds

			disconnectedPrompt.UIGradient.Color = ColorSequence.new({
				ColorSequenceKeypoint.new(0, Color3.new(1,1,1)), -- REVERSED: Was black, now white
				ColorSequenceKeypoint.new(1, Color3.new(0.180392, 0.835294, 0.835294)) -- REVERSED: Was red, now cyan
			})
		elseif disconnectType == "kick" then
			disconnectedPrompt.Content.Text = "You've been kicked, would you like to serverhop?"
			disconnectedPrompt.Action.Text = "Serverhop"
			disconnectedPrompt.Action.Size = UDim2.new(0, 114, 0, 36)

			disconnectedPrompt.UIGradient.Color = ColorSequence.new({
				ColorSequenceKeypoint.new(0, Color3.new(1,1,1)), -- REVERSED: Was black, now white
				ColorSequenceKeypoint.new(1, Color3.new(0.913725, 0.403922, 0.164706)) -- REVERSED: Was blue, now orange
			})
		elseif disconnectType == "network" then
			disconnectedPrompt.Content.Text = "You've lost connection, would you like to rejoin?"
			disconnectedPrompt.Action.Text = "Rejoin"
			disconnectedPrompt.Action.Size = UDim2.new(0, 82, 0, 36)

			disconnectedPrompt.UIGradient.Color = ColorSequence.new({
				ColorSequenceKeypoint.new(0, Color3.new(1,1,1)), -- REVERSED: Was black, now white
				ColorSequenceKeypoint.new(1, Color3.new(0.137255, 0.498039, 0.913725)) -- REVERSED: Was orange, now blue
			})
		end

		tweenService:Create(disconnectedPrompt, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {BackgroundTransparency = 0}):Play()
		tweenService:Create(disconnectedPrompt.Title, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {TextTransparency = 0}):Play()
		tweenService:Create(disconnectedPrompt.Content, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {TextTransparency = 0.3}):Play()
		tweenService:Create(disconnectedPrompt.Action, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {BackgroundTransparency = 0.7}):Play()
		tweenService:Create(disconnectedPrompt.Action, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {TextTransparency = 0}):Play()

		disconnectedPrompt.Action.MouseButton1Click:Connect(function()
			if disconnectType == "ban" then
				game:Shutdown() -- leave
			elseif disconnectType == "kick" then
				serverhop()
			elseif disconnectType == "network" then
				rejoin()
			end
		end)
	end

	if Pro then
		-- all Pro checks here!

		-- Two-Way Adaptive Latency Checks
		if checkHighPing() then
			if siriusValues.pingProfile.pingNotificationCooldown <= 0 then
				if checkSetting("Adaptive Latency Warning").current then
					queueNotification("High Latency Warning","We've noticed your latency has reached a higher value than usual, you may find that you are lagging or your actions are delayed in-game. Consider checking for any background downloads on your machine.", 4370305588)
					siriusValues.pingProfile.pingNotificationCooldown = 120
				end
			end
		end

		if siriusValues.pingProfile.pingNotificationCooldown > 0 then
			siriusValues.pingProfile.pingNotificationCooldown -= 1
		end

		-- Adaptive frame time checks
		if siriusValues.frameProfile.frameNotificationCooldown <= 0 then
			if #siriusValues.frameProfile.fpsQueue > 0 then
				local avgFPS = siriusValues.frameProfile.totalFPS / #siriusValues.frameProfile.fpsQueue

				if avgFPS < siriusValues.frameProfile.lowFPSThreshold then
					if checkSetting("Adaptive Performance Warning").current then
						queueNotification("Degraded Performance","We've noticed your client's frames per second have decreased. Consider checking for any background tasks or programs on your machine.", 4384400106)
						siriusValues.frameProfile.frameNotificationCooldown = 120	
					end
				end
			end
		end

		if siriusValues.frameProfile.frameNotificationCooldown > 0 then
			siriusValues.frameProfile.frameNotificationCooldown -= 1
		end
	end
end
