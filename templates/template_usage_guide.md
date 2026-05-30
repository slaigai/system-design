# Guide: How to Use the System Design Knowledge Base Templates

This guide explains how to effectively use both the **Concept Template** and the **Archetype Template** to build out your personalized engineering curriculum. By standardizing your study notes, you will train yourself to think like a Principal Engineer—focusing on trade-offs, hardware boundaries, systemic blast radiuses, and end-to-end component interactions.

---

## Part 1: The Concept Template (Building Blocks)

Use this template to profile specific architectural patterns, algorithms, or primitives (e.g., *Consistent Hashing*, *Circuit Breakers*, *Token Bucket Rate Limiting*).

### 1. Title
* **Goal:** Explicitly name the core architectural pattern or primitive.
* **Action:** Replace the bracketed placeholder with the exact engineering concept.

### 2. Summary
* **Goal:** Provide an elevator pitch of the pattern for high-pressure interviews or architectural review boards.
* **Action:** Keep this section brief. Use bullet points to capture the essence of the concept as a quick-reference flashcard.

### 3. The Problem It Solves
* **Goal:** Establish deep architectural empathy by understanding the pain points of scaling without this pattern.
* **Action:** * **Motivating Factors:** Detail the exact wall you hit at scale (e.g., For *Rate Limiting*: "Runaway scripts scraping endpoints and starving legitimate users of server threads").
    * **The Solution Mechanism:** Explain *why* it works mechanically under the hood.
    * **Resulting System Properties:** Explicitly map what you win versus what you lose. Remember: *There are no solutions in system design, only trade-offs.*

### 4. Implementation & Architecture Design
* **Goal:** Move from abstract theory to practical engineering implementation.
* **Action:** Drill down into specific strategies, data layouts, and wire protocols.
    * **Data Layout & Wire Protocol:** Document exact implementation details (e.g., choosing Protobuf vs. JSON for serialization overhead, or LSM-Trees vs. B-Trees for database write optimization).
    * **Visual Model:** Draw or type a basic ASCII architecture block diagram to visualize request paths or data ownership boundaries.

### 5. Trade-offs, Operational Risks & Blast Radius
* **Goal:** Uncover the hidden operational and physical costs of your design decisions.
* **Action:** Think like a Site Reliability Engineer (SRE). 
    * **Blast Radius:** If this component drops packets, experiences a memory leak, or goes completely offline, what adjacent systems fail? Does it cause a cascading outage or degrade gracefully?
    * **The 5-Year Rule:** Ask yourself: *"If our data volume or traffic grows by 50x over 5 years, where will this concept break first? RAM, CPU, Network I/O, or Disk IOPS?"* Document that exact physical bottleneck.

### 6. Real-World Examples & Case Studies
* **Goal:** Ground your knowledge in real industry execution and failures.
* **Action:** Research engineering blogs (Netflix, Uber, Meta, Stripe) for production case studies.
    * *Tip for Failure Cases:* Search for public Post-Mortems or Root Cause Analyses (RCAs) showing how a misconfiguration of this specific primitive caused a massive incident.

### 7. Technology Landscape
* **Goal:** Map abstract patterns to concrete production tooling.
* **Action:** Build a matrix of industry-standard tools that implement this concept natively out of the box (e.g., For *Circuit Breakers*: Envoy, Istio, or Resilience4j).

---

## Part 2: The Archetype Template (System Blueprints)

Use this template to design full, end-to-end applications or infrastructure ecosystems (e.g., *Designing a Geospatial Ride-Hailing App*, *Distributed Financial Ledger*, or *Video Streaming Platform*).

### 1. Core Requirements & Constraints
* **Goal:** Establish clear, unyielding design boundaries before drawing a single box.
* **Action:** * **Functional Requirements:** Define the 3-4 core actions the system *must* perform perfectly.
    * **Scale & Volumetric Estimates:** Calculate explicit targets for DAU (Daily Active Users), Write QPS, Read QPS, storage growth per year, and network bandwidth. Target a concrete performance metric (e.g., "p99 latency < 200ms").

### 2. Component Blueprint
* **Goal:** Map the macro request lifecycle and data flows.
* **Action:** Insert a comprehensive end-to-end block diagram spanning from the client/CDN, through edge gateways and microservices, down to data tiers and async workers.

### 3. Primitives Applied
* **Goal:** Interconnect your system blueprints back to your individual building blocks.
* **Action:** Explicitly list and cross-link the completed files from your `primitives/` directory to show *how* they are used here.
    * *Example:* "Ingestion Layer: Leverages `primitives/rate_limiting.md` via a Redis cluster to prevent upstream web-tier flooding during traffic spikes."

### 4. Deep-Dive Failure Scenarios & Edge Cases
* **Goal:** Prove the resilience of your macro design against inevitable real-world chaos.
* **Action:** Break your own system. Write detailed narratives on how the architecture survives:
    * **The Hot Key / Celebrity Problem:** A single entity dominates traffic.
    * **Network Partitions (Split-Brain):** Data centers lose connection to each other.
    * **Thundering Herds:** A cache invalidation triggers millions of simultaneous database reads.

---

## Recommended Study & Maintenance Workflow

To turn your curriculum into an evolving, highly organized knowledge engine over your career, adhere to this repository workflow:

1. **The Core/Primitive Phase:** Pick a concept marked with a `🔥` in your root `README.md`. Read documentation and engineering blogs, and actively synthesize it using the **Concept Template** inside `primitives/`.
2. **The Assembler/Archetype Phase:** Once you have completed the prerequisite primitives, create a full system design in `archetypes/` using the **Archetype Template**. Explicitly link back to your primitive files.
3. **The 50x Scale Test:** For every archetype and primitive profile you fill out, always explicitly note its scaling ceiling. Anchor your conclusions to physical hardware limits (RAM speed, network packet delivery, disk IOPS constraints), not magical cloud abstractions.
4. **Iterative Updates:** When you encounter an engineering outage at work, a new whitepaper, or a fascinating post-mortem blog post, go back and update the corresponding primitive or archetype file to capture that newly discovered edge case.