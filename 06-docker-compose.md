# Docker Compose

## Introduktion

Okej, så du har lärt dig Docker och kan köra containers. Härligt! Men snart märker du något frustrerande: att köra `docker run` för varje container blir jobbigt snabbt.

Tänk dig att du har en Spring Boot-app, en PostgreSQL-databas, Redis för caching, och kanske en Nginx load balancer. Ska du verkligen skriva fyra långa `docker run`-kommandon varje gång du startar projektet? Och hålla reda på alla portar, nätverk och volymer?

```bash
# Detta blir snabbt ohållbart...
docker run -d --name postgres -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres:15
docker run -d --name redis -p 6379:6379 redis:7
docker run -d --name myapp -p 8080:8080 --link postgres --link redis myapp:latest
docker run -d --name nginx -p 80:80 --link myapp nginx:latest
```

Nej tack. Det är här **Docker Compose** kommer in och räddar dagen.

### Vad är Docker Compose?

Docker Compose är ett verktyg för att definiera och köra multi-container Docker-applikationer. Istället för att skriva massa `docker run`-kommandon så beskriver du allt i en enda YAML-fil (`docker-compose.yml`), och sedan startar du allt med ett enda kommando:

```bash
docker-compose up
```

Boom! Alla dina containers startar, får rätt konfiguration, kopplas ihop i ett nätverk, och allt funkar. När du är klar stänger du ner allt lika enkelt:

```bash
docker-compose down
```

Docker Compose är **perfekt för utveckling och testning**. Det gör det enkelt att dela exakt samma miljö med hela teamet - checka in `docker-compose.yml` i git så har alla samma setup.

## Varför Docker Compose?

Låt oss räkna upp varför Docker Compose är så fantastiskt:

**🎯 En fil för allt**
Definiera alla dina containers, nätverk och volymer på ett ställe. Ingen jakt efter gamla `docker run`-kommandon i terminal-historiken.

**⚡ Ett kommando**
`docker-compose up` startar allt. `docker-compose down` stänger ner allt. Så enkelt är det.

**🔗 Automatisk networking**
Alla containers kan prata med varandra via sina namn. Din app når databasen på `jdbc:postgresql://db:5432/mydb` - inget krångel med IP-adresser.

**🌍 Environment management**
Hantera miljövariabler smidigt med `.env`-filer. Olika konfiguration för dev, test och prod.

**👥 Delbart med teamet**
Checka in `docker-compose.yml` i git. Nu kan alla på teamet starta exakt samma miljö med ett kommando. Slut på "it works on my machine".

**🔄 Reproducerbar miljö**
Din lokala utvecklingsmiljö blir identisk med testmiljön. Färre överraskningar när du deployar.

**📈 Enkelt att skala**
Behöver du 3 instanser av din app? `docker-compose up --scale app=3`. Klart.

### När ska man använda Docker Compose?

**✅ Använd Docker Compose för:**

- Lokal utveckling
- Testmiljöer
- CI/CD pipelines
- Mindre produktionssystem
- Proof-of-concepts
- Demos

**❌ Använd INTE Docker Compose för:**

- Stora produktionssystem (använd Kubernetes istället)
- System som behöver auto-scaling
- Multi-host deployments (Compose kör på en maskin)
- När du behöver avancerad orchestration

Docker Compose är fantastiskt för utveckling, men när du ska hantera hundratals containers över många servrar är det dags att kolla på Kubernetes.

## Installation

Goda nyheter: om du har Docker Desktop (Windows/Mac) så har du redan Docker Compose installerat!

Verifiera att det funkar:

```bash
docker-compose --version
# eller
docker compose version
```

Du borde se något i stil med:

```
Docker Compose version v2.20.0
```

### docker-compose vs docker compose

Notera att det finns två sätt att köra Compose:

- **`docker-compose`** - Äldre standalone-version (v1)
- **`docker compose`** - Nyare version som plugin till Docker CLI (v2)

Båda funkar, men Docker rekommenderar den nya varianten (`docker compose` utan bindestreck). I det här dokumentet använder vi `docker-compose` eftersom det är vanligast, men kommandona är desamma.

Om du använder Linux och inte har Compose, installera det:

```bash
# För Ubuntu/Debian
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

## docker-compose.yml struktur

Docker Compose använder YAML-format. Om du aldrig sett YAML innan: det är som JSON fast enklare att läsa. Viktiga regler:

- **Indentation spelar roll** - använd spaces (oftast 2), aldrig tabs
- **Key: value** - kolon följt av mellanslag
- **Listor** börjar med `-`

En basic `docker-compose.yml` ser ut så här:

```yaml
version: "3.8"

services:
  # Här definierar du dina containers
  app:
    # Konfiguration för app-containern
    image: myapp:latest

  db:
    # Konfiguration för databas-containern
    image: postgres:15

networks:
# Här definierar du nätverk (oftast behövs inte)

