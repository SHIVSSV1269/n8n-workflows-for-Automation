# 🛒 AI Price Comparison

Give it a product name. It checks five Indian retailers at once,
reads every result page with an LLM, and tells you where to buy.


## ⚙️ How It Works
1. Form Trigger takes the product, an optional budget, and preferences
2. Code node builds a search URL per retailer from one editable config
3. HTTP Request fetches all five in parallel — failures don't stop the run
4. Code node strips each page to plain text and flags CAPTCHA walls
5. LLM reads each page and returns structured listings
6. LLM compares every offer and picks the best place to buy
7. Result page shows the winner, a ranked table, and any warnings

## 🧠 Why an LLM Instead of CSS Selectors
Scraping five retailers with CSS selectors means five sets of selectors
that break independently every time a site ships a redesign.

Here the page is flattened to text and the model finds the products. It
costs tokens and runs slower, but it survives markup changes — which is
the difference between a workflow you patch weekly and one you don't.

The model is also told to return `null` over a guess. Scraped text puts
ratings, EMI amounts and offer percentages right next to prices, and a
confidently wrong price is worse than a missing one when the output is a
buying decision.

## 🔧 Setup Instructions
1. Import `price-comparison.json` into your n8n
2. Add an LLM credential to both model nodes
3. *(Recommended)* Set a scraping proxy key — see below
4. Save, activate, and open the form URL on the trigger node

## 🌐 The Scraping Problem
Amazon and Flipkart serve a CAPTCHA to datacenter IPs and render prices
in JavaScript, so a direct fetch usually returns nothing usable. The
smaller retailers are more forgiving.

The workflow runs either way. Set a [ScraperAPI](https://scraperapi.com)
key in your environment and all five become reliable:

```bash
setx SCRAPER_API_KEY "your_key"     # Windows, then restart n8n
export SCRAPER_API_KEY="your_key"   # macOS / Linux
```

Unset means direct fetch. The key is read from the environment and never
stored in the workflow file, so this JSON is safe to commit.

## 🏪 Changing Retailers
Everything lives in one array at the top of **Prepare Site Requests**.
`{q}` is replaced with the URL-encoded query:

```js
const SITES = [
  { site: 'Amazon.in', url: 'https://www.amazon.in/s?k={q}' },
  // add or remove freely
];
```

## 📊 Workflow Nodes
| Node | Purpose |
|---|---|
| Form Trigger | Takes product, budget, preferences |
| Prepare Site Requests | Builds one request per retailer |
| Fetch Each Site | Parallel fetch, tolerates failures |
| HTML to Text | Strips markup, detects blocked pages |
| Extract Products | LLM pulls structured listings |
| Collect Offers | Folds all sites into one payload |
| Pick Best Deal | LLM compares and picks a winner |
| Render Result | Builds the results page |
| Show Comparison | Displays it |

## 🧾 Sample Output
```
Best place to buy
Croma — Rs. 24,990
Sony WH-1000XM5 Wireless Noise Cancelling Headphones

Croma is Rs. 2,010 cheaper than Vijay Sales for the same model and
shows it in stock. Amazon and Flipkart returned no usable data, so
this comparison covers three of five retailers.

⚠ Worth checking
 • Vijay Sales listing may be a 2022 variant — confirm before buying
```

## ⚠️ Notes
- Prices change constantly. Always confirm on the retailer page.
- One LLM call per retailer plus one for the verdict, so ~6 per search.
  A smaller model handles the extraction step fine if cost matters.
- Built for Indian retailers, but the config is just URLs — swap them
  for any region.
