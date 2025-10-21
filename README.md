# Google-Cloud-Security-Environment

```
flowchart LR
  A[Cloud Audit Logs / Cloud Logging] --> B[Log Collector]
  subgraph GCP
    B --> C[Telemetry Storage (Log Analytics / BigQuery)]
    C --> D[SIEM / Detection Rules]
    subgraph Network
      N1[VPC Flow Logs] --> C
      N2[Cloud IDS] --> D
      N3[Cloud Armor / Load Balancer] --> NetworkBlock
    end
    subgraph Storage
      S1[Cloud Storage (Buckets)] --> S2[DLP Scanner]
      S2 --> C
      S3[KMS / CMEK / CSEK] --> S1
    end
    subgraph Identity
      I1[IAM / Service Accounts] --> D
      I2[Secret Manager] --> IdentityBlock
    end
    D --> E[Enrichment Functions (GeoIP, IOC)]
    E --> F[Containment Playbooks (Cloud Functions/Workflows)]
    F --> S1
    F --> I1
    F --> N3
  end
  style GCP fill:#f8f9fb,stroke:#333,stroke-width:1px


```
