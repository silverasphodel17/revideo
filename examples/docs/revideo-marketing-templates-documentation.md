# Revideo Marketing Templates - Complete Documentation

## Overview

This documentation provides a comprehensive analysis of the marketing-templates examples, which demonstrate how to create animated social media marketing content using Revideo. The project includes two variations of Black Friday discount advertisements: a single full-screen template and a multi-panel quad layout variation. These templates showcase dynamic text animations, grid effects, and configurable branding elements suitable for social media marketing campaigns.

## Project Structure

```
marketing-templates/
├── README.md                          # Project overview with video demonstrations
├── marketing-template/                # Single video Black Friday template
│   ├── src/
│   │   ├── project.tsx                # Main animated marketing scene
│   │   └── render.ts                  # Video rendering script
│   ├── README.md                      # Template description
│   ├── package.json                   # Dependencies and scripts
│   ├── package-lock.json              # Locked dependency versions
│   └── tsconfig.json                  # TypeScript configuration
└── multiple-videos-in-one/            # Quad-panel variation template
    ├── src/
    │   ├── project.tsx                # Four-panel animated scene
    │   └── render.ts                  # Video rendering script
    ├── README.md                      # Template description
    ├── package.json                   # Dependencies and scripts
    ├── package-lock.json              # Locked dependency versions
    └── tsconfig.json                  # TypeScript configuration
```

## Dependencies and Configuration

### Core Dependencies (Both Templates)
- **@revideo/core**: 0.10.3 - Core Revideo functionality and animation primitives
- **@revideo/2d**: 0.10.3 - 2D components and utilities for Revideo
- **@revideo/renderer**: 0.10.3 - Video rendering capabilities

### Development Dependencies (Both Templates)
- **@revideo/ui**: 0.10.3 - User interface for the Revideo editor
- **@revideo/cli**: 0.10.3 - Command-line interface for Revideo
- **typescript**: 5.5.4 - TypeScript compiler
- **vite**: ^4.5 - Build tool and development server

### Build Scripts (Both Templates)
- `start`: Launches the Revideo editor with the project file
- `render`: Compiles TypeScript and runs the render script to generate video

### TypeScript Configuration (Both Templates)
Both projects extend Revideo's base TypeScript configuration and compile to CommonJS modules in the `dist` directory with library checking skipped for faster compilation.

## Source Code Analysis - Marketing Template (Single Video)

### 1. Main Project File (marketing-template/src/project.tsx)

