# Revideo Video Stitching Example - Complete Documentation

## Overview

This documentation provides a comprehensive analysis of the stitching-videos example project, which demonstrates how to stitch multiple videos together with smooth fade transitions using Revideo. The example showcases dynamic video loading, responsive aspect ratio handling, and professional transition effects, making it ideal for creating video compilations or montages.

## File Structure

```
stitching-videos/
├── src/
│   ├── project.tsx         # Main project scene definition with video stitching logic
│   └── render.ts           # Video rendering script
├── README.md               # Project readme with basic information
├── package.json            # Project dependencies and scripts
├── package-lock.json       # Locked dependency versions
└── tsconfig.json          # TypeScript configuration
```

## Dependencies and Configuration

### Core Dependencies
- **@revideo/core**: 0.10.3 - Core Revideo functionality and animation primitives
- **@revideo/2d**: 0.10.3 - 2D components and utilities for Revideo
- **@revideo/renderer**: 0.10.3 - Video rendering capabilities

### Development Dependencies
- **@revideo/ui**: 0.10.3 - User interface for the Revideo editor
- **@revideo/cli**: 0.10.3 - Command-line interface for Revideo
- **typescript**: 5.5.4 - TypeScript compiler
- **vite**: ^4.5 - Build tool and development server

### Build Scripts
- `start`: Launches the Revideo editor with the project file
- `render`: Compiles TypeScript and runs the render script to generate video

### TypeScript Configuration
The project extends Revideo's base TypeScript configuration and compiles to CommonJS modules in the `dist` directory with library checking skipped for faster compilation.

## Source Code Analysis

### 1. Main Project File (src/project.tsx)

```typescript
import {makeProject} from '@revideo/core';

import {makeScene2D, Video} from '@revideo/2d';
import {useScene, createRef, waitFor} from '@revideo/core';

interface VideoObject {
  src: string;
  start: number;
  duration: number;
}

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {
  const videos: VideoObject[] = useScene().variables.get('videos', [])();

  for(const vid of videos){
    const videoRef = createRef<Video>();
    yield view.add(<Video src={vid.src} time={vid.start} play={true} opacity={0} ref={videoRef} />)

    if(view.width() / view.height() > videoRef().width() / videoRef().height()){
      videoRef().width("100%");
    } else {
      videoRef().height("100%");
    }

    // this is a fade-out / fade-in transition - the fade-out and in take 0.3 seconds each; just set the opacity of the video tag to 1 and only use yield* waitFor(vid.duration);
    yield* videoRef().opacity(1,0.3);
    yield* waitFor(vid.duration-0.6);
    yield* videoRef().opacity(0,0.3);
    videoRef().pause();
    videoRef().remove();
  }
});


/**
 * The final revideo project
 */
export default makeProject({
  scenes: [scene],
  variables: {
    videos: [
      {
        src: "https://revideo-example-assets.s3.amazonaws.com/beach-3.mp4",
        start: 1,
        duration: 6
      },
      {
        src: "https://revideo-example-assets.s3.amazonaws.com/beach-2.mp4",
        start: 2,
        duration: 10
      }
    ]
  },
  settings: {
    // Example settings:
    shared: {
      size: {x: 1920, y: 1080},
    },
  },
});
```

### 2. Render Script (src/render.ts)

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

## Key Revideo Concepts Demonstrated

### 1. Project Structure
- **makeProject**: Creates the main project configuration with scenes, variables, and settings
- **makeScene2D**: Creates a 2D scene container with a generator function for animation logic
- **Scene Variables**: Dynamic data passed to scenes through the variables system

### 2. Component System
- **Video Component**: Specialized component for video playback with properties:
  - `src`: Video source URL
  - `time`: Start time within the video clip
  - `play`: Boolean to control playback state
  - `opacity`: Transparency for fade effects
  - `width/height`: Dimension controls for responsive sizing
  - `pause()`: Method to pause playback
  - `remove()`: Method to remove from scene

