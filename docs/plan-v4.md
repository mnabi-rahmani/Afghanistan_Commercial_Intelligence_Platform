# Afghanistan Commercial Intelligence Platform — Version 4

**Version:** 4.0  
**Updated:** 2026-09-04  
**Working name:** Afghanistan Commercial Intelligence Platform (ACIP)  
**Primary orchestration/development agents:** Carcer + Codex  
**Initial geographic scope:** Afghanistan  
**Initial priority sources:** Meta Ad Library, public Facebook pages, public Instagram accounts, Afghan business directories, marketplaces, websites/e-commerce

---

# 1. Executive Summary

The project is no longer only a Meta Ads archive.

The goal is to build a continuously updated **Afghanistan Commercial Intelligence Platform** that discovers Afghan businesses, collects their public commercial activity, extracts products/offers/prices/locations/contact details, preserves media and historical activity, and allows AI to answer practical market questions directly from the accumulated dataset.

The platform should reduce or eventually eliminate the need to manually scroll Facebook, Instagram, marketplaces, and business pages to discover:

- businesses,
- products,
- prices,
- brands,
- shops,
- sellers,
- new machines/equipment,
- agricultural products,
- electronics,
- sewing machinery,
- batteries/solar products,
- screen-printing products,
- poultry/farm equipment,
- services,
- market trends,
- product availability,
- competitor activity.

The central asset is not the scraper.

The central asset is:

> **A persistent Afghanistan business + advertiser + product + offer + price + source-history dataset.**

The long-term objective is that the user asks the system questions such as:

> Find every poultry-related machine advertised in Afghanistan during the last six months.

> Which businesses currently advertise Jack sewing machines?

> What 5 kWh lithium batteries are being sold in Afghanistan, at what prices, and by whom?

> What agricultural machines appeared for the first time during the last 90 days?

> Which product categories are growing fastest?

> Which sellers advertise products imported from China?

> Show every current and historical offer for a specific product, including media and source links.

The system should answer from its own accumulated data first, rather than starting a new manual Facebook/Instagram search every time.

---

# 2. Core Design Principle

The platform should be organized around **commercial entities**, not around social-media platforms.

Facebook posts, Instagram reels, Meta ads, marketplace listings, and websites are only sources.

The core hierarchy is:

```text
BUSINESS
   |
   +-- social identities
   |     +-- Facebook Page
   |     +-- Instagram Account
   |     +-- website
   |     +-- marketplace profile
   |
   +-- advertisements
   +-- organic posts
   +-- reels/videos
   +-- comments
   |
   +-- PRODUCTS
   |      |
   |      +-- OFFERS
   |             |
   |             +-- price
   |             +-- seller
   |             +-- condition
   |             +-- location
   |             +-- source
   |             +-- availability
   |
   +-- contact information
   +-- locations
   +-- categories
   +-- brands
   +-- activity history
```

The system should resolve multiple source records back to the same underlying:

- business,
- product,
- offer,
- brand,
- location.

---

# 3. Primary Scope

## 3.1 Included

The initial platform should focus on public commercial activity connected to Afghanistan, including:

- Afghan businesses,
- businesses established/operating in Afghanistan,
- informal Afghan businesses,
- Facebook-only shops,
- Instagram-only shops,
- registered businesses,
- retailers,
- wholesalers,
- importers,
- distributors,
- service providers,
- manufacturers,
- farms/agricultural suppliers,
- electronics sellers,
- clothing businesses,
- machinery sellers,
- marketplace sellers.

## 3.2 Important distinction

The project should distinguish:

### Afghan business

A business established or operating in Afghanistan.

### Afghan advertiser

A verified Afghan business with a Meta advertising identity/Page ID.

### Afghanistan-targeted advertisement

An ad delivered in Afghanistan.

### International ad by Afghan advertiser

An ad run by an Afghan business outside Afghanistan.

The primary objective is:

> **Find Afghan businesses first, then monitor all observable advertising and public commercial activity from those businesses.**

This avoids polluting the dataset with foreign advertisers that merely target Afghanistan.

---

# 4. High-Level Architecture

```text
                             INTERNET
                                |
      +-------------------------+--------------------------+
      |             |             |           |            |
      v             v             v           v            v
 Meta Ad Library  Facebook     Instagram   Marketplaces   Websites
      |             |             |           |            |
      +-------------+-------------+-----------+------------+
                                |
                                v
                       COLLECTION LAYER
                                |
                       Carcer schedules/jobs
                                |
                        Codex maintenance
                                |
                                v
                          RAW ARCHIVE
                                |
              +-----------------+------------------+
              |                 |                  |
              v                 v                  v
          metadata          images/video       raw HTML/JSON
              |                 |                  |
              +-----------------+------------------+
                                |
                                v
                       NORMALIZATION LAYER
                                |
                                v
                      ENTITY EXTRACTION
                                |
          +---------------------+----------------------+
          |                     |                      |
          v                     v                      v
      BUSINESSES             PRODUCTS               OFFERS
          |                     |                      |
          +---------------------+----------------------+
                                |
                                v
                    AFGHAN COMMERCE GRAPH
                                |
                 +--------------+--------------+
                 |                             |
                 v                             v
             PostgreSQL                 Object storage
                 |
          +------+------+
          |             |
          v             v
     SQL analytics   Vector search
          |             |
          +------+------+
                 |
                 v
             AI QUERY AGENT
                 |
                 v
                USER
```

