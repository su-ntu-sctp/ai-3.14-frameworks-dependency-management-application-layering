# Lesson: Frameworks, Dependency Management, and Application Layering

## Lesson Overview
This lesson introduces the key concepts behind Spring Boot's architecture — Inversion of Control, Dependency Injection, and the Service–Repository design pattern. Students learn how Spring manages object creation through beans, apply different injection types, and refactor a simple CRM into a layered, maintainable structure using interfaces and bean configuration.

## Lesson Objectives
By the end of this lesson, students will be able to:

1. **Differentiate** frameworks and libraries and explain how Inversion of Control affects application flow
2. **Declare** and manage beans using `@Component` and `@Bean`, and apply constructor, setter, and field injection
3. **Refactor** a controller-heavy application into layered Controller, Service, and Repository components
4. **Implement** interface-based design and resolve multiple implementations using `@Primary` and `@Qualifier`

---

## Part 1: Frameworks and Libraries

Frameworks and libraries are tools that provide reusable code for developers to build applications.

They are different in the sense that frameworks provide a structure for developers to build applications, while libraries provide specific functionalities that developers can use as needed. Additionally frameworks control the flow of the application (Inversion of Control), while libraries are used to extend the functionality of the application.

For example, Spring Boot is a framework in which the flow of our code is controlled by the framework. And logback is a library that we use the functionalities as needed.

<img src="https://velog.velcdn.com/images/binest03459/post/f1f13f8b-a582-41f8-8efd-7e4cecf9ad80/image.jpg" width=450>

> Source: https://velog.io/@binest03459/Library-vs.-Framework

Further reading:
https://www.shiksha.com/online-courses/articles/framework-vs-library/

---

## Part 2: Inversion of Control (IoC) and Dependency Injection (DI)

The concept of **Inversion of Control** means that the flow of the application is controlled by the framework.

In a typical Java application, the flow of the application is controlled by the developer. The developer decides when to create instances of classes, when to call methods, etc.

In IoC, the control of creating and managing objects is inverted and given to the framework. Instead of creating instances of classes, the developer will declare the dependencies of the class and let the framework create the instances of the classes and inject them into the class. This is known as **Dependency Injection**. In this way, the components of your application are loosely coupled, which promotes modularity, reusability and testability.

<img src="https://devopedia.org/images/article/30/4020.1536743448.gif">

> Source: https://devopedia.org/dependency-injection

---

## Part 3: More on Beans and Dependency Injection

In the entry point of every Spring Boot application i.e. our `main` method, you will see the `@SpringBootApplication` annotation. This annotation is a combination of 3 annotations:

1. `@Configuration` - Indicates that the class contains `@Bean` annotations, pick them up and add them into the spring container.
2. `@ComponentScan` - To scan for all `@Component` annotated classes located in the same package (or explicitly specified) and add them to the spring container.
3. `@EnableAutoConfiguration` - Looks for auto-configuration beans (java classes) and adds them into the spring container.

Therefore, `@SpringBootApplication` annotation is a Spring Boot feature to quickly bootstrap the default/commonly used annotations into one.

Classes that are annotated with `@Component` are known as **Spring Beans**. Spring Beans are managed by the Spring IoC container, also known as the Spring Context or Application Context. The Spring IoC container is responsible for instantiating, configuring, and assembling the Spring Beans.

<img src="https://gustavopeiretti.com/spring-injection-dependencies/spring-injection-en-2.png" width=450>

> Source: https://gustavopeiretti.com/spring-injection-dependencies/

`@Autowired` is used to inject the dependencies into the class. There are multiple ways to inject dependencies into a class.

| Type | Description | Remarks |
|---|---|---|
| Constructor | Dependencies are injected through the constructor | Preferred method of dependency injection |
| Setter | Dependencies are injected through setter methods | Legacy approach — rarely used in modern Spring applications |
| Field | Dependencies are injected directly into the class property | **Do not use** — breaks testability (see note below) |

> ⚠️ **Why field injection is a problem:** When you use field injection, Spring injects the dependency using reflection behind the scenes. This means there is no way to inject a mock or a substitute during unit testing without a Spring container running. In other words, your class becomes impossible to test in isolation. Constructor injection, on the other hand, lets you pass in any implementation directly in a test — no Spring required. This is the primary reason field injection is considered bad practice in production codebases.

