# 進度遮罩

本頁說明 `convert()` 與 `download()` 期間自動出現的進度遮罩：DOM 結構、進度計算、語系與樣式。

## DOM 結構

遮罩是附加在 `<body>` 末端的單一元素，完成後自動移除：

```html
<div class="pdf2image-loading" data-percent="66">
  <p>Processing 66%</p>
</div>
```

| 屬性 | 內容 |
|---|---|
| `class` | 固定為 `pdf2image-loading` |
| `data-percent` | 0–100 的整數，樣式依此值決定進度條寬度 |
| `<p>` 文字 | 階段名稱＋百分比 |

## 進度來源

| 階段 | 計算方式 | 文字（英文／中文） |
|---|---|---|
| 轉換 | `round(已完成頁數 / 總頁數 × 100)` | `Processing N%`／`解析中 N%` |
| 壓縮 | JSZip `generateAsync` 回報的 `percent`，四捨五入 | `Preparing zip N%`／`準備壓縮檔 N%` |

轉換遮罩在 PDF 解析成功後才出現，解析失敗不會顯示。

```demo
<script>
  new MutationObserver(records => {
    for (const r of records) {
      const el = r.target.closest?.(".pdf2image-loading") || [...r.addedNodes].find(n => n.classList?.contains("pdf2image-loading"));
      if (el && r.type === "attributes") console.log("data-percent", el.dataset.percent, "|", el.innerText);
      for (const n of r.addedNodes) if (n.classList?.contains("pdf2image-loading")) console.log("overlay added");
      for (const n of r.removedNodes) if (n.classList?.contains("pdf2image-loading")) console.log("overlay removed");
    }
  }).observe(document.body, { childList: true, subtree: true, attributes: true, attributeFilter: ["data-percent"] });
  addEventListener("load", async () => {
    const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
    await new pdf2image({ file: buffer, type: "png", scale: 0.5 }).convert();
  });
</script>
```

## 語系

套件載入時讀取一次 `navigator.language`（或 `navigator.userLanguage`），開頭為 `zh`（不分大小寫）即使用中文文字，其餘一律英文。載入後變更語系不會影響。

## 樣式

樣式表在套件載入時從 `cdn.jsdelivr.net/npm/@pardnchiu/pdf2image@latest/dist/pdf2image.css` 注入：

- 遮罩為 240×16px 的圓角條，`position: absolute` 置於 `top: calc(50% - 12px)`、`left: calc(50% - 120px)`
- 進度條是 `::after` 偽元素，寬度由 `[data-percent="N"]` 選擇器對應 `N%`
- `position: absolute` 以文件而非可視範圍為基準（祖先元素未設定定位時），頁面捲動後遮罩停在文件第一屏的中央；要固定在可視範圍內，可在自己的樣式表覆寫 `div.pdf2image-loading`（例如 `position: fixed`）

## 相關頁面

- [轉換流程](/zh/conversion)
- [ZIP 下載](/zh/zip-download)
