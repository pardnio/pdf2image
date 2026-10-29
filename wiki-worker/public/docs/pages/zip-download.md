# ZIP Download

This page explains how `download()` bundles conversion results into a ZIP and triggers a browser download, and how filename templates work.

## Flow

1. When `file` is `null` / `undefined`, it resolves `undefined` immediately without converting or downloading
2. When there are no results yet, it first runs `await this.convert()`; existing results are reused without re-rendering
3. Appends the progress overlay to `<body>` and adds every image to JSZip as base64
4. Builds the ZIP with `generateAsync({ type: "blob", streamFiles: true })`, syncing compression progress to the overlay
5. Creates an `<a download>` pointing at `URL.createObjectURL(blob)`, then after 500ms resolves, clicks it, and removes the link and the overlay

```demo
<button id="save">Download sample.zip</button>
<script>
  document.getElementById("save").addEventListener("click", async e => {
    e.target.disabled = true;
    const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
    const converter = new pdf2image({ file: buffer, filename: "sample.pdf yyyy-MM-DD", type: "png", scale: 0.5 });
    try {
      await converter.download();
      console.log("downloaded", converter.images.length, "images");
    } catch (err) {
      console.log("failed:", String(err));
    }
    e.target.disabled = false;
  });
  new pdf2image({}).download().then(result => console.log("no file ->", String(result)));
</script>
```

## Filename Template

At construction `filename` is trimmed and its **first** `.pdf` is removed (case-sensitive, anywhere in the string). At download time these date tokens are replaced with local time:

| Token | Meaning | Example |
|---|---|---|
| `yyyy` | Four-digit year | `2026` |
| `MM` | Month, zero-padded | `10` |
| `DD` | Day, zero-padded | `04` |
| `hh` | Hour (24-hour), zero-padded | `13` |
| `mm` | Minute, zero-padded | `05` |

## Output Names

| Item | Rule | With `filename: "report.pdf yyyy-MM-DD"`, `type: "png"` |
|---|---|---|
| ZIP | `{name}.zip` | `report 2026-10-04.zip` |
| Image | `{name} {index}.{type}`, index starts at 0 | `report 2026-10-04 0.png`, `report 2026-10-04 1.png` |

Notes:

- The image extension comes straight from `type`; with `type: "jpg"` the extension is `.jpg` but the content is PNG (see [Conversion](/conversion))
- Without `filename`, the ZIP is named `.zip` and images get names that start with a space, such as ` 0.jpg`
- Image indexes follow the result array order, which is not guaranteed to match page order

## Failure Behavior

| Case | Result |
|---|---|
| The automatic conversion fails | Rejects with `convert()`'s rejection value; no compression overlay appears |
| JSZip fails to generate | Rejects with an empty array `[]` and logs the original error to the console; the compression overlay is not removed and stays on the page |

## Related Pages

- [Progress Overlay](/progress-overlay)
- [Methods and Properties](/api-reference-methods)
