-- ============================================================
--   Lifting Simulator | Script Local
--   Funcionalidades: Auto Treino | Auto Venda
--   Compatível com executores: Synapse X, KRNL, Fluxus, Delta
-- ============================================================

-- ╔══════════════════════════════════════════════╗
-- ║          SERVIÇOS E VARIÁVEIS GLOBAIS        ║
-- ╚══════════════════════════════════════════════╝

local Players        = game:GetService("Players")
local RunService     = game:GetService("RunService")
local TweenService   = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer    = Players.LocalPlayer
local PlayerGui      = LocalPlayer:WaitForChild("PlayerGui")

-- ╔══════════════════════════════════════════════╗
-- ║             CONFIGURAÇÕES DO SCRIPT          ║
-- ╚══════════════════════════════════════════════╝

local Config = {
    AutoTreinoAtivado  = false,
    AutoVendaAtivado   = false,
    IntervaloTreino    = 0.1,   -- segundos entre cada levantamento
    IntervaloVenda     = 3,     -- segundos entre cada venda
    VenderAoMaximo     = false, -- vender apenas quando atingir certo limite
}

-- ╔══════════════════════════════════════════════╗
-- ║            CORES E ESTILO DA GUI             ║
-- ╚══════════════════════════════════════════════╝

local Cores = {
    Fundo           = Color3.fromRGB(15, 15, 20),
    FundoSecundario = Color3.fromRGB(22, 22, 30),
    Barra           = Color3.fromRGB(30, 30, 42),
    BotaoAtivo      = Color3.fromRGB(0, 200, 100),
    BotaoInativo    = Color3.fromRGB(180, 50, 50),
    BotaoHover      = Color3.fromRGB(50, 220, 130),
    TextoPrimario   = Color3.fromRGB(240, 240, 255),
    TextoSecundario = Color3.fromRGB(160, 160, 190),
    Destaque        = Color3.fromRGB(100, 80, 255),
    Separador       = Color3.fromRGB(40, 40, 60),
    StatusAtivo     = Color3.fromRGB(0, 255, 120),
    StatusInativo   = Color3.fromRGB(255, 80, 80),
    Sombra          = Color3.fromRGB(0, 0, 0),
}

-- ╔══════════════════════════════════════════════╗
-- ║           FUNÇÕES AUXILIARES                 ║
-- ╚══════════════════════════════════════════════╝

local function criarTween(objeto, propriedades, duracao, estilo, direcao)
    local info = TweenInfo.new(
        duracao or 0.25,
        estilo or Enum.EasingStyle.Quad,
        direcao or Enum.EasingDirection.Out
    )
    return TweenService:Create(objeto, info, propriedades)
end

local function criarSombra(pai, tamanho)
    local sombra = Instance.new("ImageLabel")
    sombra.Name = "Sombra"
    sombra.BackgroundTransparency = 1
    sombra.Image = "rbxassetid://1316045217"
    sombra.ImageColor3 = Cores.Sombra
    sombra.ImageTransparency = 0.5
    sombra.ScaleType = Enum.ScaleType.Slice
    sombra.SliceCenter = Rect.new(10, 10, 118, 118)
    sombra.Size = UDim2.new(1, tamanho or 20, 1, tamanho or 20)
    sombra.Position = UDim2.new(0, -(tamanho or 20) / 2, 0, -(tamanho or 20) / 2)
    sombra.ZIndex = -1
    sombra.Parent = pai
    return sombra
end

-- ╔══════════════════════════════════════════════╗
-- ║           CONSTRUÇÃO DA GUI                  ║
-- ╚══════════════════════════════════════════════╝

-- Remove GUI antiga se existir
if PlayerGui:FindFirstChild("LiftingScriptGUI") then
    PlayerGui:FindFirstChild("LiftingScriptGUI"):Destroy()
end

-- ScreenGui principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "LiftingScriptGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 999
ScreenGui.Parent = PlayerGui

-- Janela principal
local Janela = Instance.new("Frame")
Janela.Name = "Janela"
Janela.Size = UDim2.new(0, 300, 0, 380)
Janela.Position = UDim2.new(0, 20, 0.5, -190)
Janela.BackgroundColor3 = Cores.Fundo
Janela.BorderSizePixel = 0
Janela.ClipsDescendants = true
Janela.Parent = ScreenGui

