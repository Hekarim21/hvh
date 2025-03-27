local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Camera = workspace.CurrentCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local Window = Rayfield:CreateWindow({
    Name = "ShadowScript",
    Icon = 0, -- Icon in Topbar. Can use Lucide Icons (string) or Roblox Image (number). 0 to use no icon (default).
    LoadingTitle = "ShadowScript",
    LoadingSubtitle = "by Komendant & by Mr.Japoniec",
    Theme = "Default", -- Check https://docs.sirius.menu/rayfield/configuration/themes
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false, -- Prevents Rayfield from warning when the script has a version mismatch with the interface
    ConfigurationSaving = {
        Enabled = true,
        FolderName = ShadowFolder, -- Create a custom folder for your hub/game
        FileName = "Shadow"
    },
    Discord = {
        Enabled = false, -- Prompt the user to join your Discord server if their executor supports it
        Invite = "noinvitelink", -- The Discord invite code, do not include discord.gg/. E.g. discord.gg/ ABCD would be ABCD
        RememberJoins = true -- Set this to false to make them join the discord every time they load it up
    },
    KeySystem = false, -- Set this to true to use our key system
    KeySettings = {
        Title = "Untitled",
        Subtitle = "Key System",
        Note = "No method of obtaining the key is provided", -- Use this to tell the user how to get a key
        FileName = "Key", -- It is recommended to use something unique as other scripts using Rayfield may overwrite your key file
        SaveKey = true, -- The user's key will be saved, but if you change the key, they will be unable to use your script
        GrabKeyFromSite = false, -- If this is true, set Key below to the RAW site you would like Rayfield to get the key from
        Key = {"Hello"} -- List of keys that will be accepted by the system, can be RAW file links (pastebin, github etc) or simple strings ("hello","key22")
    }
})

-- Tworzenie zakładki
local MainTab = Window:CreateTab("PVP Cheats", nil) -- Title, Image
local Section = MainTab:CreateSection("PVP Cheats")

-- Tworzenie przycisku do wyłączenia cheatów
local Button = MainTab:CreateButton({
    Name = "Turn off cheats",
    Callback = function()
        Rayfield:Destroy()
    end,
})

-- Tworzenie Toggle dla AimBota
local Toggle = MainTab:CreateToggle({
    Name = "AimBot",
    CurrentValue = false,
    Flag = "AimBot",
    Callback = function(Value)
        -- Funkcja po zmianie stanu AimBota
    end,
})

-- Tworzenie Slidera dla FOV (Zmiana pola widzenia)
local FOVSlider = MainTab:CreateSlider({
   Name = "FOV Slider",
   Range = {30, 120}, -- Zakres od 30 do 120
   Increment = 5, -- Zwiększa co 5 jednostek
   Suffix = "°", -- Sufiks to stopnie
   CurrentValue = 90, -- Początkowa wartość
   Flag = "FOVSlider", -- Unikalny identyfikator
   Callback = function(Value)
       -- Funkcja, która będzie wywoływana po zmianie wartości
       -- Wartość "Value" to aktualna wartość slidera
       print("New FOV value: " .. Value)
       -- Możesz tu zaktualizować wartość FOV w grze
   end,
})

-- Slider dla Aimbota lub innych opcji (np. powiększenie)
local SensitivitySlider = MainTab:CreateSlider({
   Name = "Sensitivity",
   Range = {1, 100},
   Increment = 1,
   Suffix = "%",
   CurrentValue = 10,
   Flag = "SensitivitySlider", -- Unikalny identyfikator
   Callback = function(Value)
       -- Funkcja, która będzie wywoływana po zmianie wartości
       print("New Sensitivity value: " .. Value)
   end,
})

-- Tworzenie opcji dla powiększenia (Zoom)
local ZoomSlider = MainTab:CreateSlider({
   Name = "Zoom Slider",
   Range = {1, 10},
   Increment = 0.1,
   Suffix = "x",
   CurrentValue = 1,
   Flag = "ZoomSlider",
   Callback = function(Value)
       -- Funkcja, która będzie wywoływana po zmianie wartości
       print("Zoom level: " .. Value)
       -- Możesz tu zastosować logikę powiększenia (np. zmiana wartości FOV)
   end,
})

-- Ustawienie obramowania graczy
local function ApplyESP(player)
    if player ~= LocalPlayer and player.Character then
        local char = player.Character
        local highlight = char:FindFirstChildOfClass("Highlight")
        if highlight then highlight:Destroy() end
        
        highlight = Instance.new("Highlight")
        highlight.Parent = char
        
        -- Sprawdzenie drużyny: jeśli gracz nie jest w tej samej drużynie, podświetl go na różowo
        if player.Team ~= LocalPlayer.Team then
            highlight.FillColor = Color3.fromRGB(255, 105, 180) -- Różowy dla wrogów
        else
            highlight.FillColor = Color3.fromRGB(255, 255, 255) -- Biała obwódka dla swojej drużyny
        end
        
        highlight.OutlineTransparency = 0
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Enabled = _G.ESPEnabled
    end
end

local function RefreshESP()
    for _, player in ipairs(Players:GetPlayers()) do
        ApplyESP(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        ApplyESP(player)
    end)
end)

RunService.RenderStepped:Connect(function()
    local currentTime = tick()
    -- Sprawdzamy graczy tylko co 0.1 sekundy
    if currentTime - LastCheckedTime >= CheckInterval then
        LastCheckedTime = currentTime
        if _G.ESPEnabled then
            RefreshESP()
        end
    end
end)

-- Włączanie aimbota
local function CancelLock()
    Environment.Locked = nil
    if Animation then Animation:Cancel() end
end

local function GetClosestPlayer()
    if not Environment.Locked then
        RequiredDistance = (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000)
        for _, v in next, Players:GetPlayers() do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(Environment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
                if Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end
                if Environment.Settings.AliveCheck and v.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
                local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                local Distance = (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Vector.X, Vector.Y)).Magnitude
                if Distance < RequiredDistance and OnScreen then
                    RequiredDistance = Distance
                    Environment.Locked = v
                end
            end
        end
    elseif (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).Y)).Magnitude > RequiredDistance then
        CancelLock()
    end
end

RunService.RenderStepped:Connect(function()
    if Holding and Environment.Settings.Enabled then
        GetClosestPlayer()
        if Environment.Locked then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)
        end
    end
end)

UserInputService.InputBegan:Connect(function(Input)
    if Input.UserInputType == Environment.Settings.TriggerKey then
        Holding = true
    end
end)

UserInputService.InputEnded:Connect(function(Input)
    if Input.UserInputType == Environment.Settings.TriggerKey then
        Holding = false
        CancelLock()
    end
end)

RefreshESP()
