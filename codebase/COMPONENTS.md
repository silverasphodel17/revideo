# REVIDEO COMPONENTS API REFERENCE

## Component Hierarchy

```
Node (base class)
├── Layout
│   ├── Shape
│   │   ├── Rect
│   │   │   ├── Media
│   │   │   │   ├── Video
│   │   │   │   └── Audio
│   │   │   └── Img
│   │   │       └── Latex
│   │   ├── Circle
│   │   ├── Line
│   │   ├── Polygon
│   │   ├── Curve
│   │   │   ├── Bezier
│   │   │   ├── QuadBezier
│   │   │   ├── CubicBezier
│   │   │   └── Spline
│   │   ├── Path
│   │   ├── Txt
│   │   │   └── TxtLeaf
│   │   └── Code (CodeBlock)
│   ├── Icon
│   ├── SVG
│   └── Rive
├── Grid
├── Ray
├── Knot
└── View2D (root component)
```

## Base Component: Node

The fundamental building block of all visual components in Revideo.

### Node Properties

```typescript
// packages/2d/src/lib/components/Node.ts
@nodeName('Node')
export class Node implements Promisable<Node> {
  // Transform Properties
  @vector2Signal('position')
  @initial([0, 0])
  public declare readonly position: Vector2Signal<this>;

  @signal()
  public declare readonly x: SimpleVector2Signal<this>; // x component of position

  @signal()
  public declare readonly y: SimpleVector2Signal<this>; // y component of position

  @initial(0)
  @signal()
  public declare readonly rotation: SimpleSignal<number, this>;

  @vector2Signal('scale')
  @initial([1, 1])
  public declare readonly scale: Vector2Signal<this>;

  @signal()
  public declare readonly scaleX: SimpleSignal<number, this>;

  @signal()
  public declare readonly scaleY: SimpleSignal<number, this>;

  @vector2Signal('skew')
  @initial([0, 0])
  public declare readonly skew: Vector2Signal<this>;

  @signal()
  public declare readonly skewX: SimpleSignal<number, this>;

  @signal()
  public declare readonly skewY: SimpleSignal<number, this>;

  @initial(0)
  @signal()
  public declare readonly zIndex: SimpleSignal<number, this>;

  // Visual Properties
  @signal()
  public declare readonly opacity: SimpleSignal<number, this>;

  @initial(null)
  @colorSignal()
  public declare readonly shadowColor: ColorSignal<this>;

  @initial(0)
  @signal()
  public declare readonly shadowBlur: SimpleSignal<number, this>;

  @vector2Signal('shadowOffset')
  public declare readonly shadowOffset: Vector2Signal<this>;

  @signal()
  public declare readonly shadowOffsetX: SimpleSignal<number, this>;

  @signal()
  public declare readonly shadowOffsetY: SimpleSignal<number, this>;

  @filtersSignal()
  public declare readonly filters: FiltersSignal<this>;

  // Layout Properties
  @initial(false)
  @signal()
  public declare readonly cache: SimpleSignal<boolean, this>;

  @spacingSignal('cachePadding')
  public declare readonly cachePadding: SpacingSignal<this>;

  @initial(false)
  @signal()
  public declare readonly composite: SimpleSignal<boolean, this>;

  @initial('source-over')
  @signal()
  public declare readonly compositeOperation: SimpleSignal<GlobalCompositeOperation, this>;

  @initial({x: 0.5, y: 0.5})
  @signal()
  public declare readonly offset: Vector2Signal<this>;

  @signal()
  public declare readonly absolutePosition: SimpleVector2Signal<this>;

  @signal()
  public declare readonly absoluteRotation: SimpleSignal<number, this>;

  @signal()
  public declare readonly absoluteScale: SimpleVector2Signal<this>;
}
```

### Node Methods

```typescript
// Transform Methods
public localToWorld(): DOMMatrix;
public worldToLocal(point: PossibleVector2): Vector2;
public localToParent(point: PossibleVector2): Vector2;
public parentToLocal(point: PossibleVector2): Vector2;

// Children Management
public add(node: ComponentChildren): this;
public insert(node: ComponentChildren, index?: number): this;
public remove(): this;
public removeChildren(): this;
public moveTo(target: Node, index?: number): this;
public moveUp(): this;
public moveDown(): this;
public moveToTop(): this;
public moveToBottom(): this;
public moveAbove(target: Node): this;
public moveBelow(target: Node): this;

// Query Methods
public children(): Node[];
public parent(): Node | null;
public findAll<T extends Node>(predicate: (node: Node) => node is T): T[];
public findFirst<T extends Node>(predicate: (node: Node) => node is T): T | null;
public findLast<T extends Node>(predicate: (node: Node) => node is T): T | null;
public findAncestor<T extends Node>(predicate: (node: Node) => node is T): T | null;

// Type-safe Child Access
public childAs<T extends Node>(index: number): T | null;
public childrenAs<T extends Node>(range?: [number, number]): T[];
public parentAs<T extends Node>(): T | null;

// State Management
public getState(): NodeState;
public applyState(state: NodeState): void;
public *restore(duration?: number, timingFunction?: TimingFunction): ThreadGenerator;
public save(): this;

// Rendering
public render(context: CanvasRenderingContext2D): void;
public hit(point: Vector2): Node | null;

// Lifecycle
public dispose(): void;
public toPromise(): Promise<this>;
```

