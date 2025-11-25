# Revideo Minimal Example - Comprehensive Documentation

## Table of Contents

1. [Overview](#overview)
2. [Setup & Installation](#setup--installation)
3. [Project Structure](#project-structure)
4. [Core Concepts](#core-concepts)
5. [Components Reference](#components-reference)
6. [Animation API](#animation-api)
7. [Project Configuration](#project-configuration)
8. [Running the Example](#running-the-example)
9. [Code Walkthrough](#code-walkthrough)
10. [API Reference](#api-reference)

---

## Overview

**Revideo** (re.video) is a video creation framework that allows developers to create videos programmatically using TypeScript/JavaScript code. This minimal example demonstrates the fundamental capabilities of Revideo through a simple but complete project.

### What This Example Demonstrates

The default example showcases:
- Loading and playing background video
- Adding background audio with time offset control
- Animating images (logo) with multiple properties
- Chaining and parallelizing animations
- JSX-based scene composition
- Programmatic video rendering

### Example Output

The example creates a video with:
- A starfield background video (full screen)
- Chill beat background music (starting at 17 seconds)
- Revideo logo that scales and rotates
- Full HD resolution (1920x1080)

---

## Setup & Installation

### Prerequisites

- Node.js (recommended: latest LTS version)
- npm or yarn package manager

### Installation

```bash
# Install dependencies
npm install
```

### Quick Start

This project can be bootstrapped using:
```bash
npm init @revideo@latest
# Select "minimal" template when prompted
```

---

## Project Structure

```
examples/default/
├── README.md           # Project documentation
├── package.json        # Dependencies and scripts
├── tsconfig.json       # TypeScript configuration
└── src/
    ├── project.tsx     # Main scene definition
    └── render.ts       # Programmatic rendering script
```

### File Descriptions

| File | Purpose |
|------|---------|
| `package.json` | Defines project dependencies, scripts, and metadata |
| `tsconfig.json` | TypeScript compiler configuration |
| `src/project.tsx` | Main entry point - defines scenes and animations |
| `src/render.ts` | Script for programmatic video rendering |

---

## Core Concepts

### 1. Generator-Based Animation

Revideo uses JavaScript generators to control animation timing and sequencing:

```typescript
makeScene2D('scene', function* (view) {
  // Generator function with yield statements
  yield* waitFor(1);  // Wait 1 second
  yield* chain(...);  // Execute animations in sequence
});
```

**Key Points:**
- `function*` declares a generator function
- `yield*` delegates to another generator
- Provides fine-grained control over animation timing

### 2. Component References

References allow programmatic control of components after they're rendered:

```typescript
const logoRef = createRef<Img>();

// In JSX
<Img ref={logoRef} src="..." />

// Animate programmatically
yield* logoRef().scale(40, 2);
```

### 3. JSX Composition

Scenes are composed using React-like JSX syntax:

```typescript
yield view.add(
  <>
    <Video src="..." size={['100%', '100%']} play={true} />
    <Audio src="..." play={true} time={17.0} />
  </>
);

// Components can be added at different points in the timeline
yield* waitFor(1);

view.add(
  <Img ref={logoRef} src="..." width={'1%'} />
);
```

### 4. Animation Primitives

**Sequential Animations** (`chain`):
```typescript
yield* chain(
  animation1,
  animation2,
  animation3
);
```

**Parallel Animations** (`all`):
```typescript
yield* all(
  animation1,
  animation2
);
```

---

## Components Reference

### Video Component

Displays and plays video files.

**Usage from Example:**
```typescript
<Video
  src={'https://revideo-example-assets.s3.amazonaws.com/stars.mp4'}
  size={['100%', '100%']}
  play={true}
/>
```

**Props:**
- `src`: Video file URL (string)
- `size`: Dimensions as `[width, height]` (strings with units or percentages)
- `play`: Auto-play the video (boolean)

### Audio Component

Plays audio files with optional time offset.

**Usage from Example:**
```typescript
<Audio
  src={'https://revideo-example-assets.s3.amazonaws.com/chill-beat.mp3'}
  play={true}
  time={17.0}
/>
```

**Props:**
- `src`: Audio file URL (string)
- `play`: Auto-play the audio (boolean)
- `time`: Start time offset in seconds (number)

### Img Component

Displays images with animatable properties.

**Usage from Example:**
```typescript
<Img
  width={'1%'}
  ref={logoRef}
  src={'https://revideo-example-assets.s3.amazonaws.com/revideo-logo-white.png'}
/>
```

**Props:**
- `src`: Image file URL (string)
- `width`: Initial width (string with units or percentage)
- `ref`: Reference for programmatic control (RefObject)

**Animatable Properties:**
- `scale(value, duration)`: Scale the image
- `rotation(degrees, duration)`: Rotate the image

---

## Animation API

### waitFor()

Pauses the animation for a specified duration.

**Signature:**
```typescript
waitFor(duration: number)
```

**Usage from Example:**
```typescript
yield* waitFor(1);  // Wait 1 second
```

### chain()

Executes animations sequentially, one after another.

**Signature:**
```typescript
chain(...animations: Generator[])
```

**Usage from Example:**
```typescript
yield* chain(
  all(logoRef().scale(40, 2), logoRef().rotation(360, 2)),
  logoRef().scale(60, 1)
);
```

This executes:
1. First: Scale and rotate simultaneously
2. Then: Scale to 60x

### all()

Executes animations in parallel, simultaneously.

**Signature:**
```typescript
all(...animations: Generator[])
```

**Usage from Example:**
```typescript
yield* all(
  logoRef().scale(40, 2),      // Scale to 40x over 2 seconds
  logoRef().rotation(360, 2)    // Rotate 360° over 2 seconds
);
```

Both animations happen at the same time.

---

## Project Configuration

### Project Setup

**Usage from Example:**
```typescript
export default makeProject({
  scenes: [scene],
  settings: {
    shared: {
      size: {x: 1920, y: 1080},  // Full HD resolution
    },
  },
});
```

**Configuration Options:**
- `scenes`: Array of scene objects
- `settings.shared.size`: Output video resolution
  - `x`: Width in pixels
  - `y`: Height in pixels

### Scene Definition

**Usage from Example:**
```typescript
const scene = makeScene2D('scene', function* (view) {
  // Add initial components to the view
  yield view.add(
    <>
      <Video ... />
      <Audio ... />
    </>
  );

  // Wait before adding more components
  yield* waitFor(1);

  // Add logo after the wait
  view.add(<Img ... />);

  // Define animation sequence
  yield* chain(...);
});
```

**Parameters:**
- First argument: Scene name (string)
- Second argument: Generator function that receives `view` parameter

---

## Running the Example

### Development Mode (Interactive Editor)

Start the Revideo editor for real-time editing and preview:

```bash
npm start
```

**What it does:**
- Opens the Revideo editor in your browser
- Allows real-time editing of `src/project.tsx`
- Provides live preview of animations
- Hot-reloads on file changes

**Command Details:**
```json
"start": "revideo editor --projectFile ./src/project.tsx"
```

### Production Mode (Render Video)

Render the final video programmatically:

```bash
npm run render
```

**What it does:**
- Compiles TypeScript files to `dist/` directory
- Executes the render script
- Outputs the final video file
- Shows progress logs during rendering

**Command Details:**
```json
"render": "tsc && node ./dist/render.js"
```

### Editing the Project

To customize the animation:
1. Edit `src/project.tsx`
2. Modify components, animations, or add new elements
3. Run `npm start` to preview changes
4. Run `npm run render` to generate final video

---

## Code Walkthrough

### src/project.tsx

This is the main entry point that defines the video scene and animations.

#### 1. Imports

```typescript
import {makeProject} from '@revideo/core';
import {Audio, Img, makeScene2D, Video} from '@revideo/2d';
import {all, chain, createRef, waitFor} from '@revideo/core';
```

**Import Sources:**
- `@revideo/core`: Core functionality (makeProject, animation utilities)
- `@revideo/2d`: 2D components (Video, Audio, Img, makeScene2D)

#### 2. Scene Definition

```typescript
const scene = makeScene2D('scene', function* (view) {
  const logoRef = createRef<Img>();

  yield view.add(
    <>
      <Video
        src={'https://revideo-example-assets.s3.amazonaws.com/stars.mp4'}
        size={['100%', '100%']}
        play={true}
      />
      <Audio
        src={'https://revideo-example-assets.s3.amazonaws.com/chill-beat.mp3'}
        play={true}
        time={17.0}
      />
    </>,
  );

  yield* waitFor(1);

  view.add(
    <Img
      width={'1%'}
      ref={logoRef}
      src={
        'https://revideo-example-assets.s3.amazonaws.com/revideo-logo-white.png'
      }
    />,
  );

  yield* chain(
    all(logoRef().scale(40, 2), logoRef().rotation(360, 2)),
    logoRef().scale(60, 1),
  );
});
```

**Breakdown:**
1. **Create Reference**: `const logoRef = createRef<Img>();`
   - Creates a typed reference to the Img component
2. **Add Initial Components**: `yield view.add(...)`
   - Background video (full screen, auto-playing)
   - Background audio (auto-playing, starting at 17s)
   - Note the `yield` keyword when adding components
3. **Wait Before Logo**: `yield* waitFor(1);`
   - Wait 1 second before adding the logo
4. **Add Logo**: `view.add(<Img ... />)`
   - Logo image (1% width, referenced for animation)
   - Added separately after the wait
5. **Animation Sequence**:
   - Simultaneously scale logo to 40x and rotate 360° (2 seconds)
   - Then scale logo to 60x (1 second)

#### 3. Project Export

```typescript
export default makeProject({
  scenes: [scene],
  settings: {
    shared: {
      size: {x: 1920, y: 1080},
    },
  },
});
```

**Configuration:**
- Includes the scene in the project
- Sets output resolution to 1920x1080 (Full HD)

### src/render.ts

This script handles programmatic video rendering.

#### Complete Code

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

**Breakdown:**
1. **Import**: `renderVideo` from `@revideo/renderer`
2. **Async Function**: Defines an async render function
3. **Render Call**: `await renderVideo({...})`
   - Takes a configuration object with two properties:
     - `projectFile`: Path to the project file
     - `settings`: Render settings object
       - `logProgress`: Enable progress logging (boolean)
   - Returns the output file path
4. **Logging**: Console logs for progress tracking
5. **Execution**: Immediately invokes the render function

---

## API Reference

### Core Functions

#### makeProject()

Creates a Revideo project with scenes and settings.

**Signature:**
```typescript
makeProject(config: ProjectConfig): Project
```

**Config Properties:**
- `scenes`: Array of scene objects
- `settings`: Project settings object
  - `shared`: Shared settings across all scenes
    - `size`: Resolution as `{x: number, y: number}`

**Example:**
```typescript
makeProject({
  scenes: [scene1, scene2],
  settings: {
    shared: {
      size: {x: 1920, y: 1080}
    }
  }
});
```

#### makeScene2D()

Creates a 2D scene with components and animations.

**Signature:**
```typescript
makeScene2D(name: string, generator: (view: View2D) => Generator): Scene
```

**Parameters:**
- `name`: Scene identifier (string)
- `generator`: Generator function that receives a `view` parameter

**Example:**
```typescript
makeScene2D('myScene', function* (view) {
  // Add components and define animations
});
```

#### createRef()

Creates a reference to a component for programmatic control.

**Signature:**
```typescript
createRef<T>(): Reference<T>
```

**Example:**
```typescript
const logoRef = createRef<Img>();
<Img ref={logoRef} ... />
logoRef().scale(2, 1);  // Animate via reference
```

#### waitFor()

Pauses animation for specified duration.

**Signature:**
```typescript
waitFor(duration: number): Generator
```

**Parameters:**
- `duration`: Time to wait in seconds

**Example:**
```typescript
yield* waitFor(2.5);  // Wait 2.5 seconds
```

#### chain()

Executes animations sequentially.

**Signature:**
```typescript
chain(...animations: Generator[]): Generator
```

**Example:**
```typescript
yield* chain(
  animation1,
  animation2,
  animation3
);
```

#### all()

Executes animations in parallel.

**Signature:**
```typescript
all(...animations: Generator[]): Generator
```

**Example:**
```typescript
yield* all(
  animation1,
  animation2
);
```

### 2D Components

#### Video

**Import:** `import {Video} from '@revideo/2d';`

**Props:**
- `src: string` - Video file URL
- `size: [string, string]` - Dimensions as `[width, height]`
- `play: boolean` - Auto-play video

#### Audio

**Import:** `import {Audio} from '@revideo/2d';`

**Props:**
- `src: string` - Audio file URL
- `play: boolean` - Auto-play audio
- `time: number` - Start time offset in seconds

#### Img

**Import:** `import {Img} from '@revideo/2d';`

**Props:**
- `src: string` - Image file URL
- `width: string` - Initial width (with units or %)
- `ref: Reference<Img>` - Component reference

**Methods:**
- `scale(value: number, duration: number)` - Animate scale
- `rotation(degrees: number, duration: number)` - Animate rotation

### Renderer

#### renderVideo()

Renders a Revideo project to a video file.

**Import:** `import {renderVideo} from '@revideo/renderer';`

**Signature:**
```typescript
renderVideo(config: RenderConfig): Promise<string>
```

**Config Properties:**
- `projectFile`: Path to the project file (string)
- `settings`: Render settings object
  - `logProgress`: Enable progress logging (boolean)

**Returns:**
- Promise that resolves to the output file path

**Example:**
```typescript
const file = await renderVideo({
  projectFile: './src/project.tsx',
  settings: {logProgress: true},
});
console.log(`Video saved to: ${file}`);
```

---

## Dependencies

### Core Dependencies

```json
{
  "@revideo/core": "0.10.3",
  "@revideo/2d": "0.10.3",
  "@revideo/renderer": "0.10.3"
}
```

**Package Descriptions:**
- `@revideo/core`: Core animation engine and utilities
- `@revideo/2d`: 2D components and scene creation
- `@revideo/renderer`: Video rendering engine

### Development Dependencies

```json
{
  "@revideo/ui": "0.10.3",
  "@revideo/cli": "0.10.3",
  "typescript": "5.5.4",
  "vite": "^4.5"
}
```

**Package Descriptions:**
- `@revideo/ui`: Interactive editor UI
- `@revideo/cli`: Command-line interface tools
- `typescript`: TypeScript compiler
- `vite`: Build tool and dev server

---

## TypeScript Configuration

### tsconfig.json

```json
{
  "extends": "@revideo/2d/tsconfig.project.json",
  "include": ["src"],
  "compilerOptions": {
    "outDir": "dist",
    "module": "commonjs",
    "skipLibCheck": true
  }
}
```

**Configuration Breakdown:**
- **extends**: Inherits base configuration from `@revideo/2d`
- **include**: Compiles files in `src` directory
- **outDir**: Outputs compiled files to `dist` directory
- **module**: Uses CommonJS module system
- **skipLibCheck**: Skips type checking of declaration files for faster builds

---

## Assets Used in Example

All assets are hosted on AWS S3 at `revideo-example-assets.s3.amazonaws.com`:

1. **stars.mp4**
   - Background video showing starfield animation
   - Used at full screen (100% x 100%)

2. **chill-beat.mp3**
   - Background music track
   - Starts at 17 seconds into the audio

3. **revideo-logo-white.png**
   - Revideo logo image (white version)
   - Animated with scale and rotation

---

## Summary

The Revideo minimal example demonstrates a complete video creation workflow:

1. **Project Setup**: Configuration with scenes and settings
2. **Component Composition**: JSX-based layout with Video, Audio, and Img
3. **Animation Control**: Generator-based timing with chain and all
4. **Interactive Development**: Live editing with the Revideo editor
5. **Programmatic Rendering**: Command-line video generation

This example serves as a foundation for building more complex video projects with Revideo, showcasing the framework's declarative API and powerful animation capabilities.
