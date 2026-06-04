# Multi-Promotion Discount Engine - Architecture Plan

## Executive Summary

A REST API that evaluates two independent promotion sets against product attributes and computes the maximum applicable discount. Discounts from both sets are summed (with a 50% cap) and returned as a structured JSON object with price and message fields.

---

## 1. High-Level Architecture

```
HTTP POST /api/discounts/compute
    ↓
[Products Array]
    ↓
Process Each Product:
  1. Evaluate Promotion Set A (pick highest matching rule)
  2. Evaluate Promotion Set B (pick highest matching rule)
  3. Sum discounts from both sets
  4. Apply 50% cap
  5. Apply default 2% if price > 1000 and no discount
    ↓
[DiscountResults with price + message]
```

---

## 2. API Design

### 2.1 Endpoint

```
POST /api/discounts/compute
Content-Type: application/json
Accept: application/json

Request: Array of ProductRequest objects
Response: Array of DiscountResult objects
```

### 2.2 HTTP Status Codes

- **200 OK**: Successful computation
- **400 Bad Request**: Invalid input (missing fields, wrong types, price ≤ 0)
- **422 Unprocessable Entity**: Validation failure (non-integer price, invalid category, etc.)
- **500 Internal Server Error**: Unexpected server error

---

## 3. Extensible Data Models

### 3.0 Extensibility Strategy

All data models are designed to be extensible:

**Products**: Add new fields without breaking existing rules
- Example: `supplier`, `color`, `size`, `region`, `brand`
- Existing rules continue to work
- New rules can use new fields

**Rules**: Add new conditions that reference new product fields
- No API contract changes
- Rules engine evaluates any product field

**Discounts**: Additional metadata can be added
- Example: `promoCode`, `expiryDate`, `applicableChannels`
- Output structure remains backward compatible

---

### 3.1 Request DTO: ProductRequest (Extensible)

**Current Fields**:
```json
{
  "product": "A123",           // string, required - product ID
  "category": "electronics",   // string, required - case-sensitive
  "inventory": 30,             // integer, required - ≥ 0
  "arrival": "NEW",            // string, required - case-sensitive
  "rating": 1.1,               // number (float), required
  "price": 2300,               // integer, required - > 0
  "origin": "Africa"           // string, required
}
```

**Future-Ready Design**:
- Can add new fields (e.g., `supplier`, `brand`, `region`)
- Existing rules unaffected
- New rules can reference new fields
- Use optional fields with null handling

**Example with Extended Fields**:
```json
{
  "product": "A123",
  "category": "electronics",
  "inventory": 30,
  "arrival": "NEW",
  "rating": 1.1,
  "price": 2300,
  "origin": "Africa",
  "supplier": "SupplierX",      // NEW - future field
  "brand": "BrandY",             // NEW - future field
  "region": "EMEA"               // NEW - future field
}
```

**Validation Rules**:
- All fields are required
- `price` must be integer and > 0
- `category` and `arrival` are case-sensitive
- `rating` is float (can be any decimal value)

### 3.2 Response DTO: DiscountResult

```json
{
  "product": "A123",
  "category": "electronics",
  "inventory": 30,
  "arrival": "NEW",
  "rating": 1.1,
  "price": 2300,
  "origin": "Africa",
  "discount": {
    "price": 161,              // integer - absolute discount amount
    "message": "7% from Set A (Africa origin)"
  }
}
```

### 3.3 Discount Object (Extensible)

**Current Structure**:
```json
{
  "price": 161,     // integer - absolute discount amount (never null)
  "message": "..."  // string - human-readable rule summary
}
```

**Message Examples**:
- `"7% from Set A (rule A1) + 12% from Set B (rule B1) = 190 (capped at 50%)"` 
- `"Default 2% discount applied (price > 1000)"`
- `"No discount applied"`

**Future Extensions** (backward compatible):
```json
{
  "price": 161,
  "message": "7% from Set A (rule A1) + 12% from Set B (rule B1)",
  "breakdown": {                // NEW - detailed breakdown
    "setA": { "ruleId": "A1", "amount": 161 },
    "setB": { "ruleId": "B1", "amount": 0 },
    "default": { "applied": false, "amount": 0 }
  },
  "promoCode": null,            // NEW - promo code used
  "expiryDate": "2026-12-31",   // NEW - discount valid until
  "channels": ["WEB", "MOBILE"] // NEW - applicable channels
}
```

**Backward Compatibility**: 
- New fields are optional and can be ignored by older clients
- Core fields (`price`, `message`) remain unchanged

---

## 4. Extensible Rule Design