local JanelaCorner = Instance.new("UICorner")
JanelaCorner.CornerRadius = UDim.new(0, 12)
JanelaCorner.Parent = Janela

local JanelaStroke = Instance.new("UIStroke")
JanelaStroke.Color = Cores.Destaque
JanelaStroke.Thickness = 1.5
JanelaStroke.Transparency = 0.5
JanelaStroke.Parent = Janela

criarSombra(Janela, 30)

-- Barra de título
local BarraTitulo = Instance.new("Frame")
BarraTitulo.Name = "BarraTitulo"
BarraTitulo.Size = UDim2.new(1, 0, 0, 50)
BarraTitulo.Position = UDim2.new(0, 0, 0, 0)
BarraTitulo.BackgroundColor3 = Cores.Barra
BarraTitulo.BorderSizePixel = 0
BarraTitulo.Parent = Janela

local BarraCorner = Instance.new("UICorner")
BarraCorner.CornerRadius = UDim.new(0, 12)
BarraCorner.Parent = BarraTitulo

-- Correção visual: cobrir os cantos inferiores arredondados da barra
local BarraFix = Instance.new("Frame")
BarraFix.Size = UDim2.new(1, 0, 0, 12)
BarraFix.Position = UDim2.new(0, 0, 1, -12)
BarraFix.BackgroundColor3 = Cores.Barra
BarraFix.BorderSizePixel = 0
BarraFix.Parent = BarraTitulo

-- Ícone decorativo (gradiente lateral)
local Gradiente = Instance.new("UIGradient")
Gradiente.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Cores.Destaque),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 180, 255))
})
Gradiente.Rotation = 90
Gradiente.Parent = BarraTitulo

-- Título do script
local Titulo = Instance.new("TextLabel")
Titulo.Name = "Titulo"
Titulo.Size = UDim2.new(1, -100, 1, 0)
Titulo.Position = UDim2.new(0, 15, 0, 0)
Titulo.BackgroundTransparency = 1
Titulo.Text = "💪 Lifting Simulator"
Titulo.TextColor3 = Cores.TextoPrimario
Titulo.TextSize = 16
Titulo.Font = Enum.Font.GothamBold
Titulo.TextXAlignment = Enum.TextXAlignment.Left
Titulo.Parent = BarraTitulo

-- Subtítulo
local Subtitulo = Instance.new("TextLabel")
Subtitulo.Name = "Subtitulo"
Subtitulo.Size = UDim2.new(1, -100, 0, 16)
Subtitulo.Position = UDim2.new(0, 15, 0, 30)
Subtitulo.BackgroundTransparency = 1
Subtitulo.Text = "Script by LocalHub"
Subtitulo.TextColor3 = Cores.TextoSecundario
Subtitulo.TextSize = 11
Subtitulo.Font = Enum.Font.Gotham
Subtitulo.TextXAlignment = Enum.TextXAlignment.Left
Subtitulo.Parent = BarraTitulo

-- Botão minimizar
local BotaoMinimizar = Instance.new("TextButton")
BotaoMinimizar.Name = "BotaoMinimizar"
BotaoMinimizar.Size = UDim2.new(0, 28, 0, 28)
BotaoMinimizar.Position = UDim2.new(1, -65, 0.5, -14)
BotaoMinimizar.BackgroundColor3 = Color3.fromRGB(255, 190, 0)
BotaoMinimizar.BorderSizePixel = 0
BotaoMinimizar.Text = "—"
BotaoMinimizar.TextColor3 = Color3.fromRGB(30, 20, 0)
BotaoMinimizar.TextSize = 14
BotaoMinimizar.Font = Enum.Font.GothamBold
BotaoMinimizar.Parent = BarraTitulo

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(1, 0)
MinCorner.Parent = BotaoMinimizar

-- Botão fechar
local BotaoFechar = Instance.new("TextButton")
BotaoFechar.Name = "BotaoFechar"
BotaoFechar.Size = UDim2.new(0, 28, 0, 28)
BotaoFechar.Position = UDim2.new(1, -32, 0.5, -14)
BotaoFechar.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
BotaoFechar.BorderSizePixel = 0
BotaoFechar.Text = "✕"
BotaoFechar.TextColor3 = Color3.fromRGB(255, 255, 255)
BotaoFechar.TextSize = 14
BotaoFechar.Font = Enum.Font.GothamBold
BotaoFechar.Parent = BarraTitulo

