# Revideo GitHub Stars Celebration - Complete Documentation

## Overview

This documentation provides a comprehensive analysis of the github-stars-celebration project, which demonstrates how to create an animated celebration video using Revideo. The example showcases a space-themed animation featuring a rocket launching from Earth, followed by a thank you message for reaching 1,000 GitHub stars. This project illustrates image manipulation, position animations, text reveals, and logo scaling effects suitable for milestone celebrations or promotional content.

## File Structure

```
github-stars-celebration/
├── public/
│   ├── earth.png           # Earth planet image asset (33KB)
│   └── rocket.png          # Rocket ship image asset (27KB)
├── src/
│   ├── global.css          # Global CSS with Lexend font import
│   ├── project.tsx         # Main animation scene definition
│   └── render.ts           # Video rendering script
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

import {Img, makeScene2D, Txt} from '@revideo/2d';
import {all, useScene, createRef, easeInCubic} from '@revideo/core';
import './global.css';

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {
  const rocketRef = createRef<Img>();
  const earthRef = createRef<Img>();
  const txtRef = createRef<Txt>();
  const logoRef = createRef<Img>();

  view.fill("#000000")

  yield view.add(
    <>
    <Img
      width={'20%'}
      ref={earthRef}
      src={'earth.png'}
      position={[-useScene().getSize().width/2+50, useScene().getSize().height/2-50]}
    />,
    <Img
      width={'5%'}
      ref={rocketRef}
      src={
        '/rocket.png'
      }
      position={[earthRef().position().x+150, earthRef().position().y-150]}
    />,
    </>
  );

  yield* rocketRef().position([1000, -600], 2, easeInCubic);

  yield view.add(
    <Txt fontFamily={"Lexend"} fill="white" ref={txtRef} textAlign={"center"} fontSize={60}></Txt>
  )

  yield* txtRef().text("Thank you for 1,000 Github stars!", 2);

  yield view.add(
    <Img
      width={'1%'}
      ref={logoRef}
      y={-50}
      src={
        'https://revideo-example-assets.s3.amazonaws.com/revideo-logo-white.png'
      }
    />,
  );

  yield* all(txtRef().position.y(150, 2), logoRef().width("30%", 2));
});


/**
 * The final revideo project
 */
export default makeProject({
  scenes: [scene],
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

### 3. Global CSS (src/global.css)

```css
@import url('https://fonts.googleapis.com/css2?family=Lexend:wght@100..900&display=swap');
```

## Key Revideo Concepts Demonstrated

### 1. Scene Creation
- **makeScene2D**: Creates a 2D scene container with generator function for animations
- **makeProject**: Wraps scenes into a complete Revideo project with settings
- **Generator Functions**: Scene logic uses `function*` for coroutine-based animation flow

### 2. Component System
- **Img**: Image component for displaying PNG files and remote images
- **Txt**: Text component for animated typography
- **View**: The main viewport with background color control

### 3. Reference System
- **createRef<T>**: Creates typed references to components
- **Reference Access**: Using `ref()` to access and manipulate component properties

### 4. Animation Primitives
- **yield**: Single-frame operations (adding components)
- **yield***: Multi-frame animations with duration
- **all**: Executes multiple animations simultaneously
- **easeInCubic**: Cubic easing function for acceleration effect

### 5. Scene Utilities
- **useScene()**: Hook to access scene context
- **getSize()**: Gets scene dimensions for relative positioning

### 6. Position and Layout
- **Percentage-based sizing**: Using '20%', '5%', '30%' for responsive scaling
- **Absolute positioning**: Calculated positions relative to scene bounds
- **Relative positioning**: Rocket positioned relative to Earth

## Animation Sequence Analysis

### 1. Scene Setup (Frame 0)
```typescript
view.fill("#000000")  // Black background for space
```
- Sets black background to create space environment
- Provides contrast for white text and logo

### 2. Initial Element Placement
```typescript
// Earth positioning
position={[-useScene().getSize().width/2+50, useScene().getSize().height/2-50]}

// Rocket positioning (relative to Earth)
position={[earthRef().position().x+150, earthRef().position().y-150]}
```

#### Earth Position Calculation:
- X: Left edge + 50px offset = -960 + 50 = -910
- Y: Top edge - 50px offset = 540 - 50 = 490
- Places Earth in top-left corner with slight padding

#### Rocket Position Calculation:
- X: Earth X + 150 = -910 + 150 = -760
- Y: Earth Y - 150 = 490 - 150 = 340
- Positions rocket to the right and below Earth

### 3. Rocket Launch Animation (0-2 seconds)
```typescript
yield* rocketRef().position([1000, -600], 2, easeInCubic);
```
- **Duration**: 2 seconds
- **End Position**: [1000, -600] (off-screen top-right)
- **Easing**: Cubic ease-in for realistic acceleration
- **Movement Path**: Diagonal trajectory simulating launch

### 4. Text Addition and Reveal (2-4 seconds)
```typescript
yield* txtRef().text("Thank you for 1,000 Github stars!", 2);
```
- **Effect**: Typewriter-style text animation
- **Duration**: 2 seconds
- **Position**: Center of screen (default)
- **Style**: White text on black background for high contrast

### 5. Logo Addition and Final Animation (4-6 seconds)
```typescript
// Initial state
width={'1%'}
y={-50}