### Node Example Usage

```typescript
// packages/examples/src/scenes/layout.tsx
import {makeScene2D, Node} from '@revideo/2d';

export default makeScene2D(function* (view) {
  const node = createRef<Node>();

  view.add(
    <Node
      ref={node}
      position={[-400, 0]}
      rotation={45}
      scale={1.5}
      opacity={0.5}
    />
  );

  // Animate transform properties
  yield* all(
    node().position.x(400, 2),
    node().rotation(360, 2),
    node().scale(2, 1),
    node().opacity(1, 1),
  );

  // Use filters
  yield* node().filters.blur(10, 1);
  yield* node().filters.brightness(1.5, 0.5);
});
```

## Layout Component

Extends Node with flexbox layout capabilities.

### Layout Properties

```typescript
// packages/2d/src/lib/components/Layout.ts
export class Layout extends Node {
  // Size Properties
  @vector2Signal('size')
  public declare readonly size: Vector2LengthSignal<this>;

  @signal()
  public declare readonly width: SimpleSignal<Length, this>;

  @signal()
  public declare readonly height: SimpleSignal<Length, this>;

  @initial('row')
  @signal()
  public declare readonly direction: SimpleSignal<FlexDirection, this>;

  @initial('nowrap')
  @signal()
  public declare readonly wrap: SimpleSignal<FlexWrap, this>;

  @initial('normal')
  @signal()
  public declare readonly alignContent: SimpleSignal<FlexContent, this>;

  @initial('stretch')
  @signal()
  public declare readonly alignItems: SimpleSignal<FlexItems, this>;

  @initial('auto')
  @signal()
  public declare readonly alignSelf: SimpleSignal<FlexItems, this>;

  @initial('start')
  @signal()
  public declare readonly justifyContent: SimpleSignal<FlexContent, this>;

  @initial(0)
  @signal()
  public declare readonly grow: SimpleSignal<number, this>;

  @initial(1)
  @signal()
  public declare readonly shrink: SimpleSignal<number, this>;

  @initial(null)
  @signal()
  public declare readonly basis: SimpleSignal<FlexBasis, this>;

  @initial(0)
  @signal()
  public declare readonly gap: Vector2LengthSignal<this>;

  @spacingSignal('margin')
  public declare readonly margin: SpacingSignal<this>;

  @spacingSignal('padding')
  public declare readonly padding: SpacingSignal<this>;

  @initial('content-box')
  @signal()
  public declare readonly layout: SimpleSignal<LayoutMode, this>;

  @initial(false)
  @signal()
  public declare readonly clip: SimpleSignal<boolean, this>;

  // Ratio Properties
  @initial(null)
  @signal()
  public declare readonly ratio: SimpleSignal<number | null, this>;

  // Font Properties
  @signal()
  public declare readonly fontFamily: SimpleSignal<string, this>;

  @signal()
  public declare readonly fontSize: SimpleSignal<number, this>;

  @signal()
  public declare readonly fontStyle: SimpleSignal<string, this>;

  @signal()
  public declare readonly fontWeight: SimpleSignal<number, this>;

  @signal()
  public declare readonly lineHeight: SimpleSignal<Length, this>;

  @signal()
  public declare readonly letterSpacing: SimpleSignal<number, this>;

  @signal()
  public declare readonly textAlign: SimpleSignal<CanvasTextAlign, this>;

  @signal()
  public declare readonly textDirection: SimpleSignal<CanvasDirection, this>;

  // Computed Properties
  @computed()
  public computedSize(): BBox {
    return this.getComputedLayout();
  }

  @computed()
  public computedPosition(): Vector2 {
    return this.getComputedLayout().position;
  }
}
```

### Layout Example

```typescript
// packages/examples/src/scenes/layout.tsx
export default makeScene2D(function* (view) {
  view.add(
    <Layout
      direction={'column'}
      width={600}
      height={400}
      gap={20}
      padding={20}
      alignItems={'center'}
      justifyContent={'space-between'}
    >
      <Rect width={100} height={100} fill={'#88C0D0'} />
      <Rect width={100} height={100} fill={'#81A1C1'} grow={1} />
      <Rect width={100} height={100} fill={'#5E81AC'} />
    </Layout>
  );
});
```

## Shape Component

Base class for all shape components with stroke and fill properties.

### Shape Properties

