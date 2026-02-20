# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

egui-arbor is a tree/outliner widget library for egui, inspired by Blender's outliner. It provides hierarchical tree views with expand/collapse, drag-and-drop, multi-selection, action icons, and inline editing.

## Build and Test Commands

```bash
# Build the library
cargo build

# Run tests
cargo test

# Run with serde feature
cargo build --features serde

# Run examples
cargo run --example basic
cargo run --example bevy_3d_outliner
```

## Architecture

### Core Design Pattern

The library follows egui's immediate mode paradigm with user-owned data:
- Users implement `OutlinerNode` on their data structures to provide hierarchy info
- Users implement `OutlinerActions` (or use `DefaultActions`) to handle interactions
- The `Outliner` widget reads user data, renders it, and returns events via `OutlinerResponse`
- State (expanded nodes, editing state) is stored in egui's memory system

### Key Flow

1. `Outliner::show()` is called each frame with user's tree data and actions handler
2. Widget reads tree and renders rows with indentation, icons, and labels
3. User interactions (clicks, drags, renames) trigger callbacks on `OutlinerActions`
4. `OutlinerResponse` is returned containing events (selection, rename, drop)
5. User modifies their tree data based on response
6. Next frame sees updated tree

### Module Structure

- `outliner.rs` - Main `Outliner` widget that renders the tree
- `traits.rs` - `OutlinerNode` and `OutlinerActions` traits
- `state.rs` - `OutlinerState` stored in egui memory (expanded nodes, editing)
- `response.rs` - `OutlinerResponse` containing interaction events
- `style.rs` - Visual styling configuration
- `drag_drop/` - Drag-drop state and visuals
- `tree_ops.rs` - Helper trait for tree manipulation (rename, remove, insert)
- `default_actions.rs` - Ready-to-use `OutlinerActions` implementation

### Trait System

`OutlinerNode` - Implement on your data structure:
- `id()` - Unique stable identifier
- `name()` - Display text
- `is_collection()` - Can have children
- `children()` / `children_mut()` - Child access
- `icon()` / `action_icons()` - Visual customization

`OutlinerActions<N>` - Handle user interactions:
- `on_select()` / `is_selected()` - Selection state
- `on_rename()` / `on_move()` - Modifications
- `is_visible()` / `is_locked()` - Icon state queries
- `on_visibility_toggle()` / `on_lock_toggle()` - Icon clicks

### Drag-Drop Pattern

Drag state is tracked internally during the operation. On drop:
1. `on_move()` callback fires
2. `DropEvent` is included in response
3. User removes node from old location and inserts at new location
4. Next frame renders updated tree

## Version Compatibility

| egui-arbor | egui | bevy | bevy_egui |
|------------|------|------|-----------|
| 0.3.x      | 0.33 | 0.17 | 0.38      |
| 0.2.x      | 0.31 | 0.16 | 0.34      |
