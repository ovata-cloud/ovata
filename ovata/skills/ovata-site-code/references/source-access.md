# 在 Codex 完成原始碼存取

此流程適用有前台程式管理權的委託交付網站。樣板方案僅日常管理，不能以重新授權或 GitHub 邀請繞過方案。Git、工具與分支由 AI 處理；只有登入、OAuth 同意及 GitHub 接受邀請需要使用者本人確認。

## 先核對本次連線

1. `get_started(site_id)` 取得 `code_project_id`，再 `check_development(site_id)`。不要從「已連 ChatGPT Codex」推斷已連 Ovata 的 GitHub。
2. 缺 `projects:build` 或 `projects:deploy-staging` 時，呼叫 `request_development_access(project_id)`。它只用原有讀取授權開始，在工具結果回傳標準 `mcp/www_authenticate` 補授權挑戰，不會修改 grant 或製造憑證。
3. 引導使用者在正常 OAuth 提示按「允許連線」；同意頁沒有其他要選的。介面未自動開啟時，在 Codex 的該 MCP 連線使用 Authenticate／重新授權；若有本機終端，AI 可先查相符伺服器再執行正常 `codex mcp login`，不要叫店長手打指令，也不要複製固定 token。
4. 本人完成後重試原工具並重查 `check_development`。只說「點完了」不等於權限生效，要以工具讀回為準。`production=true` 只在使用者也要求正式發布時使用；正式發布仍需先確認測試站。

## 使用 GitHub 的路徑

`get_source_access(project_id)` 回傳正常 Console 入口、repo／工作分支、本人已連結的 GitHub 帳號、下一工具與完整協作流程。不要索取 GitHub 密碼或 token，也不要替使用者填一個任意 username 來開權限。

| 結果 status | AI 接著做什麼 |
| --- | --- |
| `github_link_required` | 開啟 `settings_url`（有 Codex 瀏覽器時可在該瀏覽器開啟），定位「GitHub 原始碼存取」的「連結 GitHub」，請本人完成正常 OAuth；保留此對話，完成後重試同一個 `get_source_access`。不要聲稱邀請已寄出。 |
| `invitation_required` | 向使用者展示 repo 與實際連結的 GitHub login。本人確認要開放後，呼叫 `sync_source_access(project_id, confirmed=true, expected_github_user_id)`；ID 用剛讀回的 `github.github_user_id`，不是使用者任意提供的 ID。 |
| `invitation_pending` | 開啟 `invitation_url`，由本人用回傳的 GitHub 帳號接受邀請，再重查 `get_source_access`。不盲目重送邀請。 |
| `ready` | 已確認 GitHub 寫入權限；由 AI clone repo 並切到工作分支，再依下方測試發布流程執行。 |
| `project_not_ready` | 明確列出專案／原始碼尚未就緒，保留同一網站與 repo，透過需求請 Ovata 檢查。 |

`SOURCE_IDENTITY_CHANGED`、`SOURCE_IDENTITY_CONFLICT` 或 `SOURCE_ACCESS_CHANGED` 時停止同步，重新讀本人連結，核對確認的帳號；不可改成邀請另一個帳號。結果不明先 `get_source_access`，確定仍需邀請才以同一專案及同一已確認身分重試。平台重用既有待接受邀請；有效寫入權限也不重送。

## 不用 GitHub 的路徑

使用者只要在 Codex 改網站程式，不一定需要個人 GitHub。完成 MCP 開發授權後，可直接 `list_files` → `read_file` → `write_files`；這會修改同一個 repo 的真正程式碼並建立 commit。不要把缺個人 GitHub 當作不能改程式，也不要擅自把要求改成只編輯 CMS 內容。

## 修改、審核、測試與上線

- 日常修改提交到 `get_project` 的工作分支。需要人工審核時，從獨立功能分支提出 PR 回工作分支；由使用者指定維護者審查，或將 PR 附在需求請 Ovata 協助。不要宣稱平台已替 repo 設好強制 PR 審核或分支保護。
- push／合併 PR 不會觸發平台部署。AI 呼叫 `prepare_version`，只在該 `build_id` 成功後 `prepare_candidate`、`deploy_test_tag`，提供測試網址驗收；或開發途中用 `build_preview`。
- 使用者明確確認測試結果、同意正式發布後，由有正式部署授權的 Codex／維護者 `deploy_release`，或店家於 Console「環境與版本」執行；用 `get_site` 查到該次 release succeeded，並檢查正式站。
- 正式發布確認後，把 main 快轉至相同提交作為版本紀錄。這一步不等於部署。保留前一版 tag 供回退。

不同 Codex 介面的瀏覽器／原生授權提示能力可能不同；能在同一段對話引導與續做，不代表可以略過使用者 OAuth 同意，或保證所有畫面都內嵌在對話中。
