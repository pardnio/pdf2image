# 轉換流程

本頁說明 `convert()` 如何把 PDF 頁面渲染成圖片，以及解析度、輸出格式與失敗時的實際行為。

## 流程

1. 設定 `pdfjsLib.GlobalWorkerOptions.workerSrc` 為 cdnjs 上的 pdf.worker.js
2. 以 `getDocument()` 解析 `file`，CMap 從 cdnjs 取得（處理 CJK 等字型編碼）
3. 解析成功後在 `<body>` 附加進度遮罩
4. 所有頁面同時建立 `canvas` 並渲染，每完成一頁就把 Data URL 推入結果陣列並更新進度
5. 全部完成後延遲 500ms，resolve 結果陣列並移除遮罩

## 解析度：`scale`

輸出像素尺寸 = PDF 頁面尺寸（pt）× `scale`。預設 `1.5`；US Letter（612×792 pt）在 `scale: 1` 時輸出 612×792，在預設值時輸出 918×1188。

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

## 輸出格式：`type`

`type` 只接受 `png`、`jpg`、`webp`，其他值回退為 `jpg`。結果由 `canvas.toDataURL("image/" + type)` 產生，因此：

| `type` | 傳給 `toDataURL` | 實際輸出 |
|---|---|---|
| `png` | `image/png` | PNG |
| `webp` | `image/webp` | WebP（瀏覽器不支援時為 PNG） |
| `jpg`（預設） | `image/jpg` | **PNG**：瀏覽器只認 `image/jpeg`，不認得的 MIME 一律回退 PNG |

因此使用預設 `type` 時，每個 Data URL 都以 `data:image/png` 開頭。

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

## 結果順序

頁面是併發渲染，結果在「每頁完成時」依序推入陣列，而非依頁碼寫入對應位置。頁面複雜度差異大時，`images[0]` 不保證是第 1 頁。

## `file` 只能用一次

pdf.js 會把傳入的 `ArrayBuffer` 轉移（transfer）給 worker，轉換後原 buffer 的 `byteLength` 變成 0。因此：

- 同一個實例第二次呼叫 `convert()` 會以 `TypeError` reject，訊息為「Cannot perform Construct on a detached ArrayBuffer」；已轉換的 `images` 不受影響
- 同一個 buffer 不能再交給另一個實例；需要重轉時重新讀取檔案

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

## 失敗行為

| 情境 | reject 的值 |
|---|---|
| 未提供 `file` | 字串 `"error The PDF file is empty, i.e. its size is zero bytes."` |
| 不是有效的 PDF | 字串 `"error Invalid PDF structure."` |
| 頁面渲染失敗 | 字串 `"error i is not defined"`：錯誤處理引用了不存在的變數，原始錯誤訊息遺失 |
| 重複使用已轉移的 buffer | `TypeError` 物件 |
| pdf.js 尚未載入 | `ReferenceError` 物件（見 [核心概念](/zh/core-concepts)） |

reject 值有字串也有 `Error`，以 `String(err)` 處理可涵蓋兩者。pdf.js 載入失敗時，套件本身也會呼叫 `console.error("error", err)`。

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

## 相關頁面

- [進度遮罩](/zh/progress-overlay)
- [方法與屬性](/zh/api-reference-methods)
