# TraverseLink

目前版本：雙解平差、專案共用 GPS、單一計算入口，以及電子地圖／正射影像底圖切換。下方「目前操作方式」與各功能章節以現行程式為準；歷史調整紀錄僅供追溯。

**Multi-Station Traverse Coordinate Transformation Tool**

A browser-based surveying tool for transforming total station relative coordinates into absolute coordinates through multi-station traverse. Supports GPS control points at any position in the traverse chain, automatic shared-point detection, bidirectional coordinate propagation, a geometry-preserving primary solution, and a simultaneous least-squares reference solution.

---

## Live Demo

Deploy `index.html` to any static host (GitHub Pages, Netlify) and share the URL — no server-side code required.

---

## Key Features

### Coordinate Transformation
- **Default small-area terrain survey mode** for the horizontal plane: translation + rotation, with scale fixed at `1.000000`
- **Optional scale mode (simultaneous reference only)** for tunnel or long-distance traverse work: enable scale when the traverse distance is around 1–2 km and the free scale factor better represents field conditions
- **Height transfer by translation**: `H = Z + dz` (`kz = 1`)
- **Quality status** based on RMS/SD 3D and closure 3D:
  - Good: < 0.020 m
  - Attention: 0.020 – < 0.050 m
  - Needs review: ≥ 0.050 m

### Multi-Station Traverse
- **Unlimited stations** — add or remove stations dynamically
- **Automatic GPS matching**: one shared GPS list; no per-station GPS/relay switch. Stations with no direct GPS match may use shared-point connections.
- **Bidirectional propagation** — iterative solving allows shared points to come from preceding or following stations in any order
- **Flexible GPS placement** — GPS control points can be at the start, middle, end, or any combination of positions in the traverse chain
- **Clickable traverse chain** — select stations directly from the chain and see per-station status at a glance
- **Reference scale switch**: the primary always uses K=1; the fixed/free scale setting applies only to the simultaneous reference solution.

### Two Adjustment Solutions
- **Primary: preserve internal geometry**. Fit the GPS-free station network first, then fit each connected component rigidly to its unique GPS controls with K=1 and a height offset. Independent stations receive separate GPS fits.
- **Reference: simultaneous coordination**. Keep the original equal GPS/EDM coordinate-weight solve, displayed alongside primary coordinates and differences. The optional free-scale setting applies only to this reference.
- **Exports**. Standard station/merged tables, map, TSV and merged CSV use the primary solution. A dedicated comparison CSV contains both sets of coordinates. The HTML report and project JSON retain both.

### Simultaneous Reference Adjustment
- **Joint least-squares adjustment**: GPS and relay stations participate together, using equal GPS/EDM coordinate weights.
- **One GPS constraint per point name**: duplicate rows across or within stations do not increase GPS weight; differing coordinate tuples are averaged once each with a warning.
- **Source records and residuals**: retain GPS source stations and show transformed EDM minus joint coordinates, and joint coordinates minus merged GPS coordinates.
- **Shared-point summary**: reports post-adjustment differences between station observations, including adjacent GPS stations. This is not a classical traverse closure.
- **Soft GPS constraints**: adjusted results may differ from GPS inputs; disconnected or insufficiently controlled stations remain unsolved.

### Data Entry
- **Paste-in format** — directly paste data in `PointName,X,Y,Z,Attribute` format (comma or tab separated)
- **Auto shared-point detection** — points with identical names across stations are automatically treated as control points; no manual mapping required
- **GPS pairing by name** — GPS absolute coordinates are matched to total station points by name automatically
- **Inline input diagnostics** — warns about missing fields, invalid numbers, duplicate point names, insufficient control/shared points, and GPS-only points

### GIS Map and Basemaps
- **Taiwan electronic map**: NLSC EMAP, selected by default.
- **Orthophoto**: NLSC PHOTO2, selectable in the upper-right layer control.
- **Exclusive basemaps**: select one basemap at a time; GPS+EDM, GPS-only and EDM overlays can be toggled separately.
- **Selection lifetime**: the current basemap is retained when the map is rebuilt in the same page session. It is not stored in project JSON or browser storage; reopening the page starts with EMAP.
- **Data and connectivity**: the map shows primary coordinates. Basemap tiles and externally hosted map libraries require an internet connection; coordinate calculations run locally.

