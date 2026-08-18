---
title: "HLD - Distributed Web Crawler"
subject: "High Level Design"
module: "Classic System Design Core Problems"
difficulty: "Advanced"
prerequisites: "[[System Design Interview Framework - 4-Step Blueprint]], [[Back-of-the-Envelope Calculations - Throughput, Storage, and Bandwidth Estimation]]"
related: "[[Consistent Hashing - Hash Rings, Virtual Nodes, and Data Rebalancing]], [[Distributed Caching Strategies - Cache-Aside, Write-Through, Write-Around, Write-Back]], [[Search Engines - Elasticsearch, Inverted Index, TF-IDF, BM25, Relevance Scoring]]"
aliases: ["Web Crawler HLD", "Distributed Web Crawler", "Web Crawler System Design", "URL Frontier", "Politeness Policy", "Spider"]
tags: ["hld", "system-design", "web-crawler", "url-frontier", "politeness", "bloom-filter", "architecture"]
status: "Complete"
---

# HLD — Distributed Web Crawler

## Mental Model

Think of a **Distributed Web Crawler** as a massive fleet of automated digital librarians systematically mapping the entire World Wide Web. 

The crawler begins with a set of seed web page links. It fetches a web page (**HTML Downloader**), extracts all hyperlinks embedded on the page (**URL Extractor**), checks whether those links have already been visited (**URL Deduplication Filter / Bloom Filter**), and enqueues unvisited links into a prioritized queue (**URL Frontier**). 

The crawler operates under a strict **Politeness Policy**: it respects `robots.txt` files and never floods a target web server with 1,000 requests/sec (**Host Rate Limiting**).

---

## 1. Requirement & Capacity Estimation

### A. Functional & Non-Functional Requirements
1. **Scalability & Scale:** Crawl 1 Billion web pages per month.
2. **Politeness & robots.txt:** Respect `robots.txt` and rate-limit requests per domain (e.g. max 1 request/sec per host).
3. **URL Deduplication:** Prevent crawling duplicate URLs or cyclic links.
4. **Content Deduplication:** Detect duplicate web pages (e.g. mirror sites) using document fingerprints (SimHash / MurmurHash).
5. **Robustness & Extensibility:** Handle spider traps, infinite redirects, dynamic JavaScript rendering, and malformed HTML.

### B. Capacity Estimation Math
- **Crawling Throughput:**
  - 1 Billion Pages / Month = $\frac{1,000,000,000}{30 \times 86,400} \approx \mathbf{400 \text{ Pages / Second}}$
  - Peak Throughput = $400 \times 2 = \mathbf{800 \text{ Pages / Second}}$

- **Storage & Bandwidth Math:**
  - Average Web Page Size = $500 \text{ KB}$ (HTML + metadata)
  - Monthly Storage = $10^9 \times 500 \text{ KB} = \mathbf{500 \text{ Terabytes (TB) / Month}}$
  - 5-Year Storage = $500 \text{ TB} \times 12 \times 5 = \mathbf{30 \text{ Petabytes (PB)}}$
  - Network Bandwidth = $400 \text{ pages/sec} \times 500 \text{ KB} = 200 \text{ MB/sec} = \mathbf{1.6 \text{ Gbps Network Bandwidth}}$

---

## 2. System Architecture Diagram

```mermaid
flowchart TD
    Seeds["Seed URLs"] --> Frontier["URL Frontier (Priority & Politeness Queues)"]
    
    Frontier -->|1. Fetch Next Polite URL| Fetcher["HTML Fetcher Worker Pool"]
    
    Fetcher -->|2. Check `robots.txt`| RobotCache["Robots.txt Cache (Redis)"]
    Fetcher -->|3. Download HTML Payload| DNS["DNS Resolver"]
    
    Fetcher -->|4. Store Raw Page| Storage["Raw Blob Storage (AWS S3 / HDFS)"]
    
    Fetcher --> Extractor["URL Extractor & Parser"]
    
    Extractor -->|5. Extract Embedded Links| Filter["URL Deduplication (Bloom Filter)"]
    
    Filter -->|New Unseen Links| Frontier
    Filter -.->|Duplicate Seen Links| Drop["Drop Link"]
```

---

## 3. The URL Frontier Architecture (Politeness + Priority)

The **URL Frontier** is the core component that manages URL scheduling, guaranteeing **Politeness** (never overloading a single host) and **Priority** (crawling high-quality news sites before obscure blogs).

