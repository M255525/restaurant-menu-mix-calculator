# CLAUDE.md

本檔案為 Claude Code 在此子資料夾工作時的指引。此資料夾**本身是獨立 git 儲存庫**，不受根目錄工作區規則約束（除語言等全域偏好）。

## 這是什麼

**餐飲菜單組合收支平衡計算機**，單檔前端、無後端、無序號授權。2026-10-05 依使用者要求「參考亞馬遜商品組合收支平衡計算機與餐飲店評量試算工具，做一份針對餐飲行業的商品組合，以台幣計算」建置。

- 骨架沿用 `資料儀表板/amazon-listing-mix-calculator`（分類＋品項兩張動態表、單一 `calculate()` 計算源、`#verifyBox` 獨立驗算、CSV/Excel 匯入匯出、BYOK AI 診斷＋規則式 fallback、PDF 靜態報告＋浮水印、跑馬燈、PWA）。
- 餐飲業概念取自 `資料儀表板/restaurant-feasibility-calculator`（固定支出拆店租/人事/水電/其他、健康帶：食材 28–34%、人事 ≤32%、店租 ≤15%）。

### 跟 Amazon 版的對應關係

| Amazon 版 | 本工具 |
|---|---|
| 分類 Amazon 抽成% | 分類**目標食材成本率%**（超標標紅、列入資料提醒） |
| 建議售價 USD/JPY＋匯率 | **全部台幣**，售價為**含稅價**，以營業稅率換算未稅 |
| 毛利＝售價×銷量×(1−抽成) | **單份貢獻毛利**＝售價÷(1+稅)×(1−通路費率)−食材成本；通路費率＝外送占比×外送抽成＋(1−外送占比)×金流手續費 |
| 週/月獲利門檻（手填） | 每月固定支出四項加總；週固定＝月×7÷30 |
| 淨利係數 | 無；淨利＝貢獻毛利−固定支出 |
| 庫存/補貨前置期 | 備料庫存（份）/叫貨前置天數（同一套緊急 <前置、注意 <前置×1.5） |
| 上架總金額上限 | 備料庫存資金（份數×食材成本）上限 |
| 主力/輔助/附屬/聯想/刺激角色 | **菜單工程矩陣**（Kasavana & Smith）：人氣＝銷量占比 ≥ (1/N)×70%；獲利力＝單份貢獻毛利 ≥ 加權平均；明星/耕馬/謎題/瘦狗，SVG 散佈圖 |

另新增：損益兩平每日營收與每日需賣份數（假設銷售組合不變）、安全邊際、主要成本率（食材＋人事）、「每 100 元營收去向」橫條（營業稅＋通路費＋食材＋四項固定支出＋淨利＝營收，驗算有檢查此恆等式）。淨利率 ≥10% good、0–10% warn、<0 bad。

**刻意沒做**：序號授權、Word 匯出、練習證明圖檔、已儲存方案（可依需要再從姊妹專案移植）。

## 5 組範例（全虛構，數字已用獨立 Python 模型與 Playwright 核對一致）

| 範例 | 月營收 | 月淨利（淨利率） | 示範狀態 |
|---|---|---|---|
| 早午餐店 | 615,070 | 69,569（11.3%） | 健康基準組，無警示、無資料提醒 |
| 拉麵店 | 1,087,560 | 245,656（22.6%） | 2 緊急＋1 注意叫貨（前置 2 天） |
| 手搖飲店 | 581,615 | 85,284（14.7%） | 外送 40%；備料資金 34,750 > 上限 30,000 |
| 火鍋店 | 1,505,850 | −88,840（−5.9%） | 固定支出 90 萬虧損、主要成本 69.7% |
| 便當店 | 700,325 | 99,574（14.2%） | 便當食材成本率超標、手作甜湯零銷售（瘦狗、無可撐天數） |

調整範例後務必重跑：在 console 執行 `__menuMix.applyPreset(i)`（測試 hook，回傳 calc），確認 `#verifyTitle` 為 ✅ 且狀態仍成立。

## 踩坑

- `.bar-fill`／`.flow-fill` 是放在 flex 子項裡的 `<span>`，**必須 `display:block`** 否則 width 無效、橫條全空白。（Amazon 版 `.bar-fill` 也沒加，分類橫條可能同樣看不到，尚未修。）
- `applyPreset()` 在初始化（`loadState()` 首次載入）時 `rendered` 仍為 false，只改 state 不渲染，由 Init 區塊統一渲染。

## localStorage

`restaurantMenuMixState`、`restaurantMenuMixApiConfig`、`restaurantMenuMixActivePreset`、`restaurantMenuMixMarquee`。

## 指令

無建置步驟。預覽：port `8825`（根目錄 `.claude/launch.json` 的 `restaurant-menu-mix-calculator`），或 `python -m http.server 8825 --directory 資料儀表板/restaurant-menu-mix-calculator`。浮水印沿用 amazon-listing-mix-calculator 的 `watermark-source.png`，base64 注入 `WATERMARK_DATA_URI`（替換方法同該專案 CLAUDE.md）。

## 部署

尚未推 GitHub／未上 Pages（依「實驗性新工具部署前先確認」慣例，待使用者決定）。
