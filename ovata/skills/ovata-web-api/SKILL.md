---
name: ovata-web-api
description: Ovata 網站程式呼叫後端（商品、購物車、結帳、付款、配送、會員、文章、頁面內容、投稿、餐廳）的正確做法：只透過 SDK、寫程式前先讀 get_web_api_docs、金額與狀態的規則、版本與升級。寫或修改任何讀寫網站資料的程式前讀這份。
---

# 網站程式怎麼呼叫後端

網站的後端由 Ovata 提供，原始碼不在網站的 repo 裡，你看不到它。後端怎麼運作，一律以 MCP 給的文件為準，不要猜。

## 寫程式前

1. `get_web_api_docs(site_id, environment)`，回傳：
   - `backend_api_version`：這個環境上線中的網站用哪一版 Web API；照這一版的文件寫。
   - `endpoints`：每支 API 的路徑、對應的 SDK 呼叫（`sdk`）、參數、回應與注意事項。
   - `errors`：程式會收到的錯誤碼與該怎麼處理。
   - `upgrade`：升到最新版要改什麼。
2. 照文件寫。文件沒列的 API 就是不存在，不要自己編路徑，也不要直接打 Console 或後台的 API。
3. 型別：用 npm 套件 `@merchant-kit/sdk` 的網站，`npm ci` 之後 `node_modules/@merchant-kit/sdk/dist/*.d.ts` 和 README 有完整型別；模板網站的 `vendor/sdk.js` 是打包檔，對照 `get_web_api_docs` 的 `sdk` 欄位使用。

## 一定要守的規則

- 瀏覽器只用同源的 SDK client，例如 `createCommerceClient()`、`createCustomerClient()`、`createPagesClient()`。
- 網站伺服器把 `/api/`、`/robots.txt`、`/sitemap.xml` 交給 SDK 的 `createStorefrontProxy().fetch(request)`；不要自己寫死 robots.txt 或網站地圖。
- `APP_LUXIRA_API_URL`、`APP_LUXIRA_SERVER_KEY` 等 `APP_*` 設定由平台提供，只在伺服器用；不能進瀏覽器，也不能寫進 repo。
- `vendor/` 裡的 SDK 檔案（包括 `vendor/sdk/*.tgz`）不要改寫、排版或重新壓縮。
- 金額是 `currency` 那個幣別的整數元（新台幣、港幣都一樣），每個有金額的物件都帶 `currency`。用它顯示幣別，不要假設是新台幣，也不要除以 100。
- 時間是 ISO-8601 UTC 字串，顯示時轉成台灣時間。
- 付款狀態、配送狀態、配送方式、付款方式都是開放列舉：遇到不認得的值顯示通用文字並重新讀訂單，頁面不能壞。
- 錯誤回 `{ error, code }`。`error` 是給顧客看的中文、會改寫；程式只看 `code`，不認得的當一般錯誤。
- 價格、優惠、運費、免運資格、庫存都以後端回傳為準，前端不能自己算或改價格。
- 回應多出不認得的欄位時忽略它。

## 常見流程

- 商品：`products.list(filter)`、`products.get(id)`。`price` 是現在結帳的價格；`price < regularPrice` 才顯示劃線原價。
- 購物袋：`cart.get()` 已經算好優惠、加價購與免運進度；`cart.update(revision, action)` 遇到 `REVISION_CONFLICT` 就重讀再送。
- 結帳：
  1. 要運送的購物袋先用 `createShippingClient().options()` 與 `quote(input)` 取得 `quoteId`。
  2. `checkout.confirm({ idempotencyKey, snapshotHash, shipping: { quoteId } })` 建立訂單，回傳要怎麼付款（`kind`：`form`、`redirect`、`manual`，不認得的就帶顧客到訂單頁）。
  3. 付款完回到 `/checkout?order=<id>`，網站一定要有這個頁面：先 `orders.syncPayment(id)`，再 `orders.get(id)` 顯示結果。不要只看網址參數就說付款成功。
- 會員：`createCustomerClient()` 的 `me()`、`password.login()`、`orders()`、`updateMe()`。
- 頁面內容：`pages.get(pageKey)` 讀商家在 Console 改的字和圖；哪些欄位可以換，由網站程式用 `pages.sync` 登錄（格式見 `get_backend_api_docs` 的 `pages`）。
- 自訂欄位：`GET /api/v1/custom-fields?resource=…` 取得網站能用的欄位來畫表單。欄位定義在測試站改，發布時會帶到正式站。
- 文章：`createContentClient().list(filter)`、`get(slug)`。

## 版本與升級

- 網站打包的 SDK 會告訴後端它用哪一版 Web API；後端同時支援每一版，所以舊網站照舊能用，不用急著升級。
- 想用新版才有的功能：
  1. `get_web_api_sdk(site_id, environment)` 取得最新 SDK。模板網站拿 `vendor/sdk.js`、`vendor/proxy.js`；用 npm 套件的網站加 `format: "package"`，拿 `@merchant-kit/sdk` 的 tgz 與 `sdk.lock.json`。
  2. 檔案原樣寫進 repo（tgz 用 base64 解回位元組），照 `get_web_api_docs` 的 `upgrade.changes` 調整程式。
  3. 放上測試站，走一次會用到的流程（商品、購物袋、結帳、配送），確認後再發布。
- 後端功能版本（資料庫）不會自己升級。`get_merchant_backend` 看目前版本；要升級時先對測試環境用 `upgrade_merchant_backend`，在測試站確認後再升級正式環境（MCP 會擋下沒先升級測試環境的正式升級）。
