--========================================================
-- TERENGG666 CONTROL PANEL (COMPACT + ESP)
-- F = FLY | R = RUN | T = ล็อกเป้า | E = ESP | RightCtrl = ซ่อน/แสดงแผง (เปลี่ยนได้ใน SETTINGS)
-- FLY: บินตามทิศกล้อง | Space = ขึ้น | LeftCtrl = ลง
--========================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local ContentProvider = game:GetService("ContentProvider")
local Teams = game:GetService("Teams")

local LP = Players.LocalPlayer
local PG = LP:WaitForChild("PlayerGui")

local function GetCamera() return workspace.CurrentCamera end

-- รูป (Asset ID)
local IMG = {
	plus = "rbxassetid://103548309642597",
	set = "rbxassetid://129894178799203",
	fly = "rbxassetid://112912992475655",
	run = "rbxassetid://130535042170633",
	tgt = "rbxassetid://91033212456133",
	color = "rbxassetid://86637059495430",
	esp = "rbxassetid://89202919617003",
}

--========================================================
-- SETTINGS
--========================================================

local FLY_SPEED, RUN_SPEED, MAX_DIST = 80, 16, 150
local FlyOn, RunOn, TargetOn = false, false, false
local DamageNumbersOn = true
local TargetNPCOn = false -- ปิดการล็อก NPC/Dummy
local Mode = "ALL"
local TeamCheck = true
local WALL_CHECK = true
local SelectedTargets = {}

local Keys = {
	fly = Enum.KeyCode.F, run = Enum.KeyCode.R,
	target = Enum.KeyCode.T, esp = Enum.KeyCode.E, panel = Enum.KeyCode.RightControl,
}
local KeyLabels = {
	fly = "FLY  (บิน)", run = "RUN SPEED  (วิ่งเร็ว)",
	target = "TARGET LOCK  (ล็อกเป้า)", esp = "ESP  (มองเห็นผู้เล่น)", panel = "ซ่อน / แสดงแผง",
}
local KeyOrder = {"fly","run","target","esp","panel"}

local WaitingFor, OnKeySet, OnStateChanged
local SetFly, SetRun, SetTarget, SetEsp, EspPage, SetItemEsp

--========================================================
-- COLORS
--========================================================

local BLACK = Color3.fromRGB(15,15,18)
local DARK = Color3.fromRGB(25,25,30)
local DARK3 = Color3.fromRGB(45,45,54)
local WHITE = Color3.new(1,1,1)
local GREEN = Color3.fromRGB(50,200,90)
local RED = Color3.fromRGB(220,60,60)
local YELLOW = Color3.fromRGB(245,200,55)
local PURPLE = Color3.fromRGB(160,70,255)
local ACCENT = Color3.fromRGB(255,50,50)

for _,n in ipairs({"TERENGG666_CONTROL","TERENGG666_LOADING"}) do
	local old = PG:FindFirstChild(n)
	if old then old:Destroy() end
end

--========================================================
-- HELPERS
--========================================================

local function New(class,props,parent)
	local o = Instance.new(class)
	for k,v in pairs(props) do o[k] = v end
	o.Parent = parent
	return o
end

local function Round(o,r)
	New("UICorner",{CornerRadius = (r == "full") and UDim.new(1,0) or UDim.new(0,r or 8)},o)
end

local function Stroke(o,c,t,tr)
	return New("UIStroke",{Color = c or WHITE,Thickness = t or 1,Transparency = tr or 0},o)
end

--========================================================
-- LOADING SCREEN (ท้องฟ้ากลางคืน + ดาวตก + ปุ่มข้าม)
--========================================================

