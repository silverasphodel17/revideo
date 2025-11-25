# Revideo Rive Explanation Video - Complete Documentation

## Overview

This documentation provides a comprehensive analysis of the rive-explanation-video project, which demonstrates how to integrate Rive animations with Revideo to create an educational video explaining the usage of the `<Rive />` component. The project showcases live code editing animations, syntax highlighting, and the seamless integration of Rive's runtime animations within a Revideo project, culminating in a professional logo animation sequence.

## File Structure

```
rive-explanation-video/
├── src/
│   ├── global.css          # Global CSS with Lexend font import
│   ├── project.tsx         # Main animation scene with Rive integration
│   └── render.ts           # Video rendering script
├── .DS_Store              # macOS system file
├── package.json           # Project dependencies and scripts
├── README.md              # Project overview with video demo
└── tsconfig.json         # TypeScript configuration
```

## Dependencies and Configuration

### Core Dependencies
- **@revideo/2d**: 0.10.3 - 2D components including Rive support
- **@revideo/core**: 0.10.3 - Core Revideo functionality
- **@revideo/renderer**: 0.10.3 - Video rendering capabilities
- **@lezer/javascript**: ^1.4.18 - JavaScript parser for syntax highlighting

### Development Dependencies
- **@revideo/cli**: 0.10.3 - Command-line interface for Revideo
- **@revideo/ui**: 0.10.3 - User interface for the Revideo editor
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
import { waitFor, createRef, makeProject, all, Reference, chain } from '@revideo/core';
import {
  Img,
  makeScene2D,
  Rive,
  Txt,
  Layout,
  LezerHighlighter, Code, word, lines, Rect,
  View2D,
} from '@revideo/2d';
import {parser} from '@lezer/javascript';
import {tags} from '@lezer/highlight';
import {HighlightStyle} from '@codemirror/language';
import './global.css';

const MyStyle = HighlightStyle.define([
    {
      tag: tags.comment,
      color: '#8E908C',
    },
    {
      tag: [tags.variableName, tags.self, tags.propertyName, tags.attributeName, tags.regexp],
      color: '#C82829',
    },
    {
      tag: [tags.number, tags.bool, tags.null],
      color: '#F5871F',
    },
    {
      tag: [tags.className, tags.typeName, tags.definition(tags.typeName)],
      color: '#C99E00',
    },
    {
      tag: [tags.string, tags.special(tags.brace)],
      color: '#718C00',
    },
    {
      tag: tags.operator,
      color: '#3E999F',
    },
    {
      tag: [tags.definition(tags.propertyName), tags.function(tags.variableName)],
      color: '#4271AE',
    },
    {
      tag: tags.keyword,
      color: '#8959A8',
    },
    {
      tag: tags.derefOperator,
      color: '#4D4D4C',
    },
    {
      tag: tags.bracket,
      color: "grey"
    },
    {
      tag: tags.separator,
      color: "grey"
    },
    {
      tag: tags.punctuation,
      color: "grey"
    },
    {
      tag: tags.typeOperator,
      color: "grey"
    }
  ]);

Code.defaultHighlighter = new LezerHighlighter(
  parser.configure({
    dialect: 'jsx ts',
  }),
  MyStyle
);

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {

  const textRef = createRef<Txt>();
  yield view.add(
    <Txt fontFamily={"Lexend"} fontSize={100} fontWeight={600} ref={textRef}/>
  );

  yield* all(textRef().text("Rive Animations in Revideo", 1.25))
  yield* all(
      textRef().scale(0.75, 0.75),
      textRef().position.y(-400, 0.75)
  )
  yield* waitFor(0.25);

  const codeRef= createRef<Code>();
  const videoBoxRef = createRef<Layout>();
  view.add(<Layout opacity={0} position={[550, 100]} size={[600, 600]} ref={videoBoxRef} />);
  view.add(
    <Code
    ref={codeRef}
    textAlign={"left"}
    fontSize={35}
    x={-400}
    y={100}
    opacity={0}
    fontFamily={'JetBrains Mono, monospace'}
    code={
`export default makeScene2D(function* (view){
  yield view.add(
      <Img
        src={"/sf.png"}
        size={["100%", "100%"]}
      />
  );

});
`
}/>);

videoBoxRef().add(
  <Img
    src={"https://revideo-example-assets.s3.amazonaws.com/sf.png"}
    size={[videoBoxRef().width(), videoBoxRef().height()]}
  />
);

yield* all(
  codeRef().opacity(1, 0.75),
  videoBoxRef().opacity(1, 0.75)
);
yield* waitFor(0.5);

const riveRef = createRef<Rive>();
videoBoxRef().add(
    <Rive
      src={"https://revideo-example-assets.s3.amazonaws.com/emoji.riv"}
      animationId={1}
      size={[600, 600]}
      opacity={0}
      ref={riveRef}
    />
);
yield* all(codeRef().code.insert([8, 0],
  `\n   yield view.add(
      <Rive
        src={"/emoji.riv"}
        animationId={1}
        size={[600, 600]}
      />
  );\n`, 1),
  riveRef().opacity(1, 1)
);

yield* waitFor(1);

yield* codeRef().selection(lines(11, 13), 1),
yield replaceRive(videoBoxRef, riveRef)
yield* all(
  codeRef().code.replace(word(11, 17, 5), "dog", 2),
  codeRef().code.replace(word(13, 17, 3), "900", 2),
);

yield* codeRef().selection(lines(0, 17), 1);

yield* waitFor(1);
yield* all(
  codeRef().opacity(0, 1),
  videoBoxRef().opacity(0, 1),
  textRef().opacity(0, 1)
);

yield* waitFor(0.5);
yield* logoAnimation(view);
});