```typescript
// packages/2d/src/lib/components/Shape.ts
export class Shape extends Layout {
  @initial(null)
  @canvasStyleSignal()
  public declare readonly fill: CanvasStyleSignal<this>;

  @initial(null)
  @canvasStyleSignal()
  public declare readonly stroke: CanvasStyleSignal<this>;

  @initial(false)
  @signal()
  public declare readonly strokeFirst: SimpleSignal<boolean, this>;

  @initial(0)
  @signal()
  public declare readonly lineWidth: SimpleSignal<number, this>;

  @initial('butt')
  @signal()
  public declare readonly lineCap: SimpleSignal<CanvasLineCap, this>;

  @initial(0)
  @signal()
  public declare readonly lineDash: SimpleSignal<number[], this>;

  @initial(0)
  @signal()
  public declare readonly lineDashOffset: SimpleSignal<number, this>;

  @initial('miter')
  @signal()
  public declare readonly lineJoin: SimpleSignal<CanvasLineJoin, this>;

  @initial(true)
  @signal()
  public declare readonly antialiased: SimpleSignal<boolean, this>;

  // Computed Properties
  @computed()
  public getPath(): Path2D {
    // Returns the shape's path
  }
}
```

## Curve Component

Base class for curve components with start/end clipping.

### Curve Properties

```typescript
// packages/2d/src/lib/components/Curve.ts
@nodeName('Curve')
export class Curve extends Shape {
  @initial(false)
  @signal()
  public declare readonly closed: SimpleSignal<boolean, this>;

  @initial(0)
  @signal()
  public declare readonly start: SimpleSignal<number, this>;

  @initial(0)
  @signal()
  public declare readonly startOffset: SimpleSignal<number, this>;

  @initial(false)
  @signal()
  public declare readonly startArrow: SimpleSignal<boolean, this>;

  @initial(1)
  @signal()
  public declare readonly end: SimpleSignal<number, this>;

  @initial(0)
  @signal()
  public declare readonly endOffset: SimpleSignal<number, this>;

  @initial(false)
  @signal()
  public declare readonly endArrow: SimpleSignal<boolean, this>;

  @initial(24)
  @signal()
  public declare readonly arrowSize: SimpleSignal<number, this>;
}
```

## Rect Component

Rectangle shape with rounded corners.

### Rect Properties

```typescript
// packages/2d/src/lib/components/Rect.ts
export class Rect extends Shape {
  @spacingSignal('radius')
  public declare readonly radius: SpacingSignal<this>;

  @initial(0)
  @signal()
  public declare readonly smoothCorners: SimpleSignal<boolean, this>;

  @initial(0.6)
  @signal()
  public declare readonly cornerSharpness: SimpleSignal<number, this>;

  protected getPath(): Path2D {
    const path = new Path2D();
    const {width, height} = this.computedSize();
    const radius = this.radius();

    if (this.smoothCorners()) {
      // Draw smooth corners using squircle algorithm
      this.drawSmoothCorners(path, width, height, radius);
    } else {
      // Draw standard rounded rectangle
      this.drawRoundRect(path, width, height, radius);
    }

    return path;
  }
}
```

### Rect Example

```typescript
// packages/examples/src/scenes/components.tsx
export default makeScene2D(function* (view) {
  const rect = createRef<Rect>();

  view.add(
    <Rect
      ref={rect}
      width={300}
      height={200}
      fill={'#88C0D0'}
      stroke={'#5E81AC'}
      lineWidth={4}
      radius={20}
      smoothCorners={true}
      cornerSharpness={0.6}
    />
  );

  // Animate corner radius
  yield* rect().radius(50, 1);

  // Animate fill gradient
  yield* rect().fill(
    new Gradient({
      type: 'linear',
      from: [0, -100],
      to: [0, 100],
      stops: [
        {offset: 0, color: '#88C0D0'},
        {offset: 1, color: '#5E81AC'},
      ],
    }),
    1,
  );
});
```

## Circle Component

Circle shape component.

### Circle Properties

```typescript
// packages/2d/src/lib/components/Circle.ts
export class Circle extends Shape {
  @initial(0)
  @signal()
  public declare readonly startAngle: SimpleSignal<number, this>;

  @initial(360)
  @signal()
  public declare readonly endAngle: SimpleSignal<number, this>;

  @initial(false)
  @signal()
  public declare readonly closed: SimpleSignal<boolean, this>;

  @initial(false)
  @signal()
  public declare readonly counterclockwise: SimpleSignal<boolean, this>;

  protected getPath(): Path2D {
    const path = new Path2D();
    const {width, height} = this.computedSize();
    const radius = Math.min(width, height) / 2;

    path.arc(
      0,
      0,
      radius,
      (this.startAngle() * Math.PI) / 180,
      (this.endAngle() * Math.PI) / 180,
      this.counterclockwise(),
    );

    if (this.closed()) {
      path.closePath();
    }

    return path;
  }
}
```

### Circle Example

```typescript
// Arc animation example
export default makeScene2D(function* (view) {
  const circle = createRef<Circle>();

  view.add(
    <Circle
      ref={circle}
      width={200}
      height={200}
      stroke={'#88C0D0'}
      lineWidth={8}
      startAngle={0}
      endAngle={0}
    />
  );

  // Animate arc drawing
  yield* circle().endAngle(360, 2);

  // Create pie chart segment
  yield* all(
    circle().fill('#88C0D0', 0.5),
    circle().closed(true, 0.5),
  );
});
```

## Line Component

Line shape for drawing straight or curved lines.

### Line Properties

