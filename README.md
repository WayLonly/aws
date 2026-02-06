local ReplicatedStorage = game:GetService("ReplicatedStorage")

local claimRemote = ReplicatedStorage
    :WaitForChild("Packages")
    :WaitForChild("Knit")
    :WaitForChild("Services")
    :WaitForChild("GiftMachineService")
    :WaitForChild("RF")
    :WaitForChild("Claim")

local args = { "23" }

-- Controle do loop
local running = false
local loopThread

local function startLoop()
    if loopThread then return end
    loopThread = task.spawn(function()
        while running do
            local success, result = pcall(function()
                return claimRemote:InvokeServer(unpack(args))
            end)

            if success then
                print("Claim executado com sucesso:", result)
            else
                warn("Erro ao executar Claim:", result)
            end

            task.wait(10)
        end
        loopThread = nil
    end)
end

local function stopLoop()
    running = false
end
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
-- Criação da janela Fluent
local Window = Fluent:CreateWindow({
    Title = "Fluent " .. Fluent.Version,
    SubTitle = "by dawid",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

-- Toggle dentro da tab Main
local Toggle = Tabs.Main:AddToggle("ClaimToggle", {
    Title = "Auto Claim",
    Description = "Liga/Desliga o Claim automático",
    Default = false,
    Callback = function(state)
        if state then
            running = true
            startLoop()
            print("Auto Claim ligado")
        else
            stopLoop()
            print("Auto Claim desligado")
        end
    end
})
