--[[
    Script: Dark Aura BR 🇧🇷 - Painel de Visualização
    Autor: Desenvolvedor
    Versão: 2.0
    KEY CORRETA: 5609
    ⚠️ SÓ CARREGA SE A KEY FOR 5609!
--]]

-- Tentar carregar a biblioteca Rayfield com diferentes URLs
local Rayfield = nil
local RayfieldLoaded = false

local RayfieldURLs = {
    'https://sirius.menu/rayfield',
    'https://raw.githubusercontent.com/shlexware/Rayfield/main/source.lua',
    'https://pastebin.com/raw/rayfield_library'
}

for _, url in ipairs(RayfieldURLs) do
    local success, result = pcall(function()
        return loadstring(game:HttpGet(url))()
    end)
    
    if success and result then
        Rayfield = result
        RayfieldLoaded = true
        break
    end
end

if not RayfieldLoaded then
    -- Se Rayfield não carregar, criar uma interface simples alternativa
    warn("[Dark Aura] Rayfield não carregou, usando interface alternativa")
    
    -- Criar uma ScreenGui simples para mensagem
    local sg = Instance.new("ScreenGui")
    sg.Name = "DarkAuraError"
    sg.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 400, 0, 200)
    frame.Position = UDim2.new(0.5, -200, 0.5, -100)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    frame.BackgroundTransparency = 0.1
    frame.BorderSizePixel = 0
    frame.Parent = sg
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local text = Instance.new("TextLabel")
    text.Size = UDim2.new(1, 0, 1, 0)
    text.BackgroundTransparency = 1
    text.Text = "Dark Aura BR 🇧🇷\n\nErro: Não foi possível carregar a biblioteca Rayfield.\nVerifique sua conexão e tente novamente."
    text.TextColor3 = Color3.fromRGB(255, 255, 255)
    text.TextSize = 14
    text.TextWrapped = true
    text.Font = Enum.Font.Gotham
    text.Parent = frame
    
    return
end

-- Serviços necessários
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ============================================
-- 1. SISTEMA DE AUTENTICAÇÃO
-- ============================================

local IsAuthenticated = false
local CORRECT_KEY = "5609"

-- Variáveis de configuração
local Settings = {
    FOVSize = 120,
    ShowFOV = true,
    PlayerHighlight = false,
    DirectionLines = false
}

-- Variáveis do sistema
local FOVCircle = nil
local Highlights = {}
local DirectionLinesObj = {}
local MainWindow = nil
local AuthWindow = nil

-- ============================================
-- 2. CRIAR JANELA DE AUTENTICAÇÃO
-- ============================================

