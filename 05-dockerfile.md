# Dockerfile - Recept för Docker Images

## Introduktion

En Dockerfile är som ett recept - steg för steg beskriver du vad som ska finnas i din image. Det är en textfil som innehåller alla instruktioner Docker behöver för att bygga en image av din applikation.

Tänk dig att du bakar en tårta. Du behöver ett recept som säger:

1. Börja med mjöl och ägg
2. Blanda ihop
3. Tillsätt socker
4. Grädda i ugnen

En Dockerfile fungerar på samma sätt:

1. Börja med en bas-image (typ Ubuntu eller Java)
2. Kopiera in din kod
3. Installera dependencies
4. Berätta hur appen ska startas

### Varför behöver vi en Dockerfile?

Du har kanske hört att Docker låter dig köra applikationer i containers. Men hur får du din app IN i en container från början? Det är precis där Dockerfile kommer in!

Med en Dockerfile kan du:

- **Skapa custom images** som innehåller exakt din app och dess dependencies
- **Automatisera uppsättningen** så alla utvecklare och servrar kör exakt samma miljö
- **Versionshantera din infrastruktur** - Dockerfile ligger i Git tillsammans med koden
- **Göra det reproducerbart** - samma Dockerfile ger samma image varje gång

### Det är bara en textfil

Det fina med Dockerfile är att det är en helt vanlig textfil. Inget speciellt format, inga konstiga verktyg behövs. Du kan skriva den i vilken texteditor som helst.

## Hur skapar man en Dockerfile?

### Skapa filen

Det är super enkelt. Skapa en fil som heter exakt **`Dockerfile`** - inget filnamn, ingen extension, bara "Dockerfile" med stort D.

```bash
touch Dockerfile
```

Eller bara skapa den i din editor.

### Var ska den ligga?

Normalt lägger du Dockerfile i projektets root-mapp:

```
mitt-projekt/
├── src/
├── pom.xml
├── Dockerfile     ← Här!
└── README.md
```

Den kan också ligga i en undermapp om du har speciella behov, men root är standard.

### Vad ska den innehålla?

Varje rad i Dockerfile är en **instruktion**. Instruktioner skrivs med VERSALER (typ `FROM`, `COPY`, `RUN`) följt av argument.

```dockerfile
INSTRUKTION argument
```

Det är så enkelt som det låter!

## Grundläggande struktur

Låt oss börja med ett super simpelt exempel för en Spring Boot-app:

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/myapp.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

Låt oss gå igenom rad för rad:

1. **`FROM openjdk:17-jdk-slim`** - "Börja med en image som redan har Java 17 installerat". Det är som att börja med en pizzadeg innan du lägger på pålägg.

2. **`WORKDIR /app`** - "Skapa och gå in i mappen /app". Alla kommandon efter detta körs i den mappen.

3. **`COPY target/myapp.jar app.jar`** - "Kopiera JAR-filen från min dator in i imagen".

4. **`EXPOSE 8080`** - "Appen kommer lyssna på port 8080". Det här är bara dokumentation, öppnar inte porten automatiskt.

5. **`CMD ["java", "-jar", "app.jar"]`** - "När containern startar, kör det här kommandot".

Och det är faktiskt allt du **måste** ha för en enkel Spring Boot-app! Men det finns mycket mer vi kan göra.

## Dockerfile instruktioner (detaljerat)

Nu går vi igenom alla viktiga instruktioner mer i detalj.

### FROM - Bas-image

**`FROM`** är alltid första instruktionen (förutom eventuella `ARG` före den). Den säger vilken image du vill bygga vidare från.

**Syntax:**

```dockerfile
FROM image:tag
```

**Vanliga bas-images:**

```dockerfile
# Java applikationer
FROM openjdk:17-jdk-slim
FROM eclipse-temurin:17-jre-alpine

# Node.js applikationer  
FROM node:18
FROM node:18-alpine

# Python applikationer
FROM python:3.11
FROM python:3.11-slim

# Nginx för static files
FROM nginx:alpine

# Om du vill starta från scratch (avancerat)
FROM scratch
```

**Varför olika varianter?**

- **`slim`** - mindre image, färre verktyg installerade
- **`alpine`** - jätteliten Linux-distribution, minimala images
- **`jdk`** vs **`jre`** - JDK för att bygga Java-kod, JRE för att bara köra den

Tommelregel: välj den minsta image som har det du behöver. Mindre image = snabbare nedladdning och mindre säkerhetsrisker.

**Scratch base:**

```dockerfile
FROM scratch
```

Det här är en helt tom image - bokstavligen ingenting. Används för statiskt kompilerade binärer (Go, Rust) där du inte behöver ett OS. Det är advanced stuff!

### WORKDIR - Sätt arbetsmapp

**`WORKDIR`** sätter vilket directory alla följande kommandon ska köras i. Ungefär som `cd` i Linux.

**Syntax:**

```dockerfile
WORKDIR /path/to/directory
```

**Exempel:**

```dockerfile
WORKDIR /app
```

Om mappen inte finns skapar Docker den automatiskt.

**Varför använda WORKDIR?**

```dockerfile
# Dåligt - svårt att följa
RUN cd /app && npm install
RUN cd /app && npm build

# Bra - tydligt och enkelt
WORKDIR /app
RUN npm install
RUN npm build
```

**Best practice:** Använd absoluta paths (börjar med `/`), inte relativa.

```dockerfile
WORKDIR /app              # ✓ Bra
WORKDIR app               # ✗ Kan bli konstigt
```

### COPY - Kopiera filer

**`COPY`** kopierar filer och mappar från din dator (där Dockerfile ligger) in i imagen.

**Syntax:**

```dockerfile
COPY source destination
```

**Exempel:**

```dockerfile
# Kopiera en JAR-fil
COPY target/app.jar app.jar

# Kopiera en hel mapp
COPY src/ /app/src/

# Kopiera allt i current directory (var försiktig!)
COPY . .

# Kopiera med wildcard
COPY *.jar app.jar

# Kopiera flera filer
COPY package.json package-lock.json ./
```

