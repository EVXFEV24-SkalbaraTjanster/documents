# CI/CD - Continuous Integration och Continuous Deployment

## Introduktion

Tänk dig att du ska bygga, testa, och deploya manuellt varje gång. För varje liten ändring. Fixa en bug? Bygg, testa, deploya. Lägg till en feature? Bygg, testa, deploya. Det blir gammal snabbt.

CI/CD är som att ha en robot som gör allt det tråkiga åt dig - bygga, testa, deploya - medan du fokuserar på att skriva kod. När du pushar kod till Git så tar roboten över och fixar resten automatiskt.

### Varför är det här viktigt?

I moderna utvecklingsteam deployas kod flera gånger om dagen. Netflix deployar tusentals gånger per dag. Det är helt omöjligt att göra manuellt. CI/CD gör det inte bara möjligt - det gör det enkelt och säkert.

**Manuell deployment:**

- Tar 30-60 minuter
- Riskabelt (glömmer steg, olika på varje dators miljö)
- Skrämmande (ingen vill deploya på fredag eftermiddag!)
- Långsam feedback (buggar upptäcks i produktion)

**Med CI/CD:**

- Tar 5-10 minuter
- Säkert (samma process varje gång)
- Ingen stress (automatiskt och testat)
- Snabb feedback (buggar fångas direkt)

## Det manuella problemet

Så här såg det ut förr (och tyvärr fortfarande på många ställen):

### Utvecklarens manuella workflow

1. **Skriv kod** - det enda roliga steget
2. **Kör tester lokalt** - om du kommer ihåg det
3. **Bygg JAR-fil** - `mvn clean package`
4. **SSH:a till server** - leta fram rätt IP och credentials
5. **Stoppa gamla versionen** - `systemctl stop myapp`
6. **Kopiera nya versionen** - `scp target/app.jar server:/app/`
7. **Starta nya versionen** - `systemctl start myapp`
8. **Kolla loggar** - `tail -f /var/log/myapp.log`
9. **Be en stilla bön** - att allt fungerar
10. **Om något gick fel** - panik och snabb rollback!

### Problem med manuell deployment

**Tidskrävande:**

- 30-60 minuter per deployment
- Multiplicera med 5 deployments per dag = 2.5-5 timmar bortkastade

**Felbenägen:**

- Glömde köra tester? Buggar i produktion
- Glömde bygga med rätt profil? Fel konfiguration
- Kopierade fel fil? Gammal version i produktion
- Olika miljö på olika utvecklares datorer? "Works on my machine!"

**Rädsla för deployment:**

- Ingen vill deploya på fredag kväll
- Stora deployments med många ändringar
- Högre risk = mer stress

**Långsam feedback:**

- Buggar upptäcks först i produktion
- Användare drabbas innan utvecklare vet om det
- Svårt att hitta vad som gick fel i stora deployments

### Ett verkligt scenario

Det är fredag 16:00. Du har fixat en viktig bug och ska deploya:

```bash
# Bygg lokalt
$ mvn clean package
[ERROR] Tests failed! 

# Åh nej, fixa testet snabbt
$ vim src/test/...
$ mvn test
[SUCCESS]

# Ok, bygg igen
$ mvn package

# SSH till server
$ ssh prod-server
Connection timeout...

# Fel server, försök igen
$ ssh prod-server-01
Enter password: ****

# Stoppa gamla versionen
$ sudo systemctl stop myapp
$ sudo systemctl status myapp
# Glömde vilket service-namn det var...

# 30 minuter senare...
# Äntligen deployed!
# Tittar på klockan: 17:30
# Loggar visar errors...
# Veckoslut = förstört
```

Det här är varför CI/CD finns.

## Vad är CI/CD?

CI/CD står för **Continuous Integration** och **Continuous Deployment** (eller Delivery). Det är två relaterade koncept som tillsammans automatiserar hela vägen från kod till produktion.

### CI - Continuous Integration

**Continuous Integration** betyder att du kontinuerligt integrerar din kod med resten av teamets kod.

**I praktiken:**

- När du pushar kod körs automatiskt:
  - Bygget (compilation)
  - Alla tester
  - Code quality checks
  - Security scanning
- Du får omedelbart reda på om något gick fel
- Merger till main branch ofta (flera gånger om dagen)

**Fördelar:**

- Buggar fångas tidigt
- Koden är alltid i ett byggbart state
- Merge conflicts blir mindre (små, frekventa merges)
- Alla utvecklare jobbar mot samma kod

**Exempel:**

```
Du: git push
CI: "Tack! Bygger och testar..."
    ✓ Build successful
    ✓ 127 tests passed
    ✓ Code coverage 85%
    ✓ No security vulnerabilities
CI: "Allt grönt! 🎉"
```

### CD - Continuous Delivery

**Continuous Delivery** betyder att koden alltid är redo att deployas.

**I praktiken:**

- Efter CI-steget byggs en deploybar artifact (JAR, Docker image)
- Artifacten kan deployas när som helst
- Ett manuellt godkännande krävs för production
- Deployment är en knapp-tryck

**Fördelar:**

- Mindre stress - deployment är testad
- Kan deploya när du vill (inte bara kontorstid)
- Snabbare time-to-market

### CD - Continuous Deployment

**Continuous Deployment** går steget längre - varje commit som passerar testerna deployas automatiskt till produktion.

**I praktiken:**

- Ingen manuell intervention
- Från commit till produktion på minuter
- Kräver mycket bra tester och monitoring

**Fördelar:**

- Snabbaste möjliga feedback
- Användare får features direkt
- Små, frekventa deployments (lägre risk)

**Nackdel:**

- Kräver mycket förtroende för testerna
- Inte lämpligt för alla typer av applikationer

### Hela pipelinen

```
Code push → Build → Test → Package → Deploy (Staging) → Deploy (Production)
    ↓         ↓       ↓        ↓           ↓                    ↓
  GitHub   Compile  Unit    Docker      Auto               Manual/Auto
           Java     Tests   Image                          Deployment
                    Integ.
                    Tests
```

## Fördelar med CI/CD

### 1. Hastighet

**Utan CI/CD:**

- Deployment: 1-2 gånger per vecka
- Varje deployment: 30-60 minuter
- Bug fix till produktion: dagar

**Med CI/CD:**

- Deployment: 5-50 gånger per dag
- Varje deployment: 5-10 minuter
- Bug fix till produktion: minuter till timmar

**Exempel från verkliga företag:**

- **Amazon**: deployment var 11.7 sekund (2011)
- **Netflix**: tusentals deployments per dag
- **Etsy**: 50+ deployments per dag

### 2. Kvalitet

**Automatiska tester fångar buggar:**

- Unit tests körs på varje commit
- Integration tests verifierar att komponenter fungerar tillsammans
- End-to-end tests simulerar användarflöden
- Buggar fångas innan de når produktion

**Konsekvent byggprocess:**

- Samma miljö varje gång
- Inga "works on my machine"-problem
- Reproducerbara byggen

**Code quality checks:**

- Linting (kodstil)
- Code coverage (testtäckning)
- Security scanning (sårbarheter)
- Static analysis (potentiella buggar)

### 3. Självförtroende

**Innan CI/CD:**

- "Vågar vi deploya den här ändringen?"
- "Vad om något går sönder?"
- "Låt oss vänta till efter helgen..."

**Med CI/CD:**

- "Testerna är gröna, kör!"
- Litar på automatiska tester
- Snabb rollback om något går fel
- Mindre ändringar = lägre risk

### 4. Produktivitet

**Utvecklare fokuserar på kod:**

- Ingen tid spenderad på manuell deployment
- Inga tråkiga repetitiva uppgifter
- Mer tid för att skriva features

**Mindre tid för debugging:**

- Buggar fångas tidigt
- Mindre "varför fungerar det inte i produktion?"-jakt
- Tydlig feedback på vad som gick fel

**Exempel:**

```
Utan CI/CD:
- Kod: 4 timmar
- Manuell deployment: 1 timme
- Debugging produktion: 1 timme
Total: 6 timmar för en feature

Med CI/CD:
- Kod: 4 timmar
- Push + vänta på CI/CD: 10 minuter
- (Debugging sker före deployment)
Total: 4 timmar för en feature
```

### 5. Snabb feedback

**Omedelbar feedback:**

