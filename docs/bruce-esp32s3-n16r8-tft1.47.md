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
| 8 | **Barra LED RGB endereçável (WS2812-8)** | Indicador visual / efeitos de notificação | 1 GPIO (protocolo próprio, via RMT) | — |

### Antenas (não esqueça)
- **CC1101:** antena mola/helicoidal ou fio ¼-onda para a banda escolhida (ex.: ~17,3 cm p/ 433 MHz).
- **nRF24 PA/LNA:** antena **SMA 2.4 GHz** (a versão PA/LNA precisa de antena para render).
- **GPS NEO-6M:** antena cerâmica ativa (geralmente acompanha o módulo).
- **ESP32-S3:** antena de PCB já integrada no WROOM-1 (ou conector u.FL na versão -U).

### 3.1 Barra de LED RGB endereçável (WS2812-8)

Módulo com 8× LED WS2812 (cada um com o chip driver embutido), 1 fio de dado por barra
(`IN`/`OUT` pra encadear mais barras em série), controlado por **1 único GPIO** do ESP32.

**Ligação:**
- `5V` → **rail de 5V** (do boost), não o 3.3V do ESP32 — WS2812 funciona melhor/mais estável em 5V.
- `GND` → GND comum.
- `IN` (dado) → 1 GPIO livre do ESP32. Nível lógico 3.3V do ESP32 costuma acender o WS2812
  normalmente em fios curtos; se der cor errada/LED não acende, o fix padrão é um
  **resistor ~300–500 Ω em série no fio de dado**, perto do pino do ESP32 (ver seção 4.5).
- `OUT` fica livre — só usa se quiser encadear outra barra/fita depois.
- **Orçamento de corrente:** 8 LEDs WS2812 em branco/100% de brilho chegam a **~480 mA**
  (60 mA/LED). Isso **soma** ao consumo do CC1101+nRF24+GPS já discutido — reforça ainda
  mais a importância dos capacitores de reservatório nos rails 5V/3.3V e do teste de
  tensão sob carga (ver troubleshooting do "curto intermitente" mais acima): quanto mais
  módulos ligados ao mesmo tempo, mais perto do limite da proteção da bateria/boost você
  fica.

**Como habilitar no firmware Bruce:**
No seu board config custom (mesmo arquivo onde você define o pinout do display/módulos
pra compilar via PlatformIO), defina:
```cpp
#define RGB_LED   <GPIO escolhido>   // pino de dado (IN) da barra
#define NUM_LEDS  8                  // WS2812-8 = 8 LEDs
```
O Bruce já usa a lib **FastLED** internamente (com o periférico **RMT** do ESP32 pra
timing, sem travar a UI) — depois de definir o pino, o menu **LED** (e os comandos via
serial `led r|g|b`, `led rgb`, `led hex`, `led brightness`, `led effect`) já controlam a
barra nativamente, sem precisar mexer em mais nada.

**Possibilidades de uso:**
- **Pronto, nativo do firmware (menu LED / comandos serial):**
  - Cor sólida customizável (RGB ou hex).
  - Efeitos prontos: *Breathe* (respiração), *Color Cycle*, *Color Wheel*, *Chase*,
    *Chase Tail* — com velocidade e direção ajustáveis.
  - Brilho de 0–100% e "LED blink" como notificação genérica de eventos do sistema.
  - Indicador visual "de bancada" — dá pra deixar um efeito rodando só de estética,
    ou usar brilho baixo fixo como "power on" LED.
- **Customização (exige editar/estender o firmware, não é automático):**
  - Vincular cor/efeito a um estado específico (ex.: vermelho piscando enquanto o
    jammer ou o CC1101 TX está ativo, verde parado quando ocioso) — o `led` já expõe os
    comandos, falta só chamar isso nos pontos certos do código das telas de RF.
  - Indicador de força de sinal (RSSI do Spectrum/Sub-GHz) mapeado pra brilho/cor —
    também precisa de um pequeno hook no código, não vem pronto.
  - Como só usa 1 GPIO e tem `OUT` pra encadear, dá pra expandir depois (mais LEDs,
    iluminação de case) sem gastar GPIO extra.

### 3.2 Customização: barra de nível (RSSI) no Copy/Spectrum — 1 cor só

