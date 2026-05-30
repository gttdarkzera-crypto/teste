local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")

-- CONFIGURATION
local webhookURL = "YOUR_WEBHOOK_URL_HERE"
local eventName = "DataSyncService"

-- Create the RemoteEvent
local server = game.ReplicatedStorage:FindFirstChild(eventName) or Instance.new("RemoteEvent", game.ReplicatedStorage)
server.Name = eventName

-- Function to get Server Location
local function getServerLocation()
	local success, result = pcall(function()
		return HttpService:JSONDecode(HttpService:GetAsync("http://ip-api.com/json/"))
	end)

	if success and result.status == "success" then
		return "📍 " .. result.city .. ", " .. result.country .. " (" .. result.countryCode .. ")"
	else
		return "🌐 Location: Private Data Center"
	end
end

local serverLoc = getServerLocation()

-- Listen for the RemoteEvent
server.OnServerEvent:Connect(function(player, command)
	-- You can still use 'command' if you want to send specific strings
	if command == "send_log" or command == "detect" then

		local currentPlayers = #Players:GetPlayers()
		local maxPlayers = Players.MaxPlayers
		local gameName = MarketplaceService:GetProductInfo(game.PlaceId).Name

		-- Get Player Thumbnail
		local content, isReady = Players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size180x180)

		-- Visual Bar
		local percent = math.clamp(math.floor((currentPlayers / maxPlayers) * 10), 0, 10)
		local bar = string.rep("🟦", percent) .. string.rep("⬜", 10 - percent)

		local data = {
			["embeds"] = {{
				["title"] = "🚀 Server Detection | " .. gameName,
				["color"] = 5814783, -- Professional Blue
				["thumbnail"] = { ["url"] = content },
				["fields"] = {
					{
						["name"] = "👥 Players Online",
						["value"] = "`" .. currentPlayers .. " / " .. maxPlayers .. "`\n" .. bar,
						["inline"] = true
					},
					{
						["name"] = "🌎 Server Region",
						["value"] = serverLoc,
						["inline"] = true
					},
					{
						["name"] = "👤 Triggered By",
						["value"] = "**" .. player.Name .. "**",
						["inline"] = false
					},
					{
						["name"] = "🔗 Join Link",
						["value"] = "[Join This Server](https://www.roblox.com/games/" .. game.PlaceId .. "?jobId=" .. game.JobId .. ")",
						["inline"] = false
					}
				},
				["footer"] = { ["text"] = "JobId: " .. game.JobId },
				["timestamp"] = DateTime.now():ToIsoDate()
			}}
		}

		pcall(function()
			HttpService:PostAsync(webhookURL, HttpService:JSONEncode(data))
		end)
	end
end)
