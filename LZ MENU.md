-- LZ MENU EN | Misc Optimize moderado
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
pcall(function() RunService:UnbindFromRenderStep("LZ_Aim") end)

local FIXED_KEY = "K7X9-MQ2P-V8RN-4TLC-Z6WF"
local KEY_LINK = "https://link-target.net/9250743/UXxvFUtKAfId"
local Unlocked = false
local Toggles = {ESPEnemies=false, ShowNames=false, AimAssist=false, ShowFOV=true, ShowFPS=false, VisibleCheck=true, DeadCheck=true, TeamCheck=true, InfJump=false, Noclip=false, Fly=false, Optimize=false}
local FOV = 300
local FOV_MIN, FOV_MAX = 50, 1000
local WalkSpeed = 16
local JumpPower = 50
local FlySpeed = 50
local ESPColor = Color3.fromRGB(255,0,0)
local ESPTrans = 30
local ACCENT = Color3.fromRGB(0,140,255)
local ACCENT_DARK = Color3.fromRGB(12,32,68)
local ToggleUI = {}
local Configs = {}
local SelectedConfig = nil
local AutoloadConfig = nil
local CONFIG_FILE = "LZ_Configs.json"
local SavedOpt = nil
local function tween(o,p,t) TweenService:Create(o,TweenInfo.new(t or 0.18,Enum.EasingStyle.Quad,Enum.EasingDirection.Out),p):Play() end
local function norm(s)
  s = tostring(s or ""):lower()
  s = s:gsub("[áàâã]","a"):gsub("[éèê]","e"):gsub("[íìî]","i"):gsub("[óòôõ]","o"):gsub("[úùû]","u"):gsub("ç","c")
  return s:gsub("%s+","")
end
local function getRole(model, plr)
  if plr and plr.Team then return tostring(plr.Team.Name) end
  return ""
end
local function isSameRole(m)
  local plr = Players:GetPlayerFromCharacter(m)
  local a = getRole(LocalPlayer.Character, LocalPlayer)
  local b = getRole(m, plr)
  if a ~= "" and b ~= "" then return norm(a) == norm(b) end
  if plr and LocalPlayer.Team and plr.Team then return plr.Team == LocalPlayer.Team end
  return false
end
local function isAlive(m)
  local h = m:FindFirstChildOfClass("Humanoid")
  if not h or h.Health <= 0 then return false end
  return true
end
local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
rayParams.IgnoreWater = true
local function isVisible(cam, part, model)
  local o = cam.CFrame.Position
  local d = part.Position - o
  if d.Magnitude < 0.1 then return true end
  rayParams.FilterDescendantsInstances = {LocalPlayer.Character, model}
  local r = workspace:Raycast(o, d, rayParams)
  if not r then return true end
  if r.Distance >= d.Magnitude - 1 then return true end
  if r.Instance and model:IsAncestorOf(r.Instance) then return true end
  return false
end
local function getAimPart(c)
  if not c then return nil end
  local h = c:FindFirstChildOfClass("Humanoid")
  if Toggles.DeadCheck and h and h.Health <= 0 then return nil end
  return c:FindFirstChild("Head") or c:FindFirstChild("HumanoidRootPart") or c:FindFirstChildWhichIsA("BasePart", true)
end
local function getESPPart(c)
  if not c then return nil end
  return c:FindFirstChild("Head") or c:FindFirstChild("HumanoidRootPart") or c:FindFirstChildWhichIsA("BasePart", true)
end
local function getAllTargets()
  local t = {}
  for _,p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer and p.Character and getAimPart(p.Character) then table.insert(t, p.Character) end
  end
  return t
end
local function getAllESPTargets()
  local t = {}
  for _,p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer and p.Character and getESPPart(p.Character) then table.insert(t, p.Character) end
  end
  return t
end
local function myHum()
  local c = LocalPlayer.Character
  if not c then return nil end
  return c:FindFirstChildOfClass("Humanoid")
end
local function myHRP()
  local c = LocalPlayer.Character
  if not c then return nil end
  return c:FindFirstChild("HumanoidRootPart")
end
local function setCollision(noclipOn)
  local c = LocalPlayer.Character
  if not c then return end
  for _,p in pairs(c:GetDescendants()) do
    if p:IsA("BasePart") then
      if noclipOn then p.CanCollide = false
      else if p.Name == "HumanoidRootPart" then p.CanCollide = false else p.CanCollide = true end end
    end
  end
end
local function ensureName(c)
  local head = c:FindFirstChild("Head")
  if not head then return end
  local bb = head:FindFirstChild("DevName")
  if bb then return bb end
  bb = Instance.new("BillboardGui", head)
  bb.Name = "DevName"
  bb.Size = UDim2.new(0,120,0,36)
  bb.StudsOffset = Vector3.new(0,2.2,0)
  bb.AlwaysOnTop = true
  local tl = Instance.new("TextLabel", bb)
  tl.Size = UDim2.new(1,0,1,0)
  tl.BackgroundTransparency = 1
  tl.Font = Enum.Font.GothamBold
  tl.TextSize = 13
  tl.TextColor3 = Color3.new(1,1,1)
  tl.TextStrokeTransparency = 0
  return bb
