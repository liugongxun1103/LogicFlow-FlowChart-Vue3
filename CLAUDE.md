# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Vue 3 diagram/flowchart editor** built with LogicFlow, Element Plus, and Vue Color. It provides a professional UI for creating and editing flowcharts, UML diagrams, and other visual designs with 30+ predefined node types and comprehensive styling options.

## Development Commands

```bash
# Install dependencies
yarn install

# Start development server (hot reload on http://localhost:8080)
yarn serve

# Build for production
yarn build

# Run ESLint
yarn lint
```

## Architecture Overview

### Three-Panel Layout
The application uses a three-panel layout:
- **Left Panel (DiagramSidebar)**: Node palette organized by categories (General, Images, Icons)
- **Center**: LogicFlow canvas for editing diagrams
- **Right Panel (PropertyPanel)**: Style and property editor for selected nodes/edges
- **Top**: Toolbar with selection, zoom, undo/redo, and line type controls

### Node System
All node types are registered in `src/components/node/index.js` and organized into 5 categories:

1. **Basic Shapes** (`node/basic/`): Rectangle, Circle, Diamond, Ellipse, Text, RoundedRect
2. **Custom Path Shapes** (`node/path/`): Triangle, Cylinder, Parallelogram, Actor, Pentagon, Hexagon, etc. (14 total)
3. **Arrows** (`node/arrow/`): Left, Right, Horizontal, Vertical, DoubleArrow, etc. (6 total)
4. **Image Nodes** (`node/image/`): Setting, User, Cloud
5. **Icon Nodes** (`node/icon/`): Message

### Edge Types
Three edge types defined in `src/components/node/edge/`:
- **Polyline**: Broken line with right angles
- **Line**: Straight line
- **Bezier**: Curved line

### Styling System
- All nodes and edges support custom properties stored in `properties` object
- Styles handled by `getShapeStyleUtil.js` utility functions
- Supported style attributes:
  - Background color, gradient color
  - Border (color, width, style)
  - Text (color, size, font, alignment, bold, underline, italic)
- 8 quick-style presets available in PropertyPanel

### Data Persistence
- Diagram data stored in `window.sessionStorage`
- Can load specific diagrams via URL query parameters
- Supports JSON export/import

## Key Files and Directories

| Path | Purpose |
|------|---------|
| `src/main.js` | Application entry point |
| `src/App.vue` | Root component |
| `src/components/Diagram.vue` | Main editor component (orchestrates all sub-components) |
| `src/components/DiagramToolbar.vue` | Top toolbar with tools |
| `src/components/DiagramSidebar.vue` | Left node palette |
| `src/components/PropertyPanel.vue` | Right property editor |
| `src/components/node/` | All node type definitions |
| `src/components/node/index.js` | Node registration entry point |
| `src/components/node/getShapeStyleUtil.js` | Style utility functions |
| `src/components/icon/` | 44 SVG icon components |
| `src/constant/index.js` | Global style constants (colors, fonts, borders) |
| `src/assets/data/` | Default diagram data (mvp.json, design.json, overlay.json) |

## Important Development Notes

### Adding New Node Types
1. Create node class in appropriate subdirectory under `src/components/node/`
2. Export from that subdirectory's index.js
3. Register in `src/components/node/index.js` with unique type name
4. Add corresponding icon in `src/components/icon/` if needed
5. Add to sidebar categories in `DiagramSidebar.vue`

### Styling Nodes
- Use `getShapeStyleUtil.js` functions to apply consistent styles
- All style properties stored in node's `properties` object
- PropertyPanel reads/writes to `properties` for UI updates

### Event Handling
- LogicFlow events: `selection:selected`, `node:click`, `blank:click`, `edge:click`
- Component communication via Vue 3 Composition API `emit`
- Undo/redo managed by LogicFlow's built-in history

### Vue 3 Specifics
- Uses Composition API (not Options API)
- Element Plus for UI components
- No TypeScript (plain JavaScript)

## Dependencies

- **@logicflow/core** (1.2.3): Core diagram editing engine
- **@logicflow/extension** (1.2.3): LogicFlow extensions
- **vue** (3.3.0): Frontend framework
- **element-plus** (2.4.0): UI component library
- **vue-color** (2.8.1): Color picker component
- **core-js** (3.8.3): JavaScript polyfills

## Build Configuration

- **Vue CLI 5.0.0**: Build tool and dev server
- **Babel 7**: JavaScript transpilation
- **ESLint 8.0.0**: Code linting (Vue 3 essential + ESLint recommended)
- **Public path**: `/demo/dist/mvp/` (production)
- **Target browsers**: > 1%, last 2 versions, not dead
