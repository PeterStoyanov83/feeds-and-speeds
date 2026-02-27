# Coding Conventions

**Analysis Date:** 2026-02-27

## Naming Patterns

**Files:**
- React components: PascalCase (e.g., `Calculator.jsx`, `ToolLibrary.jsx`, `ChiploadsExplained.jsx`)
- Utility functions: camelCase (e.g., `calculateFeedRate.js`, `getMaterialDefaults.js`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_RPM`, `ONEFINITY_MIN_FEED`)
- Test files: `[FileName].test.jsx` or `[FileName].spec.jsx`
- Style files: PascalCase matching component (e.g., `Calculator.css`, `ToolLibrary.css`)

**Functions:**
- Verb-first naming for actions: `calculateChipload()`, `validateFeedRate()`, `formatMetricValue()`
- Getter functions: `getMaterialPresets()`, `getToolDimensions()`
- Event handlers: `handleInputChange()`, `handleFlutesSelect()`, `handleTabSwitch()`
- React hooks: `useMaterialState()`, `useCalculatorLogic()`

**Variables:**
- camelCase for all variables: `feedRate`, `spindleRpm`, `toolDiameter`, `materialPreset`
- Boolean variables: prefix with `is` or `has`: `isInRange`, `hasError`, `isLoading`
- Array variables: plural form: `tools`, `materials`, `fluteCounts`
- DOM references: `inputRef`, `canvasRef`, `containerRef`

**Types/Interfaces:**
- PascalCase with `I` prefix for interfaces (if using TypeScript): `ICalculatorState`, `IMaterialPreset`, `IToolDefinition`
- Type aliases: PascalCase: `CalculationResult`, `FeedRateRange`, `VisualTab`

## Code Style

**Formatting:**
- Prettier or similar formatter expected (not currently enforced)
- Indentation: 2 spaces (standard for React projects)
- Line length: 100 characters preferred, 120 maximum
- Use single quotes for strings (align with common React convention if formatter enforces)
- Semicolons: included (JavaScript standard)

**Linting:**
- ESLint recommended for future implementation
- Rule set: airbnb or similar React-focused configuration
- Key rules to enforce:
  - No unused variables
  - Const/let enforcement (no var)
  - Arrow functions preferred over function declarations
  - No console.log in production code

## Import Organization

**Order:**
1. React and external library imports
2. Component imports
3. Utility/helper imports
4. Constant imports
5. Style imports
6. Type imports (if using TypeScript)

**Example:**
```jsx
import React, { useState, useCallback } from 'react';
import { Chart } from 'chart-library';

import Calculator from './components/Calculator';
import ChiploadsExplained from './components/ChiploadsExplained';

import { calculateFeedRate, validateChipload } from './utils/calculations';
import { MATERIAL_PRESETS } from './constants/materials';

import styles from './App.css';
```

**Path Aliases:**
- Establish `@components`, `@utils`, `@constants` aliases if using Webpack or similar bundler
- Makes imports cleaner and refactoring easier

## Error Handling

**Patterns:**
- Input validation for all calculator parameters before calculation
- Return objects with `{ value, error }` structure for calculation results
- User-facing errors shown in UI with clear messaging
- Invalid inputs trigger visual feedback (color changes, warning text)
- Range checking: Show "in-range" / "too-high" / "too-low" feedback states

**Example:**
```jsx
function validateFeedRate(feedRate, minFeed, maxFeed) {
  if (feedRate < minFeed) {
    return { isValid: false, status: 'too-low', message: `Minimum: ${minFeed} mm/min` };
  }
  if (feedRate > maxFeed) {
    return { isValid: false, status: 'too-high', message: `Maximum: ${maxFeed} mm/min` };
  }
  return { isValid: true, status: 'in-range', message: '' };
}
```

## Logging

**Framework:** `console` (no external logging library required for client-side calculator)

**Patterns:**
- Development: `console.log()` for debug info during development
- Remove all debug logs before committing (or use a logger that strips them in production)
- Use `console.warn()` for potential issues users should know about
- Use `console.error()` for critical failures only
- Never log sensitive data (calculation parameters are okay; user workshop data is not)

**Best practice:**
```jsx
if (process.env.NODE_ENV === 'development') {
  console.log('Calculated feedRate:', feedRate);
}
```

## Comments

**When to Comment:**
- Complex calculation logic: explain the formula and why it's used that way
- Non-obvious workarounds for Buildbotics controller limitations
- Assumptions about material properties (chipload ranges)
- Browser compatibility notes if any

**JSDoc/Comments:**
- Add JSDoc comments to utility functions describing parameters and return values
- Component comments explain purpose and expected props

**Example:**
```jsx
/**
 * Calculates feed rate from spindle RPM, flute count, and chipload.
 * Formula: Feed Rate = RPM × Flutes × Chipload
 *
 * @param {number} rpm - Spindle RPM (1-40000)
 * @param {number} fluteCount - Number of flutes (1-4)
 * @param {number} chipload - Chipload in mm per tooth (material-dependent)
 * @returns {number} Feed rate in mm/min
 */
function calculateFeedRate(rpm, fluteCount, chipload) {
  return Math.round(rpm * fluteCount * chipload);
}
```

## Function Design

**Size:** Keep functions under 50 lines when possible. Break complex calculations into smaller helper functions.

**Parameters:** Maximum 3-4 parameters. Use object destructuring for related parameters:
```jsx
function calculateChipload({ feedRate, rpm, fluteCount }) {
  return feedRate / (rpm * fluteCount);
}
```

**Return Values:**
- Calculation functions: return single numbers or objects with `{ value, unit }`
- Validation functions: return `{ isValid, status, message }`
- Async operations: return Promises (if applicable in future)

## Module Design

**Exports:**
- One primary export per file (component or main utility)
- Use named exports for helper functions only when necessary
- Prefer default exports for components

**Barrel Files:**
- Create `index.js` files in directories with multiple related exports
- Example `src/components/index.js`:
```jsx
export { default as Calculator } from './Calculator';
export { default as ChiploadsExplained } from './ChiploadsExplained';
export { default as ToolLibrary } from './ToolLibrary';
```

**File Structure Principles:**
- One component per file
- Co-locate styles with components (component name matching)
- Utilities grouped by domain (`calculations.js`, `materials.js`, `tools.js`)
- Constants in dedicated `constants/` directory organized by domain

---

*Convention analysis: 2026-02-27*
