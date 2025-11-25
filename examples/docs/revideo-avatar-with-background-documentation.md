# Revideo Avatar with Background - Complete Implementation Guide

## Overview

This guide provides comprehensive documentation for implementing an avatar-with-background video composition using Revideo. The example demonstrates how to overlay a transparent video (avatar) on top of a static background image, creating social media-style content programmatically.

**Use Case:** Creating automated video content with animated avatars over static backgrounds, commonly used for social media posts, educational content, or automated video generation.

## Project Structure

```
avatar-with-background/
├── package.json          # Dependencies and build scripts
├── tsconfig.json         # TypeScript configuration
└── src/
    ├── project.tsx       # Main scene definition
    └── render.ts         # Programmatic rendering script
```

## Complete Source Code

### src/project.tsx

```typescript
import {Img, makeScene2D, Video} from '@revideo/2d';
import {useScene, createRef, waitFor, makeProject, Vector2} from '@revideo/core';

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {
  const avatarRef = createRef<Video>();
  const backgroundRef = createRef<Img>();

  yield view.add(
    <>
      <Img
        src={'https://revideo-example-assets.s3.amazonaws.com/mountains.jpg'}
        height={'100%'}
        ref={backgroundRef}
      />
      <Video
        src={'https://revideo-example-assets.s3.amazonaws.com/avatar.webm'}
        play={true}
        width={'100%'}
        ref={avatarRef}
        decoder={"slow"} // we need to use the slow decoder to support transparency
      />
    </>,
  );

  avatarRef().position.y(useScene().getSize().y / 2 - avatarRef().height() / 2);

  yield* waitFor(avatarRef().getDuration());
});

/**
 * The final revideo project
 */
export default makeProject({
  scenes: [scene],
  settings: {
    // Example settings:
    shared: {
      size: {x: 1080, y: 1080},
    },
  },
});
```

### src/render.ts

```typescript
import {renderVideo} from '@revideo/renderer';

async function render() {
  console.log('Rendering video...');

  // This is the main function that renders the video
  const file = await renderVideo({
    projectFile: './src/project.tsx',
    settings: {logProgress: true},
  });

  console.log(`Rendered video to ${file}`);
}

render();
```

## Configuration Files

### package.json

```json
{
  "name": "revideo-app",
  "private": true,
  "version": "0.0.0",
  "scripts": {
    "start": "revideo editor --projectFile ./src/project.tsx",
    "render": "tsc && node dist/render.js"
  },
  "dependencies": {
    "@revideo/core": "0.10.3",
    "@revideo/2d": "0.10.3",
    "@revideo/renderer": "0.10.3"
  },
  "devDependencies": {
    "@revideo/ui": "0.10.3",
    "@revideo/cli": "0.10.3",
    "typescript": "5.5.4",
    "vite": "^4.5"
  }
}
```

### tsconfig.json

```json
{
  "extends": "@revideo/2d/tsconfig.project.json",
  "include": ["src"],
  "compilerOptions": {
    "noEmit": false,
    "outDir": "dist",
    "module": "CommonJS",
    "skipLibCheck": true
  }
}
```

## API Reference and Concepts

### Imports from @revideo/2d

#### `Img` Component
- **Purpose:** Displays static images in the scene
- **Usage in example:** Background image display
- **Key properties:**
  - `src`: URL or path to the image
  - `height`: Size specification (can be percentage or pixels)
  - `ref`: Component reference for later manipulation

#### `Video` Component
- **Purpose:** Plays video content with optional transparency support
- **Usage in example:** Avatar video overlay
- **Key properties:**
  - `src`: URL or path to the video file
  - `play`: Boolean to auto-start playback
  - `width`: Size specification
  - `ref`: Component reference
  - `decoder`: Special setting for transparency ("slow" enables alpha channel)

#### `makeScene2D` Function
- **Purpose:** Creates a 2D scene for the animation
- **Signature:** `makeScene2D(name: string, generator: GeneratorFunction)`
- **Returns:** A scene object that can be added to the project
- **Usage:** Wraps the entire scene logic in a generator function

### Imports from @revideo/core

