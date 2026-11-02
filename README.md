# CYBER // HOUND

A defensive RF multi-tool for the **Elecrow ThinkNode M9** (ESP32-S3 + LR1110 + GNSS + keyboard).
It has the same idea as HaleHound but faces the other way: it **only listens and never transmits**
on Wi-Fi, BLE or LoRa. It tells you when someone else is up to something.

## Apps

| # | App | What it does |
|---|-----|--------------|
| 1 | **Spectrum** | LR1110 RSSI sweep with a live waterfall and peak hold. Ranges: EU SRD 863–870, ±500 kHz around your mesh frequency, 865–868.6 and US 902–928. The dotted green line is your MeshCore frequency. |
| 2 | **Detector** | Wi-Fi: deauth/disassoc floods (shows the source MAC and whether frames are broadcast), evil twins (the same SSID both open and encrypted), karma APs (one BSSID beaconing 4+ SSIDs) and beacon floods. BLE: spam floods (Apple popups, Samsung, Swift Pair, Fast Pair) counted by unique sender. Mesh: name clashes, replayed adverts, noisy nodes. |
| 3 | **Tracker hunt** | Finds AirTag/Find My (* = separated from owner), SmartTag, Tile, Chipolo, Google Find My Device and DULT tags. A tag is flagged **FOLLOW** when it's seen at 3+ places at least 250 m apart over 10+ minutes. Without GPS it falls back to 20+ minutes nearby. Press ENTER to hunt it with a Geiger-style beep and an RSSI meter. Press `A` to show all BLE devices. |
| 4 | **Cameras** | Wi-Fi cameras and doorbells: Ring, Arlo, Nest, Wyze, Hikvision, EZVIZ, Dahua/Imou, Reolink, Uniview, Axis, Amcrest, Lorex, Swann, SkyBell, Vivint and more (197 IEEE OUIs). Cameras are Wi-Fi *clients*, so the Hound also sniffs data frames, not just beacons. Setup networks and base stations (Ring-xxxx, EZVIZ_, Arlo...) are matched by SSID. **LIVE** = more than 15 frames/s, so it's probably streaming or recording. OK starts a Geiger-style locate. Logged to `/hound/cams_NNNN.csv`. Limit: brands built on generic Wi-Fi chips (many Blink, eufy and Tapo models) can't be told apart by MAC. |
| 5 | **Room sweep** | Hidden-camera / bug check for hotels and rentals. While it runs, **every** Wi-Fi client that sends data is tracked, plus Bluetooth devices. After 15 s it focuses on the busiest nearby channels. Each device gets a score: camera brand (+40), streaming upload (+35, since video goes *up*), very close (+20), Bluetooth names like cam/ipc (+35), trackers (+20); randomised phone MACs (−15) and known devices (−60) count against it. The hot/cold bar and Geiger beeps follow the top candidate. OK finds the direction of the selected one, K marks it known, S saves a report to `/hound/sweep_NNNN.txt`. |
| 6 | **Drones** | EU/ASTM **Remote ID** receiver: Wi-Fi beacon, Wi-Fi NAN, Bluetooth 4 and Bluetooth 5 long range. A radar centred on you shows each drone (triangle pointing its heading) and its **pilot** (square), with auto range. The detail page has the serial/registration, operator ID, EU category and class, height, speed, and distance and bearing to the drone and the pilot. Alerts when a new drone shows up. Track logged to `/hound/drones_NNNN.csv`. Decoding uses the official OpenDroneID library (Apache 2.0, see `LICENSE-opendroneid`). Drones without Remote ID (older or home-built) aren't visible. |
| 7 | **LoRaWAN** | Passive EU868 uplink sniffer. Hops over the 8 standard channels (867.1–868.5 MHz) and SF7–SF12, 1.5 s per step; `S` locks a spreading factor, `C` a channel. **DEVICES**: DevAddr, network (TTN 26/27xx, Helium 78/79xx, private, or NetID type), frame counter, port, RSSI, SF. **JOINS**: DevEUI + JoinEUI of devices joining (the first 3 bytes of a DevEUI are usually the maker's OUI). **STATS**: uplinks per minute and per network/SF. Payloads stay encrypted. It's one channel at a time, so it samples traffic and doesn't catch all of it. Takes over the LoRa radio while open; MeshCore resumes when you leave. |
| 8 | **Mesh health** | Passive MeshCore sniffer. **NODES**: who's advertising, with type, hops and RSSI/SNR for direct nodes. **LINKS**: the last-hop repeaters you actually hear, with link quality. **STATS**: packets per minute, airtime %, packet-type mix and a 60 s histogram. A green **RX** in the header blinks on every LoRa packet. |
| 9 | **Wardrive** | WiGLE CSV 1.4 logging (Wi-Fi + BLE, only with a GPS fix) with unique-network counter, odometer, speed and a feed of new networks (open = green). While driving (> 5 km/h) devices are logged again every 30 s so WiGLE can triangulate them; standing still, every 5 min. `W` or RIGHT opens **WiGLE upload**. |
| R | **Room watch** | Leave the M9 in your hotel room: OK arms it, you get 60 s to leave, then it takes a baseline of all Wi-Fi and Bluetooth devices around. While armed it logs when it's **moved** (QMI8658 motion sensor) and every new device that shows up close by (stronger than −65 dBm). OK disarms it and writes a timeline to `/hound/roomwatch_NNNN.txt`. Silent by default; `S` switches beeping alerts on. |
| G | **GNSS guard** | GPS jamming and spoofing monitor. It reads every satellite's signal strength (C/N0) from the receiver. **Jamming**: all signals collapse at once (≥12 dB under the learned baseline, ≤3 satellites left). **Spoofing**: position jumps faster than an airliner, altitude jumps, the GNSS clock jumping against the M9's own clock, or all satellites suspiciously equally strong. Shows a 5-minute C/N0 history and a bar per satellite (GPS/GLONASS/Galileo/BeiDou). Events go to alerts and `/hound/events.csv`. |
| S | **Satellites** | Pass predictions (SGP4) for the ISS, the Chinese space station, Hubble, NOAA 15/19 and Meteor-M2 3/4 for the next 48 h, in local Dutch time; "eye" = sunlit while you're in the dark, so you can see it. OK opens a live sky view with the pass track, rise/set countdown and an arrow (LEFT/RIGHT, look up X°). `U` downloads fresh orbits from celestrak.org over your home Wi-Fi; they're also refreshed automatically with each WiGLE upload when older than 3 days. For the arrow the compass needs north: face north and press `N`, or just walk a few steps with the M9 pointing ahead. |
| W | **Web dashboard** | For phone or laptop. **Own hotspot** (`CyberHound-XXXX`, works anywhere, no map tiles) or **home Wi-Fi** (map with all hits on OpenStreetMap). Scan the QR code on the M9 with your phone camera to join the hotspot without typing, then scan the second QR to open the page. Login: user `hound` + the PIN on the screen, which is new every session. The screen shows live hotspot events (phone joined / got IP / left + reason). Tabs: status + map, alerts, trackers, cameras, drones, mesh, GNSS, satellites, room watch, **files** (download/delete everything on the SD card, upload e.g. `tle.txt`) and **firmware update** (disabled under Launcher, because it could overwrite another app). Wi-Fi sniffing **and** the Bluetooth scanner pause while it runs: they share the one 2.4 GHz radio, and the scanner starved the hotspot handshake in v1.6. |
| T | **Meshtastic** | Passive Meshtastic watcher (EU868 presets: LongFast default, MediumFast/Slow, ShortFast/Slow, LongModerate, Narrow and Lite). Decodes everything on the default public channels: **NODES** with name, hardware, role, hops, RSSI, battery and distance (from their shared position); **CHAT** with public text messages; **STATS** with packet counts, airtime and which channel hashes are in use. `P` switches preset. Private channels and DMs are counted but stay encrypted. The radio is shared with MeshCore: Settings → *LoRa listen* picks MeshCore, Meshtastic or alternate (60 s each); opening this app or Mesh health tunes to that network while it's open. Logged to `/hound/mtastic_NNNN.csv`. |
| - | **Alerts** | Everything that triggered, with its age. A banner and beep show up on any screen. |
| 0 | **Settings** | Sound, logging, WiGLE, known devices, compass calibration, mesh preset + manual SF/BW/CR/frequency, alarm thresholds, screen flip, screen-off timeout, buttons. |

### Direction finding (compass)

Available from tracker **Hunt** (`D`), camera **Locate** (`D`) and **Room sweep** (OK).
Hold the M9 flat in front of your chest and turn slowly on the spot. Your body blocks
signals from behind, so the signal is strongest when you face the transmitter. The Hound bins
signal strength by compass heading and fits a curve. The magenta arrow points to the peak,
relative to where you're facing ("LEFT 40", "AHEAD"). Walk that way, press `R` and turn again
to refine. For Wi-Fi cameras the radio locks to the camera's channel for a steady stream of readings.

The first time, the compass needs a 30-second calibration: turn the M9 in every direction,
then turn yourself a quarter turn to the right so it learns left from right. Settings → Compass
redoes it. A simulation with 3–5 dB noise lands within about 10–30°.

### Known devices

Mark your own stuff (your Ring, your keys' tag, the neighbours' cameras) as **known**:
`K` in the tracker, camera or sweep list, or Settings → Known devices → `L` to learn every
tracker and camera around you right now (do it at home). Known devices are dimmed and never raise
an alert. `X` forgets one. Also new: Find My devices that are *near their owner* never raise
**FOLLOW** (Apple's own anti-stalking logic only warns for separated tags).

### WiGLE upload

Settings → *WiGLE upload*, or `W` on the Wardrive screen. Fill in:
- **Home Wi-Fi** and its **password**.
- **API name** and **API token**: wigle.net → Account → *API Token*. Use the pair shown under "Encoded for use", not your WiGLE password.

**Upload now** pauses the sniffers, joins your Wi-Fi and POSTs every finished `wigle_*.csv` to
`api.wigle.net` over HTTPS, with the certificate checked against the ESP-IDF root CA bundle.
Your current drive is closed first, so it's included. Uploaded files move to `/hound/sent/`.
With **Auto upload at home** on, this happens by itself when your home network is in range,
at most once every 10 minutes and only when there's something new.
The credentials are stored in the M9's flash (NVS), not encrypted.

### Mesh presets

Default is **NL (SF7)**: 869.618 MHz, BW 62.5, SF7, CR 4/5. The Dutch MeshCore community moved
to this in 2026. Also available: EU/UK narrow (SF8/CR8), EU/UK medium/wide, US narrow/wide, or set
SF/BW/CR/frequency by hand (frequency: L/R ±25 kHz, `<` `>` ±1 kHz).

## Buttons

On first boot a wizard asks you to press each front button once (SPACE skips a button).
The bottom line of the wizard shows live keyboard diagnostics: chip address, I2C bus, INT level,
register reads and errors, and the last register value. If nothing registers, send that line to Geekaboo.
With no key press for 60 s the wizard steps aside and uses the defaults.
It saves the codes, so the mapping matches your M9's keyboard firmware.
To redo it, go to Settings → Buttons, or press `L` on the button help screen.

| Button | Does |
|---|---|
| UP / DOWN | move in lists; on Spectrum, change the frequency range |
| LEFT / RIGHT | switch tabs (Mesh); move the cursor (Spectrum); change a value (Settings) |
| OK (centre) / ENTER | open, select, toggle; start a hunt from the tracker list; start/pause logging |
| BACK / DEL / `q` | back one screen (from Hunt back to the tracker list) |
| HOME | main menu from anywhere |
| MENU / `?` / `h` | button help + key tester: shows each code and lights up the button you press |
| CHAT | Alerts |
| MAP | Mesh health |
| POSITION | Wardrive + GNSS |
| `1`–`9`, `0`, `R`, `G`, `S`, `W` | jump to an app from the main menu (`0` = Settings; Alerts is on CHAT) |
| `K` | mark / unmark the selected tracker, camera or sweep device as known |
| `D` | direction finding from Hunt / Locate |
| any key while the screen is off | wakes it (the key does nothing else) |
| `P` / `R` | Spectrum: clear peaks / next range |
| `A` | Trackers: show all BLE devices or only trackers |
| `S` / `C` | LoRaWAN: lock or unlock spreading factor / channel |
| `M` | Hunt: mute or unmute the Geiger beeps |

Every key press is printed on the serial monitor as `[key] raw 0x.. -> NAME`.

## Flashing

**Ready-made image:** flash `CyberHound_M9_v1.6.1_full.bin` at offset `0x0`:

```
esptool.py --chip esp32s3 --baud 921600 write_flash 0x0 CyberHound_M9_v1.6.1_full.bin
```

Or use the web flasher at https://espressif.github.io/esptool-js/ (offset 0x0). If the M9 doesn't
show up, hold BOOT while you plug in USB.

**Dual boot with wadamesh (or MeshCore) via Launcher:**
1. Flash **Launcher** 2.10.0 or newer (it supports the ThinkNode M9) from https://bmorcelli.github.io/Launcher/
2. Copy `CyberHound_M9_v1.6.1_app.bin` (app-only, **not** the `_full` file) and the wadamesh app-only
   `.bin` (not `-merged`) from its GitHub releases onto the SD card.
3. In Launcher, install both from the SD card.
4. At power-on, Launcher shows its start screen. Press the number key of an app to boot it straight away,
   press Enter for the Launcher menu, or wait and the last app starts.
5. From Cyber Hound: Settings → *Restart / switch firmware* takes you back to the Launcher screen.

**Back to MeshCore:** flash the ThinkNode M9 firmware from https://flasher.meshcore.co.uk.
Save your MeshCore keys and contacts first if you want to keep them.

**From source (PlatformIO):** run `pio run -t upload` in this folder.

**From source (Arduino IDE / arduino-cli):** use ESP32 core 3.3.x, board *ESP32S3 Dev Module*,
Flash 16MB, PSRAM *OPI*, partition *16M Flash (3MB APP/9.9MB FATFS)*, USB CDC on boot *Disabled* (USB-C is a CH340 on UART0).
Libraries: RadioLib 7.8.x, Adafruit ST7735/ST7789, Adafruit GFX, TinyGPSPlus, NimBLE-Arduino 2.5.x, SdFat 2.3.x, SensorLib 0.5.x (QMC6309 compass, QMI8658 motion), Sgp4 (Hopperpop/Sgp4-Library).
For Bluetooth 5 long-range Remote ID, build with `-DCONFIG_BT_NIMBLE_EXT_ADV=1` (already in
`platformio.ini`; with arduino-cli add `--build-property "compiler.cpp.extra_flags=-DCONFIG_BT_NIMBLE_EXT_ADV=1"`
and the same for `compiler.c.extra_flags`). Without it, Bluetooth 4 and Wi-Fi Remote ID still work.

## Splash

The cyber werewolf is drawn by `tools/make_splash.py`, which writes `CyberHound/splash.h`
as RGB565. Edit the script and run it again to change the art.

## Hardware notes (hardware not tested yet, so check these first)

The pin map comes from the Elecrow wiki and the MeshCore/Meshtastic M9 variants. These are the
places where a hardware revision could differ:

- **Screen upside down:** v1.1 starts in the right orientation. If yours doesn't, use Settings → Screen.
- **Buttons:** learned with the wizard (see above). The keyboard chip sits at I2C 0x6C (V1.0)
  or 0x6D (V1.1) on SDA20/SCL21.
- **No GPS fix:** the GNSS enable polarity follows the keyboard revision (V1.0 LOW, V1.1 HIGH),
  and the baud rate is auto-detected (115200/9600). Settings shows the result.
- **Radio:** LR1110 with TCXO 3.3 V and the DIO5/DIO6 RF switch table taken from MeshCore.
- **microSD:** mounted with SdFat (FAT16, FAT32 and exFAT), trying 16, 8 and 4 MHz. The result,
  e.g. `exFAT 14.8 GB` or the error code, shows at boot and at the bottom of the Wardrive screen.

### Make it beep

In **Tracker hunt** (list or hunt screen) press `B` to make a tracker play its sound, so you can
find it in a bag or car. It uses the anti-stalking sound commands trackers expose for non-owners:
AirTag (`7DFC9000…`), Apple Find My accessories (`FD44`), Google Find My Device / DULT tags
(`15190001…`) and Pebblebee. Trackers only accept this when they're **away from their owner**;
otherwise the screen says the tag refused. SmartTag and Tile don't support it. BLE scanning pauses
while connected (about 10–20 s). `B` again stops the sound.

## Not in v1 yet

- ed25519 advert signature checks, so spoofed adverts can be proven and not only suspected
- Sentry (low-power) mode

## Tests

`tests/` has host-side unit tests: Remote ID and LoRaWAN parsers, a direction-finding simulation
(body-shielding model with noise), GNSS NMEA/GSV parsing plus the jamming/spoofing heuristics, the
EU summer-time rule, the Meshtastic decoder (header, AES-CTR, protobufs, preset frequencies), and an SGP4 pass-prediction check (`tests/run.sh <opendroneid> <Sgp4-Library>`). They feed frames built
by the OpenDroneID reference encoder (Wi-Fi beacon, NAN, BLE 4, BLE 5 pack) and crafted LoRaWAN
uplinks and join requests through the firmware's receive code. See `tests/run.sh`.
