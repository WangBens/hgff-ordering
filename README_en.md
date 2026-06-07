[🇹🇼 繁體中文](README.md) | [🇺🇸 English](README_en.md)

---

# 🚀 Huang Guifei Tea Room - LINE Online Ordering System Development Log

> **Development Log & Insights**
> This was an incredibly challenging journey from scratch. In the early stages of the project, I wasn't entirely clear on the overall architecture and workflows. By clarifying the requirements, I broke the system down into core modules: "Menu Display," "Shopping Cart," "Order Management," and "Checkout Calculation." During implementation, I encountered various issues, including inconsistent data formats, unresponsive buttons, incorrect cart accumulation, UI synchronization failures, and even type errors causing incorrect price calculations. By continuously refactoring the logic, separating concerns, and optimizing the UI, I ultimately built a commercial-grade system. This process taught me how to independently troubleshoot, debug, and optimize logic, significantly improving my software development skills.

---

## I. System Architecture & Database Design (Backend)

This system utilizes Google Sheets as a lightweight database, paired with Google Apps Script (GAS) to provide API services, perfectly integrating with LINE LIFF for seamless login.

* **Automated Table Generation:** Wrote the `ensureSheetsExist` function to automatically generate all necessary tables (System Settings, Admins, Member Balances, Transactions, Menu, Customizations, Orders, and Promos) upon initial setup.
* **Powerful Discount Engine:** Developed a complex pricing model (`calculateBackendTotal`) on the backend, supporting "Buy X Get Y Free (Storewide/Category/Item)" and "Combo Pricing." The system flattens the shopping cart and applies discounts prioritizing higher-priced items to ensure the best deal for the customer.

---

## II. Customer Interface Development (INDEX.html)

To provide the best mobile ordering experience, a dedicated customer webpage was built using LINE LIFF.

* **Member & Wallet System:** Customers auto-login via LINE integration, with one-click member registration. The system displays the member's wallet balance in real-time and provides a complete history of transactions and orders.
* **Dynamic Menu & Marquee:** Auto-generates categorical menus based on backend settings. If active promotions exist, a dynamic marquee appears at the top, and a "Today's Special Combos" section is independently displayed.
* **Highly Customizable Cart:** Customers can choose sizes (L/XL), sweetness, ice levels, and support "multiple toppings."
* **Seamless Checkout:** Supports both "Pay in-store" and "Wallet Deduction." The checkout screen accurately displays the system's auto-applied promotional discounts and the final total in real-time.

---

## III. Boss Management Interface (BOSS.html)

The backend system is the core of operations, featuring an intuitive graphical interface for administrators.

* **Store Status & Real-time Dashboard:** One-click toggle for "Open/Closed" status. The order dashboard auto-refreshes every 15 seconds, providing status buttons for "Accept," "Complete," and "Collected."
* **Menu & Promo Management:** Supports quick item uploads and dual-size pricing. A robust promo interface allows for item-specific or combo discounts, and supports taking down items or pausing promos at any time (with fail-safes to prevent taking down items tied to active promos).
* **Digital Wallet Management:** The boss can manually "Top-up" or "Deduct" funds for specific customers with notes. All financial flows are recorded in the transaction history for auditing.

---

## IV. Security Mechanisms

To elevate the system to a commercial level, multiple layers of protection and security mechanisms were implemented on both the frontend and backend:

* **LINE ID Token Authentication:** Instead of relying on easily spoofed `userId`s, the frontend is forced to send an `idToken`. The backend performs secondary verification with LINE servers to ensure authenticity.
* **Idempotency (Request ID Cache):** For write actions like checkout or balance adjustments, the frontend attaches a random `requestId`. The backend uses `CacheService` to lock it for 60 seconds, preventing duplicate orders even if the user spams the button due to lag.
* **Rate Limiting:** Capped API calls per user (60 write actions / 180 read actions per minute) to effectively prevent malicious script spam or DDoS attacks.
* **Concurrency Locking (LockService):** Implemented `LockService.getScriptLock().waitLock(10000)` during "wallet deduction" and "order refunds" to prevent race conditions that could lead to negative balances or corrupted order states.
* **Strict Permission Isolation:** Admin-exclusive APIs strictly cross-reference the sender against the "Admin List." Unauthorized users cannot execute actions even if they obtain the API URL.

---

## V. Free Tier Capacity & System Limits

This system is built entirely on free resources. The estimated capacity is as follows:

* **Frontend GitHub Pages (Hosting):**
  * Extremely high bandwidth (approx. 100GB/month). Since it's pure static HTML/JS, it can easily handle **thousands to tens of thousands of concurrent users**.
* **Backend Google Apps Script (API Server):**
  * Free quota allows approx. 20,000 URL Fetches/day (used for LINE Token validation) and 90 mins of total execution time/day.
  * **Concurrency Estimate:** Can stably support **30~50 people submitting orders or refreshing simultaneously "within the exact same second."** This is more than sufficient for small to medium-sized beverage stores.
* **Database Google Sheets:**
  * **Capacity Limit:** A single spreadsheet supports up to **10 million cells**. At roughly 10 cells per order, it can accumulate **hundreds of thousands of orders** before reaching limits.

---

## VI. Key Technical Challenges & Solutions

**⚠️ Challenge 1: The "Fake Display" of Order Quantities in Backend**
* **Issue:** To calculate discounts accurately, the backend flattens identical drinks into single-item arrays. This caused an order of 5 Black Teas to display as 5 separate rows in the admin dashboard, making it hard to read.
* **Solution:** Introduced a `groupedItems` logic in `BOSS.html`. It re-aggregates items with identical "size, name, sweetness, ice, and toppings," summing the `qty` to display cleanly as "[L] Black Tea x5", vastly improving readability for preparation.

