# Assignment (Optional)

## Brief

Create a Spring Boot project called ProductManagementSystem and refactor it into a proper layered architecture with dependency injection.

1. **Layered Architecture with Service and Repository Pattern**
   - Create a `Product` class with attributes:
     - `String id` (use UUID)
     - `String name`
     - `String category`
     - `double price`
   - Use Lombok's @Getter and @Setter
   - Create three layers following the Service-Repository pattern:
     - **Repository Layer**: Create `ProductRepository` class with @Repository
       - Store products in an ArrayList
       - Implement methods: `createProduct()`, `getAllProducts()`, `getProduct(int index)`, `deleteProduct(int index)`
       - Pre-populate with 3 products in constructor
     - **Service Layer**: Create `ProductService` interface and `ProductServiceImpl` class with @Service
       - Interface should declare: `createProduct()`, `getAllProducts()`, `getProduct(String id)`, `deleteProduct(String id)`
       - Implementation should contain business logic including a helper method `getProductIndex(String id)`
       - Use constructor injection to inject ProductRepository
     - **Controller Layer**: Create `ProductController` class with @RestController and @RequestMapping("/products")
       - Use constructor injection to inject ProductService
       - Implement endpoints: POST /products, GET /products, GET /products/{id}, DELETE /products/{id}
       - Use ResponseEntity with appropriate status codes
   - Test all endpoints

2. **Dependency Injection Practice**
   - Create a simple `NotificationService` interface with a method `sendNotification(String message)`
   - Create two implementations:
     - `EmailNotificationService` - prints "📧 Sending email: [message]"
     - `SMSNotificationService` - prints "📱 Sending SMS: [message]"
   - Annotate both with @Service
   - Use @Primary on EmailNotificationService
   - In ProductController, inject NotificationService using constructor injection
   - Call `sendNotification()` method in the createProduct endpoint with message "New product created: [product name]"
   - Test and verify EmailNotificationService is used by default
   - Then change to use SMSNotificationService by using @Qualifier annotation
   - Test again to verify SMSNotificationService is now used

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- PostgreSQL: https://www.postgresql.org/docs/
- OWASP: https://cheatsheetseries.owasp.org/
