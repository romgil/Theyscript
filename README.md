StatusLabel.Font = Enum.Font.SourceSans
    StatusLabel.Parent = AutoFrame
    
    return {
        PetBox = PetBox,
        AmountBox = AmountBox,
        PlayerBox = PlayerBox,
        SpawnBtn = SpawnBtn,
        GiftBtn = GiftBtn,
        AutoSpawnBtn = AutoSpawnBtn,
        AutoGiftBtn = AutoGiftBtn,
        StatusLabel = StatusLabel
    }
end

-- Pet Functions
local function spawnPet(petName, amount)
    amount = tonumber(amount) or 1
    
    for i = 1, amount do
        pcall(function()
            -- Try different remote patterns for Grow a Garden
            if GardenPetHub.Remotes.SpawnPet then
                if GardenPetHub.Remotes.SpawnPet:IsA("RemoteEvent") then
                    GardenPetHub.Remotes.SpawnPet:FireServer(petName)
                else
                    GardenPetHub.Remotes.SpawnPet:InvokeServer(petName)
                end
            end
            
            if GardenPetHub.Remotes.GardenPet then
                if GardenPetHub.Remotes.GardenPet:IsA("RemoteEvent") then
                    GardenPetHub.Remotes.GardenPet:FireServer("spawn", petName)
                else
                    GardenPetHub.Remotes.GardenPet:InvokeServer("spawn", petName)
                end
            end
            
            if GardenPetHub.Remotes.UnlockPet then
                if GardenPetHub.Remotes.UnlockPet:IsA("RemoteEvent") then
                    GardenPetHub.Remotes.UnlockPet:FireServer(petName)
                else
                    GardenPetHub.Remotes.UnlockPet:InvokeServer(petName)
                end
            end
            
            -- Generic attempts
            for _, remote in pairs(ReplicatedStorage:GetDescendants()) do
                if remote:IsA("RemoteEvent") then
                    local name = string.lower(remote.Name)
                    if string.find(name, "pet") and (string.find(name, "get") or string.find(name, "obtain")) then
                        remote:FireServer(petName)
                    end
                end
            end
        end)
        wait(GardenPetHub.Settings.SpawnDelay)
    end
end

local function giftPet(targetUsername, petName)
    local targetPlayer = Players:FindFirstChild(targetUsername)
    if not targetPlayer then
        return false, "Player not found in server"
    end
    
    local success = false
    pcall(function()
        -- Try gift remotes
        if GardenPetHub.Remotes.GiftPet then
            if GardenPetHub.Remotes.GiftPet:IsA("RemoteEvent") then
                GardenPetHub.Remotes.GiftPet:FireServer(targetPlayer, petName)
            else
                GardenPetHub.Remotes.GiftPet:InvokeServer(targetPlayer, petName)
            end
            success = true
        end
        
        -- Alternative gift methods
        for _, remote in pairs(ReplicatedStorage:GetDescendants()) do
            if remote:IsA("RemoteEvent") then
                local name = string.lower(remote.Name)
                if string.find(name, "trade") or string.find(name, "give") then
                    remote:FireServer(targetPlayer, petName)
                    success = true
                end
            end
        end
    end)
    
    return success, success and "Gift sent successfully!" or "Failed to send gift"
end

-- Initialize GUI
local GUI = createGUI()

-- Button Events
GUI.SpawnBtn.MouseButton1Click:Connect(function()
    local petName = GUI.PetBox.Text
    local amount = GUI.AmountBox.Text
    
    if petName == "" then
        GUI.StatusLabel.Text = "Status: Please enter a pet name!"
        GUI.StatusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        return
    end
    
    GUI.StatusLabel.Text = "Status: Spawning " .. petName .. " x" .. amount .. "..."
    GUI.StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
    
    spawnPet(petName, amount)
    
    GUI.StatusLabel.Text = "Status: Spawned " .. petName .. " successfully!"
    GUI.StatusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    
    wait(3)
    GUI.StatusLabel.Text = "Status: Ready to spawn pets!"
end)# Theyscript
