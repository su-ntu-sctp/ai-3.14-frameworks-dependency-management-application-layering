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

We will build a small order-processing example. Let's create `TaxCalculator.java` and `ShippingCalculator.java` classes:

`TaxCalculator.java`

```java
public class TaxCalculator {
  public String calculate() {
    return "Tax calculated at 9% GST";
  }
}
```

`ShippingCalculator.java`

```java
public class ShippingCalculator {
  public String calculate() {
    return "Shipping calculated at $4.50 flat rate";
  }
}
```

Create `OrderController.java`:

```java
@RestController
public class OrderController {
  private TaxCalculator taxCalculator = new TaxCalculator();
  private ShippingCalculator shippingCalculator = new ShippingCalculator();

  @GetMapping("/tax")
  public String tax() {
    return taxCalculator.calculate();
  }

  @GetMapping("/shipping")
  public String shipping() {
    return shippingCalculator.calculate();
  }
}
```

Test out the endpoints.

Currently we are creating the instances ourselves. Let's use dependency injection instead.

In order to use dependency injection, we need to let Spring Boot know that `TaxCalculator` and `ShippingCalculator` are Spring Beans. We can do this by annotating them with `@Component`. After you do this, you can see these beans in your Spring Boot Dashboard.

Notice these two classes are a good fit for beans — they **do work** (they calculate something) and they hold no per-request data. This is the same rule from the previous lesson: *inject the things that do work, create the things that hold data.*

Let's use field injection for the shipping calculator by adding the `@Autowired` annotation to the `shippingCalculator` field:

```java
// private ShippingCalculator shippingCalculator = new ShippingCalculator();
@Autowired
private ShippingCalculator shippingCalculator;
```

Notice now that without having to instantiate the `ShippingCalculator` class, we can still use the `shippingCalculator` bean.

Now, field injection is not ideal — see the note above for why. Let's use constructor injection instead:

```java
private ShippingCalculator shippingCalculator;

public OrderController(ShippingCalculator shippingCalculator) {
  this.shippingCalculator = shippingCalculator;
}
```

> 📝 **Note:** In older Spring code, you will often see `@Autowired` on constructors. Since Spring 4.3, if a class has only one constructor, Spring automatically uses it for injection — **`@Autowired` is no longer needed on constructors**. This is now the standard practice. You may still see it in legacy codebases, but new code should omit it.

Test it out to make sure it still works.

Behind the scenes, what Spring is doing is this:

```java
// Create a new instance of ShippingCalculator
ShippingCalculator shippingCalculator = new ShippingCalculator();
// Inject the instance into the constructor
OrderController orderController = new OrderController(shippingCalculator);
```

Now, let's see how setter injection works on the `TaxCalculator` bean:

```java
private TaxCalculator taxCalculator;

@Autowired
public void setTaxCalculator(TaxCalculator taxCalculator) {
  this.taxCalculator = taxCalculator;
}
```

> 📝 **Note:** Setter injection was more common in early Spring applications (pre-Spring 3). In modern Spring applications, constructor injection is strongly preferred. Setter injection is considered a **legacy pattern** — you may encounter it in older codebases, but it is rarely written in new production code.
>
> ⚠️ **Important:** Unlike constructor injection, setter injection is **never auto-detected** by Spring — even if there is only one setter. You must explicitly annotate the setter with `@Autowired`, or Spring will never call it, leaving the field `null` and causing a `NullPointerException` when the endpoint is hit.

Then call the `/tax` endpoint to test it out.

### 👨‍💻 Activity **(10 minutes)**

In your `di-demo` project:

1. Add a `DiscountCalculator` class and use **constructor injection** to inject it into the `OrderController`.
2. Add an `AuditLogger` class and use **setter injection** to inject it into the `OrderController`.

Add the corresponding endpoints (`/discount` and `/audit`) to test out the beans.

---

## Part 4: @Bean

