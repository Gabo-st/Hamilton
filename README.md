# Hamilton STAR/STARlet – Automated Lab Protocols

Venus HSL methods ported from the Opentrons Flex for the Hamilton STAR / STARlet liquid handler.  
**Lab:** Ignea Lab, McGill University  
**Authors:** Gabriel Straface

---

## Protocols

| Method | Description |
|---|---|
| [Miniprep_Magbead_Hamilton.hsl](Miniprep_Magbead_Hamilton.hsl) | Pellet-free plasmid miniprep using magnetic beads |
| [PCR_CSV_Hamilton.hsl](PCR_CSV_Hamilton.hsl) | Flexible PCR setup driven by a user-provided CSV well list |

---

## Miniprep – Magnetic Bead Plasmid Purification

Automates a pellet-free magnetic-bead miniprep on up to 11 samples in a deep-well plate. All pipetting uses channel 1 only (single-tip), mirroring the original Opentrons Flex protocol.

**Instruments required:**
- Hamilton ML STAR or STARlet (Venus 4.4+)
- Hamilton Heater-Shaker (HHS) with USB connection
- Static permanent-magnet block (e.g., ALPAQUA Engagé 96)
- iSWAP plate-transport arm

**Key reagents per sample:**

| Reagent | Volume |
|---|---|
| Bacterial culture (pre-loaded) | 940 µL |
| Lysis buffer | 470 µL |
| Neutralization buffer | 239 µL |
| Binding buffer | 301 µL |
| Magnetic beads | 40 µL |
| Wash buffer | 300 µL × 2 |
| Elution buffer | 30 µL |

**Output:** 30 µL of purified plasmid DNA per well in a 96-well PCR plate.

**Setup guide:** [Miniprep_Magbead_Hamilton.md](Miniprep_Magbead_Hamilton.md)

---

## PCR – CSV-Driven PCR Setup

Distributes master mix and template DNA to any subset of a 96-well PCR plate specified by a plain-text CSV file, then runs a fully customisable thermocycler program on an on-deck Inheco ODTC 96.

**Instruments required:**
- Hamilton ML STAR or STARlet (Venus 4.4+)
- Inheco ODTC 96 on-deck thermocycler

**Runtime parameters** (entered via dialog at run start):

| Parameter | Default |
|---|---|
| Well list CSV path | `C:\Hamilton\Methods\well_list.csv` |
| Master mix volume | 20 µL |
| Template DNA volume | 1 µL |
| Denaturation / annealing / extension temps | 98 / 63 / 72 °C |
| Number of PCR cycles | 30 |
| Colony PCR mode | Off |

**CSV format:**
```
columns,rows,wells
1,,
,B,
,,C3
```
Any combination of whole columns, whole rows, and individual wells is accepted. Duplicates are automatically removed.

**Setup guide:** [PCR_CSV_Hamilton.md](PCR_CSV_Hamilton.md)

---

## Repository Structure

```
Hamilton/
├── Miniprep_Magbead_Hamilton.hsl   # Venus HSL method – miniprep
├── Miniprep_Magbead_Hamilton.md    # Setup & deck layout guide
├── PCR_CSV_Hamilton.hsl            # Venus HSL method – PCR
└── PCR_CSV_Hamilton.md             # Setup & deck layout guide
```

Deck layout files (`.lay`) and CSV templates are not versioned here because they are created locally in the Venus Deck Editor and depend on the specific instrument calibration. See the per-protocol setup guides for the required sequences and carrier positions.

---

## Prerequisites

- **Hamilton Venus 4.4+** with the following HSL libraries installed:
  - `HSLSeqLib.hsl`, `HSLStrLib.hsl`, `HSLUtilLib.hsl`, `HSLFileLib.hsl` (all standard)
  - `HslHamHeaterShakerLib.hsl` (miniprep only – bundled with Venus HHS option)
  - `HSLInhecoTEC.hsl` (PCR only – from Hamilton/Inheco; request from Hamilton support if missing)
- **Hamilton ML STAR or STARlet** with iSWAP arm (miniprep) or standard arm (PCR)
- Liquid classes cloned and calibrated per the setup guides (gravimetric calibration recommended before first live run)

---

## Quick Start

1. Clone this repository to your Hamilton PC.
2. Open the relevant `.hsl` file in the Venus Method Editor.
3. Create the deck layout (`.lay`) following the setup guide for that method.
4. Define all required sequences in the Deck Layout Editor.
5. Clone and calibrate the required liquid classes.
6. Run in **Simulate** mode (Venus Run Control) to verify arm motion and iSWAP paths.
7. Perform a water test with dye before the first live run.

---

## License

MIT License – see [LICENSE](LICENSE) for details.
