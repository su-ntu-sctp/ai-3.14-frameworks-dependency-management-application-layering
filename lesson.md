
# Lesson: Frameworks, Dependency Management, and Application Layering  

## Lesson Overview  
This lesson introduces the key concepts behind Spring Boot’s architecture — Inversion of Control, Dependency Injection, and the Service–Repository design pattern. Students learn how Spring manages object creation through beans, apply different injection types, and refactor a simple CRM into a layered, maintainable structure using interfaces and bean configuration.

## Lesson Objectives  
- Differentiate frameworks and libraries, and explain how Inversion of Control affects application flow in Spring Boot.  
- Declare and manage beans using `@Component` and `@Bean`, and inject dependencies via constructor, setter, and field injection.  
- Refactor a controller-heavy application into layered Controller, Service, and Repository components.  
- Implement interface-based design and manage multiple service implementations with `@Primary` and `@Qualifier`.  
- Describe how bean scopes influence object lifecycle and application performance.

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

In IoC, the control of creating and managing objects is inverted and given to the framework. Instead of creating instances of classes, the developer will declare the dependencies of the class and let the framework create the instances of the classes and inject them into the class. This is known as **Dependency Injection**. In this way, the components of your application are loosely coupled, which promotes modularity, resuability and testability.

<img src="https://devopedia.org/images/article/30/4020.1536743448.gif">

> Source: https://devopedia.org/dependency-injection

---

## Part 3: More on Beans and Dependency Injection

In the entry point of every Spring Boot application i.e. our `main` method, you will see the `@SpringBootApplication` annotation. This annotation is a combination of 3 annotations:

1. `@Configuration` - Indicates that the class contains `@Bean` annotations, pick them up and add them into the spring container.
1. `@ComponentScan` - To scan for all `@Component` annotations classes located in the same package (or explicitly specified) and add them to the spring container.
1. `@EnableAutoConfiguration` - Looks for auto-configuration bean (java class) and add them into the spring container.

Therefore, `@SpringBootApplication` annotation is a Spring Boot feature to quickly boostrap the default/commonly used annotations into one.

Classes that are annotated with `@Component` are known as **Spring Beans**. Spring Beans are managed by the Spring IoC container, also known as the Spring Context or Application Context. The Spring IoC container is responsible for instantiating, configuring, and assembling the Spring Beans.

<img src="https://gustavopeiretti.com/spring-injection-dependencies/spring-injection-en-2.png" width=450>

> Source: https://gustavopeiretti.com/spring-injection-dependencies/

`@Autowired` is used to inject the dependencies into the class. There are multiple ways to inject dependencies into a class.

| Type        | Description                                                 | Remarks                                              |
| ----------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| Constructor | Dependencies are injected through the constructor.          | Preferred method of dependency injection.            |
| Setter      | Dependencies are injected through setter methods.           | Useful when you want to change the dependency later. |
| Field       | Dependencies are injected directly into the class property. | Not recommended.                                     |

