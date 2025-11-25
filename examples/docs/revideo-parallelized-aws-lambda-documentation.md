# Revideo Parallelized AWS Lambda - Complete Implementation Guide

## Overview

This guide provides comprehensive documentation for implementing parallelized video rendering using Revideo on AWS Lambda. The example demonstrates how to split rendering tasks across multiple Lambda functions for faster processing, particularly useful for long videos or batch processing.

**Use Case:** Scalable, serverless video rendering that distributes workload across multiple Lambda instances for faster processing times.

## Project Structure

```
parallelized-aws-lambda/
├── .dockerignore             # Docker ignore file
├── Dockerfile                # Main Docker image for Lambda deployment
├── Dockerfile.base           # Base Docker image with Chromium/dependencies
├── README.md                 # Original setup instructions
└── revideo-project/
    ├── package.json          # Dependencies and build scripts
    ├── package-lock.json     # Locked dependencies
    ├── tsconfig.json         # TypeScript configuration
    └── src/
        ├── project.tsx       # Revideo scene definition
        ├── lambda.ts         # AWS Lambda handler implementation
        └── render.ts         # Local rendering script
```

## Complete Source Code

### src/project.tsx

```typescript
import {Audio, Txt, Video, makeScene2D} from '@revideo/2d';
import {createSignal, useScene, waitFor, makeProject} from '@revideo/core';

const scene = makeScene2D("scene", function* (view) {
  const fontSize = createSignal(50);

  yield view.add(
    <>
      <Video
        src={'https://revideo-example-assets.s3.amazonaws.com/beach-4.mp4'}
        size={['100%', '100%']}
        play={true}
        decoder={useScene().variables.get("decoder", "ffmpeg")} // ffmpeg is more reliable than webcodecs decoder
      />
      <Audio
        src={'https://revideo-example-assets.s3.amazonaws.com/chill-beat-2.mp3'}
        play={true}
      />
    </>,
  );

  yield* waitFor(1);

  yield view.add(<Txt fill={"white"} fontSize={fontSize}>{useScene().variables.get("message", "Hello World!")()}</Txt>);

  yield* fontSize(100, 4); // increase font size to 100 in 4 seconds

  yield* waitFor(25); // wait for 25 seconds more
});


export default makeProject({
  scenes: [scene],
});
```

### src/lambda.ts

```typescript
import { renderPartialVideo } from '@revideo/renderer';
import { mergeAudioWithVideo, concatenateMedia } from '@revideo/ffmpeg';
import chromium from '@sparticuz/chromium';
import * as AWS from "aws-sdk";
import * as fs from "fs";

chromium.setHeadlessMode = true;

const s3 = new AWS.S3();
const lambda = new AWS.Lambda({
  httpOptions: {
    timeout: 900000, // 15 min timeout
  },
});

export const handler = async (event: any, context: any) => {
  const { jobType, jobId } = event;

  if(jobType == "partialRender"){

    try {
      const { workerId, numWorkers, variables } = event;

      const { audioFile, videoFile } = await renderPartialVideo({
        projectFile: './src/project.tsx',
        workerId: workerId,
        numWorkers: numWorkers,
        variables: variables,
        settings: { logProgress: true, viteBasePort: 5000, outDir: "/tmp/output", viteConfig: { cacheDir: "/tmp/.vite"}, puppeteer: {
          headless: chromium.headless,
          executablePath: await chromium.executablePath(),
          args: chromium.args.filter(arg =>
            !arg.startsWith('--single-process') &&
            !arg.startsWith('--use-gl=angle') &&
            !arg.startsWith('--use-angle=swiftshader') &&
            !arg.startsWith('--disable-features=')
          ),
       }}
      });

      await Promise.all([uploadFileToBucket(audioFile, `${jobId}-audio-${workerId}.wav`), uploadFileToBucket(videoFile, `${jobId}-video-${workerId}.mp4`)]);

      return {
        statusCode: 200,
        body: JSON.stringify({
          audioUrl: `https://${process.env.REVIDEO_BUCKET_NAME}.s3.amazonaws.com/${jobId}-audio-${workerId}.wav`,
          videoUrl: `https://${process.env.REVIDEO_BUCKET_NAME}.s3.amazonaws.com/${jobId}-video-${workerId}.mp4`
        }),
      };
    } catch(err) {
      return {
        statusCode: 500,
        body: JSON.stringify({
          error: `Error during partial render: ${err}`
        })
      }
    }


  } else {
    const { numWorkers, variables, jobId } = event;

    try {

      const renderPromises = [];
      for (let i = 0; i < numWorkers; i++) {
        console.log("invoking worker", i);
        renderPromises.push(invokePartialRender(variables, i, numWorkers, jobId));
      }

      console.log("now waiting");

      const results = await Promise.all(renderPromises);
      console.log("obtained results", results);
      const audios = results.map(result => result.audioUrl);
      const videos = results.map(result => result.videoUrl);

      console.log("videos", videos);
      await Promise.all([
        concatenateMedia(audios, `/tmp/${jobId}-audio.wav`),
        concatenateMedia(videos, `/tmp/${jobId}-visuals.mp4`)
      ]);

      await mergeAudioWithVideo(`/tmp/${jobId}-audio.wav`, `/tmp/${jobId}-visuals.mp4`, `/tmp/${jobId}.mp4`);
      await uploadFileToBucket(`/tmp/${jobId}.mp4`, `${jobId}.mp4`);

      return {
        statusCode: 200,
        body: JSON.stringify({
          resultUrl: `https://${process.env.REVIDEO_BUCKET_NAME}.s3.amazonaws.com/${jobId}.mp4`,
        }),
      };
    } catch(err){
      return {
        statusCode: 500,
        body: JSON.stringify({
          error: `An error occured during rendering: ${err}`
        }),
      };
    }


  }

};

