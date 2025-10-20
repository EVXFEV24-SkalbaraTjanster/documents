# Load Balancing

## Introduktion

I förra dokumentet pratade vi om horisontell skalning - att köra flera instanser av din applikation. Men här kommer frågan: hur vet användarna vilken instans de ska prata med? Och hur fördelar vi trafiken så att alla instanser får en rättvis del av jobbet?

Det är här **load balancing** kommer in i bilden!

Tänk dig att du står i kö på ICA. Det finns tre kassor öppna. Väljer alla samma kassa? Nej, folk fördelar sig mellan kassorna, och om en kassa har jättelång kö så väljer du en annan. Det är exakt vad en load balancer gör - den är "trafikpolisen" som ser till att fördelningen blir bra.

### Varför behöver vi load balancing?

När du har flera instanser av din app behöver du:

- **En enda ingång** - användare ska inte behöva veta vilka servrar som finns
- **Automatisk fördelning** - requests ska fördelas smart mellan instanserna
- **Felhantering** - om en instans kraschar ska trafiken gå till de andra
- **Jämn belastning** - ingen instans ska bli överbelastad medan andra står och gör ingenting

### Verkliga exempel

- **Netflix** använder load balancing för att fördela miljontals streamar mellan sina servrar
- **Facebook** balanserar requests mellan tusentals servrar världen över
- **Din bank** använder load balancing för att klara av alla samtidiga inloggningar

## Hur fungerar det?

Grundkonceptet är enkelt: load balancern sitter mellan användarna och dina applikationsinstanser.

```
                 Internet
                    |
                    |
              [Load Balancer]
              (port 80/443)
                    |
    +---------------+---------------+
    |               |               |
[Instans 1]    [Instans 2]    [Instans 3]
(port 8080)    (port 8080)    (port 8080)
```

### Så här går det till:

1. **Användare** skickar en request till din domän (t.ex. `api.minapp.se`)
2. **DNS** pekar på load balancerns IP-adress
3. **Load balancer** tar emot requesten
4. **Algoritm** väljer vilken instans som ska hantera requesten
5. **Request vidarebefordras** till vald instans
6. **Instans** processar requesten och skickar tillbaka svar
7. **Load balancer** skickar svaret vidare till användaren

Användaren ser aldrig vilken instans som hanterade requesten - allt ser ut som en enda server!

### Layer 4 vs Layer 7

Det finns två huvudtyper av load balancing:

**Layer 4 (TCP/UDP)** - jobbar på transport-lagret

- Snabbare
- Tittar bara på IP-adress och port
- Kan inte fatta beslut baserat på innehållet

**Layer 7 (HTTP/HTTPS)** - jobbar på applikationslagret

- Smartare
- Kan läsa HTTP headers, URLs, cookies
- Kan routa baserat på innehåll (t.ex. `/api/users` → server 1, `/api/orders` → server 2)

I den här kursen fokuserar vi på Layer 7, eftersom det är vanligast för webbapplikationer.

## Load balancing algoritmer

Nu kommer vi till det roliga - hur bestämmer load balancern vilken server som ska få nästa request? Det finns flera olika **algoritmer**, var och en med sina för- och nackdelar.

### Round-robin

Det här är den enklaste algoritmen. Instanserna får trafik i tur och ordning, som ett roterande hjul.

```
Request 1 → Instans 1
Request 2 → Instans 2
Request 3 → Instans 3
Request 4 → Instans 1  (börjar om)
Request 5 → Instans 2
Request 6 → Instans 3
```

**Fördelar:**

- Superenkelt att förstå och implementera
- Rättvis fördelning över tid
- Funkar bra för stateless applikationer
- Låg overhead

**Nackdelar:**

- Tar inte hänsyn till servrarnas faktiska belastning
- Om requests tar olika lång tid blir fördelningen ojämn
- Funkar dåligt om servrarna har olika kapacitet

**När ska du använda det?**
När alla dina instanser är identiska och requests tar ungefär lika lång tid. Det här är standardvalet för de flesta webbapplikationer.

### Least connections

Skickar nya requests till den server som har **minst antal aktiva anslutningar** just nu.

```
Instans 1: 5 aktiva anslutningar
Instans 2: 8 aktiva anslutningar
Instans 3: 3 aktiva anslutningar  ← Nästa request går hit!
```

