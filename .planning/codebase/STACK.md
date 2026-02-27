# Technology Stack

**Analysis Date:** 2026-02-27

## Languages

**Primary:**
- HTML5 - Application markup and structure
- JavaScript/JSX - Interactive calculator logic and UI

## Runtime

**Environment:**
- Browser (any modern browser supporting ES6+)

**Package Manager:**
- CDN-based distribution (no npm/yarn installation required)

## Frameworks

**Core:**
- React (version unspecified, loaded via CDN) - UI framework for interactive components

## Key Dependencies

**UI & Interactivity:**
- React (via CDN) - Interactive component framework
- React DOM (via CDN) - Browser rendering target

## Configuration

**Environment:**
- None detected - Static configuration embedded in HTML

**Build:**
- No build step required - Direct browser execution from HTML

## Platform Requirements

**Development:**
- Text editor for HTML/JS modification
- Web browser for testing
- Optional: HTTP server for local development (due to CORS restrictions when loading from file://)

**Production:**
- Static web hosting (any HTTP server or CDN)
- Modern browser (ES6 support required)

## Notable Characteristics

**Distribution:**
- Single `index.html` file with embedded or CDN-loaded React
- No npm dependencies or package.json
- Works offline after initial cache of CDN libraries
- Zero installation or build process for end users

---

*Stack analysis: 2026-02-27*
