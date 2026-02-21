local util         = require(game.ServerScriptService.Scripts.CharacterUtility)
local functions    = require(script.Functions)
local directory    = require(game.ReplicatedStorage.Modules.Directory)

local hitboxMod    = util.Hitbox
local tweenservice = util.TweenService
local damager      = util.DamageModule
local combatmodule = util.CombatModule


local RayParams = RaycastParams.new()
RayParams.FilterType = Enum.RaycastFilterType.Include
RayParams.FilterDescendantsInstances = {workspace.Map}

local module = {}

module.FindMove = function(plr,args,movename)
	if not module[movename] then warn("move not found: " .. movename) end

	module[movename](plr,args)
end


function module.M1(plr, typer)
	local m1_info = util.M1Module.M1(plr,script.M1Anims, {
		m1cd      = 0.09,
		m1finalcd = 1.5,
		dashm1cd  = 6,

		m1speed = 1.7,
		m1slow  = 16,
		m1awakenedSpeed = 2,

		m1_pullpwr = 30,

		debouncename     = "M1",
		dashdebouncename = "DashM1",

		m1damage         = 3,
		debounce_per_hit = 0.5,
	})
	if m1_info == nil then return end

	local character = plr.Character
	m1_info.ComboUpdating.Event:Connect(function(val, no)		
		
		local sound = script.Sounds:FindFirstChild("M1_"..val)
		if sound then
			sound = sound:Clone()
			sound.Parent = plr.Character:FindFirstChild("Right Arm")
			util.CombatRemote.Effect:Fire(sound,"Sound")
			game.Debris:AddItem(sound,1)
		end
		if no then return end
		util.CombatRemote.Effect:Fire(character,"Aizen","M1Slash", val)
	end)
	m1_info:Start(typer)

end

