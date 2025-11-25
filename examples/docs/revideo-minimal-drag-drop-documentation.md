# Comprehensive Documentation: minimal-drag-drop Example

## File Structure

```
minimal-drag-drop/
├── app/                        # Next.js app directory
│   ├── api/
│   │   └── render/
│   │       └── route.ts       # API endpoint for video rendering
│   ├── globals.css            # Global styles with Tailwind directives
│   ├── layout.tsx             # Root layout with metadata
│   └── page.tsx               # Main React component with drag-drop UI
├── revideo/                    # Revideo project files
│   ├── global.css             # Roboto font import for video
│   └── project.tsx            # Revideo scene definition
├── utils/
│   └── parse.ts               # Stream parsing utility
├── next-env.d.ts              # Next.js TypeScript definitions
├── next.config.mjs            # Next.js configuration
├── package.json               # Dependencies and scripts
├── postcss.config.js          # PostCSS configuration
├── README.md                  # Basic project documentation
├── tailwind.config.ts         # Tailwind CSS configuration
└── tsconfig.json              # TypeScript configuration
```

## Complete Source Code

### app/page.tsx

```typescript
'use client';

import {Player} from '@revideo/player-react';
import {useState, useEffect, useRef} from 'react';
import {LoaderCircle} from 'lucide-react';
import { parseStream } from '../utils/parse';
import {Player as CorePlayer} from '@revideo/core';
import {Scene2D, Shape} from '@revideo/2d';
import project from '@/revideo/project';


function Button({
	children,
	loading,
	onClick,
}: {
	children: React.ReactNode;
	loading: boolean;
	onClick: () => void;
}) {
	return (
		<button
			className="text-sm flex items-center gap-x-2 rounded-md p-2 bg-gray-200 text-gray-700 hover:bg-gray-300"
			onClick={() => onClick()}
			disabled={loading}
		>
			{loading && <LoaderCircle className="animate-spin h-4 w-4 text-gray-700" />}
			{children}
		</button>
	);
}

export default function Home() {
	const [nodes, setNodes] = useState<Map<string, any>>(new Map([
		['txt', { text: "Text 1", x: 0, y: 0 }]
	  ]));
	const [draggingNode, setDraggingNode] = useState<Shape | null>(null);
	const [dragOffset, setDragOffset] = useState<{ x: number, y: number } | null>(null);
	const [playerRect, setPlayerRect] = useState<DOMRect | null>(null);
	const [corePlayer, setCorePlayer] = useState<CorePlayer | null>(null);
	const [transformMatrix, setTransformMatrix] = useState<DOMMatrix | null>(null);

	const [renderLoading, setRenderLoading] = useState(false);
	const [progress, setProgress] = useState(0);
	const [downloadUrl, setDownloadUrl] = useState<string | null>(null);

	const [playing, setPlaying] = useState<boolean>(false);
	const [dimensions, setDimensions] = useState<{x: number, y: number}>({ x: 1920, y: 1080 });
	const [selectedNode, setSelectedNode] = useState<{
		key: string;
		x: number;
		y: number;
		width: number;
		height: number;
	  } | null>(null);

	  const playerContainerRef = useRef<HTMLDivElement>(null);

	useEffect(() => {
		console.log("changing", "playerrect", "dimensions", playerRect, dimensions);
	  if (playerRect) {
		setTransformMatrix(createTransformationMatrix(playerRect, dimensions));
	  }
	}, [playerRect, dimensions]);

	const handlePlayerReady = (player: CorePlayer) => {
		setCorePlayer(player);
	};

	const addTextNode = () => {
		setNodes(prevNodes => {
		  const newNodes = new Map(prevNodes);
		  const newKey = `txt_${newNodes.size+1}`;
		  newNodes.set(newKey, { text: `Text ${newNodes.size+1}`, x: 0, y: 0 });
		  return newNodes;
		});
	  };

	const handlePointerDown = (event: React.PointerEvent<HTMLDivElement>) => {
		console.log("hiiii")
		console.log("playerrect", playerRect);
		console.log("transformmatrix", transformMatrix);
		if (playerRect && transformMatrix) {
			const { x: playerX, y: playerY } = transformPoint(
			event.clientX,
			event.clientY,
			transformMatrix
			);

			console.log("AHHH")

			console.log("pos", playerX, playerY);

			let hitNode = (corePlayer?.playback.currentScene as Scene2D).getNodeByPosition(playerX, playerY) as Shape;
			while (hitNode) {
				console.log("joo");
				// We want to avoid getting TxtLeaf nodes, but only their parent Txt nodes
				if (nodes.has(hitNode.key)) {
					console.log("jaaa", hitNode.key);
					setDraggingNode(hitNode);
					const node = nodes.get(hitNode.key);
					setDragOffset({ x: playerX - node.x, y: playerY - node.y });

					const position = hitNode.position();
					const { x, y } = transformPoint(position.x+dimensions.x/2, position.y+dimensions.y/2, transformMatrix.inverse());

					setSelectedNode({
					key: hitNode.key,
					x: x,
					y: y,
					width: selectedNode?.width || hitNode.width() / transformMatrix.a,
					height: selectedNode?.height || hitNode.height() / transformMatrix.d
					});
					return;
				}
			hitNode = hitNode.parent() as Shape;
			}
			setSelectedNode(null);
		};
	}

	const handlePointerMove = (event: React.PointerEvent<HTMLDivElement>) => {
		if (draggingNode && playerRect && transformMatrix) {
			const { x: playerX, y: playerY } = transformPoint(
				event.clientX,
				event.clientY,
				transformMatrix
			);

			setNodes(prevNodes => {
				const newNodes = new Map(prevNodes);
				const node = newNodes.get(draggingNode.key);
				if (node && dragOffset) {
					node.x = playerX - dragOffset.x;
					node.y = playerY - dragOffset.y;

					const { x, y } = transformPoint(node.x+dimensions.x/2, node.y+dimensions.y/2, transformMatrix.inverse());
					setSelectedNode({
					key: draggingNode.key,
					x: x,
					y: y,
					width: selectedNode?.width || draggingNode.width() / transformMatrix.a,
					height: selectedNode?.height || draggingNode.height() / transformMatrix.d,
					});

				}
				return newNodes;
			});
		}
	};

	function createTransformationMatrix(
		playerRect: DOMRect,
		dimensions: { x: number; y: number }
	  ): DOMMatrix {
		const scaleX = dimensions.x / playerRect.width;
		const scaleY = dimensions.y / playerRect.height;

		const matrix = new DOMMatrix();
		matrix.scaleSelf(scaleX, scaleY);
		matrix.translateSelf(-playerRect.left, -playerRect.top);

		return matrix;
	  }

	function transformPoint(
		x: number,
		y: number,
		matrix: DOMMatrix
	  ): { x: number; y: number } {
		const point = new DOMPoint(x, y);
		const transformedPoint = point.matrixTransform(matrix);
		return { x: transformedPoint.x, y: transformedPoint.y };
	  }

	/**
	 * Render the video.
	 */
	async function render() {
		setRenderLoading(true);
		const res = await fetch('/api/render', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json',
			},
			body: JSON.stringify({
				variables: {
					nodes: Object.fromEntries(nodes)
				},
				streamProgress: true,
			}),
		}).catch((e) => console.log(e));

		if (!res) {
			alert('Failed to render video.');
			return;
		}

		const downloadUrl = await parseStream(res.body!.getReader(), (p) => setProgress(p));
		setRenderLoading(false);
		setDownloadUrl(downloadUrl);
	}

	return (
		<>
		  <div className="m-auto p-12 max-w-6xl flex flex-col gap-y-4">
			<div
			  className="rounded-lg overflow-hidden w-[100%] py-20 bg-gray-100 flex items-center justify-center"
			  onPointerDown={handlePointerDown}
			  onPointerMove={handlePointerMove}
			  onPointerUp={() => {
				setDraggingNode(null);
				setDragOffset(null);
			  }}
			>
			  <div className='w-[80%] h-[80%]' ref={playerContainerRef}>
				<Player
				  project={project}
				  controls={false}
				  playing={playing}
				  width={dimensions.x}
				  height={dimensions.y}
				  variables={{
					nodes: Object.fromEntries(nodes),
				  }}
				  onPlayerReady={handlePlayerReady}
				  onPlayerResize={(rect: DOMRect) => {
					setPlayerRect(rect);
				  }}
				/>
				{(selectedNode && playerRect) && (
				  <div
					className="absolute border-2 border-[#5bbad5] pointer-events-none"
					style={{
					  left: `${selectedNode.x-selectedNode.width/2}px`,
					  top: `${selectedNode.y-selectedNode.height/2}px`,
					  width: `${selectedNode.width}px`,
					  height: `${selectedNode.height}px`,
					}}
				  />
				)}
			  </div>
			</div>
				<div className="flex gap-x-4">
					{/* Progress bar */}
					<div className="text-sm flex-1 bg-gray-100 rounded-md overflow-hidden">
						<div
							className="text-gray-600 bg-gray-400 h-full flex items-center px-4 transition-all transition-200"
							style={{
								width: `${Math.round(progress * 100)}%`,
							}}
						>
							{Math.round(progress * 100)}%
						</div>
					</div>
					{downloadUrl ? (
						<a
							href={downloadUrl}
							download
							className="text-sm flex items-center gap-x-2 rounded-md p-2 bg-green-200 text-gray-700 hover:bg-gray-300"
						>
							Download video
						</a>
					) : (
						<>
						<Button
								loading={false}
								onClick={() => {
									setPlaying(!playing);
									setSelectedNode(null);
								}}
							>
								{playing ? "Pause" : "Play"}
						</Button>

						<Button onClick={addTextNode} loading={false}>
							Add Text
						</Button>

						<Button onClick={() => render()} loading={renderLoading}>
							Render video
						</Button>
						</>
					)}
				</div>
			</div>
		</>
	);
}
```

