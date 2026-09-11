-- LZ MENU com KEY | Aim | Visuals | Misc
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
pcall(function() RunService:UnbindFromRenderStep("LZ_Aim") end)

-- CONFIG KEY --
local VALID_KEYS = {["teste"]=true, ["LZ-VIP-2026"]=true}
local ALLOWED_USERIDS = {[12345678]=true} -- coloque seu UserId aqui, pula a key
local Unlocked = false

local Toggles = {ESP=false, AimAssist=false, ShowFOV=true, ShowFPS=false, VisibleCheck=true, DeadCheck=true}
local FOV = 300
local FOV_MIN, FOV_MAX = 50, 1000
local ACCENT = Color3.fromRGB(0,140,255)
local ACCENT_DARK = Color3.fromRGB(12,32,68)
local function tween(o,p,t) TweenService:Create(o,TweenInfo.new(t or 0.18,Enum.EasingStyle.Quad,Enum.EasingDirection.Out),p):Play() end

local gui = Instance.new("ScreenGui")
gui.Name="LZMENU"; gui.ResetOnSpawn=false; gui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
gui.Parent=LocalPlayer:WaitForChild("PlayerGui")

local circle = Instance.new("Frame")
circle.AnchorPoint=Vector2.new(0.5,0.5); circle.Position=UDim2.new(0.5,0,0.5,0)
circle.Size=UDim2.fromOffset(FOV*2,FOV*2); circle.BackgroundTransparency=1
circle.Visible=false; circle.Parent=gui
Instance.new("UICorner",circle).CornerRadius=UDim.new(1,0)
local cs=Instance.new("UIStroke",circle); cs.Color=ACCENT; cs.Thickness=2; cs.Transparency=0.1

local fpsLabel = Instance.new("TextLabel",gui)
fpsLabel.Size=UDim2.new(0,100,0,28); fpsLabel.Position=UDim2.new(1,-110,0,10)
fpsLabel.BackgroundColor3=Color3.fromRGB(0,0,0); fpsLabel.TextColor3=ACCENT
fpsLabel.Font=Enum.Font.GothamBold; fpsLabel.TextSize=13; fpsLabel.Text="FPS: --"; fpsLabel.Visible=false
Instance.new("UICorner",fpsLabel).CornerRadius=UDim.new(0,8)

local main = Instance.new("Frame")
main.Size=UDim2.new(0,430,0,520); main.Position=UDim2.new(0,20,0,50)
main.BackgroundColor3=Color3.fromRGB(0,0,0); main.BorderSizePixel=0
main.Active=true; main.Draggable=false; main.Parent=gui; main.Visible=false
Instance.new("UICorner",main).CornerRadius=UDim.new(0,12)
local ms=Instance.new("UIStroke",main); ms.Color=ACCENT; ms.Thickness=1; ms.Transparency=0.25
local scale=Instance.new("UIScale",main); scale.Scale=0.85

local top = Instance.new("Frame",main)
top.Size=UDim2.new(1,0,0,52); top.BackgroundColor3=Color3.fromRGB(8,8,12); top.BorderSizePixel=0; top.Active=true
Instance.new("UICorner",top).CornerRadius=UDim.new(0,12)
local fix=Instance.new("Frame",top); fix.Size=UDim2.new(1,0,0,12); fix.Position=UDim2.new(0,0,1,-12)
fix.BackgroundColor3=top.BackgroundColor3; fix.BorderSizePixel=0
local title=Instance.new("TextLabel",top); title.Size=UDim2.new(1,-60,0,22); title.Position=UDim2.new(0,14,0,6)
title.BackgroundTransparency=1; title.TextXAlignment=Enum.TextXAlignment.Left
title.Text="LZ MENU"; title.Font=Enum.Font.GothamBold; title.TextSize=16; title.TextColor3=ACCENT
local sub=Instance.new("TextLabel",top); sub.Size=UDim2.new(1,-60,0,16); sub.Position=UDim2.new(0,14,0,27)
sub.BackgroundTransparency=1; sub.TextXAlignment=Enum.TextXAlignment.Left
sub.Text="Desenvolvido por Yuri"; sub.Font=Enum.Font.Gotham; sub.TextSize=12; sub.TextColor3=Color3.fromRGB(120,170,220)
local minBtn=Instance.new("TextButton",top); minBtn.Size=UDim2.new(0,32,0,32); minBtn.Position=UDim2.new(1,-40,0,10)
minBtn.Text="—"; minBtn.Font=Enum.Font.GothamBold; minBtn.TextSize=14
minBtn.BackgroundColor3=Color3.fromRGB(15,25,45); minBtn.TextColor3=ACCENT; minBtn.AutoButtonColor=false
Instance.new("UICorner",minBtn).CornerRadius=UDim.new(0,8)
do
  local dragging=false; local dragStart; local startPos
  top.InputBegan:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then dragging=true; dragStart=i.Position; startPos=main.Position; i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then dragging=false end end) end end)
  UserInputService.InputChanged:Connect(function(i) if dragging and i.UserInputType==Enum.UserInputType.MouseMovement then local d=i.Position-dragStart; main.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+d.X,startPos.Y.Scale,startPos.Y.Offset+d.Y) end end)
