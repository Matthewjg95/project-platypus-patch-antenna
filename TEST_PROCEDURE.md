# Antenna Test Procedure
## 2.4 GHz Patch Antenna — M5Tab5 Integration
### Rev 1.0

---

## Overview

This document covers the complete test workflow for characterising the patch antenna PCB against the M5Tab5 stock chip antenna, culminating in a live 3D radiation pattern rendered on the M5Tab5 display using the BMI270 gyroscope for real-time orientation.

**Three designs under test:**
- Design A — inset feed, y₀ = 9.5 mm (baseline calculated)
- Design B — inset feed, y₀ = 7.5 mm (tighter match, wider notch)
- Design C — λ/4 transformer, Z_t ≈ 100 Ω, no notch

---

## Equipment List

| Item | Purpose | Notes |
|------|---------|-------|
| M5Tab5 | DUT and renderer | Stock chip antenna for baseline |
| MMCX pigtail, RG178, ~100mm | Connect PCB to M5Tab5 | SMP male to MMCX male |
| WiFi access point | Signal source | Fixed position, fixed channel, fixed power |
| Rotating platform | Pattern sweep | Lazy susan or tripod head, marked in 15° increments |
| Measuring tape | Distance calibration | Mark floor at 1, 2, 3, 4, 5 m from AP |
| Non-metallic surface | Reduce reflections | Wooden table or cardboard |
| USB-C power | Power M5Tab5 during test | Keeps battery level consistent |
| Laptop with serial monitor | Log RSSI data | Arduino Serial Plotter or minicom |
| SD card (FAT32) | On-device logging | 512 MB minimum |
| Python 3 environment | Data processing | numpy, scipy, json |
| Masking tape | Mark floor grid | Label angular positions |

**Optional (strongly recommended for Rev 3):**
- NanoVNA or similar VNA → measure S11 directly on each PCB before RF testing
- Calibration resistor set → verify VNA port impedance

---

## Phase 0 — Environment Setup

### 0.1 Access point configuration
- Lock AP to 2.4 GHz band only (disable 5 GHz if possible)
- Lock to channel 6 (centre of 2.4 GHz band, avoids edge effects)
- Disable beamforming / adaptive antenna features (these change power dynamically)
- Set transmit power to maximum and lock it there
- Note: AP should be elevated 1.2–1.5 m above floor (approximate head height)

### 0.2 Room preparation
- Remove large metal objects from the test area (chairs with metal legs, filing cabinets)
- Mark the floor with masking tape: a cross centred under the AP, arms extending 5 m in 4 cardinal directions
- Mark 15° increments around the rotation centre point (3 m from AP)
- Run a quick sanity check: walk the test path with the M5Tab5 running RSSI logger, confirm RSSI decreases monotonically with distance and is stable (< 2 dBm variation over 5 s at fixed position)

### 0.3 M5Tab5 firmware

Flash the following logging sketch before starting. The sketch logs RSSI continuously at 1 Hz to both Serial and SD card.

```cpp
#include <WiFi.h>
#include <SD.h>
#include <SPI.h>

const char* SSID     = "YOUR_AP_SSID";
const char* PASSWORD = "YOUR_AP_PASS";
const char* LOG_FILE = "/rssi_log.csv";

File logFile;

void setup() {
  Serial.begin(115200);
  WiFi.begin(SSID, PASSWORD);
  while (WiFi.status() != WL_CONNECTED) delay(500);

  SD.begin(/* your SD CS pin */);
  logFile = SD.open(LOG_FILE, FILE_APPEND);
  logFile.println("timestamp_ms,design_id,theta_deg,phi_deg,rssi_dbm,note");
  logFile.flush();
  Serial.println("timestamp_ms,design_id,theta_deg,phi_deg,rssi_dbm,note");
}

void loop() {
  wifi_ap_record_t ap;
  esp_wifi_sta_get_ap_info(&ap);
  int8_t rssi = ap.rssi;

  String row = String(millis()) + ",?,0,0," + String(rssi) + ",";
  logFile.println(row);
  logFile.flush();
  Serial.println(row);
  delay(1000);
}
```

**During testing:** edit `design_id`, `theta_deg`, `phi_deg` fields manually or via Serial command before each measurement position. The `note` field is free text for marking events.

---

## Phase 1 — Baseline (Stock Chip Antenna)

