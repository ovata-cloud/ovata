# Ovata AI 工具

讓 AI 開發工具（Codex、Claude）幫你修改 Ovata 網站：管理商品、文章與頁面內容，修改網站程式，先放上測試站給你看，你確認後再上線。

## 需要什麼

- Ovata 帳號，而且是網站所在工作區的管理員。要改網站程式，還要是網站程式的擁有者。
- 一個支援 MCP 的 AI 工具：Codex（ChatGPT 桌面版、Codex CLI）、Claude Code、Claude 桌面版或 claude.ai。
- 不需要會 Git，也不需要 GitHub 帳號。

## 安裝

### Codex

```sh
codex plugin marketplace add luxira-cloud/ovata
```

然後在 Codex 輸入 `/plugins`，選 **Ovata** 安裝，再開一個新的對話。ChatGPT 桌面版的 Codex 和 Codex CLI 共用設定，裝好後兩邊都能用。

### Claude Code

```text
/plugin marketplace add luxira-cloud/ovata
/plugin install ovata@ovata
```

裝好後輸入 `/mcp`，選 **ovata** 登入。

### 只要 MCP 連線

MCP 網址是 `https://platform-api.ovata.tw/mcp`。沒裝 skills 也可以用：MCP 的 `get_started` 會把同樣的流程交給 AI。

- Codex：在 `~/.codex/config.toml` 加上下面兩行，再執行 `codex mcp login ovata`。

  ```toml
  [mcp_servers.ovata]
  url = "https://platform-api.ovata.tw/mcp"
  ```

- Claude Code：`claude mcp add --transport http ovata https://platform-api.ovata.tw/mcp`
- Claude 桌面版、claude.ai：設定 › Connectors › 新增自訂 connector，貼上網址。

plugin 和單獨的 MCP 連線擇一即可，兩個都設定會出現重複的工具。

## 第一次使用

1. 對 AI 說「列出我的 Ovata 網站」。
2. 瀏覽器會打開 Ovata 的同意頁，確認是自己的帳號，按「允許連線」。要讓 AI 發布正式站，勾選「也允許部署到正式環境」。
3. 告訴 AI 你想改什麼。改網站程式時，AI 會先放上測試站給你看，你說可以再上線。

連線不會因為登出 Ovata 而中斷，閒置 90 天才需要重新連線。在 Ovata Console「個人設定 › 開發者」可以查看或撤銷連線。

## 內容

- [`ovata/`](ovata)：plugin 本體，包含 MCP 連線設定和 4 個 skills。
- `.agents/plugins/marketplace.json`（Codex）與 `.claude-plugin/marketplace.json`（Claude）：讓這個 repo 可以當成 plugin marketplace 加入。
