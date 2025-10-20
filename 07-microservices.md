# Microservices Arkitektur

## Introduktion

Tänk dig att du har en enorm app som gör allt - användare, produkter, betalningar, leverans, notifikationer. När något går sönder, går allt sönder. När du vill uppdatera betalningsfunktionen måste du deploya hela appen. När produktsökningen får mycket trafik måste du skala upp allt, även grejer som inte behöver det.

Det här är problemet med **monolitiska applikationer** - allt är ihopkopplat i en stor klump.

**Microservices** är ett annat sätt att tänka. Istället för en stor app, bygger du många små appar som samarbetar. Varje app (eller service) gör en specifik sak och gör den bra. De pratar med varandra över nätverket, men är annars helt oberoende.

Tänk dig en stor restaurang kontra flera små specialiserade ställen:

- **Monolith**: En restaurang som serverar pizza, sushi, burgare, och bakverk. En kock måste kunna allt. Om köket går sönder stänger hela stället.
- **Microservices**: En pizzeria, en sushirestaurang, en hamburgerbar, och ett bageri. Varje ställe är expert på sin grej. Om pizzerian stänger kan du fortfarande få sushi.

Microservices är inte alltid rätt lösning (vi kommer prata mycket om det!), men för stora, komplexa system kan det vara game-changing.

## Monolith vs Microservices

Låt oss titta närmare på skillnaderna.

### Monolitisk arkitektur

En **monolith** är en applikation där allt finns i en kodbas och körs som en process.

```
[Användare] 
    ↓
[Load Balancer]
    ↓
[En stor applikation]
├── User Module
├── Product Module
├── Order Module
├── Payment Module
└── Notification Module
    ↓
[Databas]
```

**Karakteristik:**

- All kod i ett projekt
- En deployment
- Alla features delar samma resurser (minne, CPU)
- Tight coupling mellan komponenter
- Vanligtvis en databas för allt

**Exempel:** En e-handel Spring Boot app där alla controllers, services, och repositories finns i samma projekt.

```java
e-commerce-app/
├── src/
│   ├── controllers/
│   │   ├── UserController.java
│   │   ├── ProductController.java
│   │   └── OrderController.java
│   ├── services/
│   └── repositories/
└── pom.xml
```

### Microservices arkitektur

**Microservices** betyder att du delar upp applikationen i flera små, självständiga services.

```
[Användare]
    ↓
[API Gateway]
    ↓
    ├─→ [User Service] → [Users DB]
    ├─→ [Product Service] → [Products DB]
    ├─→ [Order Service] → [Orders DB]
    ├─→ [Payment Service] → [Payments DB]
    └─→ [Notification Service] → [Message Queue]
```

**Karakteristik:**

- Varje service är en separat applikation
- Oberoende deployments
- Varje service har sina egna resurser
- Loose coupling via API:er
- Ofta en databas per service

**Exempel:** Samma e-handel, men uppdelad:

```
e-commerce/
├── user-service/          (Spring Boot app)
├── product-service/       (Spring Boot app)
├── order-service/         (Spring Boot app)
├── payment-service/       (Node.js app - olika tech!)
└── notification-service/  (Python app)
```

### Jämförelse sida vid sida

| Aspekt          | Monolith            | Microservices                    |
| --------------- | ------------------- | -------------------------------- |
| **Struktur**    | En applikation      | Många små applikationer          |
| **Deployment**  | Allt på en gång     | Varje service separat            |
| **Skalning**    | Skala hela appen    | Skala varje service individuellt |
| **Teknologi**   | Ett tech stack      | Olika tech per service           |
| **Development** | Ett team, en kodbas | Flera team, flera kodbaser       |
| **Complexity**  | Enklare att börja   | Mer komplex från start           |
| **Testing**     | Lättare att testa   | Svårare integration testing      |
| **Debugging**   | Allt i en process   | Distribuerat över nätverk        |

## Fördelar med microservices

### 1. Oberoende skalning

Det här är en av de största fördelarna. I en monolith måste du skala allt, även om bara en del behöver det.

**Scenario:** Din produktsökning får 10x mer trafik än allt annat.

**Med monolith:**

```
[App Instans 1] - hela appen skalas
[App Instans 2] - hela appen skalas
[App Instans 3] - hela appen skalas
[App Instans 4] - hela appen skalas
```

Du betalar för att skala funktioner som inte behöver det!

**Med microservices:**

```
[Product Service Instans 1-10]  ← Skala den här mycket
[User Service Instans 1-2]      ← Behöver inte lika mycket
[Order Service Instans 1-3]     ← Mellan
```

Du skalar precis det som behövs. Spar resurser och pengar!

### 2. Oberoende deployment

I en monolith måste du deploya hela appen när du ändrar något.

**Problem:**

- Risk: en liten ändring i produktsökning kan krasha hela appen
- Långsam: måste bygga och deploya allt
- Koordinering: flera team måste vänta på varandra

**Med microservices:**

```
Måndagen:    Deploy Product Service v2.0
Tisdagen:    Deploy User Service v1.5  
Onsdagen:    Deploy Order Service v3.2
```

Varje service deployar när det passar. Ingen väntan. Mindre risk.

### 3. Teknologival per service

Du behöver inte använda samma tech överallt!

```
User Service      → Java/Spring Boot (bra för business logic)
Product Service   → Node.js (snabb för I/O)
Payment Service   → Go (performance kritisk)
ML Service        → Python (machine learning libraries)
```

**Fördelar:**

- Använd rätt verktyg för rätt jobb
- Experimentera med ny tech i en service
- Anställ specialister för olika services

### 4. Team autonomi

Microservices passar perfekt för större organisationer med flera team.

**Teamstruktur:**

```
User Team       → Äger User Service helt
Product Team    → Äger Product Service helt
Order Team      → Äger Order Service helt
```

Varje team kan:

- Fatta egna tekniska beslut
- Deploya när de vill
- Iterera snabbt utan att vänta på andra
- Ha end-to-end ansvar

### 5. Fault isolation

När något går sönder i en monolith kan hela appen krascha. Med microservices kan fel isoleras.

**Exempel:**

```
Payment Service är nere
    ↓
Användare kan fortfarande:
- Bläddra produkter ✓
- Läsa reviews ✓
- Lägga till i varukorgen ✓

Men inte:
- Slutföra köp ✗
```

Systemet "degraderar gracefully" istället för att krascha totalt.

**Implementera med Circuit Breaker:**

```java
// Om Payment Service är nere, visa meddelande istället
@CircuitBreaker(name = "payment", fallbackMethod = "paymentFallback")
public PaymentResult processPayment(Order order) {
    return paymentService.pay(order);
}

public PaymentResult paymentFallback(Order order, Exception e) {
    return PaymentResult.temporarilyUnavailable();
}
```

### 6. Enklare att förstå

En stor monolith kan ha hundratusentals rader kod. Svårt att förstå helheten.

**Microservice:**

- Liten kodbas (kanske 5000-10000 rader)
- Gör EN sak
- Lätt att förstå och navigera
- Nya utvecklare kommer igång snabbare

## Nackdelar med microservices

Det är inte bara solsken och regnbågar. Microservices kommer med sina egna problem.

### 1. Ökad komplexitet

Istället för en applikation har du nu 10, 20, eller 50 services.

**Frågor du måste hantera:**

- Hur hittar services varandra?
- Hur monitorerar du 20 services?
- Hur debuggar du när en request går genom 5 services?
- Hur håller du koll på versioner?

**Du behöver:**

- Service discovery (Eureka, Consul)
- API Gateway
- Distributed tracing (Zipkin, Jaeger)
- Centralized logging (ELK stack)
- Service mesh (Istio, Linkerd)

Det är mycket mer att sätta upp och underhålla!

### 2. Network latency

I en monolith är funktionsanrop i minnet (nanosekunder). Med microservices är det HTTP calls över nätverk (millisekunder).

```java
// Monolith - snabbt
Product product = productService.getProduct(id);
User user = userService.getUser(userId);
// Microsekunder ⚡

// Microservices - långsammare
Product product = restTemplate.getForObject("http://product-service/products/" + id);
User user = restTemplate.getForObject("http://user-service/users/" + userId);
// Millisekunder 🐌
```

**Dessutom:**

- Nätverket kan gå ner
- Services kan vara överbelastade
- Behöver hantera timeouts och retries

### 3. Data consistency utmaningar

I en monolith kan du använda databas-transactions:

```java
@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);
    inventoryService.decreaseStock(order.getProductId());
    // Om något felar, rollback ALLT
}
```

Med microservices har varje service sin egen databas. Inga distribuerade transactions!

```
Order Service    → Skapar order ✓
Inventory Service → Minskar lager ✗ (kraschar)
// Nu har vi en order utan att lagret minskade! 💥
```

**Lösningar:**

- Eventual consistency (acceptera att data kan vara inkonsistent ett tag)
- Saga pattern (kompensating transactions)
- Event sourcing

Det är mycket mer komplext!

### 4. Testing blir svårare

**Unit tests:** Lätta, samma som innan

