# Message Brokers

## Introduktion

Ibland behöver inte saker hända direkt. Tänk dig att du beställer mat på en restaurang - behöver du stå och vänta i köket tills maten är klar? Nej, du får en nummerlapp och kan gå tillbaka till ditt bord, kanske scrolla lite på mobilen. När maten är klar ropar de upp ditt nummer.

Det är precis så **message brokers** fungerar i mjukvaruvärlden. De är som en brevlåda - du lämnar ett meddelande, och någon annan hämtar det senare när de är redo. Du behöver inte vänta på att mottagaren läser det direkt.

### Vad är en message broker?

En **message broker** är en mellanhand som tar emot meddelanden från en tjänst (producent) och levererar dem till en annan tjänst (konsument). Det är som ett postkontor för dina microservices.

```
[Service A] --meddelande--> [Message Broker] --meddelande--> [Service B]
```

Service A skickar ett meddelande och fortsätter med sitt liv. Message brokern ser till att meddelandet kommer fram till Service B när den är redo att ta emot det.

### Varför är detta bra?

Tänk dig att du bygger en e-handelssajt. När en användare lägger en order behöver massa saker hända:

- Skicka bekräftelsemejl
- Uppdatera lagersaldo
- Meddela betalningssystemet
- Skicka till logistikfirman
- Uppdatera analytics

Ska checkout-processen vänta på att ALLA dessa saker är klara? Nej! Användaren vill ha en snabb bekräftelse. Resten kan hända i bakgrunden.

### Producer/Consumer modellen

Det finns två huvudroller:

**Producer (Producent)** - skickar meddelanden

- "Jag har en order som behöver processas"
- "En ny användare har registrerat sig"
- "Denna bild behöver storleksändras"

**Consumer (Konsument)** - tar emot och processar meddelanden

- Läser meddelanden från brokern
- Utför jobbet
- Bekräftar när det är klart

En tjänst kan vara både producer och consumer!

## Synkron vs Asynkron kommunikation

Det här är en superviktig distinktion att förstå.

### Synkron kommunikation

**Synkront** betyder att du väntar på svar direkt. Som att ringa någon - du pratar inte vidare förrän de svarar.

```
[Service A] --request--> [Service B]
     |                        |
     |<------response---------|
     |
   Väntar här!
```

**Exempel:**

```java
// Service A anropar Service B synkront
String result = restTemplate.getForObject("http://service-b/api/data", String.class);
// Service A väntar här tills Service B svarar
System.out.println("Got result: " + result);
```

**Fördelar:**

- ✅ Du får svar direkt
- ✅ Enklare att förstå och debugga
- ✅ Bra när du BEHÖVER svaret för att fortsätta

**Nackdelar:**

- ❌ Tight coupling - Service A och B måste båda vara uppe
- ❌ Om Service B är långsam blir Service A långsam
- ❌ Om Service B kraschar misslyckas Service A
- ❌ Svårare att skala - alla måste vara tillgängliga samtidigt

### Asynkron kommunikation

**Asynkront** betyder att du skickar ett meddelande och fortsätter med ditt liv. Som att skicka ett mejl - du väntar inte på svar innan du gör annat.

```
[Service A] --meddelande--> [Message Broker]
     |                             |
   Fortsätter direkt!               |
                                    v
                              [Service B] (processar när den är redo)
```

**Exempel:**

```java
// Service A skickar meddelande asynkront
rabbitTemplate.convertAndSend("orders", orderMessage);
// Service A fortsätter direkt, behöver inte vänta
System.out.println("Order skickad för bearbetning!");
```

**Fördelar:**

- ✅ Loose coupling - tjänsterna behöver inte veta om varandra
- ✅ Fault tolerant - om en service är nere buffras meddelanden
- ✅ Bättre skalbarhet - producers och consumers oberoende
- ✅ Bättre prestanda - väntar inte på svar

**Nackdelar:**

- ❌ Du får inget svar direkt
- ❌ Mer komplext att sätta upp
- ❌ Svårare att debugga - vad hände med meddelandet?
- ❌ Eventual consistency - saker händer "så småningom"

### När ska man använda varje typ?

**Använd synkron** när:

- Du MÅSTE ha ett svar direkt för att fortsätta
- Exempelvis: "Finns denna produkt i lager?" (behöver veta innan checkout)
- "Är detta lösenord korrekt?" (kan inte logga in utan svar)
- "Vad kostar frakten?" (användaren behöver se priset)

**Använd asynkron** när:

- Saker kan hända senare
- Exempelvis: "Skicka ett bekräftelsemejl" (kan ta några sekunder)
- "Generera en PDF-rapport" (kan ta tid)
- "Processa denna video" (tar lång tid)
- "Uppdatera sökindex" (behöver inte ske direkt)

**Tumregel:** Om användaren inte behöver vänta på resultatet - gör det asynkront!

## Varför message brokers?

### 1. Decoupling (Lös koppling)

Utan message broker:

```
[Order Service] --HTTP--> [Email Service]
                --HTTP--> [Inventory Service]
                --HTTP--> [Analytics Service]
```

Order Service måste veta om alla andra tjänster. Om en ny tjänst behöver veta om orders måste vi ändra Order Service.

Med message broker:

```
[Order Service] --> [Message Broker] --> [Email Service]
                                     --> [Inventory Service]
                                     --> [Analytics Service]
                                     --> [New Service] (lägg bara till!)
```

Order Service bryr sig bara om att skicka ett meddelande. Vem som lyssnar är inte dess problem!

### 2. Buffert mot trafikstoppar

Säg att du får 1000 ordrar per sekund under Black Friday. Utan message broker måste alla systems processa alla ordrar direkt - risk för krasch!

Med message broker:

```
1000 orders/sec --> [Queue] --> Processas i lugn takt (kanske 100/sec)
```

Kön växer under rusningstid, men minskar sen när trafiken avtar. Inget kraschar!

### 3. Reliability (Tillförlitlighet)

Om Email Service är nere just nu - vad händer?

**Utan message broker:** Order misslyckas eller mejlet försvinner.

**Med message broker:** Meddelandet ligger kvar i kön tills Email Service är uppe igen. Inget förloras!

### 4. Load distribution

Med flera consumers kan message brokern fördela arbetet:

```
[Producer] --> [Queue] --> [Consumer 1] (processar meddelande 1, 4, 7...)
                       --> [Consumer 2] (processar meddelande 2, 5, 8...)
                       --> [Consumer 3] (processar meddelande 3, 6, 9...)
```

Automatisk lastbalansering!

### 5. Retry-logik inbyggd

Om ett meddelande misslyckas kan brokern automatiskt försöka igen. Du behöver inte bygga egen retry-logik.

### 6. Audit trail

Alla meddelanden loggas. Du kan se exakt vad som skickades och när. Perfekt för debugging och compliance.

### Verkliga användningsfall

**E-handel:**

- Order lagd → skicka mejl, uppdatera lager, starta leverans, uppdatera analytics

**Social media:**

- Post publicerad → skicka notiser till followers, uppdatera feeds, indexera för sökning

**Bildhantering:**

- Bild uppladdad → skapa thumbnails, komprimera, extrahera metadata, kör AI-analys

**Banking:**

- Transaktion genomförd → skicka bekräftelse, uppdatera saldo, kör fraud detection, logga för revision

**IoT:**

- Sensor data → spara i databas, kontrollera tröskelvärden, uppdatera dashboard

## Queue vs Topic (Pub/Sub)

Det finns två huvudsakliga mönster för hur meddelanden distribueras.

### Queue (Point-to-Point)

En **queue** (kö) fungerar som en arbetskö. Ett meddelande går till EXAKT EN consumer.

```
[Producer] --> [Queue] --> [Consumer 1]  ✓ Fick meddelandet
                       --> [Consumer 2]  (får nästa meddelande)
                       --> [Consumer 3]  (får meddelandet efter det)
```

**Hur det fungerar:**

- Flera consumers kan lyssna på samma queue
- Men varje meddelande konsumeras av bara EN av dem
- Bra för load balancing - fördela arbete mellan workers

