# RFShield LPF

<p align="center">
  <strong>Python-assisted, PCB-aware 108–500 MHz / 50 Ω seventh-order RF low-pass filter design.</strong>
</p>

<p align="center">
  <img alt="Passband" src="https://img.shields.io/badge/passband-108%E2%80%93500%20MHz-1f6feb">
  <img alt="Impedance" src="https://img.shields.io/badge/system%20impedance-50%20%CE%A9-1f6feb">
  <img alt="Order" src="https://img.shields.io/badge/filter%20order-7-1f6feb">
  <img alt="KiCad" src="https://img.shields.io/badge/KiCad-9.0.7-314cb6">
  <img alt="Final validation" src="https://img.shields.io/badge/REV__C%20validation-0%20DRC%20violations-brightgreen">
  <img alt="Status" src="https://img.shields.io/badge/status-engineering%20prototype-orange">
</p>

## Türkçe proje özeti

**RFShield LPF**, 50 Ω RF sinyal zincirlerinde **108 MHz ile 500 MHz** arasındaki işaretleri düşük kayıpla geçirmek ve daha yüksek frekans bileşenlerini bastırmak amacıyla geliştirilmiş, iki katmanlı, yedinci dereceden bir alçak geçiren filtre PCB projesidir. Tasarım; ayrık L/C değer optimizasyonu, parasitik etkileri içeren PCB-farkındalıklı analitik model, 50 Ω mikroşerit sentezi, Monte Carlo tolerans analizi, KiCad dosya üretimi, DRC doğrulaması ve üretim çıktılarının hazırlanması aşamalarını kapsar.

Bu çalışma bir **mühendislik prototipi** olarak yayımlanmaktadır. Analitik sonuçlar ve KiCad doğrulaması güçlü bir tasarım temeli sağlar; ancak fiziksel üretim sonrası VNA ölçümü, tam dalga elektromanyetik analiz ve hedef RF güç seviyesinde doğrulama yapılmadan seri üretim, maksimum güç veya mevzuat uyumluluğu iddiasında bulunulmaz.

---

## 1. Repository integrity and revision status

The public product name is **RFShield LPF**. Some checked-in files still use the original internal engineering identifier `CEM_RF_LPF_500_FINAL` and the historical revision label `REV_B`. Those identifiers are retained here for traceability and do not change the intended product name.

### Important status distinction

The repository currently contains an earlier engineering snapshot whose checked-in [`drc_summary.json`](drc_summary.json) reports:

```text
20 DRC warnings
0 unconnected items
```

All 20 warnings are of the same class: `via_dangling`. They were generated because the RF-route stitching vias were connected to the bottom ground plane but did not yet have a continuous top-copper connection.

A later **REV_C** production run corrected this by adding continuous F.Cu ground guard rails that electrically join the via fence on both copper layers. That run completed with:

```text
Found 0 violations
Found 0 unconnected items
```

Therefore:

- the RF topology and selected component values are the final intended design;
- the checked-in REV_B DRC report is historical evidence of the issue that was found;
- the corrected REV_C geometry is the manufacturing target;
- the current repository snapshot should not be treated as the final fabrication master until the corrected REV_C board source and its new DRC report are committed together.

This distinction is documented explicitly to avoid presenting a historical DRC report as if it were the final validated source.

---

## 2. Design objectives

| Parameter | Target |
|---|---:|
| Intended passband | 108–500 MHz |
| Reference impedance | 50 Ω |
| Filter class | Low-pass |
| Filter order | 7 |
| Network form | Symmetrical series-L / shunt-C ladder |
| PCB layer count | 2 |
| Nominal finished thickness | 1.60 mm |
| Nominal copper thickness | 35 µm |
| Nominal substrate | FR-4 |
| Nominal relative permittivity | 4.30 |
| Nominal loss tangent | 0.018 |
| Nominal dielectric height to ground | 1.53 mm |
| Input/output connector | 50 Ω SMA |
| Maximum analytical passband insertion-loss gate | 1.00 dB |
| Minimum analytical passband return-loss gate | 12.0 dB |
| Minimum analytical attenuation at 1 GHz | 25.0 dB |
| Minimum Monte Carlo pass gate | 95% |