function module.Kurohitsugi(plr, args)
	local movestat = {
		["CanUseInStun"] = false;
		["CanUseInBusy"] = false
	}

	local RS = game.ReplicatedStorage

	-- MOVE SETTINGS
	local total_dmg = 15
	local cd        = 20
	local duration  = 1
	local cancelled = false

	local other_dmg = {
		["AwakeningPoints"] = 10;
	}

	local debouncename = "Kurohitsugi"

	-- Check if player can use the move
	local plrdata = table.unpack(args)
	local status = util.StatusChecks.check(plr.Character, movestat)
	if status ~= true then return end

	local character = plr.Character
	local hmrp = character:FindFirstChild("HumanoidRootPart")
	local values = character:FindFirstChild("Values")
	local states = character:FindFirstChild("States")
	local humanoid = character:FindFirstChild("Humanoid")

	if util.ValueChecks:Check(character, movestat) ~= true then return end
	if states:FindFirstChild(debouncename) then return end

	local target = util.CombatModule:GetNearestTarget(character, 25)
	if not target then return end
	local targetHRP = target:FindFirstChild("HumanoidRootPart")
	if not targetHRP then return end
	
	values.Busy.Value = true
	
	local anim = script.Animations.KurohitsugiCast
	local cast = humanoid.Animator:LoadAnimation(anim)
	util.CombatModule.slow(character,0, 1.5)
	cast:Play()

	local boxTemplate = script.Instances.KurohitsugiBox 
	local box = boxTemplate:Clone()
	box.Size = Vector3.new(10, 15, 10)
	box.Anchored = true
	box.CanCollide = false
	box.Transparency = 0

	local boxHeight = box.Size.Y
	local boxPosition = targetHRP.Position + Vector3.new(0, boxHeight / 1.5, 0)
	box.CFrame = CFrame.new(boxPosition)
	
	box.CFrame = targetHRP.CFrame
	wait(0.5)


	local effectsFolder = workspace:FindFirstChild("Effects")

	box.Parent = effectsFolder

	local swords = {}


	local numSwords = 4 
	local radius = 15 


	local sideOffsets = {
		Vector3.new(radius, 0, 0),
		Vector3.new(-radius, 0, 0),
		Vector3.new(0, 0, radius),
		Vector3.new(0, 0, -radius)
	}

	for i = 1, numSwords do
		local sword = script.Instances.KurohitsugiSword:Clone()
		sword.Size = Vector3.new(1, 5, 0.2)
		sword.BrickColor = BrickColor.new("Really black")
		sword.Anchored = true
		sword.CanCollide = false

		local swordPos = box.Position + sideOffsets[i]

		sword.CFrame = CFrame.lookAt(swordPos, box.Position)

		sword.Parent = effectsFolder
		table.insert(swords, sword) 

		local targetCFrame = CFrame.new(box.Position) * CFrame.Angles(math.rad(90), 0, 0) 
		tweenservice:Create(sword, TweenInfo.new(0.5, Enum.EasingStyle.Linear), {CFrame = targetCFrame}):Play()
	end

	local hitbox = hitboxMod.new(
		plr,
		box,
		"KurohitsugiHitbox",
		0.75
	)
	box.Transparency = 0
	
	hitbox.Damaged.Event:Connect(function(targets)
		for _, target in pairs(targets) do
			local damageSuccess = damager.Damage(target, plr, total_dmg, true, other_dmg)
			if damageSuccess then

				if target.Humanoid.Health <= 15 then

					task.spawn(function()
						for i,v in pairs(target:GetDescendants()) do
							if v:IsA("BasePart") then v.Anchored = true end

							if v:IsA("Motor6D") or v:IsA("BallSocketConstraint") then v:Destroy() end
						end

						task.delay(1.3,function()

							local starKill = script.Instances.starKill:Clone()
							starKill.Parent = target.HumanoidRootPart
							starKill:Emit(30)

							for i,v in pairs(target:GetChildren()) do
								if v:IsA("BasePart") then 
									util.CombatRemote.Effect:Fire(v,"Blood",10)
								end
							end	

						end)

						task.wait(1.5)

						for i,v in pairs(target:GetDescendants()) do
							if v:IsA("BasePart") then 

								v.Anchored = false

							end
						end	


						for i,v in pairs(target:GetChildren()) do
							if v:IsA("BasePart") then 

								v:SetNetworkOwner(nil)

								local sound = script.Sounds.RushKill:Clone()
								sound.Parent = v
								util.CombatRemote.Effect:Fire(sound,"Sound")

								local cloneblo = script.Instances.DripBlood:Clone()
								cloneblo.Parent = v

								v:ApplyImpulse(Vector3.new(math.random(-10,10) * v.AssemblyMass,0,50 * v.AssemblyMass,math.random(-10,10) * v.AssemblyMass,0))
								v:ApplyAngularImpulse(Vector3.new(0,0,Random.new():NextNumber(20,30) * v.AssemblyMass ))

								util.CombatRemote.Effect:Fire(v,"Blood",10)
							end
						end	


					end)

				else
				util.CombatModule.stun(target, 2)
				
				local s = script.Sounds.Cut:Clone()
				s.Parent = target.HumanoidRootPart
				s:Play()
				game.Debris:AddItem(s, 2)
				end
			end
		end
	end)
	
	local teleportDist = 10
	local facingDir = hmrp.CFrame.LookVector.Unit
	local tpPos = box.Position + facingDir * teleportDist + Vector3.new(0, -1, 0)

	character:SetPrimaryPartCFrame(CFrame.new(tpPos, box.Position))
	
	cast:Stop()
	task.delay(duration, function()
		hitbox:Janitor()

		for _, sword in pairs(swords) do
			if sword then
				sword:Destroy()
			end
		end

		box:Destroy()
	end)

	util.CombatModule.Debounce(plr, states, debouncename, cd)
	values.Busy.Value = false
end