**Viktigt att veta:**

- Source är **relativt till där Dockerfile ligger** (byggkontexten)
- Destination är **relativt till WORKDIR** (om inte absolut path)
- Bevarar filrättigheter och timestamps

**Exempel med WORKDIR:**

```dockerfile
WORKDIR /app
COPY target/app.jar .     # Kopieras till /app/app.jar
```

### ADD - Som COPY fast med extras

**`ADD`** är som `COPY` men med extra features.

**Syntax:**

```dockerfile
ADD source destination
```

**Vad ADD kan som COPY inte kan:**

1. **Ladda ner från URL:**

```dockerfile
ADD https://example.com/file.tar.gz /tmp/
```

2. **Auto-extrahera tar-filer:**

```dockerfile
ADD archive.tar.gz /app/    # Packas upp automatiskt!
```

**När ska man använda ADD vs COPY?**

Generellt: **använd COPY**. Det är mer explicit och förutsägbart.

Använd endast ADD när du specifikt behöver:

- Ladda ner från URL
- Auto-extrahera en tar-fil

```dockerfile
COPY app.jar app.jar      # ✓ Preferred
ADD app.jar app.jar       # ✗ Onödigt
ADD archive.tar.gz /app/  # ✓ OK, du vill extrahera
```

### RUN - Kör kommandon

**`RUN`** kör kommandon **när du bygger imagen**. Inte när containern startar - utan under själva build-processen.

**Syntax:**

```dockerfile
RUN command
```

**Exempel:**

```dockerfile
# Installera paket i Ubuntu/Debian
RUN apt-get update && apt-get install -y curl

# Installera Node packages
RUN npm install

# Bygga Java-projekt
RUN mvn clean package

# Installera Python dependencies
RUN pip install -r requirements.txt

# Flera kommandon på samma rad
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean
```

**Viktigt:** Varje `RUN` skapar ett nytt **layer** i imagen. Fler layers = större image.

**Bättre:**

```dockerfile
# Ett layer
RUN apt-get update && \
    apt-get install -y curl vim git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Sämre:**

```dockerfile
# Tre layers
RUN apt-get update
RUN apt-get install -y curl vim git
RUN apt-get clean
```

**Line continuation:** Använd `\` för att dela upp långa kommandon över flera rader (för läsbarhet).

### CMD - Default kommando vid start

**`CMD`** specificerar vilket kommando som ska köras när containern **startar**.

**Viktigt:** Det körs inte under build, utan när du gör `docker run`.

**Två former:**

**Exec form (preferred):**

```dockerfile
CMD ["executable", "param1", "param2"]
```

**Shell form:**

```dockerfile
CMD command param1 param2
```

**Exempel:**

```dockerfile
# Java application
CMD ["java", "-jar", "app.jar"]

# Node.js application
CMD ["node", "server.js"]

# Python application
CMD ["python", "app.py"]

# Shell form
CMD java -jar app.jar
```

**Skillnaden mellan formerna:**

Exec form kör kommandot **direkt**, shell form kör det genom `/bin/sh -c`.

```dockerfile
# Exec form - processen får PID 1
CMD ["java", "-jar", "app.jar"]

# Shell form - sh får PID 1, java blir child process
CMD java -jar app.jar
```

För containers vill du normalt att din app är PID 1 (för rätt signal handling), så **använd exec form**.

**Endast EN CMD:**

Om du har flera CMD i din Dockerfile, bara den sista gäller.

```dockerfile
CMD ["echo", "hello"]
CMD ["echo", "world"]    # Endast denna körs
```

**CMD kan overridas:**

När du kör containern kan användaren override:a CMD:

```bash
# Kör default CMD
docker run myapp

# Override CMD
docker run myapp echo "något annat"
```

### ENTRYPOINT - Som CMD men sticky

**`ENTRYPOINT`** är som `CMD` men **svårare att override**. Används när containern ska fungera som en körbar fil.

**Syntax:**

```dockerfile
# Exec form (preferred)
ENTRYPOINT ["executable", "param1", "param2"]

# Shell form
ENTRYPOINT command param1 param2
```

**Exempel:**

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Vad är skillnaden mot CMD?**

Med CMD:

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

```bash
docker run myapp                    # Kör java -jar app.jar
docker run myapp echo "hello"       # Kör echo hello (ersätter helt)
```

Med ENTRYPOINT:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
docker run myapp                    # Kör java -jar app.jar
docker run myapp --spring.profile=prod   # Kör java -jar app.jar --spring.profile=prod
```

Argumenten **läggs till** istället för att ersätta.

**Kombinera ENTRYPOINT och CMD:**

Superkraftigt! ENTRYPOINT är kommandot, CMD är default-argument.

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
CMD ["--spring.profiles.active=dev"]
```

```bash
# Använder default CMD
docker run myapp
# Kör: java -jar app.jar --spring.profiles.active=dev

# Override CMD
docker run myapp --spring.profiles.active=prod
# Kör: java -jar app.jar --spring.profiles.active=prod
```

**När använda vad?**

- **CMD:** När containern kan köra olika saker (flexibel)
- **ENTRYPOINT:** När containern alltid ska köra samma program (som en executable)
- **Både:** När du vill ha ett fast kommando med konfigurerbara default-argument

### EXPOSE - Dokumentera portar

**`EXPOSE`** berättar vilka portar din app lyssnar på.

**Viktigt:** Det **öppnar inte faktiskt porten**! Det är bara dokumentation.

**Syntax:**

```dockerfile
EXPOSE port
```

**Exempel:**

```dockerfile
# En port
EXPOSE 8080

# Flera portar
EXPOSE 8080
EXPOSE 9090

# Eller på samma rad
EXPOSE 8080 9090