**Exempel: Bildprocessering**

```
[Upload Service] --> [image-queue] --> [Worker 1] (processar bild 1)
                                   --> [Worker 2] (processar bild 2)
                                   --> [Worker 3] (processar bild 3)
```

Varje bild processas av exakt en worker. Perfect för att dela upp arbetet!

**Användningsområden:**

- Background jobs (skicka mejl, generera rapporter)
- Task distribution (procesera bilder, videos)
- Kommandon som bara ska utföras en gång

### Topic (Publish/Subscribe)

Ett **topic** fungerar som en broadcasting-station. Ett meddelande går till ALLA consumers som prenumererar.

```
[Producer] --> [Topic] --> [Consumer 1]  ✓ Alla får
                       --> [Consumer 2]  ✓ en kopia
                       --> [Consumer 3]  ✓ av meddelandet
```

**Hur det fungerar:**

- Flera consumers kan prenumerera på samma topic
- Varje meddelande kopieras till ALLA subscribers
- Bra för events - flera system bryr sig om samma händelse

**Exempel: Användare registrerad**

```
[Auth Service] --> [user.registered] --> [Email Service] (skicka välkomstmejl)
                                     --> [Analytics] (logga ny användare)
                                     --> [Profile Service] (skapa profil)
```

Alla tre tjänsterna får meddelandet och gör sin grej!

**Användningsområden:**

- Events (användare skapad, order lagd, betalning genomförd)
- Notifikationer till flera system
- Real-time updates (aktiekurser, sportscore)
- Event-driven architecture

### När ska man använda vad?

**Använd Queue när:**

- Ett jobb ska utföras EXAKT en gång
- Du vill fördela arbete mellan workers
- Exempel: "Skicka detta mejl" (vill inte skicka 3 gånger!)

**Använd Topic när:**

- Flera tjänster bryr sig om samma händelse
- Varje tjänst gör något olika med informationen
- Exempel: "Användare registrerad" (många behöver veta!)

**Kan kombineras:**

```
[Order Service] --> [order.placed Topic] --> [email-queue] --> [Email Workers]
                                         --> [inventory-queue] --> [Inventory Workers]
                                         --> [analytics-queue] --> [Analytics Workers]
```

Topic för att broadcasta, sedan queues för att processa!

## RabbitMQ - introduktion

**RabbitMQ** är en av de mest populära message brokers. Den har funnits sedan 2007 och används av företag som Reddit, NASA, och Mercedes-Benz.

### Varför RabbitMQ?

- ✅ **Mogen och stabil** - beprövad i produktion i många år
- ✅ **Lätt att komma igång** - bra dokumentation och community
- ✅ **Management UI** - grafiskt interface för att se queues, messages
- ✅ **Många features** - stödjer både queues och topics, prioritering, retry, mm
- ✅ **AMQP-protokoll** - standardiserat protokoll
- ✅ **Multi-language** - clients för Java, Python, Node.js, Go, osv

### Nyckelkoncept i RabbitMQ

**Producer (Producent):**

- Applikationen som skickar meddelanden
- Vet inte vem som tar emot

**Queue (Kö):**

- Buffert som sparar meddelanden
- FIFO (First In, First Out) som standard
- Kan vara durable (överlever restart)

**Consumer (Konsument):**

- Applikationen som tar emot och processar meddelanden
- Bekräftar när meddelandet är processat

**Exchange:**

- Tar emot meddelanden från producers
- Routar dem till rätt queue(s)
- Olika typer bestämmer routing-logik

**Binding:**

- Regler som kopplar exchange till queues
- "Meddelanden med routing key 'error' går till error-queue"

### Exchange-typer

RabbitMQ har olika sätt att routa meddelanden:

**1. Direct Exchange**
Routar baserat på exakt routing key.

```
[Producer] --key:"error"--> [Direct Exchange] --"error"--> [Error Queue]
                                              --"info"--> [Info Queue]
```

Perfekt för att skicka olika typer av meddelanden till olika queues.

**2. Fanout Exchange**
Broadcastar till ALLA queues. Ignorerar routing key.

```
[Producer] --> [Fanout Exchange] --> [Queue 1]
                                 --> [Queue 2]
                                 --> [Queue 3]
```

Klassisk pub/sub - alla får allt!

**3. Topic Exchange**
Routar baserat på pattern matching med wildcards.

```
Routing key: "user.created.sweden"
             "order.placed.denmark"
             "product.updated.norway"

Pattern "*.created.*" --> [New Entities Queue]
Pattern "order.*.*" --> [Order Queue]
Pattern "*.*.sweden" --> [Sweden Queue]
```

Superkraftig för att matcha komplexa routing-regler!

**4. Headers Exchange**
Routar baserat på message headers istället för routing key. Mindre vanligt.

### Hur det hänger ihop

```
[Producer] --publish--> [Exchange] --routing--> [Queue] --consume--> [Consumer]
```

1. Producer publicerar till en **exchange** (inte direkt till queue!)
2. Exchange använder **bindings** för att veta vilken queue meddelandet ska till
3. Meddelandet hamnar i **queue**
4. Consumer läser från queue

## Kafka - en snabb jämförelse

**Apache Kafka** är en annan populär message broker, men designad för olika användningsfall.

### Kafka i korthet

- **Distributed streaming platform** - designad för stora datamängder
- **Log-based** - meddelanden skrivs till disk som en append-only log
- **High throughput** - kan hantera miljontals meddelanden per sekund
- **Retention** - meddelanden sparas, kan läsas om och om igen
- **Partitioning** - data delas upp för parallell processing

### RabbitMQ vs Kafka - när ska man använda vad?

**Använd RabbitMQ när:**

- ✅ Du behöver traditionella work queues
- ✅ Du vill enkel setup och maintenance
- ✅ Du behöver flexibel routing (exchanges, patterns)
- ✅ Message acknowledgments är viktiga
- ✅ Du har moderate throughput (tusentals msg/sec)
- ✅ Messages ska konsumeras och försvinna

**Använd Kafka när:**

- ✅ Du behöver mycket hög throughput (miljoner msg/sec)
- ✅ Du vill kunna "replay" messages (läsa gamla meddelanden igen)
- ✅ Event sourcing - bygga state från events
- ✅ Stream processing - real-time analytics
- ✅ Du har flera consumers som vill läsa samma data
- ✅ Data pipeline mellan system

**Enkel jämförelse:**

| Aspekt            | RabbitMQ                | Kafka                         |
| ----------------- | ----------------------- | ----------------------------- |
| Typ               | Message Broker          | Streaming Platform            |
| Användningsfall   | Task queues, routing    | Event streaming, logs         |
| Komplexitet       | Lättare att sätta upp   | Mer komplex                   |
| Throughput        | Hög (tusentals/sec)     | Mycket hög (miljoner/sec)     |
| Message retention | Tas bort när konsumerad | Sparas en tid (replay möjlig) |
| Ordning           | Per queue               | Per partition (garanterad)    |

**Tumregel:** Börja med RabbitMQ. Det är enklare och räcker för de flesta use cases. Gå till Kafka om du verkligen behöver streaming eller extrem throughput.

## Message-struktur

Vad innehåller egentligen ett meddelande?

### Body/Payload

Detta är själva datan - det du faktiskt vill skicka. Oftast JSON.

```json
{
  "eventType": "ORDER_PLACED",
  "orderId": "12345",
  "userId": "user789",
  "items": [
    { "productId": "ABC", "quantity": 2, "price": 199.99 }
  ],
  "total": 399.98,
  "timestamp": "2025-10-18T10:30:00Z"
}
```

**Best practices för payload:**

- Håll det relativt litet (några KB, inte MB)
- Använd JSON för läsbarhet
- Inkludera timestamp
- Inkludera ID:n för att kunna spåra

### Headers/Metadata

Extra information om meddelandet - inte själva datan.