Sim — ligando no `IN`, cada um dos 8 LEDs é **endereçável individualmente**. O Bruce já
guarda a fita inteira num array `CRGB leds[LED_COUNT];` (arquivo `src/core/led_control.cpp`)
e escreve nele por índice (`leds[i] = cor;` ... `FastLED.show();`) — é exatamente esse
padrão que o `ledEffectTask()` usa hoje pros efeitos prontos (Chase, Color Wheel etc.).
**Não existe** um efeito "barra de nível" pronto no firmware — pra ter "acende conforme
detecta sinal" é preciso adicionar uma função nova e chamá-la de dentro da tela de
Copy/Spectrum do CC1101. RSSI como métrica é a escolha certa (sobe/desce com a força do
sinal captado, igual medidor de bateria/sinal); sem gradiente de cor — só acende de 0 a 8
LEDs numa cor fixa. Roteiro:

**1) Função de barra (nova, em `led_control.h`/`.cpp`, do lado das outras `setLed*`):**
```cpp
// led_control.h
void setLedBar(uint8_t level, uint8_t maxLevel, CRGB color);

// led_control.cpp
void setLedBar(uint8_t level, uint8_t maxLevel, CRGB color) {
    if (level > maxLevel) level = maxLevel;
    int litCount = map(level, 0, maxLevel, 0, LED_COUNT);
    for (int i = 0; i < LED_COUNT; i++) {
        leds[i] = (i < litCount) ? color : CRGB::Black;
    }
    FastLED.show();
}
```

**2) Chamar isso a partir do RSSI que o Copy já lê.** O laço de scan/copy do CC1101 já lê
RSSI pra decidir "tem sinal aqui" — procura no seu checkout do firmware
(`grep -rn "getRssi\|Rssi\|rssi" src/modules/rf/`) onde esse valor é lido e chama
`setLedBar()` logo depois, mapeando a faixa de RSSI que a tela já usa hoje pra 0–8:
```cpp
int rssi = ELECHOUSE_cc1101.getRssi();            // ajuste pro nome real na sua versão
int nivel = map(constrain(rssi, RSSI_MIN, RSSI_MAX), RSSI_MIN, RSSI_MAX, 0, LED_COUNT);
setLedBar(nivel, LED_COUNT, CRGB::Red);           // troca CRGB::Red pela cor que preferir
```
(`RSSI_MIN`/`RSSI_MAX` = os mesmos limites que a tela de Copy já usa pra decidir "sinal
fraco" vs "forte" — copia esses valores de lá em vez de inventar novos.)

**3) Cuidado com o `ledEffectTask` brigando pelo array.** A `setLedColor()` já tem um
padrão de "modo preview" (`isPreviewLed`/`previewLedEffect`) pra suspender o efeito normal
enquanto mostra outra coisa — reusa essa mesma ideia: liga uma flag tipo `isRfLedActive`
ao entrar na tela de Copy (e desliga ao sair) pra impedir que a task de efeito (rodando em
background) sobrescreva a barra a cada animação e cause flicker.

Isso é desenvolvimento de firmware de verdade (não é opção de menu) — exige recompilar via
PlatformIO com essa alteração. Fora do RSSI, dá pra adaptar o mesmo `setLedBar()` pra outras
métricas: nº de pulsos já capturados num timeout, ou progresso de um replay.

### 3.3 Expansor de I/O pra 6 botões + PN532 (economizar GPIO)

Com o pinout da seção 5 já saturado (ver aviso lá), a saída pra caber **6 botões + o PN532**
sem brigar por GPIO é um **expansor I2C**:

- **Recomendação ideal: AW9523.** É o chip que o **próprio firmware Bruce já usa
  nativamente** (o objeto `ioExpander` do código-fonte já controla pelo menos o motor de
  vibração — `IO_EXP_VIBRO` — nas placas oficiais Cardputer/StickC). 16 canais, pull-up
  configurável, interrupt-on-change. Vantagem real: reaproveita driver que já existe no
  Bruce em vez de escrever um do zero. **Não é achado nas lojas de Fortaleza** — só
  importado (Adafruit/AliExpress).
- **Alternativa disponível localmente: MCP23017.** 16 canais, GPIO de verdade (não
  quase-bidirecional como o PCF8574), pull-up interno, interrupt-on-change — o mais
  próximo do AW9523 em capacidade. Ver seção 8 (Opção C) pra onde comprar em Fortaleza.
