--=====================================================
-- Remote Scanner GUI (Client-Side)
--=====================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer

--------------------------------------------------------
-- GUI SETUP
--------------------------------------------------------

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RemoteScanner"
ScreenGui.Parent = player:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- Main Window
local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 350, 0, 420)
Main.Position = UDim2.new(0.35, 0, 0.2, 0)
Main.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
Main.BorderSizePixel = 0
Main.Parent = ScreenGui

local mainCorner = Instance.new("UICorner", Main)
mainCorner.CornerRadius = UDim.new(0, 12)

-- Drop shadow
local shadow = Instance.new("ImageLabel")
shadow.Parent = Main
shadow.AnchorPoint = Vector2.new(0.5, 0.5)
shadow.Position = UDim2.new(0.5, 3, 0.5, 3)
shadow.Size = UDim2.new(1, 20, 1, 20)
shadow.BackgroundTransparency = 1
shadow.Image = "rbxassetid://6015897843"
shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
shadow.ImageTransparency = 0.4
shadow.ZIndex = -1

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -20, 0, 40)
Title.Position = UDim2.new(0, 10, 0, 5)
Title.BackgroundTransparency = 1
Title.Text = "Remote Scanner"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 24
Title.TextColor3 = Color3.fromRGB(0, 255, 200)
Title.Parent = Main

--------------------------------------------------------
-- SMOOTH DRAGGING SYSTEM
--------------------------------------------------------

local dragging, dragStart, startPos = false, nil, nil

local function UpdateDrag(input)
    local delta = input.Position - dragStart
    Main.Position = UDim2.new(
        startPos.X.Scale, startPos.X.Offset + delta.X,
        startPos.Y.Scale, startPos.Y.Offset + delta.Y
    )
end

Main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Main.Position
    end
end)

Main.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        local connection
        connection = UIS.InputChanged:Connect(function(movement)
            if dragging and movement == input then
                UpdateDrag(movement)
            elseif not dragging then
                connection:Disconnect()
            end
        end)
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

--------------------------------------------------------
-- SCAN BUTTON
--------------------------------------------------------

local ScanButton = Instance.new("TextButton")
ScanButton.Size = UDim2.new(0, 320, 0, 45)
ScanButton.Position = UDim2.new(0, 15, 0, 55)
ScanButton.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
ScanButton.Text = "Scan Remotes"
ScanButton.Font = Enum.Font.GothamMedium
ScanButton.TextSize = 20
ScanButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ScanButton.Parent = Main

local scanCorner = Instance.new("UICorner", ScanButton)
scanCorner.CornerRadius = UDim.new(0, 8)

ScanButton.MouseEnter:Connect(function()
    ScanButton.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
end)
ScanButton.MouseLeave:Connect(function()
    ScanButton.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
end)

--------------------------------------------------------
-- SCROLLING LIST
--------------------------------------------------------

local Scrolling = Instance.new("ScrollingFrame")
Scrolling.Size = UDim2.new(0, 320, 0, 310)
Scrolling.Position = UDim2.new(0, 15, 0, 110)
Scrolling.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
Scrolling.BorderSizePixel = 0
Scrolling.ScrollBarThickness = 6
Scrolling.CanvasSize = UDim2.new(0, 0, 0, 0)
Scrolling.Parent = Main

local scrollCorner = Instance.new("UICorner", Scrolling)
scrollCorner.CornerRadius = UDim.new(0, 8)

local UIList = Instance.new("UIListLayout")
UIList.Parent = Scrolling
UIList.Padding = UDim.new(0, 8)

--------------------------------------------------------
-- SCAN FUNCTION
--------------------------------------------------------

local function ClearList()
	for _, child in ipairs(Scrolling:GetChildren()) do
		if child:IsA("TextButton") then
			child:Destroy()
		end
	end
end

local function ScanForRemotes()
	ClearList()
	local count = 0

	for _, obj in ipairs(game:GetDescendants()) do
		if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
			count += 1

			local btn = Instance.new("TextButton")
			btn.Size = UDim2.new(1, -10, 0, 36)
			btn.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
			btn.TextColor3 = Color3.fromRGB(200, 255, 255)
			btn.Font = Enum.Font.Gotham
			btn.TextSize = 16
			btn.Text = obj.Name .. "  [" .. obj.ClassName .. "]"
			btn.Parent = Scrolling

			local bcorner = Instance.new("UICorner", btn)
			bcorner.CornerRadius = UDim.new(0, 6)

			btn.MouseEnter:Connect(function()
				btn.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
			end)
			btn.MouseLeave:Connect(function()
				btn.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
			end)

			btn.MouseButton1Click:Connect(function()
				if obj:IsA("RemoteEvent") then
					print("Firing RemoteEvent:", obj:GetFullName())
					pcall(function() obj:FireServer() end)
				else
					print("Invoking RemoteFunction:", obj:GetFullName())
					pcall(function() obj:InvokeServer() end)
				end
			end)
		end
	end

	Scrolling.CanvasSize = UDim2.new(0, 0, 0, count * 44)
end

ScanButton.MouseButton1Click:Connect(ScanForRemotes)

--------------------------------------------------------
-- FADE-IN ANIMATION
--------------------------------------------------------

Main.BackgroundTransparency = 1
Title.TextTransparency = 1
ScanButton.TextTransparency = 1

task.spawn(function()
	for i = 1, 20 do
		Main.BackgroundTransparency = 1 - (i / 20)
		Title.TextTransparency = 1 - (i / 20)
		ScanButton.TextTransparency = 1 - (i / 20)
		task.wait(0.01)
	end
end)

--=====================================================
-- END OF SCRIPT
--=====================================================
