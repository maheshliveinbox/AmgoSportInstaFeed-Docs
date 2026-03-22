# Feature 03 — Shopify Product Integration

## Overview

Every **Sunday at 6:00 AM AEST**, Workflow C syncs the active product catalogue from the AMGO Sports Shopify store into the PostgreSQL database. On **Product Spotlight days** (Monday), Workflow A reads this live product data and builds the post around a **real product** — using the actual Shopify product image, name, price, and product page URL — making posts directly shoppable.

---

## Why This Matters

Generic AI imagery for product posts is good. **Real product photos with real prices** are far more effective:

- **Trust**: Customers see exactly what they'll buy
- **Conversions**: Caption can reference actual price → drives purchase intent
- **Authenticity**: Posts look like genuine retailer content, not AI stock imagery
- **SEO**: Product name + brand in caption helps Instagram hashtag discovery

---

## How It Works

### Workflow C — Shopify Product Sync (Weekly)

**Runs**: Every Sunday at 6:00 AM AEST

```
Schedule Trigger
  → Shopify API: GET /admin/api/2024-01/products.json?limit=50&status=active
  → Filter: products with at least one image
  → Code: normalize product data
  → Dashboard API (POST /api/products): overwrite/upsert products in Database
  → Telegram: "✅ Product library updated: {n} products synced"
```

**Shopify API call:**
```
GET https://{store}.myshopify.com/admin/api/2024-01/products.json
Headers:
  X-Shopify-Access-Token: {SHOPIFY_ACCESS_TOKEN}
Query:
  limit=50
  status=active
  fields=id,title,vendor,variants,images,handle
```

**Data normalised into the Database:**
```
Product ID | Title             | Vendor | Price  | Image URL              | Product URL                     | Category | In Stock | Synced At
7123456789 | SS Ton Master 5.0 | SS     | 389.00 | https://cdn.shopify... | https://amgosports.com.au/p/... | Bats     | TRUE     | 2026-03-15
```

---

### Workflow A — Product Spotlight Integration

On **Monday** (Product Spotlight day) and any time the dashboard's **"Feature this product"** button is pressed:

#### 1. Random Product Selection
```javascript
// Read Products from Dashboard API
// Filter: inStock = TRUE
// Select: random product (or the specific productId if triggered from dashboard)
const product = products[Math.floor(Math.random() * products.length)];
```

#### 2. GPT-4o Prompt Enrichment
Instead of a generic image generation prompt, GPT-4o receives the real product data:

```
Today's content theme: Product Spotlight 🏏
Real product to feature:
  Name: SS Ton Master 5.0
  Brand: SS
  Price: $389.00 AUD
  URL: https://amgosports.com.au/products/ss-ton-master-5

Generate an Instagram caption that:
- Leads with the product name and brand  
- Highlights performance benefits (willow grade, pressing, pick-up)
- Mentions the real price as a value hook
- Includes "🔗 Link in bio" CTA pointing to the product page
- Uses #amgosports #SS #SSbats as core hashtags
```

#### 3. Real Product Image (No AI Generation)
The Shopify product image URL is used directly — no AI image generation for product posts. This:
- Saves per-image generation cost
- Shows the actual product customers will receive
- Uses high-quality Shopify product photography

The Shopify image URL is uploaded to imgbb for a permanent URL (Shopify CDN URLs can sometimes change).

---

## Dashboard Integration — "Feature This Product"

The dashboard's **Products page** shows all synced products. Clicking **"Feature this product"** on any card:

1. Sends a `POST /api/generate` request with `{ productId: "7123456789" }`
2. The API calls n8n Workflow A via webhook with the product ID
3. Workflow A skips the random product selection and uses that specific product
4. A Telegram preview arrives within ~30 seconds

This allows manual control — for example, featuring a new stock arrival or a specific seasonal product.

---

## Shopify API Setup

### Step 1 — Create a Custom App in Shopify Admin

1. Go to **Shopify Admin → Settings → Apps → Develop apps**
2. Click **Create an app**
3. Name it: `AMGO Sports Content Engine`
4. Click **Configure Admin API scopes**
5. Enable **only**: `read_products`
6. Click **Save** → **Install app**
7. Click **Reveal token** — copy the `shpat_...` access token

> ⚠️ Save the token immediately — it's only shown once.

### Step 2 — Add to n8n Credentials

Create a new **HTTP Header Auth** credential in n8n:
- **Name**: `Shopify_AMGO`
- **Header Name**: `X-Shopify-Access-Token`
- **Header Value**: `shpat_your_token_here`

### Step 3 — Add to Dashboard `.env.local`

```bash
SHOPIFY_STORE_DOMAIN=amgosports.myshopify.com
SHOPIFY_ACCESS_TOKEN=shpat_your_token_here
```

---

## Product Categories

Products are automatically categorised by their Shopify product type or tags:

| Tag / Type | Category |
|-----------|---------|
| bat, bats, batting | Bats |
| pad, pads, leg guard | Protection |
| gloves, batting gloves | Gloves |
| helmet, helmets | Helmets |
| ball, cricket ball | Balls |
| bag, kit bag | Bags & Accessories |
| clothing, whites | Clothing |

---

## Database: Products

**Required fields:**
```
Product ID | Title | Vendor | Price | Image URL | Product URL | Category | In Stock | Synced At | Featured
```

The `Featured` column can be manually set to `TRUE` to prioritise a product for selection on product spotlight days.

---

## Error Handling

| Scenario | Behaviour |
|----------|-----------|
| Shopify API rate limit | Retry after 60s delay |
| Product has no images | Filtered out — not included in sync |
| Product out of stock | `In Stock = FALSE`, excluded from random selection |
| Shopify API returns error | Telegram alert to operator, previous product list retained |

---

## Cost

**Zero additional cost.** Shopify Admin API is free with any Shopify plan. One weekly API call uses negligible quota.
