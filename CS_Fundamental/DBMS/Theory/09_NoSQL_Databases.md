# NoSQL Databases

## 1. What is NoSQL?

**NoSQL (Not Only SQL)** is a category of databases that store and retrieve data using models other than the traditional relational (table-based) model.

NoSQL databases are designed for:
- **Massive scale** — billions of records across thousands of servers
- **Flexible schema** — structure can change without migrations
- **High availability** — built to survive node failures
- **Varied data shapes** — documents, graphs, key-value pairs, time series

> "Not Only SQL" — not a replacement for SQL, but a different tool for different problems.

---

## 2. SQL vs NoSQL — Core Differences

### Data Model

```
SQL (Relational):                    NoSQL (Document):

Table: Users                         Collection: users
┌────┬───────┬─────┐                 {
│ id │ name  │ age │                   "_id": 1,
├────┼───────┼─────┤                   "name": "Alice",
│ 1  │ Alice │ 25  │                   "age": 25,
│ 2  │ Bob   │ 30  │                   "address": {
└────┴───────┴─────┘                     "city": "Mumbai",
                                         "pin": "400001"
Table: Address                         },
┌────┬────────┬────────┐               "orders": [
│ id │ userId │ city   │                 {"item": "Phone", "price": 30000},
├────┼────────┼────────┤                 {"item": "Laptop", "price": 80000}
│ 1  │ 1      │ Mumbai │               ]
└────┴────────┴────────┘             }
```

SQL splits related data across tables and joins them.  
NoSQL embeds related data inside one document — no joins needed.

---

### Full Comparison Table

| Feature | SQL | NoSQL |
|---|---|---|
| **Data model** | Tables with rows and columns | Documents, key-value, graphs, columns |
| **Schema** | Fixed — must define before inserting | Flexible — each record can have different fields |
| **Relationships** | JOINs across tables | Embedded documents or application-level joins |
| **Query language** | Standardised SQL | DB-specific API (MongoDB query, Redis commands) |
| **Scaling** | Vertical (bigger machine) | Horizontal (more machines) |
| **Transactions** | Full ACID | BASE (eventual consistency) — some support ACID now |
| **Consistency** | Strong consistency | Tunable — eventual to strong |
| **Best for** | Structured, relational data | Unstructured, hierarchical, high-scale data |
| **Examples** | MySQL, PostgreSQL, Oracle | MongoDB, Redis, Cassandra, Neo4j |

---

## 3. How SQL Scales vs How NoSQL Scales

### Vertical Scaling (Scale Up)

**Add more power to the same machine** — bigger CPU, more RAM, faster disk.

```
Before:                    After:
┌─────────────────┐        ┌─────────────────────┐
│  Server         │        │  Server (upgraded)  │
│  4 CPU cores    │  →     │  32 CPU cores       │
│  16 GB RAM      │        │  256 GB RAM         │
│  500 GB SSD     │        │  4 TB SSD           │
└─────────────────┘        └─────────────────────┘
  handles 1000 req/s          handles 8000 req/s
```

**Real Example:** MySQL server is slow → move to a larger AWS RDS instance with more vCPUs and RAM. Same single machine, just more powerful.

**Problems:**
- **Hard ceiling** — no machine can grow forever
- **Expensive** — high-end hardware costs exponentially more
- **Single point of failure** — if this one machine goes down, everything goes down
- **Downtime needed** — upgrading hardware often requires restarting the server

---

### Horizontal Scaling (Scale Out)

**Add more machines to share the load** — distribute data and traffic across many nodes.

```
Before:                    After:
┌─────────────┐            ┌───────────┐ ┌───────────┐ ┌───────────┐
│   Server    │     →      │  Node 1   │ │  Node 2   │ │  Node 3   │
│ all data    │            │ Users A-H │ │ Users I-P │ │ Users Q-Z │
│ all traffic │            └───────────┘ └───────────┘ └───────────┘
└─────────────┘              1000 req/s   1000 req/s     1000 req/s
  1000 req/s                            = 3000 req/s total
```

**Real Example:** Twitter stores tweets in Cassandra — data is sharded across hundreds of nodes by `user_id`. When load increases, they add more nodes live with no downtime.

**How Data is Split (Sharding):**
```
Shard key = user_id

user_id 1–1000     → Node 1
user_id 1001–2000  → Node 2
user_id 2001–3000  → Node 3

Query for user_id=1500 → routed directly to Node 2
```

