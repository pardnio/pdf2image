# Conversion

This page explains how `convert()` renders PDF pages into images, and what actually happens with resolution, output format, and failures.

## Flow

1. Sets `pdfjsLib.GlobalWorkerOptions.workerSrc` to pdf.worker.js on cdnjs
2. Parses `file` with `getDocument()`, fetching CMaps from cdnjs (for CJK and other font encodings)
3. Appends the progress overlay to `<body>` once parsing succeeds
4. Creates a `canvas` for every page and renders them concurrently; each finished page pushes its Data URL into the result array and updates progress
5. Waits 500ms after the last page, resolves with the result array, and removes the overlay

## Resolution: `scale`

Output pixel size = PDF page size (pt) × `scale`. The default is `1.5`; a US Letter page (612×792 pt) renders at 612×792 with `scale: 1` and at 918×1188 with the default.

```demo
<script>
  const sample = async () => (await fetch("/assets/sample.pdf")).arrayBuffer();
  const size = src => new Promise(r => {
    const img = new Image();
    img.onload = () => r(img.width + "x" + img.height);
    img.src = src;
  });
  addEventListener("load", async () => {
    for (const scale of [0.5, 1, 1.5]) {
      const [first] = await new pdf2image({ file: await sample(), type: "png", scale }).convert();
      console.log("scale", scale, "->", await size(first));
    }
  });
</script>
```

## Output Format: `type`

`type` accepts only `png`, `jpg`, and `webp`; any other value falls back to `jpg`. Results come from `canvas.toDataURL("image/" + type)`, so:

| `type` | Passed to `toDataURL` | Actual output |
|---|---|---|
| `png` | `image/png` | PNG |
| `webp` | `image/webp` | WebP (PNG where the browser lacks WebP encoding) |
| `jpg` (default) | `image/jpg` | **PNG**: browsers only recognize `image/jpeg` and fall back to PNG for unknown MIME types |

So with the default `type`, every Data URL starts with `data:image/png`.

```demo
<script>
  addEventListener("load", async () => {
    for (const type of ["png", "jpg", "webp", "gif"]) {
      const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
      const [first] = await new pdf2image({ file: buffer, type, scale: 0.5 }).convert();
      console.log(type.padEnd(4), "->", first.slice(0, first.indexOf(";")));
    }
  });
</script>
```

## Result Order

Pages render concurrently, and each result is pushed onto the array when its page finishes rather than written at its page index. When page complexity varies a lot, `images[0]` is not guaranteed to be page 1.

## `file` Is Single-Use

pdf.js transfers the `ArrayBuffer` to its worker, so after conversion the original buffer's `byteLength` becomes 0. As a result:

- A second `convert()` on the same instance rejects with a `TypeError` whose message is "Cannot perform Construct on a detached ArrayBuffer"; the already converted `images` stay intact
- The same buffer cannot be handed to another instance; read the file again to reconvert

```demo
<script>
  addEventListener("load", async () => {
    const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
    const converter = new pdf2image({ file: buffer, type: "png", scale: 0.5 });
    console.log("first:", (await converter.convert()).length, "pages");
    console.log("buffer byteLength:", buffer.byteLength);
    try {
      await converter.convert();
    } catch (err) {
      console.log("second:", String(err));
    }
    console.log("images kept:", converter.images.length);
  });
</script>
```

## Failure Behavior

| Case | Rejection value |
|---|---|
| No `file` provided | String `"error The PDF file is empty, i.e. its size is zero bytes."` |
| Not a valid PDF | String `"error Invalid PDF structure."` |
| A page fails to render | String `"error i is not defined"`: the error handler references a variable that does not exist, so the original message is lost |
| Reusing a transferred buffer | A `TypeError` object |
| pdf.js not loaded yet | A `ReferenceError` object (see [Core Concepts](/core-concepts)) |

Rejection values mix strings and `Error` objects; `String(err)` handles both. For pdf.js load failures the library also calls `console.error("error", err)` itself.

```demo
<script>
  addEventListener("load", async () => {
    const cases = { "no file": {}, "not a pdf": { file: new TextEncoder().encode("hello").buffer } };
    for (const [name, options] of Object.entries(cases)) {
      try {
        await new pdf2image(options).convert();
      } catch (err) {
        console.log(name, "->", typeof err, String(err));
      }
    }
  });
</script>
```

## Related Pages

- [Progress Overlay](/progress-overlay)
- [Methods and Properties](/api-reference-methods)
