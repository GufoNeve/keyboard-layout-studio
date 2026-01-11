## Architecture

### Tech Stack

- **Vue 3** - Progressive JavaScript framework
- **TypeScript** - Type-safe development
- **Pinia** - State management
- **Tailwind CSS** - Utility-first styling
- **Vite** - Fast build tool

### Data Model

All coordinates and dimensions use **"u" units** (1u = standard keycap size = 54px):

```typescript
interface Key {
  id: string
  x: number              // Position in u units
  y: number              // Position in u units
  width: number          // Size in u units (min: 0.25)
  height: number         // Size in u units (min: 0.25)
  legend: KeyLegend      // 9 text positions
  matrix: KeyMatrix      // Row/Col assignment
  keycodes?: Record<number, string>  // Layer → QMK keycode
  shape?: KeyShape       // Key shape type
}
```

### State Management

Centralized Pinia store (`src/stores/keyboard.ts`) manages:
- Full keyboard layout data
- Selection state (single and multi-select)
- Display mode (legend/matrix)
- Clipboard for copy/paste
- Current layer
- Undo/redo history (max 20 states)

## Design Principles

1. **Client-side only** - No server dependencies, suitable for static hosting
2. **Minimal dependencies** - Only essential libraries
3. **Type safety** - Strict TypeScript mode, no `any` types
4. **Simple over clever** - Avoid premature abstractions
5. **Keyboard-first** - Efficient keyboard shortcuts for common operations