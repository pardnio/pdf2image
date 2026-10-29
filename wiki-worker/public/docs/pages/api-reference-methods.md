# Methods and Properties

This page lists the public methods and properties of a `pdf2image` instance, with what each resolves or rejects with.

## Overview

| Member | Signature | Resolves | Rejects |
|---|---|---|---|
| `convert` | `convert(): Promise<string[]>` | Array of Data URLs (the same array as `images`) | A string or an `Error` (see [Conversion](/conversion#failure-behavior)) |
| `download` | `download(): Promise<void>` | `undefined` (download triggered, or skipped because `file` is empty) | `convert()`'s rejection value, or `[]` when JSZip fails |
| `images` | `get images(): string[]` | — | — |

## `convert()`

Renders every page concurrently and returns an array of Data URLs, showing the progress overlay meanwhile. Each instance can succeed only once, because `file` is transferred on the first conversion. See [Conversion](/conversion) for details.

## `download()`

Bundles the results into `{filename}.zip` and triggers a browser download; it calls `convert()` first when nothing has been converted. See [ZIP Download](/zip-download) for details.

## `images`

A read-only getter that returns the internal result array itself, not a copy. It is empty before conversion; the array `convert()` resolves with is the same reference, so changing either one changes the other and also changes what a later `download()` bundles.

```demo
<script>
  addEventListener("load", async () => {
    const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
    const converter = new pdf2image({ file: buffer, type: "png", scale: 0.5 });
    console.log("before:", converter.images.length);
    const result = await converter.convert();
    console.log("after:", converter.images.length, "same array:", result === converter.images);
    result.pop();
    console.log("after pop:", converter.images.length);
  });
</script>
```

## Non-Public Members

`#filename`, `#file`, `#type`, `#scale`, `#result`, and `#formatDate()` are ES private fields and cannot be accessed from outside. The overlay class `pdf2imageLoading` is used only inside the module and is not attached to `window`.

## Related Pages

- [Constructor Options](/api-reference)