// Animated to
yield* all(
  txtRef().position.y(150, 2),     // Move text down
  logoRef().width("30%", 2)        // Scale logo up
);
```
- **Simultaneous Animations**:
  - Text moves down from center to y=150
  - Logo scales from 1% to 30% width
- **Duration**: 2 seconds for both
- **Effect**: Reveals branding while repositioning message

### Total Animation Duration: ~6 seconds

## Technical Implementation Details

### 1. Image Asset Management

#### Local Assets
```typescript
src={'earth.png'}   // Loaded from public/earth.png
src={'/rocket.png'} // Loaded from public/rocket.png
```
- Local images stored in public directory
- Automatically served by development server
- Paths relative to public folder

#### Remote Assets
```typescript
src={'https://revideo-example-assets.s3.amazonaws.com/revideo-logo-white.png'}
```
- Logo loaded from AWS S3 bucket
- Ensures consistent branding across examples
- No local storage required

### 2. Responsive Sizing Strategy
```typescript
width={'20%'}  // Earth: 20% of viewport width (384px)
width={'5%'}   // Rocket: 5% of viewport width (96px)
width={'30%'}  // Logo: Scales to 30% of viewport (576px)
```
- Percentage-based sizing maintains proportions
- Adapts to different output resolutions
- Ensures visual balance across devices

### 3. Position Calculation Pattern
```typescript
-useScene().getSize().width/2+50   // Left edge with offset
useScene().getSize().height/2-50   // Top edge with offset
```
- Scene uses centered coordinate system (0,0 at center)
- Width: 1920px (±960 from center)
- Height: 1080px (±540 from center)
- Offsets provide margin from edges

### 4. Easing Functions
```typescript
easeInCubic  // Slow start, accelerating motion
```
- Creates natural acceleration for rocket launch
- Mathematical function: t³
- Enhances realism of physics-based animation

### 5. Animation Composition
```typescript
yield* all(
  txtRef().position.y(150, 2),
  logoRef().width("30%", 2)
);
```
- Parallel execution for synchronized effects
- Both animations complete simultaneously
- Creates cohesive final reveal

## Assets and Resources

### Local Image Assets

#### earth.png
- **Size**: 33,060 bytes (33KB)
- **Purpose**: Planet Earth representation
- **Usage**: Static background element
- **Display Size**: 20% of viewport width

#### rocket.png
- **Size**: 27,442 bytes (27KB)
- **Purpose**: Animated rocket ship
- **Usage**: Moving element with launch animation
- **Display Size**: 5% of viewport width

### Remote Assets

#### revideo-logo-white.png
- **Source**: AWS S3 (revideo-example-assets bucket)
- **Purpose**: Branding element
- **Usage**: Scaling animation from 1% to 30%
- **Color**: White version for dark background

### Fonts

#### Lexend Font
- **Source**: Google Fonts CDN
- **Weights**: 100-900 (variable)
- **Usage**: Text message display
- **Style**: Modern, readable sans-serif

## Styling and Visual Design

### Color Scheme
- **Background**: `#000000` (Pure black) - Space theme
- **Text Color**: `white` - High contrast for readability
- **Images**: Full color PNGs with transparency support

### Typography
- **Font Family**: Lexend
- **Font Size**: 60px for main message
- **Text Alignment**: Center
- **Color**: White on black for maximum contrast

### Visual Hierarchy
1. **Initial Focus**: Earth and rocket (visual storytelling)
2. **Message Delivery**: Centered text animation
3. **Brand Reinforcement**: Logo reveal with text repositioning

## Animation Techniques and Patterns

### 1. Staggered Reveal Pattern
- Elements appear sequentially
- Each addition builds on previous
- Maintains viewer attention throughout

### 2. Motion and Static Balance
- Static: Earth remains fixed as reference point
- Dynamic: Rocket launches with acceleration
- Contrast creates visual interest

### 3. Text Animation Style
```typescript
yield* txtRef().text("Thank you for 1,000 Github stars!", 2);
```
- Progressive text reveal (typewriter effect)
- Creates anticipation and readability
- Common pattern for message delivery

### 4. Scaling for Emphasis
```typescript
logoRef().width("30%", 2)
```
- Dramatic size change draws attention
- From nearly invisible (1%) to prominent (30%)
- Effective for logo reveals

### 5. Synchronized Finale
- Multiple properties animate together
- Creates polished, professional ending
- Reinforces brand and message simultaneously

