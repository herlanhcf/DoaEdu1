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
| 9 | **LoRa (RA-02, chip SX1278/SX1276)** | Longo alcance (chat Bruce-a-Bruce), único jeito de "enxergar" sinal LoRa no ar | SPI (compartilha SCK/MOSI/MISO com CC1101/nRF24) + NSS/DIO0/RST dedicados | 433/868/915 MHz |

### Antenas (não esqueça)
- **CC1101:** antena mola/helicoidal ou fio ¼-onda para a banda escolhida (ex.: ~17,3 cm p/ 433 MHz).
- **nRF24 PA/LNA:** antena **SMA 2.4 GHz** (a versão PA/LNA precisa de antena para render).
- **LoRa RA-02:** antena mola/helicoidal ¼-onda da banda escolhida (geralmente acompanha o módulo).
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

### 3.3 Botões — ❌ PCF8574 DESCARTADO (não cabia fisicamente) → 6 botões diretos no GPIO

> O expansor PCF8574 desta seção **não foi pra frente** — não cabia fisicamente no case.
> **Decisão final: os 6 botões vão direto em GPIO individual**, sem expansor nenhum — mapa
> completo, fiação e as ressalvas de cada pino de strap estão na **seção 5** (tabela de
> botões) e na **seção 4.6** (item por item). O **PN532 continua em I2C** (é o protocolo
> nativo do chip, não depende do expansor) — sozinho no barramento agora, sem compartilhar
> com mais nada. Texto original do PCF8574 abaixo, só como histórico:

<details>
<summary>Plano antigo do PCF8574 (descartado) — não vai ser usado</summary>

**Escolha que foi descartada: PCF8574** (expansor I2C de 8 bits) — endereço padrão `0x20`,
não colide com o `0x24` do PN532, então os dois entrariam no **mesmo barramento** sem tocar
em jumper de endereço.

- `P0` = UP, `P1` = DOWN, `P2` = LEFT, `P3` = RIGHT, `P4` = SELECT, `P5` = BACK.
- Cada botão: um lado no pino `Px` do PCF8574, outro no GND. Pull-up interno fraco do
  PCF8574 (~100 kΩ) bastaria, com reforço de 10 kΩ externo se desse bounce.

</details>

### 3.4 Módulo LoRa — ❌ DESCARTADO (decisão do usuário, não vai entrar no build)

> Ficava aqui a recomendação de módulo LoRa (RA-02) — **removido do plano** a pedido do
> usuário. Os 3 GPIOs que seriam do LoRa (**7, 42 e 47**) estão livres em reserva na
> seção 5. Texto abaixo fica só como referência caso mude de ideia depois.

<details>
<summary>Recomendação antiga (RA-02, SX1278/SX1276) — não vai ser usada</summary>

**Recomendado: RA-02 (chip SX1278, compatível com o driver "SX1276" do Bruce), interface
SPI.** ⚠️ **Não confundir com a linha Ebyte E32** (ex.: E32-433T20D) — esse é um módulo
"caixa preta" com **UART**/AT-commands, sem os pinos SPI expostos; o driver de LoRa do
Bruce fala **SPI direto com o chip** (`NSS`/`MOSI`/`MISO`/`SCK`/`DIO0`/`RST`), então o E32
**não funciona** com o Bruce. Evite qualquer módulo com "E32", "TTL" ou "AT command" no nome.

**Fiação:** o Bruce já detecta automaticamente SPI compartilhado pra LoRa
(`selectLoraSPIBus()`), então `SCK`/`MOSI`/`MISO` iriam nos mesmos fios do CC1101/nRF24
(GPIO 12/11/13), com `NSS`/`DIO0`/`RST` nos GPIOs 7/47/42 que agora estão livres.