**Fördelar:**

- Bättre än round-robin om requests tar olika lång tid
- Jämnare faktisk belastning
- Anpassar sig dynamiskt

**Nackdelar:**

- Lite mer overhead (måste räkna anslutningar)
- Kan bli ojämn fördelning om anslutningar är korta

**När ska du använda det?**
När dina requests kan ta väldigt olika lång tid - t.ex. om vissa API-anrop är snabba (lista användare) och andra långsamma (generera rapport).

### IP hash

Använder användarens **IP-adress** för att bestämma vilken server de ska få. Samma IP kommer alltid till samma server.

```
hash(193.12.45.67) % 3 = 1 → Instans 1
hash(81.224.53.11) % 3 = 2 → Instans 2
hash(193.12.45.67) % 3 = 1 → Instans 1 (samma användare, samma server!)
```

**Fördelar:**

- Användare får alltid samma server (bra för stateful apps)
- Enkel session affinity utan cookies
- Konsistent över tid

**Nackdelar:**

- Fördelningen kan bli ojämn (beror på IP-adresser)
- Om en server går ner måste alla dess användare remappas
- Funkar dåligt bakom NAT där många användare har samma IP

**När ska du använda det?**
När du har en stateful applikation som sparar data i minnet och användare måste komma till samma server varje gång. Men helst bör du göra appen stateless istället!

### Weighted round-robin

Som vanlig round-robin, men vissa servrar får **fler requests** än andra.

```
Instans 1: weight = 3
Instans 2: weight = 2
Instans 3: weight = 1

Fördelning: 1, 1, 1, 2, 2, 3, 1, 1, 1, 2, 2, 3, ...
```

**Fördelar:**

- Perfekt när servrar har olika kapacitet
- Enkel att förstå och konfigurera
- Flexibel - du bestämmer fördelningen

**Nackdelar:**

- Måste veta servrars relativa kapacitet
- Statisk - anpassar sig inte automatiskt

**När ska du använda det?**
När du har olika servrar med olika kapacitet. T.ex. en kraftfull server med weight 4 och två mindre med weight 1 vardera.

### Random

Helt slumpmässigt val av server. Enkelt som det låter!

**Fördelar:**

- Superenkelt
- Över tid blir fördelningen ganska jämn

**Nackdelar:**

- Oförutsägbart på kort sikt
- Ingen egentlig fördel över round-robin

**När ska du använda det?**
Egentligen sällan - round-robin är nästan alltid bättre. Men det funkar om du bara vill ha något enkelt.

## Nginx som load balancer

Nu ska vi titta på hur man faktiskt sätter upp load balancing i praktiken. Vi använder **Nginx** - en av de mest populära load balancers som finns.

### Varför Nginx?

- **Populär** - används av miljontals webbplatser
- **Snabb** - klarar tusentals samtidiga anslutningar
- **Gratis** - open source
- **Enkel** - relativt lätt att komma igång med
- **Flexibel** - kan vara web server, reverse proxy, och load balancer

### Grundläggande arkitektur

Nginx fungerar som en **reverse proxy** - den tar emot requests från klienter och skickar vidare till backend-servrar.

```
[Klient] → [Nginx :80] → [Spring Boot :8080]
                       → [Spring Boot :8081]
                       → [Spring Boot :8082]
```

### Viktiga konfigurationsblock

**upstream** - definierar gruppen av backend-servrar

```nginx
upstream backend {
    server server1.example.com;
    server server2.example.com;
}
```

**server** - definierar en virtual host

```nginx
server {
    listen 80;
    server_name example.com;
}
```

**location** - definierar hur olika URLs ska hanteras

```nginx
location / {
    proxy_pass http://backend;
}
```

**proxy_pass** - skickar vidare request till backend

```nginx
proxy_pass http://backend;
```

### Enkel konfiguration

Här är ett minimalt exempel med två backend-servrar:

```nginx
# Definiera gruppen av servrar
upstream myapp {
    server localhost:8080;
    server localhost:8081;
}

# Nginx server som lyssnar på port 80
server {
    listen 80;
    server_name localhost;

    # All trafik skickas till myapp-gruppen
    location / {
        proxy_pass http://myapp;
    }
}
```

