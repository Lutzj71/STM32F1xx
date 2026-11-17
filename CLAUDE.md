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

- Płyta (zdjęcie potwierdza V4.2.7 i STM32F103RET6; wymiary spisane z rysunku V4.2.2 – na zdjęciu otwory
  (20,40;3,37) i (34,02;40,85) wypadają ok. 12–15 mm dalej od krawędzi z zasilaniem, do zmierzenia): 100×70,25 mm, otwory Ø3 w (20,40;3,37) (82,55;3,37) (2,22;38,20) (34,02;40,85) (97,46;67,71)
  liczone od lewego górnego rogu, slot SD na prawej krawędzi.
- Panel: Guition JC4880P443C_I_W (4,3", 480×800 pion, ESP32-P4 + ESP32-C6, ST7701S, dotyk pojemnościowy,
  5 V / ok. 320 mA). Ze specyfikacji producenta: obrys modułu 117,01×69,41 mm, szkło 114,40×66,80 mm,
  pole aktywne 93,60×56,16 mm. Z pomiaru użytkownika: otwory montażowe M2 w rozstawie 60×108 mm,
  słupek 4,9 mm, część z elektroniką 10,3 mm, grubość całości 13,66 mm.
  Ze zdjęcia modułu: tulejki M2 są w narożnikach plastikowej ramy; opis przy CN2 to „5V TX0 RX0 GND”;
  CN2, dwa gniazda USB-C i listwa 2×13 są przy jednym krótszym boku, taśma ekranu przy drugim.
  Złącza wg schematu społecznościowego (github.com/ultramcu/guition-jc4880p443c-i-w, do potwierdzenia
  na płytce): CN2 „INPUT” MX1.25 4-pin = 1 VIN 5 V, 2 TX0 (GPIO37), 3 RX0 (GPIO38), 4 GND;
  JP1 2×13 = wolne GPIO 28–33, 49–52 (34/35 to piny startowe), 5 V na JP1 jest wyjściem, nie wejściem.
  Wcześniej błędnie zakładano Smart Panlee ZX3D50CE02S-USRC-4832 (3,5", 480×320, moduł 92×60 mm).

## Powiązane repozytoria

- Firmware panelu (ESP-IDF, LVGL): `..\guition_panel` – https://github.com/Lutzj71/guition_panel
  (7 ekranów, strumieniowanie z karty SD, Wi-Fi; niezbudowane – brak ESP-IDF; kontekst w jego `CLAUDE.md`).
  Łącze: UART0 panelu (CN2, GPIO37/38) ↔ USART3 tej płyty (PB10/PB11), 115200.
  Do sprawdzenia po stronie grblHAL: czy przy `MPG_ENABLE=2` raport statusu zawiera pole `MPG:`
  i czy płyta odpowiada na USART3 poza trybem MPG.
- Obudowy: `..\laser_enclosure` – https://github.com/Lutzj71/laser_enclosure (kontekst w jego `CLAUDE.md`).

## Powiązane materiały

- Obudowy (OpenSCAD) w `..\laser_enclosure`, obie na profil 2040, STL w `stl/`, opis w `README.md`:
  `obudowa_plyta.scad` (płyta główna + wentylator 92 mm; płyta przy ścianie z SD/USB, 15 mm na przewody
  z pozostałych stron, wyjście przewodów naprzeciw SD; wisi OBOK profilu stojącego na krótszym boku (nie na nim),
  przy bocznej ścianie 40 mm, mocowana wsuwkami w obu bocznych rowkach), `obudowa_ekran_guition.scad` (ekran pochylony 45°, też OBOK profilu, zwrócony wzdłuż profilu;
  moduł przykręcony M2 bezpośrednio do spodu obudowy, bez belek; pokrywa górna z oknem na pole aktywne).
  Stara wspólna obudowa z ekranem Panlee (`obudowa.scad`, `README_panlee.md`) zachowana – nie nadpisywać.
  OpenSCAD nie jest zainstalowany w systemie (STL generowano przenośną wersją 2021.01 z katalogu tymczasowego).
- Makieta UI dla panelu Guition (6 ekranów, 480×800 pion): https://claude.ai/artifact/QaSk7QgnDGDfL8GGXo8aZG
- Makieta UI dla Panlee (5 ekranów, 480×320) – zachowana wersja archiwalna, nie nadpisywać: https://claude.ai/artifact/AGSagpTbmMM1TQY1NAD6N7
- Stare grbl 1.1h: `..\grbl` (w `grbl/config.h` niezacommitowane zmiany bazowania z 2024 r.).
