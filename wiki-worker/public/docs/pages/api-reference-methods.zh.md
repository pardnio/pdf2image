# 方法與屬性

本頁列出 `pdf2image` 實例的公開方法與屬性，以及各自的 resolve／reject 值。

## 一覽

| 成員 | 簽章 | resolve | reject |
|---|---|---|---|
| `convert` | `convert(): Promise<string[]>` | Data URL 陣列（與 `images` 為同一個陣列） | 字串或 `Error`，見 [轉換流程](/zh/conversion#失敗行為) |
| `download` | `download(): Promise<void>` | `undefined`（下載已觸發，或 `file` 為空而略過） | `convert()` 的 reject 值，或 JSZip 失敗時的 `[]` |
| `images` | `get images(): string[]` | — | — |

## `convert()`

併發渲染所有頁面並回傳 Data URL 陣列，期間顯示進度遮罩。每個實例只能成功呼叫一次，因為 `file` 會在第一次轉換時被轉移。細節見 [轉換流程](/zh/conversion)。

## `download()`

把結果打包成 `{filename}.zip` 並觸發瀏覽器下載；尚未轉換時會先呼叫 `convert()`。細節見 [ZIP 下載](/zh/zip-download)。

## `images`

唯讀 getter，回傳內部結果陣列本身（不是複本）。轉換前為空陣列；`convert()` resolve 的陣列與它是同一個參考，修改其中一個會反映到另一個，也會影響之後 `download()` 打包的內容。

```demo
<script>
  addEventListener("load", async () => {
    const buffer = await (await fetch("/assets/sample.pdf")).arrayBuffer();
    const converter = new pdf2image({ file: buffer, type: "png", scale: 0.5 });
    console.log("before:", converter.images.length);
    const result = await converter.convert();
    console.log("after:", converter.images.length, "same array:", result === converter.images);
    result.pop();
    console.log("after pop:", converter.images.length);
  });
</script>
```

## 未公開的成員

`#filename`、`#file`、`#type`、`#scale`、`#result`、`#formatDate()` 是 ES 私有欄位，外部無法存取。進度遮罩類別 `pdf2imageLoading` 只在模組內使用，不掛到 `window`。

## 相關頁面

- [建構參數](/zh/api-reference)
