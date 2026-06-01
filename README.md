# 🍵 凰貴妃茶堂 - 線上點餐與店鋪管理系統

基於 LINE LIFF 與 Google Apps Script (GAS) 開發的輕量級、高安全性線上點餐系統。無需建置複雜的伺服器與資料庫，利用 Google Sheets 作為後端資料中心，適合中小型餐飲店鋪快速導入。

## ✨ 系統特色

### 👤 顧客端 (LINE LIFF APP)
* **無縫登入體驗**：透過 LINE LIFF 自動獲取使用者身分，一鍵加入專屬會員。
* **專屬電子錢包**：支援會員「餘額扣款」與「現場結帳」雙軌制。
* **彈性客製化點餐**：直覺的購物車系統，支援容量、甜度、冰塊設定，以及多選加料（自動計算加價）。
* **即時進度追蹤**：隨時查看未完成訂單的製作狀態（新訂單 / 製作中 / 可取餐）。
* **透明交易紀錄**：提供完整的錢包消費與儲值歷史明細。

### 👑 老闆端 (店鋪管理後台)
* **即時訂單看板**：每 10 秒自動刷新，不錯過任何新訂單。
* **一站式訂單管理**：單鍵切換訂單狀態（接單 ➔ 製作完成 ➔ 結帳），支援訂單取消與餘額自動退款。
* **會員錢包管理**：可手動為指定會員進行現金儲值、人工扣款，並附帶備註紀錄。
* **客戶關係管理**：可隨時更新會員的真實稱呼，方便現場叫號與核對身份。

---

## 🛡️ 資安防護機制 (Security Highlights)

本系統全面落實「**不信任前端輸入**」的資安原則，具備以下實戰級防護：

1. **嚴格身分驗證 (LINE idToken Validation)**
   - 放棄傳遞容易被偽造的 `userId`，改為傳遞高安全級別的 JWT (`idToken`)，由後端直接向 LINE 伺服器驗證真實身份。
2. **後端統一計價 (Source of Truth)**
   - 購物車送出後，後端會根據 Google Sheet 的原始菜單與加料價格「重新計算總價」，徹底杜絕前端惡意竄改訂單金額或數量的風險。
3. **管理員路由攔截 (Admin Route Guard)**
   - 所有後台操作（如修改餘額、變更訂單狀態、查詢所有會員）皆需通過後端 `isUserAdmin` 嚴格檢驗，拒絕任何越權的 API 呼叫。
4. **執行緒鎖定防併發 (LockService)**
   - 牽涉到錢包餘額扣款、自動退款等高敏感操作，皆導入 GAS 的 `LockService`，防止同時發送多筆請求導致的餘額錯亂 (Race Condition)。
5. **XSS & CSV Injection 防禦**
   - 前端導入 HTML 跳脫處理 (`escapeHTML`)，後端寫入試算表前進行 `sanitizeInput`，阻絕惡意腳本與試算表公式注入攻擊。

---

## 🛠️ 技術架構 (Tech Stack)

* **前端 (Frontend)**: HTML5, CSS3, Vanilla JavaScript, LINE LIFF SDK v2
* **後端 (Backend)**: Google Apps Script (GAS)
* **資料庫 (Database)**: Google Sheets
* **部署 (Deployment)**: GitHub Pages (前端) / Google Cloud (後端)

---

## 🚀 部署指南 (Setup & Deployment)

### 1. Google Sheets 資料庫建置
建立一個新的 Google 試算表，並建立以下分頁（請確保名稱完全一致）：
- `菜單` (欄位：分類、品項名稱、L杯價格、XL杯價格、狀態)
- `加料與客製化` (欄位：類別、選項名稱、加價金額、狀態)
- `訂單紀錄` (欄位：訂單編號、時間、LINE_ID、姓名暱稱、訂單內容、總金額、付款方式、訂單狀態)
- `會員與餘額` (欄位：LINE_ID、姓名暱稱、目前餘額)
- `管理員名單` (欄位：LINE_ID、姓名、權限級別)
- `交易明細` (欄位：交易時間、LINE_ID、會員名稱、交易類型、異動金額、交易後餘額、備註)

### 2. LINE Developers 設定
1. 進入 LINE Developers Console，建立一個 LINE Login Channel。
2. 取得該 Channel 的 **Channel ID**（共 10 碼數字）。
3. 在 LIFF 頁籤中新增兩個 LIFF App（一個給顧客端，一個給老闆端），並取得對應的 **LIFF ID**。
   - *注意：Scopes 務必勾選 `openid` 與 `profile`。*

### 3. Google Apps Script 後端部署
1. 在試算表上方選單點擊 `擴充功能` > `Apps Script`。
2. 將 `程式碼.gs` 貼入編輯器中。
3. 將頂部的 `LINE_CLIENT_ID` 替換為你在步驟 2 取得的 Channel ID。
4. 點擊右上角「部署」>「新增部署作業」，選擇「網頁應用程式」，權限設定為「所有人」。
5. 獲取發布後的 **網頁應用程式網址 (GAS API URL)**。

### 4. 前端網頁部署
1. 將專案中的 `index.html` 與 `boss.html` 內部的 `GAS_API_URL` 替換為步驟 3 獲取的網址。
2. 將 `CUSTOMER_LIFF_ID` 與 `BOSS_LIFF_ID` 替換為步驟 2 獲取的 LIFF ID。
3. 將修改後的 HTML 檔案上傳至 GitHub 並開啟 GitHub Pages，或部署至任何靜態網頁代管服務。
4. 回到 LINE Developers Console，將 LIFF App 的 Endpoint URL 填入你的前端網頁部署網址。

---

## 📜 授權條款 (License)
本專案僅供個人/內部學習與營運使用。
