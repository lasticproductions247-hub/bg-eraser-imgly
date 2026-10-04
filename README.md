# BG Eraser — AI Background Remover

Free AI background removal that runs **entirely in the browser**. No upload, no
server, no API key, no account. Drop in a photo, get a transparent PNG.

## What it does

- **True AI cutout** — a real segmentation model (ISNet, `small` tier) runs on
  your machine via ONNX Runtime Web.
- **Three backdrop modes** — keep the AI alpha (transparent), flatten onto a
  solid colour, or flatten onto a blurred version of the original.
- **Edge feather** — 0–5px blur on the cutout edge to soften the cut.
- **Drag-to-compare slider** — wipe between the original and the cutout.
- **Hold-to-compare button** — press and hold to see the original.
- **Full-resolution download** — the exported PNG matches your input size, not
  the on-screen preview.
- **Sample image and URL loader** for a quick trial.

## The important part: the model CDN path

`@imgly/background-removal` defaults its asset path to **data version 1.5.5**:

```
https://staticimgly.com/@imgly/background-removal-data/1.5.5/dist/
```

**That path returns HTTP 404.** Only data version **1.4.5** is actually
published, and the files inside it are content-hashed (`dist/<sha256>`), so the
obvious filenames don't exist either.

Left unpatched, the app loads cleanly and then fails on the first image with
`AI failed (...)`. The fix is to pass an explicit `publicPath`:

```js
await removeBgFn(file, {
  publicPath: 'https://staticimgly.com/@imgly/background-removal-data/1.4.5/dist/',
  model: 'small',
  output: { format: 'image/png', quality: 0.8 }
});
```

Verified on this machine at that path: all 11 model chunks and the ONNX Runtime
WASM binaries respond HTTP 206/200 with real ONNX bytes (magic
`onnx.quantized`). The `small` model is **44.3 MB** and the browser caches it
after the first run.

## Preloading

The model download starts **on page load**, not on click. A welcome screen
shows live byte progress while the weights stream in, so by the time you've read
the intro and picked a photo the model is usually already warm.

A failed preload clears both the cached promise *and* the library reference, so
the next image retries cleanly instead of failing against a half-initialised
module.

## Privacy

Your image never leaves the device. There is no upload endpoint, no fetch of
your file to any server, and no telemetry. The only network requests are for the
library itself and the model weights, both from public CDNs.

## Stack

Single self-contained `index.html`. No build step, no framework, no bundler.

| Piece | Choice |
|---|---|
| Model | `@imgly/background-removal@1.5.5` (`small` = 44.3 MB) |
| Runtime | ONNX Runtime Web (WASM) |
| CDN | jsDelivr `+esm` for code, `staticimgly.com` for weights |
| UI | Hand-written CSS, Space Grotesk + Font Awesome |

## Local use

No server needed — the file works over `file://` too, though a local server is
closer to production:

```bash
cd bg-eraser-imgly
python -m http.server 8080
# then open http://localhost:8080
```

## Deploy

GitHub Pages, workflow mode. `main` → live URL above.

## Licence

MIT. The ISNet model weights are © imgly GmbH, served from imgly's CDN under
their own licence.