**⚠️ Challenge 2: Frontend/Backend Pricing Discrepancies**
* **Issue:** The amount seen in the frontend cart differed from the actual backend deduction upon submission. The initial frontend logic couldn't perfectly handle overlapping "Combo Pricing" and "Buy X Get Y" logic.
* **Solution:** Achieved 100% logic synchronization. The strict sorting and flattening pricing logic (`calculateBackendTotal`) from the backend was completely ported to the frontend's `calculateCart()`, ensuring zero discrepancies.

**⚠️ Challenge 3: API Latency Freezing the UI (UX Pain Point)**
* **Issue:** With unstable networks or slow GAS responses, `fetch()` would wait indefinitely, freezing the screen on "Processing..." and making users think it crashed.
* **Solution:** Implemented an `AbortController` with a 15-second Timeout mechanism in the `callGAS()` wrapper. If the connection takes too long, it gracefully aborts and prompts "Server response timed out, please try again later."

**⚠️ Challenge 4: Syntax Errors from Residual Code**
* **Issue:** During repeated refactoring of the frontend pricing logic, a stray loop was left outside the main function, triggering a `SyntaxError` and causing a white screen.
* **Solution:** Re-audited the code blocks, pinpointed and removed the isolated residual code, and refactored the scope, restoring normal rendering.

---

## VII. Deployment Guide (Step-by-Step)

To deploy this system in a new environment, follow these steps sequentially.

### 📌 Prerequisites: Code Modification Checklist
Before starting, ensure you have the following three files. You must modify the IDs in these three files during deployment:
1. `GS` (Backend): Modify `LINE_CLIENT_ID`
2. `INDEX.html` (Customer Frontend): Modify `CUSTOMER_LIFF_ID` and `GAS_API_URL`
3. `BOSS.html` (Admin Frontend): Modify `BOSS_LIFF_ID` and `GAS_API_URL`

### Step 1: Apply for LINE Developers & LIFF IDs
1. Go to **[LINE Developers Console](https://developers.line.biz/)** and log in.
2. Create a **Provider** (e.g., Huang Guifei Tea Room).
3. Create a **LINE Login** Channel under this Provider.
4. Go to the Channel settings and note down the **Channel ID** (This is the `LINE_CLIENT_ID` for the backend).
5. Switch to the **LIFF tab** and click "Add" to create two LIFF Apps:
   * **Customer LIFF**: Size = `Full` or `Tall`, Scope = `profile` & `openid`.
   * **Admin LIFF**: Size = `Full`, Scope = `profile` & `openid`.
6. Note down the **LIFF IDs** generated for both apps.

### Step 2: Set up Google Sheets & Apps Script (Database & API)
1. Create a brand new **Google Sheet**, click **Extensions > Apps Script**.
2. Paste the `GS` (backend code) and replace the `LINE_CLIENT_ID` at the top with the one from Step 1.
3. **🔑 Crucial Step: Trigger Google Authorization**
   * Change the execution function dropdown at the top to `ensureSheetsExist`.
   * Click "Run".
   * A prompt will appear $\rightarrow$ Click "Review permissions" $\rightarrow$ Select your account $\rightarrow$ Click "Advanced" $\rightarrow$ Click "Go to (unsafe)" $\rightarrow$ Click "Allow".
4. Click **Deploy > New deployment** in the top right.
5. Select type: **Web App**. Execute as: **Me**. Who has access: **Anyone**.
6. After deployment, copy the **Web App URL**. This is your `GAS_API_URL`.

---

To allow your Google Apps Script to successfully call external APIs (LINE) and access your spreadsheet, you must manually trigger an authorization request before deployment.

Please paste the following code into your Apps Script editor and run it once.

> ⚠️ This function calls three different Google services, which will prompt Google to display the authorization consent screen.

```javascript
function forceAuth() {
  UrlFetchApp.fetch("https://api.line.me/");
  SpreadsheetApp.getActiveSpreadsheet().getName();
  LockService.getScriptLock();
}
```

---

### Step 3: Bind Keys & Deploy Frontend Webpages
1. Edit **INDEX.html**, replace `CUSTOMER_LIFF_ID` and `GAS_API_URL`.
2. Edit **BOSS.html**, replace `BOSS_LIFF_ID` and `GAS_API_URL`.
3. Upload both files to a static hosting service (e.g., GitHub Pages) and publish.
4. Retrieve the **public URLs** for both published files.

### Step 4: Finalize LIFF Binding & System Initialization
1. Return to the **LIFF tab** in the LINE Developers Console.
2. Fill the public URLs obtained in Step 3 into the respective **Endpoint URL** fields of the two LIFF apps and save.
3. **System Auto-Build:** Open the "Admin LIFF" link via LINE. The backend will detect the initial run and automatically generate all required tables in your Google Sheet.
4. **Set Admin Privileges:** Open the Google Sheet, navigate to the "管理員名單" (Admin List) tab, and input your LINE User ID.
5. Re-open the Admin LIFF to start configuring menus and promos!

### Step 5: Setup LINE Official Account Rich Menu
1. Log into the **[LINE Official Account Manager](https://manager.line.biz/)**.
2. Go to **Home > Rich menus**, click "Create".
3. Set the title, display period, and upload your designed menu image.
4. In "Action", select **Link (URL)**.
5. Paste your **Customer LIFF URL** (Format: `https://liff.line.me/YOUR-CUSTOMER-LIFF-ID`).
6. Click "Save". Customers can now click the menu in the chatroom to seamlessly launch the ordering system!