---

# 5. Source Strategy

The platform should use multiple acquisition sources because no single source will produce adequate Afghanistan coverage.

## 5.1 Source group A — Meta advertising

Use:

- Meta Ad Library Afghanistan discovery queries,
- known advertiser Page IDs,
- country = ALL for verified Afghan advertisers,
- incremental monitoring.

Purpose:

- capture active ads,
- discover new advertisers,
- archive creatives,
- observe advertiser activity.

## 5.2 Source group B — Public Facebook pages

Collect:

- page/profile metadata,
- posts,
- captions,
- timestamps,
- images,
- reels/videos,
- engagement counts where available,
- comments selectively,
- source URLs.

Purpose:

- discover businesses/products not visible in ads,
- preserve organic commercial activity,
- identify products/offers.

## 5.3 Source group C — Public Instagram accounts

Collect:

- profile metadata,
- posts,
- reels,
- carousels,
- captions,
- media,
- engagement counts where available,
- comments selectively,
- source URLs.

## 5.4 Source group D — Business directories

Use Afghan directories/registries as seed sources.

Purpose:

- discover businesses,
- resolve business names,
- find contact details,
- discover social-media identities.

## 5.5 Source group E — Marketplaces/classifieds

Add public marketplace/listing sources relevant to Afghanistan.

Purpose:

- capture used products,
- prices,
- local availability,
- individual sellers,
- informal commercial activity.

## 5.6 Source group F — Websites/e-commerce

Collect structured/public product data from:

- retailer websites,
- importer websites,
- Afghan e-commerce stores,
- manufacturer/distributor sites.

## 5.7 Later sources

Potential future additions:

- public Telegram channels,
- YouTube,
- TikTok where technically accessible,
- other Afghan classifieds,
- public business groups,
- industry association pages.

---

# 6. Afghanistan Advertiser Discovery

The main advertiser-discovery system should combine:

```text
                    AFGHAN ADVERTISER DISCOVERY
                               |
          +--------------------+--------------------+
          |                    |                    |
          v                    v                    v
 Business directories    Meta Ad Library       Feed sensors
 Web/social discovery    Afghanistan scan     FB / Instagram
          |                    |                    |
          +--------------------+--------------------+
                               |
                               v
                    AFGHAN BUSINESS REGISTRY
                               |
                               v
                 FACEBOOK/INSTAGRAM RESOLUTION
                               |
                               v
                    META ADVERTISER PAGE IDs
                               |
                               v
                 Meta Ad Library country = ALL
```

Once a business/Page ID is discovered and verified, it should not need to be repeatedly rediscovered.

This creates a compounding registry.

---

# 7. Afghan Business Verification

Do not rely on AI guesses alone.

Use evidence scoring.

Example signals:

| Evidence | Strength |
|---|---|
| +93 phone number | Very strong |
| Afghanistan address | Very strong |
| Kabul/Herat/Kandahar/etc. | Strong |
| Official Afghan website | Strong |
| Website links to Facebook/Instagram | Very strong |
| AFN pricing | Moderate/strong |
| +93 WhatsApp | Very strong |
| Dari/Pashto content | Moderate |
| Afghan delivery wording | Moderate |
| Matching directory/registry record | Very strong |
| Business page transparency/location | Strong |

Example classification:

```text
0.85–1.00 = verified Afghan business
0.60–0.84 = probable Afghan business
0.40–0.59 = manual review
<0.40     = likely non-Afghan
```

Store both score and supporting evidence.

---

# 8. Meta Ad Library Collection

## 8.1 Known advertiser approach

For every verified Afghan Meta Page ID:

```text
page_id = VERIFIED_PAGE_ID
country = ALL
status = ACTIVE / ALL as supported
```

This should be the primary Meta Ads monitoring method.

## 8.2 Discovery approach

Broad Afghanistan discovery should be used primarily to discover new businesses.

Examples:

### Dari/Persian

```text
افغانستان
کابل
هرات
فروش
قیمت
افغانی
سفارش
تماس
واتساپ
تخفیف
```

### English

```text
Afghanistan
Afghan
Kabul
Herat
Kandahar
Mazar
sale
price
delivery
WhatsApp
AFN
```

### Product categories

```text
solar
battery
electronics
mobile
clothing
machinery
agriculture
poultry
printing
sewing
construction
medical
education
food
beauty
real estate
travel
logistics
```

Maintain equivalent Pashto lists.

## 8.3 Adaptive discovery

Track yield per discovery query:

```text
keyword
runs
ads_found
advertisers_found
new_verified_afghan_advertisers
last_new_discovery
yield_score
```

Run high-yield queries more often.

Reduce frequency for low-yield queries.

---

# 9. Cheapest Meta Collection Strategy

The lowest-cost normal path should be:

```text
SELF-HOSTED COLLECTOR
        |
        v
KNOWN PAGE IDs
        |
        v
INCREMENTAL COLLECTION
        |
     existing?
     /      \
   yes       no
    |         |
 ignore    process
```

Recommended operating principle:

> Do not pay a third-party scraper per result unless the self-hosted collector fails or a source requires it.

Commercial services such as Apify/Bright Data should be fallback providers, not the primary permanent acquisition path.

