# Testing Patterns

**Analysis Date:** 2026-02-27

## Test Framework

**Runner:**
- Jest or Vitest recommended (not currently configured)
- Config file location: `jest.config.js` or `vitest.config.ts` (to be created)

**Assertion Library:**
- Jest built-in matchers or Vitest matchers
- Optional: `@testing-library/react` for component testing

**Run Commands:**
```bash
npm test                    # Run all tests in watch mode
npm test -- --coverage      # Run with coverage report
npm test -- --run           # Single run (CI mode)
npm test Calculator.test    # Run specific test file
```

## Test File Organization

**Location:**
- Co-located with source code in `__tests__` subdirectories
- Example: `src/components/__tests__/Calculator.test.jsx`
- Alternative: `src/components/Calculator.test.jsx` (same directory as component)

**Naming:**
- Test files: `[ComponentName].test.jsx` for components
- Test files: `[functionName].test.js` for utilities
- Test suites: Descriptive names matching the component/function being tested

**Structure:**
```
src/
├── components/
│   ├── Calculator.jsx
│   ├── Calculator.test.jsx
│   ├── ChiploadsExplained.jsx
│   ├── ChiploadsExplained.test.jsx
│   ├── ToolLibrary.jsx
│   └── ToolLibrary.test.jsx
├── utils/
│   ├── calculations.js
│   ├── calculations.test.js
│   ├── materials.js
│   └── materials.test.js
├── constants/
│   ├── materials.js
│   └── tools.js
└── __tests__/
    └── integration/
        └── CalculatorFlow.test.jsx
```

## Test Structure

**Suite Organization:**

For calculation utilities:
```javascript
describe('calculateFeedRate', () => {
  describe('valid inputs', () => {
    it('should calculate feed rate correctly with standard values', () => {
      const result = calculateFeedRate(2000, 2, 0.05);
      expect(result).toBe(200); // 2000 * 2 * 0.05
    });

    it('should handle maximum RPM', () => {
      const result = calculateFeedRate(40000, 4, 0.01);
      expect(result).toBe(1600);
    });
  });

  describe('edge cases', () => {
    it('should round fractional results', () => {
      const result = calculateFeedRate(1000, 3, 0.033);
      expect(result).toBe(99); // Rounded
    });

    it('should handle single-flute bits', () => {
      const result = calculateFeedRate(2000, 1, 0.05);
      expect(result).toBe(100);
    });
  });

  describe('invalid inputs', () => {
    it('should throw error for zero RPM', () => {
      expect(() => calculateFeedRate(0, 2, 0.05)).toThrow();
    });
  });
});
```

For React components:
```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Calculator from './Calculator';

describe('Calculator Component', () => {
  describe('rendering', () => {
    it('should render calculator form', () => {
      render(<Calculator />);
      expect(screen.getByRole('heading', { name: /calculator/i })).toBeInTheDocument();
    });

    it('should render material presets', () => {
      render(<Calculator />);
      expect(screen.getByText('Hardwood')).toBeInTheDocument();
      expect(screen.getByText('MDF')).toBeInTheDocument();
    });
  });

  describe('user interactions', () => {
    it('should update feed rate when slider changes', () => {
      render(<Calculator />);
      const slider = screen.getByRole('slider', { name: /feed rate/i });
      fireEvent.change(slider, { target: { value: 300 } });
      expect(screen.getByDisplayValue('300')).toBeInTheDocument();
    });

    it('should show chipload in range feedback', () => {
      render(<Calculator />);
      // Set inputs that result in valid chipload
      fireEvent.change(screen.getByLabelText('RPM'), { target: { value: 2000 } });
      expect(screen.getByText(/in range/i)).toBeInTheDocument();
    });
  });

  describe('validation feedback', () => {
    it('should display "too-high" message when feed rate exceeds Onefinity max', () => {
      render(<Calculator />);
      const slider = screen.getByRole('slider', { name: /feed rate/i });
      fireEvent.change(slider, { target: { value: 13000 } });
      expect(screen.getByText(/exceeds maximum/i)).toBeInTheDocument();
    });
  });
});
```

**Patterns:**
- Setup: Use `beforeEach()` for shared test data and component initialization
- Teardown: Use `afterEach()` for cleanup if needed (rarely required with modern testing libraries)
- Assertion pattern: Arrange-Act-Assert structure

## Mocking

**Framework:** Jest mocks or Vitest mocks

**Patterns:**

Mock constants:
```javascript
jest.mock('../constants/materials', () => ({
  MATERIAL_PRESETS: {
    HARDWOOD: { chiploadMin: 0.05, chiploadMax: 0.15 },
    MDF: { chiploadMin: 0.03, chiploadMax: 0.10 },
  },
}));
```