function* replaceRive(box: Reference<Layout>, rive: Reference<Rive>){
yield* rive().opacity(0, 1);
rive().remove();

const riveRef = createRef<Rive>();
box().add(
    <Rive
      src={"https://revideo-example-assets.s3.amazonaws.com/dog.riv"}
      size={[1000, 600]}
      opacity={0}
      animationId={1}
      ref={riveRef}
    />
);

yield* riveRef().opacity(1, 1);
yield* waitFor(5);
}

function* logoAnimation(view: View2D){
const block1 = createRef<Rect>()
const block2 = createRef<Rect>()
const block3 = createRef<Rect>()
const blocks = createRef<Layout>();
const logoText = createRef<Txt>();
view.add(
  <>
  <Layout ref={blocks} x={-600}>
    <Rect fill={"#151515"} height={60} y={-80} x={-620} width={180} radius={9.3} ref={block1} />
    <Rect fill={"#151515"} height={60} width={180} x={-540} radius={9.3} ref={block2} />
    <Rect fill={"#151515"} height={60} width={180} x={-460} y={80} radius={9.3} ref={block3} />
    <Txt ref={logoText} fontFamily={"Lexend"} x={53+757.5} y={20} fontSize={300} letterSpacing={-4} fontWeight={700} text={""}/>
  </Layout>
  </>
);

yield* all(
  block1().position.x(-80, 0.5),
  chain(waitFor(0.1), block2().position.x(0, 0.4)),
  chain(waitFor(0.2), block3().position.x(80, 0.3)),
)

yield* logoText().text("revideo", 1);

yield* waitFor(1);
}

/**
 * The final revideo project
 */