---

# 10. Collection Frequency

Use adaptive intervals.

## Tier A — Currently active advertiser

```text
every 4–8 hours
```

## Tier B — Advertised recently

```text
daily
```

## Tier C — Dormant advertiser

```text
every 3–7 days
```

## Tier D — Business with social page but no observed ads

```text
every 2–4 weeks
```

Broad Afghanistan discovery:

```text
adaptive / every few days
```

The objective is cost efficiency, not maximum request volume.

---

# 11. Public Facebook/Instagram Scraping Architecture

For public pages/accounts, the preferred collection priority is:

```text
1. Direct HTTP/internal-data collector where practical
2. Browser/Playwright collector if needed
3. Apify/other provider fallback
```

Conceptually:

```text
SCRAPE REQUEST
      |
      v
Own HTTP collector
      |
   succeeds?
   /      \
 yes       no
  |         |
save     Playwright
             |
          succeeds?
          /      \
        yes       no
         |         |
       save      Apify fallback
```

This keeps normal marginal acquisition cost very low.

---

# 12. Incremental Organic-Post Collection

Historical posts should be collected once.

Example:

```text
Business has 4,300 historical posts
```

Initial import:

```text
collect all 4,300
```

Future runs:

```text
fetch newest posts
       |
       v
known post_id?
   /       \
 yes        no
  |          |
 stop       save
```

Do not repeatedly crawl thousands of old posts.

---

# 13. Feed Sensors

Feed sensors are a secondary discovery source.

A feed sensor is a logged-in Facebook or Instagram session that observes ads actually delivered to that account.

Recommended first implementation:

```text
Python/Node
   |
   v
Playwright
   |
   v
persistent browser profile
   |
   v
Facebook/Instagram feed
   |
   v
scroll
   |
   v
sponsored content?
   |
   v
capture observation
```

Initial behavior should remain read-only:

- load feed,
- scroll,
- detect sponsored content,
- capture advertiser,
- capture text,
- capture image/video or screenshot,
- store timestamp/source,
- avoid unnecessary engagement.

Feed observations should be used mainly to discover advertisers that other methods miss.

---

# 14. Why Feed Sensors Are Secondary

A personal feed is personalized.

Therefore:

```text
one account
!=
Afghanistan advertising market
```

Feed delivery may depend on:

- interests,
- language,
- location,
- engagement history,
- prior activity,
- campaign targeting,
- optimization.

Feed sensors can improve discovery but cannot replace advertiser-level Meta Ad Library monitoring.

---

# 15. Phone / Android Device Farms

Phone/device farms are optional later-stage infrastructure, not an MVP requirement.

Typical architecture:

```text
Android phones / boards
        |
        v
 USB hub / LAN
        |
        v
   Host computer
        |
        v
       ADB
        |
  +-----+------+
  |            |
  v            v
scrcpy      Appium
             |
        UiAutomator2
             |
             v
       mobile apps
```

Possible hardware forms:

- complete phones,
- phones with screens off,
- stripped smartphone motherboards,
- Android boards,
- virtual/cloud Android devices.

Use cases for this project:

- native-app feed observation,
- real mobile rendering,
- mobile-specific ad-delivery research,
- app compatibility testing.

Recommended order:

```text
1. Business registry
2. Meta Ad Library
3. Public page/account collectors
4. Browser feed sensors
5. Only then evaluate Android-device sensors
```

---

# 16. Automation-Detection Boundary

The system should not depend on defeating platform anti-abuse controls.

At a high level, platforms may evaluate:

- abnormal action rates,
- repeated navigation patterns,
- session behavior,
- browser/device characteristics,
- network reputation,
- account relationships,
- unusual activity.

The project should prefer:

```text
read-only collection
+ sustainable request rates
+ persistent legitimate sessions
+ adaptive scheduling
+ caching
+ retries
+ backoff
+ public sources
```

Do not make core functionality depend on CAPTCHA bypass, fake-account generation, fingerprint spoofing, or concealment of coordinated automation.

---

# 17. Raw Data Archive

Raw data is critical.

Always preserve where useful:

```text
raw response
raw HTML
raw JSON
source URL
original caption
original image
original video
observation timestamp
collector version
parser version
```

Reason:

If extraction logic later turns out to be wrong, reprocess locally.

```text
RAW ARCHIVE
   |
   v
new parser
   |
   v
corrected structured records
```

This avoids unnecessary rescraping.

---

# 18. Core Entity Model

## 18.1 Business

```text
business_id
name
legal_name
category
subcategory
province
city
address
phone
whatsapp
email
website
afghan_business_score
verification_status
first_seen
last_seen
```

## 18.2 Social identity

```text
social_id
business_id
platform
username
profile_url
meta_page_id
match_score
verified
first_seen
last_verified
```

## 18.3 Advertiser

```text
advertiser_id
business_id
meta_page_id
page_name
page_url
first_ad_seen
last_ad_seen
is_currently_active
crawl_tier
last_crawled
```

## 18.4 Source post

```text
post_id
business_id
platform
source_post_id
post_url
post_type
caption
created_at
likes
comments
views
first_seen
last_seen
raw_payload
```

## 18.5 Advertisement

```text
ad_id
business_id
advertiser_id
meta_ad_id
status
meta_start_date
meta_end_date
first_seen
last_seen
body_text
headline
cta
destination_url
platforms
raw_payload
```