**Vantagens de ter um:**
1. **O CC1101 é literalmente surdo pra LoRa** — ele só faz FSK/OOK/ASK, não decodifica o
   espalhamento espectral chirp do LoRa de jeito nenhum. Sem um chip SX127x/SX126x de
   verdade, o Bruce fica 100% cego pra qualquer coisa em LoRa no ar (sensores LoRaWAN,
   nós Meshtastic, telemetria privada) — nem RSSI ele lê direito. É o motivo mais forte
   pra ter o módulo.
2. **Alcance muito maior** que o CC1101 — quilômetros em linha de visão, mesmo em baixa
   potência.
3. **Chat nativo do Bruce** — mensagem de texto ponto-a-ponto de longo alcance entre dois
   Bruce, sem WiFi/celular.
4. Consumo baixíssimo em modo de escuta.

⚠️ **Limitação:** o Chat do Bruce é **proprietário** — só conversa com outro Bruce, **não
interopera com Meshtastic, MeshCore nem gateways LoRaWAN genéricos**. Se o objetivo é
farejar essas redes especificamente (não só ter alcance longo), o firmware hoje só dá
visibilidade de RF crua (RSSI/presença de portadora), não decodifica o protocolo delas.

</details>

### 3.5 Percentual de bateria (você usa TP4056, sem fuel gauge)

O **TP4056** é só carregador (CC/CV) — não tem telemetria nenhuma, nem I2C nem UART, só os
LEDs de status (CHRG/STDBY). Pra ter % de bateria no Bruce, precisa de um sensor separado
lendo a tensão. **Método nativo do Bruce**: divisor de tensão + ADC — a maioria das placas
lê bateria assim, `BAT_PIN` no board config + `getBattery()` já converte pra %.

**Ligação:**
- Divisor **2× 100 kΩ** (1:1) do nó **BAT+** do TP4056 (a tensão crua da bateria, 3.0–4.2V)
  até o `BAT_PIN` — divide a tensão pela metade, já que a bateria vai até 4,2V e o ADC do
  ESP32 aguenta até 3,3V.
- `BAT_PIN` = **GPIO 6** (ver seção 5 — liberado depois que PREV/NEXT/SELECT foram pro
  PCF8574; precisa ser ADC1, GPIO 1–10; o ADC2, GPIO 11–20, falha com o WiFi ativo, que no
  Bruce é o tempo todo).
- No firmware: lê com `analogReadMilliVolts()` e multiplica por 2 pra ter a tensão real.

**Curva de conversão tensão → % (LiPo não é linear):**
| Tensão | % aproximado |
|--------|---------------|
| 4,2 V | 100% |
| 3,9 V | ~80% |
| 3,7 V | ~50% |
| 3,5 V | ~20% |
| 3,0–3,2 V | 0% (BMS de proteção já corta por aqui) |

**Alternativa mais precisa (se topar importar):** chip **MAX17048** (fuel gauge dedicado,
ModelGauge, I2C endereço `0x36`) — devolve % pronto, bem mais preciso que a curva de tensão
simples, e **não gasta GPIO novo** (entra no mesmo barramento I2C do PN532/MCP23017). Não
achado nas lojas de Fortaleza (AutoCore/SmartKits) — importado (Adafruit/AliExpress), e não
confirmei driver nativo no Bruce pra ele — a integração seria no mesmo estilo do `setLedBar()`
da seção 3.2 (ler por `Wire.h` e alimentar o `getBattery()`).

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

### 4.5 Checklist "por canto" (histórico) — ver seção 4.6 pra lista atual

> ⚠️ Essa tabela ficou desatualizada depois do redesenho dos botões/PN532 pro I2C — os
> valores de capacitor continuam válidos, mas os pinos e os resistores de botão/PN532 não.
> **Use a seção 4.6 abaixo como referência atual.**

### 4.6 Lista final consolidada — TODO componente, VCC/GND