### app/layout.tsx

```typescript
import type {Metadata} from 'next';
import {Inter} from 'next/font/google';
import './globals.css';

const inter = Inter({subsets: ['latin']});

export const metadata: Metadata = {
	title: 'Create Next App',
	description: 'Generated by create next app',
};

export default function RootLayout({
	children,
}: Readonly<{
	children: React.ReactNode;
}>) {
	return (
		<html lang="en">
			<head>
				<link rel="stylesheet" href="http://localhost:4000/player/project.css" />
			</head>
			<body className={inter.className}>{children}</body>
		</html>
	);
}
```

### app/globals.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### app/api/render/route.ts

```typescript
const RENDER_URL = 'http://localhost:4000/render';

async function getResponse(body: string) {
	return await fetch(RENDER_URL, {
		method: 'POST',
		headers: {
			'Content-Type': 'application/json',
		},
		body,
	});
}

export async function POST(request: Request) {
	const body = await request.json();

	const response = await getResponse(JSON.stringify(body));
	if (!response.ok) {
		return new Response('Failed to render', {status: 500});
	}

	return new Response(response.body, {status: 200});
}
```

### revideo/project.tsx

```typescript
/** @jsxImportSource @revideo/2d/lib */

import {
    createRef,
    Reference,
	  makeProject,
    useScene,
  	waitFor,
} from '@revideo/core';
import {Video, Txt, makeScene2D} from '@revideo/2d';

import './global.css';

const scene = makeScene2D("scene", function* (view) {
  const nodesObj = useScene().variables.get("nodes", {})() as Record<string, any>;
  const nodes = new Map(Object.entries(nodesObj));

  yield view.add(
    <Video
      src="https://revideo-example-assets.s3.amazonaws.com/stars.mp4"
      size={"100%"}
      key={'data-video'}
      play={true}
    />
  );

  const refs: Array<Reference<Txt>> = [];
  if (nodes.size > 0) {
    for (const [key, node] of nodes) {
      const ref = createRef<Txt>();
      yield view.add(
        <Txt
          key={key}
          text={node.text}
          fontSize={100}
          fontWeight={800}
          fontFamily={"Roboto"}
          fill={"white"}
          x={() => node.x}
          y={() => node.y}
          ref={ref}
        />
      )
      refs.push(ref);
    }
  }

  yield* waitFor(5);
});


export default makeProject({
	scenes: [scene],
});
```

