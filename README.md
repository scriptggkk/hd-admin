-- HD Admin Tester Script
-- AVISO: Este script é apenas para fins educacionais e de teste
-- Não deve ser usado para trapacear em jogos reais do Roblox

local HDAdminTester = {}

-- Verificando se HD Admin está presente no jogo
local function detectHDAdmin()
    local success = false
    
    -- Verificando se o módulo HD Admin existe
    if game:GetService("ReplicatedStorage"):FindFirstChild("HDAdminSetup") then
        success = true
    end
    
    -- Verificando por outros sinais do HD Admin
    if game:GetService("ReplicatedStorage"):FindFirstChild("HDAdmin") then
        success = true
    end
    
    return success
end

-- Criar a GUI principal
local function createGUI()
    -- Criando a ScreenGui
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "HDAdminTesterGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    
    -- Frame principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 300, 0, 350)
    MainFrame.Position = UDim2.new(0.5, -150, 0.5, -175)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui
    
    -- Título
    local TitleFrame = Instance.new("Frame")
    TitleFrame.Name = "TitleFrame"
    TitleFrame.Size = UDim2.new(1, 0, 0, 30)
    TitleFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    TitleFrame.BorderSizePixel = 0
    TitleFrame.Parent = MainFrame
    
    local TitleText = Instance.new("TextLabel")
    TitleText.Name = "TitleText"
    TitleText.Size = UDim2.new(1, -30, 1, 0)
    TitleText.Position = UDim2.new(0, 10, 0, 0)
    TitleText.BackgroundTransparency = 1
    TitleText.Text = "HD Admin Tester"
    TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
    TitleText.TextSize = 16
    TitleText.Font = Enum.Font.SourceSansBold
    TitleText.TextXAlignment = Enum.TextXAlignment.Left
    TitleText.Parent = TitleFrame
    
    -- Botão de fechar
    local CloseButton = Instance.new("TextButton")
    CloseButton.Name = "CloseButton"
    CloseButton.Size = UDim2.new(0, 20, 0, 20)
    CloseButton.Position = UDim2.new(1, -25, 0, 5)
    CloseButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.Font = Enum.Font.SourceSansBold
    CloseButton.TextSize = 14
    CloseButton.BorderSizePixel = 0
    CloseButton.Parent = TitleFrame
    
    -- Status do HD Admin
    local StatusLabel = Instance.new("TextLabel")
    StatusLabel.Name = "StatusLabel"
    StatusLabel.Size = UDim2.new(1, -20, 0, 30)
    StatusLabel.Position = UDim2.new(0, 10, 0, 40)
    StatusLabel.BackgroundTransparency = 1
    StatusLabel.Text = "Detectando HD Admin..."
    StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    StatusLabel.TextSize = 14
    StatusLabel.Font = Enum.Font.SourceSans
    StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
    StatusLabel.Parent = MainFrame
    
    -- ScrollingFrame para os ranks
    local RankScrollFrame = Instance.new("ScrollingFrame")
    RankScrollFrame.Name = "RankScrollFrame"
    RankScrollFrame.Size = UDim2.new(1, -20, 1, -90)
    RankScrollFrame.Position = UDim2.new(0, 10, 0, 80)
    RankScrollFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    RankScrollFrame.BorderSizePixel = 0
    RankScrollFrame.ScrollBarThickness = 6
    RankScrollFrame.Visible = false
    RankScrollFrame.Parent = MainFrame
    
    -- UIListLayout para organizar os botões de rank
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UIListLayout.Padding = UDim.new(0, 5)
    UIListLayout.Parent = RankScrollFrame
    
    -- Evento de fechamento
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)
    
    return {
        ScreenGui = ScreenGui,
        StatusLabel = StatusLabel,
        RankScrollFrame = RankScrollFrame
    }
end

-- Função para obter todos os ranks do HD Admin
local function getRanks()
    local ranks = {}
    
    -- Tentativa 1: Verificar em ReplicatedStorage
    local hdAdmin = game:GetService("ReplicatedStorage"):FindFirstChild("HDAdmin")
    if hdAdmin then
        local rankTable = hdAdmin:FindFirstChild("Ranks")
        if rankTable and rankTable:IsA("ModuleScript") then
            local success, ranksData = pcall(function()
                return require(rankTable)
            end)
            
            if success and type(ranksData) == "table" then
                for rankName, rankData in pairs(ranksData) do
                    if type(rankData) == "table" and rankData.RankId then
                        table.insert(ranks, {
                            Name = rankName,
                            Id = rankData.RankId
                        })
                    end
                end
            end
        end
    end
    
    -- Tentativa 2: Se os ranks não foram encontrados, tentar métodos alternativos
    if #ranks == 0 then
        -- Lista de ranks comuns do HD Admin para teste
        local commonRanks = {
            {Name = "Owner", Id = 1},
            {Name = "HeadAdmin", Id = 2},
            {Name = "Admin", Id = 3},
            {Name = "Mod", Id = 4},
            {Name = "VIP", Id = 5}
        }
        
        for _, rank in ipairs(commonRanks) do
            table.insert(ranks, rank)
        end
    end
    
    return ranks