local FecCorner = Instance.new("UICorner")
FecCorner.CornerRadius = UDim.new(1, 0)
FecCorner.Parent = BotaoFechar

-- Container do conteúdo
local Conteudo = Instance.new("Frame")
Conteudo.Name = "Conteudo"
Conteudo.Size = UDim2.new(1, 0, 1, -50)
Conteudo.Position = UDim2.new(0, 0, 0, 50)
Conteudo.BackgroundTransparency = 1
Conteudo.Parent = Janela

-- ── Seção: Status Geral ──────────────────────────────────────

local SecaoStatus = Instance.new("Frame")
SecaoStatus.Name = "SecaoStatus"
SecaoStatus.Size = UDim2.new(1, -24, 0, 60)
SecaoStatus.Position = UDim2.new(0, 12, 0, 12)
SecaoStatus.BackgroundColor3 = Cores.FundoSecundario
SecaoStatus.BorderSizePixel = 0
SecaoStatus.Parent = Conteudo

local SecaoStatusCorner = Instance.new("UICorner")
SecaoStatusCorner.CornerRadius = UDim.new(0, 8)
SecaoStatusCorner.Parent = SecaoStatus

local LabelStatusTitulo = Instance.new("TextLabel")
LabelStatusTitulo.Size = UDim2.new(1, -12, 0, 20)
LabelStatusTitulo.Position = UDim2.new(0, 12, 0, 8)
LabelStatusTitulo.BackgroundTransparency = 1
LabelStatusTitulo.Text = "STATUS DO SCRIPT"
LabelStatusTitulo.TextColor3 = Cores.TextoSecundario
LabelStatusTitulo.TextSize = 10
LabelStatusTitulo.Font = Enum.Font.GothamBold
LabelStatusTitulo.TextXAlignment = Enum.TextXAlignment.Left
LabelStatusTitulo.Parent = SecaoStatus

local LabelStatusValor = Instance.new("TextLabel")
LabelStatusValor.Name = "LabelStatusValor"
LabelStatusValor.Size = UDim2.new(1, -12, 0, 22)
LabelStatusValor.Position = UDim2.new(0, 12, 0, 28)
LabelStatusValor.BackgroundTransparency = 1
LabelStatusValor.Text = "⬤  Aguardando ativação..."
LabelStatusValor.TextColor3 = Cores.TextoSecundario
LabelStatusValor.TextSize = 13
LabelStatusValor.Font = Enum.Font.Gotham
LabelStatusValor.TextXAlignment = Enum.TextXAlignment.Left
LabelStatusValor.Parent = SecaoStatus

-- ── Separador ───────────────────────────────────────────────

local Sep1 = Instance.new("Frame")
Sep1.Size = UDim2.new(1, -24, 0, 1)
Sep1.Position = UDim2.new(0, 12, 0, 82)
Sep1.BackgroundColor3 = Cores.Separador
Sep1.BorderSizePixel = 0
Sep1.Parent = Conteudo

-- ── Seção: Auto Treino ───────────────────────────────────────

local SecaoTreino = Instance.new("Frame")
SecaoTreino.Name = "SecaoTreino"
SecaoTreino.Size = UDim2.new(1, -24, 0, 100)
SecaoTreino.Position = UDim2.new(0, 12, 0, 93)
SecaoTreino.BackgroundColor3 = Cores.FundoSecundario
SecaoTreino.BorderSizePixel = 0
SecaoTreino.Parent = Conteudo

local SecaoTreinoCorner = Instance.new("UICorner")
SecaoTreinoCorner.CornerRadius = UDim.new(0, 8)
SecaoTreinoCorner.Parent = SecaoTreino

local IconeTreino = Instance.new("TextLabel")
IconeTreino.Size = UDim2.new(0, 30, 0, 30)
IconeTreino.Position = UDim2.new(0, 12, 0, 10)
IconeTreino.BackgroundTransparency = 1
IconeTreino.Text = "🏋️"
IconeTreino.TextSize = 22
IconeTreino.Font = Enum.Font.Gotham
IconeTreino.Parent = SecaoTreino

