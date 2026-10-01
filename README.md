# 🏗️ System Design Architecture Portfolio

A structured collection of High-Level Design (HLD) architectures, exploring scalability, fault tolerance, and performance optimization for various distributed systems.

## 🌐 Typical High-Level Architecture (Reference)

```mermaid
graph TD
    Client[Client / Mobile App] -->|HTTPS| Route53[DNS / Cloudflare]
    Route53 --> LB[Load Balancer / Nginx]
    LB --> API[API Gateway]
    
    API --> App1[App Server 1 / Node.js]
    API --> App2[App Server 2 / FastAPI]
    
    App1 --> Cache[(Redis Cache)]
    App2 --> Cache
    
    App1 --> DB[(Primary DB / MongoDB)]
    App2 --> DB
    
    DB -.->|Asynchronous Replication| ReadDB[(Read Replica)]
    
    App1 --> MQ[Message Queue / Kafka]
    MQ --> Worker[Background Workers]