end
local function setOptimize(on)
  if on then
    if SavedOpt then return end
    SavedOpt = {effects={}, parts={}, terrain={}, lighting={}}
    for _,v in pairs(Lighting:GetChildren()) do
      if v:IsA("BloomEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("BlurEffect") then
        SavedOpt.effects[v] = v.Enabled
        v.Enabled = false
      elseif v:IsA("Atmosphere") then
        SavedOpt.effects[v] = {Density=v.Density, Haze=v.Haze, Glare=v.Glare}
        v.Density = math.min(v.Density, 0.3)
        v.Haze = math.min(v.Haze, 2)
        v.Glare = math.min(v.Glare, 2)
      end
    end
    SavedOpt.lighting.Shadows = Lighting.GlobalShadows
    local t = workspace:FindFirstChildOfClass("Terrain")
    if t then
      SavedOpt.terrain.WaterWaveSize = t.WaterWaveSize
      SavedOpt.terrain.WaterWaveSpeed = t.WaterWaveSpeed
      SavedOpt.terrain.WaterReflectance = t.WaterReflectance
      SavedOpt.terrain.Decoration = t.Decoration
      t.WaterWaveSize = 0
      t.WaterWaveSpeed = 0
      t.WaterReflectance = 0
      if t.Decoration then
        pcall(function() t.Decoration = false end)
      end
    end
    for _,d in pairs(workspace:GetDescendants()) do
      if d:IsA("ParticleEmitter") then
        if d.Rate > 25 then
          SavedOpt.parts[d] = d.Rate
          d.Rate = 25
        end
      elseif d:IsA("Trail") then
        if d.Enabled and d.Lifetime > 0.5 then
          SavedOpt.parts[d] = d.Lifetime
          d.Lifetime = 0.5
        end
      elseif d:IsA("BasePart") and not Players:GetPlayerFromCharacter(d:FindFirstAncestorOfClass("Model")) then
        if d.Size.Magnitude < 4 and d.CastShadow then
          SavedOpt.parts[d] = "shadow"
          d.CastShadow = false
        end
      end
    end
    pcall(function()
      UserSettings():GetService("UserGameSettings").SavedQualityLevel = Enum.SavedQualitySetting.QualityLevel10
    end)
  else
    if not SavedOpt then return end
    for obj,v in pairs(SavedOpt.effects) do
      if obj and obj.Parent then
        if typeof(v) == "boolean" then pcall(function() obj.Enabled = v end)
        elseif typeof(v) == "table" then pcall(function() obj.Density = v.Density obj.Haze = v.Haze obj.Glare = v.Glare end) end
      end
    end
    if SavedOpt.lighting.Shadows ~= nil then pcall(function() Lighting.GlobalShadows = SavedOpt.lighting.Shadows end) end
    local t = workspace:FindFirstChildOfClass("Terrain")
    if t then
      pcall(function()
        if SavedOpt.terrain.WaterWaveSize then t.WaterWaveSize = SavedOpt.terrain.WaterWaveSize end
        if SavedOpt.terrain.WaterWaveSpeed then t.WaterWaveSpeed = SavedOpt.terrain.WaterWaveSpeed end
        if SavedOpt.terrain.WaterReflectance then t.WaterReflectance = SavedOpt.terrain.WaterReflectance end
        if SavedOpt.terrain.Decoration ~= nil then t.Decoration = SavedOpt.terrain.Decoration end
      end)
    end
    for obj,v in pairs(SavedOpt.parts) do
      if obj and obj.Parent then
        if obj:IsA("ParticleEmitter") and typeof(v) == "number" then pcall(function() obj.Rate = v end)
        elseif obj:IsA("Trail") and typeof(v) == "number" then pcall(function() obj.Lifetime = v end)
        elseif obj:IsA("BasePart") and v == "shadow" then pcall(function() obj.CastShadow = true end) end
      end
    end
    SavedOpt = nil
  end
end
local function snapshot()
  return {Toggles={ESPEnemies=Toggles.ESPEnemies,ShowNames=Toggles.ShowNames,AimAssist=Toggles.AimAssist,ShowFOV=Toggles.ShowFOV,ShowFPS=Toggles.ShowFPS,VisibleCheck=Toggles.VisibleCheck,DeadCheck=Toggles.DeadCheck,TeamCheck=Toggles.TeamCheck,InfJump=Toggles.InfJump,Noclip=Toggles.Noclip,Fly=Toggles.Fly,Optimize=Toggles.Optimize}, FOV=FOV, WalkSpeed=WalkSpeed, JumpPower=JumpPower, FlySpeed=FlySpeed, ESPColor={ESPColor.R,ESPColor.G,ESPColor.B}, ESPTrans=ESPTrans}
end
local function saveFile()
  pcall(function() if writefile then writefile(CONFIG_FILE, HttpService:JSONEncode({configs=Configs, autoload=AutoloadConfig})) end end)
end
local function loadFile()
  pcall(function() if readfile and isfile and isfile(CONFIG_FILE) then local d = HttpService:JSONDecode(readfile(CONFIG_FILE)) if d.configs then Configs = d.configs end if d.autoload then AutoloadConfig = d.autoload end end end)
end
loadFile()

local gui = Instance.new("ScreenGui")
gui.Name = "LZMENU"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
local circle = Instance.new("Frame")
circle.AnchorPoint = Vector2.new(0.5,0.5)
circle.Position = UDim2.new(0.5,0,0.5,0)
circle.Size = UDim2.fromOffset(FOV*2, FOV*2)
circle.BackgroundTransparency = 1
circle.Visible = false
circle.Parent = gui
Instance.new("UICorner", circle).CornerRadius = UDim.new(1,0)
local cs = Instance.new("UIStroke", circle)
cs.Color = ACCENT
local fpsLabel = Instance.new("TextLabel", gui)
fpsLabel.Size = UDim2.new(0,100,0,28)
fpsLabel.Position = UDim2.new(1,-110,0,10)
fpsLabel.BackgroundColor3 = Color3.fromRGB(0,0,0)
fpsLabel.TextColor3 = ACCENT
fpsLabel.Font = Enum.Font.GothamBold
fpsLabel.TextSize = 13
fpsLabel.Text = "FPS: --"
fpsLabel.Visible = false
Instance.new("UICorner", fpsLabel).CornerRadius = UDim.new(0,8)
local main = Instance.new("Frame")
main.Size = UDim2.new(0,440,0,560)
main.Position = UDim2.new(0,20,0,20)
main.BackgroundColor3 = Color3.fromRGB(0,0,0)
main.Active = true
main.Parent = gui
main.Visible = false
Instance.new("UICorner", main).CornerRadius = UDim.new(0,12)
local ms = Instance.new("UIStroke", main)
ms.Color = ACCENT
local scale = Instance.new("UIScale", main)
scale.Scale = 0.85
local top = Instance.new("Frame", main)
top.Size = UDim2.new(1,0,0,52)
top.BackgroundColor3 = Color3.fromRGB(8,8,12)
top.Active = true
Instance.new("UICorner", top).CornerRadius = UDim.new(0,12)
local title = Instance.new("TextLabel", top)
title.Size = UDim2.new(1,-60,0,22)
title.Position = UDim2.new(0,14,0,6)
title.BackgroundTransparency = 1
title.TextXAlignment = Enum.TextXAlignment.Left
title.Text = "LZ MENU"
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextColor3 = ACCENT
local sub = Instance.new("TextLabel", top)
sub.Size = UDim2.new(1,-60,0,16)
sub.Position = UDim2.new(0,14,0,27)
sub.BackgroundTransparency = 1
sub.TextXAlignment = Enum.TextXAlignment.Left
sub.Text = "Developed by Yuri"
sub.Font = Enum.Font.Gotham
sub.TextSize = 12
sub.TextColor3 = Color3.fromRGB(120,170,220)
local minBtn = Instance.new("TextButton", top)
minBtn.Size = UDim2.new(0,32,0,32)
minBtn.Position = UDim2.new(1,-40,0,10)
minBtn.Text = "-"
minBtn.Font = Enum.Font.GothamBold
minBtn.BackgroundColor3 = Color3.fromRGB(15,25,45)
minBtn.TextColor3 = ACCENT
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,8)
do
  local dragging = false
  local ds
  local sp
  top.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true ds = i.Position sp = main.Position end end)
  UserInputService.InputChanged:Connect(function(i) if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then local d = i.Position - ds main.Position = UDim2.new(sp.X.Scale, sp.X.Offset+d.X, sp.Y.Scale, sp.Y.Offset+d.Y) end end)
  UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
