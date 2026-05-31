# ⚙️ Inspection Billing Console

A multi-month inspection allowance billing system hosted on **GitHub Pages** with a **Google Apps Script** backend connected to Google Sheets.

---

## 🏗️ Architecture Overview

```
GitHub Pages (index.html)
        │
        │  fetch() via HTTPS
        ▼
Google Apps Script Web App (Code.gs)
        │
        ├── Google Sheets  →  Employee data, Deduction History
        ├── Google Drive   →  Generated PDF bills
        └── Gmail          →  Email dispatch to employees
```

- **Frontend** — `index.html` hosted on GitHub Pages. Runs entirely in the browser.
- **Backend** — `Code.gs` deployed as a GAS Web App. Talks to Sheets, Drive, and Gmail.
- **No server needed** — GitHub Pages is free static hosting. All logic stays in GAS.

---

## 📁 File Structure

```
├── index.html      ← Frontend UI (upload to GitHub repo)
├── Code.gs         ← Backend engine (paste into Google Apps Script)
└── README.md       ← This file
```

---

## 🚀 Setup Guide

### Step 1 — Google Sheets Setup

Your spreadsheet must have these exact sheet (tab) names:

| Sheet Name         | Purpose                          |
|--------------------|----------------------------------|
| `Raw Sheet`        | Employee records (data from Row 4, Col B–M) |
| `Adjustment Form`  | Bill template — auto-filled per employee |
| `Bill F`           | Bill F template — auto-filled per employee |
| `Bill B`           | Bill B template — auto-filled per employee |
| `Deduction History`| Created automatically on first run |

---

### Step 2 — Deploy the Backend (Code.gs)

1. Open your Google Spreadsheet
2. Go to **Extensions → Apps Script**
3. Replace all content in `Code.gs` with the contents of the `Code.gs` file from this repo
4. Click **Save** (💾)
5. Click **Deploy → New Deployment** (or **Manage Deployments → Edit** if already deployed)
6. Set the following:
   - **Type:** Web App
   - **Execute as:** Me
   - **Who has access:** Anyone
7. Click **Deploy** and **authorise** when prompted
8. Copy the **Web App URL** — it looks like:
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```

> ⚠️ **Important:** Every time you edit `Code.gs`, you must click **Deploy → Manage Deployments → Edit → New Version** for changes to take effect on the live URL.

---

### Step 3 — Host the Frontend on GitHub Pages

1. Create a new GitHub repository (or use an existing one)
2. Upload `index.html` to the **root** of the repository
3. Go to **Settings → Pages**
4. Under **Source**, select `main` branch and `/ (root)` folder
5. Click **Save**
6. GitHub will give you a URL like:
   ```
   https://your-username.github.io/your-repo-name/
   ```

---

### Step 4 — Connect Frontend to Backend

1. Open your GitHub Pages URL in a browser
2. A red **⚠️ Backend URL Not Configured** banner will appear automatically
3. Paste your GAS Web App URL into the input field
4. Click **💾 Save URL**
5. The URL is saved in your browser's `localStorage` — you only do this **once per device**

> To update the URL later, click the **⚙️ API Settings** button in the Staff Registry tab.

---

## 🔄 How API Calls Work

All communication between the frontend and GAS uses standard `fetch()` HTTP calls.

### GET requests (read operations)

| Action | URL |
|--------|-----|
| Load all employees | `GET ?action=getEmployees` |
| Load deduction history | `GET ?action=getHistoricalDeductions&year=2026` |

### POST requests (write operations)

| Action | Body |
|--------|------|
| Save / update staff | `{ action: "saveStaff", staffData: {...} }` |
| Run billing pipeline | `{ action: "processAllBills", selectedMonths: [...], gridDeductionData: [...] }` |

---

## 📊 Features

### 👥 Staff Registry
- Add, edit, and search employee records
- Quick-load any employee into the form
- Live stats: Total / Active / Inactive count

### 💳 Billing & Processing
- Select 1–4 months per billing run
- Duplicate month+year prevention
- Deduction matrix — enter per-employee deductions per month
- Prior deduction warning (highlighted in yellow)
- Deduction history viewer by year

### 🚀 Pipeline Execution
- Generates filled PDF bills (Adjustment Form + Bill F + Bill B)
- Saves PDFs to Google Drive folder `Inspection Allowance Bills`
- Emails each employee their PDF automatically
- Logs all deductions to `Deduction History` sheet
- **Animated progress bar** with estimated completion time (8 sec/employee)

---

## ⚙️ Configuration Reference

### In `Code.gs`

```javascript
const RAW_SHEET_NAME       = "Raw Sheet";        // Employee data sheet
const ADJ_SHEET_NAME       = "Adjustment Form";  // Bill template sheet
const BILL_F_SHEET_NAME    = "Bill F";           // Bill F template sheet
const BILL_B_SHEET_NAME    = "Bill B";           // Bill B template sheet
const DEDUCTION_SHEET_NAME = "Deduction History";// Auto-created
const FOLDER_NAME          = "Inspection Allowance Bills"; // Drive folder
const ALLOWANCE_RATE       = 25000;              // Monthly allowance (PKR)
const PDF_MAX_RETRIES      = 3;                  // PDF generation retry attempts
const PDF_RETRY_DELAY_MS   = 2000;              // Delay between retries (ms)
```

### In `index.html`

```javascript
const AVG_SECS_PER_EMP = 8; // Progress bar speed estimate (seconds per employee)
const ALLOWANCE        = 25000; // Must match ALLOWANCE_RATE in Code.gs
```

### Budget App Link

In both files, replace this placeholder with your actual Budget frontend URL:
```
YOUR_BUDGET_FRONTEND_URL_HERE
```

---

## 🛠️ Troubleshooting

| Problem | Solution |
|---------|----------|
| "Backend URL Not Configured" banner | Click ⚙️ API Settings and paste your GAS Web App URL |
| Data not loading after editing Code.gs | You must redeploy: Deploy → Manage Deployments → Edit → New Version |
| CORS error in browser console | Make sure Web App access is set to **Anyone** in GAS deployment settings |
| PDF generation fails | GAS retries up to 3 times automatically. Check Drive quota and sheet names |
| Emails not sending | Verify employee Gmail addresses are correct in Raw Sheet |
| Progress bar finishes but no toast | Check browser console for fetch errors; usually a CORS or URL issue |
| "No Active employees found" | Set employee Status to `Active` in the Staff Registry tab |

---

## 🔐 Security Notes

- The GAS Web App URL is stored only in **your browser's localStorage** — it is never uploaded to GitHub
- Set GAS access to **"Anyone"** (not "Anyone with Google Account") so GitHub Pages can call it without login
- Employee data stays entirely within your **Google account** (Sheets + Drive + Gmail)
- No third-party servers are involved

---

## 📞 Portal Link

Employees access their performance portal at:
```
https://mwaqas3341-aeo.github.io/Inspection-Allowance-Performance/
```

---

## 📝 License

Internal government tool — not for public redistribution.
