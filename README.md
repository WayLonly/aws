local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ToolService Remotes
local toolService = ReplicatedStorage
    :WaitForChild("Packages")
    :WaitForChild("Knit")
    :WaitForChild("Services")
    :WaitForChild("ToolService")
    :WaitForChild("RE")

local onEquipRequest = toolService:WaitForChild("onEquipRequest")
local onClick = toolService:WaitForChild("onClick")

-- GiftMachine Remotes
local giftService = ReplicatedStorage
    :WaitForChild("Packages")
    :WaitForChild("Knit")
    :WaitForChild("Services")
    :WaitForChild("GiftMachineService")
    :WaitForChild("RF")

local claimRemote = giftService:WaitForChild("Claim")

-- Argumentos para equipar
local equipArgs = { "23", "Dumbells", "PlanetZorp12" }
local claimArgs = { "23" }

-- Controle AutoTrain
local runningTrain = false
local loopTrain

local function startTrainLoop()
    if loopTrain then return end
    loopTrain = task.spawn(function()
        -- Executa uma vez o equip
        local successEquip, errEquip = pcall(function()
            onEquipRequest:FireServer(unpack(equipArgs))
        end)
        if successEquip then
            print("Equip executado com sucesso")
        else
            warn("Erro ao executar Equip:", errEquip)
        end

        -- Depois entra no loop apenas do onClick
        while runningTrain do
            local successClick, errClick = pcall(function()
                onClick:FireServer()
            end)

            if not successClick then
                warn("Erro ao executar Click:", errClick)
            end

            task.wait(0) -- intervalo de 1 segundo
        end
        loopTrain = nil
    end)
end

local function stopTrainLoop()
    runningTrain = false
end

-- Controle AutoClaim
local runningClaim = false
local loopClaim

local function startClaimLoop()
    if loopClaim then return end
    loopClaim = task.spawn(function()
        while runningClaim do
            local success, result = pcall(function()
                return claimRemote:InvokeServer(unpack(claimArgs))
            end)

            if success then
                print("Claim executado com sucesso:", result)
            else
                warn("Erro ao executar Claim:", result)
            end

            task.wait(600) -- intervalo de 10 segundos
        end
        loopClaim = nil
    end)
end

local function stopClaimLoop()
    runningClaim = false
end

-- Fluent GUI
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
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

-- Toggle AutoTrain
local ToggleTrain = Tabs.Main:AddToggle("AutoTrain", {
    Title = "Auto Train",
    Description = "Equipa uma vez e depois clica em loop",
    Default = false,
    Callback = function(state)
        if state then
            runningTrain = true
            startTrainLoop()
        else
            stopTrainLoop()
        end
    end
})

-- Toggle AutoClaim
local ToggleClaim = Tabs.Main:AddToggle("AutoClaim", {
    Title = "Auto Claim",
    Description = "Liga/Desliga o Claim automático",
    Default = false,
    Callback = function(state)
        if state then
            runningClaim = true
            startClaimLoop()
        else
            stopClaimLoop()
        end
    end
})