**Integration tests:** Mycket svårare!

```
För att testa "place order" flow behöver du:
- Order Service
- Product Service  
- Inventory Service
- Payment Service
- User Service
- Alla deras databaser
- Message queue
```

**Lösningar:**

- Contract testing (Pact)
- Test doubles
- End-to-end test environment
- Mer mock:ande

Men det är mer jobb att sätta upp!

### 5. Deployment komplexitet

**Monolith:** En jar-fil, en deployment.

**Microservices:**

```
Deploy 15 services
Varje service kan ha:
- Olika versioner
- Olika konfigurationer
- Olika dependencies
- Olika databaser
```

**Du behöver:**

- CI/CD pipelines för varje service
- Container orchestration (Kubernetes)
- Configuration management
- Database migrations per service

Det är mycket mer operations-arbete!

### 6. Initial overhead

Att sätta upp microservices tar tid:

**Första veckan:**

- ❌ Bygga features
- ✓ Sätta upp API Gateway
- ✓ Konfigurera service discovery
- ✓ Sätta upp monitoring
- ✓ Sätta upp logging
- ✓ Bygga deployment pipelines

För små projekt är det overkill!

## När ska man använda microservices?

Det här är kanske det viktigaste avsnittet. Microservices är inte alltid rätt svar!

### ✅ Använd microservices när:

**1. Stor, komplex applikation**

- Över 100,000 rader kod
- Många olika domäner/features
- Svårt att överblicka

**2. Flera team**

- 3+ team som jobbar på samma produkt
- Teams trampar på varandras tår
- Långsam deployment för att alla måste synka

**3. Olika skalningsbehov**

- Vissa features får 100x mer trafik än andra
- Versölar resurser att skala allt

**4. Snabba releases**

- Vill deploya flera gånger per dag
- Olika features utvecklas i olika takt
- Kan inte vänta på monolitisk release cycle

**5. Olika teknologival behövs**

- Vissa delar passar bättre i andra språk
- Vill använda specialiserade tech
- ML-komponenter behöver Python, annat kan vara Java

**6. DevOps mognad**

- Har erfarna ops-team
- Bra CI/CD pipelines
- Monitoring och logging på plats

### ❌ ANVÄND INTE microservices när:

**1. Liten applikation eller team**

```
Team: 2-3 utvecklare
Projekt: MVP eller småskalig app
→ BYGG EN MONOLITH!
```

**2. Just starting out**

- Vet inte riktigt vad appen ska göra än
- Domängränserna är oklara
- Kommer behöva refactor mycket

**Start med monolith. Dela upp senare om det behövs!**

**3. Ingen infrastruktur-erfarenhet**

- Kan inte Docker
- Har aldrig använt Kubernetes
- Vet inte hur man debuggar distribuerade system

Microservices kräver ops-kompetens!

**4. Kan inte hantera operational complexity**

- Ingen tid för monitoring/logging setup
- Ingen CI/CD pipeline
- Manuella deployments

**5. Oklara service boundaries**

- Vet inte hur man ska dela upp systemet
- Riskerar fel gränser (behöver göra om)

### Den viktiga principen:

> **"You don't start with microservices, you refactor into them"**
>
> - Martin Fowler

**Start med en välstrukturerad monolith:**

```
my-app/
├── user/
├── product/
├── order/
└── payment/
```

Redan här kan du tänka i "bounded contexts". När/om du växer kan du extrahera dessa till services.

**Monolith First-principen:**

1. Bygg monolith
2. Identifiera bottlenecks
3. Extrahera de delarna till services
4. Repetera om nödvändigt

Det är mycket lägre risk än att gissa gränser från start!

## Service boundaries - hur dela upp?

OK, så du har bestämt dig för microservices. Hur bestämmer du vad som ska vara en service?

### Domain-Driven Design (DDD)

Det bästa sättet är att använda **Domain-Driven Design** principer.

**Bounded Context:** Ett område i din domän med tydliga gränser där termer och regler är konsekventa.

**För e-handel:**

```
User Management Context:
- Users, authentication, profiles, permissions
→ User Service

Product Catalog Context:
- Products, categories, search, recommendations
→ Product Service

Order Management Context:
- Orders, order status, order history
→ Order Service

Payment Context:
- Payments, transactions, refunds
→ Payment Service

Shipping Context:
- Deliveries, tracking, addresses
→ Shipping Service

Notification Context:
- Emails, SMS, push notifications
→ Notification Service
```

Varje context är en business capability!

### Principer för service boundaries

**1. Single Responsibility**
En service ska göra EN sak och göra den bra.

```
✗ Dåligt: UserAndProductService
✓ Bra: UserService, ProductService
```

**2. High cohesion**
Saker som hör ihop ska vara tillsammans.

```
User Service ska ha:
- User registration ✓
- User authentication ✓
- User profile ✓

Inte:
- Product search ✗ (hör till Product Service)
```

**3. Low coupling**
Services ska vara så oberoende som möjligt.

```
✗ Order Service behöver direkt tillgång till Users-tabellen
✓ Order Service anropar User Service API
```

**4. Business capabilities, inte tekniska lager**

```
✗ Dålig uppdelning (technical layers):
- Database Service
- API Service  
- UI Service

✓ Bra uppdelning (business capabilities):
- User Service
- Product Service
- Order Service
```

### Exempel på dålig vs bra uppdelning

**❌ Dåligt: För granulära services**

```
- CreateUserService
- UpdateUserService
- DeleteUserService
- GetUserService
```

Det här är för finkorning! För mycket network overhead.

**✓ Bra: Balanserad granularitet**

```
- UserService (hanterar alla user operations)
```

**❌ Dåligt: För stora services**

```
- EverythingService (users + products + orders + payments)
```

Det här är bara en monolith med fancy namn!

### Hur identifiera gränser

**Ställ dessa frågor:**

1. **Kan denna feature deployras oberoende?**
   - Om ja → egen service
   - Om nej → kanske samma service

2. **Har den sitt eget data model?**
   - Olika entities → olika services
   - Samma entities → kanske samma service

3. **Utvecklas den av olika team?**
   - Olika team → egen service per team

4. **Har den olika skalningsbehov?**
   - Olika load patterns → egen service

5. **Representerar den en business capability?**
   - Ja → bra candidate för service

**Exempel: E-handel analys**

```
Users och Authentication:
- Samma team? Ja
- Samma data? Ja
- Samma scaling? Ja
→ En service: User Service

Product Catalog och Product Search:
- Samma team? Ja
- Samma data? Ja
- Men search kan ha annat tech (Elasticsearch)
→ Kanske dela: Product Service + Search Service
   (eller behåll tillsammans om inte scaling-problem)
```

## Service communication

Services måste prata med varandra. Det finns två huvudsakliga sätt:

### Synchronous communication (Request/Response)

Service A anropar Service B och **väntar** på svar.

**REST API (vanligast):**

```
Order Service                Product Service
     |                              |
     |------- GET /products/123 --->|
     |                              |
     |<------ 200 OK + Product -----|
     |                              |
```

**Fördelar:**

- Enkelt att förstå
- Omedelbart svar
- Lätt att implementera

**Nackdelar:**

- Tight coupling (Order Service beror på att Product Service är uppe)
- Cascading failures (om Product kraschar, kan Order också krascha)
- Långsammare (network latency)

**När använda:**

- När du BEHÖVER svaret direkt
- Exempel: Kolla produktpris innan order läggs

**REST exempel:**

```java
@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public Order createOrder(Long productId, int quantity) {
        // Synkront anrop - väntar på svar
        Product product = restTemplate.getForObject(
            "http://product-service/products/" + productId,
            Product.class
        );
        
        if (product == null || product.getStock() < quantity) {
            throw new InsufficientStockException();
        }
        
        Order order = new Order(productId, quantity, product.getPrice());
        return orderRepository.save(order);
    }
}
```

**gRPC (snabbare alternativ):**

gRPC använder Protocol Buffers (binary format) istället för JSON. Mycket snabbare!

```
REST:  ~50-100ms per call (JSON parsing overhead)
gRPC:  ~10-20ms per call (binary, mindre data)
```

Men REST är enklare och mer standardiserat. Använd gRPC när performance är kritiskt.

### Asynchronous communication (Event-based)

Service A skickar ett meddelande och fortsätter **utan att vänta**.

**Message Queue:**

```
Order Service              Message Queue         Email Service
     |                           |                     |
     |---> OrderPlacedEvent ---->|                     |
     |                           |                     |
     | (fortsätter direkt)       |                     |
     |                           |<---- poll events ---|
     |                           |---> OrderPlaced --->|
     |                           |                     |
     |                           |              (skickar email)
```

**Fördelar:**

- Loose coupling (services är oberoende)
- Bättre fault tolerance (om Email Service är nere, händer det senare)
- Kan hantera load spikes (messages köas)

**Nackdelar:**

- Mer komplext
- Eventual consistency (tar tid)
- Svårare att debugga

**När använda:**

- När operationen KAN vänta
- Exempel: Skicka email, uppdatera analytics, notifikationer

**RabbitMQ exempel:**