### revideo/global.css

```css
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');
```

### utils/parse.ts

```typescript
export async function parseStream(
	reader: ReadableStreamDefaultReader<Uint8Array>,
	updateProgress: (progress: number) => void,
) {
	while (true) {
		const {done, value} = await reader.read();

		const decoded = new TextDecoder('utf-8').decode(value);
		const split = decoded.split('\n');

		if (done) {
			break;
		}

		const event = split[0].slice(6).trim();
		const data = split[1].slice(6).trim();

		if (event === 'progress') {
			const parsed = JSON.parse(data);
			updateProgress(parsed.progress);
		}

		if (event === 'completed') {
			const parsed = JSON.parse(data);
			return parsed.downloadLink as string;
		}

		if (event === 'error') {
			console.error(data);
			break;
		}
	}

	return '';
}
```

## Main Functionality and Flow

### 1. Interactive Drag-and-Drop Interface (app/page.tsx)
- Implements a React-based UI for dragging text elements over a video canvas
- Manages text nodes as a Map with x/y coordinates
- Handles pointer events for drag-and-drop functionality
- Shows selection border around active elements

### 2. Video Scene Composition (revideo/project.tsx)
- Creates a Revideo scene with background video
- Dynamically adds text nodes based on user input
- Each text node is positioned according to drag coordinates
- Scene duration set to 5 seconds