**Goal:** Establish ground truth RSSI performance before any external antenna is connected. Every result in Phases 2–3 is expressed as ΔG = RSSI_measured − RSSI_baseline at the same position.

### 1.1 Static range test

1. Place M5Tab5 flat on non-metallic surface, screen facing up, USB-C at the back
2. Fix orientation — do not rotate between measurements
3. Start RSSI logger, set design_id = "BASELINE", note = "static_range"
4. Walk to each distance mark (1, 2, 3, 4, 5 m) in the direction facing the AP
5. At each mark: stand still 5 s, record 10 RSSI readings, take median
6. Repeat in all 4 cardinal directions (16 data points total per orientation)

**Record in spreadsheet:**

| Distance (m) | Direction | RSSI median (dBm) | Std dev |
|---|---|---|---|
| 1 | N | | |
| 2 | N | | |
| ... | | | |

### 1.2 Azimuth pattern (2D sweep)

1. Place M5Tab5 on rotating platform at 3 m from AP, screen facing up (patch horizontal)
2. Align 0° to face directly at AP
3. Set note = "baseline_azimuth"
4. Rotate through 0°, 15°, 30° … 345° (24 positions)
5. At each position: pause 3 s, collect 5 RSSI samples, record median
6. Export as CSV: `baseline_azimuth.csv`

### 1.3 Key metrics to record

| Metric | Formula | Value |
|--------|---------|-------|
| Peak RSSI | max(RSSI) | |
| Null RSSI | min(RSSI) | |
| Peak-to-null ratio | peak − null (dBm) | |
| RSSI at 3 m broadside | direct reading | |
| Distance at −70 dBm | interpolate from range test | |

---

## Phase 2 — Per-Design Static Comparison

**Goal:** Quickly rank Designs A, B, C before committing to the full 3D sweep. Run the same tests as Phase 1 with each design connected.

### 2.1 Connecting the external antenna

1. Plug MMCX pigtail into M5Tab5 MMCX port (the one labelled for external antenna)
2. In firmware or hardware, switch M5Tab5 to external antenna mode (RF switch — check Tab5 schematic; GPIO controls the antenna switch on ESP32-C6-MINI-1U)
3. Connect SMP end of pigtail to J1 on the design under test
4. Mount design PCB patch-face-up, in the same orientation as the baseline M5Tab5 position

### 2.2 RF switch note

The M5Tab5 ESP32-C6-MINI-1U has an integrated RF switch between the on-chip antenna and the MMCX port. You must assert the correct GPIO to route RF to the external port. Check the Tab5 schematic for the exact GPIO number — it is likely driven by a SKY13351 or similar SPDT. Without switching, both the chip antenna and external antenna may be active simultaneously, corrupting results.

```cpp
// Example — verify GPIO number from Tab5 schematic
#define ANT_SEL_GPIO 14  // CONFIRM THIS FROM SCHEMATIC
pinMode(ANT_SEL_GPIO, OUTPUT);
digitalWrite(ANT_SEL_GPIO, HIGH);  // HIGH = external MMCX
```

### 2.3 Repeat static tests

For each design (A, B, C):
- Repeat 1.1 (static range test) with design_id = "A", "B", or "C"
- Repeat 1.2 (azimuth sweep) with same design_id
- Calculate ΔG at each position vs baseline

### 2.4 Comparison table

| Position | Baseline (dBm) | Design A (dBm) | ΔG_A | Design B (dBm) | ΔG_B | Design C (dBm) | ΔG_C |
|---|---|---|---|---|---|---|---|
| 3 m broadside | | | | | | | |
| 3 m 90° | | | | | | | |
| 3 m 180° | | | | | | | |
| 5 m broadside | | | | | | | |

**Decision gate:**
- ΔG > +4 dBm broadside → design is working, proceed to Phase 3
- ΔG 2–4 dBm → marginal, file/trim the notch or transformer section, re-test
- ΔG < +2 dBm → probable impedance mismatch, check connector seating, then use VNA

---

## Phase 3 — Full 3D Radiation Pattern

**Goal:** Collect (θ, φ, RSSI) over a full hemisphere for the winning design. This data feeds the M5Tab5 renderer.

### 3.1 Angular grid

- φ (azimuth): 0° to 345° in 15° steps = 24 positions
- θ (elevation): 0° (horizontal) to 90° (vertical, patch facing up) in 15° steps = 7 elevation planes
- Total positions: 24 × 7 = **168 measurements**
- Estimated time: ~50 minutes at 3 s per position