- Vet inom 5-10 minuter om något är fel
- Kan fixa medan du fortfarande har kontexten i huvudet
- Inte "vad gjorde jag för tre dagar sedan?"

**Feedback loop:**

```
Utan CI/CD: Kod → (dagar) → Deployment → (timmar) → Upptäckt bug → (dagar) → Fix
Med CI/CD:  Kod → (minuter) → Test fail → (minuter) → Fix
```

## CI/CD verktyg

Det finns många verktyg för CI/CD. Vi kommer fokusera på GitHub Actions i den här kursen, men här är en översikt:

### GitHub Actions ⭐ (vårt val)

**Fördelar:**

- Inbyggt i GitHub (använder du redan)
- Gratis för publika repos
- Generös gratis-tier för privata repos (2000 minuter/månad)
- Enkel YAML-konfiguration
- Stort marketplace med färdiga actions
- Bra dokumentation

**Nackdelar:**

- Vendor lock-in (bundet till GitHub)
- Mindre flexibelt än Jenkins

**Perfekt för:**

- Små till medelstora projekt
- Open source projekt
- När du redan använder GitHub

### GitLab CI/CD

**Fördelar:**

- Inbyggt i GitLab
- Mycket kraftfullt
- Gratis self-hosted
- Bra Docker-integration

**Perfekt för:**

- GitLab-användare
- Self-hosted lösningar

### Jenkins

**Fördelar:**

- Mycket flexibelt
- Tusentals plugins
- Self-hosted (full kontroll)
- Gratis och open source

**Nackdelar:**

- Mer komplext att sätta upp
- Behöver underhåll av server
- Lite äldre gränssnitt

**Perfekt för:**

- Stora företag med särskilda krav
- När du behöver maximal flexibilitet
- Self-hosted lösningar

### CircleCI

**Fördelar:**

- Snabb
- Bra för Docker
- Cloud-baserad (ingen server att underhålla)

**Nackdelar:**

- Kostar för privata repos
- Mindre generös gratis-tier

### Travis CI

**Fördelar:**

- Populärt för open source
- Lätt att komma igång
- Bra GitHub-integration

**Nackdelar:**

- Mindre populärt nuförtiden
- Begränsad gratis-tier

### Andra alternativ

- **Azure DevOps** - bra om du använder Azure
- **AWS CodePipeline** - bra om du använder AWS
- **Bitbucket Pipelines** - för Bitbucket-användare
- **TeamCity** - kraftfullt men kommersiellt

### Varför GitHub Actions för den här kursen?

1. **Du använder troligen redan GitHub** - ingen ny plattform att lära sig
2. **Gratis för det mesta** - 2000 minuter räcker långt
3. **Enkelt att komma igång** - YAML-fil i repot, klart
4. **Bra dokumentation** - lätt att hitta hjälp
5. **Stort community** - många exempel och actions att använda

## GitHub Actions - grunderna

GitHub Actions är GitHubs inbyggda CI/CD-lösning. Allt konfigureras med YAML-filer i ditt repo.

### Kärnkoncept

**Workflow:**

- En automatiserad process (t.ex. "build and test")
- Definierad i en YAML-fil
- Ligger i `.github/workflows/` directory

**Event/Trigger:**

- Vad startar workflowen? (t.ex. push, pull request)
- Kan också vara schemalagd eller manuell

**Job:**

- En grupp av steg som körs tillsammans
- Flera jobs kan köra parallellt
- Exempel: "build", "test", "deploy"

**Step:**

- En individuell uppgift i ett job
- Kör antingen ett kommando eller en action
- Exempel: "checkout code", "run tests"

**Action:**

- En återanvändbar komponent
- Finns i GitHub Marketplace eller kan vara custom
- Exempel: `actions/checkout@v3`

**Runner:**

- En server som kör workflowen
- GitHub-hosted (gratis) eller self-hosted
- Ubuntu, Windows, eller macOS

### Visuell struktur

```
Repository
└── .github/
    └── workflows/
        ├── ci.yml          ← Workflow-fil
        └── deploy.yml      ← Ytterligare workflow
```

En workflow-fil:

```
Workflow: "CI"
├── Trigger: on push to main
├── Job: "build"
│   ├── Runner: ubuntu-latest
│   └── Steps:
│       ├── Checkout code
│       ├── Setup Java
│       ├── Build with Maven
│       └── Run tests
└── Job: "deploy"
    └── Steps: ...
```

### Första exempel - Workflow syntax

Här är en minimal workflow:

```yaml
name: Mitt första workflow

on: [push] # Kör på varje push

jobs:
  hello:
    runs-on: ubuntu-latest # Vilken runner

    steps:
      - name: Säg hej
        run: echo "Hej från GitHub Actions!"
```

**Spara som:** `.github/workflows/hello.yml`

När du pushar den här filen kommer GitHub Actions automatiskt:

1. Upptäcka workflowen
2. Köra den på nästa push
3. Visa resultatet i GitHub UI under "Actions"-fliken

## Första riktiga exemplet - Spring Boot CI

Nu bygger vi en riktig CI-pipeline för en Spring Boot-app!

### Vad ska vi göra?

1. Checka ut koden
2. Installera Java 17
3. Bygga med Maven
4. Köra alla tester
5. Visa resultat

### Komplett workflow

Skapa filen `.github/workflows/ci.yml`:

```yaml
name: CI - Build and Test

# När ska workflowen köras?
on:
  push:
    branches: [main] # På push till main
  pull_request:
    branches: [main] # På pull requests mot main

# Jobben som ska köras
jobs:
  build:
    name: Build och Test
    runs-on: ubuntu-latest # Kör på Ubuntu

    steps:
      # Steg 1: Checka ut koden
      - name: Checka ut kod
        uses: actions/checkout@v3

      # Steg 2: Installera Java
      - name: Sätt upp JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: "17"
          distribution: "temurin" # Eclipse Temurin (tidigare AdoptOpenJDK)

      # Steg 3: Bygg projektet
      - name: Bygg med Maven
        run: mvn clean package -DskipTests

      # Steg 4: Kör tester
      - name: Kör tester
        run: mvn test

      # Steg 5 (optional): Visa test-resultat
      - name: Publicera test-resultat
        if: always() # Kör även om tester failar
        uses: actions/upload-artifact@v3
        with:
          name: test-results
          path: target/surefire-reports/
```

### Så här fungerar det

**När du pushar kod:**

```
1. GitHub upptäcker ny commit på main branch
2. GitHub Actions startar workflow
3. Skapar en ny Ubuntu-VM
4. Kör stegen:
   ✓ Checkout code (10 sekunder)
   ✓ Setup Java (15 sekunder)
   ✓ Build with Maven (45 sekunder)
   ✓ Run tests (30 sekunder)
5. Visar resultat i GitHub UI
```

**Total tid:** ~2 minuter

### Förklaring av varje del

**`name:`** - Namnet som visas i GitHub UI

```yaml
name: CI - Build and Test
```

**`on:`** - Triggers för workflowen

```yaml
on:
  push:
    branches: [main] # Bara på main, inte alla branches
  pull_request:
    branches: [main] # På PRs mot main
```

**`jobs:`** - Jobben som ska köras

```yaml
jobs:
  build: # Job-ID (används för dependencies)
    name: Build och Test # Visuellt namn
    runs-on: ubuntu-latest # OS för runner
```

**`steps:`** - Individuella steg

**Uses vs Run:**

```yaml
- uses: actions/checkout@v3 # Använd en färdig action
- run: mvn clean package # Kör ett kommando
```

**`with:`** - Parametrar till action

```yaml
- uses: actions/setup-java@v3
  with:
    java-version: "17" # Parameter till actionen
    distribution: "temurin"
```

## Workflow triggers - när ska det köras?

Du kan styra exakt när en workflow körs med `on:`-sektionen.

### Push till specifika branches

```yaml
# Bara på main
on:
  push:
    branches: [main]

# På main och develop
on:
  push:
    branches: [main, develop]

# På alla branches utom main
on:
  push:
    branches-ignore: [main]

# På alla branches som matchar pattern
on:
  push:
    branches:
      - "feature/**" # feature/login, feature/payment, etc.
      - "release/**"
```

### Pull requests

```yaml
# På alla PRs
on: [pull_request]

# På PRs mot specifika branches
on:
  pull_request:
    branches: [main]

# På specifika PR-events
on:
  pull_request:
    types: [opened, synchronize, reopened]
```