```java
MessageProperties props = new MessageProperties();
props.setContentType("application/json");
props.setPriority(5);
props.setTimestamp(new Date());
props.setMessageId(UUID.randomUUID().toString());
props.setHeader("source", "order-service");
```

**Vanliga headers:**

- `content-type` - vilket format? (application/json, text/plain)
- `priority` - hur viktigt är meddelandet?
- `timestamp` - när skickades det?
- `message-id` - unikt ID för spårning
- `correlation-id` - koppla samman relaterade meddelanden
- Custom headers - vad som helst du behöver!

### Routing Key

Används av exchange för att veta vart meddelandet ska.

```java
rabbitTemplate.convertAndSend(
    "orders-exchange",     // exchange
    "order.placed",        // routing key
    orderMessage           // payload
);
```

### Komplett exempel

```java
// Skapa meddelande
Order order = new Order("12345", "user789", 399.98);
String json = objectMapper.writeValueAsString(order);

// Sätt properties
MessageProperties props = new MessageProperties();
props.setContentType("application/json");
props.setDeliveryMode(MessageDeliveryMode.PERSISTENT); // Spara till disk
props.setHeader("source", "order-service");

Message message = new Message(json.getBytes(), props);

// Skicka
rabbitTemplate.send("orders-exchange", "order.placed", message);
```

## RabbitMQ i praktiken

Dags att faktiskt köra RabbitMQ!

### Starta RabbitMQ med Docker

Det enklaste sättet är med Docker:

```bash
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management
```

**Vad händer här?**

- `-d` - körs i bakgrunden
- `--name rabbitmq` - containern heter "rabbitmq"
- `-p 5672:5672` - AMQP-protokollet (här pratar dina appar)
- `-p 15672:15672` - Management UI (webgränssnitt)
- `rabbitmq:3-management` - imagen med management plugin

### Management UI

Öppna webbläsaren och gå till: `http://localhost:15672`

**Login:**

- Username: `guest`
- Password: `guest`

I UI:n kan du:

- ✅ Se alla queues och hur många meddelanden som väntar
- ✅ Se consumers och om de är aktiva
- ✅ Skicka testmeddelanden manuellt
- ✅ Se message rates (meddelanden per sekund)
- ✅ Se exchange-bindings
- ✅ Debugga när något inte funkar

**Tips:** Håll Management UI öppet när du utvecklar - invaluerbart för debugging!

## Spring Boot och RabbitMQ

Nu kopplar vi ihop Spring Boot med RabbitMQ!

### Dependencies

Lägg till i din `pom.xml`:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

Spring Boot's AMQP starter ger dig allt du behöver för att prata med RabbitMQ.

### Konfiguration

I `application.properties`:

```properties
# RabbitMQ connection
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

# Optional: connection pool settings
spring.rabbitmq.connection-timeout=30000
```

Om du kör i Docker Compose kan host vara `rabbitmq` istället för `localhost`.

### Auto-configuration

Spring Boot konfigurerar automatiskt:

- ✅ `ConnectionFactory` - anslutning till RabbitMQ
- ✅ `RabbitTemplate` - för att skicka meddelanden
- ✅ `RabbitAdmin` - för att skapa queues och exchanges

Du behöver bara börja använda dem!

## Producer - skicka meddelanden

Låt oss skapa en producer som skickar meddelanden.

### 1. Definiera queue

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue orderQueue() {
        return new Queue("orders", true); // true = durable (överlever restart)
    }
}
```

### 2. Skapa producer service

```java
@Service
public class OrderService {
    
    private final RabbitTemplate rabbitTemplate;
    
    @Autowired
    public OrderService(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }
    
    public void placeOrder(Order order) {
        // Spara order i databas
        System.out.println("Order sparad: " + order.getId());
        
        // Skicka meddelande för asynkron bearbetning
        rabbitTemplate.convertAndSend("orders", order);
        
        System.out.println("Order-meddelande skickat!");
        // Vi fortsätter direkt - väntar inte på att meddelandet processas
    }
}
```

**Vad händer här?**

- `rabbitTemplate.convertAndSend("orders", order)` - skickar order-objektet till "orders" queue
- Spring konverterar automatiskt order till JSON
- Metoden returnerar direkt - blockerar inte!

### 3. Order model

```java
public class Order {
    private String id;
    private String userId;
    private double total;
    private LocalDateTime timestamp;
    
    // Konstruktorer, getters, setters...
}
```

### 4. Controller

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @PostMapping
    public ResponseEntity<String> createOrder(@RequestBody Order order) {
        order.setId(UUID.randomUUID().toString());
        order.setTimestamp(LocalDateTime.now());
        
        orderService.placeOrder(order);
        
        // Returnerar direkt till användaren!
        return ResponseEntity.ok("Order mottagen och kommer processas!");
    }
}
```

Användaren får snabbt svar medan ordern processas i bakgrunden. Perfekt!

## Consumer - ta emot meddelanden

Nu ska vi skapa en consumer som processar ordrar.

### Enkel consumer

```java
@Component
public class OrderProcessor {
    
    @RabbitListener(queues = "orders")
    public void processOrder(Order order) {
        System.out.println("Processar order: " + order.getId());
        
        // Gör vad som behöver göras
        sendConfirmationEmail(order);
        updateInventory(order);
        notifyShipping(order);
        
        System.out.println("Order " + order.getId() + " klar!");
    }
    
    private void sendConfirmationEmail(Order order) {
        // Skicka mejl...
        System.out.println("Mejl skickat till användare");
    }
    
    private void updateInventory(Order order) {
        // Uppdatera lager...
        System.out.println("Lager uppdaterat");
    }
    
    private void notifyShipping(Order order) {
        // Meddela leveranstjänst...
        System.out.println("Leverans notifierad");
    }
}
```

**Magiskt enkelt!** Bara annotera med `@RabbitListener` och Spring hanterar resten:

- ✅ Lyssnar automatiskt på queue
- ✅ Konverterar JSON till Order-objekt
- ✅ Anropar metoden när meddelande kommer
- ✅ Hanterar acknowledgments

### Consumer med flera instanser

Du kan köra flera instanser av samma consumer för load balancing!

```
[Queue] --> [Consumer 1] (processar meddelande 1, 4, 7...)
        --> [Consumer 2] (processar meddelande 2, 5, 8...)
        --> [Consumer 3] (processar meddelande 3, 6, 9...)
```

Starta bara samma applikation flera gånger - RabbitMQ fördelar automatiskt!

### Error handling

Vad händer om processing misslyckas?

```java
@Component
public class OrderProcessor {
    
    @RabbitListener(queues = "orders")
    public void processOrder(Order order) {
        try {
            System.out.println("Processar order: " + order.getId());
            
            // Detta kan misslyckas
            riskyOperation(order);
            
            System.out.println("Order " + order.getId() + " klar!");
            
        } catch (Exception e) {
            System.err.println("Fel vid processing av order " + order.getId() + ": " + e.getMessage());
            
            // Kasta exception för att låta RabbitMQ hantera retry
            throw new AmqpRejectAndDontRequeueException("Processing misslyckades", e);
        }
    }
}
```

## Queue configuration - mer detaljer

Låt oss gå djupare på queue-konfiguration.

### Durable queue

En **durable** queue överlever när RabbitMQ startar om.

```java
@Bean
public Queue orderQueue() {
    return new Queue("orders", true); // true = durable
}
```

**Viktigt:** Även meddelanden måste vara persistenta för att överleva restart!

```java
rabbitTemplate.convertAndSend("orders", order, message -> {
    message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
    return message;
});
```

### Konfigurera exchange och binding

```java
@Configuration
public class RabbitConfig {
    
    // Queue
    @Bean
    public Queue orderQueue() {
        return new Queue("orders", true);
    }
    
    // Exchange
    @Bean
    public DirectExchange orderExchange() {
        return new DirectExchange("order-exchange");
    }
    
    // Binding - koppla queue till exchange
    @Bean
    public Binding orderBinding(Queue orderQueue, DirectExchange orderExchange) {
        return BindingBuilder
            .bind(orderQueue)
            .to(orderExchange)
            .with("order.placed"); // routing key
    }
}
```