volumes:
# Här definierar du volymer för persistent data
```

Det finns fyra huvudsektioner:

1. **`version`** - Vilken version av Compose-formatet (använd '3.8')
2. **`services`** - Dina containers (det viktigaste)
3. **`networks`** - Custom nätverk (oftast inte nödvändigt)
4. **`volumes`** - Named volumes för persistent data

## Services - definiera containers

Varje **service** blir en container när du kör `docker-compose up`. Service-namnet blir även hostname som andra containers kan använda för att nå den.

```yaml
services:
  app: # <- Service namn (blir hostname)
  # ... config för app
  db: # <- Service namn (blir hostname)
# ... config för databas
```

Nu kan din app nå databasen genom att anropa `db` (inte localhost!).

Låt oss gå igenom de viktigaste inställningarna för varje service.

### image - Vilken image

Specificerar vilken Docker image som ska användas:

```yaml
services:
  db:
    image: postgres:15

  redis:
    image: redis:7-alpine
```

Använd alltid **specifika tags** (`postgres:15`) istället för `:latest`. Då vet du exakt vilken version du kör.

### build - Bygg från Dockerfile

Om du vill bygga din egen image från en Dockerfile:

```yaml
services:
  app:
    build: . # Bygg från Dockerfile i current directory

  # Eller mer specifikt:
  app:
    build:
      context: . # Från denna mapp
      dockerfile: Dockerfile # Använd denna Dockerfile
```

Du kan även ha både `build` och `image`:

```yaml
services:
  app:
    build: .
    image: myapp:latest # Ger den byggda imagen detta namn
```

### container_name - Namnge containern

Som standard får containers automatiska namn (`project_service_1`). Du kan sätta eget namn:

```yaml
services:
  app:
    container_name: myapp
```

Då heter containern `myapp` istället för något som `myproject_app_1`.

**Varning:** Om du använder `container_name` kan du inte skala servicen (du kan inte ha två containers med samma namn).

### ports - Port mapping

Mappar portar från host-maskinen till containern. Format: `"HOST:CONTAINER"`

```yaml
services:
  app:
    ports:
      - "8080:8080" # Host port 8080 -> Container port 8080
      - "8443:8443" # Flera portar

  db:
    ports:
      - "5432:5432"
```

Nu kan du nå appen på `localhost:8080` från din host-maskin.

**Tips:** Om du inte behöver nå servicen från host, skippa `ports`. Services kan alltid nå varandra via det interna nätverket.

### environment - Miljövariabler

Sätt miljövariabler på två sätt:

**Variant 1: Lista**

```yaml
services:
  db:
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=secret123
      - POSTGRES_DB=mydb
```

**Variant 2: Map (key: value)**

```yaml
services:
  db:
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: secret123
      POSTGRES_DB: mydb
```

Båda fungerar lika bra. Välj den stil du tycker är snyggast.

**Exempel för Spring Boot:**

```yaml
services:
  app:
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/mydb
      SPRING_DATASOURCE_USERNAME: myuser
      SPRING_DATASOURCE_PASSWORD: secret123
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PORT: 6379
```

### env_file - Ladda från .env fil

Istället för att skriva alla miljövariabler i `docker-compose.yml`, ladda dem från en fil:

```yaml
services:
  app:
    env_file:
      - .env
```

Din `.env`-fil ser ut så här:

```properties
POSTGRES_USER=myuser
POSTGRES_PASSWORD=secret123
DATABASE_URL=jdbc:postgresql://db:5432/mydb
```

**Varför använda .env?**

- **Säkerhet:** Lägg `.env` i `.gitignore` - checka inte in hemligheter
- **Olika miljöer:** `.env.dev`, `.env.prod` för olika konfiguration
- **Delbart:** Skapa `.env.example` med dummy-värden som teamet kan kopiera

### depends_on - Service dependencies

Specificerar att en service behöver andra services:

```yaml
services:
  app:
    build: .
    depends_on:
      - db
      - redis

  db:
    image: postgres:15

  redis:
    image: redis:7
```

Nu startar Docker först `db` och `redis`, och sedan `app`.

**Viktigt att veta:** `depends_on` väntar bara tills containern har **startat**, inte tills den är **redo**. PostgreSQL kan behöva några sekunder efter start innan den accepterar connections. Din app kanske försöker connecta för tidigt och kraschar.

**Lösningar:**

1. **Retry-logic i din app** (bästa lösningen)
2. **Wait scripts** som `wait-for-it.sh`
3. **Healthchecks** (mer om det senare)

### volumes - Montera volymer

Det finns två typer av volymer:

**Named volumes** - Hanteras av Docker:

```yaml
services:
  db:
    volumes:
      - dbdata:/var/lib/postgresql/data # Named volume

volumes:
  dbdata: # Definiera volym här
