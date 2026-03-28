#  Database Handshake (Persistence Check)

Once the server is running, you must prove the code can successfully talk to a database. Using **H2 (In-Memory)** is the best "Pro Tip" here because it requires zero installation and confirms your JPA setup is correct.

---

###  The Strategy
The goal is to verify that your environment is ready for data. By defining a simple Java class, we check if the database engine recognizes our blueprints. If the table appears in the database console, your data layer is officially "live."

---

###  The Implementation

#### 1. The Configuration (`application.properties`)
These settings are the minimum required to make the in-memory database accessible via your browser.

```properties
# Enable the H2 Browser UI (Access via /h2-console)
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb

```

#### 2. The Test Entity

Create a minimal class to trigger the database schema generation.

```java
@Entity
public class ConnectionTest {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String status;
}

```

---

###  How to Verify

1. **Start the App:** Ensure the application context loads without JPA errors.
2. **Open the Console:** Go to `http://localhost:8080/h2-console`.
3. **Connect:** **Crucial:** Match the JDBC URL field in the login screen to `jdbc:h2:mem:testdb`.
4. **Result:** If the `CONNECTION_TEST` table is listed in the sidebar, your database handshake is successful.

---
