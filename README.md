-- ==============================================================================
-- BYTE HUB - BLOX FRUITS (INTERFACE COMPLETA EM LUAU)
-- ==============================================================================
local game = game
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

-- Variáveis de estado global da interface
local autoFarmActive = false
local farmItemActive = false
local selectedItem = ""

----------------------------------------------------------------------------------------------------------------------------------------------
-- DETECÇÃO AUTOMÁTICA DE SEA (MUNDO DO BLOX FRUITS)
----------------------------------------------------------------------------------------------------------------------------------------------
local currentSea = 1
local placeId = game.PlaceId

if placeId == 2753915549 then
    currentSea = 1
elseif placeId == 4442272183 then
    currentSea = 2
elseif placeId == 7449423635 then
    currentSea = 3
end

local itemsBySea = {
    [1] = {
        "Angel Wings (Asas de Anjo)",
        "Leather (Couro)",
        "Magma Ore (Minério de Magma)",
        "Fish Tail (Rabo de Peixe)",
        "Scrap Metal (Sucata)",
        "Fool's Gold (Ouro dos Tolos)"
    },
    [2] = {
        "Ectoplasm (Ectoplasma)",
        "Vampire Fang (Presa de Vampiro)",
        "Radioactive Material (Material Radioativo)",
        "Mystic Droplet (Gota Mística)",
        "Library Key (Chave da Biblioteca)",
        "Water Key (Chave da Água)",
        "Fist of Darkness (Punho da Escuridão)"
    },
    [3] = {
        "Bones (Ossos)",
        "Mini Tusk (Mini Presa)",
        "Conjured Cocoa (Cacau Conjurado)",
        "Dragon Scale (Escala de Dragão)",
        "Demonic Soul (Alma Demoníaca)",
        "God's Chalice (Cálice de Deus)",
        "Hallow Essence (Essência Sagrada)",
        "Mirror Fractal (Fractal de Espelho)"
    }
}

local availableItems = itemsBySea[currentSea] or itemsBySea[1]
selectedItem = availableItems[1] or "Nenhum"

----------------------------------------------------------------------------------------------------------------------------------------------
-- BOTÃO FLUTUANTE DE ATALHO NA TELA
----------------------------------------------------------------------------------------------------------------------------------------------
local function getHiddenGui()
    if syn and syn.protect_gui then
        local gui = Instance.new("ScreenGui")
        syn.protect_gui(gui)
        gui.Parent = CoreGui
        return gui
    elseif gethui then
        local gui = Instance.new("ScreenGui")
        gui.Parent = gethui()
        return gui
    else
        local gui = Instance.new("ScreenGui")
        gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
        return gui
    end
end

local screenGui = getHiddenGui()
screenGui.Name = "ByteHubGUI_Opt"
screenGui.ResetOnSpawn = false

local button = Instance.new("TextButton")
button.Name = "ToggleMenu"
button.Font = Enum.Font.SourceSansBold
button.Size = UDim2.new(0, 110, 0, 40)
button.Position = UDim2.new(0, 15, 0.5, -20)
button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.TextSize = 14
button.Text = "Byte Hub"
button.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = button

----------------------------------------------------------------------------------------------------------------------------------------------
-- CARREGAMENTO DA INTERFACE (FLUENT UI)
----------------------------------------------------------------------------------------------------------------------------------------------
local success, Fluent = pcall(function()
    return loadstring(game:HttpGet("https://raw.githubusercontent.com/DamThien332/UI-Libs/main/Main%20Fluent.lua"))()
end)

if not success or not Fluent then
    warn("[Byte Hub] Erro ao carregar a interface Fluent.")
    return
end

local Window = Fluent:CreateWindow({
    Title = "Byte Hub",
    SubTitle = "by damthien",
    TabWidth = 160,
    Size = UDim2.fromOffset(530, 370),
    Acrylic = false,
    Theme = "Black",
    MinimizeKey = Enum.KeyCode.End
})

button.MouseButton1Click:Connect(function()
    Window:Minimize()
end)

----------------------------------------------------------------------------------------------------------------------------------------------
-- CRIAÇÃO DAS ABAS
----------------------------------------------------------------------------------------------------------------------------------------------
local Tabs = {
    profile = Window:AddTab({ Title = "Information", Icon = "scan-face" }),
    Main = Window:AddTab({ Title = "Main Farm", Icon = "home" }),
    Settings = Window:AddTab({ Title = "Personalizar / Settings", Icon = "settings" })
}

----------------------------------------------------------------------------------------------------------------------------------------------
-- ABA 1: INFORMATION
----------------------------------------------------------------------------------------------------------------------------------------------
Tabs.profile:AddParagraph({
    Title = "Painel Informativo",
    Content = "Jogador: " .. LocalPlayer.Name .. "\nMundo Detectado: Sea " .. tostring(currentSea)
})

Tabs.profile:AddButton({
    Title = "Copiar Discord",
    Callback = function()
        setclipboard("https://discord.gg/PF36rvJXN4")
        Fluent:Notify({
            Title = "Byte Hub",
            Content = "Link do Discord copiado!",
            Duration = 2
        })
    end
})

----------------------------------------------------------------------------------------------------------------------------------------------
-- ABA 2: MAIN FARM
----------------------------------------------------------------------------------------------------------------------------------------------
Tabs.Main:AddSection("Auto Farm Geral")

local AutoFarmToggle = Tabs.Main:AddToggle("AutoFarmToggle", {
    Title = "Ativar Auto Farm Geral",
    Description = "Farm padrão de nível/mobs",
    Default = false
})

AutoFarmToggle:OnChanged(function(Value)
    autoFarmActive = Value
    if autoFarmActive then
        Fluent:Notify({ Title = "Auto Farm", Content = "Auto Farm Geral ativado!", Duration = 2 })
        task.spawn(function()
            while autoFarmActive do
                pcall(function()
                    -- Lógica de interface/interação aqui
                end)
                task.wait(0.2)
            end
        end)
    else
        Fluent:Notify({ Title = "Auto Farm", Content = "Auto Farm Geral desativado.", Duration = 2 })
    end
end)

Tabs.Main:AddSection("Farm de Itens (Sea " .. tostring(currentSea) .. ")")

local ItemDropdown = Tabs.Main:AddDropdown("ItemSelect", {
    Title = "Selecionar Item",
    Values = availableItems,
    Default = 1,
    Multi = false,
    Callback = function(Value)
        selectedItem = Value
    end
})

local FarmItemToggle = Tabs.Main:AddToggle("FarmItemToggle", {
    Title = "Ativar Farm do Item Selecionado",
    Description = "Focar no item escolhido