```typescript
import {makeProject} from '@revideo/core';

import {Grid, Rect, Img, makeScene2D, View2D, Txt} from '@revideo/2d';
import {all, useScene, createRef, waitFor} from '@revideo/core';

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {
  const logoRef = createRef<Img>();
  const grid = createRef<Grid>();
  const headingRef = createRef<Txt>()


  view.add(
    <>
      <Rect
        fill={useScene().variables.get("backgroundColor", "#E2FF31")()}
        size={['100%', '100%']}
      />
    </>
  );

  view.add(
    <Grid
      ref={grid}
      width={'100%'}
      height={'100%'}
      stroke={'#999'}
      end={0}
      spacing={150}
    />,
  );

  yield* all(
    grid().end(0.5, 0.5).to(1, 0.5).wait(0.1),
    grid().start(0.5, 0.5).to(0, 0.5).wait(0.1),
  );

  view.add(<Txt fontSize={1} fontWeight={600} ref={headingRef} position={[-700, -100]} opacity={0.2} textWrap={true} width={"1%"}  textAlign={"left"} fontFamily={"Sans-Serif"}>BLACK FRIDAY</Txt>)
  yield* all(headingRef().fontSize(180, 0.5), headingRef().opacity(1, 0.5));

  const txtRef = createRef<Txt>();
  const subtitle = <Txt fontSize={60} ref={txtRef} fill={useScene().variables.get("backgroundColor", useScene().variables.get("backgroundColor", "#E2FF31")())()}  zIndex={1} fontFamily={"Sans-Serif"}>Up to 95% off</Txt>

  const textBoxRef = createRef<Txt>();

  view.add(<Rect left={[headingRef().left().x, 180]} ref={textBoxRef} width={txtRef().width()+60} height={100} zIndex={0} fill={"black"}/>);
  view.add(<Txt position={textBoxRef().position} width={textBoxRef().width()} paddingLeft={20} textAlign={"left"} fontSize={60} ref={txtRef} fill={useScene().variables.get("backgroundColor", "#E2FF31")()} zIndex={1} fontFamily={"Sans-Serif"}></Txt>)

  yield* txtRef().text("Up to 95% off", 1)
  yield* addItems(view, useScene().variables.get("texts", ["events", "gift cards", "and more..."])());

  yield* waitFor(1);
});


function* addItems(view: View2D, texts: string[]){
  let yPos = 0;
  for(let i=0; i< texts.length; i++){
    const txtRef1 = createRef<Txt>();
    const subtitle1 = <Txt fontSize={90} ref={txtRef1} fill={useScene().variables.get("backgroundColor", "#E2FF31")()}  zIndex={1} fontFamily={"Sans-Serif"}>{texts[i]}</Txt>

    const textBoxRef1 = createRef<Txt>();

    view.add(<Rect left={[100, yPos]} lineWidth={2} radius={30} stroke={"black"} ref={textBoxRef1} width={txtRef1().width()+150} height={100} zIndex={0} padding={70} fill={"white"}/>);
    view.add(<Txt position={textBoxRef1().position} width={textBoxRef1().width()} textAlign={"center"} fontSize={90} fill={"black"} zIndex={1} fontFamily={"Sans-Serif"}>{texts[i]}</Txt>)
    yPos = textBoxRef1().y() + textBoxRef1().height()-10;
    yield* waitFor(0.5);

  }

}


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

### 2. Render Script (marketing-template/src/render.ts)

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

## Source Code Analysis - Multiple Videos In One (Quad Layout)

### 1. Main Project File (multiple-videos-in-one/src/project.tsx)

```typescript
import {makeProject, Vector2} from '@revideo/core';

import {Txt, Rect, View2D, Grid, makeScene2D} from '@revideo/2d';
import {all, useScene, Reference, createRef, waitFor} from '@revideo/core';

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {


  const backgroundRef = createRef<Rect>();

  yield view.add(
    <>
      <Rect
        fill={"#9accf2"}
        position={[useScene().getSize().x*0.25-1920*0.5, useScene().getSize().y*0.25-1080*0.5]}
        size={['40%', '40%']}
        ref={backgroundRef}
      />
    </>
  );

  yield* video(view, backgroundRef, "#9accf2", ["gift cards", "discounts", "+ more!!"]);

  const topRight = createRef<Rect>();
  const bottomLeft = createRef<Rect>();
  const bottomRight = createRef<Rect>();


  view.add(
    <>
      <Rect
        fill={"#FDCFE5"}
        position={[useScene().getSize().x*0.25-1920*0.5, useScene().getSize().y*0.25-1080*0.5]}
        size={['40%', '40%']}
        ref={topRight}
        zIndex={-1}
      />
      <Rect
        fill={"#FDB827"}
        position={[useScene().getSize().x*0.25-1920*0.5, useScene().getSize().y*0.25-1080*0.5]}
        size={['40%', '40%']}
        ref={bottomLeft}
        zIndex={-1}
      />
      <Rect
        fill={"#8BF8A7"}
        position={[useScene().getSize().x*0.25-1920*0.5, useScene().getSize().y*0.25-1080*0.5]}
        size={['40%', '40%']}
        ref={bottomRight}
        zIndex={-1}
      />

    </>
  );

  yield* all(
    bottomLeft().y(useScene().getSize().y*0.75-1080*0.5, 1),
    topRight().x(useScene().getSize().x*0.75-1920*0.5, 1),
    bottomRight().position([useScene().getSize().x*0.75-1920*0.5, useScene().getSize().y*0.75-1080*0.5], 1)
  );

  yield* all(
    video(view, bottomLeft, "#FDB827", ["gift cards", "discounts", "+ more!!"]),
    video(view, topRight, "#FDCFE5", ["gift cards", "discounts", "+ more!!"]),
    video(view, bottomRight, "#8BF8A7", ["gift cards", "discounts", "+ more!!"]),
  )

});