```java
@Service
public class OrderService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public Order createOrder(Long productId, int quantity) {
        Order order = new Order(productId, quantity);
        order = orderRepository.save(order);
        
        // Skicka event - väntar INTE på att någon hanterar det
        OrderPlacedEvent event = new OrderPlacedEvent(
            order.getId(), 
            order.getUserId(), 
            order.getTotal()
        );
        
        rabbitTemplate.convertAndSend("orders", "order.placed", event);
        
        return order; // Returnerar direkt
    }
}

@Service
public class EmailService {
    
    @RabbitListener(queues = "email-queue")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // Körs asynkront när meddelandet kommer
        sendConfirmationEmail(event.getUserId(), event.getOrderId());
    }
}
```

### Hybrid approach

I verkligheten använder du BÅDA!

**Exempel: User lägger en order**

**Synchronous (behöver svar direkt):**

```
Order Service → Product Service: "Finns produkt i lager?"
Order Service → Payment Service: "Är betalning OK?"
```

**Asynchronous (kan hända senare):**

```
Order Service → Message Queue: "Order lagd!"
  ↓
Email Service: Skickar bekräftelse
Analytics Service: Loggar event
Inventory Service: Uppdaterar lager
Shipping Service: Förbereder leverans
```

## API Gateway pattern

När du har många microservices behöver klienten inte prata med alla direkt. Det är där **API Gateway** kommer in.

### Vad är en API Gateway?

En **API Gateway** är en single entry point för alla klienter.

```
[Mobile App]  ─┐
               │
[Web App]     ─┤
               ├──→ [API Gateway] ──┬──→ [User Service]
[Partner API]─┤                     ├──→ [Product Service]
               │                    ├──→ [Order Service]
[Admin Panel]─┘                     └──→ [Payment Service]
```

Istället för att klienten pratar med 10 olika services, pratar den med en!

### API Gateway ansvar

**1. Request routing**

```
GET /api/users/*      → User Service
GET /api/products/*   → Product Service
GET /api/orders/*     → Order Service
```

**2. Authentication & Authorization**

```java
// Gateway kollar JWT token innan request går vidare
if (!isValidToken(request)) {
    return 401 Unauthorized;
}
```

**3. Rate limiting**

```java
// Max 100 requests per minut per användare
if (requestCount > 100) {
    return 429 Too Many Requests;
}
```

**4. Load balancing**

```
Request → Gateway → Product Service (instans 1, 2, eller 3)
```

**5. Response aggregation**

Istället för att klienten gör 3 calls:

```
Client → User Service (get user)
Client → Product Service (get favorite products)  
Client → Order Service (get recent orders)
```

Gateway kan aggregera:

```
Client → Gateway → [calls all 3 services] → Returns combined response
```

**6. Protocol translation**

```
External: HTTPS REST
Internal: gRPC (snabbare)
```

### Fördelar med API Gateway

- **Enkel klient:** Klienten behöver bara känna till gateway
- **Centraliserad säkerhet:** Auth på ett ställe
- **Flexibilitet:** Kan ändra backend utan att ändra klient
- **Monitoring:** Alla requests går genom en punkt

### Nackdelar

- **Single point of failure:** Om gateway går ner, går allt ner (därför kör flera instanser!)
- **Bottleneck:** Kan bli performance-problem
- **Complexity:** En till komponent att underhålla

### Spring Cloud Gateway exempel

```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
            // User Service routes
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .stripPrefix(1)  // Ta bort /api från path
                    .addRequestHeader("X-Gateway", "true"))
                .uri("lb://USER-SERVICE"))  // Load balance
            
            // Product Service routes  
            .route("product-service", r -> r
                .path("/api/products/**")
                .filters(f -> f.stripPrefix(1))
                .uri("lb://PRODUCT-SERVICE"))
            
            // Order Service routes
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .circuitBreaker(c -> c  // Circuit breaker!
                        .setName("ordersCB")
                        .setFallbackUri("/fallback/orders")))
                .uri("lb://ORDER-SERVICE"))
            
            .build();
    }
}
```

**application.yml:**

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

### Populära API Gateway verktyg

- **Spring Cloud Gateway** (Java, integrerat med Spring ecosystem)
- **Netflix Zuul** (äldre, Spring Cloud Gateway är bättre nu)
- **Kong** (populär open source, Lua-baserad)
- **Amazon API Gateway** (AWS managed)
- **Nginx** (för enkla use cases)

## Service discovery

Nu har vi ett nytt problem: hur vet API Gateway var services finns?

### Problemet

I microservices miljö:

- Services startar och stänger dynamiskt
- IP-adresser och portar ändras
- Auto-scaling lägger till/tar bort instanser

**Hardcoding funkar inte:**

```java
// ✗ Vad händer när IP ändras?
String url = "http://192.168.1.50:8081/products";
```

### Lösning: Service Registry

En **Service Registry** är en databas över alla services och var de finns.

```
      [Service Registry]
        /      |      \
       /       |       \
 Register  Register  Register
     |        |         |
[Service A] [Service B] [Service C]
```

**Hur det funkar:**

1. **Service startar** → Registrerar sig i registry
2. **Service behöver prata med annan service** → Frågar registry
3. **Service stängs** → Avregistrerar sig (eller registry märker att den inte svarar)

### Client-side discovery

Klienten frågar registry och får tillbaka adresser:

```
Order Service                Registry              Product Service
     |                           |                       |
     |--- "Var är Product?" ---->|                       |
     |<--- "192.168.1.50:8080" --|                       |
     |                           |                       |
     |-------------- HTTP call ---------------->         |
```

**Fördelar:** Klienten har kontroll, kan implementera egen load balancing
**Nackdelar:** Varje klient måste implementera discovery-logik

### Server-side discovery

Load balancer frågar registry:

```
Order Service    Load Balancer       Registry         Product Service
     |                 |                 |                  |
     |--- request ----> |                 |                  |
     |                 |-- "Var är?" --->|                  |
     |                 |<-- address -----|                  |
     |                 |-------------- HTTP call -------->  |
```

**Fördelar:** Klienten behöver inte veta om discovery
**Nackdelar:** Load balancer blir en extra hop

### Netflix Eureka (med Spring Cloud)

Eureka är den populäraste service registry för Spring Boot.

**Eureka Server:**

```java
@SpringBootApplication
@EnableEurekaServer  // Det är allt som behövs!
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false # Server registrerar inte sig själv
    fetch-registry: false
```

**Eureka Client (varje service):**

```java
@SpringBootApplication
@EnableDiscoveryClient  // Registrera denna service
public class ProductServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}
```

```yaml
# application.yml
spring:
  application:
    name: PRODUCT-SERVICE # Service namn i registry

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/ # Eureka server address
  instance:
    prefer-ip-address: true
```

**Använda discovery för att anropa annan service:**

```java
@Service
public class OrderService {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Autowired
    private RestTemplate restTemplate;
    
    public Product getProduct(Long productId) {
        // Få alla instanser av PRODUCT-SERVICE
        List<ServiceInstance> instances = 
            discoveryClient.getInstances("PRODUCT-SERVICE");
        
        if (instances.isEmpty()) {
            throw new ServiceUnavailableException("Product service down");
        }
        
        // Använd första instansen (i produktion: load balance)
        ServiceInstance instance = instances.get(0);
        String url = instance.getUri() + "/products/" + productId;
        
        return restTemplate.getForObject(url, Product.class);
    }
}
```

**Eller ännu enklare med Load Balancer:**

```java
@Configuration
public class Config {
    
    @Bean
    @LoadBalanced  // Detta magiska annotationen!
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public Product getProduct(Long productId) {
        // Använd service-namn istället för IP!
        // Spring resolver automatiskt via Eureka
        return restTemplate.getForObject(
            "http://PRODUCT-SERVICE/products/" + productId,
            Product.class
        );
    }
}
```

### I Kubernetes

Kubernetes har **inbyggd service discovery**!

```yaml
# product-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product
  ports:
    - port: 8080
```

Nu kan andra services anropa:

```
http://product-service:8080/products/123
```

Kubernetes DNS resolver `product-service` automatiskt! Ingen Eureka behövs.

## Database per service pattern

En av de viktigaste principerna i microservices:

> **Varje service ska ha sin egen databas**

### Varför?

**1. Loose coupling**
Services ska vara oberoende. Om de delar databas har du tight coupling.

```
✗ Dåligt:
[Order Service] ──┐
                  ├──→ [Shared Database]
[Product Service]─┘

Om Product Service ändrar schema kan Order Service gå sönder!
```

**2. Skalning**
Olika services har olika databas-behov.

```
✓ Bra:
[Product Service] → [PostgreSQL] (relationell data)
[Search Service]  → [Elasticsearch] (full-text search)
[Cache Service]   → [Redis] (snabb cache)
```

**3. Team autonomi**
Team ska kunna göra databas-ändringar utan att koordinera.

### Hur ser det ut?

```
[User Service]    → [Users Database]
[Product Service] → [Products Database]  
[Order Service]   → [Orders Database]
[Payment Service] → [Payments Database]
```

