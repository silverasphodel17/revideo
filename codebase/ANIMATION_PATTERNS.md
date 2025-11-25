# REVIDEO ANIMATION PATTERNS AND TECHNIQUES

## Flow Control Patterns

### all() - Parallel Execution

Runs multiple tasks concurrently and waits for all to finish.

```typescript
// packages/core/src/flow/all.ts
export function* all(...tasks: ThreadGenerator[]): ThreadGenerator {
  for (const task of tasks) {
    yield task; // Start all tasks concurrently
  }
  yield* join(...tasks); // Wait for all to complete
}
```

#### Usage Examples

```typescript
// packages/examples/src/scenes/quickstart.tsx
yield* all(
  myCircle().position.x(300, 1).to(-300, 1),
  myCircle().fill('#e6a700', 1).to('#e13238', 1),
);
// Both animations run simultaneously for 1 second

// Multiple properties in parallel
yield* all(
  backdrop().fill(GREEN, 0.6, easeInOutCubic, Color.createLerp('lab')),
  backdrop().size.x('60%', 0.6),
  title().text('CONTENT', 0.6),
);
// All complete after 0.6 seconds

// Combining with delays
yield* all(
  rect.opacity(1, 3),
  delay(1, rect.fill('#ff0000', 2)), // Starts after 1s delay
);
```

### chain() - Sequential Execution

Runs tasks one after another.

```typescript
// packages/core/src/flow/chain.ts
export function* chain(
  ...tasks: (ThreadGenerator | Callback)[]
): ThreadGenerator {
  for (const generator of tasks) {
    if ('next' in generator) {
      yield* generator;
    } else {
      generator();
    }
  }
}
```

#### Usage Examples

```typescript
// Sequential animations
yield* chain(
  rect.fill('#ff0000', 2),  // 0s to 2s
  rect.opacity(1, 1),        // 2s to 3s
);
// Total time: 3s

// Nested within all()
yield* all(
  rect.radius(20, 3),
  chain(
    rect.fill('#ff0000', 2),
    rect.opacity(1, 1),
  ),
);
```

### loop() - Iteration Patterns

Runs generators in a loop with support for infinite and finite iterations.

```typescript
// packages/core/src/flow/loop.ts
export function* loop(
  iterations: LoopCallback | number,
  factory?: LoopCallback,
): ThreadGenerator {
  if (typeof iterations !== 'number') {
    factory = iterations;
    iterations = Infinity;
  }

  for (let i = 0; i < iterations; i++) {
    const generator = factory!(i);
    if (generator) {
      yield* generator;
    } else {
      yield;
    }
  }
}
```

#### Usage Examples

```typescript
// packages/examples/src/scenes/presentation.tsx
// Infinite rotation loop
const loopTask = yield loop(Infinity, () =>
  rotation(-5, 1).to(5, 1)
);
// Later: cancel(loopTask);

// Finite iterations with array
const colors = ['#ff6470', '#ffc66d', '#68abdf', '#99c47a'];
yield* loop(
  colors.length,
  i => rect.fill(colors[i], 2),
);

// Simple continuous rotation
yield loop(
  () => rect.rotation(0).rotation(360, 2, linear),
);
```

### sequence() - Staggered Animations

Starts tasks with a constant delay between them.

```typescript
// packages/core/src/flow/sequence.ts
export function* sequence(
  delay: number,
  ...tasks: ThreadGenerator[]
): ThreadGenerator {
  for (const task of tasks) {
    yield task;
    yield* waitFor(delay);
  }
  yield* join(...tasks);
}
```

#### Usage Examples

```typescript
// Stagger animations by 0.1 seconds
yield* sequence(
  0.1,
  ...rects.map(rect => rect.x(100, 1))
);

// Create wave effect
const circles = createRefArray<Circle>();
yield* sequence(
  0.05,
  ...circles.map(circle => circle.scale(1.2, 0.3).to(1, 0.3))
);
```

