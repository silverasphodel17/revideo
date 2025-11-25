# Revideo Three.js Integration Example - Complete Documentation

## Overview

This documentation provides a comprehensive analysis of the three-js-example project, which demonstrates the integration of Three.js (a 3D graphics library) with Revideo (a video animation framework). The example showcases how to create animated 3D scenes within Revideo projects using WebGL rendering.

## File Structure

```
three-js-example/
├── src/
│   ├── components/
│   │   └── Three.ts         # Custom Three.js component for Revideo
│   ├── global.css           # Global CSS with font imports
│   ├── project.tsx          # Main project scene definition
│   └── render.ts            # Video rendering script
├── package.json             # Project dependencies and scripts
├── package-lock.json        # Locked dependency versions
└── tsconfig.json           # TypeScript configuration
```

## Dependencies and Configuration

### Core Dependencies
- **@revideo/2d**: 0.10.3 - Provides 2D components and utilities for Revideo
- **@revideo/core**: 0.10.3 - Core Revideo functionality and animation primitives
- **@revideo/renderer**: 0.10.3 - Video rendering capabilities
- **three**: ^0.169.0 - Three.js 3D graphics library

### Development Dependencies
- **@revideo/cli**: 0.10.3 - Command-line interface for Revideo
- **@revideo/ui**: 0.10.3 - User interface for the Revideo editor
- **@types/three**: ^0.169.0 - TypeScript type definitions for Three.js
- **typescript**: 5.5.4 - TypeScript compiler
- **vite**: ^4.5 - Build tool and development server

### Build Scripts
- `start`: Launches the Revideo editor with the project file
- `render`: Compiles TypeScript and runs the render script to generate video

### TypeScript Configuration
The project extends Revideo's base TypeScript configuration and compiles to CommonJS modules in the `dist` directory.

## Source Code Analysis

### 1. Main Project File (src/project.tsx)

```tsx
import {Three} from './components/Three';
import { makeScene2D, Txt } from '@revideo/2d';
import { makeProject, tween, waitFor, delay, createRef, all, chain, linear} from '@revideo/core';
import * as THREE from 'three';
import './global.css';

function setup3DScene() {
    const threeScene = new THREE.Scene();

    const geometry = new THREE.BoxGeometry( 0.2, 0.2, 0.2 );
    const material = new THREE.MeshNormalMaterial();

    const mesh = new THREE.Mesh( geometry, material );
    threeScene.add(mesh);

    const camera = new THREE.PerspectiveCamera(90);

    mesh.position.set(0, 0, 0);
    mesh.scale.set(1, 1, 1);
    camera.rotation.set(0, 0, 0);
    camera.position.set(0, 0, 0.5);

    return {threeScene, camera, mesh};
}

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {
  const {threeScene, camera, mesh} = setup3DScene();

  const threeRef = createRef<Three>();
  const txtRef = createRef<Txt>();

  yield view.add(
      <>
          <Three
              width={1920}
              height={1080}
              camera={camera}
              scene={threeScene}
              opacity={0}
              fontWeight={900}
              ref={threeRef}
          />
      </>
  );

  yield view.add(<Txt fill={"black"} fontFamily={"Lexend"} ref={txtRef} fontSize={80}/>)

  yield* chain(
      txtRef().text("Revideo x 3D with Three.js", 1),
      all(
        txtRef().position.y(-300, 1),
        delay(0.5, threeRef().opacity(1, 0.5))
      )
  )

  yield tween(4, value => {
      mesh.rotation.set(
          0,
          linear(value, 0, 2*3.14),
          0
      );
  });

  yield* waitFor(2);

  yield addRotatingCube(threeRef().scene(), 0.1, 0.3, -0.2, 0.1);
  yield addRotatingCube(threeRef().scene(), 0.1, -0.3, -0.2, 0.1);

  yield* waitFor(2);
});

function* addRotatingCube(threeScene: THREE.Scene, size: number, x: number, y: number, z: number){
  const geometry = new THREE.BoxGeometry( size, size, size );
  const material = new THREE.MeshNormalMaterial();
  const mesh = new THREE.Mesh( geometry, material );

  mesh.position.set(x, y, z);
  mesh.scale.set(1, 1, 1);

  threeScene.add(mesh);

  yield* tween(4, value => {
      mesh.rotation.set(
          0,
          linear(value, 0, 2*3.14),
          0
      );
  });
}

/**
 * The final revideo project
 */
export default makeProject({
  scenes: [scene],
  settings: {
    // Example settings:
    shared: {
      size: {x: 1920, y: 1080}
    },
  },
});
```

### 2. Three.js Component (src/components/Three.ts)

