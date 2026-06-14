# How Spring Boot Dependency Injection Works

## What is Dependency Injection?

Instead of a class creating its own dependencies with `new`, something external **injects** them in. The class just declares what it needs.

```java
// WITHOUT DI — class owns its dependency
public class OrderService {
    private PaymentService paymentService = new PaymentService(); // tightly coupled
}

// WITH DI — dependency is injected externally
public class OrderService {
    private final PaymentService paymentService;  // just declares the need

    public OrderService(PaymentService paymentService) {  // Spring injects it
        this.paymentService = paymentService;
    }
}
```

**Why it matters:**
- Loose coupling — `OrderService` doesn't care how `PaymentService` is built
- Easy to test — inject a mock in tests
- Spring manages lifecycle — singleton, prototype, etc.

---

## The IoC Container — The Engine Behind DI

**IoC = Inversion of Control.** Instead of your code controlling object creation, you *invert* that control to Spring.

The **ApplicationContext** is Spring's IoC container. It:
1. Scans your code for beans
2. Creates and wires them together
3. Manages their lifecycle

```
Your Code
    │  "I need a PaymentService"
    ▼
ApplicationContext (IoC Container)
    │  creates + manages all beans
    │  resolves dependencies
    ▼
Injects PaymentService into OrderService
```

---

## Step 1 — Bean Registration (How Spring finds your classes)

Spring needs to know which classes to manage. Three ways:

### A. Component Scanning (`@Component` family)

```java
@Component          // generic bean
@Service            // business logic layer (semantic alias of @Component)
@Repository         // data access layer (+ exception translation)
@Controller         // web layer
@RestController     // web layer + @ResponseBody
```

```java
@Service
public class PaymentService {
    public void processPayment() { ... }
}
```

Spring scans all packages under `@SpringBootApplication` and registers every class annotated with these.

```java
@SpringBootApplication   // includes @ComponentScan of the current package + sub-packages
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

### B. `@Bean` inside `@Configuration`

For classes you don't own (third-party libs) or need custom construction:

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();   // Spring manages this instance
    }

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper;
    }
}
```

### C. Auto-configuration (Spring Boot specific)

Spring Boot's `@EnableAutoConfiguration` scans `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` and registers beans conditionally.

```java
// Spring Boot does this automatically if you have DataSource on classpath
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceAutoConfiguration {
    @Bean
    public DataSource dataSource() { ... }
}
```

You get a `DataSource` bean without writing any config — as long as `spring.datasource.*` properties exist.

---

## Step 2 — Dependency Resolution (How Spring wires beans)

Once beans are registered, Spring resolves dependencies by **type** (and optionally by name).

### Constructor Injection (Recommended)

```java
@Service
public class OrderService {

    private final PaymentService paymentService;
    private final NotificationService notificationService;

    // Spring sees one constructor → injects automatically (no @Autowired needed in Java 8+)
    public OrderService(PaymentService paymentService,
                        NotificationService notificationService) {
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }
}
```

**Why constructor injection is best:**
- Dependencies are `final` — truly immutable
- Fails fast at startup if a dependency is missing (not at runtime)
- No reflection needed — works without Spring in unit tests

### Field Injection (`@Autowired` on field)

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;   // Spring injects via reflection
}
```

**Works but avoid it:**
- Can't make field `final`
- Hidden dependencies — not visible in constructor
- Hard to test without Spring context

### Setter Injection (`@Autowired` on setter)

```java
@Service
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Use only for **optional** dependencies.

---

## Step 3 — How `@Autowired` Resolves (Type → Name → Qualifier)

```
Spring looks for a bean to inject:

1. Match by TYPE
   └── exactly one match → inject it

2. Multiple matches by type?
   └── Match by FIELD/PARAMETER NAME as bean name
       └── match found → inject it

3. Still ambiguous?
   └── Use @Qualifier("beanName") to specify exactly which one

4. No match?
   └── NoSuchBeanDefinitionException at startup
```

### Example — Multiple implementations

```java
public interface PaymentGateway {
    void charge(double amount);
}

@Component("stripeGateway")
public class StripeGateway implements PaymentGateway { ... }

@Component("razorpayGateway")
public class RazorpayGateway implements PaymentGateway { ... }
```

```java
@Service
public class OrderService {

    private final PaymentGateway paymentGateway;

    // @Qualifier resolves ambiguity
    public OrderService(@Qualifier("stripeGateway") PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

Or mark one as primary:

```java
@Component
@Primary   // used when no @Qualifier specified
public class StripeGateway implements PaymentGateway { ... }
```

---

## Bean Scopes — How Many Instances?

```java
@Scope("singleton")   // DEFAULT — one instance per ApplicationContext
@Scope("prototype")   // new instance every time it's injected/requested
@Scope("request")     // one per HTTP request (web apps)
@Scope("session")     // one per HTTP session (web apps)
```

```java
@Service
@Scope("singleton")   // default, no need to write this
public class PaymentService { }
// Same object injected everywhere in the app
```

```java
@Component
@Scope("prototype")
public class ReportGenerator { }
// New ReportGenerator created every time it's injected
```

**Singleton is the default and the most common** — Spring creates the bean once at startup and reuses it.

---

## The Bean Lifecycle

```
ApplicationContext starts
         │
         ▼