### 4.0 Rule Definition Model (Extensible)

Rules are defined as **configuration** (not hardcoded), allowing new rules to be added without code changes.

**Rule Definition DTO**:

```json
{
  "ruleId": "A1",
  "setId": "A",
  "description": "7% off (Africa origin)",
  "discountType": "percentage",         // "percentage" or "flat"
  "discountValue": 7,                   // 7 for 7%, 100 for 100 flat
  "conditions": [
    {
      "field": "origin",
      "operator": "equals",
      "value": "Africa"
    }
  ],
  "priority": 1,                        // Higher = higher priority
  "mutualExclusionGroup": null,         // "A2" means mutually exclusive with A2a/A2b
  "active": true,
  "createdAt": "2026-01-01T00:00:00Z",
  "version": 1
}
```

### 4.0.1 Condition Model

```json
{
  "field": "origin",                    // Product field name
  "operator": "equals",                 // equals, notEquals, contains, greaterThan, lessThan, greaterThanOrEqual, in, regex, etc.
  "value": "Africa",                    // Single value or array depending on operator
  "caseSensitive": true                 // For string comparisons
}
```

**Supported Operators**:
- `equals` — exact match (case-sensitive for strings)
- `notEquals` — inverse match
- `greaterThan` — numeric >
- `lessThan` — numeric <
- `greaterThanOrEqual` — numeric >=
- `lessThanOrEqual` — numeric <=
- `in` — value in array [val1, val2, ...]
- `contains` — string contains substring
- `regex` — regex pattern match

### 4.0.2 Discount Type Model

```json
{
  "discountType": "percentage",   // "percentage" or "flat"
  "discountValue": 7,             // 7 means 7%, or 100 for 100 flat units
  "basedOnPrice": true            // For percentage: true = of price, for flat: always true
}
```

### 4.0.3 How Extensibility Works

**Adding a New Rule**:
- Create rule definition in config (YAML, JSON, or database)
- Specify field, operator, discount type, value
- No code changes needed

**Example: New Rule A4**
```json
{
  "ruleId": "A4",
  "setId": "A",
  "description": "5% off for high-rated products",
  "discountType": "percentage",
  "discountValue": 5,
  "conditions": [
    {
      "field": "rating",
      "operator": "greaterThanOrEqual",
      "value": 4.0
    }
  ],
  "priority": 2,
  "mutualExclusionGroup": null,
  "active": true
}
```

**Adding a New Promotion Set**:
- Create new set (C, D, etc.)
- Define rules under that set
- Modify discount calculation to include new set (parallel to Set A, Set B)
- Update stacking logic if needed

---

## 4. Business Logic

### 4.1 Promotion Set A Rules (Current)

| Rule ID | Condition | Discount | Notes |
|---------|-----------|----------|-------|
| A1 | origin == "Africa" | 7% of price | Straightforward |
| A2a | rating == 2.0 (exact) | 4% of price | Mutually exclusive with A2b |
| A2b | rating < 2.0 | 8% of price | Mutually exclusive with A2a |
| A3 | category in ["electronics", "furnishing"] AND price ≥ 500 | 100 flat | Case-sensitive category |

**Evaluation**:
- Check all rules
- If multiple match, apply the one with **highest discount value**
- Return rule ID and discount amount

### 4.2 Promotion Set B Rules

| Rule ID | Condition | Discount | Notes |
|---------|-----------|----------|-------|
| B1 | inventory > 20 | 12% of price | Strictly > 20, not ≥ |
| B2 | arrival == "NEW" | 7% of price | Case-sensitive |

**Evaluation**:
- Check all rules
- If multiple match, apply the one with **highest discount value**
- Return rule ID and discount amount

### 4.3 Default Discount

- **Condition**: No discount from Set A OR Set B AND price > 1000
- **Discount**: 2% of price

### 4.4 Discount Calculation Flow

For each product:

1. **Evaluate Set A**
   - Check rules A1, A2b, A2a (mutually exclusive), A3
   - Pick rule with highest discount
   - Return: `{ ruleId, amount }` or `null`

2. **Evaluate Set B**
   - Check rules B1, B2
   - Pick rule with highest discount
   - Return: `{ ruleId, amount }` or `null`

3. **Stack Discounts**
   - Sum absolute amounts from Set A and Set B
   - `totalDiscount = amountA + amountB`

4. **Apply Default**
   - If `totalDiscount == 0` AND `price > 1000`
   - `totalDiscount = price * 2 / 100`

5. **Apply Cap**
   - Max discount = `price * 50 / 100`
   - `finalDiscount = min(totalDiscount, maxDiscount)`

