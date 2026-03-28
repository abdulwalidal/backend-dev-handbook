# Tip: Always Use Wrapper Classes in Spring Boot Entities

---

## The Mistake Every Beginner Makes

When you first create a JPA entity, it's tempting to write this:

```java
@Entity
public class Product {
    @Id
    private long id;       // primitive
    private double price;  // primitive
    private int stock;     // primitive
}
```

Looks clean. Compiles fine. But it will silently break your application.

---

## Why Primitives Break JPA

### Primitives have default values. Wrapper classes have `null`.

| Type | Default Value |
|---|---|
| `long` | `0` |
| `int` | `0` |
| `double` | `0.0` |
| `Long` | `null` |
| `Integer` | `null` |
| `Double` | `null` |

This default value is the root of the problem.

---

## How JPA Decides: INSERT or UPDATE?

When you call `productRepository.save(product)`, Spring Data asks one question:

> **"Is the ID null?"**
> - `null` -> **INSERT** (new record)
> - has a value -> **UPDATE** (existing record)

```java
// Wrapper — works correctly
Product p = new Product();  // id = null
productRepository.save(p);  // id is null -> fires INSERT 

// Primitive — silent bug
Product p = new Product();  // id = 0 (default)
productRepository.save(p);  // id is 0 -> Spring Data thinks entity EXISTS -> fires UPDATE
                             // 0 rows updated -> no exception -> silent failure 
```

With a primitive `long id`, your new product is **never saved** — and the app doesn't throw an error. It just quietly does nothing.

---

## The Correct Entity Setup

```java
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    // tells JPA: "let the DB auto-generate this ID"
    private Long id;                    // null = new entity -> INSERT

    private String name;
    private Double price;               // null = price not set yet (not zero)
    private Integer stock;              // null = stock not tracked (not zero)
    private Double discountPercentage;  // null = no discount applied
}
```

---

## The Full JPA save() Flow — Step by Step

Here's exactly what happens when you save a new product:

```java
Product p = new Product();
p.setName("iPhone 15");
p.setPrice(999.99);
p.setDiscountPercentage(null);  // no discount yet

productRepository.save(p);
```

```
Step 1 -> Spring Data checks: is id null?
Step 2 -> id is null -> decides to INSERT
Step 3 -> Hibernate fires:
         INSERT INTO product (name, price, discount_percentage)
         VALUES ('iPhone 15', 999.99, NULL)
Step 4 -> Database generates id = 1 (AUTO_INCREMENT)
Step 5 -> Hibernate reads the generated key back
Step 6 -> your object p now has p.getId() = 1L 
```

---

## Real World — Null Means Something

Wrapper classes let you express intent clearly in your service layer:

```java
public Double getFinalPrice(Long productId) {
    Product product = productRepository.findById(productId)
            .orElseThrow(() -> new RuntimeException("Product not found"));

    if (product.getDiscountPercentage() == null) {
        return product.getPrice();   // no discount — return full price
    }

    return product.getPrice() * (1 - product.getDiscountPercentage() / 100);
}
```

With a primitive `double discountPercentage`, you can't tell if `0.0` means:
- "no discount applied" or
- "discount accidentally set to zero"

`null` is unambiguous. Primitives are not.

---

## The Golden Rules

1. **Always use Wrapper classes** (`Long`, `Integer`, `Double`) in JPA entities — never primitives
2. **Always add @GeneratedValue** — without it, the DB has no instruction to generate an ID
3. **null = not set. 0 = zero.** These are not the same thing — don't let your DB treat them as such
4. **Use primitives only** for local variables and pure computation — never for entity fields

---
