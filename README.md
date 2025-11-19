# Site Performance & Traffic Analyzer

An n8n workflow that finds websites by platform/keyword, analyzes performance with Google Lighthouse, and gathers traffic data via SimilarWeb. Built for developers, agencies, and researchers.

## What It Does

1. Searches Google for websites matching your criteria
2. Returns up to 30 results (configurable)
3. Analyzes each site with PageSpeed Insights + SimilarWeb traffic data
4. Outputs performance scores, traffic stats, and URLs

## Use Case Examples

**E-commerce**: `sustainable site:myshopify.com`, `handmade site:wordpress.com inurl:product`  
**Website Builders**: `photographer site:wixsite.com`, `portfolio site:webflow.io`  
**Lead Generation**: `"real estate" site:squarespace.com -zillow`, `restaurant "new york"`  
**Research**: `vegan protein site:myshopify.com`, `site:.ca "tech startup"`

[See more search operator examples](#customization)

## Setup

### Prerequisites
- Google Cloud account
- n8n instance
- SimilarWeb browser extension (Chrome/Firefox) - optional, see note below

### Step 1: Google APIs

1. Go to [Google Cloud Console](https://console.cloud.google.com/) and create a project
2. Enable these APIs:
   - [PageSpeed Insights API](https://console.cloud.google.com/apis/api/pagespeedonline.googleapis.com)
   - [Custom Search API](https://console.cloud.google.com/apis/api/customsearch.googleapis.com/metrics)
3. Go to **APIs & Services** > **Credentials** > **CREATE CREDENTIALS** > **API Key**
4. Copy your API key and optionally restrict it to the two APIs above

### Step 2: Custom Search Engine

1. Go to [Programmable Search Engine](https://cse.google.com/all) and click **Add**
2. Enter any placeholder site (e.g., `www.example.com`) and click **Create**
3. Copy your **Search engine ID** (looks like `fc1234567890123456789`)
4. **Important**: Go to **Setup** > **Basics** > Enable **Search the entire web**

### Step 3: SimilarWeb (Optional)

**Note**: The workflow includes hard-coded cookies that access SimilarWeb's extension API endpoint[web:51]. This is an undocumented but publicly accessible API used by their free browser extension[web:51]. In practice, these cookies may continue working indefinitely without rate limiting[web:51][web:83].

**If SimilarWeb stops returning data** (unlikely but possible):

1. Install [SimilarWeb extension](https://www.similarweb.com/corp/extension/)
2. Open DevTools (`F12`), go to `chrome://extensions`, enable **Developer mode**
3. Click **Inspect views: service worker** on the SimilarWeb extension
4. In **Network** tab, trigger the extension and find `api.similarweb.com` request
5. Copy **Cookie** from **Request Headers** and update the **SimilarWeb** node in n8n

### Step 4: Import Workflows

1. Import subworkflows first:
   - `subworkflows/process-lighthouse-data.json`
   - `subworkflows/format-similarweb-response.json`
2. Import main workflow: `main-workflow.json`

### Step 5: Configure in n8n

1. Create credential: **Generic Credential Type** > **Query Auth**
   - Name: `key`
   - Value: Your Google API key from Step 1
2. Assign to **PageSpeed Insights** and **Custom Search** nodes
3. Open **Custom Search** node, find `cx` parameter, replace `YOUR_CX_ID_HERE` with your Search Engine ID

### Step 6: Test

Run the workflow and verify:
- Google Custom Search returns results
- PageSpeed Insights returns data
- SimilarWeb returns traffic stats (or skip if disabled)

## Customization

### Search Query

Default: `sustainable site:myshopify.com`

**To change**: Edit the `q` parameter in the **Google Custom Search** node.

**Search operators**:
- `site:domain.com` - Specific domain
- `intitle:keyword` - Keyword in title
- `inurl:keyword` - Keyword in URL
- `-term` - Exclude term
- `OR` - Either/or matching
- `"exact phrase"` - Exact match
- `site:.ca` - Country TLD

### Results Count

Default: 30 results (3 calls × 10 each). Max: 100 total.

Edit loop/pagination logic to change. Note: Processing 30 sites takes 3-5 minutes.

## API Limits

| API | Free Tier | Notes |
|-----|-----------|-------|
| Google Custom Search | 100 queries/day | Hard limit |
| PageSpeed Insights | 25,000 requests/day | Per site audit |
| SimilarWeb (extension) | Unknown* | *Undocumented endpoint, no observed limits |

## Troubleshooting

**"Invalid API key"**: Verify key in n8n credentials, check restrictions in Google Cloud Console  
**"No results"**: Verify `cx` parameter, ensure "Search entire web" enabled  
**SimilarWeb empty**: Likely working as-is; if needed, refresh cookies (Step 3) or disable node  
**Slow workflow**: Normal—Lighthouse audits take 5-10s per site

## Contributing

PRs welcome! Improvements to search logic or alternative traffic data sources appreciated.

## License

MIT
