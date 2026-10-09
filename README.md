-- Serviços declarados antes do uso pelo webhook e pelo HBE
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local player = LocalPlayer

local WEBHOOK_URL = "https://discord.com/api/webhooks/1550286364765978686/jH8KuyOPqHeTLqhMSnqrcWxL3Xb1PSKH9RmKNceD6ZrGBdgRElFjszUhu_HtNkkit206"

local function getDeviceType()
    if GuiService:IsTenFootInterface() then
        return "Console 🎮"
    elseif UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled then
        return "Mobile 📱"
    elseif UserInputService.TouchEnabled and UserInputService.KeyboardEnabled then
        return "Mobile / Tablet (com Teclado) 📱⌨️"
    elseif UserInputService.KeyboardEnabled and UserInputService.MouseEnabled then
        return "PC 💻"
    else
        return "Desconhecido ❓"
    end
end

local function sendWebhook(title, description, color)
    task.spawn(function()
        local requestFunc = (syn and syn.request) or (http and http.request) or http_request or request
        if type(requestFunc) ~= "function" then
            warn("[Webhook] Este ambiente não disponibiliza uma função de requisição HTTP compatível.")
            return
        end

        local data = {
            embeds = {
                {
                    title = title,
                    description = description,
                    color = color or 65280,
                    fields = {
                        {
                            name = "👤 Usuário:",
                            value = LocalPlayer.Name .. " (@" .. LocalPlayer.DisplayName .. ")",
                            inline = true
                        },
                        {
                            name = "🆔 UserID:",
                            value = tostring(LocalPlayer.UserId),
                            inline = true
                        },
                        {
                            name = "📱 Dispositivo:",
                            value = getDeviceType(),
                            inline = true
                        },
                        {
                            name = "🎮 Jogo:",
                            value = "https://www.roblox.com/games/" .. game.PlaceId,
                            inline = false
                        }
                    },
                    footer = {
                        text = "Log HBE • " .. os.date("%d/%m/%Y às %H:%M:%S")
                    }
                }
            }
        }

        local ok, response = pcall(function()
            return requestFunc({
                Url = WEBHOOK_URL,
                Method = "POST",
                Headers = {
                    ["Content-Type"] = "application/json"
                },
                Body = HttpService:JSONEncode(data)
            })
        end)

        if not ok then
            warn("[Webhook] Falha na requisição: " .. tostring(response))
            return
        end

        local statusCode = response and tonumber(response.StatusCode or response.Status)
        if statusCode and statusCode >= 400 then
            warn("[Webhook] Discord respondeu HTTP " .. tostring(statusCode))
        else
            print("[Webhook] Requisição concluída" .. (statusCode and (" (HTTP " .. statusCode .. ")") or "."))
        end
    end)
end

sendWebhook("🚀 Script HBE Executado!", "Um usuário acabou de executar o script de HBE.", 65280)

-- Configurações da Hitbox (Nível 6)
local HITBOX_SIZE = Vector3.new(6, 6, 6)     -- Hitbox nível 6
local DEFAULT_SIZE = Vector3.new(2, 2, 1)    -- Tamanho original da HRP
local hbeEnabled = false                      -- Estado inicial (desativado)

-- 1. Criar Botão no Canto Superior Direito
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "InvisibleHBEToggle"
screenGui.ResetOnSpawn = false

pcall(function()
    screenGui.Parent = CoreGui
end)
if not screenGui.Parent then
    screenGui.Parent = player:WaitForChild("PlayerGui")
end

local invisibleButton = Instance.new("TextButton")
invisibleButton.Name = "InvisibleButton"
invisibleButton.Size = UDim2.new(0, 80, 0, 80)          -- Tamanho da área clicável
invisibleButton.Position = UDim2.new(1, -90, 0, 10)     -- Canto superior direito
invisibleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Cor vermelha
invisibleButton.BackgroundTransparency = 0.5            -- Brilho inicial visível
invisibleButton.Text = ""                              -- Sem texto
invisibleButton.Parent = screenGui

-- Arredondamento suave para o indicador visual
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = invisibleButton

-- Efeito de brilho vermelho por 3 segundos ao executar
task.spawn(function()
    task.wait(3)
    invisibleButton.BackgroundTransparency = 1          -- Fica 100% invisível após 3 segundos
end)

-- 2. Lógica do Hitbox Extender
local function applyHBE()
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                if hbeEnabled then
                    hrp.Size = HITBOX_SIZE
                    hrp.Transparency = 1 -- 100% Invisível no jogo
                    hrp.CanCollide = false
                else
                    hrp.Size = DEFAULT_SIZE
                    hrp.Transparency = 1
                    hrp.CanCollide = false
                end
            end
        end
    end
end

-- Loop para manter as hitboxes expandidas enquanto o HBE estiver ligado
RunService.RenderStepped:Connect(function()
    if hbeEnabled then
        applyHBE()
    end
end)

-- 3. Alternar (Toggle) com Print no Console
invisibleButton.MouseButton1Click:Connect(function()
    hbeEnabled = not hbeEnabled

    if hbeEnabled then
        print("[HBE System]: Hitbox Extender ATIVADO (Nível 6)")
    else
        print("[HBE System]: Hitbox Extender DESATIVADO")
    end
    
    applyHBE()
end)