```

**Bind mounts** - Mappar en mapp från host:

```yaml
services:
  app:
    volumes:
      - ./src:/app/src # Host ./src -> Container /app/src
      - ./target:/app/target
```

Bind mounts är perfekta för utveckling - ändringar i koden syns direkt i containern!

### networks - Anslut till nätverk

```yaml
services:
  app:
    networks:
      - frontend
      - backend

  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

Oftast behöver du inte definiera nätverk - Compose skapar ett default network automatiskt.

### restart - Restart policy

Vad ska hända om containern kraschar?

```yaml
services:
  app:
    restart: unless-stopped
```

**Alternativ:**

- `no` - Starta aldrig om automatiskt (default)
- `always` - Starta alltid om när den stoppar
- `on-failure` - Starta om bara vid fel (exit code != 0)
- `unless-stopped` - Starta alltid om, förutom om du stoppat den manuellt

För produktion, använd `unless-stopped` eller `always`.

### command - Override CMD

Overrida default-kommandot från imagen:

```yaml
services:
  app:
    image: openjdk:17
    command: java -jar /app/myapp.jar --spring.profiles.active=dev
```

## Enkelt exempel - Spring Boot + PostgreSQL

Här är ett komplett fungerande exempel:

```yaml
version: "3.8"

services:
  # Spring Boot applikation
  app:
    build: . # Bygg från Dockerfile i current directory
    container_name: spring-app
    ports:
      - "8080:8080" # Exponera port 8080
    environment:
      # Anslut till PostgreSQL via service-namnet "db"
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/mydb
      SPRING_DATASOURCE_USERNAME: myuser
      SPRING_DATASOURCE_PASSWORD: secret123
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
    depends_on:
      - db # Starta db först
    restart: unless-stopped

  # PostgreSQL databas
  db:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: secret123
      POSTGRES_DB: mydb
    ports:
      - "5432:5432" # Om du vill accessa från host
    volumes:
      - postgres-data:/var/lib/postgresql/data # Persistent data
    restart: unless-stopped

# Named volume för databas-data
volumes:
  postgres-data:
```

**Starta allt:**

```bash
docker-compose up
```

**Notera:**

- Appen ansluter till `jdbc:postgresql://db:5432/mydb` - `db` är service-namnet!
- Databasen får en persistent volym så data överlever restarts
- Både services får restart-policy

## Networks

När du kör `docker-compose up` skapas automatiskt ett **default network** där alla dina services kan prata med varandra.

```yaml
services:
  app:
  # Ingen networks: definierad
  db:
# Ingen networks: definierad
```

Båda hamnar i samma nätverk och kan nå varandra via service-namn. Spring Boot-appen når PostgreSQL på `db:5432`.

### Service namn = hostname

Det magiska är att **service-namnet blir hostname**. Om du har en service som heter `db`, så når andra services den på `db`:

```yaml
services:
  app:
    environment:
      DATABASE_HOST: db # Använd service-namnet!

  db:
    image: postgres:15
```

Inget behov av IP-adresser eller krångliga konfigurationer.

### Custom networks

Ibland vill du ha flera nätverk för att isolera services från varandra:

```yaml
version: "3.8"

services:
  frontend:
    image: nginx
    networks:
      - frontend

  backend:
    image: myapp
    networks:
      - frontend
      - backend

  db:
    image: postgres:15
    networks:
      - backend # Endast backend-nätverk

networks:
  frontend: # Definiera nätverken
  backend:
```

Nu kan:

- `frontend` nå `backend`
- `backend` nå både `frontend` och `db`
- Men `frontend` kan INTE nå `db` direkt

Det här är bra för säkerhet - databasen exponeras inte till frontend-lagret.

## Volumes

Volymer används för att **spara data mellan container-restarts**. Utan volymer försvinner all data när containern stoppas.

### Named volumes - Persistent data

Named volumes hanteras av Docker och är perfekta för data som måste överleva:

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    volumes:
      - postgres-data:/var/lib/postgresql/data # Använd volym

volumes:
  postgres-data: # Definiera volym här (tom definition = defaults)
```

**Var lagras datan?**
Någonstans i Dockers interna filsystem (oftast `/var/lib/docker/volumes`). Du behöver inte bry dig om det - Docker sköter allt.

**Livscykel:**
Volymen överlever `docker-compose down`. För att ta bort den:

```bash
docker-compose down -v        # -v = ta bort volymer också
```

### Bind mounts - Development

Bind mounts mappar en mapp från din host-maskin till containern. Perfekt för utveckling:

```yaml
services:
  app:
    build: .
    volumes:
      - ./src:/app/src # Host ./src -> Container /app/src
      - ./application.yml:/app/config/application.yml
