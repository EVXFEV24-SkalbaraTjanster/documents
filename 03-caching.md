# Caching - Snabba upp din applikation

## Introduktion

Tänk dig att du varje gång du vill ringa en kompis måste slå upp numret i telefonboken. Jobbigt, eller hur? Mycket enklare att spara numret i telefonen första gången och sedan använda det direkt. Det är exakt vad caching handlar om!

**Cache** är ett lager där vi sparar data som är dyrt eller långsamt att hämta, så att vi kan få tag på det snabbare nästa gång. Istället för att fråga databasen om samma sak om och om igen, sparar vi svaret i minnet och återanvänder det.

### Verkliga exempel

- **Din webbläsare** cachar bilder och CSS-filer så att webbplatser laddar snabbare
- **DNS** cachar IP-adresser så du inte behöver slå upp google.com varje gång
- **CDN:er** (Content Delivery Networks) cachar webbsidor närmare användarna geografiskt
- **Processorns L1/L2/L3 cache** gör beräkningar snabbare genom att ha data nära CPU:n

### Prestandaskillnaden

Låt oss titta på verkliga siffror:

```
Database query:    100 ms (0.1 sekund)
Cache lookup:      1 ms (0.001 sekund)
Skillnad:          100x snabbare! 🚀
```

Om du har 1000 användare som alla vill se samma produktlista:
- **Utan cache:** 1000 × 100ms = 100 sekunder totalt i databasen
- **Med cache:** 1 × 100ms + 999 × 1ms ≈ 1 sekund totalt

Det är därför caching är så kritiskt för skalbarhet!

## Varför cache?

### 1. Minska databasbelastningen

Din databas är ofta flaskhalsen i systemet. Varje query tar resurser och tid. Med caching kan du ta bort 80-90% av databasanropen för data som inte ändras ofta.

```
Utan cache:
User 1 -> [App] -> [Database] "SELECT * FROM products"
User 2 -> [App] -> [Database] "SELECT * FROM products"  
User 3 -> [App] -> [Database] "SELECT * FROM products"
... 1000 användare = 1000 queries

Med cache:
User 1 -> [App] -> [Database] "SELECT * FROM products" -> [Cache]
User 2 -> [App] -> [Cache] (direkt!)
User 3 -> [App] -> [Cache] (direkt!)
... 1000 användare = 1 query + 999 cache hits
```

### 2. Snabbare responstider

Användare hatar att vänta. Varje extra 100ms i laddtid påverkar användarupplevelsen negativt. Med cache går svarstiden ner dramatiskt.

### 3. Hantera mer trafik med samma resurser

Om din app kan svara 10x snabbare med cache kan samma server hantera 10x fler användare. Det betyder att du kan skala längre innan du behöver fler servrar.

### 4. Kostnadsbesparingar

Färre databasanrop = mindre databaskraft behövs = lägre kostnader i molnet. Redis (en populär cache) är ofta betydligt billigare än att skala upp en databas.

### När caching är vettigt

✅ **Bra kandidater för caching:**
- Data som läses ofta men ändras sällan
- Dyra beräkningar som alltid ger samma resultat
- API-anrop till externa tjänster
- Databas-queries som tar lång tid

❌ **Dåliga kandidater:**
- Data som ändras konstant
- Personlig/känslig data som ska vara realtid
- Transaktionsdata där precision är kritisk
- Små, snabba queries (overhead kan vara större än vinsten)

## Vad ska man cacha?

### Data som läses ofta men ändras sällan

**Produktkataloger**
```java
// Produkter ändras kanske några gånger om dagen
// men läses tusentals gånger
@Cacheable("products")
public Product getProduct(Long id) {
    return productRepository.findById(id);
}
```

**Användarprofiler**
```java
// Profilen ändras när användaren uppdaterar den
// men läses varje gång de gör något
@Cacheable("users")
public User getUserProfile(Long userId) {
    return userRepository.findById(userId);
}
```

**Konfiguration**
```java
// System-konfiguration ändras mycket sällan
@Cacheable("config")
public AppConfig getConfiguration() {
    return configRepository.getConfig();
}
```

### Dyra beräkningar

```java
// Komplicerade beräkningar med samma input
// ger alltid samma output
@Cacheable(value = "calculations", key = "#input")
public BigDecimal complexCalculation(String input) {
    // 5 sekunder av matematik...
    return result;
}
```

### Externa API-anrop

```java
// Externa API:er kan vara långsamma eller ha rate limits
@Cacheable(value = "weather", key = "#city")
public WeatherData getWeather(String city) {
    return weatherApi.fetchWeather(city); // Långsamt API-anrop
}
```

### Databas-queries med joins

```java
// Queries med många joins kan vara dyra
@Cacheable("orderSummaries")
public OrderSummary getOrderWithDetails(Long orderId) {
    // JOIN orders, items, users, products, shipping...
    return orderRepository.findCompleteOrder(orderId);
}
```

### Vad du INTE ska cacha

❌ **Banktransaktioner** - måste vara exakta i realtid
❌ **Lagerantal i e-handel** - risk för att sälja mer än du har
❌ **Lösenord eller känslig data** - säkerhetsrisk
❌ **Session data för inloggade användare** - använd Redis för sessions istället
❌ **Real-time data** - börskurser, live-sport, etc.

## Cache strategier (patterns)

Det finns flera olika sätt att använda cache. Varje strategi har sina för- och nackdelar.

### 1. Cache-Aside (Lazy Loading)

Det här är den vanligaste strategin. Applikationen pratar med både cache och databas.

**Hur det funkar:**
1. App kollar i cachen först
2. **Cache hit** → returnera direkt!
3. **Cache miss** → hämta från databas, spara i cache, returnera

```
                    ┌──────────┐
                    │   App    │
                    └────┬─────┘
                         │
                    1. Kolla cache
                         │
                    ┌────▼────┐
                    │  Cache  │
                    └────┬────┘
                         │
                  Hit?   │   Miss?
                    ◄────┴────►
                    │          │
              2. Return   3. Query DB
                              │
                         ┌────▼────┐
                         │Database │
                         └────┬────┘
                              │
                         4. Save to
                            cache
                              │
                         5. Return
```

