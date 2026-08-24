local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

local VisualsConfig = {
    Fullbright = false,
    RemoveShadows = false,
    RemoveGrass = false,
    RemoveWater = false,
    RemoveFog = false,
    RemoveEffects = false
}

local BoxESP = {
    Enabled = false,
    TeamCheck = true,
    ShowName = true,
    ShowDistance = true,
    ShowHealth = true,
    BoxColorEnemy = Color3.fromRGB(255, 0, 0),
    BoxColorAlly = Color3.fromRGB(0, 255, 0),
    BoxColorNeutral = Color3.fromRGB(255, 255, 0)
}

local PlayerStorage = {}
local BoxGUI = nil
local ScreenGui = nil
local FPSLabel = nil

local function GetPlayerModel(playerName)
    local entities = workspace:FindFirstChild("WORKSPACE_Entities")
    if not entities then return nil end
    local playersFolder = entities:FindFirstChild("Players")
    if not playersFolder then return nil end
    return playersFolder:FindFirstChild(playerName)
end

local function OptimizeObject(obj)
    pcall(function()
        if obj:IsDescendantOf(Players) then return end
        local name = string.lower(obj.Name)

        if VisualsConfig.RemoveGrass and (string.find(name, "grass") or string.find(name, "bush") or string.find(name, "plant") or string.find(name, "fern") or string.find(name, "foliage") or string.find(name, "weed") or string.find(name, "flora") or string.find(name, "shrub")) then
            if obj:IsA("BasePart") then
                obj.Transparency = 1
                obj.CanCollide = false
            end
        end

        if VisualsConfig.RemoveWater and (string.find(name, "water") or string.find(name, "river") or string.find(name, "lake") or string.find(name, "ocean")) then
            if obj:IsA("BasePart") then
                obj.Transparency = 1
                obj.CanCollide = false
            end
        end

        if VisualsConfig.RemoveEffects and (obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") or obj:IsA("Explosion") or obj:IsA("Beam")) then
            obj.Enabled = false
        end

        if VisualsConfig.RemoveShadows and obj:IsA("BasePart") then
            obj.CastShadow = false
        end
    end)
end

local function ApplyFullbright(state)
    VisualsConfig.Fullbright = state
    if state then
        Lighting.Brightness = 3
        Lighting.ClockTime = 12
        Lighting.GlobalShadows = false
        Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
        Lighting.Ambient = Color3.fromRGB(255, 255, 255)
        Lighting.FogEnd = 999999
    else
        Lighting.Brightness = 1
        Lighting.ClockTime = 14.5
        Lighting.GlobalShadows = true
        Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
        Lighting.Ambient = Color3.fromRGB(0, 0, 0)
        Lighting.FogEnd = 10000
    end
end

local function ApplyRemoveShadows(state)
    VisualsConfig.RemoveShadows = state
    Lighting.GlobalShadows = not state
    for _, obj in ipairs(Workspace:GetDescendants()) do
        pcall(function() if obj:IsA("BasePart") then obj.CastShadow = not state end end)
    end
end

local function ApplyRemoveGrass(state)
    VisualsConfig.RemoveGrass = state
    pcall(function()
        local terrain = Workspace:FindFirstChildOfClass("Terrain")
        if terrain then
            pcall(function() sethiddenproperty(terrain, "Decoration", not state) end)
            terrain.Decoration = not state
        end
    end)
    for _, obj in ipairs(Workspace:GetDescendants()) do OptimizeObject(obj) end
end

local function ApplyRemoveWater(state)
    VisualsConfig.RemoveWater = state
    pcall(function()
        local terrain = Workspace:FindFirstChildOfClass("Terrain")
        if terrain then
            terrain.WaterTransparency = state and 1 or 0.6
            terrain.WaterWaveSize = state and 0 or 1
            terrain.WaterWaveSpeed = state and 0 or 1
            terrain.WaterReflectance = state and 0 or 1
        end
    end)
    for _, obj in ipairs(Workspace:GetDescendants()) do OptimizeObject(obj) end
end

local function ApplyRemoveFog(state)
    VisualsConfig.RemoveFog = state
    if state then
        Lighting.FogEnd = 999999
        for _, v in ipairs(Lighting:GetChildren()) do if v:IsA("Atmosphere") then v.Density = 0 end end
    else
        Lighting.FogEnd = 10000
        for _, v in ipairs(Lighting:GetChildren()) do if v:IsA("Atmosphere") then v.Density = 0.3 end end
    end
end

local function ApplyRemoveEffects(state)
    VisualsConfig.RemoveEffects = state
    for _, v in ipairs(Lighting:GetChildren()) do if v:IsA("Clouds") then v.Enabled = not state end end
    for _, obj in ipairs(Workspace:GetDescendants()) do OptimizeObject(obj) end
end

Workspace.DescendantAdded:Connect(function(obj)
    OptimizeObject(obj)
end)

local function RunSuperBoost()
    _G.Settings = {
        Players = { ["Ignore Me"] = true, ["Ignore Others"] = true },
        Meshes = { Destroy = false, LowDetail = true },
        Images = { Invisible = true, LowDetail = false, Destroy = false },
        Other = {
            ["No Particles"] = true,
            ["No Camera Effects"] = true,
            ["No Explosions"] = true,
            ["No Clothes"] = true,
            ["Low Water Graphics"] = true,
            ["No Shadows"] = true,
            ["Low Rendering"] = true,
            ["Low Quality Parts"] = true
        }
    }
    
    local BadInstances = {"DataModelMesh", "FaceInstance", "ParticleEmitter", "Trail", "Smoke", "Fire", "Sparkles", "PostEffect", "Explosion", "Clothing", "BasePart"}
    local CanBeEnabled = {"ParticleEmitter", "Trail", "Smoke", "Fire", "Sparkles", "PostEffect"}
    
    local function PartOfCharacter(Instance)
        for i, v in pairs(Players:GetPlayers()) do
            if v.Character and Instance:IsDescendantOf(v.Character) then return true end
        end
        return false
    end
    
    local function CheckIfBad(Instance)
        if not Instance:IsDescendantOf(Players) and not PartOfCharacter(Instance) then
            if Instance:IsA("DataModelMesh") then
                pcall(function()
                    sethiddenproperty(Instance, "LODX", Enum.LevelOfDetailSetting.Low)
                    sethiddenproperty(Instance, "LODY", Enum.LevelOfDetailSetting.Low)
                end)
            elseif Instance:IsA("FaceInstance") then
                Instance.Transparency = 1
            elseif table.find(CanBeEnabled, Instance.ClassName) then
                Instance.Enabled = false
            elseif Instance:IsA("Explosion") then
                Instance.Visible = false
            elseif Instance:IsA("Clothing") then
                Instance:Destroy()
            elseif Instance:IsA("BasePart") then
                Instance.Material = Enum.Material.SmoothPlastic
                Instance.Reflectance = 0
            end
        end
    end
    
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        terrain.WaterWaveSize = 0
        terrain.WaterWaveSpeed = 0
        terrain.WaterReflectance = 0
        terrain.WaterTransparency = 0
    end
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    pcall(function() settings().Rendering.QualityLevel = 1 end)
    
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification", { Title = "Super Boost", Text = "Boost Activated!", Duration = 3 })
    end)
    
    task.spawn(function()
        local count = 0
        for i, v in pairs(game:GetDescendants()) do
            if not v:IsDescendantOf(Players) then
                CheckIfBad(v)
                count = count + 1
                if count >= 1000 then
                    count = 0
                    task.wait()
                end
            end
        end
    end)
    
    game.DescendantAdded:Connect(CheckIfBad)