```typescript
import { Layout, LayoutProps } from "@revideo/2d/lib/components";
import { computed, initial, signal } from "@revideo/2d/lib/decorators";
import { Vector2 } from "@revideo/core/lib/types";
import {
  Camera,
  Color,
  OrthographicCamera,
  PerspectiveCamera,
  Scene,
  WebGLRenderer,
} from "three";
import { SimpleSignal } from "@revideo/core/lib/signals";

interface RenderCallback {
  (renderer: WebGLRenderer, scene: Scene, camera: Camera): void;
}

export interface ThreeProps extends LayoutProps {
  scene?: Scene;
  camera?: Camera;
  quality?: number;
  background?: string;
  zoom?: number;
  onRender?: RenderCallback;
}

export class Three extends Layout {
  @initial(1)
  @signal()
  public declare readonly quality: SimpleSignal<number, this>;

  @initial(null)
  @signal()
  public declare readonly camera: SimpleSignal<Camera | null, this>;

  @initial(null)
  @signal()
  public declare readonly scene: SimpleSignal<Scene | null, this>;

  @initial(null)
  @signal()
  public declare readonly background: SimpleSignal<string | null, this>;

  @initial(1)
  @signal()
  public declare readonly zoom: SimpleSignal<number, this>;

  private readonly renderer: WebGLRenderer;
  private readonly context: WebGLRenderingContext;
  private readonly pixelSample = new Uint8Array(4);
  public onRender: RenderCallback;

  public constructor({ onRender, ...props }: ThreeProps) {
    super(props);
    this.renderer = borrow();
    this.context = this.renderer.getContext();
    this.onRender =
      onRender ?? ((renderer, scene, camera) => renderer.render(scene, camera));
  }

  protected override async draw(context: CanvasRenderingContext2D) {
    const { width, height } = this.computedSize();
    const quality = this.quality();
    const scene = this.configuredScene();
    const camera = this.configuredCamera();
    const renderer = this.configuredRenderer();

    if (width > 0 && height > 0) {
      this.onRender(renderer, scene, camera);
      context.imageSmoothingEnabled = false;
      context.drawImage(
        renderer.domElement,
        0,
        0,
        quality * width,
        quality * height,
        width / -2,
        height / -2,
        width,
        height,
      );
    }

    super.draw(context);
  }

  @computed()
  private configuredRenderer(): WebGLRenderer {
    const size = this.computedSize();
    const quality = this.quality();

    this.renderer.setSize(size.width * quality, size.height * quality);
    return this.renderer;
  }

  @computed()
  private configuredCamera(): Camera {
    const size = this.computedSize();
    const camera = this.camera();
    const ratio = size.width / size.height;
    const scale = this.zoom() / 2;
    if (camera instanceof OrthographicCamera) {
      camera.left = -ratio * scale;
      camera.right = ratio * scale;
      camera.bottom = -scale;
      camera.top = scale;
      camera.updateProjectionMatrix();
    } else if (camera instanceof PerspectiveCamera) {
      camera.aspect = ratio;
      camera.updateProjectionMatrix();
    }

    return camera;
  }

  @computed()
  private configuredScene(): Scene | null {
    const scene = this.scene();
    const background = this.background();
    if (scene) {
      scene.background = background ? new Color(background) : null;
    }
    return scene;
  }

  public getColorAtPoint(position: Vector2) {
    const relativePosition = position.scale(this.quality());
    this.context.readPixels(
      relativePosition.x,
      relativePosition.y,
      1,
      1,
      this.context.RGBA,
      this.context.UNSIGNED_BYTE,
      this.pixelSample,
    );
    const color = new Color();
    color.setRGB(
      this.pixelSample[0] / 255,
      this.pixelSample[1] / 255,
      this.pixelSample[2] / 255,
    );
    return color;
  }

  public dispose() {
    dispose(this.renderer);
  }
}

const pool: WebGLRenderer[] = [];
function borrow() {
  if (pool.length) {
    return pool.pop();
  } else {
    return new WebGLRenderer({
      canvas: document.createElement("canvas"),
      // antialias: true,
      alpha: true,
      stencil: true,
    });
  }
}
function dispose(renderer: WebGLRenderer) {
  pool.push(renderer);
}
```

### 3. Render Script (src/render.ts)

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

### 4. Global CSS (src/global.css)

```css
@import url("https://fonts.googleapis.com/css2?family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=Lexend:wght@600&family=Noto+Color+Emoji&display=swap");
```

## Key Revideo Concepts Demonstrated

### 1. Scene Creation
- **makeScene2D**: Creates a 2D scene container for Revideo animations
- **makeProject**: Wraps scenes into a complete Revideo project
- **Generator Functions**: Scene logic uses generator functions (`function*`) to control animation flow

