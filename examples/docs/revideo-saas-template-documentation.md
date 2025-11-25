# Revideo SaaS Template - Complete Documentation

## Overview

This documentation provides a comprehensive analysis of the saas-template project, which demonstrates how to integrate Revideo with a Next.js application to create a SaaS-style video generation platform. The template showcases a GitHub stars visualization tool that fetches repository data via the GitHub API, displays an interactive preview using the Revideo player, and allows users to render and download customized videos showing the growth of stars over time.

## File Structure

```
saas-template/
├── app/                              # Next.js App Router directory
│   ├── api/
│   │   └── render/
│   │       └── route.ts             # API endpoint for video rendering
│   ├── actions.tsx                  # Server actions for GitHub API
│   ├── globals.css                  # Global Tailwind CSS imports
│   ├── layout.tsx                   # Root layout component
│   └── page.tsx                     # Main page component with UI
├── revideo/                         # Revideo animation files
│   ├── global.css                   # Revideo-specific CSS
│   ├── project.ts                   # Revideo project configuration
│   └── scene.tsx                    # Main animation scene
├── utils/
│   └── parse.ts                     # Stream parsing utilities
├── .gitignore                       # Git ignore configuration
├── .prettierrc                      # Prettier formatting config
├── next.config.mjs                  # Next.js configuration
├── package.json                     # Project dependencies
├── postcss.config.js                # PostCSS configuration
├── README.md                        # Project documentation
├── tailwind.config.ts               # Tailwind CSS configuration
└── tsconfig.json                   # TypeScript configuration
```

## Dependencies and Configuration

### Core Dependencies
- **@revideo/2d**: 0.10.3 - 2D components and utilities for Revideo
- **@revideo/core**: 0.10.3 - Core Revideo functionality
- **@revideo/player-react**: 0.10.3 - React component for Revideo player
- **next**: ^14.2.13 - Next.js framework
- **react**: ^18 - React library
- **react-dom**: ^18 - React DOM rendering
- **p-limit**: ^3 - Promise concurrency limiter

### UI Dependencies
- **@radix-ui/react-navigation-menu**: ^1.1.4 - Navigation components
- **class-variance-authority**: ^0.7.0 - CSS class utilities
- **lucide-react**: ^0.378.0 - Icon library
- **tailwind-merge**: ^2.3.0 - Tailwind class merging
- **tailwindcss-animate**: ^1.0.7 - Animation utilities

### Development Dependencies
- **@revideo/cli**: 0.10.3 - Revideo command-line interface
- **@revideo/ui**: 0.10.3 - Revideo editor UI
- **autoprefixer**: ^10.4.19 - CSS vendor prefixing
- **tailwindcss**: ^3.4.3 - Utility-first CSS framework
- **typescript**: ^5 - TypeScript compiler

### Scripts
- `dev`: Start Next.js development server
- `build`: Build Next.js production bundle
- `start`: Start Next.js production server
- `lint`: Run ESLint
- `revideo:serve`: Start Revideo render API server (port 4000)
- `revideo:editor`: Launch Revideo editor interface

## Source Code Analysis

### 1. Main Page Component (app/page.tsx)

