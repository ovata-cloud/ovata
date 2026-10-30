# Ovata

讓 AI 開發工具幫 Ovata 網站的擁有者修改網站：管理商品、文章與頁面內容，修改網站程式，先放上測試站給擁有者看，確認後再發布正式站。不需要會 Git，也不需要 GitHub 帳號。

這個 plugin 包含：

- Ovata MCP 連線設定：`https://platform-api.ovata.tw/mcp`（Streamable HTTP，OAuth 登入）。
- 5 個 skills：
  - `ovata-workflow`：整體流程。先分清楚改內容還是改程式；改程式一律測試站 → 擁有者確認 → 正式站。
  - `ovata-site-code`：依訪客任務分頁，規劃首頁與導覽，再取得程式（Git 或不用 Git）、建置、版本、部署與退回。
  - `ovata-web-api`：網站程式怎麼正確呼叫後端（SDK、Web API 版本、金額與狀態規則）。
  - `ovata-site-content`：不改程式就能改的商品、文章、頁面內容與圖片。
  - `ovata-mail`：讀信、個人草稿與經使用者明確確認的寄信／回覆；需另授權個別信箱。

安裝方式見上一層的 [README](../README.md)。第一次呼叫工具時，瀏覽器會打開 Ovata 的同意頁；要讓 AI 發布正式站，在同意頁勾選「也允許部署到正式環境」。

連線不會因為登出 Ovata 而中斷，閒置 90 天才需要重新連線；在 Console「個人設定 › 開發者」可以查看或撤銷。只想要 MCP、不裝 skills 也可以：MCP 的 `get_started` 會把同樣的流程交給 AI。
