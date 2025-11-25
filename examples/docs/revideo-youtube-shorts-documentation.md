# YouTube Shorts Example - Comprehensive Documentation

## Overview

This example demonstrates how to create YouTube Shorts-style videos using Revideo, with synchronized captions, background images, and audio narration. The project includes optional AI-powered content generation capabilities using OpenAI, Deepgram, and ElevenLabs APIs.

## File Structure

```
youtube-shorts/
├── package.json           # Project dependencies and scripts
├── package-lock.json      # Lock file for dependencies
├── tsconfig.json         # TypeScript configuration
├── README.md            # Basic project documentation
└── src/
    ├── project.tsx       # Main Revideo project and scene definition
    ├── metadata.json     # Pre-generated asset data (audio URL, images, word timestamps)
    ├── utils.ts         # Utility functions for AI-powered content generation
    ├── get-assets.ts    # Script to generate assets using AI services
    ├── render.ts        # Script to render the final video
    └── global.css       # Font imports from Google Fonts
```

## Main Functionality and Flow

The application creates YouTube Shorts-style videos with:

1. **Synchronized captions/subtitles** that highlight words as they're spoken
2. **Background images** that cycle through the video duration
3. **Audio narration** with background music
4. **AI-powered content generation** capabilities (optional)

## Complete Source Code

### src/project.tsx

