# REVIDEO TECHNICAL ARCHITECTURE DOCUMENTATION

## Core Animation Engine

### Signal System Implementation

The signal system provides reactive state management for animations, located in `packages/core/src/signals/`.

#### Type Definitions
```typescript
// packages/core/src/signals/types.ts
export type SignalValue<TValue> = TValue | (() => TValue);

export type SignalGenerator<TSetterValue, TValue extends TSetterValue> = ThreadGenerator & {
  to: SignalTween<TSetterValue, TValue>;
  back: (time: number, timingFunction?: TimingFunction, interpolationFunction?: InterpolationFunction<TValue>) => SignalGenerator<TSetterValue, TValue>;
  wait: (duration: number) => SignalGenerator<TSetterValue, TValue>;
  run: (task: ThreadGenerator) => SignalGenerator<TSetterValue, TValue>;
  do: (callback: () => void) => SignalGenerator<TSetterValue, TValue>;
};

export interface Signal<TSetterValue, TValue extends TSetterValue, TOwner = void>
  extends SignalSetter<TSetterValue, TOwner>,
          SignalGetter<TValue>,
          SignalTween<TSetterValue, TValue> {
  reset(): TOwner;
  save(): TOwner;
  isInitial(): boolean;
  context: TContext;
}
```

#### SignalContext Class
```typescript
// packages/core/src/signals/SignalContext.ts
export class SignalContext<
  TSetterValue,
  TValue extends TSetterValue = TSetterValue,
  TOwner = void,
> extends DependencyContext<TOwner> {
  protected extensions: SignalExtensions<TSetterValue, TValue>;
  protected current: SignalValue<TSetterValue> | undefined;
  protected last: TValue | undefined;
  protected tweening = false;

  public constructor(
    private initial: SignalValue<TSetterValue> | undefined,
    private readonly interpolation: InterpolationFunction<TValue>,
    owner: TOwner = <TOwner>(<unknown>undefined),
    protected parser: (value: TSetterValue) => TValue = value => <TValue>value,
    extensions: Partial<SignalExtensions<TSetterValue, TValue>> = {},
  ) {
    super(owner);

    if (this.initial !== undefined) {
      this.current = this.initial;
      this.markDirty();

      if (!isReactive(this.initial)) {
        this.last = this.parse(this.initial);
      }
    }

    this.extensions = {
      getter: this.getter.bind(this),
      setter: this.setter.bind(this),
      tweener: this.tweener.bind(this),
      ...extensions,
    };
  }

  public getter(): TValue {
    if (this.event.isRaised() && isReactive(this.current)) {
      this.clearDependencies();
      this.startCollecting();
      try {
        this.last = this.parse(this.current());
      } catch (e: any) {
        useLogger().error({
          ...errorToLog(e),
          inspect: (<any>this.owner)?.key,
        });
      }
      this.finishCollecting();
    }
    this.event.reset();
    this.collect();

    return this.last!;
  }

  public setter(value: SignalValue<TSetterValue> | typeof DEFAULT): TOwner {
    if (value === DEFAULT) {
      value = this.initial!;
    }

    if (this.current === value) {
      return this.owner;
    }

    this.current = value;
    this.markDirty();
    this.clearDependencies();

    if (!isReactive(value)) {
      this.last = this.parse(value);
    }

    return this.owner;
  }

  public *tweener(
    value: SignalValue<TSetterValue>,
    duration: number,
    timingFunction: TimingFunction,
    interpolationFunction: InterpolationFunction<TValue>,
  ): ThreadGenerator {
    const from = this.get();
    yield* tween(duration, v => {
      this.set(
        interpolationFunction(
          from,
          this.parse(unwrap(value)),
          timingFunction(v),
        ),
      );
    });
  }

  public isInitial() {
    this.collect();
    return this.current === this.initial;
  }

  public raw() {
    return this.current;
  }
}
```

#### Signal Creation
```typescript
// packages/core/src/signals/createSignal.ts
export function createSignal<TValue, TOwner = void>(
  initial?: SignalValue<TValue>,
  interpolation: InterpolationFunction<TValue> = deepLerp,
  owner?: TOwner,
): SimpleSignal<TValue, TOwner> {
  return new SignalContext<TValue, TValue, TOwner>(
    initial,
    interpolation,
    owner,
  ).toSignal();
}
```

#### Computed Signals
```typescript
// packages/core/src/signals/ComputedContext.ts
export class ComputedContext<TValue> extends DependencyContext<any> {
  protected override invoke(...args: any[]): TValue {
    if (this.event.isRaised()) {
      this.clearDependencies();
      this.startCollecting();
      try {
        this.last = this.factory(...args);
      } catch (e: any) {
        useLogger().error({...errorToLog(e), inspect: this.owner?.key});
      }
      this.finishCollecting();
    }
    this.event.reset();
    this.collect();
    return this.last!;
  }
}
```

### Threading System

Located in `packages/core/src/threading/`, enables concurrent animation execution.

#### ThreadGenerator Type
```typescript
// packages/core/src/threading/ThreadGenerator.ts
export type ThreadGenerator = Generator<
  ThreadGenerator | Promise<any> | Promisable<any> | void,
  void,
  Thread | any
>;
```

