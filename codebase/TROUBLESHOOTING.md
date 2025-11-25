# REVIDEO TROUBLESHOOTING GUIDE

## Common Error Messages

### Thread Reuse Error

**Error Message:**
```
"The generator "{name}" is already being executed by another thread."
```

**Source:** `packages/core/src/threading/Thread.ts` (line 119-121)

**Root Cause:** Attempting to use the same generator instance multiple times.

**Example of Problem:**
```typescript
// WRONG: Reusing generator
const animation = rect().opacity(1, 1);
yield* all(animation, animation); // Error!

// WRONG: Reusing in loop
const myAnimation = function* () {
  yield* rect().x(100, 1);
};
yield* all(myAnimation(), myAnimation()); // Error!
```

**Solution:**
```typescript
// CORRECT: Create separate instances
yield* all(
  rect().opacity(1, 1),
  rect().opacity(1, 1),
);

// CORRECT: Use factory function
const myAnimation = () => function* () {
  yield* rect().x(100, 1);
}();
yield* all(myAnimation(), myAnimation());

// CORRECT: For loops, use the loop() utility
yield loop(2, () => rect().rotation(360, 1));
```

### Circular Dependency Error

**Error Message:**
```
"A circular dependency occurred between signals."
```

**Source:** `packages/core/src/signals/DependencyContext.ts`

**Root Cause:** A computed signal depends on itself directly or indirectly.

**Example of Problem:**
```typescript
// WRONG: Direct circular dependency
const a = createSignal(() => a() + 1); // Error!

// WRONG: Indirect circular dependency
const b = createSignal(() => c() * 2);
const c = createSignal(() => b() + 1); // Error!
```

**Solution:**
```typescript
// CORRECT: Break the cycle with base values
const counter = createSignal(0);
const doubled = createSignal(() => counter() * 2);

// CORRECT: Use separate signals for state and computation
const baseValue = createSignal(0);
const computed = createSignal(() => baseValue() * 2);
```

### Context Not Available

**Error Messages:**
```
"The scene is not available in the current context."
"The thread is not available in the current context."
```

**Note:** The first example "The animation player has not been properly initialized" is an illustrative example error pattern, not an exact error message from the codebase.

**Sources:**
- `packages/core/src/utils/useScene.ts` (line 11)
- `packages/core/src/utils/useThread.ts` (line 13)

**Root Cause:** Trying to access context outside of proper scope.

**Example of Problem:**
```typescript
// WRONG: Called at module level
const scene = useScene(); // Error!

export default makeScene2D(function* (view) {
  // scene usage here
});

// WRONG: Called outside generator
const thread = useThread(); // Error!
```

**Solution:**
```typescript
// CORRECT: Call inside scene generator
export default makeScene2D(function* (view) {
  const scene = useScene(); // OK
  const variables = scene.variables.get('data', []);

  // Inside a spawned generator
  yield* run(function* () {
    const thread = useThread(); // OK
    // Use thread here
  });
});
```

### Context Stack Empty

**Error Message:**
```
"collectStart/collectEnd was called out of order."
```

**Source:** `packages/core/src/signals/DependencyContext.ts` (line 97)

**Note:** This error message indicates that either the context stack is empty when trying to pop, or the wrong context was popped. The implementation uses `.pop()` which returns `undefined` for an empty stack, and then checks if the popped value matches the expected context. This is inferred behavior - the code doesn't explicitly check for an empty stack but catches both cases (empty stack or mismatched context) with a single comparison.

**Root Cause:** Mismatched context push/pop operations, often due to error handling issues in signal computations or circular dependencies.

**Solution:**
```typescript
// Ensure proper error handling in computed signals
const computed = createSignal(() => {
  try {
    return riskyComputation();
  } catch (error) {
    console.error('Computation failed:', error);
    return defaultValue; // Return safe default
  }
});
```

## Build and Dependency Issues

### TypeScript Module Resolution

**Error:**
```
Cannot find module '@revideo/2d' or its corresponding type declarations
```

**Solutions:**

1. **Check package installation:**
```bash
npm install @revideo/2d @revideo/core
```

2. **Update TypeScript config:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "moduleResolution": "node",
    "jsx": "react-jsx",
    "jsxImportSource": "@revideo/2d/lib",
    "paths": {
      "@revideo/2d": ["node_modules/@revideo/2d/lib"],
      "@revideo/core": ["node_modules/@revideo/core/lib"]
    }
  }
}
```

3. **Vite configuration:**
```typescript
// vite.config.ts
import {defineConfig} from 'vite';
import motionCanvas from '@revideo/vite-plugin';

