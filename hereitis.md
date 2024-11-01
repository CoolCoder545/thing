--- For the gamepasses! ---
local MPS = game:GetService("MarketplaceService")
local rstorage = game:GetService("ReplicatedStorage")
local pets = rstorage.Pets
local players = game:GetService("Players")
products = {
	
	
}
--- when u purchase pass ---
local passPurchases = {

}
print(1)

players.PlayerAdded:Connect(function(plr)
	task.wait(4)

	for passId, func in passPurchases do
		local owned = MPS:UserOwnsGamePassAsync(plr.UserId, passId)
		if owned then
			func(plr)
		end
	end
end)

--- processing wot the gamepass does ---
function process(info)
	
	task.wait()
	print("Product Purchased")
	local player = players:GetPlayerByUserId(info.PlayerId)

	if not player then print("Player Left") return end

	if not products[info.ProductId] then print("invalid id:", info.ProductId) return end

	products[info.ProductId](player)
	return Enum.ProductPurchaseDecision.PurchaseGranted
end
--- da ID's for the dev products and/or gamepasses --- 
MPS.ProcessReceipt = process

local premprods = require(rstorage:WaitForChild("InformationModules"):WaitForChild("PremiumProducts"))

passPurchases = {
	[premprods.Gamepasses.FasterAutoClicker or nil] = function(plr)
		plr:SetAttribute("FasterAutoClick", true)
	end,
	[premprods.Gamepasses.DoubleTaps or 662583100] = function(plr)
		plr:SetAttribute("DoubleTaps", true)
	end,
	[premprods.Gamepasses.DoubleGems or 662583100] = function(plr)
		plr:SetAttribute("DoubleGems", true)
	end,
	[premprods.Gamepasses.SuperLuck or 951195992] = function(plr)
		plr:SetAttribute("SuperLuck", true)
	end,
	[premprods.Gamepasses.MoreInventory or 951472173] = function(plr)
		plr:SetAttribute("MoreInventory", true)
	end,
	[premprods.Gamepasses.PlusSixPets or 951517775] = function(plr)
		plr:SetAttribute("6Pets", true)
	end,
	[premprods.Gamepasses.infPets or 951355774] = function(plr)
		print(plr.Name)
		plr:SetAttribute("infPets", true)
	end,
	[premprods.Gamepasses.PlusThreePets or 951104133] = function(plr)
		plr:SetAttribute("3Pets", true)
	end,
	[premprods.Gamepasses.VIP or 951433720] = function(plr)
		plr:SetAttribute("VIP", true)
	end,
	[premprods.Gamepasses.TripleHatch or 951782257] = function(plr)
		plr:SetAttribute("TripleHatch", true)
	end,
	[premprods.Gamepasses.AutoHatch or 951365695] = function(plr)
		plr:SetAttribute("AutoHatch", true)
	end,
}
--- the functions for all the gamepasses/dev products ---
local pets = rstorage.Pets
local players = game:GetService("Players")
local PotionTime = 60*15
local SeasonPassFolder = rstorage:WaitForChild("Season Pass")
local PassInformation = require(SeasonPassFolder:WaitForChild("SeasonPassInformation"))
function give_pet(player, pet)

	Instance.new("StringValue", player.Pets).Name = pet
	rstorage.SendPet:FireClient(player, pet, true)
end
products = {
	[premprods.DevProducts.DailySpins5] = function(plr)
	plr.leaderstats.Spins.Value += 5
	end,
	[premprods.DevProducts.DailySpins20] = function(plr)
		plr.leaderstats.Spins.Value += 20
	end,
	[premprods.DevProducts.GemPotion] = function(plr)
		plr.Boosts["Gem Potion"].Value += PotionTime
	end,
	[premprods.DevProducts.TapPotion] = function(plr)
		plr.Boosts["Tap Potion"].Value += PotionTime
	end,
	[premprods.DevProducts.GodPotion] = function(plr)
		plr.Boosts["God Potion"].Value += PotionTime
	end,
	[premprods.DevProducts.GoldPotion] = function(plr)
		plr.Boosts["Golden Hatch"].Value += PotionTime
	end,
	[premprods.DevProducts.LuckPotion] = function(plr)
		plr.Boosts["Super Lucky"].Value += PotionTime
	end,
	[premprods.DevProducts.DoubleMyGems] = function(plr)
		_G.FullAdd(plr,plr.leaderstats.Gems.Value, "Gems")
	end,
	[premprods.DevProducts.DoubleMyTaps] = function(plr)
		_G.FullAdd(plr, plr.leaderstats.Taps.Value, "Taps")
		--plr.leaderstats.Taps.Value *= 2
		
	end,
	[premprods.DevProducts.DoubleMyRebirths] = function(plr)
		_G.FullAdd(plr,plr.leaderstats.Rebirths.Value, "Rebirths")
	end,

	[PassInformation.Products.SkipAll] = function(plr)
		plr.SeasonPass.Level.Value = PassInformation.MaxLevel
	end,
	[PassInformation.Products.Restart] = function(plr)
		plr.SeasonPass.Level.Value = 0
		plr.SeasonPass.XP.Value = 0
		plr.Claimed:ClearAllChildren()
	end,
	[premprods.DevProducts.Titanic] = function(plr)
		give_pet(plr, "Titanic Fire Defender")
	end,
}

