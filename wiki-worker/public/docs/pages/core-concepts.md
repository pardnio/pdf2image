# Core Concepts

This page explains what pdf2image does to the page when it loads, and how that decides when you can call `convert()`.

## Dependencies Injected on Load

pdf2image does not bundle pdf.js or JSZip. As soon as the script runs, it inserts these elements into `<head>`:

| Element | Target | Purpose |
|---|---|---|
| `<link rel="preconnect">` | `https://cdnjs.cloudflare.com` | Opens the connection early |
| `<link rel="preload" as="script">` | pdf.js, pdf.worker.js, JSZip | Downloads early |
| `<script>` | pdf.js 3.4.120, JSZip 3.7.1 | Registers the globals `pdfjsLib` and `JSZip` |
| `<link rel="stylesheet">` | `@pardnchiu/pdf2image@latest/dist/pdf2image.css` | Progress overlay styles |

The dependency versions are hard-coded in the source and cannot be swapped through options. The stylesheet always comes from `@latest`, regardless of the package version you load.

## When to Call

Dynamically inserted `<script>` elements execute asynchronously, so whether `pdfjsLib` exists right after the package loads depends on cache and network timing. When it does not, `convert()` rejects with a `ReferenceError` (`pdfjsLib is not defined`). By the `window` `load` event, every injected script has executed, so wait for it before converting.

```demo
<pre id="out"></pre>
<script>
  console.log("immediately:", typeof pdfjsLib, typeof JSZip);
  addEventListener("load", () => {
    console.log("after load:", typeof pdfjsLib, typeof JSZip);
    const injected = [...document.head.querySelectorAll("script[src], link[href]")]
      .map(el => el.src || el.href)
      .filter(url => !url.includes("@pardnchiu/pdf2image@") || url.endsWith(".css"));
    document.getElementById("out").textContent = injected.join("\n");
  });
</script>
```

## The Global Class

Both the UMD and ESM builds set `window.pdf2image`; the ESM build only adds `export const pdf2image = window.pdf2image` to export the same class. Each instance holds its own `file`, `type`, `scale`, `filename`, and conversion results, with no state shared across instances.

## Related Pages

- [Conversion](/conversion)
- [Architecture](/architecture)