```typescript
// packages/2d/src/lib/components/Line.ts
export class Line extends Shape {
  @signal()
  public declare readonly points: SimpleSignal<SignalValue<Vector2>[] | Vector2Signal<any>[], this>;

  @initial(0)
  @signal()
  public declare readonly radius: SimpleSignal<number, this>;

  @initial(false)
  @signal()
  public declare readonly closed: SimpleSignal<boolean, this>;

  @initial('none')
  @signal()
  public declare readonly startArrow: SimpleSignal<boolean | ArrowInfo, this>;

  @initial('none')
  @signal()
  public declare readonly endArrow: SimpleSignal<boolean | ArrowInfo, this>;

  @initial(0)
  @signal()
  public declare readonly arrowSize: SimpleSignal<number, this>;

  public getPointAtPercentage(percentage: number): Vector2 {
    // Returns point at given percentage along the line
  }

  protected getPath(): Path2D {
    const path = new Path2D();
    const points = this.parsedPoints();

    if (points.length === 0) return path;

    path.moveTo(points[0].x, points[0].y);
    for (let i = 1; i < points.length; i++) {
      path.lineTo(points[i].x, points[i].y);
    }

    if (this.closed()) {
      path.closePath();
    }

    return path;
  }
}
```

### Line Example

```typescript
export default makeScene2D(function* (view) {
  const line = createRef<Line>();

  view.add(
    <Line
      ref={line}
      points={[
        [-200, 0],
        [0, -100],
        [200, 0],
      ]}
      stroke={'#88C0D0'}
      lineWidth={4}
      radius={20}
      endArrow
      arrowSize={30}
      end={0}
    />
  );

  // Animate line drawing
  yield* line().end(1, 2);

  // Animate points
  yield* line().points([
    [-200, 100],
    [0, -150],
    [200, 100],
  ], 1);
});
```

## Polygon Component

Shape for drawing regular polygons.

### Polygon Properties

```typescript
// packages/2d/src/lib/components/Polygon.ts
export class Polygon extends Shape {
  @initial(6)
  @signal()
  public declare readonly sides: SimpleSignal<number, this>;

  protected getPath(): Path2D {
    const path = new Path2D();
    const sides = this.sides();
    const {width, height} = this.computedSize();
    const radius = Math.min(width, height) / 2;

    for (let i = 0; i < sides; i++) {
      const angle = (i * 2 * Math.PI) / sides - Math.PI / 2;
      const x = radius * Math.cos(angle);
      const y = radius * Math.sin(angle);

      if (i === 0) {
        path.moveTo(x, y);
      } else {
        path.lineTo(x, y);
      }
    }

    path.closePath();
    return path;
  }
}
```

## Txt Component

Text rendering component that extends Shape. Font properties are inherited from Layout.

### Txt Properties

```typescript
// packages/2d/src/lib/components/Txt.ts
export class Txt extends Shape {
  @initial('')
  @signal()
  public declare readonly text: SimpleSignal<string, this>;

  // Font properties inherited from Layout:
  // fontFamily: SimpleSignal<string, this>;
  // fontSize: SimpleSignal<number, this>;
  // fontWeight: SimpleSignal<number, this>;
  // fontStyle: SimpleSignal<string, this>;
  // lineHeight: SimpleSignal<Length, this>;
  // letterSpacing: SimpleSignal<number, this>;

  @initial('left')
  @signal()
  public declare readonly textAlign: SimpleSignal<CanvasTextAlign, this>;

  @initial('pre-wrap')
  @signal()
  public declare readonly textWrap: SimpleSignal<TextWrap, this>;

  @computed()
  public parsedText(): string {
    // Processes text with potential formatting
    return this.text();
  }
}
```

### Txt Example

```typescript
export default makeScene2D(function* (view) {
  const text = createRef<Txt>();

  view.add(
    <Txt
      ref={text}
      text={''}
      fontSize={72}
      fontWeight={700}
      fill={'#88C0D0'}
      fontFamily={'JetBrains Mono'}
      letterSpacing={2}
    />
  );

  // Animate text typing
  yield* text().text('Hello, Revideo!', 2);

  // Animate color change
  yield* text().fill('#5E81AC', 1);

  // Animate letter spacing
  yield* text().letterSpacing(10, 1);
});
```

## Code Component

Syntax-highlighted code display with animation support.

### Code Properties

```typescript
// packages/2d/src/lib/components/Code.ts
export interface CodeProps extends ShapeProps {
  language?: SignalValue<string>;
  code?: SignalValue<PossibleCodeScope>;
  theme?: SignalValue<CodeTheme>;
  highlighter?: SignalValue<CodeHighlighter>;
}

@nodeName('CodeBlock')
export class Code extends Shape {
  @initial('tsx')
  @signal()
  public declare readonly language: SimpleSignal<string, this>;

  @codeSignal()
  public declare readonly code: CodeSignal<this>;

  @initial(DefaultHighlighter)
  @signal()
  public declare readonly highlighter: SimpleSignal<CodeHighlighter, this>;

  @initial(dracula)
  @signal()
  public declare readonly theme: SimpleSignal<CodeTheme, this>;

  public *selection(selection: CodeRange[], duration: number): ThreadGenerator {
    // Animate code selection
  }

  public *edit(edit: PossibleCodeEdit, duration?: number): ThreadGenerator {
    // Animate code editing
  }
}
```