#### Thread Class
```typescript
// packages/core/src/threading/Thread.ts
export class Thread {
  public children: Thread[] = [];
  /**
   * The next value to be passed to the wrapped generator.
   */
  public value: unknown;

  /**
   * The current time of this thread.
   */
  public readonly time = createSignal(0);

  /**
   * The fixed time of this thread.
   *
   * @remarks
   * Fixed time is a multiple of the frame duration.
   */
  public get fixed() {
    return this.fixedTime;
  }

  /**
   * Check if this thread or any of its ancestors has been canceled.
   */
  public get canceled(): boolean {
    return this.isCanceled || (this.parent?.canceled ?? false);
  }

  public get paused(): boolean {
    return this.isPaused || (this.parent?.paused ?? false);
  }

  public parent: Thread | null = null;
  private isCanceled = false;
  private isPaused = false;
  private fixedTime = 0;
  private queue: ThreadGenerator[] = [];

  public constructor(
    /**
     * The generator wrapped by this thread.
     */
    public readonly runner: ThreadGenerator & {task?: Thread},
  ) {
    if (this.runner.task) {
      useLogger().error({
        message: `The generator "${getTaskName(
          this.runner,
        )}" is already being executed by another thread.`,
        remarks: reusedGenerator,
      });
      this.runner = noop();
    }
    this.runner.task = this;
  }

  /**
   * Progress the wrapped generator once.
   */
  public next() {
    if (this.paused) {
      return {
        value: null,
        done: false,
      };
    }

    startThread(this);
    const result = this.runner.next(this.value);
    endThread(this);
    this.value = null;
    return result;
  }

  /**
   * Prepare the thread for the next update cycle.
   *
   * @param dt - The delta time of the next cycle.
   */
  public update(dt: number) {
    if (!this.paused) {
      this.time(this.time() + dt);
      this.fixedTime += dt;
    }
    this.children = this.children.filter(child => !child.canceled);
  }

  public spawn(
    child: ThreadGenerator | (() => ThreadGenerator),
  ): ThreadGenerator {
    if (!isThreadGenerator(child)) {
      child = child();
    }
    this.queue.push(child);
    return child;
  }

  public add(child: Thread) {
    child.parent = this;
    child.isCanceled = false;
    child.time(this.time());
    child.fixedTime = this.fixedTime;
    this.children.push(child);

    setTaskName(child.runner, `unknown ${this.children.length}`);
  }

  public cancel() {
    this.runner.return();
    this.isCanceled = true;
    this.parent = null;
    this.drain(task => task.return());
  }
}
```

### Generator-Based Animation Patterns

#### Signal Tweening Pattern
```typescript
// Example from packages/examples/
const signal = createSignal(0);

// Sequential animations (chained)
yield* signal(100, 2).to(50, 1).wait(0.5).do(() => console.log('done'));

// Concurrent animations using spawn
yield node.x(100, 1);
yield node.y(100, 1);
yield* waitFor(1); // Both animations run simultaneously
```

#### Scene Lifecycle Integration
```typescript
// packages/core/src/scenes/GeneratorScene.ts
export abstract class GeneratorScene<T>
  implements Scene<ThreadGeneratorFactory<T>>, Threadable {

  private runnerFactory: ThreadGeneratorFactory<T>;
  private runner: ThreadGenerator | null = null;
  private thread: Thread | null = null;

  public constructor(
    description: FullSceneDescription<ThreadGeneratorFactory<T>>
  ) {
    this.runnerFactory = description.config;
    decorate(this.runnerFactory, threadable(this.name));
  }

  public abstract getView(): T;

  public async reset(previousScene?: Scene): Promise<void> {
    this.logger.profile('reset');

    if (previousScene) {
      this.slides.loadData(previousScene.slides.getData());
      this.variables.updateSignals(previousScene.variables.get());
    }

    this.thread?.cancel();
    this.runner = this.runnerFactory(this.getView());
    this.thread = new Thread(this.runner);

    // Fast-forward to beginning
    await this.thread.next();
    await this.recalculate();

    this.logger.profile('reset');
  }

  public async next(): Promise<boolean> {
    await this.thread?.next();
    return this.thread?.isFinished ?? true;
  }
}
```

### Flow Control Utilities

Located in `packages/core/src/flow/`.

#### Chain (Sequential Execution)
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

#### All (Concurrent Execution)
```typescript
// packages/core/src/flow/all.ts
export function* all(...tasks: ThreadGenerator[]): ThreadGenerator {
  for (const task of tasks) {
    yield task; // Start all tasks concurrently
  }
  yield* join(...tasks); // Wait for all to complete
}
```

#### Loop
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

#### Sequence
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

#### WaitFor
```typescript
// packages/core/src/flow/scheduling.ts
decorate(waitFor, threadable());
/**
 * Wait for the given amount of time.
 *
 * @param seconds - The relative time in seconds.
 * @param after - An optional task to be run after the function completes.
 */
