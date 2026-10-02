# System Design: Like Counter

## Step 1: Understand the Problem & Establish Scope
**Functional Requirements:**
* Users can like a piece of content (post/video).
* Users can unlike content.
* Users can view the total number of likes on a piece of content.

**Non-Functional Requirements:**
* **Scale (Writes):** 10 million new likes per day.
* **Traffic Pattern (Reads):** 1,000:1 read-to-write ratio (extremely read-heavy).
* **Availability:** Highly available.
* **Consistency:** Eventual consistency is acceptable.

## Step 2a: High-Level Design & API
### Architecture Flows
* **Write Flow (Liking a post):** User → DNS → Load Balancer → Web Server → Database
* **Read Flow (Viewing likes):** User → DNS → Load Balancer → Web Server → Cache
    * *Cache Hit:* Return data immediately.
    * *Cache Miss:* Fetch from Database → Save to Cache → Return to user.

### API Design
**1. Like a post (write)**
POST /v1/posts/{post_id}/like
Headers: auth_token
Response:
{ "success": true, "post_id": "12345", "total_likes": 4821 }

**2. View a post's like count (read)**
GET /v1/posts/{post_id}/likes
Response:
{ "post_id": "12345", "total_likes": 4821 }