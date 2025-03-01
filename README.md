local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")

local ESPEnabled = false
local ESPTransparency = 0.5
local ESPUpdateDelay = 0.1

local function getRoot(model)
    if model:IsA("Model") then
        return model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")
    end
    return model
end

local function createESP(plr)
    if not plr.Character then return end
    local character = plr.Character
    local head = character:FindFirstChild("Head")
    local rootPart = getRoot(character)

    if not rootPart or not head then return end

    local espFolder = CoreGui:FindFirstChild(plr.Name .. "_ESP")
    if espFolder then
        espFolder:Destroy()
    end

    espFolder = Instance.new("Folder")
    espFolder.Name = plr.Name .. "_ESP"
    espFolder.Parent = CoreGui

    local function createBox(part)
        local box = Instance.new("BoxHandleAdornment")
        box.Name = "ESPBox"
        box.Adornee = part
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Size = part.Size
        box.Transparency = ESPTransparency
        box.Color3 = plr.TeamColor.Color
        box.Parent = espFolder
    end

    for _, part in ipairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            createBox(part)
        end
    end

    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Adornee = head
    billboardGui.Size = UDim2.new(0, 100, 0, 50)
    billboardGui.StudsOffset = Vector3.new(0, 2, 0)
    billboardGui.AlwaysOnTop = true
    billboardGui.Parent = espFolder

    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = billboardGui
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextSize = 16
    textLabel.TextColor3 = Color3.new(1, 1, 1)
    textLabel.TextStrokeTransparency = 0.5
    textLabel.Text = plr.Name

    local function updateESP()
        if not character or not rootPart then return end
        if Players.LocalPlayer.Character then
            local localRoot = getRoot(Players.LocalPlayer.Character)
            if localRoot then
                local distance = (localRoot.Position - rootPart.Position).Magnitude
                textLabel.Text = plr.Name .. " | " .. math.floor(distance) .. " studs"
            end
        end
    end

    local espLoop
    espLoop = RunService.RenderStepped:Connect(updateESP)

    plr.CharacterRemoving:Connect(function()
        espFolder:Destroy()
        espLoop:Disconnect()
    end)
end

local function toggleESP()
    ESPEnabled = not ESPEnabled
    for _, plr in ipairs(Players:GetPlayers()) do
        if ESPEnabled then
            createESP(plr)
        else
            local espFolder = CoreGui:FindFirstChild(plr.Name .. "_ESP")
            if espFolder then espFolder:Destroy() end
        end
    end
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        if ESPEnabled then
            createESP(plr)
        end
    end)
end)

toggleESP()