### Results
- **Single calculation action** — enter station observations, paste shared GPS controls, then use **計算並查看結果** below the GPS input. The duplicate action in the traverse-chain toolbar has been removed.
- **Result navigation** — calculation moves keyboard focus and the viewport to the summary below the action, followed by detailed results. It always opens the calculation summary first.
- **Calculation status** — a separate live status shows completed stations and quality warnings; editing calculated inputs clears old outputs and prompts recalculation.
- **Health summary** — station status counts for Good, Attention, Needs review / incomplete, and closure segments
- **Per-station results** — rotation, RMS/SD, shared points, and adjustment corrections
- **Merged result table** — all unique points from all stations in a single deduplicated table with source station column
- **Point source type** — merged outputs distinguish `GPS-only`, `GPS+EDM`, and `EDM`
- **RMS color coding**:
  - Green: < 0.020 m (good)
  - Yellow: 0.020 – < 0.050 m (attention)
  - Red: ≥ 0.050 m (needs review)
- **Project save/load** — save or load project JSON from the shared project toolbar above the workspace; sample data and survey clearing remain in the traverse chain control area
- **Merged output order** — Point name, Attribute, E, N, H, X/Y, WGS84 (N/E), Type, Source stations; consistent across the table, TSV, CSV, and HTML report
- **One-click exports** — copy TSV directly into Excel, download merged CSV, or download an HTML report

---

## Input Format

### Project Backup and CAD Workspace

Open `TraverseLink-CAD.html` for both surveying and AutoCAD script export. The shared **儲存專案 / 載入專案** toolbar saves both workspaces to one local JSON file, including projects that have not been calculated yet.

Version 2 project files contain:

- `schemaVersion: 2` and `savedAt`.
- `survey`: project-wide `gpsPasteText` (stored once), station observations, selected station, scale mode, calculation timestamp, results, closures, merged points, logs, TSV data, and map zone.
- `cad`: raw control-point and manhole inputs, the inherit-layer option, applied custom Code-to-block mappings, and custom mapping editor text (including unapplied edits).
- `ui`: selected workspace and calculation/closure result tab.

Loading restores the input fields, rebuilds saved calculation results and CAD checks, and allows exporting again. Invalid project data is rejected; rendering failures restore the previous project. Version 1 survey-only JSON files are supported and reset the CAD workspace to empty inputs and built-in mappings. Imported mappings apply to the current project without replacing the browser's separately saved mapping defaults.

`index.html` remains the survey-only entry point. It reads both JSON versions and preserves embedded CAD data when saving a version 2 project, even though it does not display the CAD workspace. Older copies of the application that only support version 1 cannot read version 2 files.

CAD control input uses `PointName X Y Z`; manhole input uses `PointName Code X Y [Z]`. Export produces an AutoCAD **`.scr`** file. The merged result's Attribute field remains free text and must contain a valid CAD Code for direct use in manhole input.

Run regressions with `node tests/project-roundtrip.test.js` (no npm install required).

Run the survey interaction regressions with `node tests/survey-workflow.test.js`. In PowerShell, run all tests with `Get-ChildItem tests -Filter '*.test.js' | ForEach-Object { node $_.FullName; if ($LASTEXITCODE -ne 0) { throw "Test failed: $($_.Name)" } }`.

### UI / UX review — 2026-10-20

- Removed the competing calculation buttons in both entry points and arranged the survey flow as stations → shared GPS → calculation → quality summary → detailed results.
- Corrected the empty-state guidance and separated calculation status from the action label. Changed inputs now explicitly request recalculation.
- Added pressed-state semantics to station selection, scale controls and result tabs; enlarged button targets to at least 44 px and improved narrow-screen wrapping, input readability and muted-label contrast.
- Escaped station names and pasted observations when rebuilding input fields so user text cannot break the form markup.
- Corrected CAD instructions: `12-N-E` requires Z and fills `CPTNAME` / `CPTELEV`; block `98` (default WMH) uses scale `0.5`, while other blocks use `500`.
- Verification: all 11 Node test files passed, including the new interaction regressions for the remaining calculation action, summary navigation, edited-input status and selection semantics. This was a source and behavior review: browser policy blocked inspection of the local `file:` page, so desktop/mobile screenshots and visual layout remain unverified. The local `.git` directory has no repository metadata, so no commits were created.

