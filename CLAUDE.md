# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Keyboard Layout Studio is a web-based keyboard layout editor designed as an alternative to keyboard-layout-editor.com, specifically focused on features for keyboard developers. The app is client-side only, built with Vue 3, TypeScript, and SVG-based rendering.

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (http://localhost:5173)
npm run dev

# Build for production (runs TypeScript check first)
npm run build

# Preview production build
npm run preview
```

## Architecture

### State Management (Pinia)

The entire application state is centralized in `src/stores/keyboard.ts`:

**State**:
- `layout` (KeyboardLayout) - Full keyboard layout data
- `selectedKeyIds` (string[]) - Array of selected key IDs (supports multi-select)
- `displayMode` ('legend' | 'matrix') - Toggle between legend and matrix display
- `keyboardLayout` ('ANSI' | 'JIS') - Keyboard layout type for keycode display
- `copiedKeys` (Key[]) - Keys in clipboard for copy/paste
- `currentLayer` (number) - Active layer for keycode editing (0-15)
- `history` (KeyboardLayout[]) - Undo/redo history (max 20 states)
- `historyIndex` (number) - Current position in history

**Key Actions**:
- `selectKey(keyId)` - Select single key (or deselect if null)
- `toggleKeySelection(keyId)` - Add/remove key from multi-selection
- `selectMultipleKeys(keyIds)` - Replace selection with multiple keys
- `clearSelection()` - Deselect all keys
- `selectNextKey()` - Select next key in array (for Tab navigation)
- `updateKey(keyId, updates)` - Partial update of key properties
- `moveSelectedKeys(deltaX, deltaY)` - Move all selected keys (prevents negative coordinates)
- `resizeSelectedKeys(deltaWidth, deltaHeight)` - Resize all selected keys (min 0.25u)
- `addKey()` - Add new key (positioned right of selected key, or at 0,0)
- `deleteSelectedKeys()` - Remove all selected keys
- `loadLayout(newLayout)` - Replace entire layout (clears history, migrates old formats)
- `copySelectedKeys()` - Copy selected keys to clipboard
- `pasteKeys()` - Paste clipboard keys (maintains relative positions)
- `toggleDisplayMode()` - Switch between 'legend' and 'matrix' display
- `setKeyboardLayout(layoutType)` - Set keyboard layout type ('ANSI' | 'JIS')
- `updateMetadata(updates)` - Update layout name, author, description
- `setCurrentLayer(layer)` - Change active layer (0-15)
- `incrementLayerCount()` - Add layer (1-16 range), auto-assigns KC_TRNS to all keys on new layer
- `decrementLayerCount()` - Remove layer, clears keycodes from removed layer
- `undo()` / `redo()` - Navigate history
- `createNewLayout()` - Reset to empty layout (clears history)

All mutations update `metadata.modified` timestamp and save to history automatically.

### Data Model (`src/types/keyboard.ts`)

**Key coordinates and dimensions use "u" units** (1u = standard keycap size = 54px):

```typescript
interface Key {
  id: string
  x: number              // Position in u units (min: 0)
  y: number              // Position in u units (min: 0)
  width: number          // Size in u units (min: 0.25)
  height: number         // Size in u units (min: 0.25)
  legend: KeyLegend      // 9 text positions on keycap
  matrix: KeyMatrix      // Row/Col pin assignment
  keycodes?: Record<number, string>  // Layer number → QMK keycode
  rotation?: number      // Rotation angle (degrees) - NOT IMPLEMENTED
  rotationX?: number     // Rotation origin X - NOT IMPLEMENTED
  rotationY?: number     // Rotation origin Y - NOT IMPLEMENTED
  shape?: KeyShape       // 'rectangle' | 'iso-enter' | 'big-ass-enter' | 'circle'
}

interface KeyboardLayout {
  name: string
  keys: Key[]
  layerCount: number     // Total layers (1-16)
  metadata?: {
    author?: string
    description?: string
    created?: string
    modified?: string
  }
}
```

**KeyLegend** supports 9 positions in a 3x3 grid: topLeft, topCenter, topRight, centerLeft, center, centerRight, bottomLeft, bottomCenter, bottomRight.

**KeyMatrix** contains optional `row` and `col` numbers for physical matrix pin assignments.

**KeyShape** determines visual rendering. Special shapes (iso-enter, big-ass-enter, circle) have fixed dimensions and cannot be resized.

### Component Structure

**App.vue**:
- Main application layout with header and tab system
- Handles LocalStorage save/load operations (prefix: `kls-layout-`)
- Layout/JSON tabs with dialog management (new layout, open, delete confirmation)
- Save notification toast

**KeyboardCanvas** (`src/components/KeyboardCanvas.vue`):
- Renders SVG canvas with 0.25u grid lines
- Handles keyboard shortcuts (arrow keys, Ctrl+C/V/Z/Y, Tab, Delete)
- Multi-select support (Ctrl+click to toggle, click empty space to clear)
- Canvas click deselects, key click selects
- SVG size auto-expands based on key positions + 2u padding

**KeyComponent** (`src/components/KeyComponent.vue`):
- Renders individual key as SVG group
- Supports multiple shapes: rectangle (default), iso-enter, big-ass-enter, circle
- Displays legend text in 3x3 grid positions
- Shows matrix info (R{row}C{col}) or keycode based on display mode
- Visual state changes when selected (blue stroke)

**PropertyPanel** (`src/components/PropertyPanel.vue`):
- When no selection: Shows layout metadata editor (name, author, description, stats)
- When single key selected: Shows key property editors
  - Shape selector (with fixed-size warning for special shapes)
  - Position (x, y) and size (width, height) inputs with Enter→next field navigation
  - Legend inputs (9 positions in 3x3 grid)
  - Matrix assignment (ROW, COL)
  - Keycode input with picker dialog button (per current layer)
  - Layer switcher (tabs) with ±buttons
- When multiple keys selected: Shows info message only

**JsonEditor** (`src/components/JsonEditor.vue`):
- Direct JSON editing of layout data
- Import/Export buttons:
  - Import from KLE JSON (keyboard-layout-editor.com format)
  - Export to QMK keyboard.json format (with keymap layers)
  - Export to Vial vial.json format
  - Download KLS native format (.kls.json)
- Real-time validation with error display

**MetadataEditor** (`src/components/MetadataEditor.vue`):
- Standalone component for editing layout name, author, description

**KeycodePickerDialog** (`src/components/KeycodePickerDialog.vue`):
- Modal dialog for selecting QMK keycodes
- Categories: Basic, Modifiers, Functions, Layers, etc.
- Search/filter functionality

**ConfirmDialog** (`src/components/ConfirmDialog.vue`):
- Reusable confirmation dialog for destructive actions

**GistListDialog** (`src/components/GistListDialog.vue`):
- Displays saved layouts from LocalStorage
- Delete and select actions

**AboutDialog** (`src/components/AboutDialog.vue`):
- Application information dialog
- Displays version number and GitHub link

### Utilities

**`src/utils/export.ts`**:
- `exportToQMK(layout)` - Convert to QMK keyboard.json with keymap layers
- `exportToVial(layout)` - Convert to Vial vial.json format
- `downloadJSON(content, filename)` - Trigger browser download

**`src/utils/kle-import.ts`**:
- `parseKLEJson(jsonString)` - Parse KLE JSON string
- `convertKLEToKLS(kleData)` - Convert KLE array format to KLS layout
- `parseKLELabel(label, alignment)` - Map KLE label positions (12 positions) to KLS (9 positions)
- Supports KLE metadata extraction (name, author, notes)
- **Note**: KLE alignment values other than 4 are not fully supported

**`src/data/keycodes.ts`**:
- QMK keycode definitions organized by category

**`src/data/keycode-labels.ts`**:
- Keycode display labels for ANSI/JIS layouts
- `getKeycodeLabel(keycode, layoutType)` - Convert QMK keycode to display label
- Maps keycodes to appropriate symbols based on keyboard layout type

**`src/data/shape-presets.ts`**:
- Predefined shape configurations for special keys

**`src/templates/`**:
- Template JSON files for QMK, Vial, and default layouts
- Used for export functionality and initial layout generation

### Rendering Constants

- **1u = 54px** (keyUnit constant in KeyboardCanvas and KeyComponent)
- **Grid = 0.25u** (13.5px) - minimum movement/resize increment
- **Padding = 4px** (internal key padding for top surface)

### Keyboard Shortcuts

When canvas has focus:
- **Arrow keys**: Move selected keys by 0.25u
- **Shift + Arrow keys**: Resize selected keys by 0.25u
- **Ctrl + C**: Copy selected keys
- **Ctrl + V**: Paste copied keys
- **Ctrl + Z**: Undo
- **Ctrl + Y / Ctrl + Shift + Z**: Redo
- **Tab**: Select next key
- **Delete / Backspace**: Delete selected keys
- **Ctrl + Click**: Toggle key selection (multi-select)

## Current Features

✅ SVG-based keyboard layout editor
✅ Key selection, movement (grid-based), resizing
✅ Multi-select support (Ctrl+click)
✅ Copy/paste (Ctrl+C/V) with relative positioning
✅ Undo/redo (Ctrl+Z/Y) with 20-state history
✅ Key legends (9 positions in 3x3 grid)
✅ Key matrix assignment (ROW/COL)
✅ Multi-layer support (1-16 layers) with per-layer keycodes
✅ QMK keycode picker dialog
✅ Display mode toggle (legend/matrix)
✅ Keyboard layout type selection (ANSI/JIS) with appropriate keycode labels
✅ Key shapes: rectangle, iso-enter, big-ass-enter, circle
✅ LocalStorage persistence with save/load/delete
✅ JSON editor with live validation
✅ Import from KLE (keyboard-layout-editor.com) format
✅ Export to QMK keyboard.json and Vial vial.json formats
✅ Layout metadata (name, author, description, timestamps)
✅ Dark theme UI with Tailwind CSS

## Design Principles

1. **Client-side only**: No server dependencies, suitable for static hosting
2. **Minimal dependencies**: Only Vue 3, Pinia, Tailwind CSS, TypeScript
3. **Type safety**: Strict TypeScript mode, no `any` types
4. **Simple over clever**: Avoid premature abstractions
5. **Keyboard-first**: Efficient keyboard shortcuts for all common operations

## Future Considerations

See design.md for rationale and technical decisions.

Potential enhancements (not prioritized):
- Rotation support (data model exists, but rendering not implemented)
- Drag-and-drop key positioning (currently keyboard/input only)
- Zoom and pan for large layouts
- Ruler and measurement tools
- Snap-to-grid toggle
- Light/dark theme toggle
- GitHub OAuth + Gist integration
- Export to other firmware formats (ZMK, etc.)

## Important Notes

- All position/size values in the data model are in **u units**, converted to pixels only during rendering
- Negative coordinates are prevented (clamped to 0) in moveSelectedKeys and updateKey
- Key IDs are generated using `Date.now() + index` - sufficient for single-user editing
- The store uses deep cloning for history snapshots to prevent reference sharing
- Special shapes (iso-enter, big-ass-enter, circle) have fixed dimensions and cannot be resized via UI
- SVG rendering uses Tailwind classes where possible, with inline styles for dynamic properties (transform, dimensions)
- History is cleared when loading a new layout to prevent memory leaks
- LocalStorage keys use prefix `kls-layout-` + layout name