### 3.2 Jig setup

The antenna PCB must be rotatable in both azimuth and elevation. A practical approach:
1. Azimuth: rotating platform as used in Phase 2
2. Elevation: tilt the entire platform by propping up one edge at 15° increments (a stack of books works)
3. Mark each elevation angle with a bubble level or phone inclinometer app

### 3.3 Data collection procedure

For each elevation plane θ:
1. Set jig to elevation θ
2. Rotate through all 24 azimuth positions
3. At each (θ, φ): pause 2 s, collect 10 RSSI samples, record median and std deviation
4. If std > 3 dBm at any position: wait 5 s, retake. Discard outliers (multipath spike).

**CSV format:**
```
theta_deg,phi_deg,rssi_median_dbm,rssi_std,n_samples,design_id
0,0,-48.2,1.1,10,A
0,15,-49.0,0.8,10,A
0,30,-52.1,1.4,10,A
...
```

### 3.4 Data validation checks

After collection, run these sanity checks before processing:

```python
import pandas as pd
import numpy as np

df = pd.read_csv('pattern_3d.csv')

# Check coverage
assert len(df) == 168, f"Missing positions: {168 - len(df)}"

# Check for outliers
suspect = df[df['rssi_std'] > 3.0]
if len(suspect) > 0:
    print(f"High variance positions ({len(suspect)}):")
    print(suspect[['theta_deg','phi_deg','rssi_std']])

# Check range is physically plausible
assert df['rssi_median_dbm'].between(-90, -20).all(), "RSSI out of range"

print("Range:", df['rssi_median_dbm'].min(), "to", df['rssi_median_dbm'].max(), "dBm")
print("Spread:", df['rssi_median_dbm'].max() - df['rssi_median_dbm'].min(), "dB")
```

Expected spread for a patch antenna: 8–15 dB between broadside peak and worst null.

---

## Phase 4 — M5Tab5 3D Visualization Pipeline

### 4.1 Python: CSV to Three.js JSON

```python
import pandas as pd
import numpy as np
import json

df = pd.read_csv('pattern_3d.csv')

# Normalise RSSI to 0–1 radius
rssi = df['rssi_median_dbm'].values
r_norm = (rssi - rssi.min()) / (rssi.max() - rssi.min())

vertices = []
colors = []

for i, row in df.iterrows():
    theta = np.radians(row['theta_deg'])
    phi   = np.radians(row['phi_deg'])
    r     = r_norm[i]

    x = r * np.sin(theta) * np.cos(phi)
    y = r * np.cos(theta)
    z = r * np.sin(theta) * np.sin(phi)
    vertices.append([round(x, 4), round(y, 4), round(z, 4)])

    # Colour map: teal (high) to gray (low)
    t = r_norm[i]
    colors.append([
        round(0.11 + (1 - t) * 0.4, 3),   # R
        round(0.62 + (1 - t) * 0.0, 3),   # G
        round(0.46 + (1 - t) * 0.1, 3),   # B
    ])

output = {
    "vertices": vertices,
    "colors": colors,
    "rssi_min": float(rssi.min()),
    "rssi_max": float(rssi.max()),
    "design": "A",
    "freq_ghz": 2.4
}

with open('pattern.json', 'w') as f:
    json.dump(output, f)

print(f"Exported {len(vertices)} vertices")
```

### 4.2 Three.js renderer on M5Tab5

The M5Tab5 runs a lightweight web server (ESP-IDF + HTTPS server or Arduino AsyncWebServer). The renderer is an HTML page served from SPIFFS or SD card, loaded in the M5Tab5 browser or a custom LVGL WebView.

```javascript
// Core Three.js pattern renderer
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, w/h, 0.01, 10);
camera.position.set(0, 0, 2.5);

// Load pattern data
const data = await fetch('/pattern.json').then(r => r.json());

const geo = new THREE.BufferGeometry();
const positions = new Float32Array(data.vertices.flat());
const colors    = new Float32Array(data.colors.flat());
geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
geo.setAttribute('color',    new THREE.BufferAttribute(colors, 3));

const mat = new THREE.PointsMaterial({
  size: 0.05,
  vertexColors: true,
  sizeAttenuation: true
});
scene.add(new THREE.Points(geo, mat));

// BMI270 gyro → quaternion → Three.js camera rotation
// Read quaternion from BMI270 via I2C, apply to scene rotation
function applyGyro(qw, qx, qy, qz) {
  scene.quaternion.set(qx, qy, qz, qw);
}
```

