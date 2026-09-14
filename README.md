# Google Calendar Board — a custom HTML app for Yodeck

A single self-contained HTML file that shows a scrolling Google Calendar on a Yodeck screen.
Every option lives in the URL query string, so **one hosted copy of the file can drive as many
different boards as you like** — a lobby "This Week" board, a hallway room-booking board, a
staff-only board — each just a different URL in Yodeck.

The file has two modes:

| Open it as | You get |
|---|---|
| `yodeck-google-calendar.html` (no query string) | The **setup screen**: a form with a live preview that builds your display URL |
| `yodeck-google-calendar.html?cal=…&key=…` | The **display**: the scrolling calendar board |
| `yodeck-google-calendar.html?demo=1` | The display with **sample events** — good for checking styling before Google is wired up |

---

## 1. Make the calendar readable

For each Google Calendar you want on the screen:

1. Google Calendar → hover the calendar in the left sidebar → **⋮ → Settings and sharing**
2. Under **Access permissions for events**, tick **Make available to public** and set the dropdown
   to **See all event details**. (If you only grant "See only free/busy", the board will show
   "busy" instead of event names.)
3. Scroll to **Integrate calendar** and copy the **Calendar ID**. It looks like
   `c_a1b2c3…@group.calendar.google.com`, or for a person's primary calendar just their address,
   e.g. `office@ctkcary.com`.

> If a calendar can't be made public, this approach won't reach it — an API key can only read
> public calendars. The alternative is a small server-side proxy holding a service account, which
> is a bigger project; say the word if you need it.

## 2. Create a Google API key

1. Go to <https://console.cloud.google.com/> and create a project (e.g. "CTK Signage").
2. **APIs & Services → Library →** search **Google Calendar API →** **Enable**.
3. **APIs & Services → Credentials → Create credentials → API key.** Copy it.
4. Click the new key and **restrict it** — this matters, because the key is visible in the display URL:
   - **API restrictions:** *Restrict key* → check **Google Calendar API** only.
   - **Application restrictions:** *Websites* → add the domain you'll host the file on,
     e.g. `https://signage.ctkcary.com/*`.

A key restricted this way can do exactly one thing: read calendars that are already public. That's
low risk, but still treat the display URL as semi-private — don't post it publicly.

> If the board shows a `403 requests from referer … are blocked` error after adding the website
> restriction, the Yodeck player isn't sending a referrer the way Google expects. Drop back to
> *Application restrictions: None* and keep the API restriction — the key remains read-only.

## 3. Host the file

Yodeck plays a **URL**, so the file needs to live somewhere reachable over HTTPS. Any of these work:

- **Your church website** — drop it in a folder, e.g. `https://ctkcary.com/signage/calendar.html`
- **Netlify Drop** (<https://app.netlify.com/drop>) — drag the file onto the page, get a URL in seconds, free
- **GitHub Pages** — commit the file to a repo, enable Pages
- Any static host (Cloudflare Pages, S3, your own IIS/Apache box)

It's one file with no build step and no dependencies, so "upload it" is the whole deployment.

## 4. Build your URL

Open the hosted file with **no query string** — you'll get the setup screen. Fill in the API key
and calendar ID(s), adjust the options while watching the preview, then click **Copy URL**.

## 5. Add it to Yodeck

1. In Yodeck: **Media → Add Media → Web Page**
2. Paste the display URL
3. Recommended settings:
   - **Rendering engine:** Chromium *(default — required)*
   - **Auto-adjust Zoom:** **OFF**, Zoom factor **100%**. The board sizes itself from the screen
     height, so letting Yodeck rescale the page will double-scale the text.
   - **Default Duration:** whatever suits your playlist (60–120 s is typical). If this is a
     full-time board, put it in a Layout region or a single-item playlist.
   - **Refresh interval:** leave off. The app re-fetches Google on its own (`refresh` parameter,
     10 minutes by default) without ever blanking the screen.
   - **Fallback image:** optional — shown if the player loses internet.
4. Add the Web Page to a Playlist / Layout and push to the screen.

---

## Parameter reference

Everything below is a URL query parameter. Omit a parameter to accept its default. The setup
screen writes these for you, but they're documented so you can tweak a live URL by hand.

### Calendars and data

| Parameter | Default | What it does |
|---|---|---|
| `cal` | — | **Required.** Comma-separated calendars. Each entry is `id\|label\|hexcolor` — e.g. `office@ctkcary.com\|Church\|ffb340,youth@ctkcary.com\|Youth\|4fa8ff`. Label and color are optional. The label shows as a badge when you list more than one calendar; the color tints that calendar's left edge. |
| `key` | — | **Required.** Your Google API key. |
| `tz` | browser's zone | IANA time zone, e.g. `America/New_York`. Set it explicitly so the board is right even if the player's clock isn't. |
| `days` | `7` | How many days to show, counting today. `1` = today only. |
| `max` | `150` | Hard cap on rows drawn. |
| `q` | — | Only show events whose title or location contains one of these comma-separated strings. |
| `ex` | — | Hide events containing any of these comma-separated strings — handy for `HOLD, Tentative, Setup`. |
| `refresh` | `10` | Minutes between calendar re-fetches. |

### Header

| Parameter | Default | What it does |
|---|---|---|
| `title` | `Upcoming Events` | Heading text. |
| `sub` | — | Optional second line under the heading. |
| `logo` | — | Public `https://` image URL, shown left of the heading. |
| `clock` | `1` | `1`/`0` — live clock and date in the top right. |
| `h24` | `0` | `1` for 24-hour time. |
| `footer` | `1` | `1`/`0` — bottom status line (calendar names, last-updated time, any errors). |

### What each row shows

| Parameter | Default | What it does |
|---|---|---|
| `group` | `1` | `1` = group rows under **TODAY / TOMORROW / Wednesday** headings. `0` = flat list with a date stamp on each row. |
| `showLoc` | `1` | `1`/`0` — show the location / room line. |
| `locSrc` | `auto` | `auto` (location, falling back to booked room), `location`, `rooms`, or `both`. See the note below. |
| `shortLoc` | `0` | `1` = trim the location to the text before the first comma, so a full street address becomes just the building name. |
| `allDay` | `1` | `1`/`0` — include all-day events. |
| `allDayLabel` | `All Day` | Text shown in the time column for all-day events. |
| `expand` | `1` | `1` = a multi-day all-day event appears on every day it covers. `0` = start day only. |
| `badge` | `1` | `1`/`0` — calendar-name badge (only appears when more than one calendar is listed). |
| `empty` | `No events scheduled` | Message shown when nothing matches. |

**About rooms:** Google exposes booked room resources as *attendees* on the event. Public
calendars read with an API key usually don't return attendees, so in practice the **Location**
field is what reaches the screen. The most reliable setup is to put the room name in each event's
Location field; `locSrc=auto` then shows it, and still falls back to the room resource on the
Workspace calendars that do expose them.

### Motion

| Parameter | Default | What it does |
|---|---|---|
| `speed` | `26` | Scroll speed in pixels per second on a 1080p screen (it scales with the display, so a 4K screen scrolls proportionally). **`0` disables scrolling** — the list just sits still, which is right when everything fits. |
| `pause` | `4` | Seconds the list holds still at the top of each pass. |

If the list fits on screen, the board never scrolls — no pointless creeping. When it doesn't fit,
it loops seamlessly, and a refresh that brings in new events waits for the loop to come back
around before swapping content, so the screen never jumps mid-scroll.

### Styling

| Parameter | Default | What it does |
|---|---|---|
| `bg` | `0b0f14` | Background color, hex **without** the `#`. |
| `panel` | `151d27` | Event row background. |
| `fg` | `ffffff` | Primary text. |
| `muted` | `94a7bd` | Secondary text (times, locations). |
| `accent` | `ffb340` | Accent — day headings, all-day label, rules, "happening now" outline. |
| `font` | `Inter` | Any Google Fonts family name, or `system` to skip the web font. |
| `sc` | `1` | Text scale multiplier. Try `1.2` for a screen people read from across a lobby, `0.85` to fit more rows. |
| `density` | `comfortable` | `comfortable` or `compact` (tighter padding). |
| `zebra` | `1` | `1`/`0` — alternating row shading. |

A light theme, for example:
`&bg=ffffff&panel=f2f5f9&fg=0b0f14&muted=5a6b7d&accent=b8451f`

---

## Notes and behavior

- **Portrait works.** Everything sizes off viewport height, so a 1080×1920 portrait screen is fine
  with no extra settings.
- **Happening now** events get a subtle accent outline.
- **Midnight rollover** is handled — "Today" becomes the new day and the board re-fetches, no
  reload needed.
- **If Google is unreachable**, the board keeps showing the last good data and notes the error
  quietly in the footer rather than going blank.
- **Recurring events** are expanded automatically (`singleEvents=true`), so a weekly Bible study
  shows on each of its dates.
- **No cookies, no storage, no external dependencies** other than the Google Fonts stylesheet —
  set `font=system` if your network blocks it.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| Footer says `… : Not Found` | Wrong calendar ID, or the calendar isn't public. |
| Footer says `… : API key not valid` / `403` | Key is wrong, the Calendar API isn't enabled on the project, or the referrer restriction is blocking the player. |
| Events show as "busy" with no title | Calendar is shared as free/busy only — change it to *See all event details*. |
| Text is comically large or small on the screen | Yodeck's **Auto-adjust Zoom** is on. Turn it off and use the `sc` parameter instead. |
| Times are off by an hour or more | Set `tz` explicitly to `America/New_York`. |
| Nothing scrolls | Everything already fits, or `speed=0`. |