Varje service äger sin data helt!

### Utmaningar

**1. Inga joins över services**

I en monolith:

```sql
SELECT o.*, u.name, p.name 
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN products p ON o.product_id = p.id
```

Med microservices: **Du kan inte göra den joinen!**

**Lösning A: API calls**

```java
public OrderDetails getOrderDetails(Long orderId) {
    Order order = orderRepository.findById(orderId);
    
    // Anropa andra services
    User user = restTemplate.getForObject(
        "http://USER-SERVICE/users/" + order.getUserId(),
        User.class
    );
    
    Product product = restTemplate.getForObject(
        "http://PRODUCT-SERVICE/products/" + order.getProductId(),
        Product.class
    );
    
    return new OrderDetails(order, user, product);
}
```

**Lösning B: Data duplication**

```java
// Order Service sparar data den behöver
@Entity
public class Order {
    private Long id;
    private Long userId;
    private String userName;        // Duplicerad från User Service
    private Long productId;
    private String productName;     // Duplicerad från Product Service
    private BigDecimal productPrice; // Duplicerad från Product Service
}
```

Duplication är OK i microservices! Det är trade-off för independence.

**2. Data consistency**

Mer om detta i nästa sektion...

## Data consistency utmaningar

Det här är ett av de svåraste problemen med microservices.

### ACID i monolith

I en monolith kan du använda databas-transactions:

```java
@Transactional
public void placeOrder(OrderRequest request) {
    // Allt händer i en transaktion
    Order order = createOrder(request);
    decreaseInventory(request.getProductId(), request.getQuantity());
    processPayment(order.getTotal());
    
    // Om NÅGOT felar, rollback ALLT ✓
}
```

### Problemet i microservices

Nu är det tre olika services med tre olika databaser:

```
[Order Service]     → [Orders DB]
[Inventory Service] → [Inventory DB]
[Payment Service]   → [Payments DB]
```

**Vad händer om:**

```
1. Order skapas ✓
2. Inventory minskas ✓  
3. Payment felar ✗

Nu har vi en order och minskat lager, men ingen betalning! 💥
```

**Du kan INTE ha en distribuerad transaktion över tre databaser.**
(Tekniskt möjligt med 2-phase commit, men väldigt komplext och sällansyntant)

### Lösning 1: Eventual Consistency

Acceptera att data kan vara **temporärt inkonsistent**.

**Exempel:**

```
1. Order läggs
2. Event skickas: "OrderPlaced"
3. Inventory Service lyssnar och minskar lager (några sekunder senare)
4. Email Service skickar bekräftelse (några sekunder senare)
```

För en liten stund är systemet inkonsistent, men det löser sig!

**När det funkar:**

- Icke-kritiska operationer
- Användare kan tolerera delay
- Exempel: analytics, notifikationer, cache uppdateringar

**När det INTE funkar:**

- Pengar-relaterat (betalningar!)
- Inventory (kan inte sälja mer än du har)

### Lösning 2: Saga Pattern

**Saga** är en sekvens av lokala transaktioner med **compensating transactions**.

**Example: Place Order Saga**

```
Order Service          Inventory Service      Payment Service
     |                        |                      |
     |--1. Create Order -->   |                      |
     |<-- OK ------------     |                      |
     |                        |                      |
     |--2. Reserve Stock ---->|                      |
     |<-- OK ----------------||                      |
     |                        |                      |
     |--3. Process Payment ---|--------------------->|
     |<-- FAILED -------------|----------------------|
     |                        |                      |
     |--4. Release Stock ---->| (compensating!)      |
     |<-- OK -----------------|                      |
     |                        |                      |
     |--5. Cancel Order       |                      |
```

Om Payment felar, kör vi **compensating transactions** bakåt!

**Orchestration-based Saga:**
En central orchestrator styr processen.

```java
@Service
public class OrderSaga {
    
    public void placeOrder(OrderRequest request) {
        Long orderId = null;
        boolean inventoryReserved = false;
        
        try {
            // Steg 1: Skapa order
            orderId = orderService.createOrder(request);
            
            // Steg 2: Reservera lager
            inventoryService.reserveStock(
                request.getProductId(), 
                request.getQuantity()
            );
            inventoryReserved = true;
            
            // Steg 3: Processa betalning
            paymentService.processPayment(orderId, request.getTotal());
            
            // SUCCESS! Confirm allt
            orderService.confirmOrder(orderId);
            inventoryService.confirmReservation(request.getProductId());
            
        } catch (PaymentFailedException e) {
            // Compensate!
            if (inventoryReserved) {
                inventoryService.releaseStock(
                    request.getProductId(), 
                    request.getQuantity()
                );
            }
            if (orderId != null) {
                orderService.cancelOrder(orderId);
            }
            throw new OrderFailedException("Payment failed", e);
        }
    }
}
```

**Choreography-based Saga:**
Services kommunicerar via events, ingen central orchestrator.

```java
// Order Service
@Service
public class OrderService {
    
    @Autowired
    private EventPublisher eventPublisher;
    
    public void placeOrder(OrderRequest request) {
        Order order = createOrder(request);
        
        // Publicera event
        eventPublisher.publish(new OrderCreatedEvent(
            order.getId(),
            order.getProductId(),
            order.getQuantity(),
            order.getTotal()
        ));
    }
    
    @EventListener
    public void onPaymentFailed(PaymentFailedEvent event) {
        // Compensate - cancel order
        cancelOrder(event.getOrderId());
    }
}

// Inventory Service
@Service
public class InventoryService {
    
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        try {
            reserveStock(event.getProductId(), event.getQuantity());
            
            eventPublisher.publish(new StockReservedEvent(
                event.getOrderId(),
                event.getProductId()
            ));
        } catch (InsufficientStockException e) {
            eventPublisher.publish(new StockReservationFailedEvent(
                event.getOrderId()
            ));
        }
    }
    
    @EventListener
    public void onPaymentFailed(PaymentFailedEvent event) {
        // Compensate - release stock
        releaseStock(event.getProductId(), event.getQuantity());
    }
}

// Payment Service
@Service
public class PaymentService {
    
    @EventListener
    public void onStockReserved(StockReservedEvent event) {
        try {
            processPayment(event.getOrderId());
            
            eventPublisher.publish(new PaymentSuccessEvent(
                event.getOrderId()
            ));
        } catch (PaymentException e) {
            eventPublisher.publish(new PaymentFailedEvent(
                event.getOrderId()
            ));
        }
    }
}
```

Mycket mer komplext! Men ingen central point of failure.

**Takeaway:** Data consistency i microservices är SVÅRT. Det är en stor anledning att inte börja med microservices!

## Exempel: E-commerce microservices

Låt oss titta på ett komplett exempel!

### Services översikt

```
┌─────────────────┐
│   API Gateway   │
└────────┬────────┘
         │
    ┌────┴────┬───────────┬──────────┬───────────┬─────────────┐
    │         │           │          │           │             │
┌───▼──┐  ┌──▼───┐  ┌────▼───┐  ┌──▼────┐  ┌───▼────┐  ┌────▼─────┐
│ User │  │Product│  │ Order  │  │Payment│  │Inventory│ │Notification│
│Service│ │Service│  │Service │  │Service│  │ Service│  │  Service  │
└───┬──┘  └──┬───┘  └────┬───┘  └──┬────┘  └───┬────┘  └────┬─────┘
    │        │           │          │           │             │
┌───▼──┐  ┌──▼───┐  ┌────▼───┐  ┌──▼────┐  ┌───▼────┐       │
│Users │  │Products│ │Orders │  │Payments│ │Inventory│       │
│  DB  │  │  DB   │  │  DB   │  │  DB   │  │   DB   │       │
└──────┘  └───────┘  └────────┘  └───────┘  └────────┘       │
                                                               │
                                                         ┌─────▼─────┐
                                                         │   RabbitMQ│
                                                         └───────────┘
```

### Service ansvar

**1. User Service**

```
Ansvar:
- User registration
- Authentication (login/logout)
- User profiles
- Password management

Teknologi: Spring Boot + PostgreSQL
Port: 8081
Database: users_db
```

**2. Product Service**

```
Ansvar:
- Product catalog
- Product search
- Categories
- Product recommendations

Teknologi: Spring Boot + PostgreSQL + Elasticsearch (search)
Port: 8082
Database: products_db
```

**3. Order Service**

```
Ansvar:
- Create orders
- Order history
- Order status tracking
- Saga orchestration

Teknologi: Spring Boot + PostgreSQL
Port: 8083
Database: orders_db
```

**4. Payment Service**

```
Ansvar:
- Process payments
- Payment history
- Refunds
- Integration med Stripe/PayPal

Teknologi: Node.js + MongoDB (olika tech!)
Port: 8084
Database: payments_db
```

**5. Inventory Service**

```
Ansvar:
- Stock management
- Stock reservations
- Restock notifications

Teknologi: Spring Boot + PostgreSQL
Port: 8085
Database: inventory_db
```

**6. Notification Service**