```tsx
import {Audio, Img, makeScene2D, Txt, Rect, Layout} from '@revideo/2d';
import {all, createRef, waitFor, useScene, Reference, createSignal, makeProject} from '@revideo/core';
import metadata from './metadata.json';
import './global.css';

interface Word {
  punctuated_word: string;
  start: number;
  end: number;
}

interface captionSettings {
  fontSize: number;
  textColor: string;
  fontWeight: number;
  fontFamily: string;
  numSimultaneousWords: number;
  stream: boolean;
  textAlign: "center" | "left";
  textBoxWidthInPercent: number;
  borderColor?: string;
  borderWidth?: number;
  currentWordColor?: string;
  currentWordBackgroundColor?: string;
  shadowColor?: string;
  shadowBlur?: number;
  fadeInAnimation?: boolean;
}

const textSettings: captionSettings = {
  fontSize: 80,
  numSimultaneousWords: 4, // how many words are shown at most simultaneously
  textColor: "white",
  fontWeight: 800,
  fontFamily: "Mulish",
  stream: false, // if true, words appear one by one
  textAlign: "center",
  textBoxWidthInPercent: 70,
  fadeInAnimation: true,
  currentWordColor: "cyan",
  currentWordBackgroundColor: "red", // adds a colored box to the word currently spoken
  shadowColor: "black",
  shadowBlur: 30
}

/**
 * The Revideo scene
 */
const scene = makeScene2D('scene', function* (view) {
  const images = useScene().variables.get('images', [])();
  const audioUrl = useScene().variables.get('audioUrl', 'none')();
  const words = useScene().variables.get('words', [])();

  const duration = words[words.length-1].end + 0.5;

  const imageContainer = createRef<Layout>();
  const textContainer = createRef<Layout>();

  yield view.add(
    <>
      <Layout
        size={"100%"}
        ref={imageContainer}
      />
      <Layout
        size={"100%"}
        ref={textContainer}
      />
      <Audio
        src={audioUrl}
        play={true}
      />
      <Audio
        src={"https://revideo-example-assets.s3.amazonaws.com/chill-beat-2.mp3"}
        play={true}
        volume={0.1}
      />
    </>
  );

  yield* all(
    displayImages(imageContainer, images, duration),
    displayWords(
      textContainer,
      words,
      textSettings
    )
  )
});

function* displayImages(container: Reference<Layout>, images: string[], totalDuration: number){
  for(const img of images){
    const ref = createRef<Img>();
    container().add(<Img
      src={img}
      size={["100%", "100%"]}
      ref={ref}
      zIndex={0}
    />
    )
    yield* waitFor(totalDuration/images.length);
  }
}

function* displayWords(container: Reference<Layout>, words: Word[], settings: captionSettings){
  let waitBefore = words[0].start;

  for (let i = 0; i < words.length; i += settings.numSimultaneousWords) {
    const currentBatch = words.slice(i, i + settings.numSimultaneousWords);
    const nextClipStart =
      i < words.length - 1 ? words[i + settings.numSimultaneousWords]?.start || null : null;
    const isLastClip = i + settings.numSimultaneousWords >= words.length;
    const waitAfter = isLastClip ? 1 : 0;
    const textRef = createRef<Txt>();
    yield* waitFor(waitBefore);

    if(settings.stream){
      let nextWordStart = 0;
      yield container().add(<Txt width={`${settings.textBoxWidthInPercent}%`} textWrap={true} zIndex={2} textAlign={settings.textAlign} ref={textRef}/>);

      for(let j = 0; j < currentBatch.length; j++){
        const word = currentBatch[j];
        yield* waitFor(nextWordStart);
        const optionalSpace = j === currentBatch.length-1? "" : " ";
        const backgroundRef = createRef<Rect>();
        const wordRef = createRef<Txt>();
        const opacitySignal = createSignal(settings.fadeInAnimation ? 0.5 : 1);
        textRef().add(
          <Txt
            fontSize={settings.fontSize}
            fontWeight={settings.fontWeight}
            fontFamily={settings.fontFamily}
            textWrap={true}
            textAlign={settings.textAlign}
            fill={settings.currentWordColor}
            ref={wordRef}
            lineWidth={settings.borderWidth}
            shadowBlur={settings.shadowBlur}
            shadowColor={settings.shadowColor}
            zIndex={2}
            stroke={settings.borderColor}
            opacity={opacitySignal}
          >
            {word.punctuated_word}
          </Txt>
        );
        textRef().add(<Txt fontSize={settings.fontSize}>{optionalSpace}</Txt>);
        container().add(<Rect fill={settings.currentWordBackgroundColor} zIndex={1} size={wordRef().size} position={wordRef().position} radius={10} padding={10} ref={backgroundRef} />);
        yield* all(waitFor(word.end-word.start), opacitySignal(1, Math.min((word.end-word.start)*0.5, 0.1)));
        wordRef().fill(settings.textColor);
        backgroundRef().remove();
        nextWordStart = currentBatch[j+1]?.start - word.end || 0;
      }
      textRef().remove();

    } else {
      yield container().add(<Txt width={`${settings.textBoxWidthInPercent}%`} textAlign={settings.textAlign} ref={textRef} textWrap={true} zIndex={2}/>);

      const wordRefs = [];
      const opacitySignal = createSignal(settings.fadeInAnimation ? 0.5 : 1);
      for(let j = 0; j < currentBatch.length; j++){
        const word = currentBatch[j];
        const optionalSpace = j === currentBatch.length-1? "" : " ";
        const wordRef = createRef<Txt>();
        textRef().add(
          <Txt
            fontSize={settings.fontSize}
            fontWeight={settings.fontWeight}
            ref={wordRef}
            fontFamily={settings.fontFamily}
            textWrap={true}
            textAlign={settings.textAlign}
            fill={settings.textColor}
            zIndex={2}
            stroke={settings.borderColor}
            lineWidth={settings.borderWidth}
            shadowBlur={settings.shadowBlur}
            shadowColor={settings.shadowColor}
            opacity={opacitySignal}
          >
            {word.punctuated_word}
          </Txt>
        );
        textRef().add(<Txt fontSize={settings.fontSize}>{optionalSpace}</Txt>);

        // we have to yield once to await the first word being aligned correctly
        if(j===0 && i === 0){
          yield;
        }
        wordRefs.push(wordRef);
      }

      yield* all(
        opacitySignal(1, Math.min(0.1, (currentBatch[0].end-currentBatch[0].start)*0.5)),
        highlightCurrentWord(container, currentBatch, wordRefs, settings.currentWordColor, settings.currentWordBackgroundColor),
        waitFor(currentBatch[currentBatch.length-1].end - currentBatch[0].start + waitAfter),
      );
      textRef().remove();
    }
    waitBefore = nextClipStart !== null ? nextClipStart - currentBatch[currentBatch.length-1].end : 0;
  }
}

function* highlightCurrentWord(container: Reference<Layout>, currentBatch: Word[], wordRefs: Reference<Txt>[], wordColor: string, backgroundColor: string){
  let nextWordStart = 0;

  for(let i = 0; i < currentBatch.length; i++){
    yield* waitFor(nextWordStart);
    const word = currentBatch[i];
    const originalColor = wordRefs[i]().fill();
    nextWordStart = currentBatch[i+1]?.start - word.end || 0;
    wordRefs[i]().text(wordRefs[i]().text());
    wordRefs[i]().fill(wordColor);

    const backgroundRef = createRef<Rect>();
    if(backgroundColor){
      container().add(<Rect fill={backgroundColor} zIndex={1} size={wordRefs[i]().size} position={wordRefs[i]().position} radius={10} padding={10} ref={backgroundRef} />);
    }

    yield* waitFor(word.end-word.start);
    wordRefs[i]().text(wordRefs[i]().text());
    wordRefs[i]().fill(originalColor);

    if(backgroundColor){
      backgroundRef().remove();
    }
  }
}

/**
 * The final revideo project
 */
export default makeProject({
  scenes: [scene],
  variables: metadata,
  settings: {
    shared: {
      size: {x: 1920, y: 1080},
    },
  },
});
```

