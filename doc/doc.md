# pdf2image - Documentation

> Back to [README](../README.md)

## Prerequisites

- A modern browser with ES2022 private fields (`#field`), `Promise`, `canvas`, and `URL.createObjectURL`
- Network access to `cdnjs.cloudflare.com` (pdf.js 3.4.120, JSZip 3.7.1) and `cdn.jsdelivr.net` (stylesheet)
- Browser only; Node.js lacks `document` / `window` and cannot run it directly

## Installation

### Using npm

```bash
npm i @pardnchiu/pdf2image
```

### Using CDN (UMD)

```html
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.js"></script>
```

### Using CDN (ES Module)

```javascript
import { pdf2image } from "https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.esm.js";
```

### From Source

```bash
git clone https://github.com/pardnchiu/pdf2image.git
cd pdf2image
npm install
npm run minify
```

`npm run minify` uses terser to compress `src/pdf2image.js` into `dist/pdf2image.js` and `dist/pdf2image.esm.js`; `npm run sass` watches and compiles `src/pdf2image.scss` into `dist/pdf2image.css`.

## Usage

### Basic: Convert and Get Images

```html
<input type="file" accept="application/pdf" id="pdf">
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.js"></script>
<script>
    document.getElementById("pdf").addEventListener("change", async e => {
        const file = e.target.files[0];
        if (!file) {
            return;
        }

        // file must be an ArrayBuffer
        const buffer = await file.arrayBuffer();

        const converter = new pdf2image({
            filename: file.name,
            file: buffer,
            scale: 1.5,
            type: "png"
        });

        try {
            const images = await converter.convert();
            images.forEach(src => {
                const img = document.createElement("img");
                img.src = src;
                document.body.appendChild(img);
            });
        } catch (err) {
            console.error("Conversion failed", err);
        }
    });
</script>
```

### Advanced: Date-Template Filename + ZIP Download

```javascript
import { pdf2image } from "https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.esm.js";

async function exportZip(file) {
    const converter = new pdf2image({
        // Produces report 2026-10-04 13:05.zip containing report 2026-10-04 13:05 0.webp ...
        filename: "report yyyy-MM-DD hh:mm",
        file: await file.arrayBuffer(),
        scale: 2,
        type: "webp"
    });

    try {
        // download runs convert first when nothing has been converted yet
        await converter.download();
    } catch (err) {
        console.error("Download failed", err);
    }
}
```

## API Reference

### Load-Time Side Effects

Loading `pdf2image.js` immediately injects into `<head>`:

| Kind | Target |
|------|--------|
| `preconnect` | `https://cdnjs.cloudflare.com` |
| `preload` | pdf.js, pdf.worker.js, JSZip |
| `<script>` | pdf.js 3.4.120, JSZip 3.7.1 |
| `stylesheet` | `@pardnchiu/pdf2image@latest/dist/pdf2image.css` |

### `new pdf2image(options)`

```javascript
new pdf2image({
    filename?: string,
    file: ArrayBuffer,
    scale?: number,
    type?: "png" | "jpg" | "webp"
})
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `filename` | `string` | `""` | Output filename; strips `.pdf` and accepts `yyyy` / `MM` / `DD` / `hh` / `mm` date tokens |
| `file` | `ArrayBuffer` | — | Raw PDF data, usually from `File.arrayBuffer()` or `FileReader.readAsArrayBuffer` |
| `scale` | `number` | `1.5` | pdf.js viewport scale; higher values yield sharper images and longer render time |
| `type` | `string` | `"jpg"` | Output format; falls back to `jpg` for anything other than `png` / `jpg` / `webp` |

### Methods and Properties

| Member | Signature | Description |
|--------|-----------|-------------|
| `convert` | `convert(): Promise<string[]>` | Renders all pages concurrently and resolves with Data URLs; rejects with an error message string |
| `download` | `download(): Promise<void>` | Bundles results into `{filename}.zip` and triggers a browser download; calls `convert` first when empty; returns immediately when `file` is missing |
| `images` | `get images(): string[]` | Data URLs converted so far |

### Progress Overlay

During conversion and compression the library appends `div.pdf2image-loading` to `<body>`, with a `data-percent` attribute from 0 to 100; the label reads "Processing / Preparing zip" or "解析中 / 準備壓縮檔" based on `navigator.language`.

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
