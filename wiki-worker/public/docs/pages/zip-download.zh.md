# ZIP 下載

本頁說明 `download()` 如何把轉換結果打包成 ZIP 並觸發瀏覽器下載，以及檔名樣板的規則。

## 流程

1. `file` 為 `null`／`undefined` 時直接 resolve `undefined`，不轉換也不下載
2. 尚無轉換結果時先 `await this.convert()`；已有結果則直接沿用，不重新渲染
3. 在 `<body>` 附加進度遮罩，以 JSZip 逐張加入 base64 圖片
4. `generateAsync({ type: "blob", streamFiles: true })` 產生 ZIP，壓縮進度同步到遮罩
5. 建立 `<a download>` 指向 `URL.createObjectURL(blob)`，延遲 500ms 後 resolve、點擊、移除連結與遮罩

```demo
<button id="save">Download sample.zip</button>
<script>
  document.getElementById("save").addEventListener("click", async e => {
    e.target.disabled = true;
    const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
    const converter = new pdf2image({ file: buffer, filename: "sample.pdf yyyy-MM-DD", type: "png", scale: 0.5 });
    try {
      await converter.download();
      console.log("downloaded", converter.images.length, "images");
    } catch (err) {
      console.log("failed:", String(err));
    }
    e.target.disabled = false;
  });
  new pdf2image({}).download().then(result => console.log("no file ->", String(result)));
</script>
```

## 檔名樣板

`filename` 在建構時先 `trim()`，並移除**第一個** `.pdf`（區分大小寫，不限結尾位置）。下載當下再替換下列日期記號（本地時間）：

| 記號 | 意義 | 範例 |
|---|---|---|
| `yyyy` | 四位數年份 | `2026` |
| `MM` | 月，補零 | `10` |
| `DD` | 日，補零 | `04` |
| `hh` | 時（24 小時制），補零 | `13` |
| `mm` | 分，補零 | `05` |

## 產出檔名

| 項目 | 規則 | `filename: "report.pdf yyyy-MM-DD"`、`type: "png"` 時 |
|---|---|---|
| ZIP | `{名稱}.zip` | `report 2026-10-04.zip` |
| 圖片 | `{名稱} {索引}.{type}`，索引從 0 開始 | `report 2026-10-04 0.png`、`report 2026-10-04 1.png` |

注意事項：

- 圖片副檔名直接取 `type`；`type: "jpg"` 時副檔名是 `.jpg`，但內容是 PNG（見 [轉換流程](/zh/conversion)）
- 未提供 `filename` 時 ZIP 名稱為 `.zip`，圖片為 ` 0.jpg` 這類以空白開頭的名稱
- 圖片索引依結果陣列順序，不保證等於頁碼順序

## 失敗行為

| 情境 | 結果 |
|---|---|
| 自動轉換失敗 | 以 `convert()` 的 reject 值 reject，不顯示壓縮遮罩 |
| JSZip 產生失敗 | 以空陣列 `[]` reject，並在 console 印出原始錯誤；壓縮遮罩不會被移除，會留在頁面上 |

## 相關頁面

- [進度遮罩](/zh/progress-overlay)
- [方法與屬性](/zh/api-reference-methods)