Let's create a simple Spring Boot application `di-demo` to see how all these work. Add the Spring Web and Spring Boot DevTools dependencies in `pom.xml`:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
</dependency>
```

Let's create `MathTeacher.java` and `ScienceTeacher.java` classes:

`MathTeacher.java`

```java
public class MathTeacher {
  public String teach() {
    return "Teaching Math";
  }
}
```

`ScienceTeacher.java`

```java
public class ScienceTeacher {
  public String teach() {
    return "Teaching Science";
  }
}
```

Create `TeacherController.java`:

```java
@RestController
public class TeacherController {
  private MathTeacher mathTeacher = new MathTeacher();
  private ScienceTeacher scienceTeacher = new ScienceTeacher();

  @GetMapping("/math-teacher")
  public String mathTeacher() {
    return mathTeacher.teach();
  }

  @GetMapping("/science-teacher")
  public String scienceTeacher() {
    return scienceTeacher.teach();
  }
}
```

Test out the endpoints.

Currently we are creating the instances ourselves. Let's use dependency injection instead.

In order to use dependency injection, we need to let Spring Boot know that `MathTeacher` and `ScienceTeacher` are Spring Beans. We can do this by annotating them with `@Component`. After you do this, you can see these beans in your Spring Boot Dashboard.

Let's use field injection for the science teacher by adding the `@Autowired` annotation to the `scienceTeacher` field:

```java
// private ScienceTeacher scienceTeacher = new ScienceTeacher();
@Autowired
private ScienceTeacher scienceTeacher;
```

Notice now that without having to instantiate the `ScienceTeacher` class, we can still use the `scienceTeacher` bean.

Now, field injection is not ideal — see the note above for why. Let's use constructor injection instead:

```java
private ScienceTeacher scienceTeacher;

public TeacherController(ScienceTeacher scienceTeacher) {
  this.scienceTeacher = scienceTeacher;
}
```

> 📝 **Note:** In older Spring code, you will often see `@Autowired` on constructors. Since Spring 4.3, if a class has only one constructor, Spring automatically uses it for injection — **`@Autowired` is no longer needed on constructors**. This is now the standard practice. You may still see it in legacy codebases, but new code should omit it.

Test it out to make sure it still works.

Behind the scenes, what Spring is doing is this:

```java
// Create a new instance of ScienceTeacher
ScienceTeacher scienceTeacher = new ScienceTeacher();
// Inject the instance into the constructor
TeacherController teacherController = new TeacherController(scienceTeacher);
```

Now, let's see how setter injection works on the MathTeacher bean:

```java
private MathTeacher mathTeacher;

@Autowired
public void setMathTeacher(MathTeacher mathTeacher) {
  this.mathTeacher = mathTeacher;
}
```

> 📝 **Note:** Setter injection was more common in early Spring applications (pre-Spring 3). In modern Spring applications, constructor injection is strongly preferred. Setter injection is considered a **legacy pattern** — you may encounter it in older codebases, but it is rarely written in new production code. The main use case it was designed for (optional dependencies that could be changed after construction) is now handled better through other patterns.
>
> ⚠️ **Important:** Unlike constructor injection, setter injection is **never auto-detected** by Spring — even if there is only one setter. You must explicitly annotate the setter with `@Autowired`, or Spring will never call it, leaving the field `null` and causing a `NullPointerException` when it's used.

Then call the `/math-teacher` endpoint to test it out.

Behind the scenes, Spring is doing this:

```java
// Create a new instance of MathTeacher
MathTeacher mathTeacher = new MathTeacher();
// Inject the instance into the setter
teacherController.setMathTeacher(mathTeacher);
```

### 👨‍💻 Activity **(10 minutes)**

Add a `CodingTeacher` and use constructor injection to inject it into the `TeacherController`.

Add an `AlgorithmsTeacher` and use setter injection to inject it into the `TeacherController`.

Add a `DatabaseTeacher` and use field injection to inject it into the `TeacherController`.

Add the corresponding endpoints to test out the beans.

---

## Part 4: @Bean

Other than using `@Component` to configure instances, `@Bean` can be used to configure instances that contain default configuration.

Usage examples:

- You want to autowire an email service instance with pre-defined SMTP settings such as your email address and email server details.
- You want to use an external library such as DocuSign for digital signing purposes — since you don't have access to the library's source code, you can't annotate it with `@Component`, so `@Bean` is the way to go.

Let's create a dummy email service:

```java
public class EmailService {

