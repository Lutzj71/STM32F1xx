# grblHAL dla wypalarki laserowej (Creality V4.2.7)

Kontekst projektu przeniesiony z sesji prowadzonej w folderze `..\grbl` (stan na 2026-11-11, spisany 2026-11-16).

## Cel

Wypalarka laserowa 2-osiowa (X, Y, bez Z). Stare grbl 1.1h na ATmega328P (`..\grbl`) ma zostać zastąpione
przez grblHAL na płycie **Creality V4.2.7** (STM32F103RET6, z Endera 3 Pro).
Praca idzie etapami: firmware → panel dotykowy → obudowa.

## Firmware (ten folder)

Środowisko PlatformIO: `CREALITY_V427_LASER` w `platformio.ini`. Mapa pinów: `boards/creality_v4.2.x_map.h`.

- `BOARD_CREALITY_V4_2_7` (początkowo błędnie zakładano V4.2.2 – różnica: zamienione piny STEP/DIR), `board = genericSTM32F103RE`
- `USB_SERIAL_CDC=0` – komunikacja po UART (USART1, PA9/PA10)
- `MPG_ENABLE=2` – panel dotykowy na USART3 (PB10/PB11 = złącze LCD płyty), przełączanie znakiem 0x8B
- PWM lasera na PA1 (MOSFET grzałki hotendu), TIM5_CH2
- `VECT_TAB_OFFSET=0x7000` – firmware za bootloaderem Creality
- Ustawienia domyślne przeniesione ze starego `config.h`: tryb laserowy, bazowanie najpierw X, potem Y,
  `HOMING_FORCE_SET_ORIGIN`, max S = 1000, PWM 5 kHz

Wgrywanie: skopiować `.pio/build/CREALITY_V427_LASER/firmware.bin` na kartę SD (za każdym razem pod nową nazwą)
albo ST-Link pod adres 0x08007000.

**Stan: kompiluje się, nie było testowane na sprzęcie.**

## Otwarte ryzyka do sprawdzenia przy pierwszym uruchomieniu

- PWM lasera na PA1 jest odwrócony (open-drain, pull-up) – polaryzację ustawia się przez `$16`.
  Sprawdzić przed podłączeniem lasera.
- PC7 w mapie pinów to wykrywanie karty SD na tej płycie.

## Sprzęt

- Płyta (wymiary spisane jako V4.2.2, do potwierdzenia pomiarem na V4.2.7): 100×70,25 mm, otwory Ø3 w (20,40;3,37) (82,55;3,37) (2,22;38,20) (34,02;40,85) (97,46;67,71)
  liczone od lewego górnego rogu, slot SD na prawej krawędzi.
- Panel: Guition JC4880P443 (wg wiedzy modelu, do potwierdzenia z kartą katalogową: 4,3", 480×800 pion,
  ESP32-P4 + ESP32-C6, MIPI-DSI, dotyk pojemnościowy). Wymiary mechaniczne nieznane – do zmierzenia.
  Wcześniej błędnie zakładano Smart Panlee ZX3D50CE02S-USRC-4832 (3,5", 480×320, moduł 92×60 mm) –
  pod ten moduł powstała obecna obudowa i makieta UI, obie są do przerobienia.

## Powiązane materiały

- Obudowa (OpenSCAD: podstawa, pokrywa, klipsy): `..\laser_enclosure`, STL w `stl/`.
  OpenSCAD nie jest zainstalowany w systemie.
- Makieta UI dla panelu Guition (6 ekranów, 480×800 pion): https://claude.ai/artifact/QaSk7QgnDGDfL8GGXo8aZG
- Makieta UI dla Panlee (5 ekranów, 480×320) – zachowana wersja archiwalna, nie nadpisywać: https://claude.ai/artifact/AGSagpTbmMM1TQY1NAD6N7
- Stare grbl 1.1h: `..\grbl` (w `grbl/config.h` niezacommitowane zmiany bazowania z 2024 r.).