The design is intended for use as a prototype output or interstage low-pass network in sub-500 MHz RF systems. Typical applications may include:

- harmonic suppression after a low-VHF/UHF RF stage;
- receiver front-end or laboratory filtering;
- SDR test chains;
- RF measurement fixtures;
- educational study of practical lumped-element filters and PCB parasitics.

It is not tied to one transmitter, one output power, one national emission mask, or one certified product.

---

## 3. Electrical topology

RFShield LPF uses a symmetrical seventh-order ladder:

```text
J1 ─ L1 ─●─ L3 ─●─ L5 ─●─ L7 ─ J2
         │      │      │
         C2     C4     C6
         │      │      │
        GND    GND    GND
```

Net sequence:

```text
RF_IN → L1 → N1 → L3 → N2 → L5 → N3 → L7 → RF_OUT
             │          │          │
             C2         C4         C6
             │          │          │
            GND        GND        GND
```

The symmetry reduces unnecessary input/output asymmetry and simplifies model verification.

### Selected values

| References | Quantity | Selected value | Tolerance used in analysis | Intended component family |
|---|---:|---:|---:|---|
| L1, L7 | 2 | 8.2 nH | ±5% | Coilcraft 0603HP high-Q RF inductor |
| L3, L5 | 2 | 15 nH | ±5% | Coilcraft 0603HP high-Q RF inductor |
| C2, C6 | 2 | 4.3 pF | ±0.1 pF | KYOCERA AVX 600S high-Q C0G/NP0 |
| C4 | 1 | 4.3 pF | ±0.1 pF | KYOCERA AVX 600S high-Q C0G/NP0 |
| J1, J2 | 2 | 50 Ω SMA | — | Amphenol RF 132134-10 family |

Intended example manufacturer part numbers:

| Function | Example MPN |
|---|---|
| 8.2 nH inductor | `0603HP-8N2XJRW` |
| 15 nH inductor | `0603HP-15NXJRW` |
| 4.3 pF capacitor | `600S4R3BT250XT4K` |
| SMA connector | `132134-10` |

Substitution must be based on RF characteristics, not only nominal value and package size. Inductor Q, self-resonant frequency, capacitor ESR/ESL, dielectric technology, pad geometry and connector launch geometry directly affect the upper passband and first stopband octave.

---

## 4. Why a seventh-order Chebyshev-style ladder was used

A higher-order ladder provides a steeper transition than a simple third- or fifth-order network while remaining practical with discrete components. The design began from a classical low-pass prototype approach and was then optimized using real candidate values and a PCB-aware model rather than relying on one ideal calculation followed by simple E-series rounding.

For an ideal low-pass prototype, series inductors and shunt capacitors can be scaled using:

```text
Lk = gk · Z0 / ωc
Ck = gk / (Z0 · ωc)
```

where:

- `gk` is the normalized prototype coefficient;
- `Z0` is the 50 Ω system impedance;
- `ωc` is the chosen angular cutoff frequency.

In a real PCB, the final values cannot be chosen from ideal equations alone because the transmission-line sections, component pads, vias, connector launches and finite component Q modify the response. RFShield LPF therefore performs a second-stage discrete optimization with these effects included.

---

## 5. Discrete component optimization

The engineering automation evaluates a finite candidate space rather than choosing the nearest values once.

Candidate sets:

```text
Outer inductors: 8.2, 9.1, 10, 11, 12 nH
Inner inductors: 15, 16, 18, 22, 24 nH
Outer capacitors: 4.3, 4.7, 5.1, 5.6, 6.2 pF
Center capacitor: 4.3, 4.7, 5.1, 5.6, 6.2 pF
```