1. Bean Definition loaded (scanning / @Bean methods)
         │
         ▼
2. Bean Instantiation  (constructor called)
         │
         ▼
3. Dependency Injection  (constructor args / @Autowired fields set)
         │
         ▼
4. @PostConstruct method called  (custom init logic)
         │
         ▼
5. Bean is READY — used by the app
         │
         ▼
6. ApplicationContext shuts down
         │
         ▼
7. @PreDestroy method called  (cleanup logic)
         │
         ▼
8. Bean destroyed
```

```java
@Service
public class CacheService {

    @PostConstruct
    public void init() {
        // runs after Spring injects all dependencies
        // good for: loading initial data, warming up cache
        loadCacheFromDB();
    }

    @PreDestroy
    public void cleanup() {
        // runs before Spring destroys the bean
        // good for: closing connections, flushing buffers
        cache.clear();
    }
}
```

---

## How `@SpringBootApplication` Bootstraps Everything

```java
@SpringBootApplication
// = @Configuration + @ComponentScan + @EnableAutoConfiguration
public class MyApp {
    public static void main(String[] args) {
        ApplicationContext ctx = SpringApplication.run(MyApp.class, args);
    }
}
```

Internal sequence:

```
SpringApplication.run()
        │
        ▼
1. Creates ApplicationContext (AnnotationConfigServletWebServerApplicationContext for web)
        │
        ▼
2. Loads @Configuration classes
        │
        ▼
3. Component scan — finds all @Component/@Service/@Repository/@Controller
        │
        ▼
4. Auto-configuration — reads spring.factories / AutoConfiguration.imports
        │
        ▼
5. Creates BeanDefinitions for all found classes
        │
        ▼
6. Resolves dependency graph (topological sort — no circular deps)
        │
        ▼
7. Instantiates beans in dependency order
   (dependencies created first, then beans that need them)
        │
        ▼
8. Injects dependencies
        │
        ▼
9. Calls @PostConstruct methods
        │
        ▼
10. ApplicationContext is READY
        │
        ▼
11. Starts embedded Tomcat / Netty (if web app)
```

---

## Circular Dependency Problem

```java
@Service
public class A {
    public A(B b) { }   // A needs B
}

@Service
public class B {
    public B(A a) { }   // B needs A
}
// BeanCurrentlyInCreationException — Spring can't resolve this
```

**Fix options:**

```java
// Option 1: Redesign — extract shared logic into a third bean C

// Option 2: Use @Lazy on one side
@Service
public class A {
    public A(@Lazy B b) { }   // B is created lazily, breaks the cycle
}

// Option 3: Setter injection on one side (Spring injects after construction)
@Service
public class B {
    @Autowired
    public void setA(A a) { }
}
```

---

## Practical Example — Full Flow

```java
// 1. Repository layer
@Repository
public class OrderRepository {
    public Order findById(Long id) { ... }
}

// 2. Service layer — depends on Repository
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    public OrderService(OrderRepository orderRepository,
                        PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }

    public void placeOrder(Long orderId) {
        Order order = orderRepository.findById(orderId);
        paymentService.charge(order.getAmount());
    }
}

// 3. Controller — depends on Service
@RestController
@RequestMapping("/orders")
public class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping("/{id}/place")
    public ResponseEntity<Void> place(@PathVariable Long id) {
        orderService.placeOrder(id);
        return ResponseEntity.ok().build();
    }
}
```

Spring resolves this automatically:
```
OrderRepository (no deps) → created first
PaymentService  (no deps) → created
OrderService    (needs both above) → created + injected
OrderController (needs OrderService) → created + injected
```

---

## Key Interview Points

1. **IoC Container = ApplicationContext** — Spring owns object creation and wiring, not your code.

2. **Constructor injection is best** — immutable, fail-fast, testable without Spring.

3. **Default scope is Singleton** — one instance shared everywhere; use `@Scope("prototype")` when you need a new instance each time.

4. **Resolution order: type → name → @Qualifier** — Spring first matches by type, then by name if ambiguous.

5. **`@SpringBootApplication` = `@ComponentScan` + `@Configuration` + `@EnableAutoConfiguration`** — scans your package tree and auto-wires everything.

6. **`@PostConstruct` / `@PreDestroy`** — hooks into bean lifecycle for init and cleanup logic.

7. **Circular dependencies** fail at startup with constructor injection — a feature, not a bug (forces better design).

---

## One-Line Summary for Interviews

> "Spring's IoC container scans for `@Component`-annotated classes and `@Bean` methods, builds a dependency graph, instantiates beans in dependency order (deepest dependency first), and injects them via constructor/field/setter — all managed as singletons by default so the same instance is shared across the entire application."