export default defineConfig({
  plugins: [motionCanvas()],
});
```

### FFmpeg Installation Issues

**Solutions:**

1. **Install FFmpeg:**
```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

2. **Check FFmpeg installation:**
```bash
ffmpeg -version
```

### FFmpeg SIGSEGV Error

**Error Pattern:**
```
SIGSEGV: segmentation violation
Error when evaluating SSR module /path/to/file.ts
```

**Root Cause:** FFmpeg binary compatibility issues.

**Solutions:**

1. **Clear node_modules and reinstall:**
```bash
rm -rf node_modules package-lock.json
npm install
```

2. **Use different FFmpeg installer:**
```json
// package.json
{
  "dependencies": {
    "@ffmpeg-installer/ffmpeg": "^1.1.0"
  }
}
```

## Shader and WebGL Issues

### WebGL Context Loss

**Error Message:**
```
"Failed to initialize WebGL."
```

**Source:** `packages/core/src/app/SharedWebGLContext.ts` (line 153)

**Solutions:**

1. **Handle context restoration:**
```typescript
// SharedWebGLContext handles this automatically
canvas.addEventListener('webglcontextlost', event => {
  event.preventDefault();
  // Wait for restoration
});

canvas.addEventListener('webglcontextrestored', () => {
  // Reinitialize resources
});
```

2. **Limit resource usage:**
```typescript
// Enable caching for complex scenes
<Rect cache cachePadding={20}>
  {/* Complex content */}
</Rect>
```

### Shader Compilation Error

**Error Messages:**
```
"Shader compilation error: '{token_name}' : syntax error"
"Shader compilation error: 'include' : syntax error"
```

**Note:** When using `#include` without the preprocessor enabled, the error message will specifically mention the `'include'` token with a syntax error.

**Source:** `packages/core/src/app/SharedWebGLContext.ts` (line 212, 222)

**Example of Problem:**
```glsl
// WRONG: Using #include without preprocessor
#version 300 es
#include "common.glsl"  // Error!

void main() {
  // ...
}
```

**Solutions:**

1. **Enable shader preprocessor:**
```typescript
// In your scene or project
const shaders = useScene().shaders;
shaders.setPreprocessor(true);
```

2. **Fix circular includes:**
```glsl
// common.glsl
#ifndef COMMON_GLSL
#define COMMON_GLSL
// shader code
#endif

// main.glsl
#include "common.glsl"
```

3. **Check GLSL syntax:**
```glsl
// CORRECT: Proper GLSL ES 3.0
#version 300 es
precision highp float;

in vec2 vTexCoord;
out vec4 fragColor;

void main() {
  fragColor = vec4(vTexCoord, 0.0, 1.0);
}
```

## Signal and Component Issues

### Signal Decorator Missing

**Root Cause:** Missing `@signal()` decorator on component property.

**Example of Problem:**
```typescript
// WRONG: Missing decorator
class CustomNode extends Node {
  public declare readonly myProperty: SimpleSignal<number, this>;
  // Will cause runtime error when accessed
}
```

**Solution:**
```typescript
// CORRECT: Add signal decorator
class CustomNode extends Node {
  @initial(0)
  @signal()
  public declare readonly myProperty: SimpleSignal<number, this>;
}
```

## HMR (Hot Module Reload) Issues

### HMR Error for Video Components

**Error Messages:**
```
"FfmpegVideoFrame can only be used with HMR."
"ServerSeekedVideo can only be used with HMR."
```

**Source:**
- `packages/2d/src/lib/utils/video/ffmpeg-client.ts` (line 4)
- `packages/2d/src/lib/components/Video.ts` (line 225)

**Root Cause:** These video components require HMR to be enabled for proper functionality.

**Solutions:**

1. **Ensure HMR is enabled in your development environment:**
```typescript
// vite.config.ts
export default defineConfig({
  server: {
    hmr: true // Should be enabled by default
  },
  // ...
});
```

2. **Use alternative video components for production:**
```typescript
// For production builds without HMR
const isProd = import.meta.env.PROD;

view.add(
  isProd ? (
    <Video src="video.mp4" /> // Regular video component
  ) : (
    <FfmpegVideoFrame src="video.mp4" /> // HMR-only component
  )
);
```

## Experimental Features

### Enabling Experimental Features

**Note:** Experimental features in Revideo require explicit opt-in through project configuration.

**Solution:**
```typescript
// src/project.ts
export default makeProject({
  experimentalFeatures: true,
  scenes: [/* ... */],
});
```

**Source:** `packages/core/src/utils/ExperimentalError.ts`

## Rendering and Performance Issues

### Canvas Context Issues