## 18.6 Product

```text
product_id
canonical_name
brand
model
category
subcategory
description
product_signature
first_seen
last_seen
```

## 18.7 Offer

```text
offer_id
product_id
business_id
source_type
source_record_id
price
currency
condition
availability
province
city
seller_notes
first_seen
last_seen
```

## 18.8 Media

```text
media_id
source_record_id
media_type
original_url
stored_path
sha256
phash
embedding_id
width
height
duration
first_seen
```

## 18.9 Comments

```text
comment_id
post_id
source_comment_id
text
created_at
author_public_reference
intent_type
raw_payload
```

Comments should be collected selectively, not exhaustively by default.

---

# 19. Product Resolution

A social post is not a product.

One product may appear in:

```text
3 Facebook posts
2 Instagram reels
4 Meta ads
1 website page
```

The system should resolve these into one product entity where confidence is high.

Example:

```text
PRODUCT: Jack A5E
   |
   +-- FB post
   +-- Instagram reel
   +-- Meta ad
   +-- seller A offer
   +-- seller B offer
```

Product matching signals:

- exact brand/model,
- OCR,
- caption,
- visual logo/model detection,
- product embeddings,
- seller/category,
- price range,
- image similarity.

---

# 20. Offer Resolution

Offers must be separate from products.

Example:

```text
PRODUCT: Jack A5E

Seller A
52,000 AFN
Kabul
new

Seller B
49,000 AFN
Herat
new

Seller C
31,000 AFN
Kabul
used
```

This allows price and availability analysis across sellers and time.

---

# 21. Image Processing

For each new unique image:

```text
image
 |
 v
SHA-256
 |
 v
duplicate?
 /      \
yes      no
 |        |
reuse   pHash
          |
          v
        OCR
          |
          v
 structured extraction
          |
          v
 selective vision AI
```

Extract:

- text,
- phone numbers,
- WhatsApp,
- prices,
- currency,
- location,
- product names,
- model numbers,
- brand logos,
- contact information.

---

# 22. Video Processing

Videos should be processed selectively.

Suggested pipeline:

```text
video
 |
 +-- metadata
 |
 +-- sampled frames
 |      |
 |      +-- OCR
 |      +-- vision analysis
 |
 +-- audio
        |
        +-- speech-to-text
```

Do not analyze every video frame.

Sample frames intelligently.

Example:

```text
0s
10s
20s
30s
40s
50s
```

for a 60-second reel, with smarter scene-based sampling added later.

Possible extracted information:

- product type,
- brand/model,
- visible price,
- seller/store name,
- phone number,
- spoken offer,
- location,
- availability.

---

# 23. Comments and Demand Signals

Comments can provide demand information.

Examples:

```text
قیمت؟
```

```text
Do you deliver to Herat?
```

```text
Is this available?
```

Possible derived signals:

- price inquiry,
- availability inquiry,
- purchase intent,
- delivery inquiry,
- complaint,
- positive response,
- negative response.

Do not rescrape comments on every old post forever.

Possible policy:

```text
new/high-engagement post -> collect comments
comment count changed    -> update
old inactive post        -> stop checking frequently
```

---

# 24. OCR and Deterministic Extraction

Use local processing first.

Languages:

- Dari/Persian,
- Pashto,
- English,
- Arabic/Urdu where relevant.

Extract deterministically where possible:

### Phones

```text
+93...
07...
```

### Prices

```text
2500 AFN
۲۵۰۰ افغانی
$500
500 USD
```

### Other

- email,
- URL,
- WhatsApp,
- discount,
- date,
- location,
- province,
- city,
- product code.

AI should not be used for basic regex tasks.

---

# 25. Deduplication

Use multiple layers.

## Exact media

```text
SHA-256
```

## Visually similar media

```text
pHash
```

## Semantically similar media/content

```text
embeddings
```

Distinguish:

```text
Meta Ad IDs
source posts
creative files
creative variants
underlying product
underlying campaign/design
```

This reduces storage and processing costs.

---

# 26. AI Enrichment

AI should be the last processing layer.

Bad:

```text
100,000 images
    |
    v
100,000 paid vision calls
```

Better:

```text
media
 |
 v
dedupe
 |
 v
OCR
 |
 v
rules
 |
 v
local classification/embeddings
 |
 v
uncertain?
 /      \
no      yes
 |        |
done    paid AI
```

AI tasks:

- product classification,
- visual product identification,
- brand/model inference,
- business category,
- unclear location inference,
- campaign grouping,
- offer interpretation,
- trend explanation.

---

# 27. Search Architecture

The AI layer should use both structured and semantic retrieval.

## SQL

Use SQL for questions such as:

> What is the cheapest 5 kWh battery?

> How many solar sellers are in Kabul?

> Which seller had the most ads this month?

## Semantic/vector search

Use semantic retrieval for:

> Find machines related to animal-feed processing even when the seller does not use the same terminology.

Combined architecture:

```text
User question
      |
      v
Question router
      |
  +---+---+
  |       |
  v       v
 SQL    semantic
  |       |
  +---+---+
      |
      v
 source records
      |
      v
     AI
      |
      v
 answer + evidence
```

Database computes; AI interprets.

---

# 28. Trend Analytics

Do not let an LLM decide what is trending by intuition.

