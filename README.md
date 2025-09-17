-- Đảm bảo game đã load
if not game:IsLoaded() then
    repeat task.wait() until game:IsLoaded()
end

-- Dịch vụ cần dùng
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer

-- Tạo folder nếu chưa có
local folder = "."
if makefolder then
    pcall(function() makefolder(folder) end)
end

-- Hàm lấy tên game
local function getGameName()
    local success, result = pcall(function()
        return game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name
    end)
    return success and result or "UNKNOWN"
end

-- Hàm lấy toàn bộ item trong Store (Inventory)
local function getInventoryItems()
    local itemList = {}
    local success, result = pcall(function()
        return ReplicatedStorage.Remotes.CommF_:InvokeServer("getInventory")
    end)
    if success and typeof(result) == "table" then
        for _, item in ipairs(result) do
            if item.Name then
                table.insert(itemList, item.Name)
            end
        end
    end
    return itemList
end

-- Hàm lấy Melee đang trang bị (trên tay hoặc trong Backpack)
local function getEquippedMelee()
    -- Kiểm tra tool đang cầm trên tay
    local char = player.Character
    if char then
        for _, tool in ipairs(char:GetChildren()) do
            if tool:IsA("Tool") then
                if tool.ToolTip == "Melee" or string.find(string.lower(tool.Name), "karate") or
                   string.find(string.lower(tool.Name), "style") or string.find(string.lower(tool.Name), "human") or
                   string.find(string.lower(tool.Name), "claw") or string.find(string.lower(tool.Name), "combat") then
                    return tool.Name
                end
            end
        end
    end
    -- Nếu không có trên tay, kiểm tra trong Backpack
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, tool in ipairs(backpack:GetChildren()) do
            if tool:IsA("Tool") then
                if tool.ToolTip == "Melee" or string.find(string.lower(tool.Name), "karate") or
                   string.find(string.lower(tool.Name), "style") or string.find(string.lower(tool.Name), "human") or
                   string.find(string.lower(tool.Name), "claw") or string.find(string.lower(tool.Name), "combat") then
                    return tool.Name
                end
            end
        end
    end
    return "Không có"
end

-- Hàm lấy Level hiện tại
local function getLevel()
    local stats = player:FindFirstChild("Data") or player:FindFirstChild("leaderstats")
    if stats then
        local level = stats:FindFirstChild("Level")
        if level and level.Value then
            return level.Value
        end
    end
    -- Nếu không có, thử lấy trực tiếp từ leaderstats
    if player:FindFirstChild("leaderstats") then
        local level = player.leaderstats:FindFirstChild("Level")
        if level and level.Value then
            return level.Value
        end
    end
    return "N/A"
end

-- Hàm lấy tên khu vực hiện tại (Blox Fruits)
local function getCurrentArea()
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return "Không xác định" end
    local minDist, areaName = math.huge, "Không xác định"
    local locations = workspace:FindFirstChild("_WorldOrigin") and workspace._WorldOrigin:FindFirstChild("Locations")
    if locations then
        for _,v in pairs(locations:GetChildren()) do
            if v:IsA("Part") then
                local dist = (hrp.Position - v.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    areaName = v.Name
                end
            end
        end
    end
    return areaName
end

-- Gửi dữ liệu mỗi 30 giây
while true do
    pcall(function()
        local data = {
            playerName = player.Name,
            game = getGameName(),
            time = os.date("!%Y-%m-%dT%H:%M:%SZ", workspace:GetServerTimeNow()),
            items = getInventoryItems(),
            meleeEquipped = getEquippedMelee(),
            level = getLevel(),
            area = getCurrentArea() -- Thêm dòng này
        }
        local filePath = folder .. "/" .. player.Name .. "_checkZAN.json"
        writefile(filePath, HttpService:JSONEncode(data))
        print("✅ Gửi thành công dữ liệu về: " .. filePath)
    end)
    wait(30)
end
