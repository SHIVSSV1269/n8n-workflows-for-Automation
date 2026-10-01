# 📰 Dev Daily Briefing

One URL that returns a live dashboard: the Hacker News stories that match
your interests, this week's fastest-rising GitHub repos, a 3-day weather
forecast, and exchange rates — four APIs fetched in parallel and merged
into a single page.

**No API keys, no credentials.** Import it, activate it, open the URL.

## 📸 Preview
[Add your dashboard screenshot here]

## ⚙️ How It Works
1. **Webhook** receives a GET request; query parameters override the defaults
2. **Config** builds every API URL from those settings
3. Four branches run **in parallel**:
   - **Hacker News** — fetch the top story ids, fan out one request per story
     (batched 10 at a time), then rank by points × topic matches
   - **GitHub** — search repos created in the last 7 days, sorted by stars
   - **Weather** — current conditions plus a 3-day forecast from Open-Meteo,
     with rain and heat alerts
   - **Exchange rates** — live rates against your base currency
4. **Merge** waits for all four branches and combines them
5. **Build Dashboard** renders a responsive HTML page (light and dark mode)
6. **Respond to Webhook** returns it straight to the browser

## 🛡️ Built to Degrade, Not Break
Every HTTP node continues on error and retries once. Every shaping node
emits exactly one tagged item whether its API answered or not, so the Merge
always receives all four inputs and never hangs waiting for a dead branch.

If an API is down, its card says so and the rest of the page renders
normally. The header shows how many of the four sources are live.

## 🔧 Setup
1. Import `dev-briefing.json` into n8n
2. Save and **Activate**
3. Open `http://localhost:5678/webhook/dev-briefing`

## 🎛️ Customise with the URL
| Parameter | Default | Example |
|---|---|---|
| `city` | Chennai | `city=Bangalore` |
| `lat`, `lon` | 13.0827, 80.2707 | `lat=12.97&lon=77.59` |
| `topics` | ai, llm, python, java, security, … | `topics=rust,kubernetes,startup` |
| `base` | USD | `base=EUR` |
| `hn` | 30 stories scanned (5–60) | `hn=50` |

```
/webhook/dev-briefing?city=Bangalore&lat=12.97&lon=77.59&topics=rust,ai&base=EUR
```

## 📊 Workflow Nodes
| Node | Purpose |
|---|---|
| Briefing Webhook | Entry point, GET request |
| Config | Reads query params, builds API URLs |
| HN Top Stories → Pick Top Stories → Fetch Story → Rank Stories | Fan-out over story ids, rank by interest |
| GitHub Rising Repos → Shape Repos | New repos by stars |
| Weather Forecast → Shape Weather | Forecast plus alerts |
| Exchange Rates → Shape Rates | Live currency rates |
| Merge Sections | Joins the four branches |
| Build Dashboard | Renders the HTML page |
| Return HTML | Sends it to the browser |

## 🌐 APIs Used
All free, all keyless:
[Hacker News](https://github.com/HackerNews/API) ·
[GitHub Search](https://docs.github.com/en/rest/search) ·
[Open-Meteo](https://open-meteo.com) ·
[ExchangeRate-API](https://www.exchangerate-api.com/docs/free)

GitHub allows 10 unauthenticated search requests per minute — plenty for
personal use, but don't hammer the URL in a loop.