Nu skickar vi till exchange istället:

```java
rabbitTemplate.convertAndSend("order-exchange", "order.placed", order);
```

### Arguments för advanced features

```java
@Bean
public Queue orderQueue() {
    Map<String, Object> args = new HashMap<>();
    
    // Dead letter exchange - dit misslyckade meddelanden går
    args.put("x-dead-letter-exchange", "dlx-exchange");
    
    // Message TTL - meddelanden lever max 1 timme
    args.put("x-message-ttl", 3600000);
    
    // Max längd - max 10000 meddelanden i kön
    args.put("x-max-length", 10000);
    
    return new Queue("orders", true, false, false, args);
}
```

## Message Acknowledgment

**Acknowledgment** (ack) betyder att consumern bekräftar att den processat meddelandet.

### Varför är det viktigt?

Tänk dig:

1. Consumer tar emot meddelande
2. Börjar processa
3. **KRASCHAR** mitt i processingen
4. Vad händer med meddelandet?

Utan ack: Meddelandet är borta! 💀\
Med ack: Meddelandet återställs till queue och försöks igen! ✅

### Auto acknowledgment

Default i Spring Boot. Meddelandet bekräftas automatiskt när metoden returnerar.

```java
@RabbitListener(queues = "orders")
public void processOrder(Order order) {
    // Processa...
    // När metoden returnerar skickas ack automatiskt
}
```

**Risk:** Om metoden returnerar men något i background misslyckas, tror RabbitMQ att allt är OK!

### Manual acknowledgment

Mer kontroll - du bestämmer när ack ska skickas.

```properties
# application.properties
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

```java
@Component
public class OrderProcessor {
    
    @RabbitListener(queues = "orders")
    public void processOrder(Order order, Channel channel, 
                            @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws IOException {
        try {
            System.out.println("Processar order: " + order.getId());
            
            // Gör allt som behöver göras
            riskyOperation(order);
            
            // ALLt gick bra - bekräfta!
            channel.basicAck(tag, false);
            
            System.out.println("Order " + order.getId() + " klar och bekräftad!");
            
        } catch (Exception e) {
            System.err.println("Fel! Skickar tillbaka till queue");
            
            // Något gick fel - skicka tillbaka till queue för retry
            channel.basicNack(tag, false, true); // true = requeue
        }
    }
}
```

### Vilken ska man använda?

**Auto ack:**

- ✅ Enklare kod
- ✅ Bra för enkla operationer
- ✅ När risk för failure är låg

**Manual ack:**

- ✅ Mer kontroll
- ✅ Bra för kritiska operationer (betalningar, ordrar)
- ✅ När du vill ha garanterad processing

## Dead Letter Queue (DLQ)

Vad händer med meddelanden som failar om och om igen?

### Problemet

```
Försök 1: FAIL
Försök 2: FAIL
Försök 3: FAIL
... (i all evighet?)
```

Vi vill inte att samma meddelande försöks i all evighet - det blockerar queue!

### Lösningen: Dead Letter Queue

En **Dead Letter Queue** (DLQ) är en separat queue för meddelanden som misslyckats.

```java
@Configuration
public class RabbitConfig {
    
    // Huvudqueue med DLX konfigurerad
    @Bean
    public Queue orderQueue() {
        Map<String, Object> args = new HashMap<>();
        args.put("x-dead-letter-exchange", "dlx"); // Dit failures går
        args.put("x-dead-letter-routing-key", "failed.orders");
        return new Queue("orders", true, false, false, args);
    }
    
    // Dead Letter Exchange
    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange("dlx");
    }
    
    // Dead Letter Queue - dit failures hamnar
    @Bean
    public Queue deadLetterQueue() {
        return new Queue("failed-orders", true);
    }
    
    // Binding
    @Bean
    public Binding deadLetterBinding() {
        return BindingBuilder
            .bind(deadLetterQueue())
            .to(deadLetterExchange())
            .with("failed.orders");
    }
}
```

### Hur fungerar det?

1. Meddelande kommer till `orders` queue
2. Consumer försöker processa - **FAIL**
3. Meddelande rejected (basicNack med requeue=false)
4. RabbitMQ ser: "Aha, denna queue har DLX konfigurerad"
5. Meddelande skickas till DLX → hamnar i `failed-orders` queue
6. Någon kan inspektera/fixa/retry manuellt

### Retry med TTL

Vill du auto-retry efter en stund?

```java
@Bean
public Queue retryQueue() {
    Map<String, Object> args = new HashMap<>();
    args.put("x-dead-letter-exchange", "order-exchange");
    args.put("x-dead-letter-routing-key", "order.placed");
    args.put("x-message-ttl", 30000); // 30 sekunder
    return new Queue("orders-retry", true, false, false, args);
}
```

**Flow:**

1. Processing misslyckas → skickas till retry-queue
2. Meddelande ligger i retry-queue i 30 sekunder
3. TTL går ut → meddelande går tillbaka till huvudqueue
4. Ny retry!

## Pub/Sub exempel med Topics

Nu ska vi bygga ett riktigt pub/sub-scenario!

### Scenario: Användare registrerar sig

När en användare registrerar sig vill flera tjänster veta:

- **Email Service** - skicka välkomstmejl
- **Analytics Service** - logga ny användare
- **Profile Service** - skapa användarprofil

### Configuration

```java
@Configuration
public class PubSubConfig {
    
    // Topic exchange - broadcstar till alla
    @Bean
    public TopicExchange userExchange() {
        return new TopicExchange("user-events");
    }
    
    // Queues för varje service
    @Bean
    public Queue emailQueue() {
        return new Queue("email-queue");
    }
    
    @Bean
    public Queue analyticsQueue() {
        return new Queue("analytics-queue");
    }
    
    @Bean
    public Queue profileQueue() {
        return new Queue("profile-queue");
    }
    
    // Bindings - alla lyssnar på "user.registered"
    @Bean
    public Binding emailBinding() {
        return BindingBuilder
            .bind(emailQueue())
            .to(userExchange())
            .with("user.registered");
    }
    
    @Bean
    public Binding analyticsBinding() {
        return BindingBuilder
            .bind(analyticsQueue())
            .to(userExchange())
            .with("user.registered");
    }
    
    @Bean
    public Binding profileBinding() {
        return BindingBuilder
            .bind(profileQueue())
            .to(userExchange())
            .with("user.registered");
    }
}
```

### Producer - User Service

```java
@Service
public class UserService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void registerUser(User user) {
        // Spara användare i databas
        System.out.println("Användare sparad: " + user.getId());
        
        // Publicera event - alla subscribers får det!
        UserRegisteredEvent event = new UserRegisteredEvent(
            user.getId(),
            user.getEmail(),
            user.getName(),
            LocalDateTime.now()
        );
        
        rabbitTemplate.convertAndSend("user-events", "user.registered", event);
        
        System.out.println("User registered event publicerat!");
    }
}
```

### Consumers

**Email Service:**

```java
@Component
public class EmailService {
    
    @RabbitListener(queues = "email-queue")
    public void handleUserRegistered(UserRegisteredEvent event) {
        System.out.println("Skickar välkomstmejl till: " + event.getEmail());
        
        // Skicka välkomstmejl...
        sendWelcomeEmail(event.getEmail(), event.getName());
        
        System.out.println("Välkomstmejl skickat!");
    }
}
```

**Analytics Service:**

```java
@Component
public class AnalyticsService {
    
    @RabbitListener(queues = "analytics-queue")
    public void handleUserRegistered(UserRegisteredEvent event) {
        System.out.println("Loggar ny användare i analytics: " + event.getUserId());
        
        // Logga till analytics-databas...
        logUserSignup(event);
        
        System.out.println("Analytics uppdaterad!");
    }
}
```

**Profile Service:**

```java
@Component
public class ProfileService {
    
