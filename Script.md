--[[TestToolkit v66 - Otimizado]]
local P=game:GetService("Players")local RS=game:GetService("RunService")local UI=game:GetService("UserInputService")local TS=game:GetService("TweenService")local W=game:GetService("Workspace")local L=game:GetService("Lighting")local H=game:GetService("HttpService")local PS=game:GetService("PhysicsService")local R=game:GetService("ReplicatedStorage")local VU=game:GetService("VirtualUser")local LP=P.LocalPlayer local Cam=W.CurrentCamera W:GetPropertyChangedSignal("CurrentCamera"):Connect(function()if W.CurrentCamera then Cam=W.CurrentCamera end end)
local GN=""pcall(function()GN=game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name end)
local IS_MM2=game.PlaceId==142823291 or game.PlaceId==12278902 or game.PlaceId==10231138 or game.PlaceId==66654135 or(string.find(string.lower(GN),"murder mystery")~=nil)or(string.find(string.lower(game.Name),"murder mystery")~=nil)
local S={ESP=true,Rainbow=true,ShowNames=true,ShowHealth=true,Wallhack=true,ESPTeammates=true,ESPMaxDist=600,ESPMaxTargets=12,ESPHighlight=true,ESPColor=Color3.fromRGB(255,80,80),ESPFillTrans=0.65,ESPBox=1,ESPHeadDot=false,ESPTool=false,TracerOrigin=1,TeamCheck=true,TargetMode=3,AimEnabled=true,UseLegitAim=true,UseSilentAim=false,AimMode=1,AimPart=3,AimWallCheck=true,Smoothness=0.15,SnapAngle=6,AutoPredict=true,PredictScale=0.6,AimAtCursor=true,Prediction=0,BulletSpeed=0,ShotType=1,FireLock=true,Priority=1,AimKey=Enum.UserInputType.MouseButton2,SwitchKey=Enum.KeyCode.T,FOVEnabled=true,FOVRadius=150,AimMaxDist=0,SilentAimFOV=300,SilentAimVisible=true,TriggerBot=false,TriggerBotAlways=false,TriggerFOV=100,TriggerVisible=true,TriggerDelay=0.05,HitboxExpander=false,HitboxSize=6,HitboxRange=200,HitboxInvisible=false,HitboxWallCheck=false,HitboxIgnoreAllies=true,ExpandHead=true,ExpandTorso=true,ExpandUpperTorso=true,ExpandLowerTorso=false,WalkSpeedOn=false,WalkSpeed=32,WalkAutoLimit=true,JumpOn=false,JumpPower=100,Noclip=false,NoclipKey=Enum.KeyCode.V,Fly=false,FlyKey=Enum.KeyCode.F,FlySpeed=60,FlyAutoLimit=true,AntiVoid=false,AntiVoidY=math.max(W.FallenPartsDestroyHeight+100,-400),AntiKill=false,AntiKillMargin=200,AntiFling=false,AntiFlingRestore=true,AntiFlingSpeed=160,InfJump=false,FullBright=false,NoFog=false,CamFOVOn=false,CamFOV=90,AntiAFK=true,Tracers=false,MM2Mode=true,MM2SmartAim=true,MM2EspPlayers=false,MM2EspGun=false,MM2SilentAim=false,MM2AutoShootMurderer=false,MM2AutoGrabGun=false,MM2AutoKillAll=false,MM2KillAura=false,MM2KnifeThrownAim=false,MM2AutoWin=false,MM2AutoDodge=false,CoinFarmEnabled=false,CoinFarmSpeed=25,CoinFarmResetWhenFull=false,SeatInvisible=false,SeatInvisibleX=-25.95,SeatInvisibleY=84,SeatInvisibleZ=3537.55,Fling=false,FlingKey=Enum.KeyCode.G,FlingRange=12,FlingPower=3000,FlingMode=1,FlingMode2=2,FlingBots=true,FlingPlayers=false,FlingRepeat=0.1,FlingPlayerMode=1,FlingPlayerRange=50,ShowHUD=true,HUDX=10,HUDY=10,HUDEdit=false,Notifications=true,MenuAlpha=0.1,DeviceMode=1,MobileEdit=false,MobileBtnSize=56,MobileBtnAlpha=0.15,ShowBtnFly=true,ShowBtnNoclip=true,ShowBtnDown=true,ShowBtnEsp=true,ShowBtnTrace=true,ShowBtnInf=true,ShowBtnAim=true,ShowBtnFling=true,ShowBtnGrabGun=true}
local MK=Enum.KeyCode.RightControl
local C={Murderer=Color3.fromHSV(0,0.75,1),Sheriff=Color3.fromHSV(0.6,0.7,1),Hero=Color3.fromHSV(0.14,0.8,1),Innocent=Color3.fromHSV(0.33,0.6,0.95),Gun=Color3.fromHSV(0.12,0.9,1)}
local MS={Roles={},LastRoleFetch=0,LastShoot=0,LastStab=0,ActionBusy=false,BusySince=0,ShootBusy=false,KillBusy=false,LastDodge=0,FarmBusy=false,Bag={Current=0,Max=0},SkippedCoins={},FlingBusy=false,LastFlingPlayer=0}
local GL={}
pcall(function()local r=R:WaitForChild("Remotes",10)local g=r and r:WaitForChild("Gameplay",10)local e=r and r:FindFirstChild("Extras")local function rm(f,n,c)local x=f and f:FindFirstChild(n)x=x or R:FindFirstChild(n,true)return x and x:IsA(c)and x or nil end GL.PlayerData=rm(g,"GetCurrentPlayerData","RemoteFunction")GL.CoinCollected=rm(g,"CoinCollected","BaseRemoteEvent")GL.CoinsStarted=rm(g,"CoinsStarted","BaseRemoteEvent")GL.RoundStart=rm(g,"RoundStart","BaseRemoteEvent")end)
local function rt(pl)local c=(pl or LP).Character return c and c:FindFirstChild("HumanoidRootPart")end
local function hm(pl)local c=(pl or LP).Character return c and c:FindFirstChildOfClass("Humanoid")end
local function tl(pl,n)local c=pl.Character local b=pl:FindFirstChild("Backpack")return(c and c:FindFirstChild(n))or(b and b:FindFirstChild(n))end
local function busy()return MS.ActionBusy and os.clock()-MS.BusySince<6 end
local function setBusy(b)MS.ActionBusy=b MS.BusySince=os.clock()end
local function warpf(cf,fn)local r=rt()if not r or not cf or busy()then return false end setBusy(true)local h=r.CFrame local ok=pcall(function()r.CFrame=cf r.AssemblyLinearVelocity=Vector3.zero fn()end)local c=rt()if c then c.CFrame=h end setBusy(false)return ok end
local function refreshRoles(force)if not force and os.clock()-MS.LastRoleFetch<1 then return end MS.LastRoleFetch=os.clock()if not GL.PlayerData then return end local ok,roster=pcall(function()return GL.PlayerData:InvokeServer()end)if ok and type(roster)=="table"then MS.Roles=roster end end
local function roleOf(pl)if not pl then return "Innocent"end local e=MS.Roles[pl.Name]if e and not e.Dead and e.Role then return e.Role end if tl(pl,"Knife")then return "Murderer"end if tl(pl,"Gun")then return "Sheriff"end return e and e.Dead and "Dead"or"Innocent"end
local function findRole(role)for _,pl in ipairs(P:GetPlayers())do if pl~=LP and roleOf(pl)==role and rt(pl)then return pl end end return nil end
local function getMap()for _,c in ipairs(W:GetChildren())do if c.Name~="RegularLobby"and c:IsA("Model")and c:FindFirstChild("CoinContainer")then return c end end return nil end
local function iPlay()local e=MS.Roles[LP.Name]local h=hm()return getMap()~=nil and h~=nil and h.Health>0 and e~=nil and not e.Dead end
local function gunDrop()local m=getMap()return(m and m:FindFirstChild("GunDrop",true))or W:FindFirstChild("GunDrop")end
local Sh={}
function Sh.Gun()return tl(LP,"Gun")end
function Sh.Equip(t)local h=hm()if h and t.Parent~=LP.Character then h:EquipTool(t)end end
function Sh.ClearStand(t,tr)local pa=RaycastParams.new()pa.FilterType=Enum.RaycastFilterType.Exclude pa.FilterDescendantsInstances={LP.Character,t.Character}local stands={Vector3.new(0,0,6),Vector3.new(0,0,-6),Vector3.new(6,0,0),Vector3.new(-6,0,0)}for _,o in ipairs(stands)do local p=(tr.CFrame*CFrame.new(o)).Position if not W:Raycast(p,tr.Position-p,pa)then return CFrame.lookAt(p,tr.Position)end end return CFrame.lookAt((tr.CFrame*CFrame.new(stands[1])).Position,tr.Position)end
function Sh.Shoot(t)local g,tr=Sh.Gun(),rt(t)if not g or not tr or os.clock()-MS.LastShoot<1.2 then return false end MS.LastShoot=os.clock()Sh.Equip(g)return warpf(Sh.ClearStand(t,tr),function()task.wait(0.25)local r=rt()local a=r and r:FindFirstChild("GunRaycastAttachment")local ar=rt(t)or tr g.Shoot:FireServer(a and a.WorldCFrame or r.CFrame,ar.CFrame)task.wait(0.3)end)end
function Sh.ShootM()refreshRoles(true)local m=findRole("Murderer")if not m then return false,"No murderer"end for i=1,3 do if not Sh.Gun()then break end Sh.Shoot(m)task.wait(0.8)if hm(m)and hm(m).Health<=0 then return true,m.Name end task.wait(0.4)end return false,m.Name.." survived"end
function Sh.Grab()local d=gunDrop()if not d or Sh.Gun()or not iPlay()then return false end local p=d:IsA("BasePart")and d or d:FindFirstChildWhichIsA("BasePart",true)if not p then return false end return warpf(p.CFrame,function()local r=rt()if r and firetouchinterest then pcall(function()firetouchinterest(r,p,0)firetouchinterest(r,p,1)end)end task.wait(0.35)end)end
function Sh.AutoShoot()if not S.MM2AutoShootMurderer or MS.ShootBusy then return end if not Sh.Gun()or not iPlay()then return end MS.ShootBusy=true task.spawn(function()pcall(Sh.ShootM)MS.ShootBusy=false end)end
function Sh.AutoGrab()if S.MM2AutoGrabGun and roleOf(LP)~="Murderer"and gunDrop()then task.spawn(function()pcall(Sh.Grab)end)end end
local Mu={}
function Mu.Knife()return tl(LP,"Knife")end
function Mu.Stab(t)local k,tr=Mu.Knife(),rt(t)if not k or not tr then return false end Sh.Equip(k)local e=k:FindFirstChild("Events")if not e then return false end local st=tr.CFrame*CFrame.new(0,0,2)return warpf(st,function()e.KnifeStabbed:FireServer()task.wait()local ar=rt(t)or tr e.HandleTouched:FireServer(ar)task.wait(0.9)end)end
function Mu.Victims()local v={}for _,pl in ipairs(P:GetPlayers())do local h=hm(pl)local e=MS.Roles[pl.Name]local inR=e==nil or not e.Dead if pl~=LP and h and h.Health>0 and inR and rt(pl)then table.insert(v,pl)end end table.sort(v,function(a,b)local prA=(roleOf(a)=="Sheriff"or roleOf(a)=="Hero")and 1 or(roleOf(a)=="Innocent")and 2 or 99 local prB=(roleOf(b)=="Sheriff"or roleOf(b)=="Hero")and 1 or(roleOf(b)=="Innocent")and 2 or 99 return prA<prB end)return v end
function Mu.KillAll()if not Mu.Knife()then return 0 end local k=0 for _,v in ipairs(Mu.Victims())do if not Mu.Knife()then break end if Mu.Stab(v)then k=k+1 end end return k end
function Mu.AutoKill()if not S.MM2AutoKillAll or MS.KillBusy then return end if not Mu.Knife()or not iPlay()then return end MS.KillBusy=true task.spawn(function()pcall(Mu.KillAll)MS.KillBusy=false end)end
function Mu.KillAura()local k,r=Mu.Knife(),rt()if not S.MM2KillAura or not k or not r then return end if os.clock()-MS.LastStab<0.9 then return end local e=k:FindFirstChild("Events")for _,v in ipairs(Mu.Victims())do local vr=rt(v)if e and vr and(vr.Position-r.Position).Magnitude<=18 then MS.LastStab=os.clock()Sh.Equip(k)e.KnifeStabbed:FireServer()e.HandleTouched:FireServer(vr)return end end end
function Mu.NearVictim(origin)local b,bd for _,v in ipairs(Mu.Victims())do local vr=rt(v)local d=vr and(vr.Position-origin).Magnitude if d and(not bd or d<bd)then b,bd=vr,d end end return b end
local Hook={Restore=nil}
function Hook.Sync()local want=MS.Alive and(S.MM2SilentAim or S.MM2KnifeThrownAim)if want and not Hook.Restore then pcall(function()if not getrawmetatable or not setreadonly or not hookfunction then return end local mt=getrawmetatable(game)local old=mt.__namecall setreadonly(mt,false)mt.__namecall=newcclosure(function(self,...)if not MS.Alive or getnamecallmethod()~="FireServer"or(checkcaller and checkcaller())then return old(self,...)end local pa=self.Parent if S.MM2SilentAim and self.Name=="Shoot"and pa and pa.Name=="Gun"then local m=findRole("Murderer")local mr=m and rt(m)if mr then local o=... return old(self,o,mr.CFrame)end elseif S.MM2KnifeThrownAim and self.Name=="KnifeThrown"and pa and pa.Name=="Events"then local o=... local vr=Mu.NearVictim(o.Position)if vr then return old(self,o,vr.Position)end end return old(self,...)end)setreadonly(mt,true)Hook.Restore=function()pcall(function()setreadonly(mt,false)mt.__namecall=old setreadonly(mt,true)end)end end)elseif not want and Hook.Restore then local r=Hook.Restore Hook.Restore=nil pcall(r)end end
local function Kaitun()if not S.MM2AutoWin or not iPlay()then return end if Mu.Knife()then S.MM2AutoKillAll=true end if Sh.Gun()then S.MM2AutoShootMurderer=true end S.MM2AutoGrabGun=true S.MM2AutoDodge=true S.CoinFarmEnabled=true end
local function Dodge()local r=rt()local m=findRole("Murderer")local th=m and rt(m)if not S.MM2AutoDodge or not r or not th then return end if Mu.Knife()or busy()then return end if os.clock()-MS.LastDodge<2.5 then return end if(th.Position-r.Position).Magnitude>22 then return end local mp=getMap()local b,bd if mp then for _,c in ipairs(mp.CoinContainer:GetChildren())do if c:IsA("BasePart")then local d=(c.Position-th.Position).Magnitude if not bd or d>bd then b,bd=c,d end end end end if b then MS.LastDodge=os.clock()r.CFrame=CFrame.new(b.Position+Vector3.new(0,3,0))r.AssemblyLinearVelocity=Vector3.zero end end
local Fa={}
function Fa.BagFull()return MS.Bag.Max>0 and MS.Bag.Current>=MS.Bag.Max end
function Fa.CanRun()return S.CoinFarmEnabled and iPlay()and not Fa.BagFull()end
function Fa.Attach(r)local a=Instance.new("Attachment")a.Parent=r local m=Instance.new("LinearVelocity")m.Attachment0=a m.MaxForce=math.huge m.RelativeTo=Enum.ActuatorRelativeTo.World m.VectorVelocity=Vector3.zero m.Parent=r MS.FarmMover={Attachment=a,Velocity=m}return m end
function Fa.Detach()local m=MS.FarmMover if m then pcall(function()m.Velocity:Destroy()end)pcall(function()m.Attachment:Destroy()end)MS.FarmMover=nil end end
function Fa.Glide(pos)local r=rt()local m=MS.FarmMover if not r or not m or m.Velocity.Parent~=r then Fa.Detach()m=r and{Velocity=Fa.Attach(r)}end while m and Fa.CanRun()and r.Parent do local d=pos-r.Position if d.Magnitude<=1.5 then break end m.Velocity.VectorVelocity=d.Unit*math.min(S.CoinFarmSpeed,d.Magnitude*20)RS.Heartbeat:Wait()end if m then m.Velocity.VectorVelocity=Vector3.zero end end
function Fa.NearCoin(mp,origin)local mur=not Mu.Knife()and findRole("Murderer")local th=mur and rt(mur)local b,bd for _,c in ipairs(mp.CoinContainer:GetChildren())do local vi=c:FindFirstChild("CoinVisual")local sk=MS.SkippedCoins[c]local skp=sk and os.clock()-sk<4 local dg=th and(c.Position-th.Position).Magnitude<30 if c:IsA("BasePart")and vi and not vi:GetAttribute("Collected")and not skp and not dg then local d=(c.Position-origin).Magnitude if not bd or d<bd then b,bd=c,d end end end return b end
function Fa.Run()local r=rt()if r then Fa.Attach(r)end while Fa.CanRun()do local mp,hrp=getMap(),rt()local c=mp and hrp and Fa.NearCoin(mp,hrp.Position)if c and not busy()then Fa.Glide(c.Position)task.wait(0.15)MS.SkippedCoins[c]=os.clock()else local m=MS.FarmMover if m then m.Velocity.VectorVelocity=Vector3.zero end task.wait(0.3)end end end
function Fa.Step()if MS.FarmBusy or not Fa.CanRun()then return end MS.FarmBusy=true task.spawn(function()local ok=pcall(Fa.Run)Fa.Detach()MS.FarmBusy=false local h=hm()if ok and h and S.CoinFarmResetWhenFull and Fa.BagFull()then h.Health=0 end end)end
pcall(function()if GL.CoinCollected then GL.CoinCollected.OnClientEvent:Connect(function(_,c,m)MS.Bag.Current=tonumber(c)or 0 MS.Bag.Max=tonumber(m)or 0 end)end if GL.CoinsStarted then GL.CoinsStarted.OnClientEvent:Connect(function()MS.Bag.Current,MS.Bag.Max=0,0 table.clear(MS.SkippedCoins)end)end if GL.RoundStart then GL.RoundStart.OnClientEvent:Connect(function()MS.Bag.Current,MS.Bag.Max=0,0 table.clear(MS.Roles)end)end end)
local Hums={}
local function regHum(d)if d.ClassName=="Humanoid"then local m=d.Parent if m and m:IsA("Model")then Hums[m]=d end end end
W.DescendantAdded:Connect(regHum)
W.DescendantRemoving:Connect(function(d)if d.ClassName=="Humanoid"and Hums[d.Parent]==d then Hums[d.Parent]=nil end end)
task.spawn(function()local n=0 for _,d in ipairs(W:GetDescendants())do regHum(d)n=n+1 if n%300==0 then task.wait()end end end)
local RootCache=setmetatable({},{__mode="k"})
local function getRoot(m)local r=RootCache[m]if r and r.Parent then return r end r=m:FindFirstChild("HumanoidRootPart")or m.PrimaryPart RootCache[m]=r return r end
local TgtCache={}
local function isAlive(h)if not h or h.Health<=0 then return false end local st=h:GetState()if st==Enum.HumanoidStateType.Dead or st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then return false end if h.Health<5 then return false end local r=h.Parent:FindFirstChild("HumanoidRootPart")if r then if r.CFrame.UpVector:Dot(Vector3.new(0,1,0))<0.5 then return false end if h.FloorMaterial==Enum.Material.Air and r.AssemblyLinearVelocity.Y<-50 then return false end end local hd=h.Parent:FindFirstChild("Head")if hd and r and(hd.Position.Y-r.Position.Y)<0.5 then return false end return true end
local function getTargets(purp)local n=os.clock()local c=TgtCache[purp]if not c then c={t=-1,list={}}TgtCache[purp]=c end if n-c.t<0.15 then return c.list end c.t=n local l=c.list table.clear(l)local i=0 local mc=LP.Character local md=S.TargetMode for m,h in pairs(Hums)do if m~=mc and m.Parent and h.Parent and isAlive(h)then local pl=P:GetPlayerFromCharacter(m)local ok if pl then ok=pl~=LP and md~=2 else ok=md~=1 end if ok and purp=="aim"and IS_MM2 and S.MM2SmartAim then local mr=roleOf(LP)local tr=pl and roleOf(pl)or"Innocent"if mr=="Innocent"or mr=="Sheriff"or mr=="Hero"then if tr~="Murderer"then ok=false end end end if ok then local r=getRoot(m)if r then i=i+1 l[i]={Model=m,Humanoid=h,Root=r,Player=pl}end end end end return l end
local Lp=RaycastParams.new()Lp.FilterType=Enum.RaycastFilterType.Exclude
local function hasLOS(part,mdl)if not S.AimWallCheck then return true end local fl={LP.Character,Cam}local o=Cam.CFrame.Position local d=part.Position-o for _=1,4 do Lp.FilterDescendantsInstances=fl local res=W:Raycast(o,d,Lp)if not res then return true end local hit=res.Instance if hit:IsDescendantOf(mdl)then return true end if hit.Transparency>=0.9 or hit:FindFirstAncestorOfClass("Accessory")then fl[#fl+1]=hit else return false end end return false end
local function pickAim(m,h)local r=getRoot(m)local hd=m:FindFirstChild("Head")local to=m:FindFirstChild("UpperTorso")or m:FindFirstChild("Torso")or r local hf if S.AimPart==1 then hf=true elseif S.AimPart==2 then hf=false else hf=true end local order if hf then order={hd,to,r}else order={to,r,hd}end local best,bd for _,pt in ipairs(order)do if pt and pt:IsA("BasePart")and hasLOS(pt,m)then if S.AimPart~=3 then return pt end local sp,on=Cam:WorldToViewportPoint(pt.Position)local d=on and(Vector2.new(sp.X,sp.Y)-UI:GetMouseLocation()).Magnitude or math.huge if not bd or d<bd then best,bd=pt,d end end end return best end
local Cand={}
local CandT=-1
local function getCands(force)local n=os.clock()if not force and n-CandT<0.08 then return Cand end CandT=n local mo=UI:GetMouseLocation()local cp=Cam.CFrame.Position local fo=S.FOVEnabled local fl=S.FOVRadius+80 local md=S.AimMaxDist local rough={}for _,t in ipairs(getTargets("aim"))do local rp=t.Root.Position local sp,on=Cam:WorldToViewportPoint(rp)if on then local d=(Vector2.new(sp.X,sp.Y)-mo).Magnitude local wd=(rp-cp).Magnitude if(not fo or d<=fl)and(md==0 or wd<=md)then rough[#rough+1]={T=t,Dist=d,WorldDist=wd,Health=t.Humanoid.Health}end end end table.sort(rough,function(a,b)return a.Dist<b.Dist end)local out={}for i=1,#rough do if #out>=6 then break end local r=rough[i]local t=r.T local pt=pickAim(t.Model,t.Humanoid)if pt then local sp,on=Cam:WorldToViewportPoint(pt.Position)if on then local d=(Vector2.new(sp.X,sp.Y)-mo).Magnitude if not fo or d<=S.FOVRadius then out[#out+1]={Model=t.Model,Humanoid=t.Humanoid,Player=t.Player,Part=pt,Dist=d,WorldDist=(pt.Position-cp).Magnitude,Health=r.Health}end end end end table.sort(out,function(a,b)return a.Dist<b.Dist end)return out end
local Gui=Instance.new("ScreenGui")Gui.Name="TestToolkit"Gui.ResetOnSpawn=false Gui.IgnoreGuiInset=true Gui.DisplayOrder=100 Gui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling Gui.Parent=LP:WaitForChild("PlayerGui")
local Th={Bg=Color3.fromRGB(16,16,24),Panel=Color3.fromRGB(27,27,40),PanelHover=Color3.fromRGB(38,38,56),Accent=Color3.fromRGB(125,95,255),Accent2=Color3.fromRGB(255,95,190),Off=Color3.fromRGB(60,60,78),Text=Color3.fromRGB(235,235,245),SubText=Color3.fromRGB(150,150,172),Good=Color3.fromRGB(80,255,130),Bad=Color3.fromRGB(255,90,90)}
local function tw(o,p,t,st,dr)local x=TS:Create(o,TweenInfo.new(t or 0.18,st or Enum.EasingStyle.Quad,dr or Enum.EasingDirection.Out),p)x:Play()return x end
local function cn(o,r)local c=Instance.new("UICorner")c.CornerRadius=UDim.new(0,r)c.Parent=o return c end
local function str(o,c,t,th)local s=Instance.new("UIStroke")s.Color=c s.Transparency=t or 0.6 s.Thickness=th or 1 s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border s.Parent=o return s end
local FovC=Instance.new("Frame")FovC.AnchorPoint=Vector2.new(0.5,0.5)FovC.BackgroundTransparency=1 FovC.Parent=Gui cn(FovC,9999)str(FovC,Color3.new(1,1,1),0.15,1.5)
local SilC=Instance.new("Frame")SilC.AnchorPoint=Vector2.new(0.5,0.5)SilC.BackgroundTransparency=1 SilC.Parent=Gui cn(SilC,9999)str(SilC,Color3.fromRGB(255,100,255),0.3,1.5)
local TgbC=Instance.new("Frame")TgbC.AnchorPoint=Vector2.new(0.5,0.5)TgbC.BackgroundTransparency=1 TgbC.Parent=Gui cn(TgbC,9999)str(TgbC,Color3.fromRGB(255,100,100),0.4,1.5)
local Lk=Instance.new("Frame")Lk.AnchorPoint=Vector2.new(0.5,0.5)Lk.Size=UDim2.fromOffset(24,24)Lk.BackgroundTransparency=1 Lk.Visible=false Lk.Parent=Gui cn(Lk,9999)str(Lk,Th.Bad,0,2)
local TH=Instance.new("Frame")TH.AnchorPoint=Vector2.new(1,1)TH.Position=UDim2.new(1,-16,1,-16)TH.Size=UDim2.fromOffset(280,320)TH.BackgroundTransparency=1 TH.ZIndex=60 TH.Parent=Gui
local TLy=Instance.new("UIListLayout")TLy.VerticalAlignment=Enum.VerticalAlignment.Bottom TLy.HorizontalAlignment=Enum.HorizontalAlignment.Right TLy.SortOrder=Enum.SortOrder.LayoutOrder TLy.Padding=UDim.new(0,6)TLy.Parent=TH
local TO=0
local function notif(text,kind)if not S.Notifications then return end local co=0 for _,c in ipairs(TH:GetChildren())do if c:IsA("Frame")then co=co+1 end end if co>=5 then return end TO=TO+1 local col=kind=="on"and Th.Good or kind=="off"and Th.Bad or Th.Accent local r=Instance.new("Frame")r.LayoutOrder=TO r.Size=UDim2.new(1,0,0,34)r.BackgroundTransparency=1 r.ZIndex=60 r.Parent=TH local cd=Instance.new("Frame")cd.Size=UDim2.fromScale(1,1)cd.Position=UDim2.fromOffset(320,0)cd.BackgroundColor3=Th.Bg cd.BackgroundTransparency=0.08 cd.BorderSizePixel=0 cd.ZIndex=60 cd.Parent=r cn(cd,8)str(cd,col,0.55,1)local b=Instance.new("Frame")b.Size=UDim2.new(0,4,1,-12)b.Position=UDim2.fromOffset(7,6)b.BackgroundColor3=col b.BorderSizePixel=0 b.ZIndex=60 b.Parent=cd cn(b,2)local l=Instance.new("TextLabel")l.BackgroundTransparency=1 l.Position=UDim2.fromOffset(20,0)l.Size=UDim2.new(1,-28,1,0)l.Font=Enum.Font.GothamMedium l.TextSize=13 l.TextXAlignment=Enum.TextXAlignment.Left l.TextColor3=Th.Text l.TextTruncate=Enum.TextTruncate.AtEnd l.Text=text l.ZIndex=60 l.Parent=cd tw(cd,{Position=UDim2.fromOffset(0,0)},0.32,Enum.EasingStyle.Back)task.delay(2.4,function()if not cd.Parent then return end tw(cd,{Position=UDim2.fromOffset(320,0)},0.22,Enum.EasingStyle.Quad,Enum.EasingDirection.In)task.wait(0.25)r:Destroy()end)end
_G.__notif=notif
local menu=Instance.new("CanvasGroup")menu.Name="Menu"menu.AnchorPoint=Vector2.new(0.5,0.5)menu.Size=UDim2.fromOffset(440,560)menu.Position=UDim2.fromScale(0.5,0.5)menu.BackgroundColor3=Th.Bg menu.BorderSizePixel=0 menu.GroupTransparency=1 menu.Visible=false menu.ZIndex=50 menu.Parent=Gui cn(menu,14)str(menu,Th.Accent,0.5,1.5)
local mS=Instance.new("UIScale")mS.Scale=0.94 mS.Parent=menu
local aL=Instance.new("Frame")aL.Size=UDim2.new(1,0,0,3)aL.BackgroundColor3=Color3.new(1,1,1)aL.BorderSizePixel=0 aL.Parent=menu local aG=Instance.new("UIGradient")aG.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Th.Accent),ColorSequenceKeypoint.new(0.5,Th.Accent2),ColorSequenceKeypoint.new(1,Th.Accent)})aG.Parent=aL TS:Create(aG,TweenInfo.new(2.5,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut,-1,true),{Offset=Vector2.new(0.5,0)}):Play()
local TBar=Instance.new("Frame")TBar.Position=UDim2.fromOffset(0,3)TBar.Size=UDim2.new(1,0,0,46)TBar.BackgroundTransparency=1 TBar.Parent=menu
local Ti=Instance.new("TextLabel")Ti.BackgroundTransparency=1 Ti.Position=UDim2.fromOffset(16,6)Ti.Size=UDim2.new(1,-32,0,22)Ti.Font=Enum.Font.GothamBold Ti.TextSize=17 Ti.TextXAlignment=Enum.TextXAlignment.Left Ti.TextColor3=Th.Text Ti.Text="Test Toolkit v66"Ti.Parent=TBar
local Sb=Instance.new("TextLabel")Sb.BackgroundTransparency=1 Sb.Position=UDim2.fromOffset(16,26)Sb.Size=UDim2.new(1,-60,0,16)Sb.Font=Enum.Font.Gotham Sb.TextSize=12 Sb.TextXAlignment=Enum.TextXAlignment.Left Sb.TextColor3=Th.SubText Sb.Text="Ctrl direito abre/fecha"Sb.Parent=TBar
local CB=Instance.new("TextButton")CB.AnchorPoint=Vector2.new(1,0)CB.Position=UDim2.new(1,-10,0,6)CB.Size=UDim2.fromOffset(32,32)CB.BackgroundColor3=Th.Panel CB.AutoButtonColor=false CB.Font=Enum.Font.GothamBold CB.TextSize=15 CB.TextColor3=Th.SubText CB.Text="X"CB.Parent=TBar cn(CB,8)
local TabH=Instance.new("Frame")TabH.Position=UDim2.fromOffset(8,52)TabH.Size=UDim2.new(1,-16,0,34)TabH.BackgroundTransparency=1 TabH.Parent=menu
local TabI=Instance.new("Frame")TabI.Size=UDim2.new(1/9,-4,0,3)TabI.Position=UDim2.new(0,2,1,-3)TabI.BackgroundColor3=Th.Accent TabI.BorderSizePixel=0 TabI.ZIndex=2 TabI.Parent=TabH cn(TabI,2)
local PH=Instance.new("Frame")PH.Position=UDim2.fromOffset(8,92)PH.Size=UDim2.new(1,-16,1,-100)PH.BackgroundTransparency=1 PH.ClipsDescendants=true PH.Parent=menu
do local dr,ds,sp=false,nil,nil TBar.InputBegan:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then dr=true ds=i.Position sp=menu.Position end end)UI.InputChanged:Connect(function(i)if dr and(i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch)then local d=i.Position-ds menu.Position=UDim2.new(sp.X.Scale,sp.X.Offset+d.X,sp.Y.Scale,sp.Y.Offset+d.Y)end end)UI.InputEnded:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then dr=false end end)end
local Pages={}local TabB={}local TabIx={}local CurP=nil local ActT=nil local Ord=0 local TabC=0 local TabO={}local TabS={}local TabTN=9
local function selTab(n,inst)if ActT==n then return end local ix=TabS[n]or TabIx[n]local piv=ActT and(TabS[ActT]or TabIx[ActT])or ix for nm,p in pairs(Pages)do if nm~=n then p.Visible=false end end local p=Pages[n]p.Visible=true if inst then p.Position=UDim2.new()else local dir=ix>=piv and 1 or-1 p.Position=UDim2.fromOffset(30*dir,0)tw(p,{Position=UDim2.new()},0.24,Enum.EasingStyle.Quint)end ActT=n for nm,b in pairs(TabB)do local a=nm==n tw(b,{BackgroundColor3=a and Th.PanelHover or Th.Panel,TextColor3=a and Color3.new(1,1,1)or Th.SubText},0.16)end local t=UDim2.new((ix-1)/TabTN,2,1,-3)if inst then TabI.Position=t else tw(TabI,{Position=t},0.26,Enum.EasingStyle.Quint)end end
local function layTabs()local vis={}for _,n in ipairs(TabO)do if(n~="buttons"or UI.TouchEnabled)and(n~="mm2"or IS_MM2)then vis[#vis+1]=n end end local t=math.max(#vis,1)for _,b in pairs(TabB)do b.Visible=false end table.clear(TabS)for i,n in ipairs(vis)do local b=TabB[n]b.Visible=true b.Position=UDim2.new((i-1)/t,2,0,0)b.Size=UDim2.new(1/t,-4,1,-6)TabS[n]=i end TabTN=t TabI.Size=UDim2.new(1/t,-4,0,3)if ActT and TabS[ActT]then TabI.Position=UDim2.new((TabS[ActT]-1)/t,2,1,-3)end end
local function newPage(n,l)local p=Instance.new("ScrollingFrame")p.Name=n p.Size=UDim2.fromScale(1,1)p.BackgroundTransparency=1 p.BorderSizePixel=0 p.ScrollBarThickness=3 p.ScrollBarImageColor3=Th.Accent p.AutomaticCanvasSize=Enum.AutomaticSize.Y p.CanvasSize=UDim2.new()p.Visible=false p.Parent=PH local ly=Instance.new("UIListLayout")ly.Padding=UDim.new(0,6)ly.SortOrder=Enum.SortOrder.LayoutOrder ly.Parent=p local pd=Instance.new("UIPadding")pd.PaddingRight=UDim.new(0,6)pd.PaddingBottom=UDim.new(0,6)pd.Parent=p TabC=TabC+1 TabO[#TabO+1]=n local b=Instance.new("TextButton")b.Position=UDim2.new((TabC-1)/9,2,0,0)b.Size=UDim2.new(1/9,-4,1,-6)b.BackgroundColor3=Th.Panel b.BorderSizePixel=0 b.AutoButtonColor=false b.Font=Enum.Font.GothamBold b.TextSize=11 b.TextColor3=Th.SubText b.Text=l b.Parent=TabH cn(b,8)b.MouseButton1Click:Connect(function()selTab(n)end)Pages[n]=p TabB[n]=b TabIx[n]=TabC CurP=p return p end
local Ref={}
local function newRow(h,c)Ord=Ord+1 local r=Instance.new(c or"Frame")r.Size=UDim2.new(1,0,0,h)r.BackgroundColor3=Th.Panel r.BorderSizePixel=0 r.LayoutOrder=Ord if r:IsA("TextButton")then r.AutoButtonColor=false r.Text=""end cn(r,8)r.Parent=CurP return r end
local function rlbl(pa,tx,ds,rp)local l=Instance.new("TextLabel")l.BackgroundTransparency=1 l.Position=UDim2.fromOffset(12,ds and 4 or 0)l.Size=UDim2.new(1,-(rp or 66),ds and 0 or 1,ds and 18 or 0)l.Font=Enum.Font.Gotham l.TextSize=13 l.TextXAlignment=Enum.TextXAlignment.Left l.TextColor3=Th.Text l.TextTruncate=Enum.TextTruncate.AtEnd l.Text=tx l.Parent=pa if ds then local d=Instance.new("TextLabel")d.BackgroundTransparency=1 d.Position=UDim2.fromOffset(12,22)d.Size=UDim2.new(1,-24,0,13)d.Font=Enum.Font.Gotham d.TextSize=10 d.TextXAlignment=Enum.TextXAlignment.Left d.TextColor3=Th.SubText d.TextTruncate=Enum.TextTruncate.AtEnd d.Text=ds d.Parent=pa end return l end
local function hv(r)local s=Instance.new("UIScale")s.Parent=r r.MouseEnter:Connect(function()tw(r,{BackgroundColor3=Th.PanelHover},0.12)end)r.MouseLeave:Connect(function()tw(r,{BackgroundColor3=Th.Panel},0.12)tw(s,{Scale=1},0.1)end)r.MouseButton1Down:Connect(function()tw(s,{Scale=0.97},0.07)end)r.MouseButton1Up:Connect(function()tw(s,{Scale=1},0.16,Enum.EasingStyle.Back)end)end
local function addSec(t)Ord=Ord+1 local h=Instance.new("Frame")h.BackgroundTransparency=1 h.Size=UDim2.new(1,0,0,26)h.LayoutOrder=Ord h.Parent=CurP local l=Instance.new("TextLabel")l.BackgroundTransparency=1 l.Position=UDim2.fromOffset(4,6)l.Size=UDim2.new(1,-4,0,16)l.Font=Enum.Font.GothamBold l.TextSize=12 l.TextXAlignment=Enum.TextXAlignment.Left l.TextColor3=Th.Accent l.Text=string.upper(t)l.Parent=h local li=Instance.new("Frame")li.AnchorPoint=Vector2.new(0,1)li.Position=UDim2.new(0,4,1,0)li.Size=UDim2.new(1,-4,0,1)li.BackgroundColor3=Th.Accent li.BackgroundTransparency=0.7 li.BorderSizePixel=0 li.Parent=h return h end
local function addInfo(t,h)Ord=Ord+1 local l=Instance.new("TextLabel")l.BackgroundTransparency=1 l.Size=UDim2.new(1,0,0,h or 32)l.LayoutOrder=Ord l.Font=Enum.Font.Gotham l.TextSize=12 l.TextWrapped=true l.TextXAlignment=Enum.TextXAlignment.Left l.TextYAlignment=Enum.TextYAlignment.Top l.TextColor3=Th.SubText l.Text=t l.Parent=CurP return l end
local function addTgl(tx,k,ds,on)local r=newRow(ds and 42 or 36,"TextButton")rlbl(r,tx,ds)hv(r)local p=Instance.new("Frame")p.AnchorPoint=Vector2.new(1,0.5)p.Position=UDim2.new(1,-12,0.5,0)p.Size=UDim2.fromOffset(40,20)p.BorderSizePixel=0 p.Parent=r cn(p,10)local kn=Instance.new("Frame")kn.AnchorPoint=Vector2.new(0,0.5)kn.Size=UDim2.fromOffset(14,14)kn.BackgroundColor3=Color3.new(1,1,1)kn.BorderSizePixel=0 kn.Parent=p cn(kn,7)local function rnd(an)local on2=S[k]local pc=on2 and Th.Accent or Th.Off local kp=on2 and UDim2.new(1,-17,0.5,0)or UDim2.new(0,3,0.5,0)if an then tw(p,{BackgroundColor3=pc})tw(kn,{Position=kp},0.22,Enum.EasingStyle.Back)else p.BackgroundColor3=pc kn.Position=kp end end rnd(false)table.insert(Ref,function()rnd(true)end)r.MouseButton1Click:Connect(function()S[k]=not S[k]for _,rf in ipairs(Ref)do rf()end notif(tx..(S[k]and": ligado"or": desligado"),S[k]and"on"or"off")if on then on()end end)end
local function addCyc(tx,k,op,ds,on)local r=newRow(ds and 42 or 36,"TextButton")hv(r)local l=rlbl(r,"",ds,30)local ar=Instance.new("TextLabel")ar.BackgroundTransparency=1 ar.AnchorPoint=Vector2.new(1,0.5)ar.Position=UDim2.new(1,-12,0.5,0)ar.Size=UDim2.fromOffset(16,16)ar.Font=Enum.Font.GothamBold ar.TextSize=14 ar.TextColor3=Th.Accent ar.Text=">"ar.Parent=r local function rnd()l.Text=tx..": "..op[S[k]]end rnd()r.MouseButton1Click:Connect(function()S[k]=(S[k]%#op)+1 table.clear(TgtCache)CandT=-1 rnd()if on then on()end end)end
local SlD=nil local SlP=nil
local function addSld(tx,k,mn,mx,st,dc,ds,on)local pa=CurP local r=newRow(ds and 58 or 50,"Frame")local nm=rlbl(r,tx,ds,0)nm.Size=UDim2.new(0.65,-12,0,ds and 18 or 28)local vl=Instance.new("TextLabel")vl.BackgroundTransparency=1 vl.AnchorPoint=Vector2.new(1,0)vl.Position=UDim2.new(1,-12,0,ds and 4 or 0)vl.Size=UDim2.new(0.35,-12,0,ds and 18 or 28)vl.Font=Enum.Font.GothamBold vl.TextSize=13 vl.TextXAlignment=Enum.TextXAlignment.Right vl.TextColor3=Th.SubText vl.Parent=r local ht=Instance.new("TextButton")ht.BackgroundTransparency=1 ht.Text=""ht.Position=UDim2.new(0,12,0,ds and 40 or 28)ht.Size=UDim2.new(1,-24,0,16)ht.Parent=r local b=Instance.new("Frame")b.AnchorPoint=Vector2.new(0,0.5)b.Position=UDim2.new(0,0,0.5,0)b.Size=UDim2.new(1,0,0,6)b.BackgroundColor3=Th.Off b.BorderSizePixel=0 b.Parent=ht cn(b,3)local f=Instance.new("Frame")f.Size=UDim2.new(0,0,1,0)f.BackgroundColor3=Th.Accent f.BorderSizePixel=0 f.Parent=b cn(f,3)local hd=Instance.new("Frame")hd.AnchorPoint=Vector2.new(0.5,0.5)hd.Position=UDim2.new(1,0,0.5,0)hd.Size=UDim2.fromOffset(12,12)hd.BackgroundColor3=Color3.new(1,1,1)hd.BorderSizePixel=0 hd.Parent=f cn(hd,6)local fmt="%."..(dc or 0).."f"local function rnd()local rl=math.clamp((S[k]-mn)/(mx-mn),0,1)vl.Text=string.format(fmt,S[k])f.Size=UDim2.new(rl,0,1,0)end rnd()table.insert(Ref,rnd)local function setf(x)local rl=math.clamp((x-b.AbsolutePosition.X)/b.AbsoluteSize.X,0,1)local v=mn+rl*(mx-mn)v=math.floor(v/st+0.5)*st S[k]=math.clamp(v,mn,mx)rnd()if on then on()end end ht.InputBegan:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then setf(i.Position.X)SlD=setf SlP=pa pa.ScrollingEnabled=false end end)end
UI.InputChanged:Connect(function(i)if SlD and(i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch)then SlD(i.Position.X)end end)UI.InputEnded:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then SlD=nil if SlP then SlP.ScrollingEnabled=true SlP=nil end end end)
local function addBtn(tx,ds,on)local r=newRow(ds and 42 or 36,"TextButton")hv(r)local l=rlbl(r,tx,ds,30)l.TextColor3=Th.Accent r.MouseButton1Click:Connect(function()if on then on()end end)end
local Reb=nil
local function addKb(tx,k,ds,am)local r=newRow(ds and 42 or 36,"TextButton")if UI.TouchEnabled and not UI.KeyboardEnabled then r.Visible=false end hv(r)local l=rlbl(r,"",ds,30)local function rnd()l.Text=tx..": ["..S[k].Name.."]"end rnd()r.MouseButton1Click:Connect(function()l.Text=tx..": pressione uma tecla..."Reb={mouse=am,apply=function(bd)if bd then S[k]=bd end rnd()end}end)end
local function buildUI()
local MenuOpen=false
local function refreshAll()for _,r in ipairs(Ref)do r()end end
_G.__uiRefresh=refreshAll
local function setMenu(op)MenuOpen=op local vp=Cam.ViewportSize local sc=math.clamp(math.min(vp.X/470,vp.Y/590),0.5,1)if op then menu.Visible=true mS.Scale=sc*0.94 tw(menu,{GroupTransparency=S.MenuAlpha},0.2)tw(mS,{Scale=sc},0.26,Enum.EasingStyle.Back)else tw(menu,{GroupTransparency=1},0.16)tw(mS,{Scale=sc*0.94},0.16)task.delay(0.18,function()if not MenuOpen then menu.Visible=false end end)end end
local function togMenu()setMenu(not MenuOpen)end
_G.__togMenu=togMenu
CB.MouseButton1Click:Connect(function()setMenu(false)end)
newPage("aim","Mira")
addSec("Modo")
addTgl("Aimbot","AimEnabled","Ativa a mira")
addTgl("Legit Aim","UseLegitAim","Move a câmera")
addTgl("Silent Aim","UseSilentAim","Redireciona tiro")
addCyc("Modo","AimMode",{"Segurar","Alternar","Automático"},"Como ativa")
addCyc("Parte","AimPart",{"Cabeça","Corpo","Auto"},"Parte alvo")
addCyc("Alvo","TargetMode",{"Players","Bots","Ambos"},"Quem mirar")
addTgl("Ignorar equipe","TeamCheck","Não mira aliados")
addTgl("Checar parede","AimWallCheck","Só visíveis")
addSec("Legit Aim")
addSld("Suavidade","Smoothness",0.01,0.95,0.01,2,"Menor = mais rápida")
addSld("Ângulo trava","SnapAngle",0,30,1,0,"Cola abaixo deste")
addTgl("Trava ao atirar","FireLock","Sem suavização")
addSec("Silent Aim")
addSld("FOV Silent","SilentAimFOV",50,800,10,0,"Raio")
addTgl("Só visíveis","SilentAimVisible","Com linha de visão")
addSec("Trigger Bot")
addTgl("Trigger Bot","TriggerBot","Atira automatico")
addTgl("Sempre ativo","TriggerBotAlways","Sem tecla")
addSld("FOV Trigger","TriggerFOV",5,400,1,0,"Raio")
addSld("Delay","TriggerDelay",0.01,1,0.01,2,"Entre tiros")
addSec("FOV")
addTgl("Limitar FOV","FOVEnabled","Só dentro do círculo")
addSld("Raio FOV","FOVRadius",20,600,5,0,"Pixels")
addSld("Distância máx","AimMaxDist",0,2000,50,0,"0=sem limite")
addSec("Teclas")
addKb("Tecla mira","AimKey","Ativa mira",true)
addKb("Trocar alvo","SwitchKey","Próximo alvo")
newPage("hitbox","Hitbox")
addSec("Hitbox")
addTgl("Hitbox Expander","HitboxExpander","Aumenta hitbox")
addTgl("Invisível","HitboxInvisible","Hitbox transparente")
addTgl("Checar parede","HitboxWallCheck","Só visíveis")
addSld("Tamanho","HitboxSize",5,30,1,0,"Studs")
addSld("Alcance","HitboxRange",10,1000,10,0,"Distância")
addSec("Partes")
addTgl("Head","ExpandHead","")
addTgl("Torso","ExpandTorso","")
addTgl("UpperTorso","ExpandUpperTorso","")
addTgl("LowerTorso","ExpandLowerTorso","")
newPage("esp","ESP")
addSec("ESP")
addTgl("ESP","ESP","Mostra alvos")
addTgl("Wallhack","Wallhack","Ver atrás")
addTgl("Highlight","ESPHighlight","Contorno")
addTgl("Rainbow","Rainbow","Cor animada")
addTgl("Aliados","ESPTeammates","Mostra aliados")
addCyc("Caixa","ESPBox",{"Off","Completa","Cantos"},"Moldura")
addTgl("Nomes","ShowNames","Nome+dist")
addTgl("Vida","ShowHealth","Barra")
addTgl("Ponto cabeça","ESPHeadDot","Marca")
addTgl("Item mão","ESPTool","Ferramenta")
addSec("Tracers")
addTgl("Linhas","Tracers","Linha da tela")
addCyc("Origem","TracerOrigin",{"Baixo","Centro","Cursor"},"De onde")
addSec("Limites")
addSld("Dist máx","ESPMaxDist",0,3000,50,0,"0=sem limite")
addSld("Máx alvos","ESPMaxTargets",1,30,1,0,"Menos=FPS")
addSld("Transparência","ESPFillTrans",0,1,0.05,2,"0=sólido")
if IS_MM2 then
newPage("mm2","MM2")
addSec("Modo")
addInfo("Detecção via RemoteFunction. Cores: Murder=vermelho, Sheriff=azul, Hero=amarelo, Inocente=verde.",50)
addTgl("Ativar MM2","MM2Mode","Ativa funções")
addTgl("ESP roles","MM2EspPlayers","Cores por role")
addTgl("Gun ESP","MM2EspGun","Arma dropada")
addSec("Sheriff")
addTgl("Silent Aim","MM2SilentAim","Tiro vai no Murder",function()Hook.Sync()end)
addTgl("Auto Shoot","MM2AutoShootMurderer","Atira no Murder")
addBtn("Shoot Murderer Now","Atira agora",function()task.spawn(function()local ok,d=Sh.ShootM()notif(ok and("Atirou em "..tostring(d))or tostring(d or"Sem arma"),ok and"on"or"off")end)end)
addTgl("Auto Grab Gun","MM2AutoGrabGun","Pega arma")
addBtn("Grab Gun Now","Pega agora",function()task.spawn(function()local g=Sh.Grab()notif(g and"Pego"or"Nada",g and"on"or"off")end)end)
addSec("Murderer")
addTgl("Auto Kill All","MM2AutoKillAll","Mata todos")
addBtn("Kill All Now","Mata agora",function()task.spawn(function()local k=Mu.KillAll()notif("Matou "..k,k>0 and"on"or"off")end)end)
addTgl("Kill Aura","MM2KillAura","Mata quem chegar perto")
addTgl("Knife Throw Aim","MM2KnifeThrownAim","Faca no mais próximo",function()Hook.Sync()end)
addSec("Auto Play")
addTgl("Auto Win","MM2AutoWin","Joga a rodada")
addTgl("Auto Dodge","MM2AutoDodge","Foge do Murder")
addSec("Coin Farm")
addTgl("Auto Farm","CoinFarmEnabled","Coleta moedas")
addSld("Velocidade","CoinFarmSpeed",16,28,1,0,"Farm speed")
addTgl("Reset cheio","CoinFarmResetWhenFull","Morre quando cheio")
addSec("Status")
local mstat=addInfo("Aguardando...",80)
table.insert(Ref,function()pcall(function()if not mstat or not mstat.Parent then return end refreshRoles()local mu,sh,inn=0,0,0 for m,h in pairs(Hums)do if h and h.Parent and h.Health>0 then local pl=P:GetPlayerFromCharacter(m)if pl then local r=roleOf(pl)if r=="Murderer"then mu=mu+1 elseif r=="Sheriff"or r=="Hero"then sh=sh+1 elseif r=="Innocent"then inn=inn+1 end end end end local mr=roleOf(LP)local bg=string.format("%d/%d",MS.Bag.Current or 0,MS.Bag.Max or 0)mstat.Text=string.format("Você é: %s\nMurder: %d | Sheriff/Hero: %d | Inocente: %d\nBag: %s",tostring(mr or"?"),mu,sh,inn,bg)end)end)
end
newPage("player","Jogador")
addSec("Movimento")
addTgl("Velocidade","WalkSpeedOn","Muda speed")
addSld("Valor","WalkSpeed",16,5000,5,0,"Studs/s")
addTgl("Speed segura","WalkAutoLimit","Anti-puxão")
addTgl("Pulo","JumpOn","Muda pulo")
addSld("Força pulo","JumpPower",50,900,5,0,"")
addTgl("Pulo infinito","InfJump","Pula no ar")
addTgl("Voo","Fly","Espaço/Ctrl")
addSld("Vel voo","FlySpeed",10,5000,5,0,"Studs/s")
addTgl("Voo seguro","FlyAutoLimit","Anti-puxão")
addTgl("Noclip","Noclip","Atravessa paredes")
addSec("Teclas")
addKb("Tecla voo","FlyKey","Liga voo")
addKb("Tecla noclip","NoclipKey","Liga noclip")
newPage("misc","Extras")
addSec("Visão")
addTgl("Luz total","FullBright","Sem escuridão")
addTgl("Sem neblina","NoFog","Enxerga longe")
addTgl("FOV câmera","CamFOVOn","Muda FOV")
addSld("Valor FOV","CamFOV",40,120,1,0,"")
addTgl("Anti-AFK","AntiAFK","Evita kick")
addSec("Interface")
addTgl("HUD","ShowHUD","Mostra HUD")
addTgl("Mover HUD","HUDEdit","Arrastar HUD")
addTgl("Notificações","Notifications","Avisos")
addCyc("Dispositivo","DeviceMode",{"Auto","PC","Mobile"},"Layout")
addSld("Transparência menu","MenuAlpha",0,0.6,0.05,2,"",function()if MenuOpen then menu.GroupTransparency=S.MenuAlpha end end)
newPage("fling","Fling")
addSec("Bots")
addTgl("Fling Bots","Fling","Arremessa bots")
addCyc("Modo","FlingMode",{"Todos","Mira","Próximo"},"Quem")
addCyc("Tipo","FlingMode2",{"Empurrar","Void"},"Como")
addSld("Alcance","FlingRange",3,30,1,0,"Studs")
addSld("Força","FlingPower",500,5000,100,0,"")
addSec("Players")
addInfo("Arremessa players de verdade (universal).",30)
addTgl("Fling Players","FlingPlayers","Arremessa players")
addCyc("Modo","FlingPlayerMode",{"Próximo","Mira","Todos"},"Quem")
addSld("Alcance","FlingPlayerRange",5,200,5,0,"Studs")
addSec("Teclas")
addKb("Tecla Fling","FlingKey","Liga/desliga")
newPage("security","Segurança")
addSec("Status")
local sstat=addInfo("Carregando...",44)
addSec("Proteção")
addTgl("Anti Void","AntiVoid","Volta se cair")
addTgl("Anti Kill","AntiKill","Não deixa morrer")
addTgl("Anti Fling","AntiFling","Bloqueia arremesso")
addTgl("Anti AFK","AntiAFK","Evita kick")
table.insert(Ref,function()if sstat and sstat.Parent then local p={}if S.AntiVoid then p[#p+1]="AntiVoid"end if S.AntiKill then p[#p+1]="AntiKill"end if S.AntiFling then p[#p+1]="AntiFling"end if S.AntiAFK then p[#p+1]="AntiAFK"end sstat.Text=#p==0 and"Nada ativo"or("Ativas: "..table.concat(p,", "))end end)
newPage("buttons","Botões")
addSec("Botões")
addTgl("Botão MIRA","ShowBtnAim","")
addTgl("Botão VOO","ShowBtnFly","")
addTgl("Botão NOCLIP","ShowBtnNoclip","")
addTgl("Botão DESCER","ShowBtnDown","")
addTgl("Botão ESP","ShowBtnEsp","")
addTgl("Botão LINHA","ShowBtnTrace","")
addTgl("Botão PULO","ShowBtnInf","")
addTgl("Botão FLING","ShowBtnFling","")
addTgl("Botão GRAB","ShowBtnGrabGun","")
addSec("Aparência")
addTgl("Editar","MobileEdit","Arrastar")
addSld("Tamanho","MobileBtnSize",40,90,1,0,"px")
addSld("Transparência","MobileBtnAlpha",0,0.9,0.05,2,"")
layTabs()
selTab("aim",true)
refreshAll()
end
pcall(buildUI)
_G.TT_S=S
_G.TT_Sh=Sh
_G.TT_Mu=Mu
_G.TT_GL=GL
_G.TT_MS=MS
_G.TT_Hum=Hums
_G.TT_RT=rt
_G.TT_HM=hm
_G.TT_RoleOf=roleOf
_G.TT_FindRole=findRole
_G.TT_GetMap=getMap
_G.TT_Warp=warpf
_G.TT_Busy=busy
_G.TT_SetBusy=setBusy
_G.TT_C=Color3.fromHSV

-- ESP
local ERoot=Instance.new("Frame")ERoot.Size=UDim2.fromScale(1,1)ERoot.BackgroundTransparency=1 ERoot.ZIndex=1 ERoot.Parent=Gui
local EObj={}
local function mkF(p,rd)local f=Instance.new("Frame")f.BorderSizePixel=0 f.Visible=false f.ZIndex=1 f.Parent=p if rd then cn(f,9999)end return f end
local function mkT(p,sz)local l=Instance.new("TextLabel")l.BackgroundTransparency=1 l.Font=Enum.Font.GothamBold l.TextSize=sz l.TextColor3=Color3.new(1,1,1)l.TextStrokeTransparency=0.35 l.Size=UDim2.fromOffset(220,14)l.Visible=false l.ZIndex=5 l.Parent=p return l end
local function newEsp(m)local o={Model=m,Seen=0,H=5.5,HAt=-1}local hl=Instance.new("Highlight")hl.Adornee=m hl.Enabled=false hl.FillTransparency=0.65 hl.OutlineTransparency=0 hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop hl.Parent=Gui o.HL=hl local h=Instance.new("Frame")h.BackgroundTransparency=1 h.Size=UDim2.fromScale(1,1)h.Visible=false h.ZIndex=1 h.Parent=ERoot o.Holder=h o.Box=mkF(h)o.Box.BackgroundTransparency=1 o.BS=str(o.Box,Color3.new(1,1,1),0,1.5)o.C={}for i=1,8 do o.C[i]=mkF(h)end o.HB=mkF(h)o.HB.BackgroundColor3=Color3.new(0,0,0)o.HB.BackgroundTransparency=0.4 o.HF=Instance.new("Frame")o.HF.AnchorPoint=Vector2.new(0,1)o.HF.Position=UDim2.fromScale(0,1)o.HF.BorderSizePixel=0 o.HF.ZIndex=1 o.HF.Parent=o.HB o.Name=mkT(h,13)o.Name.AnchorPoint=Vector2.new(0.5,1)o.Dot=mkF(h,true)o.Dot.AnchorPoint=Vector2.new(0.5,0.5)o.Dot.Size=UDim2.fromOffset(6,6)o.Tr=mkF(h)o.Tr.AnchorPoint=Vector2.new(0.5,0.5)return o end
local function hideE(o)o.Holder.Visible=false o.HL.Enabled=false end
local function destE(m,o)pcall(function()o.HL:Destroy()end)pcall(function()o.Holder:Destroy()end)EObj[m]=nil end
local DEsp={}
local function mkD(t)local hl=Instance.new("Highlight")hl.Adornee=t hl.FillColor=C.Gun hl.OutlineColor=C.Gun hl.FillTransparency=0.65 hl.OutlineTransparency=0 hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop hl.Parent=Gui local n=Instance.new("TextLabel")n.BackgroundTransparency=1 n.Font=Enum.Font.GothamBold n.TextSize=14 n.TextColor3=C.Gun n.TextStrokeTransparency=0.4 n.Size=UDim2.fromOffset(160,18)n.AnchorPoint=Vector2.new(0.5,1)n.Text="GUN DROP"n.ZIndex=5 n.Parent=ERoot return{HL=hl,Name=n,Tool=t}end
local function updD()if not S.MM2EspGun or not S.MM2Mode or not IS_MM2 then for t,o in pairs(DEsp)do pcall(function()o.HL:Destroy()end)pcall(function()o.Name:Destroy()end)DEsp[t]=nil end return end local d=gunDrop()if not d then for t,o in pairs(DEsp)do pcall(function()o.HL:Destroy()end)pcall(function()o.Name:Destroy()end)DEsp[t]=nil end return end if not DEsp[d]then for t,o in pairs(DEsp)do pcall(function()o.HL:Destroy()end)pcall(function()o.Name:Destroy()end)DEsp[t]=nil end DEsp[d]=mkD(d)end local o=DEsp[d]local h=d:IsA("BasePart")and d or d:FindFirstChildWhichIsA("BasePart",true)if h then local sp,on=Cam:WorldToViewportPoint(h.Position+Vector3.new(0,2,0))o.Name.Visible=on if on then o.Name.Position=UDim2.fromOffset(sp.X,sp.Y)end end end
local function px(v)return math.floor(v+0.5)end
local function drawE(o,t,d,n,vp)local m=t.Model local col=(S.Rainbow and Color3.fromHSV((n*0.25)%1,0.85,1))or S.ESPColor local rl=nil if IS_MM2 and S.MM2Mode and S.MM2EspPlayers and t.Player then rl=roleOf(t.Player)if rl=="Murderer"then col=C.Murderer elseif rl=="Sheriff"then col=C.Sheriff elseif rl=="Hero"then col=C.Hero elseif rl=="Innocent"then col=C.Innocent end end if S.ESPHighlight then local h=o.HL h.Enabled=true h.FillColor=col h.OutlineColor=col h.FillTransparency=S.ESPFillTrans h.DepthMode=S.Wallhack and Enum.HighlightDepthMode.AlwaysOnTop or Enum.HighlightDepthMode.Occluded else o.HL.Enabled=false end local rp=t.Root.Position if n-o.HAt>0.5 then o.HAt=n local ok,sz=pcall(function()return m:GetExtentsSize()end)if ok and sz.Y>1 then o.H=sz.Y end end local h=o.H local c=Cam:WorldToViewportPoint(rp)if c.Z<=0 then o.Holder.Visible=false return end local tp=Cam:WorldToViewportPoint(rp+Vector3.new(0,h*0.45,0))local bp=Cam:WorldToViewportPoint(rp-Vector3.new(0,h*0.55,0))local bh=math.max(bp.Y-tp.Y,6)local bw=bh*0.55 local x,y=px(c.X-bw/2),px(tp.Y)bw,bh=px(bw),px(bh)o.Holder.Visible=true local bm=S.ESPBox o.Box.Visible=bm==2 if bm==2 then o.Box.Position=UDim2.fromOffset(x,y)o.Box.Size=UDim2.fromOffset(bw,bh)o.BS.Color=col end if bm==3 then local L,th=math.max(4,px(bw*0.28)),2 local r,bt=x+bw,y+bh local sp={{x,y,L,th},{x,y,th,L},{r-L,y,L,th},{r-th,y,th,L},{x,bt-th,L,th},{x,bt-L,th,L},{r-L,bt-th,L,th},{r-th,bt-L,th,L}}for i=1,8 do local f,s=o.C[i],sp[i]f.BackgroundColor3=col f.Position=UDim2.fromOffset(s[1],s[2])f.Size=UDim2.fromOffset(s[3],s[4])f.Visible=true end else for i=1,8 do o.C[i].Visible=false end end if S.ShowHealth then local hm=t.Humanoid local fr=math.clamp(hm.Health/math.max(hm.MaxHealth,1),0,1)o.HB.Visible=true o.HB.Position=UDim2.fromOffset(x-6,y)o.HB.Size=UDim2.fromOffset(3,bh)o.HF.Size=UDim2.new(1,0,fr,0)o.HF.BackgroundColor3=Color3.fromHSV(fr*0.33,0.9,1)else o.HB.Visible=false end if S.ShowNames or(IS_MM2 and S.MM2EspPlayers and t.Player)then local pl=t.Player local rt2=""if IS_MM2 and S.MM2Mode and rl then rt2=" ["..string.upper(rl).."]"end o.Name.Visible=true o.Name.Text=(pl and pl.DisplayName or m.Name)..rt2.."  ["..math.floor(d).."m]"o.Name.TextColor3=col o.Name.Position=UDim2.fromOffset(px(c.X),y-2)o.Name.ZIndex=5 else o.Name.Visible=false end if S.ESPHeadDot then local hd=m:FindFirstChild("Head")if hd and hd:IsA("BasePart")then local hp,on=Cam:WorldToViewportPoint(hd.Position)o.Dot.Visible=on o.Dot.BackgroundColor3=col o.Dot.Position=UDim2.fromOffset(px(hp.X),px(hp.Y))else o.Dot.Visible=false end else o.Dot.Visible=false end if S.Tracers then local p1 if S.TracerOrigin==1 then p1=Vector2.new(vp.X/2,vp.Y)elseif S.TracerOrigin==2 then p1=vp/2 else p1=UI:GetMouseLocation()end local p2=Vector2.new(c.X,c.Y)local dd=p2-p1 local ln=dd.Magnitude if ln>2 then o.Tr.Visible=true o.Tr.BackgroundColor3=col o.Tr.Size=UDim2.fromOffset(ln,1.5)o.Tr.Position=UDim2.fromOffset((p1.X+p2.X)/2,(p1.Y+p2.Y)/2)o.Tr.Rotation=math.deg(math.atan2(dd.Y,dd.X))else o.Tr.Visible=false end else o.Tr.Visible=false end end
local EPool={}
local ESel={}
local ESelAt=-1
local EWasOn=false
local function updEsp(n)if not S.ESP then if EWasOn then EWasOn=false for _,o in pairs(EObj)do hideE(o)end end return end EWasOn=true local cp=Cam.CFrame.Position if n-ESelAt>=0.1 then ESelAt=n local md=S.ESPMaxDist table.clear(ESel)local i=0 for _,t in ipairs(getTargets("esp"))do local d=(t.Root.Position-cp).Magnitude if md==0 or d<=md then i=i+1 local e=EPool[i]if not e then e={}EPool[i]=e end e.T,e.D=t,d ESel[i]=e end end table.sort(ESel,function(a,b)return a.D<b.D end)end local vp=Cam.ViewportSize for i=1,math.min(#ESel,S.ESPMaxTargets)do local t=ESel[i].T if t.Model.Parent and t.Root.Parent then local o=EObj[t.Model]if not o then o=newEsp(t.Model)EObj[t.Model]=o end o.Seen=n drawE(o,t,(t.Root.Position-cp).Magnitude,n,vp)end end for m,o in pairs(EObj)do if o.Seen~=n then hideE(o)if n-o.Seen>3 or not m.Parent then destE(m,o)end end end updD()end
local AimTgt=nil
local AimAct=false
local Spec=false
local SilentT=nil
local SilentA=false
local Firing=false
UI.InputBegan:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 then Firing=true end end)
UI.InputEnded:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 then Firing=false end end)
local function isAim()return S.AimEnabled and(AimAct or S.AimMode==3)end
local function getVel(m,r)local tr=trackers[m]local p=r.Position if not tr then tr={pos=p,t=os.clock(),vel=r.AssemblyLinearVelocity,acc=Vector3.zero}trackers[m]=tr return tr.vel,tr.acc end local el=os.clock()-tr.t if el>=0.05 then local rw=(p-tr.pos)/el local ol=tr.vel if rw.Magnitude>400 then tr.vel=Vector3.zero tr.acc=Vector3.zero else tr.vel=ol:Lerp(rw,0.6)local a=(tr.vel-ol)/el if a.Magnitude>80 then a=a.Unit*80 end tr.acc=tr.acc:Lerp(a,0.4)end tr.pos=p tr.t=os.clock()end return tr.vel,tr.acc end
local trackers=setmetatable({},{__mode="k"})
local function predict(t,dt)local m,h,pt=t.Model,t.Humanoid,t.Part local r=getRoot(m)local p=pt.Position if not r then return p end local v,a=getVel(m,r)local st=S.ShotType local hv=Vector3.new(v.X,0,v.Z)if hv.Magnitude<1.5 then hv=Vector3.zero end local tm=S.Prediction if os.clock()-pingAt>0.5 then pingAt=os.clock()pingVal=LP:GetNetworkPing()end if st==1 then if S.AutoPredict then tm=tm+pingVal*0.5*S.PredictScale end else if S.AutoPredict then tm=tm+(pingVal+0.04)*S.PredictScale end end tm=math.clamp(tm,0,0.6)if tm<=0 then return p end return p+hv*tm+Vector3.new(0,math.clamp(v.Y,-500,500)*tm,0)end
local pingAt=-1
local pingVal=0.05
local hookInst=false
local function instHooks()if hookInst then return end if not hookfunction then return end pcall(function()local og=Cam.WorldToViewportPoint Cam.WorldToViewportPoint=function(self,pos)if SilentA and SilentT and SilentT.Part and SilentT.Part.Parent then if typeof(pos)=="Vector3"then local d=(pos-SilentT.Part.Position).Magnitude if d<2 then return og(self,SilentT.Part.Position)end end end return og(self,pos)end end)hookInst=true end
task.spawn(function()task.wait(0.5)pcall(instHooks)end)
local function updSilent()if not S.UseSilentAim or not isAim()then SilentA=false SilentT=nil return end local mo=UI:GetMouseLocation()local cp=Cam.CFrame.Position local md=S.SilentAimFOV local b,bd for _,t in ipairs(getTargets("aim"))do local sp,on=Cam:WorldToViewportPoint(t.Root.Position)if on then local d=(Vector2.new(sp.X,sp.Y)-mo).Magnitude local wd=(t.Root.Position-cp).Magnitude if d<=md then if not S.SilentAimVisible or hasLOS(t.Root,t.Model)then if wd<bd or not bd then b,bd=t,wd end end end end end if b then local hd=b.Model:FindFirstChild("Head")local to=b.Model:FindFirstChild("UpperTorso")or b.Model:FindFirstChild("Torso")local pt=hd or to or b.Model:FindFirstChild("HumanoidRootPart")if pt then b.Part=pt SilentT=b SilentA=true return end end SilentT=nil SilentA=false end
RunService.RenderStepped:Connect(function()pcall(function()if not S.AimEnabled then SilentA=false SilentT=nil return end updSilent()end)end)
RunService:BindToRenderStep("TTAim",Enum.RenderPriority.Camera.Value+1,function(dt)if Spec then AimTgt=nil Lk.Visible=false return end local n=os.clock()local mr=rt()local fp=mr and(Cam.CFrame.Position-mr.Position).Magnitude<1.5 local ao=fp and Vector2.new(Cam.ViewportSize.X/2,Cam.ViewportSize.Y/2)or UI:GetMouseLocation()if S.UseLegitAim and isAim()then if AimTgt then local m,h=AimTgt.Model,AimTgt.Humanoid if not m.Parent or not isAlive(h)then AimTgt=nil elseif S.TeamCheck and isTeammate(m,AimTgt.Player)then AimTgt=nil end end if not AimTgt then AimTgt=getCands()[1]if AimTgt then AimTgt.LastSeen=n end end if AimTgt then local pt=AimTgt.Part if not pt or not pt.Parent then AimTgt=nil else local sp,on=Cam:WorldToViewportPoint(pt.Position)local tf=S.AimMaxDist>0 and(pt.Position-Cam.CFrame.Position).Magnitude>S.AimMaxDist*1.1 local of2=S.FOVEnabled and(not on or(Vector2.new(sp.X,sp.Y)-ao).Magnitude>S.FOVRadius*1.35)if tf or of2 then AimTgt=nil end end end if AimTgt then local gp=predict(AimTgt,dt)local cc=Cam.CFrame local cp=cc.Position local tg=gp-cp if tg.Magnitude>0.01 then local gd=tg.Unit local cd=cc.LookVector local ar=math.acos(math.clamp(cd:Dot(gd),-1,1))local an=math.deg(ar)local al=1-math.pow(S.Smoothness,dt*60)al=al+(1-al)*math.clamp(an/25,0,1)*0.5 if an<=S.SnapAngle or(Firing and S.FireLock)then al=1 end local ax=cd:Cross(gd)if ax.Magnitude>1e-4 then local rt3=CFrame.fromAxisAngle(ax.Unit,ar*al)Cam.CFrame=CFrame.new(cp)*rt3*cc.Rotation end end local sp,on=Cam:WorldToViewportPoint(gp)Lk.Visible=on if on then Lk.Position=UDim2.fromOffset(sp.X,sp.Y)Lk.BackgroundColor3=Color3.new(0,0,0)local sz=24+math.sin(os.clock()*9)*3 Lk.Size=UDim2.fromOffset(sz,sz)end end end if S.UseSilentAim and SilentT and SilentT.Part and SilentT.Part.Parent then local sp,on=Cam.WorldToViewportPoint(SilentT.Part.Position)if on then Lk.Visible=true Lk.Position=UDim2.fromOffset(sp.X,sp.Y)Lk.BackgroundColor3=Color3.fromRGB(255,100,255)local sz=20+math.sin(os.clock()*12)*2 Lk.Size=UDim2.fromOffset(sz,sz)end end if not AimTgt and not(SilentA and SilentT)then Lk.Visible=false end end)
-- Hitbox, Fly, Fling, AntiKill, AntiVoid, Extras
local defWS=16
local defJP=50
local defUJP=true
local wsA=false
local jpA=false
local lastSafe=nil
local antiKillHold=0
local function destroyFly()if flyObjs then for _,o in pairs(flyObjs)do if o and o.Parent then o:Destroy()end end flyObjs=nil end if flyPlat then local h=hm()if h then h.PlatformStand=false end flyPlat=false end end
local flyObjs=nil
local flyPlat=false
local function ensureFly(r)if flyObjs and flyObjs.A and flyObjs.A.Parent==r then return flyObjs end destroyFly()local a=Instance.new("Attachment")a.Parent=r local lv=Instance.new("LinearVelocity")lv.Attachment0=a lv.RelativeTo=Enum.ActuatorRelativeTo.World lv.VelocityConstraintMode=Enum.VelocityConstraintMode.Vector lv.MaxForce=math.huge lv.VectorVelocity=Vector3.zero lv.Parent=r local ao=Instance.new("AlignOrientation")ao.Mode=Enum.OrientationAlignmentMode.OneAttachment ao.Attachment0=a ao.RigidityEnabled=false ao.MaxTorque=math.huge ao.Responsiveness=40 ao.Parent=r flyObjs={A=a,LV=lv,AO=ao}return flyObjs end
local function updFly(h,r,dt)local f=ensureFly(r)if os.clock()<antiKillHold then f.LV.VectorVelocity=Vector3.zero return end if not h.PlatformStand then h.PlatformStand=true end flyPlat=true local c=W.CurrentCamera local lk=c.CFrame.LookVector local fl=Vector3.new(lk.X,0,lk.Z)if fl.Magnitude<0.01 then fl=Vector3.new(0,0,-1)end fl=fl.Unit local fr=Vector3.new(c.CFrame.RightVector.X,0,c.CFrame.RightVector.Z)if fr.Magnitude>0.01 then fr=fr.Unit end local mv=h.MoveDirection local fw=mv:Dot(fl)local sd=mv:Dot(fr)local dir=c.CFrame.LookVector*fw+c.CFrame.RightVector*sd local tp=UI:GetFocusedTextBox()~=nil local up=0 if not tp then if UI:IsKeyDown(Enum.KeyCode.Space)or h.Jump then up=up+1 end if UI:IsKeyDown(Enum.KeyCode.LeftControl)then up=up-1 end end dir=dir+Vector3.new(0,up,0)if dir.Magnitude>1 then dir=dir.Unit end f.LV.VectorVelocity=dir*S.FlySpeed f.AO.CFrame=CFrame.lookAt(Vector3.zero,fl)end
local wsCur=16
local lastWS=nil
local function walkGuard(h,r,dt)local p=r.Position local ls=lastWS lastWS=p if not S.WalkAutoLimit or not ls then return end local n=os.clock()if n<antiKillHold+0.5 then return end local mv=h.MoveDirection local fm=Vector3.new(mv.X,0,mv.Z)if fm.Magnitude<0.1 then return end fm=fm.Unit local dp=Vector3.new(p.X-ls.X,0,p.Z-ls.Z)local bk=-dp:Dot(fm)local lm=math.max(6,wsCur*dt*2.5)if bk>lm and wsCur>20 then local nw=math.max(16,math.floor(wsCur*0.7))wsCur=nw S.WalkSpeed=math.max(16,nw)notif("Velocidade reduzida para "..nw,"off")end end
-- Fling
local lastFling=0
local lastFlN=0
local flingAct=false
local savedCol={}
local function disBotCol(m)for _,p in ipairs(m:GetDescendants())do if p:IsA("BasePart")then if savedCol[p]==nil then savedCol[p]=p.CanCollide end p.CanCollide=false end end end
local function resAllCol()for p,v in pairs(savedCol)do if p.Parent then p.CanCollide=v end end table.clear(savedCol)end
local function doFling(tr,mr,pw,md)if not tr or not tr.Parent then return false end if not mr or not mr.Parent then return false end disBotCol(tr.Parent)local mp=mr.Position local tp=tr.Position local dir=Vector3.new(tp.X-mp.X,0,tp.Z-mp.Z)if dir.Magnitude<0.05 then dir=Vector3.new(0,0,1)else dir=dir.Unit end local lv,av if md==1 then lv=dir*pw*50+Vector3.new(0,pw*10,0)av=Vector3.new(pw*50,pw*50,pw*50)else lv=Vector3.new(9e7,9e7,9e7)av=Vector3.new(9e8,9e8,9e8)end pcall(function()tr.AssemblyLinearVelocity=lv tr.AssemblyAngularVelocity=av end)return true end
local function doFlPlayer(tg)local tr=rt(tg)local mr=rt()if not tr or not mr or busy()then return false end setBusy(true)local home=mr.CFrame local sp=Instance.new("BodyAngularVelocity")sp.MaxTorque=Vector3.one*math.huge sp.AngularVelocity=Vector3.new(0,90000,0)sp.Parent=mr local sn={}local c=LP.Character if c then for _,p in ipairs(c:GetChildren())do if p:IsA("BasePart")then sn[p]=p.CanCollide p.CanCollide=false end end end local st=os.clock()while os.clock()-st<2.5 do local ct=rt(tg)if not ct or not mr.Parent then break end mr.CFrame=ct.CFrame mr.AssemblyLinearVelocity=Vector3.zero RunService.Heartbeat:Wait()end sp:Destroy()if mr.Parent then mr.AssemblyAngularVelocity=Vector3.zero mr.AssemblyLinearVelocity=Vector3.zero mr.CFrame=home end for p,cc in pairs(sn)do if p.Parent then p.CanCollide=cc end end setBusy(false)return true end
local function pickFling()local mr=rt()if not mr then return{}end local mp=mr.Position local rg=S.FlingRange local ls={}if S.FlingMode==2 and AimTgt and AimTgt.Model then local m=AimTgt.Model if m.Parent and not P:GetPlayerFromCharacter(m)then local r=getRoot(m)if r then ls[1]={Model=m,Root=r,D=0}end end return ls end for m,h in pairs(Hums)do if m~=LP.Character and m.Parent and h.Parent and h.Health>0 then local pl=P:GetPlayerFromCharacter(m)if not pl and S.FlingBots then local r=getRoot(m)if r then local d=(r.Position-mp).Magnitude if d<=rg then ls[#ls+1]={Model=m,Root=r,D=d}end end end end end table.sort(ls,function(a,b)return a.D<b.D end)if S.FlingMode==3 and#ls>1 then ls={ls[1]}end return ls end
local function stepFling()if S.Fling then local n=os.clock()local jt=(math.random()-0.5)*0.04 if n-lastFling>=S.FlingRepeat+jt then lastFling=n local mh,mr=hm(),rt()if mh and mh.Health>0 then flingAct=true local tg=pickFling()if#tg==0 then if n-lastFlN>4 then lastFlN=n notif("Fling: nenhum BOT no alcance","off")end else for i=1,#tg do doFling(tg[i].Root,mr,S.FlingPower,S.FlingMode2 or 2)end end end end elseif flingAct then flingAct=false resAllCol()end if S.FlingPlayers and not MS.FlingBusy then local n=os.clock()if n-(MS.LastFlingPlayer or 0)>=0.5 then MS.LastFlingPlayer=n MS.FlingBusy=true task.spawn(function()local mr=rt()if mr then local tg={}local mp=mr.Position local md=S.FlingPlayerMode local rg=S.FlingPlayerRange if md==2 and AimTgt and AimTgt.Player then tg={AimTgt.Player}else for _,pl in ipairs(P:GetPlayers())do if pl~=LP then local r=rt(pl)if r and(r.Position-mp).Magnitude<=rg then tg[#tg+1]=pl end end end table.sort(tg,function(a,b)local ra,rb=rt(a),rt(b)return ra and rb and(ra.Position-mp).Magnitude<(rb.Position-mp).Magnitude end)if md==1 and#tg>1 then tg={tg[1]}end end for _,t in ipairs(tg)do doFlPlayer(t)end end MS.FlingBusy=false end)end end end end
-- Hitbox
local HitboxExpander={}
do local svP=setmetatable({},{__mode="k"})local HBG="TTHB"local PG="TTP"local gr=false pcall(function()PS:RegisterCollisionGroup(HBG)PS:RegisterCollisionGroup(PG)PS:CollisionGroupSetCollidable(HBG,HBG,false)PS:CollisionGroupSetCollidable(HBG,PG,false)PS:CollisionGroupSetCollidable(PG,PG,false)PS:CollisionGroupSetCollidable(PG,"Default",true)gr=true end)local function mark(p)if p and p:IsA("BasePart")and gr then pcall(function()p.CollisionGroup=PG end)end end local function bindC(c)for _,d in ipairs(c:GetDescendants())do mark(d)end c.DescendantAdded:Connect(function(d)if d:IsA("BasePart")then task.defer(mark,d)end end)end if LP.Character then bindC(LP.Character)end LP.CharacterAdded:Connect(function(c)task.wait(0.1)bindC(c)end)local PK={{ "Head","ExpandHead"},{"UpperTorso","ExpandUpperTorso"},{"Torso","ExpandTorso"},{"LowerTorso","ExpandLowerTorso"}}local function actP()local l={}for _,e in ipairs(PK)do if S[e[2]]then l[#l+1]=e[1]end end return l end local function saveP(p)if svP[p]then return end svP[p]={Size=p.Size,CanCollide=p.CanCollide,Massless=p.Massless,Transparency=p.Transparency,Color=p.Color,Material=p.Material,CollisionGroup=p.CollisionGroup}end local function resP(p)local pr=svP[p]if not pr then return end pcall(function()p.Size=pr.Size p.CanCollide=pr.CanCollide p.Massless=pr.Massless p.Transparency=pr.Transparency p.Color=pr.Color p.Material=pr.Material if gr then p.CollisionGroup=pr.CollisionGroup end end)svP[p]=nil end local function expP(pl)local c=pl.Character if not c then return end local h=c:FindFirstChildOfClass("Humanoid")if not h or h.Health<=0 then return end for _,nm in ipairs(actP())do local p=c:FindFirstChild(nm)if p and p:IsA("BasePart")then saveP(p)local sz=S.HitboxSize pcall(function()p.Size=Vector3.new(sz,sz,sz)p.CanCollide=false p.Massless=true if gr then p.CollisionGroup=HBG end p.Transparency=S.HitboxInvisible and 1 or 0.5 end)end end end local function resPl(pl)local c=pl.Character if not c then return end for _,e in ipairs(PK)do local p=c:FindFirstChild(e[1])if p and p:IsA("BasePart")then resP(p)end end end local function shouldE(pl)local c=pl.Character if not c then return false end local h=c:FindFirstChildOfClass("Humanoid")if not h or h.Health<=0 then return false end local mr=rt()local tr=getRoot(c)if not mr or not tr then return false end local d=(tr.Position-mr.Position).Magnitude if d>math.max(10,S.HitboxRange or 200)then return false end if S.HitboxWallCheck then local hd=c:FindFirstChild("Head")if hd and not hasLOS(hd,c)then if not hasLOS(tr,c)then return false end end end return true end local la=0 local function step()if not S.HitboxExpander then for _,pl in ipairs(P:GetPlayers())do if pl~=LP then resPl(pl)end end return end local n=os.clock()if n-la<0.1 then return end la=n for _,pl in ipairs(P:GetPlayers())do if pl~=LP then if shouldE(pl)then expP(pl)else resPl(pl)end end end end HitboxExpander.step=step HitboxExpander.restoreAll=function()for _,pl in ipairs(P:GetPlayers())do if pl~=LP then resPl(pl)end end end end
task.spawn(function()while task.wait(0.1)do pcall(HitboxExpander.step)end end)
LP.CharacterAdded:Connect(function()task.wait(0.5)pcall(HitboxExpander.restoreAll)end)
-- Invisibilidade Seat
local SeatInvis={}
do local myS=nil local act=false local function clean()local e=W:FindFirstChild("invischair")if e then pcall(function()e:Destroy()end)end myS=nil end local function actv()local c=LP.Character if not c then return end local hrp=c:FindFirstChild("HumanoidRootPart")if not hrp then return end clean()local sp=hrp.CFrame local tp=Vector3.new(S.SeatInvisibleX,S.SeatInvisibleY,S.SeatInvisibleZ)c:MoveTo(tp)task.wait(0.15)local st=Instance.new("Seat")st.Name="invischair"st.Anchored=false st.CanCollide=false st.Transparency=1 st.Position=tp st.Parent=W myS=st local wl=Instance.new("Weld")wl.Part0=st wl.Part1=c:FindFirstChild("Torso")or c:FindFirstChild("UpperTorso")wl.Parent=st task.wait()st.CFrame=sp for _,d in ipairs(c:GetDescendants())do if d:IsA("BasePart")or d:IsA("Decal")then d.Transparency=0.5 end end end local function deact()clean()if LP.Character then for _,d in ipairs(LP.Character:GetDescendants())do if d:IsA("BasePart")or d:IsA("Decal")then d.Transparency=0 end end end end function SeatInvis.toggle()act=not act S.SeatInvisible=act if act then actv()else deact()end if _G.__uiRefresh then pcall(_G.__uiRefresh)end end function SeatInvis.findSafeCoords()local b=Vector3.new(-25.95,84,3537.55)local mr=rt()if not mr then return b end pcall(function()local xs,zs={},{}local co=0 for _,d in ipairs(W:GetDescendants())do if d:IsA("BasePart")and d.Anchored then xs[#xs+1]=d.Position.X zs[#zs+1]=d.Position.Z co=co+1 if co>=500 then break end end end if#xs>=10 then table.sort(xs)table.sort(zs)b=Vector3.new(xs[#xs]+5000,500,zs[1]-5000)end end)return b end task.spawn(function()while task.wait(0.5)do if act and(not myS or not myS.Parent)then act=false S.SeatInvisible=false deact()if _G.__uiRefresh then pcall(_G.__uiRefresh)end end end end)task.spawn(function()while task.wait(0.3)do if act then local c=LP.Character if c then for _,d in ipairs(c:GetDescendants())do if d:IsA("BasePart")or d:IsA("Decal")then if d.Transparency~=0.5 and d.Transparency~=1 then d.Transparency=0.5 end end end end end end end)LP.CharacterAdded:Connect(function()act=false S.SeatInvisible=false clean()end)end
_G.TT_SeatInvis=SeatInvis
-- Heartbeat principal
RunService.Heartbeat:Connect(function(dt)pcall(function()local h=hm()if not h then return end local r=rt()if S.WalkSpeedOn then if not wsA then defWS=h.WalkSpeed wsCur=h.WalkSpeed wsA=true lastWS=nil end if wsCur<S.WalkSpeed then wsCur=math.min(S.WalkSpeed,wsCur+500*dt)else wsCur=S.WalkSpeed end if h.WalkSpeed~=wsCur then h.WalkSpeed=wsCur end if r and h.Health>0 then walkGuard(h,r,dt)end elseif wsA then h.WalkSpeed=defWS wsA=false lastWS=nil end if S.JumpOn then if not jpA then defUJP=h.UseJumpPower defJP=h.JumpPower jpA=true end if not h.UseJumpPower then h.UseJumpPower=true end if h.JumpPower~=S.JumpPower then h.JumpPower=S.JumpPower end elseif jpA then h.UseJumpPower=defUJP h.JumpPower=defJP jpA=false end if S.Fly and r and h.Health>0 then updFly(h,r,dt)elseif flyObjs or flyPlat then destroyFly()end if S.AntiFling and r and h.Health>0 then local v=r.AssemblyLinearVelocity local hr=Vector3.new(v.X,0,v.Z).Magnitude local lm=S.AntiFlingSpeed if S.Fly then lm=math.max(lm,S.FlySpeed*2.5)end if S.WalkSpeedOn then lm=math.max(lm,S.WalkSpeed*3)end local fo=S.WalkSpeedOn or S.Fly local sl=fo and 120 or 40 local yl=math.max(lm,S.JumpOn and S.JumpPower*3 or 0)local fl=hr>lm or v.Y>yl or r.AssemblyAngularVelocity.Magnitude>sl if fl then if S.AntiFlingRestore and stableOld and not fo then LP.Character:PivotTo(stableOld)end r.AssemblyLinearVelocity=Vector3.zero r.AssemblyAngularVelocity=Vector3.zero local st=h:GetState()if st==Enum.HumanoidStateType.FallingDown or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.Physics then if not S.Fly then h:ChangeState(Enum.HumanoidStateType.GettingUp)end end if not S.Fly then h.PlatformStand=false end else if not stableNew then stableNew=r.CFrame elseif os.clock()-lastStT>0.25 then lastStT=os.clock()stableOld=stableNew stableNew=r.CFrame end end end if r and h.Health>0 then if r.Position.Y<S.AntiVoidY then if S.AntiVoid then local tg=lastSafe if not tg then local sp=W:FindFirstChildWhichIsA("SpawnLocation",true)tg=sp and(sp.CFrame+Vector3.new(0,5,0))or CFrame.new(0,50,0)end LP.Character:PivotTo(tg)r.AssemblyLinearVelocity=Vector3.zero end elseif h.FloorMaterial~=Enum.Material.Air then lastSafe=r.CFrame+Vector3.new(0,3,0)end end stepFling()if IS_MM2 then pcall(refreshRoles)pcall(Sh.AutoShoot)pcall(Sh.AutoGrab)pcall(Mu.AutoKill)pcall(Mu.KillAura)pcall(Kaitun)pcall(Dodge)pcall(Fa.Step)end end)end)
local stableNew=nil
local stableOld=nil
local lastStT=0
-- AntiKill
do local bnd=nil local sct=false local hist={}local ht,lastR,lastN,lastF,lastO=0,0,0,0,0 local rpC=nil local DN=Vector3.new(0,-5000,0)local rp=RaycastParams.new()rp.FilterType=Enum.RaycastFilterType.Exclude rp.RespectCanCollide=true local rp2=RaycastParams.new()rp2.FilterType=Enum.RaycastFilterType.Exclude local BAD={"kill","lava","death","void","acid","damage","lethal","magma","spike","toxic"}local function isBad(i)local n=string.lower(i.Name)for _,w in ipairs(BAD)do if string.find(n,w,1,true)then return true end end return false end local function pct(a,p)return a[math.clamp(math.floor(#a*p)+1,1,#a)]end local function scan()if sct then return end sct=true task.spawn(function()local xs,zs,n={},{},0 for _,d in ipairs(W:GetDescendants())do if d:IsA("BasePart")and d~=W.Terrain and d.Anchored and d.CanCollide and d.Size.Magnitude<2000 and not isBad(d)then xs[#xs+1]=d.Position.X zs[#zs+1]=d.Position.Z end n=n+1 if n%400==0 then task.wait()end end if#xs>=10 then table.sort(xs)table.sort(zs)bnd={minX=pct(xs,0.01)-60,maxX=pct(xs,0.99)+60,minZ=pct(zs,0.01)-60,maxZ=pct(zs,0.99)+60}end sct=false end)end local function inB(p,m)return p.X>=bnd.minX-m and p.X<=bnd.maxX+m and p.Z>=bnd.minZ-m and p.Z<=bnd.maxZ+m end local function noG(p)return W:Raycast(p,DN,rp)==nil and W:Raycast(p,DN,rp2)==nil end local function safe()local n=os.clock()local ix for i=#hist,1,-1 do if n-hist[i].t>=1 then ix=i break end end ix=ix or(#hist>0 and 1 or nil)if ix then local cf=hist[ix].cf for i=#hist,ix+1,-1 do hist[i]=nil end return cf end local sp=W:FindFirstChildWhichIsA("SpawnLocation",true)return sp and(sp.CFrame+Vector3.new(0,5,0))or CFrame.new(0,50,0)end local function resc(r,wy)local n=os.clock()lastR=n antiKillHold=n+0.5 LP.Character:PivotTo(safe())r.AssemblyLinearVelocity=Vector3.zero r.AssemblyAngularVelocity=Vector3.zero if flyObjs and flyObjs.LV then flyObjs.LV.VectorVelocity=Vector3.zero end if n-lastN>2 then lastN=n notif("AntiKill: trazido de volta","info")end end LP.CharacterAdded:Connect(function()table.clear(hist)rpC=nil end)RunService.Heartbeat:Connect(function()pcall(function()if not S.AntiKill then if next(hist)then table.clear(hist)end return end local h,r=hm(),rt()if not h or not r or h.Health<=0 then return end local c=LP.Character if rpC~=c then rpC=c rp.FilterDescendantsInstances={c}rp2.FilterDescendantsInstances={c}end if not bnd and not sct then scan()end local n=os.clock()local p,v=r.Position,r.AssemblyLinearVelocity local air=h.FloorMaterial==Enum.Material.Air local dg,wy=false,nil local dy=W.FallenPartsDestroyHeight if p.Y<dy+80 then dg,wy=true,"void"elseif v.Y<-40 and p.Y+v.Y*0.4<dy+80 and noG(p)then dg,wy=true,"falling"end if not dg and not S.Fly then if air and n>=lastF and(v.Y<-40 or(S.Noclip and v.Y<-12))then lastF=n+0.05 if noG(p)then dg,wy=true,"no ground"end end local m=S.AntiKillMargin if not dg and bnd and m>0 and n>=lastO then lastO=n+0.1 local ft=p+Vector3.new(v.X,0,v.Z)*0.25 if not inB(ft,m)and noG(ft)and noG(p)then dg,wy=true,"edge"end end end if dg then if n-lastR>=0.1 then resc(r,wy)end return end if not air and n-ht>=0.4 and v.Y>-30 then ht=n local hit=W:Raycast(p,Vector3.new(0,-12,0),rp)if hit and not isBad(hit.Instance)and(not bnd or inB(p,0))then hist[#hist+1]={cf=r.CFrame+Vector3.new(0,3,0),t=n}if#hist>10 then table.remove(hist,1)end end end end)end)end
-- Extras
UI.JumpRequest:Connect(function()if S.InfJump and not S.Fly then local h=hm()if h and h.Health>0 then h:ChangeState(Enum.HumanoidStateType.Jumping)end end end)
LP.Idled:Connect(function()if not S.AntiAFK then return end pcall(function()VU:CaptureController()VU:ClickButton2(Vector2.new())end)end)
LP.CharacterAdded:Connect(function()Spec=false end)
local camFovDef=nil
-- HUD
local Hud=Instance.new("TextLabel")Hud.Position=UDim2.fromOffset(10,10)Hud.Size=UDim2.fromOffset(700,20)Hud.BackgroundColor3=Th.Bg Hud.BackgroundTransparency=0.25 Hud.BorderSizePixel=0 Hud.Font=Enum.Font.GothamMedium Hud.TextSize=12 Hud.TextColor3=Th.Text Hud.TextXAlignment=Enum.TextXAlignment.Left Hud.ZIndex=30 Hud.Parent=Gui cn(Hud,6)local HP=Instance.new("UIPadding")HP.PaddingLeft=UDim.new(0,8)HP.Parent=Hud Hud.Active=true
local HDr,HDs,HDf=false,nil,nil Hud.InputBegan:Connect(function(i)if S.HUDEdit and(i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch)then HDr=true HDs=i.Position HDf=Vector2.new(S.HUDX,S.HUDY)end end)UI.InputChanged:Connect(function(i)if HDr and(i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch)then local d=i.Position-HDs local vp=Cam.ViewportSize S.HUDX=math.clamp(HDf.X+d.X,0,math.max(vp.X-Hud.AbsoluteSize.X,0))S.HUDY=math.clamp(HDf.Y+d.Y,0,math.max(vp.Y-Hud.AbsoluteSize.Y,0))end end)UI.InputEnded:Connect(function(i)if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then HDr=false end end)
local fpsF,fpsA,HudA,BtnA=0,0,0,0
local lastFps=60
RunService:BindToRenderStep("TTVis",Enum.RenderPriority.Camera.Value+2,function(dt)local n=os.clock()fpsF=fpsF+1 fpsA=fpsA+dt if fpsA>=0.5 then lastFps=fpsF/fpsA fpsF,fpsA=0,0 end local sF=S.FOVEnabled and S.AimEnabled and not Spec and S.UseLegitAim FovC.Visible=sF if sF then local o=UI:GetMouseLocation()FovC.Position=UDim2.fromOffset(o.X,o.Y)FovC.Size=UDim2.fromOffset(S.FOVRadius*2,S.FOVRadius*2)end local sSil=S.AimEnabled and not Spec and S.UseSilentAim SilC.Visible=sSil if sSil then local o=UI:GetMouseLocation()SilC.Position=UDim2.fromOffset(o.X,o.Y)SilC.Size=UDim2.fromOffset(S.SilentAimFOV*2,S.SilentAimFOV*2)end local sTg=S.TriggerBot and not Spec TgbC.Visible=sTg if sTg then local o=UI:GetMouseLocation()TgbC.Position=UDim2.fromOffset(o.X,o.Y)TgbC.Size=UDim2.fromOffset(math.max(S.TriggerFOV*2,4),math.max(S.TriggerFOV*2,4))end if S.CamFOVOn then if not camFovDef then camFovDef=Cam.FieldOfView end if Cam.FieldOfView~=S.CamFOV then Cam.FieldOfView=S.CamFOV end elseif camFovDef then Cam.FieldOfView=camFovDef camFovDef=nil end updEsp(n)BtnA=BtnA+dt if BtnA>=0.1 then BtnA=0 end Hud.Visible=S.ShowHUD or S.HUDEdit Hud.Position=UDim2.fromOffset(S.HUDX,S.HUDY)Hud.BackgroundColor3=S.HUDEdit and Th.Accent or Th.Bg HudA=HudA+dt if(S.ShowHUD or S.HUDEdit)and HudA>=0.25 then HudA=0 local parts={math.floor(lastFps).." FPS"}if IS_MM2 and S.MM2Mode then local mr=roleOf(LP)local sh="?"if mr=="Murderer"then sh="MURD"elseif mr=="Sheriff"then sh="SHER"elseif mr=="Hero"then sh="HERO"elseif mr=="Innocent"then sh="INOC"end parts[#parts+1]="MM2: "..sh if MS.Bag.Max>0 then parts[#parts+1]=string.format("Bag:%d/%d",MS.Bag.Current,MS.Bag.Max)end end if S.Fly then parts[#parts+1]="Fly"end if S.Noclip then parts[#parts+1]="Noclip"end if S.Fling then parts[#parts+1]="Fling"end if S.HitboxExpander then parts[#parts+1]="Hitbox"end if S.SeatInvisible then parts[#parts+1]="SeatInv"end Hud.Text=table.concat(parts,"  •  ")end end)
-- Mobile buttons
local MB={}
local MDr,MSt,MFrom=nil,nil,nil
local function mkMB(lb,ps,sk,st,od,ou)local b=Instance.new("TextButton")b.AnchorPoint=Vector2.new(0.5,0.5)b.Position=ps b.Size=UDim2.fromOffset(56,56)b.BackgroundColor3=Th.Bg b.BorderSizePixel=0 b.AutoButtonColor=false b.Font=Enum.Font.GothamBold b.TextSize=11 b.TextColor3=Th.Text b.Text=lb b.Visible=false b.ZIndex=40 b.Parent=Gui cn(b,9999)str(b,Th.Accent,0.2,1.5)b.InputBegan:Connect(function(i)if i.UserInputType~=Enum.UserInputType.Touch and i.UserInputType~=Enum.UserInputType.MouseButton1 then return end if S.MobileEdit then MDr=b MSt=i.Position MFrom=b.AbsolutePosition+b.AbsoluteSize/2 elseif od then od()end end)b.InputEnded:Connect(function(i)if(i.UserInputType==Enum.UserInputType.Touch or i.UserInputType==Enum.UserInputType.MouseButton1)and ou and not S.MobileEdit then ou()end end)MB[#MB+1]={B=b,SK=sk,State=st}end
UI.InputChanged:Connect(function(i)if MDr and(i.UserInputType==Enum.UserInputType.Touch or i.UserInputType==Enum.UserInputType.MouseMovement)then local d=i.Position-MSt MDr.Position=UDim2.fromOffset(MFrom.X+d.X,MFrom.Y+d.Y)end end)UI.InputEnded:Connect(function(i)if i.UserInputType==Enum.UserInputType.Touch or i.UserInputType==Enum.UserInputType.MouseButton1 then MDr=nil end end)
mkMB("TT",UDim2.fromScale(0.07,0.2),nil,function()return _G.__menuOpen end,function()if _G.__togMenu then _G.__togMenu()end end)
mkMB("MIRA",UDim2.fromScale(0.9,0.42),"ShowBtnAim",function()return AimAct end,function()if S.AimMode==1 then AimAct=true else AimAct=not AimAct end end,function()if S.AimMode==1 then AimAct=false end end)
mkMB("VOO",UDim2.fromScale(0.9,0.54),"ShowBtnFly",function()return S.Fly end,function()S.Fly=not S.Fly end)
mkMB("NOCLIP",UDim2.fromScale(0.9,0.66),"ShowBtnNoclip",function()return S.Noclip end,function()S.Noclip=not S.Noclip end)
mkMB("DESCER",UDim2.fromScale(0.8,0.54),"ShowBtnDown",function()return false end,nil,nil)
mkMB("ESP",UDim2.fromScale(0.8,0.66),"ShowBtnEsp",function()return S.ESP end,function()S.ESP=not S.ESP end)
mkMB("LINHA",UDim2.fromScale(0.8,0.42),"ShowBtnTrace",function()return S.Tracers end,function()S.Tracers=not S.Tracers end)
mkMB("PULO",UDim2.fromScale(0.9,0.78),"ShowBtnInf",function()return S.InfJump end,function()S.InfJump=not S.InfJump end)
mkMB("FLING",UDim2.fromScale(0.8,0.78),"ShowBtnFling",function()return S.Fling end,function()S.Fling=not S.Fling end)
mkMB("GRAB",UDim2.fromScale(0.7,0.42),"ShowBtnGrabGun",function()return false end,function()task.spawn(function()Sh.Grab()end)end)
local MShow=false
local function updMB()if not UI.TouchEnabled or UI.KeyboardEnabled then if MShow then MShow=false for _,m in ipairs(MB)do m.B.Visible=false end end return end MShow=true for _,m in ipairs(MB)do local sh2=m.SK==nil or S[m.SK]m.B.Visible=sh2 if sh2 then local sz=S.MobileBtnSize if m.B.Size.X.Offset~=sz then m.B.Size=UDim2.fromOffset(sz,sz)end m.B.BackgroundTransparency=S.MobileBtnAlpha m.B.BackgroundColor3=m.State()and Th.Accent or Th.Bg end end end
RunService.RenderStepped:Connect(function()if os.clock()-(_G.__mba or 0)>0.1 then _G.__mba=os.clock()pcall(updMB)end end)
-- Teclas
local Reb=nil
local function matchB(i,b)if typeof(b)~="EnumItem"then return false end if b.EnumType==Enum.KeyCode then return i.KeyCode==b end return i.UserInputType==b end
UI.InputBegan:Connect(function(i)if Reb then local r=Reb if i.KeyCode==Enum.KeyCode.Escape then Reb=nil r.apply(nil)return end local bd=nil if i.KeyCode~=Enum.KeyCode.Unknown then bd=i.KeyCode elseif r.mouse and(i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.MouseButton2)then bd=i.UserInputType end if bd then Reb=nil r.apply(bd)end return end if UI:GetFocusedTextBox()then return end if i.KeyCode==MK then if _G.__togMenu then _G.__togMenu()end return end if matchB(i,S.AimKey)then if S.AimMode==1 then AimAct=true elseif S.AimMode==2 then AimAct=not AimAct notif(AimAct and"Mira on"or"Mira off",AimAct and"on"or"off")end elseif matchB(i,S.SwitchKey)then if isAim()then local c=getCands(true)if#c>0 then AimTgt=c[2]or c[1]end end elseif matchB(i,S.NoclipKey)then S.Noclip=not S.Noclip elseif matchB(i,S.FlyKey)then S.Fly=not S.Fly elseif i.KeyCode==S.FlingKey then S.Fling=not S.Fling elseif i.KeyCode==Enum.KeyCode.H and IS_MM2 then task.spawn(function()Sh.Grab()end)end end)
UI.InputEnded:Connect(function(i)if S.AimMode==1 and matchB(i,S.AimKey)then AimAct=false end end)
_G.__menuOpen=false
local origSetMenu=setMenu
local function wrappedSetMenu(o)_G.__menuOpen=o origSetMenu(o)end
_G.__togMenu=function()wrappedSetMenu(not _G.__menuOpen)end
_G.__uiRefresh=function()for _,r in ipairs(Ref)do r()end end
print("[TestToolkit] v66 carregado. IS_MM2="..tostring(IS_MM2))
