# Introduktion till Docker

## Introduktion

Har du någonsin hört frasen **"but it works on my machine!"**? Det är typ den klassiska utvecklar-ursäkten när kod fungerar perfekt på din laptop men kraschar i produktion. Docker är lösningen på det problemet.

Docker är som en flyttkartong för din app - allt den behöver är med. Din kod, alla dependencies, rätt version av Java, libraries, konfiguration... hela rasket. Kartongen kan flyttas vart som helst (din dator, en server, molnet) och appen funkar likadant överallt.

### Det klassiska problemet

Tänk dig det här scenariot:

- Du utvecklar på din Mac med Java 17
- Din kollega utvecklar på Windows med Java 11
- Test-servern kör Linux med Java 16
- Produktionsservern kör något helt annat

Vad händer? Kaos. Saker funkar olika. Dependency-konflikter. Konfigurationsproblem. Huvudvärk.

### Hur Docker löser det

Med Docker blir det såhär istället:

- Du paketerar din app **med** Java 17 i en Docker-container
- Kollegan kör samma container
- Test-servern kör samma container
- Produktionen kör samma container

Samma miljö överallt. Inga överraskningar.

### Lite historia (kort version)

Docker släpptes 2013 av Solomon Hykes och exploderade snabbt i popularitet. Det revolutionerade hur vi utvecklar, testar och deployer applikationer. Innan Docker var det mycket mer komplicerat att få konsistenta miljöer.

Idag är Docker en standard inom mjukvaruutveckling, särskilt när man jobbar med microservices och molnbaserade system.

## Varför Docker?

Låt oss prata om varför Docker är så awesome.

### 1. Konsistenta miljöer

Samma miljö i utveckling, test och produktion. Inga fler "works on my machine"-problem. Om det funkar i containern på din laptop funkar det i produktion.

### 2. Enkel deployment

Istället för att skicka en massa installations-instruktioner ("först installera Java, sen PostgreSQL version 15.2, sen..."), skickar du en färdig container. En kommando startar allt.

```bash
$ docker run myapp
```

Klart.

### 3. Isolering

Varje container är isolerad från andra. Du kan köra flera appar med helt olika dependencies på samma server utan att de krockar. En app kan använda Python 2, en annan Python 3, en tredje Java 17 - inga problem.

### 4. Effektiv resursanvändning

Docker-containers delar på operativsystemets kärna. Det gör dem mycket lättare och snabbare än traditionella virtuella maskiner. Du kan köra dussintals containers på en server som bara skulle klara några få VMs.

### 5. Fungerar överallt

Docker funkar på Windows, Mac, Linux. På din laptop, på en server, i molnet (AWS, Azure, Google Cloud). Bygg en gång, kör överallt.

### 6. Perfekt för microservices

När du delar upp din app i microservices blir det lätt många små tjänster att hålla reda på. Docker gör det enkelt att hantera, starta och skala dem individuellt.

## Containers vs Virtual Machines

Det är lätt att tro att containers och virtuella maskiner är samma sak. De är det inte!

### Virtual Machines (VMs)

En VM är som en komplett dator inuti din dator. Den har sitt eget operativsystem, sin egen kärna, allt.

```
+---------------------------------------+
|          Fysisk Server                |
|                                       |
|  +------------+  +------------+       |
|  |    VM 1    |  |    VM 2    |       |
|  |            |  |            |       |
|  | Guest OS   |  | Guest OS   |       |
|  | (Linux)    |  | (Windows)  |       |
|  |            |  |            |       |
|  | App A      |  | App B      |       |
|  +------------+  +------------+       |
|                                       |
|         Hypervisor (VMware, etc)      |
|                                       |
|         Host Operating System         |
|                                       |
|            Hardware                   |
+---------------------------------------+
```

Varje VM har sitt eget fullständiga OS. Det tar upp mycket plats och resurser.

### Containers (Docker)

En container delar på värdsystemets OS-kärna. Den innehåller bara det som din app behöver.

```
+---------------------------------------+
|          Fysisk Server                |
|                                       |
| +----------+ +----------+ +----------+|
| |Container | |Container | |Container ||
| |    1     | |    2     | |    3     ||
| |          | |          | |          ||
| |  App A   | |  App B   | |  App C   ||
| |  Libs    | |  Libs    | |  Libs    ||
| +----------+ +----------+ +----------+|
|                                       |
|         Docker Engine                 |
|                                       |
|      Host Operating System            |
|                                       |
|            Hardware                   |
+---------------------------------------+
```

Alla containers delar på samma OS-kärna men är isolerade från varandra.

### Jämförelse

| Aspekt               | Virtual Machines     | Containers                 |
| -------------------- | -------------------- | -------------------------- |
| **Storlek**          | Flera GB (helt OS)   | Några MB (bara app + libs) |
| **Starttid**         | Minuter              | Sekunder                   |
| **Resursanvändning** | Hög (eget OS per VM) | Låg (delar OS-kärna)       |
| **Isolering**        | Mycket stark         | Stark men delar kärna      |
| **Performance**      | Lite overhead        | Nästan native              |
| **Portabilitet**     | Mindre (OS-beroende) | Hög (fungerar överallt)    |