6. **Build Message**
   - Summarize which rules applied
   - Note if cap was applied
   - Format: `"A1 (7%) + B2 (7%) = 140, capped at 50% = 1000"`

---

## 4.5 Rule Storage & Loading

**Storage Options** (choose one):

1. **YAML/JSON Configuration** (simplest for small rule sets)
   ```
   rules/
   ├── set-a.yaml
   └── set-b.yaml
   ```

2. **Database** (most extensible for dynamic rules)
   ```
   Table: promotion_rules
   - rule_id (PK)
   - set_id
   - description
   - discount_type
   - discount_value
   - conditions (JSON)
   - priority
   - mutual_exclusion_group
   - active
   - version
   - created_at
   - updated_at
   ```

3. **Feature Flag Service** (for A/B testing rules)
   - Load via Unleash, LaunchDarkly, or similar

**Loading Mechanism**:
- Load rules at application startup (cache)
- Optionally: Reload on-demand via API endpoint (PUT /api/rules/reload)
- Version rules to support backward compatibility

**API for Rule Management** (optional, for future editions):
```
GET  /api/rules                    → List all rules
GET  /api/rules/{ruleId}           → Get specific rule
POST /api/rules                    → Create new rule
PUT  /api/rules/{ruleId}           → Update rule
DELETE /api/rules/{ruleId}         → Deactivate rule
POST /api/rules/reload             → Force reload from storage
```

---

## 5. Key Constraints

1. **Integer Output Only** — All prices and discounts are integers (round down percentages)
2. **One Rule Per Set** — Only the highest-discount rule applies per promotion set
3. **Discount Stacking** — Set A and Set B discounts are summed
4. **50% Cap** — Final discount cannot exceed 50% of product price
5. **Case Sensitivity** — `category`, `arrival`, and `origin` are case-sensitive (configurable per rule)
6. **Exact Rating Match** — `rating == 2.0` is strict equality, not a range
7. **Rule Versioning** — Rules should be versioned for audit trail and backward compatibility

---

## 6. Use Cases & Expected Behavior

### UC-1: Single Rule Match
```
Input:  { product: "P1", category: "electronics", inventory: 30, arrival: "NEW", 
          rating: 1.1, price: 2300, origin: "Africa" }
Output: { discount: { price: 161, message: "7% from Set A (rule A1)" } }
```

### UC-2: Stacking
```
Input:  { product: "P2", category: "electronics", inventory: 25, arrival: "NEW", 
          rating: 1.1, price: 2000, origin: "Africa" }
Expected: A1 (7% = 140) + B2 (7% = 140) = 280
Output: { discount: { price: 280, message: "7% from Set A (A1) + 7% from Set B (B2)" } }
```

### UC-3: Discount Cap
```
Input:  { product: "P3", category: "electronics", inventory: 25, arrival: "NEW", 
          rating: 1.5, price: 1000, origin: "Africa" }
Expected: A1 (7% = 70) + B2 (7% = 70) = 140, but cap = 500 (50% of 1000)
Output: { discount: { price: 500, message: "...capped at 50% = 500" } }
```

### UC-4: Default Discount
```
Input:  { product: "P4", category: "furniture", inventory: 10, arrival: "OLD", 
          rating: 4.5, price: 1500, origin: "USA" }
Expected: No Set A/B match, price > 1000 → 2% = 30
Output: { discount: { price: 30, message: "Default 2% discount (price > 1000)" } }
```

### UC-5: No Discount
```
Input:  { product: "P5", category: "furniture", inventory: 10, arrival: "OLD", 
          rating: 4.5, price: 500, origin: "USA" }
Expected: No match, price < 1000 → 0
Output: { discount: { price: 0, message: "No discount applied" } }
```

### UC-6: A2a/A2b Mutual Exclusion
```
Product A: rating = 2.0   → A2a applies (4%), NOT A2b
Product B: rating = 1.9   → A2b applies (8%), NOT A2a
```

### UC-7: A3 Flat Discount
```
Input:  { product: "P6", category: "electronics", inventory: 10, arrival: "OLD", 
          rating: 3.0, price: 500, origin: "USA" }
Expected: A3 matches (electronics AND price >= 500) → 100 flat
Output: { discount: { price: 100, message: "100 flat off (rule A3)" } }
```

### UC-8: Case Sensitivity
```
Input with category: "ELECTRONICS" (uppercase)
Expected: A3 does NOT match (requires "electronics")
Result: No discount from Set A
```