export default makeProject({
  scenes: [scene],
  settings: {
    // Example settings:
    shared: {
      size: {x: 1920, 1080},
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

## Key Concepts and Patterns

### 1. Rive Component Integration

#### Basic Rive Component Usage
```typescript
<Rive
  src={"https://revideo-example-assets.s3.amazonaws.com/emoji.riv"}
  animationId={1}
  size={[600, 600]}
  opacity={0}
  ref={riveRef}
/>
```
- **src**: URL or path to the Rive animation file (.riv)
- **animationId**: Specifies which animation to play from the Rive file
- **size**: Dimensions of the animation canvas
- **opacity**: Transparency control for fade effects
- **ref**: Reference for programmatic control

### 2. Code Syntax Highlighting

#### Custom Highlight Style Definition
```typescript
const MyStyle = HighlightStyle.define([
  { tag: tags.comment, color: '#8E908C' },
  { tag: [tags.variableName, tags.self, tags.propertyName], color: '#C82829' },
  { tag: [tags.number, tags.bool, tags.null], color: '#F5871F' },
  { tag: [tags.className, tags.typeName], color: '#C99E00' },
  { tag: [tags.string], color: '#718C00' },
  { tag: tags.operator, color: '#3E999F' },
  { tag: [tags.function(tags.variableName)], color: '#4271AE' },
  { tag: tags.keyword, color: '#8959A8' },
  // Additional styling...
]);
```
- Uses Lezer parser for JavaScript/TypeScript/JSX
- Custom color scheme for different code elements
- Professional IDE-like syntax highlighting

#### Code Component Configuration
```typescript
Code.defaultHighlighter = new LezerHighlighter(
  parser.configure({
    dialect: 'jsx ts',
  }),
  MyStyle
);
```

### 3. Live Code Editing Animation

#### Code Insertion
```typescript
yield* all(codeRef().code.insert([8, 0],
  `\n   yield view.add(
      <Rive
        src={"/emoji.riv"}
        animationId={1}
        size={[600, 600]}
      />
  );\n`, 1)
);
```
- **[8, 0]**: Position (line 8, column 0)
- Animated text insertion over 1 second
- Simulates live coding

#### Code Replacement
```typescript
yield* all(
  codeRef().code.replace(word(11, 17, 5), "dog", 2),
  codeRef().code.replace(word(13, 17, 3), "900", 2),
);
```
- **word(line, column, length)**: Selects specific word
- Animated replacement over 2 seconds
- Multiple simultaneous replacements

#### Code Selection
```typescript
yield* codeRef().selection(lines(11, 13), 1)
```
- Highlights specific lines
- Visual feedback for code being modified
- Guides viewer attention

### 4. Animation Sequencing

#### Scene Flow Structure
1. **Title Introduction** (0-2.25s)
2. **Code and Preview Setup** (2.25-3.5s)
3. **Rive Component Addition** (3.5-5.5s)
4. **Animation Modification** (5.5-12.5s)
5. **Scene Cleanup** (12.5-14s)
6. **Logo Animation** (14-17s)

## Technical Implementation Details

### 1. Dynamic Component Replacement

```typescript
function* replaceRive(box: Reference<Layout>, rive: Reference<Rive>){
  yield* rive().opacity(0, 1);  // Fade out
  rive().remove();               // Remove from DOM

  const riveRef = createRef<Rive>();
  box().add(                     // Add new Rive component
    <Rive
      src={"https://revideo-example-assets.s3.amazonaws.com/dog.riv"}
      size={[1000, 600]}
      opacity={0}
      animationId={1}
      ref={riveRef}
    />
  );

  yield* riveRef().opacity(1, 1); // Fade in
  yield* waitFor(5);              // Let animation play
}
```
- Smooth transition between different Rive animations
- Proper cleanup of previous component
- Memory management through removal

### 2. Side-by-Side Layout

```typescript
view.add(<Layout opacity={0} position={[550, 100]} size={[600, 600]} ref={videoBoxRef} />);
view.add(
  <Code
    ref={codeRef}
    x={-400}
    y={100}
    // ...
  />
);
```
- Code editor on left (-400 x position)
- Preview box on right (550 x position)
- Synchronized opacity animations

### 3. Logo Animation Sequence

```typescript
function* logoAnimation(view: View2D){
  // Create three animated blocks
  const block1 = createRef<Rect>()
  const block2 = createRef<Rect>()
  const block3 = createRef<Rect>()

  // Staggered animation
  yield* all(
    block1().position.x(-80, 0.5),
    chain(waitFor(0.1), block2().position.x(0, 0.4)),
    chain(waitFor(0.2), block3().position.x(80, 0.3)),
  )

  yield* logoText().text("revideo", 1);
}
```
- Three rectangles forming abstract logo
- Staggered entrance using `chain` and `waitFor`
- Text reveal animation

### 4. Animation Techniques

#### Parallel Animations
```typescript
yield* all(
  textRef().scale(0.75, 0.75),
  textRef().position.y(-400, 0.75)
)
```
- Multiple properties animated simultaneously
- Creates smooth, professional transitions

#### Sequential Animations
```typescript
yield* textRef().text("Rive Animations in Revideo", 1.25)
yield* all(/*...*/)
yield* waitFor(0.25);
```
- Clear progression through content
- Pacing for viewer comprehension

## Assets and Resources

### Remote Assets

#### Images
- **sf.png**: San Francisco skyline image
  - Source: `https://revideo-example-assets.s3.amazonaws.com/sf.png`
  - Used as initial example image

#### Rive Animation Files
1. **emoji.riv**: Emoji animation
   - Source: `https://revideo-example-assets.s3.amazonaws.com/emoji.riv`
   - First Rive animation shown

2. **dog.riv**: Dog animation
   - Source: `https://revideo-example-assets.s3.amazonaws.com/dog.riv`
   - Replacement animation demonstrating dynamic updates

### Fonts

#### Lexend
- **Source**: Google Fonts CDN
- **Weights**: 100-900 (variable)
- **Usage**: Main UI text and logo

#### JetBrains Mono
- **Usage**: Code editor display
- **Style**: Monospace font for code readability

## Rive Integration Patterns

### 1. Basic Integration
```typescript
<Rive
  src={"path/to/animation.riv"}
  animationId={1}
  size={[width, height]}
/>
```
- Minimal setup required
- Automatic playback on load
- Respects animation timeline

### 2. Opacity Control
```typescript
const riveRef = createRef<Rive>();
// ...
yield* riveRef().opacity(0, 1);  // Fade out
yield* riveRef().opacity(1, 1);  // Fade in
```
- Smooth transitions
- Non-destructive hiding
- Maintains animation state

### 3. Dynamic Replacement
- Remove old Rive component
- Add new component with different source
- Seamless transition with opacity animations

### 4. Size Adaptation
```typescript
size={[1000, 600]}  // Dog animation
size={[600, 600]}   // Emoji animation
```
- Different sizes for different animations
- Maintains aspect ratio
- Responsive to container

## Educational Design Patterns

### 1. Progressive Disclosure
- Start with simple image display
- Introduce Rive component
- Show property modifications
- Demonstrate real-time updates

### 2. Visual Feedback
- Code highlighting for focus
- Synchronized preview updates
- Clear cause-and-effect relationship

### 3. Practical Example
- Real code that viewers can use
- Common use case (replacing static image with animation)
- Shows both code and result

### 4. Professional Presentation
- Clean typography (Lexend font)
- Syntax highlighting for readability
- Smooth animations throughout
- Branded ending with logo

## Animation Timeline

```
Time  | Action
------|----------------------------------------------------------
0.0s  | Title appears: "Rive Animations in Revideo"
1.25s | Title scales down and moves up
2.0s  | Code editor and preview box fade in
2.5s  | Initial image visible in preview
3.5s  | Rive code insertion begins
4.5s  | Emoji animation appears
5.5s  | Code selection highlights Rive parameters
6.5s  | Begin replacing emoji with dog
7.5s  | Code updates: "emoji" → "dog", size changes
8.5s  | Dog animation appears
13.5s | Everything fades out
14.0s | Logo animation begins
15.5s | "revideo" text appears
17.0s | Animation complete
```

## Use Cases

### 1. Tutorial Videos
- Teaching Rive integration
- Demonstrating animation capabilities
- Code-along tutorials

### 2. Documentation
- Visual API documentation
- Feature demonstrations
- Integration guides

### 3. Marketing Content
- Showcasing Revideo capabilities
- Rive partnership announcements
- Feature release videos

### 4. Educational Content
- Animation courses
- Web development tutorials
- Interactive design lessons

## Customization Options

### 1. Different Rive Animations
```typescript
src={"path/to/your-animation.riv"}
animationId={2}  // Different animation from the file
```

### 2. Code Examples
- Change the demonstrated code
- Show different Rive properties
- Add more complex examples

### 3. Styling
- Modify syntax highlighting colors
- Change fonts and sizes
- Adjust layout positioning

### 4. Pacing
- Adjust animation durations
- Add more wait times
- Speed up or slow down code typing

## Performance Considerations

### 1. Asset Loading
- Remote assets loaded asynchronously
- Rive files cached after first load
- Consider CDN for production

### 2. Animation Performance
- Rive animations hardware-accelerated
- Efficient runtime rendering
- Minimal CPU usage

### 3. Memory Management
- Components removed when not needed
- References cleaned up
- Prevents memory leaks

## Best Practices Demonstrated

### 1. Component References
```typescript
const riveRef = createRef<Rive>();
const codeRef = createRef<Code>();
```
- Typed references for type safety
- Clean component access
- Enables precise control

### 2. Animation Composition
```typescript
yield* all(
  animation1(),
  animation2()
);
```
- Parallel animations for efficiency
- Clear animation relationships
- Maintainable code structure

### 3. Code Organization
- Separate functions for complex sequences
- Clear variable naming
- Logical flow structure

### 4. Resource Management
- Remote asset hosting
- Proper component lifecycle
- Cleanup after use

## Summary

The Rive Explanation Video project demonstrates a sophisticated integration of Rive animations within Revideo, showcasing:

1. **Technical Excellence**: Clean integration of Rive runtime with Revideo's animation system
2. **Educational Design**: Clear, progressive teaching of concepts with visual feedback
3. **Professional Quality**: Syntax highlighting, smooth animations, and polished presentation
4. **Practical Application**: Real-world code examples that developers can use
5. **Advanced Features**: Live code editing, dynamic component replacement, and custom styling

The implementation serves as both a technical demonstration and an educational resource, showing how Revideo can be used to create engaging programming tutorials and documentation videos. The combination of code visualization, live preview, and animated graphics creates a compelling learning experience that effectively communicates complex technical concepts.