### När ska man använda vad?

**Använd VMs när:**

- Du behöver köra olika operativsystem (Linux och Windows)
- Du behöver maximal säkerhetsisolering
- Du vill simulera en hel server med allt

**Använd Containers när:**

- Du vill köra moderna applikationer och microservices
- Du vill ha snabb startup och effektiv resursanvändning
- Du vill ha enkelt dev-test-prod workflow
- Du jobbar med microservices arkitektur

I praktiken: använd containers för applikationer, VMs för när du verkligen behöver separata operativsystem.

## Grundläggande koncept

Nu går vi igenom de viktigaste koncepten i Docker.

### Images (Mallar)

En **Docker image** är en mall - en ritning - för din container. Den är **read-only** och innehåller allt din app behöver:

- Kod
- Runtime (t.ex. Java, Python, Node.js)
- System libraries
- Dependencies
- Konfigurationsfiler
- Environment variables

**Analogier:**

- Image är som ett recept - instruktioner för hur man gör
- Image är som en klass i programmering
- Image är som en installeringsmall

Images byggs från en **Dockerfile** (vi går igenom det i nästa dokument).

#### Image layers

Images är uppbyggda i **lager** (layers). Varje instruktion i en Dockerfile skapar ett nytt lager.

```
+------------------------+
| Din app-kod           |  <- Layer 4
+------------------------+
| Maven dependencies    |  <- Layer 3
+------------------------+
| Java JDK 17          |  <- Layer 2
+------------------------+
| Base OS (Ubuntu)     |  <- Layer 1
+------------------------+
```

Varför är det bra? **Återanvändning!** Om flera images använder samma base layer (t.ex. Ubuntu) behöver den bara lagras en gång.

### Containers (Körande instanser)

En **Docker container** är en körande instans av en image. Det är när appen faktiskt exekveras.

**Analogier:**

- Image är klass, container är objekt/instans
- Image är program (.exe fil), container är den körande processen
- Image är receptet, container är kakan du bakade

Från samma image kan du starta många containers:

```bash
$ docker run -d --name app1 myapp
$ docker run -d --name app2 myapp
$ docker run -d --name app3 myapp
```

Nu har du 3 körande containers från samma image!

#### Containers är efemära

Containers är designade för att vara **tillfälliga** (ephemeral). När du tar bort en container försvinner all data inuti den. Det är därför vi använder **volumes** för att spara viktig data (mer om det senare).

Tänk på det som att containers är stateless - perfekt för vad vi pratade om i skalbarhetsdokumentet!

## Docker arkitektur

Docker består av flera delar som jobbar tillsammans.

```
+------------------+
|  Docker Client   |  <- Du (skriver "docker run" etc)
+------------------+
         |
         | REST API
         v
+------------------+
|  Docker Daemon   |  <- Bakgrundstjänst som gör jobbet
|   (dockerd)      |  
+------------------+
         |
         | Hämtar images från
         v
+------------------+
| Docker Registry  |  <- Lagrar images (t.ex. Docker Hub)
|  (Docker Hub)    |
+------------------+
```

### Docker Daemon

Docker Daemon är en bakgrundstjänst som körs på din dator/server. Den gör det tunga jobbet:

- Bygger images
- Kör containers
- Hanterar nätverk
- Hanterar volumes

### Docker Client

Docker Client är kommandoradsverktyget `docker` som du använder. När du skriver `docker run`, skickar clienten en request till daemonen via REST API.

Du pratar aldrig direkt med daemonen - du använder alltid clienten.

### Docker Registry

En Docker Registry är där images lagras. Den mest kända är **Docker Hub** (som GitHub fast för Docker images).

När du kör `docker pull postgres:15` hämtar daemonen imagen från Docker Hub.

### Hur det funkar tillsammans

1. Du kör: `docker run nginx`
2. Docker Client skickar kommandot till Docker Daemon
3. Daemon kollar om imagen `nginx` finns lokalt
4. Om inte, hämtar den från Docker Hub
5. Daemon skapar och startar containern
6. Din nginx-server är igång!

## Docker Hub

**Docker Hub** är det största publika registret för Docker images. Det är som npm för Node.js eller Maven Central för Java.

### Vad finns på Docker Hub?

- **Officiella images**: Verifierade images från Docker och officiella projekt
  - `openjdk` - Java runtime
  - `postgres` - PostgreSQL databas
  - `redis` - Redis cache
  - `nginx` - Nginx webserver
  - `mysql` - MySQL databas
  - `node` - Node.js runtime

- **Community images**: Images från community och företag
- **Dina egna images**: Du kan pusha dina egna

### Hitta images