```

Nu kan du ändra koden i `./src` på din maskin, och ändringarna syns direkt i containern. Med Spring Boot DevTools får du hot-reload!

### När använda vad?

**Named volumes:**

- Databas-data (PostgreSQL, MySQL, MongoDB)
- Data som måste överleva container-restarts
- Du bryr dig inte om exakt var på disk datan ligger

**Bind mounts:**

- Utveckling (kod, config-filer)
- När du vill editera filer på host och se ändringar direkt
- Logs som du vill kunna läsa från host

## Environment variables och .env filer

Hårdkoda aldrig hemligheter i `docker-compose.yml`! Använd istället `.env`-filer.

### Skapa .env fil

Skapa en fil som heter `.env` i samma mapp som `docker-compose.yml`:

```properties
# .env
POSTGRES_USER=myuser
POSTGRES_PASSWORD=supersecret123
POSTGRES_DB=mydb

SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/mydb
SPRING_DATASOURCE_USERNAME=myuser
SPRING_DATASOURCE_PASSWORD=supersecret123

REDIS_HOST=redis
REDIS_PORT=6379
```

### Använd i docker-compose.yml

Referera till variablerna med `${VARIABLE_NAME}`:

```yaml
version: "3.8"

services:
  app:
    environment:
      SPRING_DATASOURCE_URL: ${SPRING_DATASOURCE_URL}
      SPRING_DATASOURCE_USERNAME: ${POSTGRES_USER}
      SPRING_DATASOURCE_PASSWORD: ${POSTGRES_PASSWORD}

  db:
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
```

Eller ladda hela filen:

```yaml
services:
  app:
    env_file:
      - .env
```

### Lägg till .gitignore

**Viktigt:** Checka ALDRIG in `.env` i git!

```bash
# .gitignore
.env
```

### Skapa .env.example

För att hjälpa teamet, skapa en `.env.example` med dummy-värden:

```properties
# .env.example
POSTGRES_USER=user
POSTGRES_PASSWORD=changeme
POSTGRES_DB=dbname

SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/dbname
SPRING_DATASOURCE_USERNAME=user
SPRING_DATASOURCE_PASSWORD=changeme
```

Teamet kan kopiera denna och fylla i riktiga värden:

```bash
cp .env.example .env
# Editera .env med riktiga värden
```

## depends_on och startup order

`depends_on` säger åt Docker vilken ordning services ska starta i:

```yaml
services:
  app:
    depends_on:
      - db
      - redis

  db:
    image: postgres:15

  redis:
    image: redis:7
```

Ordningen blir: `db` och `redis` startar först, sedan `app`.

### Problemet med depends_on

`depends_on` väntar bara tills containern har **startat**, inte tills servicen är **redo**.

Exempel: PostgreSQL-containern startar, men PostgreSQL själv behöver några sekunder för att initiera databasen. Om din app försöker connecta för snabbt får du fel.

### Lösningar

**1. Retry-logic i appen (bästa lösningen)**

Spring Boot försöker redan automatiskt reconnecta. Lägg till lite timeout:

```yaml
environment:
  SPRING_DATASOURCE_HIKARI_CONNECTION_TIMEOUT: 30000 # 30 sekunder
```

**2. Wait-script**

Använd ett script som väntar tills databasen svarar:

```yaml
services:
  app:
    depends_on:
      - db
    command: sh -c "./wait-for-it.sh db:5432 -- java -jar app.jar"
```

`wait-for-it.sh` är ett populärt script som pingar en port tills den svarar.

**3. Healthcheck med condition**

Docker Compose 2.1+ stödjer healthchecks:

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser"]
      interval: 5s
      timeout: 3s
      retries: 5
```

Nu väntar `app` tills `db` är healthy enligt healthchecken.

## Scaling services

Vill du köra flera instanser av en service? Använd `--scale`:

```bash
docker-compose up --scale app=3
```

Detta skapar `app_1`, `app_2`, och `app_3` - tre kopior av din app!

### Viktigt att veta

**Du kan inte använda `ports` när du skalar:**

```yaml
services:
  app:
    ports:
      - "8080:8080" # FEL! Alla tre instanser försöker använda port 8080
```

Lösningen: använd en load balancer (Nginx) framför:

```yaml
version: "3.8"

services:
  app:
    build: .
    # Ingen ports: - endast internt nätverk

  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
```

Nginx fördelar requests till alla app-instanser. Perfekt för att testa horisontell skalning!

## Docker Compose kommandon

Här är de viktigaste kommandona du behöver.

### up - Starta services

Startar alla services definierade i `docker-compose.yml`:

```bash
docker-compose up                    # Foreground (ser logs)
docker-compose up -d                 # Detached/background
docker-compose up --build            # Bygg om images först
docker-compose up --scale app=3      # Kör 3 instanser av app
docker-compose up app db             # Starta bara app och db
```

**Tips:** Kör utan `-d` första gången så du ser logs direkt.

### down - Stoppa och ta bort

Stoppar containers och tar bort dem (men inte volymer):