```typescript
'use client';

import {Player} from '@revideo/player-react';
import {getGithubRepositoryInfo} from './actions';
import {useState} from 'react';
import {LoaderCircle} from 'lucide-react';
import {parseStream} from '../utils/parse';
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

function RenderComponent({
	stargazerTimes,
	repoName,
	repoImage,
}: {
	stargazerTimes: number[];
	repoName: string;
	repoImage: string | null | undefined;
}) {
	const [renderLoading, setRenderLoading] = useState(false);
	const [progress, setProgress] = useState(0);
	const [downloadUrl, setDownloadUrl] = useState<string | null>(null);

	/**
	 * Render the video.
	 */
	async function render() {
		setRenderLoading(true);
		const res = await fetch('/api/render', {
			method: 'POST',
			headers: {
				// eslint-disable-next-line @typescript-eslint/naming-convention
				'Content-Type': 'application/json',
			},
			body: JSON.stringify({
				variables: {
					data: stargazerTimes.length ? stargazerTimes : undefined,
					repoName: repoName || undefined,
					repoImage: repoImage || undefined,
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
				<Button onClick={() => render()} loading={renderLoading}>
					Render video
				</Button>
			)}
		</div>
	);
}

export default function Home() {
	const [repoName, setRepoName] = useState<string>('');
	const [repoImage, setRepoImage] = useState<string | null>();
	const [stargazerTimes, setStargazerTimes] = useState<number[]>([]);

	const [githubLoading, setGithubLoading] = useState(false);
	const [needsKey, setNeedsKey] = useState(false);
	const [key, setKey] = useState('');
	const [error, setError] = useState<string | null>();

	/**
	 * Get information about the repository from Github.
	 */
	async function fetchInformation(repoName: `${string}/${string}`, key: string) {
		setGithubLoading(true);
		const response = await getGithubRepositoryInfo(repoName, key ?? undefined);
		setGithubLoading(false);

		if (response.status === 'rate-limit') {
			setNeedsKey(true);
			return;
		}

		if (response.status === 'error') {
			setError('Failed to fetch repository information from Github.');
			return;
		}

		setStargazerTimes(response.stargazerTimes);
		setRepoImage(response.repoImage);
	}

	return (
		<>
			<div className="m-auto p-12 max-w-7xl flex flex-col gap-y-4">
				<div>
					<div className="text-sm text-gray-700 mb-2">Repository</div>
					<div className="flex gap-x-4 text-sm">
						<input
							className="flex-1 rounded-md p-2 bg-gray-200 focus:outline-none placeholder:text-gray-400"
							placeholder="redotvideo/revideo"
							value={repoName}
							onChange={(e) => setRepoName(e.target.value)}
						/>
						{!needsKey && (
							<Button
								loading={githubLoading}
								onClick={() => fetchInformation(repoName as `${string}/${string}`, key)}
							>
								Fetch information
							</Button>
						)}
					</div>
				</div>
				{needsKey && (
					<div>
						<div className="text-sm text-blue-600 mb-2">
							You hit the Github API rate-limit. Please provide your own key. Requests to Github are
							made directly and the key stays on your device.
						</div>
						<div className="flex gap-x-4 text-sm">
							<input
								className="flex-1 rounded-md p-2 bg-gray-200 focus:outline-none placeholder:text-gray-400"
								placeholder="ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
								value={key}
								onChange={(e) => setKey(e.target.value)}
							/>
							<Button
								loading={githubLoading}
								onClick={() => fetchInformation(repoName as `${string}/${string}`, key)}
							>
								Fetch information
							</Button>
						</div>
					</div>
				)}
				{error && <div className="text-sm text-red-600">{error}</div>}
				<div>
					<div className="rounded-lg overflow-hidden">
						{/* You can find the scene code inside revideo/src/scenes/example.tsx */}
						<Player
							project={project}
							controls={true}
							variables={{
								data: stargazerTimes.length > 0 ? stargazerTimes : undefined,
								repoName: repoName ? repoName : undefined,
								repoImage: repoImage ? repoImage : undefined,
							}}
						/>
					</div>
				</div>
				<RenderComponent
					stargazerTimes={stargazerTimes}
					repoName={repoName}
					repoImage={repoImage}
				/>
			</div>
		</>
	);
}
```

### 2. Server Actions (app/actions.tsx)

```typescript
'use server';

import pLimit from 'p-limit';

const PER_PAGE = 30;

interface ReturnType {
	stargazerTimes: number[];
	status: 'success' | 'rate-limit' | 'error';
}

async function getStargazerTimesForPage(
	repo: `${string}/${string}`,
	page: number,
	headers: HeadersInit,
): Promise<ReturnType> {
	const response = await fetch(
		`https://api.github.com/repos/${repo}/stargazers?page=${page}&per_page=${PER_PAGE}`,
		{
			headers: {
				// eslint-disable-next-line @typescript-eslint/naming-convention
				Accept: 'application/vnd.github.v3.star+json',
				...headers,
			},
		},
	);

	if (response.status === 403) {
		return {stargazerTimes: [], status: 'rate-limit'};
	}

	if (!response.ok) {
		return {stargazerTimes: [], status: 'error'};
	}

	const data = await response.json();
	const dates: Date[] = data.map((stargazer: any) => new Date(stargazer.starred_at));
	const msTimes = dates.map((date) => date.getTime());

	return {
		stargazerTimes: msTimes,
		status: 'success',
	};
}

const limit = pLimit(10);