### waitFor() - Timing Control

Waits for a specified duration.

```typescript
// packages/core/src/flow/scheduling.ts
export function* waitFor(
  seconds: number,
  after?: ThreadGenerator,
): ThreadGenerator {
  const thread = useThread();
  const step = usePlayback().framesToSeconds(1);
  const targetTime = thread.time() + seconds;

  while (targetTime - step > thread.fixed) {
    yield;
  }
  thread.time(targetTime);

  if (after) {
    yield* after;
  }
}
```

#### Usage Examples

```typescript
// Simple wait
yield* waitFor(2);

// Wait with callback
yield* waitFor(1, rect.fill('#ff0000', 2));

// Creating pauses between animations
yield* rect.x(100, 1);
yield* waitFor(0.5);
yield* rect.y(100, 1);
```

### delay() - Adding Delays

Runs a task after a delay.

```typescript
// packages/core/src/flow/delay.ts
export function* delay(
  time: number,
  task: ThreadGenerator | Callback,
): ThreadGenerator {
  yield* waitFor(time);
  if ('next' in task) {
    yield* task;
  } else {
    task();
  }
}
```

#### Usage Examples

```typescript
// Delay single animation
yield* delay(1, rect.fill('#ff0000', 2));

// Multiple delays in parallel
yield* all(
  rect.opacity(1, 3),
  delay(1, rect.fill('#ff0000', 2)),
  delay(2, rect.scale(2, 1)),
);
```

### run() - Spawning Tasks

Wraps generator functions for use with flow control.

```typescript
// packages/core/src/flow/run.ts
export function run(
  runner: () => ThreadGenerator
): ThreadGenerator;
export function run(
  name: string,
  runner: () => ThreadGenerator
): ThreadGenerator;
```

#### Usage Examples

```typescript
// Anonymous task
yield* all(
  run(function* () {
    yield* rect.fill('#ff0000', 1);
    yield* rect.opacity(0, 1);
  }),
  circle.scale(2, 2),
);

// Named task for debugging
yield* run('fadeOut', function* () {
  yield* all(
    rect.opacity(0, 1),
    rect.y(100, 1),
  );
});
```

### any() - Race Conditions

Run all tasks concurrently and wait for any of them to finish.

```typescript
// packages/core/src/flow/any.ts
export function* any(...tasks: ThreadGenerator[]): ThreadGenerator {
  for (const task of tasks) {
    yield task;
  }
  yield* join(false, ...tasks);
}
```

#### Usage Examples

```typescript
// Stops when first animation completes
yield* any(
  rect.fill('#ff0000', 2),  // 2 seconds
  rect.opacity(1, 1),        // 1 second - wins
);
// Total time: 1s
```

## Signal Animation Syntax

### Basic Signal Tweening

```typescript
// packages/core/src/signals/createSignal.ts
const signal = createSignal(0);

// Animate to value over time
yield* signal(100, 2);  // 0 to 100 over 2 seconds

// Set immediately (no yield)
signal(50);  // Instant

// Get current value
const value = signal();  // Returns current value
```

### Signal Chaining

Signals support fluent API for chaining animations.

```typescript
// packages/core/src/signals/SignalContext.ts
// Chain animations with .to()
yield* myCircle().position.x(300, 1).to(-300, 1);

// Complex chain with various methods
yield* radius(4, 2)
  .to(3, 2)      // Animate to 3
  .wait(1)       // Pause for 1 second
  .to(5, 1)      // Animate to 5
  .back(1)       // Return to initial value
  .do(() => console.log('Done!'));  // Execute callback

// Available chain methods:
// .to(value, duration, timingFunction?, interpolationFunction?)
// .back(time, timingFunction?, interpolationFunction?)
// .wait(duration)
// .run(generator)
// .do(callback)
```

### Compound Signals

Multi-component signals with individual access.