Total combinations:

```text
5 × 5 × 5 × 5 = 625 candidate networks
```

Each candidate is evaluated from approximately 10 MHz to 2 GHz. The score penalizes:

- passband insertion loss;
- return loss falling below the acceptance gate;
- insufficient attenuation at 1 GHz;
- excessive passband ripple.

The best-performing candidate is selected automatically and the top 20 candidates are preserved in [`optimization_top20.json`](optimization_top20.json) for engineering traceability.

---

## 6. PCB-aware analytical model

The analysis cascades ABCD matrices for the individual network elements and converts the complete two-port network to S-parameters in a 50 Ω reference system.

### Series element matrix

```text
[ 1  Z ]
[ 0  1 ]
```

### Shunt element matrix

```text
[ 1  0 ]
[ Y  1 ]
```

### Transmission-line matrix

```text
[ cosh(γl)      Z0 sinh(γl) ]
[ sinh(γl)/Z0   cosh(γl)    ]
```

The complete matrix is the ordered product of connector, line, series-L, shunt-C and interconnect matrices.

### Included non-ideal effects

- frequency-dependent transmission-line electrical length;
- approximate conductor and dielectric loss;
- inductor finite Q;
- inductor parasitic capacitance estimated from self-resonant frequency;
- capacitor ESR;
- capacitor ESL;
- shunt-via inductance;
- dual-via ground return;
- finite shunt-stub length;
- connector series resistance and inductance;
- interstage microstrip lengths;
- real discrete candidate values.

### Effects not fully represented

- full 3D connector fields;
- solder fillets and assembly geometry;
- copper roughness tied to a specific laminate;
- vendor Touchstone models for every part;
- enclosure and cable coupling;
- radiation and higher-order mode effects;
- power-dependent heating and saturation;
- exact board-house etch and dielectric distributions.

The analytical model is therefore a strong design-screening tool, but it is not a substitute for full-wave EM simulation or calibrated measurement.

---

## 7. Predicted RF performance

The selected values produced the following PCB-aware analytical results:

| Metric | Predicted result |
|---|---:|
| Worst-case insertion loss in 108–500 MHz | **0.408 dB** |
| Worst-case return loss in 108–500 MHz | **23.370 dB** |
| Attenuation at 1 GHz | **30.837 dB** |
| S21 at 108 MHz | **−0.167 dB** |
| S21 at 500 MHz | **−0.409 dB** |
| S11 at 500 MHz | **−36.024 dB** |
| Predicted passband ripple | **0.242 dB** |

Machine-readable results are stored in [`design_summary.json`](design_summary.json).

### Interpretation

- The predicted passband loss is below the 1 dB engineering gate.
- The predicted return loss has significant margin above the 12 dB gate.
- The 1 GHz attenuation exceeds the 25 dB gate.
- These are model outputs, not measured VNA results.

No measured insertion-loss, return-loss, group-delay, power-handling or harmonic-compliance claim is made in this release.

---

## 8. Monte Carlo tolerance analysis

The selected design was evaluated with **400 deterministic, seeded Monte Carlo runs**.

Random seed:

```text
20260729
```

Tolerance assumptions:

- inductors: ±5%, represented as a normal distribution with the limit at approximately 3σ;
- capacitors: ±0.1 pF, represented in the same manner.

A run passes only when all three conditions are met:

```text
Worst passband IL ≤ 1.00 dB
Worst passband RL ≥ 12.0 dB
Attenuation at 1 GHz ≥ 25.0 dB
```

Reported model statistics:

| Metric | Result |
|---|---:|
| Monte Carlo runs | 400 |
| Passing runs | 100% |
| 95th-percentile passband IL | 0.415 dB |
| 5th-percentile passband RL | 22.161 dB |
| 5th-percentile attenuation at 1 GHz | 30.098 dB |