end

local function CreateFPSCounter()
    local fpsGui = Instance.new("ScreenGui")
    fpsGui.Name = "FPSCounterGui"
    fpsGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    fpsGui.ResetOnSpawn = false
    fpsGui.IgnoreGuiInset = true

    FPSLabel = Instance.new("TextLabel")
    FPSLabel.Name = "FPSLabel"
    FPSLabel.Size = UDim2.new(0, 100, 0, 25)
    FPSLabel.Position = UDim2.new(0.5, -50, 0, 5)
    FPSLabel.BackgroundTransparency = 0.5
    FPSLabel.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
    FPSLabel.TextColor3 = Color3.fromRGB(0, 255, 128)
    FPSLabel.TextSize = 13
    FPSLabel.Font = Enum.Font.SourceSansBold
    FPSLabel.Text = "FPS: 0"
    FPSLabel.BorderSizePixel = 0
    FPSLabel.Parent = fpsGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = FPSLabel

    local lastUpdate = 0
    local frameCount = 0

    RunService.RenderStepped:Connect(function()
        frameCount = frameCount + 1
        local now = tick()
        if now - lastUpdate >= 0.5 then
            local currentFPS = math.floor(frameCount / (now - lastUpdate))
            frameCount = 0
            lastUpdate = now
            if FPSLabel then
                FPSLabel.Text = "FPS: " + currentFPS
            end
        end
    end)
