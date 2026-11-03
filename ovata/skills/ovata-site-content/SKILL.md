---
name: ovata-site-content
description: 不改程式就能改的 Ovata 網站內容：商品、分類、優惠、文章、頁面上的字和圖、網站設定、自訂欄位、投稿、顧客匯入、訂單顧客訊息、餐廳設定與媒體上傳，用 get_backend_api_docs 與 site_backend 操作。要讀或改網站後台資料、匯入商品或上傳網站素材時讀這份。
---

# 不改程式就能改的內容

這些都不用部署：存好就生效（頁面內容是存成草稿，發布後才上網站）。網站擁有者也可以自己在 Console 改，你可以先問她要不要自己來。

## 怎麼操作

1. `get_backend_api_docs(site_id, environment, resource?)`：先讀格式、流程與範例。`resource` 可以是 `products`、`categories`、`content`、`pages`、`site`、`media`、`customFields`、`promotions`、`restaurant`、`submissions`、`customers`、`orders`、`orderMessages`、`customerImports`。
2. `site_backend(site_id, environment, path, method, input?)`：用同一個網站與環境呼叫。`method` 是 `GET`、`POST`、`PUT`、`PATCH`，沒有 DELETE。
3. 每次都要指定 `environment`：`production` 是正式站，`staging` 是測試站。兩邊的資料各一份，不會同步。樣板網站還沒上線時在 `staging` 準備，第一次上線會帶到正式站（訂單、會員不帶）。
4. 先 `GET` 讀現在的資料、ID 與 `revision`，再寫入；保留原本的資料，只改要改的欄位。

只有工作區管理員能用 `site_backend`。文章和留言另有專用工具（`list_posts`、`create_post`、`publish_post`、`list_comments`、`approve_comment`…），寫入要 `request_id`，更新與發布要目前的 `revision`。

## 結果不明時

回傳是 `{ status, data }`。`status: "outcome_unknown"` 表示寫入可能已經成功：先讀回資料核對，再決定要不要重送，不要直接重送。

## 圖片

- 上傳：`site_backend(site_id, environment, path="/api/admin/media", method="POST", media={ content_type, data_base64, filename })`。只收 JPEG、PNG、WebP，最大 8 MiB。
- `filename` 只放檔名（不含資料夾），商家的媒體庫會顯示它。
- 回傳的 `data.src` 用在商品圖片、文章封面、分享圖片；`data.key` 用在頁面的圖片位置（pages 的 `setImage`）。
- 先上傳圖片，再寫入用到圖片的資料。不要把 base64 印在對話裡。

## 網站靜態素材

- 站方管理的影片、音檔、PDF、字型與圖片直接放媒體庫，供網站重複引用；不要建立投稿、商品或文章來存這些檔案。訪客投稿仍使用投稿模組。
- 先 GET `/api/admin/media/capabilities`。`resumable=false` 時需先明確升級這個網站的 `media.library` 後端功能，不能用投稿系統繞過。
- MCP 用同一個 `site_backend` 的 `media`：帶 `content_type`、`filename`、UUID `operation_id`、`bytes`、完整小寫 `sha256`、`offset`、`data_base64`。每段 5 MiB 原始 bytes（最後可較短），依 `next_offset` 續傳；`complete=true` 才拿到可引用的 `key` 與 `src`。結果不明時沿用相同 metadata，空 `data_base64` 查進度，不換操作識別碼盲目重傳。上傳到期後才換新識別碼。
- 圖片最大 8 MiB、MP4/WebM/MOV 影片 512 MiB、MP3/M4A/WAV/OGG 音檔 128 MiB、PDF 32 MiB、WOFF/WOFF2 字型 8 MiB，網站媒體庫總額 5 GiB。格式由實際內容驗證，不收 HTML、SVG 或程式。不要在對話印出 base64。
- `data.src` 可直接放在網站的 `<video>`、`<audio>`、圖片、字型或下載連結；它屬於指定環境，測試與正式需各自上傳。素材上傳不用重新部署；改網站引用仍循測試站預覽流程。
- 網站程式引用以 PUT `/api/admin/media/references` 的 `{ownerId,label,keys}` 登錄，一次替換該元件的全部引用。使用中的素材受刪除保護，後台會顯示元件名稱；元件移除素材時同步更新引用。
- 管理端瀏覽器用 `@merchant-kit/sdk/media` 的 `createMediaClient` 與 `uploadMediaFile`，不要把管理權限放到公開前台。