```bash
docker-compose down                  # Stoppa och ta bort containers
docker-compose down -v               # Ta även bort volymer (data försvinner!)
docker-compose down --rmi all        # Ta även bort images
```

**Varning:** `-v` tar bort ALL data i volymer. Använd med försiktighet!

### ps - Lista services

Visa status på alla services:

```bash
docker-compose ps
```

Output:

```
NAME                COMMAND             SERVICE   STATUS      PORTS
spring-app          "java -jar app.jar" app       running     0.0.0.0:8080->8080/tcp
postgres-db         "postgres"          db        running     0.0.0.0:5432->5432/tcp
```

### logs - Visa logs

Se vad som händer i dina containers:

```bash
docker-compose logs                  # Alla services
docker-compose logs app              # Bara app
docker-compose logs -f               # Follow (live updates)
docker-compose logs -f --tail=100    # Sista 100 raderna
docker-compose logs -f app db        # Bara app och db
```

**Tips:** `logs -f` är ditt bästa verktyg för debugging!

### exec - Kör kommando i container

Kör ett kommando i en redan körande container:

```bash
docker-compose exec app sh                           # Öppna shell i app
docker-compose exec db psql -U myuser -d mydb        # Öppna psql
docker-compose exec app env                          # Visa environment variables
docker-compose exec redis redis-cli                  # Öppna redis-cli
```

**Notera:** Containern måste redan köra. Använd `exec`, inte `run`.

### build - Bygg images

Bygg (eller rebuilda) images för services som har `build:` definierat:

```bash
docker-compose build                 # Bygg alla
docker-compose build app             # Bygg bara app
docker-compose build --no-cache      # Bygg från scratch (ignorera cache)
```

### restart - Starta om services

Starta om services utan att ta bort dem:

```bash
docker-compose restart               # Alla services
docker-compose restart app           # Bara app
```

### stop - Stoppa utan att ta bort

Stoppa containers men behåll dem:

```bash
docker-compose stop                  # Stoppa alla
docker-compose stop app              # Stoppa bara app
```

### start - Starta stoppade services

Starta services som stoppats med `stop`:

```bash
docker-compose start                 # Starta alla
docker-compose start app             # Starta bara app
```

**Skillnad mellan start och up:**

- `up` skapar nya containers om de inte finns
- `start` startar bara befintliga stoppade containers

### pull - Uppdatera images

Dra ner senaste versionerna av images:

```bash
docker-compose pull                  # Alla images
docker-compose pull db               # Bara db imagen
```

Användbart när du använder images från Docker Hub och vill ha senaste versionen.

## Komplett exempel - Spring Boot stack

Här är ett production-like exempel med allt du behöver:

```yaml
version: "3.8"

services:
  # Nginx load balancer
  nginx:
    image: nginx:alpine
    container_name: nginx-lb
    ports:
      - "80:80" # Exponera på port 80
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - frontend
    restart: unless-stopped

  # Spring Boot applikation (kör flera instanser med --scale app=3)
  app:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      # Database
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/${POSTGRES_DB}
      SPRING_DATASOURCE_USERNAME: ${POSTGRES_USER}
      SPRING_DATASOURCE_PASSWORD: ${POSTGRES_PASSWORD}
      SPRING_JPA_HIBERNATE_DDL_AUTO: validate

      # Redis
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PORT: 6379

      # Actuator
      MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE: health,info,metrics
    depends_on:
      - db
      - redis
    networks:
      - frontend
      - backend
    restart: unless-stopped
    # Ingen ports: eftersom vi använder Nginx som load balancer

  # PostgreSQL databas
  db:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro # Initial schema
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: redis-cache
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
```

### Tillhörande .env.example

```properties
# .env.example
# Kopiera till .env och fyll i riktiga värden

# Database
POSTGRES_USER=myuser
POSTGRES_PASSWORD=changeme
POSTGRES_DB=mydb

# Application
SPRING_PROFILES_ACTIVE=dev
```

### Tillhörande nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    upstream app_servers {
        # Docker Compose skapar automatiskt round-robin mellan instanser
        server app:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://app_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### Starta allt

```bash
# Skapa .env från example
cp .env.example .env

# Editera .env med riktiga värden
nano .env

# Starta en instans
docker-compose up -d

# Eller starta 3 instanser av appen
docker-compose up -d --scale app=3

# Se logs
docker-compose logs -f

# Testa att det funkar
curl http://localhost/actuator/health
```

## Development workflow

Så här jobbar du typiskt med Docker Compose under utveckling:

### 1. Starta allt

```bash
docker-compose up
```

Kör utan `-d` så du ser logs direkt. Bra för debugging.

### 2. Gör kod-ändringar

Editera din kod som vanligt. Om du använder bind mounts och Spring Boot DevTools får du hot-reload:

```yaml
services:
  app:
    volumes:
      - ./src:/app/src # Bind mount för källkod
```

