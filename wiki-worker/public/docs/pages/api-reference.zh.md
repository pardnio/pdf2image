# 建構參數

本頁列出 `new pdf2image(options)` 的所有參數、預設值與正規化規則。

## 簽章

```typescript
new pdf2image(options?: {
  file?: ArrayBuffer;
  filename?: string;
  scale?: number;
  type?: "png" | "jpg" | "webp";
})
```

建構子只儲存參數，不讀取檔案、不觸發任何網路請求。

## 參數

| 參數 | 型別 | 預設值 | 正規化 | 說明 |
|---|---|---|---|---|
| `file` | `ArrayBuffer` | `undefined` | 無 | PDF 原始資料；轉換後會被 pdf.js 轉移而失效，見 [轉換流程](/zh/conversion#file-只能用一次) |
| `filename` | `string` | `""` | `trim()` 後移除第一個 `.pdf` | ZIP 與圖片的基礎檔名，支援 `yyyy`／`MM`／`DD`／`hh`／`mm`，見 [ZIP 下載](/zh/zip-download#檔名樣板) |
| `scale` | `number` | `1.5` | falsy 值（含 `0`）回退為 `1.5` | pdf.js viewport 縮放倍率；輸出像素 = 頁面 pt × `scale` |
| `type` | `string` | `"jpg"` | 非 `png`／`jpg`／`webp` 回退為 `"jpg"` | 輸出格式；`"jpg"` 實際產生 PNG，見 [轉換流程](/zh/conversion#輸出格式type) |

## 正規化範例

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

`type` 比對區分大小寫，`"PNG"` 也會回退為 `"jpg"`。

`filename` 只能是字串或省略：建構子直接對它呼叫 `.trim()`，傳入非字串的 truthy 值（例如 `123`）會在 `new pdf2image()` 時拋出 `TypeError`。

## 相關頁面

- [方法與屬性](/zh/api-reference-methods)