### UC-9: Boundary - Inventory
```
inventory = 20    → B1 does NOT match (requires > 20)
inventory = 21    → B1 matches
```

### UC-10: Boundary - Price
```
price = 500       → A3 matches (>= 500)
price = 499       → A3 does NOT match
price = 1000      → Default NOT applied (not > 1000)
price = 1001      → Default applies if no other discount
```

### UC-11: Batch Processing
```
POST /api/discounts/compute with 100 products
Expected: Each product processed independently, all returned in order
```

---

## 7. Testing Strategy

### Unit Test Scenarios

**Set A Rules**:
- A1 matches when origin == "Africa"
- A2a matches when rating exactly equals 2.0
- A2b matches when rating < 2.0
- A2a and A2b never both match same product
- A3 matches when category is "electronics" or "furnishing" AND price >= 500
- When multiple rules match, highest discount wins

**Set B Rules**:
- B1 matches when inventory > 20
- B2 matches when arrival == "NEW"
- When both match, highest discount wins

**Discount Calculator**:
- Correctly sums Set A + Set B discounts
- Applies 50% cap when sum exceeds cap
- Applies 2% default when totalDiscount == 0 AND price > 1000
- Builds accurate message strings

**Validator**:
- Rejects missing fields
- Rejects non-integer prices
- Rejects prices ≤ 0
- Accepts all valid inputs

### Integration Test Scenarios

- Full flow: Product → Both Sets → Stacking → Cap → Message
- Batch processing: Multiple products in single request
- Error responses: 400 for invalid input, 422 for validation, 500 for errors
- Message accuracy: Verify correct rules are cited

### Edge Cases

- Price exactly at 500, 1000 boundaries
- Rating exactly at 2.0
- Inventory exactly at 20
- Discounts that sum to > 50%
- Empty product array
- Very large prices (e.g., 999999)

---

## 8. Questions for Discussion

1. **Batch size**: Should we limit request payload (e.g., max 1000 products per request)?
2. **Message format**: Is the proposed format clear and useful, or should it be different?
3. **Rounding**: Should percentage-to-integer conversion always round down, or use standard rounding?
4. **Logging**: Should we log all computations for audit trail?
5. **Rule Storage**: YAML/JSON files, database, or feature flag service?
6. **Rule Reload**: Should rules reload on startup only, or support hot-reload?
7. **Backward Compatibility**: Should we maintain API v1 while rolling out v2, or version within JSON?
8. **Database**: Should discount results be persisted for analytics?

---

## 9. Extensibility Framework for Future Editions

### 9.1 Adding New Rules (No Code Change)

**Example: New rule for supplier-based discount**

```yaml
# rules/set-c.yaml (new file)
rules:
  - ruleId: C1
    setId: C
    description: "10% off for preferred suppliers"
    discountType: percentage
    discountValue: 10
    conditions:
      - field: supplier
        operator: in
        value: ["SupplierX", "SupplierY"]
    priority: 1
    active: true
```

Modify discount calculation to include Set C (parallel to A/B).

### 9.2 Adding New Product Fields

**Example: Add region and brand fields**

1. Update ProductRequest DTO (add optional fields)
2. Create new rules referencing these fields:
   ```
   Region = "APAC" → 5% discount
   Brand = "Premium" → 3% additional discount
   ```
3. No changes to existing rules or calculation logic

### 9.3 Extending Discount Response

**Add optional fields** to discount response:
- `appliedRules` — array of which rules matched
- `breakdown` — per-set discount details
- `metadata` — timestamps, validity dates

**Backward Compatibility**:
- Old clients can ignore new fields
- New clients can use detailed breakdown
- API versioning (v1, v2) if major changes needed

### 9.4 Typical Extension Scenarios

| Scenario | Product | Rules | Discount |
|----------|---------|-------|----------|
| Add new promo set (C, D) | No change | New rules file | Include in stacking |
| New product attribute | Add field | Reference in conditions | No change needed |
| Regional discounts | Add `region` field | New rule with region condition | Display in message |
| Time-based offers | Add `validity_date` | Check date in condition | Add to breakdown |
| Multi-tier discounts | No change | New rules with threshold conditions | Stack as usual |
| Supplier programs | Add `supplier` field | New rules referencing supplier | Include in message |

---

## 10. Non-Functional Requirements

- **Performance**: Process batch of 100 products in < 100ms
- **Scalability**: Support 1000+ concurrent requests
- **Error Handling**: Clear, actionable error messages
- **Documentation**: API docs with examples and error cases
- **Monitoring**: Track success rate, latency, rule usage
- **Extensibility**: Add new rules without code changes; add fields without API break

---