    @RabbitListener(queues = "profile-queue")
    public void handleUserRegistered(UserRegisteredEvent event) {
        System.out.println("Skapar profil för: " + event.getUserId());
        
        // Skapa defaultprofil...
        createDefaultProfile(event.getUserId());
        
        System.out.println("Profil skapad!");
    }
}
```

### Resultat

När en användare registrerar sig:

```
User Service: Publicerar "user.registered"
     ↓
[user-events exchange] broadcasts till:
     ↓
     ├─→ [email-queue] → Email Service skickar mejl
     ├─→ [analytics-queue] → Analytics loggar signup
     └─→ [profile-queue] → Profile Service skapar profil
```

Alla tre händer parallellt och oberoende! Perfekt loose coupling.

## Message-prioritet

Vissa meddelanden är viktigare än andra!

### Konfigurera priority queue

```java
@Bean
public Queue priorityQueue() {
    Map<String, Object> args = new HashMap<>();
    args.put("x-max-priority", 10); // Prioritet 0-10
    return new Queue("priority-orders", true, false, false, args);
}
```

### Skicka med prioritet

```java
public void placeUrgentOrder(Order order) {
    rabbitTemplate.convertAndSend("priority-orders", order, message -> {
        message.getMessageProperties().setPriority(10); // Högsta prioritet!
        return message;
    });
}

public void placeNormalOrder(Order order) {
    rabbitTemplate.convertAndSend("priority-orders", order, message -> {
        message.getMessageProperties().setPriority(5); // Normal prioritet
        return message;
    });
}
```

### Exempel: Support-tickets

```java
public void createTicket(SupportTicket ticket) {
    int priority;
    
    switch(ticket.getSeverity()) {
        case CRITICAL:
            priority = 10; // Systemet nere!
            break;
        case HIGH:
            priority = 7;  // Allvarligt problem
            break;
        case MEDIUM:
            priority = 5;  // Vanlig fråga
            break;
        case LOW:
            priority = 2;  // Mindre viktig
            break;
        default:
            priority = 1;
    }
    
    rabbitTemplate.convertAndSend("support-tickets", ticket, message -> {
        message.getMessageProperties().setPriority(priority);
        return message;
    });
}
```

Critical tickets processas före low priority!

## Skalning med message brokers

Message brokers är perfekta för horisontell skalning!

### Skala consumers horisontellt

Det enklaste sättet att öka throughput:

```
Före:
[Queue] → [Consumer] (processar 100 msg/sec)

Efter:
[Queue] → [Consumer 1] (100 msg/sec)
         [Consumer 2] (100 msg/sec)
         [Consumer 3] (100 msg/sec)
         [Consumer 4] (100 msg/sec)
Total: 400 msg/sec!
```

Starta bara fler instanser av samma applikation. RabbitMQ fördelar automatiskt!

```bash
# Starta flera instanser
java -jar order-processor.jar --server.port=8081 &
java -jar order-processor.jar --server.port=8082 &
java -jar order-processor.jar --server.port=8083 &
java -jar order-processor.jar --server.port=8084 &
```

Eller med Docker/Kubernetes - skala upp replicas!

### Prefetch count

**Prefetch** betyder hur många meddelanden en consumer får åt gången.

```properties
# application.properties
spring.rabbitmq.listener.simple.prefetch=1
```

**Lågt värde (1-5):**

- ✅ Bättre load balancing mellan consumers
- ✅ Bra när messages har olika processing time
- ❌ Mer overhead (många nätverksrequests)

**Högt värde (50-100):**

- ✅ Bättre throughput
- ✅ Färre nätverksrequests
- ❌ Risk för ojämn fördelning

**Exempel scenario:**

Med prefetch=1:

```
Consumer 1: [Msg 1] processas långsamt (10 sec)
Consumer 2: [Msg 2] processas snabbt (1 sec) → får direkt [Msg 3]
Consumer 3: [Msg 4] processas snabbt → får direkt [Msg 5]
```

Med prefetch=100:

```
Consumer 1: [Msg 1-100] får alla direkt, även om #1 är långsam
Consumer 2: Väntar...
Consumer 3: Väntar...
```

**Tumregel:** Börja med lågt (1-10), öka om du behöver högre throughput.

### Concurrent consumers

Varje instans kan också ha flera trådar:

```properties
spring.rabbitmq.listener.simple.concurrency=5
spring.rabbitmq.listener.simple.max-concurrency=10
```

Nu processar varje instans 5-10 meddelanden parallellt!

### Kombination för maximal skalning

```
3 instanser × 5 concurrent consumers = 15 parallella processors!

[Queue] → [Instance 1] → 5 trådar
         [Instance 2] → 5 trådar
         [Instance 3] → 5 trådar
```

## Message durability och persistence

Hur säkert är dina meddelanden?

### Tre nivåer av säkerhet

**1. Volatile queue + non-persistent messages**

```java
Queue queue = new Queue("temp-queue", false); // false = inte durable
// Messages inte persistenta heller
```

- ❌ Om RabbitMQ kraschar: allt borta
- ✅ Snabbast
- Använd för: temporary data, cache, där förlust är OK

**2. Durable queue + non-persistent messages**

```java
Queue queue = new Queue("orders", true); // true = durable
// Men messages persistenta? Nej!
```

- ✅ Queue finns kvar efter restart
- ❌ Messages som var i kön försvinner
- Använd för: när queue-konfigurationen är viktig men messages expendable

**3. Durable queue + persistent messages**

```java
Queue queue = new Queue("orders", true);

// Skicka persistent
rabbitTemplate.convertAndSend("orders", order, message -> {
    message.getMessageProperties()
           .setDeliveryMode(MessageDeliveryMode.PERSISTENT);
    return message;
});
```

- ✅ Queue finns kvar
- ✅ Messages sparas till disk
- ✅ Överlever restart
- ❌ Långsammare (disk I/O)
- Använd för: kritiska meddelanden (ordrar, betalningar, viktiga events)

### Trade-off: Reliability vs Performance

```
Snabbast                             Säkrast
   |                                    |
   v                                    v
Volatile → Durable → Durable + Persistent
```

**Välj baserat på use case:**

- **Betalningar, ordrar**: Durable + Persistent (kan inte förlora!)
- **Notifications**: Durable queue, kanske inte persistent messages
- **Temporary cache**: Volatile (snabbhet viktigare)
- **Analytics events**: Beror på - hur viktiga är de?

## Monitoring och best practices

### Vad ska man övervaka?

**1. Queue depth (antal meddelanden i kö)**

```
Normal: 0-100 messages
Varning: 1000+ messages
Kritiskt: 10000+ messages
```

Om queue växer: antingen producerar för snabbt eller consumers för långsamma!

**2. Message rate**

- Incoming messages/sec
- Outgoing messages/sec
- Är de balanserade?

**3. Consumer utilization**

- Hur många consumers är aktiva?
- Processar de aktivt eller väntar de?

**4. Message latency**

- Tid från publish till consume
- Indikerar om system är överbelastat

**5. Error rate**

- Hur många messages failar?
- Check dead letter queue!

### Monitoring i RabbitMQ Management UI

Gå till `http://localhost:15672`

**Queues tab:**

- Se message counts
- Message rates (graf)
- Consumer count
- Memory usage

**Exchanges tab:**

- Message rates in/out
- Bindings

**Alert när:**

- Queue depth växer konstant (minnesproblem!)
- Consumer count = 0 (ingen processar!)
- High error rate i logs

### Best practices

**1. Idempotent consumers**

Consumer kan få samma meddelande flera gånger (vid retry). Hantera det!

```java
@RabbitListener(queues = "orders")
public void processOrder(Order order) {
    // Kolla om vi redan processat denna order
    if (orderRepository.existsByExternalId(order.getId())) {
        System.out.println("Order redan processad, skippar");
        return; // Idempotent!
    }
    
    // Processa ordern...
    orderRepository.save(order);
}
```

**2. Små meddelanden**

Håll messages små (några KB, inte MB).