local function CreateAuthWindow()
    AuthWindow = Rayfield:CreateWindow({
        Name = "Dark Aura BR 🇧🇷 - Autenticação",
        Icon = 0,
        LoadingTitle = "Verificando Acesso...",
        LoadingSubtitle = "Insira sua key de acesso",
        ConfigurationSaving = {
            Enabled = false
        }
    })
    
    local AuthTab = AuthWindow:CreateTab("🔐 Autenticação", nil)
    AuthTab:CreateSection("Login")
    
    -- Informação da Key
    AuthTab:CreateParagraph({
        Name = "KeyInfo",
        Content = "🔑 KEY CORRETA: 5609\n\n⚠️ ATENÇÃO: Se a key estiver errada, o sistema NÃO será carregado!\n\nDigite a key abaixo para acessar o sistema."
    })
    
    -- Campo de texto para Key
    local KeyInput = AuthTab:CreateInput({
        Name = "Chave de Acesso",
        PlaceholderText = "Digite sua key aqui...",
        RemoveTextAfterFocusLoss = false,
        CurrentValue = "",
        Flag = "KeyInput"
    })
    
    -- Variável para controlar se já autenticou
    local alreadyAuthenticated = false
    
    -- Botão confirmar
    AuthTab:CreateButton({
        Name = "🔓 Confirmar Acesso",
        Callback = function()
            -- Impedir múltiplas autenticações
            if alreadyAuthenticated then
                Rayfield:Notify({
                    Title = "⚠️ ATENÇÃO",
                    Content = "Você já está autenticado!",
                    Duration = 2
                })
                return
            end
            
            local EnteredKey = KeyInput.CurrentValue
            
            -- Verificar se a key está correta (5609)
            if EnteredKey == CORRECT_KEY then
                -- ✅ AUTENTICAÇÃO BEM-SUCEDIDA
                IsAuthenticated = true
                alreadyAuthenticated = true
                
                Rayfield:Notify({
                    Title = "✅ ACESSO LIBERADO!",
                    Content = "Key correta! Carregando sistema Dark Aura BR...",
                    Duration = 2
                })
                
                -- Pequeno delay para a notificação aparecer
                task.wait(1)
                
                -- Fechar janela de autenticação
                if AuthWindow then
                    AuthWindow:Destroy()
                end
                
                -- CARREGAR INTERFACE PRINCIPAL
                LoadMainInterface()
                
            else
                -- ❌ KEY INVÁLIDA
                Rayfield:Notify({
                    Title = "❌ ACESSO NEGADO!",
                    Content = "Key inválida! A key correta é: 5609",
                    Duration = 4
                })
                
                -- Limpar campo
                KeyInput:SetValue("")
                
                print("[Dark Aura] ❌ Key inválida! Sistema NÃO será carregado.")
                print("[Dark Aura] Key digitada:", EnteredKey)
            end
        end
    })
    
    -- Botão para mostrar a key correta
    AuthTab:CreateButton({
        Name = "🔑 Esqueci a Key",
        Callback = function()
            Rayfield:Notify({
                Title = "🔑 KEY CORRETA",
                Content = "A key de acesso é: 5609",
                Duration = 3
            })
        end
    })
end

-- ============================================
-- 3. FUNÇÕES DO SISTEMA DE VISUALIZAÇÃO
-- ============================================

-- Função para verificar se um jogador é válido e visível
local function IsValidPlayer(Player)
    if not Player then return false end
    if Player == LocalPlayer then return false end
    
    local Character = Player.Character
    if not Character then return false end
    
    local Humanoid = Character:FindFirstChild("Humanoid")
    if not Humanoid or Humanoid.Health <= 0 then return false end
    
    local Head = Character:FindFirstChild("Head")
    if Head then
        local ScreenPos, OnScreen = Camera:WorldToViewportPoint(Head.Position)
        if OnScreen then
            return true
        end
    end
    
    return false
end

-- ============================================
-- 4. SISTEMA DE FOV CIRCLE
-- ============================================

local function CreateFOVCircle()
    local PlayerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not PlayerGui then
        PlayerGui = Instance.new("ScreenGui")
        PlayerGui.Name = "PlayerGui"
        PlayerGui.Parent = LocalPlayer
    end
    
    local OldCircle = PlayerGui:FindFirstChild("FOVCircleGUI")
    if OldCircle then
        OldCircle:Destroy()
    end
    
    if not Settings.ShowFOV then return nil end
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "FOVCircleGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = PlayerGui
    
    local CircleFrame = Instance.new("Frame")
    CircleFrame.Name = "FOVCircle"
    CircleFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    CircleFrame.BackgroundTransparency = 1
    CircleFrame.BorderSizePixel = 0
    CircleFrame.Size = UDim2.new(0, Settings.FOVSize * 2, 0, Settings.FOVSize * 2)
    CircleFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
    CircleFrame.Parent = ScreenGui
    
    local UIStroke = Instance.new("UIStroke")
    UIStroke.Thickness = 3
    UIStroke.Color = Color3.fromRGB(138, 43, 226)
    UIStroke.Transparency = 0.3
    UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    UIStroke.Parent = CircleFrame
    
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(1, 0)
    UICorner.Parent = CircleFrame
    
    return CircleFrame
end

local function UpdateFOVCircle()
    if FOVCircle then
        FOVCircle.Size = UDim2.new(0, Settings.FOVSize * 2, 0, Settings.FOVSize * 2)
    end
end

local function ToggleFOVCircle()
    if FOVCircle then
        local parent = FOVCircle.Parent
        if parent then
            parent:Destroy()
        end
        FOVCircle = nil
    end
    
    if Settings.ShowFOV then
        FOVCircle = CreateFOVCircle()
    end
