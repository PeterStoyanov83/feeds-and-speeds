# Codebase Concerns

**Analysis Date:** 2026-02-27

## Tech Debt

**Single monolithic HTML file:**
- Issue: Entire application (545 lines) is contained in a single `index.html` file with embedded JSX/React code, styling, and component definitions
- Files: Git history shows index.html is deployed but not committed to main branch
- Impact: No separation of concerns, difficult to test, debug, or maintain; all changes require modifying one massive file; no build tooling or TypeScript
- Fix approach: Migrate to a proper build setup (Create React App, Vite, or Next.js) with separate component files, stylesheets, and utilities; implement TypeScript for type safety

**No modular component structure:**
- Issue: Components (`ChiploadDiagram`, `VBitDiagram`, `PassDepthDiagram`) are defined inline in the same file as the main `App` component
- Files: Embedded in index.html
- Impact: Components cannot be tested in isolation, reused across projects, or version controlled independently; makes refactoring risky
- Fix approach: Extract each component to its own file (e.g., `src/components/ChiploadDiagram.tsx`); use barrel exports in `src/components/index.ts`

**Hardcoded data structures:**
- Issue: Tool library (16 bits), materials (5 presets), and chipload ranges are hardcoded as JavaScript arrays in the component
- Files: Lines ~70-95 of index.html
- Impact: Adding tools or materials requires editing the component; no ability to load from external data source or API; no data versioning
- Fix approach: Extract to `src/data/tools.ts`, `src/data/materials.ts`; consider loading from a JSON file or API endpoint for future updates

**No error handling or validation:**
- Issue: SVG rendering, calculations, and state management have no error boundaries or validation
- Files: Component functions assume valid inputs
- Impact: Malformed data or NaN calculations could cause silent failures; users see broken diagrams
- Fix approach: Add input validation in state setters; implement error boundary component; add guards for calculations

**CDN-dependent for core libraries:**
- Issue: React, ReactDOM, and Babel are loaded from CDN; no package manager or lockfile
- Files: Lines 6-8 of index.html
- Impact: First load requires internet; no control over library versions; CDN outage breaks application; cache misses on updates
- Fix approach: Use npm/yarn with a build tool; pin exact versions in package-lock.json or yarn.lock

## Known Bugs

**Chipload diagram may overflow:**
- Symptoms: The `ChiploadDiagram` SVG hardcodes viewBox width/height (320x200) but scales circle radius based on tool diameter; very large bits (12.7mm) with large chipload values could render outside bounds
- Files: Lines ~155-175 (ChiploadDiagram component)
- Trigger: Select 12.7mm tool at high RPM and feed rate
- Workaround: None; the diagram may be clipped or scaled incorrectly

**V-bit depth calculation doesn't account for bit plunge geometry:**
- Symptoms: Formula `depth = (width/2) / tan(angle/2)` assumes perfect cone geometry; real V-bits have rounded tips
- Files: Line ~102 (VBitDiagram component)
- Impact: Calculated depths are consistently slightly deeper than actual achievable depth
- Workaround: Users should reduce max depth limit in Fusion by 10-15% for micro V-bits

**Missing validation for extreme input combinations:**
- Symptoms: Setting RPM=1000, Feed=12700, Flutes=4, Bit=0.4mm results in chipload=7.925mm (300x the maximum for that bit)
- Files: App state calculations (lines ~310-330)
- Trigger: Rapidly adjust multiple sliders to extreme values
- Workaround: Users with CNC experience know to ignore wildly out-of-range values, but UI doesn't warn strongly enough

## Security Considerations

**No input sanitization in SVG rendering:**
- Risk: Component renders user-controlled values (tool diameter, channel width) into SVG text elements; no escaping
- Files: `VBitDiagram`, `ChiploadDiagram`, `PassDepthDiagram` text elements (lines 102-180)
- Current mitigation: React's JSX escapes by default; values are numeric so injection unlikely
- Recommendations: Explicitly validate that tool diameter and channel width are numbers before rendering; use defensive number formatting