function* video(view: View2D, backgroundRef: Reference<Rect>, backgroundColor: string, texts: string[]){
  const grid = createRef<Grid>();
  const headingRef = createRef<Txt>()


  backgroundRef().add(
    <Grid
      ref={grid}
      width={backgroundRef().width()}
      height={backgroundRef().height()}
      stroke={'#999'}
      end={0}
      spacing={150*0.4}
    />,
  );

  yield* all(
    grid().end(0.5, 0.5).to(1, 0.5).wait(0.1),
    grid().start(0.5, 0.5).to(0, 0.5).wait(0.1),
  );

  backgroundRef().add(<Txt fontSize={1*0.40} fontWeight={600} ref={headingRef} position={[-backgroundRef().width()*0.3645, -backgroundRef().width()*0.05208]} opacity={0.2} textWrap={true} width={"1%"}  textAlign={"left"} fontFamily={"Sans-Serif"}>BLACK FRIDAY</Txt>)
  yield* all(headingRef().fontSize(180*0.40, 0.5), headingRef().opacity(1, 0.5));

  const txtRef = createRef<Txt>();
  const subtitle = <Txt fontSize={60*0.40} ref={txtRef} fill={backgroundColor}  zIndex={1} fontFamily={"Sans-Serif"}>Up to 95% off</Txt>

  const textBoxRef = createRef<Txt>();

  backgroundRef().add(<Rect left={[headingRef().left().x, 180*0.40]} ref={textBoxRef} width={txtRef().width()+60*0.40} height={40} zIndex={0} fill={"black"}/>);
  backgroundRef().add(<Txt position={textBoxRef().position} width={textBoxRef().width()} paddingLeft={5} textAlign={"left"} fontSize={60*0.40} ref={txtRef} fill={backgroundColor} zIndex={1} fontFamily={"Sans-Serif"}></Txt>)

  yield* txtRef().text("Up to 95% off", 1)
  yield* addItems(backgroundRef, backgroundColor, texts);

  yield* waitFor(1);

}

function* addItems(background: Reference<Rect>, backgroundColor: string, texts: string[]){
  let yPos = 0;
  for(let i=0; i< texts.length; i++){
    const txtRef1 = createRef<Txt>();
    const subtitle1 = <Txt fontSize={90*0.40} ref={txtRef1} fill={backgroundColor}  zIndex={1} fontFamily={"Sans-Serif"}>{texts[i]}</Txt>

    const textBoxRef1 = createRef<Txt>();

    background().add(<Rect left={[50, yPos]} height={30*0.40} lineWidth={2} radius={30*0.40} stroke={"black"} ref={textBoxRef1} width={txtRef1().width()+150*0.40} zIndex={0} padding={70*0.40} fill={"white"}/>);
    background().add(<Txt position={textBoxRef1().position} width={textBoxRef1().width()} textAlign={"center"} fontSize={90*0.40} fill={"black"} zIndex={1} fontFamily={"Sans-Serif"}>{texts[i]}</Txt>)
    yPos = textBoxRef1().y() + textBoxRef1().height()-2.5;
    yield* waitFor(0.5);

  }

}

/**
 * The final revideo project
 */