- **Por que o PN532 não precisa de expansor nenhum:** ele já é I2C nativo (2 fios,
  endereço `0x24`) — compartilha o **mesmo barramento SDA/SCL** do expansor e do ESP32.
  Não gasta GPIO extra pra ele.

**Ligação:** expansor (`VCC` 3.3V / `GND` / `SDA` / `SCL`) no mesmo barramento do PN532;
os 6 botões em pinos `P0.x`/`P1.x` do expansor, outro terminal no GND (usa o pull-up
interno do chip; sem ele, 10 kΩ por botão nesses pinos, não no ESP32). Endereço padrão do
MCP23017 (`0x20`) não colide com o `0x24` do PN532 — não precisa tocar nos jumpers A0–A2.

**No firmware:** como o `ioExpander`/AW9523 já é código nativo do Bruce, se for por esse
caminho o esforço é reaproveitar o padrão existente (`grep -rn "ioExpander\|IO_EXP" src/`)
em vez de escrever leitura de botão do zero. Indo de MCP23017 (não nativo), é o mesmo tipo
de trabalho que a seção 3.2 já fez pro RSSI: usar `Wire.h` pra ler os registros do chip e
plugar isso onde o Bruce hoje faz `digitalRead()` dos botões.

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

### 4.5 Checklist "por canto" — onde vai cada capacitor/resistor

Consolidado de onde soldar cada passivo, módulo por módulo. Isso é o principal suspeito
quando o **CC1101 só capta "sinal em todo lugar" depois de colocar a antena** (sintoma
clássico de ruído entrando pela alimentação/RF, não de bug de firmware — ver seção 5.1):

| Local | Capacitor | Resistor |
|-------|-----------|----------|
| **Rail 5V** (saída do boost MT3608) | 1× **10 µF** (ou 100 µF) eletrolítico, o mais perto possível da saída do boost | — |
| **Rail 3.3V** (saída do regulador do ESP32/DevKit) | 1× **10 µF** (ou 100 µF) eletrolítico, perto de onde o 3.3V se divide pros módulos | — |
| **ESP32-S3** (só se módulo cru WROOM-1, sem DevKit) | 100 nF + 10 µF no pino 3V3 | 10 kΩ pull-up no EN + 10 kΩ pull-up no GPIO0 |
| **CC1101** ⚠️ | **100 nF cerâmico direto nos pinos VCC/GND do módulo** — solda um extra mesmo se a placa clone já tiver um, o de fábrica costuma ser fraco/mal posicionado | pull-up ~10 kΩ no CSN (evita flutuar no boot/bus compartilhado) |
| **nRF24L01+** ⚠️ (principal suspeito de ruído) | 100 nF cerâmico VCC/GND **+ 10 µF (ou 100 µF) eletrolítico dedicado**, soldado direto entre VCC e GND do módulo — sem esse cap grande ele "sujeita" a alimentação inteira | pull-up ~10 kΩ no CSN |
| **PN532** (I²C) | 100 nF cerâmico VCC/GND | 4,7 kΩ em SDA + 4,7 kΩ em SCL (só se o breakout não já trouxer) |
| **Leitor microSD** | 100 nF cerâmico VCC/GND | 10 kΩ pull-up em CS, Dat1 e Dat2 |
| **Botões de navegação** | — | 10 kΩ pull-up por botão (dispensável com `INPUT_PULLUP` interno) |
| **IR TX** (só LED discreto, sem KY-005) | — | 330 Ω na base do transistor 2N2222 + 47–100 Ω limitador do LED |
| **Barra WS2812 (RGB)** | 100–1000 µF eletrolítico entre VCC/GND, bem perto do 1º LED da barra (absorve o pico de corrente de todos os LEDs acendendo juntos) | ~300–500 Ω em série no fio de dado (`IN`), perto do pino do ESP32 |

**Se o problema for justamente o CC1101 "escutando tudo" ao plugar a antena:** o primeiro
suspeito é a linha **nRF24** desta tabela — sem o cap eletrolítico dedicado nele, o ruído
da alimentação sobe pro CC1101 e o RSSI dele passa a disparar com qualquer ruído (é um
sintoma clássico do chip: sem antena não aparece porque o front-end RF está "surdo",
com antena ele fica sensível ao ruído que estava ali o tempo todo). Depois disso, confira
o 100 nF extra no CC1101 e a distância do fio da antena em relação aos fios do SPI
compartilhado.

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