**Problems:**
- **Complexity** — queries that span multiple nodes are hard (cross-shard joins)
- **Consistency** — keeping all nodes in sync is difficult
- **Resharding** — when a new node is added, data must be redistributed

---

### Side-by-Side Comparison

| | Vertical Scaling | Horizontal Scaling |
|---|---|---|
| How | Bigger machine | More machines |
| Cost | Exponentially expensive | Linearly expensive (commodity hardware) |
| Ceiling | Hard limit (max hardware) | Theoretically unlimited |
| Failure risk | Single point of failure | One node dies, others continue |
| Complexity | Simple — same machine | Complex — distributed system |
| Downtime | Often needs restart | No downtime — add nodes live |
| Used by | SQL databases (mostly) | NoSQL databases (mostly) |
| Example | Bigger AWS RDS instance | Cassandra adding nodes |

### Which Databases Use What

```
Vertical Scaling:             Horizontal Scaling:
MySQL      ──→ Scale Up       Cassandra  ──→ Add nodes
PostgreSQL ──→ Scale Up       MongoDB    ──→ Sharding
Oracle     ──→ Scale Up       Redis      ──→ Redis Cluster
                              DynamoDB   ──→ Auto-scales
```

> Most SQL databases support only vertical scaling by default because JOINs require all related data to be on the same machine. NoSQL avoids JOINs — so data can safely live on different machines.

---

## 4. Types of NoSQL Databases

### 4.1 Key-Value Store

**Structure:** A giant dictionary — every value is stored and retrieved by a unique key.

```
Key          Value
──────────────────────────────────────────
"user:1"  →  {"name": "Alice", "age": 25}
"session:abc123" → {"userId": 1, "expiry": 3600}
"counter:views"  → 10482
```

**How it works internally:**
- Hash table or B-Tree maps keys to values
- Values are opaque blobs — the DB doesn't know or care about structure inside
- All operations: `GET key`, `SET key value`, `DEL key`, `EXPIRE key seconds`

**Characteristics:**
- Fastest read/write — O(1) lookup
- No query on value fields — can only look up by exact key
- Values can be strings, JSON, binary, lists, sets

**Examples:** Redis, DynamoDB, Memcached

**Use cases:**
- Session storage (`session:userId`)
- Caching (store DB query results by cache key)
- Rate limiting (increment counter per IP per minute)
- Real-time leaderboards (Redis sorted sets)

---

### 4.2 Document Store

**Structure:** Stores data as **self-describing documents** (JSON/BSON) — each document can have a different structure.

```json
// User document
{
  "_id": "u001",
  "name": "Alice",
  "email": "alice@example.com",
  "address": {
    "city": "Mumbai",
    "pin": "400001"
  },
  "tags": ["premium", "verified"],
  "orders": [
    { "orderId": "o1", "item": "Phone", "price": 30000 },
    { "orderId": "o2", "item": "Laptop", "price": 80000 }
  ]
}
```

**How it works internally:**
- Documents grouped into **collections** (like tables but schema-free)
- Each document has a unique `_id`
- Indexed on `_id` by default (B-Tree)
- Supports indexes on any nested field
- Queries can filter on any field: `{city: "Mumbai"}`, `{price: {$gt: 50000}}`

**Characteristics:**
- Flexible schema — add/remove fields per document without migration
- Embedded documents eliminate most JOINs
- Rich query language — filter, project, aggregate, sort
- Horizontal scaling via sharding on a shard key

**Examples:** MongoDB, CouchDB, Firestore

**Use cases:**
- Product catalogs (each product has different attributes)
- User profiles (varying fields per user type)
- Content management systems
- Mobile app backends

---

### 4.3 Wide-Column Store (Column-Family)

**Structure:** Data organized as rows with dynamic columns — rows in the same table can have completely different columns.

```
SQL Table (rigid):              Wide-Column (flexible):

Row │ col1 │ col2 │ col3        Row │ Columns stored
────┼──────┼──────┼──────       ────┼──────────────────────────────────
 1  │  A   │  B   │  C           1  │ {name:Alice, age:25, city:Mumbai}
 2  │  D   │  E   │  F           2  │ {name:Bob, score:90, level:5}
 3  │  G   │ NULL │  I           3  │ {name:Carol, email:c@x.com}
                                 (each row stores only its own columns)
```

**How it works internally (Cassandra):**
- Data partitioned across nodes using **consistent hashing** on the **partition key**
- Within a partition, rows sorted by **clustering key** — fast range scans within a partition
- Writes go to an in-memory **Memtable** first, then flushed to immutable **SSTable** files on disk
- No master node — every node is equal (peer-to-peer)
- Replication factor controls how many nodes store each partition

