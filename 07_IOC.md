# IOC (Inversion of Control) in Spring Boot

## What is IOC?

IOC (Inversion of Control) means Spring takes control of object creation and dependency management instead of the programmer creating objects manually using `new`.

---

#  Without Spring

```java
UserService service = new UserService();
```

Programmer is responsible for:

* Creating objects
* Managing dependencies
* Passing objects manually

### Problems

* Tight Coupling
* Difficult Testing
* Hard to Manage Large Projects

---

#  With Spring IOC

```java
@Service
public class UserService {

}
```

Spring automatically:

* Creates object
* Stores object
* Manages lifecycle

That object is called a:

```text
Bean
```

---

#  What is IOC Container?

IOC Container (`ApplicationContext`) is responsible for:

* Creating Beans
* Storing Beans
* Injecting Dependencies

---

#  What Does `@Autowired` Do?

```java
@Autowired
private UserService service;
```

`@Autowired` tells Spring:

> "Find the existing `UserService` bean and inject it here."

### Important

* `@Autowired` does NOT create object
* Spring already created it using `@Service`

---

# ⚙️ Internal Flow

```text
Application Starts
       ↓
Spring Scans Classes
       ↓
Finds @Service/@Component
       ↓
Creates Bean Objects
       ↓
Stores Beans in IOC Container
       ↓
@Autowired Requests Bean
       ↓
Spring Injects Dependency
```

---

# Common Bean Annotations

```java
@Component
@Service
@Repository
@Controller
@RestController
```

All of these create Spring-managed Beans.

---

## ➤ What is IOC?

A design principle where Spring manages object creation and dependencies.

---

## ➤ What is a Bean?

An object managed by Spring IOC Container.

---

## ➤ What does `@Autowired` do?

Injects an existing Bean automatically.

---

## ➤ Does `@Autowired` create objects?

No. Spring creates objects first, then injects them.

---

## Without Spring

```text
Classes create dependencies manually
```

## With Spring

```text
Spring creates and injects dependencies automatically
```