**Kod exempel:**
```java
public Product getProduct(Long id) {
    // 1. Kolla cache
    Product product = cache.get("product:" + id);
    
    if (product != null) {
        // 2. Cache hit!
        return product;
    }
    
    // 3. Cache miss - hämta från DB
    product = database.findById(id);
    
    // 4. Spara i cache för nästa gång
    cache.set("product:" + id, product);
    
    // 5. Returnera
    return product;
}
```

**Fördelar:**
- Enkel att förstå och implementera
- Endast data som faktiskt används hamnar i cachen
- Du har full kontroll över cache-logiken

**Nackdelar:**
- Första anropet är alltid långsamt (cache miss)
- Tre round-trips vid miss (kolla cache, hämta DB, spara cache)
- Kräver kod i applikationen för att hantera cache

### 2. Read-Through

Cache-lagret sitter mellan app och databas. När cache saknar data hämtar den själv från databasen.

```
                    ┌──────────┐
                    │   App    │
                    └────┬─────┘
                         │
                    1. Request data
                         │
                    ┌────▼────┐
                    │  Cache  │
                    │ (smart!) │
                    └────┬────┘
                         │
                  Hit? Return
                         │
                  Miss? Load from DB
                         │
                    ┌────▼────┐
                    │Database │
                    └─────────┘
```

**Fördelar:**
- Applikationen ser bara cachen
- Enklare applikationskod
- Cache-logik är centraliserad

**Nackdelar:**
- Behöver ett smart cache-system
- Mindre vanligt i praktiken
- Första anropet fortfarande långsamt

### 3. Write-Through

När data uppdateras, skrivs det till både cache och databas samtidigt.

```
                    ┌──────────┐
                    │   App    │
                    └────┬─────┘
                         │
                    1. Update data
                         │
                    ┌────▼────┐
                    │  Cache  │
                    └────┬────┘
                         │
                    2. Write to cache
                         │
                    3. Write to DB
                         │
                    ┌────▼────┐
                    │Database │
                    └─────────┘
```

**Fördelar:**
- Cache är alltid synkad med databasen
- Läsningar är alltid snabba
- Inget stale data

**Nackdelar:**
- Skrivningar blir långsammare (måste vänta på både cache och DB)
- Kan cacha data som aldrig läses
- Mer komplex implementation

### 4. Write-Behind (Write-Back)

Data skrivs till cachen direkt, och till databasen senare (asynkront).

```
                    ┌──────────┐
                    │   App    │
                    └────┬─────┘
                         │
                    1. Update data
                         │
                    ┌────▼────┐
                    │  Cache  │ ← 2. Snabb write!
                    └────┬────┘
                         │
                    3. Senare...
                   (async batch)
                         │
                    ┌────▼────┐
                    │Database │
                    └─────────┘
```

**Fördelar:**
- Mycket snabba skrivningar
- Kan batcha flera writes till DB
- Bra för write-heavy system

**Nackdelar:**
- Risk för dataförlust om cache kraschar
- Komplext att implementera
- DB kan vara out-of-sync

**I praktiken** använder de flesta **Cache-Aside** eftersom det är enklast och ger dig mest kontroll. Spring Boot's cache-annotations använder Cache-Aside.

## Cache invalidation

Det finns ett klassiskt citat inom datavetenskap:

> "There are only two hard things in Computer Science: cache invalidation and naming things."
> — Phil Karlton

Varför är cache invalidation så svårt? För att du måste balansera mellan:
- **Fräsch data** (users vill se uppdaterad info)
- **Performance** (cachen ska ju göra saker snabbare)

Om du aldrig invaliderar cachen får användare gammal data. Om du invaliderar för ofta får du ingen nytta av cachen.

### Problemet med stale data

```
Tid 0:  Product pris = 100kr (i DB och cache)
Tid 1:  Admin ändrar pris till 80kr (i DB)
        ❌ Cache säger fortfarande 100kr!
Tid 2:  Kund ser 100kr (från cache)
        Kund blir sur när de ska betala och ser 80kr
```

### Strategier för invalidation

#### 1. TTL (Time To Live)

Sätt en timer på all cachad data. Efter X sekunder/minuter raderas den automatiskt.

```java
// Cache i 5 minuter
cache.set("product:123", product, Duration.ofMinutes(5));
```

**Fördelar:**
- Enkelt - ingen kod behövs vid uppdateringar
- Garanterar att data inte är äldre än X

**Nackdelar:**
- Data kan vara gammal upp till X sekunder/minuter
- För kort TTL = mindre cache-nytta
- För lång TTL = mer stale data

#### 2. Manuell invalidation

När data uppdateras, ta bort den från cachen explicit.

```java
public void updateProduct(Product product) {
    // Uppdatera DB
    productRepository.save(product);
    
    // Ta bort från cache
    cache.delete("product:" + product.getId());
}
```

**Fördelar:**
- Precisare - cache uppdateras direkt när data ändras
- Ingen stale data (i teorin)

**Nackdelar:**
- Måste komma ihåg att invalidera överallt
- Lätt att missa ställen
- Svårt med relaterad data

#### 3. Event-baserad invalidation

Använd events/meddelanden för att notifiera när data ändras.

```java
@EventListener
public void onProductUpdated(ProductUpdatedEvent event) {
    cache.delete("product:" + event.getProductId());
}
```

**Fördelar:**
- Decoupled - olika delar av systemet kan reagera
- Fungerar bra i distribuerade system

**Nackdelar:**
- Mer komplext
- Kräver event-infrastruktur

### Best practice: Kombinera strategier

Den bästa lösningen är ofta att kombinera TTL med manuell invalidation:

```java
// Sätt TTL som backup (om vi missar att invalidera)
cache.set("product:123", product, Duration.ofHours(1));

// Men invalidera manuellt när vi vet att data ändrats
public void updateProduct(Product product) {
    productRepository.save(product);
    cache.delete("product:" + product.getId());
}
```

