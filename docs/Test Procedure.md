# Clean-Up Drive: breadboard test procedure

**Board:** breadboard build of `docs/CleanUpDrive_schematic.pdf` (v0.1). Refs and values are from the schematic.

**Expected values:** hand calculations at **+9V = 8.8 V**, so **VB = 4.4 V**. Write what you see in the *Measured* column; we'll go through it together.

## How it works (block summary)

| Block | Parts | What it does |
|---|---|---|
| Power | D1 1N5817, C1 100µ, C2/C3 100n; R1/R2 10k + C4 47µ → **VB** | Reverse protection; half-supply bias |
| Input buffer | R3 1M, C5 220n, R4 1M, **U1A** follower → **DRY** | Clean signal for the drive stage *and* the detector |
| Drive stage | **U1B**, R5 100k ∥ C6 100p. Ground leg 1: R6 1k + DRIVE. Ground leg 2: R7 1k + DEPTH + **LDR** | Gain = 1 + R5 / (leg 1 ∥ leg 2). Resting gain ×2 to ×101; the LDR adds gain as you pick harder |
| Clipping | SW1 CLIP: D2/D3 1N4148 / off / D4/D5 red LEDs, across R5 | Holds DRIVEN at about ±0.6 V (Si) or ±1.8 V (LED) |
| Detector | SENSE → **U2A** (×−10, R8/R9, C7) → SC_N; **U2B** unity inverter → SC_P; bridge D6–D9; R12 330 Ω; GR LED D10 + vactrol LED + RV4 1k trim; C8 220µ | Lights the vactrol on hard picks, so the LDR's resistance drops and the gain rises |
| Output | R13 1k + TONE (B50k) + C9 10n; C10 10µ; VOLUME | Hi-cut (16 kHz → 310 Hz) and level |

## Before you start

- **DIP-8 pinout** (OPA2134 and TL072 are the same): 1 = out A, 2 = −A, 3 = +A, **4 = V− (GND)**, 5 = +B, 6 = −B, 7 = out B, **8 = V+**. The pin-1 notch faces left.
- **Vactrol pins:**
  - **VTL5C3:** the LED end and the LDR end are marked; the LED's + lead is the anode.
  - **DIY:** LED anode → GR LED cathode; LED cathode → RV4. The LDR has no polarity.
- **Light-tight:** a DIY vactrol must be **light-tight**. Room light on the LDR lowers its resistance, which *raises the gain*. If the drive changes when you wave a hand over the breadboard, the LDR is seeing room light.
- **Lessons from the Ugly Face and Trem tests:**
  - Scope on **DC coupling**, time/div clicked into a detent, probe ×1/×10 matched to the scope.
  - With the 100 Ω method, move the decimal two places for mA.
  - A multimeter shows the **average** of a waveform; use the scope for anything that moves.
  - A 10 MΩ meter loads nodes fed through 1M (U1 pin 3), so it reads low there.

---

## 0. Before power

| # | Check | Expected | Measured | OK |
|---|---|---|---|---|
| 0.1 | Visual: U1/U2 notch, D1 and bridge D6–D9 bands, C1/C4/C8/C10 polarity, D10 and the vactrol LED orientation, CLIP diodes as an anti-parallel pair | as the schematic | | |
| 0.2 | Ohms +9V rail → GND (wait for it to settle; C1 charges) | climbs to kΩ+. **Fail: a few Ω** | | |
| 0.3 | Ohms across the LDR (U3 pins 3–4), covered/dark | VTL5C3: MΩ (10M spec). DIY: several hundred kΩ+ | | |
| 0.4 | Ohms across the LDR, lit with a red LED or a torch | drops to kΩ | | |

## 1. First power (100 Ω in series with the 9 V; SENSE min; DRIVE min; no input)

| # | Check | Expected | Measured (V across 100 Ω) | mA | OK |
|---|---|---|---|---|---|
| 1.1 | Idle current, OPA2134 in U1 | **~10–13 mA**: OPA2134 ~8 + TL072 ~3 + VB divider 0.4. That's 1.0–1.3 V across 100 Ω | | | |
| 1.2 | Idle current, TL072 in U1 | ~6–7 mA | | | |

Unplug above 2 V (20 mA). **Remove the 100 Ω before section 2.**

## 2. DC voltages (no input; black probe on GND)

