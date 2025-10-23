# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Foliate-JS is a lightweight, modular JavaScript library for rendering e-books in the browser. It's a pure JavaScript implementation with no external dependencies that supports EPUB, MOBI, KF8, FB2, CBZ, and PDF formats.

## Development Commands

### Building
```bash
npm run build
```
This bundles the vendor dependencies (fflate and zip.js) using Rollup and copies PDF.js files.

### Linting
```bash
npx eslint .
```
The project uses ESLint with specific rules: no semicolons, single quotes, 4-space indentation. The vendor directory is ignored.

### Testing
Tests run in the browser:
1. Open `tests/tests.html` in a browser
2. Check the console for assertion results

Currently only EPUB CFI parser tests exist. Tests use simple `console.assert()` patterns.

## Architecture & Code Structure

### Core Modules
- **view.js**: Main entry point and orchestrator (22.6 KB) - coordinates all other modules
- **paginator.js**: Handles reflowable pagination using CSS multi-column (44 KB)
- **fixed-layout.js**: Fixed layout rendering (12 KB)

### Format Parsers (implement "book" interface)
- **epub.js**: EPUB support with CFI, spine, metadata (43.6 KB)
- **mobi.js**: MOBI/KF8/AZW3 support (47.6 KB)
- **fb2.js**: FictionBook 2 support (12.2 KB)
- **comic-book.js**: CBZ archives (1.8 KB)
- **pdf.js**: PDF.js adapter (6.8 KB)

### Key Interfaces

**Book Interface** (all format parsers must implement):
```javascript
{
    metadata: { title, author, identifier, language, etc. },
    sections: [{
        id, linear,
        load(), unload(), createDocument(),
        mediaType, properties
    }],
    toc: [{ label, href, subitems }],
    pageList: [{ label, href }],
    resolveHref(href),
    resolveCFI(cfi),
    getCFI(index, range),
    splitTOCHref(href),
    getTOCItemOf(section)
}
```

**Renderer Interface** (paginator.js and fixed-layout.js):
```javascript
{
    open(book),
    goTo(target),
    next(), prev(),
    // Events: 'load', 'relocate', 'create-overlayer'
}
```

## Code Style Guidelines

The project enforces these rules via ESLint:
- No semicolons at statement ends
- Single quotes for strings (except to avoid escapes)
- 4-space indentation (flat ternaries, 1-level switch case indent)
- Trailing commas in multiline structures
- No trailing spaces
- Console methods allowed: debug, warn, error, assert

## Module Dependencies

The library is designed to be modular - most files can be used independently:
- Modules depend on interfaces, not each other
- No hard external dependencies (vendor libraries are bundled)
- view.js orchestrates everything but individual parsers/renderers can be used standalone

## Testing Approach

When adding features or fixing bugs:
1. Add test cases to the appropriate test file (currently only epubcfi-tests.js exists)
2. Use `console.assert()` for test assertions
3. Test in multiple browsers (WebKitGTK, Firefox, Chromium recommended)

## Browser Compatibility

Requires modern browser features:
- Blob API
- TextEncoder/TextDecoder
- DOMParser
- Web Crypto API (for EPUB fonts)
- CSS multi-column layout
- Does NOT support older browsers or Firefox ESR

## Security Considerations

The library requires careful CSP configuration when rendering content:
- EPUB/MOBI content gets rendered in iframes
- Dynamic content execution may occur
- See README.md for detailed CSP requirements

## Common Development Tasks

### Adding a New Format Parser
1. Implement the book interface (see epub.js or mobi.js as examples)
2. Handle async loading via `load()` and cleanup via `unload()`
3. Implement `createDocument()` to return a DOM Document
4. Add metadata parsing and TOC extraction

### Working with EPUB CFI
- Use epubcfi.js for parsing and resolution
- CFI strings map locations in EPUB documents
- See epubcfi-tests.js for complex examples with CDATA, entities, etc.

### Adding Text Features
- overlayer.js provides SVG-based annotation system
- search.js implements full-text search across sections
- tts.js generates SSML for text-to-speech