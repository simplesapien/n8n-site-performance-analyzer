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

## Quick Start

### Import Workflows

**Option A: From URL (Recommended)**

1. In n8n, click **⋮** (three dots menu) → **Import from URL**
2. Import subworkflows first (paste these URLs one at a time):
   ```
   https://raw.githubusercontent.com/simplesapien/n8n-site-performance-analyzer/main/subworkflows/google-serp-urls.json
   ```
   ```
   https://raw.githubusercontent.com/simplesapien/n8n-site-performance-analyzer/main/subworkflows/process-lighthouse-data.json
   ```
3. Then import main workflow:
   ```
   https://raw.githubusercontent.com/simplesapien/n8n-site-performance-analyzer/main/main-workflow.json
   ```

**Option B: Download Files**

Download the repo and import via **⋮** → **Import from File** for each JSON.

---

## Setup

### Prerequisites
- Google Cloud account
- n8n instance
- SimilarWeb browser extension (Chrome/Firefox) – optional, see note below

### Step 1: Google APIs

1. Go to [Google Cloud Console](https://console.cloud.google.com/) and create a project
2. Enable these APIs:
   - [PageSpeed Insights API](https://console.cloud.google.com/apis/api/pagespeedonline.googleapis.com)
   - [Custom Search API](https://console.cloud.google.com/apis/api/customsearch.googleapis.com/metrics)
3. Go to **APIs & Services** > **Credentials** > **CREATE CREDENTIALS** > **API Key**
4. Copy your API key and optionally restrict it to the two APIs above

### Step 2: Custom Search Engine

1. Go to [Programmable Search Engine](https://programmablesearchengine.google.com/) and click **Add**
2. Enter any placeholder site (e.g., `www.example.com`) and click **Create**
3. Copy your **Search engine ID** (looks like `fc1234567890123456789`)
4. **Important**: Go to **Setup** > **Basics** > Enable **Search the entire web**

### Step 3: SimilarWeb (Optional)

**Note:** The workflow includes hard-coded cookies for SimilarWeb's extension API endpoint. In practice, these cookies may keep working without rate limiting.

**If SimilarWeb stops returning data**:

1. Install [SimilarWeb extension](https://www.similarweb.com/corp/extension/)
2. Open DevTools (`F12`) and go to `chrome://extensions`, enable **Developer mode**
3. Click **Inspect views: service worker** on the SimilarWeb extension
4. In **Network** tab, trigger the extension and find a request to `api.similarweb.com`
5. Copy **Cookie** from **Request Headers** and update the **SimilarWeb** node in n8n

**Alternative:** Remove the SimilarWeb node entirely if you only need performance data.

### Step 4: Link Subworkflows (Manual Step)

After importing, you need to manually connect the subworkflows to the main workflow:

1. Open the **main-workflow** in n8n
2. Find the **Execute Workflow** nodes (should be two):
   - **Call Google SERP** node
   - **Call PageSpeed API** node
3. For each Execute Workflow node:
   - Click the node
   - In the **Workflow** dropdown, select the corresponding subworkflow:
     - For **Call Google SERP**: Select `Google SERP`
     - For **Call PageSpeed API**: Select `PageSpeed API Call`
4. **Configure workflow inputs** for the **Call PageSpeed API** node:
   - After selecting the workflow, go to **Workflow Inputs** section
   - Set the input mapping to pass data from the previous node:
     - Field name: `urls` (or as defined in the subworkflow)
     - Value: `{{ $json.urls }}` (maps the URL data from Google SERP)
5. Save the workflow

**Why?** Subworkflow references use per-instance IDs which can't be preserved on export. Manual linking and input configuration ensures correct execution.

### Step 5: Configure in n8n

1. After importing, nodes with ⚠️ icons need credentials assigned
2. Create credential: **Generic Credential Type** > **Query Auth**
   - Name: `key`
   - Value: Your Google API key from Step 1
3. Assign to **PageSpeed Insights** and **Custom Search** nodes
4. Open **Custom Search** node, find `cx` parameter, replace `YOUR_CX_ID_HERE` with your Search Engine ID from Step 2

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
- `site:domain.com` – Specific domain
- `intitle:keyword` – Keyword in title
- `inurl:keyword` – Keyword in URL
- `-term` – Exclude term
- `OR` – Either/or matching
- `"exact phrase"` – Exact match
- `site:.ca` – Country TLD

### Results Count

Default: 30 results (3 calls × 10 each). Max: 100 total.

Edit loop/pagination logic to change. Note: Processing 30 sites takes 3–5 minutes.

## API Limits

| API | Free Tier | Notes |
|-----|-----------|-------|
| Google Custom Search | 100 queries/day | Hard limit |
| PageSpeed Insights   | 25,000 requests/day | Per site audit |
| SimilarWeb (extension) | Unknown* | *Undocumented endpoint, no observed limits |

## Troubleshooting

**"Invalid API key":** Verify key in n8n credentials, check restrictions in Google Cloud Console  
**"No results":** Verify `cx` parameter, ensure "Search entire web" enabled  
**"Workflow input missing":** Check Step 4 – ensure workflow inputs are configured in Execute Workflow nodes  
**SimilarWeb empty:** Likely working as-is; if needed, refresh cookies (Step 3) or disable node  
**Slow workflow:** Normal—Lighthouse audits take 5–10s per site

## Contributing

PRs welcome! Improvements to search logic or alternative traffic data sources appreciated.

## License

MIT