### Code Example

```typescript
// packages/examples/src/scenes/code.tsx
export default makeScene2D(function* (view) {
  const code = createRef<Code>();

  view.add(
    <Code
      ref={code}
      fontSize={24}
      fontFamily={'JetBrains Mono'}
      language={'typescript'}
      code={`\
function hello() {
  console.log('Hello, World!');
}`}
    />
  );

  // Animate code editing
  yield* code().edit(`\
function hello(name: string) {
  console.log(\`Hello, \${name}!\`);
}`, 1);

  // Highlight specific lines
  yield* code().selection(lines(1, 2), 0.5);
});
```

## Latex Component

Mathematical notation rendering using LaTeX. Extends Img, not Shape.

### Latex Properties

```typescript
// packages/2d/src/lib/components/Latex.ts
export class Latex extends Img {
  @initial('')
  @signal()
  public declare readonly tex: SimpleSignal<string, this>;

  @initial({})
  @signal()
  public declare readonly options: SimpleSignal<MathJaxOptions, this>;

  @computed()
  public parsedTex(): string {
    // Parses LaTeX string
    return this.tex();
  }

  protected async draw(context: CanvasRenderingContext2D): Promise<void> {
    // Renders LaTeX using MathJax
    const svg = await this.renderLatex();
    context.drawImage(svg, ...);
  }
}
```

### Latex Example

```typescript
export default makeScene2D(function* (view) {
  const formula = createRef<Latex>();

  view.add(
    <Latex
      ref={formula}
      tex={''}
      fontSize={48}
      fill={'white'}
    />
  );

  // Animate formula appearance
  yield* formula().tex('E = mc^2', 1);

  // Change to more complex formula
  yield* formula().tex(
    '\\int_{-\\infty}^{\\infty} e^{-x^2} dx = \\sqrt{\\pi}',
    1,
  );
});
```

## Media Components

### Img Component

Image display component with loading and transformation support.

```typescript
// packages/2d/src/lib/components/Img.ts
export class Img extends Rect {
  @signal()
  public declare readonly src: SimpleSignal<string, this>;

  @initial(1)
  @signal()
  public declare readonly alpha: SimpleSignal<number, this>;

  @initial(true)
  @signal()
  public declare readonly smoothing: SimpleSignal<boolean, this>;

  protected image(): HTMLImageElement {
    const src = this.src();
    return this.imageCache.get(src) ?? this.loadImage(src);
  }

  protected draw(context: CanvasRenderingContext2D): void {
    const image = this.image();
    if (!image.complete) return;

    context.imageSmoothingEnabled = this.smoothing();
    context.globalAlpha = this.alpha();
    context.drawImage(image, ...);
  }
}
```

### Video Component

Video playback component with timeline control.

```typescript
// packages/2d/src/lib/components/Video.ts
export class Video extends Media {
  @initial('')
  @signal()
  public declare readonly src: SimpleSignal<string, this>;

  @initial(0)
  @signal()
  public declare readonly time: SimpleSignal<number, this>;

  @initial(false)
  @signal()
  public declare readonly playing: SimpleSignal<boolean, this>;

  @initial(false)
  @signal()
  public declare readonly loop: SimpleSignal<boolean, this>;

  @initial(1)
  @signal()
  public declare readonly volume: SimpleSignal<number, this>;

  @initial(1)
  @signal()
  public declare readonly playbackRate: SimpleSignal<number, this>;

  public play(): void {
    this.playing(true);
    this.video()?.play();
  }

  public pause(): void {
    this.playing(false);
    this.video()?.pause();
  }

  public seek(time: number): void {
    const video = this.video();
    if (video) {
      video.currentTime = time;
      this.time(time);
    }
  }

  protected video(): HTMLVideoElement | null {
    const src = this.src();
    return this.videoCache.get(src) ?? this.loadVideo(src);
  }
}
```

### Audio Component

Audio playback component for background music and sound effects.

```typescript
// packages/2d/src/lib/components/Audio.ts
export class Audio extends Media {
  @initial('')
  @signal()
  public declare readonly src: SimpleSignal<string, this>;

  @initial(0)
  @signal()
  public declare readonly time: SimpleSignal<number, this>;

  @initial(false)
  @signal()
  public declare readonly playing: SimpleSignal<boolean, this>;

  @initial(1)
  @signal()
  public declare readonly volume: SimpleSignal<number, this>;

  @initial(1)
  @signal()
  public declare readonly playbackRate: SimpleSignal<number, this>;

  protected audio(): HTMLAudioElement | null {
    const src = this.src();
    return this.audioCache.get(src) ?? this.loadAudio(src);
  }
}
```

## Specialized Components

### Icon Component

Icon rendering using icon fonts or SVG.

```typescript
// packages/2d/src/lib/components/Icon.ts
export class Icon extends Shape {
  @initial('')
  @signal()
  public declare readonly icon: SimpleSignal<string, this>;