end
local side = Instance.new("Frame", main)
side.Size = UDim2.new(0,120,1,-64)
side.Position = UDim2.new(0,8,0,58)
side.BackgroundColor3 = Color3.fromRGB(5,5,9)
Instance.new("UICorner", side).CornerRadius = UDim.new(0,10)
local sList = Instance.new("UIListLayout", side)
sList.Padding = UDim.new(0,6)
sList.HorizontalAlignment = Enum.HorizontalAlignment.Center
sList.VerticalAlignment = Enum.VerticalAlignment.Center
local body = Instance.new("Frame", main)
body.Size = UDim2.new(1,-144,1,-106)
body.Position = UDim2.new(0,136,0,64)
body.BackgroundTransparency = 1
local function newPage(n)
  local p = Instance.new("ScrollingFrame", body)
  p.Name = n
  p.Size = UDim2.new(1,0,1,0)
  p.BackgroundTransparency = 1
  p.Visible = false
  p.ScrollBarThickness = 2
  p.CanvasSize = UDim2.new(0,0,0,900)
  local l = Instance.new("UIListLayout", p)
  l.Padding = UDim.new(0,8)
  return p
end
local pageLocal = newPage("Local")
local pageAim = newPage("Aim")
local pageVis = newPage("Vis")
local pageMisc = newPage("Misc")
local pageCfg = newPage("Cfg")
local tabs = {}
local function makeTab(name, icon, page)
  local b = Instance.new("TextButton", side)
  b.Size = UDim2.new(1,0,0,34)
  b.AutoButtonColor = false
  b.BackgroundColor3 = Color3.fromRGB(12,12,18)
  b.Text = ""
  Instance.new("UICorner", b).CornerRadius = UDim.new(0,10)
  local lb = Instance.new("TextLabel", b)
  lb.Size = UDim2.new(1,0,1,0)
  lb.BackgroundTransparency = 1
  lb.Text = icon.." "..name
  lb.Font = Enum.Font.GothamMedium
  lb.TextSize = 11
  lb.TextColor3 = Color3.new(1,1,1)
  tabs[name] = {btn=b, page=page}
  b.MouseButton1Click:Connect(function()
    if not Unlocked then return end
    for n,t in pairs(tabs) do local sel = (n == name) t.page.Visible = sel tween(t.btn, {BackgroundColor3 = sel and ACCENT_DARK or Color3.fromRGB(12,12,18)}, 0.18) end
  end)