### src/utils.ts

```typescript
import OpenAI from 'openai/index.mjs';
import axios from "axios";
import * as fs from "fs";
import { createClient } from "@deepgram/sdk";

const deepgram = createClient(process.env["DEEPGRAM_API_KEY"] || "");
const openai = new OpenAI({
	apiKey: process.env['OPENAI_API_KEY'],
  });

export async function getWordTimestamps(audioFilePath: string){
    const {result} = await deepgram.listen.prerecorded.transcribeFile(fs.readFileSync(audioFilePath), {
		model: "nova-2",
		smart_format: true,
	});

    if (result) {
        return result.results.channels[0].alternatives[0].words;
    } else {
		throw Error("transcription result is null");
    }

}

export async function generateAudio(text: string, voiceName: string, savePath: string) {
	const data = {
		model_id: "eleven_multilingual_v2",
		text: text,
	};

	const voiceId = await getVoiceByName(voiceName);

	const response = await axios.post(`https://api.elevenlabs.io/v1/text-to-speech/${voiceId}`, data, {
		headers: {
			"Content-Type": "application/json",
			"xi-api-key": process.env.ELEVEN_API_KEY || "",
		},
		responseType: "arraybuffer",
	});

	fs.writeFileSync(savePath, response.data);
}

async function getVoiceByName(name: string) {
	const response = await fetch("https://api.elevenlabs.io/v1/voices", {
		method: "GET",
		headers: {
			"xi-api-key": process.env.ELEVEN_API_KEY || "",
		},
	});

	if (!response.ok) {
		throw new Error(`HTTP error! status: ${response.status}`);
	}

	const data: any = await response.json();
	const voice = data.voices.find((voice: {name: string; voice_id: string}) => voice.name === name);
	return voice ? voice.voice_id : null;
}

export async function getVideoScript(videoTopic: string) {
  const prompt = `Create a script for a youtube short. The script should be around 60 to 80 words long and be an interesting text about the provided topic, and it should start with a catchy headline, something like "Did you know that?" or "This will blow your mind". Remember that this is for a voiceover that should be read, so things like hashtags should not be included. Now write the script for the following topic: "${videoTopic}". Now return the script and nothing else, also no meta-information - ONLY THE VOICEOVER.`;

  const chatCompletion = await openai.chat.completions.create({
    messages: [{ role: 'user', content: prompt }],
    model: 'gpt-4-turbo-preview',
  });

  const result = chatCompletion.choices[0].message.content;

  if (result) {
    return result;
  } else {
    throw Error("returned text is null");
  }

}

export async function getImagePromptFromScript(script: string) {
  const prompt = `My goal is to create a Youtube Short based on the following script. To create a background image for the video, I am using a text-to-video AI model. Please write a short (not longer than a single sentence), suitable prompt for such a model based on this script: ${script}.\n\nNow return the prompt and nothing else.`;

  const chatCompletion = await openai.chat.completions.create({
    messages: [{ role: 'user', content: prompt }],
    model: 'gpt-4-turbo-preview',
    temperature: 1.0 // high temperature for "creativeness"
  });

  const result = chatCompletion.choices[0].message.content;

  if (result) {
    return result;
  } else {
    throw Error("returned text is null");
  }

}

export async function dalleGenerate(prompt: string, savePath: string) {
	const response = await openai.images.generate({
		model: "dall-e-3",
		prompt: prompt,
		size: "1024x1792",
		quality: "standard",
		n: 1,
	});

	if (!response.data || !response.data[0]) {
		throw new Error("No image generated");
	}

	const url = response.data[0].url;
	const responseImage = await axios.get(url || "", {
		responseType: "arraybuffer",
	});
	const buffer = Buffer.from(responseImage.data, "binary");

	try {
		await fs.promises.writeFile(savePath, buffer);
	  } catch (error) {
		console.error("Error saving the file:", error);
		throw error; // Rethrow the error so it can be handled by the caller
	  }
	}
```

### src/get-assets.ts

```typescript
require('dotenv').config();

import { getVideoScript, generateAudio, getWordTimestamps, dalleGenerate, getImagePromptFromScript } from './utils';
import { v4 as uuidv4 } from 'uuid';
import * as fs from 'fs';

