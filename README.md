# Daily Log — Personal calorie and weight PWA

A dependency-free, offline-capable progressive web app for iPhone (also works on desktop). No server/database account required.

## Features

- Select any date; enter or replace morning weight in pounds.
- Add calorie entries with optional meal names; delete entries.
- View daily totals and recent days.
- Local-first persistence using browser localStorage.
- Export Google Sheets-compatible CSV; export and restore full JSON backups.
- Optional manual two-way Google Sheets synchronization via OAuth and Google Sheets API.
- Install to iPhone Home Screen using Safari.

## Run locally

Serve this directory over HTTP (opening `index.html` as a file is not sufficient for PWA behavior):

```sh
python3 -m http.server 8080
```

Visit http://localhost:8080 on your computer. To use on your iPhone, deploy the directory to an HTTPS static host (e.g. GitHub Pages, Netlify, Cloudflare Pages). Avoid deploying your personal data; the app's data lives only in the browser's local storage and Google Sheet, not in the source files.

## Install on iPhone

1. Open your deployed HTTPS URL in **Safari**.
2. Use **Share → Add to Home Screen** and enable **Open as Web App** if shown.
3. Launch **Daily Log** from your Home Screen.

## Configure Google Sheets sync (one time)

1. Visit https://console.cloud.google.com/ and create or select a project.
2. Enable the **Google Sheets API** under APIs & Services → Library.
3. Configure **Google Auth Platform / OAuth consent screen**. Choose **External** for a personal Gmail account if appropriate. If in testing mode, add your Gmail address as a **test user**. The exact console labels may vary.
4. Create an **OAuth client ID**, application type **Web application**.
5. Under **Authorized JavaScript origins**, add the **exact origin** of your deployed site (for example, `https://yourname.github.io`). The origin contains no path. For local development you may add `http://localhost:8080`.
6. Copy the client ID (ends in `.apps.googleusercontent.com`). In Daily Log → Settings, paste it into **OAuth client ID**.
7. Tap **Connect Google**, grant permission, then tap **Sync now**. The first sync creates a new Google Sheet with **Records** and **Daily Summary** tabs. The spreadsheet ID is saved locally.
8. Later, tap **Connect Google** again if the access token has expired, then **Sync now**.

The OAuth scope is `https://www.googleapis.com/auth/drive.file`, intended to grant access only to files the app creates or the user explicitly opens with the app. The app never stores access tokens on disk. No client secret is used or needed for this browser application.

**Existing sheet / new device:** Paste the spreadsheet ID in Settings to select a spreadsheet that was originally created by this app, then connect and sync. The spreadsheet must be accessible to the signed-in Google account. A newly deployed app under a different OAuth client may not automatically be authorized to access a previously created file using `drive.file`; use the same client/app, or export/import a JSON backup.

## Data and sync behavior

- Each meal and each day's weight is a record with a unique ID, timestamp, and deletion marker.
- Sync reads remote records, merges by record ID using the latest `updatedAt`, and writes the merged set back to the sheet. Deleted records remain as tombstones to prevent their reappearance.
- Daily Summary is derived from the Records tab. Edit entries in the app, not directly in Google Sheets; manually changing sheet rows can break the merge format.
- **Not safe for simultaneous editing on multiple devices.** Sync is not transactional: concurrent syncs can overwrite one another. Sync each device before editing and again after editing.
- **Google sign-in may need repeating.** Google OAuth access tokens are memory-only and typically expire. In testing mode, Google's authorization can also expire and require renewed consent.
- If Google sync fails, local entries remain saved. Export JSON regularly as a full backup.
- Browser localStorage can be erased if browser/site data is cleared or the app is deleted. Google Sheets and JSON backups protect against this.
- CSV is a readable flat export. JSON is the recommended restore format, preserving IDs and deletion history.
- Offline support begins after a successful first load over HTTPS. Google sign-in and sync require internet.

## Files

- `index.html` — interface
- `style.css` — styling
- `app.js` — local data and Google sync
- `sw.js` — offline service worker
- `manifest.webmanifest` — install metadata
- `icons/` — Home Screen icons

## Security note

This is a personal-use starter application, not a production-grade multi-user service. Do not publish the Google spreadsheet ID as a credential; it is an identifier, not a password. Do not put OAuth client secrets in frontend code. OAuth consent and access depend on your Google Cloud configuration. Review Google permissions before granting access.
