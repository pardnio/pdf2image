# 快速開始

本頁說明 pdf2image 的安裝方式，以及在瀏覽器完成第一次 PDF 轉圖片。

## 前置需求

| 項目 | 需求 |
|---|---|
| 執行環境 | 支援 ES2022 私有欄位（`#field`）、`canvas`、`Promise` 的現代瀏覽器 |
| 網路 | 可連線 `cdnjs.cloudflare.com`（pdf.js 3.4.120、JSZip 3.7.1）與 `cdn.jsdelivr.net`（樣式表） |
| 輸入 | PDF 的 `ArrayBuffer`，通常來自 `File.arrayBuffer()` |

pdf2image 只能在瀏覽器執行；Node.js 沒有 `document` 與 `window`，無法直接使用。

## 安裝

### npm

```bash
npm i @pardnchiu/pdf2image
```

### CDN（UMD）

```html
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.js"></script>
```

UMD 版載入後會註冊全域 `window.pdf2image`。

### CDN（ES Module）

```javascript
import { pdf2image } from "https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.esm.js";
```

ESM 版與 UMD 版是同一份程式，只在檔尾多匯出 `window.pdf2image`。

### 從原始碼建置

```bash
git clone https://github.com/pardnchiu/pdf2image.git
cd pdf2image
npm install
npm run minify
```

| 指令 | 產出 |
|---|---|
| `npm run minify` | 以 terser 將 `src/pdf2image.js` 壓縮為 `dist/pdf2image.js` 與 `dist/pdf2image.esm.js`（ESM 檔尾附加 `export` 那一行） |
| `npm run minify-w` | 以 chokidar 監看 `src/*.js`，變動時重跑 `minify` |
| `npm run sass` | 監看 `src/`，將 `src/pdf2image.scss` 編譯為 `dist/pdf2image.css` |

建置後的腳本仍從 `cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@latest/dist/pdf2image.css` 注入樣式表，本地編譯的 `dist/pdf2image.css` 要自行以 `<link>` 引入才會生效。

## 第一次轉換

下方範例載入時會自動轉換 `/assets/sample.pdf`（3 頁），也可以從檔案選擇器挑自己的 PDF。

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

三個要點：

- `file` 必須是 `ArrayBuffer`，不是 `File` 物件
- 等 `load` 事件後再呼叫 `convert()`，因為 pdf.js 是在套件載入時才非同步注入（見 [核心概念](/zh/core-concepts)）
- 失敗時 reject 的值可能是字串也可能是 `Error`，用 `String(err)` 統一輸出（見 [轉換流程](/zh/conversion)）

## 下一步

- [轉換流程](/zh/conversion)：`scale`、輸出格式與錯誤行為
- [ZIP 下載](/zh/zip-download)：打包下載與檔名樣板
- [建構參數](/zh/api-reference)：所有選項與預設值
