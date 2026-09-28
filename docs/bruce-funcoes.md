# Funções do firmware Bruce — anotado pro seu hardware (ESP32-S3 N16R8)

Referência completa de tudo que o firmware [Bruce](https://github.com/pr3y/Bruce) faz, com
cada função marcada conforme o que **já funciona no seu build** (ver
[`bruce-esp32s3-n16r8-tft1.47.md`](./bruce-esp32s3-n16r8-tft1.47.md)), o que é **nativo do
chip ESP32-S3** (sem módulo extra) e o que **ainda falta hardware**.

**Seu hardware, resumo:** ESP32-S3 N16R8 (16MB flash / 8MB PSRAM) + TFT ST7789 172×320 +
CC1101 (Sub-GHz) + nRF24L01+ (2.4 GHz) + PN532 (NFC/RFID) + IR TX/RX + GPS NEO-6M + microSD
+ barra LED WS2812-8. LoRa (RA-02) e expansor de botões (MCP23017) recomendados, ainda não
confirmados como instalados.

Legenda: ✅ já funciona com o que você tem · 🔧 nativo do ESP32-S3, não precisa de módulo ·
🟡 precisa de módulo que você não tem ainda (ver observação) · ⚠️ limitação do chip/firmware.

---

## 📶 WiFi — 🔧 nativo do ESP32-S3 (rádio 2.4 GHz embutido, sem módulo)
| Função | O que faz |
|---|---|
| Scan / Connect | Escaneia redes, conecta em modo Station (STA) ou cria Access Point próprio |
| Deauth | Desautenticação — modo "Target" (uma rede) ou "Flood" (todas de uma vez) |
| Beacon Spam | Transmite SSIDs falsos em massa, polui a lista de redes por perto |
| Evil Portal | AP aberto com DNS/DHCP/web fake — página de login falsa que loga usuário/senha |
| EAPOL/Handshake Capture | Captura o handshake WPA de uma rede (quebra offline depois) |
| Karma Attack | Responde a probe requests fingindo ser rede já conhecida do cliente |
| Sniffer | Modo promíscuo, captura pacotes brutos (pcap) |
| Wardriving | ✅ **com seu GPS NEO-6M** — geolocaliza os escaneamentos, `.csv` pro WiGLE |
| Brucegotchi | Modo compatível com Pwnagotchi, automatiza captura de handshake + PwnGrid |
| WebUI | Hotspot com file manager web — upload de portais/scripts/configs/firmware |
| Ethernet | 🟡 precisa de um módulo PHY externo (ex. W5500 SPI ou LAN8720) — **você não tem** |

## 🔵 Bluetooth / BLE — 🔧 nativo do ESP32-S3, ⚠️ com uma ressalva importante
> **O ESP32-S3 só tem BLE (Bluetooth 5.0 Low Energy) — não tem Bluetooth Clássico (BR/EDR).**
> O ESP32 "clássico" (sem S) tem os dois. Praticamente todas as funções de BLE do Bruce usam
> só BLE mesmo, então isso não tira quase nada — é só bom saber que não dá pra fazer nada que
> dependa de BT Clássico (ex.: alguns ataques antigos de headset/A2DP não se aplicam aqui).

| Função | O que faz |
|---|---|
| BLE Scan | Reconhecimento passivo de dispositivos BLE próximos |
| Advertisement Spam | Popups falsos em massa: AppleJuice/SourApple (iOS), Swift Pair (Windows), Samsung, Fast Pair (Android) |
| iBeacon Spoof | Transmite beacons iBeacon customizados |
| Ninebot BLE Tuning | Interage via BLE UART com patinetes Ninebot/Xiaomi (hoje só destrava velocidade máx.) |
| Bad BLE | Vira teclado Bluetooth, injeta Ducky Script sem fio |
| BLE Security Suite | Ver detalhamento abaixo — só na versão **Full** (não vem na Lite) |
| **BLE Tracker Detector** ✅ | Menu **Bluetooth > BLE**. Detecta AirTag/Samsung SmartTag/Tile/óculos smart (Meta Ray-Ban) por perto, com estimativa de distância (perto/médio/longe pelo RSSI) e **alerta de "seguindo você"**: o card do tracker fica laranja depois de um tempo por perto e vermelho se continuar — pensado pra detectar stalking. Mergeado em **23/set/2026** — **precisa de firmware compilado depois dessa data**, se não aparecer no menu é só atualizar |

### BLE Security Suite, detalhado
Plataforma de teste de segurança BLE só na versão **Full** do Bruce (não existe na Lite):
- **Quick Vulnerability Scan** — testa vulnerabilidades específicas conhecidas (ex.: HFP
  CVE-2025-36911, falhas de FastPair) num dispositivo alvo.
- **Deep Device Profiling** — enumera todos os serviços/characteristics GATT do alvo (perfil
  completo do que o dispositivo expõe por BLE).
- **FastPair Suite / HFP Suite / Audio Suite / HID Suite** — conjuntos de ataque específicos
  pra cada perfil BLE (pareamento rápido Google/Android, handsfree de fone/headset, áudio,
  emulação de teclado).
- **Ataques encadeados** — tenta automaticamente HFP → HID → FastPair, conforme os serviços
  que o alvo expõe, sem precisar escolher manualmente qual ataque tentar primeiro.
- Também inclui DoS avançado e entrega de payload.

> ⚠️ **Detector de skimmer Bluetooth (leitor de cartão clonado escondido) ainda NÃO existe no
> Bruce** — é só um pedido de feature em aberto (issues #1088/#1224), diferente do Tracker
> Detector acima. Não confie que essa função já está disponível.

## 📻 Sub-GHz — ✅ com seu CC1101
| Função | O que faz |
|---|---|
| Scan/Copy | Captura sinal ASK/OOK — modo Decode (RCSwitch) ou RAW (sinal bruto) |
| Custom SubGHz | Envia sinal customizado salvo em arquivo |
| Replay | Retransmite um sinal já capturado |
| Jammer | Interferência na frequência sintonizada |
| Spectrum | Analisador de espectro + leitura de RSSI |
| **Bruteforce** | Gera e transmite sequências de código automaticamente (`rf_bruteforce.cpp`), baseado em templates de protocolo — mira portões/garagens antigas de **código fixo** (sem rolling code/criptografia). Pode usar **sequência de De Bruijn** pra cobrir todas as combinações possíveis sem repetir código à toa, bem mais rápido que testar um por um |

## 📡 2.4 GHz — ✅ com seu nRF24L01+
| Função | O que faz |
|---|---|
| Spectrum | Visualiza atividade na banda 2.4 GHz inteira |
| Jammer | Interferência (WiFi, Bluetooth, protocolos proprietários 2.4 GHz) |
| Mousejack | Sniffing de mouse/teclado wireless vulnerável (dongle não criptografado) |

## 📶 LoRa — ❌ descartado (decisão do usuário, não vai entrar no build)
| Função | O que faz |
|---|---|
| Chat | Texto ponto-a-ponto de longo alcance, só entre dispositivos Bruce (não interopera com Meshtastic/MeshCore/LoRaWAN) |

## 🔴 Infravermelho — ✅ com seu IR TX/RX
| Função | O que faz |
|---|---|
| TV-B-Gone | Sequência de códigos "desligar TV" de várias marcas (bancos NA/EU) |
| Custom IR | Envia código IR customizado de arquivo (compatível com Flipper-IRDB) |
| IR Read | Recebe/decodifica sinal IR de controle remoto pra gravar/reusar |

## 💳 RFID/NFC — ✅ com seu PN532
| Função | O que faz |
|---|---|
| TagOMatic | Leitor/gravador unificado (abstrai PN532/RC522/M5 RFID2) |
| Leitura/Dump | ISO14443A (MIFARE Classic/Ultralight) + FeliCa — salva UID/SAK/ATQA + dump em `.rfid` |
| Clonagem/Gravação | Escreve num tag (precisa de tag gravável no Bloco 0 pra clonar UID) |
| Emulação | O Bruce finge ser o próprio tag pro leitor externo |
| PN532 BLE/UART | Usa o PN532 como leitor externo controlado por outro app/celular |

## ⌨️ BadUSB & HID
| Função | O que faz | Hardware |
|---|---|---|
| BadUSB (USB) | Injeção de teclado via cabo, roda Ducky Script | ✅ **o ESP32-S3 tem USB-OTG nativo** — não precisa do chip CH9329 que outras placas (ESP32 clássico) exigem. Só plugar o cabo USB-C no alvo. |
| Bad BLE | Mesma injeção de Ducky Script, sem fio via Bluetooth | 🔧 nativo (BLE) |

## 🧠 JS Interpreter — 🔧 nativo (roda no firmware, sem hardware extra)
Motor Duktape (JS ES5) embarcado, com API própria (BJS) que acessa WiFi, RF, display, input
e hardware por script — automatiza combinações de tudo acima. Scripts `.js` na pasta
`/scripts`, aparecem no menu **Scripts**. É a base de um monte de **apps/jogos feitos pela
comunidade** que rodam em cima do interpretador (não são parte do firmware "core", mas
funcionam em qualquer Bruce com o Interpreter ativado): Tetris (BruceBlocks), Snake, um
shooter vertical, um "bichinho virtual" (tamagochi) e até um port de DOOM.

## 🖥️ Controle remoto / CLI — 🔧 nativo (WiFi + USB, sem módulo extra)
| Função | O que faz |
|---|---|
| Serial CLI | Terminal de comandos via USB (`ir`, `subghz`, `led`, `gpio`, `i2c`, `badusb`, `js`, `crypto`, `storage`, `settings`, `webui`, entre ~20 comandos) — maioria compatível com a CLI do Flipper Zero. Dá pra automatizar/rodar sem tela |
| WebUI | Além do file manager (já citado em WiFi): roda comandos serial, **vê a tela do dispositivo ao vivo pelo navegador**, e dispara payloads de IR/RF/BadUSB remotamente |
| App companion (PC/celular) | App oficial multiplataforma — grava firmware, roda comandos serial e espelha a tela do Bruce ao vivo (**Screen Mirror**), tudo num lugar só |

## 🔊 Áudio (buzzer/alto-falante) — 🟡 precisa de módulo (você não tem)
| Função | O que faz |
|---|---|
| Music Player / Tone | Toca melodias/tons pelo buzzer ou alto-falante do dispositivo |

Só funciona em placas com buzzer/speaker embutido, ou se você **adicionar um buzzer
passivo/piezo** num GPIO livre — não está na sua BOM atual.

## 🎙️ Microfone — 🟡 precisa de módulo (você não tem)
| Função | O que faz |
|---|---|
| Mic Spectrum | Espectro de áudio em tempo real do microfone embutido |
| Gravação | Grava áudio do mic, salva `.wav` |

Isso só existe em placas com microfone I2S já embutido (ex. M5Stack CoreS3/Cardputer). No
seu build custom, só funciona se você **adicionar um módulo de microfone I2S** (ex. INMP441)
— não está na sua BOM atual.

## 📱 QR Codes — 🔧 nativo (só usa o display)
Gera QR Code de URL customizada ou de **PIX** (pagamento instantâneo brasileiro).

## 📂 Arquivos / Armazenamento — ✅ com seu microSD
| Função | O que faz |
|---|---|
| File Manager | Navega SD card e LittleFS (memória interna), opera entre os dois |
| Mass Storage | Vira pendrive USB pro PC acessar o SD/LittleFS direto — ⚠️ bug conhecido: não alterna bem com o modo BadUSB sem reiniciar/deep-sleep no meio |

## 🌍 GPS — ✅ com seu NEO-6M
Wardriving (geolocaliza os escaneamentos de WiFi/BT) e tela de GPS Tracker (info/posição).

## ⚙️ Config/Others — 🔧 nativo
Clock (NTP, fuso, DST, 12h/24h), UI Theme / UI Color (tema Flipper que já configuramos), **Boot
Animation** (animação de abertura customizável, mesmos repositórios de tema da comunidade), LED
(efeitos da barra WS2812 que já configuramos), além de toggles pra Sniffer, Custom SubGHz,
PN532 BLE/UART.

---

## Resumo — o que falta pro seu build ter a lista completa
| Falta | Pra que serve | Prioridade |
|---|---|---|
| **LoRa RA-02** | Único jeito de decodificar sinal LoRa + Chat longo alcance | Já recomendado (seção 3.4 do doc de hardware) |
| **Expansor MCP23017** | Liberar GPIO pros 6 botões + já compartilha barramento do PN532 | Já recomendado (seção 3.3) |
| **Microfone I2S (ex. INMP441)** | Mic Spectrum + gravação de áudio | Opcional, função menor |
| **Módulo Ethernet (W5500/LAN8720)** | Conectividade cabeada | Opcional, raramente necessário num build portátil |

Tudo o mais (WiFi, BLE, Sub-GHz + Bruteforce, 2.4GHz, IR, NFC, BadUSB/Bad BLE, JS Interpreter,
QR Code, Arquivos, Serial CLI/WebUI/App companion) **já funciona no seu hardware atual**, sem
precisar de nada extra.

## Correção da versão anterior deste doc
- **Faltava o Bruteforce de Sub-GHz** (código fixo, De Bruijn) — adicionado na tabela de Sub-GHz.
- **Faltavam**: Serial CLI, WebUI (parte de controle remoto), App companion (Screen Mirror),
  Music Player/Tone, Boot Animation, e os apps/jogos comunitários do JS Interpreter.
- **BLE Tracker Detector** (AirTag/SmartTag/Tile) — confirmado **mergeado em 23/set/2026**,
  menu Bluetooth > BLE. Atualize o firmware se não aparecer.
- **Skimmer detector Bluetooth NÃO existe** ainda — é só pedido de feature em aberto, não
  confunda com o Tracker Detector acima (são coisas diferentes).

## Fontes
- Features and Capabilities (DeepWiki): https://deepwiki.com/pr3y/Bruce/1.1-features-and-capabilities
- WiFi Features: https://deepwiki.com/pr3y/Bruce/3-wifi-features
- BLE Features: https://deepwiki.com/pr3y/Bruce/4-ble-features
- RF Features (SubGHz + NRF24 + LoRa): https://deepwiki.com/pr3y/Bruce/5-rf-features
- IR Features: https://deepwiki.com/pr3y/Bruce/6-ir-features
- RFID Features (TagOMatic): https://deepwiki.com/pr3y/Bruce/7-rfid-features
- BadUSB and HID Emulation: https://deepwiki.com/pr3y/Bruce/8-badusb-and-hid-emulation
- JS Interpreter / BJS API: https://github.com/BruceDevices/firmware/wiki/Interpreter
- Serial Commands: https://github.com/BruceDevices/firmware/wiki/Serial
- WebUI: https://wiki.bruce.computer/controlling-device/webui/
- BLE Tracker Detector (PR comunitário): https://github.com/BruceDevices/firmware/pull/2915
- Skimmer detector (feature request, não implementada): https://github.com/BruceDevices/firmware/issues/1224
- Wiki oficial: https://wiki.bruce.computer/
- Repositório: https://github.com/pr3y/Bruce