  private String replyTo;

  public void send(String message) {
    System.out.println("📫 Sending email...");
    System.out.println("📫 Message: " + message);
    System.out.println("📫 Reply to: " + this.replyTo);
  }

  public String getReplyTo() {
    return this.replyTo;
  }

  public void setReplyTo(String replyTo) {
    this.replyTo = replyTo;
  }
}
```

And now we want to use this class as a bean. We can do this by annotating a method with `@Bean` in a `@Configuration` class. The `@Configuration` annotation indicates that the class contains `@Bean` methods which Spring will pick up and add into the spring container.

```java
@Configuration
public class EmailConfig {

  @Bean
  public EmailService emailService() {
    // Configure our email service bean
    EmailService emailService = new EmailService();
    emailService.setReplyTo("nickfury@avengers.com");
    return emailService;
  }
}
```

The `@Bean` annotation here specifies that the method should be used to create a bean of type `EmailService`. The method's return value is the instance that will be registered as a bean in the Spring container.

Now we can inject this bean in our `TeacherController` and use it:

```java
@Autowired
private EmailService emailService;

@GetMapping("/science-teacher")
public String scienceTeacher() {
  emailService.send("Hello from scienceTeacher()");
  return scienceTeacher.teach();
}
```

You can also see the beans in the Spring Boot Dashboard now.

Test calling the `/science-teacher` endpoint.

In this example, we could have annotated the `EmailService` class with `@Component` directly. However, `@Bean` is the right approach when the class belongs to an external library whose source code you cannot modify. This is just a basic example — in real-world scenarios, beans often have more complex configurations and dependencies.

---

## Part 5: Service and Repository Pattern

Let's go back to our `simple-crm` application.

So far, all of our code is in the controller layer. This is not ideal because we are mixing our business logic with our controller logic. This makes our code difficult to maintain and test.

### Single Responsibility Principle

In programming, SOLID is a mnemonic acronym for five design principles intended to make software designs more understandable, flexible and maintainable. The **Single Responsibility Principle (SRP)** states that every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class.

<img src="https://miro.medium.com/v2/resize:fit:1000/format:webp/1*PxIES4LBAMi8K4RudiP-tw.jpeg">

> Source: https://medium.com/@anisha.nicole/single-responsibility-principle-cabba52aa467

Read more about SOLID [here](https://en.wikipedia.org/wiki/SOLID).

As you might have noticed, our `CustomerController` class is doing more than one thing. It is managing the HTTP requests, handling some business logic, as well as performing CRUD operations on our `ArrayList`.

### Service and Repository

The Service and Repository pattern is a common design pattern used in Java applications.

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*neBcAZJyLGpE7KHc3sH8bw.png" width=400>

> Source: https://tom-collings.medium.com/controller-service-repository-16e29a4684e5

Instead of putting all our code in the controller layer, we can separate our code into 3 different layers:

| Layer | Purpose |
|---|---|
| Controller | Handles HTTP requests and responses |
| Service | Handles business logic |
| Repository | Handles CRUD operations |

The controller should only handle HTTP requests and responses. The repository should only handle CRUD operations. The service should handle all the business logic e.g. validation, data manipulation, etc.

Hence, it is often suggested to have **thin controllers/repositories and fat services**. This helps to keep the controllers and repositories simple and easy to maintain. More importantly, it centralizes all the business logic in the service layer.

---

## Part 6: Refactoring Our `simple-crm`

### Repository Layer

Since the repository layer is responsible for CRUD operations, we will create a `CustomerRepository` class to handle all the CRUD operations on our `ArrayList`.

Only the repository should have access to the data store. Hence the `ArrayList` should be private and only accessible within the `CustomerRepository` class.

This class also needs to be annotated with `@Repository` to let Spring Boot know that it is a Spring Bean.

> 📝 **`@Component` vs `@Service` vs `@Repository` — What's the difference?**
>
> All three annotations register a class as a Spring Bean. The difference is in **intent and behaviour**:
>
> - `@Component` — The generic stereotype. Use it when the class doesn't clearly fit as a service or repository.
> - `@Service` — A specialization of `@Component`. It carries no extra technical behaviour today, but it communicates clearly that this class contains **business logic**. Frameworks and tools can also use this marker for additional processing in future.
> - `@Repository` — A specialization of `@Component` with one important technical addition: Spring automatically translates **persistence-layer exceptions** (e.g. database errors) into Spring's unified `DataAccessException` hierarchy. This makes error handling consistent regardless of whether you're using JDBC, JPA, or any other data access technology. Always use `@Repository` on your data access classes.
>
> In short: use the most specific annotation that fits. It makes your intent clear to other developers and to the framework.

```java
@Repository
public class CustomerRepository {

