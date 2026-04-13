# TraverseLink

**Multi-Station Traverse Coordinate Transformation Tool**

A browser-based surveying tool for transforming total station relative coordinates into absolute coordinates through multi-station traverse. Supports GPS control points at any position in the traverse chain, automatic shared-point detection, bidirectional coordinate propagation, and linear closure adjustment.

---

## Live Demo

Deploy `index.html` to any static host (GitHub Pages, Netlify) and share the URL — no server-side code required.

---

## Key Features

### Coordinate Transformation
- **Helmert 4-parameter transformation** for the horizontal plane (translation, rotation, scale)
- **Linear regression** for vertical (Z/H) coordinate transfer
- **Scale validation** with color-coded warnings:
  - Normal: 0.9990 – 1.0010
  - Warning ⚠ (amber): 0.9950 – 0.9990 or 1.0010 – 1.0050
  - Alert ⚠ (red): outside 0.9500 – 1.0500

### Multi-Station Traverse
- **Unlimited stations** — add or remove stations dynamically
- **Mixed GPS/relay mode** — any station can be a GPS anchor or relay station
- **Bidirectional propagation** — iterative solving allows shared points to come from preceding or following stations in any order
- **Flexible GPS placement** — GPS control points can be at the start, middle, end, or any combination of positions in the traverse chain

### Closure Adjustment
- **Automatic segment detection** — identifies traverse segments bounded by GPS stations on both ends
- **Linear interpolation adjustment** — distributes closure error proportionally across intermediate relay stations
- **Closure summary table** — displays ΔE, ΔN, ΔH, and 3D closure per segment with color-coded quality indicators
- **Open-end handling** — single-anchor segments (no closure possible) are clearly labeled in the log

### Data Entry
- **Paste-in format** — directly paste data in `PointName,X,Y,Z,Attribute` format (comma or tab separated)
- **Auto shared-point detection** — points with identical names across stations are automatically treated as control points; no manual mapping required
- **GPS pairing by name** — GPS absolute coordinates are matched to total station points by name automatically

### Results
- **Per-station results** — individual Helmert parameters, RMS, shared points, and adjustment corrections
- **Merged result table** — all unique points from all stations in a single deduplicated table with source station column
- **RMS color coding**:
  - Green: < 0.01 m (excellent)
  - Yellow: 0.01 – 0.05 m (acceptable)
  - Red: ≥ 0.05 m (needs review)
- **One-click TSV export** — copy results directly into Excel

---

## Input Format

### Total Station (relative coordinates)
Paste directly — comma or tab separated:
```
PointName,X,Y,Z,Attribute
P1,0.000,0.000,10.250,Control
S1,30.000,160.000,11.650,Shared
T1,55.000,120.000,10.900,Power manhole
```
`Attribute` is optional and accepts any text (control point, manhole type, road edge, etc.).

### GPS Absolute Coordinates (GPS stations only)
```
PointName,E,N,H
P1,246440.000,2665050.000,52.250
P2,246501.745,2665128.661,52.820
```
Point names must exactly match the total station data. Case-sensitive.

---

## Workflow

```
1. Open index.html in browser
2. Click [+ 新增測站] to add stations (or load sample data)
3. For each station:
   ├── Set type: [有 GPS] GPS station  or  [傳遞站] Relay station
   ├── Paste total station data (PointName,X,Y,Z,Attribute)
   └── GPS station only: paste GPS coordinates (PointName,E,N,H)
4. Click [▶ 執行全站計算]
5. Review results:
   ├── Iteration log — stations solved per round
   ├── Closure adjustment — segment errors and corrections applied
   ├── Per-station results — Helmert params, RMS, adjustment column
   └── Merged result table — all unique points across all stations
6. Copy TSV to clipboard → paste into Excel
```

---

## How Bidirectional Propagation Works

```
Round 1: Solve all GPS stations → add their points to the global library
Round 2: For each unsolved relay station, check if ≥2 shared points
         are now in the library → solve if possible → add to library
Round N: Repeat until all stations are solved or no further progress
```

Relay stations can reference shared points from stations that appear **later** in the list, not just earlier ones.

---

## How Closure Adjustment Works

When a traverse segment has GPS anchors on **both ends**:

```
GPS Station A ── Relay 1 ── Relay 2 ── GPS Station B
     ↓                                       ↓
Computes shared                       Computes shared
point coords                          point coords
     └──── closure error = B − A ──────────┘
           distributed proportionally:
           Relay 1: corrected by 1/3 × error
           Relay 2: corrected by 2/3 × error
```

Single-anchor segments (relay stations before the first GPS station or after the last) are coordinate-propagated only — no closure adjustment without a second anchor.

---

## Coordinate Transformation

**Horizontal — Helmert 4-parameter (least squares):**

```
E = a·X − b·Y + tx
N = b·X + a·Y + ty

Scale    = √(a² + b²)
Rotation = atan2(b, a)
```

**Vertical — linear regression:**

```
H = kz·Z + dz
```

---

## Files

```
traverselink/
├── index.html      # Complete application — single file, zero dependencies
└── README.md       # This file
```

No build step. No npm. No server. Open `index.html` and it works.

---

## Browser Requirements

| Browser | Minimum |
|---------|---------|
| Chrome  | 90+     |
| Edge    | 90+     |
| Firefox | 88+     |
| Safari  | 14+     |

All coordinate calculations run fully offline — no internet connection required.

---

## Deployment

**Local use:**
Download `index.html` and open in any browser.

**GitHub Pages:**
1. Create a repository and upload `index.html` as the root file
2. Settings → Pages → Branch: main → Save
3. Share `https://username.github.io/repository-name`

**Netlify Drop:**
Drag and drop `index.html` at [app.netlify.com/drop](https://app.netlify.com/drop) for an instant URL.

---

## Notes

- Developed and tested with **Leica TS03** total station data and **TWD97** GPS coordinates.
- The tool runs entirely in the browser — no data is sent to any server.

---

## License

MIT License — free to use, modify, and distribute.