### Schemalagda workflows

Använd cron-syntax för schemalagda körningar:

```yaml
# Varje dag kl 02:00 UTC
on:
  schedule:
    - cron: "0 2 * * *"

# Varje måndag kl 08:00
on:
  schedule:
    - cron: "0 8 * * 1"

# Varje timme
on:
  schedule:
    - cron: "0 * * * *"
```

**Cron syntax reminder:**

```
* * * * *
│ │ │ │ │
│ │ │ │ └─── Veckodag (0-6, Söndag = 0)
│ │ │ └───── Månad (1-12)
│ │ └─────── Dag (1-31)
│ └───────── Timme (0-23)
└─────────── Minut (0-59)
```

### Manuella triggers

```yaml
# Kan köras manuellt från GitHub UI
on: workflow_dispatch

# Med inputs
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Environment att deploya till"
        required: true
        default: "staging"
        type: choice
        options:
          - staging
          - production
```

### Kombinera flera triggers

```yaml
on:
  # På push till main
  push:
    branches: [main]

  # På pull requests
  pull_request:

  # Schemalagt varje natt
  schedule:
    - cron: "0 2 * * *"

  # Manuellt
  workflow_dispatch:
```

### Path filters - kör bara när vissa filer ändras

```yaml
on:
  push:
    paths:
      - "src/**" # Kör bara om src/ ändras
      - "pom.xml" # Eller pom.xml
      - ".github/workflows/**" # Eller workflows

# Ignorera vissa filer
on:
  push:
    paths-ignore:
      - "docs/**" # Skippa om bara docs ändras
      - "**.md" # Eller markdown-filer
```

## Jobs och Steps

Jobs är byggstenarna i en workflow. De kör steg sekventiellt och kan ha dependencies till andra jobs.

### Flera jobs - parallell körning

Som standard körs alla jobs parallellt:

```yaml
jobs:
  # Körs samtidigt
  build:
    runs-on: ubuntu-latest
    steps:
      - run: mvn package

  lint:
    runs-on: ubuntu-latest
    steps:
      - run: mvn checkstyle:check

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - run: mvn dependency-check:check
```

**Timeline:**

```
0s  ━━━━━━━━━━━━━ build ━━━━━━━━━━━━━ 45s
0s  ━━━━━ lint ━━━━━ 20s
0s  ━━━━━━━━━━ security-scan ━━━━━━━━━━ 35s

Total: 45s (längsta jobbet)
```

### Dependencies mellan jobs

Använd `needs:` för att skapa sekventiell ordning:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: mvn package

  test:
    runs-on: ubuntu-latest
    needs: build # Väntar på build
    steps:
      - run: mvn test

  deploy:
    runs-on: ubuntu-latest
    needs: [build, test] # Väntar på både build OCH test
    steps:
      - run: echo "Deploying..."
```

**Timeline:**

```
0s  ━━━━━━ build ━━━━━━ 30s
30s ━━━━━━ test ━━━━━━ 25s
55s ━━━━━━ deploy ━━━━━━ 10s

Total: 65s (summan av alla)
```

### Dela data mellan jobs - artifacts

Jobs körs på olika runners och delar inte filsystem. Använd artifacts:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: mvn package

      # Ladda upp JAR-filen som artifact
      - name: Spara JAR
        uses: actions/upload-artifact@v3
        with:
          name: app-jar
          path: target/*.jar

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      # Ladda ner artifacten från build-jobbet
      - name: Hämta JAR
        uses: actions/download-artifact@v3
        with:
          name: app-jar

      - run: ls -la # JAR-filen finns nu här!
      - run: echo "Deploying..."
```

### Conditional execution - kör bara ibland

Använd `if:` för att köra steg eller jobs villkorligt:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: mvn test

      # Kör bara om tester failar
      - name: Skicka Slack-notis
        if: failure()
        run: echo "Tests failed!"

      # Kör bara på main branch
      - name: Deploy
        if: github.ref == 'refs/heads/main'
        run: echo "Deploying..."

      # Kör alltid (även om tidigare steg failar)
      - name: Cleanup
        if: always()
        run: echo "Cleaning up..."
```

**Vanliga conditions:**

- `success()` - föregående steg lyckades (default)
- `failure()` - något steg failade
- `always()` - kör alltid
- `cancelled()` - workflowen avbröts

### Matrix strategy - testa på flera versioner

Testa på flera Java-versioner eller operativsystem samtidigt:

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        java: ["11", "17", "21"]

    steps:
      - uses: actions/checkout@v3

      - name: Setup JDK ${{ matrix.java }}
        uses: actions/setup-java@v3
        with:
          java-version: ${{ matrix.java }}
          distribution: "temurin"

      - run: mvn test
```

Detta skapar 9 jobs (3 OS × 3 Java-versioner) som körs parallellt!

```
✓ ubuntu-latest + Java 11
✓ ubuntu-latest + Java 17
✓ ubuntu-latest + Java 21
✓ windows-latest + Java 11
✓ windows-latest + Java 17
✓ windows-latest + Java 21
✓ macos-latest + Java 11
✓ macos-latest + Java 17
✓ macos-latest + Java 21
```

## Actions - återanvändbara komponenter

Actions är färdiga byggblock som du kan använda i dina workflows. Det finns tusentals i GitHub Marketplace.

### Vanliga actions

**Checkout kod:**

```yaml
- uses: actions/checkout@v3
```

Checkar ut ditt repo så att workflowen kan komma åt koden.

**Setup Java:**

```yaml
- uses: actions/setup-java@v3
  with:
    java-version: "17"
    distribution: "temurin"
```

Installerar Java på runnern.

**Setup Node.js:**

```yaml
- uses: actions/setup-node@v3
  with:
    node-version: "18"
```

**Setup Python:**

```yaml
- uses: actions/setup-python@v4
  with:
    python-version: "3.11"
```

**Cache dependencies:**

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.m2
    key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