end

local side=Instance.new("Frame",main)
side.Size=UDim2.new(0,120,1,-64); side.Position=UDim2.new(0,8,0,58)
side.BackgroundColor3=Color3.fromRGB(5,5,9); side.BorderSizePixel=0
Instance.new("UICorner",side).CornerRadius=UDim.new(0,10)
local sideList=Instance.new("UIListLayout",side); sideList.Padding=UDim.new(0,8)
sideList.HorizontalAlignment=Enum.HorizontalAlignment.Center; sideList.VerticalAlignment=Enum.VerticalAlignment.Center
local sidePad=Instance.new("UIPadding",side); sidePad.PaddingLeft=UDim.new(0,8); sidePad.PaddingRight=UDim.new(0,8)
local body=Instance.new("Frame",main)
body.Size=UDim2.new(1,-144,1,-106); body.Position=UDim2.new(0,136,0,64); body.BackgroundTransparency=1
local pageTitle=Instance.new("TextLabel",main)
pageTitle.Size=UDim2.new(1,-154,0,20); pageTitle.Position=UDim2.new(0,136,1,-38)
pageTitle.BackgroundTransparency=1; pageTitle.TextXAlignment=Enum.TextXAlignment.Left
pageTitle.Font=Enum.Font.Gotham; pageTitle.TextSize=11; pageTitle.TextColor3=Color3.fromRGB(100,150,200)
pageTitle.Text="RightShift: esconder • RMB: mirar"
local function newPage(n)
  local p=Instance.new("Frame",body); p.Name=n; p.Size=UDim2.new(1,0,1,0)
  p.BackgroundTransparency=1; p.Visible=false
  local l=Instance.new("UIListLayout",p); l.Padding=UDim.new(0,8)
  return p
end
local pageAim=newPage("Aim"); local pageVis=newPage("Visuals"); local pageMisc=newPage("Misc")
local tabBtns={}
local function makeTab(name,icon,page)
  local b=Instance.new("TextButton",side); b.Size=UDim2.new(1,0,0,48); b.AutoButtonColor=false
  b.BackgroundColor3=Color3.fromRGB(12,12,18); b.Text=""
  Instance.new("UICorner",b).CornerRadius=UDim.new(0,10)
  local bar=Instance.new("Frame",b); bar.Size=UDim2.new(0,4,0,28); bar.Position=UDim2.new(0,6,0.5,-14)
  bar.BackgroundColor3=ACCENT; bar.Visible=false; bar.BorderSizePixel=0
  Instance.new("UICorner",bar).CornerRadius=UDim.new(1,0)
  local ic=Instance.new("TextLabel",b); ic.Size=UDim2.new(1,0,0,20); ic.Position=UDim2.new(0,0,0,4)
  ic.BackgroundTransparency=1; ic.Text=icon; ic.Font=Enum.Font.GothamBold; ic.TextSize=16; ic.TextColor3=ACCENT
  local lb=Instance.new("TextLabel",b); lb.Size=UDim2.new(1,0,0,16); lb.Position=UDim2.new(0,0,0,26)
  lb.BackgroundTransparency=1; lb.Text=name; lb.Font=Enum.Font.GothamMedium; lb.TextSize=12; lb.TextColor3=Color3.new(1,1,1)
  tabBtns[name]={btn=b,bar=bar,page=page}
  b.MouseButton1Click:Connect(function()
    if not Unlocked then return end
    for n,t in pairs(tabBtns) do local sel=(n==name); t.page.Visible=sel; t.bar.Visible=sel
      tween(t.btn,{BackgroundColor3=sel and ACCENT_DARK or Color3.fromRGB(12,12,18)},0.18) end
  end)
