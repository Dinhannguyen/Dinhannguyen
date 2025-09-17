-- Script kiểm tra Race và phiên bản (V1, V2, V3) trong Blox Fruits
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Tạo GUI cố định
local function createGUI()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "RaceCheckGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 200, 0, 100) -- GUI nhỏ gọn
    Frame.Position = UDim2.new(0.02, 0, 0.3, 0) -- Góc trái, giữa phần trên
    Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Frame.BackgroundTransparency = 0.2
    Frame.BorderSizePixel = 2
    Frame.BorderColor3 = Color3.fromRGB(0, 170, 255)
    Frame.Parent = ScreenGui

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 25)
    Title.Position = UDim2.new(0, 0, 0, 0)
    Title.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Text = "Race Check"
    Title.Font = Enum.Font.GothamBlack -- Phông mập mạp
    Title.TextSize = 16
    Title.Parent = Frame

    local RaceLabel = Instance.new("TextLabel")
    RaceLabel.Name = "RaceLabel"
    RaceLabel.Size = UDim2.new(1, -10, 0, 25)
    RaceLabel.Position = UDim2.new(0, 5, 0, 30)
    RaceLabel.BackgroundTransparency = 1
    RaceLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    RaceLabel.Text = "Race: Loading..."
    RaceLabel.Font = Enum.Font.GothamBlack
    RaceLabel.TextSize = 14
    RaceLabel.TextXAlignment = Enum.TextXAlignment.Left
    RaceLabel.Parent = Frame

    local VersionLabel = Instance.new("TextLabel")
    VersionLabel.Name = "VersionLabel"
    VersionLabel.Size = UDim2.new(1, -10, 0, 25)
    VersionLabel.Position = UDim2.new(0, 5, 0, 55)
    VersionLabel.BackgroundTransparency = 1
    VersionLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    VersionLabel.Text = "Version: Loading..."
    VersionLabel.Font = Enum.Font.GothamBlack
    VersionLabel.TextSize = 14
    VersionLabel.TextXAlignment = Enum.TextXAlignment.Left
    VersionLabel.Parent = Frame

    return RaceLabel, VersionLabel
end

-- Hàm lấy thông tin Race và Version
local function getRaceAndVersion()
    local race, version = "Không tìm thấy", "Không tìm thấy"

    -- Check LocalPlayer.Data.Race
    local data = LocalPlayer:FindFirstChild("Data")
    if data and data:FindFirstChild("Race") then
        race = data.Race.Value
    end

    -- Check qua CommF_ (nếu cần)
    local success, inventory = pcall(function()
        return ReplicatedStorage.Remotes.CommF_:InvokeServer("getInventory")
    end)
    if success and inventory then
        for _, item in pairs(inventory) do
            if type(item) == "table" and item.Name:find("Awakened") then
                if item.Name:find("V3") then
                    version = "V3"
                elseif item.Name:find("V2") then
                    version = "V2"
                elseif item.Name:find("V1") then
                    version = "V1"
                end
            end
        end
    end

    -- Nếu không tìm thấy version, giả sử V1 nếu race tồn tại
    if race ~= "Không tìm thấy" and version == "Không tìm thấy" then
        version = "V1" -- Mặc định V1 nếu chưa awaken
    end

    return race, version
end

-- Hàm chính
local function main()
    local raceLabel, versionLabel = createGUI()

    -- Loop cập nhật GUI
    task.spawn(function()
        while true do
            local success, race, version = pcall(getRaceAndVersion)
            if success then
                raceLabel.Text = "Race: " .. tostring(race)
                versionLabel.Text = "Version: " .. tostring(version)
                print("=== Race Check ===")
                print("Race: " .. tostring(race))
                print("Version: " .. tostring(version))
            else
                raceLabel.Text = "Race: Error"
                versionLabel.Text = "Version: Error"
                print("Lỗi khi lấy race/version, thử lại...")
            end
            task.wait(2) -- Update mỗi 2s
        end
    end)
end

-- Gọi hàm chính
local success, err = pcall(main)
if not success then
    print("Lỗi khởi tạo script: " .. err)
end
