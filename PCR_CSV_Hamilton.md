# PCR_CSV_Hamilton – Setup & Migration Notes

Ported from `Flex_PCR_CSV.py` (Gabriel Straface, Ignea Lab @ McGill)  
Target platform: **Hamilton STAR or STARlet, Venus 4.4+**

---

## Files included

| File | Purpose |
|---|---|
| `PCR_CSV_Hamilton.hsl` | Main Venus HSL method |
| `PCR_CSV_Hamilton.lay` | Deck layout (**you must create this in Venus Deck Editor**) |
| `well_list.csv` | Example CSV (same format as Opentrons version) |

---

## One-time setup before first run

### 1. Create the deck layout (`PCR_CSV_Hamilton.lay`)

Open **Venus Method Editor → Deck Layout Editor** and place:

| Carrier template | Tracks (STARlet) | Contents |
|---|---|---|
| `TIP_CAR_480_A00` | 1–6 | 2× Hamilton HTF 1000 µL filter tip racks |
| `SMP_CAR_24_A00` | 7–9 | 1.5 mL Eppendorf tube rack |
| `MTP_CAR_P3_A00` | 10–15 | 96-well PCR plate (site 1) |
| Inheco ODTC carrier | 16–19 | Inheco ODTC 96 unit |

Name the items exactly:
- Tip rack 1 → `Tips_1`  
- Tube rack → `TubeRack`  
- PCR plate → `PCRPlate`  
- ODTC → `ODTC_96`

Create these sequences and save them in the layout:
- `seqTips_1` — full tip rack 1 (positions 1–96, column-major)
- `seqMMX` — tube rack position A1 only
- `seqDNA` — tube rack position A2 only
- `seqPCRPlate` — full PCR plate (positions 1–96, column-major)
- `seqTarget` — leave empty; populated at runtime by `ParseCSVAndBuildSequence()`

### 2. Clone and tune the liquid class

1. Open **Venus Liquid Class Editor** → find `HighVolumeFilter_Water_DispenseJet_Empty`
2. Clone it; rename to `LC_PCR_Mastermix_JetEmpty`
3. Adjust for your master mix (viscosity, glycerol content):
   - Lower aspirate flow rate to ~80 µL/s
   - Increase settling time to 1 s
   - Set blowout volume to ~20 µL
   - Enable **pLLD** (pressure-based liquid level detection) for 1.5 mL tubes
4. Perform a gravimetric calibration (10 points) and enter the correction curve
5. Update the liquid class name string in `PCR_CSV_Hamilton.hsl` if you rename it

### 3. Install the Inheco HSL library

Confirm `HSLInhecoTEC.hsl` is present in your Venus library path  
(`C:\Program Files (x86)\HAMILTON\Library\`).  
If missing, request it from Hamilton support or the Inheco installation disc.

### 4. Verify the ODTC is declared in the deck layout

The ODTC must be added as an **instrument** (not just a carrier) in the Deck Layout  
Editor so that Venus routes arm motion around it correctly.

---

## CSV format

Same format as the Opentrons version:

```
columns,rows,wells
1,,
,B,
,,C3
```

- **columns** — integer column number (1–12); adds all 8 wells in that column  
- **rows** — single letter (A–H); adds all 12 wells in that row  
- **wells** — individual well name (e.g. `C3`)  

Any combination is valid. Duplicates are automatically removed.  
Save the file as plain text (`.csv`) with comma delimiters.

---

## Key differences from the Opentrons version

| Opentrons Flex | This Hamilton port | Why |
|---|---|---|
| Runtime parameters via web UI sliders | Venus `InputBox` dialogs at run start | Venus prompts the operator before the run begins; no equivalent to Flex's run-time parameter UI |
| Thermocycler module (on-deck, closed integration) | Inheco ODTC via `HSLInhecoTEC.hsl` | Standard Hamilton on-deck thermocycler; same concept, different driver |
| `p200.configure_nozzle_layout(SINGLE)` | Single channel, channel pattern = `0` (bit 0 only) | STAR uses a bitmask; `0` = channel 1 only |
| `dispense_solution()` loop — one tip, one aspirate per well | `DispenseSingleReagent()` — same logic | Direct equivalent |
| `protocol.move_labware(..., use_gripper=True)` | Not needed — ODTC is on-deck; plate stays in place | On-deck ODTC means no iSWAP/gripper transfer required |
| Debug mode | Remove the `MessageBox` trace calls; Venus Method Editor has a built-in simulator | Use **Run Control → Simulate** for dry runs |
| `protocol.pause(msg)` | `MessageBox(msg, 0)` — modal dialog pauses execution | Direct equivalent |

---

## Before your first live run

1. Run the method in **Venus Simulator** (Run Control → Simulate) with a short test CSV (e.g., `wells = A1, A2`).
2. Visually verify the 3D deck animation: arm should travel to the tube rack, pick up one tip, then visit each target well.
3. Perform a **water test** on a sacrificial plate: run with water in the tubes and confirm volumes gravimetrically.
4. Check ODTC ramp rates: actual ramp to 98 °C should be ≤60 s; if `WaitForTemperature` times out, increase the timeout argument.
