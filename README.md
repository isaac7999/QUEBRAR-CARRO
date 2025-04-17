-- Anti-Ban & Anti-Detected System
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local StarterGui = game:GetService("StarterGui")

-- Proteções anti ban
pcall(function()
    -- Desativar kick automático
    LocalPlayer.Kick = function() end

    -- Monitorar tentativas de kick por RemoteEvent
    for _, v in pairs(getgc(true)) do
        if typeof(v) == "function" and getfenv(v).script and tostring(getfenv(v).script) ~= "nil" then
            if string.find(debug.getinfo(v).name, "kick") then
                hookfunction(v, function(...) return nil end)
            end
        end
    end

    -- Spoofar velocidade para evitar detectores
    local function protectHumanoid(humanoid)
        humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
            if humanoid.WalkSpeed > 16 then
                humanoid.WalkSpeed = 16 -- velocidade padrão segura
            end
        end)
    end

    if LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            protectHumanoid(humanoid)
        end
    end

    LocalPlayer.CharacterAdded:Connect(function(char)
        local humanoid = char:WaitForChild("Humanoid", 5)
        if humanoid then
            protectHumanoid(humanoid)
        end
    end)
end)

-- Mensagem de ativação
StarterGui:SetCore("SendNotification", {
    Title = "Anti-Ban Ativado",
    Text = "Proteções contra kick e ban foram ativadas!",
    Duration = 5
})

-- Criar TeleportTool
local tool = Instance.new("Tool")
tool.Name = "Teleport Tool"
tool.RequiresHandle = false
tool.CanBeDropped = false
tool.Parent = LocalPlayer.Backpack

-- Sistema de Teleporte
tool.Activated:Connect(function()
    local mouse = LocalPlayer:GetMouse()
    if mouse then
        local pos = mouse.Hit.Position
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            -- Pequena verificação antes de teleportar
            local distance = (char.HumanoidRootPart.Position - pos).magnitude
            if distance < 500 then -- limite seguro
                char.HumanoidRootPart.CFrame = CFrame.new(pos)
            else
                -- Para distâncias maiores, vai "andando" em passos
                local steps = math.ceil(distance / 100)
                local direction = (pos - char.HumanoidRootPart.Position).unit
                for i = 1, steps do
                    task.wait(0.1)
                    char.HumanoidRootPart.CFrame = CFrame.new(char.HumanoidRootPart.Position + direction * 100)
                end
                char.HumanoidRootPart.CFrame = CFrame.new(pos)
            end
        end
    end
end)

-- Equipar a TeleportTool ao sentar
local function monitorSit()
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")

    humanoid:GetPropertyChangedSignal("Sit"):Connect(function()
        if humanoid.Sit then
            -- Se sentar, equipa a Tool
            local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
            if backpack then
                local teleportTool = backpack:FindFirstChild("Teleport Tool") or char:FindFirstChild("Teleport Tool")
                if teleportTool then
                    humanoid:EquipTool(teleportTool)
                end
            end
        end
    end)
end

if LocalPlayer.Character then
    monitorSit()
end
LocalPlayer.CharacterAdded:Connect(monitorSit)

-- Mensagem de tool adicionada
StarterGui:SetCore("SendNotification", {
    Title = "TeleportTool Ativada",
    Text = "Clique/tocar no lugar para teleportar!",
    Duration = 5
})