**No authentication or authorization:**
- Risk: Not applicable for a single-user calculator tool
- Current mitigation: No user accounts or data persistence
- Recommendations: If future version stores user presets or sessions, implement proper auth and HTTPS-only transmission

## Performance Bottlenecks

**Chipload diagram animation is CPU-intensive:**
- Problem: `requestAnimationFrame` rotates the bit at 1.5° per frame with no throttling; running continuously even when tab is not visible
- Files: Lines ~155-160 (ChiploadDiagram useEffect)
- Cause: Animation loop never pauses; no visibility detection; no requestIdleCallback
- Improvement path: Pause animation when tab hidden using `document.visibilitychange`; throttle updates to 30fps max; consider CSS animations instead

**SVG re-renders on every state change:**
- Problem: All three diagrams (`ChiploadDiagram`, `VBitDiagram`, `PassDepthDiagram`) are re-rendered even when unrelated state changes
- Files: Lines ~265-390 (tab-based rendering in App)
- Cause: No memoization with `React.memo()`; no granular state splitting
- Improvement path: Wrap diagram components with `React.memo()`; consider separating calculator state from UI state

**Reference table scrollable section recalculates on every material/tool change:**
- Problem: Large table (16 tools × 5 materials) with 80+ cells re-renders and recalculates pass depth for every tool on any state change
- Files: Lines ~430-445 (table rendering in Pass Depth tab)
- Cause: No memoization; table cells are inline-rendered
- Improvement path: Extract table to separate component; memoize with `useMemo()` keyed by material and active tool

## Fragile Areas

**SVG rendering depends on hardcoded constants:**
- Files: `VBitDiagram` (W=320, H=200), `ChiploadDiagram` (W=320, H=200), `PassDepthDiagram` (W=320, H=160)
- Why fragile: Changing tool diameter range or canvas size requires tweaking pixel-to-mm scaling factors in multiple places; no single source of truth
- Safe modification: Extract scaling constants to a config object at the top of the component file; use one set of constants for all diagrams
- Test coverage: No tests for SVG rendering; visual regression testing needed

**Chipload ranges hardcoded in `buildChiploads()` function:**
- Files: Lines ~80-95 (buildChiploads arrow function)
- Why fragile: 12 separate conditional branches for different diameter ranges; ranges are not easily editable without understanding nested conditions; scale multiplier (1.0x for hardwood, 1.15x for plywood, etc.) is magic
- Safe modification: Refactor to lookup table: `{ dia: 0.4, maxDia: 0.5, min: 0.002, max: 0.004 }` for each range; extract scale multipliers to `MATERIAL_SCALES` object
- Test coverage: No unit tests for chipload range calculations

**Material-tool-chipload relationship is tightly coupled:**
- Files: `buildChiploads()` called during material object creation (lines ~85-96)
- Why fragile: Cannot update tool chipload ranges without regenerating all materials; impossible to support dynamic material addition
- Safe modification: Store tool ranges separately; compute material-specific ranges on demand in a `getChiploadRange(tool, material)` function
- Test coverage: No tests; any future API change breaks silently

**Tab state management is simple but unscalable:**
- Files: `activeTab` state controlled as string (line ~312)
- Why fragile: Adding new tabs requires modifying hardcoded tab list (lines ~366-370) and adding new conditional render blocks (lines ~375-390); easy to forget to update both places
- Safe modification: Use tab metadata array: `const TABS = [{ id: "chipload", label: "...", component: ChiploadTab }]`; map over it
- Test coverage: No tests for tab switching

## Scaling Limits

**No data persistence:**
- Current capacity: Calculations are in-memory only; closing browser loses all settings (feed rate, RPM, flutes selection, material, tool)
- Limit: Users cannot save custom presets or reference common configurations
- Scaling path: Add localStorage to persist user preferences; consider REST API + database for shared presets between workshop users

**Single-page application without offline support:**
- Current capacity: App works on first load due to CDN cache, but requires internet for first visit
- Limit: Cannot be used on workshop floor without internet; no service worker
- Scaling path: Implement service worker with offline mode; bundle React in production build instead of loading from CDN