local TituloTreino = Instance.new("TextLabel")
TituloTreino.Size = UDim2.new(1, -60, 0, 20)
TituloTreino.Position = UDim2.new(0, 48, 0, 10)
TituloTreino.BackgroundTransparency = 1
TituloTreino.Text = "AUTO TREINO"
TituloTreino.TextColor3 = Cores.TextoPrimario
TituloTreino.TextSize = 14
TituloTreino.Font = Enum.Font.GothamBold
TituloTreino.TextXAlignment = Enum.TextXAlignment.Left
TituloTreino.Parent = SecaoTreino

local DescTreino = Instance.new("TextLabel")
DescTreino.Size = UDim2.new(1, -24, 0, 16)
DescTreino.Position = UDim2.new(0, 12, 0, 36)
DescTreino.BackgroundTransparency = 1
DescTreino.Text = "Levanta pesos automaticamente a cada " .. Config.IntervaloTreino .. "s"
DescTreino.TextColor3 = Cores.TextoSecundario
DescTreino.TextSize = 11
DescTreino.Font = Enum.Font.Gotham
DescTreino.TextXAlignment = Enum.TextXAlignment.Left
DescTreino.Parent = SecaoTreino

local StatusTreino = Instance.new("TextLabel")
StatusTreino.Name = "StatusTreino"
StatusTreino.Size = UDim2.new(0, 80, 0, 18)
StatusTreino.Position = UDim2.new(0, 12, 0, 56)
StatusTreino.BackgroundTransparency = 1
StatusTreino.Text = "● INATIVO"
StatusTreino.TextColor3 = Cores.StatusInativo
StatusTreino.TextSize = 11
StatusTreino.Font = Enum.Font.GothamBold
StatusTreino.TextXAlignment = Enum.TextXAlignment.Left
StatusTreino.Parent = SecaoTreino

local BotaoTreino = Instance.new("TextButton")
BotaoTreino.Name = "BotaoTreino"
BotaoTreino.Size = UDim2.new(1, -24, 0, 34)
BotaoTreino.Position = UDim2.new(0, 12, 0, 56)
BotaoTreino.BackgroundColor3 = Cores.BotaoInativo
BotaoTreino.BorderSizePixel = 0
BotaoTreino.Text = "▶  ATIVAR AUTO TREINO"
BotaoTreino.TextColor3 = Cores.TextoPrimario
BotaoTreino.TextSize = 13
BotaoTreino.Font = Enum.Font.GothamBold
BotaoTreino.Parent = SecaoTreino

local BotaoTreinoCorner = Instance.new("UICorner")
BotaoTreinoCorner.CornerRadius = UDim.new(0, 6)
BotaoTreinoCorner.Parent = BotaoTreino

-- ── Separador ───────────────────────────────────────────────

local Sep2 = Instance.new("Frame")
Sep2.Size = UDim2.new(1, -24, 0, 1)
Sep2.Position = UDim2.new(0, 12, 0, 203)
Sep2.BackgroundColor3 = Cores.Separador
Sep2.BorderSizePixel = 0
Sep2.Parent = Conteudo

-- ── Seção: Auto Venda ────────────────────────────────────────

local SecaoVenda = Instance.new("Frame")
SecaoVenda.Name = "SecaoVenda"
SecaoVenda.Size = UDim2.new(1, -24, 0, 100)
SecaoVenda.Position = UDim2.new(0, 12, 0, 214)
SecaoVenda.BackgroundColor3 = Cores.FundoSecundario
SecaoVenda.BorderSizePixel = 0
SecaoVenda.Parent = Conteudo

local SecaoVendaCorner = Instance.new("UICorner")
SecaoVendaCorner.CornerRadius = UDim.new(0, 8)
SecaoVendaCorner.Parent = SecaoVenda

local IconeVenda = Instance.new("TextLabel")
IconeVenda.Size = UDim2.new(0, 30, 0, 30)
IconeVenda.Position = UDim2.new(0, 12, 0, 10)
IconeVenda.BackgroundTransparency = 1
IconeVenda.Text = "💰"
IconeVenda.TextSize = 22
IconeVenda.Font = Enum.Font.Gotham
IconeVenda.Parent = SecaoVenda

