<!-- Landing README for a public GitHub repo. The Actor runs on Apify; this repo is for discovery and issues. -->
[![Run on Apify](https://img.shields.io/badge/Run%20on-Apify-0d9488?style=for-the-badge)](https://apify.com/rsiddiq95/google-sheets-sync)

# Google Sheets Sync

Send the results of any Apify Actor straight to Google Sheets, and keep them up to date without duplicates.

- ➕ **Append** new rows at the bottom.
- 🔁 **Replace** the whole tab with fresh data.
- 🔑 **Upsert by key.** Update rows that already exist (matched by a column such as `url` or `id`) and add only the new ones. Perfect for daily price trackers and lead lists.
- 📥 **Read** a sheet back into an Apify dataset, including public sheets without any setup.

## Why use this Google Sheets integration?

- **No duplicates.** Upsert mode updates existing rows instead of adding copies, and keeps the columns you added by hand (notes, statuses) untouched.
- **Your layout is kept.** Existing columns never move. New fields are added as new columns on the right.
- **Built for large datasets.** Rows are added in batches under Google's recommended request size, instead of rewriting the whole sheet each run.
- **Doesn't break on Google's limits.** When Google's per-minute quota is reached, the Actor waits and retries automatically using Google's recommended backoff.
- **Safe with scraped data.** Values are written as plain text by default, so text that starts with `=` can never run as a formula in your sheet.
- **Clear errors.** If something is wrong, for example the sheet isn't shared, the run tells you exactly what to fix.

## How to send Apify data to Google Sheets

### 1. Create a Google service account key (one time, about 3 minutes)

1. Open [Google Cloud Console](https://console.cloud.google.com/) and create a project (or pick one).
2. Go to **APIs & Services → Library**, search for **Google Sheets API**, and click **Enable**.
3. Go to **APIs & Services → Credentials → Create credentials → Service account**. Give it any name and click **Done**.
4. Open the new service account, go to **Keys → Add key → Create new key → JSON**. A `.json` file downloads.
5. Open the file in a text editor and copy everything.

Your key is stored encrypted on Apify and is only used to talk to Google.

### 2. Share your spreadsheet with the service account

In Google Sheets, click **Share** and add the service account's email (it looks like `name@project.iam.gserviceaccount.com`, shown in the key file as `client_email`) as an **Editor**.

### 3. Run the Actor

1. Paste the **spreadsheet URL** and the **service account key**.
2. Choose a **mode**.
3. Enter the **dataset ID** you want to send (shown on any run's Storage tab), or paste **raw JSON data**.
4. Click **Start**.

### Run it automatically after another Actor

Open the Actor whose results you want in Sheets, go to **Integrations → Add integration → Apify Actor**, choose **Google Sheets Sync**, and set **Dataset ID** to `{{resource.defaultDatasetId}}`. Every time that Actor finishes, its results go to your sheet.

## Input example

```json
{
    "mode": "upsert",
    "spreadsheet": "https://docs.google.com/spreadsheets/d/your-sheet-id/edit",
    "sheetName": "Prices",
    "datasetId": "{{resource.defaultDatasetId}}",
    "keyColumns": ["url"],
    "omitFields": ["html"],
    "serviceAccountKey": "{ ...contents of your key file... }"
}
```

## Output

After each run, the **Output** tab shows a summary:

```json
{
    "mode": "upsert",
    "spreadsheetTitle": "Competitor prices",
    "sheetName": "Prices",
    "appended": 12,
    "updated": 188,
    "columns": 9,
    "duplicatesInInput": 0,
    "apiRequests": 6,
    "spreadsheetUrl": "https://docs.google.com/spreadsheets/d/your-sheet-id/edit#gid=0"
}
```

In **Read** mode, each sheet row becomes a dataset item, using the header row as field names.

## How nested data is written

Nested objects become columns joined with `/`, the same as Apify's CSV export (`address/city`). Lists of plain values are joined with commas (`tags: "red, blue"`), and lists of objects are written as JSON text.

## Pricing

You pay a small fee per row written or read. See the **Pricing** tab for current prices.

## Limits

- Google allows up to 20 million cells per spreadsheet, and 50,000 characters per cell. Longer values are shortened and marked `[truncated]`.
- Google allows 60 write requests per minute per service account. The Actor batches rows, so most runs need only a few requests, and it waits automatically when the quota is reached.
- Upsert reads the whole tab to find matching rows, so it works best for tabs under a few hundred thousand rows.

## Works well with

- **[Google Ads Transparency Scraper](https://apify.com/rsiddiq95/google-ads-transparency-scraper):** keep a live sheet of competitors' Google ads and ad copy (upsert by `creativeId`).
- **[Google Trends Scraper](https://apify.com/rsiddiq95/google-trends-scraper):** log search interest for your keywords to a sheet every week.

## FAQ

**Can I write formulas or dates?**
Turn on **Let Google interpret values**. Text like `2026-01-31` becomes a date and `=SUM(A:A)` becomes a formula. Leave it off for scraped data.

**Can I read a sheet without a key?**
Yes. If the sheet is shared as **Anyone with the link**, choose **Read** mode and turn on **Public sheet**.

**My run says it has no access.**
Share the spreadsheet with the `client_email` from your key file as an Editor, then run again.

## Links

- Apify Store page: https://apify.com/rsiddiq95/google-sheets-sync
- Questions and bug reports: open an issue here or on the Actor's Issues tab.
