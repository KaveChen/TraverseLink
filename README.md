# TraverseLink

**多站連測座標轉換工具 | Multi-Station Traverse Coordinate Transformation Tool**

A browser-based tool for surveyors using total stations (e.g. Leica TS03) to transform relative local coordinates into absolute coordinates through multi-station traverse, with support for GPS control points and automatic shared-point detection.

---

## Features

- **Multi-station traverse** — No limit on the number of stations
- **Mixed mode** — Some stations with GPS control points, others connected via shared points
- **Bidirectional propagation** — Shared points can come from either preceding or following stations; iterative solving handles any connection order
- **Auto shared-point detection** — Points with identical names across stations are automatically treated as shared control points, no manual mapping required
- **Paste-in data entry** — Directly paste data in `Point,X,Y,Z,Attribute` format (comma or tab separated)
- **Helmert 4-parameter transformation** — Handles translation, rotation, and scale in the horizontal plane; separate linear regression for Z (elevation)
- **RMS quality indicators** — Color-coded RMS per station (E / N / H / 2D / 3D)
- **Per-station results + merged table** — Individual station results and a deduplicated merged table with all unique points
- **TSV export** — One-click copy for pasting directly into Excel

---

## Input Format

### Total Station (relative coordinates)
```
PointName,X,Y,Z,Attribute
A1,0.000,0.000,10.250,Control
S1,30.000,160.000,13.500,Shared
T1,55.200,120.400,11.900,Manhole
```
Attribute column is optional and can be left empty.

### GPS (absolute coordinates)
```
PointName,E,N,H
A1,246523.450,2665134.220,52.180
A2,246648.120,2665222.890,54.710
```
Point names must match the total station data exactly. The system pairs them automatically.

---

## Usage

1. Open `index.html` in any browser (Chrome / Edge / Firefox)
2. Click **+ 新增測站** to add stations, or click **載入範例資料** to load sample data
3. For each station:
   - Set station type: **有 GPS** (has GPS) or **傳遞站** (relay station)
   - Paste total station data into the upper text area
   - If GPS station: paste GPS absolute coordinates into the lower text area
   - If relay station: points with names matching any other station are automatically detected as shared points
4. Click **▶ 執行全站計算** to run the calculation
5. Results appear in two sections:
   - **各站計算結果** — Per-station results with RMS and shared point info
   - **合併結果** — All unique points merged into one table
6. Use **複製各站結果 (TSV)** or **複製合併結果 (TSV)** to copy results to clipboard, then paste into Excel

---

## How Bidirectional Propagation Works

The calculation uses iterative solving:

1. **Round 1** — Calculate all GPS stations first, add results to the global point library
2. **Round 2+** — For each uncalculated relay station, check if enough shared points are now in the library; calculate if possible, then add its points to the library
3. Repeat until all stations are solved or no further progress is possible

This means relay stations can reference shared points from stations that appear later in the list, not just earlier ones.

---

## Coordinate Transformation Method

**Horizontal (X, Y):** Helmert 4-parameter transformation (least squares)

$$E = a \cdot X - b \cdot Y + t_x$$
$$N = b \cdot X + a \cdot Y + t_y$$

Where scale $= \sqrt{a^2 + b^2}$ and rotation $= \arctan(b/a)$

**Vertical (Z):** Linear regression

$$H = k_z \cdot Z + d_z$$

---

## Files

```
traverselink/
├── index.html      # Main application (single file, no dependencies)
└── README.md       # This file
```

No installation, no dependencies, no internet connection required after the page loads.

---

## Browser Compatibility

| Browser | Support |
|---------|---------|
| Chrome 90+ | ✓ |
| Edge 90+ | ✓ |
| Firefox 88+ | ✓ |
| Safari 14+ | ✓ |

---

## License

MIT License — free to use, modify, and distribute.

---

*Built for field surveyors. Tested with Leica TS03 total station data and TWD97 GPS coordinates.*
