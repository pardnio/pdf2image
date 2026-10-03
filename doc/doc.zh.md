# pdf2image - 技術文件

> 返回 [README](./README.zh.md)

## 前置需求

- 支援 ES2022 私有欄位（`#field`）、`Promise`、`canvas` 與 `URL.createObjectURL` 的現代瀏覽器
- 可連線至 `cdnjs.cloudflare.com`（pdf.js 3.4.120、JSZip 3.7.1）與 `cdn.jsdelivr.net`（樣式表）
- 僅於瀏覽器環境運作；Node.js 無 `document`／`window`，無法直接使用

## 安裝

### 透過 npm

```bash
npm i @pardnchiu/pdf2image
```

### 透過 CDN（UMD）

```html
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.js"></script>
```

### 透過 CDN（ES Module）

```javascript
import { pdf2image } from "https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.esm.js";
```

### 從原始碼建置

```bash
git clone https://github.com/pardnchiu/pdf2image.git
cd pdf2image
npm install
npm run minify
```

`npm run minify` 以 terser 將 `src/pdf2image.js` 壓縮為 `dist/pdf2image.js` 與 `dist/pdf2image.esm.js`；`npm run sass` 監看並編譯 `src/pdf2image.scss` 至 `dist/pdf2image.css`。

## 使用方式

### 基礎：轉換並取得圖片

```html
<input type="file" accept="application/pdf" id="pdf">
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.js"></script>
<script>
    document.getElementById("pdf").addEventListener("change", async e => {
        const file = e.target.files[0];
        if (!file) {
            return;
        }

        // file 參數必須是 ArrayBuffer
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
            console.error("轉換失敗", err);
        }
    });
</script>
```

### 進階：日期樣板檔名 + ZIP 下載

```javascript
import { pdf2image } from "https://cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@1.1.2/dist/pdf2image.esm.js";

async function exportZip(file) {
    const converter = new pdf2image({
        // 產出 report 2026-10-04 13:05.zip，內含 report 2026-10-04 13:05 0.webp ...
        filename: "report yyyy-MM-DD hh:mm",
        file: await file.arrayBuffer(),
        scale: 2,
        type: "webp"
    });

    try {
        // 尚未 convert 時，download 會先自動執行 convert
        await converter.download();
    } catch (err) {
        console.error("下載失敗", err);
    }
}
```

## API 參考

### 模組載入副作用

載入 `pdf2image.js` 時會立即在 `<head>` 注入：

| 類型 | 目標 |
|------|------|
| `preconnect` | `https://cdnjs.cloudflare.com` |
| `preload` | pdf.js、pdf.worker.js、JSZip |
| `<script>` | pdf.js 3.4.120、JSZip 3.7.1 |
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

| 參數 | 型別 | 預設值 | 說明 |
|------|------|--------|------|
| `filename` | `string` | `""` | 輸出檔名；自動去除 `.pdf`，支援 `yyyy`／`MM`／`DD`／`hh`／`mm` 日期樣板 |
| `file` | `ArrayBuffer` | — | PDF 原始資料，通常來自 `File.arrayBuffer()` 或 `FileReader.readAsArrayBuffer` |
| `scale` | `number` | `1.5` | pdf.js viewport 縮放倍率，越大畫質越高、耗時越久 |
| `type` | `string` | `"jpg"` | 輸出格式；非 `png`／`jpg`／`webp` 時回退為 `jpg` |

### 方法與屬性

| 成員 | 簽章 | 說明 |
|------|------|------|
| `convert` | `convert(): Promise<string[]>` | 併發渲染所有頁面，resolve 為 Data URL 陣列；失敗時 reject 錯誤訊息字串 |
| `download` | `download(): Promise<void>` | 將結果打包為 `{filename}.zip` 並觸發瀏覽器下載；無結果時先呼叫 `convert`；`file` 為空時直接返回 |
| `images` | `get images(): string[]` | 目前已轉換的 Data URL 陣列 |

### 進度遮罩

轉換與壓縮期間會在 `<body>` 附加 `div.pdf2image-loading`，`data-percent` 屬性為 0–100 的進度值；提示文字依 `navigator.language` 顯示「解析中／準備壓縮檔」或「Processing／Preparing zip」。

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