for i, productID in PassInformation.Products.Skips do
	if products[productID] then continue end
	products[productID] = function(plr)
		print(i)
		plr.SeasonPass.Level.Value += i
	end
end


function get_luck(player)

	local Luck = 1

	for boost, luck_added in {["Extra Lucky"] = 1, ["Super Lucky"] = 2, ["Ultra Lucky"] = 4} do
		local lucky = player.leaderstats:FindFirstChild(boost)
		if not lucky  then continue end

		if lucky.Value > 1 then
			Luck += luck_added
		end
	end
	if player.leaderstats:FindFirstChild("Luck") then
		Luck += (player.leaderstats["Luck"].Value * .5) 
	end

	return math.round(Luck)
end

local pet_settings = require(rstorage:WaitForChild("InformationModules"):WaitForChild("PetSettings"))

function hatch_pet(player, egg)
	local Luck = get_luck(player)

	local chosenPet, chance = _G.choosePetWithLuck(egg, Luck)
	local base = chosenPet

	local Golden, Diamond, Rainbow = _G.get_craft_boosts(player)
	chosenPet = (Diamond and rstorage.Pets:FindFirstChild("Diamond "..chosenPet, true)) and "Diamond "..chosenPet or (Rainbow and rstorage.Pets:FindFirstChild("Rainbow "..chosenPet, true)) and "Rainbow "..chosenPet or (Golden and rstorage.Pets:FindFirstChild("Golden "..chosenPet, true)) and "Golden "..chosenPet or chosenPet
	local rarity, theme = pet_settings.getRarityInfo(chance)
	local delete = false

	local eggs_hatched = player.leaderstats:FindFirstChild("Eggs Hatched")
	if eggs_hatched then
		_G.FullAdd(player, 1, "Eggs Hatched")
	end

	local index = player["Pet Index"]:FindFirstChild(base)

	if not index then

		Instance.new("StringValue", player["Pet Index"]).Name = base
	end

	if not delete then
		local val =Instance.new("StringValue")
		val.Name = chosenPet
		val.Value = chosenPet
		val.Parent = player.Pets
		_G.pet_hatch_counts[base] = _G.pet_hatch_counts[base] and (_G.pet_hatch_counts[base] + 1) or 1
	end
	return chosenPet
end


MPS.PromptGamePassPurchaseFinished:Connect(function(player, passId, purchased)
	
	if not player or not purchased then return end
	local func = passPurchases[passId]
	if func then
		func(player)
	end
end)

for _, egg in workspace.Eggs:GetChildren() do

	local id = egg:FindFirstChild("ProductId")
	local id3 = egg:FindFirstChild("ProductId3")

	if id then
	
		products[id.Value] = function(player)
			local eggs_hatched = player.Data:FindFirstChild("Eggs Hatched")
			if eggs_hatched then
				_G.FullAdd(player, 1, "Eggs Hatched")
			end

			
			local pet = hatch_pet(player, egg)
			rstorage.EggHatchingRemote.PromptHatch:FireClient(player, egg, {pet})
			
		end
	end

	if id3 then

		products[id3.Value] = function(player)
			local eggs_hatched = player.Data:FindFirstChild("Eggs Hatched")
			if eggs_hatched then
				_G.FullAdd(player, 3, "Eggs Hatched")
			end

			rstorage.EggHatchingRemote.PromptHatch:FireClient(player,egg, {hatch_pet(player, egg), hatch_pet(player, egg), hatch_pet(player, egg)})
		end
	end
end
