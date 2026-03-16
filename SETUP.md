# Fire Safe Landing Page — Setup Guide

Two things to configure: Google Analytics and the Google Sheets form backend.

---

## 1. Google Analytics 4

1. Go to [analytics.google.com](https://analytics.google.com) and create a property
2. Grab your **Measurement ID** (looks like `G-XXXXXXXXXX`)
3. In `landing.html`, find the two instances of `G-XXXXXXXXXX` near the top and replace them with your ID

That's it. You'll see page views automatically, plus a custom `generate_lead` event every time someone submits the address form.

---

## 2. Google Sheets Form Backend (free, no server)

### Step 1: Create the Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet
2. Name it "Fire Safe Leads" (or whatever you like)
3. In Row 1, add these headers: `timestamp | address | email | source | page`

### Step 2: Add the Apps Script

1. In your spreadsheet, go to **Extensions → Apps Script**
2. Delete the placeholder code and paste this:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);

  sheet.appendRow([
    data.timestamp || new Date().toISOString(),
    data.address || '',
    data.email || '',
    data.source || '',
    data.page || ''
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Save** (name the project anything, e.g. "Fire Safe Form")

### Step 3: Deploy as Web App

1. Click **Deploy → New deployment**
2. Click the gear icon and select **Web app**
3. Set:
   - **Execute as**: Me
   - **Who has access**: Anyone
4. Click **Deploy**
5. **Authorize** when prompted (click through the "unsafe" warning — it's your own script)
6. Copy the **Web app URL** it gives you

### Step 4: Paste the URL into landing.html

In `landing.html`, find this line near the top of the `<script>`:

```
const GOOGLE_SHEET_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
```

Replace `YOUR_GOOGLE_APPS_SCRIPT_URL_HERE` with the URL you copied.

### Done!

Every form submission will now add a row to your Google Sheet with the timestamp, address, email (if provided), which form they used (hero vs bottom CTA), and the page URL.

---

## Testing

1. Open `landing.html` in a browser
2. Enter a test address and submit
3. Check your Google Sheet — a new row should appear within a few seconds
4. Check Google Analytics → Realtime → Events to see the `generate_lead` event