```typescript
// packages/2d/src/lib/components/Node.ts
// Vector2 compound signal
yield* node.position([100, 50], 0.8);  // Animate both x and y

// Individual component animation
yield* all(
  node.position.x(100, 1),
  node.position.y(50, 1.5),
);

// Spacing signals (padding, margin)
yield* rect.padding(20, 1);  // All sides
yield* rect.padding.top(10, 0.5);  // Individual side
yield* rect.padding.horizontal(15, 0.5);  // Left and right
yield* rect.padding.vertical(10, 0.5);  // Top and bottom
```

### Custom Interpolation

```typescript
// packages/core/src/tweening/interpolationFunctions.ts
// Color interpolation in different spaces
yield* backdrop().fill(
  GREEN,
  0.6,
  easeInOutCubic,
  Color.createLerp('lab')  // LAB color space
);

// Available color spaces:
Color.createLerp('rgb')   // RGB linear
Color.createLerp('hsl')   // HSL interpolation
Color.createLerp('lab')   // Perceptually uniform
Color.createLerp('lch')   // Cylindrical LAB
Color.createLerp('xyz')   // XYZ color space

// Custom interpolation function
const customLerp = (from: Vector2, to: Vector2, value: number) => {
  // Custom logic
  return new Vector2(
    from.x + (to.x - from.x) * value * value,  // Quadratic X
    from.y + (to.y - from.y) * value,          // Linear Y
  );
};

yield* signal(newValue, 2, linear, customLerp);
```

### Deep Interpolation

```typescript
// packages/core/src/tweening/interpolationFunctions.ts
export function deepLerp<T>(
  from: T,
  to: T,
  value: number,
): T {
  // Handles: numbers, strings, Colors, Vector2, objects, arrays
}

// Text interpolation
yield* tween(2, value => {
  textNode().text(deepLerp('', 'Hello World', value));
});
// Animates: '' -> 'H' -> 'He' -> 'Hel' -> ...

// Object interpolation
const start = {x: 0, y: 0, scale: 1};
const end = {x: 100, y: 50, scale: 2};
yield* tween(1, value => {
  const state = deepLerp(start, end, value);
  node.position([state.x, state.y]);
  node.scale(state.scale);
});
```

## Tweening and Timing Functions

### Easing Functions

All easing functions in `packages/core/src/tweening/timingFunctions.ts`:

#### Basic Easings

```typescript
// Linear (no easing)
linear(t: number): number

// Sine wave easing
easeInSine(t: number): number     // Slow start
easeOutSine(t: number): number    // Slow end
easeInOutSine(t: number): number  // Slow at both ends

// Power easings (2-5)
easeInQuad, easeOutQuad, easeInOutQuad      // Power of 2
easeInCubic, easeOutCubic, easeInOutCubic   // Power of 3
easeInQuart, easeOutQuart, easeInOutQuart   // Power of 4
easeInQuint, easeOutQuint, easeInOutQuint   // Power of 5

// Exponential
easeInExpo, easeOutExpo, easeInOutExpo

// Circular
easeInCirc, easeOutCirc, easeInOutCirc
```

#### Overshoot Easings

```typescript
// Back easing with overshoot
easeInBack, easeOutBack, easeInOutBack

// Custom overshoot
const customBack = createEaseOutBack(2.5);  // More overshoot
yield* rect.x(100, 1, customBack);

// Example from packages/examples/src/scenes/tweening-back.tsx
yield* all(
  ...circles.map((circle, i) =>
    circle().position.y(240, 2, createEaseOutBack(i))
  ),
);
```

#### Elastic Easings

```typescript
// Elastic spring effect
easeInElastic, easeOutElastic, easeInOutElastic

// Custom elasticity
const customElastic = createEaseOutElastic(3);
yield* rect.scale(1, 1, customElastic);
```

#### Bounce Easings

```typescript
// Bounce effect
easeInBounce, easeOutBounce, easeInOutBounce

// Custom bounce parameters
const customBounce = createEaseOutBounce(8, 3);
yield* rect.y(0, 2, customBounce);
```

