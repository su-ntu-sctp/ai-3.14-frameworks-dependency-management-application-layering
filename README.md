# [3.14] Frameworks, Dependency Management, and Application Layering

## Lesson Overview

![Frameworks, Dependency Management, and Application Layering](./assets/images/infographic-3.14-service-repository-pattern.png)

## Dependencies

- [Self Studies](./studies.md) / [Lesson](./lesson.md) / [Assignment](./assignment.md) / [Slide Deck](./slides.md)

## Lesson Objectives

By the end of this lesson, students will be able to:

* **Differentiate** frameworks and libraries and explain how Inversion of Control affects application flow
* **Declare** and manage beans using `@Component` and `@Bean`, and apply constructor, setter, and field injection
* **Refactor** a controller-heavy application into layered Controller, Service, and Repository components
* **Implement** interface-based design and resolve multiple implementations using `@Primary` and `@Qualifier`

## Lesson Plan

| Duration | What | How or Why |
|---|---|---|
| 10 min | Warm-up | Recap `@Component`, `@Autowired`, and DI from Lesson 3.11 — this lesson deepens those concepts significantly |
| 10 min | Part 1: Frameworks vs Libraries | Conceptual foundation — clarify the distinction and explain why IoC matters in Spring Boot |
| 15 min | Part 2: Inversion of Control and Dependency Injection | Explain IoC as a principle; show how DI is its practical implementation in Spring |
| 30 min | Part 3: Beans and the 3 injection types | Code-along in `di-demo` — build `MathTeacher` and `ScienceTeacher` beans; demonstrate field, constructor, and setter injection side by side |
| 10 min | Activity 1 — Add CodingTeacher, AlgorithmsTeacher, DatabaseTeacher | Students apply all 3 injection types independently |
| 15 min | Part 4: `@Bean` and `@Configuration` | Code-along — create `EmailService` and `EmailConfig`; explain when to use `@Bean` vs `@Component` |
| 10 min | Break | — |
| 15 min | Part 5: SRP + Service and Repository pattern | Introduce Single Responsibility Principle; explain the 3-layer architecture and why thin controllers matter |
| 10 min | Part 6: Repository layer — `CustomerRepository` | Code-along — extract CRUD operations from the controller into a dedicated `@Repository` class |
| 10 min | Part 6: Service layer — `CustomerService` | Code-along — create `@Service` class; move `getCustomerIndex()` helper and business logic out of controller |
| 10 min | Part 6: Controller layer — refactor `CustomerController` | Code-along — slim down controller to HTTP handling only; wire up the service |
| 10 min | Activity 2 — Switch to constructor injection | Students replace `new` instantiation in controller and service with constructor DI |
| 15 min | Part 6: Interface-based design + `@Primary` / `@Qualifier` | Code-along — extract `CustomerService` interface; create `CustomerServiceWithLoggingImpl`; resolve the ambiguity error |
| 10 min | Wrap-up | Recap the 3-layer pattern, DI types, and interface-based design; preview next lesson |
| **180 min** | **Total** | |