```
Ansvar:
- Send emails
- Send SMS
- Push notifications
- Listen för events

Teknologi: Python + Redis (queue)
Port: 8086
No database (stateless)
```

### Communication flow: User lägger order

**Steg-för-steg:**

```
1. [Web App] → [API Gateway] → [User Service]
   POST /api/login
   → JWT token tillbaka

2. [Web App] → [API Gateway] → [Product Service]
   GET /api/products/search?q=laptop
   → Lista av produkter

3. [Web App] → [API Gateway] → [Product Service]
   GET /api/products/123
   → Produkt detaljer

4. [Web App] → [API Gateway] → [Order Service]
   POST /api/orders
   {
     "productId": 123,
     "quantity": 1
   }

   Order Service kör Saga:
   
   a) [Order Service] → Skapar order i Orders DB
   
   b) [Order Service] --SYNC--> [Product Service]
      GET /products/123
      ← Får produkt info och pris
   
   c) [Order Service] --SYNC--> [Inventory Service]
      POST /inventory/reserve
      ← Reserverar lager
   
   d) [Order Service] --SYNC--> [Payment Service]
      POST /payments
      ← Processar betalning
   
   e) [Order Service] → Confirm order i Orders DB
   
   f) [Order Service] --ASYNC--> [RabbitMQ]
      Publicerar: OrderPlacedEvent
   
   → Order bekräftelse tillbaka till user

5. [Notification Service] lyssnar på RabbitMQ
   ← OrderPlacedEvent
   → Skickar bekräftelse-email

6. [Inventory Service] lyssnar på RabbitMQ
   ← OrderPlacedEvent  
   → Confirmar stock reservation
```

**Synchronous calls (behöver svar):**

- Order → Product (behöver pris)
- Order → Inventory (måste kolla lager)
- Order → Payment (måste bekräfta betalning)

**Asynchronous calls (kan vänta):**

- Order → Notification (email kan skickas senare)
- Order → Analytics (kan loggas senare)

### Project struktur

```
e-commerce-microservices/
│
├── api-gateway/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── user-service/
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/ecommerce/user/
│   │       │       ├── UserServiceApplication.java
│   │       │       ├── controller/
│   │       │       ├── service/
│   │       │       ├── repository/
│   │       │       └── model/
│   │       └── resources/
│   │           └── application.yml
│   ├── pom.xml
│   └── Dockerfile
│
├── product-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── order-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── payment-service/     (Node.js!)
│   ├── src/
│   ├── package.json
│   └── Dockerfile
│
├── inventory-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── notification-service/ (Python!)
│   ├── src/
│   ├── requirements.txt
│   └── Dockerfile
│
├── eureka-server/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
└── docker-compose.yml
```

Varje service är ett helt separat projekt!

## Spring Boot och microservices

Spring Boot är perfekt för microservices! Varje service är bara en Spring Boot app.

### Spring Cloud komponenter

**Spring Cloud** ger dig verktyg för microservices patterns:

**1. Spring Cloud Gateway**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

**2. Spring Cloud Netflix Eureka**

```xml
<!-- Eureka Server -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<!-- Eureka Client (i varje service) -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**3. Spring Cloud Config**
Centralized configuration management.

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

**4. Spring Cloud OpenFeign**
Declarative REST client - mycket enklare än RestTemplate!

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

**5. Spring Cloud Circuit Breaker**
Resilience patterns.

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

### Minimal microservice setup

**Order Service:**

```java
@SpringBootApplication
@EnableDiscoveryClient  // Registrera i Eureka
@EnableFeignClients     // Aktivera Feign clients
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

**application.yml:**