## 大量匯入

商品、分類、庫存、促銷與訂單已有後端能力：先讀對應 resource 文件，用 `site_backend` 查現況並在授權範圍內修改。自建前台用 ovata-web-api 的 SDK 串接，不另做一套商品或訂單後端。查銷售先讀訂單，網路流量與 Google 搜尋則用 `get_site(site_id).google.insights`；其中有 GA4 與 Search Console 近 28 天數字、各部分更新時間與連線狀態。沒有連線或資料時照實說明，請使用者到 Console「搜尋與分析」處理，不虛構數字，也不要求貼 Google token。

商品匯入仍逐筆整理成 JSON，失敗先讀回確認，不重複新增。

顧客用 `get_backend_api_docs(resource="customerImports")` 的分批流程：在客戶端解析原始 XLS／XLSX／CSV，保留所有原始欄位與地址續列，完整上傳後預覽，再依店主確認的範圍用同一 previewToken 分批 commit 至完成。重複 Email／來源與既有顧客預設略過，不覆寫或合併；中斷可恢復，結果有逐列理由。原購物金僅保留來源紀錄，不用於結帳，不製造歷史訂單、不變更訂閱，也不寄邀請或行銷信。

## 訂單內聯絡顧客

先讀 `orderMessages` 文件，再 GET 訂單 messages 核對收件人與客服 Reply-To。店主已授權這次寄送時，POST 純文字訊息並使用固定 operationId；寄送逾時先 GET 原識別碼，不換 ID 盲目重寄。回信送往客服信箱，不同步回訂單。

以上需要商店先明確升級後端功能。Developer 另需 orders:read／contact、customers:read／write／import 同意範圍；舊連線收到 insufficient_scope 時走正常 OAuth 重新同意。

## 頁面上的字和圖

- `GET /api/v1/pages/admin` 列出頁面與可換的欄位；改字、換圖都先存成草稿，要 `publish` 才會上網站。
- 可以換哪些字和圖，是網站程式登錄的；想新增可換的欄位或改版面，就要改網站程式（見 ovata-site-code）。

## 自訂欄位與投稿區

- 自訂欄位（`/api/business/custom-fields`）的 API 名稱和類型建立後不能改，也沒有刪除，只能封存。
- 自訂欄位的定義和投稿區設定，在測試站改好，正式發布時會帶到正式站；只在正式站改的會保留。
- 投稿圖片只能讀縮圖；永久刪除投稿只能在 Console。

## MCP 做不到的（請她在 Console 操作）

- 金流與物流的金鑰、開關結帳（Console「結帳設定」）。
- 訂單出貨、退款、確認收款。
- 刪除資料、永久刪除投稿。

工具回 `PATH_NOT_ALLOWED` 就是 MCP 不開放這個操作，請她到 Console；不要嘗試別的路徑繞過。

## 投稿的社群貼文

公開原文連結 `links` 接受 IG、Threads、TikTok 的完整影片或照片貼文網址。前台先 `session(collectionId)`，再用 SDK `resolveLink(collectionId,url)` 展開 TikTok 官方分享短網址；失敗時請貼完整公開網址。只回傳標準網址，不回傳第三方 HTML 或下載原片。直接顯示原貼文需使用平台官方嵌入；私密、刪除或禁止嵌入的貼文無法保證顯示，影片也可能需要點擊播放。訪客媒體仍是投稿，不放站方媒體庫。