end
makeTab("Aim","◎",pageAim); makeTab("Visuals","◉",pageVis); makeTab("Misc","⚙",pageMisc)
tabBtns["Aim"].page.Visible=true; tabBtns["Aim"].bar.Visible=true; tabBtns["Aim"].btn.BackgroundColor3=ACCENT_DARK

local function isAlive(model)
  local h=model:FindFirstChildOfClass("Humanoid")
  if not h then return false end
  if h.Health<=0 then return false end
  if h:GetState()==Enum.HumanoidStateType.Dead then return false end
  return true
end
local rayParams=RaycastParams.new()
rayParams.FilterType=Enum.RaycastFilterType.Exclude
rayParams.IgnoreWater=true
local function isVisible(cam, part, model)
  local origin=cam.CFrame.Position
  local dir=part.Position-origin
  local dist=dir.Magnitude
  if dist<0.1 then return true end
  rayParams.FilterDescendantsInstances={LocalPlayer.Character, model}
  local res=workspace:Raycast(origin, dir, rayParams)
  if not res then return true end
  if res.Distance>=dist-1 then return true end
  if res.Instance and model:IsAncestorOf(res.Instance) then return true end
  return false
end
local function getAimPart(c)
  if not c then return nil end
  return c:FindFirstChild("Head") or c:FindFirstChild("UpperTorso") or c:FindFirstChild("LowerTorso")
    or c:FindFirstChild("Torso") or c:FindFirstChild("HumanoidRootPart") or c:FindFirstChildWhichIsA("BasePart",true)
end
local function getAllTargets()
  local t={}; for _,p in pairs(Players:GetPlayers()) do
    if p~=LocalPlayer and p.Character and getAimPart(p.Character) then table.insert(t,p.Character) end end
  for _,m in pairs(workspace:GetDescendants()) do
    if m:IsA("Model") and m:FindFirstChildOfClass("Humanoid") then
      if m~=LocalPlayer.Character and not Players:GetPlayerFromCharacter(m) then
        if getAimPart(m) then table.insert(t,m) end end end end
  return t
end
local function clearAllESP()
  for _,m in pairs(workspace:GetDescendants()) do
    if m:IsA("Highlight") and m.Name=="DevESP" then m:Destroy() end
  end
end
local function applyESP(c)
  if not Unlocked or not Toggles.ESP then return end
  if c==LocalPlayer.Character then return end
  if Toggles.DeadCheck and not isAlive(c) then return end
  if c:FindFirstChild("DevESP") then return end
  local hl=Instance.new("Highlight"); hl.Name="DevESP"; hl.FillTransparency=0.6
  hl.FillColor=Color3.fromRGB(0,140,255); hl.OutlineColor=Color3.new(1,1,1)
  hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop; hl.Parent=c
end
local function refreshESP()
  if not Unlocked then return end
  if Toggles.ESP then for _,c in pairs(getAllTargets()) do applyESP(c) end
  else clearAllESP() end
end

