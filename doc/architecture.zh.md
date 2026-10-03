# pdf2image - 架構

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    Script[pdf2image.js 載入] --> Boot[啟動 IIFE]
    Boot --> CDN[CDN 相依注入<br/>pdf.js / JSZip / CSS]
    App[呼叫端] --> Converter[pdf2image 類別]
    Converter -->|convert| PDFJS[pdf.js 渲染<br/>canvas → Data URL]
    Converter -->|download| ZIP[JSZip 打包<br/>Blob 下載]
    PDFJS --> Loading[pdf2imageLoading 進度遮罩]
    ZIP --> Loading
    PDFJS --> Images[images 陣列]
    Images --> ZIP
```

## Module: 啟動注入

模組載入時立即執行，將外部相依掛入 `<head>`。

```mermaid
graph TB
    subgraph 啟動 IIFE
        A[preconnect<br/>cdnjs.cloudflare.com] --> B[preload<br/>pdf.js / worker / JSZip]
        B --> C[script<br/>pdf.js / JSZip]
        C --> D[stylesheet<br/>pdf2image.css]
    end
    D --> Head[document.head]
    C --> Global[全域 pdfjsLib / JSZip]
```

## Module: pdf2image

對外唯一類別，負責參數正規化、頁面渲染與 ZIP 打包。

```mermaid
classDiagram
    class pdf2image {
        -string filename
        -ArrayBuffer file
        -string type
        -number scale
        -string[] result
        +constructor(body)
        +get images() string[]
        +convert() Promise~string[]~
        +download() Promise~void~
        -formatDate(template) string
    }
    class pdf2imageLoading {
        +HTMLElement body
        +percent(index, total, prepare)
        +remove()
        -loadingText(percent, prepare) string
    }
    pdf2image ..> pdf2imageLoading : 建立
    pdf2image ..> pdfjsLib : 渲染
    pdf2image ..> JSZip : 打包
```

## Module: pdf2imageLoading

進度遮罩元件，以 `data-percent` 與文字呈現百分比，依 `navigator.language` 切換語系。

```mermaid
graph LR
    subgraph pdf2imageLoading
        Ctor[constructor<br/>建立 div.pdf2image-loading] --> Percent[percent<br/>更新 data-percent 與文字]
        Percent --> Text[loadingText<br/>zh / en 對照]
        Percent --> Remove[remove]
    end
    Lang[navigator.language] --> Text
    Ctor --> Body[document.body]
```

## 資料流

```mermaid
sequenceDiagram
    participant App as 呼叫端
    participant P as pdf2image
    participant L as 進度遮罩
    participant PDF as pdf.js
    participant Z as JSZip
    participant DOM as 瀏覽器
    App->>P: new pdf2image({ file, type, scale, filename })
    App->>P: convert()
    P->>PDF: getDocument(Uint8Array)
    PDF-->>P: pdf
    P->>L: 附加至 body
    par 每一頁併發
        P->>PDF: getPage(i) → render(canvas)
        PDF-->>P: canvas.toDataURL(type)
        P->>L: percent(completed, total)
    end
    P-->>App: images（延遲 500ms）
    P->>L: remove()
    App->>P: download()
    P->>Z: file(name, base64) × N
    Z->>L: percent(壓縮進度)
    Z-->>P: Blob
    P->>DOM: a[download].click()
    P->>L: remove()
    P-->>App: resolve
```

## 狀態機

```mermaid
stateDiagram-v2
    [*] --> 已建立: new pdf2image
    已建立 --> 轉換中: convert()
    已建立 --> 轉換中: download() 且無結果
    已建立 --> [*]: download() 且 file 為空
    轉換中 --> 已轉換: 全部頁面完成
    轉換中 --> 失敗: pdf.js 錯誤
    已轉換 --> 打包中: download()
    打包中 --> 已轉換: 下載觸發完成
    打包中 --> 失敗: JSZip 錯誤
```

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