### Total Station (relative coordinates)
Paste directly — comma or tab separated:
```
PointName,X,Y,Z,Attribute
P1,0.000,0.000,10.250,Control
S1,30.000,160.000,11.650,Shared
T1,55.000,120.000,10.900,Power manhole
```
`Attribute` is optional and accepts any text (control point, manhole type, road edge, etc.).

### Project-wide GPS Absolute Coordinates
```
PointName,E,N,H,Attribute
P1,246440.000,2665050.000,52.250,Control point
P2,246501.745,2665128.661,52.820
```
`Attribute` is optional. Point names must exactly match the total station data when used for pairing. Case-sensitive.

---

## Workflow

1. Open index.html (survey) or TraverseLink-CAD.html (survey and CAD).
2. Add or select stations and paste each station's relative observations.
3. Paste the GPS controls once into the project-wide shared GPS field. Matching is automatic by case-sensitive point name.
4. Use the single calculation button below the inputs. Both adjustment methods are computed: the primary preserves the internally adjusted geometry and fits each group to GPS with K=1; the reference simultaneously coordinates GPS and station observations.
5. Review internal residuals and GPS-fit residuals separately, then inspect the reference coordinates and Reference minus Primary differences.
6. Inspect the primary points on the GIS map. Switch between Taiwan electronic map (EMAP) and NLSC orthophoto (PHOTO2) with the upper-right layer control. Point overlay switches remain independent of the basemap.
7. Export primary coordinates with TSV/merged CSV, both solutions with the comparison CSV or HTML report, and save project JSON to retain inputs and dual results.

---

## How Reference-Solution Initialization Works

```
Round 1: Solve all GPS stations → add their points to the global library
Round 2: For each unsolved relay station, check if ≥2 shared points
         are now in the library → solve if possible → add to library
Round N: Repeat until all stations are solved or no further progress
```

Relay stations can reference shared points from stations that appear **later** in the list, not just earlier ones.

---

## How the Two Solutions Work

The primary method first groups stations by shared point names. Each group is initialized in a temporary local frame without GPS. Its shared-point coordinates and station rigid transforms are fitted iteratively, with one station fixed only to establish the local datum. A single independent station uses its own local frame.

Once internal adjustment has converged, the common points are fitted to the matching GPS coordinates using one K=1 horizontal rigid transform and one height offset per group. Every internal consensus point and station observation receives the same group transform. GPS therefore cannot stretch or reshape the internally adjusted network.

At least two geometrically usable GPS control points are required per group. The current initialization also requires stations to be connected through enough shared controls to determine orientation; an isolated one-point tie cannot determine relative rotation. Groups that cannot initialize, do not converge, or lack a usable GPS fit remain without primary coordinates. The reference solution is never silently substituted.

The simultaneous reference is calculated independently with the original GPS/EDM equal-coordinate-weight algorithm and the selected fixed/free scale setting. Its coordinates, residuals and convergence status are presented separately. For either method, repeated identical GPS tuples do not add weight; different tuples for one name are averaged into one control coordinate.

---

## Coordinate Transformation

**Horizontal — default small-area rigid transformation (`K=1`):**

```
E = cosθ·X − sinθ·Y + tx
N = sinθ·X + cosθ·Y + ty

K        = 1.000000
Rotation = θ
```

This mode is intended for general small-area terrain surveys, typically within about 500 m, where the total station and absolute coordinate systems are both in meters and the local measurement scale should be preserved. Fixing scale helps expose control-point, point-name, height, or observation issues instead of absorbing them into a free scale factor.

**Horizontal — optional scaled transformation (`開啟縮放`):**

```
E = a·X − b·Y + tx
N = b·X + a·Y + ty

K        = √(a² + b²)
Rotation = atan2(b, a)
```

Use this mode for tunnel work or long-distance traverse transfer, such as 1–2 km environments, when allowing a scale factor better fits the measurement geometry. For normal terrain survey work, keep the default `K=1` mode.

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
├── index.html              # Survey-only application
├── TraverseLink-CAD.html   # Survey and CAD workspace with shared project backup
├── tests/                  # Node.js regression tests
└── README.md               # This file
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

Coordinate calculations run locally without server-side computation. Loading the online basemaps and externally hosted map libraries requires internet access.

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
- Survey calculations and project-file handling run in the browser. Online basemaps request tiles from NLSC; map libraries are loaded from their configured CDN.

---

## License