### 3. Rendering Pipeline
- Client sends render request to `/api/render` endpoint
- API route forwards to Revideo render server at `http://localhost:4000/render`
- Progress updates streamed back to client
- Final video downloadable when complete

## Key Revideo Concepts Demonstrated

### Core Imports and Functions (revideo/project.tsx)
- `makeProject` - Creates the Revideo project configuration
- `makeScene2D` - Creates a 2D scene for compositing
- `createRef` - Creates references to scene elements
- `useScene` - Accesses scene variables from external data
- `waitFor` - Adds timing/duration to the scene

### 2D Components (revideo/project.tsx)
- `Video` - Background video component with playback control
- `Txt` - Text elements with font styling properties
- `Scene2D` - 2D scene context for element positioning

### Player Integration (app/page.tsx)
- `@revideo/player-react` - React component for video preview
- Real-time variable updates for dynamic content
- Player resize detection for coordinate transformation
- Control over playback state

### Scene Variables (revideo/project.tsx)
- Dynamic data passed via `useScene().variables`
- Reactive properties using arrow functions `x={() => node.x)`

## Important Code Patterns and Techniques

### 1. Coordinate Transformation (app/page.tsx:152-174)
```typescript
function createTransformationMatrix(
  playerRect: DOMRect,
  dimensions: { x: number; y: number }
): DOMMatrix {
  const scaleX = dimensions.x / playerRect.width;
  const scaleY = dimensions.y / playerRect.height;

  const matrix = new DOMMatrix();
  matrix.scaleSelf(scaleX, scaleY);
  matrix.translateSelf(-playerRect.left, -playerRect.top);

  return matrix;
}

function transformPoint(
  x: number,
  y: number,
  matrix: DOMMatrix
): { x: number; y: number } {
  const point = new DOMPoint(x, y);
  const transformedPoint = point.matrixTransform(matrix);
  return { x: transformedPoint.x, y: transformedPoint.y };
}
```
- Converts browser click coordinates to video canvas coordinates
- Uses DOMMatrix for scaling and translation
- Handles player container resizing

### 2. Hit Detection (app/page.tsx:94-116)
```typescript
let hitNode = (corePlayer?.playback.currentScene as Scene2D).getNodeByPosition(playerX, playerY) as Shape;
while (hitNode) {
  if (nodes.has(hitNode.key)) {
    setDraggingNode(hitNode);
    const node = nodes.get(hitNode.key);
    setDragOffset({ x: playerX - node.x, y: playerY - node.y });
    // ... position calculations
    return;
  }
  hitNode = hitNode.parent() as Shape;
}
```
- Uses `Scene2D.getNodeByPosition()` to find elements at coordinates
- Traverses parent hierarchy to find draggable nodes
- Filters out child nodes (TxtLeaf) to get parent Txt nodes

### 3. Stream Processing (utils/parse.ts)
```typescript
export async function parseStream(
  reader: ReadableStreamDefaultReader<Uint8Array>,
  updateProgress: (progress: number) => void,
) {
  while (true) {
    const {done, value} = await reader.read();

    const decoded = new TextDecoder('utf-8').decode(value);
    const split = decoded.split('\n');

    if (done) break;

    const event = split[0].slice(6).trim();
    const data = split[1].slice(6).trim();

    if (event === 'progress') {
      const parsed = JSON.parse(data);
      updateProgress(parsed.progress);
    }

    if (event === 'completed') {
      const parsed = JSON.parse(data);
      return parsed.downloadLink as string;
    }

    if (event === 'error') {
      console.error(data);
      break;
    }
  }

  return '';
}
```
- Parses Server-Sent Events for progress updates
- Extracts progress percentages and download links
- Handles error events from render process