### 2. Animation Primitives
- **tween**: Animates values over time with interpolation functions
- **waitFor**: Pauses execution for a specified duration
- **chain**: Executes animations sequentially
- **all**: Executes multiple animations simultaneously
- **delay**: Delays the start of an animation
- **linear**: Linear interpolation function for smooth animations

### 3. References and Signals
- **createRef**: Creates references to components for runtime manipulation
- **signal**: Decorator for reactive properties that can be animated
- **computed**: Decorator for derived values that update automatically
- **initial**: Decorator for setting default signal values

### 4. Component System
- **Layout**: Base class for visual components
- **Txt**: Text rendering component
- **Custom Components**: The Three component extends Layout to integrate WebGL rendering

## Key Three.js Integration Patterns

### 1. WebGL Renderer Pooling
The Three component implements a renderer pooling system to optimize performance:
- Renderers are borrowed from a pool when needed
- Disposed renderers are returned to the pool for reuse
- Prevents memory leaks and reduces initialization overhead

### 2. Scene-Camera-Mesh Pattern
The example follows the standard Three.js scene structure:
- **Scene**: Container for all 3D objects
- **Camera**: PerspectiveCamera with configurable field of view
- **Mesh**: Combination of geometry (BoxGeometry) and material (MeshNormalMaterial)

### 3. Canvas Integration
The Three component bridges WebGL and 2D Canvas contexts:
- Renders Three.js scene to WebGL canvas
- Draws WebGL output to Revideo's 2D canvas context
- Maintains proper scaling and positioning

### 4. Reactive Properties
All Three component properties are reactive signals:
- Camera configuration updates automatically
- Scene background can be animated
- Quality and zoom are animatable properties

## Animation Flow and Techniques

### 1. Scene Initialization
1. Creates Three.js scene with initial cube mesh
2. Sets up PerspectiveCamera at z=0.5
3. Adds Three component to Revideo view with initial opacity of 0

### 2. Animation Sequence
1. **Text Animation**: Displays "Revideo x 3D with Three.js" over 1 second
2. **Reveal Animation**: Moves text down while fading in 3D scene
3. **Main Rotation**: Rotates central cube 360 degrees over 4 seconds
4. **Dynamic Additions**: Adds two smaller rotating cubes to the scene
5. **Continuous Animation**: All cubes rotate simultaneously

### 3. Key Animation Patterns
- **Chained Animations**: Sequential execution using `chain()`
- **Parallel Animations**: Simultaneous execution using `all()`
- **Delayed Start**: Using `delay()` for staggered animations
- **Value Interpolation**: Using `linear()` for smooth rotations

## Technical Implementation Details

### 1. Coordinate Systems
- Three.js uses right-handed coordinate system
- Revideo uses screen coordinates (origin at center)
- The Three component handles coordinate transformation

### 2. Rendering Pipeline
1. Three.js renders to WebGL context
2. WebGL output is captured as canvas image
3. Image is drawn to Revideo's 2D context
4. Composited with other Revideo components

### 3. Performance Optimizations
- WebGL renderer pooling reduces memory allocation
- Computed decorators prevent unnecessary recalculations
- Quality setting allows resolution/performance tradeoff
- Canvas image smoothing disabled for pixel-perfect rendering

### 4. Camera Configuration
- Supports both PerspectiveCamera and OrthographicCamera
- Automatic aspect ratio adjustment
- Zoom property for field of view control
- Camera updates trigger projection matrix recalculation

## Assets and Resources

### External Resources
- **Google Fonts**: Lexend (weight 600) and Inter fonts loaded via CSS
- **Noto Color Emoji**: Emoji font support

### Generated Assets
- No static image, video, or audio assets are used in this example
- All visuals are procedurally generated using Three.js geometries and materials

## Project Execution

### Development Mode
Run `npm start` to:
1. Launch Revideo editor interface
2. Load project from `./src/project.tsx`
3. Enable real-time preview and editing
4. Provide timeline controls and export options

### Rendering Mode
Run `npm run render` to:
1. Compile TypeScript files to JavaScript
2. Execute `dist/render.js`
3. Generate video file with progress logging
4. Output final video path to console

## Summary

This Three.js integration example demonstrates a sophisticated approach to combining 3D graphics with video animation frameworks. The implementation showcases:

1. **Modular Architecture**: Clean separation between 3D setup, animation logic, and rendering
2. **Reactive Programming**: Signal-based property system for dynamic updates
3. **Performance Awareness**: Renderer pooling and computed properties for efficiency
4. **Framework Integration**: Seamless blending of Three.js WebGL with Revideo's 2D canvas
5. **Animation Composition**: Complex timing using generators, chains, and parallel execution

The example provides a foundation for creating rich, animated 3D content within the Revideo ecosystem, enabling developers to leverage both the power of Three.js for 3D graphics and Revideo's animation capabilities for video production.