```yaml
spring:
  application:
    name: ORDER-SERVICE

  datasource:
    url: jdbc:postgresql://localhost:5432/orders_db
    username: postgres
    password: password

  jpa:
    hibernate:
      ddl-auto: update

server:
  port: 8083

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

**Controller:**

```java
@RestController
@RequestMapping("/orders")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        Order order = orderService.createOrder(request);
        return ResponseEntity.ok(order);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrder(@PathVariable Long id) {
        return orderService.getOrder(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/user/{userId}")
    public ResponseEntity<List<Order>> getUserOrders(@PathVariable Long userId) {
        List<Order> orders = orderService.getOrdersByUser(userId);
        return ResponseEntity.ok(orders);
    }
}
```

Det är en vanlig Spring Boot app! Skillnaden är att den pratar med andra services.

## Inter-service communication exempel

### Med RestTemplate

```java
@Service
public class OrderService {
    
    @Autowired
    @LoadBalanced  // Load balance via Eureka
    private RestTemplate restTemplate;
    
    @Autowired
    private OrderRepository orderRepository;
    
    public Order createOrder(OrderRequest request) {
        // 1. Hämta produkt-info från Product Service
        Product product = restTemplate.getForObject(
            "http://PRODUCT-SERVICE/products/" + request.getProductId(),
            Product.class
        );
        
        if (product == null) {
            throw new ProductNotFoundException();
        }
        
        // 2. Kolla lager i Inventory Service
        InventoryResponse inventory = restTemplate.postForObject(
            "http://INVENTORY-SERVICE/inventory/check",
            new InventoryCheckRequest(request.getProductId(), request.getQuantity()),
            InventoryResponse.class
        );
        
        if (!inventory.isAvailable()) {
            throw new InsufficientStockException();
        }
        
        // 3. Skapa ordern
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setProductId(request.getProductId());
        order.setQuantity(request.getQuantity());
        order.setTotalPrice(product.getPrice().multiply(
            BigDecimal.valueOf(request.getQuantity())
        ));
        order.setStatus(OrderStatus.PENDING);
        
        order = orderRepository.save(order);
        
        // 4. Reservera lager
        restTemplate.postForObject(
            "http://INVENTORY-SERVICE/inventory/reserve",
            new ReserveStockRequest(order.getId(), request.getProductId(), request.getQuantity()),
            Void.class
        );
        
        // 5. Processa betalning
        PaymentResponse payment = restTemplate.postForObject(
            "http://PAYMENT-SERVICE/payments",
            new PaymentRequest(order.getId(), order.getTotalPrice()),
            PaymentResponse.class
        );
        
        if (payment.isSuccess()) {
            order.setStatus(OrderStatus.CONFIRMED);
            orderRepository.save(order);
        } else {
            // Rollback inventory reservation
            restTemplate.postForObject(
                "http://INVENTORY-SERVICE/inventory/release",
                new ReleaseStockRequest(request.getProductId(), request.getQuantity()),
                Void.class
            );
            throw new PaymentFailedException();
        }
        
        return order;
    }
}
```

**RestTemplate bean:**

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced  // Detta gör att service names fungerar!
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### Med Feign Client (mycket enklare!)

Feign är en **declarative REST client**. Du definierar bara interface, resten sker automagiskt!

**Product Client:**

```java
@FeignClient(name = "PRODUCT-SERVICE")  // Service namn från Eureka
public interface ProductClient {
    
    @GetMapping("/products/{id}")
    Product getProduct(@PathVariable Long id);
    
    @GetMapping("/products")
    List<Product> getAllProducts();
    
    @PostMapping("/products")
    Product createProduct(@RequestBody ProductRequest request);
}
```

**Inventory Client:**

```java
@FeignClient(name = "INVENTORY-SERVICE")
public interface InventoryClient {
    
    @PostMapping("/inventory/check")
    InventoryResponse checkAvailability(@RequestBody InventoryCheckRequest request);
    
    @PostMapping("/inventory/reserve")
    void reserveStock(@RequestBody ReserveStockRequest request);
    
    @PostMapping("/inventory/release")
    void releaseStock(@RequestBody ReleaseStockRequest request);
}
```

**Payment Client:**

```java
@FeignClient(name = "PAYMENT-SERVICE")
public interface PaymentClient {
    
    @PostMapping("/payments")
    PaymentResponse processPayment(@RequestBody PaymentRequest request);
}
```

**Använd i service:**

```java
@Service
public class OrderService {
    
    @Autowired
    private ProductClient productClient;  // Inject som vanlig bean!
    
    @Autowired
    private InventoryClient inventoryClient;
    
    @Autowired
    private PaymentClient paymentClient;
    
    @Autowired
    private OrderRepository orderRepository;
    
    public Order createOrder(OrderRequest request) {
        // Mycket renare kod!
        Product product = productClient.getProduct(request.getProductId());
        
        InventoryResponse inventory = inventoryClient.checkAvailability(
            new InventoryCheckRequest(request.getProductId(), request.getQuantity())
        );
        
        if (!inventory.isAvailable()) {
            throw new InsufficientStockException();
        }
        
        Order order = new Order();
        // ... sätt fields
        order = orderRepository.save(order);
        
        inventoryClient.reserveStock(
            new ReserveStockRequest(order.getId(), request.getProductId(), request.getQuantity())
        );
        
        PaymentResponse payment = paymentClient.processPayment(
            new PaymentRequest(order.getId(), order.getTotalPrice())
        );
        
        if (payment.isSuccess()) {
            order.setStatus(OrderStatus.CONFIRMED);
            orderRepository.save(order);
        } else {
            inventoryClient.releaseStock(
                new ReleaseStockRequest(request.getProductId(), request.getQuantity())
            );
            throw new PaymentFailedException();
        }
        
        return order;
    }
}
```

Mycket cleanare! Feign hanterar:

- Service discovery
- Load balancing
- Serialization/deserialization
- Error handling (kan konfigurera)

## Resilience patterns

När du har många services som pratar med varandra kommer saker gå fel. Services kraschar, nätverk är långsamt, timeouts händer.

Du behöver **resilience patterns** för att hantera det.

### Circuit Breaker

**Problemet:** En service är nere, men andra services fortsätter anropa den → massa timeout errors → hela systemet blir långsamt.

**Lösningen:** Circuit Breaker - sluta anropa service som är nere!

**Tre states:**

```
CLOSED (normal)
  ↓ (för många fel)
OPEN (ingen trafik till service)
  ↓ (efter timeout)
HALF_OPEN (testa om service är uppe igen)
  ↓ (success)
CLOSED
```

**Diagram:**

```
CLOSED STATE:
Requests går genom normalt
[Service A] --✓--> [Service B]
[Service A] --✓--> [Service B]
[Service A] --✗--> [Service B]  (fel räknas)
[Service A] --✗--> [Service B]  (fel räknas)
[Service A] --✗--> [Service B]  (för många fel!)
         ↓
    OPEN STATE

OPEN STATE:
Requests går inte genom, failar direkt
[Service A] --✗ CIRCUIT OPEN --> [Service B]
[Service A] --✗ CIRCUIT OPEN --> [Service B]
     (efter 60 sekunder)
         ↓
    HALF_OPEN STATE

HALF_OPEN STATE:
Testa om service är uppe
[Service A] --✓--> [Service B]  (funkar!)
         ↓
    CLOSED STATE (back to normal)
```

**Spring Cloud Circuit Breaker med Resilience4j:**

```java
@Service
public class OrderService {
    
    @Autowired
    private ProductClient productClient;
    
    @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
    public Product getProduct(Long productId) {
        return productClient.getProduct(productId);
    }
    
    // Fallback method - körs när circuit är open
    public Product getProductFallback(Long productId, Exception e) {
        // Returnera cached data eller default values
        log.warn("Product service unavailable, returning fallback for product: {}", productId);
        
        Product fallback = new Product();
        fallback.setId(productId);
        fallback.setName("Product Unavailable");
        fallback.setPrice(BigDecimal.ZERO);
        return fallback;
    }
}
```

**Konfiguration:**

```yaml
resilience4j:
  circuitbreaker:
    instances:
      productService:
        register-health-indicator: true
        sliding-window-size: 10 # Kolla senaste 10 requests
        failure-rate-threshold: 50 # 50% fel → öppna circuit
        wait-duration-in-open-state: 60s # Vänta 60s innan half-open
        permitted-number-of-calls-in-half-open-state: 3
        automatic-transition-from-open-to-half-open-enabled: true
```

### Retry

Ibland är fel temporära (network glitch). Retry automatiskt!

```java
@Service
public class OrderService {
    
    @Retry(name = "productService", fallbackMethod = "getProductFallback")
    public Product getProduct(Long productId) {
        return productClient.getProduct(productId);
    }
}
```

**Konfiguration:**

```yaml
resilience4j:
  retry:
    instances:
      productService:
        max-attempts: 3 # Försök max 3 gånger
        wait-duration: 1s # Vänta 1s mellan retries
        retry-exceptions:
          - java.net.ConnectException
          - java.net.SocketTimeoutException
```

**Varning:** Var försiktig med retries!

- Inte idempotenta operationer (t.ex. skapa order - kan skapa dubbla!)
- Kan göra problemet värre om service är överbelastad

### Timeout

Vänta inte för evigt!

```java
@Service
public class OrderService {
    
    @TimeLimiter(name = "productService")
    public CompletableFuture<Product> getProduct(Long productId) {
        return CompletableFuture.supplyAsync(() -> 
            productClient.getProduct(productId)
        );
    }
}
```

```yaml
resilience4j:
  timelimiter:
    instances:
      productService:
        timeout-duration: 2s # Max 2 sekunder
```

### Fallback

Vad gör du när allt felar?

**Options:**

1. **Cached data**

```java
public Product getProductFallback(Long productId, Exception e) {
    return cache.get("product:" + productId);
}
```

2. **Default values**

```java
public List<Product> getRecommendationsFallback(Exception e) {
    return getDefaultRecommendations(); // Populära produkter
}
```

3. **Graceful degradation**

```java
public OrderDetails getOrderDetails(Long orderId) {
    Order order = orderRepository.findById(orderId);
    
    try {
        User user = userClient.getUser(order.getUserId());
        order.setUserName(user.getName());
    } catch (Exception e) {
        // User service nere, skippa user details
        order.setUserName("User information unavailable");
    }
    
    return order;
}
```

### Kombinera patterns

Ofta använder du flera tillsammans:

```java
@CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
@Retry(name = "productService")
@TimeLimiter(name = "productService")
public CompletableFuture<Product> getProduct(Long productId) {
    return CompletableFuture.supplyAsync(() -> 
        productClient.getProduct(productId)
    );
}
```

**Flow:**

1. Timeout efter 2s
2. Om fel, retry 3 gånger
3. Om för många fel, öppna circuit
4. Om circuit open eller allt felar, kör fallback

## Monitoring och observability

Med 20 services är det mycket svårare att veta vad som händer än med en monolith!

### Utmaningen

```
User rapporterar: "Köp funkar inte!"

Var är problemet?
- API Gateway? ✓ Fungerar
- Order Service? ✓ Fungerar
- Product Service? ✓ Fungerar
- Inventory Service? ✓ Fungerar
- Payment Service? ✗ AHA! Timeout här!
```

Du behöver kunna tracka en request genom hela systemet!

### Log aggregation

**Problem:** Logs finns på 20 olika servrar.

**Lösning:** Samla alla logs på ett ställe!

**ELK Stack:**

- **Elasticsearch:** Lagrar och indexar logs
- **Logstash:** Samlar och processar logs
- **Kibana:** Visualisering och sökning

```
[Service 1] ──┐
[Service 2] ──┤
[Service 3] ──┼──→ [Logstash] ──→ [Elasticsearch] ──→ [Kibana]
[Service 4] ──┤
[Service 5] ──┘
```

**I Kibana kan du:**

```
Sök: "error" AND "order-service" AND timestamp:[now-1h TO now]
→ Alla errors från Order Service senaste timmen
```

### Distributed tracing

**Problem:** En request går genom 5 services. Vilken är långsam?

**Lösning:** Distributed tracing - följ requesten genom hela flödet!

**Verktyg:** Zipkin eller Jaeger

**Hur det funkar:**

```
Request ID: abc123

[API Gateway]       (10ms) ──────┐
[Order Service]     (50ms)       │
[Product Service]   (200ms)  ← SLOW!
[Inventory Service] (30ms)       │
[Payment Service]   (100ms) ─────┘
                    Total: 390ms
```

**Spring Cloud Sleuth:**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

```yaml
spring:
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1.0 # Tracka 100% av requests (i prod: 0.1 = 10%)
```

Sleuth lägger automatiskt till trace IDs i alla logs:

```
2024-10-18 [order-service,abc123,xyz789] Order created: 42
2024-10-18 [product-service,abc123,def456] Product fetched: 123
2024-10-18 [payment-service,abc123,ghi789] Payment processed
```

Samma **trace ID** (abc123) för hela requesten!

### Metrics

**Vad ska du monitorera?**

**Service metrics:**

- CPU/Memory usage
- Request rate (requests/second)
- Error rate (%)
- Response time (percentiles: p50, p95, p99)

**Business metrics:**

- Orders per minute
- Revenue
- Active users

**Infrastructure metrics:**

- Database connections
- Queue length
- Disk usage

**Verktyg:** Prometheus + Grafana

**Prometheus:** Samlar metrics från services
**Grafana:** Visualiserar metrics i dashboards

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

Nu exponerar varje service metrics på: `http://localhost:8080/actuator/prometheus`

Prometheus scrape:ar dem regelbundet och Grafana visualiserar!

### Health checks

Varje service bör ha en health endpoint:

```java
@RestController
public class HealthController {
    
    @Autowired
    private DataSource dataSource;
    
    @GetMapping("/health")
    public ResponseEntity<HealthStatus> health() {
        // Kolla om service är healthy
        boolean dbHealthy = checkDatabase();
        boolean dependenciesHealthy = checkDependencies();
        
        if (dbHealthy && dependenciesHealthy) {
            return ResponseEntity.ok(new HealthStatus("UP"));
        } else {
            return ResponseEntity.status(503)
                .body(new HealthStatus("DOWN"));
        }
    }
    
    private boolean checkDatabase() {
        try {
            dataSource.getConnection().close();
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```

Spring Boot Actuator ger detta gratis:

```
GET /actuator/health

{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    },
    "diskSpace": {
      "status": "UP"
    }
  }
}
```

Load balancers och orchestrators (Kubernetes) använder health checks för att veta om en instans är OK.

## Deployment strategies

### Independent deployment

En av de stora fördelarna: deploya services separat!

```
Måndag 10:00  - Deploy Order Service v2.0
Måndag 14:30  - Deploy Product Service v1.5
Tisdag 09:00  - Deploy User Service v3.1
```

Ingen koordinering behövs (idealt sett).

### API versioning

**Problem:** Du ändrar API för Product Service. Gamla klienter går sönder!

**Lösning:** Versiona ditt API!

**URL versioning:**

```java
// Version 1
@GetMapping("/api/v1/products/{id}")
public ProductV1 getProductV1(@PathVariable Long id) {
    return productService.getProductV1(id);
}

// Version 2 (med extra fields)
@GetMapping("/api/v2/products/{id}")
public ProductV2 getProductV2(@PathVariable Long id) {
    return productService.getProductV2(id);
}
```

Gamla klienter använder v1, nya använder v2!

**Header versioning:**

```java
@GetMapping(value = "/api/products/{id}", headers = "API-Version=1")
public ProductV1 getProductV1(@PathVariable Long id) {...}

@GetMapping(value = "/api/products/{id}", headers = "API-Version=2")
public ProductV2 getProductV2(@PathVariable Long id) {...}
```

### Blue-Green deployment

Ha två identiska miljöer:

```
Blue (production, v1.0)     Green (v2.0)
[Service 1-3]               [Service 1-3]
      ↑                           ↑
      |                           |
[Load Balancer] ← switch traffic här
      ↑
  All traffic
```

**Process:**

1. Deploy v2.0 till Green
2. Testa Green
3. Switch load balancer till Green
4. Om problem, switch tillbaka till Blue (instant rollback!)

### Canary deployment

Rulla ut gradvis:

```
v1.0: 90% traffic
v2.0: 10% traffic (canary)

Om inga problem:
v1.0: 50% traffic
v2.0: 50% traffic

Om fortfarande OK:
v1.0: 0% traffic
v2.0: 100% traffic
```

Minskar risk - problem påverkar bara 10% av users först!

## Testing strategies

Testing är svårare med microservices!

### Unit tests

**Samma som vanligt:** Testa business logic i isolation.

```java
@Test
public void testCalculateOrderTotal() {
    Order order = new Order();
    order.setQuantity(3);
    order.setUnitPrice(BigDecimal.valueOf(100));
    
    BigDecimal total = orderService.calculateTotal(order);
    
    assertEquals(BigDecimal.valueOf(300), total);
}
```

### Integration tests

Testa med riktiga dependencies (databas, etc.)

**Testcontainers är perfekt:**

```java
@SpringBootTest
@Testcontainers
public class OrderServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:14");
    
    @Autowired
    private OrderService orderService;
    
    @Test
    public void testCreateOrder() {
        OrderRequest request = new OrderRequest(1L, 2);
        Order order = orderService.createOrder(request);
        
        assertNotNull(order.getId());
        assertEquals(OrderStatus.PENDING, order.getStatus());
    }
}
```

### Contract testing

**Problem:** Order Service förväntar sig `{id, name, price}` från Product Service, men Product Service returnerar `{productId, title, cost}`. Integration går sönder!

**Lösning:** Contract testing - verifiera att services håller överenskommelse.

**Pact eller Spring Cloud Contract:**

```java
// Producer side (Product Service)
@Test
public void testProductContract() {
    // Verifiera att vi returnerar vad vi lovar
}

// Consumer side (Order Service)
@Test
public void testProductServiceContract() {
    // Mock Product Service enligt contract
}
```

### End-to-end tests

Testa hela flödet:

```java
@Test
public void testCompleteOrderFlow() {
    // 1. Skapa user
    User user = createUser();
    
    // 2. Login
    String token = login(user);
    
    // 3. Lägg order
    Order order = placeOrder(token, productId, quantity);
    
    // 4. Verifiera order status
    assertEquals(OrderStatus.CONFIRMED, order.getStatus());
    
    // 5. Verifiera lager minskade
    Product product = getProduct(productId);
    assertEquals(originalStock - quantity, product.getStock());
    
    // 6. Verifiera email skickades
    assertTrue(emailWasSent(user.getEmail()));
}
```

**Problem:** Långsamma, dyra, flaky.
**Tips:** Gör färre E2E tests, fokusera på kritiska flows.

## Migration från monolith

Du har bestämt dig för att gå från monolith till microservices. HUR gör du det?

### Strangler Fig pattern

**INTE rewrite allt på en gång!** Det är dömt att misslyckas.

**Istället:** "Stryp" monoliten gradvis.

```
Steg 1:
[Monolith]  ← alla requests

Steg 2:
[Proxy] ──┬──→ [Monolith] (det mesta)
          └──→ [New Service A] (user auth)

Steg 3:
[Proxy] ──┬──→ [Monolith] (mindre)
          ├──→ [Service A] (user auth)
          └──→ [Service B] (products)

Steg 4:
[Proxy] ──┬──→ [Service A]
          ├──→ [Service B]
          ├──→ [Service C]
          └──→ [Monolith] (nästan tom)

Steg 5:
[Gateway] ──┬──→ [Service A]
            ├──→ [Service B]
            ├──→ [Service C]
            └──→ [Service D]
(monolith pensionerad!)
```

### Migrationssteg

**1. Identifiera en bounded context**

Hitta en del som:

- Har tydliga gränser
- Inte är för tight kopplad
- Ger värde att extrahera (skalning, etc.)

**Exempel:** User authentication

**2. Bygg ny microservice**

Skapa en ny Spring Boot app med samma funktionalitet.

```java
// Ny User Service
@RestController
@RequestMapping("/users")
public class UserController {
    
    @PostMapping("/login")
    public LoginResponse login(@RequestBody LoginRequest request) {
        // Samma logik som i monolith
    }
}
```

**3. Setup dual writes (temporary)**

Under migration, skriv till BÅDE monolith och service:

```java
// I monolith
public User createUser(UserRequest request) {
    User user = userRepository.save(request);
    
    // Skriv också till nya service
    try {
        userServiceClient.createUser(request);
    } catch (Exception e) {
        log.error("Failed to sync to user service", e);
    }
    
    return user;
}
```

**4. Route trafik gradvis**

```
Vecka 1: 10% traffic → User Service
Vecka 2: 50% traffic → User Service
Vecka 3: 100% traffic → User Service
```

**5. Migrera data (if needed)**

Om User Service har egen databas, migrera data från monolith.

**6. Ta bort från monolith**

När User Service är stabil, ta bort user-kod från monolith.

**7. Repetera för nästa service!**

### Vilken service först?

**Bra första kandidater:**

- **Isolerade features:** Färre dependencies
- **Hög load:** Kan dra nytta av separat skalning
- **Snabb utvecklingshastighet:** Vill deployas ofta

**Dåliga första kandidater:**

- **Core business logic:** För riskfyllt
- **Tight kopplad:** Svårt att separera
- **Stabila features:** Behöver inte ofta ändringar

**Exempel ordning för e-handel:**

```
1. User Auth Service (isolerad, kritisk för säkerhet)
2. Product Service (hög read load, kan skala separat)
3. Notification Service (asynkron, lätt att separera)
4. Order Service (core logic, gör sist!)
```

## Best practices

Sammanfattning av best practices för microservices:

### Design

1. **Start med monolith** - dela upp senare när det behövs
2. **Service boundaries baserade på business capabilities** - inte tekniska lager
3. **Database per service** - äg din data
4. **Design för failure** - services kommer gå ner
5. **Loose coupling, high cohesion** - minimera dependencies

### Communication

6. **Använd async när möjligt** - events/messages istället för sync calls
7. **API Gateway för klienter** - single entry point
8. **Service discovery** - hårdkoda inte IP-adresser
9. **Versiona APIs** - backward compatibility
10. **Timeout alltid** - vänta inte för evigt

### Resilience

11. **Circuit breakers** - förhindra cascading failures
12. **Retry med exponential backoff** - men var försiktig
13. **Fallbacks** - vad händer när allt felar?
14. **Health checks** - låt orchestrator veta om service är OK

### Operations

15. **Centralized logging** - ELK stack eller liknande
16. **Distributed tracing** - följ requests genom systemet
17. **Metrics överallt** - Prometheus + Grafana
18. **Automate everything** - CI/CD pipelines
19. **Infrastructure as Code** - Terraform, CloudFormation
20. **Container orchestration** - Kubernetes för production

### Data

21. **Eventual consistency är OK** - perfekt consistency är svårt
22. **Saga pattern för distributed transactions**
23. **Event sourcing för audit trail**
24. **CQRS när läs/skriv patterns skiljer sig**

### Documentation

25. **OpenAPI/Swagger för alla APIs**
26. **Architecture Decision Records (ADRs)**
27. **Service README** - hur kör jag denna service?
28. **Runbooks** - hur debuggar jag problem?

### Team

29. **Teams äger services end-to-end** - "you build it, you run it"
30. **DevOps culture** - developers ansvarar för drift

## Common mistakes

Saker att UNDVIKA:

### 1. Premature microservices

```
❌ "Vi ska bygga microservices från dag 1!"
✅ "Vi börjar med monolith och delar upp om det behövs"
```

### 2. Fel service boundaries

```
❌ CreateUserService, UpdateUserService, DeleteUserService
✅ UserService (en service per business capability)
```

### 3. Shared database

```
❌ Order Service + Product Service → Samma databas
✅ Order Service → Order DB, Product Service → Product DB
```

### 4. Distributed monolith

Microservices i namn, men tight kopplad i praktiken:

```
❌ Alla services måste deployras tillsammans
❌ Services anropar varandra i långa kedjor
❌ Ändringar kräver koordinering mellan alla teams
```

Det är sämsta av båda världar - complexity utan benefits!

### 5. Ignorera operational complexity

```
❌ "Vi sätter upp microservices, operations får fixa resten"
✅ Planera för monitoring, logging, deployment från start
```

### 6. Ingen proper monitoring

```
❌ Logs på 20 olika servrar, ingen aggregering
✅ ELK stack + distributed tracing + metrics
```

### 7. Sync calls överallt

```
❌ Service A → sync → Service B → sync → Service C → sync → Service D
   (Om D är nere, går allt ner)
   
✅ Service A → Message Queue → Services B, C, D consume asynkront
```

### 8. Ignorera network failures

```
❌ Inga timeouts, retries, eller circuit breakers
✅ Resilience patterns överallt
```

### 9. Ingen API versioning

```
❌ Ändrar API, gamla klienter går sönder
✅ /api/v1 och /api/v2 lever side-by-side
```

### 10. Försumma testing

```
❌ "Integration testing är för svårt, vi skippar det"
✅ Contract testing + testcontainers + E2E för kritiska flows
```

## Microservices och Docker/Kubernetes

Microservices och containers är en perfekt match!

### Varför Docker?

Varje service är en egen app med egna dependencies:

```
User Service:     Java 17 + Spring Boot 3.0 + PostgreSQL driver
Product Service:  Java 11 + Spring Boot 2.7 + Elasticsearch client
Payment Service:  Node.js 18 + Express + MongoDB driver
```

**Med Docker:** Varje service får sitt eget isolerade environment!

```dockerfile
# Product Service Dockerfile
FROM openjdk:17-slim
COPY target/product-service.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### Varför Kubernetes?

När du har 20 services som Docker containers behöver du **orchestration**:

- Starta containers
- Restart om de kraschar
- Skala upp/ner automatiskt
- Load balance mellan instanser
- Service discovery
- Rolling updates

**Kubernetes gör allt detta!**

```yaml
# product-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 3 # 3 instanser
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
    spec:
      containers:
        - name: product-service
          image: myregistry/product-service:v1.0
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: production
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
---
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product-service
  ports:
    - port: 8080
      targetPort: 8080
  type: ClusterIP # Intern load balancer
```

Kubernetes hanterar:

- **Auto-scaling:** Skala baserat på CPU/minne
- **Self-healing:** Restart containers som kraschar
- **Load balancing:** Fördela trafik mellan pods
- **Service discovery:** DNS för services automatiskt
- **Rolling updates:** Uppdatera utan downtime

Vi går mer in på Kubernetes i ett dedikerat dokument!

## Sammanfattning

**Microservices** är ett arkitekturmönster där du bygger applikationen som en samling av små, oberoende services.

### Fördelar:

- ✅ Oberoende skalning per service
- ✅ Oberoende deployment
- ✅ Olika teknologier per service
- ✅ Team autonomi
- ✅ Fault isolation
- ✅ Mindre, lättare att förstå kodbaser

### Nackdelar:

- ❌ Ökad komplexitet (många moving parts)
- ❌ Network latency
- ❌ Data consistency utmaningar
- ❌ Svårare testing
- ❌ Mer operational overhead
- ❌ Inte för små projekt

### Använd microservices när:

- Stor, komplex applikation
- Flera team
- Olika skalningsbehov
- Snabba, frekventa deployments
- DevOps mognad finns

### ANVÄND INTE när:

- Liten app eller team
- Precis börjat projektet
- Ingen ops-erfarenhet
- Oklara service boundaries

### Key patterns:

- **API Gateway:** Single entry point
- **Service Discovery:** Services hittar varandra
- **Circuit Breaker:** Förhindra cascading failures
- **Database per Service:** Varje service äger sin data
- **Saga Pattern:** Distribuerade transactions
- **Event-Driven:** Async kommunikation

### Tech stack:

- **Spring Boot** för services
- **Spring Cloud** för microservices patterns
- **Eureka/Consul** för service discovery
- **RabbitMQ/Kafka** för messaging
- **Docker** för containers
- **Kubernetes** för orchestration
- **ELK** för logging
- **Prometheus + Grafana** för metrics
- **Zipkin/Jaeger** för distributed tracing

### Viktigaste takeaway:

> **Start med en välstrukturerad monolith. Refactorera till microservices när du har reala problem som microservices löser.**

Microservices är inte gratis. De kostar komplexitet. Se till att du får tillräckligt värde för den kostnaden!

**Nästa:** Message brokers och event-driven architecture för asynkron kommunikation mellan services!

## Övningsuppgifter

### Uppgift 1: Designa microservices arkitektur

Du ska bygga en **biblioteks-app** där användare kan:

- Bläddra och söka böcker
- Låna böcker (max 3 samtidigt)
- Returnera böcker
- Se sin lånehistorik
- Få påminnelser om att returnera böcker

**Din uppgift:**

1. Dela upp systemet i microservices
2. Rita ett diagram som visar services och deras ansvar
3. Visa vilka services som kommunicerar med varandra
4. Förklara varför du valde just dessa gränser

**Extra:** Vilka databaser skulle varje service ha?

### Uppgift 2: Database per service

**Scenario:** Du har en monolith e-handel med denna SQL-query:

```sql
SELECT 
    o.order_id,
    o.total,
    u.name AS user_name,
    u.email,
    p.name AS product_name,
    p.price
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN products p ON o.product_id = p.id
WHERE o.user_id = 123
```

Nu ska du migrera till microservices där:

- Order Service har Orders DB
- User Service har Users DB
- Product Service har Products DB

**Din uppgift:**

1. Hur får du samma data utan att kunna göra denna join?
2. Vilka API calls behöver Order Service göra?
3. Skriv pseudo-kod för hur Order Service hämtar order-details

**Bonus:** Vilken data skulle du duplicera i Order Service för att slippa API calls?

### Uppgift 3: Circuit breaker scenario

**Scenario:** Din Order Service anropar Payment Service för att processa betalningar. Payment Service är nere i 5 minuter.

**Utan Circuit Breaker:**

- Varje order-request väntar 30 sekunder (timeout)
- 1000 requests kommer in
- Allt blir fruktansvärt långsamt

**Din uppgift:**

1. Förklara hur Circuit Breaker skulle hjälpa i detta scenario
2. Vilka settings skulle du använda? (failure threshold, wait time, etc.)
3. Vad skulle fallback-metoden returnera till användaren?
4. Rita ett diagram som visar circuit breaker states

### Uppgift 4: Synchronous vs Asynchronous

För varje scenario, bestäm om du skulle använda **synchronous** (REST) eller **asynchronous** (message queue) kommunikation:

1. Order Service behöver kolla produktpris innan order läggs
2. Email ska skickas när order är lagd
3. Analytics Service ska logga alla orders
4. Order Service behöver verifiera att användare är inloggad
5. Inventory Service ska minska lagersaldo när order läggs
6. Recommendation Service ska uppdatera "frequently bought together"

**För varje:** Förklara VARFÖR du valde sync eller async.

### Uppgift 5: Ska du använda microservices?

**Scenario A:**

- Team: Du och en kompis
- App: Todo-app med users, tasks, och reminders
- Trafik: 100 användare

**Fråga:** Ska ni använda microservices? Varför/varför inte?

---

**Scenario B:**

- Team: 50 utvecklare i 5 team
- App: Social media platform (posts, comments, likes, messages, notifications, search)
- Trafik: 1 miljon användare
- Problem: Deployments tar 2 timmar, search feature behöver skala separat

**Fråga:** Ska ni använda microservices? Varför/varför inte? Vilka services skulle du skapa?

### Uppgift 6: Rita arkitektur diagram

Rita ett fullständigt diagram för en **food delivery app** med microservices:

**Features:**

- Användare beställer mat från restauranger
- Restauranger accepterar/decline orders
- Delivery drivers får orders och levererar
- Betalningar processas
- Push notifikationer skickas
- Tracking av delivery i realtid

**Ditt diagram ska visa:**

1. Alla services
2. API Gateway
3. Service Discovery
4. Databaser
5. Message Queue
6. Vilka kommunikationer som är sync vs async
7. Var Circuit Breakers behövs

**Bonus:** Vilka services skulle skala mest? Varför?

---

**Lycka till! 🚀** Microservices är komplext men kraftfullt när det används rätt!