export default makeProject({
  scenes: [scene],
  settings: {
    // Example settings:
    shared: {
      size: new Vector2(1920, 1080),
    },
  },
});
```

### 2. Render Script (multiple-videos-in-one/src/render.ts)

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

### 1. Scene Creation and Management
- **makeScene2D**: Creates a 2D scene container with generator function for animations
- **makeProject**: Wraps scenes into a complete Revideo project
- **View2D**: The main viewport for 2D content
- **Generator Functions**: Scene logic uses `function*` for coroutine-based animation flow

### 2. Component System
- **Rect**: Rectangle component for backgrounds and containers
- **Txt**: Text component for typography with styling options
- **Grid**: Grid component for animated background patterns
- **Img**: Image component (referenced but not used in these examples)

### 3. Reference System
- **createRef<T>**: Creates typed references to components
- **Reference<T>**: Type for component references
- **Reference Access**: Using `ref()` to access component properties

### 4. Animation Primitives
- **all**: Executes multiple animations simultaneously
- **waitFor**: Pauses execution for specified duration
- **yield**: Single-frame operations
- **yield***: Multi-frame animations
- **wait**: Adds delay to animation chains

### 5. Scene Variables
- **useScene()**: Hook to access scene context
- **variables.get()**: Retrieves configurable variables with defaults
- **getSize()**: Gets the scene dimensions

### 6. Positioning and Layout
- **Position calculations**: Dynamic positioning based on scene size
- **Percentage-based sizing**: Using '100%', '40%' for responsive layouts
- **Z-indexing**: Layer management for overlapping elements
- **Coordinate system**: Centered origin with offset calculations

## Marketing-Specific Implementation Patterns

### 1. Single Template Animation Flow

#### Grid Animation
```typescript
yield* all(
  grid().end(0.5, 0.5).to(1, 0.5).wait(0.1),
  grid().start(0.5, 0.5).to(0, 0.5).wait(0.1),
);
```
- Creates an expanding grid effect
- Grid animates from center outward
- Creates visual interest and modern aesthetic

#### Text Animation Sequence
1. **Main Heading**: "BLACK FRIDAY" fades in and scales up
2. **Subtitle**: "Up to 95% off" types out over 1 second
3. **Feature List**: Items appear sequentially with 0.5s delays

#### Dynamic Item Addition
```typescript
function* addItems(view: View2D, texts: string[])
```
- Iterates through text array
- Creates rounded rectangles with text
- Stacks items vertically with calculated positions
- Each item animates in with delay

### 2. Quad Layout Implementation

#### Panel Creation and Positioning
```typescript
// Initial position (all panels start at top-left quadrant)
position={[useScene().getSize().x*0.25-1920*0.5, useScene().getSize().y*0.25-1080*0.5]}

// Animation to final positions
yield* all(
  bottomLeft().y(useScene().getSize().y*0.75-1080*0.5, 1),
  topRight().x(useScene().getSize().x*0.75-1920*0.5, 1),
  bottomRight().position([useScene().getSize().x*0.75-1920*0.5, useScene().getSize().y*0.75-1080*0.5], 1)
);
```

#### Synchronized Multi-Panel Animations
```typescript
yield* all(
  video(view, bottomLeft, "#FDB827", texts),
  video(view, topRight, "#FDCFE5", texts),
  video(view, bottomRight, "#8BF8A7", texts),
)
```
- All four panels animate simultaneously
- Each panel has unique background color
- Content scales proportionally (0.4 factor)

#### Scaling Factor
All dimensions in quad layout are multiplied by 0.4:
- Font sizes: `180*0.40`, `60*0.40`, `90*0.40`
- Spacing: `150*0.4`
- Padding: `70*0.40`
- Heights and widths scaled accordingly

### 3. Color Schemes

#### Single Template
- Primary background: `#E2FF31` (Bright yellow-green)
- Text boxes: Black background with colored text
- Feature items: White background with black text

#### Quad Layout
- Top-left: `#9accf2` (Light blue)
- Top-right: `#FDCFE5` (Light pink)
- Bottom-left: `#FDB827` (Orange)
- Bottom-right: `#8BF8A7` (Light green)

### 4. Typography and Styling
- Font family: Sans-Serif throughout
- Font weights: 600 for headings
- Text wrapping enabled for main heading
- Dynamic width calculation for text boxes
- Rounded corners (radius: 30) for feature items

## Technical Implementation Details

### 1. Responsive Text Box Sizing
```typescript
width={txtRef().width()+60}  // Adds padding to text width
```
- Dynamically calculates box width based on text content
- Ensures text fits properly with padding

### 2. Vertical Stacking Algorithm
```typescript
yPos = textBoxRef1().y() + textBoxRef1().height()-10;
```
- Calculates next item position based on previous item
- Subtracts 10 pixels for slight overlap/tighter spacing
- Creates visually pleasing vertical rhythm

### 3. Component Hierarchy
```typescript
backgroundRef().add(<Grid ... />)
backgroundRef().add(<Txt ... />)
```
- In quad layout, elements are added to panel containers
- Creates self-contained animated units
- Allows for independent panel animations

### 4. Z-Index Management
- Grid: Default z-index (background)
- Text boxes: `zIndex={0}`
- Text: `zIndex={1}`
- Ensures proper layering of elements

