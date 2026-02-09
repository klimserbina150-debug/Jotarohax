local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local Config = {
    Aimbot = false,
    ESP_Chams = false,
    ESP_Boxes = false,
    TeamCheck = true,
    FOV = 150,
    Dist = 500,
    Sticky = 0.3,
    Speed = 16
}

-- КИК ЭКРАН (БЕЗ ЗВУКА, ЧИСТАЯ СТАБИЛЬНОСТЬ)
local function ShowKickScreen()
    local kg = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
    kg.DisplayOrder = 999999
    local f = Instance.new("Frame", kg)
    f.Size, f.BackgroundColor3, f.Active = UDim2.new(1,0,1,0), Color3.new(0,0,0), true
    
    local t = Instance.new("TextLabel", f)
    t.Size, t.BackgroundTransparency, t.TextColor3, t.Font, t.TextSize = UDim2.new(1,0,1,0), 1, Color3.new(1,0,0), 3, 28
    t.TextWrapped = true
    
    task.spawn(function()
        for i = 5, 1, -1 do
            t.Text = "ПОШЕЛ НАФИГ, ПИДОРАС!\n\nХОЧЕШЬ КЛЮЧ? ПИШИ В ТГ: @nooblo2\n\nВЫЛЕТ ЧЕРЕЗ: " .. i
            task.wait(1)
        end
        LocalPlayer:Kick("\n[JOTAROHAX]\nПошел нафиг, пидорас!\nTG: @nooblo2")
    end)
end

-- ПРОВЕРКА КЛЮЧЕЙ
local function Verify(txt)
    local k = txt:gsub("%s+", "")
    local ok = false
    if k == "JOTAROVIP" then ok = true end
    local times = {4, 5, 9, 24, 48}
    for _, h in pairs(times) do
        for i = 1, 40 do 
            if k == "J"..h.."H_"..i then ok = true break end 
        end
    end
    if not ok then ShowKickScreen() return false end
    return true
end

-- ФУНКЦИИ ЧИТА
local function Launch()
    local MainGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
    MainGui.ResetOnSpawn = false

    local Menu = Instance.new("Frame", MainGui)
    Menu.Size, Menu.Position = UDim2.new(0, 300, 0, 380), UDim2.new(0.5, -150, 0.3, 0)
    Menu.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    Menu.Active, Menu.Draggable = true, true
    Instance.new("UICorner", Menu)
    Instance.new("UIStroke", Menu).Color = Color3.fromRGB(130, 0, 255)

    local function CreateBtn(name, y, callback)
        local b = Instance.new("TextButton", Menu)
        b.Size, b.Position = UDim2.new(0, 260, 0, 40), UDim2.new(0.5, -130, 0, y)
        b.Text, b.BackgroundColor3 = name, Color3.fromRGB(30,30,30)
        b.TextColor3, b.Font = Color3.new(1,1,1), 3
        Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function() callback(b) end)
    end

    CreateBtn("AIMBOT: OFF", 30, function(b)
        Config.Aimbot = not Config.Aimbot
        b.Text = Config.Aimbot and "AIMBOT: ON" or "AIMBOT: OFF"
    end)

    CreateBtn("CHAMS: OFF", 80, function(b)
        Config.ESP_Chams = not Config.ESP_Chams
        b.Text = Config.ESP_Chams and "CHAMS: ON" or "CHAMS: OFF"
    end)

    CreateBtn("BOXES: OFF", 130, function(b)
        Config.ESP_Boxes = not Config.ESP_Boxes
        b.Text = Config.ESP_Boxes and "BOXES: ON" or "BOXES: OFF"
    end)

    CreateBtn("SPEED: 16", 180, function(b)
        Config.Speed = (Config.Speed == 16) and 80 or 16
        b.Text = "SPEED: " .. Config.Speed
    end)

    CreateBtn("CLOSE", 320, function() MainGui:Destroy() end)

    RunService.RenderStepped:Connect(function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = Config.Speed
        end

        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character then
                -- CHAMS
                local h = v.Character:FindFirstChild("JHighlight")
                if Config.ESP_Chams and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
                    if not h then h = Instance.new("Highlight", v.Character); h.Name = "JHighlight" end
                    h.FillColor = (v.Team == LocalPlayer.Team and Color3.new(0,1,0) or Color3.new(1,0,0))
                elseif h then h:Destroy() end
                
                -- BOXES
                local hrp = v.Character:FindFirstChild("HumanoidRootPart")
                local b = hrp and hrp:FindFirstChild("JBox")
                if Config.ESP_Boxes and hrp and v.Character.Humanoid.Health > 0 then
                    if not b then
                        b = Instance.new("BillboardGui", hrp); b.Name = "JBox"; b.AlwaysOnTop = true; b.Size = UDim2.new(4,0,5,0)
                        local f = Instance.new("Frame", b); f.Size, f.BackgroundTransparency = UDim2.new(1,0,1,0), 1
                        local s = Instance.new("UIStroke", f); s.Thickness = 2; s.Color = Color3.new(1,0,0)
                    end
                elseif b then b:Destroy() end
            end
        end

        if Config.Aimbot then
            local target, d = nil, Config.FOV
            for _, v in pairs(Players:GetPlayers()) do
                if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Head") then
                    local p, on = Camera:WorldToViewportPoint(v.Character.Head.Position)
                    if on then
                        local mag = (Vector2.new(p.X, p.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                        if mag < d then target = v.Character.Head; d = mag end
                    end
                end
            end
            if target then Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Position), Config.Sticky) end
        end
    end)
end

-- ВХОД
local Auth = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
local f = Instance.new("Frame", Auth)
f.Size, f.Position, f.BackgroundColor3 = UDim2.new(0, 260, 0, 160), UDim2.new(0.5, -130, 0.4, 0), Color3.new(0,0,0)
Instance.new("UICorner", f)
local i = Instance.new("TextBox", f)
i.Size, i.Position, i.PlaceholderText = UDim2.new(0.8, 0, 0, 40), UDim2.new(0.1, 0, 0.2, 0), "КЛЮЧ"
i.TextColor3, i.BackgroundColor3, i.Text = Color3.new(1,1,1), Color3.new(0.1, 0.1, 0.1), ""
local b = Instance.new("TextButton", f)
b.Size, b.Position, b.Text = UDim2.new(0.8, 0, 0, 40), UDim2.new(0.1, 0, 0.6, 0), "ВОЙТИ"
b.BackgroundColor3, b.TextColor3 = Color3.fromRGB(130, 0, 255), Color3.new(1,1,1)
b.MouseButton1Click:Connect(function() 
    if Verify(i.Text) then Auth:Destroy(); Launch() end 
end)
