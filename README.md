# JobYoinker

**JobYoinker** is a full-stack app I built to solve a problem I kept running into: finding job posts is a huge pain. Jobs are scattered across company websites and LinkedIn posts often not even in the official job tab. Manually checking all of this is a nightmare.

**Check it out live:** [https://jobscraper-collection.vercel.app/](https://jobscraper-collection.vercel.app/)

---

## Why I Built JobYoinker

* Companies post jobs in different places:

  * Some on their career pages
  * Some on LinkedIn, but as normal posts, not official job listings
* No single place has everything, and finding jobs manually is slow

JobYoinker scrapes all of this automatically, collects it in one place, and lets you filter easily so you can see new and relevant jobs fast.

---

## How It Works

```text
Scraper (Cheerio / Playwright)
        ↓
Normalization Layer
        ↓
PostgreSQL (Drizzle ORM)
        ↓
API Layer
        ↓
Next.js Frontend
```

The system is built to be:

* **Resilient**: one scraper failing doesn’t break the rest
* **Flexible**: adding a new company is easy
* **Efficient**: minimal database calls, fast scraping

---

## Scraper System

### Core Design

I made a core scraper framework so adding new companies doesn’t require rewriting everything.

* Each company has **one source of truth**: either the career page or LinkedIn
* This avoids duplicates since job titles and URLs aren’t consistent

### Career Page Scraping

* **Cheerio + Axios** for static pages
* **Playwright** for dynamic pages

Cheerio is lightweight and fast. Playwright reuses a single browser with multiple contexts to save memory. If a job has a detail page with extra info, I can scrape that too.

### LinkedIn Scraping

* Uses a dedicated LinkedIn account
* Scraping is triggered by following companies
* Only grabs jobs that are **2 week old or less** (configurable)

> LinkedIn bot protection is tricky and needs a separate doc.

### Handling Failures

* If one company fails, the rest continue
* If a job detail fails, the main job still gets saved
* Common failures: timeouts or LinkedIn protections
* All failures are logged for manual checking

---

## Preventing Duplicates

* Company names are normalized
* Unique constraints on company name and job URL
* Clear rules when adding new companies or switching sources

---

## Database Layer

* **PostgreSQL** with **Drizzle ORM**

Performance is important:

* Single transaction per scraper run
* Batch inserts and lookups
* Minimal network calls

During a full scrape:

* Only 1 network request ideally
* 14 SQL steps in a single transaction

This makes the database very efficient.

---

## Job Expiration

Each scraper run:

1. Collects all current jobs
2. Compares with existing database records
3. Marks jobs as expired if they’re no longer listed

Scrapers run:

* Twice a day via GitHub Actions
* Can also run manually

---

## API Layer

* Dedicated API routes, not relying on React server components
* Client-heavy filtering handled with TanStack Query

### Rate Limiting (Redis)

* Daily limit
* Burst limit (default 100ms)
* Based on IP

---

## Frontend

* Uses **TanStack Query** with server-side prefetch + client hydration
* Client-side filtering is fast and responsive

Custom wrappers handle differences between server and client fetching.

### State Management

* **Zustand** with cookie persistence
* Keeps client state synced with server
* Pages using this setup are dynamic (can’t be static)

### UI & Testing

* **shadcn/ui** components
* **Vitest** for tests, since many utility functions change often and need validation

---

## Takeaways

* Solves a real problem, not just a demo
* Built to be resilient, efficient, and maintainable
* Scales well as more companies are added

JobYoinker focuses on making job hunting **less painful and more automated**.
