# Architecture

**Analysis Date:** 2026-02-27

## Pattern Overview

**Overall:** Single-Page Application (SPA) with embedded React component hierarchy

**Key Characteristics:**
- Browser-based calculator application with no backend
- React 18 library loaded via CDN (development build)
- Babel for JSX transpilation in the browser
- All state managed client-side with React hooks
- Self-contained HTML file with embedded styles and logic
- No bundler or build step required

## Layers

**Presentation Layer:**
- Purpose: Render UI components and handle user interactions
- Location: Inline `<script type="text/babel">` within `index.html`
- Contains: React components (App, VBitDiagram, ChiploadDiagram, PassDepthDiagram), styling (CSS-in-JS and inline `<style>` tag)
- Depends on: React, React-DOM (via CDN), Material data and tool definitions
- Used by: Browser DOM via React-DOM rendering to `#root` element

**State Management Layer:**
- Purpose: Maintain calculator state and computed values
- Location: App component with useState hooks in index.html
- Contains: Feed rate, RPM, flutes, tool selection, material selection, V-bit parameters, active tab state
- Depends on: React hooks (useState, useEffect)
- Used by: All child components through props

**Computation Layer:**
- Purpose: Calculate chipload and derived values
- Location: Inline functions within App component (calculation performed inline in JSX)
- Contains: Chipload formula (Feed Rate ÷ (RPM × Flutes)), ideal feed calculations, pass depth ranges, plunge rate
- Depends on: Input state values
- Used by: Result display, visual diagrams, formula displays

**Visualization Layer:**
- Purpose: Render SVG diagrams and visual references
- Location: Three specialized components (ChiploadDiagram, VBitDiagram, PassDepthDiagram) in index.html
- Contains: SVG graphics, animations, gradient definitions, geometric calculations
- Depends on: Diagram parameters passed as props, requestAnimationFrame for chipload animation
- Used by: Tabbed display system

**Data Layer:**
- Purpose: Provide static reference data
- Location: TOOLS array, materials array, GROUP_COLORS, GROUP_LABELS constants in index.html
- Contains: Tool diameters with grouping (micro/small/standard/large), material properties with chipload ranges, color mappings, V-bit angles
- Depends on: None
- Used by: App component for rendering options and calculating ranges

## Data Flow

**User Input → Calculation → Output:**

1. User adjusts slider (feed rate, RPM, flutes) or clicks button (material, tool, V-angle)
2. React state updates via onChange handlers
3. Dependent values recalculate synchronously:
   - `chipload = feedRate / (rpm * flutes)`
   - `minCL` and `maxCL` retrieved from material.chipload[toolIdx]
   - Status flags computed: `inRange`, `tooHigh`, `sc` (status color)
   - Derived values: `idealFeedMin/Max`, `plungeMM`, `passDepthMM`
4. Components re-render with new values
5. Diagrams update with new parameters
6. Results display updated in real-time

**Material/Tool Selection Impact:**

1. Tool selection (toolIdx) → loads new tool diameter (diaMM) and default chipload range (minCL, maxCL)
2. Material selection (matIdx) → switches chipload scale for same tool
3. Both trigger recalculation of status and all displayed ranges

**State Management:**
- All state centralized in App component root
- State changes trigger re-render of entire component tree (no performance optimization layer)
- Child components receive state as props
- State mutations only through event handlers (onClick, onChange)

## Key Abstractions

**Tool Library (TOOLS):**
- Purpose: Define available cutting tools with calibrated chipload ranges
- Examples: `{ dia: 0.4, label: "0.4", group: "micro" }` to `{ dia: 12.7, label: "12.7 (1/2\")", group: "large" }`
- Pattern: Array of tool objects indexed by selection, with grouping metadata used for UI organization and warnings

**Material Presets (materials):**
- Purpose: Encapsulate chipload ranges and scaling for different materials
- Examples: "Hardwood" (1.00× scale), "Plywood" (1.15×), "MDF" (1.30×), "Soft Plastic" (0.90×), "Hard Plastic" (0.65×)
- Pattern: `buildChiploads()` function generates min/max arrays for each tool and material combination, allowing material-specific adjustments to be data-driven

**Diagram Components:**
- Purpose: Isolate visual rendering logic from calculation
- Examples: VBitDiagram, ChiploadDiagram, PassDepthDiagram
- Pattern: Functional React components receiving only required parameters as props, containing all SVG/animation logic self-contained

## Entry Points

**Browser Load:**
- Location: `index.html` (deployed via gh-pages branch)
- Triggers: User opens URL in browser
- Responsibilities:
  1. Load React/React-DOM/Babel via CDN
  2. Apply global styles
  3. Initialize React root element
  4. Render App component hierarchy
  5. Attach event listeners via React synthetic events

**App Component Initialization:**
- Location: Line defining `function App()` in index.html script section
- Triggers: React renders after DOM is ready
- Responsibilities:
  1. Initialize all calculator state with default values
  2. Render layout structure (header, left sidebar, right content area)
  3. Set up event handlers for all interactive elements
  4. Render tab-based content system
  5. Coordinate all child components

## Error Handling

**Strategy:** No explicit error handling — calculations are deterministic and inputs are constrained via form controls

**Patterns:**
- Form controls (sliders, buttons) prevent invalid input at source (min/max/step attributes)
- All calculations use safe arithmetic (division by non-zero values only, values are always numbers)
- No network calls or async operations, so no timeout/failure handling needed
- SVG diagrams use defensive calculations (Math.min/Math.max to prevent rendering off-canvas)

## Cross-Cutting Concerns

**Logging:** None - no logging framework; browser console available for debugging but not integrated

**Validation:** Performed at input level via form control attributes (min, max, step); no explicit validation functions

**Authentication:** Not applicable - no server, no user accounts, no data persistence

**Styling:** Mixed approach:
- Global styles in `<style>` tag (normalize, variables, component classes like `.card`, `.lbl`, `.mbtn`, `.tab`)
- Inline style objects for dynamic styling (color changes based on state, grid layouts)
- CSS class toggling for active states (`.on` class applied via className conditional)

**Material Design:**
- Chipload ranges are hardcoded in `buildChiploads()` function for each tool diameter bracket
- Material scales (1.00×, 1.15×, 1.30×, etc.) multiply base chipload values
- Diagram rendering uses hardcoded geometric relationships (diaMM × 10 for pixel scaling, trigonometry for V-bit geometry)

---

*Architecture analysis: 2026-02-27*