async function uploadFileToBucket(localPath: string, destinationPath: string) {
	const fileContent = await fs.promises.readFile(localPath);

	const params = {
		Bucket: process.env.REVIDEO_BUCKET_NAME,
		Key: destinationPath,
		Body: fileContent,
	};

	try {
		const data = await s3.upload(params).promise();
		console.log(`File uploaded successfully. ${data.Location}`);
	} catch (err) {
		console.error("Error uploading file: ", err);
	}
}

async function invokePartialRender(variables: any, workerId: number, numWorkers: number, jobId: string){
  const params = {
    FunctionName: process.env.AWS_LAMBDA_FUNCTION_NAME,
    InvocationType: 'RequestResponse',
    LogType: 'None',
    Payload: JSON.stringify({
      variables: variables,
      workerId: workerId,
      numWorkers: numWorkers,
      jobId: jobId,
      jobType: "partialRender"
    }),
  };

  try {
    const data = await lambda.invoke(params).promise();
    const payload = JSON.parse(data.Payload as string);
    console.log('Lambda invoke result:', payload);

    // Parse the body string to JSON to access the audioUrl and videoUrl
    const body = JSON.parse(payload.body);

    if(payload.statusCode !== 200){
      throw Error(body.message); // Use the parsed body for error messages
    }

    return { audioUrl: body.audioUrl, videoUrl: body.videoUrl };

  } catch (error) {
    console.error("Error during partial render:", error);
    throw error;
  }
}
```

### src/render.ts

```typescript
import {renderVideo} from '@revideo/renderer';

async function render() {
  console.log('Rendering video...');

  // This is the main function that renders the video
  console.time("render");
  const file = await renderVideo({
    projectFile: './src/project.tsx',
    settings: {logProgress: true},
  });
  console.timeEnd("render");

  console.log(`Rendered video to ${file}`);
}

render();
```

### .dockerignore

```
node_modules
dist
package-lock.json
```

### Dockerfile

```dockerfile
FROM docker.io/revideo/aws-lambda-base-image:latest

COPY ./revideo-project/ ./

RUN npm install

RUN node node_modules/puppeteer/install.mjs

RUN npx tsc && cp dist/lambda.js ./

ENV ROLLUP_CACHE=/tmp/rollup_cache

ENV FFMPEG_PATH=/var/task/node_modules/@ffmpeg-installer/linux-x64/ffmpeg

ENV HOME=/tmp

ENV DONT_WRITE_TO_META_FILES=true

CMD ["lambda.handler"]
```

### Dockerfile.base

```dockerfile
FROM public.ecr.aws/lambda/nodejs:18

RUN yum update -y && yum install -y \
 nscd \
 libnss3 \
 libdbus-1-3 \
 libatk1.0-0 \
 libatk-bridge2.0-0 \
 libcups2 \
 libxkbcommon0 \
 libxcomposite1 \
 libxdamage1 \
 libxrandr2 \
 libgbm1 \
 libpango-1.0-0 \
 libcairo2 \
 libasound2 \
 libatspi2.0-0 \
 libxfixes3

