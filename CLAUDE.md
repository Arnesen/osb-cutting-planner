# OSB Cutting Planner — Hems Flooring

## Project Overview
Single-file web app (`cutting-planner.html`) for planning how to cut two SterlingOSB Zero TG2 tongue-and-groove plates to floor a loft hatch opening (hems).

## Tech Stack
- Single HTML file with embedded CSS and vanilla JavaScript
- Canvas-based diagrams (HiDPI-aware with devicePixelRatio)
- No build tools, no dependencies — serve with any static HTTP server

## Running
```bash
cd "C:\Users\alexa\prosjekt hems"
python -m http.server 8080
# Open http://localhost:8080/cutting-planner.html
```

## Key Concepts

### Frame & Plates
- **Frame**: 252.9 cm wide x 171.3 cm deep, 4 joists at 50 cm c/c, 48 mm (6x2) lumber
- **Plates**: 2x SterlingOSB Zero TG2, each 239.7 x 119.8 cm, 18 mm thick
- T&G (tongue and groove) on long sides only: male (tongue) on top, female (groove) on bottom

### Solving Modes
1. **4-piece mode** (default): Frame split into 2 rows x 2 columns (1A, 2A, 2C, 2D). Brute-force bitmask solver tries all piece-to-plate assignments. Packing uses greedy shelf approach with T&G optimization.
2. **3-piece mode** (fallback): When 4-piece fails and both row joints are aligned (row1Joint == row2Joint), merges 2A+2D into single right strip piece "R" rotated 90 degrees on plate 2. R has no T&G requirements.
3. **Failure diagnosis**: When both solvers fail, shows all 7 unique ways to split 4 pieces across 2 plates with mini diagrams visualizing overflow.

### T&G Edge Tracking
- Row 1 pieces (back): need male edge for interlock with Row 2
- Row 2 pieces (front): need female edge for interlock with Row 1
- Solver prefers placements preserving T&G edges, but doesn't hard-enforce them

## Code Architecture (cutting-planner.html)
- **CSS**: Styles including dark theme, failure diagnosis cards, responsive grid
- **HTML**: Controls panel (left), canvas area (right) with frame, plates, failure section, pieces table
- **JavaScript**:
  - `getInputs()` — reads all form values
  - `getNeedsEdge()` / `checkTG()` — T&G constraint logic
  - `packOne/Two/Three/Four()` — plate packing strategies
  - `solvePlateAssignments()` — brute-force 4-piece solver
  - `tryThreePieceMode()` — 3-piece fallback solver
  - `analyzeFailures()` / `analyzeOnePlate()` — failure diagnosis
  - `computeLayout()` — main compute orchestrator
  - `drawFrame()` / `drawPlate()` — canvas rendering
  - `drawFailureMiniPlate()` / `renderFailureDiagnosis()` — failure visualization
  - `update()` — main UI update loop, wired to input events

## Reference Images
- `plate.jpg` — photo of the SterlingOSB TG2 plates
- `framelayout.png` — frame structure reference
- `annotated_hems.jpg` — annotated photo of the hems opening
- `goal.png` — target layout visualization
- `plate1cuts.png` / `plate2cuts.png` — cut plan references