async function getTimes(
	repo: `${string}/${string}`,
	totalPages: number,
	headers: HeadersInit,
): Promise<ReturnType> {
	const stargazerPromises: Promise<ReturnType>[] = [];

	// Fetch all pages in parallel
	for (let page = 1; page <= totalPages; page++) {
		stargazerPromises.push(limit(() => getStargazerTimesForPage(repo, page, headers)));
	}

	const stargazerTimes = await Promise.all(stargazerPromises);

	// In case of failure, return the first error
	const objectsThatFailed = stargazerTimes.filter(({status}) => status !== 'success');
	if (stargazerTimes.some(({status}) => status !== 'success')) {
		return {
			stargazerTimes: [],
			status: objectsThatFailed[0].status,
		};
	}

	const stargazerMs = stargazerTimes.map(({stargazerTimes}) => stargazerTimes).flat();
	const sortedMs = stargazerMs.sort((a, b) => a - b);

	const firstMs = sortedMs[0];
	const relativeMs = sortedMs.map((ms) => ms - firstMs);

	return {
		stargazerTimes: relativeMs,
		status: 'success',
	};
}

export async function getGithubRepositoryInfo(
	repoName: `${string}/${string}`,
	key?: string,
): Promise<ReturnType & {repoImage: string}> {
	const headers: HeadersInit = {};

	if (key) {
		headers.Authorization = `token ${key}`;
	}

	const response = await fetch(`https://api.github.com/repos/${repoName}`, {
		headers,
	});

	if (response.status === 403) {
		return {stargazerTimes: [], repoImage: '', status: 'rate-limit'};
	}

	if (!response.ok) {
		return {stargazerTimes: [], repoImage: '', status: 'error'};
	}

	const responseParsed = await response.json();
	const pages = Math.ceil(responseParsed.stargazers_count / PER_PAGE);
	const info = await getTimes(repoName, pages, headers);

	return {
		stargazerTimes: info.stargazerTimes,
		repoImage: responseParsed.owner.avatar_url,
		status: info.status,
	};
}
```

### 3. Render API Route (app/api/render/route.ts)

```typescript
const RENDER_URL = 'http://localhost:4000/render';