## Customization Options

### 1. Message Customization
```typescript
txtRef().text("Your custom message here!", 2);
```
- Easy to change celebration text
- Adaptable for different milestones
- Maintains animation timing

### 2. Color Variations
```typescript
view.fill("#001122")  // Dark blue for night sky
fill="gold"           // Golden text for premium feel
```

### 3. Image Replacements
- Swap earth.png for company logo
- Replace rocket with relevant icon
- Use different celebration imagery

### 4. Timing Adjustments
```typescript
// Faster rocket launch
yield* rocketRef().position([1000, -600], 1, easeInCubic);

// Slower text reveal
yield* txtRef().text("Message", 3);
```

### 5. Position Modifications
```typescript
// Center Earth
position={[0, 0]}

// Different rocket trajectory
yield* rocketRef().position([500, -800], 2, easeInCubic);
```

## Use Cases

### Milestone Celebrations
1. **GitHub Stars**: Current implementation
2. **Download Milestones**: "1 Million Downloads!"
3. **User Milestones**: "Welcome, User #10,000!"
4. **Anniversary Videos**: "5 Years of Innovation"
5. **Product Launches**: "Version 2.0 is Here!"

### Social Media Applications
1. **Twitter/X Announcements**: Short celebration videos
2. **LinkedIn Milestones**: Professional achievements
3. **YouTube Intros**: Channel milestone celebrations
4. **Instagram Stories**: Vertical format adaptable

### Corporate Communications
1. **Company Achievements**: Sales targets, anniversaries
2. **Team Celebrations**: Project completions
3. **Customer Thank You**: Appreciation videos
4. **Event Announcements**: Conference openings

## Performance Considerations

### Asset Optimization
- **Total Asset Size**: ~60KB for images
- **Remote Logo**: Cached after first load
- **Font Loading**: Async Google Fonts CDN
- **Render Time**: ~6 seconds of animation

### Memory Management
- Only 4 components total (efficient)
- No component removal needed
- Static images after initial placement
- Single animated element (rocket)

### Rendering Efficiency
- Simple transformations (position, scale)
- No complex filters or effects
- Hardware-accelerated animations
- Suitable for real-time preview

## Project Settings

### Video Output Configuration
```typescript
settings: {
  shared: {
    size: {x: 1920, y: 1080}
  }
}
```
- **Resolution**: 1920x1080 (Full HD)
- **Aspect Ratio**: 16:9 standard
- **Frame Rate**: Determined by Revideo defaults
- **Duration**: Approximately 6 seconds

## Project Execution

### Development Mode
Run `npm start` to:
1. Launch Revideo editor interface
2. Load project from `./src/project.tsx`
3. Preview animation in real-time
4. Adjust timing and positions interactively
5. Test with different messages

### Rendering Mode
Run `npm run render` to:
1. Compile TypeScript files to JavaScript
2. Execute `dist/render.js`
3. Process all animations sequentially
4. Generate final celebration video
5. Output file path to console

## Animation Flow Summary

```
Time | Action
-----|-------------------------------------------------------
0.0s | Black background set, Earth and Rocket appear
0.0s | Rocket begins launching (easeInCubic acceleration)
2.0s | Rocket exits frame, Text component added
2.0s | Text begins typing "Thank you for 1,000 Github stars!"
4.0s | Text fully revealed, Logo added at 1% size
4.0s | Text moves down, Logo scales up to 30%
6.0s | Animation complete, final frame holds
```

## Best Practices Demonstrated

### 1. Component References
- All interactive elements have refs
- Enables precise control and timing
- Maintains clean component access

### 2. Easing Functions
- Appropriate easing for motion type
- Cubic for acceleration (launch physics)
- Linear for text and scaling

### 3. Asset Management
- Local assets for project-specific images
- Remote assets for shared resources
- Proper path handling for both

### 4. Responsive Design
- Percentage-based sizing
- Position calculations from scene size
- Adaptable to different resolutions

### 5. Animation Composition
- Clear sequence of events
- Logical progression of reveals
- Synchronized finale for impact

## Summary

The GitHub Stars Celebration project demonstrates a complete animated celebration video with professional polish. Key achievements include:

1. **Visual Storytelling**: Rocket launch metaphor for reaching new heights
2. **Professional Animation**: Smooth transitions with appropriate easing
3. **Brand Integration**: Logo reveal maintains brand presence
4. **Customizable Design**: Easy to adapt for different celebrations
5. **Efficient Implementation**: Minimal code for maximum impact
6. **Performance Optimized**: Light assets and simple animations

The implementation showcases Revideo's ability to create engaging celebration content with cinematic quality, suitable for social media announcements, milestone celebrations, and corporate communications. The space theme and rocket launch create a memorable visual metaphor for achievement and growth, while the modular structure allows easy customization for various celebration contexts.