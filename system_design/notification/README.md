# System Design: Unread Notification Badge

The red number on the bell icon shows how many unread notifications a user has.
Twist compared to Like Counter: **"Mark all as read" resets the count to 0.**

**Progress:** Day 1 done (Step 1, flows, math) | Day 2 pending (API, DB, Step 3, Step 4)

---

## Step 1: Understand the Problem & Establish Scope

**Functional Requirements:**
* Users receive notifications (unread count goes up by 1).
* Users can view the unread count on the bell icon.
* Users can mark all notifications as read (count becomes 0).
* About 5% of notifications also trigger a push message.

**Non-Functional Requirements:**
* **Scale (Writes):** 100,000 active users/day x 20 notifications = 2 million new notifications per day.
* **Traffic Pattern (Reads):** 50:1 read-to-write ratio (read-heavy).
* **Growth:** assume 2x in 6 to 12 months.
* **Platforms:** web and mobile.
* **Availability:** Highly available. The badge may be slightly stale, but it must always load.
* **Consistency:** Eventual consistency is acceptable.
* **Latency:** badge loads in under 200 ms.

---

## Step 2a: High-Level Design & API

### Architecture Flows
* **Write Flow (New notification):** User → DNS → Load Balancer → Web Server → Database → Cache
    * *Database:* add a row to `notifications` and add 1 to `unread_count`.
    * *Cache:* update the cached count.
* **Read Flow (View badge):** User → DNS → Load Balancer → Web Server → Cache
    * *Cache Hit:* Return the count immediately.
    * *Cache Miss:* Fetch from Database → Save to Cache → Return to user.
* **Write Flow (Mark all as read):** User → DNS → Load Balancer → Web Server → Database → Cache
    * *Database:* set `is_read = true` on the user's rows and set `unread_count = 0`.
    * *Cache:* set the cached count to 0.

**Why this order?** Writes go database first, then cache, so the cache never holds something that isn't real. Reads go cache first, because it is much faster.

### API Design

**1. View unread count (read)**
```
GET /v1/notifications/unread-count
Headers: auth_token
Response:
{ "unread_count": 7 }
```

**2. Mark all as read (write)**  TODO (Day 2)

**3. Get notification list (read)**  TODO (Day 2)

**4. Create notification (internal, write)**  TODO (Day 2)

---

## Step 2b: Database Design

TODO (Day 2): a counter table (fast read) and a details table (history).

---

## Step 2c: Back-of-the-Envelope Math

**Assumptions:** 100,000 users/day, 20 notifications per user per day (the write), read:write = 50:1, 1 server = 100 QPS, retention = 30 days, 1 shard = 500 GB or 1,000 writes/sec, 5% of notifications trigger a 2 s push job, cache = 20% of the last 7 days.

**Bytes per notification (estimated):** notification_id 8 + user_id 8 + created_at 8 + is_read 1 + text about 100 = about 125 raw, x1.5 to 2 → **~200 bytes**.

| # | Calculation | Result |
|---|---|---|
| 1 | Writes/day = 100,000 x 20 | 2,000,000 |
| 2 | Avg write QPS = 2,000,000 / 86,400 | 23.15 |
| 2 | Peak write QPS = 2 x 23.15 | 46.3 |
| 3 | Avg read QPS = 23.15 x 50 | 1,157.5 |
| 3 | Peak read QPS = 2 x 1,157.5 | 2,315 |
| 4 | Web servers = ceil(2,315 / 100) | **24** |
| 5 | Storage/day = 2,000,000 x 200 B | 400 MB |
| 5 | Total (30 days) = 400 MB x 30 | **12 GB** |
| 6 | Shards = max(ceil(12 / 500), ceil(46.3 / 1,000)) | **1** |
| 7 | Push job rate = 5% x 46.3 | 2.3 jobs/sec |
| 7 | Workers = ceil(2.3 x 2 s) | **5** |
| 8 | Hot data = 400 MB x 7 | 2.8 GB |
| 8 | Cache = 20% x 2.8 GB | **~560 MB** |

**Takeaway:** reads drive the server count (24). Storage, shards and cache are all small.

---

## Step 3: Deep Dive

TODO (Day 2): what goes wrong if "mark all as read" and a new notification arrive at the same moment?

---

## Step 4: Wrap-Up

TODO (Day 2): bottlenecks, failure handling, monitoring, 10x scale.

---

## What I Got Wrong and Learned (Day 1)
* **Average vs peak:** average write x ratio = average read, then x 2 once for peak. I had doubled twice.
* **Round up:** 46.3 rounds up to 47. Servers, shards and workers are always rounded up.
* **Writes vs reads:** a write goes straight to the database, then the cache. Cache hit/miss belongs to reads.
* **Two saves per write:** the details (a row) and the number (the counter).