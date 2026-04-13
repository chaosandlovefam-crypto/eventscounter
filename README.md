# EventsCounter — FamMap Municipality Event Tracker

A Claude skill that automatically counts active/scheduled events for each municipality from the [FamMap Dashboard](https://dashboard.famies.app) and updates a Google Spreadsheet with the latest totals.

---

## What Does This Skill Do?

If you manage events across multiple cities/municipalities on FamMap, this skill saves you hours of manual work. Instead of clicking through each municipality filter one by one and counting events, Claude does it all automatically by:

1. Reading your list of cities from a Google Spreadsheet
2. Querying the FamMap dashboard API for each city's active event count
3. Updating the spreadsheet with fresh numbers
4. Adding a "Last Updated" timestamp

---

## Prerequisites

Before you can use this skill, you need:

1. **Claude Desktop App** (with Cowork mode) — [Download here](https://claude.ai/download)
2. **Claude in Chrome extension** — This lets Claude control your browser. Install it from Chrome Web Store.
3. **A FamMap Dashboard account** — You must be logged into `https://dashboard.famies.app` in Chrome
4. **A Google Spreadsheet** — Where your city names and event counts live

---

## Step-by-Step Setup Guide

### Step 1: Install the Skill

1. Download this repository as a `.zip` file (click the green "Code" button on GitHub, then "Download ZIP")
   - OR clone it: `git clone https://github.com/chaosandlovefam-crypto/eventscounter.git`
2. In Claude Desktop, go to **Settings** > **Skills** (or the skills/plugins section)
3. Click **"Install Skill"** or **"Add Skill"**
4. Point it to the `eventscounter` folder you downloaded

Alternatively, if you're using Cowork mode, you can simply tell Claude:
> "Install the eventscounter skill from https://github.com/chaosandlovefam-crypto/eventscounter.git"

### Step 2: Prepare Your Google Spreadsheet

Your spreadsheet needs to follow this format:

| | A | B | C |
|---|---|---|---|
| **1** | City Name | Total Amount | Last Updated |
| **2** | Botkyrka | | |
| **3** | Danderyd | | |
| **4** | Ekerö | | |
| ... | ... | ... | ... |

- **Column A**: Put your municipality/city names here (one per row, starting from row 2)
- **Column B**: This is where the event counts will go (Claude fills this in)
- **Column C**: "Last Updated" — Claude writes the timestamp here after updating

You can have as many or as few cities as you want. The skill reads whatever is in Column A.

Here's an example with the default 27 Stockholm-area municipalities:

```
Botkyrka, Danderyd, Ekerö, Haninge, Huddinge, Järfälla, Lidingö, 
Nacka, Norrtälje, Nykvarn, Nynäshamn, Österåker, Salem, Sigtuna, 
Södertälje, Sollentuna, Solna, Stockholm, Sundbyberg, Täby, Tyresö, 
Upplands-Bro, Upplands Väsby, Vallentuna, Värmdö, Vaxholm, Uppsala
```

### Step 3: Log Into FamMap Dashboard

1. Open Google Chrome
2. Go to `https://dashboard.famies.app`
3. Log in with your admin credentials
4. Make sure you can see the Events page (click "Events" in the left sidebar)
5. **Keep this tab open** — Claude needs it!

### Step 4: Make Sure Claude in Chrome Is Connected

1. Look for the Claude in Chrome extension icon in your browser toolbar
2. Click it and make sure it shows "Connected"
3. If not connected, click "Connect" and follow the prompts

### Step 5: Run the Skill

Now simply tell Claude in Cowork mode:

> "Update the event counts in my spreadsheet. Here's the link: [paste your Google Spreadsheet URL]"

Or be more specific:

> "Go to this spreadsheet [URL], pick the cities, then check the FamMap dashboard for scheduled events in each city and update the counts."

Claude will:
1. Open your spreadsheet and read the city names
2. Open the FamMap dashboard
3. Set the filter to "Scheduled" events
4. Query each municipality's event count via the API
5. Go back to the spreadsheet and update all the numbers
6. Add the current date/time as "Last Updated"

### Step 6 (Optional): Set Up as a Scheduled Task

If you want to run this regularly, tell Claude:

> "Create a manual scheduled task for this event counting workflow"

This creates a task you can trigger anytime from the "Scheduled" section in Claude's sidebar. Just click "Run now" whenever you want fresh numbers.

---

## Troubleshooting

### "Claude says it can't access the dashboard"
- Make sure you're logged into `https://dashboard.famies.app` in Chrome
- Make sure the Claude in Chrome extension is connected
- Try refreshing the dashboard page

### "API calls are failing with 401"
- Your login session has expired
- Go to `https://dashboard.famies.app`, log in again, and retry

### "Some cities show 0 but I know there are events"
- Double-check the city name spelling in your spreadsheet matches what FamMap uses
- The skill only counts **scheduled/upcoming** events, not ended ones
- Some municipalities genuinely may have 0 upcoming events

### "The numbers in the spreadsheet look wrong"
- Verify by going to the dashboard manually: Events > Filters > Scheduled > select the municipality
- The "Page X of Y" at the bottom shows total pages (each page has 10 events)
- Compare with what Claude reported

### "I can't install the skill"
- Make sure you have Claude Desktop installed (not just the web version)
- The skill needs to be in a folder with the `SKILL.md` file inside it
- Try restarting Claude Desktop after installing

---

## How It Works (Technical Details)

For those curious about what's happening under the hood:

1. **Authentication**: The FamMap dashboard uses JWT Bearer tokens. Claude captures the auth headers by intercepting the browser's XMLHttpRequest calls.

2. **Municipality API**: `https://api.famies.app/admin/municipalities?searchText={city}` — returns municipality IDs for matching names.

3. **Events API**: `https://api.famies.app/admin/events?isEnd=false&municipalityId={id}` — returns event list with a `totalEvents` count. The `isEnd=false` parameter filters to only active/scheduled events.

4. **Spreadsheet Update**: Claude uses browser automation to click cells and type values directly into Google Sheets.

---

## Contributing

Found a bug? Want to add a feature? Feel free to:
1. Fork this repository
2. Make your changes
3. Submit a Pull Request

---

## License

MIT License — use it however you want!

---

Made with Claude by [Famies](https://famies.app)
