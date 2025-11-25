# Re.Video Examples Documentation

Comprehensive guide to all examples in the `@revideo/examples` package, demonstrating core concepts and features of the Re.Video animation library.

---

## Table of Contents

1. [Overview](#overview)
2. [Directory Structure](#directory-structure)
3. [Dependencies & Setup](#dependencies--setup)
4. [Examples Catalog](#examples-catalog)
   - [Basic Animations](#basic-animations)
   - [Tweening & Easing](#tweening--easing)
   - [Layout & Positioning](#layout--positioning)
   - [Media & Components](#media--components)
   - [Advanced Features](#advanced-features)
5. [Asset Files](#asset-files)
6. [Key Concepts Index](#key-concepts-index)
7. [Common Patterns & Techniques](#common-patterns--techniques)

---

## Quick Reference

All 20 examples at a glance:

| # | Example | Category | Key Feature |
|---|---------|----------|-------------|
| 1 | Quickstart | Basic Animations | Parallel animations with `all()` |
| 2 | TeX/LaTeX | Basic Animations | Mathematical formula rendering |
| 3 | Tweening - Linear | Tweening & Easing | Manual tweening with `tween()` |
| 4 | Tweening - Cubic | Tweening & Easing | Easing functions (`easeInOutCubic`) |
| 5 | Tweening - Color | Tweening & Easing | Color interpolation with `Color.lerp()` |
| 6 | Tweening - Vector | Tweening & Easing | Arc paths with `Vector2.arcLerp()` |
| 7 | Tweening - Spring | Tweening & Easing | Physics-based spring animations |
| 8 | Tweening - Save/Restore | Tweening & Easing | State management with `.save()/.restore()` |
| 9 | Tweening Visualizer | Tweening & Easing | Interactive easing visualization |
| 10 | Node & Signal | Layout & Positioning | Reactive signals and computed values |
| 11 | Layout | Layout & Positioning | Flexbox-style layouts with `grow` |
| 12 | Layout Groups | Layout & Positioning | Grouping with `Node` component |
| 13 | Positioning | Layout & Positioning | Transforms (position, rotation, scale) |
| 14 | Code Block | Media & Components | Animated code with syntax highlighting |
| 15 | Media - Image | Media & Components | Image loading and animation |
| 16 | Media - Video | Media & Components | Video playback with animations |
| 17 | Custom Components | Advanced Features | Extending `Node` class (Switch example) |
| 18 | Logging | Advanced Features | Debug tools and performance profiling |
| 19 | Scene Transitions | Advanced Features | Multi-scene projects with transitions |
| 20 | Presentation | Advanced Features | Slide-based presentations with `beginSlide()` |

---

## Overview

The `@revideo/examples` package provides 20 comprehensive examples demonstrating key features and patterns of the Re.Video animation library. Each example includes both a project entry point (`.ts` file) and a scene file (`.tsx`), showing how to create animations using Re.Video's JSX-based component system, reactive signals, tweening engines, and various animation techniques.

**Target Audience:** Developers learning Re.Video who want practical examples of common animation patterns.

**Use Cases:** Educational reference, implementation patterns, feature demonstrations, starting templates for custom projects.

---

## Directory Structure

```
/packages/examples/
├── package.json                    # Project dependencies
├── tsconfig.json                   # TypeScript configuration
├── vite.config.ts                  # Vite build configuration
├── code-block.meta                 # CodeBlock example metadata
├── revideo.d.ts                    # Type definitions
├── assets/
│   ├── example.mp4                 # Sample video file
│   └── logo.svg                    # Sample logo/image
├── src/
│   ├── components/
│   │   └── Switch.tsx              # Custom Switch component
│   ├── Project Entries (20 .ts files)
│   └── scenes/ (20 .tsx files)
```

### Project Entry Files (`src/`)

Each project is defined by a `.ts` file that serves as the entry point for the build system. These files import scene files and create a Re.Video project structure.

### Scene Files (`src/scenes/`)

Each scene is a `.tsx` JSX file containing the actual animation logic as a generator function.

---

## Dependencies & Setup

### Required Dependencies (from package.json)

```
@revideo/2d        - 2D rendering and component library
@revideo/core      - Core animation engine and utilities
@revideo/ffmpeg    - Video encoding and export
@revideo/ui        - UI components for the editor
@revideo/vite-plugin - Vite integration for development
```

### Build Configuration

**TypeScript:** Extends `@revideo/2d/tsconfig.project.json` with lib check skipped for faster compilation.

**Vite:** Configured with `@revideo/vite-plugin` for hot module reloading and project management. Outputs to `../docs/static/examples`.

**Scripts:**
- `npm run dev` - Start development server with hot reloading
- `npm run build` - Build all examples for production

---

## Examples Catalog

### Basic Animations

#### 1. Quickstart

**Purpose:** Introductory example showing basic circle animation with position and color changes.

**Project Entry** (`src/quickstart.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/quickstart';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/quickstart.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {all, createRef} from '@revideo/core';

export default makeScene2D('quickstart', function* (view) {
  const myCircle = createRef<Circle>();

  view.add(
    <Circle
      ref={myCircle}
      x={-300}
      width={140}
      height={140}
      fill="#e13238"
    />,
  );

  yield* all(
    myCircle().position.x(300, 1).to(-300, 1),
    myCircle().fill('#e6a700', 1).to('#e13238', 1),
  );
});
```

**Key Concepts:**
- `makeScene2D()` - Creates a 2D scene generator
- `createRef<Circle>()` - Reference to a node for later access
- `view.add()` - Add nodes to the scene
- `.to()` - Chain animations for back-and-forth motion
- `all()` - Run multiple animations in parallel
- JSX syntax for component creation

**Demonstrates:** Basic animation, parallel animations, property tweening

---

#### 2. TeX/LaTeX Mathematical Expressions

**Purpose:** Demonstrates rendering and animating mathematical LaTeX expressions.

**Project Entry** (`src/tex.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tex';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tex.tsx`):
```tsx
import {Latex, makeScene2D} from '@revideo/2d';
import {createRef, waitFor} from '@revideo/core';

export default makeScene2D('tex', function* (view) {
  const tex = createRef<Latex>();
  view.add(
    <Latex
      ref={tex}
      tex="{\color{white} x = \sin \left( \frac{\pi}{2} \right)}"
      y={0}
      width={400}
    />,
  );

  yield* waitFor(2);
  yield* tex().opacity(0, 1);
  yield* waitFor(2);
  yield* tex().opacity(1, 1);
});
```

**Key Concepts:**
- `Latex` component for mathematical rendering
- `waitFor()` - Pause execution for a duration
- `opacity()` - Fade animations
- LaTeX formula syntax with color support

**Demonstrates:** Mathematical content, opacity animations, timing control

---

### Tweening & Easing

#### 3. Tweening - Linear

**Purpose:** Basic tweening using the `tween` function with linear interpolation.

**Project Entry** (`src/tweening-linear.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tweening-linear';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tweening-linear.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {createRef, map, tween} from '@revideo/core';

export default makeScene2D('tweening-linear', function* (view) {
  const circle = createRef<Circle>();

  view.add(
    <Circle
      ref={circle}
      x={-300}
      width={240}
      height={240}
      fill="#e13238"
    />,
  );

  yield* tween(2, value => {
    circle().position.x(map(-300, 300, value));
  });
});
```

**Key Concepts:**
- `tween()` - Manual animation with frame callback
- `map()` - Linear interpolation between start and end values
- Frame-by-frame control
- Duration in seconds (2 = 2 seconds)

**Demonstrates:** Manual tweening, linear interpolation, direct property control

---

#### 4. Tweening - Cubic Easing

**Purpose:** Tweening with easing functions (cubic in/out) for smooth acceleration/deceleration.

**Project Entry** (`src/tweening-cubic.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tweening-cubic';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tweening-cubic.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {createRef, easeInOutCubic, map, tween} from '@revideo/core';

export default makeScene2D('tweening-cubic', function* (view) {
  const circle = createRef<Circle>();

  view.add(
    <Circle
      ref={circle}
      x={-300}
      width={240}
      height={240}
      fill="#e13238"
    />,
  );

  yield* tween(2, value => {
    circle().position.x(map(-300, 300, easeInOutCubic(value)));
  });
});
```

**Key Concepts:**
- `easeInOutCubic()` - Easing function for smooth motion
- Easing applied to normalized value (0-1) before mapping
- Combined with `map()` for value interpolation

**Demonstrates:** Easing functions, smooth acceleration, advanced tweening

---

#### 5. Tweening - Color

**Purpose:** Smooth color interpolation between two colors using `Color.lerp()`.

**Project Entry** (`src/tweening-color.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tweening-color';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tweening-color.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {Color, createRef, easeInOutCubic, tween} from '@revideo/core';

export default makeScene2D('tweening-color', function* (view) {
  const circle = createRef<Circle>();

  view.add(
    <Circle
      ref={circle}
      width={240}
      height={240}
      fill="#e13238"
    />,
  );

  yield* tween(2, value => {
    circle().fill(
      Color.lerp(
        new Color('#e13238'),
        new Color('#e6a700'),
        easeInOutCubic(value),
      ),
    );
  });
});
```

**Key Concepts:**
- `Color.lerp()` - Linear interpolation in RGB space
- `Color` class for color manipulation
- Colors can be created from hex strings

**Demonstrates:** Color interpolation, Color class usage, easing in color space

---

#### 6. Tweening - Vector

**Purpose:** Interpolating 2D positions along an arc path using `Vector2.arcLerp()`.

**Project Entry** (`src/tweening-vector.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tweening-vector';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tweening-vector.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {Vector2, createRef, easeInOutCubic, tween} from '@revideo/core';

export default makeScene2D('tweening-vector', function* (view) {
  const circle = createRef<Circle>();

  view.add(
    <Circle
      ref={circle}
      x={-300}
      y={200}
      width={240}
      height={240}
      fill="#e13238"
    />,
  );

  yield* tween(2, value => {
    circle().position(
      Vector2.arcLerp(
        new Vector2(-300, 200),
        new Vector2(300, -200),
        easeInOutCubic(value),
      ),
    );
  });
});
```

**Key Concepts:**
- `Vector2.arcLerp()` - Arc interpolation for curved paths
- `Vector2` class for 2D vector operations
- Curved motion instead of linear motion

**Demonstrates:** Vector interpolation, arc paths, curved motion

---

#### 7. Tweening - Spring Physics

**Purpose:** Physics-based spring animations for natural bouncy or smooth motion.

**Project Entry** (`src/tweening-spring.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tweening-spring';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tweening-spring.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {PlopSpring, SmoothSpring, createRef, spring} from '@revideo/core';

export default makeScene2D('tweening-spring', function* (view) {
  const circle = createRef<Circle>();

  view.add(
    <Circle
      ref={circle}
      x={-400}
      size={240}
      fill={'#e13238'}
    />,
  );

  yield* spring(PlopSpring, -400, 400, 1, value => {
    circle().position.x(value);
  });
  yield* spring(SmoothSpring, 400, -400, value => {
    circle().position.x(value);
  });
});
```

**Key Concepts:**
- `spring()` - Physics-based animation engine
- `PlopSpring` - Preset with bouncy/overshoot behavior
- `SmoothSpring` - Preset with damped smooth motion
- Different spring presets create different visual effects
- Duration and callback for value updates

**Demonstrates:** Spring physics, preset animations, natural motion

---

#### 8. Tweening - Save & Restore State

**Purpose:** Saving and restoring animation states for complex sequences with state reversals.

**Project Entry** (`src/tweening-save-restore.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tweening-save-restore';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tweening-save-restore.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {all, createRef} from '@revideo/core';

export default makeScene2D('tweening-save-restore', function* (view) {
  const circle = createRef<Circle>();

  view.add(
    <Circle
      ref={circle}
      size={150}
      position={[-300, -300]}
      fill={'#e13238'}
    />,
  );

  circle().save();
  yield* all(circle().position.x(0, 1), circle().scale(1.5, 1));

  circle().save();
  yield* all(circle().position.y(0, 1), circle().scale(0.5, 1));

  circle().save();
  yield* all(circle().position.x(300, 1), circle().scale(1, 1));

  yield* circle().restore(1);
  yield* circle().restore(1);
  yield* circle().restore(1);
});
```

**Key Concepts:**
- `.save()` - Save current state to a stack
- `.restore()` - Restore to a previously saved state with animation duration
- State stacking (LIFO - Last In First Out)
- Useful for complex multi-step animations

**Demonstrates:** State management, animation reversals, complex sequences

---

#### 9. Tweening Visualizer

**Purpose:** Interactive visualization of easing functions showing how they affect animation over time.

**Project Entry** (`src/tweening-visualiser.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/tweening-visualiser';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/tweening-visualiser.tsx`):
```tsx
import {
  Circle,
  Grid,
  Layout,
  Line,
  Node,
  Rect,
  Txt,
  makeScene2D,
} from '@revideo/2d';
import {
  all,
  createSignal,
  easeInOutBounce,
  linear,
  waitFor,
} from '@revideo/core';

export default makeScene2D('tweening-visualizer', function* (view) {
  const time = createSignal(0);
  const value = createSignal(0);
  const TIME = 3.5;

  view.add(
    <Node y={-30}>
      <Grid size={700} stroke={'#444'} lineWidth={3} spacing={100}>
        <Rect
          layout
          size={100}
          offset={[-1, 1]}
          x={() => time() * 500 - 300}
          y={() => value() * -500 + 300}
        >
          <Circle size={60} fill={'#C22929'} margin={20}></Circle>
        </Rect>
      </Grid>

      <Node position={[-400, -400]}>
        <Line
          lineWidth={4}
          points={[[0, 750], [0, 35]]}
          stroke={'#DDD'}
          lineCap={'round'}
          endArrow
          arrowSize={15}
        ></Line>

        <Layout y={() => value() * -500 + 650}>
          <Txt
            fill={'#DDD'}
            text={() => value().toFixed(2).toString()}
            fontWeight={300}
            fontSize={30}
            x={-55}
            y={3}
          ></Txt>
          <Circle size={30} fill={'#DDD'}></Circle>
        </Layout>

        <Txt
          y={400}
          x={-160}
          fontWeight={400}
          fontSize={50}
          padding={20}
          fontFamily={'Candara'}
          fill={'#DDD'}
          text={'VALUE'}
        ></Txt>
      </Node>

      <Node position={[-400, -400]}>
        <Line
          lineWidth={4}
          points={[[50, 800], [765, 800]]}
          stroke={'#DDD'}
          lineCap={'round'}
          endArrow
          arrowSize={15}
        ></Line>

        <Layout y={800} x={() => time() * 500 + 150}>
          <Circle size={30} fill={'#DDD'}></Circle>
          <Txt
            fill={'#DDD'}
            text={() => (time() * TIME).toFixed(2).toString()}
            fontWeight={300}
            fontSize={30}
            y={50}
          ></Txt>
        </Layout>

        <Txt
          y={900}
          x={400}
          fontWeight={400}
          fontSize={50}
          padding={20}
          fontFamily={'Candara'}
          fill={'#DDD'}
          text={'TIME'}
        ></Txt>
      </Node>
    </Node>,
  );

  yield* waitFor(0.5);
  yield* all(time(1, TIME, linear), value(1, TIME, easeInOutBounce));
  yield* waitFor(0.8);
});
```

**Key Concepts:**
- `createSignal()` - Reactive values with dependencies
- Computed properties using arrow functions `() => ...`
- Signal dependencies update automatically
- Grid visualization of coordinate space
- `easeInOutBounce` - Bounce easing function
- `Layout` component for dynamic positioning
- Coordinate axes with labels and arrows

**Demonstrates:** Signals, easing visualization, dynamic layouts, reactive properties

---

### Layout & Positioning

#### 10. Node & Signal - Reactive Programming

**Purpose:** Demonstrates reactive signals and computed properties with real-time updates.

**Project Entry** (`src/node-signal.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/node-signal';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/node-signal.tsx`):
```tsx
import {Circle, Line, Txt, makeScene2D} from '@revideo/2d';
import {Vector2, createSignal, waitFor} from '@revideo/core';

export default makeScene2D('node-signal', function* (view) {
  const radius = createSignal(3);
  const area = createSignal(() => Math.PI * radius() * radius());

  const scale = 100;
  const textStyle = {
    fontWeight: 700,
    fontSize: 56,
    offsetY: -1,
    padding: 20,
    cache: true,
  };

  view.add(
    <>
      <Circle
        width={() => radius() * scale * 2}
        height={() => radius() * scale * 2}
        fill={'#e13238'}
      />
      <Line
        points={[
          Vector2.zero,
          () => Vector2.right.scale(radius() * scale),
        ]}
        lineDash={[20, 20]}
        startArrow
        endArrow
        endOffset={8}
        lineWidth={8}
        stroke={'#242424'}
      />
      <Txt
        text={() => `r = ${radius().toFixed(2)}`}
        x={() => (radius() * scale) / 2}
        fill={'#242424'}
        {...textStyle}
      />
      <Txt
        text={() => `A = ${area().toFixed(2)}`}
        y={() => radius() * scale}
        fill={'#e13238'}
        {...textStyle}
      />
    </>,
  );

  yield* radius(4, 2).to(3, 2);
  yield* waitFor(1);
});
```

**Key Concepts:**
- `createSignal()` - Create reactive state
- Computed signals using function syntax
- Signal dependencies automatically trigger updates
- Vector operations: `Vector2.right.scale()`
- Dynamic text with computed values
- `cache` property for performance optimization

**Demonstrates:** Reactive programming, computed values, signal dependencies, dynamic content

---

#### 11. Layout - Flexbox-style Layouts

**Purpose:** Demonstrates the Layout component with flexbox properties and animated grow factors.

**Project Entry** (`src/layout.ts`):
```typescript
import {makeProject} from '@revideo/core';
import layout from './scenes/layout';

export default makeProject({
  scenes: [layout],
});
```

**Scene** (`src/scenes/layout.tsx`):
```tsx
import {Circle, Layout, Rect, makeScene2D} from '@revideo/2d';
import {all, createRef} from '@revideo/core';

const RED = '#ff6470';

export default makeScene2D('layout', function* (view) {
  const colA = createRef<Layout>();
  const colB = createRef<Layout>();
  const rowA = createRef<Layout>();

  view.add(
    <>
      <Layout layout gap={10} padding={10} width={440} height={240}>
        <Rect ref={colA} grow={1} fill={'#242424'} radius={4} />
        <Layout gap={10} direction="column" grow={3}>
          <Rect
            ref={rowA}
            grow={8}
            fill={RED}
            radius={4}
            stroke={'#fff'}
            lineWidth={4}
            margin={2}
          >
            <Circle layout={false} width={20} height={20} fill={'#fff'} />
          </Rect>
          <Rect grow={2} fill={'#242424'} radius={4} />
        </Layout>
        <Rect ref={colB} grow={3} fill={'#242424'} radius={4} />
      </Layout>
    </>,
  );

  yield* all(colB().grow(1, 0.8), colA().grow(2, 0.8));
  yield* rowA().grow(1, 0.8);
  yield* all(colB().grow(3, 0.8), colA().grow(1, 0.8));
  yield* rowA().grow(8, 0.8);
});
```

**Key Concepts:**
- `Layout` component for flexbox layouts
- `direction` - "row" (default) or "column"
- `grow` - Flex grow factor for proportional sizing
- `gap` - Space between items
- `padding` - Internal spacing
- `margin` - External spacing
- Nested layouts for complex structures
- `layout={false}` - Disable layout for specific children

**Demonstrates:** Layout system, flex properties, nested layouts, animated flex sizes

---

#### 12. Layout Groups

**Purpose:** Grouping elements within layouts using Node without applying layout styles.

**Project Entry** (`src/layout-group.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/layout-group';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/layout-group.tsx`):
```tsx
import {Layout, Node, Rect, makeScene2D} from '@revideo/2d';
import {createRef} from '@revideo/core';

export default makeScene2D('layout-group', function* (view) {
  const group = createRef<Node>();

  view.add(
    <>
      <Layout direction={'column'} width={960} gap={40} layout>
        <Node ref={group}>
          <Rect height={240} fill={'#ff6470'} />
          <Rect height={240} fill={'#ff6470'} />
        </Node>
        <Rect height={240} fill={'#ff6470'} />
      </Layout>
    </>,
  );

  yield* group().opacity(0.1, 1).to(1, 1);
});
```

**Key Concepts:**
- `Node` for grouping without layout behavior
- Animating a group as a single unit
- Opacity animations on groups
- Groups are treated as single items in layouts

**Demonstrates:** Node grouping, group animations, layout integration

---

#### 13. Positioning & Transforms

**Purpose:** Demonstrates coordinate systems, transformations (position, rotation, scale), and visualization.

**Project Entry** (`src/positioning.ts`):
```typescript
import {makeProject} from '@revideo/core';
import node from './scenes/positioning';

export default makeProject({
  scenes: [node],
});
```

**Scene** (`src/scenes/positioning.tsx`):
```tsx
import {Circle, Grid, Line, Node, makeScene2D} from '@revideo/2d';
import {Vector2, all, createRef, createSignal} from '@revideo/core';

const RED = '#ff6470';
const GREEN = '#99C47A';
const BLUE = '#68ABDF';

export default makeScene2D('positioning', function* (view) {
  const group = createRef<Node>();
  const scale = createSignal(1);

  view.add(
    <Node ref={group} x={-100}>
      <Grid
        width={1920}
        height={1920}
        spacing={() => scale() * 60}
        stroke={'#444'}
        lineWidth={1}
        lineCap="square"
        cache
      />
      <Circle
        width={() => scale() * 120}
        height={() => scale() * 120}
        stroke={BLUE}
        lineWidth={4}
        startAngle={110}
        endAngle={340}
      />
      <Line
        stroke={RED}
        lineWidth={4}
        endArrow
        arrowSize={10}
        points={[Vector2.zero, () => Vector2.right.scale(scale() * 70)]}
      />
      <Line
        stroke={GREEN}
        lineWidth={4}
        endArrow
        arrowSize={10}
        points={[Vector2.zero, () => Vector2.up.scale(scale() * 70)]}
      />
      <Circle width={20} height={20} fill="#fff" />
    </Node>,
  );

  yield* group().position.x(100, 0.8);
  yield* group().rotation(30, 0.8);
  yield* scale(2, 0.8);
  yield* group().position.x(-100, 0.8);
  yield* all(group().rotation(0, 0.8), scale(1, 0.8));
});
```

**Key Concepts:**
- `Grid` component for coordinate system visualization
- Position animations: `position.x()`, `position.y()`
- Rotation transformations
- Scale animations on signals
- Vector axes (RED for X-axis, GREEN for Y-axis)
- `startAngle`/`endAngle` for partial arcs
- `lineCap` for line styling

**Demonstrates:** Coordinate systems, transformations, dynamic grids, vector visualization

---

### Media & Components

#### 14. Code Block - Animated Code Editing

**Purpose:** Demonstrates the CodeBlock component for animated code highlighting and editing.

**Project Entry** (`src/code-block.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/code-block';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/code-block.tsx`):
```tsx
import {makeScene2D} from '@revideo/2d';
import {
  CodeBlock,
  edit,
  insert,
  lines,
  remove,
} from '@revideo/2d/lib/components/CodeBlock';
import {all, createRef, waitFor} from '@revideo/core';

export default makeScene2D('code-block', function* (view) {
  const codeRef = createRef<CodeBlock>();

  yield view.add(<CodeBlock ref={codeRef} code={`var myBool;`} />);

  yield* codeRef().edit(1.2, false)`var myBool${insert(' = true')};`;
  yield* waitFor(0.6);
  yield* codeRef().edit(1.2)`var myBool = ${edit('true', 'false')};`;
  yield* waitFor(0.6);
  yield* all(
    codeRef().selection(lines(0, Infinity), 1.2),
    codeRef().edit(1.2, false)`var my${edit('Bool', 'Number')} = ${edit(
      'false',
      '42',
    )};`,
  );
  yield* waitFor(0.6);
  yield* codeRef().edit(1.2, false)`var myNumber${remove(' = 42')};`;
  yield* waitFor(0.6);
  yield* codeRef().edit(1.2, false)`var my${edit('Number', 'Bool')};`;
  yield* waitFor(0.6);
});
```

**Key Concepts:**
- `CodeBlock` component for syntax-highlighted code
- `edit()` - Replace text with animation
- `insert()` - Add new text
- `remove()` - Delete text
- `lines()` - Select range of lines
- `.selection()` - Highlight code sections
- Template literal syntax for inline edits
- Syntax highlighting included automatically

**Demonstrates:** Code animation, text editing, selection highlighting, code display

---

#### 15. Media - Image

**Purpose:** Loading and animating images using the Img component.

**Project Entry** (`src/media-image.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/media-image';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/media-image.tsx`):
```tsx
import {Img, makeScene2D} from '@revideo/2d';
import {all, createRef} from '@revideo/core';

import logoSvg from '@revideo/examples/assets/logo.svg';

export default makeScene2D('media-image', function* (view) {
  const imageRef = createRef<Img>();

  yield view.add(<Img ref={imageRef} src={logoSvg} scale={2} />);

  yield* all(
    imageRef().scale(2.5, 1.5).to(2, 1.5),
    imageRef().absoluteRotation(90, 1.5).to(0, 1.5),
  );
});
```

**Key Concepts:**
- `Img` component for image display
- Direct SVG/image imports with Vite
- `scale` property for sizing
- `absoluteRotation` - Rotation independent of parent transformations
- `.to()` for animations that reverse

**Demonstrates:** Image loading, scaling, rotation, asset imports

---

#### 16. Media - Video

**Purpose:** Loading and controlling video playback with animations.

**Project Entry** (`src/media-video.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/media-video';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/media-video.tsx`):
```tsx
import {Video, makeScene2D} from '@revideo/2d';
import {createRef} from '@revideo/core';

import exampleMp4 from '@revideo/examples/assets/example.mp4';

export default makeScene2D('media-video', function* (view) {
  const videoRef = createRef<Video>();

  view.add(<Video ref={videoRef} src={exampleMp4} />);

  videoRef().play();
  yield* videoRef().scale(1.25, 2).to(1, 2);
});
```

**Key Concepts:**
- `Video` component for video file playback
- `.play()` method to start playback
- Video plays in parallel with animations
- Scale animations during video playback

**Demonstrates:** Video playback, video controls, concurrent animations

---

### Advanced Features

#### 17. Custom Components - Switch Toggle

**Purpose:** Creating reusable custom components by extending the Node class.

**Project Entry** (`src/components.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/components';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/components.tsx`):
```tsx
import {makeScene2D} from '@revideo/2d';
import {createRef, waitFor} from '@revideo/core';
import {Switch} from '@revideo/examples/src/components/Switch';

export default makeScene2D('components', function* (view) {
  const switchRef = createRef<Switch>();

  view.add(<Switch ref={switchRef} initialState={true} />);

  yield* switchRef().toggle(0.6);
  yield* waitFor(1);
  yield* switchRef().toggle(0.6);
  yield* waitFor(1);
});
```

**Custom Component** (`src/components/Switch.tsx`):
```tsx
import type {NodeProps} from '@revideo/2d';
import {Circle, Node, Rect, colorSignal, initial, signal} from '@revideo/2d';
import type {
  ColorSignal,
  PossibleColor,
  SignalValue,
  SimpleSignal,
} from '@revideo/core';
import {
  Color,
  all,
  createRef,
  createSignal,
  easeInOutCubic,
  tween,
} from '@revideo/core';

export interface SwitchProps extends NodeProps {
  initialState?: SignalValue<boolean>;
  accent?: SignalValue<PossibleColor>;
}

export class Switch extends Node {
  @initial(false)
  @signal()
  public declare readonly initialState: SimpleSignal<boolean, this>;

  @initial('#68ABDF')
  @colorSignal()
  public declare readonly accent: ColorSignal<this>;

  private isOn: boolean;
  private readonly indicatorPosition = createSignal(0);
  private readonly offColor = new Color('#242424');
  private readonly indicator = createRef<Circle>();
  private readonly container = createRef<Rect>();

  public constructor(props?: SwitchProps) {
    super({
      ...props,
    });

    this.isOn = this.initialState();
    this.indicatorPosition(this.isOn ? 50 : -50);

    this.add(
      <Rect
        ref={this.container}
        fill={this.isOn ? this.accent() : this.offColor}
        size={[200, 100]}
        radius={100}
      >
        <Circle
          x={() => this.indicatorPosition()}
          ref={this.indicator}
          size={[80, 80]}
          fill="#ffffff"
        />
      </Rect>,
    );
  }

  public *toggle(duration: number) {
    yield* all(
      tween(duration, value => {
        const oldColor = this.isOn ? this.accent() : this.offColor;
        const newColor = this.isOn ? this.offColor : this.accent();

        this.container().fill(
          Color.lerp(oldColor, newColor, easeInOutCubic(value)),
        );
      }),

      tween(duration, value => {
        const currentPos = this.indicator().position();

        this.indicatorPosition(
          easeInOutCubic(value, currentPos.x, this.isOn ? -50 : 50),
        );
      }),
    );
    this.isOn = !this.isOn;
  }
}
```

**Key Concepts:**
- Extending `Node` class for custom components
- `@signal()` decorator for reactive properties
- `@initial()` decorator for default values
- `@colorSignal()` for color-specific signals
- Custom generator methods (`*toggle()`)
- TypeScript interfaces for prop types
- Private state management
- Constructor initialization
- `createRef()` for internal references

**Demonstrates:** Custom components, decorators, OOP patterns, reusable animations

---

#### 18. Logging & Debugging

**Purpose:** Using the logging system for debugging and profiling animations.

**Project Entry** (`src/logging.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/logging';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/logging.tsx`):
```tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {useLogger, waitFor} from '@revideo/core';

export default makeScene2D('logging', function* (view) {
  const logger = useLogger();

  view.add(<Circle key="circle" />);

  logger.debug('Just here to debug some code.');
  logger.info('All fine, just a little info.');
  logger.warn('Be careful, something has gone wrong.');
  logger.error('Oops. An error occured.');

  logger.debug({
    message: 'Some more advanced logging',
    inspect: 'circle',
    object: {
      someProperty: 'some property value',
    },
    durationMs: 200,
    remarks: 'Some remarks about this log. Can also contain <b>HTML</b> tags.',
    stack: new Error('').stack,
  });

  logger.profile('id');
  yield* waitFor(2);
  logger.profile('id', {
    message: 'Id second call',
    object: {
      someProperty: 'some property value',
    },
  });

  yield* waitFor(2);
});
```

**Key Concepts:**
- `useLogger()` - Get logger instance in scene
- Log levels: `debug()`, `info()`, `warn()`, `error()`
- Simple string logging
- Advanced logging with object details
- `inspect` property to show specific node state
- `remarks` with HTML support
- `profile()` for performance measurement
- Stack traces for debugging

**Demonstrates:** Debugging tools, logging, performance profiling

---

#### 19. Scene Transitions

**Purpose:** Multi-scene projects with transitions between scenes.

**Project Entry** (`src/transitions.ts`):
```typescript
import {makeProject} from '@revideo/core';

import first from './scenes/transitions-first';
import second from './scenes/transitions-second';

export default makeProject({
  scenes: [first, second],
});
```

**First Scene** (`src/scenes/transitions-first.tsx`):
```tsx
import {Rect, Txt, makeScene2D} from '@revideo/2d';
import {waitFor} from '@revideo/core';

export default makeScene2D('transitions-first', function* (view) {
  view.add(
    <Rect
      width={'100%'}
      height={'100%'}
      fill={'lightseagreen'}
      layout
      alignItems={'center'}
      justifyContent={'center'}
    >
      <Txt
        fontSize={160}
        fontWeight={700}
        fill={'#fff'}
        fontFamily={'"JetBrains Mono", monospace'}
      >
        FIRST SCENE
      </Txt>
    </Rect>,
  );

  yield* waitFor(1);
});
```

**Second Scene** (`src/scenes/transitions-second.tsx`):
```tsx
import {Rect, Txt, makeScene2D} from '@revideo/2d';
import {
  Direction,
  all,
  createRef,
  slideTransition,
  waitFor,
} from '@revideo/core';

export default makeScene2D('transitions-second', function* (view) {
  const rect = createRef<Rect>();
  const text = createRef<Txt>();

  view.add(
    <Rect
      ref={rect}
      width={'100%'}
      height={'100%'}
      fill={'lightcoral'}
      layout
      alignItems={'center'}
      justifyContent={'center'}
    >
      <Txt
        ref={text}
        fontSize={160}
        fontWeight={700}
        fill={'#fff'}
        fontFamily={'"JetBrains Mono", monospace'}
      >
        SECOND SCENE
      </Txt>
    </Rect>,
  );

  yield* slideTransition(Direction.Left);

  yield* waitFor(0.4);
  yield* all(
    rect().fill('lightseagreen', 0.6),
    text().text('FIRST SCENE', 0.6),
  );
});
```

**Key Concepts:**
- Multiple scenes in single project
- `slideTransition()` - Animated transition between scenes
- `Direction` enum: `Left`, `Right`, `Up`, `Down`
- Percentage-based sizing: `'100%'`
- `layout` - Enable flexbox layout
- `alignItems`/`justifyContent` - Layout alignment
- Scenes execute sequentially
- Transitions happen at scene start

**Demonstrates:** Multi-scene projects, scene transitions, layout properties

---

#### 20. Presentation Mode

**Purpose:** Advanced presentation with slides and interactive elements using `beginSlide()`.

**Project Entry** (`src/presentation.ts`):
```typescript
import {makeProject} from '@revideo/core';
import scene from './scenes/presentation';

export default makeProject({
  scenes: [scene],
});
```

**Scene** (`src/scenes/presentation.tsx`):
```tsx
import {Rect, Txt, makeScene2D} from '@revideo/2d';
import {
  Color,
  all,
  beginSlide,
  cancel,
  createRef,
  createSignal,
  easeInOutCubic,
  loop,
} from '@revideo/core';

const YELLOW = '#FFC66D';
const RED = '#FF6470';
const GREEN = '#99C47A';
const BLUE = '#68ABDF';

export default makeScene2D('presentation', function* (view) {
  view.fontFamily(`'JetBrains Mono', monospace`).fontWeight(700).fontSize(256);
  const backdrop = createRef<Rect>();
  const title = createRef<Txt>();
  const rotation = createSignal(0);
  const rotationScale = createSignal(0);

  view.add(
    <Rect
      cache
      ref={backdrop}
      width={'50%'}
      height={'50%'}
      fill={RED}
      radius={40}
      smoothCorners
      rotation={() => rotation() * rotationScale()}
    >
      <Txt
        ref={title}
        scale={0.5}
        compositeOperation={'destination-out'}
        rotation={() => -rotation() * rotationScale()}
      >
        START
      </Txt>
    </Rect>,
  );

  yield* beginSlide('start');
  yield* all(
    backdrop().fill(GREEN, 0.6, easeInOutCubic, Color.createLerp('lab')),
    backdrop().size.x('60%', 0.6),
    title().text('CONTENT', 0.6),
  );

  yield* beginSlide('content');
  const loopTask = yield loop(Infinity, () => rotation(-5, 1).to(5, 1));
  yield* all(
    backdrop().fill(BLUE, 0.6, easeInOutCubic, Color.createLerp('lab')),
    backdrop().size.x('70%', 0.6),
    title().text('ANIMATION', 0.6),
    rotationScale(1, 0.6),
  );

  yield* beginSlide('animation');
  yield* all(
    backdrop().fill(YELLOW, 0.6, easeInOutCubic, Color.createLerp('lab')),
    backdrop().size.x('50%', 0.6),
    title().text('FINISH', 0.6),
    rotationScale(0, 0.6),
  );
  cancel(loopTask);

  yield* beginSlide('finish');
});
```

**Key Concepts:**
- `beginSlide()` - Mark presentation slides for interactive navigation
- `loop()` - Infinite repeating animations
- `cancel()` - Stop running tasks/loops
- `Color.createLerp('lab')` - LAB color space interpolation for natural colors
- `compositeOperation` - Blend modes (e.g., 'destination-out' for cutout)
- `smoothCorners` - Anti-aliased corner rendering
- `cache` - Cache rendered output for performance
- View-level style settings: `fontFamily()`, `fontWeight()`, `fontSize()`
- Size properties: `size.x()` for width animations

**Demonstrates:** Presentation mode, loops, task cancellation, color spaces, blend modes

---

## Asset Files

### /packages/examples/assets/

**logo.svg** - SVG logo file used in the media-image example. Imported and scaled for animation demonstrations.

**example.mp4** - Sample video file used in the media-video example. Demonstrates video playback integration with animations.

Assets are imported using Vite's import system and can be used with `Img` and `Video` components.

---

## Key Concepts Index

### Core Animation Concepts

| Concept | Example | Purpose |
|---------|---------|---------|
| **Basic Animation** | Quickstart | Simple property tweening |
| **Parallel Animations** | Quickstart | Multiple animations at once with `all()` |
| **Sequential Animations** | Most examples | Using `yield*` to order animations |
| **Manual Tweening** | Tweening-Linear | Frame-by-frame control with `tween()` |
| **Easing Functions** | Tweening-Cubic | Smooth acceleration with easing |
| **Color Interpolation** | Tweening-Color | Smooth color transitions |
| **Vector Interpolation** | Tweening-Vector | Curved motion with `Vector2.arcLerp()` |
| **Spring Physics** | Tweening-Spring | Natural bouncy motion |
| **State Management** | Tweening-Save-Restore | Save/restore animation states |

### Reactive Programming

| Concept | Example | Purpose |
|---------|---------|---------|
| **Signals** | Node-Signal, Tweening-Visualizer | Reactive state values |
| **Computed Signals** | Node-Signal | Automatic updates from dependencies |
| **Dynamic Properties** | All examples | Arrow functions in properties |
| **Signal Dependencies** | Node-Signal | Reactive calculations |

### Layout & Positioning

| Concept | Example | Purpose |
|---------|---------|---------|
| **Layout Component** | Layout | Flexbox-style layouts |
| **Grow Factor** | Layout | Proportional sizing |
| **Direction** | Layout, Layout-Group | Row/column layout direction |
| **Spacing** | Layout | Gap, padding, margin properties |
| **Grid** | Positioning | Coordinate system visualization |
| **Transforms** | Positioning | Position, rotation, scale animations |
| **Node Grouping** | Layout-Group | Group items without layout |

### Components

| Concept | Example | Purpose |
|---------|---------|---------|
| **Circle/Rect/Shapes** | Quickstart | Basic drawable shapes |
| **Text** | Most examples | Text rendering with styling |
| **Latex** | TeX | Mathematical formula rendering |
| **CodeBlock** | Code-Block | Syntax-highlighted code with animations |
| **Image** | Media-Image | Image display and animation |
| **Video** | Media-Video | Video playback |
| **Layout** | Layout | Flexbox containers |
| **Grid** | Positioning | Coordinate grid visualization |
| **Line** | Node-Signal, Positioning | Line drawing with arrows |

### Advanced Features

| Concept | Example | Purpose |
|---------|---------|---------|
| **Custom Components** | Components | Extending Node class |
| **Decorators** | Components | @signal, @initial, @colorSignal |
| **Logging** | Logging | Debug and profiling tools |
| **Scene Transitions** | Transitions | Animated transitions between scenes |
| **Presentation Mode** | Presentation | Interactive slide navigation |
| **Loops** | Presentation | Infinite repeating animations |
| **Task Cancellation** | Presentation | Stop running tasks |
| **Color Spaces** | Presentation | LAB color interpolation |
| **Blend Modes** | Presentation | compositeOperation property |
| **Caching** | Presentation, Positioning | Performance optimization |

---

## Common Patterns & Techniques

### Animation Patterns

#### 1. Basic Property Animation
```typescript
yield* circle().position.x(300, 1);  // Animate to value over 1 second
yield* circle().position.x(300, 1).to(-300, 1);  // Chain animations
```

#### 2. Parallel Animations
```typescript
yield* all(
  circle().position.x(300, 1),
  circle().scale(2, 1),
  circle().fill('#FF0000', 1),
);
```

#### 3. Sequential Animations
```typescript
yield* circle().position.x(300, 1);
yield* circle().scale(2, 1);
yield* circle().opacity(0, 1);
```

#### 4. Manual Tweening with Easing
```typescript
yield* tween(2, value => {
  circle().position.x(map(-300, 300, easeInOutCubic(value)));
});
```

#### 5. Color Interpolation
```typescript
yield* tween(2, value => {
  circle().fill(
    Color.lerp(
      new Color('#e13238'),
      new Color('#e6a700'),
      easeInOutCubic(value),
    ),
  );
});
```

### Signal Patterns

#### 1. Creating Reactive State
```typescript
const radius = createSignal(3);
yield* radius(5, 2);  // Animate signal value
```

#### 2. Computed Signals
```typescript
const radius = createSignal(3);
const area = createSignal(() => Math.PI * radius() * radius());
```

#### 3. Dynamic Properties
```typescript
<Circle width={() => radius() * scale * 2} />
```

### Layout Patterns

#### 1. Flexbox Layout
```typescript
<Layout layout gap={10} padding={10} direction="row">
  <Rect grow={1} />
  <Rect grow={2} />
</Layout>
```

#### 2. Nested Layouts
```typescript
<Layout direction="column">
  <Layout direction="row" grow={1}>
    <Rect grow={1} />
    <Rect grow={1} />
  </Layout>
  <Rect grow={1} />
</Layout>
```

#### 3. Layout Groups
```typescript
<Node>
  <Rect />
  <Rect />
</Node>
```

### Component Patterns

#### 1. Creating References
```typescript
const circle = createRef<Circle>();
// Later: circle().position.x(300, 1)
```

#### 2. Percentage-Based Sizing
```typescript
<Rect width={'100%'} height={'100%'} />
```

#### 3. Centered Content
```typescript
<Rect layout alignItems={'center'} justifyContent={'center'}>
  <Txt>Centered</Txt>
</Rect>
```

### Custom Component Patterns

#### 1. Extending Node
```typescript
export class Switch extends Node {
  public constructor(props?: SwitchProps) {
    super({...props});
    // Initialize component
  }

  public *toggle(duration: number) {
    // Generator method for animations
  }
}
```

#### 2. Signal Properties
```typescript
@initial(false)
@signal()
public declare readonly initialState: SimpleSignal<boolean, this>;
```

### Debugging Patterns

#### 1. Logging
```typescript
const logger = useLogger();
logger.debug('Message');
logger.info('Info');
logger.warn('Warning');
logger.error('Error');
```

#### 2. Advanced Logging
```typescript
logger.debug({
  message: 'Description',
  inspect: 'nodeKey',
  object: {prop: 'value'},
  durationMs: 200,
  remarks: 'HTML remarks',
});
```

#### 3. Performance Profiling
```typescript
logger.profile('id');
yield* waitFor(2);
logger.profile('id', {message: 'Completed'});
```

### Presentation Patterns

#### 1. Slide Navigation
```typescript
yield* beginSlide('slide-name');
// Animations for this slide
yield* beginSlide('next-slide');
```

#### 2. Infinite Loops
```typescript
const task = yield loop(Infinity, () =>
  rotation(-5, 1).to(5, 1)
);
// Later:
cancel(task);
```

#### 3. Color Space Interpolation
```typescript
yield* color(targetColor, duration, easeInOutCubic, Color.createLerp('lab'));
```

---

## Configuration Reference

### Project Configuration (vite.config.ts)

The examples package is configured with:
- **Plugin:** `@revideo/vite-plugin`
- **Output Directory:** `../docs/static/examples`
- **Build Target:** Separate JavaScript bundles per project

Each project file listed in `project` array generates a separate animation that can be independently rendered and exported.

### TypeScript Configuration (tsconfig.json)

- Extends `@revideo/2d/tsconfig.project.json`
- Includes `src` directory
- Skips library type checking for faster compilation

### Code Block Configuration (code-block.meta)

Preview and rendering settings for the Code Block example:
- Preview: 30 fps, 1x resolution scale
- Rendering: 60 fps, 2x resolution scale
- Output: PNG sequence, 100% quality
- Size: 1920x1080

---

## Tips for Learning & Development

1. **Start with Quickstart** - Understand basic animation patterns first
2. **Explore Tweening** - Learn different animation techniques before moving to advanced features
3. **Understand Signals** - Reactive programming is essential for complex animations
4. **Study Layouts** - Flexbox-style layouts are used in most real projects
5. **Use Logging** - Debug your animations using the logger
6. **Reference Components** - Look at Switch example to understand custom component patterns
7. **Experiment** - Modify examples to learn how properties affect animations
8. **Read Comments** - Each example includes helpful comments explaining the code

---

## Running Examples

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

All examples are available in the development server and can be rendered individually or as part of the full build process.

---

## Search Keywords Index

Keyword index to help find relevant examples quickly. Keywords are organized by topic.

### Animation Techniques
- **Parallel animations**: Quickstart (#1), Node-Signal (#10), Layout (#11)
- **Sequential animations**: All examples
- **Tweening**: Linear (#3), Cubic (#4), Color (#5), Vector (#6), Spring (#7), Visualizer (#9)
- **Easing**: Cubic (#4), Color (#5), Vector (#6), Visualizer (#9), Presentation (#20)
- **Spring physics**: Spring (#7), Presentation (#20)
- **State save/restore**: Save-Restore (#8)
- **Loops**: Presentation (#20)
- **Transitions**: Scene Transitions (#19), Presentation (#20)
- **Interpolation**: Linear (#3), Color (#5), Vector (#6), all tweening examples

### Components & Elements
- **Circle**: Quickstart (#1), all tweening examples, Node-Signal (#10), Positioning (#13)
- **Rectangle (Rect)**: Layout (#11), Layout-Group (#12), Transitions (#19), Presentation (#20)
- **Text (Txt)**: Node-Signal (#10), Transitions (#19), Presentation (#20)
- **LaTeX/Math**: TeX (#2)
- **Code**: Code Block (#14)
- **Image**: Media-Image (#15)
- **Video**: Media-Video (#16)
- **Grid**: Tweening-Visualizer (#9), Positioning (#13)
- **Layout**: Layout (#11), Layout-Group (#12), Transitions (#19)
- **Line**: Node-Signal (#10), Positioning (#13)
- **Node**: Layout-Group (#12), Positioning (#13), Custom Components (#17)
- **Custom components**: Custom Components (#17)

### Reactive Programming
- **Signals**: Node-Signal (#10), Tweening-Visualizer (#9), Positioning (#13), Custom Components (#17)
- **Computed signals**: Node-Signal (#10)
- **Signal dependencies**: Node-Signal (#10), Tweening-Visualizer (#9)
- **createSignal**: Node-Signal (#10), Tweening-Visualizer (#9), Positioning (#13), Presentation (#20)
- **Reactive properties**: Node-Signal (#10), all signal-based examples

### Layout & Positioning
- **Flexbox**: Layout (#11)
- **Layout component**: Layout (#11), Layout-Group (#12), Transitions (#19)
- **Grow factor**: Layout (#11)
- **Gap/padding/margin**: Layout (#11)
- **Direction (row/column)**: Layout (#11), Layout-Group (#12)
- **Position**: Quickstart (#1), Positioning (#13), all examples
- **Rotation**: Positioning (#13), Media-Image (#15), Presentation (#20)
- **Scale**: Positioning (#13), Media-Image (#15), Media-Video (#16), Save-Restore (#8)
- **Transform**: Positioning (#13), all examples
- **Coordinate system**: Positioning (#13), Tweening-Visualizer (#9)
- **Grouping**: Layout-Group (#12)

### Visual Effects
- **Opacity**: TeX (#2), Layout-Group (#12)
- **Color**: Quickstart (#1), Color (#5), Presentation (#20)
- **Fill**: Quickstart (#1), all shape examples
- **Stroke**: Layout (#11), Positioning (#13)
- **Blend modes**: Presentation (#20)
- **Caching**: Node-Signal (#10), Positioning (#13), Presentation (#20)
- **Smooth corners**: Presentation (#20)

### Advanced Features
- **Decorators**: Custom Components (#17) - @signal, @initial, @colorSignal
- **Class extension**: Custom Components (#17)
- **Generator methods**: Custom Components (#17), all scenes
- **TypeScript**: All examples
- **JSX**: All examples
- **Logging**: Logging (#18)
- **Debugging**: Logging (#18)
- **Performance profiling**: Logging (#18)
- **Presentation mode**: Presentation (#20)
- **Slides**: Presentation (#20)
- **Task cancellation**: Presentation (#20)

### Color & Styling
- **Color class**: Color (#5), Presentation (#20)
- **Color.lerp**: Color (#5), Custom Components (#17)
- **Color spaces (LAB)**: Presentation (#20)
- **Hex colors**: All examples
- **Font**: Transitions (#19), Presentation (#20)
- **Radius**: Layout (#11), Presentation (#20)

### Media & Assets
- **SVG**: Media-Image (#15)
- **MP4**: Media-Video (#16)
- **Asset imports**: Media-Image (#15), Media-Video (#16)
- **Video playback**: Media-Video (#16)
- **Image display**: Media-Image (#15)

### Code Features
- **Syntax highlighting**: Code Block (#14)
- **Code editing**: Code Block (#14)
- **Code animation**: Code Block (#14)
- **insert/edit/remove**: Code Block (#14)
- **Line selection**: Code Block (#14)

### Mathematical & Scientific
- **LaTeX**: TeX (#2)
- **Mathematical formulas**: TeX (#2)
- **Area calculation**: Node-Signal (#10)
- **Vector math**: Vector (#6), Node-Signal (#10), Positioning (#13)
- **Arc interpolation**: Vector (#6)

### Control Flow
- **Generator functions**: All scenes
- **yield**: All scenes
- **yield***: All scenes
- **waitFor**: TeX (#2), all examples
- **all()**: Quickstart (#1), many examples
- **loop()**: Presentation (#20)
- **cancel()**: Presentation (#20)
- **beginSlide()**: Presentation (#20)

### Project Structure
- **makeProject**: All project entries
- **makeScene2D**: All scenes
- **Multiple scenes**: Scene Transitions (#19)
- **Scene lifecycle**: All examples
- **References (createRef)**: All examples

### Common Use Cases
- **Getting started**: Quickstart (#1)
- **Math content**: TeX (#2)
- **Code tutorials**: Code Block (#14)
- **Image gallery**: Media-Image (#15)
- **Video editing**: Media-Video (#16)
- **UI components**: Custom Components (#17)
- **Presentations**: Presentation (#20)
- **Data visualization**: Node-Signal (#10), Tweening-Visualizer (#9)
- **Educational content**: All examples
- **Motion graphics**: All tweening examples

---

## Summary

This examples package demonstrates all major features of Re.Video through 20 comprehensive, well-documented examples. Each example is self-contained, well-commented, and demonstrates specific patterns and techniques that can be adapted for your own projects.

The examples are organized by complexity and feature set, making it easy to learn Re.Video step-by-step while building progressively more sophisticated animations.