local TituloVenda = Instance.new("TextLabel")
TituloVenda.Size = UDim2.new(1, -60, 0, 20)
TituloVenda.Position = UDim2.new(0, 48, 0, 10)
TituloVenda.BackgroundTransparency = 1
TituloVenda.Text = "AUTO VENDA"
TituloVenda.TextColor3 = Cores.TextoPrimario
TituloVenda.TextSize = 14
TituloVenda.Font = Enum.Font.GothamBold
TituloVenda.TextXAlignment = Enum.TextXAlignment.Left
TituloVenda.Parent = SecaoVenda

local DescVenda = Instance.new("TextLabel")
DescVenda.Size = UDim2.new(1, -24, 0, 16)
DescVenda.Position = UDim2.new(0, 12, 0, 36)
DescVenda.BackgroundTransparency = 1
DescVenda.Text = "Vende músculo automaticamente a cada " .. Config.IntervaloVenda .. "s"
DescVenda.TextColor3 = Cores.TextoSecundario
DescVenda.TextSize = 11
DescVenda.Font = Enum.Font.Gotham
DescVenda.TextXAlignment = Enum.TextXAlignment.Left
DescVenda.Parent = SecaoVenda

local BotaoVenda = Instance.new("TextButton")
BotaoVenda.Name = "BotaoVenda"
BotaoVenda.Size = UDim2.new(1, -24, 0, 34)
BotaoVenda.Position = UDim2.new(0, 12, 0, 56)
BotaoVenda.BackgroundColor3 = Cores.BotaoInativo
BotaoVenda.BorderSizePixel = 0
BotaoVenda.Text = "▶  ATIVAR AUTO VENDA"
BotaoVenda.TextColor3 = Cores.TextoPrimario
BotaoVenda.TextSize = 13
BotaoVenda.Font = Enum.Font.GothamBold
BotaoVenda.Parent = SecaoVenda

local BotaoVendaCorner = Instance.new("UICorner")
BotaoVendaCorner.CornerRadius = UDim.new(0, 6)
BotaoVendaCorner.Parent = BotaoVenda

-- ── Rodapé ───────────────────────────────────────────────────

local Rodape = Instance.new("TextLabel")
Rodape.Size = UDim2.new(1, 0, 0, 20)
Rodape.Position = UDim2.new(0, 0, 1, -24)
Rodape.BackgroundTransparency = 1
Rodape.Text = "Pressione [Insert] para mostrar/ocultar"
Rodape.TextColor3 = Cores.TextoSecundario
Rodape.TextSize = 10
Rodape.Font = Enum.Font.Gotham
Rodape.Parent = Conteudo

-- ╔══════════════════════════════════════════════╗
-- ║          ARRASTAR JANELA (DRAG)              ║
-- ╚══════════════════════════════════════════════╝

local arrastando = false
local offsetArrastar = Vector2.new()

BarraTitulo.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or
       input.UserInputType == Enum.UserInputType.Touch then
        arrastando = true
        offsetArrastar = Vector2.new(
            input.Position.X - Janela.AbsolutePosition.X,
            input.Position.Y - Janela.AbsolutePosition.Y
        )
    end
end)

BarraTitulo.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or
       input.UserInputType == Enum.UserInputType.Touch then
        arrastando = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if arrastando and (input.UserInputType == Enum.UserInputType.MouseMovement or
       input.UserInputType == Enum.UserInputType.Touch) then
        Janela.Position = UDim2.new(
            0, input.Position.X - offsetArrastar.X,
            0, input.Position.Y - offsetArrastar.Y
        )
    end
end)

-- ╔══════════════════════════════════════════════╗
-- ║       MINIMIZAR / FECHAR / TOGGLE GUI        ║
-- ╚══════════════════════════════════════════════╝

local minimizado = false

local function toggleMinimizar()
    minimizado = not minimizado
    local alvo = minimizado and UDim2.new(0, 300, 0, 50) or UDim2.new(0, 300, 0, 380)
    criarTween(Janela, {Size = alvo}, 0.3, Enum.EasingStyle.Quart):Play()
    BotaoMinimizar.Text = minimizado and "□" or "—"
end

BotaoMinimizar.MouseButton1Click:Connect(toggleMinimizar)

BotaoFechar.MouseButton1Click:Connect(function()
    criarTween(Janela, {Size = UDim2.new(0, 0, 0, 0)}, 0.3, Enum.EasingStyle.Quart):Play()
    task.wait(0.35)
    ScreenGui:Destroy()
end)