  @initial('#000000')
  @colorSignal()
  public declare readonly color: ColorSignal<this>;

  protected draw(context: CanvasRenderingContext2D): void {
    // Render icon from icon set
  }
}
```

### SVG Component

SVG element rendering with manipulation support.

```typescript
// packages/2d/src/lib/components/SVG.ts
export class SVG extends Shape {
  @initial(null)
  @signal()
  public declare readonly svg: SimpleSignal<string | SVGSVGElement | null, this>;

  protected getPath(): Path2D | null {
    const svg = this.svg();
    if (!svg) return null;

    // Convert SVG to Path2D
    return this.svgToPath(svg);
  }
}
```

### Rive Component

Rive animation integration.

```typescript
// packages/2d/src/lib/components/Rive.ts
export class Rive extends Shape {
  @initial('')
  @signal()
  public declare readonly src: SimpleSignal<string, this>;

  @initial('')
  @signal()
  public declare readonly artboard: SimpleSignal<string, this>;

  @initial('')
  @signal()
  public declare readonly animation: SimpleSignal<string, this>;

  @initial(false)
  @signal()
  public declare readonly playing: SimpleSignal<boolean, this>;

  protected async draw(context: CanvasRenderingContext2D): Promise<void> {
    const rive = await this.loadRive();
    // Render Rive animation
  }
}
```

## Curve Components

### Bezier Curve

Base class for bezier curves.

```typescript
// packages/2d/src/lib/components/Bezier.ts
export class Bezier extends Curve {
  @initial([[0, 0]])
  @signal()
  public declare readonly points: SimpleSignal<BezierPoint[], this>;

  protected getPath(): Path2D {
    const path = new Path2D();
    const points = this.points();

    if (points.length === 0) return path;

    path.moveTo(points[0].position.x, points[0].position.y);

    for (let i = 1; i < points.length; i++) {
      const prev = points[i - 1];
      const curr = points[i];

      path.bezierCurveTo(
        prev.handleOut.x,
        prev.handleOut.y,
        curr.handleIn.x,
        curr.handleIn.y,
        curr.position.x,
        curr.position.y,
      );
    }

    if (this.closed()) {
      path.closePath();
    }

    return path;
  }
}
```

### QuadBezier

Quadratic bezier curve.

```typescript
// packages/2d/src/lib/components/QuadBezier.ts
export class QuadBezier extends Bezier {
  @initial([0, 0])
  @signal()
  public declare readonly p0: Vector2Signal<this>;

  @initial([0, 0])
  @signal()
  public declare readonly p1: Vector2Signal<this>;

  @initial([0, 0])
  @signal()
  public declare readonly p2: Vector2Signal<this>;

  protected getPath(): Path2D {
    const path = new Path2D();
    const p0 = this.p0();
    const p1 = this.p1();
    const p2 = this.p2();

    path.moveTo(p0.x, p0.y);
    path.quadraticCurveTo(p1.x, p1.y, p2.x, p2.y);

    return path;
  }
}
```

### CubicBezier

Cubic bezier curve.

```typescript
// packages/2d/src/lib/components/CubicBezier.ts
export class CubicBezier extends Bezier {
  @initial([0, 0])
  @signal()
  public declare readonly p0: Vector2Signal<this>;

  @initial([0, 0])
  @signal()
  public declare readonly p1: Vector2Signal<this>;

  @initial([0, 0])
  @signal()
  public declare readonly p2: Vector2Signal<this>;

  @initial([0, 0])
  @signal()
  public declare readonly p3: Vector2Signal<this>;

  protected getPath(): Path2D {
    const path = new Path2D();
    const p0 = this.p0();
    const p1 = this.p1();
    const p2 = this.p2();
    const p3 = this.p3();

    path.moveTo(p0.x, p0.y);
    path.bezierCurveTo(p1.x, p1.y, p2.x, p2.y, p3.x, p3.y);

    return path;
  }
}
```

### Spline

Smooth curve through multiple points.

```typescript
// packages/2d/src/lib/components/Spline.ts
export class Spline extends Curve {
  @initial([])
  @signal()
  public declare readonly points: SimpleSignal<Vector2[], this>;

  @initial(1)
  @signal()
  public declare readonly smoothness: SimpleSignal<number, this>;

  @initial(false)
  @signal()
  public declare readonly closed: SimpleSignal<boolean, this>;

  protected getPath(): Path2D {
    const points = this.points();
    const smoothness = this.smoothness();

    // Generate smooth curve through points using Catmull-Rom spline
    return this.generateSpline(points, smoothness, this.closed());
  }
}
```

## Utility Components

### Grid Component

Grid overlay for alignment and debugging.

```typescript
// packages/2d/src/lib/components/Grid.ts
export class Grid extends Shape {
  @initial(80)
  @vector2Signal('spacing')
  public declare readonly spacing: Vector2Signal<this>;

  @initial(1)
  @signal()
  public declare readonly lineWidth: SimpleSignal<number, this>;

  @initial('#ffffff20')
  @colorSignal()
  public declare readonly stroke: ColorSignal<this>;

