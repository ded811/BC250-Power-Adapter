# BC-250 J2000/J2001 Power Adapter

> ## ⚠️ UNTESTED
> **I'm still waiting on my boards from fab. This design has not been built or tested on real hardware yet.** Posting early since others were interested. Build at your own risk until this notice is removed.

A simple PCB that adapts the BC-250's J2000 and J2001 power ports to standard PSU cables. One side has two Micro-Fit connectors that plug into the board, the other side has a PCIe 8-pin (GPU cable) and an EPS 8-pin (CPU cable) header for your PSU cables. No components, just copper connecting one side to the other.

KiCad project files are in this repo.

---

## ⚠️ Warnings

- **Use a single-rail PSU (or one with bridged rails) if you power the board from more than one connector.** That means both headers on this adapter, or this adapter plus the board's own GPU connector (J1000). Two independent rails tied together will trip over-current protection at best, damage the PSU at worst. Multi-rail server and mining PSUs are the usual culprits.

---

## Parts

Both options need the same two BC-250-side connectors:

| Qty | Part | Function | Source |
|---|---|---|---|
| 2 | Molex **0447690801**, 8-pos Micro-Fit 3.0 BMI receptacle | Plugs into J2000 / J2001 | [DigiKey](https://www.digikey.com/en/products/detail/molex/0447690801/513218) |

For the PSU-side headers, pick one:

### Option 1: Keyed (preferred)

| Qty | Part | Function | Source |
|---|---|---|---|
| 1 | 8-pin PCIe male header, 90°, keyed | Takes a GPU cable | [ModDIY](https://www.moddiy.com/products/2427/8-Pin-Graphics-Card-PCIe-Male-Header-Connector-90-Degree-Angled-Black.html) |
| 1 | 8-pin EPS male header, 90°, keyed | Takes a CPU cable | [ModDIY](https://www.moddiy.com/products/4298/8-Pin-CPU-EPS-Male-Header-Connector-90-Degree-Angled-Black.html) |

Keyed connectors physically block the wrong cable type. Molex doesn't sell these to individuals, so buy from a trusted reseller like ModDIY (avoid the cheap AliExpress ones).

### Option 2: Unkeyed (cheaper)

| Qty | Part | Function | Source |
|---|---|---|---|
| 2 | Molex **0039301080**, Mini-Fit Jr 8-pos male header, 90° | Takes either cable, ~$0.85 each | [DigiKey](https://www.digikey.com/en/products/detail/molex/0039301080/561081) |

Cheaper mostly because you can combine shipping with the 0447690801 connectors in one DigiKey order instead of also paying for shipping from ModDIY. The catch is below.

> **⚠️ Never plug a CPU cable into the GPU header or the other way around.** PCIe and EPS pinouts are mirrored, so the wrong cable puts 12V where ground should be. This can kill the board and the PSU. The keyed connectors in Option 1 physically block the wrong cable. The unkeyed ones in Option 2 accept either, so if you go that route, label your headers and double check before powering on.

---

## Pinout (BC-250 side)

From `mothenjoyer69/bc250-documentation`:

```
        J2000                    J2001
     v                        v
[ LED1  12V   12V   12V ]  [ 12V   12V   12V   GND ]
[ LED2  GND   GND   GND ]  [ GND   GND   GND   PGD ]
```

| Pin | Function |
|---|---|
| LED1 | Active-low output, mirrors the rack's green backplane LED |
| LED2 | Active-low output, mirrors the rack's red backplane LED |
| PGD | Power-good signal, reads 5V when a second PSU is on the rack's distribution board |

LED1 and LED2 are broken out as labeled headers on this adapter. They're logic outputs, not a power source. To add a status LED, wire it from the pin to 12V through a resistor, cathode toward the pin. Drive strength isn't documented, so size the resistor conservatively.

---

## Power

**BC-250 side:** Micro-Fit contacts are rated 5A each. Three 12V pins per connector gives 6 x 5A x 12V = **360W** across both.

**PSU side:**

| Header | Safe continuous | Theoretical max |
|---|---|---|
| PCIe 8-pin | ~150W | ~324W |
| EPS 8-pin | ~235W | ~432W |

Plan around the safe numbers. The max figures are raw pin ratings with zero margin.

**Check your cable gauge.** The ratings above assume decent wire. 16 AWG handles the full connector rating, 18 AWG has melted on this board under sustained load, and 20 AWG (common on cheap or SFF cables) is only good for roughly half the rated current per wire. This adapter exists partly for that reason: splitting the load across both a GPU and a CPU cable keeps the current per wire low enough that even thin cables stay safe.

**What the BC-250 actually draws:**

| Scenario | Draw |
|---|---|
| Idle | ~70-95W |
| Stock 24 CU gaming | ~150-200W, peaks near 250W |
| 40 CU @ 1500 MHz | ~125-150W |
| 40 CU @ 2 GHz | ~181W GPU alone, more with CPU load |
| Sustained full load | Can pass 320W |

One EPS header alone is tight even at stock, since gaming can peak past its 235W rating. **Use both headers for anything overclocked or under sustained load.** Together they're good for ~385W, which covers everything the board can realistically pull.

PSU sizing from the community: 250W+ on the 12V rail before overclocking, 300W+ for heavy overclocks, 400-500W+ for sustained full-CU compute.

---

## Disclaimer

Based on community reverse engineering, not official schematics. Verify pinouts against your own board before applying power. Use at your own risk.