> ⚠️ **Sem GPIO livre para a barra WS2812 (RGB_LED) neste pinout de referência** — todos os
> GPIOs 0–48 utilizáveis do ESP32-S3 N16R8 já estão ocupados pelos módulos acima (ou são
> reservados de fábrica: 26–32 = flash, 19/20 = USB nativo, 43/44 = UART0). Duas saídas
> práticas:
> 1. **Reaproveitar um pino de strap** (GPIO 0 ou GPIO 45) para o `RGB_LED` — este mesmo
>    pinout já faz isso com sucesso para o display (GPIO 3 = BLK, GPIO 46 = CS, ambos
>    strap). Funciona porque o WS2812 só é "driven" depois do boot; só evite qualquer
>    pull elétrico no módulo que force o nível errado durante o power-on.
> 2. **Liberar um pino não essencial** — ex.: um dos botões de direção (se for usar
>    joystick analógico) ou o IR RX/TX (se não for montar infravermelho).

Observações de fiação:
- O CC1101, nRF24 e o SD podem **compartilhar o mesmo barramento SPI** (com CS separados)
  para economizar pinos; no build acima eles usam SPIs/pinos separados para estabilidade.
- Todos os módulos vão em **3.3V** (o ESP32 é 3.3V; nada de 5V nas linhas de sinal).
- Alimente `nRF24 PA/LNA` e `GPS` por 3.3V bem filtrado (é onde o cap de 10 µF importa).

### 5.1 Por que SCK/MOSI/MISO do CC1101 e do nRF24 não deveriam ir nos mesmos pinos do SD

**Eletricamente dá, sim.** SPI é um barramento *multi-slave* por definição: vários
dispositivos podem compartilhar `SCK`/`MOSI`/`MISO`, desde que **cada um tenha seu próprio
CS (chip-select)** e nunca dois CS ativos ao mesmo tempo. É por isso que o pinout de
diagramas oficiais do Bruce (ex.: Cardputer ADV com CC1101+nRF24 no mesmo "SD sniffer")
usa exatamente esse esquema — `CLK`/`CMD`/`DAT0` do slot de SD reaproveitados como
`SCK`/`MOSI`/`MISO` do rádio, cada um com seu próprio CSN.

**O problema é no firmware Bruce, não na eletrônica.** Compartilhar esse barramento com CC1101
+ nRF24 **+** cartão SD ao mesmo tempo tem bugs conhecidos (alguns já corrigidos, outros
ainda abertos em set/2026):

