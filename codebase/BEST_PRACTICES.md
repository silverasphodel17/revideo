# REVIDEO BEST PRACTICES AND PATTERNS

## Project Structure

### Standard Project Layout

```
project/
├── src/
│   ├── project.ts           # Main project configuration
│   ├── render.ts            # Rendering script with variables
│   ├── scenes/
│   │   ├── scene-1.tsx      # Scene generator functions
│   │   └── scene-2.tsx
│   ├── components/          # Reusable custom components
│   │   └── CustomNode.tsx
│   └── global.css           # Global styles
├── package.json
├── tsconfig.json
└── vite.config.ts          # Vite configuration
```

### Project Configuration Pattern

```typescript
// src/project.ts
import {makeProject} from '@revideo/core';
import example from './scenes/example';

export default makeProject({
  name: 'project',
  scenes: [example],
  variables: {
    fill: 'green',  // Can be overridden at render time
  },
});
```

### File Naming Conventions

- Scene files: `kebab-case.tsx` (e.g., `quickstart.tsx`, `data-viz.tsx`)
- Component files: `PascalCase.tsx` (e.g., `Switch.tsx`, `Card.tsx`)
- Project files: `kebab-case.ts`
- Maintain parallel structure: `project.ts` → `scenes/project.tsx`

## Signal Usage Patterns

### Creating and Using Signals

```typescript
// packages/core/src/signals/createSignal.ts
import {createSignal} from '@revideo/core';

// Simple value signal
const opacity = createSignal(1);
opacity();           // Read value: 1
opacity(0.5);        // Set value instantly
yield* opacity(0, 1); // Animate to 0 over 1 second

// Computed signal (depends on other signals)
const radius = createSignal(3);
const area = createSignal(() => Math.PI * radius() * radius());

// Computed signals update automatically
yield* radius(5, 2);  // area() automatically recalculates
```

### Signal Best Practices

#### Use Signals for Reactive Properties

```typescript
// Good: Signal values update everywhere they're referenced
const scale = createSignal(1);

view.add(
  <>
    <Circle size={() => scale() * 100} />
    <Rect width={() => scale() * 200} />
  </>
);

yield* scale(2, 1); // Both elements scale together
```

#### Create Computed Signals for Derived Values

```typescript
// Good: Computed signal maintains relationship
const diameter = createSignal(() => radius() * 2);

// Avoid: Manual updates get out of sync
const diameter = createSignal(radius() * 2);
radius(5); // diameter is now stale!
```

#### Signal Decorators in Components

```typescript
// packages/2d/src/lib/components/Node.ts
export class CustomNode extends Node {
  @initial(false)
  @signal()
  public declare readonly active: SimpleSignal<boolean, this>;

  @initial('#88C0D0')
  @colorSignal()
  public declare readonly accent: ColorSignal<this>;
}
```

#### Signal Animation Chaining

```typescript
// Chain animations for complex sequences
yield* myCircle().position.x(300, 1)
  .to(-300, 1)        // Back animation
  .wait(0.5)          // Pause
  .to(0, 0.5)         // Return to center
  .do(() => console.log('Done!'));
```

## Ref Management Best Practices

### Creating and Using Refs

```typescript
// packages/examples/src/scenes/quickstart.tsx
import {createRef, createRefArray} from '@revideo/core';

// Single element ref
const myCircle = createRef<Circle>();
view.add(<Circle ref={myCircle} />);
myCircle().opacity(0); // Access the element

// Multiple element refs - illustrative example
// Note: This pattern is shown for educational purposes.
// For actual usage example, see packages/2d/src/lib/components/__tests__/query.test.tsx
const rects = createRefArray<Rect>();
view.add(
  <>
    <Rect ref={rects} />
    <Rect ref={rects} />
    <Rect ref={rects} />
  </>
);

// Animate all rects
yield* all(
  ...rects.map(rect => rect().opacity(1, 0.5))
);
```

### Ref Best Practices

#### Type Refs Explicitly

```typescript
// Good
const circle = createRef<Circle>();

// Avoid
const circle = createRef<any>();
```

#### Access Refs Safely

```typescript
// Good: Always call as function
myCircle().opacity(1);

// Avoid: Refs are functions, not values
myCircle.opacity(1); // Type error
```