### Tween Generator

```typescript
// packages/core/src/tweening/tween.ts
export function* tween(
  seconds: number,
  onProgress: (value: number, time: number) => void,
  onEnd?: (value: number, time: number) => void,
): ThreadGenerator {
  const thread = useThread();
  const startTime = thread.time();
  const endTime = startTime + seconds;

  onProgress(0, 0);
  while (endTime > thread.fixed) {
    const time = thread.fixed - startTime;
    const value = time / seconds;
    if (time > 0) {
      onProgress(value, time);
    }
    yield;
  }
  thread.time(endTime);
  onProgress(1, seconds);
  onEnd?.(1, seconds);
}
```

#### Usage Examples

```typescript
// packages/examples/src/scenes/tweening.tsx
// Manual position animation
yield* tween(2, value => {
  circle().position.x(map(-300, 300, easeInOutCubic(value)));
});

// Color transition
yield* tween(2, value => {
  circle().fill(
    Color.lerp(
      new Color('#e13238'),
      new Color('#e6a700'),
      easeInOutCubic(value),
    ),
  );
});

// Multiple properties
yield* tween(1.5, value => {
  const easedValue = easeInOutQuad(value);
  rect().position.x(map(-200, 200, easedValue));
  rect().rotation(map(0, 360, easedValue));
  rect().scale(map(0.5, 2, easedValue));
});
```

### Interpolation Helpers

```typescript
// packages/core/src/tweening/interpolationFunctions.ts
// Linear interpolation
map(from: number, to: number, value: number): number
// Example: map(0, 100, 0.5) = 50

// Remap between ranges
remap(
  fromIn: number,
  toIn: number,
  fromOut: number,
  toOut: number,
  value: number
): number
// Example: remap(0, 1, -300, 300, 0.5) = 0

// Clamping
clamp(min: number, max: number, value: number): number
clampRemap(...): number  // Remap with clamping
```

## Spring Animations

### Spring Physics

```typescript
// packages/core/src/tweening/spring.ts
export interface Spring {
  mass: number;           // Object mass
  stiffness: number;      // Spring constant (k)
  damping: number;        // Damping coefficient (c)
  initialVelocity?: number;
}

// Physics: F = -k*x - c*v
// Critical damping: c = 2 * sqrt(k * m)
```

### Spring Presets

```typescript
// packages/examples/src/scenes/tweening-spring.tsx
// Pre-configured springs
export const PlopSpring = makeSpring(0.2, 20.0, 0.68, 0.0);
export const BeatSpring = makeSpring(0.13, 5.7, 1.2, 10.0);
export const BounceSpring = makeSpring(0.08, 4.75, 0.05, 0.0);
export const SwingSpring = makeSpring(0.39, 19.85, 2.82, 0.0);
export const JumpSpring = makeSpring(0.04, 10.0, 0.7, 8.0);
export const StrikeSpring = makeSpring(0.03, 20.0, 0.9, 4.8);
export const SmoothSpring = makeSpring(0.16, 15.35, 1.88, 0.0);

// Critical damping example
export const CriticalSpring = {
  mass: 0.1,
  stiffness: 20,
  damping: 2 * Math.sqrt(20 * 0.1),  // ≈ 2.83
};
```

### Spring Usage

```typescript
// packages/examples/src/scenes/tweening-spring.tsx
import {spring, PlopSpring, SmoothSpring} from '@revideo/core';

// Basic spring animation
yield* spring(
  PlopSpring,
  -400,  // from
  400,   // to
  1,     // settlement tolerance
  value => {
    circle().position.x(value);
  }
);

// Custom spring
yield* spring(
  {mass: 0.1, stiffness: 30, damping: 2},
  0,
  100,
  0.01,  // Lower tolerance = more accurate
  value => rect().y(value)
);

// Chained springs
yield* chain(
  spring(PlopSpring, -400, 400, 1, v => circle().x(v)),
  spring(SmoothSpring, 400, -400, 1, v => circle().x(v)),
);
```