async function getResponse(body: string) {
	return await fetch(RENDER_URL, {
		method: 'POST',
		headers: {
			// eslint-disable-next-line @typescript-eslint/naming-convention
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

### 4. Revideo Scene (revideo/scene.tsx)

```typescript
/** @jsxImportSource @revideo/2d/lib */
import {Gradient, Img, Layout, Line, Rect, Spline, Txt, makeScene2D} from '@revideo/2d';
import {all, createRef, useScene, Vector2, waitFor} from '@revideo/core';

/**
 * Some example data to use in the scene when no data is provided
 */
const exampleData = [
	0, 105826000, 265664000, 265671000, 265684000, 265689000, 265694000, 335596000, 414108000,
	416767000, 425249000, 674375000, 964912000, 1177339000, 1186686000, 1213498000, 1214781000,
	1214895000, 1219921000, 1221668000, 1222281000, 1222410000, 1222433000, 1227527000, 1228300000,
	1230497000, 1230501000, 1231234000, 1231239000, 1231314000, 1232727000, 1233520000, 1234669000,
	1236349000, 1236806000, 1237795000, 1245092000, 1251533000, 1254263000, 1262147000, 1262899000,
	1264370000, 1267519000, 1268870000, 1271198000, 1271847000, 1274347000, 1276515000, 1276671000,
	1279966000, 1280551000, 1283338000, 1283777000, 1285088000, 1286336000, 1286728000, 1293071000,
	1293863000, 1294963000, 1295005000, 1301398000, 1303551000, 1312541000, 1317615000, 1321096000,
	1323718000, 1337789000, 1343521000, 1344711000, 1346543000, 1371003000, 1389862000, 1494428000,
	1525657000, 1533978000, 1591597000, 1654009000, 1738062000, 1817754000, 1860276000, 1883450000,
	1883998000, 1891635000, 1930667000, 2055652000, 2201181000, 2216214000, 2246708000, 2324529000,
	2366960000, 2366996000, 2391904000, 2479357000, 2596772000, 2601046000, 2615944000, 2637502000,
	2689660000, 2733368000, 2737046000, 2812890000, 2863564000, 2955232000, 2955857000, 2961163000,
	2983003000, 2984020000, 2987437000, 2990281000, 2996230000, 3007072000, 3007175000, 3013062000,
	3016417000, 3018616000, 3019154000, 3026357000, 3029804000, 3036897000, 3037202000, 3037298000,
	3037737000, 3038639000, 3039016000, 3039322000, 3042145000, 3042529000, 3043490000, 3044558000,
	3046472000, 3047534000, 3048199000, 3048524000, 3048862000, 3051935000, 3058895000, 3065009000,
	3072790000, 3074109000, 3075041000, 3079153000, 3079875000, 3080083000, 3099026000, 3099874000,
	3103039000, 3109664000, 3112885000, 3127743000, 3134934000, 3140075000, 3173641000, 3173788000,
	3176205000, 3176720000, 3178996000, 3183882000, 3184287000, 3186716000, 3191153000, 3196320000,
	3196620000, 3198638000, 3213076000, 3234269000, 3263068000, 3270668000, 3278896000, 3284646000,
	3290556000, 3294331000, 3297742000, 3332896000, 3344044000, 3368767000, 3378844000, 3398863000,
	3434188000, 3435264000, 3435696000, 3435803000, 3442071000, 3517995000, 3519849000, 3554474000,
	3558289000, 3611948000, 3616675000, 3618144000, 3622775000, 3635577000, 3640600000, 3669592000,
	3679346000, 3698749000, 3715355000, 3729047000, 3759434000, 3787838000, 3801028000, 3817911000,
	3878742000, 3983973000, 4006119000, 4067980000, 4087451000, 4101992000, 4200703000, 4212009000,
	4212143000, 4212882000, 4213748000, 4213977000, 4214180000, 4214445000, 4220193000, 4220422000,
	4222468000, 4236874000, 4258899000, 4326288000, 4334389000, 4401276000, 4416803000, 4421444000,
	4437462000, 4501703000, 4556531000, 4598409000, 4690540000, 4736772000, 4742561000, 4803793000,
	4834054000, 4866127000, 4868886000, 4873958000, 4891455000, 4921352000, 4941837000, 4953890000,
	4956018000, 4979428000, 4985123000, 5062420000, 5108308000, 5333539000, 5459835000, 5461002000,
	5521826000, 5584695000, 5586217000, 5598497000, 5604962000, 5625413000, 5636146000, 5637453000,
	5682023000, 5718405000, 5722303000, 5760543000, 5769209000, 5772790000, 5880569000, 6110458000,
	6135651000, 6185575000, 6202790000, 6232205000, 6284650000, 6297771000, 6392835000, 6401789000,
	6480914000, 6566872000, 6643031000, 6695443000, 6696088000, 6710689000, 6712966000, 6727715000,
	6733098000, 6783253000, 6805478000, 6875821000, 6904033000, 6971558000, 6972735000,
];

const exampleRepoName = 'redotvideo/revideo';
const exampleRepoImage = 'https://avatars.githubusercontent.com/u/133898679';

export default makeScene2D('main', function* (view) {
	// Get variables
	const repoName = useScene().variables.get('repoName', exampleRepoName);
	const repoImage = useScene().variables.get('repoImage', exampleRepoImage);
	const data = useScene().variables.get('data', exampleData);

	const max = Math.max(...data());
	const videoLength = 5; // seconds
	const totalValues = data().length;

	// Black background
	view.fill('#000000');

	// Calculate coordinates for each timestamp
	const linePoints = data().map((ms, i) => {
		const x = (ms / max) * view.width();
		const xShifted = x - view.width() / 2;

		const y = ((-i / totalValues) * view.height()) / 2;
		const yShifted = y + view.height() / 4;

		return new Vector2(xShifted, yShifted);
	});

	// Coordinates of the bottom corners
	const bottomCorners = [
		new Vector2(view.width() / 2, view.height() / 2),
		new Vector2(-view.width() / 2, view.height() / 2),
	];

	// Background gradient
	const gradient = new Gradient({
		type: 'linear',
		from: [0, 0],
		to: [0, view.height()],
		stops: [
			{offset: 0, color: '#000000'},
			{offset: 1, color: 'green'},
		],
	});

	// Refs, used to animate elements
	const outerLayoutRef = createRef<Layout>();
	const innerLayoutRef = createRef<Layout>();
	const rectRef = createRef<Rect>();

	// Add elements to the view
	yield view.add(
		<>
			<>
				<Line points={linePoints} lineWidth={30} stroke={'#3EAC45'} />
				<Spline points={[...linePoints, ...bottomCorners]} fill={gradient} />
				<Rect
					ref={rectRef}
					x={view.width() / 2}
					y={0}
					width={view.width() * 2}
					height={view.height()}
					fill={'#000000'}
				/>
			</>
			<Layout
				ref={outerLayoutRef}
				layout
				alignItems={'center'}
				gap={40}
				x={-870}
				y={-400}
				offset={[-1, 0]}
			>
				<Img
					src={repoImage()}
					width={100}
					height={100}
					stroke={'#555555'}
					lineWidth={8}
					strokeFirst={true}
					radius={10}
				/>
				<Layout ref={innerLayoutRef} direction={'column'}>
					<Txt
						fontFamily={'Roboto'}
						text={repoName()}
						fill={'#ffffff'}
						x={-520}
						y={-395}
						fontSize={50}
						fontWeight={600}
					/>
				</Layout>
			</Layout>
		</>,
	);

	// Resize the rectangle to reveal the scene
	yield* rectRef().width(0, videoLength);

	// Make rectangle transparent and cover the scene again
	rectRef().fill('#00000000');
	rectRef().width(view.width() * 2);

	// Cover the scene while the Layout block
	// is centered
	yield* all(
		rectRef().fill('#000000', 2),
		outerLayoutRef().x(0, 2),
		outerLayoutRef().y(-50, 2),
		outerLayoutRef().offset([0, 0], 2),
	);

	// Add text with the total number of stars
	const starTextRef = createRef<Txt>();
	innerLayoutRef().add(
		<Txt
			fontFamily={'Roboto'}
			ref={starTextRef}
			text={`${totalValues} stars`}
			fill={'#000000'}
			x={0}
			y={0}
			fontSize={40}
			fontWeight={500}
			marginBottom={-45}
		/>,
	);
	yield* all(starTextRef().fill('#ffffff', 2), starTextRef().margin(0, 2));

	// Wait for 2 seconds
	yield* waitFor(2);
});
```

### 5. Revideo Project (revideo/project.ts)

```typescript
import {makeProject} from '@revideo/core';

import './global.css';

import example from './scene';

export default makeProject({
	scenes: [example],
});
```

### 6. Stream Parser Utility (utils/parse.ts)

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

### 7. Layout Component (app/layout.tsx)

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

### 8. Global CSS Files

#### app/globals.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### revideo/global.css
```css
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');
```

## Key Concepts and Patterns

### 1. SaaS Architecture Pattern

#### Client-Server Separation
- **Frontend**: Next.js app with React components
- **Backend**: Server actions for GitHub API calls
- **Render Service**: Separate Revideo render server (port 4000)

#### API Architecture
```
User → Next.js Frontend → Server Actions → GitHub API
                      ↓
                 Render API → Revideo Server → Video File
```

### 2. GitHub API Integration

#### Rate Limiting Handling
```typescript
if (response.status === 403) {
  return {stargazerTimes: [], status: 'rate-limit'};
}
```
- Detects rate limit (403 status)
- Prompts user for personal access token
- Stores token client-side only

#### Parallel Data Fetching
```typescript
const limit = pLimit(10);  // Max 10 concurrent requests

for (let page = 1; page <= totalPages; page++) {
  stargazerPromises.push(limit(() => getStargazerTimesForPage(repo, page, headers)));
}
```
- Fetches multiple pages concurrently
- Uses p-limit for concurrency control
- Prevents API rate limit violations

#### Data Processing
1. Fetches star timestamps from GitHub
2. Converts to milliseconds since epoch
3. Normalizes to relative time from first star
4. Passes to visualization

### 3. Revideo Integration

#### Player Component
```typescript
<Player
  project={project}
  controls={true}
  variables={{
    data: stargazerTimes,
    repoName: repoName,
    repoImage: repoImage,
  }}
/>
```
- Embeds Revideo player in React
- Live preview with controls
- Dynamic variable passing

#### Variable System
```typescript
const repoName = useScene().variables.get('repoName', exampleRepoName);
const repoImage = useScene().variables.get('repoImage', exampleRepoImage);
const data = useScene().variables.get('data', exampleData);
```
- Scene receives dynamic data
- Falls back to example data
- Enables customization without code changes

### 4. Video Rendering Pipeline

#### Streaming Progress
```typescript
const downloadUrl = await parseStream(res.body!.getReader(), (p) => setProgress(p));
```
- Server-sent events for progress updates
- Real-time progress bar
- Download link on completion

#### Event Stream Format
```
event: progress
data: {"progress": 0.5}

event: completed
data: {"downloadLink": "http://..."}
```

### 5. Visualization Techniques

#### Graph Calculation
```typescript
const linePoints = data().map((ms, i) => {
  const x = (ms / max) * view.width();
  const xShifted = x - view.width() / 2;

  const y = ((-i / totalValues) * view.height()) / 2;
  const yShifted = y + view.height() / 4;

  return new Vector2(xShifted, yShifted);
});
```
- Maps time to X-axis (0 to max time)
- Maps star count to Y-axis (inverted)
- Creates growth curve visualization

#### Gradient Fill
```typescript
const gradient = new Gradient({
  type: 'linear',
  from: [0, 0],
  to: [0, view.height()],
  stops: [
    {offset: 0, color: '#000000'},
    {offset: 1, color: 'green'},
  ],
});
```
- Creates visual depth with gradient
- Black to green transition
- Applied to area under curve

### 6. Animation Sequence

#### Phase 1: Graph Reveal (0-5 seconds)
```typescript
yield* rectRef().width(0, videoLength);
```
- Black rectangle slides to reveal graph
- Duration matches videoLength (5 seconds)

#### Phase 2: Reposition (5-7 seconds)
```typescript
yield* all(
  rectRef().fill('#000000', 2),
  outerLayoutRef().x(0, 2),
  outerLayoutRef().y(-50, 2),
  outerLayoutRef().offset([0, 0], 2),
);
```
- Fades to black
- Centers repository info
- Simultaneous animations

#### Phase 3: Star Count (7-9 seconds)
```typescript
yield* all(
  starTextRef().fill('#ffffff', 2),
  starTextRef().margin(0, 2)
);
```
- Reveals total star count
- Text color transition
- Margin animation

#### Phase 4: Hold (9-11 seconds)
```typescript
yield* waitFor(2);
```
- Displays final result
- Allows viewer to read

## UI Components and Styling

### 1. Button Component
- Loading state with spinner
- Disabled during operations
- Tailwind styling with hover effects

### 2. Progress Bar
- Visual feedback during rendering
- Percentage display
- Smooth width transitions

### 3. Input Fields
- Repository name input
- GitHub token input (when needed)
- Placeholder guidance

### 4. Error Handling
- Rate limit notifications
- Error messages
- User-friendly alerts

### 5. Responsive Layout
- Max width container (7xl)
- Flexbox layouts
- Gap utilities for spacing

## Technical Implementation Details

### 1. Server Actions
- Next.js 14 server actions
- Direct database/API access
- Type-safe with TypeScript

### 2. Streaming Response
- ReadableStream API
- Progressive updates
- Binary data handling

### 3. CORS and Security
- API routes proxy external requests
- Token stored client-side only
- Direct GitHub API access from server

### 4. Performance Optimizations
- Concurrent API requests (10 max)
- Efficient data processing
- Stream parsing for large responses

### 5. Error Recovery
- Graceful degradation
- Fallback to example data
- Clear error messaging

## Deployment Considerations

### 1. Environment Setup
- Next.js app on standard port (3000)
- Revideo server on port 4000
- Both services required for full functionality

### 2. Production Configuration
- Update RENDER_URL for production
- Configure CORS headers
- Set up proper authentication

### 3. Scaling Considerations
- Render server can be horizontally scaled
- Consider queue system for large renders
- Cache rendered videos

## Use Cases

### 1. GitHub Analytics
- Repository growth visualization
- Milestone celebrations
- Project comparisons

### 2. SaaS Applications
- Custom video generation
- Marketing content creation
- Data visualization services

### 3. Developer Tools
- Project documentation
- Release announcements
- Community engagement

### 4. Educational Content
- Programming tutorials
- Open source contributions
- Growth metrics teaching

## Customization Options

### 1. Visual Styling
- Colors and gradients
- Fonts and sizes
- Animation timings

### 2. Data Sources
- Different GitHub metrics
- Other API integrations
- Custom data formats

### 3. Output Formats
- Video resolution
- Frame rate
- Compression settings

### 4. UI Customization
- Tailwind configuration
- Component styling
- Layout modifications

## Summary

The SaaS template demonstrates a complete full-stack implementation combining:

1. **Modern Stack**: Next.js 14 with App Router, React 18, TypeScript 5
2. **Real API Integration**: GitHub API with rate limiting and pagination
3. **Video Generation**: Revideo for programmatic video creation
4. **User Experience**: Live preview, progress tracking, download management
5. **Production Ready**: Error handling, loading states, responsive design
6. **Scalable Architecture**: Separated concerns, microservice pattern

This template serves as an excellent foundation for building SaaS applications that generate custom videos based on user data, showcasing the power of combining modern web technologies with Revideo's animation capabilities. The implementation provides a complete workflow from data fetching through visualization to final video delivery, making it suitable for various commercial and educational applications.