MIT License — free to use, modify, and distribute.


### Historical reference algorithm: joint adjustment (2026-10-20)

GPS availability no longer excludes cross-station observations. GPS coordinates act as soft, equal-weight coordinate constraints; they may move in the joint solution. Each station keeps a rigid horizontal transform (K=1) by default, or a similarity transform in free-scale mode, and a height offset. Alternating least-squares fits update station transforms and shared point coordinates until coordinate movement is below 1e-8 m (maximum 2000 iterations). Nonconvergence is reported explicitly.

Per-station outputs retain transformed observations and show correction vectors plus EDM-to-joint and joint-to-GPS residuals. The merged output uses the common joint coordinates instead of the last station's coordinates. Pairwise shared-point summaries include adjacent GPS stations; their 3D value is the RMS of post-adjustment station differences, not a classical traverse closure. GPS-only points without any EDM observations retain the existing import/merge behavior.

Stations still require two distinct horizontal control/shared points for initial orientation. A single shared point contributes a constraint once a station is independently oriented. Disconnected or insufficiently controlled stations remain unsolved. This is a coordinate transformation network adjustment, not a raw angle/distance observation adjustment. Coordinate precision settings and fixed GPS constraints are not yet exposed; current weights are equal.

Validation: `node tests/joint-adjustment.test.js` covers adjacent GPS stations, mixed GPS/relay stations, exact consistency, station-order independence and fixed/free scale. Run all `tests/*.test.js` for CAD and project-save regression coverage.

GPS constraints are grouped by point name across stations. Identical coordinate tuples are deduplicated; distinct tuples for one name are averaged into one equal-weight soft constraint, with a visible inconsistency warning. GPS residuals retain all source station names, the distinct-coordinate count and maximum distance from the average. Repeating the same GPS data in additional stations never increases its constraint weight. This applies to GPS participating in the joint adjustment; standalone GPS-only export merging remains unchanged.

### GPS 同點號合併與殘差判讀（2026-10-20）

兩個入口 `index.html`、`TraverseLink-CAD.html` 採用相同規則：GPS 站也參與聯測；同點號 GPS 合併成一筆等權重軟約束，保留來源測站。同站重複貼入與跨站重複貼入皆適用，點號區分大小寫。

| 資料情況 | 計算方式 |
| --- | --- |
| 同點號、相同 E/N/H | 只保留一組座標，不增加 GPS 權重 |
| 同點號、不同 E/N/H | 不同座標組各算一次，取平均形成一筆約束，並提示差異 |
| 各站實際量到同點號 | 各站 EDM 觀測分別保留，參與聯合平差 |
| 來源測站 | 保留所有來源測站；同站重複列不重複顯示站名 |

例如三站皆貼入 P01 的同一份 GPS 成果，並各自量到 P01，計算為「3 筆 EDM 觀測＋1 筆 GPS 約束」。聯合座標是求解結果，不是額外一筆觀測。各站殘差表可能重複顯示同一個 GPS 殘差，僅供各站檢視，不代表重複加權。

「本站轉換 − 聯合座標」表示本站轉換結果與共同成果的差；「聯合座標 − GPS」表示共同成果與合併後 GPS 約束的差。ΔE、ΔN、ΔH 分別為東向、北向與高程差；3D 為三者平方和開根號，單位公尺。GPS 有不同座標時，「最大偏離平均值」是原 GPS 座標到其平均值的最大 3D 距離，與平差殘差不同。

目前 GPS 為軟約束，因此 GPS 殘差不一定為零；殘差小表示資料相互吻合，不能直接當成實際定位精度。沒有 EDM 觀測的 GPS-only 點維持原匯出合併行為，未納入上述聯合平差約束。

驗證：`node tests/joint-adjustment.test.js` 涵蓋三站重複 GPS、同站重複列、不同座標平均、來源紀錄、無效高程排除，以及固定／開放比例模式。

完整回歸（PowerShell）：

```powershell
Get-ChildItem tests/*.test.js | ForEach-Object {
    node $_.FullName
    if ($LASTEXITCODE -ne 0) { throw "Test failed: $($_.Name)" }
}
```

### 專案共用 GPS 清單（2026-10-20 更新）

