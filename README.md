# Skillversity - Asset Issuance Form & Portal

A modern, responsive web application for allocating equipment, issuing assets, generating official PDF reports, and logging records to Google Sheets in real-time.

## 🚀 Features
- **Modern UI/UX**: Built with Skillversity brand design system (Royal Blue & Electric Orange).
- **Responsive Layout**: Optimized for both Desktop and Mobile viewports.
- **14 Multi-Select Assets**: Grid selection for Laptops, Mobiles, Tablets, Peripherals, and Storage Devices.
- **Hardware Serial Number Inputs**: Complete hardware specification recording (Laptop Serial, Charger Serial, IMEI, etc.).
- **PDF Report Generation**: Instant client-side download of official **1-Page Asset Issuance Report** (`Skillversity_Asset_Report_[Name]_[ID].pdf`) complete with dual signature blocks.
- **Google Sheets Real-time Sync**: Automatic background delivery to Google Sheets Web App endpoint (`Asset Issuance Records` tab).
- **Duplicate Prevention**: Prevents double submissions per Employee ID / Email ID with a modern custom popup modal.
- **Auto-Reset**: Resets form state 1.8 seconds after submission for seamless next-entry handling.

## 📁 Files Included in Git Repository
```
├── index.html        # Main application page (Form, PDF Builder, Modal & Logic)
├── index.css         # Complete CSS Design System & Media Queries
├── logo.png          # Skillversity Official Logo
├── favicon.png       # Web Browser Favicon
└── README.md         # Documentation & Setup Guide
```

## 🛠️ How to Deploy to GitHub & GitHub Pages

### Option A: Using Git Commands
```bash
git init
git add .
git commit -m "Initial commit - Skillversity Asset Issuance Portal"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/asset-issuance-form.git
git push -u origin main
```

### Option B: Direct Drag & Drop on GitHub.com
1. Go to [GitHub.com](https://github.com) and create a **New Repository**.
2. Click **uploading an existing file**.
3. Select and drag all 4 files (`index.html`, `index.css`, `logo.png`, `favicon.png`) into GitHub.
4. Click **Commit changes**.

---
© Skillversity 2026 | IT & OPERATIONS