Nu har du både precision OCH ett säkerhetsnät!

## TTL (Time To Live)

TTL är tiden som data får leva i cachen innan den raderas automatiskt. Det är som ett bäst-före-datum.

### Hur väljer man rätt TTL?

Det beror helt på hur ofta datan ändras och hur viktigt det är med fräsch data.

**Exempel:**

| Data typ | TTL | Motivering |
|----------|-----|------------|
| Användarprofil | 1 timme | Uppdateras sällan, okej om lite gammal |
| Produktpris | 5 minuter | Ändras ibland, viktigt att vara rätt |
| Produktbeskrivning | 24 timmar | Ändras sällan, okej om lite gammal |
| System-config | 24 timmar | Nästan aldrig ändras |
| Väderprognoser | 15 minuter | Ändras varje timme från API |
| Aktiekurser | INGEN CACHE | Ändras konstant, måste vara real-time |

### Tradeoffs

**Lång TTL (t.ex. 24 timmar):**
- ✅ Bättre cache hit ratio
- ✅ Mindre databasbelastning  
- ❌ Mer stale data
- ❌ Användare ser gammal info längre

**Kort TTL (t.ex. 1 minut):**
- ✅ Fräschare data
- ✅ Mindre risk för problem
- ❌ Sämre cache hit ratio
- ❌ Mer databasbelastning

### Implementera TTL i Spring

```java
@Cacheable(value = "products")
public Product getProduct(Long id) {
    return productRepository.findById(id);
}

// I application.properties
spring.cache.redis.time-to-live=600000  # 10 minuter i millisekunder
```

Eller per cache:

```java
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(10))
        .disableCachingNullValues();
}

@Bean
public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
    return (builder) -> builder
        .withCacheConfiguration("products",
            RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1)))
        .withCacheConfiguration("users",
            RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30)));
}
```

## Redis introduction

**Redis** (REmote DIctionary Server) är en in-memory databas som är perfekt för caching. Tänk på det som en superlightning-snabb HashMap som flera servrar kan dela.

### Varför Redis?

**In-memory = supersnabbt**
Data lagras i RAM istället för på disk, vilket gör reads och writes extremt snabba (mikrosekunder).

**Stödjer olika datastrukturer**
- **Strings** - vanliga värden
- **Hashes** - objekt med fält
- **Lists** - ordnade listor
- **Sets** - unika element
- **Sorted Sets** - sets med ranking
- och många fler...

**Distribuerad cache**
Flera instanser av din app kan dela samma Redis-cache. Det är därför Redis är bättre än en HashMap i minnet!

```
Lokalt minne (dåligt):
[App 1] ─ HashMap  ← Bara App 1 kan använda
[App 2] ─ HashMap  ← Separat cache!
[App 3] ─ HashMap  ← Ingen delning!

Redis (bra):
[App 1] ─┐
[App 2] ─┼─ [Redis] ← Alla delar samma cache!
[App 3] ─┘
```

### Redis vs. applikationsminne

| Aspekt | HashMap i app | Redis |
|--------|---------------|-------|
| **Hastighet** | Snabbast (ingen nätverkstid) | Mycket snabb (1-2ms) |
| **Delad mellan instanser** | ❌ Nej | ✅ Ja |
| **Överlever restart** | ❌ Nej | ✅ Kan konfigureras |
| **Memory limit** | Begränsad av app | Dedikerad minne |
| **Skalbarhet** | ❌ Funkar inte med flera instanser | ✅ Perfekt för horisontell skalning |

### Redis datatypes (snabb översikt)

**String** - enklaste typen
```
SET user:123:name "Anna"
GET user:123:name
→ "Anna"
```

**Hash** - perfekt för objekt
```
HSET user:123 name "Anna" age 25 email "anna@example.com"
HGET user:123 name
→ "Anna"
HGETALL user:123
→ {name: "Anna", age: 25, email: "anna@example.com"}
```

**List** - ordnade listor
```
LPUSH notifications "New message!"
LPUSH notifications "Order shipped"
LRANGE notifications 0 -1
→ ["Order shipped", "New message!"]
```

**Set** - unika element
```
SADD tags:product:123 "electronics" "sale" "new"
SMEMBERS tags:product:123
→ ["electronics", "sale", "new"]
```

### När ska man använda Redis?

✅ **Sessions** - för inloggade användare
✅ **Cache** - för databas-queries
✅ **Rate limiting** - räkna requests per användare
✅ **Leaderboards** - med sorted sets
✅ **Pub/Sub** - real-time meddelanden
✅ **Queues** - enklare än RabbitMQ/Kafka för vissa use cases

## Redis i praktiken

### Installera och kör Redis

Det enklaste sättet är med **Docker**:

```bash
# Kör Redis i en container
docker run -d -p 6379:6379 --name my-redis redis:latest

# Kolla att det funkar
docker ps
```

Eller installera lokalt:

```bash
# macOS
brew install redis
redis-server

# Ubuntu
sudo apt install redis-server
sudo systemctl start redis

# Windows
# Använd Docker eller WSL2
```

### Anslut till Redis CLI

```bash
# Om du kör Docker
docker exec -it my-redis redis-cli

# Om du installerade lokalt
redis-cli
```

### Grundläggande Redis-kommandon

```bash
# SET - spara ett värde
127.0.0.1:6379> SET name "Johan"
OK

# GET - hämta ett värde
127.0.0.1:6379> GET name
"Johan"

# EXPIRE - sätt TTL (i sekunder)
127.0.0.1:6379> EXPIRE name 60
(integer) 1
# Nu raderas "name" efter 60 sekunder

# SETEX - SET med TTL samtidigt
127.0.0.1:6379> SETEX session:abc123 3600 "user_data"
OK
# Sparar och sätter TTL på 1 timme (3600 sekunder)

# TTL - kolla hur lång tid kvar
127.0.0.1:6379> TTL name
(integer) 45

# DEL - radera
127.0.0.1:6379> DEL name
(integer) 1

# EXISTS - kolla om key finns
127.0.0.1:6379> EXISTS name
(integer) 0
# 0 = finns inte, 1 = finns

# KEYS - hitta nycklar (OBS: använd ej i produktion på stora DBs)
127.0.0.1:6379> KEYS user:*
1) "user:123"
2) "user:456"

# FLUSHALL - radera ALLT (farligt!)
127.0.0.1:6379> FLUSHALL
OK
```