Så enkelt kan det vara! Nu fördelas requests automatiskt mellan port 8080 och 8081 med round-robin.

### Välja algoritm

Som standard använder Nginx round-robin. Men du kan lätt ändra:

```nginx
# Round-robin (standard)
upstream myapp {
    server server1:8080;
    server server2:8080;
}

# Least connections
upstream myapp {
    least_conn;
    server server1:8080;
    server server2:8080;
}

# IP hash
upstream myapp {
    ip_hash;
    server server1:8080;
    server server2:8080;
}

# Weighted round-robin
upstream myapp {
    server server1:8080 weight=3;
    server server2:8080 weight=1;
}
```

## Health checks

Det här är superviktigt! Vad händer om en av dina instanser kraschar? Du vill inte att load balancern ska skicka trafik till en död server, eller hur?

### Passiva health checks

Nginx kollar automatiskt om backend-servrar svarar. Om en server inte svarar flera gånger markeras den som "down" och får ingen trafik.

```nginx
upstream myapp {
    server server1:8080 max_fails=3 fail_timeout=30s;
    server server2:8080 max_fails=3 fail_timeout=30s;
}
```

**max_fails** - hur många misslyckade försök innan servern markeras som down
**fail_timeout** - hur länge servern är markerad som down innan nya försök görs

**Exempel:**

```
Request 1 → server1 → FAIL
Request 2 → server1 → FAIL
Request 3 → server1 → FAIL
(server1 markeras som down i 30 sekunder)
Request 4 → server2 → OK
Request 5 → server2 → OK
(Efter 30 sekunder testas server1 igen)
```

### Aktiva health checks (Nginx Plus)

Den gratis versionen av Nginx har bara passiva checks. Men Nginx Plus (kommersiell version) kan aktivt polla servrarna med health checks.

```nginx
upstream myapp {
    server server1:8080;
    server server2:8080;
    
    # Kräver Nginx Plus
    health_check interval=5s fails=3 passes=2;
}
```

Det här skickar health check requests var 5:e sekund. Om 3 misslyckande i rad → down. Om 2 lyckade i rad → up igen.

### Health check endpoints

Din Spring Boot-app bör ha en health check endpoint:

```java
@RestController
public class HealthController {
    
    @GetMapping("/health")
    public ResponseEntity<String> health() {
        // Kolla att allt är OK
        if (database.isConnected() && allServicesRunning()) {
            return ResponseEntity.ok("OK");
        }
        return ResponseEntity.status(503).body("UNHEALTHY");
    }
}
```