```mermaid
flowchart TD
    subgraph PrioritySelector["1. Priority Component (Quality & Freshness)"]
        InLinks["Incoming Candidate URLs"] --> Prioritizer["Priority Evaluator (PageRank / Freshness)"]
        Prioritizer --> P1["High Priority Queue"] & P2["Medium Priority Queue"] & P3["Low Priority Queue"]
        P1 & P2 & P3 --> QueueSelector["Priority Queue Selector"]
    end

    subgraph PolitenessRouter["2. Politeness Component (Host Rate Limiting)"]
        QueueSelector --> HostRouter["Host Router / Mapper"]
        HostRouter --> HQ1["Host Queue 1 (example.com)"] & HQ2["Host Queue 2 (wikipedia.org)"] & HQ3["Host Queue 3 (github.com)"]
        
        HQ1 & HQ2 & HQ3 --> DelayPoliteness["Politeness Manager\n(Ensures 1-second delay per host queue using Priority Queue)"]
        
        DelayPoliteness --> FetchWorkers["HTML Fetcher Worker Threads"]
    end
```

---

## 4. URL & Content Deduplication

How does the crawler prevent infinite duplicate crawling?

### A. URL Deduplication (Bloom Filter)
Before adding a candidate URL to the URL Frontier, the crawler passes the URL through an in-memory **Bloom Filter**:
- If Bloom Filter returns `FALSE` $\to$ URL is 100% new! Add to URL Frontier and set Bloom Filter bit.
- If Bloom Filter returns `TRUE` $\to$ URL is likely visited. Drop.

```mermaid
flowchart LR
    CandidateURL["Candidate URL: `https://example.com/about`"] --> HashFuncs["3 Independent Hash Functions"]
    HashFuncs --> BitArray["Bit Array: [0, 1, 0, 0, 1, 1, 0, 1]"]
    BitArray --> Check{"All 3 Bits Set to 1?"}
    Check -->|NO| NewURL["NEW URL -> Enqueue in URL Frontier"]
    Check -->|YES| Duplicate["DUPLICATE URL -> Drop"]
```

---

### B. Content Deduplication (SimHash / Document Fingerprinting)
Websites often serve identical content on different URLs (`http://site.com/item?id=1` vs `http://site.com/print/item?id=1`). 
1. The crawler computes a 64-bit **SimHash fingerprint** for the HTML text body.
2. If two documents have a **Hamming Distance $\le 3$**, they are classified as duplicate content and stored once.

---

## 5. Failure Modes and Trade-offs

1. **Spider Traps & Infinite Dynamic Loops** — Malicious or broken websites generating infinite URL paths (`http://site.com/foo/bar/foo/bar/foo...`). The crawler gets trapped in an infinite loop downloading gigabytes of junk. *Mitigation*: Limit maximum URL path depth (max 10 levels) and enforce maximum URL string length (max 2,048 chars).
2. **DNS Resolver Bottleneck** — Fetching 400 pages/sec requires resolving 400 domain names/sec. Standard synchronous DNS lookups block fetcher threads. *Mitigation*: Maintain a local **DNS Cache Cluster** and use asynchronous non-blocking DNS resolvers (c-ares).
3. **Dynamic JavaScript SPA Rendering** — Modern web pages (React / Vue) render content dynamically via client-side JavaScript (`<div id="root"></div>`). Traditional HTML parsers extract 0 links. *Mitigation*: Use headless browser renderers (Puppeteer / Playwright) for dynamic domains.

---

## 6. Active-Recall Prompts

1. **Explain the 2 components of the URL Frontier (Priority Selection vs. Politeness Rate Limiting).**
2. **How does a Bloom Filter achieve $O(1)$ space-efficient URL deduplication with zero false negatives?**
3. **What is a Spider Trap, and how do depth limits and URL length bounds mitigate it?**
4. **How does SimHash fingerprinting detect duplicate web page content across different URLs?**

---

## Related Notes

- [[Consistent Hashing - Hash Rings, Virtual Nodes, and Data Rebalancing]]
- [[Distributed Caching Strategies - Cache-Aside, Write-Through, Write-Around, Write-Back]]
- [[Search Engines - Elasticsearch, Inverted Index, TF-IDF, BM25, Relevance Scoring]]
- [[System Design Interview Framework - 4-Step Blueprint]]

> **Interview Style Question:** *"Design a Distributed Web Crawler to index 1 Billion web pages per month. Calculate storage math (30 PB 5-year), design a polite URL Frontier architecture with per-host queues, demonstrate Bloom Filter URL deduplication, detect spider traps, and handle dynamic JavaScript rendering."*

---
