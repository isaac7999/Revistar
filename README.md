-- Proteções Anti-Kick / Anti-Ban / Anti-Detected
if not game:IsLoaded() then game.Loaded:Wait() end
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TextChatService = game:GetService("TextChatService")

-- Proteção contra Kick manual e console logs
local mt = getrawmetatable(game)
setreadonly(mt, false)
local oldNamecall = mt.__namecall

mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if method == "Kick" or method == "kick" then
        warn("[Proteção]: Tentaram te kickar! Bloqueado.")
        return nil
    end
    return oldNamecall(self, ...)
end)

-- Proteção contra kick direto
LocalPlayer.Kick = function()
    return warn("[Proteção]: Kick bloqueado!")
end

-- Proteção contra logs suspeitos no console
local oldPrint = print
local oldWarn = warn
local blockedWords = { "Exploit", "Executor", "Ban", "Detected", "Script", "Hack" }

print = function(...)
    local args = {...}
    for _, v in ipairs(args) do
        for _, word in ipairs(blockedWords) do
            if type(v) == "string" and v:lower():find(word:lower()) then
                return
            end
        end
    end
    oldPrint(unpack(args))
end

warn = function(...)
    local args = {...}
    for _, v in ipairs(args) do
        for _, word in ipairs(blockedWords) do
            if type(v) == "string" and v:lower():find(word:lower()) then
                return
            end
        end
    end
    oldWarn(unpack(args))
end

-- Enviar comando /revistar morto
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

-- Monitorar players próximos com 0 de vida
RunService.Heartbeat:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local distance = (player.Character.PrimaryPart.Position - LocalPlayer.Character.PrimaryPart.Position).Magnitude
            if distance <= 200 and humanoid.Health <= 0 then
                sendRevistarMessage()
                task.wait(1.5) -- Espera para evitar spam
            end
        end
    end
end)