async function createAssets(topic: string, voiceName: string){
    const jobId = uuidv4();

    console.log("Generating assets...")
    const script = await getVideoScript(topic);
    console.log("script", script);

    await generateAudio(script, voiceName, `./public/${jobId}-audio.wav`);
    const words = await getWordTimestamps(`./public/${jobId}-audio.wav`);

    console.log("Generating images...")
    const imagePromises = Array.from({ length: 5 }).map(async (_, index) => {
        const imagePrompt = await getImagePromptFromScript(script);
        await dalleGenerate(imagePrompt, `./public/${jobId}-image-${index}.png`);
        return `/${jobId}-image-${index}.png`;
    });

    const imageFileNames = await Promise.all(imagePromises);
    const metadata = {
      audioUrl: `${jobId}-audio.wav`,
      images: imageFileNames,
      words: words
    };

    await fs.promises.writeFile(`./public/${jobId}-metadata.json`, JSON.stringify(metadata, null, 2));
}

createAssets("The moon landing", "Sarah")
```

### src/render.ts

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

### src/global.css

```css
@import url('https://fonts.googleapis.com/css2?family=Rubik:ital,wght@0,300..900;1,300..900&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Mulish:ital,wght@0,200..1000;1,200..1000&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Luckiest+Guy&family=Mulish:ital,wght@0,200..1000;1,200..1000&display=swap');
```

### src/metadata.json

```json
{
    "audioUrl": "https://revideo-example-assets.s3.amazonaws.com/12c4cb20-36b8-4ccf-a1bd-7a68b6b47173/audio.wav",
    "images": [
      "https://revideo-example-assets.s3.amazonaws.com/12c4cb20-36b8-4ccf-a1bd-7a68b6b47173/image-0.png",
      "https://revideo-example-assets.s3.amazonaws.com/12c4cb20-36b8-4ccf-a1bd-7a68b6b47173/image-1.png",
      "https://revideo-example-assets.s3.amazonaws.com/12c4cb20-36b8-4ccf-a1bd-7a68b6b47173/image-2.png",
      "https://revideo-example-assets.s3.amazonaws.com/12c4cb20-36b8-4ccf-a1bd-7a68b6b47173/image-3.png",
      "https://revideo-example-assets.s3.amazonaws.com/12c4cb20-36b8-4ccf-a1bd-7a68b6b47173/image-4.png"
    ],
    "words": [
      {
        "word": "ready",
        "start": 0.08,
        "end": 0.39999998,
        "confidence": 0.9988054,
        "punctuated_word": "Ready"
      },
      {
        "word": "to",
        "start": 0.39999998,
        "end": 0.48,
        "confidence": 0.99993455,
        "punctuated_word": "to"
      },
      {
        "word": "have",
        "start": 0.48,
        "end": 0.71999997,
        "confidence": 0.99994206,
        "punctuated_word": "have"
      },
      {
        "word": "your",
        "start": 0.71999997,
        "end": 0.88,
        "confidence": 0.9997631,
        "punctuated_word": "your"
      },
      {
        "word": "mind",
        "start": 0.88,
        "end": 1.28,
        "confidence": 0.99993217,
        "punctuated_word": "mind"
      },
      {
        "word": "blown",
        "start": 1.28,
        "end": 1.68,
        "confidence": 0.99995804,
        "punctuated_word": "blown"
      },
      {
        "word": "by",
        "start": 1.68,
        "end": 1.8399999,
        "confidence": 0.9996729,
        "punctuated_word": "by"
      },
      {
        "word": "the",
        "start": 1.8399999,
        "end": 2,
        "confidence": 0.99946827,
        "punctuated_word": "the"
      },
      {
        "word": "world's",
        "start": 2,
        "end": 2.48,
        "confidence": 0.9884629,
        "punctuated_word": "world's"
      },
      {
        "word": "largest",
        "start": 2.48,
        "end": 2.98,
        "confidence": 0.9995808,
        "punctuated_word": "largest"
      },
      {
        "word": "volksfest",
        "start": 3.04,
        "end": 3.54,
        "confidence": 0.94936365,
        "punctuated_word": "Volksfest?"
      },
      {
        "word": "welcome",
        "start": 4.16,
        "end": 4.56,
        "confidence": 0.999281,
        "punctuated_word": "Welcome"
      },
      {
        "word": "to",
        "start": 4.56,
        "end": 4.64,
        "confidence": 0.99980026,
        "punctuated_word": "to"
      },
      {
        "word": "the",
        "start": 4.64,
        "end": 4.7999997,
        "confidence": 0.99983585,
        "punctuated_word": "the"
      },
      {
        "word": "heart",
        "start": 4.7999997,
        "end": 5.04,
        "confidence": 0.9977786,
        "punctuated_word": "heart"
      },
      {
        "word": "of",
        "start": 5.04,
        "end": 5.2,
        "confidence": 0.9982095,
        "punctuated_word": "of"
      },
      {
        "word": "munich",
        "start": 5.2,
        "end": 5.68,
        "confidence": 0.99278367,
        "punctuated_word": "Munich,"
      },
      {
        "word": "germany",
        "start": 5.68,
        "end": 6.16,
        "confidence": 0.999559,
        "punctuated_word": "Germany"
      },
      {
        "word": "for",
        "start": 6.16,
        "end": 6.3999996,
        "confidence": 0.8465627,
        "punctuated_word": "for"
      },
      {
        "word": "the",
        "start": 6.3999996,
        "end": 6.48,
        "confidence": 0.99971575,
        "punctuated_word": "the"
      },
      {
        "word": "legendary",
        "start": 6.48,
        "end": 6.98,
        "confidence": 0.99840206,
        "punctuated_word": "legendary"
      },
      {
        "word": "oktoberfest",
        "start": 7.3599997,
        "end": 7.8599997,
        "confidence": 0.9987529,
        "punctuated_word": "Oktoberfest."
      },
      {
        "word": "imagine",
        "start": 8.724999,
        "end": 9.045,
        "confidence": 0.9981363,
        "punctuated_word": "Imagine"
      },
      {
        "word": "a",
        "start": 9.045,
        "end": 9.285,
        "confidence": 0.99952865,
        "punctuated_word": "a"
      },
      {
        "word": "festival",
        "start": 9.285,
        "end": 9.764999,
        "confidence": 0.99793196,
        "punctuated_word": "festival"
      },
      {
        "word": "stretching",
        "start": 9.764999,
        "end": 10.245,
        "confidence": 0.9995197,
        "punctuated_word": "stretching"
      },
      {
        "word": "over",
        "start": 10.245,
        "end": 10.485,
        "confidence": 0.9997055,
        "punctuated_word": "over"
      },
      {
        "word": "18",
        "start": 10.485,
        "end": 10.884999,
        "confidence": 0.9971296,
        "punctuated_word": "18"
      },
      {
        "word": "days",
        "start": 10.884999,
        "end": 11.384999,
        "confidence": 0.901999,
        "punctuated_word": "days,"
      },
      {
        "word": "featuring",
        "start": 11.684999,
        "end": 12.084999,
        "confidence": 0.9996741,
        "punctuated_word": "featuring"
      },
      {
        "word": "the",
        "start": 12.084999,
        "end": 12.245,
        "confidence": 0.99989414,
        "punctuated_word": "the"
      },
      {
        "word": "world's",
        "start": 12.245,
        "end": 12.6449995,
        "confidence": 0.99881995,
        "punctuated_word": "world's"
      },
      {
        "word": "most",
        "start": 12.6449995,
        "end": 13.125,
        "confidence": 0.99981004,
        "punctuated_word": "most"
      },
      {
        "word": "extravagant",
        "start": 13.125,
        "end": 13.625,
        "confidence": 0.9998173,
        "punctuated_word": "extravagant"
      },
      {
        "word": "beer",
        "start": 13.764999,
        "end": 14.084999,
        "confidence": 0.9997545,
        "punctuated_word": "beer"
      },
      {
        "word": "halls",
        "start": 14.084999,
        "end": 14.584999,
        "confidence": 0.99831164,
        "punctuated_word": "halls,"
      },
      {
        "word": "thrilling",
        "start": 14.804999,
        "end": 15.205,
        "confidence": 0.99957234,
        "punctuated_word": "thrilling"
      },
      {
        "word": "fun",
        "start": 15.205,
        "end": 15.525,
        "confidence": 0.9991985,
        "punctuated_word": "fun"
      },
      {
        "word": "rides",
        "start": 15.525,
        "end": 16.025,
        "confidence": 0.99723136,
        "punctuated_word": "rides,"
      },
      {
        "word": "traditional",
        "start": 16.28,
        "end": 16.78,
        "confidence": 0.99739826,
        "punctuated_word": "traditional"
      },
      {
        "word": "bavarian",
        "start": 16.84,
        "end": 17.320002,
        "confidence": 0.99255073,
        "punctuated_word": "Bavarian"
      },
      {
        "word": "music",
        "start": 17.320002,
        "end": 17.800001,
        "confidence": 0.97201467,
        "punctuated_word": "music,"
      },
      {
        "word": "and",
        "start": 17.800001,
        "end": 18.12,
        "confidence": 0.99851114,
        "punctuated_word": "and"
      },
      {
        "word": "lederhosen",
        "start": 18.12,
        "end": 18.62,
        "confidence": 0.849826,
        "punctuated_word": "Lederhosen"
      },
      {
        "word": "as",
        "start": 19.080002,
        "end": 19.240002,
        "confidence": 0.81338906,
        "punctuated_word": "as"
      },
      {
        "word": "far",
        "start": 19.240002,
        "end": 19.400002,
        "confidence": 0.9995309,
        "punctuated_word": "far"
      },
      {
        "word": "as",
        "start": 19.400002,
        "end": 19.560001,
        "confidence": 0.99795973,
        "punctuated_word": "as"
      },
      {
        "word": "the",
        "start": 19.560001,
        "end": 19.720001,
        "confidence": 0.99738353,
        "punctuated_word": "the"
      },
      {
        "word": "eye",
        "start": 19.720001,
        "end": 19.960001,
        "confidence": 0.9982356,
        "punctuated_word": "eye"
      },
      {
        "word": "can",
        "start": 19.960001,
        "end": 20.12,
        "confidence": 0.99665534,
        "punctuated_word": "can"
      },
      {
        "word": "see",
        "start": 20.12,
        "end": 20.62,
        "confidence": 0.99476826,
        "punctuated_word": "see."
      },
      {
        "word": "starting",
        "start": 20.920002,
        "end": 21.240002,
        "confidence": 0.998594,
        "punctuated_word": "Starting"
      },
      {
        "word": "in",
        "start": 21.240002,
        "end": 21.400002,
        "confidence": 0.9951212,
        "punctuated_word": "in"
      },
      {
        "word": "late",
        "start": 21.400002,
        "end": 21.640001,
        "confidence": 0.9984485,
        "punctuated_word": "late"
      },
      {
        "word": "september",
        "start": 21.640001,
        "end": 22.140001,
        "confidence": 0.99702036,
        "punctuated_word": "September,"
      },
      {
        "word": "over",
        "start": 22.52,
        "end": 22.84,
        "confidence": 0.9900007,
        "punctuated_word": "over"
      },
      {
        "word": "6",
        "start": 22.84,
        "end": 23.055,
        "confidence": 0.6389271,
        "punctuated_word": "6"
      },
      {
        "word": "million",
        "start": 23.295,
        "end": 23.535,
        "confidence": 0.54406315,
        "punctuated_word": "million"
      },
      {
        "word": "people",
        "start": 23.535,
        "end": 23.935,
        "confidence": 0.9996222,
        "punctuated_word": "people"
      },
      {
        "word": "from",
        "start": 23.935,
        "end": 24.175001,
        "confidence": 0.9993486,
        "punctuated_word": "from"
      },
      {
        "word": "around",
        "start": 24.175001,
        "end": 24.415,
        "confidence": 0.99978036,
        "punctuated_word": "around"
      },
      {
        "word": "the",
        "start": 24.415,
        "end": 24.575,
        "confidence": 0.99894863,
        "punctuated_word": "the"
      },
      {
        "word": "globe",
        "start": 24.575,
        "end": 25.055,
        "confidence": 0.9992418,
        "punctuated_word": "globe"
      },
      {
        "word": "gather",
        "start": 25.055,
        "end": 25.375,
        "confidence": 0.9074952,
        "punctuated_word": "gather"
      },
      {
        "word": "to",
        "start": 25.375,
        "end": 25.535,
        "confidence": 0.9997744,
        "punctuated_word": "to"
      },
      {
        "word": "celebrate",
        "start": 25.535,
        "end": 26.015,
        "confidence": 0.9998642,
        "punctuated_word": "celebrate"
      },
      {
        "word": "bavarian",
        "start": 26.015,
        "end": 26.515,
        "confidence": 0.9886686,
        "punctuated_word": "Bavarian"
      },
      {
        "word": "culture",
        "start": 26.575,
        "end": 27.075,
        "confidence": 0.9953433,
        "punctuated_word": "culture,"
      },
      {
        "word": "savor",
        "start": 27.375,
        "end": 27.855,
        "confidence": 0.98999786,
        "punctuated_word": "savor"
      },
      {
        "word": "mouthwatering",
        "start": 27.855,
        "end": 28.355,
        "confidence": 0.8768036,
        "punctuated_word": "mouthwatering"
      },
      {
        "word": "sausages",
        "start": 28.575,
        "end": 29.075,
        "confidence": 0.99394596,
        "punctuated_word": "sausages,"
      },
      {
        "word": "and",
        "start": 29.455,
        "end": 29.775,
        "confidence": 0.83777654,
        "punctuated_word": "and,"
      },
      {
        "word": "of",
        "start": 29.775,
        "end": 29.855,
        "confidence": 0.99985945,
        "punctuated_word": "of"
      },
      {
        "word": "course",
        "start": 29.855,
        "end": 30.340313,
        "confidence": 0.999547,
        "punctuated_word": "course,"
      },
      {
        "word": "indulge",
        "start": 30.580313,
        "end": 30.940311,
        "confidence": 0.99718326,
        "punctuated_word": "indulge"
      },
      {
        "word": "in",
        "start": 30.940311,
        "end": 31.300312,
        "confidence": 0.99884796,
        "punctuated_word": "in"
      },
      {
        "word": "over",
        "start": 31.300312,
        "end": 31.620314,
        "confidence": 0.9988391,
        "punctuated_word": "over"
      },
      {
        "word": "7,700,000",
        "start": 31.620314,
        "end": 32.120316,
        "confidence": 0.94937944,
        "punctuated_word": "7,700,000"
      },
      {
        "word": "liters",
        "start": 33.060314,
        "end": 33.46031,
        "confidence": 0.9917587,
        "punctuated_word": "liters"
      },
      {
        "word": "of",
        "start": 33.46031,
        "end": 33.62031,
        "confidence": 0.99943703,
        "punctuated_word": "of"
      },
      {
        "word": "beer",
        "start": 33.62031,
        "end": 34.12031,
        "confidence": 0.99747306,
        "punctuated_word": "beer."
      },
      {
        "word": "now",
        "start": 34.42031,
        "end": 34.660313,
        "confidence": 0.99840266,
        "punctuated_word": "Now"
      },
      {
        "word": "that's",
        "start": 34.660313,
        "end": 34.900314,
        "confidence": 0.9490575,
        "punctuated_word": "that's"
      },
      {
        "word": "a",
        "start": 34.900314,
        "end": 35.060314,
        "confidence": 0.99784327,
        "punctuated_word": "a"
      },
      {
        "word": "party",
        "start": 35.060314,
        "end": 35.380314,
        "confidence": 0.9999093,
        "punctuated_word": "party"
      },
      {
        "word": "not",
        "start": 35.380314,
        "end": 35.62031,
        "confidence": 0.9994562,
        "punctuated_word": "not"
      },
      {
        "word": "to",
        "start": 35.62031,
        "end": 35.700314,
        "confidence": 0.9987685,
        "punctuated_word": "to"
      },
      {
        "word": "be",
        "start": 35.700314,
        "end": 35.860313,
        "confidence": 0.99990654,
        "punctuated_word": "be"
      },
      {
        "word": "missed",
        "start": 35.860313,
        "end": 36.360313,
        "confidence": 0.9969932,
        "punctuated_word": "missed."
      }
    ]
  }