end
makeTab("Localplayer","🏃",pageLocal)
makeTab("Aim","◎",pageAim)
makeTab("Visuals","◉",pageVis)
makeTab("Misc","⚙",pageMisc)
makeTab("Config","💾",pageCfg)
tabs["Localplayer"].page.Visible = true
local Boxes = {}
local function refreshToggleVisual(key)
  local u = ToggleUI[key]
  if not u then return end
  tween(u.sw, {BackgroundColor3 = Toggles[key] and ACCENT or Color3.fromRGB(40,40,55)}, 0.15)
  tween(u.knob, {Position = Toggles[key] and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)}, 0.15)
end
local function refreshAllVisuals()
  for k,_ in pairs(ToggleUI) do refreshToggleVisual(k) end
  for k,b in pairs(Boxes) do
    if k == "FOV" then b.Text = tostring(FOV)
    elseif k == "WS" then b.Text = tostring(WalkSpeed)
    elseif k == "JP" then b.Text = tostring(JumpPower)
    elseif k == "FS" then b.Text = tostring(FlySpeed)
    elseif k == "ET" then b.Text = tostring(ESPTrans) end
  end
  circle.Size = UDim2.fromOffset(FOV*2, FOV*2)
  fpsLabel.Visible = Toggles.ShowFPS
end
local function makeToggle(parent,label,key,onChange)
  local row = Instance.new("TextButton", parent)
  row.Size = UDim2.new(1,0,0,42)
  row.AutoButtonColor = false
  row.BackgroundColor3 = Color3.fromRGB(10,10,16)
  row.Text = ""
  Instance.new("UICorner", row).CornerRadius = UDim.new(0,10)
  local lbl = Instance.new("TextLabel", row)
  lbl.Size = UDim2.new(1,-70,1,0)
  lbl.Position = UDim2.new(0,12,0,0)
  lbl.BackgroundTransparency = 1
  lbl.TextXAlignment = Enum.TextXAlignment.Left
  lbl.Text = label
  lbl.Font = Enum.Font.GothamMedium
  lbl.TextSize = 12
  lbl.TextColor3 = Color3.new(1,1,1)
  local sw = Instance.new("Frame", row)
  sw.Size = UDim2.new(0,46,0,24)
  sw.Position = UDim2.new(1,-56,0.5,-12)
  sw.BackgroundColor3 = Toggles[key] and ACCENT or Color3.fromRGB(40,40,55)
  Instance.new("UICorner", sw).CornerRadius = UDim.new(1,0)
  local knob = Instance.new("Frame", sw)
  knob.Size = UDim2.new(0,18,0,18)
  knob.Position = Toggles[key] and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)
  knob.BackgroundColor3 = Color3.new(1,1,1)
  Instance.new("UICorner", knob).CornerRadius = UDim.new(1,0)
  ToggleUI[key] = {sw=sw, knob=knob}
  row.MouseButton1Click:Connect(function()
    if not Unlocked then return end
    Toggles[key] = not Toggles[key]
    refreshToggleVisual(key)
    fpsLabel.Visible = Toggles.ShowFPS
    if onChange then onChange(Toggles[key]) end
  end)
end
local function pillRow(parent, label, unit, getV, setV, boxKey)
  local r = Instance.new("Frame", parent)
  r.Size = UDim2.new(1,0,0,42)
  r.BackgroundColor3 = Color3.fromRGB(18,18,24)
  Instance.new("UICorner", r).CornerRadius = UDim.new(0,8)
  local a = Instance.new("TextLabel", r)
  a.Size = UDim2.new(0.5,0,1,0)
  a.Position = UDim2.new(0,12,0,0)
  a.BackgroundTransparency = 1
  a.TextXAlignment = Enum.TextXAlignment.Left
  a.Text = label
  a.Font = Enum.Font.Gotham
  a.TextSize = 12
  a.TextColor3 = Color3.fromRGB(200,200,220)
  local pill = Instance.new("Frame", r)
  pill.Size = UDim2.new(0,120,0,28)
  pill.Position = UDim2.new(1,-130,0.5,-14)
  pill.BackgroundColor3 = Color3.fromRGB(90,90,100)
  pill.BackgroundTransparency = 0.35
  Instance.new("UICorner", pill).CornerRadius = UDim.new(1,0)
  local box = Instance.new("TextBox", pill)
  box.Size = UDim2.new(0.55,0,1,0)
  box.BackgroundTransparency = 1
  box.Text = tostring(getV())
  box.Font = Enum.Font.GothamBold
  box.TextSize = 12
  box.TextColor3 = Color3.new(1,1,1)
  Boxes[boxKey] = box
  local lb = Instance.new("TextLabel", pill)
  lb.Size = UDim2.new(0.45,0,1,0)
  lb.Position = UDim2.new(0.55,0,0,0)
  lb.BackgroundTransparency = 1
  lb.TextXAlignment = Enum.TextXAlignment.Left
  lb.Text = unit
  lb.Font = Enum.Font.Gotham
  lb.TextSize = 11
  lb.TextColor3 = Color3.new(1,1,1)
  box.FocusLost:Connect(function(e) if e and Unlocked then setV(tonumber(box.Text) or getV()) box.Text = tostring(getV()) end end)