  private ArrayList<Customer> customers = new ArrayList<>();

  // Preload data here now
  public CustomerRepository() {
    customers.add(new Customer("Peter", "Parker"));
    customers.add(new Customer("Stephen", "Strange"));
    customers.add(new Customer("Steve", "Rogers"));
  }

  // Create
  public Customer createCustomer(Customer customer) {
    customers.add(customer);
    return customer;
  }

  // Get One
  public Customer getCustomer(int index) {
    return customers.get(index);
  }

  // Get All
  public List<Customer> getAllCustomers() {
    return customers;
  }

  // Update
  public Customer updateCustomer(int index, Customer customer) {
    Customer customerToUpdate = customers.get(index);
    customerToUpdate.setFirstName(customer.getFirstName());
    customerToUpdate.setLastName(customer.getLastName());
    customerToUpdate.setEmail(customer.getEmail());
    customerToUpdate.setContactNo(customer.getContactNo());
    customerToUpdate.setJobTitle(customer.getJobTitle());
    customerToUpdate.setYearOfBirth(customer.getYearOfBirth());
    return customerToUpdate;
  }

  // Delete
  public void deleteCustomer(int index) {
    customers.remove(index);
  }
}
```

> 📝 **Why `List<Customer>` instead of `ArrayList<Customer>`?** Coding to an interface applies to collections too, not just your own classes. By returning `List<Customer>` instead of `ArrayList<Customer>`, you keep the flexibility to swap the underlying implementation (e.g. to `LinkedList`) without changing any calling code. This is standard industry practice — always return the interface type, not the concrete collection class.

As you can see, the purpose of this layer is just to perform CRUD operations on our `ArrayList`. It does not contain any business logic.

### Service Layer

Next, we will create a `CustomerService` class to handle all the business logic.

`CustomerService` will need to call our `CustomerRepository` to perform CRUD operations since updating the data store is the responsibility of the repository layer. We also want to move our helper function `getCustomerIndex()` from `CustomerController` to `CustomerService` because it is part of the business logic.

The service class needs to be annotated with `@Service` to let Spring Boot know that it is a Spring Bean. The `@Service` annotation is a specialization of the `@Component` annotation.

```java
@Service
public class CustomerService {

  private final CustomerRepository customerRepository;

  public CustomerService(CustomerRepository customerRepository) {
    this.customerRepository = customerRepository;
  }

  public Customer createCustomer(Customer customer) {
    return customerRepository.createCustomer(customer);
  }

  public Customer getCustomer(String id) {
    return customerRepository.getCustomer(getCustomerIndex(id));
  }

  public List<Customer> getAllCustomers() {
    return customerRepository.getAllCustomers();
  }

  public Customer updateCustomer(String id, Customer customer) {
    return customerRepository.updateCustomer(getCustomerIndex(id), customer);
  }

  public void deleteCustomer(String id) {
    customerRepository.deleteCustomer(getCustomerIndex(id));
  }

  private int getCustomerIndex(String id) {
    for (Customer customer : customerRepository.getAllCustomers()) {
      if (customer.getId().equals(id)) {
        return customerRepository.getAllCustomers().indexOf(customer);
      }
    }
    throw new CustomerNotFoundException(id);
  }
}
```

### Controller Layer

Finally, we will modify our `CustomerController` to use the `CustomerService` class. Notice we are using constructor injection here — we let Spring manage the `CustomerService` instance for us instead of creating it with `new`.

> 📝 **Why not use `new CustomerService()` here?** Service classes like `CustomerService` are designed to provide functionality — not to hold data. We only ever need one instance of it in the entire application. If every class that needs `CustomerService` called `new CustomerService()`, we'd end up with multiple unnecessary instances. By using constructor injection, Spring creates exactly one instance and reuses it everywhere — this is the **Singleton pattern**, which is the default behaviour for all Spring beans. You can read more about bean scopes [here](https://www.baeldung.com/spring-bean-scopes).

```java
@RestController
@RequestMapping("/customers")
public class CustomerController {