### Användbara debug-kommandon

```bash
# PING - testa connection
127.0.0.1:6379> PING
PONG

# INFO - systeminfo
127.0.0.1:6379> INFO
# Visar massa statistik

# DBSIZE - antal nycklar
127.0.0.1:6379> DBSIZE
(integer) 42

# MONITOR - se alla kommandon i realtid
127.0.0.1:6379> MONITOR
OK
# Nu syns alla Redis-operationer live
```

## Spring Boot och caching

Spring Boot har ett abstrakt cache-API som funkar med flera olika cache-providers (Redis, Caffeine, EhCache, etc.). Det betyder att du kan byta cache-implementation utan att ändra din kod!

### Lägg till dependencies

I din `pom.xml`:

```xml
<dependencies>
    <!-- Spring Cache abstraction -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    
    <!-- Redis implementation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    
    <!-- JSON serialization (för att spara objekt i Redis) -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

### Konfigurera Redis

I `application.properties`:

```properties
# Redis connection
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.data.redis.password=  # Inget lösenord lokalt
spring.data.redis.timeout=60000

# Cache configuration
spring.cache.type=redis
spring.cache.redis.time-to-live=600000  # 10 minuter i millisekunder
spring.cache.redis.cache-null-values=false  # Cacha inte null-värden
```

Eller med YAML (`application.yml`):

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      timeout: 60000
  cache:
    type: redis
    redis:
      time-to-live: 600000  # 10 minuter
      cache-null-values: false
```

### Aktivera caching

Lägg till `@EnableCaching` på din main-klass eller en konfigurationsklass:

```java
@SpringBootApplication
@EnableCaching  // ← Detta aktiverar cache-annotations!
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### Konfigurera custom cache-inställningar

Om du vill ha mer kontroll, skapa en konfigurationsklass:

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .disableCachingNullValues()
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );
    }
    
    @Bean
    public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        return (builder) -> builder
            .withCacheConfiguration("products",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofHours(1)))
            .withCacheConfiguration("users",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(30)));
    }
}
```

Nu är allt setup! Nu kan vi börja använda cache-annotations.

## Spring Cache annotations

Spring ger oss tre huvudannotations för att jobba med cache. De är super-enkla att använda!

### @Cacheable - Hämta från cache

`@Cacheable` kollar först i cachen. Om data finns där, returneras det direkt. Annars körs metoden och resultatet sparas i cachen.

```java
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Cacheable("products")  // ← "products" är cache-namnet
    public Product getProduct(Long id) {
        System.out.println("Hämtar från databas..."); // Syns bara vid cache miss
        return productRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("Product not found"));
    }
}
```

**Vad händer:**

1. Första anropet: `getProduct(123)`
   - Cache miss → metoden körs
   - "Hämtar från databas..." printas
   - Resultat sparas i cache med key `products::123`
   - Resultat returneras

2. Andra anropet: `getProduct(123)`
   - Cache hit! Metoden körs INTE
   - Inget printas
   - Resultat hämtas direkt från cache
   - Blazing fast! 🚀

**Custom cache key:**

```java
@Cacheable(value = "products", key = "#id")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}

// Kombinera flera parametrar i key
@Cacheable(value = "products", key = "#category + '-' + #page")
public List<Product> getProductsByCategory(String category, int page) {
    return productRepository.findByCategory(category, PageRequest.of(page, 20));
}
```

**Conditionals:**

```java
// Cacha bara om id > 0
@Cacheable(value = "products", condition = "#id > 0")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}

// Cacha INTE om resultat är null
@Cacheable(value = "products", unless = "#result == null")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElse(null);
}
```

### @CachePut - Uppdatera cache

`@CachePut` kör alltid metoden OCH uppdaterar cachen med resultatet. Använd när du uppdaterar data och vill hålla cachen synkad.

```java
@CachePut(value = "products", key = "#product.id")
public Product updateProduct(Product product) {
    System.out.println("Uppdaterar i databas...");
    Product updated = productRepository.save(product);
    // Cachen uppdateras automatiskt med nya värdet
    return updated;
}
```

**Skillnad mellan @Cacheable och @CachePut:**

```java
@Cacheable("products")  // Kör INTE metoden om cache hit
public Product get(Long id) { ... }

@CachePut("products")   // Kör ALLTID metoden, uppdaterar cache
public Product update(Product p) { ... }
```

### @CacheEvict - Ta bort från cache

`@CacheEvict` raderar data från cachen. Använd när du tar bort eller uppdaterar data.

```java
@CacheEvict(value = "products", key = "#id")
public void deleteProduct(Long id) {
    productRepository.deleteById(id);
    // Produkten tas bort från cache också
}

// Ta bort hela cachen
@CacheEvict(value = "products", allEntries = true)
public void deleteAllProducts() {
    productRepository.deleteAll();
}

// Evict efter metoden kör klart (default)
@CacheEvict(value = "products", key = "#id", beforeInvocation = false)
public void deleteProduct(Long id) {
    productRepository.deleteById(id);
}

// Evict INNAN metoden kör (om metoden kan faila)
@CacheEvict(value = "products", key = "#id", beforeInvocation = true)
public void riskyDelete(Long id) {
    // Cache raderas innan detta körs
    productRepository.deleteById(id);
}
```

### Kombinera flera cache-operationer

Ibland behöver du både evict och uppdatera:

```java
@Caching(
    evict = {
        @CacheEvict(value = "products", key = "#product.id"),
        @CacheEvict(value = "productList", allEntries = true)
    },
    put = {
        @CachePut(value = "products", key = "#product.id")
    }
)
public Product updateProduct(Product product) {
    return productRepository.save(product);
}
```

## Komplett exempel

Låt oss bygga en komplett ProductService med caching!