export function* waitFor(
  seconds = 0,
  after?: ThreadGenerator,
): ThreadGenerator {
  const thread = useThread();
  const step = usePlayback().framesToSeconds(1);

  const targetTime = thread.time() + seconds;
  // subtracting the step is not necessary, but it keeps the thread time ahead
  // of the project time.
  while (targetTime - step > thread.fixed) {
    yield;
  }
  thread.time(targetTime);

  if (after) {
    yield* after;
  }
}
```

## Package Architecture

### Monorepo Structure

```
revideo/
├── packages/
│   ├── core/           # Core animation engine
│   ├── 2d/             # 2D rendering components
│   ├── vite-plugin/    # Vite build plugin
│   ├── ui/             # Editor UI
│   ├── ffmpeg/         # FFmpeg integration
│   ├── examples/       # Example projects
│   ├── template/       # Project template
│   ├── telemetry/      # Usage telemetry
│   ├── internal/       # Internal utilities
│   └── create/         # Project creation tool
├── package.json        # Root workspace configuration
└── tsconfig.json       # TypeScript configuration
```

### Core Package (`@revideo/core`)

#### Export Structure
```typescript
// packages/core/src/index.ts
export * from './app';         // Project, Player, Renderer
export * from './decorators';  // threadable, decorate
export * from './events';      // EventDispatcher, ValueDispatcher
export * from './exporter';    // Exporter interface, implementations
export * from './flow';        // chain, all, loop, sequence, etc.
export * from './media';       // Media utilities
export * from './plugin';      // Plugin system
export * from './scenes';      // Scene, GeneratorScene
export * from './signals';     // createSignal, createComputed
export * from './threading';   // Thread, ThreadGenerator
export * from './transitions'; // Transition system
export * from './tweening';    // tween, easing functions
export * from './types';       // Vector2, Color, BBox, etc.
export * from './utils';       // Utility functions
```

#### App Module Structure
```typescript
// packages/core/src/app/
Project.ts          // Project configuration and settings
Player.ts           // Real-time playback controller
Renderer.ts         // Export/rendering pipeline
PlaybackManager.ts  // Scene sequence management
PlaybackStatus.ts   // Frame/time tracking
Stage.ts           // Canvas rendering target
Logger.ts          // Logging infrastructure
SharedWebGLContext.ts // WebGL resource sharing
```

### 2D Package (`@revideo/2d`)

#### Component Hierarchy
```
Node (base class)
├── Layout
│   ├── Shape
│   │   ├── Circle
│   │   ├── Rect
│   │   ├── Line
│   │   ├── Path
│   │   └── Polygon
│   ├── Txt
│   ├── Media
│   │   ├── Img
│   │   ├── Video
│   │   └── Audio
│   └── Code/CodeBlock
└── View2D (root component)
```

#### Node Class Implementation
```typescript
// packages/2d/src/lib/components/Node.ts
@nodeName('Node')
export class Node implements Promisable<Node> {
  // Transform signals
  @signal() x: SimpleVector2Signal<this>;
  @signal() y: SimpleVector2Signal<this>;
  @signal() position: Vector2Signal<this>;
  @signal() rotation: SimpleSignal<number, this>;
  @signal() scaleX: SimpleSignal<number, this>;
  @signal() scaleY: SimpleSignal<number, this>;
  @signal() scale: Vector2Signal<this>;
  @signal() skewX: SimpleSignal<number, this>;
  @signal() skewY: SimpleSignal<number, this>;
  @signal() zIndex: SimpleSignal<number, this>;

  // Visual properties
  @signal() opacity: SimpleSignal<number, this>;
  @signal() filters: FiltersSignal<this>;
  @signal() shadowColor: ColorSignal<this>;
  @signal() shadowBlur: SimpleSignal<number, this>;
  @signal() shadowOffset: Vector2Signal<this>;

  // Layout and caching
  @signal() cache: SimpleSignal<boolean, this>;
  @signal() cachePadding: SpacingSignal<this>;
  @signal() composite: SimpleSignal<boolean, this>;
  @signal() compositeOperation: SimpleSignal<GlobalCompositeOperation, this>;

  public children: Node[] = [];
  private parent: Node | null = null;

  public async render(context: CanvasRenderingContext2D): Promise<void> {
    // Apply transforms
    context.save();

    // Apply opacity
    const opacity = this.opacity();
    if (opacity < 1) {
      context.globalAlpha *= opacity;
    }

    // Apply transforms
    this.applyTransform(context);

    // Draw self
    await this.draw(context);

    // Draw children
    for (const child of this.sortedChildren()) {
      await child.render(context);
    }

    context.restore();
  }
}
```

## Vite Plugin System

### Main Plugin Configuration
```typescript
// packages/vite-plugin/src/main.ts
export interface MotionCanvasPluginConfig {
  project?: string | string[];  // Default: './src/project.ts'
  output?: string;              // Default: './output'
  bufferedAssets?: RegExp | false;
  editor?: string;              // Default: '@revideo/ui'
  buildForEditor?: boolean;
}

