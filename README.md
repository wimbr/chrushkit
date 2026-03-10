# CrushKit

**Browser-based image compressor. Drop images, get compressed JPG + WebP. Nothing ever leaves your device.**

---

## What it does

CrushKit takes your JPG or PNG images and compresses them into two optimized formats:

- **JPG** — resized to a maximum of 1920×1920 px, quality 82
- **WebP** — compressed to a target size of ~300 KB using a quality binary search

All output files are packaged into a single ZIP download with two folders:

```
crushkit_TIMESTAMP.zip
├── crushkit_jpg/
│   ├── photo1.jpg
│   └── photo2.jpg
└── crushkit_webp/
    ├── photo1.webp
    └── photo2.webp
```

## How to use

1. Open `index.html` in a browser (or host it statically)
2. Wait for the engine to load (~5 MB WASM download, cached after first visit)
3. Drag and drop images or folders onto the drop zone
4. A ZIP file downloads automatically when processing is done

Supported input formats: JPG, PNG (and any format ImageMagick can decode).
Folders are supported — files are collected recursively.

## Privacy — no uploads, ever

CrushKit runs entirely in your browser using **WebAssembly (WASM)**.

- No server receives your images
- No API calls are made during processing
- No analytics, no tracking
- Your files never leave your machine

The only network requests are:
1. The initial page load (HTML, JS, CSS)
2. The `magick.wasm` engine file (~5 MB), downloaded once and cached by the browser

After the engine is loaded, CrushKit works fully offline.

## How it works under the hood

CrushKit uses [ImageMagick compiled to WebAssembly](https://github.com/dlemstra/magick-wasm) (`magick-wasm`) to run the full ImageMagick engine directly in the browser.

Processing pipeline per image:

1. **Read** — image bytes are passed to ImageMagick WASM via `ImageMagick.read()`
2. **Resize** — if either dimension exceeds 1920 px, the image is scaled down proportionally to fit within a 1920×1920 box (never upscaled)
3. **JPG output** — written at quality 82
4. **WebP output** — quality is binary-searched between 1 and 75 to find the highest quality that keeps the file at or under 300 KB
5. **ZIP** — all outputs are packed in memory using [fflate](https://github.com/101arrowz/fflate) with store compression (level 0, since images are already compressed), then downloaded as a single ZIP

The service worker (`coi-serviceworker.js`) is used to set `Cross-Origin-Opener-Policy` and `Cross-Origin-Embedder-Policy` headers, which are required for `SharedArrayBuffer` — a dependency of the WASM engine.

## Running locally

No build step required. Just serve the files over HTTP (not `file://`, due to service worker requirements):

```bash
npx serve .
# or
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

## License

CrushKit is open source. The bundled `magick-wasm` engine is licensed under the [Apache 2.0 License](package/LICENSE).