# Med protokoll (default är TCP)
EXPOSE 8080/tcp
EXPOSE 53/udp
```

**Vad gör den egentligen?**

`EXPOSE` är metadata. När någon kör `docker ps` eller läser Dockerfilen ser de vilka portar som används.

För att **faktiskt** publicera porten måste du använda `-p` när du kör containern:

```bash
docker run -p 8080:8080 myapp
```

**Varför använda EXPOSE då?**

- **Dokumentation** - andra vet vilka portar som används
- **Docker Compose** - kan automatiskt hitta portarna
- **Best practice** - gör det tydligt

### ENV - Environment variables

**`ENV`** sätter environment variables som finns både under build OCH när containern körs.

**Syntax:**

```dockerfile
# Modern syntax
ENV KEY=value

# Gammal syntax (funkar också)
ENV KEY value

# Flera på en gång
ENV KEY1=value1 KEY2=value2
```

**Exempel:**

```dockerfile
# Spring Boot profile
ENV SPRING_PROFILES_ACTIVE=production

# Database connection
ENV DATABASE_URL=jdbc:postgresql://db:5432/mydb
ENV DATABASE_USER=admin

# Node environment
ENV NODE_ENV=production

# Custom app settings
ENV APP_PORT=8080
ENV LOG_LEVEL=INFO
```

**Använda i andra kommandon:**

```dockerfile
ENV APP_HOME=/app
WORKDIR $APP_HOME
COPY . $APP_HOME
```

**Override vid runtime:**

ENV kan overridas när du kör containern:

```bash
docker run -e SPRING_PROFILES_ACTIVE=dev myapp
```

**Best practice:**

För **känslig data** (lösenord, API-nycklar) - använd INTE ENV i Dockerfile! Använd istället:

- `-e` flag vid `docker run`
- Docker secrets
- Environment files

```dockerfile
# ✗ Dåligt - lösenord i imagen
ENV DB_PASSWORD=supersecret123

