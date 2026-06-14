 How have you used Spring Cloud Config Server in your projects?
  Explain the internal working of ConcurrentHashMap .
---
13. HashMap vs ConcurrentHashMap vs Hashtable
14. JVM Memory Model: Heap, Stack, GC
15. ExecutorService, Callable, and Future
16. How Spring Boot Dependency Injection Works
17. REST vs Kafka/RabbitMQ in Microservices
18. Transaction Management in Spring Boot
19. SQL vs NoSQL
20. Designing Scalable REST APIs
21. How Transactional Fails Silently
22. Thread Safety: synchronized vs volatile vs atomic
23. How HikariCP Connection Pooling Works
24. Circuit Breaker Pattern with Resilience4J


--- 

1. Your API is working fine, but users report slow responses. Where would you start?

2. Why might a "@Transactional" method fail to roll back changes?

3. Your application works locally but fails in production. What's your first step?

4. Kafka consumers are running, but lag keeps increasing. Why?

5. A microservice goes down. How do you prevent other services from failing?

6. Database connections suddenly get exhausted. What could be causing it?

7. Retry logic creates duplicate transactions. How would you fix it?

8. A cache improves performance but starts serving stale data.

9. Health checks are green, but customers still report issues.

10. One slow API starts affecting multiple services.

11. Autoscaling adds more pods, but performance doesn't improve.

12. Logs show no errors, but requests are timing out. 


13. Thread pools are exhausted while CPU usage remains low.

14. A deployment increases latency without increasing traffic.

15. Circuit breakers are enabled, but cascading failures still happen.

16. Message ordering breaks between services.

17. One database query starts impacting the entire application.

18. The same Kafka event is processed twice.

19. Tracing a single request across services becomes difficult.

20. When would you choose a monolith instead of microservices?


I've been asked ALL of these in the last 3 weeks:


1️⃣ "What happens when a Spring Boot application starts?"
→ Most say: "It scans for beans and starts Tomcat."
→ Senior answer: Walk through SpringApplication.run() → Environment setup → ApplicationContext creation → Auto-configuration via spring.factories / META-INF → Bean lifecycle → Embedded server start
↳ Follow-up: "How does auto-configuration decide which beans to create?"


2️⃣ "@Transactional — what actually happens under the hood?"
→ Spring creates a PROXY around your class. The proxy intercepts the method call, opens a transaction, and commits/rolls back.
↳ Killer follow-up: "What happens if you call a @Transactional method from WITHIN the same class?"
→ Answer: It BYPASSES the proxy. The transaction doesn't apply. This catches 90% of developers off guard.


3️⃣ "N+1 query problem — explain, detect, fix."
→ 1 query to fetch parents + N queries to fetch each child individually
→ Detection: EXPLAIN ANALYZE or Hibernate SQL logging
→ Fix: JOIN FETCH, @EntityGraph, or batch fetching
↳ Follow-up: "Show me what the Hibernate log output looks like for N+1."


4️⃣ "Lazy vs Eager loading — trade-offs?"
→ Lazy: loads on access. Risk: LazyInitializationException outside session
→ Eager: loads immediately. Risk: loading the entire database graph
↳ Follow-up: "How do you handle lazy loading in a REST API response?"


5️⃣ "Monolith to microservices — how would you actually do it?"
→ Strangler Fig Pattern. Not a rewrite.
→ Identify bounded contexts → Extract one service at a time → Dual-write during migration → Cut over
↳ Follow-up: "What's the biggest risk during migration?" → Data consistency across the boundary


6️⃣ "Why doesn't @Transactional work across microservices?"
→ Because ACID transactions assume a single database. Distributed systems need eventual consistency.
→ Solution: Saga Pattern — either Choreography (event-driven) or Orchestration (central coordinator)
↳ Follow-up: "Choreography vs Orchestration — when to use which?"


7️⃣ "Circuit Breaker — when and why?"
→ When a downstream service is failing, you stop calling it to prevent cascade failure
→ States: Closed → Open → Half-Open
→ Tool: Resilience4j
↳ Follow-up: "How is a circuit breaker different from a retry with exponential backoff?"


These aren't trick questions. But the DEPTH of your answer tells the interviewer if you're a 3 YoE developer with 6 years on your resume — or a genuine senior engineer.


---
1. Spring Boot auto-config, configuration profiles, Actuator/health
 2. REST design: validation, versioning, idempotency; SAGA/outbox patterns
 3. Kafka (ordering, retries, DLQs) and RabbitMQ (work queues) in payments flows
 4. Observability: metrics/tracing; dealing with 300B+ metrics/day style scale
 5. Index design & trade-offs; deduping rows; window functions vs GROUP BY
 6. Transaction isolation, pagination strategies; read-vs-write models for feeds

 "Explain how HashMap works internally. What happens during a collision?"


 1. How does Spring Boot auto-configuration work internally?


2. What exactly happens when a Spring Boot application starts?


3. Difference between @Component, @Service, @Repository, and @Controller?