-- Tecla [Insert] para mostrar/ocultar
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Insert then
        Janela.Visible = not Janela.Visible
    end
end)

-- ╔══════════════════════════════════════════════╗
-- ║         ATUALIZAR STATUS NA GUI              ║
-- ╚══════════════════════════════════════════════╝

local function atualizarStatus()
    local treino = Config.AutoTreinoAtivado
    local venda  = Config.AutoVendaAtivado

    if treino and venda then
        LabelStatusValor.Text = "⬤  Treino + Venda Ativos"
        LabelStatusValor.TextColor3 = Cores.StatusAtivo
    elseif treino then
        LabelStatusValor.Text = "⬤  Auto Treino Ativo"
        LabelStatusValor.TextColor3 = Cores.StatusAtivo
    elseif venda then
        LabelStatusValor.Text = "⬤  Auto Venda Ativa"
        LabelStatusValor.TextColor3 = Color3.fromRGB(255, 200, 0)
    else
        LabelStatusValor.Text = "⬤  Aguardando ativação..."
        LabelStatusValor.TextColor3 = Cores.TextoSecundario
    end

    -- Botão Treino
    if treino then
        BotaoTreino.Text = "⏹  DESATIVAR AUTO TREINO"
        criarTween(BotaoTreino, {BackgroundColor3 = Cores.BotaoAtivo}, 0.2):Play()
    else
        BotaoTreino.Text = "▶  ATIVAR AUTO TREINO"
        criarTween(BotaoTreino, {BackgroundColor3 = Cores.BotaoInativo}, 0.2):Play()
    end

    -- Botão Venda
    if venda then
        BotaoVenda.Text = "⏹  DESATIVAR AUTO VENDA"
        criarTween(BotaoVenda, {BackgroundColor3 = Cores.BotaoAtivo}, 0.2):Play()
    else
        BotaoVenda.Text = "▶  ATIVAR AUTO VENDA"
        criarTween(BotaoVenda, {BackgroundColor3 = Cores.BotaoInativo}, 0.2):Play()
    end
end

-- ╔══════════════════════════════════════════════╗
-- ║       LÓGICA: AUTO TREINO (LiftWeight)       ║
-- ╚══════════════════════════════════════════════╝

--[[
    O Lifting Simulator usa o RemoteEvent dentro de ReplicatedStorage.
    O evento "LiftWeight" é disparado para registrar cada levantamento.
    Também tentamos equipar a ferramenta de peso automaticamente.
]]

local function obterRemote()
    -- Tenta localizar o RemoteEvent principal do jogo
    local rs = game:GetService("ReplicatedStorage")

    -- Método 1: RemoteEvent direto
    local remote = rs:FindFirstChild("RemoteEvent")
    if remote then return remote end

    -- Método 2: Dentro de uma pasta
    for _, v in ipairs(rs:GetDescendants()) do
        if v:IsA("RemoteEvent") then
            return v
        end
    end

    return nil
end

local function equiparPeso()
    local char = LocalPlayer.Character
    if not char then return end
    local backpack = LocalPlayer.Backpack
    for _, tool in ipairs(backpack:GetChildren()) do
        if tool:IsA("Tool") then
            -- Equipa qualquer ferramenta disponível (peso)
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                tool.Parent = char
            end
        end
    end
end

local function executarLevantamento()
    -- Tenta via RemoteEvent direto
    local remote = obterRemote()
    if remote then
        pcall(function()
            remote:FireServer("LiftWeight")
        end)
    end

    -- Tenta via ReplicatedStorage.RemoteEvent (padrão do jogo)
    pcall(function()
        game:GetService("ReplicatedStorage").RemoteEvent:FireServer("LiftWeight")
    end)

    -- Tenta equipar e usar a ferramenta fisicamente
    equiparPeso()
    local char = LocalPlayer.Character
    if char then
        local tool = char:FindFirstChildOfClass("Tool")
        if tool then
            local activateEvent = tool:FindFirstChild("Activate")
            if activateEvent then
                pcall(function() activateEvent:FireServer() end)
            end
            -- Simula ativação da ferramenta
            pcall(function()
                tool:Activate()
            end)
        end
    end
end

-- Loop de Auto Treino
task.spawn(function()
    while true do
        if Config.AutoTreinoAtivado then
            pcall(executarLevantamento)
        end
        task.wait(Config.IntervaloTreino)
    end
end)