### 3. Reference System
- **createRef<T>**: Creates typed references to components for runtime manipulation
- **Reference Access**: Using `ref()` to access component properties and methods

### 4. Animation Flow
- **Generator Functions**: Scene logic uses `function*` for coroutine-based animations
- **yield**: Single-frame operations (e.g., adding components to view)
- **yield***: Multi-frame animations (e.g., opacity transitions, waiting)
- **waitFor**: Pauses execution for specified duration in seconds

### 5. Scene Variables
- **useScene()**: Hook to access scene context
- **variables.get()**: Retrieves variables passed to the scene
- **Default Values**: Fallback to empty array if no videos provided

## Video Stitching Implementation Details

### 1. Data Structure
```typescript
interface VideoObject {
  src: string;    // URL or path to video file
  start: number;  // Start time within the video (seconds)
  duration: number; // How long to play this video (seconds)
}
```

### 2. Stitching Algorithm

#### Step-by-Step Process:
1. **Video Loading**: Iterate through array of VideoObject configurations
2. **Initial Setup**: Add each video to scene with opacity 0 (invisible)
3. **Aspect Ratio Handling**:
   - Compare viewport aspect ratio with video aspect ratio
   - Set width to 100% if viewport is wider
   - Set height to 100% if viewport is taller
   - Ensures video covers entire viewport without distortion
4. **Fade-In Transition**: Animate opacity from 0 to 1 over 0.3 seconds
5. **Playback Duration**: Wait for (video.duration - 0.6) seconds
6. **Fade-Out Transition**: Animate opacity from 1 to 0 over 0.3 seconds
7. **Cleanup**: Pause video and remove from scene to free memory

### 3. Transition System

#### Timing Breakdown:
- **Fade-in duration**: 0.3 seconds
- **Fade-out duration**: 0.3 seconds
- **Total transition time**: 0.6 seconds
- **Visible playback time**: video.duration - 0.6 seconds

#### Transition Characteristics:
- Smooth opacity-based transitions
- No gap between videos (seamless stitching)
- Professional-looking cross-fade effect
- Configurable through duration parameters

### 4. Responsive Aspect Ratio Handling

```typescript
if(view.width() / view.height() > videoRef().width() / videoRef().height()){
  videoRef().width("100%");  // Viewport is wider than video
} else {
  videoRef().height("100%"); // Viewport is taller than video
}
```

This algorithm ensures:
- Videos always cover the entire viewport
- Maintains aspect ratio without stretching
- Centers video content automatically
- Works with any video dimensions

### 5. Memory Management
- Videos are paused after playback
- Components are removed from scene
- Prevents memory leaks with multiple videos
- Efficient for long video compilations

## Technical Implementation Patterns

### 1. Generator-Based Animation Flow
The scene uses a generator function that yields control back to Revideo's animation system:
- Synchronous-looking code for asynchronous operations
- Natural flow for sequential animations
- Easy to understand and maintain

### 2. Dynamic Content Loading
Videos are configured through project variables:
- Easy to modify video sources without code changes
- Supports any number of videos
- Configuration separated from logic

### 3. Component Lifecycle
Each video follows a clear lifecycle:
1. Creation with initial state
2. Configuration (sizing, positioning)
3. Animation (fade-in, playback, fade-out)
4. Cleanup (pause, remove)

### 4. Time-Based Animations
All animations are time-based rather than frame-based:
- Consistent playback across different frame rates
- Precise timing control
- Professional quality output

## Sample Configuration

The example includes two beach videos:

```typescript
videos: [
  {
    src: "https://revideo-example-assets.s3.amazonaws.com/beach-3.mp4",
    start: 1,      // Start at 1 second into the video
    duration: 6    // Play for 6 seconds total
  },
  {
    src: "https://revideo-example-assets.s3.amazonaws.com/beach-2.mp4",
    start: 2,      // Start at 2 seconds into the video
    duration: 10   // Play for 10 seconds total
  }
]
```