export default (config: MotionCanvasPluginConfig = {}): Plugin[] => {
  return [
    metaPlugin(),
    settingsPlugin(),
    exporterPlugin({outputPath}),
    ffmpegBridgePlugin({output: outputPath}),
    editorPlugin({editor, projects}),
    projectsPlugin({projects, plugins, buildForEditor}),
    assetsPlugin({bufferedAssets}),
    wasmExporterPlugin(),
    rivePlugin(),
    webglPlugin(),
    metricsPlugin(),
  ];
};
```

### Projects Plugin
```typescript
// packages/vite-plugin/src/partials/projects.ts
export function projectsPlugin({
  buildForEditor,
  projects,
}: ProjectPluginConfig): Plugin {
  return {
    name: 'revideo:project',
    config(config) {
      return {
        build: {
          target: buildForEditor ? 'esnext' : 'modules',
          rollupOptions: {
            input: Object.fromEntries(
              projects.list.map(project => [project.name, project.url]),
            ),
          },
        },
        esbuild: {
          jsx: 'automatic',
          jsxImportSource: '@revideo/2d/lib',
        },
        optimizeDeps: {
          entries: projects.list.map(project => project.url),
          exclude: ['preact', 'preact/*', '@preact/signals'],
        },
      };
    },
    async configResolved(resolvedConfig) {
      // Plugin coordination
      plugins.push(
        ...resolvedConfig.plugins
          .filter(isPlugin)
          .map(plugin => plugin[PLUGIN_OPTIONS]),
      );
      await Promise.all(
        plugins.map(plugin =>
          plugin.config?.({
            output: outputPath,
            projects: projects.list,
          }),
        ),
      );
    },
  };
}
```

### Export Systems

#### Exporter Interface
```typescript
// packages/core/src/exporter/Exporter.ts
export interface Exporter {
  configuration?(): Promise<RendererSettings | void>;
  start?(): Promise<void>;
  handleFrame(
    canvas: HTMLCanvasElement,
    frame: number,
    sceneFrame: number,
    sceneName: string,
    signal: AbortSignal,
  ): Promise<void>;
  mergeMedia?(): Promise<void>;
  generateAudio?(
    assetsInfo: AssetInfo[][],
    startFrame: number,
    endFrame: number,
  ): Promise<void>;
  stop?(): Promise<RendererResult>;
}
```

#### Image Sequence Exporter
```typescript
// packages/core/src/exporter/ImageExporter.ts
export interface ImageExporterOptions {
  quality: number;
  fileType: CanvasOutputMimeType;
  groupByScene: boolean;
}

export class ImageExporter implements Exporter {
  public static readonly id = '@revideo/core/image-sequence';
  public static readonly displayName = 'Image sequence';

  public async handleFrame(
    canvas: HTMLCanvasElement,
    frame: number,
    sceneFrame: number,
    sceneName: string,
    signal: AbortSignal,
  ): Promise<void> {
    const blob = await new Promise<Blob>((resolve, reject) => {
      canvas.toBlob(blob => {
        if (blob) resolve(blob);
        else reject('Failed to create blob');
      }, this.fileType, this.quality);
    });

    await this.server.writeFrame(frame, blob);
  }
}
```

#### FFmpeg Bridge
```typescript
// packages/vite-plugin/src/partials/ffmpegBridge.ts
export function ffmpegBridgePlugin({output}: ExporterPluginConfig): Plugin {
  return {
    name: 'revideo/ffmpeg',
    configureServer(server) {
      const ffmpegBridge = new FFmpegBridge(server.ws, {output});

      server.middlewares.use('/audio-processing/generate-audio', (req, res) =>
        handlePostRequest(req, res, async ({tempDir, assets, ...rest}) =>
          generateAudio({outputDir: output, tempDir, assets, ...rest}),
        ),
      );

      server.middlewares.use('/audio-processing/merge-media', (req, res) =>
        handlePostRequest(req, res, async ({outputFilename, tempDir, ...rest}) =>
          mergeMedia(outputFilename, output, tempDir, format),
        ),
      );
    },
  };
}
```

## Rendering Pipeline

### Scene Graph and Node Hierarchy

#### View2D Component
```typescript
// packages/2d/src/lib/components/View2D.ts
@nodeName('View2D')
export class View2D extends Rect {
  @lazy(() => {
    const frameID = 'revideo-2d-frame';
    let frame = document.querySelector<HTMLDivElement>(`#${frameID}`);
    if (!frame) {
      frame = document.createElement('div');
      frame.id = frameID;
      frame.style.position = 'absolute';
      document.body.prepend(frame);
    }
    return frame.shadowRoot ?? frame.attachShadow({mode: 'open'});
  })
  public static shadowRoot: ShadowRoot;

  @initial(PlaybackState.Paused)
  @signal()
  public declare readonly playbackState: SimpleSignal<PlaybackState, this>;

  @initial(0)
  @signal()
  public declare readonly globalTime: SimpleSignal<number, this>;

  public async render(context: CanvasRenderingContext2D) {
    this.computedSize();
    this.computedPosition();
    await super.render(context);
  }
}
```

#### Scene2D Rendering
```typescript
// packages/2d/src/lib/scenes/Scene2D.ts
export class Scene2D extends GeneratorScene<View2D> {
  public async draw(context: CanvasRenderingContext2D) {
    context.save();
    this.renderLifecycle.dispatch([SceneRenderEvent.BeforeRender, context]);

    context.save();
    this.renderLifecycle.dispatch([SceneRenderEvent.BeginRender, context]);

    this.getView()
      .playbackState(this.playback.state)
      .globalTime(this.playback.time)
      .fps(this.playback.fps);

    await this.getView().render(context);

    this.renderLifecycle.dispatch([SceneRenderEvent.FinishRender, context]);
    context.restore();

    this.renderLifecycle.dispatch([SceneRenderEvent.AfterRender, context]);
    context.restore();
  }
}
```

### Frame Rendering Process

#### Renderer Class
```typescript
// packages/core/src/app/Renderer.ts
export class Renderer {
  public readonly stage = new Stage();
  public readonly estimator = new TimeEstimator();

  private readonly lock = new Semaphore();
  private readonly playback: PlaybackManager;
  private readonly status: PlaybackStatus;
  private readonly sharedWebGLContext: SharedWebGLContext;
  private exporter: Exporter | null = null;
  private abortController: AbortController | null = null;