end

-- ============================================
-- 5. SISTEMA DE HIGHLIGHT
-- ============================================

local function ApplyHighlight(Player)
    if not Settings.PlayerHighlight then return end
    if not IsValidPlayer(Player) then return end
    
    local Character = Player.Character
    if not Character then return end
    
    local ExistingHighlight = Highlights[Player.Name]
    if ExistingHighlight then return end
    
    local Highlight = Instance.new("Highlight")
    Highlight.Name = "DarkAuraHighlight"
    Highlight.FillColor = Color3.fromRGB(138, 43, 226)
    Highlight.FillTransparency = 0.5
    Highlight.OutlineColor = Color3.fromRGB(255, 0, 255)
    Highlight.OutlineTransparency = 0.3
    Highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    Highlight.Parent = Character
    
    Highlights[Player.Name] = Highlight
end

local function UpdateHighlights()
    if not Settings.PlayerHighlight then
        for PlayerName, Highlight in pairs(Highlights) do
            if Highlight then
                Highlight:Destroy()
            end
        end
        Highlights = {}
        return
    end
    
    for PlayerName, Highlight in pairs(Highlights) do
        local Player = Players:FindFirstChild(PlayerName)
        if not Player or not IsValidPlayer(Player) then
            if Highlight then
                Highlight:Destroy()
            end
            Highlights[PlayerName] = nil
        end
    end
    
    for _, Player in ipairs(Players:GetPlayers()) do
        if IsValidPlayer(Player) then
            ApplyHighlight(Player)
        end
    end
end

-- ============================================
-- 6. SISTEMA DE LINHAS DE DIREÇÃO
-- ============================================

local function CreateDirectionLine(Player)
    if not Settings.DirectionLines then return nil end
    if not IsValidPlayer(Player) then return nil end
    
    local Character = Player.Character
    local LocalCharacter = LocalPlayer.Character
    
    if not Character or not LocalCharacter then return nil end
    
    local HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
    local LocalRootPart = LocalCharacter:FindFirstChild("HumanoidRootPart")
    
    if not HumanoidRootPart or not LocalRootPart then return nil end
    
    local Line = Instance.new("LineHandleAdornment")
    Line.Name = "DirectionLine_" .. Player.Name
    Line.Adornee = LocalRootPart
    Line.PointA = Vector3.new(0, 1, 0)
    Line.PointB = HumanoidRootPart.Position - LocalRootPart.Position
    Line.Color3 = Color3.fromRGB(0, 255, 255)
    Line.Thickness = 2
    Line.Transparency = 0.5
    Line.ZIndex = 0
    Line.AlwaysOnTop = true
    Line.Parent = LocalCharacter
    
    return Line
end

local function UpdateDirectionLines()
    if not Settings.DirectionLines then
        for PlayerName, Line in pairs(DirectionLinesObj) do
            if Line then
                Line:Destroy()
            end
        end
        DirectionLinesObj = {}
        return
    end
    
    local LocalCharacter = LocalPlayer.Character
    if not LocalCharacter then return end
    
    local LocalRootPart = LocalCharacter:FindFirstChild("HumanoidRootPart")
    if not LocalRootPart then return end
    
    for PlayerName, Line in pairs(DirectionLinesObj) do
        local Player = Players:FindFirstChild(PlayerName)
        if not Player or not IsValidPlayer(Player) then
            if Line then
                Line:Destroy()
            end
            DirectionLinesObj[PlayerName] = nil
        end
    end
    
    for _, Player in ipairs(Players:GetPlayers()) do
        if IsValidPlayer(Player) then
            local Character = Player.Character
            if Character then
                local TargetRootPart = Character:FindFirstChild("HumanoidRootPart")
                if TargetRootPart then
                    local Line = DirectionLinesObj[Player.Name]
                    if not Line then
                        Line = CreateDirectionLine(Player)
                        if Line then
                            DirectionLinesObj[Player.Name] = Line
                        end
                    elseif Line and Line:IsA("LineHandleAdornment") then
                        Line.PointB = TargetRootPart.Position - LocalRootPart.Position
                    end
                end
            end
        end
    end
