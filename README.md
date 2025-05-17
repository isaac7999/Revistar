-- Proteções Anti-Kick / Anti-Ban / Anti-Detected
if not game:IsLoaded() then game.Loaded:Wait() end
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TextChatService = game:GetService("TextChatService")

-- Bloquear kicks
local mt = getrawmetatable(game)
setreadonly(mt, false)
local oldNamecall = mt.__namecall

mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if method == "Kick" or method == "kick" then
        warn("[Proteção]: Kick bloqueado.")
        return nil
    end
    return oldNamecall(self, ...)
end)

LocalPlayer.Kick = function()
    return warn("[Proteção]: Kick manual bloqueado.")
end

-- Proteção de console
local blockedWords = { "Exploit", "Ban", "Detected", "Script", "Kick", "Executor" }
local oldPrint, oldWarn = print, warn

print = function(...)
    for _, v in ipairs({...}) do
        for _, word in ipairs(blockedWords) do
            if type(v) == "string" and v:lower():find(word:lower()) then return end
        end
    end
    oldPrint(...)
end

warn = function(...)
    for _, v in ipairs({...}) do
        for _, word in ipairs(blockedWords) do
            if type(v) == "string" and v:lower():find(word:lower()) then return end
        end
    end
    oldWarn(...)
end

-- Enviar o /revistar morto 3x
local function sendRevistarMessage()
    local success, err = pcall(function()
        local channel = TextChatService:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
        for i = 1, 3 do
            channel:SendAsync("/revistar morto")
            task.wait(0.3)
        end
    end)
    if not success then
        warn("[RevistarErro]:", err)
    end
end

-- Verificar players mortos próximos (até 100 studs) e acionar se estiver a 60 ou menos
RunService.Heartbeat:Connect(function()
    pcall(function()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character:FindFirstChild("HumanoidRootPart") then
                local humanoid = player.Character.Humanoid
                local distance = (player.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude

                if humanoid.Health <= 0 and distance <= 100 then
                    if distance <= 60 then
                        sendRevistarMessage()
                        task.wait(2) -- Aguarda para não enviar repetido
                    end
                end
            end
        end
    end)
end)