### 3. Rebuilda vid Dockerfile-ändringar

Om du ändrat Dockerfile eller dependencies:

```bash
docker-compose up --build
```

Eller för bara en service:

```bash
docker-compose up --build app
```

### 4. Visa logs

I annat terminal-fönster:

```bash
docker-compose logs -f app
```

### 5. Accessa databasen

Öppna psql direkt i db-containern:

```bash
docker-compose exec db psql -U myuser -d mydb
```

Kör queries:

```sql
SELECT * FROM users;
```

### 6. Testa Redis

```bash
docker-compose exec redis redis-cli

# I redis-cli
> KEYS *
> GET some_key
```

### 7. Stoppa allt

När du är klar för dagen:

```bash
docker-compose down
```

Data i volymer sparas till nästa gång.

### 8. Rensa helt

Om något är trasigt och du vill börja om från scratch:

```bash
docker-compose down -v          # Ta bort volymer
docker-compose build --no-cache # Rebuilda utan cache
docker-compose up
```

## Hot reload för utveckling

För effektiv utveckling vill du ha **hot reload** - ändringar i koden reflekteras direkt utan att behöva rebuilda containern.

### Spring Boot med DevTools

**1. Lägg till DevTools i pom.xml:**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>
</dependency>
```

**2. Använd bind mounts i docker-compose.yml:**

```yaml
services:
  app:
    build: .
    volumes:
      - ./src:/app/src # Source code
      - ./target:/app/target # Compiled classes
    environment:
      SPRING_DEVTOOLS_RESTART_ENABLED: "true"
```

**3. Kör Maven watch i containern:**

```yaml
services:
  app:
    build: .
    command: mvn spring-boot:run
    volumes:
      - .:/app
      - ~/.m2:/root/.m2 # Cache Maven dependencies
```

Nu kompileras och restartas appen automatiskt när du ändrar kod!

### För Gradle

```yaml
services:
  app:
    build: .
    command: ./gradlew bootRun --continuous
    volumes:
      - .:/app
      - ~/.gradle:/root/.gradle
```

## Multi-file compose

Du kan dela upp konfigurationen i flera filer för olika miljöer.

### Grundläggande struktur

**docker-compose.yml** - Bas-konfiguration:

```yaml
version: "3.8"

services:
  app:
    build: .
    environment:
      SPRING_PROFILES_ACTIVE: ${SPRING_PROFILES_ACTIVE}

  db:
    image: postgres:15
```

**docker-compose.override.yml** - Development (läses automatiskt):

```yaml
version: "3.8"

services:
  app:
    volumes:
      - ./src:/app/src # Bind mounts för dev
    environment:
      SPRING_PROFILES_ACTIVE: dev
      DEBUG: "true"
    ports:
      - "8080:8080" # Exponera för local testing

  db:
    ports:
      - "5432:5432" # Exponera för local access
```

**docker-compose.prod.yml** - Production:

```yaml
version: "3.8"

services:
  app:
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: prod
    # Inga ports - använd reverse proxy

  db:
    restart: always
    # Ingen ports - inte exponerad utåt
```

### Använda filerna

**Development (default):**

```bash
docker-compose up              # Läser docker-compose.yml + docker-compose.override.yml
```

**Production:**

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

**Custom:**

```bash
docker-compose -f docker-compose.yml -f docker-compose.test.yml up
```

## Healthchecks

Healthchecks låter Docker övervaka om en service är healthy:

```yaml
services:
  app:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s # Kolla var 30:e sekund
      timeout: 10s # Max väntetid per check
      retries: 3 # Antal misslyckade försök innan unhealthy
      start_period: 40s # Vänta 40s innan första check

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### Visa health status

```bash
docker-compose ps
```

Output visar health status:

```
NAME         STATUS                    HEALTH
app          Up 2 minutes             healthy
db           Up 2 minutes             healthy
```

### Använd med depends_on

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy # Vänta tills db är healthy

  db:
    healthcheck:
# ... healthcheck config
```

Nu startar inte `app` förrän `db` är redo!

## Best practices

### ✅ Gör detta

**1. Använd .env för secrets**

```yaml
environment:
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD} # Från .env
```

**2. Lägg .env i .gitignore**

```bash
echo ".env" >> .gitignore
```

**3. Använd specifika image tags**

```yaml
image: postgres:15 # BRA
image: postgres:latest # DÅLIGT - kan ändras
```

**4. Använd named volumes för viktig data**

```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data
```

**5. En service per container**

```yaml
# BRA - separata services
services:
  app:
  db:
  redis:

# DÅLIGT - allt i en container
services:
  monolith:
```

**6. Använd depends_on**

```yaml
app:
  depends_on:
    - db
```

**7. Lägg till restart policies för produktion**

```yaml
restart: unless-stopped
```

**8. Använd healthchecks**

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
```

### ❌ Undvik detta