```

## Configuration Files

### package.json

```json
{
  "name": "revideo-app",
  "private": true,
  "version": "0.0.0",
  "scripts": {
    "start": "revideo editor --projectFile ./src/project.tsx",
    "render": "tsc && node dist/render.js"
  },
  "dependencies": {
    "@deepgram/sdk": "^3.3.1",
    "@revideo/2d": "0.10.3",
    "@revideo/core": "0.10.3",
    "@revideo/renderer": "0.10.3",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "openai": "^4.40.2",
    "uuid": "^9.0.1"
  },
  "devDependencies": {
    "@revideo/cli": "0.10.3",
    "@revideo/ui": "0.10.3",
    "@types/uuid": "^10.0.0",
    "typescript": "5.5.4",
    "vite": "^4.5"
  }
}
```

### tsconfig.json

```json
{
  "extends": "@revideo/2d/tsconfig.project.json",
  "include": ["src"],
  "compilerOptions": {
    "noEmit": false,
    "outDir": "dist",
    "module": "CommonJS",
    "skipLibCheck": true
  }
}
```

### README.md

```markdown
# Revideo Minimal Starter Template

This is a minimal Revideo starter project. It can be bootstrapped using
`npm init @revideo@latest` and selecting the minimal template.

It includes a simple scene with a background video, music and the Revideo logo.
You can get started by editing the `project.tsx` file in the `src` folder.