end
local function sectionL(parent, t)
  local l = Instance.new("TextLabel", parent)
  l.Size = UDim2.new(1,0,0,16)
  l.BackgroundTransparency = 1
  l.TextXAlignment = Enum.TextXAlignment.Left
  l.Text = t
  l.Font = Enum.Font.GothamBold
  l.TextSize = 11
  l.TextColor3 = Color3.fromRGB(120,170,220)
end
pillRow(pageLocal,"Walkspeed","Speed", function() return WalkSpeed end, function(v) WalkSpeed = math.clamp(math.floor(v),0,500) local h = myHum() if h then h.WalkSpeed = WalkSpeed end end, "WS")
pillRow(pageLocal,"Jump Power","Power", function() return JumpPower end, function(v) JumpPower = math.clamp(math.floor(v),0,500) local h = myHum() if h then if h.UseJumpPower then h.JumpPower = JumpPower else h.JumpHeight = JumpPower/7 end end end, "JP")
makeToggle(pageLocal,"Infinite Jump","InfJump")
makeToggle(pageLocal,"Noclip","Noclip", function(on) if not on then setCollision(false) end end)
makeToggle(pageLocal,"Fly","Fly")
pillRow(pageLocal,"Fly Speed","Speed", function() return FlySpeed end, function(v) FlySpeed = math.clamp(math.floor(v),1,300) end, "FS")
makeToggle(pageAim,"Aim Assist (RMB)","AimAssist")
makeToggle(pageAim,"Show FOV","ShowFOV")
makeToggle(pageAim,"Visible Check","VisibleCheck")
makeToggle(pageAim,"Ignore Dead","DeadCheck")
makeToggle(pageAim,"Team Check","TeamCheck")
pillRow(pageAim,"FOV","", function() return FOV end, function(v) FOV = math.clamp(math.floor(v),FOV_MIN,FOV_MAX) circle.Size = UDim2.fromOffset(FOV*2,FOV*2) end, "FOV")
local statusL = Instance.new("TextLabel", pageAim)
statusL.Size = UDim2.new(1,0,0,36)
statusL.BackgroundColor3 = Color3.fromRGB(10,10,16)
statusL.Text = "Status..."
statusL.Font = Enum.Font.Code
statusL.TextSize = 11
statusL.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", statusL).CornerRadius = UDim.new(0,8)
sectionL(pageVis, "ESP")
makeToggle(pageVis,"Enemy ESP","ESPEnemies", function() refreshESP() end)
makeToggle(pageVis,"Show Names","ShowNames", function() refreshESP() end)
sectionL(pageVis, "Color")
local colorRow = Instance.new("Frame", pageVis)
colorRow.Size = UDim2.new(1,0,0,40)
colorRow.BackgroundColor3 = Color3.fromRGB(10,10,16)
Instance.new("UICorner", colorRow).CornerRadius = UDim.new(0,8)
local clist = Instance.new("UIListLayout", colorRow)
clist.FillDirection = Enum.FillDirection.Horizontal
clist.Padding = UDim.new(0,6)
clist.HorizontalAlignment = Enum.HorizontalAlignment.Center
clist.VerticalAlignment = Enum.VerticalAlignment.Center
local COLORS = {{n="Red", c=Color3.fromRGB(255,0,0)}, {n="Blue", c=Color3.fromRGB(0,140,255)}, {n="Green", c=Color3.fromRGB(0,255,120)}, {n="Yellow", c=Color3.fromRGB(255,210,0)}, {n="Purple", c=Color3.fromRGB(170,80,255)}, {n="White", c=Color3.new(1,1,1)}}
for _,e in pairs(COLORS) do
  local b = Instance.new("TextButton", colorRow)
  b.Size = UDim2.new(0,32,0,28)
  b.BackgroundColor3 = e.c
  b.Text = ""
  Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
  b.MouseButton1Click:Connect(function() if not Unlocked then return end ESPColor = e.c refreshESP() end)