local function makeToggle(parent,label,key,onChange)
  local row=Instance.new("TextButton",parent); row.Size=UDim2.new(1,0,0,40); row.AutoButtonColor=false
  row.BackgroundColor3=Color3.fromRGB(10,10,16); row.Text=""; row.Active=true
  Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
  local lbl=Instance.new("TextLabel",row); lbl.Size=UDim2.new(1,-70,1,0); lbl.Position=UDim2.new(0,12,0,0)
  lbl.BackgroundTransparency=1; lbl.TextXAlignment=Enum.TextXAlignment.Left
  lbl.Text=label; lbl.Font=Enum.Font.GothamMedium; lbl.TextSize=12; lbl.TextColor3=Color3.new(1,1,1)
  local sw=Instance.new("Frame",row); sw.Size=UDim2.new(0,46,0,24); sw.Position=UDim2.new(1,-56,0.5,-12)
  sw.BackgroundColor3=Toggles[key] and ACCENT or Color3.fromRGB(40,40,55); sw.BorderSizePixel=0
  Instance.new("UICorner",sw).CornerRadius=UDim.new(1,0)
  local knob=Instance.new("Frame",sw); knob.Size=UDim2.new(0,18,0,18)
  knob.Position=Toggles[key] and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)
  knob.BackgroundColor3=Color3.new(1,1,1); knob.BorderSizePixel=0
  Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
  row.MouseButton1Click:Connect(function()
    if not Unlocked then return end
    Toggles[key]=not Toggles[key]; local on=Toggles[key]
    tween(sw,{BackgroundColor3=on and ACCENT or Color3.fromRGB(40,40,55)},0.18)
    tween(knob,{Position=on and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)},0.18)
    fpsLabel.Visible=Toggles.ShowFPS
    if onChange then onChange(on) end
  end)
end
local function makeStepper(parent, getVal, setVal)
  local row=Instance.new("Frame",parent); row.Size=UDim2.new(1,0,0,40)
  row.BackgroundColor3=Color3.fromRGB(10,10,16); Instance.new("UICorner",row).CornerRadius=UDim.new(0,10)
  local minus=Instance.new("TextButton",row); minus.Size=UDim2.new(0,40,0,30); minus.Position=UDim2.new(0,6,0.5,-15)
  minus.Text="-"; minus.Font=Enum.Font.GothamBold; minus.TextSize=16
  minus.BackgroundColor3=ACCENT_DARK; minus.TextColor3=ACCENT; minus.AutoButtonColor=false
  Instance.new("UICorner",minus).CornerRadius=UDim.new(0,8)
  local plus=Instance.new("TextButton",row); plus.Size=UDim2.new(0,40,0,30); plus.Position=UDim2.new(1,-46,0.5,-15)
  plus.Text="+"; plus.Font=Enum.Font.GothamBold; plus.TextSize=16
  plus.BackgroundColor3=ACCENT_DARK; plus.TextColor3=ACCENT; plus.AutoButtonColor=false
  Instance.new("UICorner",plus).CornerRadius=UDim.new(0,8)
  local box=Instance.new("TextBox",row); box.Size=UDim2.new(1,-104,0,30); box.Position=UDim2.new(0,52,0.5,-15)
  box.Text=tostring(getVal()); box.Font=Enum.Font.GothamMedium; box.TextSize=13
  box.BackgroundColor3=Color3.fromRGB(0,0,0); box.TextColor3=Color3.new(1,1,1)
  Instance.new("UICorner",box).CornerRadius=UDim.new(0,8)
  minus.MouseButton1Click:Connect(function() if not Unlocked then return end setVal(getVal()-5); box.Text=tostring(getVal()) end)
  plus.MouseButton1Click:Connect(function() if not Unlocked then return end setVal(getVal()+5); box.Text=tostring(getVal()) end)
  box.FocusLost:Connect(function(e) if e and Unlocked then setVal(tonumber(box.Text) or getVal()); box.Text=tostring(getVal()) end end)
end