```
Cassandra Data Model:

Keyspace (like a database)
  └── Table
        └── Partition Key  → determines which node stores the data
              └── Clustering Key → ordering within the partition
                    └── Columns   → the actual data
```

**Examples:** Cassandra, HBase, Google Bigtable

**Use cases:**
- Time-series data (IoT sensor readings, stock prices)
- Event logging (billions of events per day)
- Analytics at scale (write-heavy workloads)
- Netflix watch history, Instagram activity feeds

---

### 4.4 Graph Database

**Structure:** Data stored as **nodes** (entities) and **edges** (relationships), each with properties.

```
(Alice) ──[FRIENDS_WITH]──→ (Bob)
  │                           │
  └──[WORKS_AT]──→ (Google)   └──[WORKS_AT]──→ (Amazon)
                    │
              [LOCATED_IN]──→ (California)
```

**How it works internally:**
- Each node stores a pointer to its first relationship
- Each relationship stores pointers to both nodes and the next relationship for each node
- Traversal follows pointers directly — no JOIN computation needed
- Query language: **Cypher** (Neo4j), **Gremlin**

```cypher
-- Find friends of Alice who work at the same company as her
MATCH (alice:Person {name:"Alice"})-[:FRIENDS_WITH]->(friend)
      -[:WORKS_AT]->(company)<-[:WORKS_AT]-(alice)
RETURN friend.name
```

**Characteristics:**
- Relationship traversal is O(1) per hop — constant time, not dependent on table size
- SQL JOIN equivalent gets slower as data grows; graph traversal stays fast
- Natural fit for highly connected data

**Examples:** Neo4j, Amazon Neptune, ArangoDB

**Use cases:**
- Social networks (friends of friends, recommendations)
- Fraud detection (detect suspicious transaction patterns)
- Knowledge graphs
- Route/path finding (Google Maps-style)

---

## 5. CAP Theorem

Every distributed database can guarantee only **2 out of 3** properties:

```
              Consistency
                  /\
                 /  \
                /    \
               /  ??  \
              /________\
     Availability    Partition
                     Tolerance
```

| Property | Meaning |
|---|---|
| **Consistency (C)** | Every read returns the most recent write — all nodes see the same data |
| **Availability (A)** | Every request gets a response (may not be the latest data) |
| **Partition Tolerance (P)** | System keeps working even if network partitions split nodes |

> In a real distributed system, **network partitions always happen** — so P is non-negotiable. The real choice is **C vs A**.

### CAP Choices

| Choice | Behaviour | Examples |
|---|---|---|
| **CP** (Consistency + Partition) | Returns error if it can't guarantee latest data | HBase, Zookeeper, MongoDB (default) |
| **AP** (Availability + Partition) | Returns data even if it might be stale | Cassandra, CouchDB, DynamoDB |
| **CA** (Consistency + Availability) | Only works if no partitions — not realistic in distributed systems | Traditional SQL (single node) |

### Example

```
Node A and Node B both hold user balance.
Network partition: A and B can't communicate.

CP choice: Return error — "Cannot guarantee latest data"
AP choice: Return whatever Node A has (may be stale) — availability wins
```

---

## 6. BASE vs ACID

NoSQL systems follow **BASE** instead of ACID:

| | ACID (SQL) | BASE (NoSQL) |
|---|---|---|
| **Full form** | Atomicity, Consistency, Isolation, Durability | Basically Available, Soft state, Eventually consistent |
| **Consistency** | Strong — always consistent | Eventual — becomes consistent over time |
| **Availability** | May sacrifice for consistency | Always available |
| **Transactions** | Full multi-row transactions | Limited or no transactions |
| **Design goal** | Correctness | Availability + Performance |

### Eventual Consistency — What it means

```
User updates profile photo on Node A.
                ↓
Node A → Node B replication (takes 200ms)
                ↓
Another user reads from Node B during those 200ms
→ sees the OLD photo  ← eventually consistent

After replication completes:
→ Both nodes have new photo ← consistent now
```

The data is not immediately consistent — but it **will become consistent eventually**.

---

## 7. When to Use NoSQL vs SQL

### Use SQL When:

| Scenario | Reason |
|---|---|
| Data is structured and relational | Tables and JOINs are exactly right for this |
| ACID transactions are critical | Banking, payments, inventory — correctness over speed |
| Complex queries with JOINs | SQL is expressive and optimised for this |
| Schema is stable | Relational schema design pays off when structure doesn't change |
| Reporting and analytics on structured data | SQL aggregations are powerful |
| Team knows SQL well | Mature tooling, well-understood |

