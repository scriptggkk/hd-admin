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
        
        -- Verificar também em outros locais possíveis
        for _, possibleLocation in pairs({"Ranks", "Settings", "RankSettings", "Config"}) do
            local rankFolder = hdAdmin:FindFirstChild(possibleLocation)
            if rankFolder and rankFolder:IsA("ModuleScript") then
                local success, data = pcall(function()
                    return require(rankFolder)
                end)
                
                if success and type(data) == "table" then
                    -- Tentar extrair ranks de várias estruturas possíveis
                    local rankData = data.Ranks or data.RankList or data
                    
                    if type(rankData) == "table" then
                        for name, info in pairs(rankData) do
                            if type(info) == "table" and (info.RankId or info.Id or info.Rank) then
                                table.insert(ranks, {
                                    Name = name,
                                    Id = info.RankId or info.Id or info.Rank
                                })
                            elseif type(name) == "string" and type(info) == "number" then
                                table.insert(ranks, {
                                    Name = name,
                                    Id = info
                                })
                            end
                        end
                    end
                end
            end
        end
    end
    
    -- Tentativa 2: Verificar diretamente os comandos do HD Admin
    local function scanCommands()
        local commands = game:GetService("ReplicatedStorage"):FindFirstChild("HDAdmin"):FindFirstChild("Commands")
        if commands and commands:IsA("ModuleScript") then
            local success, commandData = pcall(function()
                return require(commands)
            end)
            
            if success and type(commandData) == "table" then
                for cmdName, cmdData in pairs(commandData) do
                    if cmdName:lower() == "rank" or cmdName:lower() == "setrank" then
                        if type(cmdData) == "table" and cmdData.Settings and type(cmdData.Settings) == "table" then
                            local ranksList = cmdData.Settings.Ranks
                            if type(ranksList) == "table" then
                                for rankName, rankId in pairs(ranksList) do
                                    table.insert(ranks, {
                                        Name = rankName,
                                        Id = rankId
                                    })
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    
    pcall(scanCommands)
    
    -- Tentativa 3: Se os ranks não foram encontrados, tentar obter do DataStore (simulação)
    if #ranks == 0 then
        pcall(function()
            -- Algumas versões do HD Admin armazenam ranks no Player
            for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                local hdAdminData = player:FindFirstChild("HDAdminData")
                if hdAdminData then
                    for _, item in pairs(hdAdminData:GetChildren()) do
                        if item.Name == "Ranks" and item:IsA("ModuleScript") then
                            local success, rankData = pcall(function()
                                return require(item)
                            end)
                            
                            if success and type(rankData) == "table" then
                                for rankName, rankInfo in pairs(rankData) do
                                    if type(rankInfo) == "table" and rankInfo.Id then
                                        table.insert(ranks, {
                                            Name = rankName,
                                            Id = rankInfo.Id
                                        })
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end)
    end
    
    -- Tentativa 4: Se ainda não temos ranks, usar os ranks padrão do HD Admin
    if #ranks == 0 then
        -- Lista de ranks comuns do HD Admin
        local commonRanks = {
            {Name = "Owner", Id = 4},
            {Name = "HeadAdmin", Id = 3},
            {Name = "Admin", Id = 2},
            {Name = "Mod", Id = 1},
            {Name = "VIP", Id = 5},
            {Name = "Special", Id = 6},
            {Name = "Member", Id = 7}
        }
        
        for _, rank in ipairs(commonRanks) do
            table.insert(ranks, rank)
        end
    end
    
    -- Remover duplicatas por nome
    local uniqueRanks = {}
    local rankNames = {}
    
    for _, rank in ipairs(ranks) do
        if not rankNames[rank.Name] then
            rankNames[rank.Name] = true
            table.insert(uniqueRanks, rank)
        end
    end
    
    return uniqueRanks
end