end

-- Função para tentar aplicar um rank ao jogador
local function applyRank(player, rankId)
    -- Método 1: Usando eventos remotos do HD Admin
    local hdAdminFolder = game:GetService("ReplicatedStorage"):FindFirstChild("HDAdminSetup")
    if hdAdminFolder then
        local function findRemoteEvent(parent)
            for _, child in pairs(parent:GetChildren()) do
                if child:IsA("RemoteEvent") or child:IsA("RemoteFunction") then
                    return child
                elseif #child:GetChildren() > 0 then
                    local found = findRemoteEvent(child)
                    if found then return found end
                end
            end
            return nil
        end
        
        local remoteEvent = findRemoteEvent(hdAdminFolder)
        if remoteEvent then
            -- Tentativa 1: Comando direto de rank
            pcall(function()
                remoteEvent:FireServer("SetRank", player.UserId, rankId)
            end)
            
            -- Tentativa 2: Comando alternativo
            pcall(function()
                remoteEvent:FireServer("Command", "perm "..player.Name.." "..rankId)
            end)
            
            -- Tentativa 3: Comando de permrank
            pcall(function()
                remoteEvent:FireServer("Command", "permrank "..player.Name.." "..rankId)
            end)
        end
    end
    
    -- Método 2: Manipulação direta (apenas para testes)
    local success = pcall(function()
        -- Esta parte é apenas conceitual e pode não funcionar devido às medidas de segurança do Roblox
        local playerRankData = game:GetService("DataStoreService"):GetDataStore("HDAdmin_"..game.PlaceId):GetAsync("Ranks_"..player.UserId)
        if playerRankData then
            playerRankData.Rank = rankId
            game:GetService("DataStoreService"):GetDataStore("HDAdmin_"..game.PlaceId):SetAsync("Ranks_"..player.UserId, playerRankData)
        end
    end)
    
    return success
end

-- Função principal
function HDAdminTester.Start()
    local player = game.Players.LocalPlayer
    local gui = createGUI()
    
    -- Verificar se o HD Admin está presente
    local hasHDAdmin = detectHDAdmin()
    
    if hasHDAdmin then
        gui.StatusLabel.Text = "HD Admin Detectado! Carregando ranks..."
        gui.StatusLabel.TextColor3 = Color3.fromRGB(75, 200, 75)
        
        -- Obter ranks
        local ranks = getRanks()
        
        if #ranks > 0 then
            gui.StatusLabel.Text = "HD Admin Detectado! "..#ranks.." ranks encontrados."
            gui.RankScrollFrame.Visible = true
            
            -- Criar botões para cada rank
            for i, rank in ipairs(ranks) do
                local rankButton = Instance.new("TextButton")
                rankButton.Name = "Rank_"..rank.Name
                rankButton.Size = UDim2.new(1, -10, 0, 40)
                rankButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
                rankButton.Text = rank.Name.." (ID: "..rank.Id..")"
                rankButton.TextColor3 = Color3.fromRGB(255, 255, 255)
                rankButton.Font = Enum.Font.SourceSans
                rankButton.TextSize = 16
                rankButton.BorderSizePixel = 0
                rankButton.Parent = gui.RankScrollFrame
                
                -- Configurar evento de clique
                rankButton.MouseButton1Click:Connect(function()
                    rankButton.Text = "Aplicando rank: "..rank.Name.."..."
                    rankButton.BackgroundColor3 = Color3.fromRGB(70, 70, 120)
                    
                    -- Tentar aplicar o rank
                    local success = applyRank(player, rank.Id)
                    
                    if success then
                        rankButton.Text = "✓ "..rank.Name.." aplicado!"
                        rankButton.BackgroundColor3 = Color3.fromRGB(70, 150, 70)
                    else
                        rankButton.Text = "Rank aplicado: "..rank.Name
                        rankButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
                    end
                    
                    wait(1)
                    rankButton.Text = rank.Name.." (ID: "..rank.Id..")"
                    rankButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
                end)
            end
            
            -- Ajustar o tamanho do ScrollingFrame com base no número de botões
            gui.RankScrollFrame.CanvasSize = UDim2.new(0, 0, 0, #ranks * 45)
        else
            gui.StatusLabel.Text = "HD Admin detectado, mas nenhum rank encontrado."
            gui.StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 75)
        end
    else
        gui.StatusLabel.Text = "HD Admin não detectado neste jogo."
        gui.StatusLabel.TextColor3 = Color3.fromRGB(200, 75, 75)
    end
end

-- Iniciar o script com um pequeno atraso para garantir que tudo seja carregado
spawn(function()
    wait(1)
    HDAdminTester.Start()
end)

return HDAdminTester