GPS 現在只需在「專案共用 GPS 控制點」貼一次，格式為 `點號,E,N,H,屬性`，支援逗號或 Tab。測站只需貼入各站相對座標，系統依同名點自動配對；不再逐站貼 GPS，也不需手動選擇 GPS／傳遞站。測站下方會顯示配對到的 GPS 點號，修改本站資料後同步更新。

- GPS 與跨站共同點一起參與平差；沒有 GPS 配對的測站仍可利用共同點傳遞。
- 初始定向至少需要兩個不同水平位置的可用控制／共同點；共用清單不會補足缺少的觀測或定向。
- 同點號 GPS 仍只形成一筆軟約束：完全重複座標不加權，不同座標組取平均並提示差異。初始轉換與 GPS-only 匯出也使用不同座標組的平均值。
- GPS 殘差的來源標示為「共用 GPS 清單」。在多站顯示同一 GPS 殘差，並不代表重複加權。
- 未被任何測站量到的 GPS 點仍可作為 GPS-only 點匯出，即使沒有測站配對到 GPS 也會保留。
- 專案 JSON 維持 schemaVersion 2，新增 `survey.gpsPasteText` 儲存共用原文；各站 `gpsPasteText` 存為空字串，避免重複儲存。內部測站仍保留派生的配對資料供計算使用。
- 載入舊版 v1／v2 專案時，若沒有共用欄位，會將原 GPS 測站清單集中帶入；原始資料保留，由計算進行同點號去重。若新格式已提供共用欄位（包含空字串），以共用欄位為準。
- 舊版程式不支援讀取新共用欄位，請以更新後頁面開啟新存檔。清空資料會同時清空共用 GPS；刪除個別測站不會刪除共用清單。

驗證：`node tests/shared-gps.test.js` 涵蓋三站自動配對、舊格式轉入、不同 GPS 座標平均、共用欄位編輯與清空、GPS-only 匯出、存檔還原及無效欄位拒絕。

### 輸入順序與操作調整（2026-10-20）

- 兩個入口皆改為：測站切換列 → 本站相對座標窗格 → 專案共用 GPS 窗格 → 計算並查看結果 → 各站與合併結果。
- 現行版本只保留輸入區底部的「計算並查看結果」，已移除上方重複計算按鈕；計算完成後定位至結果摘要。
- 編輯共用 GPS 時僅更新配對狀態、診斷與預覽，不再重建本站輸入框。
- 測站名稱與相對座標輸入框補上可存取名稱，GPS 輸入框連結格式說明。

本次為程式層面的操作流程檢查，既有計算、CAD、存檔還原與共用 GPS 測試通過。瀏覽器安全政策禁止讀取本機 file:// 頁面，未完成截圖、實際點擊、窄螢幕、縮放與輔助科技測試，不據此宣稱完成視覺或無障礙稽核。

### 雙解成果：主要保留幾何，參考共同協調（2026-10-20）

本節取代前文單一共同協調解作為一般匯出成果的說明。按一次「計算並查看結果」會同時計算兩種方法：

| 項目 | 主要成果：保留內部幾何 | 參考成果：共同協調 |
| --- | --- | --- |
| 第一階段 | 不使用 GPS，先以跨站共同點做內部聯測平差 | GPS 與跨站共同點共同參與求解 |
| GPS 作用 | 內部完成後，各群組整體平移、旋轉與高程位移套合 | GPS 為軟約束，可影響各站相對關係與合併點幾何 |
| 水平比例 | 全程固定 K＝1 | 依參考解比例選項，固定或開放 |
| 獨立站 | 各自建立局部座標，再各自套合對應 GPS | 各自與對應 GPS 求解 |
| 一般表格、地圖、TSV、合併 CSV | 使用主要成果 | 不覆蓋主要成果 |
| 雙解比較 CSV／HTML 報告 | 與參考成果並列 | 顯示座標與參考減主要的差值 |

**主要算法**：同點號形成聯測連通群組；每群組暫定一站的位置、方向與高程以建立局部基準（不是把該站當成已知世界座標）。各站共同點迭代等權重擬合，不使用 GPS；收斂後，再以共同點聯合座標對 GPS 做一次 K＝1 水平剛體擬合與高程平移。第一階段可能調整原始觀測間的不一致；第二階段保留的是「內部平差完成後」的點間幾何，並非保證所有原始量測距離完全不變。

每個 GPS 點號在整體套合中只用一次，不按觀測到它的測站數增加權重。同點號不同座標組先平均；完全重複組不增加權重。GPS 不被強制固定，所以主要成果仍可有 GPS 殘差。