end

local function CreateBoxGUI()
    if BoxGUI then BoxGUI:Destroy() end
    BoxGUI = Instance.new("ScreenGui")
    BoxGUI.Name = "CustomBoxESP"
    BoxGUI.Parent = LocalPlayer:WaitForChild("PlayerGui")
    BoxGUI.ResetOnSpawn = false
    BoxGUI.IgnoreGuiInset = true
    return BoxGUI
end

local function CreateBoxForPlayer(targetPlayer)
    local playerName = typeof(targetPlayer) == "Instance" and targetPlayer.Name or targetPlayer.Name
    if playerName == LocalPlayer.Name then return nil end
    if PlayerStorage[targetPlayer] then return PlayerStorage[targetPlayer] end
    
    local boxContainer = Instance.new("Frame")
    boxContainer.Name = playerName .. "_Box"
    boxContainer.BackgroundTransparency = 1
    boxContainer.Visible = false
    boxContainer.Parent = BoxGUI
    
    local function CreateLine(name)
        local line = Instance.new("Frame")
        line.Name = name
        line.BackgroundColor3 = BoxESP.BoxColorEnemy
        line.BorderSizePixel = 0
        line.Parent = boxContainer
        return line
    end
    
    local NameLabel = Instance.new("TextLabel")
    NameLabel.BackgroundTransparency = 1
    NameLabel.Size = UDim2.new(1, 0, 0, 15)
    NameLabel.Position = UDim2.new(0, 0, 0, -18)
    NameLabel.Font = Enum.Font.SourceSansBold
    NameLabel.TextSize = 12
    NameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    NameLabel.TextStrokeTransparency = 0.5
    NameLabel.Text = playerName
    NameLabel.Parent = boxContainer

    local InfoLabel = Instance.new("TextLabel")
    InfoLabel.BackgroundTransparency = 1
    InfoLabel.Size = UDim2.new(1, 0, 0, 15)
    InfoLabel.Position = UDim2.new(0, 0, 1, 3)
    InfoLabel.Font = Enum.Font.SourceSansBold
    InfoLabel.TextSize = 11
    InfoLabel.TextColor3 = Color3.fromRGB(0, 255, 128)
    InfoLabel.TextStrokeTransparency = 0.5
    InfoLabel.Text = "100 HP | 0m"
    InfoLabel.Parent = boxContainer

    local boxData = {
        Container = boxContainer,
        Top = CreateLine("Top"),
        Bottom = CreateLine("Bottom"),
        Left = CreateLine("Left"),
        Right = CreateLine("Right"),
        Name = NameLabel,
        Info = InfoLabel
    }
    
    PlayerStorage[targetPlayer] = boxData
    return boxData
end

local function GetBoxColor(targetPlayer)
    if not BoxESP.TeamCheck then return BoxESP.BoxColorNeutral end
    if typeof(targetPlayer) == "Instance" and targetPlayer.Team and LocalPlayer.Team then
        if targetPlayer.Team == LocalPlayer.Team then return BoxESP.BoxColorAlly
        else return BoxESP.BoxColorEnemy end
    end
    return BoxESP.BoxColorNeutral
end

local function InitializeBoxESP()
    CreateBoxGUI()
    PlayerStorage = {}
    local entities = workspace:FindFirstChild("WORKSPACE_Entities")
    if entities and entities:FindFirstChild("Players") then
        for _, model in ipairs(entities.Players:GetChildren()) do
            if model:IsA("Model") and model.Name ~= LocalPlayer.Name then
                local playerObj = Players:FindFirstChild(model.Name)
                if playerObj then
                    CreateBoxForPlayer(playerObj)
                else
                    CreateBoxForPlayer({Name = model.Name, Team = nil})
                end
            end
        end
    end
end