| # | Node | Where | Expected | Measured |
|---|---|---|---|---|
| 2.1 | +9V | U1 pin 8, U2 pin 8 | supply − 0.2–0.3 V (D1) | |
| 2.2 | **VB** | C4 + | **4.4 V** | |
| 2.3 | U1 pin 3 | via R4 1M | ≈ 4.4 V; a 10 MΩ meter reads ~10 % low | |
| 2.4 | **DRY** | U1 pin 1 | 4.4 V | |
| 2.5 | **DRIVEN** | U1 pin 7 | 4.4 V ± ~50 mV (offset × the DC gain, up to ×101 at DRIVE max) | |
| 2.6 | SC_N / SC_P | U2 pin 1 / pin 7 | 4.4 V each, **equal within ~20 mV** | |
| 2.7 | Across the bridge (BR_P to BR_N) | D6/D7 cathodes to D8/D9 anodes | ~0 V; GR LED dark | |

If 2.6 differs by more than a few tens of mV, the GR LED may glow at idle. Check R10/R11 and U2B.

## 3. Drive stage, static (signal generator, 1 kHz sine; SENSE min so the LDR stays dark; CLIP off)

| # | Check | Expected | Seen |
|---|---|---|---|
| 3.1 | DRY vs IN, 100 mVpp | equal (gain 1) | |
| 3.2 | DRIVEN, **DRIVE min**, 100 mVpp in | ~200 mVpp (gain 1 + 100k/101k ≈ **×2**) | |
| 3.3 | DRIVEN, **DRIVE max**, **20 mVpp** in | ~2 Vpp (gain 1 + 100k/1k ≈ **×101**) | |
| 3.4 | DRIVE max, raise the input until DRIVEN clips (CLIP off) | the op-amp clips at about 1–7.8 V (OPA2134 on 8.8 V) | |
| 3.5 | Same, **CLIP = Si** | DRIVEN flat-tops at about **±0.6 V around 4.4 V (~1.2 Vpp)**, soft-edged | |
| 3.6 | Same, **CLIP = LED** | about **±1.8 V (~3.6 Vpp)**, louder and more open | |
| 3.7 | High-frequency roll-off, DRIVE max (20 mVpp in) | gain −3 dB near **16 kHz** (R5 100k × C6 100p) | |

**Output section** (DRIVE min, 100 mVpp in):

| # | Check | Expected | Seen |
|---|---|---|---|
| 3.8 | TONE fully one way (R13 1k only) | hi-cut corner ≈ 16 kHz: flat across the audio band | |
| 3.9 | TONE the other way (1k + 50k) | corner ≈ 310 Hz: 1 kHz down about 10 dB | |
| 3.10 | VOLUME | smooth; OUT centred on 0 V (after C10) | |

## 4. Detector (1 kHz sine; scope on SC_N, SC_P and across R12)

**Threshold prediction:** the bridge conducts when |SC_P − SC_N| exceeds the voltage the LED string needs to turn on:

| | Volts |
|---|---|
| Two Schottkys | 2 × 0.3 |
| GR LED (red) | 1.8 |
| Vactrol LED | ~1.7 |
| **Total** | **≈ 4.1 V** |

At SENSE max the bridge sees 20 × DRY, so the **GR LED starts to light at about 0.2 V peak in (≈ 0.4 Vpp)**. With D10 jumpered out, that falls to about 0.23 Vpp.

| # | Check | Expected | Seen |
|---|---|---|---|
| 4.1 | SC_N at SENSE max, 100 mVpp in | 1 Vpp, **inverted** (U2A ×−10) | |
| 4.2 | SC_P | 1 Vpp, in phase with IN (U2B inverts SC_N) | |
| 4.3 | SENSE at noon (linear pot) | SC_N/SC_P about half of 4.1 | |
| 4.4 | SENSE max: raise the input until the GR LED just glows | ≈ 0.4 Vpp in | |
| 4.5 | LED current at 1 Vpp in, RV4 at minimum | V across R12 ÷ 330 Ω, peaks of a few mA (U2 swings ~±2.9 V each side) | |
| 4.6 | RV4 trim: how far the GR LED dims across the trim's range | dimmer as the resistance rises | |
| 4.7 | Across C8 (BR_P − BR_N) with a steady 1 Vpp in | DC ≈ the LED string voltage (~3.5–4 V), with a small ripple at 2 kHz | |

## 5. The dynamics (scope: CH1 = IN, CH2 = DRIVEN; CLIP Si; DRIVE min; DEPTH max effect; SENSE max)

