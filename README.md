# openGSMTower

A GSM base-station sound + signal simulator. Ships as `openGSMTower.cpp` — a
"C++ app that's actually just Python" (valid polyglot: it compiles as C++ into
a stub banner, but the real engine runs with `python openGSMTower.cpp`).
`openGSMTower.exe` is the standalone Windows build, no Python required.

It synthesizes the classic GSM TDMA buzz the way it physically happens:
8-slot frames at 4.615 ms (216.7 Hz), BCCH slot 0 always on, traffic-gated
slots, 26-multiframe idle rhythm, 28 us PA ramps, envelope-detected
breakthrough hash, edge clicks, and site ambient — plus a true RF-rate GMSK
IQ exporter so the signal also *looks* like a tower in an SDR waterfall.

## Quick start

1. Run `openGSMTower.exe` (accept the license dialog).
2. **Pick your output device first** — the selector is visible before Start so
   nothing blasts the wrong speakers.
3. Set band / load / proximity, then press **Start**. Graphs and slot LEDs
   appear while running; **Stop** hides them again.

> Audio can get LOUD (especially with SDR drive up). Check volume first.

## Controls

| Control | What it does |
|---|---|
| Band | GSM-450 / 480 / 850 / 900 / R / 1800 / 1900 — real UL/DL ranges shown, each with its own voice (hash brightness, mains hum, PSU whine, ring level) |
| Cell load | Traffic occupancy on slots 1–7 |
| Proximity | Far / Street / Near / Tower-close presets (drive + load + dummy bursts) |
| SDR drive | Extra gain into the virtual cable, soft-clipped (S-meter estimate shown) |
| Solid BCCH | Dummy bursts on idle slots = solid C0 carrier like a real tower |
| Wide carrier | Brighter wideband hash for the SDR line |
| Output | Audio device picker (switchable live) |
| Export IQ | Renders an RF-rate capture for SDR apps, with Width / TRX / Scene options |

## SDR guide

Two ways to see it in an SDR:

- **Live cable** (48 kHz audio into a virtual cable): max ±24 kHz wide. Good
  for the buzz + comb. Urban scene adds a 19 kHz pilot and stray tones.
- **IQ export** (the good stuff): full-rate GMSK at 1083333 Hz, FCCH/SCH/BCCH
  structure, ~200 kHz occupancy, 45 dB SNR. In SDR++ use File Source with
  the `.cu8` as U8 @ 1083333 Hz (or the `_iq16.wav` as IQ WAV).

Suggested recipes:

- Authentic tower: Width Normal, TRX 1, Scene Clean tower
- Dense urban 900 look: Width Wide, TRX 2–3, Scene Dense urban 900
- Full firehose: Width XXL Splatter, TRX 4, Scene Dense urban 900 (off-spec,
  maximum spray — not what a real filtered BTS looks like)

Tip: view GSM with a ~200 kHz-wide filter, not a narrow AM slice, or you'll
only ever see a hairline.

## CLI

The exe and the script both accept:

```text
openGSMTower.exe --test-synth        # self-checks (PASS / IQ-PASS / WIDE-PASS / BAND-PASS / SCENE-PASS / LIVE-PASS)
openGSMTower.exe --export-iq [secs] [basename]
```

Windowed builds report these via popup dialogs; console runs print + exit codes.

## Run from source

```text
python openGSMTower.cpp
```

Requires: `numpy`, `scipy`, `matplotlib`, `sounddevice` (plus stdlib `tkinter`).

## Build the exe yourself

```text
pip install pyinstaller
copy openGSMTower.cpp openGSMTower_app.py
python -m PyInstaller --noconfirm --clean --onefile --windowed --name openGSMTower ^
  --icon openGSMTower.ico --add-data "openGSMTower.ico;." ^
  --hidden-import sounddevice --hidden-import scipy.signal ^
  --hidden-import scipy.special --collect-all sounddevice openGSMTower_app.py
```

Icon workflow: put square artwork in `OpenGSMTowerNEW.png`, run
`python make_icon.py` to regenerate `openGSMTower.ico`, rebuild.

## Files

| File | Purpose |
|---|---|
| `openGSMTower.cpp` | Source of truth (C++/Python polyglot) |
| `openGSMTower_app.py` | Build copy of the above for PyInstaller |
| `openGSMTower.exe` | Standalone Windows build |
| `openGSMTower.ico` / `OpenGSMTowerNEW.png` | App icon + its source artwork |
| `make_icon.py` | Artwork → multi-size `.ico` converter + build notes |

## License

Provided AS IS, no warranty. Loud audio — check your levels. IQ captures are
simulated for education/testing only; do not transmit them over the air and
follow your local radio laws. First launch shows the full terms gate.