`@Component` works when the class is yours — you can open the file and add the annotation. But what about a class you did not write? A class that lives inside a library or inside the JDK itself? You cannot add an annotation to it.

That is what `@Bean` is for. Instead of annotating the class, you write a **method** that builds the object and hands it to Spring. You put that method inside a class annotated with `@Configuration`, which tells Spring: this class contains bean-producing methods, go look inside it.

Let's use `java.util.Random` as our example. It is part of the JDK, so there is no way for us to put `@Component` on it.

```java
package sg.edu.ntu.di_demo;

import java.util.Random;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

  @Bean
  public Random random() {
    return new Random(42);
  }
}
```

Read it from the inside out. The method creates the object with `new`, configures it however you need, and returns it. The `@Bean` annotation tells Spring to call that method at startup and keep the returned object in the container. From that point on it behaves like any other bean — you inject it by type, exactly as you would a `@Component`.

Now inject it into `OrderController` the same way as everything else:

```java
private final Random random;

// add Random random to the existing constructor parameters,
// and inside the constructor: this.random = random;

@GetMapping("/order-number")
public String orderNumber() {
  return "Order #" + random.nextInt(10000);
}
```

Notice this also solves a second problem: **configuration**. `@Component` gives you no place to set anything up. Here you have a whole method body, so you can pass in a seed value, read settings from a properties file, or build something that needs several steps before it is usable.

> 📝 **Is a `@Bean` object a singleton?** Yes — exactly like a `@Component`. Spring calls the method **once** at startup, stores the returned object, and hands that same instance to everyone who asks for it. The method does not run again. The two annotations produce the same result; they differ only in how you tell Spring to build the object.

> 📝 **You will see this pattern a lot.** External clients for calling other APIs, a custom `ObjectMapper` for JSON handling, and almost everything in Spring Security — `PasswordEncoder` and the security filter chain are both declared as `@Bean` methods in a `@Configuration` class. There is no `@Component` option for any of them, because none of those classes are yours to annotate. Recognise the shape now and it will be familiar when we get there.

**The rule:** `@Component` for classes you own. `@Bean` for classes you don't, or when the object needs configuring before it is usable.

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

We continue with the same **`simple-crm`** project from the previous lesson. Do not create a new project.

> **Folder structure reminder — standing rule for `simple-crm`.** Every class goes in a folder matching its layer, inside your base package (`sg.edu.ntu.simple_crm`):
> - `CustomerController` → `controller` folder
> - `Customer` → `model` folder
> - `CustomerService` / `CustomerServiceImpl` → `service` folder (create this folder now)
> - `CustomerRepository` → `repository` folder (create this folder now)
> - `CustomerNotFoundException` → `exceptions` folder
>
> Use **right-click on the class name → Refactor → Move** so the `package` line and all imports update automatically. If the app then fails to start with a `ConflictingBeanDefinitionException`, an old copy of the class is still in the original location — delete it and run `mvn clean`.

> 📝 **If your code looks slightly different from the examples below, that is fine.** In the previous lesson's activity you may have chosen `204 No Content` instead of `200 OK` for delete, or used `ResponseEntity<Object>` to return the exception message. Keep whichever version you built — just apply the same refactoring pattern to it.

### Repository Layer

Since the repository layer is responsible for CRUD operations, we will create a `CustomerRepository` class to handle all the CRUD operations on our `ArrayList`.

Only the repository should have access to the data store. Hence the list should be private and only accessible within the `CustomerRepository` class.

This class also needs to be annotated with `@Repository` to let Spring Boot know that it is a Spring Bean.

> 📝 **`@Component` vs `@Service` vs `@Repository` — What's the difference?**
>
> All three annotations register a class as a Spring Bean. The difference is in **intent and behaviour**:
>
> - `@Component` — The generic stereotype. Use it when the class doesn't clearly fit as a service or repository.
> - `@Service` — A specialization of `@Component`. It carries no extra technical behaviour today, but it communicates clearly that this class contains **business logic**.
> - `@Repository` — A specialization of `@Component` with one important technical addition: Spring automatically translates **persistence-layer exceptions** (e.g. database errors) into Spring's unified `DataAccessException` hierarchy. This makes error handling consistent regardless of whether you're using JDBC, JPA, or any other data access technology. Always use `@Repository` on your data access classes.
>
> In short: use the most specific annotation that fits.