DEPTH's maximum effect is its **lowest** resistance; check which way your pot turns.

**Gain prediction:**
- At rest (LDR dark): ×2.
- LDR lit to ~2k: leg 2 = R7 1k + 2k = 3k, in parallel with leg 1 (101k) ≈ 2.9k, so **≈ ×35**.
- That's a jump of about **+25 dB**, which slams the diodes.

| # | Check | Expected | Seen |
|---|---|---|---|
| 5.1 | Step the generator from 50 mVpp to 1 Vpp (or use burst mode). Trigger single-shot on CH1 | DRIVEN jumps into clipping within **a few ms** (LDR turn-on ~2.5 ms) | |
| 5.2 | Step back down to 50 mVpp | the gain falls back from ×35 to ×2 over **~50–150 ms** (the C8 hold, then the LDR's ~35 ms release) | |
| 5.3 | Repeat 5.2 with C8 = 100µ and 470µ | faster / slower clean-up | |
| 5.4 | DEPTH from min to max at a steady 1 Vpp | the clipping amount changes; at least effect, ×2 only | |
| 5.5 | **Guitar:** single notes, pick hard then let ring. CH1 = IN, CH2 = DRIVEN, single-shot | clipped attack, then the tail un-clips as it decays | |
| 5.6 | Low E, hard pick, short C8 (100µ) | look for **flutter**: gain wobbling at 2× the note frequency (envelope ripple) | |

## 6. Listening tests (guitar → amp)

Start with: DRIVE min, DEPTH max effect, SENSE noon, CLIP Si, TONE noon, VOLUME to unity.

| # | Try | Listen for | Notes |
|---|---|---|---|
| 6.1 | Soft vs hard picking | Clean when soft, crunch when hard. Where's the knee? Smooth or a "switch"? | |
| 6.2 | Let chords ring | Does the tail clean up? Is the drop in level natural, or does it feel like a gate? | |
| 6.3 | SENSE sweep | How hard you must pick before it bites | |
| 6.4 | DEPTH sweep | How much extra drive a hard pick adds | |
| 6.5 | DRIVE up a bit | Resting crunch with "more on attack" vs a clean resting tone | |
| 6.6 | CLIP Si / LED / off | Si tight; LED louder and open; off = pure upward expander (watch the level!) | |
| 6.7 | C8: 100µ / 220µ / 470µ | Speed of the clean-up | |
| 6.8 | Jumper out the GR LED (D10) | More sensitive detector: it bites with lighter picking | |
| 6.9 | R5 100k → 47k | Smaller level jump between clean and clipped | |
| 6.10 | **U1: OPA2134 ↔ TL072** at DRIVE max | Hiss; clarity in the clean region; any difference in the clipping | |
| 6.11 | Single-coils vs humbuckers | Same settings: does the knee move a lot? (affects SENSE range) | |
| 6.12 | Before / after a fuzz or another drive | Out of interest: in front of a Big Muff it should act like a pick-sensitive boost | |

## 7. Troubleshooting

| Symptom | Likely cause | Check |
|---|---|---|
| GR LED never lights | SENSE at min; bridge or vactrol reversed; U2 not powered | 4.1/4.2 waveforms; D6–D9 bands; vactrol LED orientation |
| GR LED lit at idle | SC_N ≠ SC_P DC (offset), or U2 oscillating | 2.6; scope SC_P for HF oscillation |
| Drive changes when you move your hand | Room light on the LDR | Cover the vactrol |
| Distorts even soft at DRIVE min | LDR not dark (light leak, or a slow LDR still recovering); DEPTH at max effect | Measure the LDR (0.3) in situ with the power off |
| Squeal or hiss bursts at DRIVE max | ×101 on a breadboard: stray capacitance from DRIVEN back to the inputs | Shorten the U1B wiring; try C6 220p–470p; keep IN away from DRIVEN |
| Tick or thump with each hard pick | LED current pulses on the supply → VB | Add 100 µF at U2's supply; return the bridge/LED ground directly to C1 |
| Flutter on low notes | Envelope ripple through the LDR | Bigger C8; 1k–4.7k in series with the vactrol LED |
| Output level jumps a lot between clean and clipped | Too much envelope gain | Less DEPTH / SENSE; R5 47k; LED clip |

## Notes / surprises

(Fill in while testing.)