RUN yum update -y && yum install -y \
 alsa-lib \
 at-spi2-atk \
 atk \
 cups-libs \
 gtk3 \
 ipa-gothic-fonts \
 libXcomposite \
 libXcursor \
 libXdamage \
 libXext \
 libXi \
 libXrandr \
 libXScrnSaver \
 libXtst \
 pango \
 xorg-x11-fonts-100dpi \
 xorg-x11-fonts-75dpi \
 xorg-x11-fonts-cyrillic \
 xorg-x11-fonts-misc \
 xorg-x11-fonts-Type1 \
 xorg-x11-utils \
 nss \
 libX11 \
 GConf2 \
 dbus-glib \
 dbus-libs \
 libXrender \
 libxcb \
 libdrm \
 libxshmfence \
 mesa-libgbm \
 libatk \
 libxkbcommon

RUN curl -L https://github.com/eugeneware/ffmpeg-static/releases/download/b6.0/ffprobe-linux-x64 -o ffprobe-linux-x64

RUN chmod +x ffprobe-linux-x64

RUN mv ffprobe-linux-x64 ffprobe

RUN mv ffprobe /usr/local/bin/

ENV FFPROBE_PATH=ffprobe
```

## API Reference and Concepts

### Imports from @revideo/2d

#### `makeScene2D` Function
- **Purpose:** Creates a 2D scene for the animation
- **Usage in example:** Creates main scene with video and text overlays

#### `Audio` Component
- **Purpose:** Audio playback in the scene
- **Key properties:**
  - `src`: URL to audio file
  - `play`: Boolean to auto-start playback

#### `Video` Component
- **Purpose:** Video playback in the scene
- **Key properties:**
  - `src`: URL to video file
  - `size`: Dimensions specification
  - `play`: Auto-start playback
  - `decoder`: Decoder selection ("ffmpeg" vs "webcodecs")

#### `Txt` Component
- **Purpose:** Text rendering
- **Key properties:**
  - `fill`: Text color
  - `fontSize`: Signal for animatable font size

### Imports from @revideo/core

#### `createSignal<T>()` Function
- **Purpose:** Creates reactive signal for animations
- **Usage:** Font size animation from 50 to 100 pixels

#### `useScene()` Hook
- **Purpose:** Access scene variables
- **Methods used:**
  - `variables.get()`: Retrieves runtime variables (message, decoder)

#### `waitFor(duration)` Generator
- **Purpose:** Timing control in animation
- **Usage:** Waits 1 second before showing text, 25 seconds total duration

#### `makeProject()` Function
- **Purpose:** Creates final project configuration
- **Configuration:** Includes scene array

### Imports from @revideo/renderer

#### `renderPartialVideo()` Function
- **Purpose:** Renders a portion of video for parallelization
- **Key parameters:**
  - `projectFile`: Path to project file
  - `workerId`: Worker index (0 to numWorkers-1)
  - `numWorkers`: Total number of workers
  - `variables`: Runtime variables
  - `settings`: Rendering configuration including Puppeteer settings
- **Returns:** Object with audioFile and videoFile paths

#### `renderVideo()` Function
- **Purpose:** Standard full video rendering (for local testing)
- **Parameters:**
  - `projectFile`: Path to project file
  - `settings`: Rendering configuration

### Imports from @revideo/ffmpeg

#### `concatenateMedia()` Function
- **Purpose:** Concatenates multiple media files
- **Usage:** Combines partial audio/video files from workers

#### `mergeAudioWithVideo()` Function
- **Purpose:** Merges audio and video tracks
- **Usage:** Combines concatenated audio with concatenated video

### AWS SDK Imports

#### `AWS.S3`
- **Purpose:** Upload rendered videos to S3 bucket
- **Methods:**
  - `upload()`: Uploads file content to bucket

#### `AWS.Lambda`
- **Purpose:** Invoke Lambda functions for parallelization
- **Configuration:** 15-minute timeout for long renders
- **Methods:**
  - `invoke()`: Triggers Lambda function execution

### External Dependencies

#### `@sparticuz/chromium`
- **Purpose:** Headless Chrome for Lambda environment
- **Configuration:** Custom args filtering for Lambda compatibility

## Key Implementation Patterns

### 1. Parallelized Rendering Architecture
```typescript
// Main orchestrator divides work
for (let i = 0; i < numWorkers; i++) {
  renderPromises.push(invokePartialRender(variables, i, numWorkers, jobId));
}
const results = await Promise.all(renderPromises);
```

### 2. Lambda Handler Pattern
```typescript
export const handler = async (event: any, context: any) => {
  const { jobType, jobId } = event;

  if(jobType == "partialRender"){
    // Render partial video segment
  } else {
    // Orchestrate full render
  }
}
```

### 3. Puppeteer Configuration for Lambda
```typescript
puppeteer: {
  headless: chromium.headless,
  executablePath: await chromium.executablePath(),
  args: chromium.args.filter(arg =>
    !arg.startsWith('--single-process') &&
    !arg.startsWith('--use-gl=angle')
    // Additional filters...
  ),
}
```

### 4. Environment Variable Configuration
```typescript
decoder={useScene().variables.get("decoder", "ffmpeg")}
// Message customization
{useScene().variables.get("message", "Hello World!")()}
```

### 5. S3 Upload Pattern
```typescript
async function uploadFileToBucket(localPath: string, destinationPath: string) {
  const fileContent = await fs.promises.readFile(localPath);
  const params = {
    Bucket: process.env.REVIDEO_BUCKET_NAME,
    Key: destinationPath,
    Body: fileContent,
  };
  const data = await s3.upload(params).promise();
}
```

## Docker Configuration

### Base Image Setup
- Based on AWS Lambda Node.js 18 runtime
- Includes Chromium dependencies for headless browser
- Installs ffprobe for media processing
- Pre-built image available: `docker.io/revideo/aws-lambda-base-image:latest`

### Main Image Build
- Copies project files
- Installs npm dependencies
- Compiles TypeScript
- Sets environment variables:
  - `ROLLUP_CACHE`: Temp directory for build cache
  - `FFMPEG_PATH`: Path to ffmpeg binary
  - `HOME`: Temp directory for Chrome
  - `DONT_WRITE_TO_META_FILES`: Disable metadata writes

## Lambda Execution Flow

### Full Render (Orchestrator)
1. Receives request with `jobType: "fullRender"`
2. Invokes N worker lambdas with `jobType: "partialRender"`
3. Each worker renders 1/N of the video
4. Waits for all partial renders to complete
5. Downloads partial files from S3
6. Concatenates audio and video segments
7. Merges final audio with video
8. Uploads complete video to S3

### Partial Render (Worker)
1. Receives worker ID and total worker count
2. Renders specific frame range using `renderPartialVideo()`
3. Uploads audio and video segments to S3
4. Returns S3 URLs to orchestrator

## Environment Variables

- `REVIDEO_BUCKET_NAME`: S3 bucket for storing renders
- `AWS_LAMBDA_FUNCTION_NAME`: Lambda function name for self-invocation
- `AWS_ACCESS_KEY_ID`: AWS credentials
- `AWS_SECRET_ACCESS_KEY`: AWS credentials
- `AWS_DEFAULT_REGION`: AWS region

## Assets and Resources

### External Assets Used
1. **Background Video**
   - URL: `https://revideo-example-assets.s3.amazonaws.com/beach-4.mp4`
   - Type: MP4 video
   - Usage: Full-screen background

