flowchart LR
    subgraph PublisherTier [Publisher]
        API[Core API]
    end

    subgraph BrokerTier [Message Broker]
        Kafka{Apache Kafka / RabbitMQ}
    end

    subgraph WorkerTier [Consumers & Workers]
        Worker1[👷 Email/SMS Worker]
        Worker2[👷 Analytics Aggregator]
        Worker3[👷 Image/Video Processor]
    end

    subgraph OutputTier [Output]
        SES[AWS SES]
        DataWarehouse[(Data Warehouse)]
        Bucket[S3 Bucket]
    end

    API -->|Publish Event| Kafka
    Kafka -->|Consume Topic A| Worker1
    Kafka -->|Consume Topic B| Worker2
    Kafka -->|Consume Topic C| Worker3

    Worker1 --> SES
    Worker2 --> DataWarehouse
    Worker3 --> Bucket