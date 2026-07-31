# Skillversity - Asset Issuance Form & HR Administrator Portal

A modern, responsive web application for allocating equipment, issuing assets, generating official PDF reports, auto-fetching previous employee records, and logging data to Google Sheets in real-time.

---

## 🚀 Features

- **HR Administrator Portal Access**: Secured gatekeeper login screen (Default Username: `admin` / Password: `admin123`).
- **Automatic Admin Auto-Logout**: Auto-logs out and redirects to the login screen immediately after PDF generation & data sync.
- **15 Multi-Select Assets**: Grid selection for Laptops, Laptop Charger, Laptop Bag, Mobiles, Mobile Charger, Tablets, Tablet Charger, Pendrive, Hard Disk, Desktop PC, Mouse, Keyboard, Memory Card, Headphones, and **SIM CARD**.
- **Hardware Serial Number Inputs**: Complete hardware specifications and serial numbers (Laptop, Charger, Headphone, Mouse, Keyboard, Mobile, **SIM / Phone Number**, IMEI, Charger, Tablet, Storage).
- **Multi-Issuance Date Tracking**: Supports **1st Issuance Date**, **2nd Issuance Date**, and **3rd Issuance Date** for tracking equipment provided across multiple dates.
- **🔍 Record Lookup & Auto-Populate (`🔍 LOAD RECORD`)**: Enter Employee ID (e.g. `QLI0001`) to automatically fetch previous issuance history from Google Sheets and pre-fill the form for updates.
- **Up to 3 Submissions per Employee**: Allows up to 3 submissions per Employee ID / Email ID.
- **PDF Report Generation**: Instant client-side download of official **1-Page Asset Issuance Report** (`Skillversity_Asset_Report_[Name]_[ID].pdf`) complete with dual signature blocks.
- **Google Sheets Real-time Sync**: Background delivery to Google Sheets Web App endpoint (`Asset Issuance Records` tab).

---

## 📁 Files Included in Repository

```
├── index.html        # Main application page (HR Login, Asset Form, PDF Builder, Modal & Logic)
├── index.css         # Complete CSS Design System & Media Queries
├── logo.png          # Skillversity Official Logo
├── favicon.png       # Web Browser Favicon
└── README.md         # Documentation & Setup Guide
```

---

## 📜 Google Apps Script (32 Columns with Lookup & Update)

Copy and paste this script into your Google Sheets Apps Script editor:

```javascript
var SPREADSHEET_ID = "1G2QmvAjKL1c-oim1BmG-JtAH0-EbaHMvYSGTT9SmzoI";
var SHEET_NAME = "Asset Issuance Records";

var HEADERS = [
  "Timestamp", "Admin User", "1st Issuance Date", "2nd Issuance Date", "3rd Issuance Date", 
  "Employee Joining Date", "Asset Received Date", "Employee Name", "Employee Email", 
  "Employee ID", "Department", "Asset Issued By", "Assets Issued", "Laptop Brand", 
  "Laptop Serial", "Laptop Charger Serial", "Headphone Brand", "Headphone Serial", 
  "Mouse Type", "Mouse Serial", "Keyboard Type", "Keyboard Serial", "Mobile Brand", 
  "SIM / Phone Number", "Mobile Serial", "Mobile IMEI", "Mobile Charger Serial", 
  "Tab Brand", "Tab Serial", "Storage Device", "Storage Capacity", "Remarks"
];

function getTargetSheet() {
  var ss;
  try { ss = SpreadsheetApp.openById(SPREADSHEET_ID); } catch (e) { ss = SpreadsheetApp.getActiveSpreadsheet(); }
  if (!ss) ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(SHEET_NAME);
  if (!sheet) { sheet = ss.getSheets()[0]; try { sheet.setName(SHEET_NAME); } catch(e){} }
  return sheet;
}

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
}

function doGet(e) {
  try {
    var sheet = getTargetSheet();
    if (e && e.parameter && e.parameter.action === "lookup") {
      var query = (e.parameter.empId || e.parameter.empEmail || "").trim().toUpperCase();
      if (!query || sheet.getLastRow() < 2) return ContentService.createTextOutput(JSON.stringify({ "found": false })).setMimeType(ContentService.MimeType.JSON);
      
      var data = sheet.getDataRange().getValues();
      for (var i = data.length - 1; i >= 1; i--) {
        var row = data[i];
        var rowEmail = (row[8] || "").toString().trim().toUpperCase(); 
        var rowId = (row[9] || "").toString().trim().toUpperCase();    
        
        if (query === rowId || query === rowEmail) {
          return ContentService.createTextOutput(JSON.stringify({
            "found": true,
            "record": {
              "timestamp": row[0], "adminUser": row[1], "date": row[2], "date2": row[3], "date3": row[4],
              "joiningDate": row[5], "receivedDate": row[6], "employeeName": row[7], "employeeEmail": row[8],
              "employeeId": row[9], "department": row[10], "issuedBy": row[11], "assetsIssued": row[12],
              "laptopBrand": row[13], "laptopSerial": row[14], "laptopChargerSerial": row[15],
              "headphoneBrand": row[16], "headphoneSerial": row[17], "mouseType": row[18], "mouseSerial": row[19],
              "keyboardType": row[20], "keyboardSerial": row[21], "mobileBrand": row[22], "simNumber": row[23],
              "mobileSerial": row[24], "mobileImei": row[25], "mobileChargerSerial": row[26], "tabBrand": row[27],
              "tabSerial": row[28], "storageDevice": row[29], "storageCapacity": row[30], "remarks": row[31]
            }
          })).setMimeType(ContentService.MimeType.JSON);
        }
      }
      return ContentService.createTextOutput(JSON.stringify({ "found": false })).setMimeType(ContentService.MimeType.JSON);
    }
    return ContentService.createTextOutput(JSON.stringify({ "status": "online" })).setMimeType(ContentService.MimeType.JSON);
  } catch (error) { return ContentService.createTextOutput(JSON.stringify({ "found": false, "error": error.toString() })).setMimeType(ContentService.MimeType.JSON); }
}

function doPost(e) {
  try {
    var sheet = getTargetSheet();
    sheet.getRange(1, 1, 1, HEADERS.length).setValues([HEADERS]);
    var data = JSON.parse(e.postData.contents);
    
    var empId = (data.employeeId || "").trim().toUpperCase();
    var empEmail = (data.employeeEmail || "").trim().toUpperCase();
    
    var rowValues = [
      data.timestamp || new Date().toLocaleString(), data.adminUser || "admin",
      data.date || "", data.date2 || "NIL", data.date3 || "NIL",
      data.joiningDate || "NIL", data.receivedDate || "NIL", data.employeeName || "",
      data.employeeEmail || "", data.employeeId || "", data.department || "", data.issuedBy || "",
      data.assetsIssued || "", data.laptopBrand || "", data.laptopSerial || "", data.laptopChargerSerial || "",
      data.headphoneBrand || "", data.headphoneSerial || "NIL", data.mouseType || "", data.mouseSerial || "NIL",
      data.keyboardType || "", data.keyboardSerial || "NIL", data.mobileBrand || "", data.simNumber || "NIL",
      data.mobileSerial || "", data.mobileImei || "", data.mobileChargerSerial || "", data.tabBrand || "",
      data.tabSerial || "", data.storageDevice || "NIL", data.storageCapacity || "", data.remarks || "N/A"
    ];

    var lastRow = sheet.getLastRow();
    var updated = false;
    if (lastRow > 1) {
      var allData = sheet.getRange(2, 9, lastRow - 1, 2).getValues();
      for (var i = 0; i < allData.length; i++) {
        var existingEmail = (allData[i][0] || "").toString().trim().toUpperCase();
        var existingId = (allData[i][1] || "").toString().trim().toUpperCase();
        if ((empId && existingId === empId) || (empEmail && existingEmail === empEmail)) {
          sheet.getRange(i + 2, 1, 1, rowValues.length).setValues([rowValues]);
          updated = true;
          break;
        }
      }
    }
    if (!updated) sheet.appendRow(rowValues);
    sheet.autoResizeColumns(1, HEADERS.length);
    return ContentService.createTextOutput(JSON.stringify({ "result": "success", "updated": updated })).setMimeType(ContentService.MimeType.JSON);
  } catch (error) { return ContentService.createTextOutput(JSON.stringify({ "result": "error", "message": error.toString() })).setMimeType(ContentService.MimeType.JSON); }
}
```

---

## 🛠️ How to Deploy to GitHub

```bash
git init
git add .
git commit -m "Update Skillversity Asset Issuance Form and HR Admin Portal"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/asset-issuance-form.git
git push -u origin main
```

---

© Skillversity 2026 | IT & OPERATIONS