Let's create a simple Spring Boot application `di-demo` to see how all these works. Add the Spring Web and Spring Boot devtools dependencies in `pom.xml`:

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
</dependencies>
```

Let's create `MathTeacher.java` and `ScienceTeacher.java` classes:

`MathTeacher.java`

```java
public class MathTeacher  {
  public String teach() {
    return "Teaching Math";
  }
}
```

`ScienceTeacher.java`

```java
public class ScienceTeacher  {
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

In order to use dependency injection, we need to let Spring Boot know that `MathTeacher` and `ScienceTeacher` are Spring Beans. Remember that beans are just classes that are managed by Spring. We can do this by annotating them with `@Component`. After you do this, you can see these beans in your Spring Boot Dashboard.

Let's use field injection for the science teacher by adding the `Autowired` annotation to the `scienceTeacher` field:

```java
// private ScienceTeacher scienceTeacher = new ScienceTeacher();
@Autowired
private ScienceTeacher scienceTeacher;
```

Notice now that without having to instantiate the `ScienceTeacher` class, we can still use the `scienceTeacher` bean.

Now, field injection is not ideal because it makes it difficult to test the class (testing is covered in a later lesson). Let's use constructor injection instead:

```java
// @Autowired
private ScienceTeacher scienceTeacher;

@Autowired
public TeacherController(ScienceTeacher scienceTeacher) {
  this.scienceTeacher = scienceTeacher;
}
```

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

Then call the `/math-teacher` endpoint to test it out.

Behind the scenes, Spring is doing this:

```java
// Create a new instance of MathTeacher
MathTeacher mathTeacher = new MathTeacher();
// Inject the instance into the setter
teacherController.setMathTeacher(mathTeacher);
```

### 👨‍💻 Activity

Add a `CodingTeacher` and use constructor injection to inject into the `TeacherController`.

Add an `AlgorithmsTeacher` and use setter injection to inject into the `TeacherController`.

Add a `DatabaseTeacher` and use field injection to inject into the `TeacherController`.

Add the corresponding endpoints to test out the beans.

---

## Part 4: @Bean

Other than using `@Component` to configure instances, `@Bean` can be used to configure instances that contains default configuration.

Usage examples:

- You want to autowire an email service instance with pre-defined SMTP settings. SMTP Settings such as your email address and email server details. These details are specific to the organization.
- You want to use an external library such as DocuSign for digital signing purposes.

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

And now we want to use this class as a bean. We can do this by annotating a method with `@Bean` in a `@Configuration` class. The `@Configuration` annotation indicates that the class contains `@Bean` annotations which Spring will pick up and add them into the spring container.

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

You can also see the beans in Spring Boot Dashboard now.

Test calling the `/science-teacher` endpoint.

In this example, if you noticed, actually we could have annotated the `EmailService` class with `@Component` instead of creating a `@Configuration` class. And that would allow us to inject the `EmailService` bean directly into the `TeacherController` without having to create a `@Bean` method.

However, sometimes, we may be using some other library that we do not have access to the source code of the class that we want to use as a bean. In such cases, we can use `@Bean` to create a bean of that class.

This is just a basic example to demonstrate the use of `@Bean`. In real-world scenarios, beans often have more complex configurations and dependencies.

---

## Part 5: Service and Repository Pattern

Let's go back to our `simple-crm` application.

So far, all of our code is in the controller layer. This is not ideal because we are mixing our business logic with our controller logic. This makes our code difficult to maintain and test.

### Single Responsibility Principle

In programming, SOLID is a mnemonic acronym for five design principles intended to make software designs more understandable, flexible and maintainable. The **Single Responsibility Principle (SRP)** states that every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class. All its services should be narrowly aligned with that responsibility.

<img src="https://miro.medium.com/v2/resize:fit:1000/format:webp/1*PxIES4LBAMi8K4RudiP-tw.jpeg">

> Source: https://medium.com/@anisha.nicole/single-responsibility-principle-cabba52aa467

Read more about SOLID [here](https://en.wikipedia.org/wiki/SOLID).

As you might have noticed, our `CustomerController` class is doing more than one thing. It is managing the HTTP requests, handling some business logic, as well as performing CRUD operations on our `ArrayList`.

### Service and Repository

The Service and Repository pattern is a common design pattern used in Java applications.

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*neBcAZJyLGpE7KHc3sH8bw.png" width=400>

> Source: https://tom-collings.medium.com/controller-service-repository-16e29a4684e5

Instead of putting all our code in the controller layer, we can separate our code into 3 different layers:

| Layer      | Purpose                             |
| ---------- | ----------------------------------- |
| Controller | Handles HTTP requests and responses |
| Service    | Handles business logic              |
| Repository | Handles CRUD operations             |

The controller should only handle HTTP requests and responses. The repository should only handle CRUD operations. The service should handle all the business logic e.g. validation, data manipulation, etc.

Hence, it is often suggested to have **thin controllers/repositories and fat services**. This helps to keep the controllers and repositories simple and easy to maintain. More importantly, it centralizes all the business logic in the service layer.

This can help to achieve the goal of creating an architecture that supports easy maintenance, testing, and scalability, while keeping your codebase organized and readable.

---

## Part 6: Refactoring Our `simple-crm`

### Repository Layer

Since the repository layer is responsible for CRUD operations, we will create a `CustomerRepository` class to handle all the CRUD operations on our `ArrayList`.

Only the repository should have access to the data store. Hence the `ArrayList` should be private and only accessible within the `CustomerRepository` class.

This class also needs to be annotated with `@Repository` to let Spring Boot know that it is a Spring Bean. The `@Repository` annotation is a specialization of the `@Component` annotation with additional functionality for persistence layer exceptions.

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
  public ArrayList<Customer> getAllCustomers() {
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

As you can see, the purpose of this layer is just to perform CRUD operations on our `ArrayList`. It does not contain any business logic.

### Service Layer

Next, we will create a `CustomerService` class to handle all the business logic (i.e. decisions, processing or computations regarding our data).

`CustomerService` will need to call our `CustomerRepository` to perform CRUD operations on our `ArrayList` since updating the data store is the responsibility of the repository layer. That also means that `CustomerService` needs an instance of `CustomerRepository` to perform CRUD operations.

We also want to move our helper function `getCustomerIndex()` from `CustomerController` to `CustomerService` because it is part of the business logic.

The service class needs to be annotated with `@Service` to let Spring Boot know that it is a Spring Bean. The `@Service` annotation is a specialization of the `@Component` annotation with additional functionality for service layer classes.

```java
@Service
public class CustomerService {

  private CustomerRepository customerRepository = new CustomerRepository();

  public Customer createCustomer(Customer customer) {
    return customerRepository.createCustomer(customer);
  }

  public Customer getCustomer(String id) {
    return customerRepository.getCustomer(getCustomerIndex(id));
  }

  public ArrayList<Customer> getAllCustomers() {
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

Finally, we will modify our `CustomerController` to use the `CustomerService` class.

```java
@RestController
@RequestMapping("/customers")
public class CustomerController {

  private CustomerService customerService = new CustomerService();

  // CREATE
  @PostMapping("")
  public ResponseEntity<Customer> createCustomer(@RequestBody Customer customer) {
    Customer newCustomer = customerService.createCustomer(customer);
    return new ResponseEntity<>(newCustomer, HttpStatus.CREATED);
  }

  // READ (GET ALL)
  @GetMapping("")
  public ResponseEntity<ArrayList<Customer>> getAllCustomers() {
    ArrayList<Customer> allCustomers = customerService.getAllCustomers();
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

Test the endpoints again after refactoring the code. They should still work as before.

### Using Dependency Injection

In our current code, we have been creating new instances of our service and repository classes.

The problem with this approach is that we are tightly coupling our code to the implementation of the service and repository classes. We are also creating unnecessary instances of our service and repository classes when we only need one instance.

In Java, we have data-holding classes whose purpose is to store data e.g. POJOs. At the same time, we also have service classes that are designed to provide specific functionalities. Our `CustomerService` class, for example, is designed to do that.

So, how many instances of `CustomerService` do we need in one application? Actually, we only need one. Creating more instances is a waste of resources because we only need one instance to provide the functionalities that we need.

This is why we should not create instances of our service and repository classes. Instead, we should let Spring Boot create the instances for us and inject them into our controller.

### 👨‍💻 Activity

Modify the code to use dependency injection (constructor) instead.

### Coding to an Interface

Recall that an interface is a contract that specifies the methods that a class must implement e.g.

```java
public interface Food {
  // This method must be implemented by any class that implements the Food interface
  String getFlavour();
}
```

**Coding to an interface** means writing our code to be dependent on an interface instead of a concrete class. This promotes loose coupling and makes our code more flexible.

For our service layer, it is a good practice to code to an interface. This is because we may want to change the implementation of our service layer in the future.

Let's rename our `CustomerService.java` to `CustomerServiceImpl.java` and create a new interface called `CustomerService.java` with the all the method signatures:

```java
public interface CustomerService {
  Customer createCustomer(Customer customer);
  Customer getCustomer(String id);
  ArrayList<Customer> getAllCustomers();
  Customer updateCustomer(String id, Customer customer);
  void deleteCustomer(String id);
}
```

Next, our `CustomerServiceImpl` class should implement the `CustomerService` interface:

```java
@Service
public class CustomerServiceImpl implements CustomerService {
  // ...
}
```

Note that we do not have to change anything in `CustomerController.java` as it is already using the `CustomerService` bean.

```java
private CustomerService customerService;

// Constructor Injection
public CustomerController(CustomerService customerService) {
  this.customerService = customerService;
}
```

When Spring Boot encounters a `CustomerService` type dependency in the `CustomerController`, it will look for a bean that implements the `CustomerService` interface. Since we have annotated our `CustomerServiceImpl` class with `@Service`, Spring Boot will create a bean of type `CustomerServiceImpl` and inject it into the `CustomerController`.

Test the endpoints again to make sure they still work.

Why is this useful? Because now we can easily change the implementation of our service layer without having to change the code in our controller layer.

Let's say we want to have an implementation of our service layer that logs all the method calls.

Create a new `CustomerService` implementation called `CustomerServiceWithLoggingImpl.java`. Copy the code from `CustomerServiceImpl.java` and paste it into `CustomerServiceWithLoggingImpl.java`.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class CustomerServiceWithLoggingImpl implements CustomerService {

  private final Logger logger = LoggerFactory.getLogger(CustomerServiceWithLoggingImpl.class);
  // ...

  @Override
  public ArrayList<Customer> getAllCustomers() {
    logger.info("CustomerServiceWithLoggingImpl.getAllCustomers() called");
    return customerRepository.getAllCustomers();
  }
}
```

Now when you try to run the application, you will get an error like this:

```
Parameter 0 of constructor in com.example.simplecrm.controller.CustomerController required a single bean, but 2 were found:
```

Why did this happen?

This is because Spring Boot does not know which bean to inject into the `CustomerController` since we have 2 beans that implement the `CustomerService` interface.

How do we tell Spring Boot which implementation to use?

The first way is to annotate the `CustomerServiceImpl` class with `@Primary`:

```java
@Primary
@Service
public class CustomerServiceImpl implements CustomerService {
  // ...
}
```

The second way is to specify the name of the bean in the `@Qualifier` annotation when we inject the dependency into the `CustomerController`.

The name of the bean is the name of the class with the first letter in lowercase.

```java
private CustomerService customerService;

// Constructor Injection
public CustomerController(@Qualifier("customerServiceWithLoggingImpl") CustomerService customerService) {
  this.customerService = customerService;
}
```

Thus, we can see that by coding to an interface, we can easily change the implementation of our service layer without having to change the code in our controller layer.

---

## Part 7: Bean Scope

The scope of a bean defines the lifecycle of a bean. It tells Spring Boot how long to keep the bean around and when to create a new instance of the bean.

By default the scope of a bean is singleton but this can be changed as needed. This means that Spring Boot will only create one instance of the bean and reuse it whenever the bean is requested. The singleton is not a Spring Boot concept but a design pattern - you can read more about the Java Singleton design pattern [here](https://www.baeldung.com/java-singleton).

You can read more about bean scope [here](https://www.baeldung.com/spring-bean-scopes).

---

END