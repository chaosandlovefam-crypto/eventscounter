---
name: eventscounter
description: "Count scheduled/active events per municipality from the FamMap dashboard (dashboard.famies.app) and update a Google Spreadsheet with the latest totals. Use this skill whenever the user mentions updating event counts, checking live events across cities, syncing FamMap dashboard data to a spreadsheet, or anything related to municipality event tracking on FamMap. Also trigger when the user says 'update events', 'event count', 'city events', 'municipality events', 'fammap events', or 'weekly events update'."
---

# EventsCounter — FamMap Municipality Event Tracker

This skill counts the number of active/scheduled (not ended) events for each municipality on the FamMap dashboard and updates a Google Spreadsheet with those counts.

## Prerequisites

Before running this skill, the user must have:

1. **Chrome browser logged into FamMap dashboard** at `https://dashboard.famies.app`
2. **Access to the Google Spreadsheet** they want to update (must be open/accessible in Chrome)
3. **Browser automation tools** (Claude in Chrome / MCP browser tools) available

## Spreadsheet Format

The target Google Spreadsheet should follow this structure:

| Column A | Column B | Column C |
|----------|----------|----------|
| City Name | Total Amount | Last Updated |
| Botkyrka | 48 | |
| Danderyd | 1 | |
| ... | ... | |

- **Column A**: Municipality/city names (rows 2 onward, row 1 is header)
- **Column B**: Event count numbers (this is what gets updated)
- **Column C**: "Last Updated" header — timestamp goes in the row after the last city

## Step-by-Step Workflow

### Step 1: Get the Spreadsheet URL and City List

Ask the user for their Google Spreadsheet URL if not already provided. Open the spreadsheet in Chrome and read all city names from Column A. Note which row each city is in (typically rows 2 through N).

### Step 2: Navigate to FamMap Dashboard Events Page

1. Open `https://dashboard.famies.app/events` in Chrome
2. Click the **"Filters"** button to expand the filter panel
3. Click **"Scheduled"** under Event Status — this filters to only active/upcoming events (not ended ones)

### Step 3: Capture the API Auth Headers

The FamMap dashboard uses an API at `api.famies.app` with Bearer token authentication. To get the auth headers:

1. Set up an XMLHttpRequest header interceptor in the browser console:

```javascript
const origSetHeader = XMLHttpRequest.prototype.setRequestHeader;
window._xhrHeaders = {};
XMLHttpRequest.prototype.setRequestHeader = function(name, value) {
  if (!window._xhrHeaders[name]) window._xhrHeaders[name] = value;
  return origSetHeader.apply(this, arguments);
};
```

2. Trigger any page interaction that makes an API call (click a pagination button, change a filter, etc.)
3. Read the captured headers from `window._xhrHeaders` — you need:
   - `Authorization`: Bearer token
   - `Accept`: application/json
   - `Platform`: web
   - `Version`: 0.0.0

### Step 4: Query Event Counts via API

For each city, make two API calls:

**4a. Get Municipality ID:**
```
GET https://api.famies.app/admin/municipalities?page=1&limit=100&searchText={cityName}
```
- The response contains `data.municipalities[]` array
- Each municipality object has `municipalityId` (NOT `id`), `name`, and `region`
- Find the matching municipality by name

**4b. Get Event Count:**
```
GET https://api.famies.app/admin/events?page=1&limit=1&isEnd=false&municipalityId={municipalityId}
```
- `isEnd=false` filters to only active/scheduled events
- The response contains `data.totalEvents` — this is the total count you need
- If no events exist, `totalEvents` will be 0

Use synchronous XMLHttpRequest with the captured auth headers for efficiency — this allows querying all cities in a single JavaScript execution block rather than waiting for async callbacks.

**Example JavaScript for all cities at once:**

```javascript
const headers = {
  'Accept': 'application/json, text/plain, */*',
  'Authorization': window._xhrHeaders['Authorization'],
  'Platform': 'web',
  'Version': '0.0.0'
};

function apiGet(url) {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url, false);
  Object.entries(headers).forEach(([k,v]) => xhr.setRequestHeader(k,v));
  xhr.send();
  return JSON.parse(xhr.responseText);
}

const cities = ['Botkyrka', 'Danderyd', ...]; // from spreadsheet

const results = {};
for (const city of cities) {
  const munData = apiGet('https://api.famies.app/admin/municipalities?page=1&limit=100&searchText=' + encodeURIComponent(city));
  const mun = (munData.data?.municipalities || []).find(m => m.name?.toLowerCase().includes(city.toLowerCase()));
  if (mun) {
    const evData = apiGet('https://api.famies.app/admin/events?page=1&limit=1&isEnd=false&municipalityId=' + mun.municipalityId);
    results[city] = evData.data?.totalEvents || 0;
  } else {
    results[city] = 0;
  }
}
```

### Step 5: Update the Google Spreadsheet

1. Switch to the Google Spreadsheet tab in Chrome
2. Click on cell **B2** (first city's Total Amount cell)
3. Type each count value followed by Enter to move down to the next row
4. All old values get replaced — cities with 0 events get 0 written

### Step 6: Add the Timestamp

1. After updating all city counts, click on the cell in Column C in the row immediately after the last city (e.g., if cities go from row 2 to row 28, click C29)
2. Type the current date and time in format: `YYYY-MM-DD HH:MM`
3. Press Enter to save

### Step 7: Verify

Scroll through the spreadsheet to confirm:
- All cities have updated numbers in Column B
- The timestamp is visible in Column C
- No cells were accidentally shifted or overwritten

## Important Notes

- **Only scheduled events count** — the API filter `isEnd=false` ensures ended/expired events are excluded
- **0 is a valid count** — some municipalities may have no active events, and that's fine
- **Auth tokens expire** — if API calls start failing with 401, the user needs to refresh the dashboard page to get a new token
- **Municipality search is fuzzy** — the API search matches partial names, so pick the closest match from results
- The default 27 municipalities are Stockholm-area cities, but this workflow works for any set of municipalities the user has in their spreadsheet