end
sectionL(pageVis, "Thickness")
pillRow(pageVis,"Fill transparency","0-100", function() return ESPTrans end, function(v) ESPTrans = math.clamp(math.floor(v),0,95) refreshESP() end, "ET")
sectionL(pageMisc, "Performance")
makeToggle(pageMisc,"Show FPS","ShowFPS")
makeToggle(pageMisc,"Optimize Graphics","Optimize", function(on) setOptimize(on) end)
local optInfo = Instance.new("TextLabel", pageMisc)
optInfo.Size = UDim2.new(1,0,0,50)
optInfo.BackgroundColor3 = Color3.fromRGB(10,10,16)
optInfo.TextWrapped = true
optInfo.Text = "Light boost: no shadows off, keeps textures. Disables bloom/rays, lowers particles and water."
optInfo.Font = Enum.Font.Gotham
optInfo.TextSize = 11
optInfo.TextColor3 = Color3.fromRGB(140,140,160)
Instance.new("UICorner", optInfo).CornerRadius = UDim.new(0,8)
local cfgName = Instance.new("TextBox", pageCfg)
cfgName.Size = UDim2.new(1,0,0,40)
cfgName.PlaceholderText = "Config name..."
cfgName.Text = ""
cfgName.BackgroundColor3 = Color3.fromRGB(10,10,16)
cfgName.TextColor3 = Color3.new(1,1,1)
cfgName.Font = Enum.Font.GothamMedium
cfgName.TextSize = 13
Instance.new("UICorner", cfgName).CornerRadius = UDim.new(0,8)
local function cfgBtn(label, fn)
  local b = Instance.new("TextButton", pageCfg)
  b.Size = UDim2.new(1,0,0,36)
  b.Text = label
  b.Font = Enum.Font.GothamBold
  b.TextSize = 12
  b.BackgroundColor3 = ACCENT_DARK
  b.TextColor3 = ACCENT
  Instance.new("UICorner", b).CornerRadius = UDim.new(0,8)
  b.MouseButton1Click:Connect(function() if Unlocked then fn() end end)
  return b
end
local cfgList = Instance.new("ScrollingFrame", pageCfg)
cfgList.Size = UDim2.new(1,0,0,180)
cfgList.BackgroundColor3 = Color3.fromRGB(8,8,12)
cfgList.ScrollBarThickness = 2
cfgList.CanvasSize = UDim2.new(0,0,0,0)
Instance.new("UICorner", cfgList).CornerRadius = UDim.new(0,8)
local cfgLayout = Instance.new("UIListLayout", cfgList)
cfgLayout.Padding = UDim.new(0,6)
local function applyData(d)
  if not d then return end
  if d.Toggles then for k,v in pairs(d.Toggles) do if Toggles[k] ~= nil then Toggles[k] = v end end end
  if d.FOV then FOV = math.clamp(d.FOV, FOV_MIN, FOV_MAX) end
  if d.WalkSpeed then WalkSpeed = math.clamp(d.WalkSpeed,0,500) end
  if d.JumpPower then JumpPower = math.clamp(d.JumpPower,0,500) end
  if d.FlySpeed then FlySpeed = math.clamp(d.FlySpeed,1,300) end
  if d.ESPColor then pcall(function() ESPColor = Color3.new(d.ESPColor[1],d.ESPColor[2],d.ESPColor[3]) end) end
  if d.ESPTrans then ESPTrans = math.clamp(d.ESPTrans,0,95) end
  local h = myHum()
  if h then h.WalkSpeed = WalkSpeed if h.UseJumpPower then h.JumpPower = JumpPower else h.JumpHeight = JumpPower/7 end end
  if not Toggles.Noclip then setCollision(false) end
  if Toggles.Optimize then setOptimize(true) else setOptimize(false) end
  refreshAllVisuals()
  refreshESP()
end
local function refreshList()
  for _,c in pairs(cfgList:GetChildren()) do if c:IsA("TextButton") then c:Destroy() end end
  local y = 0
  for name,_ in pairs(Configs) do
    local b = Instance.new("TextButton", cfgList)
    b.Size = UDim2.new(1,-8,0,32)
    local star = (name == AutoloadConfig) and "★ " or ""
    local sel = (name == SelectedConfig) and "● " or ""
    b.Text = sel..star..name
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 12
    b.BackgroundColor3 = (name == SelectedConfig) and ACCENT_DARK or Color3.fromRGB(15,15,22)
    b.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
    b.MouseButton1Click:Connect(function() SelectedConfig = name applyData(Configs[name]) refreshList() end)
    y = y + 38
  end
  cfgList.CanvasSize = UDim2.new(0,0,0,y)
end
cfgBtn("Create config", function()
  local n = tostring(cfgName.Text):gsub("^%s+",""):gsub("%s+$","")
  if n == "" then return end
  Configs[n] = snapshot()
  SelectedConfig = n
  saveFile()
  refreshList()
end)
cfgBtn("Refresh list", function() refreshList() end)
cfgBtn("Overwrite config", function() if not SelectedConfig then return end Configs[SelectedConfig] = snapshot() saveFile() refreshList() end)
cfgBtn("Set as autoload", function() if not SelectedConfig then return end AutoloadConfig = SelectedConfig saveFile() refreshList() end)
cfgBtn("Delete config", function() if not SelectedConfig then return end Configs[SelectedConfig] = nil if AutoloadConfig == SelectedConfig then AutoloadConfig = nil end SelectedConfig = nil saveFile() refreshList() end)
refreshList()
local hidden = false
local rmbHeld = false
local flyK = {W=false,A=false,S=false,D=false,Up=false,Down=false}
local function setVisible(v)
  if not Unlocked then return end
  hidden = not v
  if v then main.Visible = true circle.Visible = Toggles.ShowFOV and Toggles.AimAssist tween(scale,{Scale=1},0.18)
  else main.Visible = false circle.Visible = false end