# ✓ Bra - sätts vid runtime
ENV DB_PASSWORD=
```

### ARG - Build-time variables

**`ARG`** är variabler som bara finns **under build**, inte när containern körs.

**Syntax:**

```dockerfile
ARG VARIABLE_NAME
ARG VARIABLE_NAME=default_value
```

**Exempel:**

```dockerfile
ARG VERSION=1.0
ARG JAR_FILE=target/*.jar

FROM openjdk:17-jre-slim
COPY ${JAR_FILE} app.jar
LABEL version=${VERSION}
```

**Använd vid build:**

```bash
docker build --build-arg VERSION=2.0 --build-arg JAR_FILE=target/myapp.jar -t myapp .
```

**ARG vs ENV:**

| ARG                     | ENV                      |
| ----------------------- | ------------------------ |
| Endast under build      | Build OCH runtime        |
| Sätts med `--build-arg` | Sätts med `-e` eller ENV |
| Försvinner efter build  | Finns kvar i containern  |

**Använda tillsammans:**

```dockerfile
ARG APP_VERSION=1.0
ENV VERSION=$APP_VERSION    # Gör ARG tillgänglig i runtime
```

**ARG före FROM:**

ARG kan användas före FROM för att parametrisera bas-imagen:

```dockerfile
ARG JAVA_VERSION=17
FROM openjdk:${JAVA_VERSION}-jre-slim
```

### LABEL - Metadata

**`LABEL`** lägger till metadata till imagen. Key-value pairs.

**Syntax:**

```dockerfile
LABEL key="value"
```

**Exempel:**

```dockerfile
LABEL version="1.0"
LABEL description="Min awesome Spring Boot app"
LABEL maintainer="dittnamn@example.com"

# Flera på samma rad
LABEL version="1.0" \
      description="My app" \
      maintainer="dittnamn@example.com"
```

**Se labels:**

```bash
docker inspect myapp
```

Användbart för organisering, automation, och dokumentation.

### USER - Byt användare

**`USER`** sätter vilken användare (och grupp) som ska köra kommandona.

**Syntax:**

```dockerfile
USER username
USER uid
USER username:groupname
```

**Varför är detta viktigt?**

Default kör allt som **root** i containern. Det är en säkerhetsrisk!

**Best practice:**

```dockerfile
FROM openjdk:17-jre-slim

# Skapa en användare
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Sätt upp din app
WORKDIR /app
COPY target/app.jar app.jar

# Byt till non-root användare
USER appuser

# Nu körs detta som appuser
CMD ["java", "-jar", "app.jar"]
```

**Exempel med Alpine:**

```dockerfile
FROM openjdk:17-jre-alpine

# Alpine använder addgroup/adduser
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

USER appuser
```

### VOLUME - Mount points

**`VOLUME`** skapar en mount point för persistent data.

**Syntax:**

```dockerfile
VOLUME /path/to/volume
```

**Exempel:**

```dockerfile
VOLUME /data
VOLUME /var/log/myapp
```

**Vad gör den?**

Markerar att denna mapp ska innehålla data som behöver överleva när containern tas bort.

**Viktigt:** Vi kommer gå igenom volumes mycket mer i Docker Compose-dokumentet. Det här är bara en introduktion.

```dockerfile
FROM postgres:15
VOLUME /var/lib/postgresql/data    # Database data
```

## CMD vs ENTRYPOINT - skillnaden fördjupat

Det här kan vara förvirrande, så låt oss verkligen förstå skillnaden.

### Scenario 1: Bara CMD

```dockerfile
FROM alpine
CMD ["echo", "Hello World"]
```

```bash
docker run myimage                # Output: Hello World
docker run myimage echo "Hej"     # Output: Hej (override helt)
```

### Scenario 2: Bara ENTRYPOINT

```dockerfile
FROM alpine
ENTRYPOINT ["echo", "Hello"]
```

```bash
docker run myimage                # Output: Hello
docker run myimage "World"        # Output: Hello World (läggs till)
```

### Scenario 3: Både ENTRYPOINT och CMD

```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

```bash
docker run myimage                # Output: Hello World
docker run myimage "Hej"          # Output: Hej (ersätter CMD)
```

### Praktiskt exempel - Spring Boot

**Med bara CMD:**

```dockerfile
FROM openjdk:17-jre-slim
COPY app.jar .
CMD ["java", "-jar", "app.jar"]
```

**Med ENTRYPOINT + CMD:**

```dockerfile
FROM openjdk:17-jre-slim
COPY app.jar .
ENTRYPOINT ["java", "-jar", "app.jar"]
CMD []
```

**Med ENTRYPOINT + CMD för flexibilitet:**

```dockerfile
FROM openjdk:17-jre-slim
COPY app.jar .
ENTRYPOINT ["java"]
CMD ["-jar", "app.jar"]
```

Nu kan du enkelt override:

```bash
docker run myapp -version              # Kör: java -version
docker run myapp -jar app.jar --debug  # Kör: java -jar app.jar --debug
```

### När använda vad?

**Använd CMD när:**

- Containern kan köra olika kommandon
- Du vill lätt kunna köra andra saker i containern
- Det är en general-purpose image

**Använd ENTRYPOINT när:**

- Containern är som en executable
- Du alltid vill köra samma program
- Du vill att argument ska läggas till, inte ersätta

**Använd båda när:**

- Du har ett fast kommando med default-argument
- Du vill flexibilitet att ändra argumenten

## Image tags och versioner

### Vad är en tag?

En **tag** är som ett version-nummer eller namn för din image.

**Format:**

```
imagename:tag
```

**Exempel:**

```
myapp:1.0.0
myapp:latest
myapp:dev
nginx:1.25-alpine
openjdk:17-jre-slim
```

Om du inte anger tag används **`:latest`** automatiskt:

```bash
docker pull nginx        # Samma som nginx:latest
```

### Varför är :latest problematiskt?

`:latest` betyder inte "senaste versionen" - det betyder bara "default tag".

**Problem:**

```bash
# Idag
docker pull myapp:latest    # Får version 1.0

# En vecka senare
docker pull myapp:latest    # Får version 2.0 (breaking changes!)
```

Din app kanske slutar fungera eftersom du fick en helt annan version!

**I produktion:**

```dockerfile
# ✗ Dåligt - okontrollerbart
FROM node:latest

# ✓ Bra - vet exakt vad du får
FROM node:18.17-alpine
```

### Semantic versioning

Använd meningsfulla versions-nummer:

```
myapp:1.0.0    # Major.Minor.Patch
myapp:1.0.1    # Bug fix
myapp:1.1.0    # New feature
myapp:2.0.0    # Breaking change
```

### Flera tags för samma image

En image kan ha flera tags:

```bash
docker tag myapp:1.0.0 myapp:1.0
docker tag myapp:1.0.0 myapp:1
docker tag myapp:1.0.0 myapp:latest
```

Nu pekar alla dessa på samma image:

- `myapp:1.0.0`
- `myapp:1.0`
- `myapp:1`
- `myapp:latest`

### Bygga med tag

**Vid build:**

```bash
docker build -t myapp:1.0.0 .
docker build -t myapp:dev .
docker build -t myapp:$(git rev-parse --short HEAD) .  # Git commit hash
```

**Tagga befintlig image:**

```bash
docker tag myapp:1.0.0 myapp:latest
docker tag myapp:1.0.0 myregistry.com/myapp:1.0.0
```

## Bygga en image

Nu kör vi! Dags att faktiskt bygga en image från vår Dockerfile.

### Grundkommandot

```bash
docker build -t name:tag .
```

Punkten i slutet är viktigt! Den säger "använd denna mapp som **build context**".

**Exempel:**

```bash
cd mitt-projekt/
docker build -t myapp:1.0 .
```

### Vad händer?

När du kör kommandot ser du något liknande:

```
[+] Building 23.5s (10/10) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/openjdk:17-jre-slim
 => [1/4] FROM docker.io/library/openjdk:17-jre-slim
 => [internal] load build context
 => [2/4] WORKDIR /app
 => [3/4] COPY target/app.jar app.jar
 => [4/4] RUN some command
 => exporting to image
 => => naming to docker.io/library/myapp:1.0
```

Varje `=>` är ett steg från Dockerfilen.

### Build context

**Build context** är alla filer som skickas till Docker daemon när du bygger.

```bash
docker build -t myapp .    # Skickar allt i current directory
```

Om du har mycket filer (node_modules, .git, etc) blir det långsamt. Lösning: `.dockerignore` (kommer snart!).

### Vanliga flags

```bash
# Namnge imagen
docker build -t myapp:1.0 .

# Flera tags samtidigt
docker build -t myapp:1.0 -t myapp:latest .

# Skicka build arguments
docker build --build-arg VERSION=2.0 -t myapp .

# Använd annan Dockerfile
docker build -f Dockerfile.production -t myapp:prod .

# Bygg utan cache (force full rebuild)
docker build --no-cache -t myapp .

# Visa mer output
docker build --progress=plain -t myapp .
```

### Exempel build session

```bash
# Navigera till projektet
cd ~/projekt/min-spring-app

# Bygg JAR-filen först (om inte gjort)
mvn clean package

# Bygg Docker image
docker build -t min-spring-app:1.0.0 .

# Verifiera att den skapades
docker images

# Kör den
docker run -p 8080:8080 min-spring-app:1.0.0
```

## Layers och caching

Det här är **super viktigt** att förstå för att bygga effektiva Dockerfiles!

### Vad är ett layer?

Varje instruktion i Dockerfile skapar ett **layer** (lager). Imagen är uppbyggd av dessa layers ovanpå varandra.

```dockerfile
FROM openjdk:17-jre-slim    # Layer 1
WORKDIR /app                # Layer 2
COPY app.jar .              # Layer 3
CMD ["java", "-jar", "app.jar"]  # Layer 4
```

### Layers cachas

När du bygger en image första gången tar det tid. Men andra gången?

**Om en instruktion inte ändrats, använder Docker cached layer!**

```bash
# Första bygget
docker build -t myapp .    # 2 minuter

# Inget ändrat, bygg igen
docker build -t myapp .    # 1 sekund! (använder cache)
```

### När invalideras cachen?

När en instruktion ändras, invalideras **det lagret och alla layer efter**.

**Exempel:**

```dockerfile
FROM openjdk:17-jre-slim    # Layer 1 - cached
WORKDIR /app                # Layer 2 - cached
COPY pom.xml .              # Layer 3 - ÄNDRAD! Cache invalideras här
RUN mvn dependency:go-offline  # Layer 4 - måste byggas om
COPY src/ .                 # Layer 5 - måste byggas om
RUN mvn package             # Layer 6 - måste byggas om
CMD ["java", "-jar", "app.jar"]  # Layer 7 - måste byggas om
```

Eftersom layer 3 ändrades måste allt från och med layer 3 byggas om.

### Ordning spelar roll!

Sätt instruktioner som ändras sällan **först**, och de som ändras ofta **sist**.

**Tänk så här:**

1. **Ändras aldrig:** Base image (FROM)
2. **Ändras sällan:** System dependencies (apt-get install)
3. **Ändras ibland:** App dependencies (package.json, requirements.txt)
4. **Ändras ofta:** Din källkod

## Optimering - layer caching exempel

Låt oss se konkreta exempel på dålig vs bra Dockerfile struktur.

### Exempel: Node.js app

**❌ Dålig - Ineffektiv:**

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Kopierar ALLT först
COPY . .

# Installerar dependencies
RUN npm install

CMD ["node", "app.js"]
```

**Problem:** Varje gång du ändrar källkoden (vilket händer ofta), måste `npm install` köras om. Det tar tid!

**✅ Bra - Effektiv:**

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Kopiera bara package files först
COPY package*.json ./

# Installera dependencies (cachas om package.json inte ändrats)
RUN npm install

# Kopiera källkod (ändras ofta, men påverkar inte npm install)
COPY . .

CMD ["node", "app.js"]
```

**Varför är detta bättre?**

1. Första bygget: tar lika lång tid
2. Du ändrar `app.js`
3. Andra bygget:
   - `FROM`, `WORKDIR`, `COPY package*.json` - cached
   - `RUN npm install` - cached! (package.json ändrades inte)
   - `COPY . .` - körs (koden ändrades)
   - Resultat: Bygget tar sekunder istället för minuter

### Exempel: Python app

**❌ Dålig:**

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**✅ Bra:**

```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Dependencies först
COPY requirements.txt .
RUN pip install -r requirements.txt

# Kod sist
COPY . .

CMD ["python", "app.py"]
```

### Exempel: Spring Boot med Maven

**❌ Dålig:**

```dockerfile
FROM maven:3.9-eclipse-temurin-17
WORKDIR /app
COPY . .
RUN mvn clean package
CMD ["java", "-jar", "target/app.jar"]
```

**✅ Bra:**

```dockerfile
FROM maven:3.9-eclipse-temurin-17
WORKDIR /app

# Kopiera dependency-filer först
COPY pom.xml .
RUN mvn dependency:go-offline

# Kopiera källkod och bygg
COPY src ./src
RUN mvn clean package

CMD ["java", "-jar", "target/app.jar"]
```

Nu cachas alla Maven-dependencies så länge `pom.xml` inte ändras!

## Multi-stage builds

Multi-stage builds är ett av de kraftfullaste verktygen för att skapa små, effektiva images.

### Vad är multi-stage builds?

Du har **flera FROM-statements** i samma Dockerfile. Varje FROM börjar ett nytt "stage".

Du kan bygga din app i ett stage (med alla build-tools) och sedan kopiera bara den färdiga artefakten till ett annat stage (utan build-tools).

**Resultat:** Mycket mindre final image!

### Varför använda dem?

**Problem utan multi-stage:**

För att bygga en Java-app behöver du JDK (Java Development Kit) - stor image med massa verktyg.

Men för att **köra** Java-appen behöver du bara JRE (Java Runtime Environment) - mycket mindre.

Om du har allt i en stage:

```dockerfile
FROM openjdk:17-jdk-slim    # 400 MB
# Bygga och köra här
```

Med multi-stage:

```dockerfile
# Stage 1: Bygg med JDK - 400 MB (kastas bort)
# Stage 2: Kör med JRE - 200 MB (detta sparas)
```

### Syntax

```dockerfile
# Stage 1
FROM image1 AS stage-name
# Kommandon...

# Stage 2
FROM image2
COPY --from=stage-name /path/to/file /destination
```

### Exempel: Spring Boot app

**Komplett multi-stage Dockerfile:**

```dockerfile
# ========================================
# Stage 1: BUILD
# ========================================
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

# Kopiera dependency-filer och ladda ner dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Kopiera källkod och bygg
COPY src ./src
RUN mvn clean package -DskipTests

# Nu har vi en JAR-fil i /app/target/

# ========================================
# Stage 2: RUNTIME
# ========================================
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Kopiera JAR från build stage
COPY --from=build /app/target/*.jar app.jar

# Skapa non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Vad händer här?**

1. **Build stage:** Använder Maven + JDK för att bygga JAR-filen
2. **Runtime stage:** Använder bara JRE (mycket mindre) och kopierar färdig JAR
3. Final image innehåller INTE Maven, källkod, eller JDK - bara JRE och JAR-filen

**Storleksjämförelse:**

- Utan multi-stage: ~500 MB
- Med multi-stage: ~200 MB

### Exempel: Node.js app med build step

```dockerfile
# ========================================
# Stage 1: Build
# ========================================
FROM node:18-alpine AS build

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy source and build
COPY . .
RUN npm run build

# ========================================
# Stage 2: Runtime
# ========================================
FROM node:18-alpine

WORKDIR /app

# Copy node_modules från build stage
COPY --from=build /app/node_modules ./node_modules

# Copy built files
COPY --from=build /app/dist ./dist
COPY package*.json ./

# Non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

### Exempel: Go app (minimal image!)

Go kan kompilera till en enda statisk binär som inte behöver något OS.

```dockerfile
# Build stage
FROM golang:1.21-alpine AS build

WORKDIR /app
COPY . .
RUN go build -o myapp

# Runtime stage - FRÅN SCRATCH!
FROM scratch

COPY --from=build /app/myapp /myapp

EXPOSE 8080
CMD ["/myapp"]
```

Final image är typ 10 MB! 🤯

## .dockerignore

Precis som `.gitignore` fast för Docker!

### Vad är .dockerignore?

En fil som talar om för Docker vilka filer/mappar som INTE ska inkluderas i build context.

**Varför behövs det?**

När du kör `docker build .` skickas alla filer i mappen till Docker daemon. Om du har massa skit (node_modules, .git, build artifacts) blir det långsamt och imagen kan bli onödigt stor.

### Skapa .dockerignore

Skapa en fil som heter `.dockerignore` i samma mapp som Dockerfile:

```
mitt-projekt/
├── Dockerfile
├── .dockerignore    ← Här!
└── ...
```

### Exempel innehåll

```
# Git
.git
.gitignore
.gitattributes

# IDE
.idea
.vscode
*.swp
*.swo
*~

# Build artifacts
target/
build/
dist/
*.class

# Dependencies
node_modules/
vendor/

# Logs
*.log
logs/

# OS
.DS_Store
Thumbs.db

# Documentation
README.md
CHANGELOG.md
docs/

# Environment files
.env
.env.local

# Test files
test/
tests/
**/*_test.go
*.test