2. **Background Audio**
   - URL: `https://revideo-example-assets.s3.amazonaws.com/chill-beat-2.mp3`
   - Type: MP3 audio
   - Usage: Background music

### Output Specifications
- **Format:** MP4 with merged audio/video
- **Duration:** ~30 seconds (1s wait + 4s animation + 25s wait)
- **Text:** Customizable via `message` variable
- **Font animation:** 50px to 100px over 4 seconds

## Technical Notes

### Lambda Limitations
- Maximum execution time: 15 minutes
- Memory: 3000 MB recommended
- Temp storage: /tmp directory (512 MB)
- Requires custom Chromium build for Lambda

### Performance Optimization
- Parallel rendering reduces total time by factor of N workers
- Each worker processes independent frame ranges
- Audio rendered once, duplicated across workers
- Concatenation happens in parallel for audio/video

### Error Handling
- Partial render failures caught and reported
- Full render failure returns error status
- S3 upload errors logged but don't fail render

### Deployment Considerations
- Docker image must be built for Linux/AMD64
- ECR repository required for Lambda deployment
- S3 bucket must allow public read access
- Lambda needs invoke permissions for self-invocation

## Common Use Cases

This implementation pattern is suitable for:
- Long-form video generation (>5 minutes)
- Batch video processing at scale
- Template-based video generation with variables
- Automated video responses with custom messages
- Social media content generation pipelines

## Testing

### Local Testing
```bash
npm run render
```
Renders complete video locally without parallelization.

### Lambda Testing
```json
{
  "jobId": "12345",
  "numWorkers": 5,
  "jobType": "fullRender",
  "variables": {"message": "Hello from Revideo!"}
}
```
Invoke Lambda with this payload to test parallelized rendering.

## File Output

- **Partial renders:** `{jobId}-audio-{workerId}.wav`, `{jobId}-video-{workerId}.mp4`
- **Concatenated files:** `{jobId}-audio.wav`, `{jobId}-visuals.mp4`
- **Final output:** `{jobId}.mp4` in S3 bucket