# TraverseLink

**Multi-Station Traverse Coordinate Transformation Tool**

A browser-based surveying tool for transforming total station relative coordinates into absolute coordinates through multi-station traverse. Supports GPS control points at any position in the traverse chain, automatic shared-point detection, bidirectional coordinate propagation, and linear closure adjustment.

---

## Live Demo

Deploy `index.html` to any static host (GitHub Pages, Netlify) and share the URL — no server-side code required.

---

## Key Features

### Coordinate Transformation
- **Small-area terrain survey mode** for the horizontal plane: translation + rotation, with scale fixed at `1.000000`
- **Height transfer by translation**: `H = Z + dz` (`kz = 1`)
- **Quality status** based on RMS/SD 3D and closure 3D:
  - Good: < 0.020 m
  - Attention: 0.020 – < 0.050 m
  - Needs review: ≥ 0.050 m

### Multi-Station Traverse
- **Unlimited stations** — add or remove stations dynamically
- **Mixed GPS/relay mode** — any station can be a GPS anchor or relay station
- **Bidirectional propagation** — iterative solving allows shared points to come from preceding or following stations in any order
- **Flexible GPS placement** — GPS control points can be at the start, middle, end, or any combination of positions in the traverse chain
- **Clickable traverse chain** — select stations directly from the chain and see per-station status at a glance

### Closure Adjustment
- **Automatic segment detection** — identifies traverse segments bounded by GPS stations on both ends
- **Linear interpolation adjustment** — distributes closure error proportionally across intermediate relay stations
- **Closure summary table** — displays ΔE, ΔN, ΔH, and 3D closure per segment with color-coded quality indicators
- **Open-end handling** — single-anchor segments (no closure possible) are clearly labeled in the log

### Data Entry
- **Paste-in format** — directly paste data in `PointName,X,Y,Z,Attribute` format (comma or tab separated)
- **Auto shared-point detection** — points with identical names across stations are automatically treated as control points; no manual mapping required
- **GPS pairing by name** — GPS absolute coordinates are matched to total station points by name automatically
- **Inline input diagnostics** — warns about missing fields, invalid numbers, duplicate point names, insufficient control/shared points, and GPS-only points

### Results
- **Priority result tabs** — calculation process and closure adjustment are shown immediately below the title
- **Health summary** — station status counts for Good, Attention, Needs review / incomplete, and closure segments
- **Per-station results** — rotation, RMS/SD, shared points, and adjustment corrections
- **Merged result table** — all unique points from all stations in a single deduplicated table with source station column
- **Point source type** — merged outputs distinguish `GPS-only`, `GPS+EDM`, and `EDM`
- **RMS color coding**:
  - Green: < 0.020 m (good)
  - Yellow: 0.020 – < 0.050 m (attention)
  - Red: ≥ 0.050 m (needs review)
- **Project save/load** — save project JSON, load existing project JSON, load sample data, or clear all from the traverse chain control area
- **One-click exports** — copy TSV directly into Excel, download merged CSV, or download an HTML report

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
PointName,E,N,H,Attribute
P1,246440.000,2665050.000,52.250,Control point
P2,246501.745,2665128.661,52.820
```
`Attribute` is optional. Point names must exactly match the total station data when used for pairing. Case-sensitive.

---

## Workflow

```
1. Open index.html in browser
2. Use the 座標傳遞鏈 control area to add stations, load/save projects, load sample data, or clear all
3. For each station:
   ├── Set type: [有 GPS] GPS station  or  [傳遞站] Relay station
   ├── Paste total station data (PointName,X,Y,Z,Attribute)
   └── GPS station only: paste GPS coordinates (PointName,E,N,H)
4. Click [▶ 執行全站計算]
5. Review results:
   ├── Health summary — Good / Attention / Needs review status
   ├── Calculation process — stations solved per round
   ├── Closure adjustment — segment errors and corrections applied
   ├── Per-station results — rotation, RMS/SD, adjustment column
   └── Merged result table — all unique points across all stations
6. Copy TSV, download CSV, or download the HTML report
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

**Horizontal — small-area rigid transformation (scale fixed at 1):**

```
E = cosθ·X − sinθ·Y + tx
N = sinθ·X + cosθ·Y + ty

Scale    = 1.000000
Rotation = θ
```

This mode is intended for small-area terrain surveys, typically within about 500 m, where the total station and absolute coordinate systems are both in meters and the local measurement scale should be preserved. Fixing scale helps expose control-point, point-name, height, or observation issues instead of absorbing them into a free scale factor.

**Vertical — height translation:**

```
H = Z + dz
kz = 1
```

## Quality Thresholds

The health status shown in the top result tabs, traverse chain, station summary, closure table, and report uses these defaults:

| Status | Threshold |
|--------|-----------|
| Good | RMS/SD 3D or closure 3D < 0.020 m |
| Attention | 0.020 m to < 0.050 m |
| Needs review | ≥ 0.050 m |

These thresholds are tuned for general small-area terrain survey checks. For stricter engineering layout or structural survey work, use tighter acceptance criteria outside this tool.

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