Calculate trend metrics.

Per product/category:

```text
ads_last_30_days
posts_last_30_days
unique_sellers
new_sellers
engagement
price_inquiries
growth_vs_previous_period
new_creatives
offer_count
price_change
```

Example trend score:

```text
35% advertiser/seller growth
25% posting/ad growth
20% engagement growth
10% new sellers
10% inquiry growth
```

The weighting should be configurable and validated.

AI should explain the trend after the database calculates it.

---

# 29. Example Intelligence Queries

The platform should eventually answer:

## Products

> Show all poultry incubators currently advertised in Afghanistan.

> Find industrial humidity sensors sold by Afghan businesses.

> Find machines related to animal-feed production.

## Sellers

> Who sells Jack A5E machines in Afghanistan?

> Which businesses in Herat sell solar inverters?

## Price

> What is the current price range for 5 kWh LiFePO4 batteries?

> Show price history for Jack K7-D.

## New products

> Which agricultural products appeared for the first time in the last 90 days?

## Trends

> What product categories are growing fastest this month?

> Which Chinese sewing-machine brands are gaining advertising activity?

## Competition

> Which Kabul solar sellers advertise most aggressively?

## Availability

> Find every current offer for a specific model and rank by price.

## Import intelligence

> Which Afghan businesses appear to import electronics from China?

## Historical

> Show every observed advertisement and offer for this product during the last year.

---

# 30. Carcer + Codex Operating Model

Carcer should be treated as the **orchestration and autonomous operations layer**, while Codex handles code generation, repair, tests, migrations, and development tasks.

Conceptual responsibilities:

## Carcer

- schedule collectors,
- trigger recurring jobs,
- maintain job state,
- resume interrupted work,
- run retries,
- watch health metrics,
- dispatch maintenance tasks,
- trigger Codex when repair is needed,
- coordinate backlogs.

## Codex

- build collectors,
- modify parsers,
- repair broken integrations,
- write tests,
- run fixtures,
- implement schema changes,
- improve extraction,
- maintain documentation,
- propose and implement fixes.

Do not assume zero maintenance.

The objective is:

> **Low-human-maintenance autonomous operation.**

---

# 31. Scheduled Jobs

Suggested jobs:

```text
COLLECT_META_ACTIVE_ADVERTISERS
every 4–8 hours

COLLECT_RECENT_ADVERTISERS
daily

COLLECT_DORMANT_ADVERTISERS
weekly

DISCOVER_AFGHAN_ADVERTISERS
daily/adaptive

COLLECT_FACEBOOK_PUBLIC_PAGES
adaptive

COLLECT_INSTAGRAM_PUBLIC_ACCOUNTS
adaptive

RUN_FEED_SENSORS
periodic

PROCESS_NEW_MEDIA
continuous/queued

OCR_NEW_MEDIA
continuous/queued

PROCESS_NEW_VIDEOS
queued

EXTRACT_PRODUCTS
queued

RESOLVE_PRODUCTS
queued

EXTRACT_OFFERS
queued

UPDATE_TRENDS
daily

HEALTH_CHECK_COLLECTORS
hourly

RETRY_FAILED_JOBS
hourly

BACKUP_RAW_ARCHIVE
daily

CANARY_TEST
scheduled
```

---

# 32. Self-Healing Collector Design

Do not simply prompt Carcer:

> Maintain everything.

Collectors need measurable health conditions.

Store:

```text
collector_name
last_success
last_run_started
last_run_finished
results_last_run
new_records
expected_minimum
error_rate
request_latency
duplicate_rate
media_success_rate
parser_version
collector_version
```

Example:

```text
Normal Instagram collector:
800–2,000 records/run

Current:
0 records
```

Trigger:

```text
COLLECTOR_HEALTH_FAILURE
```

Carcer can then dispatch Codex to:

1. reproduce failure,
2. inspect current implementation,
3. compare fixtures/live output,
4. identify changed source structure,
5. patch parser/collector,
6. run tests,
7. only restore production after tests pass.

---

# 33. Canary Sources

Maintain known public sources that should reliably return data.

Example:

```text
5 Facebook pages
5 Instagram accounts
5 Meta advertisers
```

Canary workflow:

```text
run canary
    |
    v
expected records?
expected fields?
pagination?
media download?
    |
    v
pass?
 /   \
yes   no
 |     |
run   halt/limit large crawl
       |
       v
    repair
```

This prevents silent data corruption.

---

# 34. Parser and Collector Versioning

Every structured record should retain:

```text
collector_version
parser_version
analysis_version
```

Example:

```text
facebook_collector_v12
facebook_parser_v19
product_resolver_v7
```

If a parser bug is discovered, identify affected records and reprocess raw data.

---

# 35. Retry and Resume

Every large collection process should be resumable.

Do not design jobs as all-or-nothing.

Use state:

```text
job_id
source
cursor
page_number
last_processed_id
attempt_count
started_at
last_checkpoint
status
```

On interruption:

```text
load checkpoint
      |
      v
resume from last safe cursor
```

---

# 36. Failure Isolation

One broken source should not break the entire platform.

```text
Meta collector fails
      |
      +-- Facebook still runs
      +-- Instagram still runs
      +-- marketplace still runs
      +-- raw processing still runs
```

Collectors should be modular adapters behind common interfaces.