function module.Hado_88(plr, args)
	local movestat = {
		["CanUseInStun"] = false;
		["CanUseInBusy"] = false
	}

	local RS = game.ReplicatedStorage

	-- MOVE SETTINGS
	local damage = 15
	local cd     = 22
	local delayt = 1

	local cancelled = false

	local other_dmg = {
		["AwakeningPoints"] = 2
	}

	local debouncename = "Hado 88"

	local plrdata = table.unpack(args) 
	local status = util.StatusChecks.check(plr.Character,movestat)
	if status ~= true then return end

	local character = plr.Character

	local hmrp = character:FindFirstChild("HumanoidRootPart")
	local values = character:FindFirstChild("Values")
	local states = character:FindFirstChild("States")
	local chrstats = character:FindFirstChild("Stats")
	local humanoid = character:FindFirstChild("Humanoid")

	if util.ValueChecks:Check(character,movestat) ~= true then return end
	if states:FindFirstChild(debouncename) then return end

	local anim = script.Animations.Hado88Start
	local slash = humanoid.Animator:LoadAnimation(anim)

	slash:Play()

	values.Busy.Value = true
	util.CombatRemote.Effect:Fire(character,"Aizen","GetsugaStart", delayt)

	local starslow = util.CombatModule.slow(character,6,1)

	local cancel_connection
	cancel_connection = states.ChildAdded:Connect(function(child)
		if states:FindFirstChild(directory.states.Stunned) == nil then return end
		cancel_connection:Disconnect()

		slash:Stop()
		values.Busy.Value  = false
		cancelled          = true
		util.CombatModule.Debounce(plr,states,debouncename,cd/2)

		game.Debris:AddItem(starslow,0.01)
	end)


	task.wait(delayt)
	if cancelled then return end
	cancel_connection:Disconnect()

	local getsuga_slash = script.Instances.proj:Clone()
	getsuga_slash.Parent = workspace.Effects

	getsuga_slash:SetNetworkOwner(nil)

	getsuga_slash.CFrame = hmrp.CFrame + hmrp.CFrame.LookVector*1
	getsuga_slash.CFrame = getsuga_slash.CFrame * CFrame.fromOrientation(math.rad(0),math.rad(-90),0)
	getsuga_slash.CFrame = getsuga_slash.CFrame + Vector3.new(0,1,3)

	getsuga_slash.Attachment.WorldPosition = getsuga_slash.AssemblyCenterOfMass

	local linear_vel = getsuga_slash.Attachment.LinearVelocity
	linear_vel.VectorVelocity = -getsuga_slash.CFrame.RightVector * 150
	linear_vel.MaxForce = math.huge

	local name = "getsugaslash"..plr.Name
	local called_hit = util.Hitbox.new(
		plr,
		getsuga_slash.Hitbox,
		name,
		0.1,
		"GroundDetection",

		{   ["Duration"] = 3;
			["Type"]  = "ProjectileHit"
		}
	)

	local hit_detecting; 
	local alr_removed = false

	task.delay(0.3,function()
		if alr_removed then return end

		--getsuga_slash.Transparency = 0
		for i,v in pairs(getsuga_slash:GetDescendants()) do
			if v:IsA("ParticleEmitter") or v:IsA("Beam") then
				v.Enabled = true
			end
		end

	end)
	slash:Stop()
	local function remove()
		if alr_removed then return end
		alr_removed = true

		hit_detecting:Disconnect()
		called_hit:Janitor()
		game.Debris:AddItem(getsuga_slash,3)

		for i,v in pairs(getsuga_slash:GetDescendants()) do
			if v:IsA("ParticleEmitter") then
				v.Enabled = false
			end
		end

		getsuga_slash.Transparency = 1
	end

	delay(3,function()
		remove()
	end)

	hit_detecting = called_hit.Damaged.Event:Connect(function(v)
		remove()
		getsuga_slash.Anchored = true

		local pos = getsuga_slash.Position
		util.CombatRemote.Effect:Fire(pos,"Aizen","GetsugaBoom")

		local hitbox = script.Hitboxes.GetsugaTenshouHitbox:Clone()
		hitbox.Parent = workspace.Effects
		hitbox.Position = pos

		local name = "getsugahit"..plr.Name

		local called_hit = hitboxMod.new(
			plr,
			hitbox,
			name,
			1
		)

		called_hit.Damaged.Event:Connect(function(targets)
			for i,v in pairs(targets) do
				local damage = damager.Damage(v,plr,damage,true,other_dmg)

				if damage == true then

					if v.Humanoid.Health <= 0 then
						combatmodule.iframe(v,nil)
						combatmodule.UnitKnockback(v, v.HumanoidRootPart, 100, 0.4, 0 , 200 ,0, hmrp.Position)

						util.CombatRemote.Effect:Fire({v, character},"Ichigo","ImpactFrame")
					else
						combatmodule.UnitKnockback(character, v.HumanoidRootPart, 40, 0.2, 0 , 60 ,0, hitbox.Position)
						combatmodule.stun(v,2)
						util.Ragdoll.ragdoll(v,true,2)
					end

				end

			end

			called_hit:Janitor()
		end)

	end)

	values.Busy.Value = false

	util.CombatModule.Debounce(plr,states,debouncename,cd)
end