# CI/CD
.github/
.gitlab-ci.yml
Jenkinsfile
```

### Vanliga patterns

```
# Ignorera specifik fil
secret.txt

# Ignorera alla filer med extension
*.md
*.log

# Ignorera mapp
node_modules/
target/

# Ignorera allt i undermappar
**/temp/

# Exception - inkludera specifik fil trots pattern
!README.md    # Inkludera README trots *.md
```

### Java/Maven projekt

```
.git
.gitignore
.idea
*.iml
target/
*.log
README.md
```

### Node.js projekt

```
.git
.gitignore
node_modules/
npm-debug.log
.env
.vscode
coverage/
```

### Python projekt

```
.git
.gitignore
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
venv/
.env
.pytest_cache/
```

## Spring Boot Dockerfile - komplett exempel

Här är en production-ready Dockerfile för Spring Boot med allt vi lärt oss!

```dockerfile
# ========================================
# Multi-stage Dockerfile för Spring Boot
# ========================================

# ========================================
# Stage 1: BUILD
# ========================================
FROM maven:3.9-eclipse-temurin-17 AS build

# Arbeta i /build mappen
WORKDIR /build

# Kopiera Maven-filer först för dependency caching
COPY pom.xml .
COPY .mvn .mvn

# Ladda ner alla dependencies (cachas om pom.xml inte ändras)
RUN mvn dependency:go-offline -B