```java
// DÅLIGT - skickar hela bilden
byte[] imageData = ... // 10 MB!
rabbitTemplate.convertAndSend("images", imageData);

// BRA - skicka referens
String imageUrl = "s3://bucket/image123.jpg";
rabbitTemplate.convertAndSend("images", new ImageProcessingRequest(imageUrl));
```

**3. Sätt TTL (time-to-live)**

Förhindra att gamla meddelanden hänger kvar för evigt:

```java
@Bean
public Queue orderQueue() {
    Map<String, Object> args = new HashMap<>();
    args.put("x-message-ttl", 86400000); // 24 timmar
    return new Queue("orders", true, false, false, args);
}
```

**4. Använd Dead Letter Queues**

Alltid ha DLQ för viktiga queues - så kan du inspektera failures!

**5. Version your message formats**

När du ändrar message-struktur, hantera gamla format också:

```java
public class OrderMessage {
    private String version = "2.0"; // Lägg till version!
    private String orderId;
    // ...
}

@RabbitListener(queues = "orders")
public void processOrder(OrderMessage msg) {
    if ("1.0".equals(msg.getVersion())) {
        // Hantera gamla formatet
        processV1(msg);
    } else {
        // Nya formatet
        processV2(msg);
    }
}
```

**6. Monitera och logga**

Logga viktiga events:

```java
@RabbitListener(queues = "orders")
public void processOrder(Order order) {
    log.info("Received order: {}", order.getId());
    
    try {
        process(order);
        log.info("Successfully processed order: {}", order.getId());
    } catch (Exception e) {
        log.error("Failed to process order: {}", order.getId(), e);
        throw e;
    }
}
```

**7. Använd inte för request/response**

Message queues är för asynkron kommunikation. För request/response - använd REST eller gRPC!

```java
// DÅLIGT - väntar på svar från queue
String result = waitForResponse(sendMessage(request)); // Varför inte bara REST?

// BRA - fire and forget
sendMessage(order); // Fortsätt direkt!
```

## Event-driven architecture

Message brokers är hjärtat i event-driven system!

### Vad är event-driven architecture?

Istället för att tjänster pratar direkt med varandra, publicerar de **events** när något händer. Andra tjänster lyssnar på events de bryr sig om.

```
Traditional:
[Order Service] --HTTP--> [Email Service]
                --HTTP--> [Inventory Service]
                --HTTP--> [Analytics]

Event-driven:
[Order Service] --"OrderPlaced"--> [Event Bus]
                                        ↓
                    ┌───────────────────┼───────────────────┐
                    ↓                   ↓                   ↓
            [Email Service]     [Inventory Service]    [Analytics]
```

### Fördelar

**1. Loose coupling**

- Order Service vet inget om vem som lyssnar
- Lägg till nya tjänster utan att ändra gamla

**2. Scalability**

- Varje tjänst skalar oberoende
- Events buffras i broker

**3. Resilience**

- Om en tjänst är nere, händer resten ändå
- Events väntar tills tjänsten är uppe igen

**4. Audit trail**

- Alla events loggas
- Perfekt för debugging och compliance

### Komplett exempel: E-commerce

**Events:**

```java
// OrderPlacedEvent
public class OrderPlacedEvent {
    private String orderId;
    private String userId;
    private List<OrderItem> items;
    private double total;
    private LocalDateTime timestamp;
}

// PaymentProcessedEvent
public class PaymentProcessedEvent {
    private String orderId;
    private String paymentId;
    private double amount;
    private LocalDateTime timestamp;
}

// OrderShippedEvent
public class OrderShippedEvent {
    private String orderId;
    private String trackingNumber;
    private LocalDateTime timestamp;
}
```

**Order Service:**

```java
@Service
public class OrderService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void placeOrder(Order order) {
        // Spara order
        orderRepository.save(order);
        
        // Publicera event
        OrderPlacedEvent event = new OrderPlacedEvent(order);
        rabbitTemplate.convertAndSend("events", "order.placed", event);
    }
}
```

**Payment Service:**

```java
@Component
public class PaymentService {
    
    // Lyssnar på order.placed
    @RabbitListener(queues = "payment-queue")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // Processa betalning
        Payment payment = processPayment(event);
        
        // Publicera nytt event
        PaymentProcessedEvent paymentEvent = new PaymentProcessedEvent(payment);
        rabbitTemplate.convertAndSend("events", "payment.processed", paymentEvent);
    }
}
```

**Inventory Service:**

```java
@Component
public class InventoryService {
    
    // Lyssnar också på order.placed
    @RabbitListener(queues = "inventory-queue")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // Reservera produkter
        reserveProducts(event.getItems());
    }
    
    // Lyssnar på payment.processed
    @RabbitListener(queues = "inventory-payment-queue")
    public void handlePaymentProcessed(PaymentProcessedEvent event) {
        // Bekräfta reservation
        confirmReservation(event.getOrderId());
    }
}
```

**Email Service:**

```java
@Component
public class EmailService {
    
    @RabbitListener(queues = "email-queue")
    public void handleOrderPlaced(OrderPlacedEvent event) {
        sendOrderConfirmation(event);
    }
    
    @RabbitListener(queues = "email-payment-queue")
    public void handlePaymentProcessed(PaymentProcessedEvent event) {
        sendPaymentConfirmation(event);
    }
    
    @RabbitListener(queues = "email-shipping-queue")
    public void handleOrderShipped(OrderShippedEvent event) {
        sendShippingNotification(event);
    }
}
```

### Event flow

```
1. Order Service: OrderPlaced event
        ↓
    [Event Bus]
        ↓
   ┌────┴────┬─────────┬──────────┐
   ↓         ↓         ↓          ↓
Payment  Inventory  Email    Analytics
Service   Service   Service   Service
   ↓
PaymentProcessed event
        ↓
    [Event Bus]
        ↓
   ┌────┴────┬─────────┐
   ↓         ↓         ↓
Inventory  Email   Shipping
 Service  Service  Service
```

Varje service reagerar på events den bryr sig om!

## Komplett exempel - Order processing system

Nu bygger vi ett komplett system från scratch!

### Docker Compose setup

```yaml
version: "3.8"

services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  order-service:
    build: ./order-service
    ports:
      - "8080:8080"
    environment:
      SPRING_RABBITMQ_HOST: rabbitmq
    depends_on:
      rabbitmq:
        condition: service_healthy

  email-service:
    build: ./email-service
    environment:
      SPRING_RABBITMQ_HOST: rabbitmq
    depends_on:
      rabbitmq:
        condition: service_healthy

  inventory-service:
    build: ./inventory-service
    environment:
      SPRING_RABBITMQ_HOST: rabbitmq
    depends_on:
      rabbitmq:
        condition: service_healthy
```

### Order Service (Producer)

**OrderController.java:**

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        return ResponseEntity.ok(new OrderResponse(order.getId(), "Order mottagen!"));
    }
}
```

**OrderService.java:**

```java
@Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public Order createOrder(CreateOrderRequest request) {
        // Skapa och spara order
        Order order = new Order();
        order.setId(UUID.randomUUID().toString());
        order.setUserId(request.getUserId());
        order.setItems(request.getItems());
        order.setTotal(calculateTotal(request.getItems()));
        order.setStatus(OrderStatus.PENDING);
        order.setCreatedAt(LocalDateTime.now());
        
        orderRepository.save(order);
        
        // Skicka meddelande för asynkron bearbetning
        OrderMessage message = new OrderMessage(
            order.getId(),
            order.getUserId(),
            order.getItems(),
            order.getTotal()
        );
        
        rabbitTemplate.convertAndSend("order-exchange", "order.placed", message);
        
        log.info("Order {} skapad och skickad för bearbetning", order.getId());
        
        return order;
    }
    
    private double calculateTotal(List<OrderItem> items) {
        return items.stream()
                   .mapToDouble(item -> item.getPrice() * item.getQuantity())
                   .sum();
    }
}
```

**RabbitConfig.java:**

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange("order-exchange");
    }
}
```

### Email Service (Consumer)

**EmailProcessor.java:**

