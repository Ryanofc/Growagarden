local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

local ScreenGui = Instance.new("ScreenGui", PlayerGui)
ScreenGui.Name = "AutoHarvestSellGui"
ScreenGui.ResetOnSpawn = false

-- Fundo preto transparente
local backgroundFrame = Instance.new("Frame", ScreenGui)
backgroundFrame.Size = UDim2.new(1, 0, 1, 0)
backgroundFrame.BackgroundColor3 = Color3.new(0, 0, 0)
backgroundFrame.BackgroundTransparency = 0.7
backgroundFrame.ZIndex = 0

-- Função botão de alternância
local function createToggleButton(name, position)
	local button = Instance.new("TextButton", ScreenGui)
	button.Size = UDim2.new(0, 120, 0, 40)
	button.Position = position
	button.Text = name .. ": OFF"
	button.BackgroundColor3 = Color3.new(0, 0, 0)
	button.BackgroundTransparency = 0.3
	button.TextColor3 = Color3.new(1, 1, 1)
	button.Font = Enum.Font.SourceSansBold
	button.TextSize = 18
	button.Name = name .. "Button"
	button.ZIndex = 2
	return button
end

-- Botões principais
local harvestButton = createToggleButton("Auto Harvest", UDim2.new(0, 10, 0, 100))
local sellButton = createToggleButton("Auto Sell", UDim2.new(0, 10, 0, 150))

-- Botão GUI toggle
local toggleGuiButton = Instance.new("TextButton", ScreenGui)
toggleGuiButton.Size = UDim2.new(0, 60, 0, 30)
toggleGuiButton.Position = UDim2.new(0, 10, 0, 10)
toggleGuiButton.Text = "GUI"
toggleGuiButton.BackgroundColor3 = Color3.new(0, 0, 0)
toggleGuiButton.BackgroundTransparency = 0.3
toggleGuiButton.TextColor3 = Color3.new(1, 1, 1)
toggleGuiButton.Font = Enum.Font.SourceSansBold
toggleGuiButton.TextSize = 18
toggleGuiButton.ZIndex = 2

-- Botão mostrar frutas
local showFruitsButton = Instance.new("TextButton", ScreenGui)
showFruitsButton.Size = UDim2.new(0, 120, 0, 40)
showFruitsButton.Position = UDim2.new(0, 10, 0, 200)
showFruitsButton.Text = "Show Fruits"
showFruitsButton.BackgroundColor3 = Color3.new(0, 0, 0)
showFruitsButton.BackgroundTransparency = 0.3
showFruitsButton.TextColor3 = Color3.new(1, 1, 1)
showFruitsButton.Font = Enum.Font.SourceSansBold
showFruitsButton.TextSize = 18
showFruitsButton.ZIndex = 2

-- Frame de frutas
local fruitsFrame = Instance.new("Frame", ScreenGui)
fruitsFrame.Size = UDim2.new(0, 140, 0, 300)
fruitsFrame.Position = UDim2.new(0, 140, 0, 60)
fruitsFrame.BackgroundColor3 = Color3.new(0, 0, 0)
fruitsFrame.BackgroundTransparency = 0.7
fruitsFrame.Visible = false
fruitsFrame.ZIndex = 2

local UIListLayout = Instance.new("UIListLayout", fruitsFrame)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 5)

local plantNames = {
	"Bamboo", "Corn", "Strawberry", "Carrot", "Potato", "Tomato", "Apple", "Orange", "Grape", "Pumpkin"
}

local selectedFruit = nil