  public async render(settings: RendererSettings) {
    if (this.state.current !== RendererState.Initial) return;
    await this.lock.acquire();
    this.estimator.reset();
    this.state.current = RendererState.Working;
    let result: RendererResult;
    try {
      this.abortController = new AbortController();
      result = await this.run(settings, this.abortController.signal);
    } catch (e: any) {
      this.project.logger.error(e);
      result = RendererResult.Error;
      // Error handling...
    }

    this.estimator.update(1);
    this.state.current = RendererState.Initial;
    this.finished.dispatch(result);
    this.sharedWebGLContext.dispose();
    this.lock.release();
  }

  private async run(
    settings: RendererSettings,
    signal: AbortSignal,
  ): Promise<RendererResult> {
    // Select and instantiate exporter
    const exporters: ExporterClass[] = [
      FFmpegExporterClient,
      ImageExporter,
      WasmExporter,
    ];

    const exporterClass = exporters.find(
      exporter => exporter.id === settings.exporter.name,
    );

    this.exporter = await exporterClass.create(this.project, settings);

    // Configure and reset
    this.stage.configure(settings);
    this.playback.fps = settings.fps;
    this.playback.state = PlaybackState.Rendering;

    const from = this.status.secondsToFrames(settings.range[0]);
    const to = Math.min(
      this.playback.duration,
      this.status.secondsToFrames(settings.range[1]),
    );

    await this.reloadScenes(settings);
    await this.playback.recalculate();
    await this.playback.reset();
    await this.playback.seek(from);

    if (signal.aborted) return RendererResult.Aborted;

    await this.exporter.start?.();

    // Main rendering loop
    try {
      this.estimator.reset(1 / (to - from));
      await this.exportFrame(signal);

      let finished = false;
      while (!finished) {
        await this.playback.progress();
        await this.exportFrame(signal);

        if (this.playback.finished || this.playback.frame >= to) {
          finished = true;
        }
        if (signal.aborted) {
          result = RendererResult.Aborted;
          finished = true;
        }
      }
    } catch (e: any) {
      this.project.logger.error(e);
      result = RendererResult.Error;
    }

    await this.exporter.stop?.(result);
    return result;
  }
}
```

#### PlaybackManager
```typescript
// packages/core/src/app/PlaybackManager.ts
export class PlaybackManager {
  public frame = 0;
  public speed = 1;
  public fps = 30;
  public duration = 0;
  public finished = false;
  public slides: Slide[] = [];

  public previousScene: Scene | null = null;
  public state = PlaybackState.Paused;

  private currentSceneReference: ValueDispatcher<Scene> | null = null;
  private scenes = new ValueDispatcher<Scene[]>([]);

  public async seek(frame: number): Promise<boolean> {
    if (
      frame <= this.frame ||
      (this.currentScene.isCached() && this.currentScene.lastFrame < frame)
    ) {
      const scene = this.findBestScene(frame);
      if (scene !== this.currentScene) {
        this.currentScene.stopAllMedia();
        this.previousScene = null;
        this.currentScene = scene;

        this.frame = this.currentScene.firstFrame;
        await this.currentScene.reset();
      } else if (this.frame >= frame) {
        this.previousScene = null;
        this.frame = this.currentScene.firstFrame;
        await this.currentScene.reset();
      }
    }

    this.finished = false;
    while (this.frame < frame && !this.finished) {
      this.finished = await this.next();
    }

    return this.finished;
  }

