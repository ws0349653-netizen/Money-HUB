--====================================================
-- Money Hub - Ultimate Boxing Game
-- Script ÚNICO | Freeze Trade ON/OFF | Simulator | UI
--====================================================

--========================
-- SERVIÇOS
--========================
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

--========================
-- CONFIG
--========================
local ADMINS = {
	["WilliamSilva"] = true, -- ALTERE para seu username real
}

--========================
-- ESTADO GLOBAL
--========================
local TradeFrozen = false

--========================
-- REMOTES
--========================
local Remotes = ReplicatedStorage:FindFirstChild("MoneyHubRemotes")
if not Remotes then
	Remotes = Instance.new("Folder")
	Remotes.Name = "MoneyHubRemotes"
	Remotes.Parent = ReplicatedStorage
end

local ToggleFreeze = Instance.new("RemoteEvent", Remotes)
ToggleFreeze.Name = "ToggleFreeze"

local RequestState = Instance.new("RemoteFunction", Remotes)
RequestState.Name = "RequestFreezeState"

local SimulateTrade = Instance.new("RemoteEvent", Remotes)
SimulateTrade.Name = "SimulateTrade"

local TradeResult = Instance.new("RemoteEvent", Remotes)
TradeResult.Name = "TradeResult"

--========================
-- FREEZE TRADE ON / OFF
--========================
ToggleFreeze.OnServerEvent:Connect(function(player)
	if not ADMINS[player.Name] then
		warn("[Money Hub] Acesso negado:", player.Name)
		return
	end

	TradeFrozen = not TradeFrozen

	print("[Money Hub] Freeze Trade:", TradeFrozen and "ON" or "OFF")

	-- Atualiza TODOS os admins conectados
	for _, plr in pairs(Players:GetPlayers()) do
		TradeResult:FireClient(plr, TradeFrozen)
	end
end)

RequestState.OnServerInvoke = function()
	return TradeFrozen
end

--========================
-- EXEMPLO DE BLOQUEIO REAL
-- (integre com o sistema de trade do jogo)
--========================
ReplicatedStorage:SetAttribute("CanTrade", function()
	return not TradeFrozen
end)

--========================
-- TRADE SIMULATOR
--========================
SimulateTrade.OnServerEvent:Connect(function(player, item)
	task.wait(0.5)
	TradeResult:FireClient(player, TradeFrozen)
end)

--========================
-- UI (GERADA AUTOMATICAMENTE)
--========================
local function createUI(player)
	local gui = Instance.new("ScreenGui")
	gui.Name = "MoneyHubUI"
	gui.ResetOnSpawn = false
	gui.Parent = player:WaitForChild("PlayerGui")

	local frame = Instance.new("Frame", gui)
	frame.Size = UDim2.fromScale(0.32, 0.36)
	frame.Position = UDim2.fromScale(0.34, 0.32)
	frame.BackgroundColor3 = Color3.fromRGB(18,18,18)
	frame.BorderSizePixel = 0
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 14)

	local title = Instance.new("TextLabel", frame)
	title.Size = UDim2.fromScale(1, 0.2)
	title.BackgroundTransparency = 1
	title.Text = "Money Hub"
	title.Font = Enum.Font.GothamBold
	title.TextScaled = true
	title.TextColor3 = Color3.new(1,1,1)

	local freezeBtn = Instance.new("TextButton", frame)
	freezeBtn.Size = UDim2.fromScale(0.8, 0.22)
	freezeBtn.Position = UDim2.fromScale(0.1, 0.35)
	freezeBtn.Font = Enum.Font.GothamBold
	freezeBtn.TextScaled = true
	freezeBtn.BackgroundColor3 = Color3.fromRGB(35,35,35)
	freezeBtn.TextColor3 = Color3.new(1,1,1)

	local simBtn = Instance.new("TextButton", frame)
	simBtn.Size = UDim2.fromScale(0.8, 0.22)
	simBtn.Position = UDim2.fromScale(0.1, 0.65)
	simBtn.Text = "Simulate Trade"
	simBtn.Font = Enum.Font.Gotham
	simBtn.TextScaled = true
	simBtn.BackgroundColor3 = Color3.fromRGB(35,35,35)
	simBtn.TextColor3 = Color3.new(1,1,1)

	--========================
	-- FUNÇÕES UI
	--========================
	local function updateButton(state)
		if state then
			freezeBtn.Text = "Freeze Trade: ON"
			freezeBtn.BackgroundColor3 = Color3.fromRGB(120, 30, 30)
		else
			freezeBtn.Text = "Freeze Trade: OFF"
			freezeBtn.BackgroundColor3 = Color3.fromRGB(30, 120, 30)
		end
	end

	updateButton(RequestState:InvokeServer())

	--========================
	-- CONEXÕES
	--========================
	freezeBtn.MouseButton1Click:Connect(function()
		ToggleFreeze:FireServer()
	end)

	simBtn.MouseButton1Click:Connect(function()
		SimulateTrade:FireServer("Test Item")
	end)

	TradeResult.OnClientEvent:Connect(function(state)
		updateButton(state)
	end)
end

--========================
-- PLAYER ADDED
--========================
Players.PlayerAdded:Connect(function(player)
	task.wait(1)
	createUI(player)
end)

print("[Money Hub] Script carregado com sucesso.")