local function createFruitButtons()
	for _, fruit in ipairs(plantNames) do
		local fruitButton = Instance.new("TextButton", fruitsFrame)
		fruitButton.Size = UDim2.new(1, 0, 0, 30)
		fruitButton.Text = fruit
		fruitButton.BackgroundColor3 = Color3.new(0, 0, 0)
		fruitButton.BackgroundTransparency = 0.3
		fruitButton.TextColor3 = Color3.new(1, 1, 1)
		fruitButton.Font = Enum.Font.SourceSans
		fruitButton.TextSize = 16
		fruitButton.ZIndex = 2

		fruitButton.MouseButton1Click:Connect(function()
			for _, btn in pairs(fruitsFrame:GetChildren()) do
				if btn:IsA("TextButton") then
					btn.BackgroundColor3 = Color3.new(0, 0, 0)
					btn.BackgroundTransparency = 0.3
				end
			end
			fruitButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
			fruitButton.BackgroundTransparency = 0
			selectedFruit = fruit
		end)
	end
end

createFruitButtons()

-- Botão Buy Fruit
local buyFruitButton = Instance.new("TextButton", ScreenGui)
buyFruitButton.Size = UDim2.new(0, 120, 0, 40)
buyFruitButton.Position = UDim2.new(0, 10, 0, 250)
buyFruitButton.Text = "Buy Fruit"
buyFruitButton.BackgroundColor3 = Color3.new(0, 0, 0)
buyFruitButton.BackgroundTransparency = 0.3
buyFruitButton.TextColor3 = Color3.new(1, 1, 1)
buyFruitButton.Font = Enum.Font.SourceSansBold
buyFruitButton.TextSize = 18
buyFruitButton.ZIndex = 2

local function buyFruit(fruitName)
	if not fruitName then
		warn("Nenhuma fruta selecionada!")
		return
	end

	local success, err = pcall(function()
		ReplicatedStorage.GameEvents.BuySeedStock:FireServer(fruitName)
	end)

	if not success then
		warn("Erro ao comprar fruta:", err)
	end
end

buyFruitButton.MouseButton1Click:Connect(function()
	buyFruit(selectedFruit)
end)

showFruitsButton.MouseButton1Click:Connect(function()
	fruitsFrame.Visible = not fruitsFrame.Visible
end)

-- Toggle GUI visibilidade
local guiVisible = true
toggleGuiButton.MouseButton1Click:Connect(function()
	guiVisible = not guiVisible
	for _, obj in pairs(ScreenGui:GetChildren()) do
		if obj:IsA("TextButton") or obj:IsA("Frame") then
			if obj ~= toggleGuiButton then
				obj.Visible = guiVisible
			end
		end
	end
end)

-- Toggle estados
local harvesting = false
local selling = false

local function toggleButton(button, state)
	button.Text = button.Name:gsub("Button", "") .. (state and ": ON" or ": OFF")
	button.BackgroundColor3 = state and Color3.fromRGB(30, 180, 60) or Color3.new(0, 0, 0)
	button.BackgroundTransparency = state and 0 or 0.3
end

harvestButton.MouseButton1Click:Connect(function()
	harvesting = not harvesting
	toggleButton(harvestButton, harvesting)
end)

sellButton.MouseButton1Click:Connect(function()
	selling = not selling
	toggleButton(sellButton, selling)
end)

-- Função real de colheita
local function harvestAllPlants()
	local inputGateway = player:WaitForChild("PlayerScripts"):WaitForChild("InputGateway")
	local activationRemote = inputGateway:WaitForChild("Activation")

	local plot = workspace:FindFirstChild("Plots") and workspace.Plots:FindFirstChild(player.Name)
	if not plot then return end

	for _, plant in pairs(plot:GetDescendants()) do
		if plant:IsA("BasePart") and plant.Name == "ClickPart" and plant:FindFirstChild("Plant") then
			local cf = plant.CFrame
			local args = { false, cf }
			activationRemote:FireServer(unpack(args))
			task.wait(0.1)
		end
	end
end

-- Loop de auto funções
task.spawn(function()
	while true do
		if harvesting then
			harvestAllPlants()
		end
		if selling then
			pcall(function()
				ReplicatedStorage.GameEvents.Sell_Inventory:FireServer()
			end)
		end
		task.wait(3)
	end
end)