  @initial('start')
  @signal()
  public declare readonly start: SimpleSignal<number, this>;

  @initial('end')
  @signal()
  public declare readonly end: SimpleSignal<number, this>;

  protected draw(context: CanvasRenderingContext2D): void {
    const spacing = this.spacing();
    const {width, height} = this.computedSize();

    // Draw vertical lines
    for (let x = -width / 2; x <= width / 2; x += spacing) {
      context.moveTo(x, -height / 2);
      context.lineTo(x, height / 2);
    }

    // Draw horizontal lines
    for (let y = -height / 2; y <= height / 2; y += spacing) {
      context.moveTo(-width / 2, y);
      context.lineTo(width / 2, y);
    }

    context.stroke();
  }
}
```

### Ray Component

Infinite ray for directional indicators.

```typescript
// packages/2d/src/lib/components/Ray.ts
export class Ray extends Line {
  @initial([0, 0])
  @vector2Signal()
  public declare readonly from: Vector2Signal<this>;

  @initial([100, 0])
  @vector2Signal()
  public declare readonly to: Vector2Signal<this>;

  protected getPath(): Path2D {
    const path = new Path2D();
    const from = this.from();
    const to = this.to();
    const direction = to.sub(from).normalized;

    // Extend ray to viewport bounds
    const extended = this.extendToViewport(from, direction);

    path.moveTo(from.x, from.y);
    path.lineTo(extended.x, extended.y);

    return path;
  }
}
```

### Knot Component

Control point for curve editing.

```typescript
// packages/2d/src/lib/components/Knot.ts
export class Knot extends Node {
  @initial([0, 0])
  @vector2Signal()
  public declare readonly position: Vector2Signal<this>;

  @initial([0, 0])
  @vector2Signal()
  public declare readonly handleIn: Vector2Signal<this>;

  @initial([0, 0])
  @vector2Signal()
  public declare readonly handleOut: Vector2Signal<this>;

  @initial('auto')
  @signal()
  public declare readonly auto: SimpleSignal<KnotAuto, this>;

  protected draw(context: CanvasRenderingContext2D): void {
    super.draw(context);

    // Draw handles
    this.drawHandle(context, this.handleIn(), '#ff0000');
    this.drawHandle(context, this.handleOut(), '#00ff00');
  }
}
```

## View2D - Root Component

The root component for 2D scenes.

### View2D Properties

```typescript
// packages/2d/src/lib/components/View2D.ts
@nodeName('View2D')
export class View2D extends Rect {
  @initial(PlaybackState.Paused)
  @signal()
  public declare readonly playbackState: SimpleSignal<PlaybackState, this>;

  @initial(0)
  @signal()
  public declare readonly globalTime: SimpleSignal<number, this>;

  @initial(0)
  @signal()
  public declare readonly fps: SimpleSignal<number, this>;

  @lazy(() => {
    const frameID = 'revideo-2d-frame';
    let frame = document.querySelector<HTMLDivElement>(`#${frameID}`);
    if (!frame) {
      frame = document.createElement('div');
      frame.id = frameID;
      frame.style.position = 'absolute';
      frame.style.pointerEvents = 'none';
      frame.style.top = '0';
      frame.style.left = '0';
      frame.style.fontFeatureSettings = 'normal';
      frame.style.opacity = '0';
      frame.style.overflow = 'hidden';
      document.body.prepend(frame);
    }
    return frame.shadowRoot ?? frame.attachShadow({mode: 'open'});
  })
  public static shadowRoot: ShadowRoot;

  public async render(context: CanvasRenderingContext2D): Promise<void> {
    this.computedSize();
    this.computedPosition();
    await super.render(context);
  }
}
```

### View2D Example

```typescript
// packages/template/src/project.ts
import {makeProject} from '@revideo/core';
import {makeScene2D} from '@revideo/2d';

export default makeProject({
  scenes: [
    makeScene2D(function* (view) {
      // view is the View2D component
      view.fill('#141414'); // Set background

      // Add components to the scene
      view.add(
        <>
          <Rect
            width={300}
            height={200}
            fill={'#88C0D0'}
            radius={20}
          />
          <Txt
            text={'Hello, World!'}
            fontSize={48}
            fill={'white'}
            y={-100}
          />
        </>
      );

      // Access scene properties
      console.log(view.fps()); // 0
    }),
  ],
});
```

## Signal Decorators

### @signal()

Basic signal decorator for reactive properties.

```typescript
class MyComponent extends Node {
  @signal()
  public declare readonly myProperty: SimpleSignal<number, this>;
}
```

### @initial()

Sets initial value for a signal.

```typescript
class MyComponent extends Node {
  @initial(100)
  @signal()
  public declare readonly width: SimpleSignal<number, this>;
}
```

### @computed()

Creates a computed signal that recalculates when dependencies change.

```typescript
class MyComponent extends Node {
  @computed()
  public area(): number {
    return this.width() * this.height();
  }
}
```

### @vector2Signal()

Creates a compound vector signal with x and y components.

```typescript
class MyComponent extends Node {
  @vector2Signal('position')
  public declare readonly position: Vector2Signal<this>;
  // Automatically creates position.x and position.y sub-signals
}
```

### @colorSignal()

Creates a color signal with color interpolation support.

```typescript
class MyComponent extends Node {
  @initial('#88C0D0')
  @colorSignal()
  public declare readonly fill: ColorSignal<this>;
}
```

### @spacingSignal()

Creates a spacing signal with top, bottom, left, right components.

```typescript
class MyComponent extends Node {
  @spacingSignal('padding')
  public declare readonly padding: SpacingSignal<this>;
  // Creates padding.top, padding.bottom, padding.left, padding.right
}
```

## Component Lifecycle Methods

### Initialization

```typescript
public constructor(props?: ComponentProps) {
  super(props);
  // Initialize component
}
```

### Rendering

```typescript
protected async draw(context: CanvasRenderingContext2D): Promise<void> {
  // Custom drawing logic
}