---

# 37. Generic Collector Interfaces

Example:

```python
class SocialCollector:
    def get_profile(...):
        ...

    def get_posts(...):
        ...

    def get_post(...):
        ...

    def get_comments(...):
        ...

    def get_media(...):
        ...
```

Implementations:

```text
InstagramHTTPCollector
InstagramPlaywrightCollector
InstagramApifyCollector

FacebookHTTPCollector
FacebookPlaywrightCollector
FacebookApifyCollector

MetaAdLibraryCollector
MetaAdLibraryFallbackCollector
```

The rest of the platform should consume normalized records rather than source-specific structures.

---

# 38. Storage Strategy

## MVP

Possible:

```text
SQLite
local media storage
```

for very small proof-of-concept work.

## Production direction

Recommended:

```text
PostgreSQL
+
S3-compatible object storage / Cloudflare R2
+
pgvector or equivalent
```

Do not add Elasticsearch/Kafka/Kubernetes until real scale requires them.

---

# 39. Cost-Minimization Strategy

The cheapest useful architecture is:

```text
own collectors
+ incremental crawling
+ adaptive scheduling
+ local OCR
+ deterministic extraction
+ deduplication
+ local/cheap embeddings
+ selective paid AI
+ commercial scraping fallback only when necessary
```

Main cost-saving rules:

1. Discover business/Page ID once.
2. Never repeatedly re-import unchanged history.
3. Only process new media.
4. Deduplicate before OCR/AI.
5. Avoid full video archival unless necessary.
6. Use local OCR.
7. Use SQL/rules before AI.
8. Crawl active businesses more frequently than dormant ones.
9. Reduce low-yield discovery queries.
10. Use Apify/other providers only as fallback.
11. Start without proxies; add only if actually required.
12. Run feed sensors periodically, not continuously, unless data proves longer sessions add unique value.

---

# 40. Coverage Objective

Absolute 100% coverage is not realistic.

A mature target should be:

```text
75–90% of identifiable active Afghan Meta advertisers
90–98% of observable active ads from known advertiser Page IDs
70–90% overall active Afghan-business Meta ad coverage
```

These are planning targets, not guaranteed published statistics.

The platform should measure coverage using:

```text
known businesses
known social identities
verified Afghan advertisers
new advertisers discovered/week
source overlap
category coverage
province coverage
new discovery yield
```

Coverage should be estimated empirically over time.

---

# 41. Coverage Saturation

Track:

```text
new_verified_businesses_per_week
new_verified_advertisers_per_week
new_products_per_week
new_sellers_per_category
```

Example saturation pattern:

```text
Month 1  +1200 advertisers
Month 2   +650
Month 3   +310
Month 4   +140
Month 5    +62
Month 6    +24
```

Declining new-discovery rate indicates improving registry saturation.

Break coverage down by:

- province,
- city,
- business category,
- product category,
- source.

---

# 42. Repository Structure

```text
afghanistan-commercial-intelligence/
|
+-- README.md
+-- docker-compose.yml
+-- .env.example
|
+-- apps/
|   +-- api/
|   +-- worker/
|   +-- dashboard/
|   +-- ai_query/
|
+-- collectors/
|   +-- meta_ads/
|   |   +-- client.py
|   |   +-- parser.py
|   |   +-- pagination.py
|   |   +-- fallback.py
|   |
|   +-- facebook/
|   |   +-- http_collector.py
|   |   +-- playwright_collector.py
|   |   +-- apify_fallback.py
|   |
|   +-- instagram/
|   |   +-- http_collector.py
|   |   +-- playwright_collector.py
|   |   +-- apify_fallback.py
|   |
|   +-- marketplaces/
|   +-- websites/
|   +-- directories/
|   +-- feed_sensors/
|   +-- device_farm/
|
+-- discovery/
|   +-- keywords_dari.txt
|   +-- keywords_pashto.txt
|   +-- keywords_english.txt
|   +-- categories.yml
|   +-- geographies.yml
|   +-- business_resolver.py
|   +-- advertiser_classifier.py
|
+-- processing/
|   +-- media_downloader.py
|   +-- hashing.py
|   +-- ocr.py
|   +-- video_frames.py
|   +-- speech_to_text.py
|   +-- phones.py
|   +-- prices.py
|   +-- locations.py
|   +-- product_extractor.py
|   +-- product_resolver.py
|   +-- offer_extractor.py
|   +-- comment_intent.py
|   +-- embeddings.py
|
+-- analytics/
|   +-- trends.py
|   +-- price_history.py
|   +-- seller_activity.py
|   +-- coverage.py
|
+-- database/
|   +-- migrations/
|   +-- models/
|   +-- repositories/
|
+-- orchestration/
|   +-- carcer_jobs/
|   +-- health_checks/
|   +-- retry/
|   +-- canaries/
|
+-- jobs/
|   +-- collect_meta.py
|   +-- collect_facebook.py
|   +-- collect_instagram.py
|   +-- discover_businesses.py
|   +-- process_media.py
|   +-- extract_products.py
|   +-- extract_offers.py
|   +-- compute_trends.py
|
+-- tests/
|   +-- fixtures/
|   +-- unit/
|   +-- integration/
|   +-- canary/
|
+-- docs/
    +-- architecture.md
    +-- schema.md
    +-- source-adapters.md
    +-- collector-health.md
    +-- trend-model.md
    +-- coverage-model.md
```