**Error Patterns:**
```
"Cannot read properties of null (reading 'getContext')"
"Invalid canvas size"
```

**Solutions:**

1. **Ensure proper canvas sizing:**
```typescript
// In project settings
export default makeProject({
  settings: {
    shared: {
      size: new Vector2(1920, 1080),  // Valid size
    },
    rendering: {
      resolutionScale: 1,  // Avoid too high values
    },
  },
});
```

2. **Handle DPI scaling:**
```typescript
// DPI awareness is handled automatically
// But can be configured:
const dpr = window.devicePixelRatio || 1;
canvas.width = size.x * dpr;
canvas.height = size.y * dpr;
ctx.scale(dpr, dpr);
```

### Z-Index Not Working

**Common Issue:** Z-index has no effect on components.

**Root Cause:** Components are rendered in tree order, not z-index order.

**Solutions:**

1. **Use proper ordering:**
```typescript
// WRONG: Expecting z-index to control order
view.add(
  <>
    <Circle zIndex={2} />
    <Rect zIndex={1} />  // Still renders on top
  </>
);

// CORRECT: Control render order
const circle = createRef<Circle>();
const rect = createRef<Rect>();

view.add(
  <>
    <Rect ref={rect} />
    <Circle ref={circle} />
  </>
);

// Or reorder dynamically
circle().moveBelow(rect());
```

2. **Use Layout for stacking:**
```typescript
<Layout layout direction="stack">
  <Rect />  // Bottom
  <Circle />  // Top
</Layout>
```

### Memory Leaks in Animations

**Symptoms:**
- Increasing memory usage over time
- Performance degradation
- Browser tab crashes

**Common Causes and Solutions:**

1. **Uncancelled loops:**
```typescript
// PROBLEM: Infinite loop never cancelled
yield loop(Infinity, () => rect().rotation(360, 2));

// SOLUTION: Store and cancel
const task = yield loop(Infinity, () => rect().rotation(360, 2));
// Later:
cancel(task);
```

2. **Unremoved elements:**
```typescript
// PROBLEM: Elements created but never removed
for (let i = 0; i < 1000; i++) {
  view.add(<Circle />);
}

// SOLUTION: Clean up
const refs = createRefArray<Circle>();
// ... add circles with refs
// ... animate
refs.forEach(ref => ref().remove());
```

3. **Event listener leaks:**
```typescript
// PROBLEM: Listener never removed
window.addEventListener('resize', handler);

// SOLUTION: Clean up in dispose
class CustomNode extends Node {
  private handler = () => { /* ... */ };

  constructor() {
    super();
    window.addEventListener('resize', this.handler);
  }

  dispose() {
    window.removeEventListener('resize', this.handler);
    super.dispose();
  }
}
```

### Caching Issues

**Error:**
```
"Could not create a cache canvas"
```

**Note:** The previous examples "Failed to cache" and "Cache canvas creation failed" were illustrative error descriptions. The actual error message from the codebase is shown above.

**Source:** `packages/2d/src/lib/components/Node.ts` (line 1353)

**Solutions:**

1. **Provide cache padding for effects:**
```typescript
// Effects like blur need extra space
<Rect
  cache
  cachePadding={20}  // Extra space for blur
  filters={{blur: 10}}
>
  {/* content */}
</Rect>
```

2. **Disable cache for dynamic content:**
```typescript
// Don't cache frequently changing content
<Rect cache={false}>
  <Txt text={() => `Time: ${time()}`} />
</Rect>
```

3. **Clear cache when needed:**
```typescript
const rect = createRef<Rect>();
// ... later
rect().cacheBBox = null;  // Force recache
```

## Animation and Timing Issues

### yield vs yield* Confusion

**Common Mistake:** Using `yield` when `yield*` is needed.

**Symptoms:**
- Animation doesn't run
- Unexpected parallel execution
- Timing issues

**Examples:**
```typescript
// WRONG: Animation doesn't run
yield rect().opacity(1, 1);  // Spawns parallel task

// CORRECT: Animation runs and waits
yield* rect().opacity(1, 1);  // Executes and waits

// For parallel execution:
yield all(
  rect().opacity(1, 1),
  rect().scale(2, 1),
);
yield* waitFor(1);  // Wait for all to complete
```

### Animation Not Visible

**Common Causes:**

1. **Zero duration:**
```typescript
// WRONG: Instant change, no animation
yield* rect().x(100, 0);  // Duration is 0

// CORRECT: Animated over time
yield* rect().x(100, 1);  // 1 second duration
```