makeToggle(pageAim,"Aim Assist (RMB)","AimAssist")
makeToggle(pageAim,"Mostrar FOV","ShowFOV")
makeToggle(pageAim,"Check Visivel","VisibleCheck")
makeToggle(pageAim,"Ignorar Mortos","DeadCheck")
local fovLabel=Instance.new("TextLabel",pageAim)
fovLabel.Size=UDim2.new(1,0,0,16); fovLabel.BackgroundTransparency=1
fovLabel.TextXAlignment=Enum.TextXAlignment.Left; fovLabel.Text="FOV [50-1000]"
fovLabel.Font=Enum.Font.Gotham; fovLabel.TextSize=11; fovLabel.TextColor3=Color3.fromRGB(120,170,220)
makeStepper(pageAim, function() return FOV end, function(v) FOV=math.clamp(math.floor(v),FOV_MIN,FOV_MAX) end)
local statusLabel=Instance.new("TextLabel",pageAim)
statusLabel.Size=UDim2.new(1,0,0,40); statusLabel.BackgroundColor3=Color3.fromRGB(10,10,16)
statusLabel.TextXAlignment=Enum.TextXAlignment.Left; statusLabel.TextYAlignment=Enum.TextYAlignment.Top
statusLabel.Text="Status..."; statusLabel.Font=Enum.Font.Code; statusLabel.TextSize=11; statusLabel.TextColor3=Color3.new(1,1,1)
statusLabel.TextWrapped=true; Instance.new("UICorner",statusLabel).CornerRadius=UDim.new(0,8)
local testBtn=Instance.new("TextButton",pageAim)
testBtn.Size=UDim2.new(1,0,0,32); testBtn.Text="TESTE: olhar pra cima"; testBtn.Font=Enum.Font.GothamBold; testBtn.TextSize=12
testBtn.BackgroundColor3=ACCENT_DARK; testBtn.TextColor3=ACCENT; testBtn.AutoButtonColor=false
Instance.new("UICorner",testBtn).CornerRadius=UDim.new(0,8)
testBtn.MouseButton1Click:Connect(function()
  if not Unlocked then return end
  local cam=workspace.CurrentCamera
  if cam then cam.CFrame=CFrame.new(cam.CFrame.Position, cam.CFrame.Position+Vector3.new(0,50,0)) end
end)

makeToggle(pageVis,"ESP","ESP", function() refreshESP() end)
makeToggle(pageMisc,"Mostrar FPS","ShowFPS")

local hidden=false; local rmbHeld=false
local function setVisible(v)
  if not Unlocked then return end
  hidden=not v
  if v then main.Visible=true; circle.Visible=Toggles.ShowFOV and Toggles.AimAssist; fpsLabel.Visible=Toggles.ShowFPS; tween(scale,{Scale=1},0.18)
  else local tw=TweenService:Create(scale,TweenInfo.new(0.15),{Scale=0.9}); tw:Play()
    tw.Completed:Connect(function() if hidden then main.Visible=false end end)
    circle.Visible=false; fpsLabel.Visible=false end
end
minBtn.MouseButton1Click:Connect(function() setVisible(false) end)
UserInputService.InputBegan:Connect(function(input,gpe)
  if not gpe and input.KeyCode==Enum.KeyCode.RightShift and Unlocked then setVisible(hidden) end
  if input.UserInputType==Enum.UserInputType.MouseButton2 then rmbHeld=true end
end)
UserInputService.InputEnded:Connect(function(input)
  if input.UserInputType==Enum.UserInputType.MouseButton2 then rmbHeld=false end
end)

local function hookPlayer(p)
  p.CharacterAdded:Connect(function(c) c:WaitForChild("Humanoid",5); task.wait(0.5); applyESP(c) end)
  if p.Character then applyESP(p.Character) end
end
for _,p in pairs(Players:GetPlayers()) do hookPlayer(p) end
Players.PlayerAdded:Connect(hookPlayer)
workspace.DescendantAdded:Connect(function(d)
  if d:IsA("Humanoid") then
    local m=d.Parent
    if m and m:IsA("Model") and m~=LocalPlayer.Character and not Players:GetPlayerFromCharacter(m) then
      task.wait(0.5); applyESP(m)
    end
  end
end)

local function getClosest(cam)
  local ctr=cam.ViewportSize/2
  local cands={}
  for _,c in pairs(getAllTargets()) do
    if Toggles.DeadCheck and not isAlive(c) then continue end
    local p=getAimPart(c); if not p then continue end
    local pos,ok=cam:WorldToViewportPoint(p.Position); if not ok then continue end
    local d=(Vector2.new(pos.X,pos.Y)-ctr).Magnitude
    if d<FOV then table.insert(cands,{part=p,model=c,dist=d}) end
  end
  table.sort(cands,function(a,b) return a.dist<b.dist end)
  for _,e in pairs(cands) do
    if Toggles.VisibleCheck and not isVisible(cam,e.part,e.model) then continue end
    return e.part
  end
  return nil
end

local frames=0; local lastFps=tick()
RunService.RenderStepped:Connect(function()
  if not Unlocked then return end
  frames+=1
  if tick()-lastFps>=0.5 then
    if Toggles.ShowFPS then fpsLabel.Text="FPS: "..math.floor(frames/(tick()-lastFps)) end
    frames=0; lastFps=tick()
  end
  circle.Size=UDim2.fromOffset(FOV*2,FOV*2)
  if not hidden then circle.Visible=Toggles.ShowFOV and Toggles.AimAssist end
end)