Spring Boot Actuator ger dig detta gratis:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health
```

Nu finns `/actuator/health` som returnerar server-status!

## Sticky sessions (Session affinity)

Ibland vill du att en användare alltid ska komma till samma server. Det kallas **sticky sessions** eller **session affinity**.

### Varför behövs det?

Om din app är **stateful** och sparar användardata i minnet:

```
Användare loggar in → Instans 1 (sparar session i minnet)
Nästa request       → Instans 2 (vet inget om användaren!) ❌
```

Med sticky sessions:

```
Användare loggar in → Instans 1 (sparar session i minnet)
Nästa request       → Instans 1 (hittar sessionen!) ✓
```

### Hur implementerar man det i Nginx?

**Alternativ 1: IP hash**

```nginx
upstream myapp {
    ip_hash;  # Samma IP → samma server
    server server1:8080;
    server server2:8080;
}
```

**Alternativ 2: Cookie-baserat (Nginx Plus)**

```nginx
upstream myapp {
    server server1:8080;
    server server2:8080;
    
    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

### När INTE använda sticky sessions

Helst ska du **undvika** sticky sessions! De har flera nackdelar:

- **Sämre load balancing** - fördelningen blir ojämn över tid
- **Komplicerar skalning** - svårt att ta bort servrar
- **Problem vid server-krasch** - användare förlorar sina sessions

**Bättre lösning:** Gör appen **stateless**!

```nginx
# Istället för att spara i minnet:
private Map<String, User> sessions = new HashMap<>();

# Använd Redis:
@Autowired
private RedisTemplate<String, User> redis;
```

Nu kan vilken instans som helst hantera vilken request som helst!

## Komplett konfigurationsexempel

Nu sätter vi ihop allt vi lärt oss till en komplett, produktionsklar konfiguration!

### Scenario

Vi har en Spring Boot-app som körs i 3 instanser på olika portar:

- app1: localhost:8080
- app2: localhost:8081
- app3: localhost:8082

```nginx
# nginx.conf - komplett exempel för load balancing

# upstream block - definiera våra backend-servrar
upstream spring_backend {
    # Använd least connections algoritm för jämnare belastning
    least_conn;
    
    # Våra tre Spring Boot-instanser
    server localhost:8080 max_fails=3 fail_timeout=30s;
    server localhost:8081 max_fails=3 fail_timeout=30s;
    server localhost:8082 max_fails=3 fail_timeout=30s;
    
    # Alternativt: ge en server mer vikt om den är kraftigare
    # server localhost:8080 weight=2 max_fails=3 fail_timeout=30s;
    
    # Håll anslutningar öppna för bättre prestanda
    keepalive 32;
}

# Server block - hantera inkommande requests
server {
    # Lyssna på port 80 (HTTP)
    listen 80;
    
    # Domännamn (ändra till ditt)
    server_name myapp.example.com;
    
    # Loggar
    access_log /var/log/nginx/myapp-access.log;
    error_log /var/log/nginx/myapp-error.log;
    
    # Root location - skicka allt till backend
    location / {
        # Skicka requesten till vårt upstream
        proxy_pass http://spring_backend;
        
        # Viktiga headers för att backend ska veta om användaren
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # WebSocket support (om du använder det)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # Health check endpoint - gå direkt till backend utan balancing
    location /health {
        proxy_pass http://spring_backend/actuator/health;
        proxy_set_header Host $host;
        access_log off;  # Logga inte health checks
    }
    
    # Statiska filer kan servas direkt av Nginx (snabbare!)
    location /static/ {
        alias /var/www/myapp/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # API-specifik konfiguration
    location /api/ {
        proxy_pass http://spring_backend;
        
        # Extra headers för API
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # API kan ha längre processing-tid
        proxy_read_timeout 120s;
        
        # CORS headers (om du behöver det)
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
    }
}

# HTTPS server (produktion)
server {
    listen 443 ssl http2;
    server_name myapp.example.com;
    
    # SSL certifikat (använd Let's Encrypt!)
    ssl_certificate /etc/nginx/ssl/myapp.crt;
    ssl_certificate_key /etc/nginx/ssl/myapp.key;
    
    # Moderna SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # Samma proxy-konfiguration som ovan
    location / {
        proxy_pass http://spring_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;  # Viktigt för HTTPS!
    }
}

# Redirect HTTP till HTTPS
server {
    listen 80;
    server_name myapp.example.com;
    return 301 https://$server_name$request_uri;
}
```

### Testa konfigurationen

```bash
# Kolla att konfigurationen är giltig
nginx -t

# Ladda om Nginx med ny konfiguration
nginx -s reload

# Eller starta om helt
systemctl restart nginx
```

### Testa load balancing

```bash
# Skicka några requests och se vilken server som svarar
for i in {1..10}; do
    curl http://localhost/api/info
    sleep 0.5
done
```

Din Spring Boot-app kan returnera vilken instans som svarade:

```java
@GetMapping("/info")
public Map<String, String> info() {
    return Map.of(
        "instance", "app1",  // eller läs från environment variable
        "timestamp", Instant.now().toString()
    );
}
```

## Best practices

Här är några gyllene regler när du jobbar med load balancing:

### 1. Håll instanserna stateless

Det här kan inte upprepas nog! Spara INGENTING viktigt i minnet.

**Dåligt:**

```java
private Map<String, User> activeUsers = new HashMap<>();
```

**Bra:**

```java
@Autowired
private RedisTemplate<String, User> redis;  // Delad mellan alla instanser
```

### 2. Konfigurera alltid health checks

Utan health checks skickar load balancern trafik till döda servrar.

```nginx
upstream myapp {
    server server1:8080 max_fails=2 fail_timeout=10s;
    server server2:8080 max_fails=2 fail_timeout=10s;
}
```

Sätt `max_fails` lågt (2-3) och `fail_timeout` kort (10-30s) för snabb failover.

### 3. Välj rätt algoritm

- **Stateless API?** → Round-robin (standard) är perfekt
- **Requests med varierande längd?** → Least connections
- **Måste ha sticky sessions?** → IP hash (men gör helst appen stateless!)

### 4. Monitorera fördelningen

Se till att trafiken faktiskt fördelas jämnt:

```bash
# Räkna requests per instans i access log
grep "server1:8080" access.log | wc -l
grep "server2:8080" access.log | wc -l
grep "server3:8080" access.log | wc -l
```

Alla ska ha ungefär lika många requests!

### 5. Planera för failure

Vad händer om en instans kraschar? Du bör ha:

- Minst 3 instanser (för redundans)
- Automatisk restart av krashade instanser
- Alerting när instanser går ner
- Capacity för att klara trafiken även med färre instanser

**Tumregel:** Om du har N instanser bör du klara trafiken med N-1 instanser.

### 6. Använd connection pooling

```nginx
upstream myapp {
    server server1:8080;
    server server2:8080;
    keepalive 32;  # Behåll 32 connections öppna per worker
}
```

Det här minskar latency genom att återanvända TCP-connections.

### 7. Sätt vettiga timeouts

```nginx
proxy_connect_timeout 5s;   # Snabbt connect
proxy_send_timeout 30s;     # Skicka request
proxy_read_timeout 30s;     # Vänta på svar
```

Om din app behöver längre tid för vissa endpoints, konfigurera dem separat:

```nginx
location /api/reports {
    proxy_pass http://myapp;
    proxy_read_timeout 300s;  # 5 minuter för stora rapporter
}
```

### 8. Logga rätt information

Inkludera vilken backend-server som hanterade requesten:

```nginx
log_format upstreamlog '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $body_bytes_sent '
                       '"$upstream_addr" "$upstream_response_time"';

access_log /var/log/nginx/access.log upstreamlog;
```

Nu kan du se vilken server som svarade och hur lång tid det tog!

## I Docker Compose

Load balancing blir extra användbart när du jobbar med Docker! Men det tar vi i nästa dokument.

Kort teaser: I docker-compose kan vi enkelt sätta upp nginx som load balancer för våra instanser:

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - app1
      - app2
      - app3

  app1:
    image: myapp:latest

  app2:
    image: myapp:latest

  app3:
    image: myapp:latest
```

Nginx upptäcker automatiskt alla tre instanserna och balanserar mellan dem. Coolt, eller hur? Vi går igenom det i detalj i docker-compose-dokumentet!

## Sammanfattning

**Load balancing** är nyckeln till att få horisontell skalning att fungera:

- **Load balancer** = trafikpolis som fördelar requests mellan instanser
- **Round-robin** = enklast, fungerar för de flesta stateless appar
- **Least connections** = bättre när requests tar olika lång tid
- **IP hash** = för sticky sessions (men gör helst appen stateless!)
- **Nginx** = populär, snabb, gratis load balancer
- **Health checks** = kritiskt för att upptäcka och undvika döda servrar
- **Stateless** = gör instanserna utbytbara, använd Redis/databas för state

Nästa steg: Docker och containerization, där load balancing blir ännu enklare!

## Övningsuppgifter

1. **Rita diagram**: Rita ett system med en load balancer, 4 backend-instanser, en databas och Redis. Visa hur en user-request flödar genom systemet.

2. **Algoritm-analys**: Du har en app där 90% av requests är snabba (10ms) men 10% är långsamma (5s). Förklara varför round-robin kan ge ojämn belastning och vilken algoritm som är bättre.

3. **Nginx-konfiguration**: Skriv en nginx upstream-block för 3 servrar där:
   - En server är dubbelt så kraftfull som de andra
   - Health checks ska misslyckas efter 2 fel
   - Servrar ska testas igen efter 20 sekunder

4. **Stateful problem**: Din app sparar kundvagnar i minnet (HashMap). Du har 3 instanser med round-robin. Användaren lägger till vara → instans 1. Sedan går till kassan → instans 2. Vad händer? Hur löser du det?

5. **Diskussion**: Varför är sticky sessions oftast en dålig idé? Ge minst tre konkreta problem de orsakar.

6. **Monitoring**: Du misstänker att load balancern inte fördelar trafik jämnt. Hur kan du verifiera det? Vilka metrics ska du titta på?