2. **Element not in scene:**
```typescript
// WRONG: Animating before adding
const rect = createRef<Rect>();
yield* rect().x(100, 1);  // Error: rect not found

// CORRECT: Add first
view.add(<Rect ref={rect} />);
yield* rect().x(100, 1);  // OK
```

3. **Wrong property type:**
```typescript
// WRONG: Setting complex property directly
rect().position({x: 100, y: 50});  // Not animated

// CORRECT: Use proper signal
yield* rect().position([100, 50], 1);  // Animated
```

## Project Configuration Issues

### Node Version Requirements

**Error:**
```
"engines" : { "node" : ">=20.0.0" }
```

**Solution:**
```bash
# Check Node version
node --version

# Install Node 20+
# Using nvm:
nvm install 20
nvm use 20

# Or download from nodejs.org
```

### TypeScript Path Mapping

**Error:**
```
"Cannot find module '@/components/...' or its corresponding type declarations"
```

**Solution:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@scenes/*": ["src/scenes/*"]
    }
  }
}
```

```typescript
// vite.config.ts
import {resolve} from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@components': resolve(__dirname, './src/components'),
      '@scenes': resolve(__dirname, './src/scenes'),
    },
  },
});
```

## Debugging Tips

### Enable Logging

```typescript
import {useLogger} from '@revideo/core';

export default makeScene2D(function* (view) {
  const logger = useLogger();

  logger.profile('setup');  // Start profiling
  // ... setup code
  logger.profile('setup');  // End profiling

  logger.log('Animation starting');
  logger.warn('Potential issue');
  logger.error('Error occurred');
});
```

### Console Debugging

```typescript
// Debug signal values
const signal = createSignal(0);
console.log('Signal value:', signal());

// Debug ref access
const rect = createRef<Rect>();
view.add(<Rect ref={rect} />);
console.log('Rect position:', rect().position());

// Debug animation state
yield* tween(2, value => {
  console.log('Animation progress:', value);
  rect().x(map(-100, 100, value));
});
```

### Performance Monitoring

```typescript
export default makeScene2D(function* (view) {
  const startTime = performance.now();

  // ... setup

  console.log(`Setup took ${performance.now() - startTime}ms`);

  const animStart = performance.now();
  yield* /* animation */;
  console.log(`Animation took ${performance.now() - animStart}ms`);
});
```

## Common Gotchas

### Async in Generators

**WRONG:**
```typescript
export default makeScene2D(async function* (view) {  // Don't use async
  const data = await fetch('/api/data');  // Can't await in generator
});
```

**CORRECT:**
```typescript
export default makeScene2D(function* (view) {
  const data = yield* waitFor(async () => {
    return await fetch('/api/data').then(r => r.json());
  });
});
```

### Signal Dependencies

**WRONG:**
```typescript
// Accessing signal in tight loop
for (let i = 0; i < 1000; i++) {
  const value = expensiveSignal();  // Recalculates every time
  // use value
}
```

**CORRECT:**
```typescript
// Cache value outside loop
const value = expensiveSignal();
for (let i = 0; i < 1000; i++) {
  // use cached value
}
```

### Component Children

**WRONG:**
```typescript
// Modifying children directly
const rect = createRef<Rect>();
rect().children.push(new Circle());  // Don't modify directly
```

**CORRECT:**
```typescript
// Use proper methods
rect().add(<Circle />);
// Or
rect().insert(<Circle />, 0);  // At index
```

## Best Practices Summary

### Generators

1. Never use `async function*`
2. Use `yield*` to wait for animations
3. Use `yield` (without *) for parallel tasks
4. Don't reuse generator instances

### Signals

1. Always use decorators in components
2. Create computed signals for derived values
3. Avoid circular dependencies
4. Cache expensive computations

### Components

1. Always type refs explicitly
2. Add to scene before animating
3. Remove when no longer needed
4. Use caching for complex static content

### WebGL/Shaders

1. Handle context loss gracefully
2. Limit texture/buffer usage
3. Use proper GLSL syntax
4. Avoid circular shader includes

## Additional Resources

- GitHub Issues: https://github.com/revideo/revideo/issues
- Documentation: https://docs.re.video
- Discord Community: https://discord.gg/revideo

## Quick Error Reference

| Error | Likely Cause | Quick Fix |
|-------|--------------|-----------|
| "Generator already being executed" | Reusing generator | Create new instance |
| "useScene() outside scene" | Wrong context | Call inside generator |
| "Failed to initialize WebGL" | GPU overload | Enable caching |
| "HMR required" | Video components need HMR | Enable HMR in dev |
| "Cache failed" | Missing padding | Add cachePadding |
| "Animation not visible" | Zero duration | Add duration > 0 |