### 4. State Management Patterns
- Uses React hooks for local state management
- Maps for dynamic node collections
- Refs for direct DOM/player access

## Dependencies and Configuration

### Core Dependencies (package.json)
```json
{
  "@revideo/2d": "0.10.3",
  "@revideo/core": "0.10.3",
  "@revideo/player-react": "0.10.3",
  "next": "^14.2.13",
  "react": "^18",
  "tailwindcss": "^3.4.3",
  "lucide-react": "^0.378.0"
}
```

### Development Tools (package.json)
```json
{
  "@revideo/cli": "0.10.3",
  "@revideo/ui": "0.10.3",
  "typescript": "^5",
  "@types/react": "^18",
  "@types/react-dom": "^18"
}
```

### Build Configuration
- Next.js with App Router
- TypeScript with strict mode enabled
- Tailwind CSS with custom gray color palette (tailwind.config.ts)
- PostCSS with autoprefixer

### Available Scripts (package.json)
- `npm run dev` - Start Next.js development server
- `npm run build` - Build Next.js project
- `npm run start` - Start Next.js production server
- `npm run revideo:serve` - Start Revideo render API
- `npm run revideo:editor` - Start Revideo editor

## Assets Used

### 1. Video Asset (revideo/project.tsx:20)
- URL: `https://revideo-example-assets.s3.amazonaws.com/stars.mp4`
- Used as background video that plays automatically
- Full screen size (`size="100%"`)

### 2. Font Asset (revideo/global.css)
- Google Fonts: Roboto (weights 400, 700)
- Applied to text elements in the video
- Import URL: `https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap`

### 3. External Stylesheet (app/layout.tsx:20)
- Links to `http://localhost:4000/player/project.css`
- Provides styles for the Revideo player component

## Key Features Implemented

1. **Dynamic Text Addition** - Users can add multiple text nodes via "Add Text" button
2. **Real-time Preview** - Changes reflected immediately in player without rendering
3. **Drag-and-Drop Positioning** - Interactive element placement with pointer events
4. **Visual Selection** - Blue border highlight (#5bbad5) for active elements
5. **Video Rendering** - Export final composition as downloadable video file
6. **Progress Tracking** - Real-time render progress display with percentage
7. **Play/Pause Control** - Video playback management with state tracking

## Component State Variables (app/page.tsx)

- `nodes`: Map of text nodes with positions and content
- `draggingNode`: Currently dragged Shape element
- `dragOffset`: Offset between pointer and node position
- `playerRect`: DOMRect of player container
- `corePlayer`: Reference to Revideo Core Player instance
- `transformMatrix`: DOMMatrix for coordinate conversion
- `renderLoading`: Rendering status flag
- `progress`: Render progress (0-1)
- `downloadUrl`: URL of rendered video
- `playing`: Playback state
- `dimensions`: Video dimensions (1920x1080)
- `selectedNode`: Currently selected node with position/size

## API Routes

### POST /api/render/route.ts
- Accepts JSON body with variables and streamProgress flag
- Forwards request to Revideo render server
- Returns streaming response with progress updates
- Error handling returns 500 status on failure

## Project Structure Details

### Revideo Project (revideo/project.tsx)
- Single scene defined with `makeScene2D`
- Generator function pattern for scene definition
- Dynamic node creation from variables
- Fixed 5-second duration with `waitFor`

### React Integration (app/page.tsx)
- Client-side component (`'use client'`)
- Player component with custom controls
- Event handlers for drag-and-drop
- Responsive layout with Tailwind CSS

### Styling
- Tailwind CSS for UI components
- Custom gray color palette
- Responsive design with flexbox
- Rounded corners and hover states

## Technical Implementation Notes

- Coordinate system transformation required for accurate drag positioning
- Parent node traversal needed to avoid selecting text leaf nodes
- Server-Sent Events used for real-time progress updates
- DOMMatrix operations for scaling between screen and video coordinates
- React state updates trigger re-renders of Revideo player variables
- Background video plays continuously during interaction