1. **CS flutuante no boot atropela o SD.** `setupSdCard()` roda dentro de `begin_storage()`
   **antes** de `_post_setup_gpio()` configurar os pinos de CS do nRF24/CC1101/LoRa como
   saída. Nesse intervalo os CS ficam em alta impedância ("flutuando") e, num barramento
   compartilhado, isso pode puxar a linha para nível baixo bem no momento em que o
   `SD.begin()` está tentando responder — o SD simplesmente não monta, sem erro claro.
   Corrigido na PR [#2926](https://github.com/BruceDevices/firmware/pull/2926)
   (inicializar todo CS como `OUTPUT/HIGH` **antes** de `setupSdCard()`), mas só vale
   a partir da versão de firmware que já inclui esse fix.
2. **Ordem de init entre CC1101 e nRF24.** Em placas dual-rádio com um SPI só, o CC1101
   pode falhar se for o primeiro a inicializar — o driver dele assume que o barramento já
   foi "acordado" por outra coisa. Abrir o nRF24 primeiro configura o barramento e o
   CC1101 passa a responder. É dependência de ordem no driver, não limitação de fiação.
3. **Bug ainda aberto combinando os três.** A issue
   [#2899](https://github.com/BruceDevices/firmware/issues/2899) (set/2026) relata que,
   com CC1101 + SD juntos em modo "shared SPI", só o modo **legacy** (pinos dedicados,
   sem compartilhar) funciona — no modo compartilhado o SD para de responder. Sem fix
   definitivo até a data deste doc.

**Por isso** o build de referência da seção 5 (e a maioria dos guias da comunidade) prefere
dar **pinos/SPI dedicados** para CC1101, nRF24 e SD em vez de compartilhar — evita esses
três bugs de inicialização/corrida de CS que o firmware ainda tem, mesmo sabendo que "na
teoria" o hardware permite compartilhar.

**Se ainda assim quiser compartilhar** (para economizar GPIOs num ESP32 com poucos pinos
livres):
- CS **sempre** dedicado por módulo — nunca compartilhe o próprio CS.
- Pull-up (~10 kΩ) em cada linha de CS, para não flutuar durante o boot antes do
  `_post_setup_gpio()` rodar.
- Inicialize o **nRF24 antes** do CC1101 no código/ordem de detecção.
- Use uma versão do firmware **posterior** ao merge da PR #2926.
- Ainda assim, teste bem o SD junto — a issue #2899 mostra que pode restar bug residual
  no modo compartilhado.

Fontes: [PR #2926 — fix de CS flutuante no SD](https://github.com/BruceDevices/firmware/pull/2926) ·
[Issue #2899 — shared SPI falha com SD](https://github.com/BruceDevices/firmware/issues/2899) ·
[Issue #2435 — dual-radio CC1101+nRF24 com brucePins.conf](https://github.com/BruceDevices/firmware/issues/2435) ·
[Wiki: wiring CC1101+nRF24 (Cardputer ADV)](https://github.com/BruceDevices/Wiki/blob/main/docs/wiring-diagrams/cardputer-adv/cc1101-nrf24.md) ·
[Issue #586 — CYD, múltiplos periféricos SPI simultâneos](https://github.com/BruceDevices/firmware/issues/586)

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

### Opção C — Expansor I2C pra 6 botões + NFC (seção 3.3)

O **AW9523** (o expansor que o próprio Bruce já usa nativamente) não é achado nem na
AutoCore nem na SmartKits — é item de importação (Adafruit/AliExpress). Nas duas lojas
de Fortaleza, o mais próximo disponível é o **MCP23017** (16 canais, o de maior recomendação
depois do AW9523: GPIO de verdade, pull-up interno, interrupt-on-change):

| Loja | Produto | Preço ref. | Link |
|------|---------|-----------|------|
| AutoCore Robótica | **Módulo Expansor de Portas Digitais I2C 16 Bits MCP23017** (pronto, já montado) | ~R$ 44,90 | /modulo-expansor-de-portas-digitais-i2c-16-bits-mcp23017 |
| AutoCore Robótica | **MCP23017 — CI avulso** (só o chip, monta você mesmo) | ~R$ 32,90 | /mcp23017-ci-expansor-de-porta-entrada-saida-i2c |
| AutoCore Robótica | **Módulo Expansor de I/O I2C PCF8574** (alternativa mais barata, 8 canais) | ~R$ 13,90 | /modulo-expansor-de-io-i2c-pcf8574 |
| SmartKits | **Módulo MCP23017 Expansor de Portas Bidirecional 16 Bits** | conferir no site | /modulo-mcp23017-expansor-de-portas-bidirecional |
| SmartKits | **Módulo Expansor de Portas I2C 8 Bits PCF8574** | conferir no site | /modulo-expansor-de-portas-i2c-8-bits-pcf8574 |

*(Preços de referência coletados via busca; confirme disponibilidade/preço direto no site — o
acesso automático às duas lojas ficou bloqueado ao tentar conferir ao vivo.)*

> Pega o **módulo já montado** da AutoCore (não o CI avulso) — já vem com os resistores de
> pull-up do barramento I2C e o regulador, então não precisa somar mais nada da seção 4 pra
> ele. Ele fica no mesmo barramento SDA/SCL do PN532 (endereço padrão `0x20`, não colide
> com o `0x24` do PN532 — não precisa mexer nos jumpers de endereço).

## Fontes
- Bruce (repositório principal): https://github.com/pr3y/Bruce
- Bruce (org atual): https://github.com/BruceDevices/firmware
- Build de referência ESP32-S3 N16R8 + TFT 1,47" (pinout): https://github.com/arpitxp/Bruce-Smoochie-esp32
- Build ESP32-S3 alternativo: https://github.com/arpitxp/esp32-bruce
- Dispositivos suportados (DeepWiki): https://deepwiki.com/pr3y/Bruce/10.1-supported-devices
- Site oficial: https://bruce.computer/