**No mobile optimization:**
- Current capacity: Responsive grid (320px + 1fr) works on desktop but labels and buttons are tiny on phones
- Limit: Calculator unusable on mobile devices; CNC operators often reference on tablet/phone while at machine
- Scaling path: Add mobile breakpoint; stack left/right panels vertically on small screens; increase touch target sizes to 44px minimum

## Dependencies at Risk

**React 18 from CDN (development build):**
- Risk: Using `react.development.js` instead of production build; ~40KB unminified; slow parsing and slower React internals
- Impact: Slow initial render, especially on older workshop machines
- Migration plan: Switch to `react.production.min.js` or use a build tool (Vite/Webpack) with tree-shaking

**Babel transpilation in browser:**
- Risk: `babel-standalone` (130KB) transpiles JSX at runtime; security exposure to malformed code; very slow on first load
- Impact: First page load takes 2-3 seconds on slow connections; JSX syntax is not cached
- Migration plan: Move to build-time transpilation; pre-compile JSX before deployment

**External Google Fonts over HTTP:**
- Risk: Font loading blocks rendering; no fallback fonts if Google is unreachable
- Impact: First paint delayed; calculator shows default fonts momentarily
- Migration plan: Self-host fonts or use system font stack as fallback

## Missing Critical Features

**No print mode:**
- Problem: Workshop reference guides need to be printed or displayed on static poster; current calculator is interactive-only
- Blocks: Cannot post reference chart on shop wall; operators cannot quickly look up chipload ranges offline
- Fix approach: Add `@media print` stylesheet; export chipload tables as PDF

**No spindle-specific RPM presets:**
- Problem: Onefinity Woodworker and Buildbotics have specific RPM capabilities (discrete steps, not continuous); hardcoded range 1000-40000 may not match actual machine
- Blocks: Users must manually find correct RPM from machine documentation, then dial in calculator
- Fix approach: Add spindle presets: "Onefinity @ 40kRPM", "Onefinity @ 10kRPM", etc.; show available RPM steps

**No feed rate safety limits per tool:**
- Problem: Very small micro bits (0.4-0.6mm) should never exceed certain feed rates even with high RPM
- Blocks: UI allows dangerous combinations (e.g., 0.4mm at 12700 mm/min feed = almost certain bit breakage)
- Fix approach: Add per-tool `maxFeedRate` in tool data; disable or red-highlight feed rate slider when exceeded

**No chip evacuation guidance:**
- Problem: Calculator shows chipload but doesn't mention that micro bits require continuous air blast or flood cooling
- Blocks: User might cut with correct chipload but no coolant, causing bit to burn
- Fix approach: Add chip evacuation tab with material-specific guidance (air blast pressure, flood coolant type)

## Test Coverage Gaps

**No unit tests:**
- Untested areas: All calculation functions (`chipload = feedRate / (rpm * flutes)`, pass depth math, V-bit geometry)
- Files: Calculations embedded in App component (lines ~310-330)
- Risk: Any refactoring breaks calculations silently; no regression protection
- Priority: High - calculations are safety-critical for CNC operations

**No component integration tests:**
- Untested areas: Tab switching, state transitions, diagram rendering
- Files: All components
- Risk: UI changes break diagram rendering without detection
- Priority: Medium - affects user experience but not safety

**No visual regression tests:**
- Untested areas: SVG diagrams render correctly at different tool/material combinations
- Files: All diagram components
- Risk: Scaling or positioning bugs go unnoticed; diagrams become unreadable without code review
- Priority: Medium - diagrams are critical for understanding results

**No edge case testing:**
- Untested areas: Extreme values (RPM=1000, Feed=12700), very small bits (0.4mm), very large bits (12.7mm)
- Files: App component state calculations
- Risk: Undefined behavior at boundaries (NaN, Infinity, negative values)
- Priority: Medium - users may accidentally create invalid combinations

---

*Concerns audit: 2026-02-27*