### 5. Animation Timing
- Grid animation: 0.5s expand + 0.1s wait
- Heading: 0.5s fade and scale
- Subtitle typing: 1 second
- Item appearance: 0.5s delays between items
- Total animation: ~4-5 seconds depending on items

## Customization Options

### 1. Configurable Variables
```typescript
useScene().variables.get("backgroundColor", "#E2FF31")
useScene().variables.get("texts", ["events", "gift cards", "and more..."])
```
- Background color can be passed as project variable
- Text items customizable without code changes
- Defaults provided for all variables

### 2. Layout Modifications
- Panel sizes: Change '40%' to adjust quad layout
- Spacing: Modify the 150 pixel grid spacing
- Positions: Adjust calculation factors for different layouts

### 3. Animation Speed
- Modify duration parameters in animations
- Adjust delays between item appearances
- Change typing speed for text reveals

### 4. Visual Styling
- Colors: Update hex values for different branding
- Fonts: Change fontFamily for different typography
- Borders: Modify stroke, lineWidth, and radius values

## Use Cases

### Marketing Applications
1. **Black Friday Sales**: Current implementation
2. **Product Launches**: Highlight features sequentially
3. **Event Promotions**: Showcase event details
4. **Social Media Ads**: Instagram, TikTok, YouTube shorts
5. **Email Campaign Headers**: Animated GIF exports
6. **Website Hero Sections**: Embedded video content
7. **Digital Signage**: In-store promotional displays

### Template Advantages

#### Single Template
- Full-screen impact
- Clear hierarchy of information
- Professional transitions
- Suitable for primary marketing messages

#### Quad Layout
- Multiple messages simultaneously
- Comparison presentations
- Category showcases
- A/B testing different color schemes
- Social media grid posts

## Assets and Resources

### Fonts
- System Sans-Serif font used throughout
- No external font files required
- Ensures consistent rendering across platforms

### Media Files
- No external images or videos used
- All graphics procedurally generated
- Colors and text are the only customizable assets

## Project Settings

### Video Output Configuration
```typescript
settings: {
  shared: {
    size: {x: 1920, y: 1080}  // or new Vector2(1920, 1080)
  }
}
```
- **Resolution**: 1920x1080 (Full HD)
- **Aspect Ratio**: 16:9
- **Frame Rate**: Determined by Revideo defaults
- **Duration**: Approximately 4-5 seconds

## Project Execution

### Development Mode
Run `npm start` to:
1. Launch Revideo editor interface
2. Load project from `./src/project.tsx`
3. Preview animations in real-time
4. Adjust colors and text interactively
5. Test different configurations

### Rendering Mode
Run `npm run render` to:
1. Compile TypeScript files to JavaScript
2. Execute `dist/render.js`
3. Generate final marketing video
4. Output file path to console
5. Ready for upload to social media

## Performance Considerations

### Single Template
- Simple component hierarchy
- Efficient sequential animations
- Minimal memory usage
- Fast rendering times

### Quad Layout
- Four parallel animation tracks
- Higher memory usage for multiple panels
- Synchronization overhead
- Still maintains good performance

### Optimization Tips
- Reuse components when possible
- Minimize complex calculations in render loops
- Use appropriate quality settings for target platform
- Consider file size for social media uploads

## Summary

The marketing templates demonstrate professional animated marketing content creation with Revideo. Key strengths include:

1. **Professional Design**: Modern grid effects and smooth animations create polished marketing content
2. **Flexibility**: Configurable colors, text, and layouts for brand customization
3. **Reusability**: Template structure allows easy adaptation for different campaigns
4. **Performance**: Efficient animations suitable for social media platforms
5. **Scalability**: Quad layout shows how to create complex multi-panel presentations
6. **Code Organization**: Clean separation of animation logic and helper functions

The implementation showcases Revideo's capability to create engaging marketing content with minimal code, making it ideal for:
- Rapid campaign development
- A/B testing different designs
- Consistent brand animations
- Multi-platform content generation
- Automated marketing video creation

These templates serve as excellent starting points for creating professional marketing animations that capture attention and deliver messages effectively across digital platforms.