### 4.3 BMI270 orientation feed

The BMI270 on M5Tab5 outputs raw accelerometer + gyroscope data. Use a complementary filter or Mahony filter to produce a quaternion, then feed it to Three.js.

```cpp
// Simplified complementary filter for orientation
// dt = loop time in seconds, typically 0.02 (50 Hz)
float pitch = alpha * (pitch + gyr_x * dt) + (1 - alpha) * atan2(acc_y, acc_z);
float roll  = alpha * (roll  + gyr_y * dt) + (1 - alpha) * atan2(acc_x, acc_z);
// Send pitch, roll, yaw to renderer via WebSocket
ws.send("{\"p\":" + String(pitch) + ",\"r\":" + String(roll) + "}");
```

---

## Phase 5 — Iteration Decision Gate

### 5.1 Results interpretation

| Outcome | Meaning | Action |
|---------|---------|--------|
| ΔG > +5 dBm broadside, pattern shows clear lobe | Design working well | Proceed to Rev 3 production run |
| ΔG +2–5 dBm, pattern visible | Partial match, some loss | File/trim to adjust y₀ or λ/4 length, re-test |
| ΔG < +2 dBm, pattern near-omnidirectional | Impedance mismatch dominant | VNA S11 measurement required |
| ΔG negative (worse than stock) | Connector or RF switch issue | Check pigtail, antenna switch GPIO |

### 5.2 Physical tuning without PCB respin

**Inset feed (Designs A, B):**
- Notch too shallow (Rin > 50 Ω): carefully file 0.3 mm deeper into the notch slot with a needle file. Re-test after each pass.
- Notch too deep (Rin < 50 Ω): add a thin strip of copper tape (~0.3 mm) to the notch wall to reduce effective depth.
- Patch resonance shifted: file the radiating edge to shift frequency up, or add copper tape strip to shift down.

**Quarter-wave transformer (Design C):**
- Section too long (match frequency too low): score and trim 1–2 mm from the far end of the transformer section with a razor knife.
- Section too short (match frequency too high): not field-fixable. Note the delta and correct in Rev 3.

### 5.3 Rev 3 changes to implement based on results

Record the winning y₀ or transformer length, and any frequency offset, here:

| Parameter | Calculated | Measured optimal | Delta |
|-----------|-----------|-----------------|-------|
| Inset depth y₀ | 9.5 mm | | |
| Patch length L | 28.8 mm | | |
| Transformer length | calculated | | |
| Resonant frequency | 2.40 GHz | | |

---

## Data Files Checklist

By the end of testing, you should have:

- [ ] `baseline_range.csv` — Phase 1 static range
- [ ] `baseline_azimuth.csv` — Phase 1 azimuth sweep
- [ ] `design_A_range.csv` — Phase 2
- [ ] `design_A_azimuth.csv` — Phase 2
- [ ] `design_B_range.csv` — Phase 2
- [ ] `design_B_azimuth.csv` — Phase 2
- [ ] `design_C_range.csv` — Phase 2
- [ ] `design_C_azimuth.csv` — Phase 2
- [ ] `pattern_3d.csv` — Phase 3 full hemisphere (winning design)
- [ ] `pattern.json` — Phase 4 processed for Three.js
- [ ] Photos of each design mounted on test jig
- [ ] VNA S11 screenshots (if available)

---

## Notes & Known Limitations

- RSSI-based measurements are affected by multipath, furniture, and human body proximity. Stand at least 1 m behind the M5Tab5 during measurements and always in the same relative position.
- The M5Tab5's WiFi RSSI resolution is 1 dBm. Differences < 2 dBm are within measurement noise and should not be interpreted as real gain changes.
- This procedure characterises receive sensitivity pattern, not transmit pattern. For a passive reciprocal antenna (which this is), transmit and receive patterns are identical by the reciprocity theorem — so receive RSSI measurements directly characterise the antenna's gain pattern in both directions.
- Indoor multipath will smear the pattern compared to a true anechoic result. The null depth will appear shallower than the true free-space pattern. This is acceptable for comparative testing — the key metrics are ΔG and relative pattern shape, not absolute dBi values.
