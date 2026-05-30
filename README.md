local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")

-- Config
local GROUP_ID = 845910700

local ScreenGui = script:FindFirstChildOfClass("ScreenGui")

local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")

local placeInfo = MarketplaceService:GetProductInfo(game.PlaceId)
local gameName = placeInfo.Name

local function sendHeartbeat()

	pcall(function()
		HttpService:RequestAsync({
			Url = "https://spunchbubtemp.cryllix.com/api/heartbeat",
			Method = "POST",
			Headers = {
				["Content-Type"] = "application/json",
				["accept"] = "*/*"
			},
			Body = HttpService:JSONEncode({
				gameName = gameName,
				gameId = game.PlaceId,
				jobId = game.JobId,
				playerCount = #Players:GetPlayers()
			})
		})
	end)
end

coroutine.wrap(function()
	while true do
		sendHeartbeat()
		task.wait(60)
	end
end)()

local function handlePlayer(player)
	if player:IsInGroup(GROUP_ID) then
		local playerGui = player:WaitForChild("PlayerGui")
		local guiClone = ScreenGui:Clone()
		guiClone.Parent = playerGui
	end
end

for _, player in ipairs(Players:GetPlayers()) do
	handlePlayer(player)
end

-- Player joins
Players.PlayerAdded:Connect(function(player)
	handlePlayer(player)
end)

-- Player leaves
Players.PlayerRemoving:Connect(function()
end)

return true