```java
@Component
public class EmailProcessor {
    
    @Autowired
    private EmailSender emailSender;
    
    @RabbitListener(queues = "email-queue")
    public void handleOrderPlaced(OrderMessage order) {
        log.info("Skickar bekräftelsemejl för order {}", order.getOrderId());
        
        try {
            // Skicka mejl
            emailSender.sendOrderConfirmation(
                order.getUserId(),
                order.getOrderId(),
                order.getTotal()
            );
            
            log.info("Bekräftelsemejl skickat för order {}", order.getOrderId());
            
        } catch (Exception e) {
            log.error("Misslyckades skicka mejl för order {}", order.getOrderId(), e);
            throw new AmqpRejectAndDontRequeueException("Email sending failed", e);
        }
    }
}
```

**RabbitConfig.java:**

```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue emailQueue() {
        return new Queue("email-queue", true);
    }
    
    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange("order-exchange");
    }
    
    @Bean
    public Binding emailBinding() {
        return BindingBuilder
            .bind(emailQueue())
            .to(orderExchange())
            .with("order.placed");
    }
}
```

### Inventory Service (Consumer)

**InventoryProcessor.java:**

```java
@Component
public class InventoryProcessor {
    
    @Autowired
    private InventoryRepository inventoryRepository;
    
    @RabbitListener(queues = "inventory-queue")
    public void handleOrderPlaced(OrderMessage order) {
        log.info("Uppdaterar lager för order {}", order.getOrderId());
        
        try {
            // Dra av produkter från lager
            for (OrderItem item : order.getItems()) {
                Inventory inventory = inventoryRepository.findByProductId(item.getProductId())
                    .orElseThrow(() -> new ProductNotFoundException(item.getProductId()));
                
                if (inventory.getQuantity() < item.getQuantity()) {
                    throw new InsufficientStockException(item.getProductId());
                }
                
                inventory.setQuantity(inventory.getQuantity() - item.getQuantity());
                inventoryRepository.save(inventory);
            }
            
            log.info("Lager uppdaterat för order {}", order.getOrderId());
            
        } catch (Exception e) {
            log.error("Misslyckades uppdatera lager för order {}", order.getOrderId(), e);
            throw new AmqpRejectAndDontRequeueException("Inventory update failed", e);
        }
    }
}
```

### Starta systemet

```bash
# Starta allt med Docker Compose
docker-compose up --build

# Skapa en order
curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "user123",
    "items": [
      {"productId": "prod1", "quantity": 2, "price": 99.99},
      {"productId": "prod2", "quantity": 1, "price": 49.99}
    ]
  }'

# Responsen kommer direkt!
# Men email och inventory uppdateras asynkront i bakgrunden
```

Kolla RabbitMQ UI på `http://localhost:15672` för att se meddelanden flöda!

## Common pitfalls - vanliga misstag

### 1. Använda för request/response

```java
// DÅLIGT! Använd inte message queue för detta
public String getUserName(String userId) {
    rabbitTemplate.convertAndSend("user-requests", userId);
    // Nu väntar vi på svar... detta är ineffektivt!
    return waitForResponse();
}

// BRA! Använd REST för synkron kommunikation
public String getUserName(String userId) {
    return restTemplate.getForObject(
        "http://user-service/users/" + userId,
        String.class
    );
}
```

**Regel:** Queue = asynkront. REST/gRPC = synkront.

### 2. Inte hantera message failures

```java
// DÅLIGT - exception försvinner
@RabbitListener(queues = "orders")
public void processOrder(Order order) {
    try {
        riskyOperation(order);
    } catch (Exception e) {
        log.error("Error", e);
        // Exception "sväljs" - meddelande försvinner!
    }
}

// BRA - kasta vidare för retry eller DLQ
@RabbitListener(queues = "orders")
public void processOrder(Order order) {
    try {
        riskyOperation(order);
    } catch (Exception e) {
        log.error("Error processing order {}", order.getId(), e);
        throw new AmqpRejectAndDontRequeueException("Processing failed", e);
        // Går till DLQ om konfigurerad
    }
}
```

### 3. Inte övervaka queue depth

Queue växer och växer... tills minnet tar slut och RabbitMQ kraschar! 💥

**Sätt upp alerts:**

- Varna när queue > 1000 messages
- Kritiskt när queue > 10000 messages
- Lägg till fler consumers!

### 4. För stora meddelanden

```java
// DÅLIGT - skickar 100 MB video
byte[] videoData = Files.readAllBytes(videoPath);
rabbitTemplate.convertAndSend("videos", videoData); // RIP memory

// BRA - skicka referens
String videoUrl = uploadToS3(videoPath);
VideoProcessingRequest request = new VideoProcessingRequest(videoUrl);
rabbitTemplate.convertAndSend("videos", request);
```

### 5. Ingen retry-strategi

Ett tillfälligt nätverksfel → meddelande förlorat. Inte bra!

**Lösning:** Konfigurera retry och DLQ (som vi gått igenom tidigare).

### 6. Inte göra consumers idempotent

```java
// DÅLIGT - kör flera gånger = problem
@RabbitListener(queues = "payments")
public void processPayment(PaymentRequest request) {
    creditCard.charge(request.getAmount()); // Debiterar flera gånger!
}

// BRA - idempotent
@RabbitListener(queues = "payments")
public void processPayment(PaymentRequest request) {
    if (paymentRepository.existsByRequestId(request.getId())) {
        log.info("Payment already processed, skipping");
        return; // Safe!
    }
    
    creditCard.charge(request.getAmount());
    paymentRepository.save(new Payment(request.getId()));
}
```

### 7. Glömma message ordering

Messages i distribuerade system kommer inte nödvändigtvis i ordning!

```
Skickat: [1, 2, 3, 4, 5]
Mottaget: [1, 3, 2, 5, 4] // Oops!
```

**Om ordning är viktig:**

- Använd samma queue/partition
- Inkludera sequence number
- Håll koll på ordning i consumer

## Felsökning

### Problem: Messages tas inte emot

**Checklista:**

1. ✅ Är RabbitMQ igång? `docker ps`
2. ✅ Är consumer-appen igång?
3. ✅ Finns queue? Kolla Management UI
4. ✅ Är binding korrekt? Exchange → Queue
5. ✅ Matchar routing key?
6. ✅ Kolla consumer-logs för exceptions

**Debugga:**

```java
@RabbitListener(queues = "orders")
public void processOrder(Order order) {
    log.info("=== RECEIVED ORDER: {} ===", order.getId());
    // Om detta inte loggas - message kommer inte fram
}
```

### Problem: Queue fylls upp

**Orsaker:**

- Consumers för långsamma
- Consumers kraschat
- För få consumers

**Lösningar:**

1. Starta fler consumer-instanser
2. Öka concurrent consumers
3. Optimera processing-logik
4. Kolla om consumers håller på med långsam I/O

### Problem: Messages försvinner

**Orsaker:**

- Queue inte durable
- Messages inte persistent
- Auto-ack men consumer kraschar

**Lösning:**

```java
// Durable queue
@Bean
public Queue queue() {
    return new Queue("orders", true); // true = durable
}

// Persistent messages
rabbitTemplate.convertAndSend("orders", order, message -> {
    message.getMessageProperties()
           .setDeliveryMode(MessageDeliveryMode.PERSISTENT);
    return message;
});

// Manual ack
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

### Problem: Consumer konsumerar samma message om och om igen

**Orsak:** Exception kastas men message requeued varje gång.

**Lösning:** Använd DLQ!

```java
throw new AmqpRejectAndDontRequeueException("Give up", e);
// Går till DLQ istället för att försöka igen
```

### Debugging tips

**1. Använd Management UI**

- Ser du meddelanden i queue?
- Är consumers anslutna?
- Message rates?

**2. Logga allt viktigt**

```java
log.info("Sending message to queue: {}", queueName);
log.info("Received message from queue: {}", queueName);
log.info("Successfully processed: {}", id);
log.error("Failed to process: {}", id, exception);
```

**3. Skicka testmeddelande manuellt**
I Management UI → Queues → din queue → Publish message

**4. Kolla connection**

```java
@Component
public class RabbitHealthCheck {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @EventListener(ApplicationReadyEvent.class)
    public void checkConnection() {
        try {
            rabbitTemplate.execute(channel -> {
                log.info("✅ Successfully connected to RabbitMQ!");
                return null;
            });
        } catch (Exception e) {
            log.error("❌ Failed to connect to RabbitMQ", e);
        }
    }
}
```

## Integration med Docker Compose

Komplett setup för lokal utveckling!

### docker-compose.yml

```yaml
version: "3.8"

