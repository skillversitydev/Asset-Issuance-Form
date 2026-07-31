# Skillversity - Asset Issuance Form & HR Administrator Portal

A modern, responsive web application for allocating equipment, issuing assets, generating official PDF reports with logo branding, auto-fetching previous employee records via Google Sheets, managing local admin database tables, and logging data in real-time.

---

## 🚀 Features

- **Two-Tier HR Administrator Login**:
  - **`admin` (Standard Admin)**: Password `admin123` — Can fill forms, lookup records, submit data, and generate PDF reports.
  - **`pladmin` (Power Level Admin)**: Password `pladmin123` — Exclusive access to the **`📊 VIEW RECORDS TABLE`** dashboard & CSV export!
- **Clean Security Warning**: Displays clean error messages (`Incorrect Username or Password! Please enter correct Username and Password`) without exposing credentials.
- **Local Admin Database Table**: Exclusive dashboard for `pladmin` to view, filter search, load (`📝 LOAD`), delete (`🗑️`), and export all local records to Excel/CSV (`📥 EXPORT CSV`).
- **Automatic Admin Auto-Logout**: Auto-logs out and redirects to the login screen immediately after PDF generation & data sync.
- **15 Multi-Select Assets**: Grid selection for Laptops, Laptop Charger, Laptop Bag, Mobiles, Mobile Charger, Tablets, Tablet Charger, Pendrive, Hard Disk, Desktop PC, Mouse, Keyboard, Memory Card, Headphones, and **SIM CARD**.
- **Hardware Serial Number Inputs**: Complete hardware specifications and serial numbers (Laptop, Charger, **Head Phone Serial / Model Number**, Mouse, Keyboard, Mobile, **SIM / Phone Number**, IMEI, Charger, Tablet, Storage).
- **Multi-Storage Dynamic Specs Loading**: Check multiple storage options (Pendrive, Hard Disk, Memory Card) to dynamically generate separate Capacity and Serial Number fields for each device!
- **Multi-Issuance Date Tracking**: Supports **Asset Received Date**, **2nd Asset Issuance Date**, and **3rd Asset Issuance Date** for tracking equipment provided across multiple dates.
- **🔍 Record Lookup & Auto-Populate (`🔍 LOAD RECORD`)**: Enter Employee ID (e.g. `QLI0001`) to automatically fetch previous issuance history from Google Sheets via 100% CORS-proof JSONP and pre-fill the form for updates.
- **Up to 3 Submissions per Employee**: Allows up to 3 submissions per Employee ID / Email ID.
- **100% Mobile & Touch Friendly**: Touch-optimized inputs (`16px` iOS auto-zoom prevention), responsive single-column grid stacking on smartphones/tablets.
- **High-Contrast PDF Report Generation**: Instant client-side download of official **1-Page Asset Issuance Report** (`Skillversity_Asset_Report_[Name]_[ID].pdf`) complete with logo branding, deep navy headers, and dual signature blocks.
- **Google Sheets Real-time Sync**: Background delivery to Google Sheets Web App endpoint (`Asset Issuance Records` tab).

---

## 📁 Files Included in Repository

```
├── index.html        # Main application page (HR Login, Asset Form, PDF Builder, Modal & Logic)
├── index.css         # Complete CSS Design System & Mobile Responsive Rules
├── logo.png          # Skillversity Official Logo
├── favicon.png       # Web Browser Favicon
└── README.md         # Documentation & Setup Guide
```

---

## 📜 Latest Google Apps Script (33 Columns with Dual JSON & JSONP Support)

Copy and paste this script into your Google Sheets Apps Script editor:

```javascript
// ====================================================================
// SKILLVERSITY ASSET ISSUANCE - COMPLETE DUAL FETCH & SAVE SCRIPT
// Spreadsheet ID: 1G2QmvAjKL1c-oim1BmG-JtAH0-EbaHMvYSGTT9SmzoI
// Sheet Tab Name: Asset Issuance Records
// ====================================================================

var SPREADSHEET_ID = "1G2QmvAjKL1c-oim1BmG-JtAH0-EbaHMvYSGTT9SmzoI";
var SHEET_NAME = "Asset Issuance Records";

var HEADERS = [
  "Timestamp", 
  "Admin User", 
  "Asset Received Date", 
  "2nd Asset Issuance Date", 
  "3rd Asset Issuance Date", 
  "Employee Joining Date", 
  "Additional Received Date", 
  "Employee Name", 
  "Employee Email", 
  "Employee ID", 
  "Department", 
  "Asset Issued By", 
  "Assets Issued", 
  "Laptop Brand", 
  "Laptop Serial", 
  "Laptop Charger Serial", 
  "Headphone Brand", 
  "Headphone Serial / Model", 
  "Mouse Type", 
  "Mouse Serial", 
  "Keyboard Type", 
  "Keyboard Serial", 
  "Mobile Brand", 
  "SIM / Phone Number", 
  "Mobile Serial", 
  "Mobile IMEI", 
  "Mobile Charger Serial", 
  "Tab Brand", 
  "Tab Serial", 
  "Storage Device(s)", 
  "Storage Capacity", 
  "Storage Serial Number", 
  "Remarks"
];

// Safe target sheet accessor
function getTargetSheet() {
  var ss;
  try { 
    ss = SpreadsheetApp.openById(SPREADSHEET_ID); 
  } catch (e) { 
    ss = SpreadsheetApp.getActiveSpreadsheet(); 
  }
  if (!ss) {
    ss = SpreadsheetApp.getActiveSpreadsheet();
  }
  
  var sheet = ss.getSheetByName(SHEET_NAME);
  if (!sheet) { 
    sheet = ss.getSheets()[0]; 
    try { 
      sheet.setName(SHEET_NAME); 
    } catch(e) {} 
  }
  return sheet;
}

/**
 * 1. FORCE RESET & UPDATE HEADERS FUNCTION
 * Select 'forceResetHeaders' from dropdown and click 'Run' ▶️ 
 * Force clears Row 1 and updates all 33 Navy Blue header columns!
 */
function forceResetHeaders() {
  var sheet = getTargetSheet();
  
  sheet.getRange("1:1").clearContent().clearFormat();
  sheet.getRange(1, 1, 1, HEADERS.length).setValues([HEADERS]);
  
  var headerRange = sheet.getRange(1, 1, 1, HEADERS.length);
  headerRange.setBackground("#1E3A8A"); 
  headerRange.setFontColor("#FFFFFF");  
  headerRange.setFontWeight("bold");
  sheet.setFrozenRows(1);
  sheet.autoResizeColumns(1, HEADERS.length);
  
  Logger.log("✅ Success! Updated all 33 column headers in Google Sheet.");
}

function setupSheet() {
  forceResetHeaders();
}

/**
 * 2. GET REQUEST HANDLER (FETCH DATA FROM SHEET)
 * Supports both JSON & JSONP for 100% CORS-proof record search
 */
function doGet(e) {
  try {
    var sheet = getTargetSheet();
    var callback = (e && e.parameter && e.parameter.callback) ? e.parameter.callback : null;
    var result = { "found": false };

    if (e && e.parameter && e.parameter.action === "lookup") {
      var query = (e.parameter.empId || e.parameter.empEmail || "").toString().trim().toUpperCase();
      if (query && sheet.getLastRow() >= 2) {
        var data = sheet.getDataRange().getValues();
        for (var i = data.length - 1; i >= 1; i--) {
          var row = data[i];
          var rowEmail = (row[8] || "").toString().trim().toUpperCase(); // Col I
          var rowId = (row[9] || "").toString().trim().toUpperCase();    // Col J
          
          if (query === rowId || query === rowEmail) {
            result = {
              "found": true,
              "record": {
                "timestamp": row[0], "adminUser": row[1], "date": row[2], "date2": row[3], "date3": row[4],
                "joiningDate": row[5], "receivedDate": row[6], "employeeName": row[7], "employeeEmail": row[8],
                "employeeId": row[9], "department": row[10], "issuedBy": row[11], "assetsIssued": row[12],
                "laptopBrand": row[13], "laptopSerial": row[14], "laptopChargerSerial": row[15],
                "headphoneBrand": row[16], "headphoneSerial": row[17], "mouseType": row[18], "mouseSerial": row[19],
                "keyboardType": row[20], "keyboardSerial": row[21], "mobileBrand": row[22], "simNumber": row[23],
                "mobileSerial": row[24], "mobileImei": row[25], "mobileChargerSerial": row[26], "tabBrand": row[27],
                "tabSerial": row[28], "storageDevice": row[29], "storageCapacity": row[30], "storageSerial": row[31], "remarks": row[32]
              }
            };
            break;
          }
        }
      }
    } else {
      result = { "status": "online" };
    }

    var outputText = callback ? (callback + "(" + JSON.stringify(result) + ")") : JSON.stringify(result);
    var mimeType = callback ? ContentService.MimeType.JAVASCRIPT : ContentService.MimeType.JSON;
    return ContentService.createTextOutput(outputText).setMimeType(mimeType);

  } catch (error) {
    var errObj = { "found": false, "error": error.toString() };
    var outputErr = callback ? (callback + "(" + JSON.stringify(errObj) + ")") : JSON.stringify(errObj);
    return ContentService.createTextOutput(outputErr).setMimeType(callback ? ContentService.MimeType.JAVASCRIPT : ContentService.MimeType.JSON);
  }
}

/**
 * 3. POST REQUEST HANDLER (SAVE / UPDATE DATA IN SHEET)
 * Inserts a new row or updates existing row matching Employee ID / Email
 */
function doPost(e) {
  try {
    var sheet = getTargetSheet();
    sheet.getRange(1, 1, 1, HEADERS.length).setValues([HEADERS]);
    var data = JSON.parse(e.postData.contents);
    
    var empId = (data.employeeId || "").trim().toUpperCase();
    var empEmail = (data.employeeEmail || "").trim().toUpperCase();
    
    var rowValues = [
      data.timestamp || new Date().toLocaleString(),
      data.adminUser || "admin",
      data.date || "",
      data.date2 || "NIL",
      data.date3 || "NIL",
      data.joiningDate || "NIL",
      data.receivedDate || "NIL",
      data.employeeName || "",
      data.employeeEmail || "",
      data.employeeId || "",
      data.department || "",
      data.issuedBy || "",
      data.assetsIssued || "",
      data.laptopBrand || "",
      data.laptopSerial || "",
      data.laptopChargerSerial || "",
      data.headphoneBrand || "",
      data.headphoneSerial || "NIL",
      data.mouseType || "",
      data.mouseSerial || "NIL",
      data.keyboardType || "",
      data.keyboardSerial || "NIL",
      data.mobileBrand || "",
      data.simNumber || "NIL",
      data.mobileSerial || "",
      data.mobileImei || "",
      data.mobileChargerSerial || "",
      data.tabBrand || "",
      data.tabSerial || "",
      data.storageDevice || "NIL",
      data.storageCapacity || "",
      data.storageSerial || "NIL",
      data.remarks || "N/A"
    ];

    var lastRow = sheet.getLastRow();
    var updated = false;
    if (lastRow > 1) {
      var allData = sheet.getRange(2, 9, lastRow - 1, 2).getValues(); // Col I & J (Email & Emp ID)
      for (var i = 0; i < allData.length; i++) {
        var existingEmail = (allData[i][0] || "").toString().trim().toUpperCase();
        var existingId = (allData[i][1] || "").toString().trim().toUpperCase();
        if ((empId && existingId === empId) || (empEmail && existingEmail === empEmail)) {
          var targetRow = i + 2;
          sheet.getRange(targetRow, 1, 1, rowValues.length).setValues([rowValues]);
          updated = true;
          break;
        }
      }
    }

    if (!updated) {
      sheet.appendRow(rowValues);
    }
    
    sheet.autoResizeColumns(1, HEADERS.length);
    return ContentService.createTextOutput(JSON.stringify({ "result": "success", "updated": updated }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ "result": "error", "message": error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

---

## 🛠️ How to Deploy to GitHub

```bash
git init
git add .
git commit -m "Update Skillversity Asset Issuance Portal with Dynamic Storage, Two-Tier Roles & Local Database"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/asset-issuance-form.git
git push -u origin main
```

---

© Skillversity 2026 | IT & OPERATIONS
