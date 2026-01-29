## Trade-offs in Revenue Modeling

This document captures the **deliberate trade-offs** made while designing the revenue model in this folder.

There is no universally “correct” revenue model. Every design reflects priorities — clarity vs simplicity, flexibility vs performance, correctness vs speed. The goal here is to make those choices explicit.

---

## 1️⃣ Separate facts vs single revenue fact

### Choice made

Revenue-related activity is split across multiple fact tables:

* `fact_orders`
* `fact_order_items`
* `fact_return_items`

### Why

* Preserves **event intent** (order placed vs item sold vs item returned)
* Makes grains explicit and enforceable
* Simplifies reasoning about gross vs net revenue

### Cost

* More tables to maintain
* Queries require joins or unions across facts

### Alternative

A single fact table with positive and negative revenue rows.

### Why not

* Blurs meaning of rows
* Makes debugging and reconciliation harder
* Encourages hidden business logic in queries

---

## 2️⃣ Explicit returns vs negative quantities

### Choice made

Returns are modeled as **first-class events** in `fact_return_items`.

### Why

* Supports independent analysis of return behavior
* Avoids mixing corrections with original sales
* Aligns better with operational systems

### Cost

* Net revenue is not directly stored
* Analysts must understand how to combine facts

### Alternative

Storing returned quantities as negative values in sales facts.

### Why not

* Makes event timelines unclear
* Breaks assumptions around monotonic metrics
* Can cause subtle aggregation errors

---

## 3️⃣ Analytics-facing model vs transactional mirror

### Choice made

This model is **analytics-facing**, not a 1:1 copy of source systems.

### Why

* Optimized for querying and interpretation
* Decouples analytics from upstream schema churn
* Allows clearer metric ownership

### Cost

* Requires transformation layer
* Adds latency between source and reporting

### Alternative

Expose transactional tables directly to analysts.

### Why not

* Schema complexity leaks into analytics
* High cognitive load for consumers
* Fragile metrics when sources change

---

## 4️⃣ Grain-first design vs metric-first design

### Choice made

Grain is defined **before** metrics are discussed.

### Why

* Prevents accidental double counting
* Makes joins predictable
* Keeps metrics portable across tools

### Cost

* Slower initial design process
* Requires discipline and documentation

### Alternative

Define metrics first, then back into tables.

### Why not

* Metrics become tightly coupled to specific queries
* Harder to evolve as questions change

---

## 5️⃣ Normalization vs query simplicity

### Choice made

Facts are normalized by event type rather than collapsed for convenience.

### Why

* Keeps models flexible
* Supports new use cases without rewrites
* Makes logic explicit

### Cost

* Slightly more complex queries
* Requires good documentation

### Alternative

Denormalize heavily for one-click queries.

### Why not

* Optimizes for current questions only
* Leads to duplication and drift over time

---

## 6️⃣ Performance vs correctness

### Choice made

Correctness and explainability are prioritized over minimal query cost.

### Why

* Revenue is a high-stakes metric
* Small inaccuracies erode trust quickly

### Cost

* Additional joins
* Higher compute usage at scale

### Mitigation

* Incremental models
* Materialized aggregates
* Warehouse-specific optimizations

---

## 7️⃣ Cost awareness (explicitly acknowledged)

While cost optimization is not the primary focus of this example, it is **considered a modeling concern**, not a post-hoc fix.

Examples of future cost controls:

* partitioning or clustering on dates
* incremental fact builds
* selective materialization of net metrics

---

## 🧠 Final note

Trade-offs are not flaws — they are signals of intent.

A good revenue model makes it easy to answer *why* a number looks the way it does. When that happens, stakeholders stop arguing about data and start discussing decisions.
