# Miniprep_Magbead_Hamilton – Setup & Migration Notes

Ported from `flex_miniprep_omega.py`  
Original: Gabriel Straface & Dan Voicu (Ignea Lab @ McGill University)  
Target: **Hamilton STAR or STARlet, Venus 4.4+**

---

## Files

| File | Purpose |
|---|---|
| `Miniprep_Magbead_Hamilton.hsl` | Main Venus HSL method |
| `Miniprep_Magbead_Hamilton.lay` | Deck layout (**create in Venus Deck Editor**) |

---

## One-time setup

### 1. Deck layout (`Miniprep_Magbead_Hamilton.lay`)

| Carrier template | Tracks (STARlet) | Item name in layout |
|---|---|---|
| `TIP_CAR_480_A00` | 1–6 | 2× HTF 1000 µL filter tip racks: `Tips_1000_1`, `Tips_1000_2` |
| `TIP_CAR_480_A00` | 7–12 | 2× STF 50 µL filter tip racks: `Tips_50_1`, `Tips_50_2` |
| `SMP_CAR_24_A00` | 13–15 | 1.5 mL Eppendorf tube rack: `TubeRack` |
| `DW96_CAR_P2_A00` | 16–18 | Deep-well plate carrier (2 sites): site 1 = `InitialPlate`, site 2 = `MagPlate` |
| `HHS_CAR_A00` | 19–21 | Hamilton Heater-Shaker: `HHS` |
| `RES12_CAR_A00` | 22–27 | 12-well 22 mL reservoir: `Reservoir` |
| `PCR_CAR_A00` | 28–30 | Elution PCR plate: `ElutePlate` |

**Magnetic block:** The Hamilton equivalent of the Flex's `magneticBlockV1` is a static permanent-magnet block (e.g., ALPAQUA Engagé 96 or equivalent) placed on site 2 of `DW96_CAR_P2`. The mag plate sits on this block when not on the HHS. No active driver needed.

#### Required sequences (define in layout editor)

| Sequence name | Points to |
|---|---|
| `seqTips_1000_1` | Full tip rack `Tips_1000_1` (96 positions, column-major) |
| `seqTips_1000_2` | Full tip rack `Tips_1000_2` |
| `seqTips_50_1` | Full tip rack `Tips_50_1` |
| `seqTips_50_2` | Full tip rack `Tips_50_2` |
| `seqMagbeads` | `TubeRack` position A1 |
| `seqElutionBuf` | `TubeRack` position A2 |
| `seqLysisConc` | `TubeRack` position A3 |
| `seqNeutConc` | `TubeRack` position A4 |
| `seqLysis` | `Reservoir` position A1 |
| `seqNeut` | `Reservoir` position A2 |
| `seqBindBuf` | `Reservoir` position A3 |
| `seqPB` | `Reservoir` position A4 |
| `seqPE` | `Reservoir` position A5 |
| `seqEthanol` | `Reservoir` position A6 |
| `seqWaste` | Waste trough or chute target position |
| `MagPlate_MagBlock_Seq` | `MagPlate` at its mag-block site (iSWAP source) |
| `MagPlate_HHS_Seq` | `MagPlate` position on the HHS deck (iSWAP destination) |

**Labware names used in `WellSeq()` calls must match your layout item names exactly:**  
`"InitialPlate"`, `"MagPlate"`, `"ElutePlate"`

### 2. Liquid classes

Clone from the standard Venus libraries and rename:

| Name in method | Clone from | Key tuning |
|---|---|---|
| `HighVolumeFilter_Water_DispenseJet_Empty` | `HighVolumeFilter_Water_DispenseJet_Empty` | Lower aspirate flow to ~150 µL/s for lysis/neut buffers; enable pLLD for tubes; 15 µL blowout |
| `StandardVolume_Water_DispenseJet_Empty` | `StandardVolume_Water_DispenseJet_Empty` | Tune for bead/elution buffers; enable pLLD; 5 µL blowout |

Perform gravimetric calibration on both classes before first live run.

### 3. HHS (Heater-Shaker) setup

- Confirm `HslHamHeaterShakerLib.hsl` is in your Venus library path.
- Connect the HHS via USB. The method uses `CreateUSBDevice(hhs_device, 0)` — change the index if multiple USB devices are present.
- **Teach the iSWAP positions** for both the mag-block site and the HHS deck in the Deck Layout Editor. This is the most failure-prone step: verify in simulation first.

### 4. Waste configuration

Set `seqWaste` to point to either:
- A large waste trough on the deck (recommended: 300 mL trough at end of deck), or
- The Hamilton Waste Chute if your STAR is equipped with one.

---

## Adapting the well list

Edit the `InitialiseWellData()` submethod to change sample wells, wash protocols, and regular/concentrated assignments. Each entry is three parallel array values:

```hsl
SAMPLE_WELLS[n]   = "X#";   // well name, e.g. "A1"
WASH_PROTOCOL[n]  = 0;      // 0=two_wash, 1=PB×2, 2=PE×2, 3=EtOH×2
IS_CONC[n]        = 0;      // 0=regular (tube rack), 1=conc (reservoir)
```

Update `N_SAMPLES` to match the total number of entries.

---

## Key differences from the Opentrons Flex version

| Opentrons Flex | This Hamilton port | Notes |
|---|---|---|
| `smart_pick_up()` — complex tip-packing algorithm | Sequential tip counter (`tipPos_1000`, `tipPos_50`) advancing through rack columns | The smart packer exploited Flex's partial-column nozzle mode; STAR single-channel doesn't need it. Tips are consumed one-by-one in order. |
| `configure_nozzle_layout(SINGLE/PARTIAL/ALL)` | Channel pattern `0` (bit 0 = channel 1 only) on every step | STAR 1 mL channels are individually addressable; all pipetting here uses channel 1 only, matching the original single-tip approach |
| `protocol.move_labware(use_gripper=True)` | `ML_STAR.iSWAP_Move_Plate()` | iSWAP is the STAR's plate-transport arm. Teach source/destination positions in the deck layout. |
| `heater_shaker.set_and_wait_for_shake_speed(1000)` | `HslHamHeaterShakerLib.StartShaker(hhs_device, 1000)` | Direct equivalent |
| `magneticBlockV1` (active, Flex only) | Static permanent magnet block (e.g., ALPAQUA 96) | No Hamilton equivalent to an active magnetic module; a permanent magnet block is the standard STAR approach and works identically for bead capture |
| `reservoir_vol_to_height(vol)` | `ReservoirHeight(vol_mL)` — identical formula: `h = -2.5×V + 50` | Direct port of your original function |
| `temp_module` (declared but unused in original) | Not included | Omitted as it performed no actions in the original protocol |
| Waste chute | `seqWaste` sequence pointing to waste trough or chute | Configure in layout |

---

## Before first live run

1. **Simulate** in Venus Run Control (Simulate mode) — verify all iSWAP moves are collision-free in the 3D view.
2. **Water test** with dye: run lysis and transfer steps only, confirm volumes gravimetrically.
3. **Bead test**: verify beads remain on the magnet after the 90-second settle; if not, extend the delay in `Step_HHS_Incubate()`.
4. **Elution test**: confirm 30 µL transfer recovers from the bead pellet cleanly; adjust `DEPTH_OFFSET` if the tip contacts beads.
5. Check iSWAP grip width and height offsets for your specific deep-well plate brand.
