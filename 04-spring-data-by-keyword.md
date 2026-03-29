# Tip: Spring Data — Don't Forget `By` in Repository Methods

---

## The Mistake

```java
// Missing 'By' — app crashes at startup
List<Product> findProductNameLikeIgnoreCase(String keyword);
```

No runtime warning. No compile error. Just this at startup:

```
PropertyReferenceException: No property 'productNameLikeIgnoreCase' found for type 'Product'
```

---

## Why It Happens

Spring Data parses your method name like a sentence:

```
findBy  →  ProductName  →  Containing  →  IgnoreCase
  ↑            ↑               ↑              ↑
action      field name       modifier       modifier
```

`By` is the **mandatory separator** between the action and the field name.

Without it, Spring Data treats everything after `find` as a field name — looks for `productNameLikeIgnoreCase` on your entity, doesn't find it, crashes.

---

## The Fix

```java
//  'By' present — Spring Data parses correctly
List<Product> findByProductNameContainingIgnoreCase(String keyword);
```

Generates:
```sql
WHERE LOWER(product_name) LIKE LOWER('%keyword%')
```

---

## Quick Reference — Valid Prefixes + `By`

```java
findBy...       // SELECT
readBy...       // SELECT (alias)
getBy...        // SELECT (alias)
searchBy...     // SELECT (alias)
existsBy...     // SELECT → returns boolean
countBy...      // SELECT COUNT
deleteBy...     // DELETE
```

All of them require `By` immediately before the field name. No exceptions.

---

## Why It Crashes at Startup — Not Runtime

Spring Data validates all repository methods when the `ApplicationContext` boots — not when the method is first called. This is intentional: **fail fast.** A bad method name is a deployment blocker, not a surprise in production.

---

## Golden Rule

> Every Spring Data query method needs `By` as the separator.  
> No `By` = Spring Data is lost = startup crash.

```java
//  findProductNameContaining(...)
//  findByProductNameContaining(...)
```

---