**Examples:** Banking systems, ERP, e-commerce order management, HR systems

---

### Use NoSQL When:

| Scenario | Reason | NoSQL Type |
|---|---|---|
| Massive read/write throughput | Horizontal scaling across nodes | Cassandra, DynamoDB |
| Flexible / evolving schema | Product catalog, user profiles vary by type | MongoDB |
| Hierarchical / nested data | Avoid expensive JOINs, embed related data | MongoDB |
| Caching / session management | Sub-millisecond key lookup | Redis |
| Highly connected data | Relationship traversal, graph queries | Neo4j |
| Time-series / event data | Billions of writes per day, time-ordered | Cassandra, InfluxDB |
| Real-time applications | Low latency over strong consistency | Redis, DynamoDB |
| Geographic distribution | Multi-region with local reads | Cassandra, CouchDB |

**Examples:** Twitter timelines (Cassandra), Netflix recommendations (Graph), Uber location tracking (Redis), Product catalog (MongoDB)

---

## 8. Real-World Architecture — SQL + NoSQL Together

Most large systems use **both** — each for what it does best:

```
User places an order:

┌──────────────────────────────────────────────────────────┐
│                    Application Layer                      │
└───────┬──────────────────┬──────────────────┬────────────┘
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────────┐  ┌──────────────┐
│  PostgreSQL  │  │     MongoDB      │  │    Redis     │
│              │  │                  │  │              │
│  Orders      │  │  Product catalog │  │  Sessions    │
│  Payments    │  │  User profiles   │  │  Cart cache  │
│  Inventory   │  │  Reviews         │  │  Rate limits │
│              │  │                  │  │              │
│  (ACID txns  │  │  (flexible       │  │  (sub-ms     │
│   critical)  │  │   schema)        │  │   lookup)    │
└──────────────┘  └──────────────────┘  └──────────────┘
```

---

## 9. Summary

| | Key-Value | Document | Wide-Column | Graph |
|---|---|---|---|---|
| **Data shape** | key → blob | JSON documents | Rows with dynamic columns | Nodes + edges |
| **Query** | By key only | By any field | By partition/cluster key | Traversal / pattern match |
| **Best for** | Cache, session | Profiles, catalog | Time-series, events | Social, fraud detection |
| **Examples** | Redis | MongoDB | Cassandra | Neo4j |
| **Scale** | Horizontal | Horizontal | Horizontal | Vertical (mostly) |

---

## 10. Interview Questions

1. **What is NoSQL and why was it created?**
   → NoSQL databases were created to handle massive scale, flexible schemas, and high availability that traditional relational DBs struggle with — especially at internet scale.

2. **What is the CAP theorem?**
   → A distributed system can guarantee only 2 of 3: Consistency, Availability, Partition Tolerance. Since partitions always happen in distributed systems, the real choice is C vs A.

3. **What is eventual consistency?**
   → Data may be temporarily inconsistent across nodes after a write, but all nodes will converge to the same value eventually once replication completes.

4. **What is the difference between ACID and BASE?**
   → ACID (SQL) prioritises correctness — strong consistency, full transactions. BASE (NoSQL) prioritises availability — eventual consistency, always responds even if data is stale.

5. **When would you choose MongoDB over MySQL?**
   → When schema is flexible (product catalog with varying attributes), data is hierarchical (nested documents avoid JOINs), or scale requires horizontal sharding.

6. **When would you choose Cassandra over MongoDB?**
   → When write throughput is extremely high (billions of events/day), data is time-series or event-based, and you need multi-region availability with no single point of failure.

7. **Why are graph databases better than SQL for relationship queries?**
   → SQL JOINs get slower as table size grows (O(n) at minimum). Graph traversal follows direct pointers between nodes — O(1) per hop regardless of total data size.

8. **What is sharding?**
   → Splitting data across multiple nodes based on a shard/partition key — each node stores a subset of the data. Enables horizontal scaling by distributing load.

9. **Can NoSQL databases support transactions?**
   → Some do — MongoDB 4.0+ supports multi-document ACID transactions, DynamoDB supports transactions within a single table. But full cross-table transactions are still limited compared to SQL.

10. **What is the difference between a document store and a key-value store?**
    → Key-value stores treat the value as an opaque blob — you can only retrieve it by key. Document stores understand the structure inside the document — you can query, index, and filter on any nested field.
