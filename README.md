local player = game.Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- GUI
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "AutoFarmGUI"
gui.ResetOnSpawn = false
gui.DisplayOrder = 1000

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 300)
frame.Position = UDim2.new(0, 10, 0.4, 0)
frame.BackgroundTransparency = 0.4
frame.BackgroundColor3 = Color3.new(0, 0, 0)

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

-- Botão GUI
local toggleGuiButton = Instance.new("TextButton", gui)
toggleGuiButton.Size = UDim2.new(0, 50, 0, 30)
toggleGuiButton.Position = UDim2.new(0, 10, 0.35, 0)
toggleGuiButton.Text = "GUI"
toggleGuiButton.BackgroundTransparency = 0.2
toggleGuiButton.BackgroundColor3 = Color3.new(0, 0, 0)

toggleGuiButton.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

-- Estados
local harvesting = false
local selling = false
local selectedFruit = nil

-- Botões principais
local harvestButton = Instance.new("TextButton", frame)
harvestButton.Size = UDim2.new(1, -20, 0, 40)
harvestButton.Position = UDim2.new(0, 10, 0, 10)
harvestButton.Text = "Auto Harvest: OFF"
harvestButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local sellButton = Instance.new("TextButton", frame)
sellButton.Size = UDim2.new(1, -20, 0, 40)
sellButton.Position = UDim2.new(0, 10, 0, 60)
sellButton.Text = "Auto Sell: OFF"
sellButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

-- Lista personalizada de frutas
local fruitList = {
	"Cenoura",
	"Mirtilo",
	"Laranjeira",
	"Caneta Azul",
	"Marshmallow",
	"Jambo",
	"Pistache",
	"Mango",
	"Uva"
}

-- Dropdown de frutas
local fruitDropdown = Instance.new("TextButton", frame)
fruitDropdown.Size = UDim2.new(1, -20, 0, 40)
fruitDropdown.Position = UDim2.new(0, 10, 0, 110)
fruitDropdown.Text = "Select Fruit"
fruitDropdown.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

local fruitMenu = Instance.new("Frame", frame)
fruitMenu.Size = UDim2.new(1, -20, 0, #fruitList * 25)
fruitMenu.Position = UDim2.new(0, 10, 0, 150)
fruitMenu.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
fruitMenu.Visible = false
Instance.new("UICorner", fruitMenu).CornerRadius = UDim.new(0, 6)

for i, fruit in ipairs(fruitList) do
	local b = Instance.new("TextButton", fruitMenu)
	b.Size = UDim2.new(1, 0, 0, 25)
	b.Position = UDim2.new(0, 0, 0, (i - 1) * 25)
	b.Text = fruit
	b.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	b.MouseButton1Click:Connect(function()
		selectedFruit = fruit
		fruitDropdown.Text = "Selected: " .. fruit
		fruitMenu.Visible = false
	end)
end

fruitDropdown.MouseButton1Click:Connect(function()
	fruitMenu.Visible = not fruitMenu.Visible
end)

-- Botão comprar fruta
local buyFruitButton = Instance.new("TextButton", frame)
buyFruitButton.Size = UDim2.new(1, -20, 0, 40)
buyFruitButton.Position = UDim2.new(0, 10, 0, 160 + #fruitList * 25)
buyFruitButton.Text = "Buy Fruit"
buyFruitButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

buyFruitButton.MouseButton1Click:Connect(function()
	if selectedFruit then
		ReplicatedStorage.Eventos.ComprarSemente:FireServer(selectedFruit)
	end
end)

-- Alternar funções
harvestButton.MouseButton1Click:Connect(function()
	harvesting = not harvesting
	harvestButton.Text = "Auto Harvest: " .. (harvesting and "ON" or "OFF")
end)

sellButton.MouseButton1Click:Connect(function()
	selling = not selling
	sellButton.Text = "Auto Sell: " .. (selling and "ON" or "OFF")
end)

-- Loop principal
task.spawn(function()
	while true do
		if harvesting then
			pcall(function()
				for _, v in pairs(workspace:GetDescendants()) do
					if v:IsA("ClickDetector") and v.Parent and v.Parent:IsA("Model") then
						fireclickdetector(v)
						task.wait(0.1)
					end
				end
			end)
		end

		if selling then
			pcall(function()
				ReplicatedStorage.Eventos.Vender["Vender inventário"]:FireServer()
			end)
		end

		task.wait(2)
	end
end)