```java
@Repository
public class CustomerRepository {

  private List<Customer> customers = new ArrayList<>();

  // Preload data here now — moved out of the controller
  public CustomerRepository() {
    customers.add(new Customer("Bruce", "Banner"));
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

  // Update (full replace of the customer's data)
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

Note the field is declared as `List<Customer>`, not `ArrayList<Customer>`. Coding to an interface applies to collections too — this keeps the flexibility to swap the underlying implementation later without changing any calling code.

> ⚠️ **Note the change to update — and why the `id` no longer changes.**
>
> Jackson always builds a **new** `Customer` object from the JSON in the request body, and because `Customer` generates its `id` inline, that new object always arrives with a new `id`. That happens in both versions — it is not the thing that changed.
>
> What changed is what we do with it. Last lesson we wrote `customers.set(index, customer)`, which **puts that new object into the list**, replacing the old one. So the new object's `id` became the stored `id`.
>
> The repository does it the other way round. It fetches the **existing** customer out of the list and copies the values onto it, field by field. The new object is only used as a source of values and is then discarded. And notice there is no `setId()` line — there cannot be, because `id` is `final` and Lombok will not generate a setter for a final field. So the stored customer keeps its original `id`.

> 📝 **Production note — returning the internal list.** `getAllCustomers()` returns the repository's actual list, not a copy. That means anything holding that reference can add or remove customers directly, bypassing the repository entirely. In production you would return a copy (`new ArrayList<>(customers)`) or an unmodifiable view (`Collections.unmodifiableList(customers)`) to protect the data store. We leave it as-is here for simplicity, but this is exactly the kind of encapsulation leak that causes hard-to-trace bugs in real systems.

As you can see, the purpose of this layer is just to perform CRUD operations. It contains no business logic.

### Service Layer

Next, we will create a `CustomerService` class to handle all the business logic.

`CustomerService` will need to call our `CustomerRepository` to perform CRUD operations, since updating the data store is the responsibility of the repository layer. We also want to move our helper method `getCustomerIndex()` from `CustomerController` into the service, because finding a customer is business logic.

The service class needs to be annotated with `@Service` to let Spring Boot know that it is a Spring Bean.

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

> 📝 **Our exception handling does not change.** `CustomerNotFoundException` is now thrown from the service instead of the controller, but it is an **unchecked** exception (`extends RuntimeException`), so it travels up through the layers on its own — no `throws` clause needed anywhere. The `try`/`catch` blocks in the controller stay exactly as they are and keep working. Later in the module we will move this handling out of the controller entirely using `@ControllerAdvice`.

### Controller Layer

Finally, we modify `CustomerController` to use the `CustomerService` class. Notice we are using constructor injection — we let Spring manage the `CustomerService` instance for us instead of creating it with `new`.

The controller no longer needs the `ArrayList`, the preloaded data in its constructor, or the `getCustomerIndex()` helper. Delete all three — they now live in the repository and service.

> 📝 **Why not use `new CustomerService()` here?** Service classes are designed to provide functionality — not to hold data. We only ever need one instance in the entire application. If every class that needed `CustomerService` called `new CustomerService()`, we'd end up with multiple unnecessary instances. By using constructor injection, Spring creates exactly one instance and reuses it everywhere — the **Singleton pattern**, which is the default behaviour for all Spring beans.

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
  @GetMapping("/{id}")
  public ResponseEntity<Customer> getCustomer(@PathVariable String id) {
    try {
      Customer foundCustomer = customerService.getCustomer(id);
      return new ResponseEntity<>(foundCustomer, HttpStatus.OK);
    } catch (CustomerNotFoundException e) {
      return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
  }

  // UPDATE
  @PutMapping("/{id}")
  public ResponseEntity<Customer> updateCustomer(@PathVariable String id, @RequestBody Customer customer) {
    try {
      Customer updatedCustomer = customerService.updateCustomer(id, customer);
      return new ResponseEntity<>(updatedCustomer, HttpStatus.OK);
    } catch (CustomerNotFoundException e) {
      return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
  }

  // DELETE
  @DeleteMapping("/{id}")
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

Test all the endpoints again after refactoring. They should work as before — and check that the `id` now stays the same after a `PUT`.

### Coding to an Interface

**Coding to an interface** means writing our code to be dependent on an interface instead of a concrete class. This promotes loose coupling and makes our code more flexible and easy to change.

For our service layer, it is good practice to code to an interface, because we may want to change the implementation in the future.

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

Next, our `CustomerServiceImpl` class should implement the `CustomerService` interface. **Remember to add the `implements CustomerService` clause** — renaming the class alone does not make it implement the new interface. If this step is missed, `CustomerServiceImpl` will not be seen as a candidate for `CustomerService` injection, and the behaviour you get later will be confusing rather than a clear error.

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

Note that we do not have to change anything in `CustomerController.java`, as it is already using the `CustomerService` type:

```java
private final CustomerService customerService;

