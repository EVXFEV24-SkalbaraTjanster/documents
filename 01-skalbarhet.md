# Skalbarhet och Instansiering

## Introduktion

Välkommen till kursen om skalbara tjänster! I det här dokumentet ska vi prata om vad skalbarhet egentligen är, varför det är viktigt, och hur man tänker när man bygger system som ska klara av att växa.

## Varför behöver vi skalbarhet?

Tänk dig att du byggt en superbra app som plötsligt går viralt. I går hade du 50 användare - idag har du 5000. Din server klarar inte av trycket och kraschar. Inte så kul, eller hur?

Det är här skalbarhet kommer in i bilden. Det handlar om att din tjänst ska klara av att hantera fler användare utan att gå ner eller bli långsam som sirap.

### Verkliga exempel

- **Netflix** hanterar miljontals användare samtidigt som streamar filmer
- **Instagram** måste klara av att ladda upp tusentals bilder per sekund
- **Spotify** spelar musik för användare över hela världen samtidigt

Alla dessa tjänster har något gemensamt: de är byggda för att skala.

## Vad är skalbarhet?

Skalbarhet är systemets förmåga att hantera ökad belastning. Det kan handla om:

- Fler användare
- Mer data
- Fler requests per sekund
- Mer komplex logik

Ett skalbart system kan växa utan att behöva skrivas om från grunden.

## Två sätt att skala

### Vertikal skalning (Scale Up)

**Vertikal skalning** betyder att du uppgraderar din befintliga server. Mer RAM, snabbare CPU, större hårddisk.

**Fördelar:**

- Enkelt - ingen kodändring behövs
- Ingen komplex arkitektur
- Funkar bra för många mindre applikationer

**Nackdelar:**

- Det finns ett tak - du kan inte uppgradera i all evighet
- Dyrt - kraftfull hårdvara kostar mycket
- Single point of failure - om servern går ner är allt nere
- Downtime vid uppgradering

**Exempel:**

```
Från: 4GB RAM, 2 CPU cores
Till:  32GB RAM, 16 CPU cores
```

### Horisontell skalning (Scale Out)

**Horisontell skalning** betyder att du lägger till fler servrar. Istället för en jätteserver har du flera mindre som delar på jobbet.

**Fördelar:**

- Nästan obegränsad skalning - lägg bara till fler servrar
- Billigare - många små servrar är ofta billigare än en jätte
- Redundans - om en server går ner finns det andra
- Ingen downtime vid skalning

**Nackdelar:**

- Mer komplex arkitektur
- Kräver att applikationen är byggd för det
- Behöver load balancing
- Data synkronisering kan vara knepigt

**Exempel:**

```
Från: 1 server
Till:  5 servrar som delar arbetet
```

I den här kursen fokuserar vi mest på horisontell skalning eftersom det är så moderna molnbaserade system brukar byggas.

## Instanser och instansiering

### Vad är en instans?

En **instans** är helt enkelt en körande kopia av din applikation. Om du startar din Spring Boot-app två gånger på olika portar har du två instanser.

```
Instans 1: körs på port 8080
Instans 2: körs på port 8081
Instans 3: körs på port 8082
```

Alla tre kör samma kod, men är separata processer.

### Varför flera instanser?

Med flera instanser kan du:

- **Dela upp trafik** - användare fördelar sig mellan instanserna
- **Hantera mer last** - fler instanser = mer kapacitet
- **Ha backup** - om en instans kraschar finns det andra
- **Uppdatera utan downtime** - starta nya instanser innan du stänger gamla

### Exempel scenario

Säg att din app kan hantera 100 requests per sekund. Om du får 300 requests per sekund behöver du minst 3 instanser.

```
Utan skalning:
[Server] <- 300 req/s = KRASCHAR

Med skalning:
[Server 1] <- 100 req/s ✓
[Server 2] <- 100 req/s ✓
[Server 3] <- 100 req/s ✓
```

## Stateful vs Stateless

Det här är ett av de viktigaste koncepten när man pratar om skalbarhet.

### Stateless services

En **stateless** service sparar inget mellan requests. Varje request är helt oberoende.

**Exempel:**

```java
@GetMapping("/calculate")
public int add(@RequestParam int a, @RequestParam int b) {
    return a + b;  // Inget sparas mellan calls
}
```

Varje gång någon anropar `/calculate` så räknar vi ut summan och skickar tillbaka den. Vi sparar inget minne om tidigare requests.

**Fördelar med stateless:**

- Lätt att skala horisontellt
- Vilken instans som helst kan hantera vilken request som helst
- Inga problem om en instans kraschar

### Stateful services

En **stateful** service sparar information mellan requests, ofta i minnet.

**Exempel:**

```java
private Map<String, User> loggedInUsers = new HashMap<>();

@PostMapping("/login")
public void login(@RequestBody User user) {
    loggedInUsers.put(user.getId(), user);  // Sparas i minnet
}

@GetMapping("/profile")
public User getProfile(@RequestParam String userId) {
    return loggedInUsers.get(userId);  // Hämtar från minnet
}
```

Om användaren loggar in på instans 1 men nästa request går till instans 2, så finns inte användaren där!

**Problem med stateful:**

- Svårt att skala - requests måste gå till rätt instans
- Sticky sessions behövs (användare "fastnar" på en instans)
- Om instans kraschar förlorar du datan

