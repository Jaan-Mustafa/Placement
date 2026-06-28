# Design a Web Crawler (HLD)

A **web crawler** (aka spider/bot) systematically browses the web: starts from some seed URLs, downloads pages, extracts links from them, and repeats — discovering and downloading more and more pages. Used by search engines (Google), web archiving, data mining, monitoring, etc.

## 1. Requirements

### Functional
- Given a set of **seed URLs**, download all reachable web pages.
- **Extract links** from downloaded pages and add them to the crawl queue.
- Store the downloaded content (for indexing / processing).

### Non-Functional
- **Scalability** — the web has billions of pages; must crawl in parallel across many machines.
- **Politeness** — don't overload any single website (rate-limit per host).
- **Robustness** — handle bad HTML, dead links, timeouts, traps, crashes.
- **Extensibility** — easy to add new content types (images, PDFs) later.
- **Freshness** — re-crawl pages periodically to keep content up to date.

### Back-of-envelope (typical interview numbers)
```
1 billion pages/month  →  ~400 pages/second average  →  ~800 QPS peak
Avg page size 500 KB   →  1B × 500KB = 500 TB/month of storage
Store 5 years          →  500 TB × 60 ≈ 30 PB
```

## 2. High-Level Architecture

```
        ┌──────────────┐
        │  Seed URLs   │
        └──────┬───────┘
               ▼
        ┌──────────────┐      ┌─────────────────────┐
        │  URL Frontier│◄─────│  URL Filter / Dedup │
        │  (the queue) │      │  (seen before?)     │
        └──────┬───────┘      └─────────▲───────────┘
               ▼                        │
        ┌──────────────┐                │
        │ HTML Downloader│  ── fetch ──► Internet
        │  (workers)    │                │
        └──────┬───────┘                │
               ▼                        │
        ┌──────────────┐                │
        │ Content Parser│ (validate HTML)│
        └──────┬───────┘                │
               ▼                        │
   ┌────────────────────┐              │
   │ Content Seen? (dedup)│            │
   └──────┬─────────────┘              │
          ▼                            │
   ┌──────────────┐   ┌──────────────┐│
   │ Content Store │   │ Link Extractor├┘
   └──────────────┘   └──────────────┘
                          extracts URLs → back to URL Filter → Frontier (loop)
```

## 3. Components (the core flow)

**1. Seed URLs** — starting points. Chosen smartly (e.g. popular sites, or by topic/region/category) to reach as much of the web as possible.