This result does not include every manufacturing variable. In particular, laminate Er variation, dielectric thickness tolerance, copper thickness, etch bias, connector spread and complete package-model uncertainty are not all randomized. The result must not be interpreted as a guaranteed production yield.

---

## 9. Stack-up and controlled impedance

Nominal two-layer stack-up:

```text
Top solder mask
F.Cu: 35 µm copper — RF components and signal routing
FR-4: Er = 4.30, tanδ = 0.018, nominal dielectric height = 1.53 mm
B.Cu: 35 µm copper — RF ground reference
Bottom solder mask
Nominal finished thickness: 1.60 mm
```

The microstrip width is solved with a Hammerstad-style quasi-static approximation.

| Parameter | Calculated value |
|---|---:|
| Target characteristic impedance | 50.000 Ω |
| Solved trace width | 2.9995 mm |
| Routed nominal trace width | 3.000 mm |
| Effective relative permittivity | 3.2683 |
| Calculated impedance at solved width | 50.000 Ω |

### Critical manufacturing warning

“1.6 mm FR-4” is not a complete controlled-impedance specification. Different fabricators may use different core/prepreg constructions, resin content, copper roughness and actual dielectric thickness.

Before ordering an impedance-controlled lot:

1. obtain the board house's actual stack-up;
2. confirm the dielectric height from F.Cu to B.Cu;
3. update Er, loss tangent and copper thickness;
4. recalculate the required trace width;
5. regenerate the PCB;
6. rerun DRC;
7. request an impedance coupon where available.

A 3.0 mm trace is appropriate only for the nominal stack-up used in this model. It is not a universal 50 Ω width.

---

## 10. PCB layout architecture

Nominal board dimensions:

```text
74 mm × 50 mm
```

### 10.1 RF signal path

The main RF route is kept linear and symmetrical. The nominal 50 Ω sections use a 3.0 mm microstrip width. Narrow necks are limited to locations where 0603 pad geometry requires a transition.

### 10.2 Ground reference

The bottom copper is intended as a continuous RF ground reference. A continuous reference plane minimizes loop area, lowers return-path inductance and makes the microstrip impedance definition physically meaningful.

### 10.3 Shunt-capacitor return paths

Each shunt capacitor requires a short and low-inductance path to ground. Dual local ground vias are used so that the shunt branches remain effective near the upper passband and into the stopband.

A long shared ground bus would add parasitic inductance, detune the ladder and reduce stopband attenuation. The layout avoids that architecture.

### 10.4 SMA launches

The SMA launches include nearby ground connections and stitching vias. The goal is to provide a short transition from the connector ground tabs to the bottom reference plane and to reduce connector-launch inductance.

The exact connector drawing must be checked against the purchased Amphenol part before fabrication. Similar-looking vertical SMA connectors are not footprint-compatible by default.

### 10.5 Via fence

The RF route is bordered by a ground via fence with an approximate 5 mm pitch. In the corrected REV_C geometry, front-copper guard rails connect the via fence on F.Cu while the bottom plane connects the same vias on B.Cu.

This change serves two purposes:

- it removes KiCad `via_dangling` warnings;
- it creates a physically continuous two-layer ground structure rather than a decorative set of vias connected on only one copper layer.

### 10.6 Local footprints

The project includes custom footprint sources:

- [`RF_0603.kicad_mod`](RF_0603.kicad_mod)
- [`SMA_AMPHENOL_132134_10.kicad_mod`](SMA_AMPHENOL_132134_10.kicad_mod)

The [`fp-lib-table`](fp-lib-table) expects a project-local library called `CUSTOM.pretty`. For a clean local checkout, place the two `.kicad_mod` files inside:

```text
CUSTOM.pretty/
```

so that the structure becomes:

```text
RFShield-LPF/
├── CEM_RF_LPF_500_FINAL.kicad_pcb
├── CEM_RF_LPF_500_FINAL.kicad_pro
├── fp-lib-table
└── CUSTOM.pretty/
    ├── RF_0603.kicad_mod
    └── SMA_AMPHENOL_132134_10.kicad_mod
```

The footprints embedded in the board allow the PCB to open, but restoring the intended local-library structure is recommended for future edits and reproducibility.

---

## 11. KiCad design files

| File | Purpose |
|---|---|
| [`CEM_RF_LPF_500_FINAL.kicad_pcb`](CEM_RF_LPF_500_FINAL.kicad_pcb) | Main PCB layout and embedded footprint geometry |
| [`CEM_RF_LPF_500_FINAL.kicad_pro`](CEM_RF_LPF_500_FINAL.kicad_pro) | KiCad project settings and net classes |
| [`CEM_RF_LPF_500_FINAL.kicad_prl`](CEM_RF_LPF_500_FINAL.kicad_prl) | KiCad local UI/session state; not required for fabrication |
| [`fp-lib-table`](fp-lib-table) | Project-local footprint library mapping |
| [`RF_0603.kicad_mod`](RF_0603.kicad_mod) | Custom RF 0603 land pattern |
| [`SMA_AMPHENOL_132134_10.kicad_mod`](SMA_AMPHENOL_132134_10.kicad_mod) | Custom SMA connector footprint |

The `.kicad_prl` file is machine/session specific and is normally excluded from a clean source repository. It is kept in the current snapshot only because it was included in the uploaded engineering package.

---

## 12. Reports and traceability files

| File | Description |
|---|---|
| [`design_summary.json`](design_summary.json) | Selected values, analytical metrics, Monte Carlo statistics and nominal stack-up |
| [`optimization_top20.json`](optimization_top20.json) | Ranked top 20 component combinations from the 625-candidate search |
| [`drc_report.json`](drc_report.json) | Historical KiCad 9.0.7 REV_B DRC report containing 20 `via_dangling` warnings |
| [`drc_summary.json`](drc_summary.json) | Compact summary of the historical REV_B DRC execution |

The report files are intended to make the design decision auditable instead of presenting only a final layout with no engineering history.

---

## 13. Opening the project

Recommended environment:

- Windows 10 or Windows 11;
- KiCad 9.0.7 or a compatible KiCad 9 release.

Procedure:

1. clone or download the repository;
2. create a `CUSTOM.pretty` folder in the repository root;
3. move the two `.kicad_mod` files into `CUSTOM.pretty`;
4. open `CEM_RF_LPF_500_FINAL.kicad_pro`;
5. open PCB Editor;
6. inspect the board stack-up, design rules, footprints and net classes;
7. refill or regenerate copper only when intentionally modifying the design;
8. run DRC locally before exporting any fabrication files.

Command-line DRC example:

```powershell
& "C:\Program Files\KiCad\9.0\bin\kicad-cli.exe" pcb drc `
  --output .\drc_report_new.json `
  --format json `
  --severity-all `
  --exit-code-violations `
  .\CEM_RF_LPF_500_FINAL.kicad_pcb