### Domain model

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String description;
    private BigDecimal price;
    private String category;
    private Integer stock;
    
    // Getters, setters, constructors...
}
```

### Repository

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByCategory(String category);
}
```

### Service med caching

```java
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    /**
     * Hämta en produkt - cachas i 1 timme
     */
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        System.out.println("Cache MISS - hämtar produkt " + id + " från DB");
        return productRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Product not found: " + id));
    }
    
    /**
     * Hämta alla produkter - cachas i 10 minuter
     */
    @Cacheable("allProducts")
    public List<Product> getAllProducts() {
        System.out.println("Cache MISS - hämtar alla produkter från DB");
        return productRepository.findAll();
    }
    
    /**
     * Hämta produkter per kategori - cachas med kategori i key
     */
    @Cacheable(value = "productsByCategory", key = "#category")
    public List<Product> getProductsByCategory(String category) {
        System.out.println("Cache MISS - hämtar kategori " + category + " från DB");
        return productRepository.findByCategory(category);
    }
    
    /**
     * Skapa ny produkt - invaliderar "allProducts" cache
     */
    @CacheEvict(value = "allProducts", allEntries = true)
    public Product createProduct(Product product) {
        System.out.println("Skapar ny produkt - invaliderar cache");
        return productRepository.save(product);
    }
    
    /**
     * Uppdatera produkt - uppdaterar cache OCH invaliderar listor
     */
    @Caching(
        put = @CachePut(value = "products", key = "#product.id"),
        evict = {
            @CacheEvict(value = "allProducts", allEntries = true),
            @CacheEvict(value = "productsByCategory", allEntries = true)
        }
    )
    public Product updateProduct(Product product) {
        System.out.println("Uppdaterar produkt " + product.getId());
        return productRepository.save(product);
    }
    
    /**
     * Ta bort produkt - invaliderar allt relaterat
     */
    @Caching(evict = {
        @CacheEvict(value = "products", key = "#id"),
        @CacheEvict(value = "allProducts", allEntries = true),
        @CacheEvict(value = "productsByCategory", allEntries = true)
    })
    public void deleteProduct(Long id) {
        System.out.println("Tar bort produkt " + id);
        productRepository.deleteById(id);
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.getProduct(id);
    }
    
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }
    
    @GetMapping("/category/{category}")
    public List<Product> getByCategory(@PathVariable String category) {
        return productService.getProductsByCategory(category);
    }
    
    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.createProduct(product);
    }
    
    @PutMapping("/{id}")
    public Product updateProduct(@PathVariable Long id, @RequestBody Product product) {
        product.setId(id);
        return productService.updateProduct(product);
    }
    
    @DeleteMapping("/{id}")
    public void deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
    }
}
```

### Configuration

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        return (builder) -> builder
            // Individuella produkter - cache i 1 timme
            .withCacheConfiguration("products",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofHours(1)))
            // Produktlistor - cache i 10 minuter (ändras oftare)
            .withCacheConfiguration("allProducts",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(10)))
            .withCacheConfiguration("productsByCategory",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(10)));
    }
}
```

### application.properties

```properties
# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/productdb
spring.datasource.username=postgres
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update

# Redis
spring.data.redis.host=localhost
spring.data.redis.port=6379

# Cache
spring.cache.type=redis
spring.cache.redis.cache-null-values=false

# Logging (för att se cache-effekten)
logging.level.com.example.service=DEBUG
```

### Testa det!

```bash
# Starta Redis
docker run -d -p 6379:6379 redis

# Starta appen
./mvnw spring-boot:run

# Första requesten (cache miss)
curl http://localhost:8080/api/products/1
# Output: "Cache MISS - hämtar produkt 1 från DB"

# Andra requesten (cache hit!)
curl http://localhost:8080/api/products/1
# Output: (inget i loggen - kom från cache)

# Uppdatera produkten
curl -X PUT http://localhost:8080/api/products/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated Product","price":99.99}'
# Output: "Uppdaterar produkt 1"

# Nästa GET kommer från cache med nya värdet!
curl http://localhost:8080/api/products/1
```

## Cache warming

**Cache warming** betyder att du fyller cachen med data innan användare börjar fråga efter den. Istället för att första användaren får vänta på en cache miss, har du redan laddat datan.

### När är cache warming bra?

✅ **Vid app-start** - ladda de vanligaste produkterna
✅ **Schemalagda jobb** - uppdatera cache varje natt
✅ **Efter deployment** - förhindra "cold start" problem
✅ **Populära produkter** - ladda top 100 i förväg

### Exempel: Warm cache vid startup

```java
@Service
public class CacheWarmupService {
    
    @Autowired
    private ProductService productService;
    
    @Autowired
    private ProductRepository productRepository;
    
    /**
     * Körs när appen startar
     */
    @PostConstruct
    public void warmUpCache() {
        System.out.println("🔥 Warming up cache...");
        
        // Ladda de 100 mest populära produkterna
        List<Product> topProducts = productRepository.findTop100ByOrderByViewsDesc();
        
        for (Product product : topProducts) {
            productService.getProduct(product.getId());
        }
        
        System.out.println("✅ Cache warmed up with " + topProducts.size() + " products");
    }
}
```

### Schemalagd cache warming

```java
@Service
public class CacheScheduler {
    
    @Autowired
    private ProductService productService;
    
    /**
     * Kör varje natt kl 02:00
     */
    @Scheduled(cron = "0 0 2 * * *")
    public void refreshCache() {
        System.out.println("🔄 Refreshing cache...");
        
        // Töm gamla cachen
        cacheManager.getCache("products").clear();
        
        // Ladda ny data
        List<Product> products = productRepository.findAll();
        for (Product product : products) {
            productService.getProduct(product.getId());
        }
        
        System.out.println("✅ Cache refreshed");
    }
}

