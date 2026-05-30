# System Archetype: [e.g., Distributed Notification Engine]

## 1. Core Requirements & Constraints
* **Functional Requirements:** [What must the system do?]
* **Scale & Volumetric Estimates:** [e.g., 100M DAU, 10k read QPS, 100k write QPS, 500ms p99 latency target.]

## 2. Component Blueprint
[Insert End-to-End Architectural Block Diagram]

## 3. Primitives Applied
*This maps back to your individual concept notes.*
* **Ingestion Layer:** Leverages **[Rate Limiting]** and **[Message Queues]** to prevent upstream flooding.
* **Storage Layer:** Uses **[Database Sharding]** keyed by User ID to handle high write-throughput.
* **Delivery Layer:** Employs the **[Bulkhead Pattern]** to isolate slow third-party SMS gateways.

## 4. Deep-Dive Failure Scenarios
* **Scenario A (Data Center Outage):** How the system behaves during a network partition.
* **Scenario B (Hot Key / Celebrity Problem):** What happens when a single entity dominates traffic.