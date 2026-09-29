# XeWe LED - Wardrobe — smart wardrobe lighting on an ESP8266

Personal project · 2024 · Solo: Max Dokukin · Status: Completed

![Controller board: ESP8266 module, level-shifter board and regulator on perfboard](static/media/resources/IMG_9684.webp)

## Overview

Smarthome Lights ESP — Wardrobe is ESP8266 firmware that connects an addressable LED strip inside a wardrobe to the Apple HomeKit and
Amazon Alexa smart home systems. The LED has two modes: solid color, and Perlin fade (slow noise-driven color waves around the chosen hue).
Every change — on/off, brightness, color, mode — is eased over 900 ms instead of jumping, and the last state is saved to EEPROM so the lights
come back the way they were after a power cut. Both voice/app ecosystems stay in sync: a change made through one is pushed back to the other.

## Highlights

- One device, two ecosystems: a HomeKit bridge with a color lightbulb and a "Wardrobe Lights Fade" switch, plus two Espalexa devices with the
  same names — `my_accessory.c`, `HomeKit.h`, `Alexa.h`
- Smooth transitions: 900 ms color cross-fade, 900 ms brightness ramp and 900 ms hue glide in Perlin mode — `LedController.h`, `PerlinFade.h`
- Perlin mode maps FastLED `inoise8` over LED index and time onto a hue band around the chosen color (hue gap 10000 of 65535, saturation 245–255,
  brightness 100–255), narrowing the band to 20% near red — `PerlinFade.h`
- Persistent state: 6 bytes (R, G, B, brightness, mode, on/off) in EEPROM, restored at boot — `MemoryController.h`
- Non-blocking main loop on a 10 ms frame, 906 lines of C/C++ in 9 files

## How it works

```
HomeKit (arduino_homekit_server) ─┐                       ┌→ EEPROM (6 bytes)
                                  ├→ LedController setters ┤
Alexa (Espalexa) ─────────────────┘                       └→ valuesUpdated → sync both ecosystems
loop() every 10 ms → LedController.frame() → SOLID_COLOR | SOLID_COLOR_TRANSITION | PERLIN → brightness ramp → NeoPixel show()
```

- **`Smarthome-Lights-ESP-Wardrobe.ino`** — setup (serial 115200, EEPROM 4096 bytes, WiFi, LedController, HomeKit, Alexa) and the main loop;
  strip of 10 LEDs on pin D6, `NEO_GRB + NEO_KHZ800`.
- **`LedController.h`** — state machine for mode and on/off, HSV/RGB setters, color cross-fade, brightness ramp, EEPROM writes, and the
  `valuesUpdated` flag that triggers two-way sync.
- **`PerlinFade.h`** — the Perlin-noise animation and its hue glide.
- **`HomeKit.h` + `my_accessory.c`** — HomeKit accessory database (bridge, lightbulb with on/brightness/saturation/hue, fade switch) and setters.
- **`Alexa.h`** — Espalexa color device "Wardrobe Lights" and on/off device "Wardrobe Lights Fade".
- **`MemoryController.h`** — byte-level EEPROM storage at a configurable offset (1500 in this sketch).
- **`Color.h`** — RGB↔HSV conversion, color mixing, channel extraction. **`enums.h`** — modes and states.

In the installed wardrobe, the strip is 12 V addressable LED, the 3.3 V data signal is shifted to 5 V on a small level-shifter board, and two
magnetic sensors on the sliding doors fade the lights in on open and out on close (see the demo video in `static/media/resources/demo.mp4`).
That door-sensor logic is not part of the firmware in this repository, which is controlled through HomeKit and Alexa only.

## Results

| Metric | Value | Note |
|---|---|---|
| Smart home integrations | 2 (Apple HomeKit, Amazon Alexa) | kept in sync after every change |
| Light modes | 2 (solid color, Perlin fade) | plus an internal cross-fade state |
| Transition time | 900 ms | color, brightness and Perlin hue |
| Frame period | 10 ms | non-blocking `millis()` scheduling |
| Persisted state | 6 bytes in EEPROM | restored at boot |
| Firmware size | 906 lines, 9 files | `wc -l`, excluding the WiFi header |

The project is a working home installation rather than an experiment, so its results are the features the firmware delivers.

## Getting started

```text
1. Arduino IDE with the ESP8266 board package; select a NodeMCU / ESP-12E board.
2. Install the libraries the sketch includes: Adafruit NeoPixel, FastLED (for inoise8), Espalexa,
   and Arduino-HomeKit-ESP8266 (arduino_homekit_server.h).
3. Provide wifi_info.h with a wifi_connect() function for your network (it is included by the sketch).
4. Set LED_PIN / LED_NUM in Smarthome-Lights-ESP-Wardrobe.ino for your strip, then upload.
5. Add "XeWe Lights Wardrobe" in the Apple Home app with the setup code from my_accessory.c,
   and ask Alexa to discover devices.
```

Library versions are not pinned in the repository. Serial debug output runs at 115200 baud (`LED_CONRTROLLER_DEBUG` in `LedController.h`).

## Documents

- [Demo video](static/media/resources/demo.mp4) · [Controller board, top](static/media/resources/IMG_9684.webp) · [Controller board, solder side](static/media/resources/IMG_9686.webp)
- Related: XeWe LED OS — [project page](https://maxdokukin.com/projects/xewe-led-os) · [github.com/xewe-labs/xewe-led-os](https://github.com/xewe-labs/xewe-led-os)
- Project page: [maxdokukin.com/projects/xewe-led-wardrobe](https://maxdokukin.com/projects/xewe-led-wardrobe)
