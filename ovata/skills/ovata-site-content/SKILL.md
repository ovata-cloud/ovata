---
name: ovata-site-content
description: 不改程式就能改的 Ovata 網站內容：商品、分類、優惠、文章、頁面上的字和圖、網站設定、自訂欄位、投稿、餐廳設定與圖片上傳，用 get_backend_api_docs 與 site_backend 操作。要讀或改網站後台資料、匯入商品或上傳圖片時讀這份。
---

# 不改程式就能改的內容

這些都不用部署：存好就生效（頁面內容是存成草稿，發布後才上網站）。網站擁有者也可以自己在 Console 改，你可以先問她要不要自己來。

## 怎麼操作

1. `get_backend_api_docs(site_id, environment, resource?)`：先讀格式、流程與範例。`resource` 可以是 `products`、`categories`、`content`、`pages`、`site`、`media`、`customFields`、`promotions`、`restaurant`、`submissions`。
2. `site_backend(site_id, environment, path, method, input?)`：用同一個網站與環境呼叫。`method` 是 `GET`、`POST`、`PUT`、`PATCH`，沒有 DELETE。
3. 每次都要指定 `environment`：`production` 是正式站，`staging` 是測試站。兩邊的資料各一份，不會同步。
4. 先 `GET` 讀現在的資料、ID 與 `revision`，再寫入；保留原本的資料，只改要改的欄位。

只有工作區管理員能用 `site_backend`。文章和留言另有專用工具（`list_posts`、`create_post`、`publish_post`、`list_comments`、`approve_comment`…），寫入要 `request_id`，更新與發布要目前的 `revision`。

## 結果不明時

回傳是 `{ status, data }`。`status: "outcome_unknown"` 表示寫入可能已經成功：先讀回資料核對，再決定要不要重送，不要直接重送。

## 圖片

- 上傳：`site_backend(site_id, environment, path="/api/admin/media", method="POST", media={ content_type, data_base64, filename })`。只收 JPEG、PNG、WebP，最大 8 MiB。
- `filename` 只放檔名（不含資料夾），商家的媒體庫會顯示它。
- 回傳的 `data.src` 用在商品圖片、文章封面、分享圖片；`data.key` 用在頁面的圖片位置（pages 的 `setImage`）。
- 先上傳圖片，再寫入用到圖片的資料。不要把 base64 印在對話裡。

## 大量匯入

沒有批次工具：把網站擁有者給的檔案整理成 JSON，一筆一筆送；中途失敗就從失敗那筆繼續，先讀回確認沒有重複。

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
- 網站的會員與顧客個資。

工具回 `PATH_NOT_ALLOWED` 就是 MCP 不開放這個操作，請她到 Console；不要嘗試別的路徑繞過。
