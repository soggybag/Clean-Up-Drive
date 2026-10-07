# Clean-Up Drive: breadboard notes

## How it works

1. **U1A** buffers the guitar (1M input). That clean signal (DRY) feeds both the drive stage and the detector.
2. **U1B drive stage:** gain = 1 + R5 / (ground legs).
   - **Leg 1:** R6 1k + DRIVE (C100k rheostat). Sets the **resting** gain, from ×2 to ×101.
   - **Leg 2:** R7 1k + DEPTH (B50k rheostat) + the **LDR**. Dark, the LDR is megohms and does nothing. As you pick harder it lights, pulls this leg down and raises the gain. DEPTH sets how far.
   - **Clipping** (SW1 CLIP: Si / off / LED): the diodes across R5 hold the output at about ±0.6 V (1N4148) or ±1.8 V (red LED). The op-amp itself never runs out of swing.
3. **Detector (U2):** the Flatline+ side chain, fed from DRY through SENSE.
   - U2A gain 10 and U2B a unity inverter, so the bridge sees 0–20× the clean signal.
   - The full-wave 1N5817 bridge → GR LED → vactrol LED → 1k trim, with C8 220 µF setting the release.
4. **Output:** TONE (hi-cut, 16 kHz → 310 Hz) and VOLUME.

**The feel:** a hard pick lights the LDR (~2.5 ms), the gain jumps and the note clips. As the note decays the LED dims, the LDR recovers (~35 ms, plus C8's hold) and the gain drops back to DRIVE's resting level. The tail is cleaner *and* quieter.

## Parts (all through-hole)

| Qty | Part | Where |
|---|---|---|
| 1 | OPA2134PA (DIP-8), and a TL072 to A/B | U1 (socket it) |
| 1 | TL072 (DIP-8) | U2 |
| 1 | VTL5C3 / Xvive VTL5C3, or a red LED + LDR in heat-shrink | U3 |
| 5 | 1N5817 | D1 (power), D6–D9 (bridge) |
| 2 | 1N4148 | D2, D3 (Si clip) |
| 3 | red 3 mm LED | D4, D5 (LED clip), D10 (GR LED) |
| 1 | ON-OFF-ON mini toggle | SW1 CLIP |
| 6 | pots: C100k DRIVE, B50k DEPTH, B100k SENSE, B50k TONE, A100k VOLUME; 1k trimmer | RV1–RV6 |
| | resistors: 1M ×2, 100k ×2, 10k ×5, 1k ×4, 330 Ω | |
| | caps: 220n film, 100p, 220p, 10n, 100n ×2, 10 µF, 47 µF, 100 µF, 220 µF | |

DRIVE is drawn as C100k (reverse log) so CW = more gain with the wiper wired as shown. An A100k works too; it just turns the other way.

## First power-up

1. Set DRIVE minimum (resting gain ~×2), DEPTH full (most envelope gain), SENSE noon, CLIP on Si, TONE noon, VOLUME low, trim mid.
2. Check:
   - VB = 4.5 V at U1 pin 3 (via R4), U2 pins 3/5, and U3 pin 3;
   - DRY ≈ 4.5 V DC;
   - DRIVEN ≈ 4.5 V DC.
3. Play: the GR LED should flash on hard picks. If it never lights, turn up SENSE.

## What to listen for / try

- **The knee:** with DRIVE low, soft picking should be clean and hard picking crunchy. If the change is too abrupt, lower DEPTH or SENSE.
- **Decay:** how fast the tail cleans up. Try C8 at 100 µF (faster), 220 µF and 470 µF (slower).
- **CLIP:** Si is tighter and quieter; LED is louder and more open. Off = a pure upward expander, with no diodes. Keep VOLUME low and watch U1B's output for clipping on the scope. It's a good demonstration of why the diodes matter.
- **GR LED:** it adds ~1.8 V to the detector threshold. Jumper it out (short D10) to make the detector more sensitive.
- **Level jump between the clean and clipped regions:** if it's too big, try R5 47k (less gain overall) or the LED clip.
- **TL072 vs OPA2134 in U1:** swap them at high DRIVE and listen for hiss and for the clean-region clarity.
- **Envelope ripple:** low notes with a fast release may "flutter". If so, increase C8, or add 1k–4k7 in series with the vactrol LED (slower attack).

## Measure (scope)

- **Compare in vs out:** IN and DRIVEN on two channels while picking. See how the gain changes between attack and decay.
- **The envelope itself:** U3 pin 1 (vactrol LED anode) against BR_N.