  private async next(): Promise<boolean> {
    if (this.previousScene) {
      await this.previousScene.next();
      if (this.currentScene.isFinished()) {
        this.previousScene = null;
      }
    }

    this.frame += this.speed;

    if (this.currentScene.isFinished()) {
      return true;
    }

    await this.currentScene.next();
    if (this.previousScene && this.currentScene.isAfterTransitionIn()) {
      this.previousScene = null;
    }

    if (this.currentScene.canTransitionOut()) {
      this.previousScene = this.currentScene;
      const nextScene = this.getNextScene(this.previousScene);
      if (nextScene) {
        this.previousScene.stopAllMedia();
        this.currentScene = nextScene;
        await this.currentScene.reset(this.previousScene);
      }
      if (!nextScene || this.currentScene.isAfterTransitionIn()) {
        this.previousScene = null;
      }
    }

    return this.currentScene.isFinished();
  }
}
```

## Development Workflow

### Build Scripts and Configuration

#### Root Package Scripts
```json
{
  "scripts": {
    "core:dev": "npm run dev -w packages/core",
    "core:build": "npm run build -w packages/core",
    "core:bundle": "npm run bundle -w packages/core",
    "core:test": "npm run test -w packages/core",
    "2d:dev": "npm run dev -w packages/2d",
    "2d:build": "npm run build -w packages/2d",
    "2d:bundle": "npm run bundle -w packages/2d",
    "ui:dev": "npm run dev -w packages/ui",
    "ui:build": "npm run build -w packages/ui",
    "vite-plugin:dev": "npm run dev -w packages/vite-plugin",
    "vite-plugin:build": "npm run build -w packages/vite-plugin",
    "e2e:test": "npm run test -w packages/e2e",
    "eslint": "eslint \"**/*.ts?(x)\"",
    "prettier": "prettier --check ."
  }
}
```

### TypeScript Configuration

#### Core TypeScript Config
```json
{
  "compilerOptions": {
    "outDir": "lib",
    "inlineSourceMap": true,
    "noImplicitAny": true,
    "strict": true,
    "module": "esnext",
    "target": "es2020",
    "allowJs": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "incremental": true
  },
  "include": ["src"],
  "exclude": ["src/**/*.test.*"]
}
```

## Tweening and Animation System

### Tween Function
```typescript
// packages/core/src/tweening/tween.ts
export function* tween(
  seconds: number,
  onProgress: (value: number, time: number) => void,
  onEnd?: (value: number, time: number) => void,
): ThreadGenerator {
  const thread = useThread();
  const startTime = thread.time();
  const endTime = thread.time() + seconds;

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

### Timing Functions (Easing)
```typescript
// packages/core/src/tweening/timingFunctions.ts
export interface TimingFunction {
  (value: number, from?: number, to?: number): number;
}

// Available easing functions:
// - linear
// - easeInSine, easeOutSine, easeInOutSine
// - easeInQuad, easeOutQuad, easeInOutQuad
// - easeInCubic, easeOutCubic, easeInOutCubic
// - easeInQuart, easeOutQuart, easeInOutQuart
// - easeInQuint, easeOutQuint, easeInOutQuint
// - easeInExpo, easeOutExpo, easeInOutExpo
// - easeInCirc, easeOutCirc, easeInOutCirc
// - easeInElastic, easeOutElastic, easeInOutElastic
// - easeInBack, easeOutBack, easeInOutBack
// - easeInBounce, easeOutBounce, easeInOutBounce
```

### Spring Animation
```typescript
// packages/core/src/tweening/spring.ts
export interface Spring {
  mass: number;
  stiffness: number;
  damping: number;
  initialVelocity?: number;
}

export function makeSpring(
  mass: number,
  stiffness: number,
  damping: number,
  initialVelocity?: number,
): Spring {
  return {
    mass,
    stiffness,
    damping,
    initialVelocity,
  };
}

// Preset spring configurations:
export const BeatSpring: Spring = makeSpring(0.13, 5.7, 1.2, 10.0);
export const PlopSpring: Spring = makeSpring(0.2, 20.0, 0.68, 0.0);
export const BounceSpring: Spring = makeSpring(0.08, 4.75, 0.05, 0.0);
export const SwingSpring: Spring = makeSpring(0.39, 19.85, 2.82, 0.0);
export const JumpSpring: Spring = makeSpring(0.04, 10.0, 0.7, 8.0);
export const StrikeSpring: Spring = makeSpring(0.03, 20.0, 0.9, 4.8);
export const SmoothSpring: Spring = makeSpring(0.16, 15.35, 1.88, 0.0);
```

### Interpolation Functions
```typescript
// packages/core/src/tweening/interpolationFunctions.ts
export interface InterpolationFunction<T, TRest extends any[] = any[]> {
  (from: T, to: T, value: number, ...args: TRest): T;
}

export function deepLerp<T>(
  from: T,
  to: T,
  value: number,
  suppressWarnings = false,
): T {
  if (value === 0) return from;
  if (value === 1) return to;

  if (typeof from === 'number' && typeof to === 'number') {
    return map(from, to, value);
  }

  if (from instanceof Color && to instanceof Color) {
    return Color.lerp(from, to, value) as T;
  }

  if (from instanceof Vector2 && to instanceof Vector2) {
    return new Vector2(
      map(from.x, to.x, value),
      map(from.y, to.y, value),
    ) as T;
  }

  // Handle objects, arrays, maps recursively
  // ...
}

// Text interpolation functions
export function textLerp(from: string, to: string, value: number): string {
  const fromLength = from.length;
  const toLength = to.length;
  const length = Math.round(map(fromLength, toLength, value));

  let result = '';
  for (let i = 0; i < length; i++) {
    const fromChar = from[i] ?? '';
    const toChar = to[i] ?? '';
    const ratio = i / Math.max(1, length - 1);

    if (value < ratio) {
      result += fromChar;
    } else {
      result += toChar;
    }
  }

  return result;
}
```

## Type System

### Core Types

#### Vector2
```typescript
// packages/core/src/types/Vector.ts
export type PossibleVector2<T = number> =
  | SerializedVector2<T>
  | {width: T; height: T}
  | T
  | [T, T]
  | undefined;

export class Vector2 implements Type, WebGLConvertible {
  public static readonly zero = new Vector2();
  public static readonly one = new Vector2(1, 1);
  public static readonly right = new Vector2(1, 0);
  public static readonly left = new Vector2(-1, 0);
  public static readonly up = new Vector2(0, 1);
  public static readonly down = new Vector2(0, -1);
  /**
   * A constant equal to `Vector2(0, -1)`
   */
  public static readonly top = new Vector2(0, -1);
  /**
   * A constant equal to `Vector2(0, 1)`
   */
  public static readonly bottom = new Vector2(0, 1);
  /**
   * A constant equal to `Vector2(-1, -1)`
   */
  public static readonly topLeft = new Vector2(-1, -1);
  /**
   * A constant equal to `Vector2(1, -1)`
   */
  public static readonly topRight = new Vector2(1, -1);
  /**
   * A constant equal to `Vector2(-1, 1)`
   */
  public static readonly bottomLeft = new Vector2(-1, 1);
  /**
   * A constant equal to `Vector2(1, 1)`
   */
  public static readonly bottomRight = new Vector2(1, 1);

  public x = 0;
  public y = 0;

  public constructor();
  public constructor(from: PossibleVector2);
  public constructor(x: number, y: number);

  public constructor(one?: PossibleVector2 | number, two?: number) {
    if (one === undefined || one === null) {
      return;
    }

    if (typeof one !== 'object') {
      this.x = one;
      this.y = two ?? one;
      return;
    }

    if (Array.isArray(one)) {
      this.x = one[0];
      this.y = one[1];
      return;
    }

    if ('width' in one) {
      this.x = one.width;
      this.y = one.height;
      return;
    }

    this.x = one.x;
    this.y = one.y;
  }

  public add(possibleVector: PossibleVector2) {
    const vector = new Vector2(possibleVector);
    return new Vector2(this.x + vector.x, this.y + vector.y);
  }

  public sub(possibleVector: PossibleVector2) {
    const vector = new Vector2(possibleVector);
    return new Vector2(this.x - vector.x, this.y - vector.y);
  }

  public scale(value: number) {
    return new Vector2(this.x * value, this.y * value);
  }

  public dot(possibleVector: PossibleVector2): number {
    const vector = new Vector2(possibleVector);
    return this.x * vector.x + this.y * vector.y;
  }

  public cross(possibleVector: PossibleVector2): number {
    const vector = new Vector2(possibleVector);
    return this.x * vector.y - this.y * vector.x;
  }

  public get magnitude(): number {
    return Vector2.magnitude(this.x, this.y);
  }

  public get normalized(): Vector2 {
    return this.scale(1 / Vector2.magnitude(this.x, this.y));
  }

  /**
   * Return the angle in degrees between the vector and the positive x-axis.
   */
  public get degrees() {
    return Vector2.degrees(this.x, this.y);
  }
}
```

#### Color
```typescript
// packages/core/src/types/Color.ts
export type PossibleColor = SerializedColor | number | Color | ColorObject;

export interface ColorObject {
  r: number;
  g: number;
  b: number;
  a: number;
}

export class Color implements Type, WebGLConvertible {
  public readonly r: number;
  public readonly g: number;
  public readonly b: number;
  public readonly a: number;

  public static lerp(from: Color, to: Color, value: number): Color {
    // Uses culori library for interpolation in different color spaces
    return new Color({
      r: map(from.r, to.r, value),
      g: map(from.g, to.g, value),
      b: map(from.b, to.b, value),
      a: map(from.a, to.a, value),
    });
  }

  public serialize(): SerializedColor {
    return `rgba(${Math.round(this.r * 255)}, ${Math.round(this.g * 255)}, ${Math.round(this.b * 255)}, ${this.a})`;
  }
}
```

#### BBox (Bounding Box)
```typescript
// packages/core/src/types/BBox.ts
export class BBox implements Type {
  public readonly x: number;
  public readonly y: number;
  public readonly width: number;
  public readonly height: number;

  public get left(): number {
    return this.x;
  }

  public get right(): number {
    return this.x + this.width;
  }

  public get top(): number {
    return this.y;
  }

  public get bottom(): number {
    return this.y + this.height;
  }

  public get center(): Vector2 {
    return new Vector2(
      this.x + this.width / 2,
      this.y + this.height / 2,
    );
  }

  public contains(point: Vector2): boolean {
    return (
      point.x >= this.left &&
      point.x <= this.right &&
      point.y >= this.top &&
      point.y <= this.bottom
    );
  }

  public intersects(other: BBox): boolean {
    return !(
      other.left > this.right ||
      other.right < this.left ||
      other.top > this.bottom ||
      other.bottom < this.top
    );
  }
}
```

## Event System

### Event Dispatchers
```typescript
// packages/core/src/events/EventDispatcher.ts
export class EventDispatcher<T> {
  private handlers: ((value: T) => void)[] = [];

  public subscribe(handler: (value: T) => void): () => void {
    this.handlers.push(handler);
    return () => {
      const index = this.handlers.indexOf(handler);
      if (index >= 0) {
        this.handlers.splice(index, 1);
      }
    };
  }

  public dispatch(value: T): void {
    for (const handler of this.handlers) {
      handler(value);
    }
  }
}

// ValueDispatcher for value change notifications
export class ValueDispatcher<T> extends EventDispatcher<T> {
  private current: T;

  public constructor(initial: T) {
    super();
    this.current = initial;
  }

  public get value(): T {
    return this.current;
  }

  public set value(value: T) {
    if (this.current !== value) {
      this.current = value;
      this.dispatch(value);
    }
  }
}

// AsyncEventDispatcher for async handlers
export class AsyncEventDispatcher<T> {
  private handlers: ((value: T) => Promise<void>)[] = [];

  public async dispatch(value: T): Promise<void> {
    await Promise.all(this.handlers.map(handler => handler(value)));
  }
}
```

## Plugin System

### Plugin Interface
```typescript
// packages/core/src/plugin/Plugin.ts
export interface Plugin {
  name: string;
  player?(player: Player): void;
  renderer?(renderer: Renderer): void;
  project?(project: Project): void;
  config?(config: PluginConfig): void | Promise<void>;
}

export interface PluginConfig {
  output: string;
  projects: ProjectData[];
}
```

### Default Plugin
```typescript
// packages/core/src/plugin/DefaultPlugin.ts
export class DefaultPlugin implements Plugin {
  public readonly name = 'revideo-core-plugin';

  public project(project: Project) {
    // Register default exporters
    project.exporters.push(
      ImageExporter,
      WasmExporter,
    );
  }
}
```

## Project Configuration

### Project Settings
```typescript
// packages/core/src/app/Project.ts
export interface ProjectSettings {
  shared: {
    background: Color | string;
    range: [number, number];
    size: Vector2;
  };
  rendering: {
    exporter: ExporterSettings;
    fps: number;
    resolutionScale: number;
    colorSpace: CanvasColorSpace;
  };
  preview: {
    fps: number;
    resolutionScale: number;
  };
}

export type ExporterSettings =
  | {name: '@revideo/core/image-sequence'; options: ImageExporterOptions}
  | {name: '@revideo/ffmpeg'; options: FfmpegExporterOptions}
  | {name: '@revideo/core/wasm'};
```

### Project Class
```typescript
// packages/core/src/app/Project.ts
export class Project {
  public readonly name: string;
  public readonly scenes: Scene[];
  public readonly settings: ProjectSettings;
  public readonly exporters: Exporter[] = [];
  public readonly plugins: Plugin[] = [];

  public constructor(config: ProjectConfig) {
    this.name = config.name ?? 'project';
    this.settings = config.settings ?? defaultSettings;
    this.scenes = config.scenes.map(sceneConfig => {
      if (typeof sceneConfig === 'function') {
        return new GeneratorScene({
          config: sceneConfig,
          name: sceneConfig.name ?? 'scene',
        });
      }
      return new sceneConfig.klass(sceneConfig);
    });

    // Initialize plugins
    for (const plugin of this.plugins) {
      plugin.project?.(this);
    }
  }
}
```

## Architecture Patterns

### Decorator Pattern
```typescript
// packages/core/src/decorators/threadable.ts
export function threadable(customName?: string): MethodDecorator {
  return function (
    _: unknown,
    propertyKey: string | symbol,
    descriptor: PropertyDescriptor,
  ) {
    descriptor.value.prototype.name = customName ?? propertyKey;
    descriptor.value.prototype.threadable = true;
  };
}

// Usage
@threadable('myAnimation')
function* animate() {
  // Animation code
}
```

### Signal Decorators
```typescript
// packages/2d/src/lib/decorators/signal.ts
export function signal<T>(): PropertyDecorator {
  return function (target: any, propertyKey: string | symbol) {
    const signalSymbol = Symbol.for(`__signal_${String(propertyKey)}`);

    Object.defineProperty(target, propertyKey, {
      get() {
        if (!this[signalSymbol]) {
          this[signalSymbol] = createSignal();
        }
        return this[signalSymbol];
      },
      set(value) {
        this[propertyKey](value);
      },
    });
  };
}

// Usage
class MyNode extends Node {
  @signal()
  public declare readonly myProperty: SimpleSignal<number, this>;
}
```

### Lazy Evaluation Pattern
```typescript
// packages/core/src/decorators/lazy.ts
export function lazy<T>(factory: () => T): PropertyDecorator {
  return function (target: any, propertyKey: string | symbol) {
    const valueSymbol = Symbol.for(`__lazy_${String(propertyKey)}`);

    Object.defineProperty(target, propertyKey, {
      get() {
        if (!this[valueSymbol]) {
          this[valueSymbol] = factory.call(this);
        }
        return this[valueSymbol];
      },
    });
  };
}
```

## Performance Optimizations

### Caching System
- Node-level rendering cache with configurable padding
- Scene-level cache metadata tracking
- Dirty flag optimization for signal updates

### Dependency Tracking
- Signals only recalculate when dependencies change
- Circular dependency detection
- Promise collection for async operations

### Multi-Threading
- Independent thread time tracking
- Fixed-time synchronization across threads
- Pausing and cancellation support

## Asset Management

### AssetInfo Structure
```typescript
// packages/core/src/app/AssetInfo.ts
export interface AssetInfo {
  key: string;
  type: 'video' | 'audio';
  src: string;
  playbackRate: number;
  volume: number;
  currentTime: number;
  duration: number;
  decoder?: string | null;
}
```

### Media Components
```typescript
// packages/2d/src/lib/components/Media.ts
export abstract class Media extends Rect {
  @signal()
  public declare readonly src: SimpleSignal<string, this>;

  @signal()
  public declare readonly time: SimpleSignal<number, this>;

  @signal()
  public declare readonly playing: SimpleSignal<boolean, this>;

  protected abstract getElement(): HTMLMediaElement;

  public play(): void {
    this.playing(true);
    this.getElement().play();
  }

  public pause(): void {
    this.playing(false);
    this.getElement().pause();
  }

  public seek(time: number): void {
    this.time(time);
    this.getElement().currentTime = time;
  }
}
```

## Summary

The Revideo architecture is built on:

1. **Signal-Based Reactivity**: Comprehensive reactive programming model with automatic dependency tracking
2. **Generator-Based Control Flow**: Intuitive animation sequencing using JavaScript generators
3. **Thread System**: Concurrent animation execution with hierarchical thread management
4. **Component System**: Flexible 2D rendering components with extensive property control
5. **Plugin Architecture**: Modular build system using Vite plugins
6. **Export Pipeline**: Multi-format export capabilities with FFmpeg integration
7. **Type Safety**: Full TypeScript coverage with comprehensive type definitions
8. **Performance**: Optimized rendering with caching, dependency tracking, and WebGL support

Total: 398 TypeScript files across 17 packages implementing a complete animation framework.