# LitRAG Fork Modifications

This document describes the custom modifications made to foliate-js for integration with the LitRAG ebook reader application.

## Overview

This fork adds RAG (Retrieval-Augmented Generation) chatbot integration features to the foliate-js ebook reader. The modifications enable users to select text while reading, highlight passages, and ask AI-powered questions about the content.

## Modified Files

- **paginator.js** - Text selection events and color scheme fixes
- **reader.js** - Chat sidebar integration and UI cleanup
- **view.js** - Context menu system and highlight management

## Detailed Changes

### 1. Chat Sidebar Integration (reader.js)

Added a dedicated chat sidebar that can be toggled alongside the existing navigation sidebar:

- New `#chat-side-bar-button` trigger for opening chat panel
- Shares the dimming overlay with the navigation sidebar
- `openEbook()` function exported for external integration
- Properly handles both sidebars through the `closeSideBar()` method

### 2. Text Selection & Context Menu (view.js)

Implemented a sophisticated text selection system with contextual actions:

- **Selection Detection**: Captures text selection events from iframe documents
- **Context Menu**: Two-option floating menu ("Highlight" and "Ask question")
- **CFI Tracking**: Stores precise location using EPUB Canonical Fragment Identifiers
- **Global State**: Makes selected text available via `globalThis.selectedText`
- **Event System**: Custom events for selection and menu actions

### 3. Highlight System (view.js & paginator.js)

Two-tier highlighting system using SVG overlays:

- **Temporary Highlights**: Gray (rgba(128, 128, 128, 0.75)) - shown while context menu is open
- **Permanent Highlights**: Yellow (rgba(245, 221, 66, 0.85)) - saved user highlights
- **Overlayer Integration**: Leverages foliate-js's existing SVG annotation system
- **Automatic Cleanup**: Removes temporary highlights when clicking away

### 4. UI Cleanup for Embedded Usage (reader.js)

Streamlines the interface when opening books:

- Removes `#drop-target` file drop zone
- Removes `#book-grid` book selection interface
- Removes `#welcome-msg` introductory text
- Creates a cleaner, focused reading experience

### 5. Color Scheme Fixes (paginator.js)

Addresses visual consistency issues:

- Forces light color scheme preference (was hardcoded to dark)
- Handles Standard Ebooks black-on-transparent images by removing `se:image.color-depth.black-on-transparent` type attributes
- Comments out automatic color-scheme CSS to prevent conflicts

### 6. Selection Event System (paginator.js)

Enhanced iframe communication for text selection:

- Listens for `mouseup` events in content iframes
- Dispatches custom `text-selected` events with iframe document reference
- Automatically clears selections after processing
- Enables cross-frame communication for selection handling

## Integration Points

### Custom Events

The fork dispatches these custom events for integration:

- `text-selected` - Fired when user selects text
  - Detail: `{ iframeDocument: Document }`
- `context-menu-action` - Fired when user clicks a context menu item
  - Detail: `{ action: 'highlight' | 'ask-question' }`

### Global Variables

- `globalThis.selectedText` - Currently selected text string
- `globalThis.reader` - Reader instance reference

### Exported Functions

- `openEbook(file)` - Opens an ebook file and initializes the reader

## Usage Example

```javascript
import { openEbook } from './foliate-js/reader.js';

// Open an ebook
await openEbook(epubFile);

// Listen for text selection
document.addEventListener('text-selected', (e) => {
    const selectedText = globalThis.selectedText;
    // Send to RAG backend for processing
});

// Listen for context menu actions
document.addEventListener('context-menu-action', (e) => {
    if (e.detail.action === 'ask-question') {
        // Open chat interface with selected text
    }
});
```

## Compatibility Notes

- Maintains full compatibility with upstream foliate-js book formats (EPUB, MOBI, FB2, etc.)
- Context menu and selection features work across all supported formats
- Color scheme fixes specifically target Standard Ebooks EPUBs

## Future Improvements

- [ ] Make color scheme preference configurable
- [ ] Add persistent storage for highlights
- [ ] Support multiple highlight colors/categories
- [ ] Add annotation/note capabilities
- [ ] Improve context menu positioning near viewport edges

## Upstream Tracking

Based on foliate-js commit: `c49ded0` (Fix indentation)
Custom modifications commit: `9359e7a` (Custom LitRAG modifications for ebook reader)