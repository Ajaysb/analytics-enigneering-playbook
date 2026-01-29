## Revenue Modeling

This section explores **how to design revenue models that are trustworthy, explainable, and resilient to real-world complexity**.

Revenue is one of the most commonly used metrics — and also one of the easiest to get subtly wrong. Returns, partial refunds, late-arriving data, and changing business definitions can all distort numbers if the underlying model is not designed carefully.

Rather than jumping straight into SQL, this section focuses first on **modeling decisions**.

---

## 🎯 Business goal

Enable consistent, analytics-ready revenue reporting that supports:

* daily and monthly revenue tracking
* item-level analysis
* returns and refunds analysis
* reconciliation with finance

The goal is not just to compute a number, but to ensure the number:

* can be explained
* can be reconciled
* can evolve as questions change

---

## 🧱 Core modeling idea

Revenue is modeled using **event-based facts** at well-defined grains.

Instead of a single overloaded fact table, revenue-related activity is split into multiple facts based on *what actually happened*:

### Fact tables

#### `fact_orders`

* Grain: **1 row = 1 order**
* Represents the creation of an order
* Used for order-level metrics (order count, order value at creation)

#### `fact_order_items`

* Grain: **1 row = 1 order line item**
* Represents items sold as part of an order
* Primary driver for gross revenue calculations
* Allows item-level analysis and partial returns

#### `fact_return_items`

* Grain: **1 row = 1 returned item**
* Represents return events
* Enables return-specific metrics without overloading sales facts

Returns are modeled as **separate events**, not as negative sales rows. This keeps intent clear and avoids hidden assumptions.

---

## 🧠 Why separate return facts?

Common alternatives include:

* subtracting returned quantities from sales
* storing negative revenue rows in sales tables

These approaches may look simpler, but they blur event meaning and make downstream logic harder to reason about.

A separate returns fact allows:

* clean gross vs net revenue logic
* independent analysis of return behavior
* simpler reconciliation with operational systems

Net revenue becomes a **derived metric**, not a stored one.

---

## 📐 Grain clarity (non-negotiable)

Every fact table has a single, explicit grain:

| Table             | Grain         |
| ----------------- | ------------- |
| fact_orders       | Order         |
| fact_order_items  | Order item    |
| fact_return_items | Returned item |

All metrics are defined relative to these grains. If a question cannot be answered cleanly at a grain, it is a modeling smell.

---

## 🔗 Dimensions

At a minimum, revenue facts join to:

* `dim_customer`
* `dim_product`

Dimensions are shared across facts to ensure:

* consistent slicing
* simpler joins
* predictable performance

Additional dimensions (date, channel, geography) can be added as needed, but are intentionally omitted here to keep the example focused.

---

## ⚖️ Trade-offs considered

This model prioritizes:

* clarity over compactness
* explicit events over implicit math
* flexibility over minimal table count

Costs include:

* more tables to manage
* additional joins in queries

These are acceptable trade-offs when correctness and explainability matter.

Detailed trade-offs are documented separately in `tradeoffs.md`.

---

## 🚀 How this model can evolve

Possible future enhancements include:

* incremental loading strategies
* late-arriving return handling
* revenue recognition logic
* data quality checks and reconciliation queries
* performance optimizations for large-scale warehouses

The model is intentionally designed to support these without major rewrites.

---

## 📌 Scope note

This is an **analytics-facing model**, not a transactional schema.

It assumes upstream systems already capture raw events. The focus here is on shaping that data into forms that analysts and decision-makers can trust.

---

## 🧠 Key takeaway

Revenue modeling is less about arithmetic and more about **respecting events, grain, and intent**.

When those are clear, the SQL becomes straightforward — and the numbers stop surprising people.