You can run the Revideo editor using `npm start` and render the final video
using `npm run render`.

If you want to see what's possible with Revideo, check out the other templates
in the [Revideo Examples repository](https://github.com/redotvideo/examples), or
the [Revideo documentation](https://docs.re.video).

If you want to include your Revideo project in a website, you can use the
Next.js template. It can be found in the Revideo Examples repository or
bootstrapped using `npm init @revideo@latest` and selecting the Next.js
template.
```

## Key Revideo Concepts Demonstrated

### Core Components from '@revideo/2d':
- **`Audio`**: Plays both narration (audioUrl) and background music
- **`Img`**: Displays background images with full viewport sizing
- **`Txt`**: Renders text captions with styling and animations
- **`Rect`**: Creates colored backgrounds for highlighted words
- **`Layout`**: Container components for organizing images and text

### Core Functions from '@revideo/core':
- **`makeScene2D`**: Creates the main 2D scene
- **`makeProject`**: Defines the project configuration
- **`createRef`**: Creates references to components for manipulation
- **`createSignal`**: Creates reactive signals for animations (opacity changes)
- **`useScene`**: Accesses scene variables (images, audioUrl, words)
- **`waitFor`**: Controls timing and synchronization
- **`all`**: Runs multiple animations/tasks in parallel

## Important Code Patterns and Techniques

### 1. Caption Display System

The caption system supports two display modes:

**Streaming Mode**: Words appear one by one as they're spoken
- Each word fades in individually
- Previous words remain visible
- Current word is highlighted with a different color and background

**Batch Mode**: Multiple words appear together
- Configured via `numSimultaneousWords` setting
- All words in batch fade in together
- Current word being spoken is highlighted

### 2. Word Synchronization

The `highlightCurrentWord` function:
- Uses word timing data from speech transcription
- Highlights current word with color change
- Adds temporary background rectangle during word duration
- Calculates precise wait times between words
- Smoothly transitions highlighting between words

### 3. Image Slideshow

The `displayImages` function:
- Distributes total duration evenly across all images
- Images displayed as full-screen backgrounds (zIndex: 0)
- Simple fade transitions between images

### 4. Scene Variables

Variables are passed through the `metadata` object and accessed via:
```typescript
const images = useScene().variables.get('images', [])();
const audioUrl = useScene().variables.get('audioUrl', 'none')();
const words = useScene().variables.get('words', [])();
```

## AI-Powered Asset Generation

### Workflow (get-assets.ts):

1. **Generate Script**: Creates video script from topic using GPT-4
2. **Generate Audio**: Converts script to speech using ElevenLabs
3. **Transcribe Audio**: Gets word-level timestamps using Deepgram
4. **Generate Images**: Creates 5 background images using DALL-E 3
5. **Save Metadata**: Stores all asset references and timing data

### Environment Variables Required:
- `OPENAI_API_KEY`: For GPT-4 and DALL-E access
- `DEEPGRAM_API_KEY`: For speech transcription
- `ELEVEN_API_KEY`: For text-to-speech

## Execution Scripts

### Development Mode
```bash
npm start
```
- Launches Revideo editor
- Hot reload enabled
- Interactive preview

### Production Rendering
```bash
npm run render
```
- Compiles TypeScript to JavaScript
- Renders final video file
- Outputs to default location

## Project Settings

### Video Configuration:
- **Resolution**: 1920x1080 (HD)
- **Format**: Designed for vertical video (YouTube Shorts)
- **Scene Management**: Single scene with parallel animations

### Rendering Process:
1. Scene initialization loads variables from metadata
2. Audio tracks begin playing immediately
3. Images cycle in background layer
4. Words display with synchronization to audio
5. All animations run via generator functions

## Usage Examples

### Basic Usage with Pre-generated Assets:
1. Ensure `metadata.json` contains valid asset URLs
2. Run `npm start` to preview in editor
3. Run `npm run render` to generate video

### Generating New Content:
1. Set environment variables:
   - `OPENAI_API_KEY`
   - `DEEPGRAM_API_KEY`
   - `ELEVEN_API_KEY`
2. Edit topic in `get-assets.ts`:
   ```typescript
   createAssets("Your Topic Here", "Voice Name")
   ```
3. Run `node dist/get-assets.js` after compilation
4. Update `metadata.json` with new asset URLs
5. Render video

## Performance Considerations

- Images are loaded asynchronously
- Word highlighting uses efficient ref-based updates
- Background rectangles are added/removed dynamically
- Opacity animations use signals for smooth transitions
- Parallel animations via `all()` for better performance

## Customization Options

### Visual Styling:
- Modify `captionSettings` in project.tsx
- Update fonts in global.css
- Adjust image display duration
- Change background music volume

### Content Generation:
- Modify prompts in utils.ts
- Adjust script length parameters
- Change image generation settings
- Select different AI models

## Troubleshooting

### Common Issues:
1. **Missing Assets**: Check metadata.json URLs are accessible
2. **Font Loading**: Ensure internet connection for Google Fonts
3. **API Errors**: Verify environment variables are set
4. **Rendering Issues**: Check TypeScript compilation succeeded

### Debug Tips:
- Use browser DevTools in editor mode
- Check console for error messages
- Verify asset URLs return valid content
- Test with simplified metadata first