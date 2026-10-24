# Console 草稿預覽

Console「頁面內容」用 iframe 打開網站，網址帶一小時有效的 `lx-preview` token，讓商家先看草稿再發布。這是讀後台內容草稿，和建置程式的 `build_preview` 是兩回事；只有正式站的網站不必為了預覽另外準備測試站。

## 取得與讀取草稿

- 網站擁有者先讀 `get_backend_api_docs(site_id, environment, resource="pages")`，再用 `site_backend(site_id, environment, method="POST", path="/api/v1/pages/admin/preview", input={})` 取得 `token` 與 `expiresAt`。token 不要寫進 repo 或公開的回報。
- 打開網站實際網址加上 `?lx-preview=<token>`。伺服器產生頁面時用 `previewOptions(request.url)`，瀏覽器用 `previewOptions()`，把結果傳給 `pages.get(pageKey, options)`。帶 token 的回應是草稿，不要快取。
- 只有後端回傳 `page.preview === true` 才代表讀到有效草稿；網址參數本身不代表什麼。
- 用 `page.content`、`page.shared`、`page.images`、`page.siteImages` 畫頁面。頁面欄位寫成 `區塊.欄位`，全站共用欄位寫成 `*:區塊.欄位`，標在元素的 `data-lx-field` 上，瀏覽器呼叫 `enablePreview()`。
- 伺服器產生的網站：站內連結（包括動態產生的商品連結）要保留 `lx-preview` 與 `lx-version`，外部連結和下載不要帶。

## 讓 Console 能嵌入

網站的 HTML 回應要允許 Console 嵌入草稿頁：

```http
Content-Security-Policy: …; frame-ancestors https://console.ovata.tw https://console.luxshop.tw https://console.luxira.net; …
Cache-Control: no-store
Referrer-Policy: no-referrer
```

- `frame-ancestors` 要放在 HTTP 回應標頭，不能用 meta。草稿回應不要帶衝突的 `X-Frame-Options: DENY` 或 `SAMEORIGIN`。一般公開頁面可以維持 `frame-ancestors 'none'`，不要允許任意來源。
- SDK 會插入一段 style 畫選取框。`style-src 'self'` 的網站，每個 HTML 回應產生新的 nonce，加進 `style-src 'nonce-<nonce>'`，再用 `enablePreview({ styleNonce: nonce })` 傳給 SDK。
- 文字標記要沿用原本的字型樣式；`display: contents` 的元素畫不出選取框。
- 響應式圖片用 `<picture>`：手機版放 `<source media>`，電腦版放 `<img data-lx-field="…">`。

## 檢查

在 Console 裡實際打開，檢查電腦與手機的草稿文字、點選欄位、即時更新、換圖與版本紀錄。iframe 灰掉通常是嵌入政策或導覽失敗；網站有顯示但點不到欄位，多半是 SDK 沒初始化、預覽 token 沒傳下去，或來源檢查不符。token 過期就重新取得。
