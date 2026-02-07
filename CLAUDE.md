# OSB Cutting Planner

## Project Overview
Single-file web app (`cutting-planner.html`) for planning how to cut OSB plates to cover a rectangular frame. Supports any frame size, any plate dimensions, and any number of plates. The solver automatically determines the optimal grid layout, plate orientation, and cut positions.

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
- **Frame**: User-configurable width and depth, with joists at configurable spacing
- **Plates**: User-configurable length, width, and count available
- Default config: 252.9 cm × 171.3 cm frame, 239.7 × 119.8 cm plates, 4 joists at 50 cm c/c

### Solver Pipeline
1. **Grid generation** (`generateOptimizedGrids`): For each of 4 orientations (horizontal/vertical × plate normal/rotated), generates multiple candidate grids with different cut positions:
   - Uniform plate-width intervals
   - Cuts at joist positions (for structural support)
   - Cuts at plate width (for side-by-side packing)
   - Half-frame and half-plate splits
2. **Piece validation**: Each candidate grid's pieces are verified to fit on a plate in some rotation before being considered.
3. **Plate assignment** (`assignPlatePacking`): For each grid candidate, pieces are assigned to physical plates:
   - Brute-force partitioning for ≤6 pieces (optimal)
   - Greedy bin-packing fallback for larger counts
4. **Ranking**: Results sorted by: success → fewest plates used → most joist-aligned cuts → fewest pieces (simpler cuts)

### Packing with Rotation
All pack functions (`packOne` through `packShelf`) try each piece in both orientations (normal and rotated 90°), enabling more efficient plate utilization. The `pieceOrientations()` helper generates the rotation variants.

### Orientation Modes
- **Horizontal**: Plates run along frame width, rows stack along frame depth
- **Vertical**: Plates run along frame depth, rows stack along frame width
- Each mode also tries plates rotated (swapping plate L/W)
- Total: 4 base orientations × multiple cut positions = many candidates evaluated

## Code Architecture (cutting-planner.html)
- **CSS**: Dark theme, responsive grid layout, validation banner styles, solver info panel
- **HTML**: Controls panel (left) with frame/plate inputs, canvas area (right) with frame diagram, dynamic plate canvases, and pieces table
- **JavaScript**:
  - `getInputs()` — reads all form values (frame, plate, and plate count)
  - `nearJoist()` — checks if a cut position aligns with a joist
  - `pieceOrientations()` — generates rotation variants for a piece
  - `generateGrid()` — creates uniform grid at plate-width intervals
  - `generateOptimizedGrids()` — creates multiple grids with optimized cut positions
  - `generateGridWithCols()` — creates grid from explicit column widths
  - `packOne/Two/Three/Four/Shelf()` — plate packing strategies (all rotation-aware)
  - `packOnePlate()` — dispatcher for pack functions by piece count
  - `assignPlatePacking()` — assigns pieces to physical plates (brute-force + greedy)
  - `bruteForceAssign()` — optimal partition for ≤6 pieces
  - `estimateMinPlates()` — estimates minimum plates needed
  - `computeLayout()` — main solver orchestrator (generates candidates, packs, ranks)
  - `drawFrame()` — renders frame with pieces, joists, cut lines, dimension labels
  - `drawPlate()` — renders individual plate with placed pieces and cut lines
  - `ensurePlateCanvases()` — dynamically creates/removes plate canvas elements
  - `updatePiecesTable()` — populates pieces info table
  - `updateValidation()` — updates success/failure banner and issue list
  - `updateSolverInfo()` — shows solver details (orientation, grid, joist alignment)
  - `update()` — main UI update loop, wired to input events

## Reference Images
- `plate.jpg` — photo of the SterlingOSB TG2 plates
- `framelayout.png` — frame structure reference
- `annotated_hems.jpg` — annotated photo of the hems opening
- `goal.png` — target layout visualization
- `plate1cuts.png` / `plate2cuts.png` — cut plan references