Sök på [hub.docker.com](https://hub.docker.com) eller använd:

```bash
$ docker search postgres
```

### Image tags och versioner

Images har **tags** för att specificera olika versioner:

```
postgres:15       <- Version 15
postgres:14       <- Version 14
postgres:latest   <- Senaste versionen (var försiktig!)
openjdk:17-slim   <- Java 17, slim variant (mindre)
openjdk:17-alpine <- Java 17, baserad på Alpine Linux (minimal)
```

**Best practice:** Använd alltid specifika versioner i produktion, inte `:latest`. Med `:latest` vet du inte exakt vad du får.

```bash
# Bra - specifik version
$ docker run postgres:15

# Mindre bra - okänd version
$ docker run postgres:latest
```

## Installation

Jag går inte igenom hela installationsprocessen här (den ändras och varierar mellan system), men här är grunderna:

### Windows och Mac

Installera **Docker Desktop**:

- Gå till [docker.com/get-started](https://www.docker.com/get-started)
- Ladda ner Docker Desktop för ditt OS
- Installera och starta

Docker Desktop ger dig ett grafiskt gränssnitt och allt du behöver.

### Linux

Installera **Docker Engine**:

- Följ instruktionerna för din Linux-distribution på [docs.docker.com](https://docs.docker.com/engine/install/)
- Vanligtvis via pakethanteraren (apt, yum, etc.)

### Verifiera installation

När du installerat Docker, testa att det funkar:

```bash
$ docker --version
Docker version 24.0.6, build ed223bc

$ docker run hello-world
```

Om du ser ett välkomstmeddelande fungerar Docker!

## Grundläggande Docker-kommandon

Nu till det roliga - att faktiskt använda Docker! Här är de viktigaste kommandona.

### docker pull - Ladda ner en image

Hämtar en image från Docker Hub (eller annat registry).

**Syntax:**

```bash
$ docker pull IMAGE_NAME:TAG
```

**Exempel:**

```bash
$ docker pull postgres:15
15: Pulling from library/postgres
a2abf6c4d29d: Pull complete
e1769f49f910: Pull complete
...
Status: Downloaded newer image for postgres:15
```

Nu har du postgres-imagen lokalt!

### docker images - Lista lokala images

Visar alla images du har på din dator.

**Syntax:**

```bash
$ docker images
```

**Exempel output:**

```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
postgres      15        3e9f3c69c71a   2 weeks ago    379MB
openjdk       17-slim   5e28ba2b4cdb   3 weeks ago    411MB
redis         7         e40e2763392d   1 month ago    117MB
nginx         latest    d453dd892d93   2 months ago   187MB
```

### docker run - Skapa och starta container

Det här är det mest använda kommandot. Det skapar en ny container från en image och startar den.

**Grundläggande syntax:**

```bash
$ docker run IMAGE_NAME
```

**Viktiga flaggor:**

| Flagga   | Vad den gör                                   |
| -------- | --------------------------------------------- |
| `-d`     | Detached mode - kör i bakgrunden              |
| `--name` | Ge containern ett namn                        |
| `-p`     | Port mapping (HOST:CONTAINER)                 |
| `-e`     | Sätt environment variable                     |
| `--rm`   | Ta bort container automatiskt när den stoppas |
| `-it`    | Interaktiv terminal                           |
| `-v`     | Mount volume för data                         |

**Exempel 1: Enkel nginx-server**

```bash
$ docker run -d -p 8080:80 nginx
```

- `-d`: Kör i bakgrunden
- `-p 8080:80`: Mappa port 8080 på din dator till port 80 i containern
- Nu kan du öppna http://localhost:8080 och se nginx!

**Exempel 2: PostgreSQL med konfiguration**

```bash
$ docker run -d \
  --name mydb \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=myapp \
  postgres:15
```

- `--name mydb`: Containern heter "mydb"
- `-p 5432:5432`: PostgreSQL port
- `-e`: Sätter environment variables för konfiguration

**Exempel 3: Temporär container**

```bash
$ docker run --rm -it ubuntu bash
```

- `--rm`: Ta bort containern när du är klar
- `-it`: Interaktiv terminal
- `bash`: Kör bash-kommandot
- Nu är du inne i en Ubuntu-container!

### docker ps - Lista containers

Visar körande containers.

**Syntax:**

```bash
$ docker ps      # Bara körande
$ docker ps -a   # Alla (även stoppade)
```

**Exempel output:**

```
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
9a8c7f3b21e4   postgres:15   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:5432->5432/tcp   mydb
f3d2a1b09c5e   nginx         "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp     web
```

### docker stop - Stoppa container

Stoppar en körande container (graciöst).

**Syntax:**

```bash
$ docker stop CONTAINER_NAME_OR_ID
```

**Exempel:**

```bash
$ docker stop mydb
```

Containern stoppas men finns kvar (bara inte körs längre).

### docker start - Starta stoppad container

Startar en container som stoppats.

**Syntax:**

```bash
$ docker start CONTAINER_NAME_OR_ID
```

**Exempel:**

```bash
$ docker start mydb
```

### docker restart - Starta om container

Stoppar och startar om en container.

**Syntax:**

```bash
$ docker restart CONTAINER_NAME_OR_ID
```

**Exempel:**

```bash
$ docker restart mydb
```

Användbart när du ändrat konfiguration eller när container beter sig konstigt.

### docker rm - Ta bort container

Tar bort en container permanent.

**Obs:** Containern måste vara stoppad först (eller använd `-f` för force).

**Syntax:**

```bash
$ docker rm CONTAINER_NAME_OR_ID
$ docker rm -f CONTAINER_NAME_OR_ID  # Force (även körande)
```

**Exempel:**

```bash
$ docker stop mydb
$ docker rm mydb
```

Eller i ett svep:

```bash
$ docker rm -f mydb
```

### docker rmi - Ta bort image

Tar bort en image från din dator.

**Obs:** Inga containers får använda imagen.

**Syntax:**

```bash
$ docker rmi IMAGE_NAME:TAG
```

**Exempel:**

```bash
$ docker rmi postgres:15
```

### docker logs - Visa container-loggar

Visar vad containern skriver ut (stdout/stderr).

**Syntax:**

```bash
$ docker logs CONTAINER_NAME_OR_ID
$ docker logs -f CONTAINER_NAME_OR_ID  # Follow (realtid)
```

**Exempel:**

```bash
$ docker logs mydb
$ docker logs -f mydb  # Följ loggen i realtid (ctrl+c för att avsluta)
```

Super användbart för debugging!

### docker exec - Kör kommando i container

Kör ett kommando inuti en körande container.

**Syntax:**

```bash
$ docker exec CONTAINER_NAME_OR_ID COMMAND
$ docker exec -it CONTAINER_NAME_OR_ID bash  # Interaktiv shell
```

**Exempel 1: Öppna bash-shell i container**

```bash
$ docker exec -it mydb bash
root@9a8c7f3b21e4:/#
```

Nu är du inne i containern och kan köra kommandon!

**Exempel 2: Kör PostgreSQL-kommando**

```bash
$ docker exec mydb psql -U myuser -d myapp -c "SELECT version();"
```

**Exempel 3: Se filer i container**

```bash
$ docker exec mydb ls -la /var/lib/postgresql/data
```

### docker inspect - Detaljerad information

Ger dig all information om en container eller image i JSON-format.

**Syntax:**

```bash
$ docker inspect CONTAINER_OR_IMAGE
```

**Exempel:**

```bash
$ docker inspect mydb
```

Du får se allt: nätverk, volumes, miljövariabler, konfiguration osv.

## Portar och port mapping

En av de viktigaste sakerna att förstå är hur portar fungerar med Docker.

### Varför port mapping?

Containers är isolerade. En app inuti en container lyssnar kanske på port 8080, men din dator vet inget om det. Du måste **mappa** en port på din dator till en port i containern.

### Hur det funkar

```
Din dator (host)           Container
     :3000        <-->      :8080
```

När något anropar `localhost:3000` på din dator skickar Docker det vidare till port 8080 i containern.

### Syntax

```bash
-p HOST_PORT:CONTAINER_PORT
```

### Exempel

**Exempel 1: Samma port**

```bash
$ docker run -d -p 8080:8080 myapp
```

Host port 8080 → Container port 8080

**Exempel 2: Olika portar**

```bash
$ docker run -d -p 3000:8080 myapp
```

Host port 3000 → Container port 8080

Användbart om port 8080 redan används på din dator!

**Exempel 3: Flera portar**

```bash
$ docker run -d \
  -p 8080:8080 \
  -p 9090:9090 \
  myapp
```

**Exempel 4: Alla gränssnitt**

```bash
$ docker run -d -p 5432:5432 postgres:15
```

Lyssnar på alla nätverksgränssnitt

**Exempel 5: Bara localhost**

```bash
$ docker run -d -p 127.0.0.1:5432:5432 postgres:15
```

Lyssnar bara lokalt (mer säkert)

### Varför det är viktigt för skalning

När du kör flera instanser av samma app måste varje container ha sin egen host-port:

```bash
$ docker run -d -p 8081:8080 --name app1 myapp
$ docker run -d -p 8082:8080 --name app2 myapp
$ docker run -d -p 8083:8080 --name app3 myapp
```

Nu har du 3 instanser! Sedan sätter du en load balancer framför som fördelar trafik mellan port 8081, 8082 och 8083.

## Environment variables

Environment variables är hur du konfigurerar containers utan att ändra själva imagen.

### Varför environment variables?

- Olika konfiguration för dev, test, prod
- Inga secrets hardkodade i kod
- Samma image, olika beteende
- Följer "12-factor app" principer

### Sätta environment variables

Använd `-e` flaggan:

```bash
$ docker run -e VARIABLE_NAME=value myapp
```

### Exempel

**Exempel 1: Spring Boot profil**

```bash
$ docker run -d \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e SERVER_PORT=8080 \
  myapp
```

**Exempel 2: Databas-konfiguration**

```bash
$ docker run -d \
  -p 8080:8080 \
  -e DB_HOST=localhost \
  -e DB_PORT=5432 \
  -e DB_NAME=myapp \
  -e DB_USER=myuser \
  -e DB_PASSWORD=secret \
  myapp
```

**Exempel 3: Flera variables**

```bash
$ docker run -d \
  -e ENV=production \
  -e LOG_LEVEL=info \
  -e MAX_CONNECTIONS=100 \
  myapp
```

### Läsa från fil

Om du har många variables kan du använda en fil:

```bash
$ docker run --env-file ./config.env myapp
```

**config.env:**

```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
LOG_LEVEL=debug
```

## Volumes (kort introduktion)

Som vi sagt tidigare: när du tar bort en container försvinner all data inuti den. Det är inte alltid vad vi vill.

### Problemet

```bash
$ docker run -d --name mydb postgres:15
# Databas skapar data...
$ docker rm -f mydb
# All data är borta! 😱
```

### Lösningen: Volumes

**Volumes** är mappningar som låter data överleva utanför containern.

```bash
$ docker run -d \
  --name mydb \
  -v mydata:/var/lib/postgresql/data \
  postgres:15
```

Nu sparas PostgreSQL-datan i en volume som heter `mydata`. Om du tar bort containern finns datan kvar!

### Typer av volumes

**Named volumes (rekommenderat):**

```bash
-v mydata:/path/in/container
```

**Bind mounts (utveckling):**

```bash
-v /path/on/host:/path/in/container
```

Exempel:

```bash
$ docker run -d \
  -v $(pwd)/src:/app/src \
  myapp
```

Nu mappas din lokala `src`-mapp in i containern. Ändringar syns direkt!

**Vi går igenom volumes mer detaljerat i Docker Compose-dokumentet.**

## Praktiskt exempel: PostgreSQL

Låt oss gå igenom ett komplett exempel steg för steg.

### Steg 1: Hämta imagen

```bash
$ docker pull postgres:15
15: Pulling from library/postgres
...
Status: Downloaded newer image for postgres:15
docker.io/library/postgres:15
```

### Steg 2: Kör containern

```bash
$ docker run -d \
  --name mypostgres \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=myappdb \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15
```

**Vad gör varje del?**

- `-d`: Kör i bakgrunden
- `--name mypostgres`: Container heter "mypostgres"
- `-p 5432:5432`: Mappa PostgreSQL-porten
- `-e POSTGRES_PASSWORD=...`: Sätt lösenord (required!)
- `-e POSTGRES_USER=myuser`: Skapa användare "myuser"
- `-e POSTGRES_DB=myappdb`: Skapa databas "myappdb"
- `-v pgdata:/var/lib/postgresql/data`: Spara data i volume
- `postgres:15`: Imagen att använda

### Steg 3: Kolla att det funkar

```bash
$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
a3f9d2c8e7b1   postgres:15   "docker-entrypoint.s…"   30 seconds ago  Up 29 seconds  0.0.0.0:5432->5432/tcp   mypostgres
```

### Steg 4: Visa loggar

```bash
$ docker logs mypostgres
...
2024-10-18 10:15:42.123 UTC [1] LOG:  database system is ready to accept connections
```

Perfekt! Databasen är igång.

### Steg 5: Anslut till databasen

**Option A: Från din dator**

Använd vilket PostgreSQL-verktyg som helst (DBeaver, pgAdmin, psql):

- Host: `localhost`
- Port: `5432`
- Database: `myappdb`
- User: `myuser`
- Password: `mysecretpassword`

**Option B: Inifrån containern**

```bash
$ docker exec -it mypostgres psql -U myuser -d myappdb
psql (15.4)
Type "help" for help.

myappdb=# \l
myappdb=# CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));
myappdb=# \dt
myappdb=# \q
```

### Steg 6: Stoppa och starta

```bash
$ docker stop mypostgres
$ docker start mypostgres
```

Datan finns kvar tack vare volymen!

### Steg 7: Städa upp (senare)

```bash
$ docker stop mypostgres
$ docker rm mypostgres
$ docker volume rm pgdata  # Ta även bort volymen
```

## Praktiskt exempel: Spring Boot app

Låt oss säga att du har en färdig Spring Boot-image (vi lär oss bygga den i nästa dokument).

### Scenario

Du har en image som heter `myspringapp:1.0` som:

- Är en Spring Boot-app
- Lyssnar på port 8080
- Behöver ansluta till PostgreSQL

### Steg 1: Starta databasen

```bash
$ docker run -d \
  --name postgres-db \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=appdb \
  postgres:15
```

### Steg 2: Starta din app

```bash
$ docker run -d \
  --name myapp \
  -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://host.docker.internal:5432/appdb \
  -e SPRING_DATASOURCE_USERNAME=postgres \
  -e SPRING_DATASOURCE_PASSWORD=secret \
  -e SPRING_PROFILES_ACTIVE=prod \
  myspringapp:1.0
```

**Notera `host.docker.internal`** - det är hur en container når din dators localhost.

### Steg 3: Testa att det funkar

```bash
$ curl http://localhost:8080/actuator/health
{"status":"UP"}

$ curl http://localhost:8080/api/users
[]
```

### Steg 4: Kolla loggar

```bash
$ docker logs myapp
...
2024-10-18 10:20:15.234  INFO --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http)
2024-10-18 10:20:15.245  INFO --- [main] com.example.MyApplication                : Started MyApplication in 3.456 seconds
```

### Flera instanser för load balancing

```bash
$ docker run -d --name myapp1 -p 8081:8080 -e ... myspringapp:1.0
$ docker run -d --name myapp2 -p 8082:8080 -e ... myspringapp:1.0
$ docker run -d --name myapp3 -p 8083:8080 -e ... myspringapp:1.0
```

Nu har du 3 instanser på port 8081, 8082 och 8083!

## Container livscykel

En container går igenom olika stadier i sin livscykel.

```
+---------+
| Created |  <- docker create (skapad men ej startad)
+---------+
     |
     | docker start
     v
+---------+
| Running |  <- docker run (körs)
+---------+
     |
     | docker stop / docker kill
     v
+---------+
| Stopped |  <- Stoppad men finns kvar
+---------+
     |
     | docker rm
     v
+---------+
| Removed |  <- Borttagen permanent
+---------+
```

### State-övergångar

**Created → Running:**

```bash
$ docker create --name myapp nginx
$ docker start myapp
```

Eller direkt:

```bash
$ docker run --name myapp nginx
```

**Running → Stopped:**

```bash
$ docker stop myapp   # Graceful (ger tid att stänga ner)
$ docker kill myapp   # Force (dödar direkt)
```

**Stopped → Running:**

```bash
$ docker start myapp
```

**Stopped/Running → Removed:**

```bash
$ docker rm myapp      # Måste vara stoppad
$ docker rm -f myapp   # Force-tar bort även körande
```

### Vad händer med data?

- **Created/Running/Stopped:** Data inuti containern finns kvar
- **Removed:** All data i containern försvinner (utom i volumes!)

Det är därför volumes är så viktiga för persistent data.

## Networking (grundläggande)

Containers behöver ofta prata med varandra. Docker skapar automatiskt nätverk för detta.

### Default networks

Docker skapar tre nätverk automatiskt:

- **bridge** - Standard för containers på samma host
- **host** - Containern använder host-nätverket direkt
- **none** - Inget nätverk

### Container-till-container kommunikation

Containers på samma nätverk kan prata med varandra via container-namn:

```bash
$ docker network create mynetwork
$ docker run -d --name db --network mynetwork postgres:15
$ docker run -d --name app --network mynetwork myapp
```

Nu kan `app` ansluta till databasen via `db:5432` istället för `localhost:5432`.

### Teaser: Docker Compose

Det här blir mycket enklare med Docker Compose! Där skapas nätverk automatiskt och containers kan prata med varandra via sina namn. Vi går igenom det i nästa dokument.

## Best practices

Här är några bra regler att följa när du jobbar med Docker.

### 1. Använd officiella images

När det finns en officiell image, använd den:

```bash
# Bra
$ docker run postgres:15

# Mindre bra
$ docker run random-persons-postgres
```

Officiella images är verifierade, uppdaterade och säkra.

### 2. Specificera alltid version/tag

```bash
# Bra - du vet exakt vad du får
$ docker run postgres:15.2

# Dåligt - vem vet vad du får?
$ docker run postgres:latest
```

I produktion: använd aldrig `:latest`. Det kan ändras under dig.

### 3. Använd meningsfulla namn

```bash
# Bra
$ docker run --name users-api myapp
$ docker run --name postgres-prod postgres:15

# Dåligt (random generated names)
$ docker run myapp  # heter typ "hungry_newton"
```

### 4. Städa upp regelbundet

Containers och images tar plats. Ta bort det du inte använder:

```bash
# Ta bort stoppade containers
$ docker container prune

# Ta bort oanvända images
$ docker image prune

# Ta bort allt oanvänt (VARNING: aggressivt!)
$ docker system prune -a
```

### 5. Kör inte som root

Containers kör default som root, vilket är en säkerhetsrisk. Din Dockerfile bör skapa en user:

```dockerfile
USER appuser  # Inte root!
```

(Mer om detta i Dockerfile-dokumentet)

### 6. Använd .dockerignore

Liksom `.gitignore` men för Docker. Exkludera filer som inte behövs i imagen:

```
node_modules/
.git/
*.log
.env
```

(Mer om detta i nästa dokument)

## Felsökning

När saker går fel, här är hur du debuggar.

### Container startar inte

**Problem:** `docker run` ger error eller container stoppar direkt.

**Lösning:**

```bash
# Kolla vad som gick fel
$ docker logs CONTAINER_NAME

# Kör utan -d för att se output direkt
$ docker run myapp  # Inte -d

# Kolla detaljerad info
$ docker inspect CONTAINER_NAME
```

### Port redan används

**Problem:**

```
Error: bind: address already in use
```

**Lösning:**

```bash
# Använd en annan host-port
$ docker run -p 8081:8080 myapp  # Istället för 8080:8080

# Eller hitta vad som använder porten
$ lsof -i :8080  # Mac/Linux
$ netstat -ano | findstr :8080  # Windows
```

### Kan inte ansluta till container

**Problem:** `curl localhost:8080` funkar inte.

**Lösningar:**

1. Kolla att port mapping är korrekt:

```bash
$ docker ps
# Se till att PORTS kolumnen visar 0.0.0.0:8080->8080/tcp
```

2. Kolla att appen lyssnar på rätt port inuti containern:

```bash
$ docker exec CONTAINER_NAME netstat -tulpn
```

3. Kolla firewall-regler på din dator

### Container startar om hela tiden

**Problem:** Container går ner och startar om igen och igen.

**Lösning:**

```bash
# Kolla loggar för error
$ docker logs CONTAINER_NAME

# Kolla om det är en crash loop
$ docker ps -a  # STATUS: Restarting
```

Ofta är det ett fel i applikationen eller fel konfiguration.

### Se processer i container

```bash
$ docker top CONTAINER_NAME
```

Visar vilka processer som körs inuti containern.

### Få en shell i container

```bash
# Bash (om tillgänglig)
$ docker exec -it CONTAINER_NAME bash

# Sh (finns alltid)
$ docker exec -it CONTAINER_NAME sh

# Alpine Linux använder sh
$ docker exec -it CONTAINER_NAME sh
```

## Skillnad mot utveckling lokalt

Låt oss jämföra traditionell lokal utveckling med Docker.

### Traditionellt (lokalt)

**Setup för nytt projekt:**

1. Installera rätt version av Java/Python/Node.js
2. Installera PostgreSQL lokalt
3. Installera Redis lokalt
4. Konfigurera allt
5. Hantera port-konflikter
6. Hoppas att alla har samma setup

**Problem:**

- "Works on my machine!"
- Olika versioner mellan utvecklare
- Svårt att rensa upp dependencies
- Kräver mycket manuellt jobb

### Med Docker

**Setup för nytt projekt:**

1. `docker-compose up`

Klart! Allt är konfigurerat och kör.

**Fördelar:**

- Identisk miljö för alla
- Inget installerat på din dator
- Lätt att städa upp (ta bort container)
- Samma setup i dev, test och prod
- Tydlig dokumentation (Dockerfile/docker-compose.yml)

### Exempel: Onboarding ny utvecklare

**Utan Docker:**

```
"Okej, först installera Java 17, sen PostgreSQL 15, sen Redis 7, 
sen konfigurera connection strings, sen... 
varför får du fel? Hmm, kanske fel Java-version?"
```

**Tid:** Flera timmar, ofta hela dagen

**Med Docker:**

```
"Kör 'docker-compose up'"
```

**Tid:** 5 minuter

## Sammanfattning

Vi har täckt en massa! Här är nyckelpunkterna:

**Docker** paketerar din app med allt den behöver - kod, runtime, libraries, dependencies.

**Images** är mallar (read-only templates) - som ritningar eller klasser.

**Containers** är körande instanser av images - som objekt från en klass.

**Viktiga fördelar:**

- Konsistenta miljöer överallt
- Enkel deployment
- Isolering mellan appar
- Effektiv resursanvändning
- Perfekt för microservices

**Containers vs VMs:**

- Containers är lättare och snabbare
- Delar OS-kärna men isolerade från varandra
- VMs har eget komplett OS

**Docker arkitektur:**

- Client → Daemon → Registry
- Du pratar med clienten, daemon gör jobbet
- Images lagras i registry (Docker Hub)

**Grundläggande kommandon:**

- `docker pull` - Ladda ner image
- `docker run` - Skapa och starta container
- `docker ps` - Lista containers
- `docker stop/start/restart` - Hantera containers
- `docker logs` - Se output från container
- `docker exec` - Kör kommando i container
- `docker rm/rmi` - Ta bort container/image

**Port mapping:**

- `-p HOST:CONTAINER` mappar portar
- Nödvändigt för att nå container från din dator
- Flera containers behöver olika host-portar

**Environment variables:**

- Konfigurera containers med `-e`
- Samma image, olika beteende
- Inga hardkodade secrets

**Volumes:**

- Spara data som överlever container
- `-v name:/path` för persistent data

**Best practices:**

- Använd officiella images
- Specificera versioner (inte :latest)
- Meningsfulla container-namn
- Städa upp regelbundet

I nästa dokument lär vi oss bygga egna images med **Dockerfile**!

## Övningsuppgifter

### Uppgift 1: Nginx

Starta en nginx-container som kör på port 8080 på din dator.

**Krav:**

- Containern ska ha namnet "myweb"
- Ska köra i bakgrunden
- Port 80 i containern ska mappas till 8080 på din dator
- Verifiera att det funkar genom att öppna http://localhost:8080

<details>
<summary>Lösning</summary>

```bash
$ docker run -d --name myweb -p 8080:80 nginx
$ curl http://localhost:8080
```

</details>

### Uppgift 2: Redis

1. Hämta Redis image (version 7)
2. Starta en Redis-container i detached mode
3. Använd `docker ps` för att se att den körs
4. Visa loggarna från Redis
5. Öppna redis-cli inuti containern och kör `PING` (ska svara `PONG`)
6. Stoppa och ta bort containern

<details>
<summary>Lösning</summary>

```bash
$ docker pull redis:7
$ docker run -d --name myredis -p 6379:6379 redis:7
$ docker ps
$ docker logs myredis
$ docker exec -it myredis redis-cli
127.0.0.1:6379> PING
PONG
127.0.0.1:6379> exit
$ docker stop myredis
$ docker rm myredis
```

</details>

### Uppgift 3: PostgreSQL med volume

Starta en PostgreSQL-container med en volume så att data inte försvinner.

**Krav:**

- Använd postgres:15
- Port 5432
- Lösenord: "testpass"
- Databas: "testdb"
- Volume: "pgvol" som mappas till `/var/lib/postgresql/data`
- Anslut till databasen och skapa en tabell
- Ta bort containern men INTE volymen
- Starta en ny container med samma volume
- Verifiera att tabellen finns kvar

<details>
<summary>Lösning</summary>

```bash
# Starta container
$ docker run -d \
  --name testpg \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=testpass \
  -e POSTGRES_DB=testdb \
  -v pgvol:/var/lib/postgresql/data \
  postgres:15

# Skapa tabell
$ docker exec -it testpg psql -U postgres -d testdb
testdb=# CREATE TABLE test (id INT, name VARCHAR(50));
testdb=# INSERT INTO test VALUES (1, 'Hello');
testdb=# \q

# Ta bort container (inte volume)
$ docker stop testpg
$ docker rm testpg

# Starta ny container med samma volume
$ docker run -d \
  --name testpg2 \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=testpass \
  -e POSTGRES_DB=testdb \
  -v pgvol:/var/lib/postgresql/data \
  postgres:15

# Kolla att data finns kvar
$ docker exec -it testpg2 psql -U postgres -d testdb
testdb=# SELECT * FROM test;
 id | name  
----+-------
  1 | Hello
```

</details>

### Uppgift 4: Förklara port mapping

**Fråga:** Du har en Spring Boot-app som lyssnar på port 8080 i containern. Du vill köra 3 instanser av appen på samma dator. Förklara:

1. Varför behöver vi port mapping överhuvudtaget?
2. Hur skulle du mappa portarna för de 3 instanserna?
3. Vad händer om du försöker köra två containers med `-p 8080:8080`?

<details>
<summary>Svar</summary>

1. **Varför port mapping?**
   - Containern är isolerad från host-systemet
   - Port 8080 i containern är inte samma som port 8080 på din dator
   - Port mapping skapar en "bro" mellan host-port och container-port
   - Utan det kan vi inte nå appen från utsidan

2. **Tre instanser:**

```bash
$ docker run -d --name app1 -p 8081:8080 myapp
$ docker run -d --name app2 -p 8082:8080 myapp
$ docker run -d --name app3 -p 8083:8080 myapp
```

- Varje container lyssnar internt på 8080
- Men de mappas till olika portar på host (8081, 8082, 8083)

3. **Två containers med samma port:**
   - Du får error: "port is already allocated"
   - Två processer kan inte lyssna på samma port
   - Du måste använda olika host-portar

</details>

### Uppgift 5: Rita Docker-arkitektur

Rita ett diagram som visar relationerna mellan:

- Docker Client
- Docker Daemon
- Docker Registry (Docker Hub)
- Image
- Container

Visa med pilar hur de interagerar när du kör `docker run nginx`.

<details>
<summary>Exempel-svar</summary>

```
När du kör: docker run nginx

1. [Du] 
     |
     | skriver "docker run nginx"
     v
2. [Docker Client (CLI)]
     |
     | Skickar API-request
     v
3. [Docker Daemon]
     |
     | Kollar: finns nginx-imagen lokalt? Nej!
     |
     | Hämtar image
     v
4. [Docker Hub (Registry)]
     |
     | Skickar nginx-image
     v
5. [Docker Daemon]
     |
     | Skapar container från image
     v
6. [Container] - nginx körs!

Permanent struktur:
+-------------------+
| Docker Client     | <- Du interagerar här
+-------------------+
         |
         | REST API
         v
+-------------------+
| Docker Daemon     | <- Gör allt jobb
+-------------------+
    |            |
    | hanterar   | hämtar från
    v            v
+--------+   +-------------+
|Container|  | Docker Hub  |
| (nginx) |  | (Registry)  |
+--------+   +-------------+
    ^
    | skapas från
    |
+--------+
| Image  |
| (nginx)|
+--------+
```

</details>

---

**Bra jobbat!** Nu kan du grunderna i Docker. I nästa dokument bygger vi våra egna images med Dockerfile!