end
minBtn.MouseButton1Click:Connect(function() setVisible(false) end)
UserInputService.InputBegan:Connect(function(inp,gpe)
  if not gpe and inp.KeyCode == Enum.KeyCode.RightShift and Unlocked then setVisible(hidden) end
  if inp.UserInputType == Enum.UserInputType.MouseButton2 then rmbHold = true end
  if inp.KeyCode == Enum.KeyCode.W then flyK.W = true end
  if inp.KeyCode == Enum.KeyCode.A then flyK.A = true end
  if inp.KeyCode == Enum.KeyCode.S then flyK.S = true end
  if inp.KeyCode == Enum.KeyCode.D then flyK.D = true end
  if inp.KeyCode == Enum.KeyCode.Space then flyK.Up = true end
  if inp.KeyCode == Enum.KeyCode.LeftShift then flyK.Down = true end
end)
UserInputService.InputEnded:Connect(function(inp)
  if inp.UserInputType == Enum.UserInputType.MouseButton2 then rmbHold = false end
  if inp.KeyCode == Enum.KeyCode.W then flyK.W = false end
  if inp.KeyCode == Enum.KeyCode.A then flyK.A = false end
  if inp.KeyCode == Enum.KeyCode.S then flyK.S = false end
  if inp.KeyCode == Enum.KeyCode.D then flyK.D = false end
  if inp.KeyCode == Enum.KeyCode.Space then flyK.Up = false end
  if inp.KeyCode == Enum.KeyCode.LeftShift then flyK.Down = false end
end)
UserInputService.JumpRequest:Connect(function() if Unlocked and Toggles.InfJump then local h = myHum() if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end end end)
LocalPlayer.CharacterAdded:Connect(function(c)
  c:WaitForChild("Humanoid", 10)
  task.wait(0.3)
  local h = c:FindFirstChildOfClass("Humanoid")
  if h then h.WalkSpeed = WalkSpeed if h.UseJumpPower then h.JumpPower = JumpPower else h.JumpHeight = JumpPower/7 end end
  if Toggles.Noclip then setCollision(true) else setCollision(false) end
  applyESP(c)
end)
local function clearAllESP()
  for _,m in pairs(workspace:GetDescendants()) do
    if m:IsA("Highlight") and m.Name == "DevESP" then m:Destroy() end
    if m:IsA("BillboardGui") and m.Name == "DevName" then m:Destroy() end
  end
end
local function applyESP(c)
  if not Unlocked or not Toggles.ESPEnemies then return end
  if c == LocalPlayer.Character or not getESPPart(c) or isSameRole(c) then
    local h = c:FindFirstChild("DevESP") if h then h:Destroy() end
    local head = c:FindFirstChild("Head")
    if head then local bb = head:FindFirstChild("DevName") if bb then bb:Destroy() end end
    return
  end
  local ex = c:FindFirstChild("DevESP")
  if not ex then
    ex = Instance.new("Highlight")
    ex.Name = "DevESP"
    ex.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    ex.Parent = c
  end
  ex.FillColor = ESPColor
  ex.OutlineColor = Color3.new(1,1,1)
  ex.FillTransparency = ESPTrans/100
  local head = c:FindFirstChild("Head")
  if Toggles.ShowNames and head then
    local bb = ensureName(c)
    local plr = Players:GetPlayerFromCharacter(c)
    local d = 0
    pcall(function() d = math.floor((workspace.CurrentCamera.CFrame.Position - head.Position).Magnitude) end)
    bb:FindFirstChildOfClass("TextLabel").Text = (plr and plr.Name or c.Name).." ["..d.."m]"
    bb:FindFirstChildOfClass("TextLabel").TextColor3 = ESPColor
  else
    if head then local bb = head:FindFirstChild("DevName") if bb then bb:Destroy() end end
  end
end
function refreshESP()
  if not Unlocked then return end
  if not Toggles.ESPEnemies then
    for _,c in pairs(getAllESPTargets()) do
      local h = c:FindFirstChild("DevESP") if h then h:Destroy() end
      local hd = c:FindFirstChild("Head")
      if hd then local bb = hd:FindFirstChild("DevName") if bb then bb:Destroy() end end
    end
    return
  end
  for _,c in pairs(getAllESPTargets()) do applyESP(c) end
end
local function hookP(p)
  p.CharacterAdded:Connect(function(c) c:WaitForChild("HumanoidRootPart",10) task.wait(0.3) applyESP(c) end)
  if p.Character then applyESP(p.Character) end
end
for _,p in pairs(Players:GetPlayers()) do hookP(p) end
Players.PlayerAdded:Connect(hookP)
local function getClosest(cam)
  local ctr = cam.ViewportSize/2
  local cds = {}
  for _,c in pairs(getAllTargets()) do
    if Toggles.DeadCheck and not isAlive(c) then continue end
    if Toggles.TeamCheck and isSameRole(c) then continue end
    local pt = getAimPart(c)
    if not pt then continue end
    local pos,ok = cam:WorldToViewportPoint(pt.Position)
    if not ok then continue end
    local d = (Vector2.new(pos.X,pos.Y)-ctr).Magnitude
    if d < FOV then table.insert(cds, {part=pt, model=c, dist=d}) end
  end
  table.sort(cds, function(a,b) return a.dist < b.dist end)
  for _,e in pairs(cds) do
    if Toggles.VisibleCheck and not isVisible(cam, e.part, e.model) then continue end
    return e.part
  end
  return nil