RunService:BindToRenderStep("LZ_Aim", Enum.RenderPriority.Last.Value, function()
  if not Unlocked then return end
  local cam = workspace.CurrentCamera
  if not cam then return end
  local holding = rmbHeld or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
  if not (Toggles.AimAssist and holding) then
    statusLabel.Text = Toggles.AimAssist and "Aim ON | segure RMB" or "Aim OFF"
    return
  end
  local target=getClosest(cam)
  if not target then statusLabel.Text="RMB ON | sem alvo visivel/vivo"; return end
  statusLabel.Text="MIRANDO "..target:GetFullName()
  local goal = CFrame.lookAt(cam.CFrame.Position, target.Position)
  cam.CFrame = goal
  cam.Focus = CFrame.new(target.Position)
end)

-- TELA DE KEY --
local keyFrame=Instance.new("Frame",gui)
keyFrame.Size=UDim2.new(0,300,0,200); keyFrame.Position=UDim2.new(0.5,-150,0.5,-100)
keyFrame.BackgroundColor3=Color3.fromRGB(0,0,0); keyFrame.BorderSizePixel=0
Instance.new("UICorner",keyFrame).CornerRadius=UDim.new(0,12)
local ks=Instance.new("UIStroke",keyFrame); ks.Color=ACCENT
local keyTitle=Instance.new("TextLabel",keyFrame)
keyTitle.Size=UDim2.new(1,0,0,40); keyTitle.BackgroundTransparency=1
keyTitle.Text="LZ MENU • KEY"; keyTitle.Font=Enum.Font.GothamBold; keyTitle.TextSize=15; keyTitle.TextColor3=ACCENT
local keyBox=Instance.new("TextBox",keyFrame)
keyBox.Size=UDim2.new(1,-30,0,40); keyBox.Position=UDim2.new(0,15,0,55)
keyBox.PlaceholderText="Digite sua key..."; keyBox.Text=""
keyBox.Font=Enum.Font.Gotham; keyBox.TextSize=13
keyBox.BackgroundColor3=Color3.fromRGB(12,12,18); keyBox.TextColor3=Color3.new(1,1,1)
Instance.new("UICorner",keyBox).CornerRadius=UDim.new(0,8)
local keyBtn=Instance.new("TextButton",keyFrame)
keyBtn.Size=UDim2.new(1,-30,0,40); keyBtn.Position=UDim2.new(0,15,0,105)
keyBtn.Text="DESBLOQUEAR"; keyBtn.Font=Enum.Font.GothamBold; keyBtn.TextSize=13
keyBtn.BackgroundColor3=ACCENT_DARK; keyBtn.TextColor3=ACCENT; keyBtn.AutoButtonColor=false
Instance.new("UICorner",keyBtn).CornerRadius=UDim.new(0,8)
local keyMsg=Instance.new("TextLabel",keyFrame)
keyMsg.Size=UDim2.new(1,-30,0,20); keyMsg.Position=UDim2.new(0,15,0,150)
keyMsg.BackgroundTransparency=1; keyMsg.Text=""; keyMsg.Font=Enum.Font.Gotham; keyMsg.TextSize=11
keyMsg.TextColor3=Color3.fromRGB(255,90,90)

local function unlock()
  Unlocked=true
  keyFrame:Destroy()
  main.Visible=true
  tween(scale,{Scale=1},0.25)
  refreshESP()
end
keyBtn.MouseButton1Click:Connect(function()
  if VALID_KEYS[keyBox.Text] then unlock()
  else
    keyMsg.Text="Key inválida!"
    tween(keyFrame,{Position=keyFrame.Position+UDim2.new(0,6,0,0)},0.05)
    task.wait(0.05)
    tween(keyFrame,{Position=UDim2.new(0.5,-150,0.5,-100)},0.1)
  end
end)
keyBox.FocusLost:Connect(function(e) if e and VALID_KEYS[keyBox.Text] then unlock() end end)
if ALLOWED_USERIDS[LocalPlayer.UserId] then unlock() end