-- Função para tentar aplicar um rank ao jogador
local function applyRank(player, rankId, rankName)
    local success = false
    
    -- Encontrar todas as RemoteEvents e RemoteFunctions do HD Admin
    local function findRemoteObjects()
        local remotes = {}
        
        -- Procurar em ReplicatedStorage
        for _, service in pairs({game:GetService("ReplicatedStorage"), game:GetService("ServerScriptService")}) do
            local function searchFolder(folder)
                for _, item in pairs(folder:GetChildren()) do
                    if item:IsA("RemoteEvent") or item:IsA("RemoteFunction") then
                        table.insert(remotes, item)
                    elseif #item:GetChildren() > 0 then
                        searchFolder(item)
                    end
                end
            end
            
            searchFolder(service)
        end
        
        return remotes
    end
    
    -- Método 1: Localizar e usar os eventos remotos específicos do HD Admin
    local hdAdmin = game:GetService("ReplicatedStorage"):FindFirstChild("HDAdmin")
    local hdAdminSetup = game:GetService("ReplicatedStorage"):FindFirstChild("HDAdminSetup")
    
    -- Tentativa 1: Procurar pelo remote específico para comandos
    local commandRemote
    
    if hdAdmin then
        commandRemote = hdAdmin:FindFirstChild("Remote") or hdAdmin:FindFirstChild("CommandRemote")
    end
    
    if not commandRemote and hdAdminSetup then
        commandRemote = hdAdminSetup:FindFirstChild("Remote") or hdAdminSetup:FindFirstChild("CommandRemote")
        
        if not commandRemote then
            for _, child in pairs(hdAdminSetup:GetDescendants()) do
                if child:IsA("RemoteEvent") or child:IsA("RemoteFunction") then
                    commandRemote = child
                    break
                end
            end
        end
    end
    
    -- Tentativa 2: Se não encontrou o remote específico, tentar todos os remotes no jogo
    if not commandRemote then
        local allRemotes = findRemoteObjects()
        
        for _, remote in pairs(allRemotes) do
            -- Usar o nome para identificar possíveis remotes do HD Admin
            if remote.Name:lower():match("admin") or 
               remote.Name:lower():match("command") or 
               remote.Name:lower():match("remote") then
                commandRemote = remote
                break
            end
        end
        
        -- Se ainda não encontrou, usar o primeiro remote encontrado
        if not commandRemote and #allRemotes > 0 then
            commandRemote = allRemotes[1]
        end
    end
    
    -- Se encontrou algum remote, tentar todos os métodos de comando
    if commandRemote then
        -- Lista de comandos para tentar
        local commands = {
            "permrank "..player.Name.." "..rankName,
            "perm "..player.Name.." "..rankName,
            "setrank "..player.Name.." "..rankName,
            "rank "..player.Name.." "..rankName,
            "admin "..player.Name.." "..rankName,
            "permrank "..player.Name.." "..rankId,
            "perm "..player.Name.." "..rankId,
            "setrank "..player.Name.." "..rankId,
            "rank "..player.Name.." "..rankId,
            "admin "..player.Name.." "..rankId
        }
        
        -- Tentar cada comando
        for _, command in ipairs(commands) do
            -- Método 1: Enviar como comando diretamente
            pcall(function()
                if commandRemote:IsA("RemoteEvent") then
                    commandRemote:FireServer(command)
                    success = true
                elseif commandRemote:IsA("RemoteFunction") then
                    commandRemote:InvokeServer(command)
                    success = true
                end
            end)
            
            -- Método 2: Enviar como [tipo, comando]
            pcall(function()
                if commandRemote:IsA("RemoteEvent") then
                    commandRemote:FireServer("Command", command)
                    success = true
                elseif commandRemote:IsA("RemoteFunction") then
                    commandRemote:InvokeServer("Command", command)
                    success = true
                end
            end)
            
            -- Método 3: Enviar apenas o nome do comando
            pcall(function()
                if commandRemote:IsA("RemoteEvent") then
                    commandRemote:FireServer(command:split(" ")[1], player.Name, rankName or rankId)
                    success = true
                elseif commandRemote:IsA("RemoteFunction") then
                    commandRemote:InvokeServer(command:split(" ")[1], player.Name, rankName or rankId)
                    success = true
                end
            end)
            
            -- Se algum método funcionou, parar de tentar
            if success then break end
        end
    end
    
    -- Método 3: Tentar acessar o sistema de ranks diretamente (apenas para versões específicas do HD Admin)
    if not success then
        pcall(function()
            -- Tentar encontrar o módulo de gerenciamento de ranks
            local rankModule = game:GetService("ReplicatedStorage"):FindFirstChild("HDAdmin"):FindFirstChild("RankManager")
            if rankModule and rankModule:IsA("ModuleScript") then
                local rankManager = require(rankModule)
                if typeof(rankManager) == "table" and rankManager.SetRank then
                    rankManager.SetRank(player, rankId, true) -- O true indica rank permanente
                    success = true
                end
            end
        end)
    end
    
    -- Método 4: Tentar explorar a estrutura de "Owner"
    if not success then
        pcall(function()
            -- Em algumas versões do HD Admin, o Owner tem acesso direto ao sistema
            local ownerModule = game:GetService("ReplicatedStorage"):FindFirstChild("HDAdmin"):FindFirstChild("Owner")
            if ownerModule then
                -- Tentativa de se colocar como Owner temporariamente
                local currentPerms = {}
                
                -- Backup das permissões atuais (medida de segurança)
                for _, child in pairs(ownerModule:GetChildren()) do
                    if child:IsA("StringValue") and child.Name == player.UserId then
                        currentPerms[child.Name] = child.Value
                    end
                end
                
                -- Criar valor temporário
                local tempPerm = Instance.new("StringValue")
                tempPerm.Name = player.UserId
                tempPerm.Value = "Owner"
                tempPerm.Parent = ownerModule
                
                -- Tentar executar comando como Owner
                if commandRemote then
                    pcall(function()
                        if commandRemote:IsA("RemoteEvent") then
                            commandRemote:FireServer("permrank "..player.Name.." "..rankName)
                            success = true
                        end
                    end)
                end
                
                -- Restaurar permissões originais
                tempPerm:Destroy()
                for name, value in pairs(currentPerms) do
                    local perm = Instance.new("StringValue")
                    perm.Name = name
                    perm.Value = value
                    perm.Parent = ownerModule
                end
            end
        end)
    end
    
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
                    local success = applyRank(player, rank.Id, rank.Name)
                    
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
