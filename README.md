# SchedulingSystem
排班系統（SchedulingSystem）

部署：GitHub Pages（靜態單頁）

前端：React 18（UMD）、Tailwind、Babel（inline）

後端：Firebase Auth + Firestore

目前版本：20250922-36

1) 快速開始

在 Firebase 建立專案並啟用：

Email/Password 登入

Cloud Firestore（Production 或 Test 皆可）

將 index.html 內的 firebaseConfig 換成你的專案設定值。

推上 GitHub，開啟 GitHub Pages（root 使用 index.html）。

造訪 GitHub Pages 網址，註冊/登入即可使用。

2) Firestore 結構

根節點：artifacts/{APP_ID}/workspaces/{WORKSPACE_ID}/…（多人共用同一份資料）

artifacts/{APP_ID}/workspaces/{WORKSPACE_ID}/
  members/{uid}                  # 個人資料（displayName、email、updatedAt）
  staffs/staffList               # 員工名單：{ names: [ "Alice", "Bob", ... ] }
  schedules/{YYYY-MM}            # 月份班表：{ schedule: JSON.stringify([...]) }
  attendance/{YYYY-MM-DD}/logs/{autoId}
                                 # 每日打卡明細（IN/OUT，含 serverTimestamp）
  attendance_flat_logs/{autoId}  # 扁平化打卡彙總（含 dateKey、monthKey）


索引：若月彙總查詢有提示，依 Console 引導建立 attendance_flat_logs 的複合索引（例如 monthKey == 'YYYY-MM'）。

3) Firestore 規則（摘要）

workspaces 下通用：已登入者可讀。

members：本人（request.auth.uid == uid）才可寫。

attendance：允許 create/read，禁止 update/delete。

staffs / schedules：已登入者可讀寫（可依實際權限需求再縮限）。

註：規則需與實際集合路徑一致（attendance_flat_logs 名稱要完全相同）。

4) 使用流程（操作動線）

註冊/登入

支援記住 Email；登入後顯示當前版本與工作區資訊。

第一次打卡前 → 姓名設定

若尚未設定「我的姓名」，按「上班/下班打卡」會先跳出彈窗要求輸入姓名，儲存後自動續打卡（體驗最順）。

員工名單

在「控制列」輸入姓名 → 按「新增」。

刪除員工會有確認視窗，成功後自動關閉並更新班表與統計。

編排班表

主表按格子循環切換：早班 → 早+午班 → 午班 → 午+晚班 → 晚班 → 休 → 支援樹林 → 請假。

系統會依 營業時間規則 自動計算顯示：

週一～五 09:00–22:00

週六 09:00–21:00

週日 09:30–21:00

自動儲存（debounce）；也可按「手動儲存」。

打卡

打卡寫入 attendance/{date}/logs，同時 mirror 到 attendance_flat_logs（含 dateKey、monthKey）。

頁面顯示：

個人今日最近兩筆

全員「最新上班 / 最新下班」各一筆（不同底色）

匯出班表（行事曆格式）

按「匯出班表」→ 彈出視窗（週一→週日 顯示、每格含 月/日）。

右上角：

下載（PDF）：A4 橫式、單頁；若人多可在列印對話框把縮放調至 80～90%。

友善列印：另開視窗（同 A4 橫式樣式）。

僅顯示「姓名：班別」，不顯示時段（和需求一致）。

出勤彙總（本月）

從 attendance_flat_logs 依 monthKey 聚合：

出勤天數、缺卡天數、總時數（依營業窗口裁切計算）

可 下載 CSV 或 友善列印。

5) 常見問題（FAQ）

A：新使用者登入會看到上一位的姓名？
現已在偵測使用者切換時清空個人狀態（姓名、今日打卡、彈窗），避免殘留。

B：PDF/列印跑版？
已預設 A4 橫式與單頁佈局。如果人數過多造成溢出，列印時可把「縮放」調整至 80–90%。

C：打卡無法寫入 / 規則被拒？
檢查集合名稱與規則是否一致（特別是 attendance_flat_logs），並確認已登入。

6) 發布與版控

版本號規則：YYYYMMDD-序號，例如 20250922-36。

顯示位置：登入頁與主畫面右上角皆顯示 CODE_VERSION。

建議作法：

每次改動先更新 CODE_VERSION 常數。

Commit message 帶上版本號。
3.（可選）打 Git tag：git tag v20250922-36 && git push --tags

7) 版本變更紀錄（CHANGELOG）

在每次發版時，將最新條目補在最上方。

20250922-36

新增：使用者切換帳號時，自動清空上一位使用者的個人狀態（姓名、今日打卡、彈窗開關等）。

改善：行動裝置上「新增員工姓名」輸入框最小寬度（min-w-[240px]），避免過窄。

維持：匯出班表為月份行事曆（週一→週日），PDF 與友善列印預設 A4 橫式單頁。

20250922-35

新增：首次打卡若未設定姓名，先跳出「設定姓名」彈窗；儲存後自動續打卡。

修正：attendance_flat_logs 集合命名與程式一致，避免無效路徑。

修正：行事曆匯出模板改為純字串組裝，避免「Unterminated template/script」錯誤。

20250922-34

新增：匯出班表（行事曆格式）彈出視窗：下載 PDF、友善列印。

新增：今日所有人的最新上/下班各一筆（不同底色顯示）。

調整：姓名顯示與打卡紀錄顯示一致化。

早期版本略…

8) 發版前快速檢查（Release Checklist）

 已更新 CODE_VERSION。

 打卡：首次未填姓名 → 會跳出姓名彈窗 → 儲存後自動打卡成功。

 新增/刪除員工：狀態提示正確、刪除確認窗會自動關閉。

 手動儲存：換瀏覽器或重新整理仍可看到相同班表。

 匯出班表：PDF 下載與友善列印皆為 A4 橫式；內容為月曆格、週一→週日、只顯示「姓名：班別」。

 本月出勤彙總：可載入、下載 CSV、友善列印。

 行動裝置：新增員工輸入框寬度正常、按鈕不被遮擋。

 Firestore 規則與實際集合名稱完全一致。