RunService.RenderStepped:Connect(function()
    if not BoxESP.Enabled then
        if BoxGUI then BoxGUI.Enabled = false end
        return
    else
        if BoxGUI then BoxGUI.Enabled = true end
    end

    for targetPlayer, boxData in pairs(PlayerStorage) do
        if boxData then
            local playerName = typeof(targetPlayer) == "Instance" and targetPlayer.Name or targetPlayer.Name
            local playerModel = GetPlayerModel(playerName)
            
            if playerModel then
                local rootPart = playerModel:FindFirstChild("HumanoidRootPart") or playerModel:FindFirstChild("Torso") or playerModel.PrimaryPart
                local humanoid = playerModel:FindFirstChildOfClass("Humanoid")
                
                if rootPart then
                    local screenPos, onScreen = camera:WorldToViewportPoint(rootPart.Position)
                    if onScreen then
                        local color = GetBoxColor(targetPlayer)
                        local distance = (camera.CFrame.Position - rootPart.Position).Magnitude
                        local height = math.clamp(2500 / distance, 20, 400)
                        local width = height * 0.65
                        
                        local x = screenPos.X - width / 2
                        local y = screenPos.Y - height / 2
                        
                        boxData.Container.Position = UDim2.new(0, x, 0, y)
                        boxData.Container.Size = UDim2.new(0, width, 0, height)
                        boxData.Container.Visible = true
                        
                        local thickness = 1.5
                        boxData.Top.Size = UDim2.new(1, 0, 0, thickness)
                        boxData.Top.BackgroundColor3 = color
                        boxData.Bottom.Size = UDim2.new(1, 0, 0, thickness)
                        boxData.Bottom.Position = UDim2.new(0, 0, 1, -thickness)
                        boxData.Bottom.BackgroundColor3 = color
                        boxData.Left.Size = UDim2.new(0, thickness, 1, 0)
                        boxData.Left.BackgroundColor3 = color
                        boxData.Right.Size = UDim2.new(0, thickness, 1, 0)
                        boxData.Right.Position = UDim2.new(1, -thickness, 0, 0)
                        boxData.Right.BackgroundColor3 = color
                        
                        boxData.Name.Visible = BoxESP.ShowName
                        boxData.Name.TextColor3 = color
                        
                        if BoxESP.ShowDistance or BoxESP.ShowHealth then
                            local hp = humanoid and math.floor(humanoid.Health) or 100
                            local infoText = ""
                            if BoxESP.ShowHealth then infoText = hp .. " HP" end
                            if BoxESP.ShowDistance then
                                if infoText ~= "" then infoText = infoText .. " | " end
                                infoText = infoText .. math.floor(distance) .. "m"
                            end
                            boxData.Info.Text = infoText
                            boxData.Info.Visible = true
                        else
                            boxData.Info.Visible = false
                        end
                    else
                        boxData.Container.Visible = false
                    end
                else
                    boxData.Container.Visible = false
                end
            else
                boxData.Container.Visible = false
            end
        end
    end
end)

