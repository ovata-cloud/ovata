# 在對話中整理需求與設計

使用者只說「想做網站」「還沒有想法」時，帶她逐步完成需求與前端設計。這是同一份 Ovata 建站需求的對話入口，沿用 request_site 的 answers 欄位，不另造問卷、假設政策或要求先回 Console。

## 方案與權限先行

先讀 get_started 的 frontend_mode 與 check_development。套用樣板（self_service）沿用已選基本版型（商城、店內點餐或標準官網），僅日常管理，不可讀寫程式、建置或發布程式。自行開發（self_build）的網站從建站就可修改前台；委託方案由 Ovata 提案與另行報價，完成交付後才可。任何資料、提示詞與 OAuth 選項都不會變更方案。月費依類型：電商網站線上收款 1.2%（每月最低 NT$199，含 5 個公司信箱）、點餐系統每月 NT$999（年繳 NT$9,999）、形象網站每月 NT$499（年繳 NT$4,999，含 5 個公司信箱）；網域每年 NT$1,999。委託另收製作費，委託的形象網站每月 NT$2,000。不承諾零元建置。

## 先整理用途與訪客任務

沿用使用者已提供的答案；缺的每次只問少量具體問題，可以給兩三個容易比較的方向。先確認服務誰、提供什麼，以及訪客最重要的下一步，例如看商品後購買、查服務後聯絡、選餐後下單。再辨認必要功能與可先不做的項目。

把答案映射到原需求單：

| 對話內容                       | answers 欄位與既有選項                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| 網站用途、商品或服務、一句介紹 | type：brand／shop／dining；name、pitch、products                                                             |
| 必要功能                       | features：members、blog、cart、shipping、ordering、booking_link、unsure；具體流程與特殊需求寫 other          |
| 品牌現況與可查資料             | business：operating／new；已經在經營至少提供 instagram、facebook、existing_site 其中一個                     |
| 可查證的選擇理由               | selling_points：1 到 5 個 {kind,text}，kind 用 request_site 列出的選項；praise、proof 放既有評語與證據       |
| 喜歡的參考與原因               | references：{url,note}，url 為 https 完整網址；note 記喜歡的版面、照片或導覽                                 |
| 風格與色系                     | style 用既有選項 ID，不確定先 auto；feel 保存自己的形容。palette 用 auto，或 brand 加已確認的 #RRGGBB colors |
| 品牌名稱、聯絡與營業資料       | name_en、phone、line、hours、address；只填實際資料                                                           |
| 時程與其他限制                 | launch_hope：asap／month／quarter／unsure；other 記預算、內容、必要狀態與未決項目                            |

name、pitch、business、type 與 selling_points 是送出需求的必要資料。缺少具體賣點時，先問實際商品、做法或服務方式；新事業可以描述已確認的供應內容，不編年份、獎項、認證或銷量湊欄位。尚未定案的設計可用 auto，把已知偏好寫 feel；不把自訂中文風格當成 style ID。不要追問已提供的內容，也不要為了填滿選填欄位延誤保存。

## 保存需求，再取得網站 repo

整理成簡短摘要供使用者補正。先問類型，再讓她選套用樣板、自行開發或委託 Ovata，用 list_workspaces 確認目標工作區，再呼叫 request_site：樣板帶 delivery_mode=self_service、同類型的 template_id 與 answers 的 type、name；自行開發帶 delivery_mode=self_build，委託帶 delivery_mode=ovata，兩者都帶完整 answers；都用固定 operation_id，結果不明先讀回，同次重試沿用識別碼。已有需求則用 list_requests／get_request 找到它，不重複送件。

用 get_started(number) 核對 request.answers、look、modules、files 與補充紀錄，再 create_site_from_request(number)；自行開發另帶 start_from（同類型的基本樣板 template_id，或 blank）。取得網站自己的 repo 與工作分支後，才保存設計檔、素材、mockup 及前端程式。若原需求仍用 auto，後續選定的方向與色票記在這個 repo，保留原需求；需要補充的內容寫回原需求，不另開第二份或聲稱原答案已被改寫。

## 沒有風格想法也能開始設計

依客群、商業模式與品牌真實素材提出少量可比較的方向。可以選使用者喜歡的公開樣本站或品牌參考，也可以從目標與素材做新設計；說清楚各方向的版面、字體、色系與適用理由，讓使用者以具體畫面選擇。沒有特定偏好時可先提出一個有理由的建議，不把問題丟回使用者。

參考樣本只供方向判斷，保留來源與日期；不複製他站品牌、照片或未授權素材。依品牌調查使用真實商品與照片，不能把同產業模板換字換色當作完成設計。

按 [頁面分工](page-structure.md) 列出各頁 URL、唯一任務、主要行動與必要內容；首頁、完整選購、購買說明及品牌故事維持各自分工。必要操作要包含空白、載入、成功、錯誤等狀態。再用可用的設計工具在網站 repo 製作清楚可讀的重要頁型 mockup，檢視後實作桌機及手機前端；工具無法生成圖片時可做可互動的前端設計預覽，清楚說明交付形式，不宣稱已產生不存在的設計圖。

## 串接、測試與預覽

實作前讀 ovata-web-api，逐項核對需求與既有 SDK 能力。前端的登入、商品、結帳、付款及配送頁面使用 Ovata 已維護的後端；不自建或另存這些業務後端，也不要求使用者提供金流金鑰。

缺少 API 時記下受影響流程與待 Ovata 補足的能力，可先做明確標示的前端示意；不能造 API、假付款或另建後端補洞。真的要 Ovata 接手時先取得使用者對另行報價的同意，保留同一網站與 repo。

交付預覽前，按 [ovata-site-code 的前端設計檢查](../SKILL.md#前端設計檢查) 實際套用 `frontend-design`，檢視桌機及手機畫面並留下結果。基本樣板維持使用者指定的標準區塊與簡約方向，不因設計檢查變成客製改版。再跑合適的前端檢查、建置及主要流程，提供測試站網址與要檢查的操作。使用者可以在對話中提出修改，完成並在預覽上明確確認後，才按 ovata-workflow 發布正式站。