// Glöm inte @EnableScheduling!
@SpringBootApplication
@EnableCaching
@EnableScheduling
public class MyApplication {
    // ...
}
```

### Selektiv warming

Istället för att cacha ALLT, cacha bara det som verkligen behövs:

```java
@PostConstruct
public void warmUpCache() {
    // Bara produkter med många visningar
    productRepository.findByViewsGreaterThan(1000)
        .forEach(p -> productService.getProduct(p.getId()));
    
    // Bara aktiva kampanjer
    campaignRepository.findByActiveTrue()
        .forEach(c -> campaignService.getCampaign(c.getId()));
}
```

## Monitoring och metrics

Hur vet du att din cache faktiskt fungerar? Du måste mäta!

### Cache hit ratio

Den viktigaste metriken är **cache hit ratio** - hur många requests som träffar cachen vs. totalt:

```
Hit Ratio = Cache Hits / (Cache Hits + Cache Misses)
```

**Exempel:**
- 900 cache hits
- 100 cache misses
- Hit ratio = 900 / (900 + 100) = 0.90 = **90%**

### Vad är en bra hit ratio?

Det beror på use caset, men generellt:

- **>80%** = Utmärkt! Din cache fungerar bra
- **50-80%** = Okej, kan förbättras
- **<50%** = Något är fel - kanske för kort TTL eller fel data cachas

### Logga cache-statistik

Spring Cache har built-in support för metrics med Micrometer:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# Aktivera cache metrics
management.endpoints.web.exposure.include=health,metrics,caches
management.metrics.enable.cache=true
```

Nu kan du se statistik på:
```
http://localhost:8080/actuator/metrics/cache.gets?tag=name:products
http://localhost:8080/actuator/metrics/cache.puts?tag=name:products
```

### Custom logging

```java
@Aspect
@Component
public class CacheLoggingAspect {
    
    private static final Logger log = LoggerFactory.getLogger(CacheLoggingAspect.class);
    
    @Around("@annotation(cacheable)")
    public Object logCacheUsage(ProceedingJoinPoint joinPoint, Cacheable cacheable) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long duration = System.currentTimeMillis() - start;
        
        if (duration < 10) {
            log.info("✅ Cache HIT - {}() took {}ms", methodName, duration);
        } else {
            log.warn("❌ Cache MISS - {}() took {}ms", methodName, duration);
        }
        
        return result;
    }
}
```

### Redis INFO kommando

Redis själv har massa bra statistik:

```bash
redis-cli INFO stats

# Viktiga metrics:
# - keyspace_hits: antal träffar
# - keyspace_misses: antal missar
# - used_memory: minne som används
# - evicted_keys: nycklar som kastats ut pga memory limit
```

### Verktyg för monitoring

- **Redis Commander** - web UI för Redis
- **RedisInsight** - officiell Redis GUI
- **Grafana + Prometheus** - för production monitoring
- **Spring Boot Actuator** - built-in metrics

## Best practices

### 1. Cacha det som är värt att cacha

Undvik att cacha data som:
- Ändras hela tiden
- Är unik per request
- Är snabb att hämta ändå

```java
// DÅLIGT - ändras konstant
@Cacheable("stock")
public int getCurrentStock(Long productId) { ... }

// BRA - ändras sällan
@Cacheable("productInfo")
public Product getProductInfo(Long productId) { ... }
```

### 2. Använd lämpliga TTL-värden

```java
// För kort TTL - ingen poäng med cache
@Cacheable(value = "products")  // TTL = 10 sekunder
// Folk kollar på samma produkt inom sekunder, ej minuter

// För lång TTL - stale data
@Cacheable(value = "stockLevels")  // TTL = 24 timmar
// Lager kan ändras varje minut!

// LAGOM
@Cacheable(value = "products")  // TTL = 1 timme
@Cacheable(value = "stockLevels")  // TTL = 5 minuter
```

### 3. Hantera cache-failures gracefully

Om Redis är nere ska appen inte krascha:

```java
@Bean
public CacheErrorHandler errorHandler() {
    return new CacheErrorHandler() {
        @Override
        public void handleCacheGetError(RuntimeException exception, Cache cache, Object key) {
            log.warn("Cache GET error - falling back to database", exception);
            // Fortsätt utan cache
        }

        @Override
        public void handleCachePutError(RuntimeException exception, Cache cache, Object key, Object value) {
            log.warn("Cache PUT error - continuing anyway", exception);
            // Fortsätt utan att spara i cache
        }

        @Override
        public void handleCacheEvictError(RuntimeException exception, Cache cache, Object key) {
            log.warn("Cache EVICT error - continuing anyway", exception);
        }

        @Override
        public void handleCacheClearError(RuntimeException exception, Cache cache) {
            log.warn("Cache CLEAR error - continuing anyway", exception);
        }
    };
}
```

### 4. Versionera cache-keys vid datastrukturändringar

Om du ändrar strukturen på Product, kan gammal cachad data krascha:

```java
// Version 1
public class Product {
    String name;
    BigDecimal price;
}

// Version 2 - la till fält
public class Product {
    String name;
    BigDecimal price;
    String description;  // NYTT FÄLT
    LocalDateTime createdAt;  // NYTT FÄLT
}
```

Lösning - versionera cache-namnet:

```java
@Cacheable("products:v2")  // Ändra från "products" till "products:v2"
public Product getProduct(Long id) { ... }
```

### 5. Övervaka minnesanvändning

Redis har begränsat minne. Sätt en max-memory policy:

```bash
# I redis.conf
maxmemory 256mb
maxmemory-policy allkeys-lru  # Kasta ut minst använda först
```

Alternativ:
- `noeviction` - returnera error när fullt
- `allkeys-lru` - ta bort minst nyligen använda (bra default)
- `volatile-lru` - ta bort bland keys med TTL
- `allkeys-random` - ta bort random keys

### 6. Använd rätt serialisering

JSON är vanligast för readability:

```java
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
        .serializeValuesWith(
            RedisSerializationContext.SerializationPair.fromSerializer(
                new GenericJackson2JsonRedisSerializer()
            )
        );
}
```

### 7. Tänk på key-naming

Använd tydliga, strukturerade keys:

```java
// BRA
"product:123"
"user:456:profile"
"cart:session:abc123"

// DÅLIGT
"p123"
"u456"
"abc123"
```