## Common Animation Recipes

### Text Animations

```typescript
// Character-by-character reveal
yield* textNode().text('', 0).text('Hello World', 2);

// Using deepLerp for typing effect
yield* tween(2, value => {
  textNode().text(deepLerp('', 'Target Text', value));
});

// Text morphing
const texts = ['First', 'Second', 'Third'];
for (const text of texts) {
  yield* textNode().text(text, 1);
  yield* waitFor(0.5);
}

// Code editing animation (packages/examples/src/scenes/code.tsx)
yield* codeBlock().edit(`
  function hello() {
    console.log('Hello');
  }
`, 1);

yield* codeBlock().selection(lines(1, 2), 0.5);  // Highlight lines
```

### Stagger Animations

```typescript
// Using sequence()
yield* sequence(
  0.1,
  ...rects.map(rect => rect.opacity(1, 0.5))
);

// Manual stagger with delay()
yield* all(
  ...rects.map((rect, i) =>
    delay(i * 0.1, rect.scale(1.2, 0.3).to(1, 0.3))
  )
);

// Wave effect
const items = createRefArray<Rect>();
yield* all(
  ...items.map((item, i) =>
    delay(
      i * 0.05,
      chain(
        item().y(-20, 0.3, easeOutCubic),
        item().y(0, 0.3, easeInCubic),
      )
    )
  )
);
```

### Transform Animations

```typescript
// Basic transforms
yield* all(
  rect().position([100, 50], 1),
  rect().rotation(45, 1),
  rect().scale(1.5, 1),
);

// Skew and complex transforms
yield* all(
  rect().skew([0.2, 0], 1),
  rect().position.x(100, 1),
);

// Orbit animation
yield* tween(2, value => {
  const angle = value * Math.PI * 2;
  circle().position([
    Math.cos(angle) * 200,
    Math.sin(angle) * 200,
  ]);
});
```

### Shape Morphing

```typescript
// Morph rectangle corners
yield* rect().radius(0, 0).radius(50, 1);

// Circle to ellipse
yield* circle().size([200, 200], 0).size([300, 150], 1);

// Path morphing with Line
const line = createRef<Line>();
yield* line().points(
  [[-100, 0], [0, -100], [100, 0]],
  1
);
yield* line().points(
  [[-100, 100], [0, 0], [100, 100]],
  1
);
```

### Path Drawing

```typescript
// Line drawing animation
yield* line().end(0, 0).end(1, 2);

// Draw and erase
yield* chain(
  line().end(1, 1),      // Draw
  waitFor(0.5),
  line().start(1, 1),    // Erase from start
);

// Bezier curve animation
const bezier = createRef<QuadBezier>();
yield* bezier().end(0, 0).end(1, 1);
yield* bezier().start(0.5, 1);  // Erase from middle
```

### Color Animations

```typescript
// Simple color change
yield* rect().fill('#ff0000', 1);

// Gradient animation
yield* rect().fill(
  new Gradient({
    type: 'linear',
    from: [0, -100],
    to: [0, 100],
    stops: [
      {offset: 0, color: '#ff0000'},
      {offset: 1, color: '#0000ff'},
    ],
  }),
  2
);

// Color in different spaces
yield* all(
  rect1().fill('#00ff00', 1, linear, Color.createLerp('rgb')),
  rect2().fill('#00ff00', 1, linear, Color.createLerp('hsl')),
  rect3().fill('#00ff00', 1, linear, Color.createLerp('lab')),
);
```

### Particle Effects