protected drawChildren(context: CanvasRenderingContext2D): void {
  // Draw child components
}

protected drawOverlay(context: CanvasRenderingContext2D): void {
  // Draw overlays on top
}
```

### State Management

```typescript
public getState(): ComponentState {
  // Capture current state
}

public applyState(state: ComponentState): void {
  // Apply saved state
}

public *restore(duration?: number, timingFunction?: TimingFunction): ThreadGenerator {
  // Animate back to saved state
}
```

### Resource Management

```typescript
public collectAsyncResources(): Promise<void>[] {
  // Collect async resources for preloading
}

public dispose(): void {
  // Clean up resources
}
```

## Animation Methods

All components inherit animation capabilities through signals:

```typescript
// Animate single property
yield* component.propertyName(targetValue, duration, timingFunction);

// Animate multiple properties
yield* all(
  component.x(100, 1),
  component.y(200, 1),
  component.rotation(45, 1),
);

// Chain animations
yield* chain(
  component.x(100, 1),
  component.y(200, 1),
);

// Sequence with delay
yield* sequence(
  0.5,
  component.x(100, 1),
  component.y(200, 1),
);

// Loop animation
yield* loop(3, () => component.rotation(360, 1));
```

## Real-World Examples

### Card Component

```typescript
// packages/examples/src/components/Card.tsx
export function Card({title, content, ...props}: CardProps) {
  return (
    <Rect
      layout
      direction={'column'}
      padding={20}
      gap={10}
      radius={10}
      fill={'#1e1e1e'}
      {...props}
    >
      <Txt
        text={title}
        fontSize={24}
        fontWeight={700}
        fill={'white'}
      />
      <Txt
        text={content}
        fontSize={16}
        fill={'#888'}
      />
    </Rect>
  );
}
```

### Progress Bar

```typescript
export function* ProgressBar(view: View2D) {
  const progress = createRef<Rect>();

  view.add(
    <Rect
      width={400}
      height={30}
      fill={'#333'}
      radius={15}
    >
      <Rect
        ref={progress}
        width={0}
        height={30}
        fill={'#88C0D0'}
        radius={15}
        alignSelf={'stretch'}
      />
    </Rect>
  );

  // Animate progress
  yield* progress().width(400, 2);
}
```

### Animated Graph

```typescript
export function* Graph(view: View2D) {
  const line = createRef<Line>();

  const points = [
    [-200, 50],
    [-100, -30],
    [0, 20],
    [100, -60],
    [200, 40],
  ];

  view.add(
    <>
      <Grid
        width={500}
        height={300}
        spacing={50}
        stroke={'#ffffff20'}
      />
      <Line
        ref={line}
        points={points}
        stroke={'#88C0D0'}
        lineWidth={3}
        radius={10}
        end={0}
      />
    </>
  );

  // Animate line drawing
  yield* line().end(1, 2);
}
```

## Performance Tips

### Caching

Enable caching for complex components that don't change frequently:

```typescript
<Rect
  cache
  cachePadding={20}
  // Complex children here
/>
```

### Composite Operations

Use composite operations for special effects:

```typescript
<Rect
  composite
  compositeOperation={'multiply'}
/>
```

### Filters

Apply filters efficiently:

```typescript
// Chain multiple filters
yield* all(
  node.filters.blur(10, 1),
  node.filters.brightness(1.5, 1),
  node.filters.contrast(1.2, 1),
);
```

### Layout Optimization

Use layout modes appropriately:

```typescript
<Layout
  layout={'content-box'} // or 'border-box'
  // Optimizes layout calculations
/>
```

## Summary

The Revideo component system provides:

1. **Hierarchical Structure**: Clear inheritance from Node → Layout → Shape → specific components
2. **Signal-Based Properties**: All properties are reactive signals enabling smooth animations
3. **Comprehensive Components**: 20+ components covering shapes, text, media, and utilities
4. **Flexible Animation**: Every property can be animated with timing and interpolation
5. **Layout System**: Full flexbox support for responsive designs
6. **Performance Features**: Caching, compositing, and optimization options
7. **Extensibility**: Easy to create custom components by extending base classes

Each component is designed to be composable, animatable, and performant, providing a complete toolkit for creating sophisticated 2D animations.