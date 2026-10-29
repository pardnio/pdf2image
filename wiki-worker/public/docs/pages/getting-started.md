# Getting Started

This page covers installing pdf2image and running your first PDF-to-image conversion in the browser.

## Prerequisites

| Item | Requirement |
|---|---|
| Runtime | A modern browser with ES2022 private fields (`#field`), `canvas`, and `Promise` |
| Network | Access to `cdnjs.cloudflare.com` (pdf.js 3.4.120, JSZip 3.7.1) and `cdn.jsdelivr.net` (stylesheet) |
| Input | The PDF as an `ArrayBuffer`, usually from `File.arrayBuffer()` |

pdf2image runs in the browser only; Node.js has no `document` or `window` and cannot run it directly.

## Installation

### npm

```bash
npm i @pardnchiu/pdf2image
```

### CDN (UMD)

```html
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.js"></script>
```

The UMD build registers the global `window.pdf2image` on load.

### CDN (ES Module)

```javascript
import { pdf2image } from "https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.esm.js";
```

The ESM build is the same code as the UMD build, with one extra line that exports `window.pdf2image`.

### From Source

```bash
git clone https://github.com/pardnchiu/pdf2image.git
cd pdf2image
npm install
npm run minify
```

| Script | Output |
|---|---|
| `npm run minify` | Compresses `src/pdf2image.js` with terser into `dist/pdf2image.js` and `dist/pdf2image.esm.js` (the ESM file gets the extra `export` line appended) |
| `npm run minify-w` | Watches `src/*.js` with chokidar and reruns `minify` on change |
| `npm run sass` | Watches `src/` and compiles `src/pdf2image.scss` into `dist/pdf2image.css` |

The built script still injects the stylesheet from `cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@latest/dist/pdf2image.css`, so a locally compiled `dist/pdf2image.css` only takes effect when you link it yourself.

## First Conversion

The example below converts `/assets/sample.pdf` (3 pages) on load; you can also pick your own PDF with the file input.

```demo
<input type="file" accept="application/pdf" id="pick">
<div id="pages"></div>
<script>
  async function run(buffer) {
    const converter = new pdf2image({ file: buffer, type: "png", scale: 0.5 });
    try {
      const images = await converter.convert();
      document.getElementById("pages").replaceChildren(
        ...images.map(src => Object.assign(document.createElement("img"), { src, width: 120 }))
      );
      console.log("pages:", images.length);
    } catch (err) {
      console.log("failed:", String(err));
    }
  }
  document.getElementById("pick").addEventListener("change", async e => {
    const file = e.target.files[0];
    if (file) run(await file.arrayBuffer());
  });
  addEventListener("load", async () => {
    run(await (await fetch("/assets/sample.pdf")).arrayBuffer());
  });
</script>
```

Three things to note:

- `file` must be an `ArrayBuffer`, not a `File` object
- Call `convert()` after the `load` event, because pdf.js is injected asynchronously when the package loads (see [Core Concepts](/core-concepts))
- The rejection value can be a string or an `Error`; `String(err)` prints both (see [Conversion](/conversion))

## Next Steps

- [Conversion](/conversion): `scale`, output formats, and error behavior
- [ZIP Download](/zip-download): bundling and filename templates
- [Constructor Options](/api-reference): every option and its default
