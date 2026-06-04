# Problem Brief: Multi-Promotion Discount Engine

## Overview

Design and implement a backend system that applies multiple **Promotion Sets** to a list of products, computing the best applicable discount for each promotion set and returning enriched product objects with discount information.

---

## Domain Model

### Product (Input)

| Field      | Type   | Example        | Description                              |
|------------|--------|----------------|------------------------------------------|
| `product`  | String | `"A123"`       | Unique product identifier                |
| `category` | String | `"electronics"`| Product category                         |
| `inventory`| Number | `30`           | Units available in stock                 |
| `arrival`  | String | `"NEW"`        | Arrival status (`NEW` / other)           |
| `rating`   | Number | `1.1`          | Product rating (float)                   |
| `price`    | Number | `2300`         | Listed price                             |
| `origin`   | String | `"Africa"`     | Country/region of origin                 |

### Product (Output)

Same as input, with one additional field:

| Field      | Type   | Description                                                   |
|------------|--------|---------------------------------------------------------------|
| `discount` | Object | Final discount applied with `price` (number) and `message` (string) |

---

## Sample Input

```json
[
  {
    "product": "A123",
    "category": "electronics",
    "inventory": 30,
    "arrival": "NEW",
    "rating": 1.1,
    "price": 2300,
    "origin": "Africa"
  }
]
```

## Sample Output

```json
[
  {
    "product": "A123",
    "category": "electronics",
    "inventory": 30,
    "arrival": "NEW",
    "rating": 1.1,
    "price": 2300,
    "origin": "Africa",
    "discount": {
      "price": 161,
      "message": "7% off from Promotion Set A (Africa origin)"
    }
  }
]
```

---

## Business Rules

### Promotion Set A

Each rule produces a candidate discount. **Only the highest discount among matching rules is applied.**

| Rule | Condition                                                                 | Discount                          |
|------|---------------------------------------------------------------------------|-----------------------------------|
| A1   | `origin == "Africa"`                                                      | 7% off price                      |
| A2a  | `rating == 2`                                                             | 4% off price                      |
| A2b  | `rating < 2`                                                              | 8% off price                      |
| A3   | `category in ["electronics", "furnishing"]` AND `price >= 500`            | Flat 100 off                      |

> **Note:** Rules A2a and A2b are mutually exclusive. Evaluate all applicable rules and apply the one with the **highest discount value**.

---

### Promotion Set B

Each rule produces a candidate discount. **Only the highest discount among matching rules is applied.**

| Rule | Condition              | Discount        |
|------|------------------------|-----------------|
| B1   | `inventory > 20`       | 12% off price   |
| B2   | `arrival == "NEW"`     | 7% off price    |

> **Note:** Evaluate all applicable rules and apply the one with the **highest discount value**.

---

### Default Discount

| Condition                                                                      | Discount     |
|--------------------------------------------------------------------------------|--------------|
| `price > 1000` AND **no discount was applied** from any Promotion Set         | 2% off price |

---

## Core Constraints

1. **One discount per Promotion Set** — within a given Promotion Set, only one discount rule may be applied (the one yielding the highest value to the customer).
2. **Best-deal selection** — when multiple rules in a set are satisfied, always pick the rule that gives the customer the **maximum discount**.
3. **Default discount fallback** — the 2% default discount applies only when `price > 1000` and **neither** Promotion Set A nor Promotion Set B applied any discount.
4. **Discount stacking** — discounts from Set A and Set B are applied independently and their absolute values are summed for the final discount.
5. **Integer prices only** — all prices and discounts must be whole numbers (no decimals).
6. **Maximum discount cap** — the final discount cannot exceed 50% of the product price.

---

## Expected System Behaviour

```
For each product:
  1. Evaluate all rules in Promotion Set A → pick highest applicable discount (Set A Discount)
  2. Evaluate all rules in Promotion Set B → pick highest applicable discount (Set B Discount)
  3. Sum the absolute discount amounts from Set A and Set B
  4. If total discount is 0 AND price > 1000:
       Apply default 2% discount
  5. Cap the final discount at 50% of product price
  6. Attach the computed discount as { "price": final_discount_amount, "message": rule_summary } to the product output object
```

---

## Clarifications

- **Discount output**: The `discount` field is a JSON object with:
  - `price`: the absolute discount amount (integer)
  - `message`: human-readable description of applied rules
- **Discount stacking**: Set A and Set B discounts are summed, then capped at 50% of product price.
- **Rating equality**: `rating == 2` is strict equality (exactly 2.0).
- **Case sensitivity**: `"NEW"` and category values are case-sensitive.

---

## Scope

- **In scope**: Discount computation logic, promotion rule engine, input/output contract.
- **Out of scope**: Payment processing, user authentication, product catalog management.