Referência única e atual, batendo com o pinout da seção 5 (PN532 sozinho no I2C, 6 botões
diretos no GPIO sem expansor, chave seletora de rádio, LoRa fora, bateria no GPIO 6).
**Usando só o que você já comprou:**
capacitor cerâmico **104 (= 100 nF)**, capacitor **10 µF**, resistores, e o módulo **IR
KY-005** (já traz transistor+resistores → você NÃO precisa de transistor nem dos resistores
do IR TX). Onde eu antes sugeria eletrolítico grande, agora está adaptado pro seu 10 µF.

| # | Componente | VCC | GND | Capacitor (104 / 10 µF) | Resistor |
|---|-----------|-----|-----|--------------------------|----------|
| 1 | **Rail 5V** (saída do boost MT3608) | bat/USB → boost | comum | 1× **10 µF** na saída do boost | — |
| 2 | **Rail 3.3V** (regulador ESP32/DevKit) | 5V → regulador | comum | 1× **10 µF** onde o 3.3V se divide pros módulos | — |
| 3 | **ESP32-S3** (só se WROOM-1 cru, sem DevKit) | 3.3V | comum | 104 + 10 µF no 3V3 | 10 kΩ no EN · 10 kΩ no GPIO0 + botão BOOT→GND |
| 4 | **Display ST7789** | 3.3V | comum | 1× **104** VCC/GND | — |
| 5 | **CC1101** ⚠️ | 3.3V (**via chave**) | comum | 1× **104** direto nos pinos VCC/GND (solda extra mesmo se o clone já tiver) | 10 kΩ pull-up no CS (GPIO 10) |
| 6 | **nRF24L01+** ⚠️ | 3.3V (**via chave**), bem filtrado | comum | 1× **104 + 1× 10 µF dedicado**, direto no VCC/GND do módulo | 10 kΩ pull-up no CSN (GPIO 40) |
| 7 | **microSD** | 3.3V | comum | 1× **104** VCC/GND | 10 kΩ pull-up no CS (GPIO 15) |
| 8 | **IR RX** (TSOP38238/VS1838B) | 3.3V | comum | 1× **104** VCC/GND (filtra ruído do sensor) | — |
| 9 | **IR TX (módulo KY-005)** | 3.3V/5V | comum | — | **nada** — KY-005 já tem transistor+resistor embutidos; liga o `S` direto no GPIO 2 |
| 10 | **Barra WS2812×8** | 5V | comum | 1× **10 µF** perto do 1º LED (é menos que o ideal de 100–1000 µF, mas ok pra brilho moderado; se piscar/glitch em branco 100%, some outro 10 µF em paralelo) | **330–470 Ω** em série no `DIN` (GPIO 38), perto do ESP32 |
| 11 | **PN532** (I2C, SDA=48/SCL=7, endereço 0x24 — sem expansor, sozinho no barramento) | 3.3V | comum | 1× **104** VCC/GND | **4,7 kΩ em SDA + 4,7 kΩ em SCL** (só se o breakout não já trouxer de fábrica) |
| 12 | ↳ **LEFT** (botão) | — | via pull-up | — | pull-up **10 kΩ** no GPIO 47, botão fecha pro GND |
| 13 | ↳ **SELECT** (botão) | — | via pull-up | — | pull-up **10 kΩ** no GPIO 0, botão fecha pro GND |
| 14 | ↳ **UP** (botão) | — | via pull-up | — | pull-up **10 kΩ** no GPIO 3, botão fecha pro GND |
| 15 | ↳ **DOWN** (botão) | — | via pull-up | — | pull-up **10 kΩ** no GPIO 43 (⚠️ confirme devkit — nota seção 5), botão fecha pro GND |
| 16 | ↳ **RIGHT** (botão) | — | via pull-up | — | pull-up **10 kΩ** no GPIO 44 (⚠️ confirme devkit — nota seção 5), botão fecha pro GND |
| 17 | ↳ **BACK** (botão) ⚠️ fiação invertida | — | via pull-down | — | pull-**down** **10 kΩ** no GPIO 46 (resistor pro GND), botão fecha pro **3.3V** (não pro GND — é o único ao contrário) |
| 18 | **GPS NEO-6M** | 3.3V bem filtrado | comum | 1× **104** VCC/GND | — |
| 19 | **Bateria — divisor ADC** (BAT_PIN = GPIO 6) | BAT+ do TP4056 | comum | — | **2× resistor IGUAL em série** BAT+→GND, ponto médio no GPIO 6. Ideal **100 kΩ+100 kΩ** (gasta só ~16 µA); se o kit não tiver 100 k, qualquer par igual serve (ex. 10 k+10 k, gasta mais) |
| 20 | **⚡ Chave seletora CC1101/nRF24** (SPDT) | comum = 3.3V; A→VCC CC1101; B→VCC nRF24 | GND dos rádios sempre ligado | — | — (item 5 e 6 já têm os 10 kΩ nos CS que a chave exige) |

