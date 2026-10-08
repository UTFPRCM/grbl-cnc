# 🛠️ Manual Prático de CNC

> Guia de utilização da CNC no dia a dia — desde a montagem do hardware até a execução de projetos.

---

## 📋 Sumário

- [Hardware](#hardware)
- [Software](#software)
- [Calibrando a CNC](#calibrando-a-cnc)
- [Gerando o G-Code](#gerando-o-g-code-nc)
- [Ligando a CNC](#ligando-a-cnc)
- [Executando seu Projeto sem Falhas](#executando-seu-projeto-sem-falhas)
- [Dicas de Desenho de Circuito Eletrônico](#dicas-de-desenho-de-circuito-eletrônico)

---

## Hardware

Lista de componentes necessários para o funcionamento da CNC:

| # | Componente | Descrição |
|---|-----------|-----------|
| 1 | **Motor de Passo (Nema 17 - 4kgf)** | Responsável pelo movimento dos eixos. Verificar o datasheet para ordem e posição dos fios antes de conectar ao Shield. |
| 2 | **CNC Shield 3.0** | Centraliza e organiza todas as conexões da CNC. Controla motores (via drivers), fins de curso, autonivelamento, acionamento automático e micropassos (¼, ⅛, 1/16, 1/32). Ordem de ligação dos motores: `AA'B'B` ou `BB'A'A`. |
| 3 | **Driver DRV8825** | Controla a corrente em cada fase dos motores de passo (até 2A por fase). Para configurar, utilizar a equação: `Vref = Imax × 8 × Rs`. Medir Vref com multímetro no GND e no potenciômetro do driver. |
| 4 | **Chave Fim de Curso** | Chave mecânica que sinaliza quando um eixo atingiu seu limite. Conectar em normalmente aberto nas entradas `X+`, `X-`, `Y+`, `Y-`, `Z+`, `Z-` do Shield. |
| 5 | **Retífica** | Ferramenta responsável pelo trabalho (corte, furo, obstrução). O tipo de ponta/broca define o trabalho a realizar. |
| 6 | **Módulo Relé** | Permite o acionamento automático da retífica. Alimentar com 5V e conectar o pino `In` ao pino `SpnEn` do Shield. Sem ele, a retífica deve ser ligada/desligada manualmente. |
| 7 | **Ponteira de Prova** | Usada para o autonivelamento. Conectar uma ponteira no terra/ground e a outra no pino `SCL` do Shield. |

---

## Software

Serão utilizados **3 softwares** e gravado **1 firmware** no microcontrolador.

### Firmware: GRBL

1. Baixe a última versão do GRBL no [GitHub](https://github.com/grbl/grbl).
2. Descompacte e localize o arquivo `config.h` dentro da pasta `grbl`.
3. Edite o arquivo `config.h`:
   - **Comentar** (adicionar `//`) a linha **247**
   - **Descomentar** (remover `//`) a linha **173**
4. Salve o arquivo, compacte a pasta `grbl` e adicione como biblioteca no Arduino IDE.
5. Abra o Arduino IDE, conecte o Arduino (Uno ou Nano) via serial e faça o upload do GRBL.

### Softwares Utilizados

| Software | Função |
|----------|--------|
| **Arduino IDE** | Gravação do firmware GRBL no microcontrolador. |
| **FlatCAM** | Geração do G-Code a partir dos arquivos Gerber (CAD eletrônico). |
| **bCNC** | Interface de controle entre o computador e a CNC. Permite controlar e acompanhar todo o processo. |

---

## Calibrando a CNC

### Cálculo Step/mm

Configurar os parâmetros `$100`, `$101` e `$102` no GRBL via Universal G-Code Sender, utilizando a fórmula:

**Para fuso/barra roscada:**
```
K_xyz = (Mr × (1/Sm)) / Br
```

**Para correia/polia:**
```
K_xyz = (Mr × (1/Sm)) / Pc
```

| Variável | Descrição |
|----------|-----------|
| `K_xyz` | Fator de calibração dos eixos X, Y e Z (step/mm) |
| `Sm` | Configuração de micropassos da CNC Shield (¼, ⅛, 1/16, 1/32) |
| `Mr` | Passos por revolução do motor (geralmente **200**, ver datasheet) |
| `Br` | Deslocamento em mm por revolução da barra roscada/fuso |
| `Pc` | Pitch da correia × número de dentes da polia (ex: correia GT2 = 2mm, polia GT2 = 20 dentes → Pc = 40mm) |

### Teste Prático

Após atualizar as configurações do GRBL:

1. Fixe uma **caneta ou lápis** no eixo Z.
2. Ordene à CNC percorrer **10 cm**.
3. Meça o deslocamento real com um **paquímetro**.
4. Ajuste o fator de calibração até que o valor comandado seja igual ao valor medido.

---

## Gerando o G-Code (.nc)

Fluxo para geração do G-Code de um circuito eletrônico (usando Proteus + FlatCAM):

1. No **Proteus**, com o esquemático e PCB prontos, exporte os arquivos Gerber (`Output > Generate Gerber/Excellon Files`).
2. Confirme as configurações: rotação X Horizontal, reflexão Mirror, unidades Metric (mm), formato RS274X.
3. Descompacte o arquivo CADCAM gerado e abra o **FlatCAM**.
4. `File > Open Gerber…` → abra o arquivo `Bottom Copper.GRB`.
5. `File > Open Excellon…` → abra o arquivo `Drill.DRL`.
6. Na aba **Project**, clique em `Bottom Copper.GBR` → aba **Selected**:
   - Tool dia: `0.25` | Width: `1` | Pass overlap: `0.15`
   - Clique em **Generate Geometry**.
7. Na aba **Project**, clique em `Drill.DRL` → aba **Selected**:
   - Cut Z: `-2.2` | Travel Z: `2.0` | Feed rate: `90.0`
   - Clique em **Generate**.
8. Clique em `Bottom Copper.GBR_iso` → aba **Selected**:
   - Cut Z: `-0.125` | Travel Z: `2` | Feed Rate: `38` | Tool dia: `0.25`
   - Clique em **Generate**.
9. Para cada arquivo `_cnc` gerado, vá em **Selected > Export G-Code** e salve:
   - `Drill.DRL_cnc` → salvar como `furos-placa.nc`
   - `Bottom Copper.GBR_iso_cnc` → salvar como `trilhas-placa.nc`

---

## Ligando a CNC

> ⚠️ Antes de ligar a fonte, sempre verifique:

- **Curtos:** Nenhum fio conectado ao Shield deve estar em curto-circuito.
- **Limites dos eixos:** Certifique-se de que nenhum eixo está na posição limite ao iniciar.
- **Comandos pendentes:** Aguarde o fim de qualquer comando em execução e limpe os dados de posicionamento (X, Y, Z).

### Brocas

Para fabricação de circuitos eletrônicos, são necessários ao menos dois tipos de brocas:

| Broca | Uso |
|-------|-----|
| **Broca em V 30°** (broca #1) | Fresamento das trilhas do circuito |
| **Broca 0.7mm** (broca #2) | Furação para os componentes |

> ⚠️ **Cuidado:** Ambas as brocas quebram com facilidade. Verifique sempre velocidade e distância de movimentação antes de executar.

---

## Executando seu Projeto sem Falhas

### Autonivelamento

1. Fixe a placa de fenolite na base da CNC e coloque a **broca #1** na retífica.
2. Conecte as **ponteiras de prova**: uma na superfície de cobre da placa e a outra na broca #1.
3. Abra o **bCNC**, conecte o cabo serial e clique em **Abrir**.
4. Na aba **Controle**, clique em **Destravar**. Reposicione a retífica próxima a uma extremidade da placa e clique em **XYZ=0**.
5. Na aba **Sonda > Sondagem de Nível**, preencha os valores Min/Max de X, Y e Z conforme o tamanho da placa e defina a quantidade de pontos (N) por linha/coluna. Clique em **Scanear**.
6. Após o escaneamento, clique em **Zero** e salve o arquivo como `nome.probe`.
7. Para cada projeto, repita este processo. Abra o arquivo `.nc` e em seguida o arquivo `.probe` correspondente.

---

## Dicas de Desenho de Circuito Eletrônico

### Malha de Ground
- Configurar no Proteus com as seguintes propriedades:
  - Net: `GND=POWER`
  - Layer: `Bottom Copper`
  - Boundary e Relief: `T30`
  - Type: `Solid` | Step: `25th`
  - Clearance: `25th`
  - Habilitar: Relieve Pins, Suppress Islands, Lock to Board Edge, Exclude Board Edge

### Through Hole Pad
- Para transistores e componentes com pouco espaço entre terminais, é possível e recomendável utilizar pads de até o tamanho **90-40**.

---

## 📁 Arquivos Necessários por Projeto

```
projeto/
├── trilhas-placa.nc    # G-Code das trilhas
├── furos-placa.nc      # G-Code dos furos
└── nome.probe          # Dados do autonivelamento
```

---

*Manual baseado no documento "Como utilizar a CNC no dia a dia".*