**2. URL Frontier** — the heart of the crawler: a queue of URLs *to be crawled*. Not a simple FIFO — it manages **politeness** (don't hammer one host) and **priority** (crawl important pages first). *(See section 4.)*

**3. HTML Downloader (Fetcher)** — fetches the page for a URL over HTTP. Many parallel workers across many machines. Respects **robots.txt** (rules on what a site allows crawling).

**4. DNS Resolver** — resolves the URL's hostname → IP before fetching. DNS is slow, so results are **cached** (a common bottleneck).

**5. Content Parser** — parses & validates the downloaded HTML (malformed pages waste resources and can crash workers).

**6. "Content Seen?" (content dedup)** — many pages have duplicate/near-duplicate content (mirrors, copies). Hash the content and skip if seen. Saves storage and compute. (Often uses a hash/checksum or a similarity hash for near-dups.)

**7. Content Storage** — store the page content. Mostly on disk (huge volume); hot/popular pages cached in memory.

**8. Link Extractor** — pulls all hyperlinks (`<a href>`) out of the page, converts relative URLs to absolute.

**9. URL Filter** — excludes unwanted URLs (blacklisted sites, certain file types, error URLs).

**10. "URL Seen?" (URL dedup)** — avoid crawling the same URL twice (and infinite loops). Track seen URLs with a **Bloom filter** or hash set (Bloom filter = space-efficient, allows tiny false-positive rate).

**11. Loop** — new, unseen URLs go back into the **URL Frontier**, and the whole cycle repeats.

## 4. The URL Frontier (most important component)

A naive queue isn't enough. It must handle two things:

### Politeness (don't overload a website)
- Send requests to the **same host one at a time**, with a delay between them.
- Implementation: **per-host queues** — one sub-queue per hostname, and a worker for that host pulls with a delay. URLs from the same domain map to the same queue.

### Priority (crawl important pages first)
- Not all pages are equal — a news homepage matters more than an obscure page.
- A **prioritizer** assigns priority (by PageRank, traffic, update frequency) → URL goes into a high/low priority queue.

```
URL Frontier (simplified):

  Prioritizer → [ Priority queues f1..fn ] ─┐
                                            ▼
                            (front queues select by priority)
                                            ▼
                  Back queues b1..bn (one per host → politeness)
                                            ▼
                              Worker pulls (with per-host delay)
```

So: **front queues = priority**, **back queues = politeness**. The Frontier balances "crawl important stuff" with "be polite to each site."

## 5. Key Design Problems & Solutions

| Problem | Solution |
|---|---|
| **Crawling same URL twice / loops** | "URL Seen?" check with **Bloom filter** / hash set |
| **Duplicate content across pages** | "Content Seen?" with content **hash / checksum** |
| **Overloading a website** | **Politeness** — per-host queues + delays, respect `robots.txt` |
| **Crawler traps** (infinite URL spaces, calendars) | Max URL length, max depth, detect/avoid dynamic loops |
| **Slow DNS** | **DNS cache** |
| **Huge scale** | Distribute frontier + downloaders across many machines (shard by host) |
| **Robustness (crashes)** | Checkpoint frontier state to disk; retry failed fetches; consistent hashing for worker assignment |
| **Freshness** | Re-crawl based on page change frequency (re-add to frontier periodically) |

## 6. BFS vs DFS — how to traverse

The web is a giant graph (pages = nodes, links = edges). Crawlers use **BFS** (breadth-first), implemented via the **FIFO-ish URL Frontier queue**:
- BFS explores level by level → naturally spreads across many sites (good for politeness & coverage).
- DFS could go very deep into one site (bad: hammers one host, can fall into traps).

> The URL Frontier is essentially a smart BFS queue with politeness + priority layered on top.

## 7. Politeness & robots.txt
- **robots.txt** — a file at `site.com/robots.txt` where a site declares what crawlers may/may not access. A polite crawler **reads and obeys it** (cache it per host).
- **Rate limiting per host** — a delay (e.g. 1 request/sec/host) so you never overwhelm a server.
- Identify yourself with a **User-Agent** string.

## 8. Scaling the Crawler
- **Distributed downloaders** — many crawler servers, each handling a subset of hosts (shard by hostname via **consistent hashing**).
- **Distributed frontier** — frontier partitioned across machines; each owns certain hosts (keeps politeness correct — one host handled by one machine).
- **Geo-distribution** — place crawl servers near the sites they crawl to cut latency.

## One-Line Summary
**A web crawler is a BFS over the web graph: seed URLs → URL Frontier (queue with politeness + priority) → downloader fetches pages → parser + link extractor pull new URLs → dedup (URL Seen via Bloom filter, Content Seen via hashing) → store content → loop, all distributed across many machines and polite to each host.**

---

## URL Frontier — Full Internals (Mercator design)

The Frontier is NOT one queue. It's a multi-stage structure. All of these are sub-components **of the URL Frontier**:

```
            New URLs in
                │
                ▼
        ┌───────────────┐
        │  PRIORITIZER  │   ← assigns priority to each URL
        └───────┬───────┘
                ▼
   ┌─────────────────────────┐
   │  FRONT QUEUES  f1...fn   │   ← one queue per priority level
   │  (handle PRIORITY)       │     (f1 = highest ... fn = lowest)
   └───────────┬─────────────┘
               ▼
        ┌───────────────┐
        │ FRONT QUEUE   │   ← picks which front queue to read from
        │   SELECTOR    │     (biased toward higher priority)
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ BACK QUEUE    │   ← routes each URL to the back queue for its HOST
        │   ROUTER      │
        └───────┬───────┘
                ▼
   ┌─────────────────────────┐
   │  BACK QUEUES  b1...bn    │   ← each queue = exactly ONE host
   │  (handle POLITENESS)     │
   └───────────┬─────────────┘
               ▼
        ┌───────────────┐
        │ BACK QUEUE    │   ← picks next back queue/host to crawl,
        │   SELECTOR    │     respecting per-host delay
        └───────┬───────┘
                ▼
          Worker fetches the URL
```

### The two halves (key mental model)
```
FRONT side  = PRIORITY    → "crawl the important pages first"
              (prioritizer + front queues + front selector)

BACK side   = POLITENESS  → "don't hammer any single host"
              (back router + back queues + back selector)
```

### Component-by-component

| Component | Job | Handles |
|---|---|---|
| **Prioritizer** | Scores each incoming URL (PageRank, traffic, update freq) → drops it in matching priority queue | Priority |
| **Front queues (f1…fn)** | One queue per priority level (f1 high … fn low) | Priority |
| **Front queue selector** | Picks which front queue to dequeue next | Priority |
| **Back queue router** | Routes a URL to the back queue for its **host** | Politeness |
| **Back queues (b1…bn)** | Each holds URLs for **exactly one host** | Politeness |
| **Back queue selector** | Picks next back queue (host) to crawl, enforcing per-host delay | Politeness |

### Front Queue Selector — what it does
Decides **which front (priority) queue to pull from next**, biased toward higher-priority queues so important pages crawl first.

```
   f1 (high) ──► picked very often   ████████
   f2        ──► picked often         █████
   f3        ──► picked sometimes      ███
   fn (low)  ──► picked rarely          █
```

**Crucial subtlety — avoid starvation:** it uses a **weighted/probabilistic** pick, NOT strict "always highest first" — otherwise low-priority queues would never get crawled.
```
Strict priority (BAD):   always drain f1 → fn never runs = starvation ❌
Weighted (GOOD):         f1 50%, f2 30%, f3 15%, fn 5% → high dominates, low still progresses ✓
```

### Back Queue Router — what it does
Reads each URL's **host** and routes it into the single back queue dedicated to that host.

```
   reddit.com/page1, reddit.com/page2, wikipedia.org/A
            │ router reads host
            ▼
   b1 → reddit.com    [page1, page2]   ← all reddit URLs here
   b2 → wikipedia.org [A]              ← all wikipedia URLs here
```

**Key rule: one host → one back queue.** This is the foundation of politeness:
- Each back queue is crawled by one worker with a delay → reddit gets ~1 req/sec → polite ✓
- If reddit's URLs were split across 5 queues/workers → 5 simultaneous hits → hammering ❌

It uses a **mapping table** `host → back queue` (often via **consistent hashing** of the hostname) so a host always lands in the same queue.

---

## How Do We Handle 10,000 Domains? (worker scaling)

**Misconception:** "one host → one back queue" means one dedicated worker per domain → 10k domains = 10k workers. **Wrong.**

### Fact 1: Number of back queues / workers is FIXED and small
≈ a few **hundred** (e.g. 500), NOT the number of domains. It's a tuning parameter.
```
500 back queues / 500 workers   ← fixed
10,000 domains                  ← can be huge
→ At any INSTANT you crawl only ~500 domains; the other 9,500 WAIT in the frontier.
```

### Fact 2: Workers are REUSED across domains over time
A worker isn't married to a domain. When a host's back queue empties, it's refilled with a **new host** and the worker moves on.
```
Worker 1 over time:
   crawls reddit.com → empties → b1 refilled with github.com → then nytimes.com → ...
One worker serves MANY domains, one at a time.
```

> Real invariant: **at any moment a host is crawled by at most ONE worker** — not "each host owns a worker."

### The flow for 10k domains
```
10,000 domains' URLs → all sit in FRONT queues (the backlog)
        ▼ back queue router assigns hosts to free back queues
   500 back queues (1 host each) → only 500 "active" hosts at a time
        ▼
   500 workers crawl them (per-host delay)
        ▼
   a back queue empties → refilled with the NEXT waiting host
```
Like a bank with 500 tellers serving 10,000 customers: customers wait in line, tellers serve one at a time then move to the next.

### Politeness when a worker returns to a host
Each back queue tracks a **timestamp** of when the next request to that host is allowed (e.g. 1 req/sec/host). Politeness is enforced by **time**, not by dedicating a worker.

### Scaling beyond one machine
Shard domains across machines by **consistent hashing of the hostname**:
```
Machine A → reddit.com, github.com, ...   (own frontier + 500 workers)
Machine B → wikipedia.org, nytimes.com, ... (own frontier + 500 workers)
→ 10 machines × 500 = 5,000 concurrent crawls
→ each host handled by exactly ONE machine → politeness preserved
```

### Summary
- NOT one worker per domain. Workers/back queues are a **fixed pool** (~hundreds), reused across domains.
- At any instant you crawl ~(number of back queues) domains; the rest **wait in the frontier**.
- Back queue empties → **refilled with next waiting host** → worker moves on.
- Politeness = "one host crawled by one worker *at a time*" + per-host **delay timestamps**.
- Go bigger by **sharding domains across machines** with consistent hashing on the hostname.


abcdedfghi