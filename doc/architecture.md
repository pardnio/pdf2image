# pdf2image - Architecture

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    Script[pdf2image.js loaded] --> Boot[Bootstrap IIFE]
    Boot --> CDN[CDN dependency injection<br/>pdf.js / JSZip / CSS]
    App[Caller] --> Converter[pdf2image class]
    Converter -->|convert| PDFJS[pdf.js render<br/>canvas → Data URL]
    Converter -->|download| ZIP[JSZip bundle<br/>Blob download]
    PDFJS --> Loading[pdf2imageLoading overlay]
    ZIP --> Loading
    PDFJS --> Images[images array]
    Images --> ZIP
```

## Module: Bootstrap Injection

Runs immediately on load and attaches external dependencies to `<head>`.

```mermaid
graph TB
    subgraph Bootstrap IIFE
        A[preconnect<br/>cdnjs.cloudflare.com] --> B[preload<br/>pdf.js / worker / JSZip]
        B --> C[script<br/>pdf.js / JSZip]
        C --> D[stylesheet<br/>pdf2image.css]
    end
    D --> Head[document.head]
    C --> Global[global pdfjsLib / JSZip]
```

## Module: pdf2image

The only public class; normalizes options, renders pages, and bundles the ZIP.

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
    pdf2image ..> pdf2imageLoading : creates
    pdf2image ..> pdfjsLib : renders
    pdf2image ..> JSZip : bundles
```

## Module: pdf2imageLoading

Progress overlay that shows a percentage through `data-percent` and a label localized by `navigator.language`.

```mermaid
graph LR
    subgraph pdf2imageLoading
        Ctor[constructor<br/>create div.pdf2image-loading] --> Percent[percent<br/>update data-percent and label]
        Percent --> Text[loadingText<br/>zh / en lookup]
        Percent --> Remove[remove]
    end
    Lang[navigator.language] --> Text
    Ctor --> Body[document.body]
```

## Data Flow

```mermaid
sequenceDiagram
    participant App as Caller
    participant P as pdf2image
    participant L as Overlay
    participant PDF as pdf.js
    participant Z as JSZip
    participant DOM as Browser
    App->>P: new pdf2image({ file, type, scale, filename })
    App->>P: convert()
    P->>PDF: getDocument(Uint8Array)
    PDF-->>P: pdf
    P->>L: append to body
    par every page concurrently
        P->>PDF: getPage(i) → render(canvas)
        PDF-->>P: canvas.toDataURL(type)
        P->>L: percent(completed, total)
    end
    P-->>App: images (after 500ms)
    P->>L: remove()
    App->>P: download()
    P->>Z: file(name, base64) × N
    Z->>L: percent(compression progress)
    Z-->>P: Blob
    P->>DOM: a[download].click()
    P->>L: remove()
    P-->>App: resolve
```

## State Machine

```mermaid
stateDiagram-v2
    [*] --> Created: new pdf2image
    Created --> Converting: convert()
    Created --> Converting: download() with no results
    Created --> [*]: download() with empty file
    Converting --> Converted: all pages done
    Converting --> Failed: pdf.js error
    Converted --> Bundling: download()
    Bundling --> Converted: download triggered
    Bundling --> Failed: JSZip error
```

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