-- ╔══════════════════════════════════════════════╗
-- ║       LÓGICA: AUTO VENDA (SellMuscle)        ║
-- ╚══════════════════════════════════════════════╝

local function executarVenda()
    -- Método 1: RemoteEvent direto com "SellMuscle"
    pcall(function()
        game:GetService("ReplicatedStorage").RemoteEvent:FireServer("SellMuscle")
    end)

    -- Método 2: Busca por qualquer RemoteEvent e tenta vender
    local remote = obterRemote()
    if remote then
        pcall(function()
            remote:FireServer("SellMuscle")
        end)
    end

    -- Método 3: Tenta via tabela de argumentos (formato alternativo do jogo)
    pcall(function()
        local tbl = {"SellMuscle"}
        game:GetService("ReplicatedStorage").RemoteEvent:FireServer(unpack(tbl))
    end)
end

-- Loop de Auto Venda
task.spawn(function()
    while true do
        if Config.AutoVendaAtivado then
            pcall(executarVenda)
        end
        task.wait(Config.IntervaloVenda)
    end
end)

-- ╔══════════════════════════════════════════════╗
-- ║         EVENTOS DOS BOTÕES                   ║
-- ╚══════════════════════════════════════════════╝

BotaoTreino.MouseButton1Click:Connect(function()
    Config.AutoTreinoAtivado = not Config.AutoTreinoAtivado
    atualizarStatus()

    -- Animação de clique
    criarTween(BotaoTreino, {Size = UDim2.new(1, -28, 0, 30)}, 0.08):Play()
    task.wait(0.08)
    criarTween(BotaoTreino, {Size = UDim2.new(1, -24, 0, 34)}, 0.12):Play()
end)

BotaoVenda.MouseButton1Click:Connect(function()
    Config.AutoVendaAtivado = not Config.AutoVendaAtivado
    atualizarStatus()

    -- Animação de clique
    criarTween(BotaoVenda, {Size = UDim2.new(1, -28, 0, 30)}, 0.08):Play()
    task.wait(0.08)
    criarTween(BotaoVenda, {Size = UDim2.new(1, -24, 0, 34)}, 0.12):Play()
end)

-- Efeito hover nos botões
BotaoTreino.MouseEnter:Connect(function()
    if not Config.AutoTreinoAtivado then
        criarTween(BotaoTreino, {BackgroundColor3 = Color3.fromRGB(220, 70, 70)}, 0.15):Play()
    end
end)
BotaoTreino.MouseLeave:Connect(function()
    if not Config.AutoTreinoAtivado then
        criarTween(BotaoTreino, {BackgroundColor3 = Cores.BotaoInativo}, 0.15):Play()
    end
end)

BotaoVenda.MouseEnter:Connect(function()
    if not Config.AutoVendaAtivado then
        criarTween(BotaoVenda, {BackgroundColor3 = Color3.fromRGB(220, 70, 70)}, 0.15):Play()
    end
end)
BotaoVenda.MouseLeave:Connect(function()
    if not Config.AutoVendaAtivado then
        criarTween(BotaoVenda, {BackgroundColor3 = Cores.BotaoInativo}, 0.15):Play()
    end
end)

-- ╔══════════════════════════════════════════════╗
-- ║         ANIMAÇÃO DE ENTRADA DA GUI           ║
-- ╚══════════════════════════════════════════════╝

Janela.Size = UDim2.new(0, 0, 0, 0)
Janela.Position = UDim2.new(0, 20, 0.5, 0)

task.spawn(function()
    task.wait(0.1)
    criarTween(Janela, {
        Size = UDim2.new(0, 300, 0, 380),
        Position = UDim2.new(0, 20, 0.5, -190)
    }, 0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out):Play()
end)

-- ╔══════════════════════════════════════════════╗
-- ║              INICIALIZAÇÃO                   ║
-- ╚══════════════════════════════════════════════╝

atualizarStatus()

print("[LiftingScript] ✅ Script carregado com sucesso!")
print("[LiftingScript] 🏋️  Auto Treino: PRONTO")
print("[LiftingScript] 💰  Auto Venda:  PRONTO")
print("[LiftingScript] ⌨️   Tecla [Insert] para mostrar/ocultar a GUI")