**1. Hårdkoda secrets**

```yaml
environment:
  PASSWORD: secret123 # DÅLIGT!
```

**2. Använd :latest**

```yaml
image: postgres:latest # Kan bryta när imagen uppdateras
```

**3. Kör utan volumes**

```yaml
db:
  image: postgres
  # Ingen volumes: - data försvinner vid restart!
```

**4. Exponera databaser onödigt**

```yaml
db:
  ports:
    - "5432:5432" # Behövs inte om bara app ska använda den
```

**5. Glöm bort .gitignore**

```bash
# Committar .env med lösenord -> DÅLIGT!
```

## Common pitfalls

### Port redan i användning

**Problem:**

```
Error: bind: address already in use
```

**Lösning:**
Ändra host-porten:

```yaml
ports:
  - "8081:8080" # Använd 8081 istället
```

Eller stoppa det som använder porten:

```bash
lsof -i :8080                 # Hitta process
kill -9 <PID>                 # Stoppa den
```

### Service name vs container name

**Problem:** Förväxlar service name och container name.

**Kom ihåg:**

- **Service name** används i nätverk: `jdbc:postgresql://db:5432`
- **Container name** är bara ett ID: `docker exec -it myapp sh`

```yaml
services:
  db: # <- Service name (använd i JDBC URL)
    container_name: my-db # <- Container name (för docker commands)
```

### Volymer kvarstår efter down

**Problem:** Kör `docker-compose down` men gamla data finns kvar.

**Förklaring:** Volymer raderas INTE med `down`. Det är avsiktligt (du vill inte förlora databas-data).

**Lösning:**

```bash
docker-compose down -v        # Ta bort volymer också
```

### depends_on väntar inte på "ready"

**Problem:** App startar innan databas är redo.

**Lösning:** Lägg till retry-logic i app, eller använd healthcheck:

```yaml
app:
  depends_on:
    db:
      condition: service_healthy
```

### YAML indentation errors

**Problem:**

```
ERROR: yaml.scanner.ScannerError: mapping values are not allowed here
```

**Lösning:** Dubbelkolla indentation. Använd spaces, inte tabs. Varje nivå är 2 spaces.

```yaml
# RÄTT
services:
  app:
    image: myapp

# FEL (tab istället för spaces)
services:
  app:
    image: myapp
```

### Glömt rebuilda efter Dockerfile-ändring

**Problem:** Ändrat Dockerfile men ser ingen effekt.

**Lösning:**

```bash
docker-compose up --build     # Rebuilda images
```

Eller:

```bash
docker-compose build --no-cache
docker-compose up
```

### Environment variables laddar inte

**Problem:** .env-filen läses inte.

**Checka:**

1. Ligger .env i samma mapp som docker-compose.yml?
2. Är syntaxen rätt? `KEY=value` utan spaces
3. Har du refererat rätt? `${KEY}`

```bash
# Validera compose-filen
docker-compose config         # Visar merged configuration
```

## Felsökning

### Service startar inte

**1. Kolla logs:**

```bash
docker-compose logs app
```

**2. Kolla status:**

```bash
docker-compose ps
```

**3. Kolla healthcheck:**

```bash
docker inspect <container_name> | grep Health -A 10
```

### Network issues

**Problem:** App kan inte nå databas.

**Checka:**

1. Använder du service-namnet? `db:5432` inte `localhost:5432`
2. Är båda i samma nätverk?
3. Har databasen startat klart?

```bash
# Testa connectivity från app till db
docker-compose exec app ping db
docker-compose exec app nc -zv db 5432
```

### Volume issues

**Problem:** Data sparas inte eller är trasig.

**Lösning:**

```bash
# Ta bort volumes och börja om
docker-compose down -v
docker-compose up
```

**Lista volumes:**

```bash
docker volume ls
docker volume inspect <volume_name>
```

### Port conflicts

**Problem:** Port redan används.

```bash
# Hitta vad som använder porten
lsof -i :8080

# Eller ändra port i docker-compose.yml
ports:
  - "8081:8080"
```

### Build issues

**Problem:** Build failar eller använder gammal cache.

```bash
# Rebuilda utan cache
docker-compose build --no-cache

# Rensa allt Docker cacheminne
docker system prune -a
```

### Validera compose-filen

```bash
# Kolla syntax och visa merged config
docker-compose config

# Kolla vilka variabler som används
docker-compose config | grep -i password
```

### Debug mode

Kör container interaktivt för att debugga:

```bash
# Override command för att starta shell istället
docker-compose run --rm app sh

# Eller exec in i running container
docker-compose exec app sh
```

## Docker Compose vs Kubernetes

Båda är container orchestration-verktyg, men för olika användningsfall:

### Docker Compose

**Styrkor:**

- ✅ Enkelt att lära sig
- ✅ Perfekt för utveckling
- ✅ En fil, ett kommando
- ✅ Körs på en maskin
- ✅ Bra för små deployments