## Vanliga fallgropar

### 1. Glömma att invalidera cachen

```java
// DÅLIGT - cachen uppdateras aldrig!
public void updateProduct(Product product) {
    productRepository.save(product);
    // Ingen cache invalidation!
}

// BRA
@CacheEvict(value = "products", key = "#product.id")
public void updateProduct(Product product) {
    productRepository.save(product);
}
```

### 2. Cacha data som ändras ofta

```java
// DÅLIGT - lagerantal ändras varje order
@Cacheable("stock")
public int getStockLevel(Long productId) {
    return inventory.getCurrentStock(productId);
}
```

Om du MÅSTE cacha det, använd mycket kort TTL (sekunder, inte minuter).

### 3. För generösa cache-keys

```java
// DÅLIGT - ALLA användare får samma cachade resultat!
@Cacheable("orders")
public List<Order> getMyOrders() {
    Long userId = getCurrentUserId();
    return orderRepository.findByUserId(userId);
}

// BRA - olika cache per användare
@Cacheable(value = "orders", key = "#userId")
public List<Order> getUserOrders(Long userId) {
    return orderRepository.findByUserId(userId);
}
```

### 4. Cacha känslig data i Redis utan kryptering

```java
// RISKABELT
@Cacheable("users")
public User getUserWithPassword(Long id) {
    return userRepository.findById(id);  // Innehåller lösenord!
}

// BÄTTRE - cacha bara säker data
@Cacheable("users")
public UserDTO getUserProfile(Long id) {
    User user = userRepository.findById(id);
    return new UserDTO(user.getName(), user.getEmail());  // Inget lösenord
}
```

### 5. Ingen monitoring

Du tror din cache fungerar, men har den verkligen bra hit ratio? Mät alltid!

### 6. Cache utan TTL

```java
// FARLIGT - data blir aldrig uppdaterad
@Cacheable("products")  // Ingen TTL!
public Product getProduct(Long id) { ... }
```

Sätt alltid en TTL som backup, även om du invaliderar manuellt.

### 7. Cache av stora objekt

```java
// DÅLIGT - 10MB per produkt!
@Cacheable("products")
public Product getProductWithAllImages(Long id) {
    Product p = productRepository.findById(id);
    p.setImages(loadAllHighResImages(id));  // 10MB data!
    return p;
}

// BRA - bara den data som behövs
@Cacheable("products")
public ProductSummary getProductSummary(Long id) {
    return productRepository.findSummaryById(id);  // 1KB data
}
```

## Sammanfattning

Låt oss sammanfatta allt vi lärt oss om caching!

### Nyckelpunkter

1. **Cache är som att komma ihåg** istället för att slå upp varje gång
2. **Performance boost är enorm** - 100x snabbare är inte ovanligt
3. **Redis är standard** för distribuerad caching i Spring Boot
4. **Cache-Aside är vanligast** - app pratar med både cache och DB
5. **TTL är din vän** - sätt alltid en timeout som backup
6. **Invalidation är svårt** - planera noga när och hur du uppdaterar cachen
7. **Mät alltid** - cache hit ratio berättar om det fungerar

### När ska du använda caching?

✅ **Read-heavy workloads** - mycket läsning, lite skrivning
✅ **Dyra queries** - joins över många tabeller
✅ **Externa API-anrop** - långsamma eller rate-limitade
✅ **Statisk data** - produkter, konfiguration, användarinfo

### Quick reference: Spring annotations

```java
@Cacheable("products")  // Hämta från cache, spara om miss
public Product get(Long id) { ... }

@CachePut("products")   // Uppdatera cache
public Product update(Product p) { ... }

@CacheEvict("products") // Ta bort från cache
public void delete(Long id) { ... }
```

### Setup checklist

- [ ] Lägg till dependencies (spring-boot-starter-cache, spring-boot-starter-data-redis)
- [ ] Konfigurera Redis connection i application.properties
- [ ] Lägg till @EnableCaching på Application-klassen
- [ ] Sätt lämpliga TTL-värden per cache
- [ ] Lägg till @Cacheable på GET-metoder
- [ ] Lägg till @CacheEvict på UPDATE/DELETE-metoder
- [ ] Implementera cache error handling
- [ ] Sätt upp monitoring för hit ratio
- [ ] Testa att cache faktiskt fungerar!

### Viktiga frågor att ställa sig

1. **Hur ofta ändras denna data?** → Bestämmer TTL
2. **Hur kritiskt är det med fräsch data?** → Påverkar cache-strategi
3. **Hur många användare läser samma data?** → Påverkar cache-nytta
4. **Vad händer om cachen är gammal?** → Risk assessment
5. **Hur invaliderar vi när data ändras?** → Invalidation-strategi

Nu har du alla verktyg för att bygga snabba, skalbara applikationer med caching! 🚀

## Övningsuppgifter

### Uppgift 1: Räkna ut cache-vinsten

Din databas-query tar **50ms** och ett cache-lookup tar **1ms**.

Du har 1000 requests under en dag för samma data.

**a)** Hur lång total tid tar det UTAN cache?

**b)** Hur lång total tid tar det MED cache och 90% hit ratio (d.v.s. 900 hits, 100 misses)?

**c)** Hur mycket snabbare är det med cache?

<details>
<summary>Visa lösning</summary>

**a) Utan cache:**
```
1000 requests × 50ms = 50,000ms = 50 sekunder
```

**b) Med cache (90% hit ratio):**
```
900 cache hits: 900 × 1ms = 900ms
100 cache misses: 100 × 50ms = 5,000ms
Total: 900ms + 5,000ms = 5,900ms ≈ 6 sekunder
```

**c) Speedup:**
```
50 sekunder / 6 sekunder ≈ 8.3x snabbare!
```

</details>

### Uppgift 2: Skriv @Cacheable annotation

Du har en service-metod som hämtar en användare från databasen:

```java
public User getUserById(Long userId) {
    return userRepository.findById(userId).orElseThrow();
}
```