#### `createRef<T>()` Function
- **Purpose:** Creates a reference to a component for later manipulation
- **Type parameter:** Component type (e.g., `Video`, `Img`)
- **Usage pattern:**
  ```typescript
  const ref = createRef<ComponentType>();
  // Later access: ref().property
  ```

#### `useScene()` Hook
- **Purpose:** Access current scene properties
- **Available methods:**
  - `getSize()`: Returns scene dimensions as {x, y} object
- **Usage in example:** Calculate avatar positioning

#### `waitFor(duration)` Generator
- **Purpose:** Pauses animation execution for specified duration
- **Usage:** `yield* waitFor(duration)`
- **In example:** Waits for avatar video duration to complete

#### `makeProject()` Function
- **Purpose:** Creates the final Revideo project configuration
- **Configuration object:**
  - `scenes`: Array of scenes to include
  - `settings.shared.size`: Output video dimensions

#### `Vector2` Type
- **Purpose:** Represents 2D coordinates/dimensions
- **Note:** Imported but not directly used in this example

### Imports from @revideo/renderer

#### `renderVideo()` Function
- **Purpose:** Programmatically renders the video without UI
- **Parameters:**
  - `projectFile`: Path to the project TypeScript file
  - `settings.logProgress`: Boolean for progress logging
- **Returns:** Promise resolving to the output file path

## Key Implementation Patterns

### 1. Component Reference Pattern
```typescript
const componentRef = createRef<ComponentType>();
// Access after creation:
componentRef().property
```

### 2. Generator Function Animation
```typescript
function* (view) {
  // Setup phase
  yield view.add(components);

  // Animation phase
  yield* waitFor(duration);
}
```

### 3. JSX Component Composition
```tsx
<>
  <Img {...props} />
  <Video {...props} />
</>
```

### 4. Dynamic Positioning Calculation
```typescript
avatarRef().position.y(
  useScene().getSize().y / 2 - avatarRef().height() / 2
);
```
**Purpose:** Positions avatar at bottom of frame by:
1. Getting half of scene height (center point)
2. Subtracting half of avatar height
3. Result: Avatar bottom-aligned in scene

### 5. Transparency Support Configuration
```typescript
decoder={"slow"}
```
**Critical:** Required for WebM videos with alpha channel. Without this, transparency won't work.

## Running the Example

### Interactive Editor Mode
```bash
npm start
```
- Opens Revideo editor UI at default port
- Provides real-time preview and timeline control
- Allows interactive development and testing

### Programmatic Rendering
```bash
npm run render
```
- Compiles TypeScript to JavaScript
- Executes render.ts script
- Outputs video file to default location
- Shows progress in console

## Assets and Resources

### External Assets Used

1. **Background Image**
   - URL: `https://revideo-example-assets.s3.amazonaws.com/mountains.jpg`
   - Type: Static JPEG image
   - Usage: Scene background

2. **Avatar Video**
   - URL: `https://revideo-example-assets.s3.amazonaws.com/avatar.webm`
   - Type: WebM video with transparency
   - Usage: Foreground animated element

### Output Specifications
- **Resolution:** 1080x1080 pixels (square format)
- **Format:** Optimized for social media platforms
- **Layer order:** Background image → Avatar video

## Technical Notes

### Video Transparency Requirements
- Video format must support alpha channel (WebM recommended)
- Must set `decoder="slow"` on Video component
- Transparency preserved in final render

### Scene Coordinate System
- Origin (0,0) is at scene center
- Positive Y moves downward
- Dimensions specified in project settings apply to full canvas

### Component Lifecycle
1. Components created with JSX syntax
2. Added to scene with `yield view.add()`
3. References available immediately after add
4. Properties can be modified post-creation

### Generator Function Execution
- Code before first `yield` runs during setup
- `yield` statements control animation flow
- `yield*` delegates to other generators
- Function completes when generator exhausts

## Common Use Cases

This implementation pattern is suitable for:
- Social media content with animated presenters
- Educational videos with avatar guides
- Automated video responses
- Marketing content with brand avatars
- Video templates with replaceable backgrounds

## File Output

The rendered video is saved to the project's output directory (location determined by Revideo's default settings or custom configuration if specified).