# Kopiera källkod
COPY src ./src

# Bygg applikationen (skippa tester för snabbare build, kör dom i CI/CD istället)
RUN mvn clean package -DskipTests -B

# Verifiera att JAR-filen skapades
RUN ls -la target/

# ========================================
# Stage 2: RUNTIME
# ========================================
FROM eclipse-temurin:17-jre-alpine

# Metadata om imagen
LABEL maintainer="dittnamn@example.com"
LABEL version="1.0"
LABEL description="Min Spring Boot applikation"

# Installera curl för health checks (optional)
RUN apk add --no-cache curl

# Skapa applikationsmapp
WORKDIR /app

# Skapa en dedikerad användare (säkerhet!)
RUN addgroup -S spring && adduser -S spring -G spring

# Kopiera JAR-filen från build stage
COPY --from=build /build/target/*.jar app.jar

# Ändra ägare på filer till spring-användaren
RUN chown -R spring:spring /app

# Byt till non-root användare
USER spring:spring

# Exponera porten (dokumentation)
EXPOSE 8080

# Environment variables med defaults
ENV SPRING_PROFILES_ACTIVE=production
ENV JAVA_OPTS="-Xmx512m -Xms256m"

# Health check (Docker kan använda detta för att se om containern är healthy)
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Starta applikationen
# Använd exec form för proper signal handling
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**Vad gör denna Dockerfile?**

1. ✅ **Multi-stage build** - Liten final image
2. ✅ **Layer caching** - Dependencies cachas separat från kod
3. ✅ **Non-root user** - Säkerhet
4. ✅ **Health check** - Docker kan monitorera app-status
5. ✅ **Environment variables** - Konfigurerbart
6. ✅ **Alpine base** - Minimal storlek
7. ✅ **Metadata** - Dokumentation via labels
8. ✅ **Proper signal handling** - Med exec form

**Använd den:**

```bash
# Bygg
docker build -t min-spring-app:1.0.0 .

# Kör med custom settings
docker run -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=dev \
  -e JAVA_OPTS="-Xmx1g" \
  min-spring-app:1.0.0

# Kör i bakgrunden med automatisk restart
docker run -d \
  --name my-app \
  --restart unless-stopped \
  -p 8080:8080 \
  min-spring-app:1.0.0
```

## Best practices

Här är de viktigaste best practices samlade:

### 1. Använd specifika base image tags

```dockerfile
# ✗ Dåligt
FROM node:latest
FROM openjdk

# ✓ Bra
FROM node:18.17-alpine
FROM eclipse-temurin:17-jre-alpine
```

### 2. Använd multi-stage builds för compiled languages

```dockerfile
# För Java, Go, Rust, C++, etc
FROM build-image AS build
# Build här

FROM runtime-image
COPY --from=build ...
```

### 3. Ordna instruktioner från minst till mest föränderlig

```dockerfile
FROM ...                    # Ändras aldrig
RUN apt-get install ...     # Ändras sällan
COPY package.json ...       # Ändras ibland
RUN npm install ...         # Ändras ibland
COPY . .                    # Ändras ofta
```

### 4. Kombinera RUN-kommandon

```dockerfile
# ✗ Dåligt - många layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim
RUN apt-get clean

# ✓ Bra - ett layer
RUN apt-get update && \
    apt-get install -y curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 5. Använd .dockerignore

```
# .dockerignore
.git
node_modules
*.md
.env
```

### 6. Kör inte som root

```dockerfile
# Skapa och byt till non-root user
RUN addgroup -S appuser && adduser -S appuser -G appuser
USER appuser
```

### 7. Håll images små

- Använd `-slim` eller `-alpine` variants
- Multi-stage builds
- Städa upp i samma layer:

```dockerfile
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 8. En process per container

Kör inte flera services i samma container. Varje service = en container.

```dockerfile
# ✗ Dåligt
CMD service nginx start && service mysql start && ...

# ✓ Bra
CMD ["nginx", "-g", "daemon off;"]
```

### 9. Använd COPY istället för ADD

```dockerfile
# ✓ Default till COPY
COPY app.jar .

# Använd ADD bara när du behöver dess special features
ADD https://example.com/file.tar.gz .
```

### 10. Lagra aldrig secrets i imagen

```dockerfile
# ✗ ALDRIG NÅGONSIN
ENV DB_PASSWORD=secret123

# ✓ Sätt vid runtime
docker run -e DB_PASSWORD=${DB_PASSWORD} myapp
```

### 11. Använd exec form för CMD/ENTRYPOINT

```dockerfile
# ✓ Bra - exec form
CMD ["java", "-jar", "app.jar"]

# ✗ Sämre - shell form
CMD java -jar app.jar
```

### 12. Lägg till health checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1
```

### 13. Dokumentera med EXPOSE och LABEL

```dockerfile
EXPOSE 8080
LABEL version="1.0" \
      description="My application"
```

## Common patterns för olika språk

### Java/Spring Boot

**Variant 1: Enkel (pre-built JAR):**

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

**Variant 2: Multi-stage (build i Docker):**

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

### Node.js

**Express/Next.js app:**

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Python/Flask/FastAPI

```dockerfile
FROM python:3.11-slim AS build
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app

# Kopiera installerade packages
COPY --from=build /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

COPY . .

# Non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### React/Vue/Angular (Static build)

```dockerfile
# Build stage
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Serve with nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Go

```dockerfile
# Build
FROM golang:1.21-alpine AS build
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o app

# Runtime - minimal!
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=build /app/app .
EXPOSE 8080
CMD ["./app"]
```

## Felsökning

### Build failures

**Problem:** Build misslyckas med felmeddelande

**Lösning:**

1. Läs felmeddelandet noga - det säger ofta vad som är fel
2. Kolla syntax i Dockerfilen
3. Verifiera att filer som COPY:as faktiskt finns

```bash
# Se mer detaljerad output
docker build --progress=plain -t myapp .

# Build utan cache
docker build --no-cache -t myapp .
```

### Cache-problem

**Problem:** Ändringar syns inte i imagen

**Lösning:**

```bash
# Force rebuild utan cache
docker build --no-cache -t myapp .
```

### Image för stor

**Problem:** Imagen är flera GB

**Lösningar:**

- Använd multi-stage builds
- Byt till -alpine eller -slim variants
- Städa upp i samma RUN-kommando
- Använd .dockerignore

```bash
# Se hur stor imagen är
docker images myapp

# Se vad varje layer innehåller
docker history myapp
```

### Kan inte hitta filer

**Problem:** `COPY failed: file not found`

**Lösning:**

- Kolla build context (punkten i `docker build .`)
- COPY är relativ till där Dockerfile ligger
- Kolla .dockerignore - kanske filen ignoreras

```bash
# Se vad som skickas som build context
docker build -t myapp . 2>&1 | grep "Sending build context"
```

### Container startar inte

**Problem:** Container kraschar direkt efter start

**Lösningar:**

```bash
# Se logs
docker logs container-name

# Se vad som hände
docker ps -a

# Starta med interaktiv shell för debugging
docker run -it myapp sh

# Se om filer finns där du tror
docker run myapp ls -la /app
```

### Permission-problem

**Problem:** `Permission denied` errors

**Lösning:**

```dockerfile
# Se till att non-root user har rätt permissions
COPY --chown=appuser:appuser . .

# Eller
COPY . .
RUN chown -R appuser:appuser /app
```

### Debugging-tips

**1. Shell in i running container:**

```bash
docker exec -it container-name sh
```

**2. Shell in i image:**

```bash
docker run -it --entrypoint sh myapp
```

**3. Se image history:**

```bash
docker history myapp
```

**4. Inspektera image:**

```bash
docker inspect myapp
```

**5. Build för specific stage:**

```bash
docker build --target build -t myapp:build .
```

## Testing your Dockerfile

### Bygga och verifiera

```bash
# 1. Bygg imagen
docker build -t myapp:test .

# 2. Kolla att den finns
docker images myapp:test

# 3. Kör den
docker run --rm -p 8080:8080 myapp:test

# 4. Testa i annan terminal
curl http://localhost:8080
```

### Inspektera innehållet

```bash
# Se vilka filer som finns
docker run --rm myapp:test ls -la /app

# Kolla environment variables
docker run --rm myapp:test env

# Shell in och utforska
docker run --rm -it myapp:test sh
```

### Verifiera image-storlek

```bash
# Lista images med storlek
docker images myapp

# Se detaljerad info om layers
docker history myapp:test
```

### Kolla logs

```bash
# Starta container
docker run -d --name myapp-test myapp:test

# Följ logs
docker logs -f myapp-test

# Se tidigare logs
docker logs myapp-test

# Städa upp
docker stop myapp-test
docker rm myapp-test
```

### Test i olika environments

```bash
# Test med olika environment variables
docker run --rm -e SPRING_PROFILES_ACTIVE=dev myapp:test

# Test med volume mount
docker run --rm -v $(pwd)/config:/app/config myapp:test

# Test networking
docker run --rm --network mynet myapp:test
```

## Sammanfattning

Vi har gått igenom mycket! Här är nyckelpunkterna:

### Dockerfile är ett recept

- Steg-för-steg instruktioner för att bygga en image
- Varje instruktion skapar ett layer
- Layers cachas för snabbare builds

### Viktigaste instruktionerna

- **FROM** - Bas-image att utgå från
- **WORKDIR** - Sätt arbetsmapp
- **COPY** - Kopiera filer in i imagen
- **RUN** - Kör kommandon under build
- **CMD** - Default kommando vid container start
- **ENTRYPOINT** - Gör containern till en executable
- **EXPOSE** - Dokumentera portar
- **ENV** - Environment variables

### Best practices

1. Använd specifika image tags (inte :latest)
2. Multi-stage builds för mindre images
3. Ordna instruktioner från minst till mest föränderlig
4. Kombinera RUN-kommandon för färre layers
5. Använd .dockerignore
6. Kör som non-root user
7. Håll images små med -alpine/-slim
8. Lagra aldrig secrets i imagen

### Multi-stage = mindre images

```dockerfile
FROM build-image AS build
# Bygg här

FROM runtime-image
COPY --from=build /app/output .
```

### Bygg images

```bash
docker build -t name:tag .
docker build --no-cache -t name:tag .
docker build --build-arg VERSION=1.0 -t name:tag .
```

Nu har du allt du behöver för att skriva effektiva Dockerfiles! 🚀

## Övningsuppgifter

### Övning 1: Basic Spring Boot Dockerfile

Skapa en Dockerfile för en Spring Boot-app. Du har en JAR-fil i `target/myapp-1.0.jar`.

**Krav:**

- Använd Java 17 JRE (inte JDK)
- Lägg JAR-filen i `/app`
- Exponera port 8080
- Starta appen med `java -jar`

<details>
<summary>Visa lösning</summary>

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/myapp-1.0.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

</details>

### Övning 2: Förklara caching

Förklara varför denna ordning är bättre för Node.js:

**Variant A:**

```dockerfile
COPY package.json .
RUN npm install
COPY . .
```

**Variant B:**

```dockerfile
COPY . .
RUN npm install
```

<details>
<summary>Visa svar</summary>

**Variant A är bättre** eftersom:

1. När du ändrar källkoden (vilket händer ofta), påverkar det inte `npm install`-steget
2. `npm install` cachas så länge `package.json` inte ändras
3. Detta sparar massor av tid - du behöver inte re-installera alla packages varje gång du ändrar en rad kod

**Variant B** innebär att `npm install` måste köras om varje gång du ändrar någon fil i projektet, vilket är väldigt långsamt.

</details>

### Övning 3: Multi-stage conversion

Konvertera denna single-stage Dockerfile till multi-stage:

```dockerfile
FROM maven:3.9-eclipse-temurin-17
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package
CMD ["java", "-jar", "target/myapp.jar"]
```

<details>
<summary>Visa lösning</summary>

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package

# Runtime stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/myapp.jar app.jar
CMD ["java", "-jar", "app.jar"]
```

**Fördelar:**

- Mindre image (JRE istället för JDK + Maven)
- Ingen källkod i final image
- Ingen Maven i final image

</details>

### Övning 4: Skapa .dockerignore

Du har ett Java Maven-projekt med denna struktur:

```
myproject/
├── src/
├── target/
├── .git/
├── .idea/
├── *.iml
├── logs/
├── README.md
├── pom.xml
└── Dockerfile
```

Skapa en `.dockerignore` som exkluderar onödiga filer.

<details>
<summary>Visa lösning</summary>

```
# .dockerignore
.git
.gitignore
.idea
*.iml
target/
logs/
*.log
README.md
.DS_Store
```

**Varför:**

- `.git` och `.idea` behövs inte i imagen
- `target/` kan vara gammal build (vi bygger i Dockerfile)
- `logs/` och `*.log` är runtime-data
- `README.md` är dokumentation

</details>

### Övning 5: Fixa denna dåliga Dockerfile

Denna Dockerfile har flera problem. Identifiera och fixa dem:

```dockerfile
FROM openjdk:latest
COPY . .
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim
EXPOSE 8080
CMD java -jar app.jar
```

<details>
<summary>Visa lösning</summary>

**Problem:**

1. Använder `:latest` tag
2. Kopierar allt direkt (ingen WORKDIR, ingen layer optimization)
3. Många RUN-kommandon (många layers)
4. Ingen cleanup efter apt-get
5. CMD i shell form istället för exec form
6. Saknar non-root user
7. Stora beroenden som kanske inte behövs

**Fixad version:**

```dockerfile
FROM eclipse-temurin:17-jre-alpine

# Alpine använder apk, inte apt-get
RUN apk add --no-cache curl

WORKDIR /app

COPY app.jar .

# Skapa non-root user
RUN addgroup -S appuser && adduser -S appuser -G appuser
USER appuser

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

**Förbättringar:**

- Specifik tag (17-jre-alpine)
- Alpine base = mindre image
- WORKDIR för organisation
- Kombinerade kommandon med cleanup
- Non-root user för säkerhet
- Exec form för CMD

</details>

### Övning 6: Praktisk lab

**Uppgift:** Skapa och kör din egen Dockerfile

1. Skapa en enkel Spring Boot app (eller använd en befintlig)
2. Skriv en Dockerfile för den
3. Skapa en `.dockerignore`
4. Bygg imagen
5. Kör den och verifiera att den fungerar
6. Tagga den med ett versionsnummer
7. Prova att ändra något i koden och bygg om - notera hur caching fungerar

**Steg:**

```bash
# 1. Bygg din Spring Boot JAR
mvn clean package

# 2. Skapa Dockerfile (använd tidigare exempel)

# 3. Skapa .dockerignore

# 4. Bygg imagen
docker build -t myapp:1.0.0 .

# 5. Kör den
docker run -p 8080:8080 myapp:1.0.0

# 6. Testa i browser/curl
curl http://localhost:8080

# 7. Tagga med "latest"
docker tag myapp:1.0.0 myapp:latest

# 8. Ändra något i koden, rebuilda och notera skillnaden
```

---

Grattis! Nu kan du Dockerfiles! 🎉 Nästa steg är att lära dig Docker Compose för att orkestrera flera containers tillsammans.
