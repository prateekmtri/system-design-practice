# 🚀 The System Design Playbook

Welcome to my daily High-Level Design (HLD) practice repository. This repo documents my journey from writing feature-level code to designing scalable, fault-tolerant, and globally distributed systems. 

**Goal:** Solve and document 2 comprehensive system design problems per week, heavily focusing on trade-offs, bottlenecks, and capacity planning.

---

## 🏗️ 1. The Global Architecture Blueprint

This is the standard, battle-tested microservices architecture pattern I use as a baseline for read-heavy and write-heavy applications.

```mermaid
flowchart TB
    subgraph Client Tier
        Mobile[📱 Mobile App React Native]
        Web[💻 Web App React.js]
    end

    subgraph Edge / Network Tier
        CDN[🌐 CDN / Cloudflare]
        Route53[🗺️ DNS]
        CDN <--> Route53
    end

    subgraph API / Load Balancing Tier
        Nginx[🚦 Nginx Reverse Proxy / Load Balancer]
        Gateway[🚪 API Gateway]
    end

    subgraph Compute / Service Tier
        Auth[🔐 Auth Service Node.js / Express]
        CoreAPI[⚙️ Core Service FastAPI]
        Payment[💳 Payment Gateway Razorpay]
    end

    subgraph Data / Caching Tier
        Redis[(⚡ Redis Cache)]
        MongoPrimary[(🗄️ MongoDB Primary)]
        MongoReplica[(🗂️ MongoDB Read Replica)]
        S3[📦 AWS S3 / Blob Storage]
    end

    %% Connections
    Client Tier -->|HTTPS| Edge / Network Tier
    Edge / Network Tier -->|Traffic Routing| Nginx
    Nginx --> Gateway
    
    Gateway --> Auth
    Gateway --> CoreAPI
    Gateway --> Payment
    
    CoreAPI --> Redis
    Auth --> Redis
    
    CoreAPI -->|Writes| MongoPrimary
    CoreAPI -->|Reads| MongoReplica
    MongoPrimary -.->|Async Sync| MongoReplica
    
    CoreAPI -->|Media Uploads| S3