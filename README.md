# Dual-Stage Hydrolock Prevention System

Prototype test and validation package from **RFL Academy | Innovation Lab**.

The system protects an engine intake from water ingestion (hydrolock) in two stages:

1. **Passive stage:** a 50 mm, 5 µm hydrophobic PTFE membrane in a 40 / 50 / 40 mm transition housing lets air through and blocks bulk water from the moment it arrives, with no electronics involved.
2. **Active stage:** an ESP32 watches four conductive sensor rings and drives a purge blower, two pumps, a normally closed drain solenoid and an engine kill relay to remove the water the membrane has stopped.

> **Scope:** this is a proof-of-concept prototype tested on our own intake test rig. It was not fitted to a vehicle, and the results do not represent on-road performance. The datasets are representative values consistent with the behaviour observed on the prototype.

## Repository contents

```
reports/   Hydrolock_Test_Validation_Report.docx   7-page test and validation report
data/      12 CSV datasets
graphs/    charts used in the report
images/    CAD renders and prototype photos
```

## System overview

| Element | Position / function | Firmware signal |
|---|---|---|
| Ring 1, Ring 2 (S1) | Upstream of the membrane; first detection stage | ADC raw counts, threshold 1000 of 4095 |
| Ring 3 (S3) | Collection funnel below the membrane | Starts Pump A and opens the NC solenoid |
| Ring 4 (S2) | Downstream of the membrane; safety stage | Feeds the engine kill condition |
| Purge blower | Reverse airflow across the membrane face | ESC throttle 1000 µs (idle) / 1400 µs (purge) |
| Pump B / Pump A | Pump B clears the membrane face; Pump A empties the funnel | H-bridge PWM1 / PWM2 |
| NC solenoid | Drain; opens only together with Pump A, closes on power loss | MOSFET output |
| Engine kill relay | Final fail-safe | Ring 4 wet 200 to 300 ms AND Ring 1 wet AND purge active |

## Test conditions

| Parameter | Value |
|---|---|
| Intake air speed (40 mm inlet) | 8.03 to 15.95 m/s (mean 11.73), i.e. 36 to 72 m³/h |
| Membrane face velocity | mean 7.5 m/s |
| Water injection rate | 33.0 to 38.5 L/min (mean 35.9) |
| Water volume per injection | 1.10 to 1.28 L per 2 s burst |
| Bypass injection (kill relay tests) | 17.6 to 23.3 L/min onto Ring 4 |
| Detection threshold | 1000 ADC counts |
| Confirmation windows | S1 100 to 300 ms, S2 200 to 300 ms |
| Supply | 12 V lead acid battery through buck converters |

## Key results

| Metric | Result |
|---|---|
| Airflow preserved downstream (median) | 64.5 % |
| Membrane pressure drop (mean / max) | 1.88 / 4.76 kPa (fit: ΔP = 0.012 · v²) |
| Water passing the membrane | 0.00 L/min in every run (100 % blocked) |
| S1 confirmation time (mean) | 208 ms |
| False trigger rate | 12.7 % |
| Drain clearance time (mean) | 11.99 s (8.30 to 14.30 s) |
| Backflow through the NC solenoid | 0.00 L/min in every run |
| Kill relay trips / false shutdowns | 3 / 4 in 94 events |
| Average stage efficiency | Membrane 100 %, drain 80 %, purge 78 %, overall 77 % |

## Datasets

**Full-system logs (all sensors)**

| File | Rows | Columns | Dates | Contents |
|---|---|---|---|---|
| `hydrolock_full_integrated_testing_oct.csv` | 113 | 31 | 1 to 2 Oct 2026 | All stages running together, stage efficiencies |
| `hydrolock_final_validation_run1_oct.csv` | 1500 | 27 | 2 Oct 2026 | 100 ms log, four rings and all actuators, three water bursts |
| `hydrolock_final_validation_run2_oct.csv` | 1500 | 27 | 2 Oct 2026 | 100 ms log, two water bursts plus a bypass event that fires the kill relay |

**Focused tests**

| File | Rows | Columns | Dates | Contents |
|---|---|---|---|---|
| `hydrolock_airflow_membrane_testing_sept.csv` | 128 | 12 | 16 to 23 Sep 2026 | Air speed in/out, ΔP, water in/out, membrane status |
| `hydrolock_sensor_response_testing_sept.csv` | 79 | 13 | 16 to 23 Sep 2026 | S1 confirmation, S2 persistence, false triggers |
| `hydrolock_purge_drain_testing_sept.csv` | 67 | 16 | 16 to 23 Sep 2026 | Clearance time, efficiencies, valve sealing, blower current |
| `hydrolock_safety_kill_testing_sept.csv` | 94 | 15 | 16 to 23 Sep 2026 | Kill condition, response, false shutdowns, recovery |
| `hydrolock_ring_sensor_calibration_sept.csv` | 120 | 8 | 16 to 17 Sep 2026 | Ring ADC readings when dry, wet and splashed |
| `hydrolock_membrane_pressure_sweep_sept.csv` | 51 | 10 | 18 Sep 2026 | P1, P2 and ΔP across 8 to 16 m/s |
| `hydrolock_water_injection_calibration_sept.csv` | 48 | 7 | 19 Sep 2026 | Pump setting against measured flow and burst volume |
| `hydrolock_blower_current_sweep_sept.csv` | 45 | 7 | 20 Sep 2026 | Throttle 1000 to 1400 µs against current and power |
| `hydrolock_pump_solenoid_timing_sept.csv` | 40 | 11 | 21 to 22 Sep 2026 | Pump A and valve open/close timing, drain time |

Each dataset keeps a few clearly off-nominal readings (sensor saturation, a wetted membrane, loss of pump prime, and similar). Section 5 of the report discusses them.

## Instrumentation

| Quantity | Instrument |
|---|---|
| Air speed | Vane anemometers at inlet (A1) and outlet (A2) |
| Water flow | Inline turbine flow meters: injection (F1), downstream catch (F2), bypass port (F3) |
| Membrane pressure drop | Two BMP280 absolute pressure sensors either side of the membrane |
| Blower current | Hall-effect current sensor |
| Sensor rings | ESP32 12-bit ADC |
| Timing | ESP32 `millis()` event log |
