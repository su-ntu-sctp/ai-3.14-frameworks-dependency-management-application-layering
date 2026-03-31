# Self Studies: Frameworks, Dependency Management, and Application Layering

## Overview

This lesson takes a deeper dive into how Spring Boot manages objects and dependencies, and introduces the layered architecture pattern used in professional Java applications. The self-study materials below will help you arrive with a solid understanding of Dependency Injection and why application layering matters — so you can focus on the refactoring code-along during the lesson.

**Estimated Prep Time:** 60–80 minutes

---

## Task 1: Spring Boot Dependency Injection

This video covers how Spring Boot manages beans and the different ways to inject dependencies — field, setter, and constructor injection. This maps directly to Parts 2, 3, and 4 of the lesson.

**Watch:** Spring Boot Dependency Injection Tutorial
🎬 https://www.youtube.com/watch?v=TBlB2_4_Sqo

**Then read:** Lesson 3.14 — Parts 1 to 4

**Guiding Questions:**
- What is the difference between a framework and a library?
- What does Inversion of Control mean, and how does Spring Boot implement it?
- What are the three types of dependency injection, and which is preferred and why?
- When would you use `@Bean` instead of `@Component`?

---

## Task 2: Spring Boot Service and Repository Layer

This video walks through the 3-layer architecture — Controller, Service, and Repository — and shows how to refactor a Spring Boot application to follow it. This maps directly to Parts 5 and 6 of the lesson.

**Watch:** Spring Boot Service and Repository Layer Tutorial
🎬 https://www.youtube.com/watch?v=Kzrm-BdLckE

**Then read:** Lesson 3.14 — Parts 5 and 6

**Guiding Questions:**
- What is the Single Responsibility Principle and how does it apply to our CRM application?
- What is the responsibility of each layer — Controller, Service, and Repository?
- What does "thin controllers, fat services" mean in practice?
- Why is coding to an interface more flexible than coding to a concrete class?

---

## Active Engagement Strategies

- As you watch the first video, pause after each injection type (field, setter, constructor) and try to write it from memory before moving on
- After watching the second video, sketch a simple diagram showing how `CustomerController`, `CustomerService`, and `CustomerRepository` connect to each other — include the direction of dependency
- Think about the `simple-crm` project from Lesson 3.13 — identify which parts of the `CustomerController` belong in the service layer and which belong in the repository layer

---

## Additional Reading Material

- [Spring Bean Scopes — Baeldung](https://www.baeldung.com/spring-bean-scopes)
- [SOLID Principles — Wikipedia](https://en.wikipedia.org/wiki/SOLID)
- [@Primary and @Qualifier in Spring — Baeldung](https://www.baeldung.com/spring-qualifier-annotation)
- [Controller-Service-Repository Pattern — Medium](https://tom-collings.medium.com/controller-service-repository-16e29a4684e5)