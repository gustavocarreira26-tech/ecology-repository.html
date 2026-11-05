

<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width , initial-scale=1.0, maximum-scale=5.0">
    <title>EcoComércio - Monitor de Energia Simplificado</title>
    <!-- Tailwind CSS para estilização leve e otimizada -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#f0fdf4',
                            100: '#dcfce7',
                            500: '#22c55e',
                            600: '#16a34a',
                            700: '#15803d',
                            900: '#14532d',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        /* Estilos CSS nativos leves para suporte e acessibilidade */
        body {
            font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            -webkit-tap-highlight-color: transparent;
            font-size: 16px;
        }

        /* Classes dinâmicas para alteração do tamanho do texto via JavaScript */
        .text-size-sm body { font-size: 14px; }
        .text-size-md body { font-size: 16px; }
        .text-size-lg body { font-size: 19px; }
        .text-size-xl body { font-size: 22px; }

        /* Botões focados em acessibilidade ao toque para celulares */
        .touch-target {
            min-height: 48px;
            min-width: 48px;
        }

        /* Transição suave de tema sem impactar CPU/bateria */
        .theme-transition {
            transition: background-color 0.2s ease, color 0.2s ease, border-color 0.2s ease;
        }

        /* Scrollbar customizada ultra-fina */
        ::-webkit-scrollbar {
            width: 4px;
            height: 4px;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        .dark ::-webkit-scrollbar-thumb {
            background: #475569;
        }
    </style>
</head>
<body class="bg-gray-100 dark:bg-gray-950 text-gray-900 dark:text-gray-100 min-h-screen theme-transition pb-24">

    <!-- CABEÇALHO & BARRA DE ACESSIBILIDADE FIXA -->
    <header class="sticky top-0 z-50 bg-white dark:bg-gray-900 border-b border-gray-200 dark:border-gray-800 shadow-sm px-3 py-2">
        <div class="max-w-md mx-auto flex items-center justify-between gap-2">
            
            <!-- Título e Nome do Comércio -->
            <div class="flex items-center gap-2">
                <div class="bg-brand-600 text-white p-2 rounded-xl font-bold flex items-center justify-center w-10 h-10 shadow-sm" aria-hidden="true">
                    ⚡
                </div>
                <div>
                    <h1 class="font-extrabold text-base leading-tight text-gray-900 dark:text-white flex items-center gap-1">
                        EcoComércio
                    </h1>
                    <button onclick="alterarNomeComercio()" class="text-xs text-brand-700 dark:text-brand-400 font-medium underline flex items-center gap-1" title="Clique para mudar o nome do seu comércio">
                        <span id="nome-comercio-display">Meu Comércio Local</span> ✏️
                    </button>
                </div>
            </div>

            <!-- Ferramentas de Acessibilidade e Tema -->
            <div class="flex items-center gap-1 bg-gray-100 dark:bg-gray-800 p-1 rounded-xl border border-gray-200 dark:border-gray-700">
                <!-- Tamanho da Fonte -->
                <button onclick="ajustarFonte(-1)" class="touch-target px-2 font-bold text-xs hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg" title="Diminuir fonte" aria-label="Diminuir tamanho da fonte">A-</button>
                <button onclick="resetarFonte()" class="touch-target px-1.5 font-semibold text-xs hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg" title="Resetar fonte" aria-label="Tamanho normal da fonte">100%</button>
                <button onclick="ajustarFonte(1)" class="touch-target px-2 font-bold text-xs hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg" title="Aumentar fonte" aria-label="Aumentar tamanho da fonte">A+</button>
                
                <!-- Leitor de Voz -->
                <button onclick="ouvirResumoVoz()" id="btn-voz" class="touch-target px-2 text-base hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg text-brand-600 dark:text-brand-400" title="Ouvir resumo em voz alta" aria-label="Ouvir resumo do dia em voz alta">
                    🔊
                </button>

                <!-- Modo Escuro -->
                <button onclick="alternarTema()" id="btn-tema" class="touch-target px-2 text-base hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg" title="Alternar Modo Escuro" aria-label="Alternar modo claro e escuro">
                    🌙
                </button>
            </div>
        </div>
    </header>

    <!-- CONTEÚDO PRINCIPAL (MÁXIMO 480px PARA DESIGN MOBILE PERFECT) -->
    <main class="max-w-md mx-auto px-4 pt-4 space-y-4">

        <!-- ALERTA RÁPIDO DE CONSUMO FANTASMA (VISÍVEL APENAS QUANDO HOUVER STANDBY) -->
        <div id="alerta-fantasma-topo" class="hidden bg-amber-500/10 border-2 border-amber-500 text-amber-900 dark:text-amber-200 p-3.5 rounded-2xl flex items-start gap-3 shadow-sm">
            <span class="text-2xl flex-shrink-0" aria-hidden="true">⚠️</span>
            <div class="flex-1 text-xs sm:text-sm">
                <p class="font-bold text-amber-900 dark:text-amber-300">Atenção! Gasto sem uso detectado</p>
                <p id="alerta-fantasma-texto" class="mt-0.5 opacity-90">Existem equipamentos no modo "Standby" jogando dinheiro fora agora.</p>
            </div>
            <button onclick="irParaAba('relatorio')" class="text-xs font-bold bg-amber-500 text-gray-900 px-2.5 py-1.5 rounded-xl self-center shadow-xs hover:bg-amber-400">
                Ver
            </button>
        </div>

        <!-- ABA 1: RESUMO DO DIA -->
        <section id="aba-resumo" class="space-y-4">
            
            <!-- Cartão Principal de Gasto Estimado -->
            <div class="bg-gradient-to-br from-brand-600 to-brand-800 text-white p-5 rounded-3xl shadow-lg relative overflow-hidden">
                <div class="flex justify-between items-center mb-1">
                    <span class="text-xs font-semibold tracking-wide uppercase text-brand-100">Custo Estimado Hoje</span>
                    <span class="text-xs bg-white/20 backdrop-blur-sm px-2.5 py-0.5 rounded-full text-white font-medium" id="data-hoje">Hoje</span>
                </div>

                <div class="flex items-baseline gap-1 my-1">
                    <span class="text-xl font-bold">R$</span>
                    <span id="resumo-custo-total" class="text-4xl font-black tracking-tight">0,00</span>
                    <span class="text-xs text-brand-100 ml-1">/dia</span>
                </div>

                <!-- Detalhes do Consumo em KWh e Comparador -->
                <div class="mt-4 pt-3 border-t border-white/20 grid grid-cols-2 gap-2 text-xs">
                    <div>
                        <p class="text-brand-100">Consumo em energia</p>
                        <p class="font-bold text-sm mt-0.5"><span id="resumo-kwh-total">0.0</span> kWh/dia</p>
                    </div>
                    <div class="text-right">
                        <p class="text-brand-100">Projeção Mensal (30d)</p>
                        <p class="font-bold text-sm mt-0.5 text-emerald-200">R$ <span id="resumo-projecao-mes">0,00</span></p>
                    </div>
                </div>
            </div>

            <!-- Preço da tarifa editável em destaque -->
            <div class="bg-white dark:bg-gray-900 p-3 rounded-2xl border border-gray-200 dark:border-gray-800 flex justify-between items-center text-xs shadow-xs">
                <span class="text-gray-600 dark:text-gray-400">Valor cobrado pela concessionária:</span>
                <button onclick="alterarTarifa()" class="font-bold text-gray-900 dark:text-white bg-gray-100 dark:bg-gray-800 px-3 py-1.5 rounded-xl underline" title="Clique para alterar o valor do kWh">
                    R$ <span id="tarifa-display">0.85</span> / kWh ✏️
                </button>
            </div>

            <!-- Três Cartões Rápidos de Status -->
            <div class="grid grid-cols-3 gap-2">
                <div class="bg-white dark:bg-gray-900 p-3 rounded-2xl border border-gray-200 dark:border-gray-800 text-center shadow-xs">
                    <span class="text-lg block mb-1">🟢</span>
                    <span class="text-[11px] text-gray-500 dark:text-gray-400 font-medium block">Em Uso</span>
                    <strong id="qtd-ligados" class="text-lg font-bold text-gray-800 dark:text-gray-100">0</strong>
                </div>
                <div class="bg-white dark:bg-gray-900 p-3 rounded-2xl border border-gray-200 dark:border-gray-800 text-center shadow-xs">
                    <span class="text-lg block mb-1">🟡</span>
                    <span class="text-[11px] text-gray-500 dark:text-gray-400 font-medium block">Sem Uso</span>
                    <strong id="qtd-standby" class="text-lg font-bold text-amber-600 dark:text-amber-400">0</strong>
                </div>
                <div class="bg-white dark:bg-gray-900 p-3 rounded-2xl border border-gray-200 dark:border-gray-800 text-center shadow-xs">
                    <span class="text-lg block mb-1">🔴</span>
                    <span class="text-[11px] text-gray-500 dark:text-gray-400 font-medium block">Desligados</span>
                    <strong id="qtd-desligados" class="text-lg font-bold text-gray-400">0</strong>
                </div>
            </div>

            <!-- Gráfico Leve em SVG de Consumo por Equipamento -->
            <div class="bg-white dark:bg-gray-900 p-4 rounded-3xl border border-gray-200 dark:border-gray-800 shadow-xs space-y-3">
                <div class="flex justify-between items-center">
                    <h2 class="font-bold text-sm text-gray-800 dark:text-gray-100 flex items-center gap-1.5">
                        📊 Distribuição de Gasto Diário
                    </h2>
                    <span class="text-[10px] bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-300 px-2 py-0.5 rounded-full font-medium">Por Aparelho</span>
                </div>

                <!-- Simulação Visual em Barras SVG -->
                <div class="space-y-2 pt-1" id="grafico-barras-container">
                    <!-- Gerado via JS -->
                </div>
            </div>

            <!-- Card Dica Rápida -->
            <div class="bg-emerald-500/10 border border-emerald-500/30 p-4 rounded-2xl flex items-start gap-3">
                <span class="text-2xl">💡</span>
                <div class="text-xs text-emerald-900 dark:text-emerald-200 space-y-1">
                    <strong class="block text-emerald-800 dark:text-emerald-300 font-bold text-sm">Dica Simples de Economia:</strong>
                    <p id="dica-rapida-home">Mantenha as borrachas de vedação de freezers e balcões limpas para evitar fuga de ar frio.</p>
                </div>
            </div>

        </section>

        <!-- ABA 2: MEUS EQUIPAMENTOS -->
        <section id="aba-equipamentos" class="space-y-4 hidden">
            
            <div class="flex items-center justify-between">
                <div>
                    <h2 class="font-extrabold text-lg text-gray-900 dark:text-white">Seus Equipamentos</h2>
                    <p class="text-xs text-gray-500 dark:text-gray-400">Toque nos botões para mudar o status</p>
                </div>
                <button onclick="abrirModalAdd()" class="touch-target bg-brand-600 hover:bg-brand-700 text-white px-3 py-2 rounded-2xl text-xs font-bold shadow-md flex items-center gap-1">
                    ➕ Adicionar
                </button>
            </div>

            <!-- Lista de Equipamentos -->
            <div id="lista-equipamentos" class="space-y-3">
                <!-- Itens injetados via JavaScript -->
            </div>

        </section>

        <!-- ABA 3: RELATÓRIO DIÁRIO E ALERTAS -->
        <section id="aba-relatorio" class="space-y-4 hidden">
            
            <div class="flex items-center justify-between">
                <div>
                    <h2 class="font-extrabold text-lg text-gray-900 dark:text-white">Relatório Completo do Dia</h2>
                    <p class="text-xs text-gray-500 dark:text-gray-400">Análise automática e simples do consumo</p>
                </div>
                <button onclick="imprimirRelatorio()" class="touch-target bg-gray-800 dark:bg-gray-700 hover:bg-gray-900 text-white px-3 py-2 rounded-2xl text-xs font-bold shadow-md flex items-center gap-1">
                    🖨️ Exportar / Imprimir
                </button>
            </div>

            <!-- Card: Campeão de Gasto -->
            <div class="bg-white dark:bg-gray-900 p-4 rounded-3xl border border-gray-200 dark:border-gray-800 shadow-xs space-y-2">
                <div class="flex items-center gap-2 text-red-600 dark:text-red-400 font-bold text-xs uppercase tracking-wider">
                    <span>👑</span> O Vilão da Conta de Luz
                </div>
                <div id="relatorio-campeao" class="flex items-center justify-between pt-1">
                    <!-- Injetado via JS -->
                </div>
            </div>

            <!-- Card: Alerta Standby / Consumo Fantasma -->
            <div class="bg-white dark:bg-gray-900 p-4 rounded-3xl border border-gray-200 dark:border-gray-800 shadow-xs space-y-3">
                <div class="flex items-center gap-2 text-amber-600 dark:text-amber-400 font-bold text-xs uppercase tracking-wider">
                    <span>👻</span> Desperdício Vampiro (Sem Uso)
                </div>
                <div id="relatorio-fantasma-conteudo">
                    <!-- Injetado via JS -->
                </div>
            </div>

            <!-- Card: Horário de Pico de Consumo -->
            <div class="bg-white dark:bg-gray-900 p-4 rounded-3xl border border-gray-200 dark:border-gray-800 shadow-xs space-y-2">
                <div class="flex items-center gap-2 text-blue-600 dark:text-blue-400 font-bold text-xs uppercase tracking-wider">
                    <span>⏰</span> Dicas para Horários de Pico
                </div>
                <div id="relatorio-pico" class="text-xs text-gray-600 dark:text-gray-300 space-y-1">
                    <!-- Injetado via JS -->
                </div>
            </div>

            <!-- Lista de Dicas Personalizadas -->
            <div class="bg-white dark:bg-gray-900 p-4 rounded-3xl border border-gray-200 dark:border-gray-800 shadow-xs space-y-3">
                <h3 class="font-bold text-sm text-gray-900 dark:text-white flex items-center gap-1.5">
                    <span>🎯</span> Dicas Práticas de Economia
                </h3>
                <ul id="lista-dicas-personalizadas" class="space-y-2 text-xs text-gray-600 dark:text-gray-300">
                    <!-- Injetado via JS -->
                </ul>
            </div>

        </section>

    </main>

    <!-- NAVEGAÇÃO INFERIOR FIXA (MOBILE BOTTOM BAR) -->
    <nav class="fixed bottom-0 left-0 right-0 z-40 bg-white/95 dark:bg-gray-900/95 backdrop-blur-md border-t border-gray-200 dark:border-gray-800 px-3 py-1.5">
        <div class="max-w-md mx-auto grid grid-cols-3 gap-1">
            <button onclick="irParaAba('resumo')" id="nav-resumo" class="touch-target flex flex-col items-center justify-center rounded-2xl py-1 text-brand-600 dark:text-brand-400 font-bold text-xs transition-colors">
                <span class="text-lg leading-none mb-1">🏠</span>
                <span>Início</span>
            </button>
            <button onclick="irParaAba('equipamentos')" id="nav-equipamentos" class="touch-target flex flex-col items-center justify-center rounded-2xl py-1 text-gray-500 dark:text-gray-400 font-medium text-xs transition-colors">
                <span class="text-lg leading-none mb-1">🔌</span>
                <span>Equipamentos</span>
            </button>
            <button onclick="irParaAba('relatorio')" id="nav-relatorio" class="touch-target flex flex-col items-center justify-center rounded-2xl py-1 text-gray-500 dark:text-gray-400 font-medium text-xs transition-colors">
                <span class="text-lg leading-none mb-1">📈</span>
                <span>Relatório</span>
            </button>
        </div>
    </nav>

    <!-- MODAL DE ADICIONAR / EDITAR EQUIPAMENTO -->
    <div id="modal-equipamento" class="fixed inset-0 z-50 bg-black/60 backdrop-blur-xs flex items-end sm:items-center justify-center p-0 sm:p-4 hidden">
        <div class="bg-white dark:bg-gray-900 w-full max-w-md rounded-t-3xl sm:rounded-3xl p-5 space-y-4 max-h-[90vh] overflow-y-auto border border-gray-200 dark:border-gray-800">
            <div class="flex justify-between items-center border-b border-gray-100 dark:border-gray-800 pb-3">
                <h3 class="font-extrabold text-base text-gray-900 dark:text-white" id="modal-titulo">Novo Equipamento</h3>
                <button onclick="fecharModalAdd()" class="touch-target text-gray-400 hover:text-gray-600 text-xl font-bold px-2">✕</button>
            </div>

            <form id="form-equipamento" onsubmit="salvarEquipamento(event)" class="space-y-4">
                <input type="hidden" id="eq-id">

                <!-- Selecionar Pré-definido ou Personalizado -->
                <div>
                    <label class="block text-xs font-bold text-gray-700 dark:text-gray-300 mb-1">Escolher Equipamento Comum:</label>
                    <select id="eq-predefinido" onchange="selecionarPredefinido(this.value)" class="w-full touch-target p-2.5 rounded-xl border border-gray-300 dark:border-gray-700 bg-gray-50 dark:bg-gray-800 text-sm font-medium text-gray-900 dark:text-white">
                        <option value="">-- Escolher da lista pronta --</option>
                        <option value="Geladeira Comercial|750|🧊">🧊 Geladeira Comercial (Médio/Grande)</option>
                        <option value="Freezer Vertical|500|❄️">❄️ Freezer Vertical</option>
                        <option value="Balcão Frigorífico|900|🥩">🥩 Balcão Frigorífico / Açougue</option>
                        <option value="Fritadeira Elétrica|2500|🍟">🍟 Fritadeira Elétrica Comercial</option>
                        <option value="Chapa Elétrica|3000|🍔">🍔 Chapa Elétrica Comercial</option>
                        <option value="Ar Condicionado|1400|❄️">❄️ Ar Condicionado (12.000 BTU)</option>
                        <option value="Cafeteira Industrial|1500|☕">☕ Cafeteira Industrial</option>
                        <option value="Iluminação da Loja|200|💡">💡 Iluminação Geral (Lâmpadas)</option>
                        <option value="Computador / Caixa|150|🖥️">🖥️ Computador / Sistema de Caixa</option>
                    </select>
                </div>

                <div>
                    <label class="block text-xs font-bold text-gray-700 dark:text-gray-300 mb-1">Nome do Equipamento:</label>
                    <input type="text" id="eq-nome" required placeholder="Ex: Freezer de Sorvetes" class="w-full touch-target p-2.5 rounded-xl border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 text-sm text-gray-900 dark:text-white">
                </div>

                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-bold text-gray-700 dark:text-gray-300 mb-1">Potência (Watts):</label>
                        <input type="number" id="eq-watts" required min="1" max="20000" placeholder="Ex: 50