**兩種誤差分開判讀**：各站 RMS/SD 與「本站轉換 − 聯合座標」描述內部吻合；群組 GPS 套合 RMS 與「聯合座標 − GPS」描述對外部基準的吻合。GPS 套合 RMS 超過原提醒門檻時，也會影響測站狀態，避免內部誤差很小但對 GPS 偏差很大時仍顯示良好。這些數字是吻合指標，尚不是完整協方差或原始角距觀測模型推估的定位精度。

**失敗處理**：初始聯測須能由已定位的共同點建立各站定向；只有一個跨站共同點時，不能獨立決定相對旋轉。即使 GPS 足以讓參考解求解，主要解也不會偷偷使用 GPS 建立內部幾何。部分複雜網形可能需要更完整的初始化算法，目前會明確標示無法建立初始定向。內部聯測最多 4000 次迭代、移動小於 1e-8 m 判為收斂；未收斂、GPS 不足或控制幾何退化的群組不產生主要成果。每群組至少需要兩個可用於定向的 GPS 控制點。其他獨立群組仍可正常產生主要成果。

主要解「平差修正」欄表示：內部聯測迭代相對初始傳遞的改正量，旋轉至最後世界座標方向；不把整體移往世界座標的平移量當成誤差。參考解仍保留原來由初始 GPS／共同點轉換至共同協調解的殘差與狀態。

**存檔與匯出**：維持 JSON schemaVersion 2，新增 `survey.adjustmentComparison`，儲存群組狀態、GPS 套合指標及參考解；主要成果仍在 `survey.results`。載入舊檔不會自動把舊共同協調成果標為新主要成果，重新計算後才產生雙解。主要 CSV 檔名含 `primary_geometry_K1`；專用比較 CSV 欄名區分 Primary／Reference，差值為 Reference − Primary，涵蓋有測站觀測且至少一解可算出的點。GPS-only 點照原方式列入一般合併匯出。HTML 報告同時保留雙解，原 CAD 作業仍維持使用者貼入的資料，不自動替換。

**驗證**：`node tests/dual-adjustment.test.js` 涵蓋 GPS 改動後內部幾何保持、三維距離保持、主要／參考差異、固定與開放比例隔離、獨立群組、無 GPS 站的跨站傳遞、測站順序、GPS 重複去重、單點聯測不足、GPS 缺少、存檔還原與匯出。全部既有測試亦需通過。


## 目前操作方式與 GIS 底圖（2026-10-20）

目前兩個 HTML 入口的測量功能一致。操作順序為：選擇／新增測站 → 貼入各站相對座標 → 貼入一次共用 GPS → 按下「計算並查看結果」→ 檢查雙解與地圖 → 匯出／儲存專案。

| 地圖項目 | 目前行為 |
| --- | --- |
| 台灣通用電子地圖 | 國土測繪中心 EMAP，預設底圖 |
| 正射影像 | 國土測繪中心 PHOTO2，可在地圖右上角切換 |
| 底圖切換 | 兩種底圖擇一，不改變成果座標 |
| 點位圖層 | GPS+EDM、GPS-only、EDM 可分別開關，不受底圖切換影響 |
| 切換選擇保留 | 同一頁面期間重建地圖時保留；未寫入專案 JSON 或瀏覽器儲存，重新開啟頁面回到電子地圖 |
| 地圖成果來源 | 使用主要解：先聯測，再以 K＝1 整體套合 GPS |
| 網路需求 | 底圖圖磚與外部地圖套件需連網；座標計算在本機瀏覽器執行 |

底圖圖磚來源：

- 電子地圖：https://wmts.nlsc.gov.tw/wmts/EMAP/default/GoogleMapsCompatible/{z}/{y}/{x}
- 正射影像：https://wmts.nlsc.gov.tw/wmts/PHOTO2/default/GoogleMapsCompatible/{z}/{y}/{x}

請同時檢查內部聯測殘差與 GPS 套合殘差；兩種底圖只協助辨識位置，不能據此認定測量精度。主要／參考成果的算法、失敗處理、存檔與匯出規則，請見上方「雙解成果」章節。

本次文件更新依兩個 HTML 的現行實作核對，未修改運算或底圖程式；底圖服務的即時可用性與影像顯示未在本次重新驗證。