### Timeline:
- **0.0s - 0.3s**: First video fades in
- **0.3s - 5.4s**: First video plays (5.1 seconds visible)
- **5.4s - 5.7s**: First video fades out
- **5.7s - 6.0s**: Second video fades in
- **6.0s - 15.4s**: Second video plays (9.4 seconds visible)
- **15.4s - 15.7s**: Second video fades out
- **Total Duration**: ~15.7 seconds

## Assets and Resources

### External Resources
- Videos hosted on AWS S3 bucket
- No local media files required
- Example uses publicly accessible beach footage

### Video Sources
- `beach-3.mp4`: First beach scene video
- `beach-2.mp4`: Second beach scene video
- Both videos are hosted at: `https://revideo-example-assets.s3.amazonaws.com/`

## Project Settings

### Video Output Configuration
```typescript
settings: {
  shared: {
    size: {x: 1920, y: 1080}  // Full HD resolution
  }
}
```

- **Resolution**: 1920x1080 (Full HD)
- **Aspect Ratio**: 16:9
- **Frame Rate**: Determined by Revideo defaults

## Customization Options

### 1. Adding More Videos
Simply add more VideoObject entries to the variables array:
```typescript
videos: [
  { src: "video1.mp4", start: 0, duration: 5 },
  { src: "video2.mp4", start: 0, duration: 5 },
  { src: "video3.mp4", start: 0, duration: 5 }
]
```

### 2. Adjusting Transition Duration
Modify the opacity animation duration and adjust waitFor accordingly:
```typescript
// For 0.5 second transitions:
yield* videoRef().opacity(1, 0.5);
yield* waitFor(vid.duration - 1.0);
yield* videoRef().opacity(0, 0.5);
```

### 3. Different Transition Effects
The current implementation uses fade transitions, but could be modified for:
- Cross-dissolve (overlapping videos)
- Slide transitions (position animations)
- Zoom effects (scale animations)
- Custom transition patterns

### 4. Removing Transitions
For hard cuts between videos:
```typescript
videoRef().opacity(1);  // Instant opacity change
yield* waitFor(vid.duration);
videoRef().pause();
videoRef().remove();
```

## Project Execution

### Development Mode
Run `npm start` to:
1. Launch Revideo editor interface
2. Load project from `./src/project.tsx`
3. Preview video stitching in real-time
4. Adjust parameters interactively
5. Export when satisfied with result

### Rendering Mode
Run `npm run render` to:
1. Compile TypeScript files to JavaScript
2. Execute `dist/render.js`
3. Process all videos according to configuration
4. Generate final stitched video file
5. Output file path to console

### Performance Considerations
- Videos are loaded on-demand
- Only one video plays at a time (except during transitions)
- Proper cleanup prevents memory accumulation
- Suitable for both short clips and long compilations

## Use Cases

This video stitching example is ideal for:
1. **Video Compilations**: Combining multiple clips into one video
2. **Highlight Reels**: Creating best-of videos with smooth transitions
3. **Slideshow Videos**: Transitioning between different scenes
4. **Video Montages**: Artistic combinations of multiple sources
5. **Automated Video Generation**: Programmatically creating videos from URL lists
6. **Social Media Content**: Creating engaging multi-clip videos
7. **Educational Content**: Combining instructional video segments

## Summary

The video stitching example demonstrates a practical and elegant approach to combining multiple videos with professional transitions. Key strengths include:

1. **Simple Yet Powerful**: Minimal code achieves professional results
2. **Flexible Configuration**: Easy to adapt for different video sources and timings
3. **Responsive Design**: Automatic aspect ratio handling for any video dimensions
4. **Smooth Transitions**: Professional fade effects between videos
5. **Memory Efficient**: Proper cleanup and resource management
6. **Production Ready**: Suitable for real-world video generation tasks

The implementation showcases Revideo's capability to handle complex video operations with clean, maintainable code, making it an excellent starting point for video compilation projects or automated video generation systems.