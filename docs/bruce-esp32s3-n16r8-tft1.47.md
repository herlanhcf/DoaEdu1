# Montando um "Bruce" com firmware Bruce — ESP32-S3 N16R8 + TFT 1,47"

Lista de materiais (BOM) e guia de componentes para montar um dispositivo estilo
multi-ferramenta ("Flipper-like") rodando o **firmware Bruce** (projeto
[pr3y/Bruce](https://github.com/pr3y/Bruce) / [BruceDevices/firmware](https://github.com/BruceDevices/firmware)),
usando um **ESP32-S3 N16R8** e um **display TFT de 1,47" (ST7789, 172×320)**.

> **Contexto de uso:** Bruce é um firmware aberto de pesquisa de segurança / red team.
> Monte e use este hardware apenas em equipamentos e redes próprios ou com autorização
> explícita, e respeitando a legislação de RF do seu país (as bandas Sub-GHz 315/433/868/915 MHz
> têm regras diferentes por região).

---

## 1. Visão geral: o que é "obrigatório" x "operacional"

| Categoria | O que é | Precisa para bootar o Bruce? |
|-----------|---------|------------------------------|
| **Núcleo** | ESP32-S3 N16R8 + TFT 1,47" + alimentação + navegação | **Sim (obrigatório)** |
| **Rádio nativo** | Wi-Fi 2.4 GHz + Bluetooth/BLE | Já vem **dentro** do ESP32-S3 (não precisa módulo) |
| **Módulos operacionais** | CC1101, nRF24L01+, PN532, IR, GPS, SD | Não — são os que dão as funções de "multi-tool" |
| **Passivos** | Capacitores, resistores, transistor | Alguns obrigatórios, outros por módulo |

Ponto importante: **Wi-Fi e Bluetooth/BLE são internos do ESP32-S3**. As funções de
Wi-Fi (deauth, evil portal, sniffer, etc.) e BLE (spam, etc.) **não exigem módulo extra**.
Os módulos abaixo adicionam Sub-GHz (RF), 2.4 GHz bruto, NFC/RFID, Infravermelho, GPS e armazenamento.

---

## 2. Componentes OBRIGATÓRIOS (núcleo mínimo)

### 2.1 Placa / MCU
- **1× ESP32-S3 N16R8** — 16 MB de Flash, 8 MB de PSRAM.
  - A PSRAM (o "R8") é praticamente obrigatória para o Bruce rodar confortável
    (a UI e alguns módulos usam bastante RAM).
  - Pode ser um **DevKit ESP32-S3-DevKitC / N16R8** (mais fácil: já tem regulador 3.3V,
    USB, botões BOOT/RESET e capacitores de desacoplamento) ou o **módulo cru ESP32-S3-WROOM-1 N16R8**
    (aí você precisa adicionar os passivos da seção 4.4).

### 2.2 Display
- **1× TFT IPS 1,47"** — controlador **ST7789**, resolução **172×320**, interface **SPI**.
  - Normalmente **sem touch** → por isso os botões de navegação são obrigatórios (seção 2.4).
  - Sinais usados: `VCC (3.3V)`, `GND`, `SCLK`, `MOSI/SDA`, `CS`, `DC`, `RST`, `BLK` (backlight).

### 2.3 Alimentação
Mínimo para bancada: alimentar por **USB-C** já funciona. Para versão portátil:
- **1× Bateria Li-ion/LiPo 3,7V** (ex.: 800–1200 mAh).
- **1× Módulo carregador TP4056** (de preferência a versão com proteção **DW01 + FS8205**).
- **1× Conversor Boost DC-DC** (MT3608 ou similar) ajustado para **5V**, OU um regulador
  3.3V estável se for alimentar a linha 3.3V diretamente. Muitos módulos (nRF24 PA/LNA, GPS,
  CC1101) ficam mais estáveis alimentados a partir de um 5V limpo → 3.3V do regulador do ESP32.
- **1× Interruptor** liga/desliga (deslizante).

### 2.4 Navegação (obrigatório sem touch)
- **5× Push-buttons táteis** (UP, DOWN, LEFT, RIGHT, SELECT) — o mínimo funcional são 3
  (CIMA/BAIXO/OK), mas 5 dá navegação completa.
- Alternativa: um **joystick de 5 direções** (5-way) ou encoder rotativo com botão.

---

## 3. Módulos OPERACIONAIS (o que faz dele um "Bruce" completo)

Todos abaixo são opcionais individualmente — adicione conforme as funções que você quer.

| # | Módulo | Função no Bruce | Barramento | Frequência |
|---|--------|-----------------|-----------|------------|
| 1 | **CC1101** | Sub-GHz (captura/replay RF, jammer, etc.) | SPI | 315 / 433 / 868 / 915 MHz |
| 2 | **nRF24L01+** (idealmente **PA/LNA**) | 2.4 GHz: jammer, spectrum, Mousejack | SPI | 2.4 GHz |
| 3 | **PN532** | NFC/RFID (ler/emular tags 13.56 MHz) | I²C (ou SPI) | 13.56 MHz |
| 4 | **IR TX** (LED IR ou módulo KY-005) | Transmitir infravermelho (TV-B-Gone etc.) | GPIO | 38 kHz |
| 5 | **IR RX** (TSOP38238 / VS1838B) | Receber/gravar códigos IR | GPIO | 38 kHz |
| 6 | **GPS NEO-6M / NEO-8M** | Wardriving, geotag | UART | — |
| 7 | **Leitor microSD** | Armazenar scripts, dumps, capturas | SPI | — |

### Antenas (não esqueça)
- **CC1101:** antena mola/helicoidal ou fio ¼-onda para a banda escolhida (ex.: ~17,3 cm p/ 433 MHz).
- **nRF24 PA/LNA:** antena **SMA 2.4 GHz** (a versão PA/LNA precisa de antena para render).
- **GPS NEO-6M:** antena cerâmica ativa (geralmente acompanha o módulo).
- **ESP32-S3:** antena de PCB já integrada no WROOM-1 (ou conector u.FL na versão -U).

---

## 4. Passivos: CAPACITORES, RESISTORES e TRANSISTOR

Esta é a parte que costuma faltar nos tutoriais. Divididos por finalidade.

### 4.1 Capacitores (obrigatórios / recomendados)

| Qtd | Valor | Onde / para quê |
|-----|-------|-----------------|
| 1–2 | **10 µF** (eletrolítico ou cerâmico) | Filtragem/reservatório na linha 5V e/ou 3.3V (perto do boost e perto do ESP32) |
| 3–5 | **100 nF (0,1 µF) cerâmico** | Desacoplamento — **1 por módulo** junto do VCC/GND (ESP32, CC1101, PN532, nRF24) |
| **1** | **10 µF (até 100 µF) no nRF24L01+** | **Praticamente obrigatório**: soldar direto entre VCC e GND do nRF24. Sem ele o rádio reinicia/trava, principalmente na versão PA/LNA |
| 1 | **1 µF** (opcional) | No pino EN/RESET do ESP32 se usar módulo cru (auto-reset) |

> Regra prática: **1× 100 nF em cada módulo** + **1× 10 µF por trilho de alimentação** +
> **o cap dedicado do nRF24**.

### 4.2 Resistores

| Qtd | Valor | Onde / para quê |
|-----|-------|-----------------|
| 3–5 | **10 kΩ** | Pull-up dos botões de navegação (1 por botão) — *ou* dispensar usando `INPUT_PULLUP` interno do ESP32 |
| 2–3 | **10 kΩ** | Pull-ups do leitor microSD (linhas Dat1, Dat2 e CS) para estabilidade |
| 2 | **4,7 kΩ** | Pull-ups do barramento I²C do PN532 (SDA/SCL) — **em geral já vêm no módulo breakout** |
| 1 | **10 kΩ** | Pull-up do pino EN do ESP32 (apenas se módulo cru, não no DevKit) |
| 1 | **~1 kΩ** | Resistor de base do transistor do IR TX (só se montar LED IR discreto, não com KY-005) |
| 1 | **~4,7 Ω a 100 Ω** | Resistor limitador do LED IR (valor depende da tensão/driver; só p/ LED discreto) |

### 4.3 Transistor (para o IR de alta potência)
- **1× transistor NPN 2N2222** (nome completo; "222" é só o apelido). Equivalentes:
  **2N2222A / PN2222A / P2N2222A**, ou **S8050 / 2N3904**, ou MOSFET **2N7000**.
  Serve para chavear o **LED IR** com mais corrente do que o GPIO entrega sozinho → alcance muito maior.
- Ligação:
  ```
  GPIO (IR TX) ──[ R base 330Ω–1kΩ ]──► BASE
  EMISSOR ─────────────────────────────► GND
  3.3V/5V ──► LED IR (anodo) ; LED IR (catodo) ──[ R limitador ~33–100Ω ]──► COLETOR
  ```
- ⚠️ **Pinout varia por fabricante:** PN2222A costuma ser **E–B–C**; o P2N2222A/metálico costuma
  ser **C–B–E**. Confira o datasheet do SEU modelo antes de soldar.
- **Se usar o módulo KY-005**, ele **já traz o transistor + resistores** — nesse caso você
  NÃO precisa do 2N2222 nem dos resistores do IR TX; liga direto no GPIO.

### 4.4 Passivos extras SÓ se usar o módulo cru ESP32-S3-WROOM-1 (sem DevKit)
- **10 kΩ** no EN (pull-up) + **1 µF** do EN para GND.
- **10 kΩ** no GPIO0 (pull-up) + botão BOOT para GND.
- **10 µF + 100 nF** no 3V3 do módulo.
- Regulador 3.3V (ex.: **AMS1117-3.3** ou, melhor para bateria, um **TLV1117/ME6211/HT7333**) + seus caps de entrada/saída.
- Conversor USB-serial (CH340/CP2102) se quiser gravar/depurar sem USB nativo — o S3 tem USB nativo, então geralmente dispensa.

---

## 5. Pinout de referência (ESP32-S3 N16R8)

Baseado em um build público praticamente idêntico ao seu
([arpitxp/Bruce-Smoochie-esp32](https://github.com/arpitxp/Bruce-Smoochie-esp32) —
ESP32-S3 N16R8 + TFT 1,47" ST7789 172×320). **Ajuste conforme o seu `platformio.ini`/board config**;
esses valores são um ponto de partida validado.

```
DISPLAY (ST7789 SPI)          CC1101 (SPI)              nRF24L01+ (SPI)
  SCLK ....... GPIO 12          MOSI ..... GPIO 17        MOSI .... GPIO 37
  MOSI/SDA ... GPIO 11          MISO ..... GPIO 8         MISO .... GPIO 38
  CS ......... GPIO 46          SCK ...... GPIO 18        SCK ..... GPIO 36
  DC ......... GPIO 9           CSN ...... GPIO 15        CSN ..... GPIO 35
  RST ........ GPIO 10          GDO0 ..... GPIO 16        CE ...... GPIO 7
  BLK ........ GPIO 3           GDO2 ..... GPIO 6

PN532 (I2C)                   IR                        GPS NEO-6M (UART)
  SDA ........ GPIO 2           RX (TSOP) . GPIO 4        RX ...... GPIO 39
  SCL ........ GPIO 1           TX (KY-005) GPIO 5        TX ...... GPIO 40

microSD (SPI)                 BOTÕES
  MOSI/CMD ... GPIO 41           UP ...... GPIO 14
  MISO/Dat0 .. GPIO 19           DOWN .... GPIO 13
  SCK/CLK .... GPIO 20           LEFT .... GPIO 47
  CS/Dat3 .... GPIO 42           RIGHT ... GPIO 21
                                 SELECT .. GPIO 48
```

Observações de fiação:
- O CC1101, nRF24 e o SD podem **compartilhar o mesmo barramento SPI** (com CS separados)
  para economizar pinos; no build acima eles usam SPIs/pinos separados para estabilidade.
- Todos os módulos vão em **3.3V** (o ESP32 é 3.3V; nada de 5V nas linhas de sinal).
- Alimente `nRF24 PA/LNA` e `GPS` por 3.3V bem filtrado (é onde o cap de 10 µF importa).

---

## 6. Lista consolidada com quantidades (BOM)

### 6.1 Núcleo — OBRIGATÓRIO
| Qtd | Item | Observação |
|-----|------|-----------|
| 1 | ESP32-S3 **N16R8** | DevKit-C recomendado (já tem regulador, USB e passivos) |
| 1 | TFT **1,47" ST7789** 172×320 SPI | display principal |
| 5 | Push-buttons táteis 6mm | UP/DOWN/LEFT/RIGHT/SELECT (ou 1 joystick 5-way) |
| 1 | Bateria LiPo 3.7V (800–1200 mAh) | versão portátil |
| 1 | Módulo carregador **TP4056** (c/ proteção) | 1 |
| 1 | Conversor boost **MT3608** (ajustar 5V) | 1 |
| 1 | Interruptor liga/desliga | deslizante |
| 1 | Cabo USB-C | gravação/energia |

### 6.2 Módulos operacionais — OPCIONAIS (as funções do "multi-tool")
| Qtd | Item | Função |
|-----|------|--------|
| 1 | **CC1101** | Sub-GHz (315/433/868/915 MHz) |
| 1 | **nRF24L01+ PA/LNA** | 2.4 GHz (jammer/spectrum/Mousejack) |
| 1 | **PN532** | NFC/RFID 13.56 MHz |
| 1 | LED IR 5mm 940nm *(ou 1 módulo KY-005)* | IR TX |
| 1 | Receptor IR **TSOP38238** (ou VS1838B) | IR RX |
| 1 | GPS **NEO-6M** | wardriving/geotag |
| 1 | Leitor microSD (SPI) | armazenamento |
| 1 | Cartão microSD (4–32 GB) | scripts/dumps |

### 6.3 Passivos — capacitores, resistores e transistor
| Qtd | Valor | Para quê |
|-----|-------|----------|
| 3 | Capacitor eletrolítico **10 µF** (ou **100 µF** — ver nota) | 1 no rail 5V, 1 no 3.3V, **1 dedicado no nRF24** |
| 5 | Capacitor **100 nF (0,1 µF) CERÂMICO ("104")** | desacoplamento, 1 por módulo |
| 8 | Resistor **10 kΩ** | 5 pull-ups dos botões* + 3 do microSD (Dat1/Dat2/CS) |
| 2 | Resistor **4,7 kΩ** | pull-ups I²C do PN532 (só se o módulo não tiver) |
| 1 | Resistor **330 Ω** | base do transistor IR |
| 1 | Resistor **47 Ω** | limitador do LED IR |
| 1 | Transistor **2N2222** (ou PN2222A/S8050/2N7000) | driver do LED IR |

\* Os 5 resistores dos botões são **dispensáveis** se usar `INPUT_PULLUP` interno do ESP32.
Se usar o **módulo KY-005**, dispensa o LED IR + transistor + resistores 330 Ω e 47 Ω.

### 6.4 Antenas
| Qtd | Item |
|-----|------|
| 1 | Antena Sub-GHz p/ CC1101 (mola/¼-onda da banda, ex. 433 MHz) |
| 1 | Antena SMA 2.4 GHz p/ nRF24 PA/LNA |
| 1 | Antena GPS cerâmica ativa (normalmente acompanha o NEO-6M) |

### 6.5 Montagem / diversos
| Qtd | Item |
|-----|------|
| 1 | PCB perfurada / protoboard / PCB custom |
| 1 | Conector JST 2 pinos p/ bateria |
| — | Headers macho/fêmea, jumpers, fio, solda |
| 1 | Case (impressão 3D, opcional) |

### 6.6 SÓ se usar o módulo cru ESP32-S3-WROOM-1 (sem DevKit)
| Qtd | Item |
|-----|------|
| 1 | Regulador 3.3V (AMS1117-3.3, ou melhor HT7333/ME6211 p/ bateria) |
| 2 | Resistor 10 kΩ (pull-up EN e GPIO0) |
| 1 | Capacitor 1 µF (no EN, auto-reset) |
| 2 | Botões táteis (BOOT e RESET) |
| 1 | Conversor USB-serial CH340/CP2102 *(dispensável: o S3 tem USB nativo)* |

---

## 7. Firmware

- Código e instruções: [github.com/pr3y/Bruce](https://github.com/pr3y/Bruce) /
  [github.com/BruceDevices/firmware](https://github.com/BruceDevices/firmware).
- Como o seu hardware é **custom**, você vai compilar via **PlatformIO** com uma
  *board config* própria (definindo os GPIOs da seção 5, driver `ST7789`, resolução
  `172×320`, e habilitando os módulos que soldou). A instalação web (bruce.computer)
  serve para placas já suportadas; para o seu build o caminho é compilar do fonte.
- Ative a **PSRAM** e selecione a partição de 16 MB no build.

---

## 8. Onde comprar os passivos — AutoCore Robótica (Fortaleza/CE)

Loja: Rua Rubra Sampaio, 1329 – Farias Brito, Fortaleza/CE · https://www.autocorerobotica.com.br
*(Preços de referência coletados via busca; confirme no site, podem variar.)*

### Opção A — avulso (compra só o que precisa)
| BOM | Qtd | Produto na AutoCore | Preço ref. | Link |
|-----|-----|---------------------|-----------|------|
| Transistor IR | 1 | **2N222 – Transistor NPN** (é o 2N2222, TO-92, até 1A/50V) | R$ 0,40 | /2n222-transistor-npn |
| Resistores (10k/4k7/330/47) | 4 packs | **Resistor CR25 1/4W – x10 pcs** (escolhe o valor) | ~R$ 0,50 /pack | /resistores-diversos |
| Cap eletrolítico (rails + nRF24) | 3 | **Capacitor Eletrolítico 100uF 50V** (substitui o 10 µF; 100 µF é melhor p/ nRF24) | ~R$ 0,30 /un | /capacitor-eletrolitico-100uf-50v |
| Cap 100 nF (104) — **CERÂMICO** | 5 | **Capacitor Cerâmico 50V** (selecionar 100nF) | centavos /un | /capacitor-ceramico-50v- |
| (só WROOM-1 cru) | 1 | **Capacitor Eletrolítico 1uF** (pino EN) | ~R$ 0,25 | /capacitores |

> ⚠️ **Eletrolítico x cerâmico:** a AutoCore pode estar com estoque só de **1 µF / 100 µF / 1000 µF** nos eletrolíticos.
> Sem problema: use **100 µF** no lugar do 10 µF (rails e nRF24). O **1000 µF** é dispensável (grande demais);
> o **1 µF** só serve p/ o EN do WROOM-1 cru. Os **100 nF são CERÂMICOS**, item à parte (não eletrolítico).

### Opção B — kits (mais prático, sobra pra outros projetos)
| Cobre | Produto na AutoCore | Link |
|-------|---------------------|------|
| Todos os resistores | **Kit 560 Resistores 1/4W 5% – 56 valores** | /kit-560-resistores-14w-5-56-valores |
| Todos os 100 nF (e outros cerâmicos) | **Kit Capacitores Cerâmicos 300pcs 10pF–100nF 50V** | /kit-capacitores-ceramicos-variados-300pcs-10pf-a-100nf-50v |
| Eletrolíticos (kit cerâmico não inclui) | comprar **10 µF / 100 µF** avulsos | /capacitores |
| Transistor | **2N222 – Transistor NPN** | /2n222-transistor-npn |

Categorias úteis: Resistores `/resistores` · Capacitores `/capacitores` · Transistores `/transistores`.

> Dica: para o desacoplamento (100 nF) prefira **cerâmico (104)** ao poliéster.
> A loja também tem o **Capacitor Poliéster 100nF/250V** (`/capacitor-poliester-100nf-250v`),
> que funciona, mas o cerâmico é o ideal junto de cada módulo.

## Fontes
- Bruce (repositório principal): https://github.com/pr3y/Bruce
- Bruce (org atual): https://github.com/BruceDevices/firmware
- Build de referência ESP32-S3 N16R8 + TFT 1,47" (pinout): https://github.com/arpitxp/Bruce-Smoochie-esp32
- Build ESP32-S3 alternativo: https://github.com/arpitxp/esp32-bruce
- Dispositivos suportados (DeepWiki): https://deepwiki.com/pr3y/Bruce/10.1-supported-devices
- Site oficial: https://bruce.computer/