services:
  # RabbitMQ broker
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    ports:
      - "5672:5672" # AMQP
      - "15672:15672" # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: secret
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Producer service
  producer:
    build: ./producer-service
    container_name: producer
    ports:
      - "8080:8080"
    environment:
      SPRING_RABBITMQ_HOST: rabbitmq
      SPRING_RABBITMQ_PORT: 5672
      SPRING_RABBITMQ_USERNAME: admin
      SPRING_RABBITMQ_PASSWORD: secret
    depends_on:
      rabbitmq:
        condition: service_healthy
    networks:
      - app-network

  # Consumer service (kan skala!)
  consumer:
    build: ./consumer-service
    environment:
      SPRING_RABBITMQ_HOST: rabbitmq
      SPRING_RABBITMQ_PORT: 5672
      SPRING_RABBITMQ_USERNAME: admin
      SPRING_RABBITMQ_PASSWORD: secret
    depends_on:
      rabbitmq:
        condition: service_healthy
    networks:
      - app-network
    # Skala med: docker-compose up --scale consumer=3

volumes:
  rabbitmq_data:

networks:
  app-network:
    driver: bridge
```

### application.yml för services

```yaml
spring:
  rabbitmq:
    host: ${SPRING_RABBITMQ_HOST:localhost}
    port: ${SPRING_RABBITMQ_PORT:5672}
    username: ${SPRING_RABBITMQ_USERNAME:guest}
    password: ${SPRING_RABBITMQ_PASSWORD:guest}
    listener:
      simple:
        concurrency: 5
        max-concurrency: 10
        prefetch: 10
        retry:
          enabled: true
          initial-interval: 3000
          max-attempts: 3
          multiplier: 2
```

### Starta och skala

```bash
# Starta allt
docker-compose up

# Starta i bakgrunden
docker-compose up -d

# Skala consumers
docker-compose up --scale consumer=5

# Se logs
docker-compose logs -f consumer

# Stoppa allt
docker-compose down

# Stoppa och ta bort volumes (börja från scratch)
docker-compose down -v
```

## Sammanfattning

Message brokers är ett kraftfullt verktyg för att bygga skalbara och robusta system!

### Nyckelkoncept

**Message Broker:**

- Mellanhand för meddelanden mellan tjänster
- Buffrar meddelanden
- Garanterar leverans
- Kopplar loss tjänster från varandra

**Synkron vs Asynkron:**

- Synkron = vänta på svar (REST, phone call)
- Asynkron = skicka och fortsätt (Queue, email)
- Använd asynkron när det går!

**Queue vs Topic:**

- Queue = one-to-one (work distribution)
- Topic = one-to-many (event broadcasting)
- Välj baserat på use case

**RabbitMQ:**

- Populär message broker
- AMQP-protokoll
- Management UI
- Stödjer både queues och topics
- Perfekt för de flesta användningsfall

**Spring Boot Integration:**

- `spring-boot-starter-amqp`
- `@RabbitListener` för consumers
- `RabbitTemplate` för producers
- Auto-configuration = enkelt!

**Skalning:**

- Lägg till fler consumer-instanser
- Öka concurrent consumers
- Justera prefetch count
- Horizontell skalning FTW!

**Best Practices:**

- Idempotent consumers
- Små meddelanden
- Använd DLQ
- Monitoring!
- Durable + Persistent för kritiska messages
- Inte för request/response

**Event-Driven Architecture:**

- Tjänster publicerar events
- Andra lyssnar och reagerar
- Loose coupling
- Skalbart och resilient

### När ska man använda message brokers?

✅ **Använd när:**

- Background processing (skicka mejl, generera rapporter)
- Event-driven architecture
- Microservices-kommunikation
- Behöver buffra requests
- Vill decoupla tjänster
- Behöver reliability

❌ **Använd INTE när:**

- Behöver svar direkt (använd REST)
- Real-time request/response
- Enkel CRUD-operation
- Liten monolitisk app

### Nästa steg

Nu när du förstår message brokers är du redo för:

- **Kubernetes** - orkestrering av containers
- **Service mesh** - avancerad microservice-kommunikation
- **Event sourcing** - bygga state från events
- **CQRS** - Command Query Responsibility Segregation

Message brokers är en grundsten i moderna distribuerade system. Master det här så är du på god väg att bygga skalbara system som kan hantera vad som helst! 🚀

## Övningsuppgifter

### 1. Förklara skillnaden

**Uppgift:** Förklara skillnaden mellan Queue och Topic med egna exempel från verkliga system. När skulle du använda varje typ?

**Exempel på svar att tänka på:**

- Queue för att processa ordrar (bara en ska processa)
- Topic för user registered (många vill veta)
- Queue för bildprocessering (fördela arbete)
- Topic för aktiekursuppdateringar (alla vill ha samma data)

### 2. Bygg en producer

**Uppgift:** Skapa en Spring Boot-applikation som:

- Har ett REST API för att skapa "tasks"
- Skickar varje task till en RabbitMQ queue
- Loggar när task skickats

**Steg:**

1. Setup Spring Boot projekt med AMQP
2. Konfigurera RabbitMQ connection
3. Skapa Task model
4. Skapa TaskController med POST endpoint
5. Skapa TaskProducer som skickar till queue
6. Testa med Postman/curl

### 3. Bygg en consumer

**Uppgift:** Skapa en consumer-applikation som:

- Lyssnar på task-queue
- Processar varje task (simulera med Thread.sleep)
- Loggar start och slut
- Hantera exceptions

**Extra:** Konfigurera manual acknowledgment och testa vad som händer om consumer kraschar mitt i processing.

### 4. Docker Compose setup

**Uppgift:** Sätt upp ett komplett system med:

- RabbitMQ i Docker
- Producer-service
- Consumer-service (minst 2 instanser)
- docker-compose.yml som startar allt

**Testa:**

- Skicka 100 tasks
- Se hur de fördelas mellan consumers
- Kolla Management UI

### 5. Design async processing

**Scenario:** En användare laddar upp en bild till din webapp. Bilden behöver:

1. Sparas i original-storlek
2. Skapas thumbnail (small)
3. Skapas medium-storlek
4. Köras genom AI för content moderation
5. Användaren ska få notifikation när klart

**Uppgift:** Rita och förklara:

- Hur skulle du designa detta med message queues?
- Vilka queues/topics behövs?
- Vilka services?
- Flöde från upload till notification?
- Vad händer om en service är nere?

### 6. Idempotency diskussion

**Uppgift:** Förklara varför idempotency är viktigt för message consumers. Ge exempel på:

- Vad som kan gå fel utan idempotency
- Hur man gör en consumer idempotent
- Konkret kod-exempel

**Scenario att fundera på:**

- Betalning processas
- Consumer kraschar precis efter betalning men innan ack
- Message försöks igen
- Utan idempotency: användaren debiteras två gånger! 💸

### 7. Event-driven flow

**Uppgift:** Rita ett event-driven flow för "User registers on website". Inkludera:

- User Service (producent)
- Email Service (consumer) - skicka välkomstmejl
- Profile Service (consumer) - skapa default-profil
- Analytics Service (consumer) - logga signup
- Loyalty Program Service (consumer) - ge välkomstbonus

Rita:

- Exchanges och queues
- Message flow
- Vad händer om Email Service är nere?

---

**Bonus:** Implementera ett av systemen ovan fullständigt och deploya med Docker Compose. Skala consumer-services och se hur load fördelas!