### Hur gör man det stateless?

Istället för att spara i minnet, använd externa system:

- **Databas** - för persistent data
- **Redis/Memcached** - för cache och sessions
- **JWT tokens** - för authentication state

```java
// Stateful (dåligt för skalning)
private Map<String, String> cache = new HashMap<>();

// Stateless (bra för skalning)
@Autowired
private RedisTemplate<String, String> redis;
```

Nu spelar det ingen roll vilken instans som hanterar requesten - alla kan komma åt Redis.

## Load balancing - en snabb introduktion

När du har flera instanser behöver du något som fördelar trafiken mellan dem. Det kallas **load balancing**.

```
             [Load Balancer]
                    |
     +--------------+--------------+
     |              |              |
[Instans 1]    [Instans 2]    [Instans 3]
```

Load balancern tar emot alla requests och skickar dem vidare till en av instanserna. Vi går igenom det här mer detaljerat i nästa dokument!

## Flaskhalsar (Bottlenecks)

När du skalar din applikation kommer du förr eller senare hitta **flaskhalsar** - delar av systemet som inte klarar av lasten.

### Vanliga flaskhalsar:

**1. Databas**
Problem: Alla instanser pratar med samma databas som blir överbelastad.
Lösningar:

- Read replicas (läsande kopior av databasen)
- Database sharding (dela upp datan)
- Caching (minska antalet databasanrop)

**2. API-anrop till externa tjänster**
Problem: Varje request anropar en långsam extern API.
Lösningar:

- Caching av svar
- Asynkron kommunikation
- Rate limiting

**3. File I/O**
Problem: Många instanser försöker läsa/skriva samma filer.
Lösningar:

- Använd objektlagring (S3, Azure Blob)
- CDN för statiska filer
- Databas istället för filsystem

**4. Memory**
Problem: Applikationen använder för mycket minne.
Lösningar:

- Optimera datastrukturer
- Använd streaming för stora dataset
- Öka memory limit

## Mätning och monitoring

Du kan inte förbättra det du inte mäter. Viktiga metrics att hålla koll på:

- **Response time** - hur lång tid tar en request?
- **Throughput** - hur många requests/sekund klarar systemet?
- **Error rate** - hur många requests misslyckas?
- **CPU/Memory usage** - hur mycket resurser används?
- **Database query time** - hur snabba är databasanropen?

Verktyg som kan hjälpa: Prometheus, Grafana, New Relic, Datadog.

## Design principer för skalbara system

### 1. Design för failure

Anta att saker kommer gå sönder. En instans kommer krascha, ett nätverk kommer gå ner, en databas kommer bli otillgänglig.

**Implementera:**

- Health checks
- Graceful degradation
- Retry logic
- Circuit breakers

### 2. Gör det stateless

Som vi pratade om tidigare - undvik att spara state i applikationen. Använd externa system för det.

### 3. Asynkron kommunikation

Allt behöver inte hända direkt. Använd message queues för tasks som kan vänta.

```
User action -> Queue -> Worker process
```

Användaren får snabbt svar, och jobbet görs i bakgrunden.

### 4. Cache aggressivt

Många requests vill ha samma data. Varför fråga databasen 1000 gånger när du kan fråga en gång och cacha resultatet?

### 5. Dela upp systemet

Istället för en stor monolitisk applikation, dela upp i mindre **microservices**. Då kan du skala varje del oberoende.

## Cloud och auto-scaling

I molnmiljöer (AWS, Azure, Google Cloud) kan du sätta upp **auto-scaling** - systemet lägger automatiskt till eller tar bort instanser baserat på belastning.

```
Låg trafik:    [Instans 1] [Instans 2]
Hög trafik:    [Instans 1] [Instans 2] [Instans 3] [Instans 4] [Instans 5]
Låg trafik:    [Instans 1] [Instans 2]
```

Det här sparar pengar - du betalar bara för det du använder.

### Scaling triggers

Auto-scaling kan baseras på:

- CPU användning (>70% → skala upp)
- Antal requests per sekund
- Response time
- Queue length
- Custom metrics

## Sammanfattning

**Skalbarhet** handlar om att bygga system som kan växa:

- **Vertikal skalning** = kraftigare server
- **Horisontell skalning** = fler servrar (bättre!)
- **Instanser** = körande kopior av din app
- **Stateless** = spara inget i minnet, använd externa system
- **Load balancing** = fördela trafik mellan instanser
- **Flaskhalsar** = hitta och lös dem en i taget

I nästa dokument går vi igenom load balancing mer detaljerat. Sedan kommer vi lära oss Docker, som gör det enkelt att köra flera instanser av din applikation!

## Övningsuppgifter

1. **Diskussion**: Hitta tre verkliga tjänster (webbplatser/appar) och fundera på hur de skalar. Vilka flaskhalsar kan de ha?

2. **Analys**: Din Spring Boot-app sparar användarsessions i en HashMap i minnet. Förklara varför det är problematiskt när du vill köra flera instanser.

3. **Design**: Rita ett diagram över hur ett system med 3 instanser, en load balancer, en databas och Redis för caching skulle se ut.

4. **Räkna**: Om varje instans klarar 50 requests/sekund och du får 400 requests/sekund, hur många instanser behöver du minst? Varför kanske du vill ha fler än minimumet?
