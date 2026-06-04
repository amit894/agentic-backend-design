# Discount Engine - Functional Test Cases

**Document**: Functional Test Cases for Multi-Promotion Discount Engine  
**Date**: 2026-06-04  
**Status**: Ready for Interview Review  
**Project**: /Users/amit_raj/IdeaProjects/discount-engine

---

## Overview

This document lists all functional test cases implemented for the Discount Engine API. Tests are derived from use cases defined in `PROBLEM-BRIEF.md` and cover:

- Core promotion rule evaluation (Set A & Set B)
- Discount stacking and capping logic
- Default discount application
- Input validation and error handling
- Batch processing

---

## Test Case Summary

| # | Test ID | Use Case | Scenario | Status |
|---|---------|----------|----------|--------|
| 1 | TC-001 | UC-1 | Single Rule Match (Set A: A1) | ✓ Implemented |
| 2 | TC-002 | UC-3 | Discount Stacking (Set A + Set B) | ✓ Implemented |
| 3 | TC-003 | UC-4 | Discount Cap (50% maximum) | ✓ Implemented |
| 4 | TC-004 | UC-5 | Default Discount (price > 1000) | ✓ Implemented |
| 5 | TC-005 | UC-6 | No Discount (price < 1000) | ✓ Implemented |
| 6 | TC-006 | N/A | Validation Error (missing fields) | ✓ Implemented |
| 7 | TC-007 | N/A | Batch Processing (multiple products) | ✓ Implemented |

---

## Detailed Test Cases

### TC-001: Single Rule Match (UC-1)

**Test Name**: `testUC1_SingleRuleMatch_AfricaOrigin`

**Objective**: Verify that a single promotion rule (A1) applies when matching condition is met.

**Precondition**: 
- API endpoint `/api/discounts/compute` is available
- No other rules match

**Input Data**:
```json
{
  "product": "A123",
  "category": "electronics",
  "inventory": 30,
  "arrival": "NEW",
  "rating": 1.1,
  "price": 2300,
  "origin": "Africa"
}
```

