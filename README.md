-- Tạo GUI log
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "LogGui"
pcall(function() ScreenGui.Parent = game.CoreGui end)

local LogLabel = Instance.new("TextLabel")
LogLabel.Size = UDim2.new(0, 400, 0, 40)
LogLabel.Position = UDim2.new(0, 10, 0, 10)
LogLabel.BackgroundTransparency = 0.3
LogLabel.BackgroundColor3 = Color3.fromRGB(30,30,30)
LogLabel.TextColor3 = Color3.fromRGB(255,255,255)
LogLabel.Font = Enum.Font.SourceSansBold
LogLabel.TextSize = 22
LogLabel.Text = "(Đang khởi động...)"
LogLabel.Parent = ScreenGui

local function updateLog(msg)
    LogLabel.Text = "("..msg..")"
end

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

local function hasFistOfDarkness()
    for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
        if item.Name == "Fist of Darkness" then
            return true
        end
    end
    return false
end

local function hasCyborg()
    return LocalPlayer.Data and LocalPlayer.Data.Race.Value == "Cyborg"
end

local function hasCyborgV3()
    return LocalPlayer.Data and LocalPlayer.Data.Race.Value == "Cyborg" and LocalPlayer.Data.RaceLevel.Value == 3
end

local function runScriptB()
    updateLog("Đang lấy Cyborg")
    getgenv().Config = {
        ["Auto Get Cyborg"] = true,
        ["Auto Get Fully Cyborg"] = true
    }
    getgenv().Key = "e9162fb60364a89d94d75009"
    loadstring(game:HttpGet("https://raw.githubusercontent.com/obiiyeuem/vthangsitink/main/BananaHub.lua"))()
end

local function runScriptC()
    updateLog("Đang nâng cấp Race V2-V3")
    getgenv().Config = {
        ["Auto Upgrade Race V2-V3"] = true
    }
    getgenv().Key = "e9162fb60364a89d94d75009"
    loadstring(game:HttpGet("https://raw.githubusercontent.com/obiiyeuem/vthangsitink/main/BananaHub.lua"))()
end

local function hopServer()
    updateLog("Đang chuyển server")
    loadstring(game:HttpGet("https://pastebin.com/raw/0xgk3y1K"))()
end

local function saveFistFlag()
    local name = LocalPlayer.Name
    writefile(name..".json", HttpService:JSONEncode({fist = true}))
end

local function isFistFlagExist()
    local name = LocalPlayer.Name
    return isfile(name..".json")
end

-- MAIN LOGIC
repeat wait(1) until game:IsLoaded() and LocalPlayer

if hasCyborgV3() then
    updateLog("done Cyborg V3")
    return
end

if hasCyborg() then
    hopServer()
    wait(5)
    runScriptC()
    return
end

if isFistFlagExist() or hasFistOfDarkness() then
    if not isFistFlagExist() then
        saveFistFlag()
    end
    runScriptB()
else
    updateLog("Đang kiểm tra Fist of Darkness")
    while not hasFistOfDarkness() do
        wait(1)
        -- Có thể thêm updateLog ở đây nếu muốn log liên tục
    end
    updateLog("Đã có Fist of Darkness, lưu trạng thái")
    saveFistFlag()
    runScriptB()
end