```typescript
// Ripple effect
yield* shape().ripple(1);

// Particle system
const particles = createRefArray<Circle>();
yield* all(
  ...particles.map((p, i) =>
    all(
      p().position(
        Vector2.fromRadians(i * Math.PI / 4).scale(200),
        1,
        easeOutCubic
      ),
      p().opacity(0, 1),
    )
  )
);

// Explosion effect
function* explode(node: Node) {
  const particles = [];
  for (let i = 0; i < 20; i++) {
    const angle = (i / 20) * Math.PI * 2;
    const particle = (
      <Circle
        size={10}
        fill={'#ff0000'}
        position={node.position()}
      />
    );
    node.parent().add(particle);
    particles.push(particle);
  }

  yield* all(
    ...particles.map((p, i) =>
      all(
        p.position(
          Vector2.fromRadians((i / 20) * Math.PI * 2).scale(200),
          0.5,
          easeOutQuad
        ),
        p.opacity(0, 0.5),
      )
    )
  );

  particles.forEach(p => p.remove());
}
```

### Layout Animations

```typescript
// packages/examples/src/scenes/layout.tsx
// Flex grow animations
yield* all(
  colA().grow(2, 0.8),
  colB().grow(1, 0.8),
);

// Gap animation
yield* layout().gap(50, 1);

// Direction change
yield* layout().direction('column', 0.5);

// Justify and align
yield* all(
  layout().justifyContent('space-between', 0.5),
  layout().alignItems('center', 0.5),
);
```

### Loop Animations

```typescript
// Continuous rotation
const rotationTask = yield loop(
  () => rect().rotation(360, 2, linear)
);

// Breathing effect
yield loop(3, () =>
  circle().scale(1.2, 1, easeInOutSine).to(1, 1, easeInOutSine)
);

// Color cycle
const colors = ['#ff6470', '#ffc66d', '#68abdf', '#99c47a'];
yield loop(Infinity, i =>
  rect().fill(colors[i % colors.length], 1)
);

// Cancel loop
cancel(rotationTask);
```

## Advanced Animation Patterns

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

### Procedural Animations

```typescript
// Grid animation
const rows = 5;
const cols = 5;
const cells = [];

for (let r = 0; r < rows; r++) {
  for (let c = 0; c < cols; c++) {
    const cell = (
      <Rect
        size={50}
        position={[(c - 2) * 60, (r - 2) * 60]}
        fill={'#333'}
      />
    );
    view.add(cell);
    cells.push({cell, r, c});
  }
}

// Diagonal wave
yield* all(
  ...cells.map(({cell, r, c}) =>
    delay(
      (r + c) * 0.05,
      cell.fill('#88C0D0', 0.3).to('#333', 0.3)
    )
  )
);
```

### Camera Effects

```typescript
// Zoom effect
function* zoomTo(target: Node, scale = 2, duration = 1) {
  const view = target.view();
  const targetPos = target.absolutePosition();

  yield* all(
    view.position(targetPos.scale(-1), duration, easeInOutQuad),
    view.scale(scale, duration, easeInOutQuad),
  );
}

// Shake effect
function* shake(node: Node, intensity = 10, duration = 0.5) {
  const originalPos = node.position();
  const samples = Math.floor(duration * 60);  // 60fps

  yield* chain(
    ...Array.from({length: samples}).map(() =>
      node.position(
        originalPos.add([
          (Math.random() - 0.5) * intensity * 2,
          (Math.random() - 0.5) * intensity * 2,
        ]),
        1/60
      )
    )
  );

  yield* node.position(originalPos, 0.1);
}
```

### State-Based Animations

```typescript
// Animation state machine
class AnimationState {
  private currentState = createSignal('idle');

  *transition(to: string, animation: ThreadGenerator) {
    if (this.currentState() !== to) {
      this.currentState(to);
      yield* animation;
    }
  }
}

const state = new AnimationState();

// Usage
yield* state.transition('active',
  all(
    rect().fill('#00ff00', 0.5),
    rect().scale(1.2, 0.5),
  )
);
```

### Dynamic Signal Dependencies