```

**Upload artifacts:**

```yaml
- uses: actions/upload-artifact@v3
  with:
    name: my-artifact
    path: target/*.jar
```

**Download artifacts:**

```yaml
- uses: actions/download-artifact@v3
  with:
    name: my-artifact
```

### Docker actions

**Login till Docker Hub:**

```yaml
- uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

**Build och push Docker image:**

```yaml
- uses: docker/build-push-action@v4
  with:
    context: .
    push: true
    tags: myapp:latest
```

**Setup Docker Buildx:**

```yaml
- uses: docker/setup-buildx-action@v2
```

### Hitta fler actions

**GitHub Marketplace:** https://github.com/marketplace?type=actions

Sök efter vad du behöver:

- Slack notifications
- Deploy to AWS/Azure/GCP
- Security scanning
- Code coverage
- Database setup
- Osv...

### Använda actions

**Syntax:**

```yaml
- uses: owner/repo@version
  with:
    parameter: value
```

**Versioner:**

```yaml
- uses: actions/checkout@v3 # Specifik major version (rekommenderat)
- uses: actions/checkout@v3.5.2 # Exakt version
- uses: actions/checkout@main # Senaste från main (riskabelt)
- uses: actions/checkout@abc123 # Specifik commit
```

**Rekommendation:** Använd major version (`@v3`) för stabilitet men få bugfixar.

## Environment variables och Secrets

### Environment variables

Sätt variabler som kan användas i hela workflowen:

```yaml
# Global level - tillgänglig överallt
env:
  JAVA_VERSION: "17"
  APP_NAME: "myapp"

jobs:
  build:
    # Job level - bara i detta job
    env:
      MAVEN_OPTS: "-Xmx1024m"

    steps:
      # Step level - bara i detta steg
      - name: Build
        env:
          BUILD_ENV: "production"
        run: |
          echo "Java: $JAVA_VERSION"
          echo "App: $APP_NAME"
          echo "Maven opts: $MAVEN_OPTS"
          echo "Build env: $BUILD_ENV"
          mvn package
```

### Default environment variables

GitHub Actions tillhandahåller många inbyggda variabler:

```yaml
steps:
  - run: |
      echo "Repo: ${{ github.repository }}"
      echo "Branch: ${{ github.ref }}"
      echo "Commit: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Run ID: ${{ github.run_id }}"
      echo "Run number: ${{ github.run_number }}"
```

**Vanliga variabler:**

- `github.repository` - `owner/repo-name`
- `github.ref` - `refs/heads/main`
- `github.sha` - commit SHA
- `github.actor` - vem som triggade workflowen
- `github.event_name` - `push`, `pull_request`, etc.

### Secrets - hantera känsliga uppgifter

Secrets är säkert lagrade värden för känslig data:

- Passwords
- API keys
- Tokens
- SSH keys
- Certificates

**Varför secrets?**

- Aldrig synliga i logs
- Krypterade i GitHub
- Inte åtkomliga för forks (för säkerhet)
- Kan sättas per repo eller organization

### Lägga till secrets

**I GitHub UI:**

1. Gå till ditt repo
2. Settings → Secrets and variables → Actions
3. New repository secret
4. Namn: `DOCKER_PASSWORD`
5. Value: `dittLösenord123`
6. Add secret

### Använda secrets i workflows

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Login till Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Deploy till server
        env:
          SSH_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          # Secrets är tillgängliga som env vars
          echo "$SSH_KEY" > key.pem
          chmod 600 key.pem
          # Använd API_KEY här...
```

**OBS:** Secrets skrivs aldrig ut i logs. Om du försöker `echo ${{ secrets.PASSWORD }}` ser du bara `***`.

### Best practices för secrets

✅ **Gör:**

- Använd secrets för ALL känslig data
- Ge secrets tydliga namn: `PROD_DB_PASSWORD`, inte bara `PASSWORD`
- Rotera secrets regelbundet
- Använd olika secrets för olika miljöer

❌ **Gör inte:**

- Committa secrets i kod
- Dela secrets mellan många projekt (begränsa scope)
- Använd samma secret för produktion och development
- Skriva ut secrets i logs (även om GitHub censurerar dem)

### Organization secrets

För team/företag kan du sätta secrets på organization-nivå:

```
Organization Secrets → delas mellan alla repos
Repository Secrets → bara för ett repo
Environment Secrets → bara för en specifik environment
```

## Caching dependencies

Varje workflow-körning startar från en tom runner. Att ladda ner dependencies tar tid. Caching fixar det!

### Varför cache?

**Utan cache:**

```
1. Ladda ner Maven dependencies (60 sekunder)
2. Bygg (30 sekunder)
Total: 90 sekunder
```

**Med cache:**

```
1. Återställ cache (5 sekunder)
2. Bygg (30 sekunder)
Total: 35 sekunder
```

**Sparat:** 55 sekunder per körning!

### Cache Maven dependencies

```yaml
steps:
  - uses: actions/checkout@v3

  - uses: actions/setup-java@v3
    with:
      java-version: "17"
      distribution: "temurin"

  # Cache Maven dependencies
  - name: Cache Maven packages
    uses: actions/cache@v3
    with:
      path: ~/.m2/repository
      key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
      restore-keys: |
        ${{ runner.os }}-m2-

  - run: mvn package
```

**Hur det fungerar:**

1. **Key:** `ubuntu-latest-m2-abc123def` (baserat på pom.xml hash)
2. **Första körningen:** Cache miss → laddar ner dependencies → sparar i cache
3. **Nästa körning:** Cache hit → återställer från cache → snabbt!
4. **Om pom.xml ändras:** Nytt hash → cache miss → laddar ner nya dependencies

### Cache npm dependencies

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### Cache Gradle dependencies

```yaml
- uses: actions/cache@v3
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
    restore-keys: |
      ${{ runner.os }}-gradle-
```

### Restore-keys - fallback cache

Om exact match inte finns, försök med fallback:

```yaml
key: ubuntu-m2-abc123def # Försök exact match först
restore-keys: |
  ubuntu-m2-abc123                 # Sedan liknande
  ubuntu-m2-                       # Sedan vilken ubuntu-m2 som helst
```

### Cache size limits

- Max 10 GB per repo
- Gamla caches rensas automatiskt (inte använd på 7 dagar)
- Kom ihåg: cache är för snabbhet, inte för persistence!

## Bygga och testa - komplett exempel

Nu sätter vi ihop allt vi lärt oss till en komplett CI-pipeline!

```yaml
name: Java CI - Build and Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  JAVA_VERSION: "17"

jobs:
  build:
    name: Build och Unit Tests
    runs-on: ubuntu-latest

    steps:
      # Checkout kod
      - name: Checkout kod
        uses: actions/checkout@v3

      # Setup Java med cache
      - name: Setup JDK ${{ env.JAVA_VERSION }}
        uses: actions/setup-java@v3
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: "temurin"
          cache: "maven" # Inbyggd Maven cache!

      # Bygg utan tester (tester körs separat)
      - name: Bygg med Maven
        run: mvn -B clean package -DskipTests

      # Spara JAR för senare jobs
      - name: Upload JAR artifact
        uses: actions/upload-artifact@v3
        with:
          name: app-jar
          path: target/*.jar
          retention-days: 7

  test:
    name: Integration Tests
    runs-on: ubuntu-latest
    needs: build # Kör efter build

    # Kör med flera Java-versioner
    strategy:
      matrix:
        java: ["17", "21"]

    steps:
      - uses: actions/checkout@v3

      - name: Setup JDK ${{ matrix.java }}
        uses: actions/setup-java@v3
        with:
          java-version: ${{ matrix.java }}
          distribution: "temurin"
          cache: "maven"

      # Kör alla tester
      - name: Kör tester
        run: mvn -B test

      # Generera test coverage report
      - name: Generate coverage report
        run: mvn -B jacoco:report

      # Ladda upp test results
      - name: Upload test results
        if: always() # Kör även om tester failar
        uses: actions/upload-artifact@v3
        with:
          name: test-results-java-${{ matrix.java }}
          path: target/surefire-reports/

      # Ladda upp coverage report
      - name: Upload coverage report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report-java-${{ matrix.java }}
          path: target/site/jacoco/

  code-quality:
    name: Code Quality Checks
    runs-on: ubuntu-latest
    needs: build

    steps:
      - uses: actions/checkout@v3

      - name: Setup JDK
        uses: actions/setup-java@v3
        with:
          java-version: "17"
          distribution: "temurin"
          cache: "maven"

      # Checkstyle - kod-stil
      - name: Run Checkstyle
        run: mvn -B checkstyle:check

      # SpotBugs - hitta buggar
      - name: Run SpotBugs
        run: mvn -B spotbugs:check

      # OWASP Dependency Check - säkerhet
      - name: Run OWASP Dependency Check
        run: mvn -B dependency-check:check
```

### Vad händer när du pushar?

```
Push till main
    ↓
GitHub Actions startar
    ↓
    ├─→ build (30s)
    │   ├─ Checkout
    │   ├─ Setup Java
    │   ├─ Build
    │   └─ Upload JAR
    │
    ├─→ test (parallellt efter build)
    │   ├─ Java 17 (40s)
    │   └─ Java 21 (40s)
    │
    └─→ code-quality (parallellt efter build)
        ├─ Checkstyle (10s)
        ├─ SpotBugs (15s)
        └─ OWASP Check (20s)

Total tid: ~70s (build + längsta parallella jobbet)
```

### Resultat i GitHub UI

✅ **Alla gröna:**

```
✓ build - Build och Unit Tests
✓ test (17) - Integration Tests
✓ test (21) - Integration Tests  
✓ code-quality - Code Quality Checks
```

❌ **Om något failar:**

```
✓ build - Build och Unit Tests
✗ test (17) - Integration Tests (2 failed)
✓ test (21) - Integration Tests
✓ code-quality - Code Quality Checks
```

Du kan klicka på det röda krysset för att se exakt vad som gick fel!

## Bygga Docker images

Nu tar vi det ett steg längre - bygg Docker images i CI-pipelinen!

### Varför bygga Docker images i CI?

✅ **Fördelar:**

- Samma image i development, test, och produktion
- Ingen "works on my machine"-problem
- Snabb deployment (bara pull image)
- Versionshantering med tags
- Lätt att rulla tillbaka

### Komplett Docker workflow

```yaml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags:
      - "v*" # Push on version tags (v1.0.0, v2.1.3, etc.)

env:
  DOCKER_IMAGE: username/myapp # Ändra till ditt användarnamn

jobs:
  docker:
    name: Build och Push Docker Image
    runs-on: ubuntu-latest

    steps:
      # Checkout kod
      - name: Checkout kod
        uses: actions/checkout@v3

      # Setup Docker Buildx (förbättrad build-engine)
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v2

      # Login till Docker Hub
      - name: Login till Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # Extrahera metadata (tags, labels)
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.DOCKER_IMAGE }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      # Bygg och pusha image
      - name: Build och push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache
          cache-to: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache,mode=max
```

### Exempel Dockerfile för Spring Boot

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Vad händer?

**Vid push till main:**

```
Image taggas som:
- username/myapp:main
- username/myapp:sha-abc123
```

**Vid push av version tag (v1.2.3):**

```
Image taggas som:
- username/myapp:1.2.3
- username/myapp:1.2
- username/myapp:latest
- username/myapp:sha-abc123
```

### Förbättra build-tiden med cache

Docker Buildx cachar layers mellan builds:

```yaml
- name: Build och push
  uses: docker/build-push-action@v4
  with:
    context: .
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    # Cache från och till registry
    cache-from: type=registry,ref=username/myapp:buildcache
    cache-to: type=registry,ref=username/myapp:buildcache,mode=max
```

**Första build:** 5 minuter
**Nästa build (med cache):** 30 sekunder!

### Multi-platform images

Bygg för både x86 och ARM:

```yaml
- name: Build multi-platform image
  uses: docker/build-push-action@v4
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: username/myapp:latest
```

Nu fungerar imagen på:

- Intel/AMD servrar (x86)
- ARM servrar (t.ex. AWS Graviton)
- Apple Silicon Macs (M1/M2)

### Scannar för säkerhetsrisker

Lägg till säkerhetsscanning:

```yaml
# Efter build, scanna imagen
- name: Scan image med Trivy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: username/myapp:${{ github.sha }}
    format: "sarif"
    output: "trivy-results.sarif"

- name: Upload scan results
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: "trivy-results.sarif"
```

## Deployment - från CI till CD

Nu kommer den spännande delen - automatisk deployment!

### Deployment strategies

Det finns flera sätt att deploya:

#### 1. Manuell deployment (Continuous Delivery)

**Workflow:** Build → Test → Docker Image → [Vänta på godkännande] → Deploy

**Användning:**

- Production deployments
- När du vill ha mänsklig kontroll
- Regulatoriska krav

**Exempel:**

```yaml
jobs:
  deploy-prod:
    needs: [build, test]
    runs-on: ubuntu-latest
    environment: production # Kräver manuellt godkännande

    steps:
      - name: Deploy till produktion
        run: echo "Deploying..."
```

**I GitHub:** Settings → Environments → production → Required reviewers

#### 2. Automatisk deployment (Continuous Deployment)

**Workflow:** Build → Test → Docker Image → Deploy (automatiskt)

**Användning:**

- Development/Staging miljöer
- När du har bra tester
- För att få snabb feedback

**Exempel:**

```yaml
jobs:
  deploy-staging:
    needs: [build, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest

    steps:
      - name: Deploy till staging
        run: echo "Auto-deploying..."
```

### Deployment exempel - SSH till server

Deploya genom att SSH:a till server och starta ny container:

```yaml
name: Deploy to Server

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy till VPS
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/myapp
            docker-compose pull
            docker-compose up -d
            docker-compose logs -f --tail=50
```

**Sätt upp secrets:**

- `SERVER_HOST`: `192.168.1.100` (eller domän)
- `SERVER_USER`: `deploy`
- `SSH_PRIVATE_KEY`: Din privata SSH-nyckel

### Deployment med Docker Compose

**På servern - docker-compose.yml:**

```yaml
version: "3.8"

services:
  app:
    image: username/myapp:latest
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - DB_HOST=db
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  db-data:
```

**Deployment script:**

```bash
#!/bin/bash
cd /opt/myapp
docker-compose pull app  # Hämta senaste imagen
docker-compose up -d     # Starta med nya imagen
```

### Health checks och rollback

Verifiera att deployment lyckades:

```yaml
- name: Deploy
  run: |
    ssh user@server 'cd /opt/myapp && docker-compose up -d'

- name: Wait for app to start
  run: sleep 30

- name: Health check
  run: |
    response=$(curl -s -o /dev/null -w "%{http_code}" http://server:8080/actuator/health)
    if [ $response != "200" ]; then
      echo "Health check failed! Rolling back..."
      ssh user@server 'cd /opt/myapp && docker-compose down && docker-compose up -d'
      exit 1
    fi
    echo "Deployment successful!"
```

## Deployment patterns

Nu kollar vi på mer avancerade deployment-strategier.

### Rolling Update

Den enklaste strategin - uppdatera instanser en i taget.

**Hur det fungerar:**

```
Start: [V1] [V1] [V1]
Steg 1: [V2] [V1] [V1]  ← Uppdatera en
Steg 2: [V2] [V2] [V1]  ← Uppdatera nästa
Steg 3: [V2] [V2] [V2]  ← Uppdatera sista
```

**Fördelar:**

- Ingen downtime
- Simpel att implementera
- Fungerar med Docker Compose, Kubernetes, etc.

**Nackdelar:**

- Både gamla och nya versioner körs samtidigt
- Långsam rollback (måste uppdatera alla tillbaka)
- Svårt att testa nya versionen isolerat

**Exempel med Docker Compose:**

```bash
docker-compose up -d --no-deps --scale app=3 --no-recreate app
```

### Blue-Green Deployment

Två identiska miljöer - en aktiv (blue), en inaktiv (green).

**Hur det fungerar:**

```
Blue (Active)      Green (Inactive)
[V1] [V1] [V1]     [Empty]

Deploy V2 till Green:
[V1] [V1] [V1]     [V2] [V2] [V2]

Switch traffic:
[V1] [V1] [V1]     [V2] [V2] [V2] ← Active
  (Inactive)

Om något gick fel → Switch tillbaka till Blue!
```

**Fördelar:**

- Instant rollback (switch tillbaka)
- Testa nya versionen innan switch
- Noll downtime

**Nackdelar:**

- Dubbla resurser (dyrt)
- Database migrations kan vara knepiga
- Kräver load balancer

**Exempel setup:**

```yaml
# docker-compose-blue.yml
services:
  app-blue:
    image: myapp:v1
    ports:
      - "8080:8080"

# docker-compose-green.yml
services:
  app-green:
    image: myapp:v2
    ports:
      - "8081:8080"
```

**Nginx load balancer config:**

```nginx
upstream backend {
    server localhost:8080;  # Blue (aktiv)
    # server localhost:8081;  # Green (inaktiv)
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

**För att switcha:** Ändra config, reload Nginx:

```bash
# Update config to point to Green
sed -i 's/8080/8081/' /etc/nginx/conf.d/myapp.conf
nginx -s reload
```

### Canary Deployment

Deploya till en liten del av användarna först, sedan gradvis mer.

**Hur det fungerar:**

```
Start - alla på V1:
100% [V1] [V1] [V1] [V1]

Canary - 10% på V2:
10%  [V2]
90%  [V1] [V1] [V1]

Öka till 50%:
50%  [V2] [V2]
50%  [V1] [V1]

Alla på V2:
100% [V2] [V2] [V2] [V2]
```

**Fördelar:**

- Minimal risk - bara några % drabbas om det går fel
- Real-world testing
- Gradvis rollout

**Nackdelar:**

- Mer komplex setup
- Kräver metrics och monitoring
- Kräver smart load balancing

**Exempel med Nginx:**

```nginx
upstream backend {
    server v1-server:8080 weight=9;  # 90% trafik
    server v2-server:8080 weight=1;  # 10% trafik
}
```

**Workflow:**

1. Deploy V2 till 10% av servrar
2. Övervaka metrics (error rate, latency, osv.)
3. Om OK → öka till 25%
4. Om OK → öka till 50%
5. Om OK → öka till 100%
6. Om INTE OK → rulla tillbaka!

### Feature Flags

Deploya kod, men behåll features avstängda tills du är redo.

**Hur det fungerar:**

```java
@RestController
public class MyController {
    
    @Autowired
    private FeatureFlagService flags;
    
    @GetMapping("/feature")
    public Response getFeature() {
        if (flags.isEnabled("new-algorithm")) {
            return newAlgorithm();  // Ny kod
        } else {
            return oldAlgorithm();  // Gamla koden
        }
    }
}
```

**Configuration:**

```yaml
# application.yml
features:
  new-algorithm: false # Avstängd för alla

# Eller per användare/miljö
features:
  new-algorithm:
    enabled: true
    percentage: 10 # 10% av användare
    users: [admin, testuser]
```

**Fördelar:**

- Instant rollback (bara toggle flag)
- Testa i produktion för specifika användare
- Gradvis rollout utan nya deployments
- A/B testing

**Nackdelar:**

- Kod blir mer komplex
- Gamla feature flags måste städas bort
- Teknisk skuld om inte underhållet

**Verktyg för feature flags:**

- LaunchDarkly
- Unleash (open source)
- Split.io
- ConfigCat

## Notifications och statusrapportering

Få reda på vad som händer i din pipeline!

### Slack notifications

```yaml
jobs:
  notify:
    runs-on: ubuntu-latest
    if: always() # Kör även om andra jobs failar
    needs: [build, test, deploy]

    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: "Deployment ${{ job.status }}"
          webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
          fields: repo,message,commit,author
```

**Setup Slack webhook:**

1. Gå till Slack
2. Apps → Incoming Webhooks
3. Add to Slack
4. Välj kanal
5. Kopiera Webhook URL till GitHub Secrets

### Discord notifications

```yaml
- name: Discord notification
  uses: sarisia/actions-status-discord@v1
  if: always()
  with:
    webhook: ${{ secrets.DISCORD_WEBHOOK }}
    title: "Deployment"
    description: "Build ${{ job.status }}"
    color: 0x00ff00 # Green for success
```

### Email notifications

GitHub skickar email automatiskt vid failures, men du kan customizera:

```yaml
- name: Send email
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: "Build failed: ${{ github.repository }}"
    body: "Check it out: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
    to: team@example.com
```

### Status badges

Visa build-status i din README!

```markdown
![CI](https://github.com/username/repo/workflows/CI/badge.svg)
![Deploy](https://github.com/username/repo/workflows/Deploy/badge.svg)
```

Resultat: ![CI](https://img.shields.io/badge/build-passing-brightgreen) ![Deploy](https://img.shields.io/badge/deploy-success-brightgreen)

**Custom badge med shields.io:**

```markdown
![Coverage](https://img.shields.io/badge/coverage-85%25-green)
![Tests](https://img.shields.io/badge/tests-127%20passed-brightgreen)
```

## Testing strategies i CI/CD

Bra tester är grunden för lyckad CI/CD.

### Test pyramid

```
      ╱╲
     ╱E2E╲         ← Få, långsamma (minuter)
    ╱──────╲
   ╱ Integr ╲      ← Några, medel (sekunder)
  ╱──────────╲
 ╱Unit Tests  ╲    ← Många, snabba (millisekunder)
╱──────────────╲
```

### Unit tests

**Vad:** Testa enskilda funktioner/metoder
**Hur snabbt:** Millisekunder per test
**När:** På varje commit

```yaml
- name: Unit tests
  run: mvn test -Dtest=*UnitTest
```

**Exempel:**

```java
@Test
void testAddition() {
    Calculator calc = new Calculator();
    assertEquals(4, calc.add(2, 2));
}
```

### Integration tests

**Vad:** Testa komponenter tillsammans (databas, API, etc.)
**Hur snabbt:** Sekunder per test
**När:** På varje merge till main

```yaml
- name: Integration tests
  run: mvn verify -Dtest=*IntegrationTest
```

**Med Testcontainers:**

```java
@SpringBootTest
@Testcontainers
class UserServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15");
    
    @Test
    void testUserCreation() {
        // Testa mot riktig databas!
    }
}
```

### End-to-end tests

**Vad:** Testa hela systemet som användare
**Hur snabbt:** Minuter
**När:** Innan deployment eller nattliga

```yaml
- name: E2E tests
  run: mvn test -Dtest=*E2ETest
```

**Med Selenium:**

```java
@Test
void testLoginFlow() {
    driver.get("http://localhost:8080");
    driver.findElement(By.id("username")).sendKeys("testuser");
    driver.findElement(By.id("password")).sendKeys("password");
    driver.findElement(By.id("login")).click();
    
    assertTrue(driver.getCurrentUrl().contains("/dashboard"));
}
```

### Code coverage

Mät hur mycket av koden som täcks av tester:

```yaml
- name: Generate coverage report
  run: mvn jacoco:report

- name: Upload coverage
  uses: codecov/codecov-action@v3
  with:
    files: target/site/jacoco/jacoco.xml
```

**Badge:**

```markdown
![Coverage](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)
```

### Static code analysis

**Checkstyle** - kod-stil:

```yaml
- name: Checkstyle
  run: mvn checkstyle:check
```

**SpotBugs** - hitta buggar:

```yaml
- name: SpotBugs
  run: mvn spotbugs:check
```

**SonarQube** - omfattande analys:

```yaml
- name: SonarQube scan
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  run: mvn sonar:sonar
```

### Security scanning

**OWASP Dependency Check** - sårbara dependencies:

```yaml
- name: OWASP Check
  run: mvn dependency-check:check
```

**Snyk** - vulnerability scanning:

```yaml
- name: Snyk scan
  uses: snyk/actions/maven@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

**Trivy** - container scanning:

```yaml
- name: Scan Docker image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: username/myapp:latest
```

## Komplett exempel - Full CI/CD pipeline

Nu sätter vi ihop ALLT till en komplett pipeline från kod till produktion!

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  JAVA_VERSION: "17"
  DOCKER_IMAGE: username/myapp

jobs:
  # ============================================
  # STEG 1: BUILD & TEST
  # ============================================
  build:
    name: Build Application
    runs-on: ubuntu-latest

    steps:
      - name: Checkout kod
        uses: actions/checkout@v3

      - name: Setup JDK
        uses: actions/setup-java@v3
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: "temurin"
          cache: "maven"

      - name: Build med Maven
        run: mvn -B clean package -DskipTests

      - name: Upload JAR
        uses: actions/upload-artifact@v3
        with:
          name: app-jar
          path: target/*.jar

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: build

    steps:
      - uses: actions/checkout@v3

      - name: Setup JDK
        uses: actions/setup-java@v3
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: "temurin"
          cache: "maven"

      - name: Run unit tests
        run: mvn test

      - name: Run integration tests
        run: mvn verify -DskipUnitTests

      - name: Generate coverage report
        run: mvn jacoco:report

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: target/site/jacoco/jacoco.xml

  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    needs: build

    steps:
      - uses: actions/checkout@v3

      - name: Setup JDK
        uses: actions/setup-java@v3
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: "temurin"
          cache: "maven"

      - name: Checkstyle
        run: mvn checkstyle:check

      - name: SpotBugs
        run: mvn spotbugs:check

      - name: OWASP Dependency Check
        run: mvn dependency-check:check

  # ============================================
  # STEG 2: DOCKER BUILD
  # ============================================
  docker:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [test, quality]
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login till Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.DOCKER_IMAGE }}
          tags: |
            type=sha
            type=raw,value=latest

      - name: Build och push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache
          cache-to: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache,mode=max

      - name: Scan image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.DOCKER_IMAGE }}:${{ github.sha }}
          format: "sarif"
          output: "trivy-results.sarif"

      - name: Upload scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: "trivy-results.sarif"

  # ============================================
  # STEG 3: DEPLOY STAGING (automatisk)
  # ============================================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: docker
    environment: staging

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/myapp
            docker-compose pull
            docker-compose up -d
            docker-compose logs -f --tail=50 &
            sleep 30

      - name: Health check
        run: |
          response=$(curl -s -o /dev/null -w "%{http_code}" ${{ secrets.STAGING_URL }}/actuator/health)
          if [ $response != "200" ]; then
            echo "Health check failed!"
            exit 1
          fi
          echo "Staging deployment successful!"

      - name: Run smoke tests
        run: |
          # Kör grundläggande tester mot staging
          curl -f ${{ secrets.STAGING_URL }}/api/status || exit 1
          echo "Smoke tests passed!"

  # ============================================
  # STEG 4: DEPLOY PRODUCTION (manuellt godkännande)
  # ============================================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production # Kräver manuellt godkännande i GitHub

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PRODUCTION_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/myapp
            docker-compose pull
            docker-compose up -d --no-deps app
            docker-compose logs -f --tail=50 &
            sleep 30

      - name: Health check
        run: |
          response=$(curl -s -o /dev/null -w "%{http_code}" ${{ secrets.PRODUCTION_URL }}/actuator/health)
          if [ $response != "200" ]; then
            echo "Production health check failed! Rolling back..."
            # Trigger rollback här
            exit 1
          fi
          echo "Production deployment successful!"

      - name: Send success notification
        uses: 8398a7/action-slack@v3
        with:
          status: success
          text: "🚀 Production deployment successful!"
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}

  # ============================================
  # NOTIFIKATIONER
  # ============================================
  notify-failure:
    name: Notify on Failure
    runs-on: ubuntu-latest
    if: failure()
    needs: [build, test, quality, docker, deploy-staging, deploy-production]

    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          text: "❌ Pipeline failed!"
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
          fields: repo,message,commit,author
```

### Vad händer i denna pipeline?

```
Push till main
    ↓
┌─────────────┐
│   BUILD     │ (30s)
│ + Unit Test │
└─────────────┘
    ↓
┌─────────────┬─────────────┐
│    TEST     │   QUALITY   │ (parallellt, 40s)
│ Integration │  Checkstyle │
│   + E2E     │  SpotBugs   │
└─────────────┴─────────────┘
    ↓
┌─────────────┐
│   DOCKER    │ (2 min)
│ Build+Push  │
│ + Scan      │
└─────────────┘
    ↓
┌─────────────┐
│   STAGING   │ (automatisk, 1 min)
│   Deploy    │
│ + Smoke Test│
└─────────────┘
    ↓
[Vänta på godkännande]
    ↓
┌─────────────┐
│ PRODUCTION  │ (manuell, 1 min)
│   Deploy    │
│ + Health    │
└─────────────┘
    ↓
Slack: "🚀 Deployed to prod!"
```

**Total tid:** ~5-7 minuter från push till staging, +1 min till produktion efter godkännande.

## Best practices för CI/CD

### 1. Håll builds snabba

❌ **Dåligt:**

```
Build: 10 minuter
Test: 15 minuter
Total: 25 minuter
```

Utvecklare väntar 25 minuter på feedback!

✅ **Bra:**

```
Build: 2 minuter
Unit tests: 3 minuter
Total: 5 minuter
```

**Hur:**

- Cacha dependencies
- Kör tester parallellt
- Använd matrix strategy
- Dela upp i flera jobs
- Kör E2E-tester separat (nightly)

### 2. Fail fast

Kör de snabbaste testerna först:

```yaml
jobs:
  lint: # 10 sekunder
  unit-test: # 2 minuter - kör bara om lint OK
  build: # 3 minuter - kör bara om unit-test OK
  e2e: # 10 minuter - kör bara om build OK
```

Ingen idé att vänta 10 minuter på E2E om lint failar på 10 sekunder!

### 3. Reproducerbara builds

✅ **Gör:**

- Pinna versioner: `actions/checkout@v3`, inte `@main`
- Använd samma Java-version överallt
- Samma Maven-version
- Samma Docker base image

❌ **Gör inte:**

- Förlita dig på externa tjänster utan fallback
- Använda `latest` tags
- Olika versioner i dev vs CI

### 4. Säkerhet

✅ **Gör:**

- Använd secrets för ALL känslig data
- Scanna dependencies för vulnerabilities
- Scanna Docker images
- Minimal permissions för deployment
- Rotera secrets regelbundet

❌ **Gör inte:**

- Committa secrets i kod
- Printa secrets i logs
- Ge full server access till CI
- Ignorera security warnings

### 5. Testing

✅ **Gör:**

- Hög testtäckning (>80%)
- Snabba, reliable tester
- Test pyramid (många unit, färre E2E)
- Tester ska vara deterministiska

❌ **Gör inte:**

- Flaky tester (ibland pass, ibland fail)
- Skippa tester i CI
- Bara E2E-tester (långsamt)
- Tester som beror på externa tjänster utan mock

### 6. Deployment

✅ **Gör:**

- Små, frekventa deployments
- Automated rollback på failure
- Health checks efter deployment
- Deploy till staging först
- Manuellt godkännande för produktion

❌ **Gör inte:**

- Stora deployments med många ändringar
- Deploya utan tester
- Deploya fredag kväll
- Glömma rollback-plan

### 7. Monitoring och logging

✅ **Gör:**

- Notifications på failures
- Status badges i README
- Logga vad som händer
- Tracka build times
- Mät deployment frequency

❌ **Gör inte:**

- Ignorera failures
- För många notifications (alert fatigue)
- Ingen logging
- Ingen metrics

### 8. Documentation

✅ **Gör:**

- Dokumentera workflows
- Kommentera komplexa steg
- README med badges
- Runbook för troubleshooting

❌ **Gör inte:**

- Komplicerade workflows utan dokumentation
- Kryptiska job-namn
- Ingen förklaring av secrets

## Vanliga fallgropar

### 1. Flaky tester

**Problem:** Tester som ibland failar, ibland passerar (utan kodändringar).

**Orsaker:**

- Timing issues (race conditions)
- Beror på externa tjänster
- Random data som inte är deterministiskt
- Delat state mellan tester

**Lösning:**

```java
// Dåligt - race condition
@Test
void test() {
    service.asyncOperation();
    assertEquals(expected, service.getResult());  // Kan faila!
}

// Bra - vänta på resultat
@Test
void test() {
    service.asyncOperation();
    await().atMost(5, SECONDS)
           .until(() -> service.isComplete());
    assertEquals(expected, service.getResult());
}
```

### 2. Långsamma builds

**Problem:** Builds tar 10-20 minuter.

**Lösningar:**

- Cache dependencies
- Parallellisera jobb
- Kör bara nödvändiga tester
- Använd matrix för flera versioner

```yaml
# Dåligt - sekventiell
jobs:
  build-and-test: # 15 minuter

# Bra - parallellt
jobs:
  build: # 3 minuter
  unit: # 2 minuter (parallellt med build)
  integration: # 5 minuter (efter build)
```

### 3. "Works on my machine"

**Problem:** Fungerar lokalt, failar i CI.

**Orsaker:**

- Olika Java-version
- Olika dependencies
- Olika miljövariabler
- Tidszoner

**Lösning:**

```yaml
# Samma miljö i CI som lokalt
- uses: actions/setup-java@v3
  with:
    java-version: "17" # Samma som lokalt!
    distribution: "temurin"
```

### 4. Secrets i logs

**Problem:** Råkar logga känslig data.

```yaml
# Dåligt
- run: echo "Password: ${{ secrets.PASSWORD }}"

# GitHub censurerar automatiskt, men undvik ändå!
# Output: Password: ***
```

### 5. No rollback plan

**Problem:** Deployment failar i produktion, ingen plan för att rulla tillbaka.

**Lösning:**

- Spara föregående Docker image: `myapp:previous`
- Blue-green deployment
- Feature flags för att stänga av features

```yaml
- name: Rollback on failure
  if: failure()
  run: |
    docker-compose pull myapp:previous
    docker-compose up -d
```

### 6. Deploying utan testing

**Problem:** Skippar tester för att "gå snabbare".

**Resultat:** Buggar i produktion, mer tid spenderad på att fixa.

**Regel:** Testerna måste vara gröna innan deployment!

```yaml
deploy:
  needs: [build, test, quality] # Kör INTE deploy om dessa failar
```

### 7. Alert fatigue

**Problem:** För många notifications - börjar ignorera dem.

**Lösning:**

- Notifiera bara på viktiga events (failures, production deployments)
- Gruppera notifications
- Använd rätt kanal (Slack för team, email för kritiskt)

## Debugging workflows

### Titta på logs

I GitHub UI: Actions → Välj workflow run → Klicka på job → Se logs

**Varje steg visar:**

- Vad som kördes
- Output
- Exit code

### Re-run failed jobs

Klicka "Re-run failed jobs" för att köra om bara det som failade.

### Lägga till debug logging

```yaml
- name: Debug information
  run: |
    echo "Runner OS: ${{ runner.os }}"
    echo "Java version:"
    java -version
    echo "Maven version:"
    mvn -version
    echo "Environment variables:"
    env | sort
```

### Enable debug logging

För mer detaljerade logs, sätt GitHub Secret:

- `ACTIONS_RUNNER_DEBUG` = `true`
- `ACTIONS_STEP_DEBUG` = `true`

### SSH into runner (för debugging)

```yaml
- name: Setup tmate session
  uses: mxschmitt/action-tmate@v3
  if: failure() # Bara om något failar
```

Detta ger dig en SSH-session in i runnern så du kan debugga!

### Testa workflow lokalt

Använd `act` för att köra GitHub Actions lokalt:

```bash
# Installera act
brew install act  # macOS
# eller
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Kör workflow
act

# Kör specifik job
act -j build

# Kör med secrets
act -s DOCKER_PASSWORD=secret123
```

## Kostnader och resource limits

### GitHub Actions free tier

**Publika repos:**

- Obegränsade minuter ✅
- Gratis för alltid

**Privata repos:**

- 2000 minuter/månad (gratis tier)
- Därefter $0.008/minut

**Beräkning:**

```
Build: 5 minuter
Körs: 50 gånger/dag
Månadsförbrukning: 5 × 50 × 30 = 7500 minuter

Kostnad: (7500 - 2000) × $0.008 = $44/månad
```

### Optimera costs

**1. Cacha dependencies:**
Sparar 1-2 minuter per build = 30% kostnadsminskning!

**2. Kör bara vid behov:**

```yaml
on:
  push:
    branches: [main] # Inte alla branches
    paths-ignore:
      - "docs/**" # Skippa om bara docs ändras
```

**3. Self-hosted runners:**
Kör på egna servrar = gratis minuter!

```yaml
jobs:
  build:
    runs-on: self-hosted # Din egen server
```

**4. Parallellisera:**
5 jobb á 2 minuter (parallellt) = 2 minuter total
5 jobb á 2 minuter (sekventiellt) = 10 minuter total

### Storage limits

- Artifacts: max 10 GB per repo
- Logs: sparas i 90 dagar
- Caches: max 10 GB per repo

**Rensa gamla artifacts:**

```yaml
- uses: actions/upload-artifact@v3
  with:
    name: my-artifact
    path: target/*.jar
    retention-days: 7 # Radera efter 7 dagar
```

## Nästa steg

### 1. Börja enkelt

Starta med basic CI:

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: mvn package
      - run: mvn test
```

### 2. Lägg till mer över tid

- Lägg till caching
- Lägg till code quality checks
- Bygg Docker image
- Deploy till staging
- Deploy till produktion

### 3. Iterera och förbättra

- Mät build times - optimera
- Lägg till fler tester
- Förbättra notifications
- Lägg till monitoring

### 4. Lär från failures

Varje failure är en learning opportunity:

- Varför failade det?
- Hur kan vi förhindra det?
- Behöver vi fler tester?
- Behöver vi bättre error handling?

### 5. Dela med teamet

- Dokumentera workflows
- Teacha teamet om CI/CD
- Code review på workflow-ändringar
- Kontinuerlig förbättring

## Sammanfattning

### Vad är CI/CD?

**CI (Continuous Integration):**

- Automatisk build och test på varje commit
- Fångar buggar tidigt
- Håller koden i deploybart state

**CD (Continuous Delivery/Deployment):**

- Automatisk deployment (med eller utan manuellt godkännande)
- Snabbare time-to-market
- Mindre risk (små, frekventa deployments)

### Varför CI/CD?

✅ **Hastighet:** Deploya många gånger per dag istället för per vecka
✅ **Kvalitet:** Automatiska tester fångar buggar
✅ **Självförtroende:** Lita på processen
✅ **Produktivitet:** Fokusera på kod, inte deployment
✅ **Feedback:** Omedelbar feedback på ändringar

### GitHub Actions basics

**Workflow:** Automatiserad process i YAML
**Trigger:** När körs det? (push, PR, schedule, manual)
**Job:** Grupp av steg
**Step:** Individuell uppgift
**Action:** Återanvändbar komponent

### Grundläggande workflow

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: "17"
      - run: mvn package
      - run: mvn test
```

### Best practices

1. **Snabba builds** - cache dependencies, parallellisera
2. **Fail fast** - kör snabba tester först
3. **Säkerhet** - använd secrets, scanna vulnerabilities
4. **Testing** - hög täckning, reliable tester
5. **Små deployments** - deploy ofta med små ändringar
6. **Monitoring** - notifications, logs, metrics
7. **Documentation** - dokumentera workflows

### Deployment strategies

- **Rolling update:** Uppdatera instanser en i taget
- **Blue-green:** Två miljöer, switch mellan dem
- **Canary:** Gradvis rollout till fler användare
- **Feature flags:** Deploy kod, toggle features

### Framgång med CI/CD

🎯 Börja enkelt → Lägg till funktionalitet gradvis → Iterera och förbättra

💡 Varje failure = learning opportunity

🚀 Målet: Deploya med självförtroende, ofta, och säkert!

## Övningsuppgifter

### 1. Grundläggande CI workflow

**Uppgift:** Skapa en CI-workflow för din Spring Boot-app.

**Krav:**

- Körs på push till main och på pull requests
- Checka ut koden
- Setup Java 17
- Bygg med Maven
- Kör alla tester

**Extra:** Ladda upp JAR-filen som artifact.

### 2. Lägg till caching

**Uppgift:** Förbättra din workflow från uppgift 1 med Maven-cache.

**Jämför:**

- Hur lång tid tar första build?
- Hur lång tid tar andra build (med cache)?
- Hur mycket tid sparas?

### 3. Docker build och push

**Uppgift:** Skapa en workflow som bygger och pushar Docker image till Docker Hub.

**Krav:**

- Körs bara på main branch
- Login till Docker Hub (använd secrets)
- Bygg Docker image
- Pusha till Docker Hub
- Tagga med både `latest` och commit SHA

**Secrets du behöver:**

- `DOCKER_USERNAME`
- `DOCKER_PASSWORD`

### 4. Multi-environment deployment

**Uppgift:** Skapa workflows för deployment till staging och production.

**Krav:**

- **Staging:** Automatisk deployment vid push till main
- **Production:** Manuellt godkännande krävs
- Health check efter deployment
- Rollback om health check failar

**Tips:** Använd GitHub Environments.

### 5. Matrix testing

**Uppgift:** Testa din app på flera Java-versioner.

**Krav:**

- Testa på Java 17 och Java 21
- Kör alla tester på båda versionerna
- Visa resultat separat för varje version

**Bonus:** Lägg till testing på olika operativsystem (Ubuntu, Windows, macOS).

### 6. Förklara skillnaden

**Fråga:** Vad är skillnaden mellan Continuous Delivery och Continuous Deployment?

**Förklara:**

- Hur workflows skiljer sig
- När man använder vilket
- För- och nackdelar med vardera

### 7. Design en pipeline

**Uppgift:** Du har ett system med 3 microservices (user-service, order-service, payment-service).

**Design:**

- Hur skulle du strukturera CI/CD-pipelines?
- En workflow per service eller en för alla?
- Hur hanterar du dependencies mellan services?
- Deployment-ordning?

**Rita:** Ett diagram över hela pipelinen.

### 8. Jämför deployment strategies

**Fråga:** Jämför Rolling Update, Blue-Green, och Canary deployment.

**För varje, beskriv:**

- Hur det fungerar
- Fördelar
- Nackdelar
- När det är lämpligt att använda

**Exempel:** "Vi har en e-handel med 100 000 användare/dag. Vilken strategi skulle du rekommendera?"

### 9. Troubleshooting

**Scenario:** Din workflow är långsam - tar 10 minuter att bygga och testa.

**Uppgift:** Lista minst 5 optimeringar du skulle försöka.

**Förklara:**

- Vad varje optimering gör
- Ungefär hur mycket tid den sparar
- Eventuella nackdelar

### 10. Security audit

**Uppgift:** Granska denna workflow och hitta säkerhetsproblem:

```yaml
name: Deploy

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@main
      - run: |
          echo "Deploying with password: ${{ secrets.PASSWORD }}"
          ssh root@${{ secrets.SERVER }} "
            cd /app
            git pull
            docker-compose up -d
          "
```

**Hitta och fixa:**

- Säkerhetsproblem
- Best practice violations
- Potentiella bugs

**Förklara varför varje ändring är viktig.**

---

**Lycka till! 🚀**

Nu har du all kunskap du behöver för att bygga robusta CI/CD-pipelines. Börja enkelt och bygg vidare över tid. Varje automatisering du lägger till sparar tid och minskar risken för fel. Happy deploying!