end

-- ============================================
-- 7. INTERFACE PRINCIPAL
-- ============================================

local function LoadMainInterface()
    -- Criar janela principal
    MainWindow = Rayfield:CreateWindow({
        Name = "Dark Aura BR 🇧🇷",
        Icon = 0,
        LoadingTitle = "Carregando Dark Aura...",
        LoadingSubtitle = "Sistema de Visualização Avançado",
        ConfigurationSaving = {
            Enabled = true,
            FolderName = "DarkAuraBR",
            FileName = "Config"
        },
        Discord = {
            Enabled = false
        },
        KeySystem = false
    })
    
    -- Aba: Visual FOV
    local FOVTab = MainWindow:CreateTab("🎯 Visual FOV", nil)
    FOVTab:CreateSection("Configuração do Círculo FOV")
    
    FOVTab:CreateSlider({
        Name = "Tamanho do Círculo FOV",
        Range = {120, 1000},
        Increment = 10,
        Suffix = "px",
        CurrentValue = 120,
        Flag = "FOVSize",
        Callback = function(Value)
            Settings.FOVSize = Value
            UpdateFOVCircle()
        end
    })
    
    FOVTab:CreateToggle({
        Name = "Mostrar Círculo FOV",
        CurrentValue = true,
        Flag = "ShowFOV",
        Callback = function(Value)
            Settings.ShowFOV = Value
            ToggleFOVCircle()
        end
    })
    
    -- Aba: Visual de Players
    local VisualTab = MainWindow:CreateTab("👁️ Visual Players", nil)
    VisualTab:CreateSection("Destaque de Jogadores")
    
    VisualTab:CreateToggle({
        Name = "Highlight em Jogadores",
        CurrentValue = false,
        Flag = "PlayerHighlight",
        Callback = function(Value)
            Settings.PlayerHighlight = Value
            UpdateHighlights()
        end
    })
    
    -- Aba: Linhas de Debug
    local DebugTab = MainWindow:CreateTab("📏 Linhas Debug", nil)
    DebugTab:CreateSection("Linhas de Direção")
    
    DebugTab:CreateToggle({
        Name = "Linhas de Direção",
        CurrentValue = false,
        Flag = "DirectionLines",
        Callback = function(Value)
            Settings.DirectionLines = Value
            if not Value then
                for _, Line in pairs(DirectionLinesObj) do
                    if Line then
                        Line:Destroy()
                    end
                end
                DirectionLinesObj = {}
            end
        end
    })
    
    -- Aba: Informações
    local InfoTab = MainWindow:CreateTab("ℹ️ Informações", nil)
    InfoTab:CreateSection("Sobre o Dark Aura BR")
    
    InfoTab:CreateParagraph({
        Name = "Credits",
        Content = "Dark Aura BR 🇧🇷\nVersão: 2.0\n\n✅ SISTEMA AUTENTICADO COM SUCESSO!\n\nDesenvolvido para comunidade brasileira"
    })
    
    -- Inicializar o círculo FOV
    FOVCircle = CreateFOVCircle()
    
    -- Loop principal
    RunService.RenderStepped:Connect(function()
        if IsAuthenticated then
            if Settings.PlayerHighlight then
                UpdateHighlights()
            end
            
            if Settings.DirectionLines then
                UpdateDirectionLines()
            end
        end
    end)
    
    -- Notificação de boas-vindas
    Rayfield:Notify({
        Title = "✅ Dark Aura BR 🇧🇷",
        Content = "Sistema carregado com sucesso! Key: 5609",
        Duration = 3
    })
    
    print("[Dark Aura BR] ✅ SISTEMA CARREGADO COM SUCESSO!")
end

-- ============================================
-- 8. INICIAR SISTEMA
-- ============================================

-- Aguardar o jogador carregar
local function Start()
    -- Aguardar o PlayerGui
    local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
    
    -- Criar janela de autenticação
    CreateAuthWindow()
    
    print("[Dark Aura BR] 🔐 Sistema de autenticação iniciado.")
    print("[Dark Aura BR] ⚠️ Digite a key: 5609")
end

-- Iniciar o sistema
pcall(Start)