#### Ref Lifecycle Management

```typescript
// Refs enable late binding
const myCircle = createRef<Circle>();

// Safe to create animations before element exists
const animation = function* () {
  yield* myCircle().position.x(100, 1);
};

view.add(<Circle ref={myCircle} />);
yield* animation(); // Now safe to run
```

## Performance Optimization

### Caching Strategies

```typescript
// packages/2d/src/lib/components/Node.ts
// Enable caching for complex static content
view.add(
  <Rect
    cache              // Enable rendering cache
    cachePadding={20}  // Padding for effects like blur
    width={'50%'}
    height={'50%'}
  >
    {/* Complex children that don't change often */}
  </Rect>
);
```

### Batch Updates

```typescript
// Good: Update multiple properties together
yield* all(
  ...nodes.map(node => node.opacity(1, 0))  // Instant batch
);

// Then animate
yield* all(
  ...nodes.map(node => node.scale(1, 1))
);
```

### Avoiding Unnecessary Recalculations

```typescript
// Good: Computed signal with dependencies
const area = createSignal(() => Math.PI * radius() * radius());

// Use in reactive context
<Txt text={() => `Area: ${area().toFixed(2)}`} />

// Avoid: Function called every frame
<Txt text={() => calculateExpensiveValue()} />
```

### Memory Management

```typescript
// Clean up removed elements
function* removeAnimation(node: Node) {
  yield* node.opacity(0, 1);
  node.remove();  // Remove from DOM
}

// Cancel long-running loops
const rotationTask = yield loop(Infinity, () =>
  rect().rotation(360, 2, linear)
);

// Later...
cancel(rotationTask);
```

### Conditional Animations

```typescript
// Skip animations based on conditions
const shouldAnimate = createSignal(true);

function* conditionalAnimation() {
  if (shouldAnimate()) {
    yield* rect().fill('#ff0000', 1);
  } else {
    rect().fill('#ff0000');  // Instant
  }
}
```

## Code Organization

### Component Composition Pattern

```typescript
// Simplified from packages/examples/src/components/Switch.tsx
import type {NodeProps} from '@revideo/2d';
import {Circle, Node, Rect, colorSignal, initial, signal} from '@revideo/2d';
import type {SignalValue, SimpleSignal} from '@revideo/core';
import {Color, all, createRef, createSignal, easeInOutCubic, tween} from '@revideo/core';

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
  private readonly indicator = createRef<Circle>();
  private readonly container = createRef<Rect>();

  public constructor(props?: SwitchProps) {
    super({...props});
    this.isOn = this.initialState();
    this.indicatorPosition(this.isOn ? 50 : -50);

    this.add(
      <Rect
        ref={this.container}
        fill={this.isOn ? this.accent() : '#242424'}
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
        const oldColor = this.isOn ? this.accent() : new Color('#242424');
        const newColor = this.isOn ? new Color('#242424') : this.accent();
        this.container().fill(
          Color.lerp(oldColor, newColor, easeInOutCubic(value)),
        );
      }),
      this.indicatorPosition(this.isOn ? -50 : 50, duration, easeInOutCubic),
    );
    this.isOn = !this.isOn;
  }
}
```

### Scene Structure Pattern

```typescript
// packages/examples/src/scenes/quickstart.tsx
export default makeScene2D(function* (view) {
  // 1. Setup phase: Create refs and signals
  const circle = createRef<Circle>();
  const scale = createSignal(1);

  // 2. Element creation phase
  view.add(
    <Circle
      ref={circle}
      size={() => scale() * 100}
      fill="#e13238"
    />,
  );

  // 3. Animation phase
  yield* all(
    circle().position.x(300, 1),
    scale(2, 1),
  );

  // 4. Pause/wait phase
  yield* waitFor(1);

  // 5. Cleanup/transition phase
  yield* circle().opacity(0, 1);
});
```

### Import Organization

```typescript
// 1. Framework imports
import {Circle, makeScene2D, Txt} from '@revideo/2d';
import {all, createRef, createSignal, waitFor} from '@revideo/core';

// 2. Local imports
import {CustomComponent} from '../components/CustomComponent';

// 3. Type imports
import type {NodeProps} from '@revideo/2d';

// 4. Constants
const RED = '#ff6470';
const GREEN = '#99C47A';
```