---

# 43. Development Plan

## Phase 0 — Reference Dataset

Use a small number of known Afghan businesses.

For each:

- known Facebook page,
- known Instagram account,
- known Meta Page ID if available,
- several public posts,
- several ads.

Optionally use Apify once to create a reference extraction dataset.

Goal:

> Establish what “complete enough” extraction looks like before building every collector.

---

## Phase 1 — Core Database + Raw Archive

Implement:

- PostgreSQL schema,
- raw source storage,
- businesses,
- social identities,
- posts,
- ads,
- media,
- products,
- offers,
- collector health tables.

---

## Phase 2 — Meta Ads Collector

Implement:

- Page ID input,
- country=ALL where supported,
- pagination,
- incremental state,
- first_seen/last_seen,
- media downloading,
- raw payload preservation,
- retries/backoff,
- canary tests.

---

## Phase 3 — Facebook Public Page Collector

Implement:

- public profile metadata,
- incremental post collection,
- image/video references,
- raw archive,
- media download,
- normalized posts.

Use:

```text
HTTP collector
-> Playwright fallback
-> Apify fallback
```

---

## Phase 4 — Instagram Public Account Collector

Same architecture:

```text
HTTP collector
-> Playwright fallback
-> Apify fallback
```

Capture:

- posts,
- reels,
- carousels,
- captions,
- timestamps,
- media,
- public metrics where available.

---

## Phase 5 — Afghan Business Registry

Build:

- directory importers,
- business matching,
- Facebook/Instagram identity resolution,
- Afghan-business evidence scoring,
- manual review queue.

---

## Phase 6 — Advertiser Discovery

Build:

- Meta Afghanistan discovery,
- Dari queries,
- Pashto queries,
- English queries,
- geography terms,
- product categories,
- adaptive query yield scoring.

---

## Phase 7 — Media Intelligence

Implement:

- SHA-256,
- pHash,
- image dedupe,
- OCR,
- phone extraction,
- price extraction,
- location extraction,
- product/model extraction.

---

## Phase 8 — Product + Offer Graph

Implement:

- product entity extraction,
- brand/model normalization,
- product deduplication,
- seller/product relationships,
- offer extraction,
- price history.

This is the phase where the project becomes a commercial-intelligence platform rather than a social-media archive.

---

## Phase 9 — Video Intelligence

Add:

- thumbnails,
- selected frame extraction,
- frame OCR,
- speech-to-text,
- selective vision analysis.

---

## Phase 10 — Comments / Demand Signals

Collect selectively:

- recent/high-engagement post comments,
- inquiry detection,
- price questions,
- availability questions,
- delivery questions.

---

## Phase 11 — Feed Sensors

Add one:

```text
Facebook Playwright sensor
Instagram Playwright sensor
```

Use primarily for new advertiser/business discovery.

Do not build a phone farm yet.

---

## Phase 12 — Trend Analytics

Implement:

- seller count changes,
- ad/post volume changes,
- engagement changes,
- new product detection,
- new seller detection,
- price movement,
- trend scores.

---

## Phase 13 — AI Query Agent

Implement:

```text
question
-> SQL planner
-> semantic retrieval
-> evidence selection
-> answer
```

Answers should link back to source records/media.

---

## Phase 14 — Carcer Autonomous Operations

Add:

- schedules,
- recurring jobs,
- checkpoints,
- health monitoring,
- retries,
- resume,
- canary execution,
- Codex repair dispatch,
- test-gated recovery.

---

## Phase 15 — Dashboard

Views:

### Business

- details,
- social accounts,
- contacts,
- locations,
- products,
- ads,
- organic posts,
- activity.

### Product

- canonical product,
- media,
- sellers,
- current offers,
- historical prices,
- related posts/ads,
- trend score.

### Offer

- seller,
- price,
- location,
- condition,
- source,
- first/last seen.

### Market

- categories,
- trending products,
- new products,
- top advertisers,
- active sellers,
- geographic distribution.

### System health

- collector status,
- failed jobs,
- source coverage,
- backlog,
- canary status,
- cost metrics.

---

# 44. First Carcer/Codex Development Task

Do not build the entire system in one prompt.

First task:

> Build the core repository and a production-quality proof-of-concept collector framework. Create PostgreSQL models for businesses, social identities, source posts, advertisements, media, products, offers, raw source payloads, collector runs, and job checkpoints. Implement a Meta Ad Library collector adapter that accepts verified Page IDs, performs incremental collection, normalizes ads, stores raw payloads, downloads new image creatives, computes SHA-256, tracks first_seen/last_seen, and supports retries/backoff and resumable pagination. Keep the collector behind an interface so commercial providers can be used as fallback implementations. Add fixture-based tests, a small canary suite, structured logging, and a README.

Acceptance criteria:

1. PostgreSQL runs locally through Docker Compose.
2. Collector inputs verified Page IDs.
3. Existing records are not duplicated.
4. New ads are inserted.
5. Existing ads update last_seen.
6. Raw payloads are preserved.
7. New images are archived.
8. SHA-256 is stored.
9. Pagination is resumable.
10. Collector run health is recorded.
11. Retry/backoff exists.
12. Tests use fixtures.
13. Canary tests are separate.
14. No OCR/AI yet.
15. README documents limitations and recovery behavior.