Mock utility functions:
```javascript
jest.mock('../utils/calculations', () => ({
  calculateFeedRate: jest.fn(() => 250),
  calculateChipload: jest.fn(() => 0.05),
}));

import { calculateFeedRate } from '../utils/calculations';

it('should call calculateFeedRate with correct params', () => {
  // Test code
  expect(calculateFeedRate).toHaveBeenCalledWith(2000, 2, 0.05);
});
```

Mock window/browser APIs (if needed for future features):
```javascript
global.requestAnimationFrame = jest.fn((cb) => cb(0));
```

**What to Mock:**
- External API calls (if calculator becomes connected)
- Complex utility functions when testing components in isolation
- Browser APIs when testing non-visual logic
- LocalStorage/sessionStorage for state management tests

**What NOT to Mock:**
- Calculation utility functions in component tests (test the real integration)
- React components used within other components (test the real component tree)
- Material presets or tool library data (test with real data)

## Fixtures and Factories

**Test Data:**

Create factory functions for common test objects:
```javascript
// __tests__/factories/materials.js
export const createMaterialPreset = (overrides = {}) => ({
  name: 'Hardwood',
  chiploadMin: 0.05,
  chiploadMax: 0.15,
  ...overrides,
});

export const createToolDefinition = (overrides = {}) => ({
  diameter: 6.35, // 1/4"
  group: 'standard',
  use: 'General routing',
  ...overrides,
});

// In test file:
import { createMaterialPreset } from '../factories/materials';

describe('calculation', () => {
  it('should respect material chipload bounds', () => {
    const material = createMaterialPreset({ chiploadMax: 0.10 });
    const result = validateChipload(0.12, material);
    expect(result.isValid).toBe(false);
  });
});
```

**Location:**
- `src/__tests__/factories/` directory for shared factories
- Organize by domain: `materials.js`, `tools.js`, `calculations.js`

## Coverage

**Requirements:** Target 80% coverage for utilities, 60% for components (not enforced yet)

**View Coverage:**
```bash
npm test -- --coverage
```

**Coverage output:**
- Statements: percentage of lines executed
- Branches: percentage of if/else branches executed
- Functions: percentage of functions called
- Lines: percentage of executable lines run

**Focus areas for testing:**
- Calculation utilities (highest priority): 90%+ coverage
- Input validation functions: 90%+ coverage
- Material/tool constants integration: 70%+ coverage
- Component interactions: 60%+ coverage
- Edge cases and error states: 70%+ coverage

## Test Types

**Unit Tests:**
- Scope: Individual calculation functions, validators
- Approach: Test function in isolation with various inputs
- Files: `src/utils/*.test.js`
- Examples: `calculateFeedRate`, `validateChipload`, `getMaterialDefaults`

**Integration Tests:**
- Scope: Multiple functions working together (e.g., full calculation flow)
- Approach: Set up initial state, perform calculations, verify final result
- Files: `src/__tests__/integration/CalculatorFlow.test.jsx`
- Example:
```javascript
it('should calculate correct feed rate from high-level inputs', () => {
  const material = getMaterialPreset('hardwood');
  const tool = getToolDefinition(6.35);
  const chipload = calculateChipload(2000, 100, 2);
  expect(chipload).toBeWithinRange(material.chiploadMin, material.chiploadMax);
});
```

**E2E Tests:**
- Framework: Cypress or Playwright (not currently configured)
- Scope: Full user workflows (open calculator, set values, verify results)
- Files: `cypress/e2e/` or `tests/e2e/`
- Not required for initial implementation but recommended for UI validation

## Common Patterns

**Async Testing:**

For future async operations (API calls, etc.):
```javascript
it('should fetch material data', async () => {
  const result = await fetchMaterialPresets();
  expect(result).toHaveLength(5);
});
```

Using `waitFor` for component state updates:
```jsx
it('should display results after calculation', async () => {
  render(<Calculator />);
  fireEvent.click(screen.getByText('Calculate'));

  await waitFor(() => {
    expect(screen.getByText(/feed rate:/i)).toBeInTheDocument();
  });
});
```

**Error Testing:**

Test validation errors:
```javascript
describe('validateFeedRate', () => {
  it('should return error when feed rate exceeds maximum', () => {
    const result = validateFeedRate(13000, 50, 12700);
    expect(result.isValid).toBe(false);
    expect(result.status).toBe('too-high');
    expect(result.message).toMatch(/maximum/i);
  });
});
```

Test component error boundaries (if implemented):
```jsx
it('should display error message when calculation fails', () => {
  const badProps = { rpm: NaN };
  render(<Calculator {...badProps} />);
  expect(screen.getByText(/invalid input/i)).toBeInTheDocument();
});
```

---

*Testing analysis: 2026-02-27*