```typescript
// packages/examples/src/scenes/node-signal.tsx
const radius = createSignal(3);
const area = createSignal(() => Math.PI * radius() * radius());

view.add(
  <>
    <Circle
      size={() => radius() * 100}
      fill={'#e13238'}
    />
    <Txt
      text={() => `Area: ${area().toFixed(2)}`}
      y={100}
    />
  </>
);

yield* radius(5, 2);  // Area updates automatically
```

### Complex Timing Patterns

```typescript
// Overlap animations
function* overlap(tasks: ThreadGenerator[], overlap = 0.5) {
  for (let i = 0; i < tasks.length; i++) {
    if (i === 0) {
      yield tasks[i];
    } else {
      yield* all(
        delay(overlap, tasks[i]),
        waitFor(overlap),
      );
    }
  }
}

// Usage
yield* overlap([
  rect1().x(100, 1),
  rect2().x(100, 1),
  rect3().x(100, 1),
], 0.7);  // 70% overlap
```

## Performance Optimization

### Signal Caching

```typescript
// Cache expensive computations
const expensiveSignal = createComputed(() => {
  // Complex calculation
  return heavyComputation();
});

// Use cache on nodes
<Rect
  ref={rect}
  cache  // Enable rendering cache
  cachePadding={20}  // Padding for effects
>
  {/* Complex children */}
</Rect>
```

### Batch Updates

```typescript
// Batch property updates
yield* all(
  ...nodes.map(node => node.opacity(1, 0))  // Instant
);

// Then animate together
yield* all(
  ...nodes.map(node => node.scale(1, 1))
);
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

## Real Examples from Codebase

### Complete Animation Scene

```typescript
// packages/examples/src/scenes/quickstart.tsx
import {Circle, makeScene2D} from '@revideo/2d';
import {all, createRef} from '@revideo/core';

export default makeScene2D(function* (view) {
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

### Complex Presentation

```typescript
// packages/examples/src/scenes/presentation.tsx
const backdrop = createRef<Rect>();
const title = createRef<Txt>();
const rotation = createSignal(0);
const rotationScale = createSignal(0);

view.add(
  <Rect
    ref={backdrop}
    width={'50%'}
    height={'50%'}
    fill={RED}
    radius={40}
    rotation={() => rotation() * rotationScale()}
  >
    <Txt
      ref={title}
      rotation={() => -rotation() * rotationScale()}
      text={'START'}
      fontSize={48}
      fontWeight={700}
      fill={'white'}
    />
  </Rect>,
);

// Start rotation loop
const loopTask = yield loop(Infinity, () =>
  rotation(-5, 1).to(5, 1)
);

// Transition animation
yield* all(
  backdrop().fill(GREEN, 0.6, easeInOutCubic, Color.createLerp('lab')),
  backdrop().size.x('60%', 0.6),
  title().text('CONTENT', 0.6),
  rotationScale(1, 0.6),
);

// Stop rotation
cancel(loopTask);
yield* rotationScale(0, 0.6);
```

### Spring Animation Example

```typescript
// packages/examples/src/scenes/tweening-spring.tsx
const circle = createRef<Circle>();

view.add(
  <Circle
    ref={circle}
    x={-400}
    size={240}
    fill={'#e13238'}
  />,
);

// Bouncy spring
yield* spring(PlopSpring, -400, 400, 1, value => {
  circle().position.x(value);
});

yield* waitFor(0.5);

// Smooth spring back
yield* spring(SmoothSpring, 400, -400, 1, value => {
  circle().position.x(value);
});
```

## Summary

Revideo's animation system provides:

1. **Flow Control**: Comprehensive patterns for parallel, sequential, and complex timing
2. **Signal System**: Reactive properties with chainable animations
3. **Timing Functions**: 30+ easing functions including elastic, bounce, and spring
4. **Spring Physics**: Realistic spring animations with presets
5. **Common Patterns**: Ready-to-use recipes for text, transforms, morphing, and effects
6. **Performance**: Optimized with caching, batching, and conditional execution

All animations are composable using generators, making complex sequences readable and maintainable.