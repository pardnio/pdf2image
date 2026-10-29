# Progress Overlay

This page covers the progress overlay that appears automatically during `convert()` and `download()`: its DOM, how progress is computed, localization, and styles.

## DOM Structure

The overlay is a single element appended to the end of `<body>` and removed automatically when work finishes:

```html
<div class="pdf2image-loading" data-percent="66">
  <p>Processing 66%</p>
</div>
```

| Attribute | Content |
|---|---|
| `class` | Always `pdf2image-loading` |
| `data-percent` | An integer from 0 to 100; the stylesheet sizes the bar from this value |
| `<p>` text | Phase label plus percentage |

## Progress Sources

| Phase | Computation | Label (English / Chinese) |
|---|---|---|
| Conversion | `round(finished pages / total pages × 100)` | `Processing N%` / `解析中 N%` |
| Compression | The `percent` reported by JSZip `generateAsync`, rounded | `Preparing zip N%` / `準備壓縮檔 N%` |

The conversion overlay appears only after the PDF parses successfully; a parse failure never shows it.

```demo
<script>
  new MutationObserver(records => {
    for (const r of records) {
      const el = r.target.closest?.(".pdf2image-loading") || [...r.addedNodes].find(n => n.classList?.contains("pdf2image-loading"));
      if (el && r.type === "attributes") console.log("data-percent", el.dataset.percent, "|", el.innerText);
      for (const n of r.addedNodes) if (n.classList?.contains("pdf2image-loading")) console.log("overlay added");
      for (const n of r.removedNodes) if (n.classList?.contains("pdf2image-loading")) console.log("overlay removed");
    }
  }).observe(document.body, { childList: true, subtree: true, attributes: true, attributeFilter: ["data-percent"] });
  addEventListener("load", async () => {
    const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
    await new pdf2image({ file: buffer, type: "png", scale: 0.5 }).convert();
  });
</script>
```

## Localization

The package reads `navigator.language` (or `navigator.userLanguage`) once at load. A value starting with `zh` (case-insensitive) selects Chinese labels; anything else uses English. Changing the language after load has no effect.

## Styles

The stylesheet is injected at load from `cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@latest/dist/pdf2image.css`:

- The overlay is a 240×16px rounded bar, `position: absolute` at `top: calc(50% - 12px)` and `left: calc(50% - 120px)`
- The progress bar is the `::after` pseudo-element, whose width maps `[data-percent="N"]` selectors to `N%`
- `position: absolute` positions it relative to the document rather than the viewport (unless an ancestor of the overlay is positioned), so on a scrolled page it sits at the middle of the document's first screen; override `div.pdf2image-loading` in your own stylesheet (for example `position: fixed`) to pin it to the viewport

## Related Pages

- [Conversion](/conversion)
- [ZIP Download](/zip-download)