**Contagem rápida do que soldar (fora os módulos):**
- **Capacitor 104 (100 nF):** ~7 un → itens 3,4,5,6,7,8,11,18 (1 por módulo — **sem** PCF8574, que saiu da lista).
- **Capacitor 10 µF:** ~4 un → rail 5V, rail 3.3V, nRF24 dedicado, WS2812.
- **Resistor 10 kΩ:** **~9 un** → CS do CC1101, CSN do nRF24, CS do SD, **+ 6 botões diretos** (5 pull-up + 1 pull-down) — subiu bastante em relação à versão com PCF8574, é o custo de não usar expansor.
- **Resistor 4,7 kΩ:** 2 un → pull-up do I2C do PN532 (se o breakout não trouxer).
- **Resistor 330–470 Ω:** 1 un → série do WS2812.
- **Resistor p/ divisor de bateria:** 2 un iguais (100 kΩ ideal).
- **Chave SPDT:** 1 un → seletora de rádio.
- **Transistor:** **nenhum** (KY-005 resolve o IR TX).
- **PCF8574:** **removido da lista** — não é mais usado.

**Se o problema for o CC1101 "escutando tudo" ao plugar a antena:** o primeiro suspeito é a
linha **nRF24** (#6) — sem o cap eletrolítico dedicado, o ruído da alimentação sobe pro
CC1101 e o RSSI dispara com qualquer ruído (sem antena não aparece porque o front-end RF
está "surdo"; com antena, fica sensível ao ruído que já estava ali). Depois, confira o
100 nF extra no CC1101 (#5) e a distância do fio da antena em relação ao SPI compartilhado.

**O que saiu da lista** (não precisa mais desses componentes, mesmo que tenha visto em
revisões antigas deste doc): pull-up direto de botão no ESP32 (agora é no PCF8574, item 14),
pull-up de PN532 em pino dedicado (agora é o pull-up único do barramento, item 11), qualquer
passivo de LoRa (módulo descartado).

---

## 5. Pinout final (ESP32-S3 N16R8) — FIXO vs MANIPULÁVEL

Inspirado na densidade de pinos do build de referência
([arpitxp/Bruce-Smoochie-esp32 / ARTFOR](https://github.com/arpitxp/Bruce-Smoochie-esp32) —
mesmo ESP32-S3 N16R8 + TFT 1,47"). Duas travas de projeto:

- 🔒 **FIXO (já soldado, NÃO mexer):** Display, CC1101, nRF24, GPS.
- 🔧 **MANIPULÁVEL (livre pra otimizar):** SD, IR, LED, NFC, botões, bateria, chave seletora.

```
🔒 FIXO (soldado)
DISPLAY (ST7789 SPI)          CC1101 (SPI)              nRF24L01+ (SPI)
  SCK ........ GPIO 41          SCK ...... GPIO 12         SCK ..... GPIO 12 (compartilhado)
  MOSI ....... GPIO 21          MOSI ..... GPIO 11         MOSI .... GPIO 11 (compartilhado)
  DC ......... GPIO 4           MISO ..... GPIO 13         MISO .... GPIO 13 (compartilhado)
  CS ......... GPIO 5           CS ....... GPIO 10         CE ...... GPIO 39
  RST ........ GPIO 14          GDO0 ..... GPIO 9          CSN ..... GPIO 40

GPS NEO-6M (UART)             ⚡ CHAVE SELETORA DE RÁDIO (SPDT, na linha VCC — NÃO gasta GPIO)
  RX ......... GPIO 8            comum ...... 3.3V
  TX ......... GPIO 42           posição A .. VCC do CC1101   (nRF24 desligado)
                                 posição B .. VCC do nRF24    (CC1101 desligado)

🔧 MANIPULÁVEL (otimizado)
microSD (SPI)                 IR (módulo KY-005)        LED WS2812 ×8
  CS ......... GPIO 15           RX ...... GPIO 1           DIN ..... GPIO 38 (+ R série 330–470 Ω)
  SCK ........ GPIO 18           TX ...... GPIO 2
  MISO ....... GPIO 17
  MOSI ....... GPIO 16         BATERIA (ADC1)
                                 BAT_PIN .. GPIO 6 (divisor 2× resistor igual)

I2C — só o PN532 (sem expansor, botões agora são diretos)
  SDA ........ GPIO 48
  SCL ........ GPIO 7
    → PN532 (NFC), endereço 0x24

BOTÕES — 6× direto no GPIO (SEM PCF8574), pull-up ou pull-down conforme o pino
  LEFT ..... GPIO 47  (limpo, pull-up 10 kΩ)
  SELECT ... GPIO 0   (strap seguro, pull-up 10 kΩ)
  UP ....... GPIO 3   (strap seguro, pull-up 10 kΩ)
  DOWN ..... GPIO 43  (UART0 RX — ⚠️ confirme que seu devkit não usa, ver nota)
  RIGHT .... GPIO 44  (UART0 TX — ⚠️ confirme que seu devkit não usa, ver nota)
  BACK ..... GPIO 46  (strap ⚠️ — pull-DOWN 10 kΩ + botão fecha pro 3.3V, ver nota)

LIVRES / RESERVA — GPIO 45 (strap, sobrou sem uso) · 19, 20 (USB nativo, intocado)
```

**⚠️ Botões 100% diretos no GPIO — sem PCF8574** (decisão do usuário: o módulo expansor não
cabia fisicamente no case). Isso força o uso de **5 dos 7 pinos que sobravam**, incluindo
3 de strap — cada um com uma ressalva diferente:

| Botão | GPIO | Por que esse pino | Fiação |
|---|---|---|---|
| LEFT | **47** | único 100% limpo que sobrava | pull-up 10 kΩ padrão (repouso HIGH, pressiona→GND) |
| SELECT | **0** | strap, mas pede HIGH no boot — igual ao repouso do pull-up padrão | pull-up 10 kΩ padrão |
| UP | **3** | strap (seleção de JTAG), mesma lógica do GPIO0 | pull-up 10 kΩ padrão |
| DOWN | **43** | UART0 RX — livre **só se seu devkit tiver 1 porta USB só** (nativa) | pull-up 10 kΩ padrão |
| RIGHT | **44** | UART0 TX — mesma ressalva do 43 | pull-up 10 kΩ padrão |
| BACK | **46** | strap, mas pede **LOW** no boot (VDD_SPI/ROM msg) — o **oposto** dos outros | **pull-DOWN** 10 kΩ (resistor pro GND) + botão fecha pro **3.3V** (não pro GND) |

**Antes de soldar DOWN/RIGHT (GPIO 43/44):** confira se seu devkit tem só **1 porta USB-C**
(a nativa do ESP32-S3, usada tanto pra gravar quanto pro Serial CLI). Se tiver **2 portas**
(uma nativa + uma via chip CP2102/CH340 separado), esse chip provavelmente já usa 43/44 por
dentro da placa — nesse caso esses 2 botões não têm pino seguro sobrando; a saída seria usar
o GPIO 45 (o único que ainda ficou de reserva) com a mesma fiação invertida do 46, e reduzir
pra 5 botões usando o outro.

**GPIO 45 fica de reserva** — não precisei arriscar os 2 piores strap ao mesmo tempo, só o 46.

**Pinos reservados do ESP32-S3 N16R8 — confirmado via datasheet (pra futuras expansões):**
| Faixa | Motivo | Pode usar? |
|-------|--------|------------|
| GPIO 26–32 | Flash SPI interno | ❌ nunca |
| GPIO 33–34 | Nem existem fisicamente no módulo WROOM-1 (não são pinos de fora) | ❌ não existem no seu módulo |
| GPIO 35–37 | PSRAM Octal interna — **todo módulo R8 (8MB PSRAM) tem isso reservado** | ❌ nunca no N16R8 |
| GPIO 19–20 | USB nativo (D-/D+) | ❌ intocado — precisa pro BadUSB por cabo |
| GPIO 43–44 | UART0 padrão (console serial) | ⚠️ usado nos botões DOWN/RIGHT — confirme seu devkit (nota acima) |
| GPIO 0, 3, 46 | Strap (boot mode/JTAG/VDD_SPI) | ⚠️ usado nos botões SELECT/UP/BACK — ver fiação na tabela acima |
| GPIO 45 | Strap (VDD_SPI) | Livre de reserva |

**O que mudou e por quê:**
- **PN532 continua em I2C** (SDA=48/SCL=7) — é o protocolo nativo do chip, não tem como
  contornar isso sem trocar de módulo. O que saiu foi só o **PCF8574** (expansor de botões):
  os 6 botões agora são **GPIO direto**, um por um, cada um com pull-up (ou pull-down no
  caso do GPIO46) individual — não tem mais "P0–P7" de expansor.
- Isso **liberou 5 GPIOs** que antes eram PREV(6)/NEXT(7)/SELECT(47) e o hack
  "GPS/NFC compartilhado, alterna por firmware" (8/42) — esse hack deixa de existir: GPS
  agora tem UART fixo e dedicado (RX 8, TX 42), sem precisar alternar nada em firmware.
- Dos 5 pinos liberados, sobrou um pra **bateria** (`BAT_PIN` = **GPIO 6**, ADC1 limpo, não é
  strap — resolve a seção 3.5 sem precisar mexer em mais nada).
- **LoRa descartado** (decisão do usuário — ver seção 3.4, marcada como não planejada). Dos
  5 pinos liberados, GPIO 6→bateria, GPIO 8→GPS RX e GPIO 42→GPS TX já têm dono; sobrava
  GPIO 7 e GPIO 47 — o **GPIO 7 virou SCL do PN532**, e o **GPIO 47 agora é o botão LEFT**.

Observações de fiação:
- **CC1101 e nRF24 compartilham SCK/MOSI/MISO (GPIO 12/11/13)**, cada um com seu CS. Com a
  chave seletora (seção 5.2), só um deles fica energizado por vez — some de vez a briga de
  barramento (init do CC1101, CS flutuante) descrita na seção 5.1.
- O **SD fica em barramento SPI separado** (GPIO 15/18/17/16) — evita os bugs de CS
  flutuante/init da seção 5.1.
- Todos os módulos vão em **3.3V** (o ESP32 é 3.3V; nada de 5V nas linhas de sinal).
- Alimente `nRF24 PA/LNA` e `GPS` por 3.3V bem filtrado (é onde o cap de 10 µF importa).

### 5.2 Chave seletora CC1101 ⇄ nRF24 (energiza um rádio por vez)

**Ideia:** os dois rádios continuam com SPI compartilhado (SCK 12 / MOSI 11 / MISO 13) e
com seus CS/controle fixos (CC1101: CS 10, GDO0 9 · nRF24: CSN 40, CE 39). A chave só
decide **qual dos dois recebe 3.3V**. Assim nunca há dois escravos energizados no mesmo
barramento → zero contenção elétrica, e o bug de ordem de init do CC1101 deixa de importar
(só existe o que está ligado).

**Chave:** 1× **SPDT** (deslizante ou gangorra, 3 terminais — "liga-liga", não "liga-desliga").
```
        3.3V (rail)
           │
        [comum]            chave SPDT
        /      \
   [pos. A]   [pos. B]
      │            │
  VCC CC1101   VCC nRF24
```
- **Posição A:** CC1101 ligado, nRF24 sem energia (Sub-GHz ativo).
- **Posição B:** nRF24 ligado, CC1101 sem energia (2.4 GHz ativo).
- GND dos dois rádios continua **sempre ligado** (comum) — a chave mexe só no VCC.

**Cuidados (com os componentes que você tem — 104, 10 µF, resistores):**
1. **Pull-up de 10 kΩ em cada CS** (CC1101 CS=10 e nRF24 CSN=40) → mantém o rádio
   desenergizado "deselecionado", pra ele não puxar o MISO compartilhado enquanto está sem
   VCC.
2. **Cada rádio mantém seu 104 (100 nF)** no VCC/GND, e o **nRF24 mantém o 10 µF dedicado** —
   como só um liga por vez, o 10 µF do nRF24 dá conta do pico dele sozinho.
3. **No firmware:** o Bruce vai detectar só o rádio energizado no boot. Se você **virar a
   chave com o aparelho ligado**, reinicie (ou entre/saia do menu de RF) pra ele redetectar
   — trocar a energia "por baixo" do firmware em runtime pode deixar o driver confuso até
   um novo `begin()`.

> Alternativa mais robusta (opcional, se sobrar uma chave DPDT): usar o 2º polo da DPDT pra
> **desconectar também o MISO** do rádio desligado do barramento — elimina qualquer resíduo
> de carga do pino MISO sem energia. Com SPDT + os pull-ups do item 1 já funciona na prática;
> a DPDT é só o "cinto e suspensório".

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

### Opção D — Módulo LoRa (seção 3.4)

| Loja | Produto | Obs. | Link |
|------|---------|------|------|
| AutoCore Robótica | **Módulo Transceptor Longo Alcance LoRa SX1276 433MHz** (RA-02) | SPI — o certo pro Bruce | /modulo-transceptor-longo-alcance-lora-sx1276-433mhz |
| SmartKits | **Módulo Transceptor LoRa 433MHz SX1278 com Antena** | SPI — o certo pro Bruce | /modulo-transceptor-lora-433mhz-sx1278-com-antena |

> ⚠️ As duas lojas também vendem a linha **Ebyte E32** (ex.: `E32-433T20D`,
> `E32-TTL-100`) — **não é esse**. O E32 é UART/AT-commands, sem os pinos SPI que o Bruce
> precisa (`NSS`/`MOSI`/`MISO`/`SCK`/`DIO0`/`RST`). Confirma que o produto lista esses
> pinos SPI antes de comprar.

## Fontes
- Bruce (repositório principal): https://github.com/pr3y/Bruce
- Bruce (org atual): https://github.com/BruceDevices/firmware
- Build de referência ESP32-S3 N16R8 + TFT 1,47" (pinout): https://github.com/arpitxp/Bruce-Smoochie-esp32
- Build ESP32-S3 alternativo: https://github.com/arpitxp/esp32-bruce
- Dispositivos suportados (DeepWiki): https://deepwiki.com/pr3y/Bruce/10.1-supported-devices
- Site oficial: https://bruce.computer/
