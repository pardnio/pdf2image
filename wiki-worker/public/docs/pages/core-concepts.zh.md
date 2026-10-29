# 核心概念

本頁說明 pdf2image 載入時對頁面做了什麼，以及這如何決定 `convert()` 的呼叫時機。

## 載入即注入相依

pdf2image 不打包 pdf.js 與 JSZip。腳本一執行，就立即在 `<head>` 插入下列元素：

| 元素 | 目標 | 用途 |
|---|---|---|
| `<link rel="preconnect">` | `https://cdnjs.cloudflare.com` | 提前建立連線 |
| `<link rel="preload" as="script">` | pdf.js、pdf.worker.js、JSZip | 提前下載 |
| `<script>` | pdf.js 3.4.120、JSZip 3.7.1 | 註冊全域 `pdfjsLib`、`JSZip` |
| `<link rel="stylesheet">` | `@pardnchiu/pdf2image@latest/dist/pdf2image.css` | 進度遮罩樣式 |

相依版本寫死在原始碼，無法透過參數替換。樣式表固定取 `@latest`，與你載入的套件版本無關。

## 呼叫時機

動態插入的 `<script>` 是非同步執行，套件載入後 `pdfjsLib` 是否已存在取決於快取與網路時序。尚未存在時呼叫 `convert()` 會以 `ReferenceError`（`pdfjsLib is not defined`）reject。等到 `window` 的 `load` 事件，注入的腳本都已執行完畢，因此請在此之後再轉換。

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

## 全域類別

UMD 與 ESM 兩種建置都會設定 `window.pdf2image`；ESM 只是額外以 `export const pdf2image = window.pdf2image` 匯出同一個類別。每個實例只持有自己的 `file`、`type`、`scale`、`filename` 與轉換結果，沒有跨實例共享的狀態。

## 相關頁面

- [轉換流程](/zh/conversion)
- [架構](/zh/architecture)