## Variable System Usage

### Project Variables

```typescript
// src/project.ts
export default makeProject({
  variables: {
    title: 'Default Title',
    count: 42,
    colors: ['#ff0000', '#00ff00'],
  },
});

// Render with override
const file = await renderVideo({
  projectFile: './src/project.ts',
  variables: {
    title: 'Custom Title',
    count: 100,
  },
});
```

### Using Variables in Scenes

```typescript
// packages/create/examples/youtube-shorts/src/project.tsx
import {useScene} from '@revideo/core';

export default makeScene2D(function* (view) {
  const title = useScene().variables.get('title', 'Default')();
  const data = useScene().variables.get('data', [])();

  view.add(
    <Txt text={title} fontSize={48} />
  );

  // Create visualizations from data
  const max = Math.max(...data);
  // ... use data for animations
});
```

## Scene Composition Patterns

### Multi-Scene Projects

```typescript
// src/project.ts
import {makeProject} from '@revideo/core';
import scene1 from './scenes/scene1';
import scene2 from './scenes/scene2';

export default makeProject({
  scenes: [scene1, scene2], // Rendered sequentially
});
```

### Scene Transitions

```typescript
// Fade transition
function* fadeTransition(from: Node, to: Node) {
  to.opacity(0);
  from.parent().add(to);

  yield* all(
    from.opacity(0, 1),
    to.opacity(1, 1),
  );

  from.remove();
}

// Slide transition
function* slideTransition(from: Node, to: Node, direction = 1) {
  to.position.x(600 * direction);
  from.parent().add(to);

  yield* all(
    from.position.x(-600 * direction, 1, easeInCubic),
    to.position.x(0, 1, easeOutCubic),
  );

  from.remove();
}
```

### Slides for Presentations

```typescript
// packages/examples/src/scenes/presentation.tsx
import {beginSlide} from '@revideo/core';

export default makeScene2D(function* (view) {
  const backdrop = createRef<Rect>();

  view.add(
    <Rect ref={backdrop} width={'50%'} height={'50%'} fill={RED}>
      <Txt>START</Txt>
    </Rect>
  );

  yield* beginSlide('intro');
  yield* backdrop().fill(GREEN, 0.6);

  yield* beginSlide('content');
  yield* backdrop().fill(BLUE, 0.6);

  yield* beginSlide('outro');
});
```

## Common Anti-Patterns to Avoid

### Anti-Pattern: Stale Computed Values

```typescript
// AVOID: Manually updated computed value
const diameter = createSignal(radius() * 2);
yield* radius(5, 2);
// diameter is now stale!

// GOOD: Use computed signal
const diameter = createSignal(() => radius() * 2);
yield* radius(5, 2);
// diameter automatically updates
```

### Anti-Pattern: Creating Refs Inside Loops

```typescript
// AVOID: Ref created inside loop loses reference
for (let i = 0; i < 3; i++) {
  const rect = createRef<Rect>();
  view.add(<Rect ref={rect} />);
  // rect goes out of scope!
}

// GOOD: Use createRefArray (illustrative pattern)
// For actual usage example, see packages/2d/src/lib/components/__tests__/query.test.tsx
const rects = createRefArray<Rect>();
view.add(
  <>
    {Array.from({length: 3}).map(() => (
      <Rect ref={rects} />
    ))}
  </>
);
```

### Anti-Pattern: Not Caching Complex Scenes

```typescript
// AVOID: Complex elements without caching
<Rect>
  {/* Many nested children that don't animate */}
</Rect>

// GOOD: Cache static content
<Rect cache cachePadding={20}>
  {/* Complex static structure */}
</Rect>
```

### Anti-Pattern: Memory Leaks

```typescript
// AVOID: Creating elements without cleanup
function* addElements() {
  for (let i = 0; i < 100; i++) {
    view.add(<Circle />);
    // Elements never removed
  }
}

// GOOD: Clean up after animations
const refs = createRefArray<Circle>();
view.add(
  <>
    {Array.from({length: 100}).map(() => (
      <Circle ref={refs} />
    ))}
  </>
);

yield* sequence(0.01, ...refs.map(ref => ref().opacity(0, 0.5)));
refs.forEach(ref => ref().remove());
```

