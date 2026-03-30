# Tip: @NotBlank Does Nothing Without @Valid

---

## The Mistake

```java
// Constraint declared on the field
public class Product {
    @NotBlank(message = "Product name cannot be blank")
    private String productName;
}

// But @Valid is missing on the controller
@PostMapping("/products")
public ResponseEntity<ProductDTO> addProduct(@RequestBody Product product) {
    // @NotBlank is completely ignored
    // blank productName passes through silently
}
```

User sends:
```json
{
  "productName": ""
}
```

No error. No exception. Empty string goes straight to your service and into the DB.

---

## Why It Happens

`@NotBlank` is just a **marker annotation** — it declares a constraint but does not enforce it.

`@Valid` tells Spring to trigger the **Hibernate Validator** on that object before the method executes.

```
@Valid present   → Hibernate Validator runs → reads @NotBlank → enforces it 
@Valid missing   → Hibernate Validator never runs → @NotBlank ignored        
```

They only work as a pair. One without the other is broken validation.

---

## The Fix

```java
// Controller — @Valid triggers validation
@PostMapping("/products")
public ResponseEntity<ProductDTO> addProduct(@Valid @RequestBody Product product) {
    // now @NotBlank is enforced before method body executes
}

// Entity / DTO — constraint declared
public class Product {
    @NotBlank(message = "Product name cannot be blank")
    private String productName;

    @NotNull(message = "Price cannot be null")
    private Double price;

    @Min(value = 0, message = "Discount cannot be negative")
    private Double discount;
}
```

When validation fails, Spring automatically returns:
```json
{
  "status": 400,
  "errors": ["Product name cannot be blank"]
}
```


## Common Validation Annotations

```java
@NotNull       // field cannot be null
@NotBlank      // String cannot be null or empty or whitespace
@NotEmpty      // String/Collection cannot be null or empty
@Min(value)    // number must be >= value
@Max(value)    // number must be <= value
@Size(min, max)// String/Collection size must be within range
@Email         // must be a valid email format
@Positive      // number must be > 0
```

---

## Golden Rule

> `@NotBlank` declares the rule.
> `@Valid` enforces it.
> Without `@Valid` on the controller parameter — validation never runs.

---