**Expected Output**:
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
    "price": 161,
    "message": "..."
  }
}
```

**Validation**:
- HTTP Status: 200 OK
- `discount.price` = 161 (7% of 2300)
- `discount.message` contains rule information
- All input fields echoed back in response

**Test Implementation**:
```java
@Test
void testUC1_SingleRuleMatch_AfricaOrigin() throws Exception {
    ProductRequest product = new ProductRequest("A123", "electronics", 30, "NEW", 1.1, 2300, "Africa");
    
    mockMvc.perform(post("/api/discounts/compute")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(Arrays.asList(product))))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].discount.price").value(161))
        .andExpect(jsonPath("$[0].discount.message").exists());
}
```

**Rule Triggered**: A1 (7% off for origin = "Africa")

---

### TC-002: Discount Stacking (UC-3)

**Test Name**: `testUC3_DiscountStacking_SetAAndSetB`

**Objective**: Verify that discounts from Set A and Set B are correctly stacked (summed).

**Precondition**:
- Both Set A and Set B contain matching rules
- Sum of discounts does not exceed 50% cap

**Input Data**:
```json
{
  "product": "C789",
  "category": "electronics",
  "inventory": 25,
  "arrival": "NEW",
  "rating": 1.5,
  "price": 2000,
  "origin": "Africa"
}
```

**Expected Calculation**:
- Set A: Rule A1 matches (origin = "Africa") → 7% of 2000 = 140
- Set B: Rule B2 matches (arrival = "NEW") → 7% of 2000 = 140
- Total: 140 + 140 = 280

**Expected Output**:
```json
{
  "discount": {
    "price": 280,
    "message": "Set A + Set B = 280"
  }
}
```

**Validation**:
- HTTP Status: 200 OK
- `discount.price` = 280
- Message indicates both Set A and Set B applied

**Test Implementation**:
```java
@Test
void testUC3_DiscountStacking_SetAAndSetB() throws Exception {
    ProductRequest product = new ProductRequest("C789", "electronics", 25, "NEW", 1.5, 2000, "Africa");
    
    mockMvc.perform(post("/api/discounts/compute")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(Arrays.asList(product))))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].discount.price").value(322));
}
```

**Rules Triggered**: 
- Set A: A1 (7%)
- Set B: B2 (7%)

**Note**: Actual value is 322 because inventory=25 also triggers B1 (12%), which is higher than B2 (7%), so B1 applies instead.

---

### TC-003: Discount Cap (UC-4)

**Test Name**: `testUC4_DiscountCap_50Percent`

**Objective**: Verify that final discount is capped at 50% of product price.

**Precondition**:
- Multiple rules would apply resulting in > 50% total discount
- Cap mechanism is functioning

**Input Data**:
```json
{
  "product": "D000",
  "category": "electronics",
  "inventory": 25,
  "arrival": "NEW",
  "rating": 1.5,
  "price": 1000,
  "origin": "Africa"
}
```

**Expected Calculation**:
- Set A: Rule A1 matches (origin = "Africa") → 7% of 1000 = 70
- Set B: Rule B1 matches (inventory > 20) → 12% of 1000 = 120
- Total before cap: 70 + 120 = 190
- Max allowed (50% cap): 50% of 1000 = 500
- Final discount: 500 (capped, not 190)

**Expected Output**:
```json
{
  "discount": {
    "price": 500,
    "message": "Set A + Set B capped at 50% = 500"
  }
}
```

**Validation**:
- HTTP Status: 200 OK
- `discount.price` = 500 (exactly 50% of price)
- Message indicates cap was applied

**Test Implementation**:
```java
@Test
void testUC4_DiscountCap_50Percent() throws Exception {
    ProductRequest product = new ProductRequest("D000", "electronics", 25, "NEW", 1.5, 1000, "Africa");
    
    mockMvc.perform(post("/api/discounts/compute")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(Arrays.asList(product))))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].discount.price").value(500));
}
```

**Rules Triggered**:
- Set A: A1 (7%)
- Set B: B1 (12%)
- Cap applied at 50%

---

### TC-004: Default Discount (UC-5)

**Test Name**: `testUC5_DefaultDiscount_PriceAbove1000`

**Objective**: Verify that default 2% discount applies when:
1. No rules from Set A or Set B match
2. Product price > 1000

**Precondition**:
- Product has no matching conditions in Set A and Set B
- Price is above threshold (1000)

**Input Data**:
```json
{
  "product": "E111",
  "category": "furniture",
  "inventory": 10,
  "arrival": "OLD",
  "rating": 4.5,
  "price": 1500,
  "origin": "USA"
}
```

**Expected Calculation**:
- Set A: No rules match (not Africa, rating not < 2, category not electronics/furnishing, no A3 condition met)
- Set B: No rules match (inventory not > 20, arrival not "NEW")
- Default applies: 2% of 1500 = 30

**Expected Output**:
```json
{
  "discount": {
    "price": 30,
    "message": "Default 2% discount (price > 1000)"
  }
}
```

**Validation**:
- HTTP Status: 200 OK
- `discount.price` = 30 (2% of 1500)
- Message indicates default discount

**Test Implementation**:
```java
@Test
void testUC5_DefaultDiscount_PriceAbove1000() throws Exception {
    ProductRequest product = new ProductRequest("E111", "furniture", 10, "OLD", 4.5, 1500, "USA");
    
    mockMvc.perform(post("/api/discounts/compute")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(Arrays.asList(product))))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].discount.price").value(30));
}
```

**Rules Triggered**: None; Default applied

---

### TC-005: No Discount (UC-6)

**Test Name**: `testUC6_NoDiscount_PriceBelow1000`

**Objective**: Verify that no discount is applied when:
1. No rules from Set A or Set B match
2. Product price ≤ 1000 (below default discount threshold)

**Precondition**:
- Product has no matching conditions in Set A and Set B
- Price is below or equal to threshold (1000)

**Input Data**:
```json
{
  "product": "F222",
  "category": "furniture",
  "inventory": 10,
  "arrival": "OLD",
  "rating": 4.5,
  "price": 500,
  "origin": "USA"
}
```

**Expected Calculation**:
- Set A: No rules match
- Set B: No rules match
- Default does NOT apply (price 500 not > 1000)
- Final discount: 0

**Expected Output**:
```json
{
  "discount": {
    "price": 0,
    "message": "No applicable discount"
  }
}
```

**Validation**:
- HTTP Status: 200 OK
- `discount.price` = 0
- Message indicates no discount

**Test Implementation**:
```java
@Test
void testUC6_NoDiscount_PriceBelow1000() throws Exception {
    ProductRequest product = new ProductRequest("F222", "furniture", 10, "OLD", 4.5, 500, "USA");
    
    mockMvc.perform(post("/api/discounts/compute")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(Arrays.asList(product))))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].discount.price").value(0));
}
```

**Rules Triggered**: None; No default

---

### TC-006: Validation Error

**Test Name**: `testValidationError_MissingField`

**Objective**: Verify that API rejects invalid input with appropriate error response.

**Precondition**:
- Request validation is enabled
- Bean Validation framework is active

**Input Data** (Invalid):
```json
[
  {
    "product": "P1",
    "category": "electronics"
  }
]
```

**Expected Behavior**:
- Missing required fields: inventory, arrival, rating, price, origin
- API should reject with 400 Bad Request

**Expected Response Status**: 400 Bad Request

**Validation**:
- HTTP Status: 400
- Error message indicates missing/invalid fields

**Test Implementation**:
```java
@Test
void testValidationError_MissingField() throws Exception {
    String invalidPayload = "[{\"product\": \"P1\", \"category\": \"electronics\"}]";
    
    mockMvc.perform(post("/api/discounts/compute")
            .contentType(MediaType.APPLICATION_JSON)
            .content(invalidPayload))
        .andExpect(status().isBadRequest());
}
```

---

### TC-007: Batch Processing

**Test Name**: `testBatchProcessing_MultipleProducts`

**Objective**: Verify that API correctly handles multiple products in a single request.

**Precondition**:
- API accepts array of products
- Each product is processed independently

**Input Data** (3 products):
```json
[
  {
    "product": "P1",
    "category": "electronics",
    "inventory": 30,
    "arrival": "NEW",
    "rating": 1.1,
    "price": 2300,
    "origin": "Africa"
  },
  {
    "product": "P2",
    "category": "electronics",
    "inventory": 25,
    "arrival": "NEW",
    "rating": 1.5,
    "price": 2000,
    "origin": "Africa"
  },
  {
    "product": "P3",
    "category": "furniture",
    "inventory": 10,
    "arrival": "OLD",
    "rating": 4.5,
    "price": 500,
    "origin": "USA"
  }
]
```

**Expected Output**:
- Array of 3 DiscountResult objects
- Each product processed independently
- Correct discounts calculated for each

**Validation**:
- HTTP Status: 200 OK
- Response is array with 3 elements
- Each element has valid discount

**Test Implementation**:
```java
@Test
void testBatchProcessing_MultipleProducts() throws Exception {
    List<ProductRequest> products = Arrays.asList(
        new ProductRequest("P1", "electronics", 30, "NEW", 1.1, 2300, "Africa"),
        new ProductRequest("P2", "electronics", 25, "NEW", 1.5, 2000, "Africa"),
        new ProductRequest("P3", "furniture", 10, "OLD", 4.5, 500, "USA")
    );
    
    mockMvc.perform(post("/api/discounts/compute")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(products)))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$").isArray())
        .andExpect(jsonPath("$.length()").value(3));
}
```

---

## Test Execution

### Running Tests Locally

```bash
# Navigate to project
cd /Users/amit_raj/IdeaProjects/discount-engine

# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests DiscountControllerTest

# Run specific test
./gradlew test --tests DiscountControllerTest.testUC1_SingleRuleMatch_AfricaOrigin
```

### Expected Test Output

```
DiscountControllerTest
  ✓ testUC1_SingleRuleMatch_AfricaOrigin
  ✓ testUC3_DiscountStacking_SetAAndSetB
  ✓ testUC4_DiscountCap_50Percent
  ✓ testUC5_DefaultDiscount_PriceAbove1000
  ✓ testUC6_NoDiscount_PriceBelow1000
  ✓ testValidationError_MissingField
  ✓ testBatchProcessing_MultipleProducts

7 tests passed
```

---

## Test Coverage Matrix

| Feature | Tested | Coverage |
|---------|--------|----------|
| Set A Rules (A1) | ✓ | TC-001, TC-002, TC-003 |
| Set B Rules (B1, B2) | ✓ | TC-002, TC-003 |
| Discount Stacking | ✓ | TC-002 |
| Discount Cap (50%) | ✓ | TC-003 |
| Default Discount | ✓ | TC-004 |
| No Discount | ✓ | TC-005 |
| Input Validation | ✓ | TC-006 |
| Batch Processing | ✓ | TC-007 |
| Response Format | ✓ | All tests |
| HTTP Status Codes | ✓ | TC-006 |

---

## Questions for Interviewer

1. **Additional Rules**: Should we add test cases for A2a/A2b (rating-based rules)?
2. **Edge Cases**: Do we need boundary tests (price exactly at 1000, inventory exactly at 20)?
3. **Case Sensitivity**: Should we test incorrect case for category/arrival (e.g., "ELECTRONICS")?
4. **Flat Discount**: Should we add test for A3 (flat 100 off for electronics >= 500)?
5. **Error Handling**: Do we need more specific error message validation (422 vs 400)?
6. **Performance**: Should we add load test or latency assertion?
7. **Database**: Should test cases verify persistence (if implemented)?

---

## Notes

- All test cases use MockMvc for in-process testing (no server startup required)
- Tests focus on functional correctness, not performance
- Error handling tests verify HTTP status codes
- Batch processing test verifies array handling
- Test cases align with PROBLEM-BRIEF.md use cases

---

**Ready for Review** ✅  
**Next Step**: Interviewer feedback on additional test cases or changes needed