public CustomerController(CustomerService customerService) {
  this.customerService = customerService;
}
```

When Spring Boot encounters a `CustomerService` type dependency in the `CustomerController`, it looks for a bean that implements the `CustomerService` interface. Since we annotated `CustomerServiceImpl` with `@Service`, Spring Boot creates that bean and injects it into the controller.

Test the endpoints again to make sure they still work.

### @Primary and @Qualifier

Now let's say we want a second implementation of our service layer that logs method calls. Create `CustomerServiceWithLoggingImpl.java` in the `service` folder.

It must implement all five interface methods, but we only need logging on a couple of them to see it working:

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
  public List<Customer> getAllCustomers() {
    logger.info("CustomerServiceWithLoggingImpl.getAllCustomers() called");
    return customerRepository.getAllCustomers();
  }

  @Override
  public Customer getCustomer(String id) {
    return customerRepository.getCustomer(getCustomerIndex(id));
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

Now when you try to run the application, you will get an error:

```
Parameter 0 of constructor in CustomerController required a single bean, but 2 were found
```

This is because Spring Boot does not know which bean to inject — we now have 2 beans implementing the `CustomerService` interface. There are two ways to resolve this.

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

Try the `@Qualifier` version and hit `GET /customers` — you should see the log line appear in your console, confirming the logging implementation is the one being used.

By coding to an interface, we can swap implementations without touching any of the controller's endpoint code.

> 📝 **Note:** `@Primary` and `@Qualifier` can be used together. If both are present, `@Qualifier` at the injection point wins over `@Primary` on the bean — it is a more specific instruction at the point of use.

> ⚠️ **Before you finish — clean up.** You are ending this lesson with two `CustomerService` implementations. If you leave both in place with **no** `@Primary` and **no** `@Qualifier`, `simple-crm` will not start next lesson. Do one of these before you close:
> - Keep `@Primary` on `CustomerServiceImpl`, **or**
> - Delete `CustomerServiceWithLoggingImpl` now that you have seen how it works
>
> Remember `simple-crm` is the project we carry forward for the rest of the module — it needs to be in a working state at the end of every lesson.

---

> 📖 **Further Reading: Bean Scope**
>
> The scope of a bean defines its lifecycle — how long Spring keeps the bean around and when to create a new instance. By default, all Spring beans are **singleton** scoped, meaning Spring creates only one instance and reuses it for the entire application lifetime. This can be changed as needed for your use case.
>
> - [Java Singleton Pattern — Baeldung](https://www.baeldung.com/java-singleton)
> - [Spring Bean Scopes — Baeldung](https://www.baeldung.com/spring-bean-scopes)

---

END