```

Do not assume that the historical checked-in report describes a modified local board. Always regenerate the report from the exact board revision intended for fabrication.

---

## 14. Intended automated production pipeline

The project was developed through a Python-driven engineering pipeline with the following sequence:

1. define RF and PCB requirements;
2. solve the nominal 50 Ω microstrip width;
3. validate board and component-placement constraints;
4. evaluate 625 discrete L/C candidate networks;
5. select the best candidate using PCB-aware metrics;
6. run 400 Monte Carlo tolerance trials;
7. enforce RF acceptance gates;
8. generate the KiCad PCB and project files;
9. generate custom project-local footprints;
10. generate analytical reports;
11. call KiCad CLI DRC;
12. require zero violations and zero unconnected items for release;
13. export Gerbers;
14. export PTH and NPTH drill files;
15. export drill maps;
16. export CSV component positions;
17. verify expected fabrication outputs;
18. create a production ZIP.

The current repository snapshot contains the generated design and reports, but it does not yet include the full Python automation source or the final REV_C fabrication archive. For complete reproducibility, those files should be versioned in a future repository cleanup.

---

## 15. Expected fabrication outputs

A complete release package should contain at minimum:

### Gerbers

- F.Cu;
- B.Cu;
- F.Paste;
- F.Silkscreen;
- B.Silkscreen;
- F.Mask;
- B.Mask;
- Edge.Cuts.

### Drill and assembly

- plated-through-hole Excellon file;
- non-plated-through-hole Excellon file;
- drill map;
- component position CSV;
- bill of materials;
- stack-up and impedance note;
- DRC report from the released source revision.

Gerber export example for KiCad 9:

```powershell
& "C:\Program Files\KiCad\9.0\bin\kicad-cli.exe" pcb export gerbers `
  --output .\fabrication\gerbers `
  --layers F.Cu,B.Cu,F.Paste,F.Silkscreen,B.Silkscreen,F.Mask,B.Mask,Edge.Cuts `
  .\CEM_RF_LPF_500_FINAL.kicad_pcb
```

Drill export example:

```powershell
& "C:\Program Files\KiCad\9.0\bin\kicad-cli.exe" pcb export drill `
  --output .\fabrication\drill `
  --format excellon `
  --excellon-units mm `
  --generate-map `
  --map-format pdf `
  --excellon-separate-th `
  .\CEM_RF_LPF_500_FINAL.kicad_pcb