**Skriv om metoden** med en `@Cacheable` annotation som:
- Använder cache-namnet "users"
- Använder userId som key
- Cachar INTE om userId är null

<details>
<summary>Visa lösning</summary>

```java
@Cacheable(value = "users", key = "#userId", condition = "#userId != null")
public User getUserById(Long userId) {
    return userRepository.findById(userId).orElseThrow();
}
```

Alternativt, enklare version (funkar också):
```java
@Cacheable("users")
public User getUserById(Long userId) {
    return userRepository.findById(userId).orElseThrow();
}
```

</details>

### Uppgift 3: Horisontell skalning med cache

Du har en app som håller användarnas sessions i ett HashMap i minnet:

```java
@Service
public class SessionService {
    private Map<String, UserSession> sessions = new HashMap<>();
    
    public void createSession(String sessionId, UserSession session) {
        sessions.put(sessionId, session);
    }
    
    public UserSession getSession(String sessionId) {
        return sessions.get(sessionId);
    }
}
```

**Förklara:**

**a)** Varför är detta ett problem när du vill köra flera instanser av appen?

**b)** Hur skulle Redis lösa problemet?

**c)** Vad händer när en instans kraschar i HashMap-lösningen vs Redis-lösningen?

<details>
<summary>Visa lösning</summary>

**a) Problem med HashMap:**
- Varje instans har sin egen HashMap i minnet
- Om User loggar in på Instance 1, sparas sessionen bara där
- Nästa request kan gå till Instance 2 som inte känner till sessionen
- User tror de är utloggade och måste logga in igen!

```
User login → [Instance 1: session ABC finns] 
User request → [Instance 2: session ABC finns INTE] → 403 Unauthorized
```

**b) Redis-lösning:**
```java
@Service
public class SessionService {
    @Autowired
    private RedisTemplate<String, UserSession> redis;
    
    public void createSession(String sessionId, UserSession session) {
        redis.opsForValue().set(sessionId, session, Duration.ofHours(1));
    }
    
    public UserSession getSession(String sessionId) {
        return redis.opsForValue().get(sessionId);
    }
}
```

Nu spelar det ingen roll vilken instans som tar emot requesten - alla läser från samma Redis!

```
User login → [Instance 1] → [Redis: session ABC]
User request → [Instance 2] → [Redis: session ABC] → ✅ Funkar!
```

**c) Vid krasch:**

**HashMap:** Om Instance 1 kraschar försvinner alla sessions för användare som loggade in där. De måste logga in igen.

**Redis:** Om en instans kraschar finns sessions kvar i Redis. Andra instanser kan fortsätta servera användare utan problem. (Om Redis kraschar är det förstås ett problem - då kan du ha Redis i cluster-mode med replicas.)

</details>

### Uppgift 4: Cache-strategi

Du bygger en e-handelsplats med produkter. Produktkatalogen har 10,000 produkter.

**Situation:**
- Produkter läses 100,000 gånger per dag
- Produkter uppdateras ca 50 gånger per dag (pris, beskrivning)
- 80% av läsningarna är för samma 100 "populära" produkter
- Databasen börjar bli överbelastad

**Designa en cache-strategi:**

**a)** Vilken data ska cachas?

**b)** Vilket TTL-värde skulle du välja?

**c)** Hur ska du invalidera cachen när en produkt uppdateras?

**d)** Ska du använda cache warming? I så fall, hur?

**e)** Vad händer om Redis går ner? Hur hanterar du det?

<details>
<summary>Visa lösning</summary>

**a) Vad ska cachas:**
```java
@Cacheable("products")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}

@Cacheable("popularProducts")
public List<Product> getPopularProducts() {
    return productRepository.findTop100ByOrderBySalesDesc();
}
```

Cacha individuella produkter OCH listan med populära produkter.

**b) TTL-värde:**

För individuella produkter: **1 timme**
- Uppdateras bara 50 gånger/dag (varannan timme i snitt)
- Okej om pris är lite gammalt i upp till 1 timme
- Ger fortfarande stor cache-nytta

För populära-listan: **30 minuter**
- Listan kan ändras oftare
- Mer kritiskt att den är uppdaterad

```java
@Bean
public RedisCacheManagerBuilderCustomizer cacheConfig() {
    return builder -> builder
        .withCacheConfiguration("products",
            RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1)))
        .withCacheConfiguration("popularProducts",
            RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30)));
}
```

**c) Invalidation vid uppdatering:**
```java
@Caching(
    put = @CachePut(value = "products", key = "#product.id"),
    evict = @CacheEvict(value = "popularProducts", allEntries = true)
)
public Product updateProduct(Product product) {
    return productRepository.save(product);
}
```

- Uppdatera den specifika produkten i cache med @CachePut
- Invalidera hela "popularProducts" listan (den kan ha ändrats)

**d) Cache warming:**

Ja! Ladda de 100 populära produkterna vid startup:

```java
@PostConstruct
public void warmCache() {
    List<Product> popular = productRepository.findTop100ByOrderBySalesDesc();
    for (Product p : popular) {
        productService.getProduct(p.getId()); // Laddar till cache
    }
    log.info("Cache warmed with {} products", popular.size());
}
```

Detta förhindrar att första 100 användarna får långsamma svar.

**e) Redis-failure handling:**
```java
@Bean
public CacheErrorHandler errorHandler() {
    return new CacheErrorHandler() {
        @Override
        public void handleCacheGetError(RuntimeException e, Cache cache, Object key) {
            log.error("Redis GET error - falling back to database", e);
            // Fortsätt utan cache, hämta från DB
        }
        
        @Override
        public void handleCachePutError(RuntimeException e, Cache cache, Object key, Object value) {
            log.error("Redis PUT error - continuing without caching", e);
            // Fortsätt utan att cacha
        }
        // ... implementera övriga metoder
    };
}
```

Med detta kommer appen fortsätta fungera (men långsammare) om Redis går ner.

</details>

---

Grattis! Nu kan du caching! 🎉 Nästa steg är att faktiskt implementera det i ditt projekt och se performance-skillnaden med egna ögon.

