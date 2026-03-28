# Post-Initialization: The Health Check

After generating a project from **Spring Initializr**, the first priority is verifying that the Tomcat server and Spring Context are running correctly. Do this **before** touching the database or business logic.

---

###  The Strategy
Create a "shallow" endpoint. It doesn't use a service or a database; it simply returns a string. If this works, your **Port (8080)**, **Java Version**, and **Dependencies** are all synced.

###  The Implementation
Create this class in your `controller` package:

```java
package com.yourname.project.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.time.LocalDateTime;

@RestController
public class HealthController {

    @GetMapping("/api/health")
    public String check() {
        return "Backend is running!  Time: " + LocalDateTime.now();
    }
}

```

---

###  Verification Steps

1. **Run the App:** Use your IDE's "Run" button or terminal: `./mvnw spring-boot:run`.
2. **Verify Logs:** Look for the message: `Started [ApplicationName] in X seconds`.
3. **Hit the URL:** Open your browser or Postman to: `http://localhost:8080/api/health`

---

###  Why do this first?

* **Port Conflicts:** If another app is using `8080`, you’ll find out now instead of 2 hours later.
* **Dependency Validation:** If you selected incompatible versions in Initializr, the app will crash immediately.
* **Logical Baseline:** If the app fails later, you know the foundation was solid.

###  Common Pitfalls

* **Package Scanning:** Ensure your controller is in a sub-package of your main `@SpringBootApplication` class. If it is "above" or "outside" that folder, Spring won't find it.
* **Annotation Check:** Forgetting `@RestController` will result in a **404 Not Found**, even if the code is correct.

```