end
RunService.Stepped:Connect(function() if Unlocked and Toggles.Noclip then setCollision(true) end end)
RunService.Heartbeat:Connect(function(dt)
  if not Unlocked then return end
  local hrp = myHRP()
  local h = myHum()
  if not hrp or not h then return end
  if Toggles.Fly then
    local cam = workspace.CurrentCamera
    if not cam then return end
    local cf = cam.CFrame
    local mv = Vector3.new()
    if flyK.W then mv += cf.LookVector end
    if flyK.S then mv -= cf.LookVector end
    if flyK.A then mv -= cf.RightVector end
    if flyK.D then mv += cf.RightVector end
    if flyK.Up then mv += Vector3.new(0,1,0) end
    if flyK.Down then mv -= Vector3.new(0,1,0) end
    if mv.Magnitude > 0 then mv = mv.Unit * FlySpeed * dt else mv = Vector3.new() end
    hrp.CFrame = hrp.CFrame + mv
    hrp.AssemblyLinearVelocity = Vector3.new()
  end
end)
RunService.RenderStepped:Connect(function()
  if not Unlocked then return end
  circle.Size = UDim2.fromOffset(FOV*2, FOV*2)
  if not hidden then circle.Visible = Toggles.ShowFOV and Toggles.AimAssist end
  if Toggles.ESPEnemies and Toggles.ShowNames then
    for _,c in pairs(getAllESPTargets()) do if c:FindFirstChild("DevESP") then applyESP(c) end end
  end
end)
RunService:BindToRenderStep("LZ_Aim", Enum.RenderPriority.Last.Value, function()
  if not Unlocked then return end
  local cam = workspace.CurrentCamera
  if not cam then return end
  local holding = rmbHeld or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
  if not (Toggles.AimAssist and holding) then statusL.Text = "Aim OFF" return end
  local target = getClosest(cam)
  if not target then statusL.Text = "No target" return end
  statusL.Text = "LOCK "..target:GetFullName()
  local goal = CFrame.lookAt(cam.CFrame.Position, target.Position)
  cam.CFrame = goal
  cam.Focus = CFrame.new(target.Position)
end)
local keyF = Instance.new("Frame", gui)
keyF.Size = UDim2.new(0,340,0,230)
keyF.Position = UDim2.new(0.5,-170,0.5,-115)
keyF.BackgroundColor3 = Color3.fromRGB(0,0,0)
Instance.new("UICorner", keyF).CornerRadius = UDim.new(0,12)
local kst = Instance.new("UIStroke", keyF)
kst.Color = ACCENT
local kt = Instance.new("TextLabel", keyF)
kt.Size = UDim2.new(1,0,0,40)
kt.BackgroundTransparency = 1
kt.Text = "LZ MENU - KEY"
kt.Font = Enum.Font.GothamBold
kt.TextSize = 15
kt.TextColor3 = ACCENT
local kb = Instance.new("TextBox", keyF)
kb.Size = UDim2.new(1,-30,0,40)
kb.Position = UDim2.new(0,15,0,55)
kb.PlaceholderText = "Enter key..."
kb.Text = ""
kb.BackgroundColor3 = Color3.fromRGB(12,12,18)
kb.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", kb).CornerRadius = UDim.new(0,8)
local getB = Instance.new("TextButton", keyF)
getB.Size = UDim2.new(0.5,-22,0,40)
getB.Position = UDim2.new(0,15,0,105)
getB.Text = "GET KEY"
getB.Font = Enum.Font.GothamBold
getB.TextSize = 13
getB.BackgroundColor3 = Color3.fromRGB(18,28,48)
getB.TextColor3 = ACCENT
Instance.new("UICorner", getB).CornerRadius = UDim.new(0,8)
local checkB = Instance.new("TextButton", keyF)
checkB.Size = UDim2.new(0.5,-22,0,40)
checkB.Position = UDim2.new(0.5,7,0,105)
checkB.Text = "CHECK KEY"
checkB.Font = Enum.Font.GothamBold
checkB.TextSize = 13
checkB.BackgroundColor3 = ACCENT_DARK
checkB.TextColor3 = ACCENT
Instance.new("UICorner", checkB).CornerRadius = UDim.new(0,8)
local km = Instance.new("TextLabel", keyF)
km.Size = UDim2.new(1,-30,0,40)
km.Position = UDim2.new(0,15,0,150)
km.BackgroundTransparency = 1
km.Text = ""
km.TextWrapped = true
km.Font = Enum.Font.Gotham
km.TextSize = 11
km.TextColor3 = Color3.fromRGB(255,90,90)
local function unlock()
  if Unlocked then return end
  Unlocked = true
  keyF:Destroy()
  main.Visible = true
  tween(scale, {Scale=1}, 0.25)
  if AutoloadConfig and Configs[AutoloadConfig] then SelectedConfig = AutoloadConfig applyData(Configs[AutoloadConfig]) refreshList() end
  refreshESP()
end
local function tryK(k)
  k = tostring(k or ""):gsub("%s+",""):upper()
  if k == string.upper(FIXED_KEY) then unlock() else km.Text = "Invalid key!" end
end
checkB.MouseButton1Click:Connect(function() tryK(kb.Text) end)
getB.MouseButton1Click:Connect(function()
  pcall(function() if setclipboard then setclipboard(KEY_LINK) end end)
  pcall(function() GuiService:OpenBrowserWindow(KEY_LINK) end)
  km.TextColor3 = ACCENT
  km.Text = "Link copied! Complete Linkvertise to get key."
end)
