# Setting up the Google Sheets backend

This takes about 5 minutes. Once done, every form submission on your landing page will automatically appear as a new row in a Google Sheet — no Google Forms needed.

The page now sends **two kinds of submission** — session **feedback** and **abstracts**. Each payload carries a `type` field, and the script below routes each to its own tab in the same spreadsheet.

## Step 1 — Create a Google Sheet with two tabs

1. Go to https://sheets.google.com and create a new spreadsheet.
2. Name it something like **INTIMATE-Cambridge26 Submissions**.
3. Rename the first tab (bottom-left) to **Feedback**, and in **row 1** add these headers (exactly):

   | A | B | C | D | E |
   |---|---|---|---|---|
   | Timestamp | Name | Email | Selected themes | Suggestions |

4. Add a second tab (the **+** at the bottom-left), name it **Abstracts**, and in **row 1** add these headers (exactly):

   | A | B | C | D | E | F | G |
   |---|---|---|---|---|---|---|
   | Timestamp | Contact email | Title | Authors & affiliations | Presenting author | Format | Abstract |

5. Note the spreadsheet URL — you'll need it in Step 2.

> Tab names must match **Feedback** and **Abstracts** exactly (the script creates them automatically if missing, but it's cleaner to set the headers yourself).

## Step 2 — Create a Google Apps Script

1. In your Google Sheet, go to **Extensions → Apps Script**.
2. Delete any existing code and paste the following:

```javascript
function doPost(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var data = JSON.parse(e.postData.contents);

  if (data.type === 'abstract') {
    var abs = ss.getSheetByName('Abstracts') || ss.insertSheet('Abstracts');
    abs.appendRow([
      data.timestamp || new Date().toISOString(),
      data.email || '',
      data.title || '',
      data.authors || '',
      data.presenting || '',
      data.format || '',
      data.abstract || ''
    ]);
  } else {
    var fb = ss.getSheetByName('Feedback') || ss.insertSheet('Feedback');
    fb.appendRow([
      data.timestamp || new Date().toISOString(),
      data.name || '',
      data.email || '',
      data.themes || '',
      data.suggestions || ''
    ]);
  }

  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Save** (Ctrl+S) and name the project (e.g. "INTIMATE Submissions").

## Step 3 — Deploy as a web app

1. Click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Set:
   - **Description**: INTIMATE feedback collector
   - **Execute as**: Me
   - **Who has access**: Anyone
4. Click **Deploy**.
5. Google will ask you to authorise — click through and allow access to your spreadsheet.
6. Copy the **Web app URL** that appears (it looks like `https://script.google.com/macros/s/AKfyc.../exec`).

## Step 4 — Paste the URL into the landing page

Open `INTIMATE-Cambridge26-feedback.html` and find this line near the bottom:

```javascript
const APPS_SCRIPT_URL = '';
```

Paste your URL between the quotes:

```javascript
const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/AKfyc.../exec';
```

Save the file. That's it — submissions will now go straight to your Google Sheet.

## Downloading as Excel

At any point, open the Google Sheet and go to **File → Download → Microsoft Excel (.xlsx)** to get a local `.xlsx` copy.

## Notes

- The web app URL is public but obscure (long random string). Only people who have the URL can submit data.
- You can share the Google Sheet with Christine or other co-organisers so everyone can see responses in real time.
- If you ever need to update the script, go to **Extensions → Apps Script**, edit the code, then **Deploy → Manage deployments → Edit → New version → Deploy**.
