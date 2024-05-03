[![NPM Version](https://img.shields.io/npm/v/@transcribe/transcriber?style=flat&color=green)](https://www.npmjs.com/package/@transcribe/transcriber)

# Transcribe.js

Transcribe speech to text in the browser. Based on a wasm build of [whisper.cpp](https://github.com/ggerganov/whisper.cpp).

- [Docs](https://transcribejs.dev)
- [Example File Transcriber](https://examples.transcribejs.dev/examples/index.html)
- [Example Stream Transcriber (experimental)](https://examples.transcribejs.dev/examples/stream.html)

## Packages

All packages are under [@transcribe](https://www.npmjs.com/search?q=%40transcribe) namespace.

| Package                     | Description                                                                                                                                                |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **@transcribe/shout**       | Wasm build based on [whisper.cpp](https://github.com/ggerganov/whisper.cpp). Contains Module file including the wasm binary and a separate webworker file. |
| **@transcribe/transcriber** | `FileTranscriber` and `StreamTranscriber` for transcribing media files or streams.                                                                         |

## Prerequisite

### Webserver

Your webserver must serve the files with cross origin headers.

`"Cross-Origin-Embedder-Policy": "require-corp"`  
`"Cross-Origin-Opener-Policy": "same-origin"`

### Model File

You need a ggml model file to run Transcribe.js. You can download them on hugging face https://huggingface.co/ggerganov/whisper.cpp/tree/main . You should start with the (quantized) tiny or base models. Larger models propably won't work but you can try it, though.

## Installation

### NPM

Install shout wasm and transcriber packages

```bash
npm install --save @transcribe/transcriber
```

copy the shout.wasm and webworker files to your project directory

```bash
# copy shout wasm
cp node_modules/@transcribe/shout/src/shout/shout.wasm.worker.mjs /your/project
cp node_modules/@transcribe/shout/src/shout/shout.wasm.js /your/project

# copy audio-worklets, only needed if you want to use StreamTranscriber
cp -r node_modules/@transcribe/transcriber/src/audio-worklets /your/project
```

### Manual Installation

You can use Transcribe.js without a bundler or package manager. Download the files from this repository, copy the `src/*` directories to your webserver and include the following into your HTML. Make sure to set the correct paths in the import map.

```html
<!-- set paths to js files -->
<script type="importmap">
  {
    "imports": {
      "@transcribe/shout": "/src/shout/shout.wasm.js",
      "@transcribe/transcriber": "/src/index.js"
    }
  }
</script>

<!-- use type="module" for es6 imports -->
<script type="module">
  import { FileTranscriber } from "@transcribe/transcriber";

  ...
</script>
```

## Usage

For full code example and advanced usage please check out https://www.transcribejs.dev or check out the [File Transcriber Example](https://examples.transcribejs.dev/examples/index.html).

```js
import { FileTranscriber } from "@transcribe/transcriber";

// create new instance
const transcriber = new FileTranscriber({
  model: "/your/project/ggml-tiny-q5_1.bin", // path to ggml model file
  workerPath: "/your/project", // directory of shout.wasm.worker.mjs copied before
});

// init wasm transcriber worker
await transcriber.init();

// transcribe audio/video file
const result = await transcriber.transcribe("/your/project/my.mp3");

console.log(result);
```

The `result` is an JSON object containg the text segements and timestamps.

```js
{
  "result": {
    "language": "en"
  },
  "transcription": [
    {
      "timestamps": {
        "from": "00:00:00,000",
        "to": "00:00:11,000"
      },
      "offsets": {
        "from": 0,
        "to": 11000
      },
      "text": " And so my fellow Americans ask not what your country can do for you, ask what you can do for your country.",
      "tokens": [
        {
          "text": " And",
          "timestamps": {
            "from": "00:00:00,320",
            "to": "00:00:00,350"
          },
          "offsets": {
            "from": 320,
            "to": 350
          },
          "id": 400,
          "p": 0.726615 // propability, aka. how likely the estimate is true, 0..1, 1 is best
        },
        // ... one token per word
      ]
    }
  ]
}
```

## Development

Clone the repository, install dependencies, start the dev server and open `http://localhost:9876/examples/index.html` in your browser.

```bash
git clone https://github/transcribejs/transcribe.js
cd transcribe
npm install
npm run dev
```

### Types

The library is not written in typescript. This way no extra build step is needed during development and in production.

To still get proper type support type definitions get generated from JSDoc comments.

```bash
npm run generate-types
```

### Wasm build

The `whisper.cpp` repository is a git submodule. To get the latest version of `whisper.cpp` go into the directory and pull the latest changes from github.

```bash
cd shout.wasm/whisper.cpp
git pull origin master
```

The wasm files are build from `shout.wasm/src/whisper.wasm.cpp`. If you want to add new functions from whisper.cpp to the wasm build this is the file to add them.

> I'm pretty sure that this will not compile on every machine/architecture, but I'm no expert in C++. If you know how to optimize the build process please let me know or create a pull request. Maybe this should be dockerized.?

```bash
# run cmake to build wasm
npm run wasm:build

# copy emscripten build files to project
npm run wasm:copy
```

### Tests

Unit/functional tests for the `Transcriber` functions.

```bash
npm run test:unit
```

E2E tests using Playwright. Firefox somehow needs waaaaaay longer during e2e test than in a the "real" browser.

```bash
npm run test:e2e
```

or use the Playwright UI for details

```bash
npm run test:e2e-ui
```

## Credits

Thank you to the creators and contributors of the following open source libraries that were used in this project:

### Libraries

- **whisper.cpp**: A C++ implementation of whisper. [GitHub Repository](https://github.com/ggerganov/whisper.cpp)
- **emscripten**: A toolchain for compiling C and C++ code to WebAssembly. [Official Site](https://emscripten.org/)
- **water.css**: A minimal CSS framework for styling HTML. [Official Site](https://watercss.kognise.dev/)
- **fft.js**: A library for Fast Fourier Transform calculations. [GitHub Repository](https://github.com/indutny/fft.js)
- **Moattar, Mohammad & Homayoonpoor, Mahdi. (2010).** A simple but efficient real-time voice activity detection algorithm. [Research Paper](https://www.researchgate.net/publication/255667085_A_simple_but_efficient_real-time_voice_activity_detection_algorithm)
- **vitest**: A website for testing voice recognition. [Official Site](https://vitest.dev/)
- **Playwright**: A tool for automating browser testing. [Official Site](https://playwright.dev/)

### Audio Test Files

- `examples/albert.ogg`
  <a href="https://commons.wikimedia.org/wiki/File:03_ALBERT_EINSTEIN.ogg">Radio Universidad Nacional de La Plata</a>, <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0</a>, via Wikimedia Commons
- `examples/jfk.wav`: <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0</a>, via Wikimedia Commons
