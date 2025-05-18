local Players = game:GetService("Players")
local player = Players.LocalPlayer
local mouse = player:GetMouse()

mouse.Button1Down:Connect(function()
	local target = mouse.Target
	if target then
		print(">> Você clicou em:", target.Name)
		print(">> Caminho completo:", target:GetFullName())
	end
end)