---

# 45. Second Carcer/Codex Task

> Add generic public social collectors and implement Facebook and Instagram adapters with an own-collector-first architecture and commercial fallback interfaces. Support incremental profile/post collection, media references, raw-data retention, collector health metrics, checkpoints, and fixture testing. Do not add product AI yet.

---

# 46. Third Carcer/Codex Task

> Implement Afghan Business Registry ingestion and social-identity resolution. Import businesses from configurable directory sources, resolve Facebook/Instagram identities, score Afghan-business evidence, and create accepted/rejected/manual-review states. Link verified businesses to known Meta advertiser Page IDs.

---

# 47. Fourth Carcer/Codex Task

> Build the media-intelligence and product/offer pipeline. Deduplicate images, run local OCR, extract phone numbers, prices, currencies, locations, brands/models, create canonical products, create offers linked to sellers, and maintain historical prices. Use deterministic methods first and send only uncertain records to a configurable AI provider.

---

# 48. Fifth Carcer/Codex Task

> Build the AI query layer using SQL plus semantic retrieval. The agent must answer questions using structured data and source evidence, distinguish products from offers, support price/trend queries, and return links to original ads/posts/media.

---

# 49. Operational Metrics

Track:

```text
verified_businesses
verified_social_accounts
verified_meta_advertisers
active_advertisers
new_businesses_per_week
new_advertisers_per_week
new_products_per_week
new_offers_per_day
posts_collected
ads_collected
unique_creatives
duplicate_ratio
OCR_success_rate
product_resolution_confidence
offer_extraction_rate
collector_success_rate
media_download_success
failed_jobs
retry_success
canary_status
cost_per_1000_new_records
source_coverage
province_coverage
category_coverage
```

---

# 50. What Not to Build Initially

Do not start with:

- Kubernetes,
- Kafka,
- massive microservice architecture,
- Elasticsearch unless actually needed,
- phone farms,
- dozens of feed accounts,
- full video archival of everything,
- full comment archival,
- paid AI on every record,
- expensive proxy infrastructure,
- permanent dependence on Apify.

These can be added when measured requirements justify them.

---

# 51. Long-Term Platform Value

After months/years, the platform can become a uniquely valuable Afghanistan market-intelligence dataset.

Potential intelligence:

- business discovery,
- competitor tracking,
- product availability,
- product-price history,
- new-product detection,
- seller/importer mapping,
- geographic activity,
- advertising intensity,
- market trends,
- used-equipment monitoring,
- commercial contact discovery,
- sourcing opportunities,
- brand penetration,
- product-category growth,
- demand signals.

The platform should become increasingly valuable simply because it preserves public commercial activity that later disappears.

---

# 52. Final Recommended Architecture

```text
                     DISCOVERY SOURCES
                           |
     +---------------------+----------------------+
     |                     |                      |
     v                     v                      v
Directories/Web       Meta AF discovery      Feed sensors
     |                     |                      |
     +---------------------+----------------------+
                           |
                           v
                 AFGHAN BUSINESS REGISTRY
                           |
                           v
                 SOCIAL IDENTITY GRAPH
                           |
                           v
              VERIFIED BUSINESS/SOURCE IDs
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
 Meta Ad Library       Facebook pages     Instagram accounts
        |                  |                  |
        +------------------+------------------+
                           |
                           v
                   RAW SOURCE ARCHIVE
                           |
                           v
                    NORMALIZATION
                           |
            +--------------+--------------+
            |                             |
            v                             v
        MEDIA PIPELINE                TEXT/METADATA
            |                             |
            +--------------+--------------+
                           |
                           v
                   ENTITY EXTRACTION
                           |
       +-------------------+-------------------+
       |                   |                   |
       v                   v                   v
   BUSINESSES           PRODUCTS             OFFERS
       |                   |                   |
       +-------------------+-------------------+
                           |
                           v
                  AFGHAN COMMERCE GRAPH
                           |
               +-----------+-----------+
               |                       |
               v                       v
          SQL ANALYTICS          VECTOR SEARCH
               |                       |
               +-----------+-----------+
                           |
                           v
                      AI QUERY AGENT
                           |
                           v
                          USER
```

Supporting operational architecture:

```text
                      CARCER
                         |
            schedules / state / retries
                         |
                         v
                       CODEX
              build / repair / test
                         |
                         v
                   COLLECTOR FLEET
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       own HTTP       Playwright     provider fallback
       collectors      collectors      when necessary
```

The design goal is:

> **Low acquisition cost, high data ownership, incremental collection, autonomous maintenance, measurable coverage, and AI-first access to Afghanistan commercial intelligence.**

---

# 53. Practical End State

When mature, the user should no longer need to manually scroll through Facebook or Instagram for most routine market-research questions.

Instead of manually searching:

```text
Facebook
Instagram
marketplaces
business pages
```

the user should ask:

> What farm-related products are currently available in Afghanistan?

> Find every seller of this machine.

> Which products are becoming more common?

> What are the current prices?

> What new electronics appeared this month?

> Which businesses imported or advertise products from China?

The system searches its own accumulated structured and semantic dataset, returns source-backed results, and sends the user back to Facebook/Instagram only when direct source inspection is useful.

This is the intended end state of Version 4.
