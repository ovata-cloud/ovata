---
name: ovata-web-api
description: 將網站前端透過 SDK 接上 Ovata 維護的商品、訂單、金流、物流、會員與內容後端；先查 get_web_api_docs、核對能力與版本，缺 API 時列出或委託 Ovata，避免另建業務後端。寫或修改任何讀寫網站資料的前端程式前讀這份。
---

# 網站程式怎麼呼叫後端

網站的後端與資安由 Ovata 提供並維護，原始碼不在網站的 repo 裡。你負責前端頁面、版面、互動與 SDK 串接，後端怎麼運作一律以 MCP 文件為準。

不要自行實作或另存會員帳號與驗證、訂單、付款、退款、物流或後端權限資料；不能以網站自己的 DB、檔案、瀏覽器持久儲存或第三方服務另做一套業務後端。前端可保留表單輸入及顯示所需的暫存狀態，業務紀錄與成功狀態以 Ovata 後端為準。也不自行接付款商或要求金流金鑰；使用者只在 Console 既有的金流物流設定輸入憑證。

## 先核對功能是否已有支援

把需求逐項對到文件列出的 SDK 呼叫與資料：可直接串接、需要使用者設定、目前缺少 API。商品、會員、訂單、付款、配送與內容能力能用就用，不重做它們。後台資料整理先讀 ovata-site-content；前端讀寫才用這份 SDK 流程。

現有版本沒有需要的能力時，先查文件的版本與升級說明；只有文件明列支援時才升級。仍沒有就列出缺少的輸入／回應、受影響頁面與目前能完成的前端範圍，讓使用者決定暫留或委託 Ovata。自建需求用已同意的 handoff_site_request，既有網站用 request_change；不因查到缺項就自動發包或同意報價。尚未串接的預覽可用清楚標示的示意資料，不製造假 API、假登入、假訂單或付款成功，不能上線成可交易流程。

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
- 會員寄售：`createConsignmentClient()` 查改登入會員自己的寄售、上傳商品照片／私人照片、送審與撤回；先明確升級 `commerce.consignments`。名稱、說明、售價之外要問的（品牌、尺寸、來源、撥款方式…）是商店的商品自訂欄位，勾「寄售會員可填」，表單用 `fields()` 畫、值放 `customFields`；不公開的欄位只有會員本人與後台看得到。會員不可設定審核結果或上架狀態，管理員在 Console「寄售審核」要求補件或核准並上架；審核中與核准後商店也能改資料與售價、補店家照片、排商品圖順序、重新上架，記在寄售的 `events`（`edit` 帶 `changes`），網站在會員的寄售詳情列出。商品清單沿用 `products.list`（單件，`variants[0].stock` 為 0 就是已售出或交易中）。會員自己的寄售有 `sale`：`available` 可售、`pending` 交易中、`sold` 已售出（`soldAt`、`shipTo` 是要寄到的店家地址，沒填為 null）；寄件期限等規則由網站自己寫。審核結果與售出依商店「Email 通知」寄信給會員。分潤與購物金結算不由寄售 API 執行。
- 會員：`createCustomerClient()` 的 `me()`、`password.login()`、`orders()`、`updateMe()`。
- 密碼註冊與重設：`password.register(email)`、`password.forgot(email)` 會寄信，連結固定是網站的 `/account/verify#token=…&purpose=…`，開放密碼登入的網站一定要有這個頁面。開頁後從 fragment 讀出 `token` 與 `purpose`（`vendor/sdk.js` 的 `accountProof(location.hash)`），立刻用 `history.replaceState` 把 fragment 從網址拿掉，不存進瀏覽器，也不送分析工具；再顯示設定密碼的表單（`register` 另要填姓名），送 `password.confirm({ token, purpose, password, name })`，成功後請顧客重新登入。
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