### Anti-Pattern: Infinite Loops Without Control

```typescript
// AVOID: Uncontrolled infinite loop
yield loop(Infinity, () => rect().rotation(360, 2));

// GOOD: Store and cancel loop
const rotationTask = yield loop(Infinity, () =>
  rect().rotation(360, 2)
);
// Later...
cancel(rotationTask);
```

## Flow Control Best Practices

### all() - Parallel Execution

```typescript
// packages/core/src/flow/all.ts
// Run multiple animations simultaneously
yield* all(
  myCircle().position.x(300, 1),
  myCircle().fill('#e6a700', 1),
);
// Both complete after 1 second
```

### chain() - Sequential Execution

```typescript
// packages/core/src/flow/chain.ts
// Run animations one after another
yield* chain(
  rect.fill('#ff0000', 2),  // 0s to 2s
  rect.opacity(1, 1),        // 2s to 3s
);
// Total: 3s
```

### sequence() - Staggered Animations

```typescript
// packages/core/src/flow/sequence.ts
// Start with constant delay between
yield* sequence(
  0.1,  // 100ms delay
  ...rects.map(rect => rect.x(100, 1))
);
```

### loop() - Iteration Patterns

```typescript
// packages/core/src/flow/loop.ts
// Finite iterations
const colors = ['#ff6470', '#ffc66d', '#68abdf', '#99c47a'];
yield* loop(
  colors.length,
  i => rect.fill(colors[i], 2),
);

// Infinite with cancellation
const task = yield loop(Infinity, () =>
  rotation(-5, 1).to(5, 1)
);
cancel(task);
```

### waitFor() and delay()

```typescript
// packages/core/src/flow/scheduling.ts
// Simple wait
yield* waitFor(2);

// Delayed execution
yield* delay(1, rect.fill('#ff0000', 2));

// Multiple delays in parallel
yield* all(
  rect.opacity(1, 3),
  delay(1, rect.fill('#ff0000', 2)),
  delay(2, rect.scale(2, 1)),
);
```

## Animation Techniques

### Tweening with Custom Timing

```typescript
// packages/core/src/tweening/tween.ts
import {tween, map, easeInOutCubic} from '@revideo/core';

// Manual animation control
yield* tween(2, value => {
  circle().position.x(map(-300, 300, easeInOutCubic(value)));
});

// Multiple properties
yield* tween(1.5, value => {
  const eased = easeInOutQuad(value);
  rect().position.x(map(-200, 200, eased));
  rect().rotation(map(0, 360, eased));
  rect().scale(map(0.5, 2, eased));
});
```

### Spring Animations

```typescript
// packages/examples/src/scenes/tweening-spring.tsx
import {spring, PlopSpring, SmoothSpring} from '@revideo/core';

// Pre-configured springs
yield* spring(
  PlopSpring,  // Bouncy preset
  -400,        // from
  400,         // to
  1,           // tolerance
  value => circle().position.x(value)
);

// Custom spring
yield* spring(
  {mass: 0.1, stiffness: 30, damping: 2},
  0, 100, 0.01,
  value => rect().y(value)
);

// Available presets:
// PlopSpring, BeatSpring, BounceSpring, SwingSpring
// JumpSpring, StrikeSpring, SmoothSpring
```

### Color Space Interpolation

```typescript
// packages/core/src/tweening/interpolationFunctions.ts
// Animate in different color spaces
yield* all(
  rect1().fill('#00ff00', 1, linear, Color.createLerp('rgb')),
  rect2().fill('#00ff00', 1, linear, Color.createLerp('hsl')),
  rect3().fill('#00ff00', 1, linear, Color.createLerp('lab')),
);
```

### Text Animations

```typescript
// Character-by-character reveal
yield* textNode().text('', 0).text('Hello World', 2);

// Using deepLerp for typing
yield* tween(2, value => {
  textNode().text(deepLerp('', 'Target Text', value));
});
```

### Stagger Effects

```typescript
// Using sequence
yield* sequence(
  0.1,
  ...rects.map(rect => rect.opacity(1, 0.5))
);

// Manual stagger with delay
yield* all(
  ...rects.map((rect, i) =>
    delay(i * 0.1, rect.scale(1.2, 0.3).to(1, 0.3))
  )
);
```