-- Run the optional loading screen safely so an unsupported loading-screen
-- property cannot stop the main control panel from being created.
local loadingOk, loadingErr = pcall(function()
	local rng, running = Random.new(), true

	local lg = New("ScreenGui",{
		Name = "TERENGG666_LOADING",IgnoreGuiInset = true,ScreenInsets = Enum.ScreenInsets.None,
		ResetOnSpawn = false,DisplayOrder = 10000,
	},PG)

	local sky = New("Frame",{
		AnchorPoint = Vector2.new(0.5,0.5),Position = UDim2.fromScale(0.5,0.5),
		Size = UDim2.new(1,600,1,600),BackgroundColor3 = WHITE,BorderSizePixel = 0,
	},lg)
	New("UIGradient",{Rotation = 90,Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0,Color3.fromRGB(8,12,40)),
		ColorSequenceKeypoint.new(0.6,Color3.fromRGB(20,28,80)),
		ColorSequenceKeypoint.new(1,Color3.fromRGB(60,40,120)),
	})},sky)

	local bg = New("CanvasGroup",{
		Size = UDim2.fromScale(1,1),BackgroundTransparency = 1,BorderSizePixel = 0,
		ClipsDescendants = false,ZIndex = 1,
	},lg)

	local moon = New("Frame",{
		AnchorPoint = Vector2.new(1,0),Position = UDim2.new(1,-60,0,50),Size = UDim2.fromOffset(46,46),
		BackgroundColor3 = Color3.fromRGB(235,238,255),BorderSizePixel = 0,ZIndex = 2,
	},bg)
	Round(moon,"full")
	Stroke(moon,Color3.fromRGB(150,170,255),6,0.8)

	local twinkle = {}
	for _ = 1,90 do
		local sz = rng:NextInteger(1,3)
		local s = New("Frame",{
			Position = UDim2.fromScale(rng:NextNumber(),rng:NextNumber(0,0.9)),Size = UDim2.fromOffset(sz,sz),
			BackgroundColor3 = Color3.fromRGB(215,225,255),BackgroundTransparency = rng:NextNumber(0.1,0.8),
			BorderSizePixel = 0,ZIndex = 1,
		},bg)
		Round(s,"full")
		table.insert(twinkle,s)
	end

	task.spawn(function()
		while running do
			local s = twinkle[rng:NextInteger(1,#twinkle)]
			TweenService:Create(s,TweenInfo.new(rng:NextNumber(0.4,1)),{BackgroundTransparency = rng:NextNumber(0.05,0.9)}):Play()
			task.wait(0.05)
		end
	end)

	local tx = New("TextLabel",{
		AnchorPoint = Vector2.new(0.5,0.5),Position = UDim2.fromScale(0.5,0.45),Size = UDim2.fromOffset(400,40),
		BackgroundTransparency = 1,Text = "TERENGG666 | Loading... 0%",TextColor3 = WHITE,
		TextSize = 22,Font = Enum.Font.GothamBold,ZIndex = 5,
	},bg)

	local bb = New("Frame",{
		AnchorPoint = Vector2.new(0.5,0.5),Position = UDim2.fromScale(0.5,0.55),Size = UDim2.fromOffset(320,12),
		BackgroundColor3 = Color3.fromRGB(30,34,62),BorderSizePixel = 0,ZIndex = 5,
	},bg)
	Round(bb,"full")
	Stroke(bb,ACCENT,2,0.55)

	local bar = New("Frame",{Size = UDim2.fromScale(0,1),BackgroundColor3 = ACCENT,BorderSizePixel = 0,ZIndex = 6},bb)
	Round(bar,"full")

	local skipped, MAX_LOAD = false, 6   -- โหลดนานสุดกี่วินาที

	local skipBtn = New("TextButton",{
		AnchorPoint = Vector2.new(0.5,0.5),Position = UDim2.fromScale(0.5,0.65),Size = UDim2.fromOffset(150,40),
		BackgroundColor3 = Color3.fromRGB(30,34,62),BorderSizePixel = 0,Text = "ข้าม  ▶",TextColor3 = WHITE,
		TextSize = 16,Font = Enum.Font.GothamBold,AutoButtonColor = false,ZIndex = 7,
	},bg)
	Round(skipBtn,10)
	Stroke(skipBtn,ACCENT,2)
	skipBtn.MouseButton1Click:Connect(function() skipped = true end)

	local starColors = {
		Color3.fromRGB(255,80,80),Color3.fromRGB(255,160,40),Color3.fromRGB(255,225,60),Color3.fromRGB(70,230,120),
		Color3.fromRGB(60,200,255),Color3.fromRGB(80,120,255),Color3.fromRGB(190,90,255),Color3.fromRGB(255,100,200),
	}

	task.spawn(function()
		while running do
			local cam = GetCamera()
			if not cam then task.wait(0.1) continue end

			local vp = cam.ViewportSize
			local color = starColors[rng:NextInteger(1,#starColors)]
			local thick = rng:NextInteger(2,4)
			local startX = rng:NextNumber(-0.1,1.0) * vp.X
			local dist = (vp.X + vp.Y) * 0.9

			local star = New("Frame",{
				AnchorPoint = Vector2.new(1,0.5),Size = UDim2.fromOffset(rng:NextInteger(90,200),thick),
				Position = UDim2.fromOffset(startX,-30),Rotation = 45,BackgroundColor3 = color,
				BorderSizePixel = 0,ZIndex = 3,
			},bg)
			Round(star,"full")
			New("UIGradient",{Transparency = NumberSequence.new({
				NumberSequenceKeypoint.new(0,1),NumberSequenceKeypoint.new(1,0),
			})},star)

			local head = New("Frame",{
				AnchorPoint = Vector2.new(0.5,0.5),Position = UDim2.fromScale(1,0.5),
				Size = UDim2.fromOffset(thick * 3,thick * 3),BackgroundColor3 = WHITE,BorderSizePixel = 0,ZIndex = 4,
			},star)
			Round(head,"full")
			Stroke(head,color,2,0.3)

			local tw = TweenService:Create(star,TweenInfo.new(rng:NextNumber(0.8,1.5),Enum.EasingStyle.Linear),{
				Position = UDim2.fromOffset(startX + dist * 0.7071,-30 + dist * 0.7071),
			})
			tw:Play()
			tw.Completed:Once(function() star:Destroy() end)

			task.wait(rng:NextNumber(0.08,0.18))
		end
	end)

	local done, pct = false, 0

	task.spawn(function()
		if not game:IsLoaded() then game.Loaded:Wait() end

		local assets = workspace:GetDescendants()
		local total = math.max(#assets,1)

		for i = 1,total,100 do
			if skipped then break end

			local batch = {}
			for j = i,math.min(i + 99,total) do table.insert(batch,assets[j]) end

			pcall(function() ContentProvider:PreloadAsync(batch) end)
			pct = math.min(i + 99,total) / total
		end

		done = true
	end)

	local t0 = os.clock()

	while not done and not skipped and os.clock() - t0 < MAX_LOAD do
		TweenService:Create(bar,TweenInfo.new(0.1),{Size = UDim2.fromScale(pct,1)}):Play()
		tx.Text = "TERENGG666 | Loading... " .. math.floor(pct * 100) .. "%"
		task.wait(0.05)
	end

	skipped = true
	bar.Size = UDim2.fromScale(1,1)
	tx.Text = "TERENGG666 | Ready"
	running = false
	task.wait(0.4)

	TweenService:Create(bg,TweenInfo.new(0.6),{GroupTransparency = 1}):Play()
	TweenService:Create(sky,TweenInfo.new(0.6),{BackgroundTransparency = 1}):Play()
	task.wait(0.65)
	lg:Destroy()
end)

if not loadingOk then
	warn("TERENGG666 loading screen error: " .. tostring(loadingErr))
	local failedLoadingGui = PG:FindFirstChild("TERENGG666_LOADING")
	if failedLoadingGui then
		failedLoadingGui:Destroy()
	end
end

--========================================================
-- MAIN GUI
--========================================================

local Gui = New("ScreenGui",{
	Name = "TERENGG666_CONTROL",ResetOnSpawn = false,IgnoreGuiInset = true, Enabled = true,
	DisplayOrder = 9999,ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
},PG)

local Cross = New("ImageLabel",{
	AnchorPoint = Vector2.new(0.5,0.5),Position = UDim2.fromScale(0.5,0.5),Size = UDim2.fromOffset(80,45),
	BackgroundTransparency = 1,Image = "rbxassetid://105732453868721",
	ScaleType = Enum.ScaleType.Fit,Visible = false,ZIndex = 10000,
},Gui)

local TargetName = New("TextLabel",{
	AnchorPoint = Vector2.new(0.5,0),Position = UDim2.new(0.5,0,0.5,26),Size = UDim2.fromOffset(360,44),
	BackgroundTransparency = 1,Text = "",TextColor3 = WHITE,TextSize = 16,TextWrapped = true,
	Font = Enum.Font.GothamBold,Visible = false,ZIndex = 10000,
},Gui)
Stroke(TargetName,BLACK,2)

local Main = New("Frame",{
	Size = UDim2.fromOffset(470,400),Position = UDim2.new(0.5,-235,0.5,-200),BackgroundColor3 = BLACK,
	BorderSizePixel = 0,Active = true,ClipsDescendants = true,
},Gui)
Round(Main,12)
Stroke(Main,WHITE,2,0.15)

local Header = New("Frame",{Size = UDim2.new(1,0,0,48),BackgroundColor3 = DARK,BorderSizePixel = 0},Main)
Round(Header,12)

New("TextLabel",{
	Size = UDim2.new(1,-110,1,0),Position = UDim2.fromOffset(14,0),BackgroundTransparency = 1,
	Text = "TERENGG666  |  CONTROL PANEL",TextColor3 = WHITE,TextSize = 17,Font = Enum.Font.GothamBold,
	TextXAlignment = Enum.TextXAlignment.Left,TextTruncate = Enum.TextTruncate.AtEnd,
},Header)

local MinBtn = New("TextButton",{
	Size = UDim2.fromOffset(42,38),Position = UDim2.new(1,-47,0,5),BackgroundColor3 = DARK3,Text = "−",
	TextColor3 = WHITE,TextSize = 24,Font = Enum.Font.GothamBold,AutoButtonColor = false,
},Header)
Round(MinBtn)

local Side = New("Frame",{
	Size = UDim2.new(0,135,1,-60),Position = UDim2.fromOffset(8,54),BackgroundColor3 = DARK,BorderSizePixel = 0,
},Main)
Round(Side,10)
Stroke(Side,WHITE,1,0.5)

local Content = New("Frame",{
	Size = UDim2.new(1,-151,1,-60),Position = UDim2.fromOffset(143,54),BackgroundColor3 = DARK,
	BorderSizePixel = 0,ClipsDescendants = true,
},Main)
Round(Content,10)
Stroke(Content,WHITE,1,0.5)

--========================================================
-- ปุ่มลัดบนจอ (มือถือ) - เปิดปิดได้ใน SETTINGS
--========================================================

local QuickOn = UIS.TouchEnabled

local Quick = New("Frame",{
	AnchorPoint = Vector2.new(0,0.5),Position = UDim2.new(0,10,0.5,0),Size = UDim2.fromOffset(52,172),
	BackgroundTransparency = 1,Visible = QuickOn,ZIndex = 9000,
},Gui)
New("UIListLayout",{Padding = UDim.new(0,8),SortOrder = Enum.SortOrder.LayoutOrder},Quick)

local QuickItems = {}
local QuickVisible = {fly = true, run = true, target = true, esp = true}

local function MakeQuick(id,image,order,getState,action)
	local b = New("ImageButton",{
		Name = "Quick_" .. id, Size = UDim2.fromOffset(52,52),BackgroundColor3 = BLACK,Image = image,ScaleType = Enum.ScaleType.Fit,
		AutoButtonColor = false,LayoutOrder = order,ZIndex = 9000,Visible = QuickVisible[id],
	},Quick)
	Round(b,10)
	b.MouseButton1Click:Connect(action)
	table.insert(QuickItems,{id = id, button = b, stroke = Stroke(b,RED,3),get = getState})
end

local function RefreshQuick()
	for _,q in ipairs(QuickItems) do
		q.stroke.Color = q.get() and GREEN or RED
		q.button.Visible = QuickOn and QuickVisible[q.id]
	end
end

local function SetQuickVisible(id,state)
	QuickVisible[id] = state
	for _,q in ipairs(QuickItems) do
		if q.id == id then q.button.Visible = QuickOn and state break end
	end
end

local EspOn = false

MakeQuick("fly",IMG.fly,1,function() return FlyOn end,function() SetFly(not FlyOn) end)
MakeQuick("run",IMG.run,2,function() return RunOn end,function() SetRun(not RunOn) end)
MakeQuick("target",IMG.tgt,3,function() return TargetOn end,function() SetTarget(not TargetOn) end)
MakeQuick("esp",IMG.esp,4,function() return EspOn end,function() SetEsp(not EspOn) end)

Quick.Size = UDim2.fromOffset(52,232)

--========================================================
-- ลากหน้าต่าง / ปุ่มย่อ (ตัวช่วยตัวเดียว)
--========================================================

local function Draggable(handles,target,pad,onTap)
	local dragging, moved, startIn, startPos = false, false

	for _,h in ipairs(handles) do
		h.InputBegan:Connect(function(i)
			if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
				dragging, moved, startIn, startPos = true, false, i.Position, target.Position

				i.Changed:Connect(function()
					if i.UserInputState == Enum.UserInputState.End then
						dragging = false
						if onTap and not moved then onTap() end
					end
				end)
			end
		end)
	end

	UIS.InputChanged:Connect(function(i)
		if not dragging then return end
		if i.UserInputType ~= Enum.UserInputType.MouseMovement and i.UserInputType ~= Enum.UserInputType.Touch then return end

		local d = i.Position - startIn
		if d.Magnitude > 6 then moved = true end
		if onTap and not moved then return end

		local cam = GetCamera()
		if not cam then return end

		local vp, size = cam.ViewportSize, target.AbsoluteSize
		local x = math.clamp(startPos.X.Offset + d.X,-vp.X / 2 + pad,vp.X / 2 - size.X - pad)
		local y = math.clamp(startPos.Y.Offset + d.Y,-vp.Y / 2 + pad,vp.Y / 2 - size.Y - pad)

		target.Position = UDim2.new(0.5,x,0.5,y)
	end)
end

Draggable({Header,Main,Side,Content},Main,10)

local PlusBtn = New("ImageButton",{
	Size = UDim2.fromOffset(56,56),BackgroundColor3 = Color3.fromRGB(22,22,22),Image = IMG.plus,
	ScaleType = Enum.ScaleType.Crop,AutoButtonColor = false,Visible = false,ZIndex = 10001,
},Gui)
Round(PlusBtn,6)
Stroke(PlusBtn,PURPLE,2)

local function Minimize()
	PlusBtn.Position = Main.Position
	Main.Visible = false
	PlusBtn.Visible = true
end

local function Restore()
	Main.Position = PlusBtn.Position
	PlusBtn.Visible = false
	Main.Visible = true
end

MinBtn.MouseButton1Click:Connect(Minimize)
Draggable({PlusBtn},PlusBtn,5,Restore)
-- Item display is integrated into the existing ESP card; no separate billboard/button.

--========================================================
-- HOTKEYS
--========================================================

UIS.InputBegan:Connect(function(i,gp)
	-- โหมดตั้งปุ่มใหม่ (Esc = ยกเลิก)
	if WaitingFor then
		if i.UserInputType == Enum.UserInputType.Keyboard then
			local k, msg = i.KeyCode, nil

			if k == Enum.KeyCode.Escape then
				msg = "ยกเลิกแล้ว"
			elseif k == Enum.KeyCode.Unknown then
				msg = "ปุ่มนี้ใช้ไม่ได้ ลองปุ่มอื่น"
			else
				local clash
				for name,key in pairs(Keys) do
					if name ~= WaitingFor and key == k then clash = name end
				end

				if clash then
					msg = "ปุ่ม " .. k.Name .. " ถูกใช้กับ " .. KeyLabels[clash] .. " แล้ว"
				else
					Keys[WaitingFor] = k
					msg = "ตั้งปุ่มเรียบร้อย : " .. k.Name
				end
			end

			WaitingFor = nil
			if OnKeySet then OnKeySet(msg) end
		end
		return
	end

	if gp or i.UserInputType ~= Enum.UserInputType.Keyboard then return end
	local k = i.KeyCode

	if k == Keys.target then SetTarget(not TargetOn)
	elseif k == Keys.fly then SetFly(not FlyOn)
	elseif k == Keys.run then SetRun(not RunOn)
	elseif k == Keys.esp then SetEsp(not EspOn)
	elseif k == Keys.panel then
		if PlusBtn.Visible then Restore() else Main.Visible = not Main.Visible end
	end
end)

--========================================================
-- UI HELPERS
--========================================================

local function Btn(parent,text,pos,size)
	local b = New("TextButton",{
		Size = size,Position = pos,BackgroundColor3 = DARK3,BorderSizePixel = 0,Text = text,
		TextColor3 = WHITE,TextSize = 14,Font = Enum.Font.GothamBold,AutoButtonColor = false,
	},parent)
	Round(b)
	Stroke(b,WHITE,1,0.55)
	return b
end

local function Label(parent,text,pos,size,ts)
	return New("TextLabel",{
		Size = size,Position = pos,BackgroundTransparency = 1,Text = text,TextColor3 = WHITE,
		TextSize = ts or 14,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
	},parent)
end

local function IconBtn(parent,image,text,pos,size)
	local b = Btn(parent,"",pos,size)
	New("ImageLabel",{Position = UDim2.new(0,8,0.5,-14),Size = UDim2.fromOffset(28,28),BackgroundTransparency = 1,Image = image},b)
	New("TextLabel",{
		Position = UDim2.fromOffset(40,0),Size = UDim2.new(1,-44,1,0),BackgroundTransparency = 1,Text = text,
		TextColor3 = WHITE,TextSize = 12,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
	},b)
	return b
end

local function IconHeading(parent,image,text)
	New("ImageLabel",{Position = UDim2.fromOffset(12,12),Size = UDim2.fromOffset(30,30),BackgroundTransparency = 1,Image = image},parent)
	Label(parent,text,UDim2.fromOffset(48,10),UDim2.new(1,-60,0,35),21)
end

local Conns = {}

local function Clear()
	WaitingFor, OnKeySet, OnStateChanged = nil, nil, nil

	for _,c in ipairs(Conns) do pcall(function() c:Disconnect() end) end
	table.clear(Conns)

	for _,o in ipairs(Content:GetChildren()) do
		if not o:IsA("UICorner") and not o:IsA("UIStroke") then o:Destroy() end
	end
end

local function Visual(b,name,state)
	b.Text = name .. " : " .. (state and "ON" or "OFF")
	b.BackgroundColor3 = state and GREEN or RED
end

-- ช่องปรับค่า [−][ตัวเลข][+] : พิมพ์แล้วใช้ทันที / กดค้างเพื่อเพิ่ม-ลดต่อเนื่อง ปล่อยแล้วหยุด
local function Stepper(parent,y,get,set,step,mn,mx)
	local minus = Btn(parent,"−",UDim2.fromOffset(10,y),UDim2.fromOffset(45,38))
	local plus = Btn(parent,"+",UDim2.new(1,-55,0,y),UDim2.fromOffset(45,38))

	local box = New("TextBox",{
		Size = UDim2.new(1,-120,0,38),Position = UDim2.fromOffset(60,y),BackgroundColor3 = DARK3,TextColor3 = WHITE,
		Text = tostring(get()),PlaceholderText = mn .. "-" .. mx,TextSize = 16,Font = Enum.Font.GothamBold,ClearTextOnFocus = true,
	},parent)
	Round(box)
	Stroke(box,WHITE,1,0.5)

	local function apply(v)
		set(math.clamp(math.floor(v),mn,mx))
		box.Text = tostring(get())
	end

	box:GetPropertyChangedSignal("Text"):Connect(function()
		if not box:IsFocused() then return end

		local clean = box.Text:gsub("[^%d]","")
		if clean ~= box.Text then box.Text = clean return end

		local n = tonumber(clean)
		if n and n >= mn and n <= mx then set(n) end
	end)

	box.FocusLost:Connect(function()
		local n = tonumber(box.Text)
		if n then apply(n) else box.Text = tostring(get()) end
	end)

	local holdToken

	local function bindHold(btn,delta)
		btn.InputBegan:Connect(function(i)
			if i.UserInputType ~= Enum.UserInputType.MouseButton1 and i.UserInputType ~= Enum.UserInputType.Touch then return end

			apply(get() + delta)

			local token = {}
			holdToken = token

			i.Changed:Connect(function()
				if i.UserInputState == Enum.UserInputState.End and holdToken == token then holdToken = nil end
			end)

			task.delay(0.4,function()
				while holdToken == token and btn.Parent do
					apply(get() + delta)
					task.wait(0.05)
				end
			end)
		end)
	end

	bindHold(minus,-step)
	bindHold(plus,step)

	table.insert(Conns,UIS.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
			holdToken = nil
		end
	end))
end

--========================================================
-- SIDE MENU
--========================================================

local FlyMenu = IconBtn(Side,IMG.fly,"FLY",UDim2.fromOffset(8,12),UDim2.new(1,-16,0,55))
local RunMenu = IconBtn(Side,IMG.run,"RUN SPEED",UDim2.fromOffset(8,77),UDim2.new(1,-16,0,55))
local TgtMenu = IconBtn(Side,IMG.tgt,"TARGET LOCK",UDim2.fromOffset(8,142),UDim2.new(1,-16,0,55))
local EspMenu = IconBtn(Side,IMG.esp,"ESP",UDim2.fromOffset(8,207),UDim2.new(1,-16,0,55))
local ScriptMenu = IconBtn(Side,IMG.set,"SCRIPT EDITOR",UDim2.fromOffset(8,272),UDim2.new(1,-16,0,55))

local LastActive, InSettings = FlyMenu, false

local function Highlight(active)
	if active then LastActive, InSettings = active, false else InSettings = true end

	for _,b in ipairs({FlyMenu,RunMenu,TgtMenu,EspMenu,ScriptMenu}) do
		local s = b:FindFirstChildOfClass("UIStroke")
		if s then
			local on = (b == active)
			s.Color = on and ACCENT or WHITE
			s.Transparency = on and 0 or 0.55
			s.Thickness = on and 2 or 1
		end
	end
end

--========================================================
-- CHARACTER
--========================================================

local function Hum() return LP.Character and LP.Character:FindFirstChildOfClass("Humanoid") end
local function Root() return LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") end

--========================================================
-- FLY (ท่าซูเปอร์แมน R15 + R6 / ท่าลงพื้น) ไม่ใช้ ID แอนิเมชัน
--========================================================

local FV, FG

local IDLE_LEAN = math.rad(20)   -- เอนตัวตอนลอยนิ่ง (0 = ยืนตรง)
local DASH_BURST = 1.6           -- แรงพุ่งช่วงแรก (1 = ไม่พุ่ง)
local BURST_TIME = 0.3
local POSE_SPEED = 0.18

local POSE = {
	idle = {arm = math.rad(35),spread = math.rad(14),legR = math.rad(8),legL = math.rad(26),knee = math.rad(28),head = math.rad(12),drop = 0},
	dash = {arm = math.rad(172),spread = math.rad(6),legR = math.rad(4),legL = math.rad(4),knee = 0,head = math.rad(40),drop = 0},
	land = {arm = math.rad(45),spread = math.rad(45),legR = math.rad(-62),legL = math.rad(-62),knee = math.rad(80),head = math.rad(-5),drop = 0.7},
	stand = {arm = 0,spread = 0,legR = 0,legL = 0,knee = 0,head = 0,drop = 0},
}

local JOINT_R15 = {rs = "RightShoulder",ls = "LeftShoulder",rh = "RightHip",lh = "LeftHip",rk = "RightKnee",lk = "LeftKnee",neck = "Neck",root = "Root"}
local JOINT_R6 = {rs = "Right Shoulder",ls = "Left Shoulder",rh = "Right Hip",lh = "Left Hip",neck = "Neck",root = "RootJoint"}

local Joints, OrigC0, CurOff = {}, {}, {}
local IsR6, PoseInfo = false, "-"
local landToken, LandConn = 0, nil
local Controls

task.spawn(function()
	local ok,res = pcall(function()
		return require(LP:WaitForChild("PlayerScripts"):WaitForChild("PlayerModule")):GetControls()
	end)
	if ok then Controls = res end
end)

-- x = ขวา, z = ถอยหลัง (เทียบกับกล้อง)
local function GetMoveInput(h)
	if Controls then
		local ok,v = pcall(function() return Controls:GetMoveVector() end)
		if ok and typeof(v) == "Vector3" then return Vector3.new(v.X,0,v.Z) end
	end

	local cam = GetCamera()
	if cam then
		local rel = cam.CFrame:VectorToObjectSpace(h.MoveDirection)
		return Vector3.new(rel.X,0,rel.Z)
	end

	return Vector3.zero
end

local function FindJoints()
	table.clear(Joints) table.clear(OrigC0) table.clear(CurOff)

	local c = LP.Character
	if not c then PoseInfo = "ไม่พบตัวละคร" return end

	local map = {}
	for _,m in ipairs(c:GetDescendants()) do
		if m:IsA("Motor6D") then map[m.Name] = m end
	end

	IsR6 = (map["Right Shoulder"] ~= nil)

	local count = 0
	for key,jointName in pairs(IsR6 and JOINT_R6 or JOINT_R15) do
		local m = map[jointName]
		if m then
			Joints[key], OrigC0[key], CurOff[key] = m, m.C0, CFrame.new()
			count += 1
		end
	end

	PoseInfo = (IsR6 and "R6" or "R15") .. " • พบข้อต่อ " .. count
end

local function ComputeTargets(p)
	local t = {}

	if IsR6 then
		t.rs = CFrame.Angles(0,0,p.arm) * CFrame.Angles(-p.spread,0,0)
		t.ls = CFrame.Angles(0,0,-p.arm) * CFrame.Angles(-p.spread,0,0)
		t.rh = CFrame.Angles(0,0,-p.legR)
		t.lh = CFrame.Angles(0,0,p.legL)
		t.neck = CFrame.Angles(-p.head,0,0)
		t.root = CFrame.new(0,0,-p.drop)
	else
		t.rs = CFrame.Angles(p.arm,0,0) * CFrame.Angles(0,0,p.spread)
		t.ls = CFrame.Angles(p.arm,0,0) * CFrame.Angles(0,0,-p.spread)
		t.rh = CFrame.Angles(-p.legR,0,0)
		t.lh = CFrame.Angles(-p.legL,0,0)
		t.rk = CFrame.Angles(-p.knee,0,0)
		t.lk = CFrame.Angles(-p.knee,0,0)
		t.neck = CFrame.Angles(p.head,0,0)
		t.root = CFrame.new(0,-p.drop,0)
	end

	return t
end

-- จัดท่าที่ C0 ของข้อต่อ จึงไม่ถูกแอนิเมชันของ Roblox เขียนทับ
local function ApplyPose(p,a)
	local t = ComputeTargets(p)

	for k,m in pairs(Joints) do
		if m.Parent and t[k] and OrigC0[k] then
			CurOff[k] = CurOff[k]:Lerp(t[k],a)
			m.C0 = OrigC0[k] * CurOff[k]
		end
	end
end

local function ResetPose()
	for k,m in pairs(Joints) do
		if m.Parent and OrigC0[k] then m.C0 = OrigC0[k] end
		CurOff[k] = CFrame.new()
	end
end

local function SetAnimate(on)
	local anim = LP.Character and LP.Character:FindFirstChild("Animate")
	if anim then pcall(function() anim.Enabled = on end) end
end

local function StopTracks()
	local h = Hum()
	local animator = h and h:FindFirstChildOfClass("Animator")
	if animator then
		for _,t in ipairs(animator:GetPlayingAnimationTracks()) do t:Stop(0.1) end
	end
end

local function FreezeAnims()
	SetAnimate(false)
	StopTracks()
end

local function PlayLanding(token)
	task.spawn(function()
		FreezeAnims()
		FindJoints()

		local function run(pose,dur,a)
			local t0 = os.clock()
			while os.clock() - t0 < dur do
				if landToken ~= token or FlyOn then return false end
				ApplyPose(pose,a)
				RunService.RenderStepped:Wait()
			end
			return true
		end

		if run(POSE.land,0.16,0.4) and run(POSE.land,0.12,0.5) and run(POSE.stand,0.22,0.35) then
			ResetPose()
			SetAnimate(true)
		end
	end)
end

local function WatchLanding()
	if LandConn then LandConn:Disconnect() LandConn = nil end

	local h, r = Hum(), Root()
	if not h or not r then return end

	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = {LP.Character}

	-- อยู่ใกล้พื้นอยู่แล้ว ไม่ต้องทำท่าลงพื้น
	if workspace:Raycast(r.Position,Vector3.new(0,-9,0),params) then return end

	local token = landToken

	LandConn = h.StateChanged:Connect(function(_,new)
		if new == Enum.HumanoidStateType.Landed or new == Enum.HumanoidStateType.Running then
			if LandConn then LandConn:Disconnect() LandConn = nil end
			if landToken == token and not FlyOn then PlayLanding(token) end
		end
	end)

	task.delay(8,function()
		if LandConn and landToken == token then LandConn:Disconnect() LandConn = nil end
	end)
end

local function StopFly(land)
	landToken += 1

	local r, h, cam = Root(), Hum(), GetCamera()

	if FV then FV:Destroy() FV = nil end
	if FG then FG:Destroy() FG = nil end

	ResetPose()
	table.clear(Joints) table.clear(OrigC0) table.clear(CurOff)
	SetAnimate(true)

	if h then
		h.PlatformStand = false
		h.AutoRotate = true
		if h.Health > 0 then h:ChangeState(Enum.HumanoidStateType.GettingUp) end
	end

	if r and cam then
		local look = cam.CFrame.LookVector
		local flat = Vector3.new(look.X,0,look.Z)
		if flat.Magnitude > 0.01 then r.CFrame = CFrame.lookAt(r.Position,r.Position + flat) end
	end

	if land then WatchLanding() end
end

local function StartFly()
	local r, h = Root(), Hum()
	if not r or not h then return end

	StopFly()
	FreezeAnims()
	FindJoints()

	h.AutoRotate = false
	h.PlatformStand = true

	FV = New("BodyVelocity",{Name = "TerenggFlyVelocity",MaxForce = Vector3.new(1e6,1e6,1e6),P = 10000,Velocity = Vector3.zero},r)
	FG = New("BodyGyro",{Name = "TerenggFlyGyro",MaxTorque = Vector3.new(1e6,1e6,1e6),P = 12000,D = 600,CFrame = r.CFrame},r)
end

local lastMoving, burstUntil, bobT, animT = false, 0, 0, 0

RunService.RenderStepped:Connect(function(dt)
	if not FlyOn then return end

	if not FV or not FG or not FV.Parent or not FG.Parent then
		StartFly()
		return
	end

	local h, r, cam = Hum(), Root(), GetCamera()
	if not h or not r or not cam then return end

	animT += dt
	if animT > 0.4 then animT = 0 StopTracks() end

	local dir = Vector3.zero
	local mv = GetMoveInput(h)

	if mv.Magnitude > 0.05 then
		dir = cam.CFrame.RightVector * mv.X + cam.CFrame.LookVector * -mv.Z
	end
	if UIS:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
	if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end

	local moving = dir.Magnitude > 0.05

	if moving and not lastMoving then burstUntil = os.clock() + BURST_TIME end
	lastMoving = moving

	if moving then
		local speed = os.clock() < burstUntil and FLY_SPEED * DASH_BURST or FLY_SPEED
		FV.Velocity = FV.Velocity:Lerp(dir.Unit * speed,0.35)
	else
		bobT += dt
		FV.Velocity = FV.Velocity:Lerp(Vector3.new(0,math.sin(bobT * 2) * 1.5,0),0.2)
	end

	local look = cam.CFrame.LookVector
	local heading = Vector3.new(look.X,0,look.Z)

	if heading.Magnitude < 0.05 then
		local rl = r.CFrame.LookVector
		heading = Vector3.new(rl.X,0,rl.Z)
	end
	if heading.Magnitude < 0.01 then heading = Vector3.new(0,0,-1) end

	local theta = -IDLE_LEAN

	if moving then
		local d = dir.Unit
		local flatD = Vector3.new(d.X,0,d.Z)
		if flatD.Magnitude > 0.05 then heading = flatD.Unit end
		theta = math.asin(math.clamp(d.Y,-1,1)) - math.pi / 2
	end

	FG.CFrame = FG.CFrame:Lerp(CFrame.lookAt(r.Position,r.Position + heading.Unit) * CFrame.Angles(theta,0,0),0.25)

	ApplyPose(moving and POSE.dash or POSE.idle,POSE_SPEED)
end)

--========================================================
-- RUN SPEED
--========================================================

local function ApplyRun()
	local h = Hum()
	if h and RunOn then h.WalkSpeed = RUN_SPEED end
end

RunService.Heartbeat:Connect(ApplyRun)

--========================================================
-- TARGET FUNCTIONS (PLAYER + NPC / DUMMY)
--========================================================

local function Alive(p)
	local c = p.Character
	if not c then return false end
	local h = c:FindFirstChildOfClass("Humanoid")
	return h ~= nil and c:FindFirstChild("HumanoidRootPart") ~= nil and h.Health > 0
end

local function Valid(p,ignoreDist)
	if not p or p == LP or not p.Parent then return false end
	if TeamCheck and LP.Team ~= nil and p.Team == LP.Team then return false end
	if not Alive(p) then return false end
	if ignoreDist then return true end

	local mr = Root()
	local tr = p.Character:FindFirstChild("HumanoidRootPart")	
	return mr ~= nil and tr ~= nil and (tr.Position - mr.Position).Magnitude <= MAX_DIST
end

local function GetTargetModel(target)
	if typeof(target) ~= "Instance" then return nil end
	if target:IsA("Player") then return target.Character end
	if target:IsA("Model") then return target end
	return nil
end

local function GetTargetRoot(target)
	local model = GetTargetModel(target)
	return model and model:FindFirstChild("HumanoidRootPart") or nil
end

local function RefreshNPCModels()
	table.clear(NPCModels)
	local seen = {}
	local playerCharacters = {}
	for _,p in ipairs(Players:GetPlayers()) do
		if p.Character then table.insert(playerCharacters,p.Character) end
	end

	for _,obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("Model") and not seen[obj] and obj ~= LP.Character then
			local insidePlayer = false
			for _,character in ipairs(playerCharacters) do
				if obj == character or obj:IsDescendantOf(character) then
					insidePlayer = true
					break
				end
			end
			if not insidePlayer then
				local hum = obj:FindFirstChildOfClass("Humanoid")
				local root = obj:FindFirstChild("HumanoidRootPart")
				if hum and root and hum.Health > 0 then
					table.insert(NPCModels,obj)
					seen[obj] = true
				end
			end
		end
	end
end

local function ValidNPC(model,ignoreDist)
	if not model or not model:IsA("Model") or not model.Parent or not model:IsDescendantOf(workspace) then return false end
	if model == LP.Character then return false end
	local h = model:FindFirstChildOfClass("Humanoid")
	local tr = model:FindFirstChild("HumanoidRootPart")
	if not h or not tr or h.Health <= 0 then return false end
	if ignoreDist then return true end
	local mr = Root()
	return mr ~= nil and (tr.Position - mr.Position).Magnitude <= MAX_DIST
end

-- ไฮไลต์ผู้เล่นและ NPC แยกชุดกัน
local HL = {}        -- Player -> Highlight
local HLShown = {}   -- Player -> true ถ้ากำลังแสดงอยู่
local NPC_HL = {}    -- Model -> Highlight

local function SetHL(p,on)
	if on then
		local c = p.Character
		if not c then return end
		local h = HL[p]
		if not h or h.Parent ~= c then
			if h then h:Destroy() end
			h = New("Highlight",{
				Name = "TerenggTargetHighlight", Adornee = c,
				FillTransparency = 0.45, OutlineTransparency = 0,
				DepthMode = Enum.HighlightDepthMode.AlwaysOnTop, Enabled = false,
			},c)
			HL[p] = h
		end
		h.FillColor = PURPLE
		h.OutlineColor = PURPLE
		h.Enabled = true
		HLShown[p] = true
	else
		local h = HL[p]
		if h then h.Enabled = false end
		HLShown[p] = nil
	end
end

local function SetNPCHL(model,on)
	if on then
		if not model or not model.Parent then return end
		local h = NPC_HL[model]
		if not h or h.Parent ~= model then
			if h then h:Destroy() end
			h = New("Highlight",{
				Name = "TerenggNPCAndDummyHighlight", Adornee = model,
				FillTransparency = 0.48, OutlineTransparency = 0,
				DepthMode = Enum.HighlightDepthMode.AlwaysOnTop, Enabled = false,
			},model)
			NPC_HL[model] = h
		end
		h.FillColor = YELLOW
		h.OutlineColor = YELLOW
		h.Enabled = true
	else
		local h = NPC_HL[model]
		if h then
			if not model.Parent then h:Destroy() NPC_HL[model] = nil else h.Enabled = false end
		end
	end
end

local function AddHL(p) SetHL(p,true) end
local function RemoveHL(p) SetHL(p,false) end

local function ClearHL()
	for p in pairs(HL) do SetHL(p,false) end
	for model in pairs(NPC_HL) do SetNPCHL(model,false) end
end

-- เช็กกำแพงบังสายตา โดยไม่ให้ตัวละครอื่นมาบัง raycast
local LOSParams = RaycastParams.new()
LOSParams.FilterType = Enum.RaycastFilterType.Exclude
LOSParams.RespectCanCollide = true
LOSParams.IgnoreWater = true

local function CanSee(target)
	if not WALL_CHECK then return true end
	local myChar = LP.Character
	local tChar = GetTargetModel(target)
	if not myChar or not tChar then return false end
	local myHead = myChar:FindFirstChild("Head") or myChar:FindFirstChild("HumanoidRootPart")
	if not myHead then return false end

	local ignore = {}
	for _,pl in ipairs(Players:GetPlayers()) do
		if pl.Character and pl.Character ~= tChar then table.insert(ignore,pl.Character) end
	end
	LOSParams.FilterDescendantsInstances = ignore

	local origin = myHead.Position
	for _,name in ipairs({"HumanoidRootPart","Head"}) do
		local part = tChar:FindFirstChild(name)
		if part then
			local res = workspace:Raycast(origin,part.Position - origin,LOSParams)
			if not res or res.Instance:IsDescendantOf(tChar) then return true end
		end
	end
	return false
end

-- หาเป้าหมายที่ใกล้ที่สุด: ผู้เล่นตามโหมดเดิม และเพิ่ม NPC/Dummy เมื่อเปิดใน SETTINGS
local function CurrentTarget()
	local mr = Root()
	if not mr then return nil end
	local cands = {}

	if Mode == "ALL" then
		for _,p in ipairs(Players:GetPlayers()) do
			if Valid(p) then table.insert(cands,p) end
		end
	else
		for p in pairs(SelectedTargets) do
			if Valid(p,true) then table.insert(cands,p) end
		end
	end

	if TargetNPCOn then
		for _,model in ipairs(NPCModels) do
			if ValidNPC(model,false) then table.insert(cands,model) end
		end
	end

	local list = {}
	for _,target in ipairs(cands) do
		local tr = GetTargetRoot(target)
		if tr then table.insert(list,{target = target,d = (tr.Position - mr.Position).Magnitude}) end
	end
	table.sort(list,function(a,b) return a.d < b.d end)

	for _,entry in ipairs(list) do
		if CanSee(entry.target) then return entry.target end
	end
	return nil
end

local function UpdateSelectedHighlights()
	local mr = Root()
	local wantedPlayers, wantedNPCs = {}, {}

	if TargetOn and mr then
		for _,p in ipairs(Players:GetPlayers()) do
			if p ~= LP and Valid(p,true) then
				local tr = GetTargetRoot(p)
				if tr then
					local d = (tr.Position - mr.Position).Magnitude
					if Mode == "ALL" then
						local lim = HLShown[p] and MAX_DIST * 1.15 or MAX_DIST
						if d <= lim then table.insert(wantedPlayers,{target = p,d = d}) end
					elseif SelectedTargets[p] then
						table.insert(wantedPlayers,{target = p,d = d})
					end
				end
			end
		end

		if TargetNPCOn then
			for _,model in ipairs(NPCModels) do
				if ValidNPC(model,true) then
					local tr = GetTargetRoot(model)
					if tr then
						local d = (tr.Position - mr.Position).Magnitude
						local old = NPC_HL[model]
					local lim = (old and old.Enabled) and MAX_DIST * 1.15 or MAX_DIST
						if d <= lim then table.insert(wantedNPCs,{target = model,d = d}) end
					end
				end
			end
		end
	end

	table.sort(wantedPlayers,function(a,b) return a.d < b.d end)
	table.sort(wantedNPCs,function(a,b) return a.d < b.d end)

	local onPlayers = {}
	for i,entry in ipairs(wantedPlayers) do
		if i > 30 then break end
		onPlayers[entry.target] = true
		SetHL(entry.target,true)
	end
	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= LP and not onPlayers[p] then SetHL(p,false) end
	end

	local onNPCs = {}
	for i,entry in ipairs(wantedNPCs) do
		if i > 10 then break end
		onNPCs[entry.target] = true
		SetNPCHL(entry.target,true)
	end
	for model in pairs(NPC_HL) do
		if not onNPCs[model] then SetNPCHL(model,false) end
	end
end

local function NoTargetReason()
	local me = Root()
	local pool, others = {}, 0
	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= LP then
			others += 1
			if Mode == "ALL" or SelectedTargets[p] then table.insert(pool,p) end
		end
	end

	if others == 0 and not TargetNPCOn then return "ไม่มีผู้เล่นคนอื่นในเกม" end
	if Mode == "LIST" and #pool == 0 and not TargetNPCOn then return "ยังไม่ได้เลือกชื่อในรายชื่อ" end

	local alive, sameTeam, inRange, seen = 0, 0, 0, 0
	for _,p in ipairs(pool) do
		if TeamCheck and LP.Team ~= nil and p.Team == LP.Team then
			sameTeam += 1
		elseif Alive(p) then
			alive += 1
			local tr = GetTargetRoot(p)
			if tr and (Mode == "LIST" or (me and (tr.Position - me.Position).Magnitude <= MAX_DIST)) then
				inRange += 1
				if CanSee(p) then seen += 1 end
			end
		end
	end

	local npcAlive, npcRange, npcSeen = 0, 0, 0
	if TargetNPCOn then
		for _,model in ipairs(NPCModels) do
			if ValidNPC(model,true) then
				npcAlive += 1
				local tr = GetTargetRoot(model)
				if tr and me and (tr.Position - me.Position).Magnitude <= MAX_DIST then
					npcRange += 1
					if CanSee(model) then npcSeen += 1 end
				end
			end
		end
	end

	if seen > 0 or npcSeen > 0 then return "" end
	if alive == 0 and npcAlive == 0 then
		if sameTeam > 0 then return "ผู้เล่นที่พบอยู่ทีมเดียวกับคุณ" end
		return TargetNPCOn and "ไม่พบ NPC/Dummy ที่มี Humanoid และ HumanoidRootPart" or "เป้าหมายตายอยู่ / ยังไม่เกิด"
	end
	if inRange == 0 and npcRange == 0 then return "เป้าหมายอยู่นอกระยะ Range" end
	return WALL_CHECK and "มีกำแพงบังอยู่ (ปิดเช็กกำแพงได้ใน SETTINGS)" or "ไม่พบเป้าหมายที่ใช้งานได้"
end

local function UpdateTargetDisplay()
	if not TargetOn then
		Cross.Visible, TargetName.Visible, TargetName.Text = false, false, ""
		return
	end

	Cross.Visible = true
	local target = CurrentTarget()
	if target then
		TargetName.TextColor3 = WHITE
		if target:IsA("Player") then
			TargetName.Text = target.DisplayName .. "  (@" .. target.Name .. ")"
		else
			TargetName.Text = target.Name .. "  (NPC / Dummy)"
		end
	else
		local why = NoTargetReason()
		TargetName.TextColor3 = YELLOW
		TargetName.Text = "ไม่พบเป้าหมาย" .. (why ~= "" and (" : " .. why) or "")
	end
	TargetName.Visible = true
end

--========================================================
-- DAMAGE NUMBERS (ตัวเลขดาเมจลอยเหนือหัวทุกครั้งที่ HP ลด)
--========================================================

local DamagePopups = {}
local TrackedHumanoids = setmetatable({}, {__mode = "k"})

local function ClearDamagePopups()
	for _,g in ipairs(DamagePopups) do
		if g and g.Parent then g:Destroy() end
	end
	table.clear(DamagePopups)
end

local function ShowDamage(model,amount)
	if not DamageNumbersOn or not model or not model.Parent or amount <= 0.01 then return end
	local anchor = model:FindFirstChild("Head") or model:FindFirstChild("HumanoidRootPart")
	if not anchor then return end
	local amountText = math.floor(amount + 0.5)
	if amountText <= 0 then return end

	local gui = New("BillboardGui",{
		Name = "TERENGG666_DamagePopup",Adornee = anchor,AlwaysOnTop = true,
		Size = UDim2.fromOffset(110,42),StudsOffset = Vector3.new(math.random(-10,10)/10,2.5,0),
		LightInfluence = 0,MaxDistance = 500,Enabled = true,
	},PG)
	table.insert(DamagePopups,gui)

	local label = New("TextLabel",{
		Size = UDim2.fromScale(1,1),BackgroundTransparency = 1,Text = "-" .. tostring(amountText),
		TextColor3 = amountText >= 30 and RED or (amountText >= 10 and YELLOW or WHITE),
		TextTransparency = 0,TextSize = 24,Font = Enum.Font.GothamBlack,
		TextStrokeColor3 = BLACK,TextStrokeTransparency = 0.15,
	},gui)
	local stroke = New("UIStroke",{Color = BLACK,Thickness = 1.5,Transparency = 0.12},label)

	TweenService:Create(gui,TweenInfo.new(0.75,Enum.EasingStyle.Quad,Enum.EasingDirection.Out),{
		StudsOffset = gui.StudsOffset + Vector3.new(0,1.7,0),
	}):Play()
	TweenService:Create(label,TweenInfo.new(0.75,Enum.EasingStyle.Quad,Enum.EasingDirection.Out),{
		TextTransparency = 1,TextStrokeTransparency = 1,
	}):Play()
	TweenService:Create(stroke,TweenInfo.new(0.75),{Transparency = 1}):Play()

	task.delay(0.82,function()
		if gui and gui.Parent then gui:Destroy() end
		for i = #DamagePopups,1,-1 do
			if DamagePopups[i] == gui then table.remove(DamagePopups,i) break end
		end
	end)
end

local function TrackHumanoid(model,humanoid)
	if not humanoid or TrackedHumanoids[humanoid] then return end
	TrackedHumanoids[humanoid] = true
	local lastHealth = humanoid.Health
	humanoid.HealthChanged:Connect(function(newHealth)
		if newHealth < lastHealth then ShowDamage(model,lastHealth - newHealth) end
		lastHealth = newHealth
	end)
end

local function ScanDamageTargets()
	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= LP and p.Character then
			local h = p.Character:FindFirstChildOfClass("Humanoid")
			if h then TrackHumanoid(p.Character,h) end
		end
	end
end

ScanDamageTargets()
task.spawn(function()
	while Gui.Parent do
		task.wait(0.8)
		ScanDamageTargets()
	end
end)

--========================================================
-- ฟังก์ชันกลาง: ปุ่มในแผง / ปุ่มลัดคีย์บอร์ด / ปุ่มบนจอ ใช้อันเดียวกัน
--========================================================

local function Changed()
	RefreshQuick()
	if OnStateChanged then OnStateChanged() end
end

SetFly = function(state)
	FlyOn = state
	if FlyOn then StartFly() else StopFly(true) end
	Changed()
end

SetRun = function(state)
	RunOn = state

	if RunOn then
		ApplyRun()
	else
		local h = Hum()
		if h then h.WalkSpeed = 16 end
	end

	Changed()
end

SetTarget = function(state)
	TargetOn = state
	if not TargetOn then ClearHL() end   -- รายชื่อที่เลือกไว้ยังอยู่
	UpdateTargetDisplay()
	Changed()
end

--========================================================
-- TARGET LOOPS
--========================================================

task.spawn(function()
	while task.wait(0.25) do
		if TargetOn then
			for p in pairs(SelectedTargets) do
				if not Valid(p,true) then SelectedTargets[p] = nil end
			end
			UpdateSelectedHighlights()
			UpdateTargetDisplay()
		else
			ClearHL()
		end
	end
end)

-- ล็อกกล้อง/ตัวละคร ทำงานหลังกล้องของ Roblox
local wasLocking = false

RunService:BindToRenderStep("TerenggLock",Enum.RenderPriority.Camera.Value + 1,function()
	local mr, cam = Root(), GetCamera()
	local target = (TargetOn and mr and cam) and CurrentTarget() or nil
	local targetModel = target and GetTargetModel(target) or nil
	local tr = targetModel and targetModel:FindFirstChild("HumanoidRootPart")

	if not tr then
		if wasLocking then
			wasLocking = false
			local h = Hum()
			if h and not FlyOn then h.AutoRotate = true end
		end
		return
	end

	if not wasLocking then
		wasLocking = true
		local h = Hum()
		if h then h.AutoRotate = false end
	end

	local flat = Vector3.new(tr.Position.X,mr.Position.Y,tr.Position.Z)

	if not FlyOn and (flat - mr.Position).Magnitude > 0.1 then
		mr.CFrame = CFrame.lookAt(mr.Position,flat)
	end

	cam.CFrame = CFrame.lookAt(cam.CFrame.Position,tr.Position)
end)

Players.PlayerRemoving:Connect(function(p)
	SelectedTargets[p] = nil

	if HL[p] then HL[p]:Destroy() end
	HL[p], HLShown[p] = nil, nil

	task.defer(UpdateTargetDisplay)
end)

LP.CharacterAdded:Connect(function()
	StopFly()
	task.wait(0.5)
	if FlyOn then StartFly() end
	ApplyRun()
	UpdateTargetDisplay()
end)

--========================================================
-- PAGES: FLY / RUN
--========================================================

local function FlyPage()
	Clear()
	Highlight(FlyMenu)
	IconHeading(Content,IMG.fly,"FLY")

	local t = Btn(Content,"",UDim2.fromOffset(10,55),UDim2.new(1,-20,0,48))
	Visual(t,"FLY",FlyOn)
	t.MouseButton1Click:Connect(function() SetFly(not FlyOn) end)

	Label(Content,"Fly Speed",UDim2.fromOffset(10,112),UDim2.new(1,-20,0,30),14)
	Stepper(Content,148,function() return FLY_SPEED end,function(v) FLY_SPEED = v end,1,10,500)

	Label(Content,"ดันจอยไปทางที่กล้องมอง = บินไปทางนั้น",UDim2.fromOffset(10,196),UDim2.new(1,-20,0,20),12)
	Label(Content,"Space = ขึ้น  |  LeftCtrl = ลง",UDim2.fromOffset(10,218),UDim2.new(1,-20,0,20),12)

	local poseLbl = Label(Content,"",UDim2.fromOffset(10,246),UDim2.new(1,-20,0,18),11)
	poseLbl.TextColor3 = Color3.fromRGB(170,175,190)

	local function ShowPose()
		poseLbl.Text = "ท่าบิน : " .. (FlyOn and PoseInfo or "เปิด FLY เพื่อดูสถานะ")
	end
	ShowPose()

	OnStateChanged = function()
		if t.Parent then
			Visual(t,"FLY",FlyOn)
			ShowPose()
		end
	end
end

local function RunPage()
	Clear()
	Highlight(RunMenu)
	IconHeading(Content,IMG.run,"RUN SPEED")

	local t = Btn(Content,"",UDim2.fromOffset(10,55),UDim2.new(1,-20,0,48))
	Visual(t,"RUN SPEED",RunOn)
	t.MouseButton1Click:Connect(function() SetRun(not RunOn) end)

	OnStateChanged = function()
		if t.Parent then Visual(t,"RUN SPEED",RunOn) end
	end

	Label(Content,"Walk Speed",UDim2.fromOffset(10,112),UDim2.new(1,-20,0,30),14)
	Stepper(Content,148,function() return RUN_SPEED end,function(v) RUN_SPEED = v ApplyRun() end,1,1,500)
end

--========================================================
-- PAGE: TARGET LOCK (รายชื่อแถวเล็ก)
--========================================================

local function HealthColor(pc)
	return pc > 0.6 and GREEN or (pc > 0.3 and YELLOW or RED)
end

local function TargetPage()
	Clear()
	Highlight(TgtMenu)
	IconHeading(Content,IMG.tgt,"TARGET LOCK")

	local master = Btn(Content,"",UDim2.fromOffset(10,52),UDim2.new(1,-20,0,38))
	Visual(master,"TARGET LOCK",TargetOn)
	master.MouseButton1Click:Connect(function() SetTarget(not TargetOn) end)

	OnStateChanged = function()
		if master.Parent then Visual(master,"TARGET LOCK",TargetOn) end
	end

	local modeBtn = Btn(Content,"MODE : LOCK " .. Mode,UDim2.fromOffset(10,96),UDim2.new(1,-20,0,34))

	-- โหมด ALL: ช่องปรับระยะ
	local rangeGroup = New("Frame",{
		Position = UDim2.fromOffset(0,136),Size = UDim2.new(1,0,0,64),BackgroundTransparency = 1,Visible = (Mode == "ALL"),
	},Content)
	Label(rangeGroup,"Range (ระยะล็อก)",UDim2.fromOffset(10,0),UDim2.new(1,-20,0,18),12)
	Stepper(rangeGroup,20,function() return MAX_DIST end,function(v) MAX_DIST = v end,1,25,1000)

	-- โหมด LIST: รายชื่อผู้เล่น
	local listGroup = New("Frame",{
		Position = UDim2.fromOffset(0,136),Size = UDim2.new(1,0,1,-140),BackgroundTransparency = 1,Visible = (Mode == "LIST"),
	},Content)
	local listTitle = Label(listGroup,"",UDim2.fromOffset(10,0),UDim2.new(1,-20,0,18),12)

	local list = New("ScrollingFrame",{
		Position = UDim2.fromOffset(10,20),Size = UDim2.new(1,-20,1,-24),BackgroundColor3 = DARK,BorderSizePixel = 0,
		ScrollBarThickness = 4,CanvasSize = UDim2.new(),AutomaticCanvasSize = Enum.AutomaticSize.Y,ClipsDescendants = true,
	},listGroup)
	Round(list)
	Stroke(list,WHITE,1,0.5)
	New("UIPadding",{PaddingTop = UDim.new(0,4),PaddingBottom = UDim.new(0,4),PaddingLeft = UDim.new(0,4),PaddingRight = UDim.new(0,4)},list)
	New("UIListLayout",{Padding = UDim.new(0,4),SortOrder = Enum.SortOrder.LayoutOrder},list)

	local emptyLabel = New("TextLabel",{
		Size = UDim2.new(1,-4,0,60),BackgroundTransparency = 1,Text = "ไม่มีผู้เล่นคนอื่นในเกมตอนนี้",
		TextColor3 = Color3.fromRGB(190,190,195),TextSize = 13,Font = Enum.Font.GothamBold,Visible = false,
	},list)

	local function UpdateCount()
		local n = 0
		for _ in pairs(SelectedTargets) do n += 1 end
		listTitle.Text = "รายชื่อผู้เล่น  (เลือกแล้ว " .. n .. " คน)"
	end

	local function RowStyle(row,selected)
		row.BackgroundColor3 = selected and PURPLE:Lerp(BLACK,0.72) or DARK3
		local s = row:FindFirstChildOfClass("UIStroke")
		if s then
			s.Color = selected and PURPLE or WHITE
			s.Transparency = selected and 0 or 0.75
			s.Thickness = selected and 2 or 1
		end
	end

	local function CreatePlayerRow(p)
		local row = New("TextButton",{Size = UDim2.new(1,-4,0,42),BorderSizePixel = 0,Text = "",AutoButtonColor = false},list)
		Round(row,7)
		Stroke(row)
		RowStyle(row,SelectedTargets[p])

		local avatar = New("ImageLabel",{
			Size = UDim2.fromOffset(34,34),Position = UDim2.fromOffset(4,4),BackgroundColor3 = DARK,BorderSizePixel = 0,Image = "",
		},row)
		Round(avatar,6)

		task.spawn(function()
			local ok,image = pcall(Players.GetUserThumbnailAsync,Players,p.UserId,Enum.ThumbnailType.HeadShot,Enum.ThumbnailSize.Size100x100)
			if ok and image and avatar.Parent then avatar.Image = image end
		end)

		New("TextLabel",{
			Size = UDim2.new(1,-108,0,18),Position = UDim2.fromOffset(44,3),BackgroundTransparency = 1,Text = p.DisplayName,
			TextColor3 = WHITE,TextSize = 12,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
			TextTruncate = Enum.TextTruncate.AtEnd,
		},row)

		local hpText = New("TextLabel",{
			AnchorPoint = Vector2.new(1,0.5),Position = UDim2.new(1,-6,0.5,0),Size = UDim2.fromOffset(54,18),
			BackgroundTransparency = 1,Text = "--",TextColor3 = WHITE,TextSize = 11,Font = Enum.Font.GothamBold,
			TextXAlignment = Enum.TextXAlignment.Right,
		},row)

		local hpBG = New("Frame",{
			Size = UDim2.new(1,-108,0,6),Position = UDim2.fromOffset(44,28),BackgroundColor3 = Color3.fromRGB(70,70,75),BorderSizePixel = 0,
		},row)
		Round(hpBG,"full")

		local hpFill = New("Frame",{Size = UDim2.fromScale(0,1),BackgroundColor3 = GREEN,BorderSizePixel = 0},hpBG)
		Round(hpFill,"full")

		task.spawn(function()
			while row.Parent do
				local c = p.Character
				local h = c and c:FindFirstChildOfClass("Humanoid")

				if h then
					local max = math.max(h.MaxHealth,1)
					local hp = math.clamp(h.Health,0,max)
					hpFill.Size = UDim2.fromScale(hp / max,1)
					hpFill.BackgroundColor3 = HealthColor(hp / max)
					hpText.Text = math.floor(hp) .. "/" .. math.floor(max)
				else
					hpFill.Size = UDim2.fromScale(0,1)
					hpText.Text = "--"
				end

				task.wait(0.15)
			end
		end)

		row.MouseButton1Click:Connect(function()
			if SelectedTargets[p] then
				SelectedTargets[p] = nil
				RemoveHL(p)
			else
				SelectedTargets[p] = true
				if TargetOn then AddHL(p) end
			end

			RowStyle(row,SelectedTargets[p])
			UpdateCount()
			UpdateTargetDisplay()
		end)
	end

	local function Refresh()
		for _,c in ipairs(list:GetChildren()) do
			if c:IsA("TextButton") then c:Destroy() end
		end

		local shown = 0

		for _,p in ipairs(Players:GetPlayers()) do
			if p ~= LP and p.Parent and (not TeamCheck or LP.Team == nil or p.Team ~= LP.Team) then
				CreatePlayerRow(p)
				shown += 1
			end
		end

		emptyLabel.Visible = (shown == 0)
		UpdateCount()
	end

	Refresh()

	table.insert(Conns,Players.PlayerAdded:Connect(function() task.wait(0.1) Refresh() end))
	table.insert(Conns,Players.PlayerRemoving:Connect(function(p)
		SelectedTargets[p] = nil
		task.wait()
		Refresh()
	end))

	modeBtn.MouseButton1Click:Connect(function()
		ClearHL()
		table.clear(SelectedTargets)

		Mode = (Mode == "ALL") and "LIST" or "ALL"

		modeBtn.Text = "MODE : LOCK " .. Mode
		rangeGroup.Visible = (Mode == "ALL")
		listGroup.Visible = (Mode == "LIST")

		Refresh()
		UpdateSelectedHighlights()
		UpdateTargetDisplay()
	end)
end

do -- ESP FEATURE SCOPE: keep its locals out of the main Luau scope
--========================================================
-- ESP (เรืองแสงทะลุกำแพง + รูป/ชื่อ/เลือด/ระยะ เหนือหัว)
-- เขียว = ฝั่งเดียวกับเรา | แดง = คนละฝั่ง | เหลือง = เกมไม่มีทีม
--========================================================

local ESP_GREEN = Color3.fromRGB(40,255,90)
local ESP_RED = Color3.fromRGB(255,45,45)
local ESP_YELLOW = Color3.fromRGB(255,225,40)

local itemEspOn = false
SetItemEsp = function(state) itemEspOn = state end
local EspSelectedOnly = false   -- false = ทุกคน / true = เฉพาะที่เลือกใน TARGET LOCK
local ESP_RANGE = 500
local EspData = {}
local AvatarCache = {}

local function EspColor(p,noTeam)
	if noTeam then return ESP_YELLOW end
	return (p.Team == LP.Team) and ESP_GREEN or ESP_RED
end

local function EspDestroy(p)
	local d = EspData[p]

	if d then
		if d.hl then d.hl:Destroy() end
		if d.gui then d.gui:Destroy() end
		EspData[p] = nil
	end
end

local function EspCreate(p)
	local c = p.Character
	local head = c and c:FindFirstChild("Head")
	if not head then return nil end

	local d = EspData[p]
	if d and d.char == c and d.gui.Parent and d.hl.Parent then return d end

	EspDestroy(p)

	local hl = New("Highlight",{
		Name = "TerenggESP",Adornee = c,FillTransparency = 0.5,OutlineTransparency = 0,
		DepthMode = Enum.HighlightDepthMode.AlwaysOnTop,Enabled = false,
	},c)

	local gui = New("BillboardGui",{
		Name = "TerenggESPGui",Adornee = head,AlwaysOnTop = true,Size = UDim2.fromOffset(190,78),
		StudsOffset = Vector3.new(0,3.1,0),LightInfluence = 0,ResetOnSpawn = false,Enabled = false,
	},head)

	local card = New("Frame",{
		Size = UDim2.fromScale(1,1),BackgroundColor3 = Color3.fromRGB(10,10,14),
		BackgroundTransparency = 0.25,BorderSizePixel = 0,
	},gui)
	Round(card,8)
	local cardStroke = Stroke(card,ESP_YELLOW,2)

	local avatar = New("ImageLabel",{
		Size = UDim2.fromOffset(34,34),Position = UDim2.fromOffset(5,5),
		BackgroundColor3 = Color3.fromRGB(35,35,42),BorderSizePixel = 0,Image = "",
	},card)
	Round(avatar,6)

	task.spawn(function()
		local img = AvatarCache[p.UserId]

		if not img then
			local ok,res = pcall(Players.GetUserThumbnailAsync,Players,p.UserId,Enum.ThumbnailType.HeadShot,Enum.ThumbnailSize.Size100x100)
			if ok then img = res; AvatarCache[p.UserId] = res end
		end

		if img and avatar.Parent then avatar.Image = img end
	end)

	New("TextLabel",{
		Position = UDim2.fromOffset(44,4),Size = UDim2.new(1,-49,0,16),BackgroundTransparency = 1,Text = p.DisplayName,
		TextColor3 = WHITE,TextSize = 13,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
	},card)

	local hpBG = New("Frame",{
		Position = UDim2.fromOffset(44,23),Size = UDim2.new(1,-49,0,7),BackgroundColor3 = Color3.fromRGB(60,60,66),BorderSizePixel = 0,
	},card)
	Round(hpBG,"full")

	local hpFill = New("Frame",{Size = UDim2.fromScale(1,1),BackgroundColor3 = GREEN,BorderSizePixel = 0},hpBG)
	Round(hpFill,"full")

	local info = New("TextLabel",{
		Position = UDim2.fromOffset(44,32),Size = UDim2.new(1,-49,0,14),BackgroundTransparency = 1,Text = "",
		TextColor3 = Color3.fromRGB(200,205,215),TextSize = 10,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
	},card)

	local itemRow = New("Frame",{
        Name = "ItemIconsRow", Position = UDim2.fromOffset(5,50),
        Size = UDim2.new(1,-10,0,23), BackgroundTransparency = 1, BorderSizePixel = 0,
    },card)
    d = {char = c,hl = hl,gui = gui,stroke = cardStroke,hpFill = hpFill,info = info,itemRow = itemRow,itemSignature = "",shown = false}
	EspData[p] = d

	return d
end

local function UpdateItemRow(p,d)
    if not d.itemRow then return end
    if not itemEspOn then
        d.itemRow:ClearAllChildren(); d.itemSignature = ""
        return
    end
    local tools = {}
    local function collect(container)
        if container then
            for _,tool in ipairs(container:GetChildren()) do
                if tool:IsA("Tool") and tool.TextureId and tool.TextureId ~= "" then
                    table.insert(tools,tool)
                end
            end
        end
    end
    collect(p.Character)
    collect(p:FindFirstChildOfClass("Backpack"))
    local ids = {}
    for _,tool in ipairs(tools) do table.insert(ids,tool:GetFullName() .. "|" .. tool.TextureId) end
    local signature = table.concat(ids,";")
    if signature == d.itemSignature then return end
    d.itemRow:ClearAllChildren(); d.itemSignature = signature
    if #tools == 0 then return end
    local cell = 22
    local maxIcons = 8
    for i,tool in ipairs(tools) do
        if i > maxIcons then break end
        local col = (i-1) % 8
        New("ImageLabel",{
            Name = "ItemIcon"..i, Size = UDim2.fromOffset(cell,cell),
            Position = UDim2.fromOffset(col*23,0), BackgroundColor3 = Color3.fromRGB(35,35,42),
            BackgroundTransparency = 0.05, BorderSizePixel = 0, Image = tool.TextureId,
            ScaleType = Enum.ScaleType.Fit,
        },d.itemRow)
    end
    if #tools > maxIcons then
        New("TextLabel",{Name="MoreItems",Size=UDim2.fromOffset(26,22),Position=UDim2.fromOffset(8*23,0),
            BackgroundColor3=Color3.fromRGB(35,35,42),BorderSizePixel=0,Text="+"..tostring(#tools-maxIcons),
            TextColor3=WHITE,TextSize=10,Font=Enum.Font.GothamBold},d.itemRow)
    end
end

local function EspUpdate()
	local mr = Root()
	local list = {}

	-- เกมไม่มีทีม (หรือเราไม่มีทีม) = เรืองแสงสีเหลือง
	local noTeam = (#Teams:GetTeams() == 0) or (LP.Team == nil)

	if EspOn and mr then
		for _,p in ipairs(Players:GetPlayers()) do
			if p ~= LP and Alive(p) and (not EspSelectedOnly or SelectedTargets[p]) then
				local tr = p.Character.HumanoidRootPart
				local dist = (tr.Position - mr.Position).Magnitude

				-- ช่วงกันกระพริบที่ขอบระยะ
				local d0 = EspData[p]
				local lim = (d0 and d0.shown) and ESP_RANGE * 1.1 or ESP_RANGE

				if dist <= lim then table.insert(list,{p = p,d = dist}) end
			end
		end
	end

	table.sort(list,function(a,b) return a.d < b.d end)

	local on = {}

	for i,e in ipairs(list) do
		if i > 40 then break end

		local p = e.p
		local d = EspCreate(p)

		if d then
			on[p] = true
			d.shown = true

			local col = EspColor(p,noTeam)
			d.hl.FillColor = col
			d.hl.OutlineColor = col
			d.hl.Enabled = (i <= 20)   -- Roblox วาดไฮไลต์ได้จำกัด เลยเรืองแสง 20 คนที่ใกล้ที่สุด
			d.stroke.Color = col
			d.gui.Enabled = true

			local h = p.Character:FindFirstChildOfClass("Humanoid")

			if h then
				local max = math.max(h.MaxHealth,1)
				local hp = math.clamp(h.Health,0,max)

				d.hpFill.Size = UDim2.fromScale(hp / max,1)
				d.hpFill.BackgroundColor3 = HealthColor(hp / max)
				d.info.Text = "HP " .. math.floor(hp) .. "/" .. math.floor(max) .. "  •  " .. math.floor(e.d) .. "m"
			end
            UpdateItemRow(p,d)
		end
	end

	for p,d in pairs(EspData) do
		if not on[p] then
			d.hl.Enabled = false
			d.gui.Enabled = false
			d.shown = false
		end
	end
end

task.spawn(function()
	while task.wait(0.12) do
		if EspOn then EspUpdate() end
	end
end)

Players.PlayerRemoving:Connect(EspDestroy)

SetEsp = function(state)
	EspOn = state
	if SetItemEsp then SetItemEsp(state) end

	if EspOn then
		EspUpdate()
	else
		for p in pairs(EspData) do EspDestroy(p) end
	end

	Changed()
end

EspPage = function()
	Clear()
	Highlight(EspMenu)
	IconHeading(Content,IMG.esp,"ESP")

	local t = Btn(Content,"",UDim2.fromOffset(10,52),UDim2.new(1,-20,0,38))
	Visual(t,"ESP",EspOn)
	t.MouseButton1Click:Connect(function() SetEsp(not EspOn) end)

	OnStateChanged = function()
		if t.Parent then Visual(t,"ESP",EspOn) end
	end

	local modeB = Btn(Content,"",UDim2.fromOffset(10,96),UDim2.new(1,-20,0,34))
	modeB.TextSize = 12

	local function ShowMode()
		modeB.Text = EspSelectedOnly and "แสดง : เฉพาะที่เลือกใน TARGET LOCK" or "แสดง : ทุกคน"
	end
	ShowMode()

	modeB.MouseButton1Click:Connect(function()
		EspSelectedOnly = not EspSelectedOnly
		ShowMode()
		if EspOn then EspUpdate() end
	end)

	Label(Content,"Range (ระยะแสดง)",UDim2.fromOffset(10,136),UDim2.new(1,-20,0,18),12)
	Stepper(Content,154,function() return ESP_RANGE end,function(v) ESP_RANGE = v end,1,50,3000)

	local g = Label(Content,"●  เขียว = ฝั่งเดียวกับเรา",UDim2.fromOffset(10,200),UDim2.new(1,-20,0,18),12)
	g.TextColor3 = ESP_GREEN

	local r = Label(Content,"●  แดง = คนละฝั่ง",UDim2.fromOffset(10,218),UDim2.new(1,-20,0,18),12)
	r.TextColor3 = ESP_RED

	local y = Label(Content,"●  เหลือง = เกมไม่มีทีม",UDim2.fromOffset(10,236),UDim2.new(1,-20,0,18),12)
	y.TextColor3 = ESP_YELLOW

	local n1 = Label(Content,"เหนือหัว : รูป • ชื่อ • เลือด • ระยะ",UDim2.fromOffset(10,262),UDim2.new(1,-20,0,16),11)
	n1.TextColor3 = Color3.fromRGB(170,175,190)

	local n2 = Label(Content,"มองทะลุกำแพง (เรืองแสง 20 คนที่ใกล้ที่สุด)",UDim2.fromOffset(10,280),UDim2.new(1,-20,0,16),11)
	n2.TextColor3 = Color3.fromRGB(170,175,190)
end
end -- ESP FEATURE SCOPE

--========================================================
-- colorui (เครื่องทำสี) - แท็บ + แตะแล้วเปลี่ยนทันที
--========================================================

local NEON = Color3.fromRGB(30,230,110)
local ACCENT_DEFAULT = Color3.fromRGB(255,50,50)
local PURPLE_DEFAULT = Color3.fromRGB(160,70,255)
local THEME_DEFAULT = Color3.fromRGB(15,15,18)
local DARK_DEFAULT = Color3.fromRGB(25,25,30)
local DARK3_DEFAULT = Color3.fromRGB(45,45,54)

local function C(name,r,g,b) return {name,Color3.fromRGB(r,g,b)} end

local Palettes = {
	{ -- สีเมนู
		C("ขาว",245,245,245),C("เทา",130,135,145),C("แดง",255,50,50),C("ส้ม",255,120,10),C("เหลือง",255,195,10),C("เขียวอ่อน",100,235,20),
		C("เขียว",20,175,50),C("ฟ้าเขียว",20,190,210),C("ฟ้า",20,170,255),C("น้ำเงิน",20,90,235),C("ม่วง",160,70,255),C("ชมพู",245,50,235),
	},
	{ -- สีล็อกเป้า
		C("ขาว",245,245,245),C("แดง",255,50,50),C("ส้ม",255,120,10),C("เหลือง",255,215,20),
		C("เขียว",30,200,90),C("ฟ้า",20,170,255),C("ม่วง",160,70,255),C("ชมพู",245,50,235),
	},
	{ -- สีหน้าต่าง
		{"ดำ",THEME_DEFAULT},C("น้ำเงิน",22,38,92),C("เขียวป่า",16,62,40),C("ม่วง",62,24,88),
		C("แดงเข้ม",88,24,30),C("เขียวน้ำทะเล",14,64,72),C("น้ำตาล",78,52,28),C("เทา",70,76,92),
	},
}

local TabNames = {"สีเมนู","สีล็อกเป้า","สีหน้าต่าง"}


--========================================================
-- PAGE: SCRIPT EDITOR (added to the existing panel)
--========================================================
local function ScriptEditorPage()
    Clear()
    Highlight(ScriptMenu)
    IconHeading(Content,IMG.set,"SCRIPT EDITOR")

    Label(Content,"เขียน/แก้ไขโค้ด Luau",UDim2.fromOffset(10,48),UDim2.new(1,-20,0,22),13)
    local request = New("TextBox",{
        Position=UDim2.fromOffset(10,73),Size=UDim2.new(1,-20,0,34),
        BackgroundColor3=DARK3,TextColor3=WHITE,PlaceholderText="ตัวอย่าง: red part / welcome UI",
        Text="",ClearTextOnFocus=false,TextSize=12,Font=Enum.Font.Gotham,
    },Content)
    Round(request)
    local code = New("TextBox",{
        Name="ScriptEditorCode",Position=UDim2.fromOffset(10,115),Size=UDim2.new(1,-20,1,-205),
        BackgroundColor3=Color3.fromRGB(5,8,14),TextColor3=Color3.fromRGB(190,225,255),
        Text='-- Write Luau code here\nlocal part = Instance.new("Part")\npart.Name = "RedPart"\npart.Size = Vector3.new(4, 4, 4)\npart.Color = Color3.fromRGB(255, 0, 0)\npart.Anchored = true\npart.Parent = workspace',
        ClearTextOnFocus=false,MultiLine=true,TextWrapped=false,TextXAlignment=Enum.TextXAlignment.Left,
        TextYAlignment=Enum.TextYAlignment.Top,Font=Enum.Font.Code,TextSize=13,
    },Content)
    Round(code); Stroke(code,ACCENT,1,0.35)
    local status = New("TextLabel",{
        Position=UDim2.new(0,10,1,-82),Size=UDim2.new(1,-20,0,28),
        BackgroundColor3=Color3.fromRGB(35,20,20),TextColor3=YELLOW,
        Text="ตรวจสอบพื้นฐานเท่านั้น ไม่ใช่ตัวตรวจ Luau เต็มรูปแบบ",TextWrapped=true,
        Font=Enum.Font.Gotham,TextSize=11,
    },Content)
    Round(status)
    local function smallButton(text,x,w)
        local b=Btn(Content,text,UDim2.new(x,0,1,-47),UDim2.new(w,-4,0,34))
        b.TextSize=11
        return b
    end
    local check=smallButton("CHECK",0,0.25)
    local generate=smallButton("GENERATE",0.25,0.25)
    local selectCode=smallButton("SELECT CODE",0.5,0.25)
    local clearCode=smallButton("CLEAR",0.75,0.25)

    local function basicCheck(src)
        local stack, pairsMap = {}, {[")"]="(",["]"]="[",["}"]="{"}
        local quote, escaped, line = nil, false, 1
        for i=1,#src do
            local c=src:sub(i,i)
            if c=="\n" then line+=1 end
            if quote then
                if escaped then escaped=false elseif c=="\\" then escaped=true elseif c==quote then quote=nil end
            else
                if c=='"' or c=="'" then quote=c
                elseif c=="(" or c=="[" or c=="{" then table.insert(stack,{c,line})
                elseif pairsMap[c] then
                    local last=stack[#stack]
                    if not last or last[1]~=pairsMap[c] then return false,"ข้อผิดพลาดที่อาจเกิดใกล้บรรทัด "..line..": วงเล็บไม่ตรงกัน" end
                    table.remove(stack)
                end
            end
        end
        if quote then return false,"ข้อผิดพลาดที่อาจเกิด: เครื่องหมายข้อความยังไม่ปิด" end
        if #stack>0 then return false,"ข้อผิดพลาดที่อาจเกิด: วงเล็บบรรทัด "..stack[#stack][2].." ยังไม่ปิด" end
        return true,"ตรวจวงเล็บพื้นฐานผ่าน แต่ยังอาจมีข้อผิดพลาดอื่น"
    end
    check.Activated:Connect(function()
        local ok,msg=basicCheck(code.Text); status.Text=msg
        status.TextColor3=ok and GREEN or Color3.fromRGB(255,70,70)
        status.BackgroundColor3=ok and Color3.fromRGB(12,35,20) or Color3.fromRGB(45,10,15)
    end)
    generate.Activated:Connect(function()
        local q=string.lower(request.Text)
        if q:find("part") and q:find("red") then
            code.Text='local part = Instance.new("Part")\npart.Name = "RedPart"\npart.Size = Vector3.new(4, 4, 4)\npart.Color = Color3.fromRGB(255, 0, 0)\npart.Anchored = true\npart.Parent = workspace'
            status.Text="สร้างตัวอย่างโค้ด Part สีแดงแล้ว"
        elseif q:find("part") then
            code.Text='local part = Instance.new("Part")\npart.Name = "NewPart"\npart.Size = Vector3.new(4, 1, 4)\npart.Anchored = true\npart.Position = Vector3.new(0, 5, 0)\npart.Parent = workspace'
            status.Text="สร้างตัวอย่างโค้ด Part แล้ว"
        elseif q:find("ui") or q:find("welcome") then
            code.Text='-- Basic welcome UI\nlocal Players = game:GetService("Players")\nlocal player = Players.LocalPlayer\nlocal gui = Instance.new("ScreenGui")\ngui.Name = "WelcomeUI"\ngui.Parent = player:WaitForChild("PlayerGui")\nlocal label = Instance.new("TextLabel")\nlabel.Size = UDim2.new(0,260,0,60)\nlabel.Position = UDim2.new(0.5,-130,0.2,0)\nlabel.Text = "Welcome!"\nlabel.TextScaled = true\nlabel.Parent = gui'
            status.Text="สร้างตัวอย่างโค้ด UI แล้ว"
        else
            status.Text="คำสั่งนี้ยังไม่รองรับ ลอง red part หรือ welcome UI"
            status.TextColor3=YELLOW; return
        end
        status.TextColor3=GREEN
    end)
    selectCode.Activated:Connect(function()
        code:CaptureFocus()
        status.Text="เลือกข้อความในช่องโค้ดด้วยตนเอง; LocalScript ปกติสั่งคัดลอกคลิปบอร์ดโดยตรงไม่ได้"
        status.TextColor3=YELLOW
    end)
    clearCode.Activated:Connect(function() code.Text=""; status.Text="ล้างโค้ดแล้ว"; status.TextColor3=WHITE end)
end

local Pages = {[FlyMenu] = FlyPage,[RunMenu] = RunPage,[TgtMenu] = TargetPage,[EspMenu] = EspPage,[ScriptMenu] = ScriptEditorPage}

local saved = {ACCENT,PURPLE,BLACK}   -- สีที่บันทึกไว้
local pend = {ACCENT,PURPLE,BLACK}    -- สีที่กำลังเลือก
local ActiveTab = 1

local Picker = New("Frame",{
	AnchorPoint = Vector2.new(0.5,0.5),Position = UDim2.fromScale(0.5,0.5),Size = UDim2.fromOffset(340,316),
	BackgroundColor3 = Color3.fromRGB(10,14,20),BorderSizePixel = 0,Active = true,Visible = false,ZIndex = 200,
},Gui)
Round(Picker,12)
Stroke(Picker,Color3.fromRGB(40,60,70),2)

local PickerScale = New("UIScale",{},Picker)

local PHeader = New("Frame",{
	Position = UDim2.fromOffset(8,8),Size = UDim2.new(1,-16,0,40),BackgroundColor3 = Color3.fromRGB(12,20,26),BorderSizePixel = 0,
},Picker)
Round(PHeader,10)
Stroke(PHeader,NEON,2)

New("ImageLabel",{Position = UDim2.fromOffset(10,6),Size = UDim2.fromOffset(28,28),BackgroundTransparency = 1,Image = IMG.color},PHeader)
New("TextLabel",{
	Position = UDim2.fromOffset(46,0),Size = UDim2.new(1,-92,1,0),BackgroundTransparency = 1,RichText = true,
	Text = 'color<font color="rgb(30,230,110)">ui</font>',TextColor3 = WHITE,TextSize = 20,
	Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
},PHeader)

local PClose = New("TextButton",{
	AnchorPoint = Vector2.new(1,0.5),Position = UDim2.new(1,-6,0.5,0),Size = UDim2.fromOffset(36,30),BackgroundTransparency = 1,
	Text = "✕",TextColor3 = WHITE,TextSize = 22,Font = Enum.Font.GothamBold,AutoButtonColor = false,
},PHeader)

local NameLabel = New("TextLabel",{
	Position = UDim2.fromOffset(14,96),Size = UDim2.new(1,-28,0,20),BackgroundTransparency = 1,RichText = true,Text = "",
	TextColor3 = WHITE,TextSize = 14,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
},Picker)

local ApplyColors, MarkAll, ShowTab
local TabBtns, Grids, Cells = {}, {}, {}

local function Recolor(oldB,oldD,oldD3)
	local list = {Main}
	for _,o in ipairs(Main:GetDescendants()) do table.insert(list,o) end
	for _,o in ipairs(Quick:GetDescendants()) do table.insert(list,o) end

	for _,o in ipairs(list) do
		if o:IsA("GuiObject") then
			local c = o.BackgroundColor3
			if c == oldB then o.BackgroundColor3 = BLACK
			elseif c == oldD then o.BackgroundColor3 = DARK
			elseif c == oldD3 then o.BackgroundColor3 = DARK3 end
		end
	end
end

ApplyColors = function(m,t,th)
	ACCENT, PURPLE = m, t

	local oldB, oldD, oldD3 = BLACK, DARK, DARK3
	BLACK = th

	if th == THEME_DEFAULT then
		DARK, DARK3 = DARK_DEFAULT, DARK3_DEFAULT
	else
		DARK, DARK3 = th:Lerp(WHITE,0.12), th:Lerp(WHITE,0.26)
	end

	Recolor(oldB,oldD,oldD3)

	local st = PlusBtn:FindFirstChildOfClass("UIStroke")
	if st then st.Color = PURPLE end

	Highlight(not InSettings and LastActive or nil)

	-- ไฮไลต์ที่เปิดอยู่จะเปลี่ยนเป็นสีใหม่ทันที (ไม่ต้องปิดแล้วสร้างใหม่)
	UpdateSelectedHighlights()
end

local function ApplyPending() ApplyColors(pend[1],pend[2],pend[3]) end

MarkAll = function()
	for i = 1,3 do
		for _,c in ipairs(Cells[i]) do
			local on = (c.color == pend[i])
			c.stroke.Transparency = on and 0 or 1
			c.check.Visible = on
		end
	end

	local name = "-"
	for _,c in ipairs(Cells[ActiveTab]) do
		if c.color == pend[ActiveTab] then name = c.name end
	end

	NameLabel.Text = 'เลือกอยู่ :  <font color="rgb(30,230,110)">' .. name .. '</font>'
end

ShowTab = function(i)
	ActiveTab = i

	for k = 1,3 do
		local on = (k == i)
		Grids[k].Visible = on
		TabBtns[k].stroke.Transparency = on and 0 or 1
		TabBtns[k].btn.BackgroundColor3 = on and Color3.fromRGB(25,70,52) or Color3.fromRGB(30,36,46)
	end

	MarkAll()
end

for i = 1,3 do
	local tb = New("TextButton",{
		Position = UDim2.fromOffset(10 + (i - 1) * 107,56),Size = UDim2.fromOffset(103,34),BackgroundColor3 = Color3.fromRGB(30,36,46),
		BorderSizePixel = 0,Text = TabNames[i],TextColor3 = WHITE,TextSize = 14,Font = Enum.Font.GothamBold,AutoButtonColor = false,
	},Picker)
	Round(tb)
	TabBtns[i] = {btn = tb,stroke = Stroke(tb,NEON,2,1)}
	tb.MouseButton1Click:Connect(function() ShowTab(i) end)

	Grids[i] = New("Frame",{Position = UDim2.fromOffset(0,122),Size = UDim2.new(1,0,0,104),BackgroundTransparency = 1,Visible = (i == 1)},Picker)
	Cells[i] = {}

	for idx,entry in ipairs(Palettes[i]) do
		local sw = New("TextButton",{
			Position = UDim2.fromOffset(18 + ((idx - 1) % 6) * 52,math.floor((idx - 1) / 6) * 52),Size = UDim2.fromOffset(44,44),
			BackgroundColor3 = entry[2],Text = "",AutoButtonColor = false,BorderSizePixel = 0,
		},Grids[i])
		Round(sw,10)

		local ck = New("TextLabel",{
			Position = UDim2.new(1,-12,0,-6),Size = UDim2.fromOffset(18,18),BackgroundColor3 = NEON,BorderSizePixel = 0,Text = "✓",
			TextColor3 = WHITE,TextSize = 12,Font = Enum.Font.GothamBold,Visible = false,ZIndex = 3,
		},sw)
		Round(ck,"full")

		Cells[i][idx] = {stroke = Stroke(sw,WHITE,3,1),check = ck,color = entry[2],name = entry[1]}

		sw.MouseButton1Click:Connect(function()
			pend[i] = entry[2]
			MarkAll()
			ApplyPending()
		end)
	end
end

-- ปิด colorui: ถ้าไม่ได้กดบันทึก สีจะย้อนกลับเป็นสีที่บันทึกไว้
local function ClosePicker(keep)
	if not Picker.Visible then return end
	Picker.Visible = false

	if not keep then
		pend = {saved[1],saved[2],saved[3]}
		ApplyPending()
	end
end

local SaveBtn = New("TextButton",{
	Position = UDim2.fromOffset(13,236),Size = UDim2.fromOffset(214,44),BackgroundColor3 = Color3.fromRGB(25,185,85),BorderSizePixel = 0,
	Text = "💾  บันทึกสี",TextColor3 = WHITE,TextSize = 18,Font = Enum.Font.GothamBold,AutoButtonColor = false,
},Picker)
Round(SaveBtn)
Stroke(SaveBtn,NEON,2)

local ResetBtn = New("TextButton",{
	Position = UDim2.fromOffset(233,236),Size = UDim2.fromOffset(94,44),BackgroundColor3 = Color3.fromRGB(45,45,54),BorderSizePixel = 0,
	Text = "คืนค่าเดิม",TextColor3 = WHITE,TextSize = 13,Font = Enum.Font.GothamBold,AutoButtonColor = false,
},Picker)
Round(ResetBtn)

New("TextLabel",{
	Position = UDim2.fromOffset(13,286),Size = UDim2.new(1,-26,0,22),BackgroundTransparency = 1,
	Text = "แตะสีเพื่อดูตัวอย่างทันที  •  บันทึกเพื่อเก็บไว้  •  ปิด = ยกเลิก",
	TextColor3 = Color3.fromRGB(150,160,175),TextSize = 11,Font = Enum.Font.Gotham,
},Picker)

SaveBtn.MouseButton1Click:Connect(function()
	saved = {pend[1],pend[2],pend[3]}
	ClosePicker(true)
end)

ResetBtn.MouseButton1Click:Connect(function()
	pend = {ACCENT_DEFAULT,PURPLE_DEFAULT,THEME_DEFAULT}
	MarkAll()
	ApplyPending()
end)

PClose.MouseButton1Click:Connect(function() ClosePicker() end)

local function OpenPicker()
	if Picker.Visible then ClosePicker() return end

	pend = {saved[1],saved[2],saved[3]}

	local cam = GetCamera()
	if cam then PickerScale.Scale = math.clamp(cam.ViewportSize.Y / 350,0.5,1) end

	Picker.Visible = true
	ShowTab(1)
end

Main:GetPropertyChangedSignal("Visible"):Connect(function()
	if not Main.Visible then ClosePicker() end
end)

--========================================================
-- SETTINGS (หน้าตั้งค่าแบบเลื่อนดู แบ่งหมวด)
--========================================================

local SetBtn = New("ImageButton",{
	Name = "SettingsButton", Size = UDim2.fromOffset(42,38),Position = UDim2.new(1,-93,0,5),BackgroundColor3 = DARK3,Image = IMG.set,
	ScaleType = Enum.ScaleType.Fit,AutoButtonColor = true,Active = true,Selectable = true,ZIndex = 50,
},Header)
Round(SetBtn)

local KeyNameMap = {
	RightControl = "R-Ctrl",LeftControl = "L-Ctrl",RightShift = "R-Shift",
	LeftShift = "L-Shift",RightAlt = "R-Alt",LeftAlt = "L-Alt",
}

local SettingsOpen = false
local function SettingsPage()
	SettingsOpen = true
	Clear()
	Highlight(nil)

	Label(Content,"SETTINGS",UDim2.fromOffset(12,8),UDim2.new(1,-24,0,32),21)

	local scroll = New("ScrollingFrame",{
		Position = UDim2.fromOffset(6,44),Size = UDim2.new(1,-12,1,-50),BackgroundTransparency = 1,BorderSizePixel = 0,
		ScrollBarThickness = 4,CanvasSize = UDim2.new(),AutomaticCanvasSize = Enum.AutomaticSize.Y,
	},Content)
	New("UIListLayout",{Padding = UDim.new(0,6),SortOrder = Enum.SortOrder.LayoutOrder},scroll)
	New("UIPadding",{PaddingLeft = UDim.new(0,4),PaddingRight = UDim.new(0,8),PaddingTop = UDim.new(0,2),PaddingBottom = UDim.new(0,8)},scroll)

	local order = 0
	local function nxt() order += 1 return order end

	local function Section(text)
		New("TextLabel",{
			Size = UDim2.new(1,0,0,22),BackgroundTransparency = 1,Text = text,TextColor3 = Color3.fromRGB(170,175,190),
			TextSize = 12,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,LayoutOrder = nxt(),
		},scroll)
	end

	local function ListBtn(text,h)
		local b = Btn(scroll,text,UDim2.new(),UDim2.new(1,0,0,h or 38))
		b.TextSize = 13
		b.LayoutOrder = nxt()
		return b
	end

	-- ① หน้าตา
	Section("①  หน้าตา")

	local colorBtn = IconBtn(scroll,IMG.color,"colorui   ▸",UDim2.new(),UDim2.new(1,0,0,40))
	colorBtn.LayoutOrder = nxt()
	colorBtn.MouseButton1Click:Connect(OpenPicker)

	-- ② ปุ่มลัดคีย์บอร์ด (เรียงตามเมนู)
	Section("②  ปุ่มลัดคีย์บอร์ด  (เรียงตามเมนู)")

	New("TextLabel",{
		Size = UDim2.new(1,0,0,30),BackgroundTransparency = 1,
		Text = "แตะปุ่มด้านขวา แล้วกดตัวอักษรภาษาอังกฤษบนคีย์บอร์ด  (Esc = ยกเลิก)",
		TextColor3 = Color3.fromRGB(150,160,175),TextSize = 11,TextWrapped = true,Font = Enum.Font.Gotham,
		TextXAlignment = Enum.TextXAlignment.Left,LayoutOrder = nxt(),
	},scroll)

	local keyBtns = {}

	local function ShowKeys()
		for _,name in ipairs(KeyOrder) do
			keyBtns[name].Text = KeyNameMap[Keys[name].Name] or Keys[name].Name
		end
	end

	for _,name in ipairs(KeyOrder) do
		local row = New("Frame",{Size = UDim2.new(1,0,0,38),BackgroundColor3 = DARK3,BorderSizePixel = 0,LayoutOrder = nxt()},scroll)
		Round(row)

		local quickIdForKey = { fly = "fly", run = "run", target = "target", esp = "esp" }
		local quickId = quickIdForKey[name]
		New("TextLabel",{
			Position = UDim2.fromOffset(10,0),Size = UDim2.new(1,-132,1,0),BackgroundTransparency = 1,Text = KeyLabels[name],
			TextColor3 = WHITE,TextSize = 12,Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,
			TextTruncate = Enum.TextTruncate.AtEnd,
		},row)

		local kb = New("TextButton",{
			AnchorPoint = Vector2.new(1,0.5),Position = UDim2.new(1,-40,0.5,0),Size = UDim2.fromOffset(58,26),BackgroundColor3 = DARK,
			BorderSizePixel = 0,Text = "",TextColor3 = WHITE,TextSize = 13,Font = Enum.Font.GothamBold,AutoButtonColor = false,
		},row)
		Round(kb,6)
		Stroke(kb,WHITE,1,0.45)

		-- Checkbox nhỏ ngay cạnh phím tiếng Anh: hiển thị/ẩn nút vuông trên màn hình
		if quickId then
			local toggle = New("TextButton",{
				Name = "ShowQuick_" .. quickId,AnchorPoint = Vector2.new(1,0.5),Position = UDim2.new(1,-6,0.5,0),
				Size = UDim2.fromOffset(28,28),BackgroundColor3 = QuickVisible[quickId] and GREEN or RED,
				BorderSizePixel = 0,Text = QuickVisible[quickId] and "✓" or "×",TextColor3 = WHITE,
				TextSize = 16,Font = Enum.Font.GothamBold,AutoButtonColor = false,
			},row)
			Round(toggle,6)
			local function refreshInlineToggle()
				local on = QuickVisible[quickId]
				toggle.BackgroundColor3 = on and GREEN or RED
				toggle.Text = on and "✓" or "×"
			end
			toggle.Activated:Connect(function()
				SetQuickVisible(quickId, not QuickVisible[quickId])
				RefreshQuick()
				refreshInlineToggle()
			end)
		end

		keyBtns[name] = kb

		kb.MouseButton1Click:Connect(function()
			WaitingFor = name
			ShowKeys()
			kb.Text = "กดปุ่ม..."
		end)
	end

	ShowKeys()

	local keyMsg = New("TextLabel",{
		Size = UDim2.new(1,0,0,18),BackgroundTransparency = 1,Text = "",TextColor3 = YELLOW,TextSize = 11,
		Font = Enum.Font.GothamBold,TextXAlignment = Enum.TextXAlignment.Left,LayoutOrder = nxt(),
	},scroll)

	OnKeySet = function(msg)
		if keyMsg.Parent then
			ShowKeys()
			keyMsg.Text = msg or ""
		end
	end

	-- ตัวเลือกแสดง/ซ่อนปุ่มลัดถูกวางไว้ในแต่ละแถว FLY/RUN/TARGET/ESP ด้านบนแล้ว

	local wallBtn = ListBtn("",36)
	Visual(wallBtn,"เช็กกำแพงก่อนล็อกเป้า",WALL_CHECK)
	wallBtn.MouseButton1Click:Connect(function()
		WALL_CHECK = not WALL_CHECK
		Visual(wallBtn,"เช็กกำแพงก่อนล็อกเป้า",WALL_CHECK)
		UpdateTargetDisplay()
	end)

	-- ④ ระบบต่อสู้ / ตัวเลขดาเมจ / ล็อก NPC (อยู่ใน SETTINGS เดิม)
	Section("④  ระบบต่อสู้ / ดาเมจ")

	local damageBtn = ListBtn("",38)
	Visual(damageBtn,"แสดงตัวเลขดาเมจต่อครั้ง",DamageNumbersOn)
	damageBtn.MouseButton1Click:Connect(function()
		DamageNumbersOn = not DamageNumbersOn
		if not DamageNumbersOn then ClearDamagePopups() end
		Visual(damageBtn,"แสดงตัวเลขดาเมจต่อครั้ง",DamageNumbersOn)
	end)

	local damageNote = New("TextLabel",{
		Size = UDim2.new(1,0,0,34),BackgroundTransparency = 1,
		Text = "เปิด/ปิดตัวเลขดาเมจที่แสดงเมื่อ HP ของผู้เล่นคนอื่นลดลง",
		TextColor3 = Color3.fromRGB(155,165,180),TextSize = 11,TextWrapped = true,
		Font = Enum.Font.Gotham,TextXAlignment = Enum.TextXAlignment.Left,LayoutOrder = nxt(),
	},scroll)

	local backBtn = ListBtn("◀  กลับ",38)
	backBtn.Activated:Connect(function()
		SettingsOpen = false
		ClosePicker()
		;(Pages[LastActive] or FlyPage)()
	end)
end

SetBtn.Activated:Connect(function()
	local ok, err = pcall(function()
		if SettingsOpen then
			SettingsOpen = false
			ClosePicker()
			;(Pages[LastActive] or FlyPage)()
		else
			SettingsPage()
		end
	end)
	if not ok then warn("TERENGG666 SETTINGS ERROR: " .. tostring(err)) end
end)

--========================================================
-- RESIZE HANDLE (มุมขวาล่าง)
--========================================================

local ResizeHandle = New("TextButton",{
	AnchorPoint = Vector2.new(1,1),Position = UDim2.new(1,0,1,0),Size = UDim2.fromOffset(42,42),BackgroundTransparency = 1,
	Text = "◢",TextColor3 = WHITE,TextSize = 22,Font = Enum.Font.GothamBold,AutoButtonColor = false,ZIndex = 100,
},Main)

local resizing, resizeStart, startSize = false

ResizeHandle.InputBegan:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
		resizing, resizeStart, startSize = true, i.Position, Main.AbsoluteSize

		i.Changed:Connect(function()
			if i.UserInputState == Enum.UserInputState.End then resizing = false end
		end)
	end
end)

UIS.InputChanged:Connect(function(i)
	if not resizing then return end
	if i.UserInputType ~= Enum.UserInputType.MouseMovement and i.UserInputType ~= Enum.UserInputType.Touch then return end

	local cam = GetCamera()
	if not cam then return end

	local d, vp = i.Position - resizeStart, cam.ViewportSize

	Main.Size = UDim2.fromOffset(
		math.clamp(startSize.X + d.X,300,vp.X - 20),
		math.clamp(startSize.Y + d.Y,260,vp.Y - 20)
	)
end)

--========================================================
-- MENU + START
--========================================================

for menu,page in pairs(Pages) do
	menu.MouseButton1Click:Connect(function()
		ClosePicker()
		page()
	end)
end

FlyPage()
RefreshQuick()
UpdateTargetDisplay()