**Svagheter:**

- ❌ Körs bara på en host (single machine)
- ❌ Ingen auto-scaling
- ❌ Begränsad high availability
- ❌ Mindre lämpligt för stora produktionssystem

**Använd för:**

- Lokal utveckling
- Testing
- CI/CD
- Små produktionssystem (1-2 servrar)

### Kubernetes

**Styrkor:**

- ✅ Multi-host clustering
- ✅ Auto-scaling
- ✅ Self-healing
- ✅ Rolling updates
- ✅ Service discovery
- ✅ Load balancing
- ✅ Stora produktionssystem

**Svagheter:**

- ❌ Komplex att lära sig
- ❌ Mycket konfiguration
- ❌ Overkill för små projekt
- ❌ Kräver mer resurser

**Använd för:**

- Stora produktionssystem
- Microservices
- Multi-host deployments
- När du behöver auto-scaling
- High availability requirements

### När ska man byta?

**Stanna med Compose om:**

- Teamet är litet (1-5 personer)
- Systemet körs på 1-2 servrar
- Du behöver inte auto-scaling
- Enkelhet är viktigare än features

**Byt till Kubernetes när:**

- Du behöver köra på många servrar
- Auto-scaling är nödvändigt
- High availability är kritisk
- Teamet kan hantera komplexiteten
- Systemet växer till 100+ containers

Det finns ingen skam i att köra Docker Compose i produktion för mindre system. Det är betydligt enklare att underhålla!

## Sammanfattning

**Docker Compose** är verktyget för att hantera multi-container applikationer med enkelhet:

🎯 **Nyckelkoncept:**

- En YAML-fil (`docker-compose.yml`) definierar alla services
- Ett kommando (`docker-compose up`) startar allt
- Service-namn blir hostnames i nätverket
- Volymer för persistent data
- Environment variables från `.env`-filer

📦 **Services:**

- Varje service = en container
- Definiera med `image:` eller `build:`
- Konfigurera ports, volumes, environment
- Använd `depends_on` för startup order

🔗 **Networking:**

- Automatiskt default network
- Services når varandra via service-namn
- Custom networks för isolation

💾 **Volumes:**

- Named volumes för persistent data (databaser)
- Bind mounts för utveckling (hot reload)

⚙️ **Commands du behöver:**

- `up` - Starta allt
- `down` - Stoppa och ta bort
- `logs -f` - Visa logs
- `exec` - Kör kommando i container
- `build` - Rebuilda images

✨ **Best practices:**

- Använd `.env` för secrets
- Specifika image tags (inte :latest)
- Named volumes för databaser
- Restart policies för produktion
- Healthchecks för viktig services

Docker Compose är perfekt för utveckling och mindre produktionssystem. När du växer till hundratals containers över många servrar är det dags att kolla på Kubernetes!

## Övningsuppgifter

**1. Basic setup**

Skapa en `docker-compose.yml` med:

- En Spring Boot app
- En PostgreSQL databas
- Appen ska kunna connecta till databasen

Starta det och verifiera att det funkar.

**2. Lägg till Redis**

Utöka din setup från uppgift 1 med:

- En Redis cache
- Konfigurera Spring Boot att använda Redis
- Verifiera att caching funkar

**3. Environment variables**

Flytta alla secrets till en `.env`-fil:

- Skapa `.env` med databas-credentials
- Skapa `.env.example` för teamet
- Lägg `.env` i `.gitignore`
- Referera till variablerna i `docker-compose.yml`

**4. Load balancing**

Lägg till Nginx som load balancer:

- Nginx exponerar port 80
- Kör 3 instanser av din app med `--scale app=3`
- Nginx fördelar requests mellan instanserna
- Testa att det funkar med `curl`

**5. Förklara**

Varför connectar Spring Boot till `jdbc:postgresql://db:5432/mydb` istället för `jdbc:postgresql://localhost:5432/mydb`?

Skriv ett kort svar som förklarar hur Docker Compose networking fungerar.

**6. Felsökning**

Din kompis har problem med sin docker-compose.yml:

```yaml
services:
  app:
    ports:
      - "8080:8080"
  app:
    ports:
      - "8080:8080"
```

Appen startar med `docker-compose up --scale app=2` men får error. Vad är problemet och hur fixar man det?

**7. Development workflow**

Dokumentera din development workflow:

- Hur startar du miljön?
- Hur ser du logs när du debuggar?
- Hur kommer du åt databasen för att köra queries?
- Hur stoppar du allt?
- Hur rensar du allt och börjar om?

Skriv ner kommandona du använder för varje steg.

---

**Grattis!** Nu kan du Docker Compose! Detta är ett av de mest användbara verktygen i din toolkit som utvecklare. I nästa dokument ska vi titta på microservices-arkitektur och hur man designar distribuerade system.
