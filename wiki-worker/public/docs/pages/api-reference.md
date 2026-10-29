# Constructor Options

This page lists every `new pdf2image(options)` option with its default and normalization rule.

## Signature

```typescript
new pdf2image(options?: {
  file?: ArrayBuffer;
  filename?: string;
  scale?: number;
  type?: "png" | "jpg" | "webp";
})
```

The constructor only stores options; it reads no file and makes no network request.

## Options

| Option | Type | Default | Normalization | Description |
|---|---|---|---|---|
| `file` | `ArrayBuffer` | `undefined` | None | Raw PDF data; pdf.js transfers it during conversion, which invalidates it (see [Conversion](/conversion#file-is-single-use)) |
| `filename` | `string` | `""` | `trim()`, then the first `.pdf` is removed | Base name for the ZIP and images; accepts `yyyy` / `MM` / `DD` / `hh` / `mm` (see [ZIP Download](/zip-download#filename-template)) |
| `scale` | `number` | `1.5` | Falsy values (including `0`) fall back to `1.5` | pdf.js viewport scale; output pixels = page pt × `scale` |
| `type` | `string` | `"jpg"` | Anything other than `png` / `jpg` / `webp` falls back to `"jpg"` | Output format; `"jpg"` actually produces PNG (see [Conversion](/conversion#output-format-type)) |

## Normalization Examples

```demo
<script>
  const size = src => new Promise(r => {
    const img = new Image();
    img.onload = () => r(img.width + "x" + img.height);
    img.src = src;
  });
  addEventListener("load", async () => {
    const options = [{}, { scale: 0 }, { type: "gif" }, { type: "PNG" }];
    for (const extra of options) {
      const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
      const [first] = await new pdf2image({ file: buffer, ...extra }).convert();
      console.log(JSON.stringify(extra), "->", first.slice(5, first.indexOf(";")), await size(first));
    }
  });
</script>
```

`type` matching is case-sensitive, so `"PNG"` also falls back to `"jpg"`.

`filename` must be a string or omitted: the constructor calls `.trim()` on it directly, so a truthy non-string value (such as `123`) throws a `TypeError` at `new pdf2image()`.

## Related Pages

- [Methods and Properties](/api-reference-methods)