```

---

## 16. Manufacturing checklist

Before ordering the PCB:

- use the corrected REV_C source, not the historical REV_B snapshot;
- confirm DRC is 0 violations and 0 unconnected items;
- verify every Gerber in an independent viewer;
- verify board dimensions and units;
- verify plated and non-plated drill separation;
- confirm the exact board-house stack-up;
- recalculate the 50 Ω width for that stack-up;
- confirm the exact SMA connector mechanical drawing;
- confirm 0603 component land patterns;
- verify solder-mask openings;
- verify via drill and annular-ring capability;
- verify copper-to-edge clearance;
- verify the BOM and current component availability;
- verify inductor Q and SRF for the full operating range;
- verify capacitor technology is C0G/NP0 or equivalent RF grade;
- order a small prototype quantity first;
- request impedance control or an impedance coupon where practical.

A DRC-clean board can still be electrically unsuitable at RF. Manufacturing review must include return-current paths, connector launches, trace impedance and parasitic behavior—not only geometric rule compliance.

---

## 17. Prototype assembly guidance

Recommended assembly sequence:

1. inspect the bare PCB for shorts, damaged mask and incorrect drill registration;
2. install the three shunt capacitors;
3. install the four series inductors;
4. install the SMA connectors with complete ground-tab soldering;
5. clean flux residues appropriate to the assembly process;
6. verify continuity from each ground pad and via to SMA ground;
7. verify there is no DC short from RF input to ground;
8. verify there is no DC short from RF output to ground;
9. inspect component alignment under magnification.

Because the component values are very small, excess solder, lifted parts, long solder fillets and incorrect component technology may noticeably shift the response.

---

## 18. VNA validation plan

Recommended first measurement:

1. warm up the VNA according to the instrument procedure;
2. use high-quality 50 Ω cables and minimize adapters;
3. perform SOLT calibration at the cable ends or use an appropriate fixture calibration;
4. sweep at least 10 MHz to 2 GHz;
5. record S11, S21, S12 and S22;
6. save Touchstone data;
7. compare measured S21 with the analytical response;
8. inspect passband loss, return loss, cutoff shift and first stopband resonance;
9. repeat on a second assembled PCB;
10. document the exact board stack-up and component lot.

Recommended plots:

- S21 magnitude from 10 MHz to 2 GHz;
- S11 and S22 magnitude;
- passband-only S21 zoom from 108 MHz to 500 MHz;
- group delay in the passband;
- measured-versus-modeled overlay.

Possible causes of model/measurement mismatch:

- actual Er or dielectric thickness differs from nominal;
- component Q/SRF differs from the simplified model;
- connector launch parasitics;
- incorrect or substitute components;
- via inductance and ground-return geometry;
- solder and pad parasitics;
- calibration-plane error;
- cable or adapter repeatability.

---

## 19. Power-path validation

Small-signal VNA data is not sufficient to establish a power rating.

For transmitter or PA use, perform a controlled power test with:

- a rated 50 Ω load;
- directional couplers;
- suitable attenuators;
- a spectrum analyzer or power meter;
- thermal monitoring;
- appropriate shielding and RF safety precautions.

Validate:

- insertion loss at operating power;
- harmonic attenuation;
- inductor current and temperature;
- capacitor RF voltage stress;
- connector heating;
- solder-joint heating;
- response drift with temperature;
- behavior under load mismatch.

No maximum RF power is claimed by this repository.

---

## 20. Known limitations

- The checked-in source snapshot is the historical REV_B package, not the corrected REV_C manufacturing master.
- The checked-in DRC report contains 20 `via_dangling` warnings.
- The final REV_C 0/0 DRC report is not yet committed.
- Physical VNA data is not yet published.
- Full-wave 2.5D/3D EM results are not yet published.
- The nominal FR-4 stack-up is not tied to one fabricator.
- The model uses engineering approximations for line and component parasitics.
- Vendor S-parameter files are not included.
- Maximum RF power has not been established.
- Thermal behavior has not been measured.
- Regulatory harmonic-emission compliance is not claimed.
- The filter has not been co-simulated with a specific PA output impedance.
- The current root-level footprint placement should be reorganized into `CUSTOM.pretty`.
- The complete automation source and production ZIP are not included in the present repository snapshot.

---

## 21. Engineering roadmap

Planned next steps:

- commit the corrected REV_C KiCad source;
- replace the historical DRC report with a revision-matched 0/0 report while preserving REV_B history in a separate folder;
- restore `CUSTOM.pretty` directory structure;
- add the full Python automation source;
- add BOM and component lifecycle information;
- add generated Gerbers and drills as a tagged release asset;
- perform ADS Momentum, Sonnet, CST or equivalent EM extraction;
- import vendor S-parameter models;
- manufacture at least two prototypes;
- publish calibrated VNA Touchstone files;
- correlate analytical and measured responses;
- evaluate power handling and thermal rise;
- optimize against a specific board-house stack-up;
- add automated repository validation and release checks.

---

## 22. Safety and regulatory notice

RF hardware can cause interference, equipment damage, burns and unlawful emissions when used incorrectly. The builder and operator are responsible for:

- complying with local spectrum regulations;
- using legal frequencies and power levels;
- using properly rated loads, cables, attenuators and instruments;
- preventing exposure to hazardous RF levels;
- ensuring that a transmitter system satisfies applicable harmonic and spurious-emission requirements.

This repository is provided as an engineering and educational project, not as regulatory certification.

---

## 23. Project metadata

| Field | Value |
|---|---|
| Public name | RFShield LPF |
| Internal engineering identifier | CEM_RF_LPF_500_FINAL |
| Intended final hardware revision | REV_C |
| Frequency range | 108–500 MHz |
| System impedance | 50 Ω |
| Filter order | 7 |
| PCB layers | 2 |
| Primary EDA tool | KiCad 9.0.7 |
| Analysis environment | Python, NumPy-based numerical model |
| Author | Cem Sondur |
| Project year | 2026 |

---

## 24. License and use

Unless a separate license file is added, all project files remain copyright of **Cem Sondur**.

No warranty is provided. Anyone fabricating, modifying or operating the design is responsible for independent review, measurement, safety and regulatory compliance.
