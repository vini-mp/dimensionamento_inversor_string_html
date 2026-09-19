# Dimensionamento de Inversor String

Ferramenta web para dimensionar a quantidade de módulos fotovoltaicos por MPPT em um inversor do tipo string. A partir dos dados do módulo e do inversor, a aplicação calcula os limites de potência, tensão e corrente e apresenta os resultados junto com os valores de conferência de cada cálculo.

O projeto é um único arquivo `index.html`, sem dependências e sem necessidade de instalação.

## Funcionalidades

- Correção do Voc do módulo em função da variação de temperatura
- Cálculo da quantidade máxima de módulos pela potência de entrada CC do inversor
- Cálculo da quantidade máxima de módulos por MPPT (limites de tensão)
- Cálculo da quantidade mínima de módulos por MPPT (tensão de partida)
- Verificação da quantidade de entradas utilizáveis em cada MPPT (limites de corrente)
- Suporte a inversores com MPPT iguais ou com especificações diferentes
- Validação dos campos em tempo real, com bloqueio do cálculo em caso de inconsistência
- Tabela de valores de conferência, com a conta de cada etapa
- Exportação e importação dos dados em arquivo JSON
- Geração de relatório para impressão ou salvamento em PDF

## Como usar

1. Baixe ou clone o repositório.
2. Abra o arquivo `index.html` em qualquer navegador moderno.
3. Preencha os dados do módulo fotovoltaico e do inversor.
4. Clique em **Calcular Dimensionamento**.

> As fontes (Inter e Space Grotesk) são carregadas do Google Fonts. Sem conexão com a internet a ferramenta continua funcionando, apenas com a fonte padrão do navegador.

### Publicação no GitHub Pages (opcional)

Em **Settings → Pages**, selecione a branch `main` e a pasta `/ (root)`. Após alguns instantes a ferramenta ficará disponível em um endereço público.

## Dados de entrada

### Módulo fotovoltaico

| Campo | Descrição |
|-------|-----------|
| Voc | Tensão de circuito aberto (V) |
| Icc | Corrente de curto-circuito (A) |
| Vmax | Tensão de máxima potência (V) |
| Imax | Corrente de máxima potência (A) |
| Potência do módulo | Potência nominal (W) |
| Coeficiente de temperatura do Voc | Variação do Voc por °C (%/°C) |
| Variação de temperatura | Diferença de temperatura em relação à condição de referência (°C) |

### Inversor

| Campo | Descrição |
|-------|-----------|
| Potência máxima de entrada CC | Potência máxima admitida na entrada (W) |
| Tensão máxima de entrada | Limite absoluto de tensão suportado (V) |
| Tensão máxima da faixa de operação | Limite superior da faixa de operação do MPPT (V) |
| Tensão de partida | Tensão mínima para o inversor iniciar a operação (V) |
| Número de MPPT | Quantidade de rastreadores de ponto de máxima potência |
| Corrente máxima de entrada por MPPT | Limite de corrente de operação (A) |
| Corrente máxima de curto por MPPT | Limite de corrente de curto-circuito (A) |
| Número de entradas do MPPT | Quantidade de entradas (strings) de cada MPPT |

Quando a opção **Os MPPT são iguais** está marcada, as especificações informadas são aplicadas a todos os MPPT. Desmarcada, cada MPPT recebe seus próprios campos.

## Cálculos realizados

**Voc corrigido pela temperatura**

```
Voc_corr = Voc + (ΔT × (Coef_Temp / 100) × Voc)
```

> O sinal do coeficiente e o da variação de temperatura são usados diretamente na fórmula. Para condições de frio (ΔT negativo), o Voc corrigido só resulta maior que o Voc nominal se o coeficiente também for informado com sinal negativo (por exemplo, `-0.29`).

**Cálculos de conferência**

| # | Cálculo | Uso |
|---|---------|-----|
| 1 | Potência máx. CC ÷ Potência do módulo | Máximo de módulos pela potência (arredondado para baixo) |
| 2 | Tensão máx. de entrada ÷ Voc corrigido | Limite de módulos em série pela tensão máxima |
| 3 | Tensão máx. da faixa de operação ÷ Voc corrigido | Limite de módulos em série pela faixa de operação |
| 4 | Tensão de partida ÷ Vmax | Mínimo de módulos por MPPT (arredondado para cima) |
| 5 | Corrente máx. de entrada do MPPT ÷ Imax | Limite de strings em paralelo por corrente de operação |
| 6 | Corrente máx. de curto do MPPT ÷ Icc | Limite de strings em paralelo por corrente de curto |

## Resultados apresentados

- **Quantidade máxima de módulos de acordo com a potência total:** resultado do cálculo 1, arredondado para baixo.
- **Quantidade mínima de módulos por MPPT:** resultado do cálculo 4, arredondado para cima.
- **Quantidade máxima de módulos por MPPT:** menor valor entre os cálculos 2 e 3, arredondado para baixo. Para cada MPPT são listadas as entradas utilizáveis, definidas pelo menor valor entre os cálculos 5 e 6 (limitado ao número de entradas do MPPT).

## Validações

O botão de cálculo fica desabilitado enquanto alguma das regras abaixo não for atendida:

- O Voc deve ser maior que o Vmax
- O Icc deve ser maior que o Imax
- A tensão máxima de entrada deve ser maior que a tensão de partida

Além disso, todos os campos obrigatórios precisam estar preenchidos no momento do cálculo.

## Exportação e importação

- **Exportar Dados:** disponível após o cálculo, gera o arquivo `dimensionamento_string.json` com todos os valores preenchidos.
- **Importar Dados:** disponível no cabeçalho, carrega um arquivo exportado anteriormente e preenche os campos automaticamente.

Exemplo de estrutura do arquivo:

```json
{
  "voc": "45.5",
  "icc": "10.2",
  "vmax": "37.8",
  "imax": "9.5",
  "wmod": "360",
  "coefTemp": "-0.29",
  "deltaTemp": "-10",
  "pMaxCC": "8000",
  "vMaxEnt": "1000",
  "vMaxOp": "800",
  "vPartida": "200",
  "nMPPT": "2",
  "mpptIguais": true,
  "mppts": [
    { "iMaxMPPT": "12.5", "iCurtMPPT": "15", "nEntMPPT": "2" }
  ]
}
```

## Relatório em PDF

O botão **Gerar PDF** abre o relatório em uma nova aba, contendo os dados informados, os valores de conferência e os resultados. Nessa aba, use **Imprimir / Salvar como PDF** e escolha "Salvar como PDF" como destino.

> O navegador pode bloquear a abertura da nova aba. Nesse caso, permita pop-ups para o site.

## Tecnologias

- HTML5
- CSS3
- JavaScript (sem frameworks ou bibliotecas)

## Estrutura do projeto

```
.
├── index.html   # aplicação completa (HTML, CSS e JavaScript)
└── README.md
```

## Aviso

Esta ferramenta é um auxílio para o pré-dimensionamento. Os resultados devem ser conferidos com as folhas de dados dos fabricantes e com as normas aplicáveis antes da elaboração do projeto definitivo.