local function CreateMenu()
    if ScreenGui then ScreenGui:Destroy() end
    
    ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "StarHubMenu"
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    ScreenGui.ResetOnSpawn = false
    
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 250, 0, 315)
    MainFrame.Position = UDim2.new(0.5, -125, 0.5, -157)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    MainFrame.BackgroundTransparency = 0.1
    MainFrame.BorderSizePixel = 1
    MainFrame.BorderColor3 = Color3.fromRGB(60, 60, 70)
    MainFrame.Active = true
    MainFrame.Selectable = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui
    
    local TitleBar = Instance.new("Frame")
    TitleBar.Name = "TitleBar"
    TitleBar.Size = UDim2.new(1, 0, 0, 30)
    TitleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame
    
    local HideButton = Instance.new("TextButton")
    HideButton.Name = "HideButton"
    HideButton.Size = UDim2.new(0, 25, 0, 25)
    HideButton.Position = UDim2.new(1, -30, 0.5, -12)
    HideButton.Text = "━"
    HideButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    HideButton.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    HideButton.BorderSizePixel = 0
    HideButton.Font = Enum.Font.SourceSansBold
    HideButton.TextSize = 16
    HideButton.Parent = TitleBar
    
    local Title = Instance.new("TextLabel")
    Title.Name = "Title"
    Title.Size = UDim2.new(1, -40, 1, 0)
    Title.Position = UDim2.new(0, 10, 0, 0)
    Title.Text = ""
    Title.BackgroundTransparency = 1
    Title.Parent = TitleBar
    
    local ContentFrame = Instance.new("Frame")
    ContentFrame.Name = "ContentFrame"
    ContentFrame.Size = UDim2.new(1, -20, 1, -40)
    ContentFrame.Position = UDim2.new(0, 10, 0, 35)
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.Parent = MainFrame
    
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.Padding = UDim.new(0, 6)
    UIListLayout.Parent = ContentFrame
    
    local function CreateButton(name, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 25)
        frame.BackgroundTransparency = 1
        frame.Parent = ContentFrame

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0, 120, 1, 0)
        label.Text = name
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.BackgroundTransparency = 1
        label.Font = Enum.Font.SourceSans
        label.TextSize = 12
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = frame

        local button = Instance.new("TextButton")
        button.Size = UDim2.new(0, 50, 0, 20)
        button.Position = UDim2.new(1, -50, 0.5, -10)
        button.Text = "RUN"
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
        button.BorderSizePixel = 1
        button.BorderColor3 = Color3.fromRGB(80, 80, 80)
        button.Font = Enum.Font.SourceSansBold
        button.TextSize = 11
        button.Parent = frame

        button.MouseButton1Click:Connect(function()
            callback()
        end)
    end

    local function CreateToggle(name, initialState, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 0, 25)
        frame.BackgroundTransparency = 1
        frame.Parent = ContentFrame

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0, 120, 1, 0)
        label.Text = name
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.BackgroundTransparency = 1
        label.Font = Enum.Font.SourceSans
        label.TextSize = 12
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = frame

        local button = Instance.new("TextButton")
        button.Size = UDim2.new(0, 50, 0, 20)
        button.Position = UDim2.new(1, -50, 0.5, -10)
        button.Text = initialState and "ON" or "OFF"
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.BackgroundColor3 = initialState and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
        button.BorderSizePixel = 1
        button.BorderColor3 = Color3.fromRGB(80, 80, 80)
        button.Font = Enum.Font.SourceSans
        button.TextSize = 11
        button.Parent = frame

        button.MouseButton1Click:Connect(function()
            initialState = not initialState
            button.Text = initialState and "ON" or "OFF"
            button.BackgroundColor3 = initialState and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
            callback(initialState)
        end)
    end

    CreateToggle("2D Boxes", BoxESP.Enabled, function(state)
        BoxESP.Enabled = state
        if state then InitializeBoxESP() end
    end)
    CreateToggle("Fullbright", VisualsConfig.Fullbright, ApplyFullbright)
    CreateToggle("Remove Shadows", VisualsConfig.RemoveShadows, ApplyRemoveShadows)
    CreateToggle("Remove Grass", VisualsConfig.RemoveGrass, ApplyRemoveGrass)
    CreateToggle("Remove Water", VisualsConfig.RemoveWater, ApplyRemoveWater)
    CreateToggle("Remove Fog", VisualsConfig.RemoveFog, ApplyRemoveFog)
    CreateToggle("Remove Effects", VisualsConfig.RemoveEffects, ApplyRemoveEffects)
    
    CreateButton("Super Boost", RunSuperBoost)
    
    local isHidden = false
    local hiddenSize = UDim2.new(0, 40, 0, 40)
    local shownSize = UDim2.new(0, 250, 0, 315)

    HideButton.MouseButton1Click:Connect(function()
        isHidden = not isHidden
        ContentFrame.Visible = not isHidden
        TitleBar.Size = isHidden and UDim2.new(1, 0, 1, 0) or UDim2.new(1, 0, 0, 30)
        HideButton.Text = isHidden and "+" or "━"
        MainFrame.Size = isHidden and hiddenSize or shownSize
    end)
    
    UserInputService.InputBegan:Connect(function(input, processed)
        if not processed and input.KeyCode == Enum.KeyCode.Insert then
            isHidden = not isHidden
            ContentFrame.Visible = not isHidden
            TitleBar.Size = isHidden and UDim2.new(1, 0, 1, 0) or UDim2.new(1, 0, 0, 30)
            HideButton.Text = isHidden and "+" or "━"
            MainFrame.Size = isHidden and hiddenSize or shownSize
        end
    end)
    
    return ScreenGui
end

CreateFPSCounter()
CreateMenu()
InitializeBoxESP()