4. Why is constructor injection preferred over field injection?


5. What is the bean lifecycle in Spring?


6. Difference between BeanFactory and ApplicationContext?


7. How does @Transactional work internally?


8. What is propagation in transactions?


9. Difference between Lazy Loading and Eager Loading?


10. What is the N+1 problem in Hibernate? How did you solve it?


11. Difference between CrudRepository and JpaRepository?


12. How does Spring Data JPA create queries automatically?


13. Difference between @RestController and @Controller?


14. How does @RequestBody work internally?


15. Difference between @PathVariable and @RequestParam?


16. How do you handle global exception handling using @ControllerAdvice?


17. Difference between Filters and Interceptors?


18. How does Spring Security filter chain work?


19. How JWT authentication works internally?


20. Difference between authentication and authorization?


21. How do microservices communicate with each other in your project?


22. RestTemplate vs WebClient vs Feign Client?


23. How did you implement Circuit Breaker in microservices?


24. How do you handle distributed transactions in microservices?


25. What problems did Kafka solve in your project?


26. How do you avoid duplicate Kafka message processing?


27. What happens if Kafka consumer crashes while processing data?


28. How do you externalize configuration in Spring Boot?


29. How do you troubleshoot slow APIs in production?


30. What production issue did you face in Spring Boot and how did you fix it?


Core Java

• Explain JVM Memory Management (Heap, Stack, Metaspace, GC)

• What is a Memory Leak? How do you identify it in Production?

• HashMap vs ConcurrentHashMap

• How does ConcurrentHashMap handle concurrent updates?

• What happens if both "try" and "finally" have a return statement?

• Can we use "finally" without "catch"?

• Explain Polymorphism with Parent and Child class examples.

• Static Method Hiding vs Method Overriding

• Which Design Patterns have you used in your project?

Java 8

• Features of Java 8 used in your project

• Stream API

• Intermediate vs Terminal Operations

• map() vs flatMap()

• Lambda Expressions

• Optional Class and its use cases

Spring Boot

• Difference between @Component, @Service and @Repository

• If we interchange these annotations, will the application still work?

• Global Exception Handling using @ControllerAdvice

• Filters vs Interceptors

• Parent POM vs Dependency POM

Microservices

• How do Microservices communicate with each other?

• Synchronous vs Asynchronous Communication

• What is Saga Pattern?

• How does rollback happen in Saga?

• Circuit Breaker Pattern and its advantages

Security

• How does JWT Authentication work?

• From where does the JWT Token come?

• How is JWT Token validated?

• JWT Signing Algorithm and Secret Key

• JWT Validation at API Gateway or Service Level?

• Keycloak Integration in Microservices

Database & Performance

• API is slow. How will you troubleshoot it?

• How do you identify bottlenecks in a service?

• Why can repo.findAll() become slow?

• What indexing strategies help improve performance?

Docker & Kubernetes

• Dockerization steps for a Spring Boot application

• JAR → Docker Image → Container flow

• Kubernetes basics

• Embedded Tomcat vs JBoss

• How to deploy Spring Boot applications on Kubernetes?

Coding Questions

• Move all zeroes to the right side of an array in O(n)

• Java Inheritance Output-Based Questions

• String and Collection-based coding scenarios

 Q2: Core Java Topics
 - == vs equals()
 - How HashMap works internally
 - Collision handling (LinkedList vs Tree in Java 8+)
 - Load factor reasoning (0.75)
 - Java Memory Model
 - Thread safety & concurrency

 Q2: Spring Boot Deep Dive
 - @Component vs @Service vs @Repository
 - Bean scopes and lifecycle
 - Startup process of Spring Boot
 - Internals of @Transactional and pitfalls

  Spring Boot annotations for REST APIs — @RestController, @RequestMapping, @PathVariable, @RequestBody and more

-> Exception handling with @ControllerAdvice — how to return clean error responses instead of raw stack traces


 𝗝𝗩𝗠 𝗶𝗻𝘁𝗲𝗿𝗻𝗮𝗹𝘀 & 𝗰𝗼𝗻𝗰𝘂𝗿𝗿𝗲𝗻𝗰𝘆 𝗮𝗿𝗲 𝗯𝗮𝗰𝗸 𝗶𝗻 𝗳𝗼𝗰𝘂𝘀.
Garbage collection, thread safety, Virtual Threads (Java 21) — companies want developers who understand what's happening under the hood, especially when things break in production.

𝟒. 𝗦𝗲𝗰𝘂𝗿𝗶𝘁𝘆 𝗶𝘀 𝗻𝗼𝘄 𝗽𝗮𝗿𝘁 𝗼𝗳 𝘁𝗵𝗲 𝗷𝗼𝗯 𝗱𝗲𝘀𝗰𝗿𝗶𝗽𝘁𝗶𝗼𝗻.
Spring Security, RBAC, JWT, Zero-Trust architecture — especially in fintech, they don't want someone who'll "figure it out later." Security is baked in from Day 1 now.