  private final CustomerService customerService;

  public CustomerController(CustomerService customerService) {
    this.customerService = customerService;
  }

  // CREATE
  @PostMapping("")
  public ResponseEntity<Customer> createCustomer(@RequestBody Customer customer) {
    Customer newCustomer = customerService.createCustomer(customer);
    return new ResponseEntity<>(newCustomer, HttpStatus.CREATED);
  }

  // READ (GET ALL)
  @GetMapping("")
  public ResponseEntity<List<Customer>> getAllCustomers() {
    List<Customer> allCustomers = customerService.getAllCustomers();
    return new ResponseEntity<>(allCustomers, HttpStatus.OK);
  }

  // READ (GET ONE)
  @GetMapping("{id}")
  public ResponseEntity<Customer> getCustomer(@PathVariable String id) {
    try {
      Customer foundCustomer = customerService.getCustomer(id);
      return new ResponseEntity<>(foundCustomer, HttpStatus.OK);
    } catch (CustomerNotFoundException e) {
      return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
  }

  // UPDATE
  @PutMapping("{id}")
  public ResponseEntity<Customer> updateCustomer(@PathVariable String id, @RequestBody Customer customer) {
    try {
      Customer updatedCustomer = customerService.updateCustomer(id, customer);
      return new ResponseEntity<>(updatedCustomer, HttpStatus.OK);
    } catch (CustomerNotFoundException e) {
      return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
  }

  // DELETE
  @DeleteMapping("{id}")
  public ResponseEntity<HttpStatus> deleteCustomer(@PathVariable String id) {
    try {
      customerService.deleteCustomer(id);
      return new ResponseEntity<>(HttpStatus.NO_CONTENT);
    } catch (CustomerNotFoundException e) {
      return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
  }
}
```

Notice that `customerService` is declared `final`. This is a best practice with constructor injection — since the dependency is set once in the constructor and never changes, marking it `final` makes that explicit and prevents accidental reassignment.

Test the endpoints again after refactoring the code. They should still work as before.

### Coding to an Interface

**Coding to an interface** means writing our code to be dependent on an interface instead of a concrete class. This promotes loose coupling and makes our code more flexible and easy to change.

For our service layer, it is a good practice to code to an interface. This is because we may want to change the implementation of our service layer in the future.

Let's rename our `CustomerService.java` to `CustomerServiceImpl.java` and create a new interface called `CustomerService.java` with all the method signatures:

```java
public interface CustomerService {
  Customer createCustomer(Customer customer);
  Customer getCustomer(String id);
  List<Customer> getAllCustomers();
  Customer updateCustomer(String id, Customer customer);
  void deleteCustomer(String id);
}
```

Next, our `CustomerServiceImpl` class should implement the `CustomerService` interface. **Remember to add the `implements CustomerService` clause** — renaming the class alone does not make it implement the new interface, and if this step is missed, `CustomerServiceImpl` will not be considered a valid candidate for `CustomerService` type injection later on:

```java
@Service
public class CustomerServiceImpl implements CustomerService {

  private final CustomerRepository customerRepository;

  public CustomerServiceImpl(CustomerRepository customerRepository) {
    this.customerRepository = customerRepository;
  }

  @Override
  public Customer createCustomer(Customer customer) {
    return customerRepository.createCustomer(customer);
  }

  @Override
  public Customer getCustomer(String id) {
    return customerRepository.getCustomer(getCustomerIndex(id));
  }

  @Override
  public List<Customer> getAllCustomers() {
    return customerRepository.getAllCustomers();
  }

  @Override
  public Customer updateCustomer(String id, Customer customer) {
    return customerRepository.updateCustomer(getCustomerIndex(id), customer);
  }

  @Override
  public void deleteCustomer(String id) {
    customerRepository.deleteCustomer(getCustomerIndex(id));
  }

  private int getCustomerIndex(String id) {
    for (Customer customer : customerRepository.getAllCustomers()) {
      if (customer.getId().equals(id)) {
        return customerRepository.getAllCustomers().indexOf(customer);
      }
    }
    throw new CustomerNotFoundException(id);
  }
}
```

Note that we do not have to change anything in `CustomerController.java` as it is already using the `CustomerService` type:

```java
private final CustomerService customerService;

public CustomerController(CustomerService customerService) {
  this.customerService = customerService;
}
```

When Spring Boot encounters a `CustomerService` type dependency in the `CustomerController`, it will look for a bean that implements the `CustomerService` interface. Since we have annotated our `CustomerServiceImpl` class with `@Service`, Spring Boot will create a bean of type `CustomerServiceImpl` and inject it into the `CustomerController`.

Test the endpoints again to make sure they still work.

### @Primary and @Qualifier

Now let's say we want a second implementation of our service layer that logs all method calls. Create `CustomerServiceWithLoggingImpl.java`:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class CustomerServiceWithLoggingImpl implements CustomerService {

  private static final Logger logger = LoggerFactory.getLogger(CustomerServiceWithLoggingImpl.class);
  private final CustomerRepository customerRepository;

  public CustomerServiceWithLoggingImpl(CustomerRepository customerRepository) {
    this.customerRepository = customerRepository;
  }

  @Override
  public Customer createCustomer(Customer customer) {
    logger.info("CustomerServiceWithLoggingImpl.createCustomer() called");
    return customerRepository.createCustomer(customer);
  }

  @Override
  public Customer getCustomer(String id) {
    logger.info("CustomerServiceWithLoggingImpl.getCustomer() called");
    return customerRepository.getCustomer(getCustomerIndex(id));
  }

  @Override
  public List<Customer> getAllCustomers() {
    logger.info("CustomerServiceWithLoggingImpl.getAllCustomers() called");
    return customerRepository.getAllCustomers();
  }

  @Override
  public Customer updateCustomer(String id, Customer customer) {
    logger.info("CustomerServiceWithLoggingImpl.updateCustomer() called");
    return customerRepository.updateCustomer(getCustomerIndex(id), customer);
  }

  @Override
  public void deleteCustomer(String id) {
    logger.info("CustomerServiceWithLoggingImpl.deleteCustomer() called");
    customerRepository.deleteCustomer(getCustomerIndex(id));
  }

  private int getCustomerIndex(String id) {
    for (Customer customer : customerRepository.getAllCustomers()) {
      if (customer.getId().equals(id)) {
        return customerRepository.getAllCustomers().indexOf(customer);
      }
    }
    throw new CustomerNotFoundException(id);
  }
}
```

Now when you try to run the application, you will get an error:

```
Parameter 0 of constructor in CustomerController required a single bean, but 2 were found
```

This is because Spring Boot does not know which bean to inject since we have 2 beans that implement the `CustomerService` interface. There are two ways to resolve this.

The first way is to annotate `CustomerServiceImpl` with `@Primary` to mark it as the default implementation:

```java
@Primary
@Service
public class CustomerServiceImpl implements CustomerService {
  // ...
}
```

The second way is to use `@Qualifier` in the controller to specify exactly which implementation to inject. The bean name is the class name with the first letter in lowercase:

```java
public CustomerController(@Qualifier("customerServiceWithLoggingImpl") CustomerService customerService) {
  this.customerService = customerService;
}
```

By coding to an interface, we can easily swap implementations without touching the controller at all.

> 📝 **Note:** `@Primary` and `@Qualifier` can be used together. If both are present, `@Qualifier` at the injection point wins over `@Primary` on the bean — it's a more specific instruction at the point of use.

---

> 📖 **Further Reading: Bean Scope**
>
> The scope of a bean defines its lifecycle — how long Spring keeps the bean around and when to create a new instance. By default, all Spring beans are **singleton** scoped, meaning Spring creates only one instance and reuses it for the entire application lifetime. This can be changed as needed for your use case.
>
> - [Java Singleton Pattern — Baeldung](https://www.baeldung.com/java-singleton)
> - [Spring Bean Scopes — Baeldung](https://www.baeldung.com/spring-bean-scopes)

---

END