## Real-World Patterns

### Common Animation Utilities

These are conceptual examples of common animation patterns that can be implemented in your projects:

```typescript
// Example: Data Visualization Pattern
// This shows how you might structure a data-driven animation
export default makeScene2D(function* (view) {
  const data = useScene().variables.get('data', [10, 20, 30])();
  const max = Math.max(...data);

  // Create bars from data
  const bars = createRefArray<Rect>();
  // ... implementation
});

// Example: Grid Animation Pattern
// This demonstrates a procedural grid creation approach
function* createGrid(view, rows: number, cols: number) {
  const cells = [];
  // Create grid cells procedurally
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      // Position cells in a grid
    }
  }
  // Apply wave animations
}

// Example: Camera Utility Functions (conceptual)
// These patterns show how you might implement camera movements
function* zoomTo(target: Node, scale = 2, duration = 1) {
  // Implementation would manipulate view position and scale
}

function* shake(node: Node, intensity = 10, duration = 0.5) {
  // Implementation would apply random position offsets
}
```

**Note:** The above are conceptual patterns. For actual working examples, refer to the scene files in `packages/examples/src/scenes/`.

## Testing and Debugging

### Console Logging

```typescript
import {useLogger} from '@revideo/core';

export default makeScene2D(function* (view) {
  const logger = useLogger();

  logger.log('Scene started');

  const circle = createRef<Circle>();
  view.add(<Circle ref={circle} />);

  logger.log('Circle created');

  yield* circle().position.x(300, 1);

  logger.log('Animation complete');
});
```

### Performance Profiling

```typescript
export default makeScene2D(function* (view) {
  const logger = useLogger();

  const startTime = performance.now();

  // Complex setup
  view.add(/* ... */);

  logger.log(`Setup time: ${performance.now() - startTime}ms`);

  // Animation
  yield* /* ... */;

  logger.log(`Total time: ${performance.now() - startTime}ms`);
});
```

## Summary

### Key Principles

1. **Structure**: Use consistent project layout with clear separation of scenes and components
2. **Signals**: Use signals for reactive state; computed signals for derived values
3. **Refs**: Type explicitly, access through function calls
4. **Animation**: Use `all()` for parallel, `chain()` for sequential, `sequence()` for staggered
5. **Performance**: Enable caching for complex static content; batch updates
6. **Components**: Extend Node, use decorators, implement generator methods
7. **Variables**: Use the variable system for data-driven animations
8. **Cleanup**: Remove elements when done; cancel long-running tasks
9. **Testing**: Use logging and performance profiling
10. **Quality**: Flatten nested animations; use TypeScript; follow conventions

### Quick Reference

| Pattern | Use Case |
|---------|----------|
| `all()` | Run animations in parallel |
| `chain()` | Run animations sequentially |
| `sequence()` | Stagger animations with delay |
| `loop()` | Repeat animations |
| `waitFor()` | Pause execution |
| `delay()` | Delay a single animation |
| `createSignal()` | Create reactive value |
| `createRef()` | Reference single element |
| `createRefArray()` | Reference multiple elements |
| `cache` | Optimize complex static content |

### Files Referenced

#### Actual Example Files
- `packages/template/src/project.ts` - Project configuration template
- `packages/examples/src/scenes/quickstart.tsx` - Basic animation example
- `packages/examples/src/scenes/presentation.tsx` - Slide-based presentation with beginSlide
- `packages/examples/src/scenes/tweening-spring.tsx` - Spring animation examples
- `packages/examples/src/components/Switch.tsx` - Custom component example
- `packages/create/examples/youtube-shorts/src/project.tsx` - Variable usage example
- `packages/2d/src/lib/components/__tests__/query.test.tsx` - createRefArray usage

#### Core Library Files
- `packages/core/src/flow/*.ts` - Flow control functions (all, chain, sequence, loop)
- `packages/core/src/signals/*.ts` - Signal system implementation
- `packages/core/src/tweening/*.ts` - Tweening and interpolation functions
- `packages/core/src/utils/createRefArray.ts` - Ref array implementation
- `packages/2d/src/lib/components/Node.ts` - Base node class