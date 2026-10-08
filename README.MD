# 🛠️ CNC Caseira de MDF para Placas de Circuito Impresso

> Projeto desenvolvido durante a faculdade (2016–2018): uma fresadora CNC de 3 eixos construída de forma totalmente caseira, com estrutura em MDF, Arduino e firmware GRBL, para **confeccionar placas de circuito impresso (PCI) por fresamento** — sem percloreto, sem transferência térmica.

Este repositório reúne o registro de todo o processo — peças, montagem, eletrônica, firmware, calibração e o fluxo de software do desenho do circuito até a placa pronta — para que outras pessoas possam usar este trabalho como referência e não precisem partir do zero.

<p align="center">
  <img src="docs/image/v2/gif/usinando-placa.gif" alt="CNC fresando as trilhas de uma placa de circuito" width="480">
  <br><sub>A CNC fresando as trilhas de uma placa</sub>
</p>

---

## 📋 Sumário

- [Visão geral](#-visão-geral)
- [Materiais e peças](#-materiais-e-peças)
- [Montagem passo a passo](#-montagem-passo-a-passo)
- [Eletrônica](#-eletrônica)
- [Firmware GRBL](#-firmware-grbl)
- [Fluxo de software: do circuito à placa](#-fluxo-de-software-do-circuito-à-placa)
- [Calibração](#-calibração)
- [Operando a CNC](#-operando-a-cnc)
- [Resultados](#-resultados)
- [Dicas de desenho de PCI para fresamento](#-dicas-de-desenho-de-pci-para-fresamento)
- [Estrutura do repositório](#-estrutura-do-repositório)
- [Referências](#-referências)

---

## 🔎 Visão geral

| Item | Escolha neste projeto |
|------|-----------------------|
| Estrutura | Chapas de MDF (pintadas de preto na versão final) |
| Eixos | 3 (X, Y, Z) com motores de passo Nema 17 |
| Transmissão | Barra roscada (fuso) com porca embutida |
| Guias | Barras lisas de aço (X e Z) e corrediças telescópicas (Y, sob a mesa) |
| Controlador | Arduino Uno + CNC Shield V3 + drivers DRV8825 |
| Firmware | [GRBL](https://github.com/gnea/grbl) 0.9j (cópia em [`firmware/`](firmware/)) |
| Ferramenta | Micro retífica Black & Decker 180 W |
| Alimentação | Fonte ATX 350 W |
| Recursos | Autonivelamento por sonda (probe) e acionamento automático da retífica por relé |

📄 O **[Manual Prático da CNC (PDF)](docs/Manual%20CNC.pdf)** escrito na época complementa este README com capturas de tela de cada etapa. A versão anterior deste README, em texto, está preservada em **[GUIA.MD](GUIA.MD)**.

---

## 🧰 Materiais e peças

Antes da montagem, todas as peças foram cortadas e separadas: chapas de MDF (laterais trapezoidais, base, mesa e a placa do eixo Z), blocos de MDF para os mancais, barras lisas, barras roscadas, corrediças telescópicas, parafusos, porcas e chapas metálicas.

<table>
  <tr>
    <td><img src="docs/image/v2/20170712-WA0008.jpg" alt="Chapas de MDF cortadas e blocos dos mancais"></td>
    <td><img src="docs/image/v2/20170712-WA0013.jpg" alt="Peças de MDF, barras, corrediças telescópicas e ferramentas"></td>
  </tr>
  <tr>
    <td align="center"><sub>Chapas de MDF cortadas e blocos dos mancais</sub></td>
    <td align="center"><sub>Barras lisas, barras roscadas, corrediças telescópicas e parafusos</sub></td>
  </tr>
</table>

### Lista de componentes

**Mecânica**

| Peça | Uso |
|------|-----|
| Chapas de MDF | Laterais, base, mesa móvel, carro do eixo Z e blocos de mancal |
| Barras lisas de aço | Guias lineares dos eixos X e Z |
| Barras roscadas (fuso) | Transmissão do movimento nos três eixos |
| Corrediças telescópicas (de gaveta) | Guias do eixo Y, sob a mesa |
| Barra chata / chapa metálica com porcas | Suporte das porcas que transformam a rotação do fuso em translação |
| Chapas de alumínio | Suporte dos motores |
| Acoplamentos, parafusos, porcas e arruelas | Fixação geral |
| Abraçadeiras metálicas | Fixação da retífica no eixo Z |

**Eletrônica** (detalhes em [Eletrônica](#-eletrônica))

| # | Componente | Função |
|---|-----------|--------|
| 1 | **Motor de passo Nema 17 (4 kgf·cm)** × 3 | Movimento dos eixos. Verifique no datasheet a ordem dos fios antes de ligar ao shield. |
| 2 | **Arduino Uno** | Microcontrolador que roda o GRBL. |
| 3 | **CNC Shield V3** | Concentra as ligações: motores (via drivers), fins de curso, sonda, relé e jumpers de micropasso (¼, ⅛, 1/16, 1/32). |
| 4 | **Driver DRV8825** × 3 | Controla a corrente de cada fase dos motores (até 2 A por fase). |
| 5 | **Chave fim de curso** | Sinaliza que um eixo chegou ao limite. |
| 6 | **Micro retífica** | Ferramenta de corte; a broca define o trabalho (trilhas, furos, corte). |
| 7 | **Módulo relé 5 V** | Liga e desliga a retífica automaticamente. |
| 8 | **Ponteiras de prova (jacaré)** | Sonda para o autonivelamento. |
| 9 | **Fonte ATX 350 W** | Alimentação dos motores (12 V). |

**Ferramentas de corte**

| Broca | Uso |
|-------|-----|
| **#1 — Broca em V 30°** | Fresamento (isolação) das trilhas |
| **#2 — Broca 0,7 mm** | Furação dos terminais dos componentes |

<table>
  <tr>
    <td width="60%"><img src="docs/image/v2/brocas.jpeg" alt="Jogos de brocas para PCB"></td>
    <td width="40%"><img src="docs/image/v2/gif/broca-pcb-v.gif" alt="Detalhe da broca em V"></td>
  </tr>
  <tr>
    <td align="center"><sub>Jogos de brocas de metal duro para PCI</sub></td>
    <td align="center"><sub>Detalhe da ponta da broca em V</sub></td>
  </tr>
</table>

> ⚠️ As brocas de PCI são muito frágeis. Antes de rodar qualquer teste, confira velocidade e distância de cada movimento — um descuido custa uma broca.

---

## 🔩 Montagem passo a passo

A montagem foi feita em julho de 2017. As fotos abaixo seguem a ordem real da construção.

<table>
  <tr>
    <td width="50%"><img src="docs/image/v2/20170715_214842.jpg" alt="Base com as corrediças telescópicas do eixo Y e as laterais"></td>
    <td width="50%"><img src="docs/image/v2/20170716_165255.jpg" alt="Laterais com as barras lisas e mesa sobre as corrediças"></td>
  </tr>
  <tr>
    <td align="center"><sub><b>1.</b> Base com as corrediças telescópicas do eixo Y e as laterais</sub></td>
    <td align="center"><sub><b>2.</b> Barras lisas do eixo X com os mancais de MDF; mesa sobre as corrediças</sub></td>
  </tr>
  <tr>
    <td><img src="docs/image/v2/20170718_172835.jpg" alt="Chapa metálica com porcas soldadas"></td>
    <td><img src="docs/image/v2/20170719-WA0013.jpg" alt="Motores de passo nos suportes de alumínio"></td>
  </tr>
  <tr>
    <td align="center"><sub><b>3.</b> Chapa (barra chata) com porcas soldadas, que se move ao longo da barra roscada</sub></td>
    <td align="center"><sub><b>4.</b> Motores Nema 17 montados nos suportes de alumínio</sub></td>
  </tr>
  <tr>
    <td><img src="docs/image/v2/20170719_015503.jpg" alt="Carro do eixo Z montado"></td>
    <td><img src="docs/image/v2/20170719_205648.jpg" alt="Vista frontal com barras roscadas e porca do eixo X"></td>
  </tr>
  <tr>
    <td align="center"><sub><b>5.</b> Carro do eixo Z montado sobre as barras do eixo X</sub></td>
    <td align="center"><sub><b>6.</b> Barra roscada e porca do eixo X; motores posicionados</sub></td>
  </tr>
  <tr>
    <td><img src="docs/image/v2/20170720-WA0013.jpg" alt="Estrutura com os três motores instalados"></td>
    <td><img src="docs/image/v2/20170720-WA0016.jpg" alt="Vista frontal com os três eixos montados"></td>
  </tr>
  <tr>
    <td align="center"><sub><b>7.</b> Os três motores instalados</sub></td>
    <td align="center"><sub><b>8.</b> Estrutura mecânica completa, ainda em MDF cru</sub></td>
  </tr>
  <tr>
    <td><img src="docs/image/v2/20180131-WA0022.jpg" alt="CNC pintada de preto"></td>
    <td><img src="docs/image/v2/20180131-WA0023.jpg" alt="CNC pintada, vista lateral"></td>
  </tr>
  <tr>
    <td align="center"><sub><b>9.</b> Acabamento: estrutura pintada de preto (jan/2018)</sub></td>
    <td align="center"><sub><b>10.</b> Vista lateral, já ligada ao notebook</sub></td>
  </tr>
</table>

### Máquina finalizada e operacional (mar/2018)

Com a retífica fixada no eixo Z por abraçadeiras, a mesa de sacrifício instalada e a eletrônica presa na lateral:

<table>
  <tr>
    <td width="33%"><img src="docs/image/v2/20180307_181949.jpg" alt="CNC finalizada com a retífica"></td>
    <td width="33%"><img src="docs/image/v2/20180307_182018.jpg" alt="CNC finalizada, vista frontal"></td>
    <td width="33%"><img src="docs/image/v2/20180307_182109.jpg" alt="CNC finalizada, lateral com a eletrônica"></td>
  </tr>
</table>

---

## ⚡ Eletrônica

Toda a eletrônica fica fixada na lateral da máquina: Arduino Uno com CNC Shield V3 e drivers DRV8825, módulo relé da retífica e um cooler para os drivers.

<table>
  <tr>
    <td width="50%"><img src="docs/image/v2/circuito-eletronico.jpeg" alt="Arduino, CNC Shield e relé na lateral da CNC"></td>
    <td width="50%"><img src="docs/image/v2/circuito-eletronico2.jpeg" alt="Eletrônica vista de outro ângulo"></td>
  </tr>
</table>

### Esquema de ligação

<p align="center">
  <img src="docs/image/v2/conexao-eletrica.jpeg" alt="Esquema de ligação da CNC Shield com motores e fins de curso" width="640">
  <br><sub>Diagrama de referência para ligação da CNC Shield V3 (motores, fins de curso e filtro anti-ruído)</sub>
</p>

### Ligações e ajustes

- **Motores:** a ordem dos fios no conector do shield é `A A' B' B` (ou `B B' A' A`). Confira os pares de bobina no datasheet do motor.
- **Micropasso:** definido pelos jumpers `M0/M1/M2` sob cada driver (¼, ⅛, 1/16 ou 1/32). Esse valor entra no cálculo de step/mm.
- **Corrente do DRV8825:** ajustada no potenciômetro do driver, medindo a tensão de referência entre o GND e o potenciômetro:

  ```
  Vref = Imax × 8 × Rs
  ```

  `Imax` é a corrente máxima do motor e `Rs` o resistor *sense* (normalmente `R100` = 0,1 Ω). Para medir, a fonte precisa estar alimentando o shield e o USB alimentando o Arduino.
- **Fins de curso:** ligados em normalmente aberto nas entradas `X+/X-`, `Y+/Y-`, `Z+/Z-` do shield.
- **Relé da retífica:** alimentado com 5 V, com o pino `IN` ligado ao `SpnEn` do shield. Sem ele, a retífica é ligada e desligada manualmente.
- **Sonda (autonivelamento):** uma ponteira no GND e a outra no pino `SCL` (A5 — entrada *probe* do GRBL). Na operação, uma garra vai na face de cobre da placa e a outra na broca.

---

## 💾 Firmware GRBL

O [GRBL](https://github.com/gnea/grbl) é um firmware de código aberto que roda no ATmega328P (Arduino Uno/Nano), interpreta G-code e gera os pulsos de passo com controle de aceleração. A versão usada no projeto (**0.9j**) está em [`firmware/`](firmware/).

### Ajustes no `config.h`

Antes de gravar, edite [`firmware/grbl/config.h`](firmware/grbl/config.h):

| Linha | Alteração | Motivo |
|-------|-----------|--------|
| [173](firmware/grbl/config.h#L173) | **Descomentar** `#define INVERT_SPINDLE_ENABLE_PIN` | Inverte a lógica do pino *spindle enable* (`SpnEn`), que aciona o módulo relé da retífica. |
| [247](firmware/grbl/config.h#L247) | **Comentar** `#define VARIABLE_SPINDLE` | Com essa opção ativa, o GRBL 0.9 troca os pinos D11 e D12 (limite Z ↔ spindle enable), o que não bate com a CNC Shield V3. A retífica é só liga/desliga, então o PWM não é necessário. |

> ℹ️ A cópia em `firmware/` está com os valores padrão; as duas alterações acima precisam ser aplicadas antes da gravação.

### Gravando no Arduino

1. Instale a [Arduino IDE](https://www.arduino.cc/en/software).
2. Compacte a pasta `firmware/grbl` em `.zip` e adicione como biblioteca: `Sketch > Incluir Biblioteca > Adicionar biblioteca .ZIP`.
3. Abra `Arquivo > Exemplos > grbl > grblUpload`.
4. Selecione a placa (Uno ou Nano) e a porta serial, e faça o upload.

Depois disso o Arduino é reconhecido pelos programas de controle (bCNC, Universal G-Code Sender etc.).

---

## 🖥️ Fluxo de software: do circuito à placa

```
 KiCad ──(Gerber + Excellon)──▶ FlatCAM ──(G-code .nc)──▶ bCNC ──(serial)──▶ GRBL / CNC
 desenho do circuito            caminhos de ferramenta     autonivelamento e envio
```

Todos os programas abaixo são **gratuitos e de código aberto**.

### 1. KiCad — desenho do circuito

<a href="https://www.kicad.org/"><b>kicad.org</b></a> · <a href="https://gitlab.com/kicad/code/kicad">código-fonte (GitLab)</a>

O KiCad é uma suíte EDA (Electronic Design Automation) completa e multiplataforma (Windows, macOS, Linux), mantida pela comunidade com apoio da Linux Foundation e do CERN. Com ele você desenha o **esquemático** (Eeschema), associa os *footprints* e faz o **layout da placa** (Pcbnew), com verificação de regras elétricas (ERC) e de projeto (DRC) e visualização 3D.

Para a CNC, o resultado que importa são os arquivos de fabricação:

- **Gerber** da camada de cobre usada (normalmente `B.Cu` para placas de face simples) e do contorno (`Edge.Cuts`) — em `Arquivo > Plotar`;
- **Excellon** (arquivo de furação `.drl`) — no botão *Gerar arquivos de furação* da mesma janela.

> Na época o manual usava o **Proteus** (`Output > Generate Gerber/Excellon Files`, unidades em mm, formato RS274X e reflexão *Mirror*). O fluxo seguinte é o mesmo para qualquer programa que exporte Gerber.

### 2. FlatCAM — Gerber para G-code

<a href="http://flatcam.org/"><b>flatcam.org</b></a> · <a href="https://bitbucket.org/jpcgt/flatcam">código-fonte (Bitbucket)</a>

O FlatCAM é um CAM (*Computer-Aided Manufacturing*) escrito em Python, criado por Juan Pablo Caram especificamente para **prototipagem de PCI por fresamento**. Ele abre arquivos Gerber, Excellon e G-code, calcula a **isolação** das trilhas (o caminho que a broca percorre em volta do cobre para separá-lo do resto), gera os ciclos de furação e o recorte da placa, e exporta tudo como G-code.

Configuração usada neste projeto:

1. `File > Open Gerber…` → camada de cobre (`Bottom Copper.GBR`).
2. `File > Open Excellon…` → furação (`Drill.DRL`).
3. Selecione o **Gerber** → aba *Selected* → *Isolation Routing*:
   `Tool dia: 0.25` · `Width (# passes): 1` · `Pass overlap: 0.15` → **Generate Geometry**.
4. Selecione o **Excellon** → *Create CNC Job*:
   `Cut Z: -2.2` · `Travel Z: 2.0` · `Feed rate: 90` → **Generate**.
5. Selecione a geometria `…GBR_iso` → *Create CNC Job*:
   `Cut Z: -0.125` · `Travel Z: 2` · `Feed rate: 38` · `Tool dia: 0.25` → **Generate**.
6. Para cada objeto `_cnc`, clique em **Export G-Code**:
   - `Drill.DRL_cnc` → `furos-placa.nc`
   - `Bottom Copper.GBR_iso_cnc` → `trilhas-placa.nc`

### 3. bCNC — controle da máquina e autonivelamento

<a href="https://github.com/vlachoudis/bCNC"><b>github.com/vlachoudis/bCNC</b></a>

O bCNC é um *sender* de G-code para GRBL, escrito em Python por Vasilis Vlachoudis, que roda em Windows, Linux, macOS e até no Raspberry Pi. Além de enviar o G-code pela serial, ele oferece:

- **Controle manual (jog)** dos eixos, zeramento (`XYZ=0`), *unlock* e *homing*;
- **Autonivelamento (Autolevel):** sonda uma grade de pontos na superfície da placa e corrige o G-code em tempo real conforme as irregularidades — essencial em PCI, onde a isolação tem décimos de milímetro de profundidade;
- **Editor e visualizador** de G-code, com ferramentas CAM simples;
- **Atalhos de teclado e botões configuráveis** para as ações do dia a dia.

Instalação: `pip install bCNC` e depois `python -m bCNC`.

> Para configurar os parâmetros `$` do GRBL também dá para usar o [Universal G-Code Sender](https://github.com/winder/Universal-G-Code-Sender), que tem uma tela de configuração dedicada.

---

## 📐 Calibração

### Step/mm

Os parâmetros `$100`, `$101` e `$102` do GRBL definem quantos passos o motor dá para mover 1 mm em X, Y e Z:

```
Fuso / barra roscada:   K = Mr × (1/Sm) / Br
Correia e polia:        K = Mr × (1/Sm) / Pc
```

| Variável | Significado |
|----------|-------------|
| `K` | Passos por milímetro (step/mm) |
| `Mr` | Passos por volta do motor (normalmente **200**) |
| `Sm` | Micropasso configurado no shield (¼, ⅛, 1/16, 1/32) |
| `Br` | Avanço da barra roscada por volta, em mm (medir com paquímetro) |
| `Pc` | Passo da correia × dentes da polia (ex.: GT2 2 mm × 20 dentes = 40 mm) |

Configuração do GRBL usada nesta CNC (trecho):

```
$100=570.000   (x, step/mm)        $110=420.000   (x max rate, mm/min)
$101=570.000   (y, step/mm)        $111=420.000   (y max rate, mm/min)
$102=570.000   (z, step/mm)        $112=420.000   (z max rate, mm/min)
$120=10.000    (x accel, mm/s²)    $130=240.000   (x max travel, mm)
$121=10.000    (y accel, mm/s²)    $131=240.000   (y max travel, mm)
$122=10.000    (z accel, mm/s²)    $132=100.000   (z max travel, mm)
```

### Teste prático

1. Prenda uma caneta ou lápis no eixo Z.
2. Mande a máquina percorrer **100 mm**.
3. Meça o deslocamento real com um paquímetro.
4. Ajuste o step/mm proporcionalmente (`novo = atual × comandado / medido`) e repita até coincidir.

---

## ▶️ Operando a CNC

### Antes de ligar a fonte

- Verifique se **nenhum fio** ligado ao shield está em curto.
- Confira se **nenhum eixo está no limite** — comandos residuais podem forçar os motores contra o batente.
- Espere terminar qualquer comando pendente e **zere as posições** (X, Y, Z).

### Autonivelamento no bCNC

<p align="center">
  <img src="docs/image/v2/gif/nivelamento-automatico.gif" alt="Sondagem de nível sobre a placa de fenolite" width="420">
  <br><sub>Sondagem de nível: a broca toca a placa em vários pontos da grade</sub>
</p>

1. Fixe a placa de fenolite na mesa e coloque a **broca #1 (V 30°)**.
2. Conecte as garras: uma no cobre da placa e outra na broca.
3. No bCNC, selecione a porta serial e clique em **Abrir**.
4. Aba **Controle** → **Destravar**. Leve a broca até um canto da placa e clique em **XYZ=0**.
5. Aba **Sonda → Sondagem de nível**: preencha Min/Max de X, Y e Z conforme o tamanho da placa e o número de pontos `N`. Clique em **Scanear**.
6. Ao terminar, clique em **Zero** e salve o mapa como `nome.probe`.
7. Abra o `.nc` e em seguida o `.probe` correspondente — o bCNC aplica a correção ao G-code.

### Execução

1. Rode `trilhas-placa.nc` com a broca em V.
2. Troque para a broca de 0,7 mm, refaça o zero do Z e rode `furos-placa.nc`.

<p align="center">
  <img src="docs/image/v2/gif/perfurando-placa.gif" alt="CNC furando a placa" width="420">
  <br><sub>Furação dos terminais com a broca de 0,7 mm</sub>
</p>

Arquivos de cada projeto:

```
projeto/
├── trilhas-placa.nc    # G-code das trilhas (isolação)
├── furos-placa.nc      # G-code da furação
└── nome.probe          # mapa do autonivelamento
```

---

## ✅ Resultados

<table>
  <tr>
    <td width="55%"><img src="docs/image/v2/placa-usinada-cnc.jpeg" alt="Placa recém-usinada na CNC"></td>
    <td width="45%"><img src="docs/image/v2/placa-pronta.jpeg" alt="Placa finalizada"></td>
  </tr>
  <tr>
    <td align="center"><sub>Placa recém-saída da CNC</sub></td>
    <td align="center"><sub>Placa limpa e pronta para receber os componentes</sub></td>
  </tr>
</table>

---

## 💡 Dicas de desenho de PCI para fresamento

- **Malha de GND (plano de terra):** reduz a área que a broca precisa remover e o tempo de usinagem. Valores usados no Proteus: Net `GND=POWER`, camada `Bottom Copper`, Boundary/Relief `T30`, tipo `Solid`, Clearance `25th`, com *Relieve Pins*, *Suppress Islands*, *Lock to Board Edge* e *Exclude Board Edge* habilitados.
- **Pads through-hole:** em transistores e componentes com terminais próximos, use pads de até **90-40** (diâmetro 90 mil, furo 40 mil) para sobrar cobre em volta do furo.
- **Trilhas e espaçamentos largos:** a broca em V corta mais largo quanto mais fundo vai; evite trilhas e espaçamentos muito finos.

---

## 📁 Estrutura do repositório

```
grbl-cnc/
├── README.MD              # este documento
├── GUIA.MD                # versão anterior do guia (texto)
├── docs/
│   ├── Manual CNC.pdf     # manual prático original, com capturas de tela
│   └── image/
│       └── v2/            # fotos e vídeos da construção (versão documentada)
│           └── gif/       # trechos dos vídeos convertidos para GIF
└── firmware/              # GRBL 0.9j (Arduino Uno / ATmega328P)
```

---

## 🔗 Referências

| Projeto | Links |
|---------|-------|
| GRBL | [github.com/gnea/grbl](https://github.com/gnea/grbl) · [wiki](https://github.com/gnea/grbl/wiki) · [repositório antigo (0.9)](https://github.com/grbl/grbl) |
| KiCad | [kicad.org](https://www.kicad.org/) · [gitlab.com/kicad](https://gitlab.com/kicad/code/kicad) |
| FlatCAM | [flatcam.org](http://flatcam.org/) · [bitbucket.org/jpcgt/flatcam](https://bitbucket.org/jpcgt/flatcam) |
| bCNC | [github.com/vlachoudis/bCNC](https://github.com/vlachoudis/bCNC) |
| Arduino IDE | [arduino.cc/en/software](https://www.arduino.cc/en/software) |
| Universal G-Code Sender | [github.com/winder/Universal-G-Code-Sender](https://github.com/winder/Universal-G-Code-Sender) |

---

<sub>Firmware GRBL distribuído sob a licença GPLv3 (ver <a href="firmware/COPYING">firmware/COPYING</a>).</sub>
