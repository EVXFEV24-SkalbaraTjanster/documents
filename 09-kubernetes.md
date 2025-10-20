# Kubernetes - Container Orchestration i Praktiken

## Introduktion

Docker Compose är som att dirigera en liten orkester i ditt vardagsrum. Kubernetes är som att dirigera en symfoniorkester på en stor konsertsal - med hundratals musiker, flera dirigenter, och publik från hela världen.

Tänk dig att du har 100 containers - ska du verkligen starta, stoppa och övervaka alla manuellt? Och vad händer när en container kraschar klockan 03:00 på natten? Ska du vakna och starta om den? Absolut inte!

Det är här **Kubernetes** (ofta förkortat **K8s**) kommer in i bilden. Det är plattformen som gör det möjligt att köra containers i produktion på riktigt stort - med automatisk skalning, self-healing, och allt det där som gör att du faktiskt kan sova på nätterna.

### Vad är Kubernetes?

Kubernetes är ett **container orchestration system** - ett system som automatiskt hanterar, schemaläggerar och övervakar containers över flera servrar. Det utvecklades ursprungligen av Google baserat på deras interna system "Borg" som de använt i över 15 år för att hantera miljontals containers.

Idag är Kubernetes open source och har blivit industristandard för container orchestration. Det är det system som driver Netflix, Spotify, Airbnb, och tusentals andra företag som kör containers i produktion.

## Varför Kubernetes?

### Problemet

När din applikation växer stöter du på problem som Docker Compose inte kan lösa:

- **Flera servrar**: Compose körs på en enda maskin. Vad händer när du behöver 10, 20, eller 100 servrar?
- **Ingen auto-scaling**: Du måste manuellt ändra antal containers
- **Ingen self-healing**: Om en container kraschar måste du starta om den manuellt
- **Manual deployment**: Du måste SSH:a in på varje server och uppdatera
- **Ingen load balancing**: Du måste själv sätta upp nginx eller liknande
- **Inget automatiskt failover**: Om en server dör, vad händer med containers som körde där?

### Lösningen - Vad Kubernetes Ger Dig

Kubernetes löser alla dessa problem och mer därtill:

**1. Multi-host deployment**

- Kör containers över många servrar automatiskt
- Du bryr dig inte om vilken server - K8s bestämmer

**2. Auto-scaling**

- Lägg till fler containers när lasten ökar
- Ta bort containers när lasten minskar
- Spara pengar automatiskt

**3. Self-healing**

- Container kraschar? K8s startar en ny automatiskt
- Server dör? K8s flyttar containers till andra servrar
- Health checks övervakar allt

**4. Rolling updates**

- Uppdatera din app utan downtime
- Gradvis utrullning med automatisk rollback vid problem

**5. Built-in load balancing**

- Fördela trafik mellan containers automatiskt
- Ingen extra nginx behövs

**6. Service discovery**

- Services hittar varandra automatiskt
- Inget hårdkodat IPs eller DNS

**7. Secret management**

- Hantera känsliga data säkert
- API-nycklar, lösenord, certifikat

**8. Resource management**

- Fördela CPU och minne effektivt
- Förhindra att en app tar alla resurser

### Verkligt Scenario

Säg att du har en e-handelsplattform med 50 microservices:

- Varje service behöver 3-10 instanser
- Det blir 200+ containers
- Som körs på 20 servrar
- Och ska vara tillgängliga 24/7

**Utan Kubernetes:**

- Manuell deployment på varje server
- Manuell övervakning av 200+ containers
- Manuell skalning vid trafikökningar
- Panik när något går sönder klockan 03:00
- = Omöjligt att hantera

**Med Kubernetes:**

- Deploy med ett kommando
- Automatisk övervakning och restart
- Automatisk skalning baserat på last
- Self-healing - du sover gott
- = Hanterbart och skalbart

## Docker Compose vs Kubernetes

Låt oss göra en tydlig jämförelse:

### Docker Compose

**Användningsområde:**

- Utveckling och testmiljöer
- Små applikationer
- Single-host (en server)
- Personliga projekt
- Prototyper

**Fördelar:**

- Super enkelt att komma igång
- Läsbar YAML-fil
- Perfekt för utveckling
- Ingen komplex infrastruktur

**Begränsningar:**

- Endast en server
- Ingen auto-scaling
- Ingen self-healing
- Ingen high availability
- Manual updates

**Exempel:**

```yaml
version: "3"
services:
  app:
    image: myapp:1.0
    ports:
      - "8080:8080"
  db:
    image: postgres:15
```

Simple! Men vad händer om servern går ner? Hela din app är nere.

### Kubernetes

**Användningsområde:**

- Produktion
- Flera servrar (cluster)
- Stora applikationer
- Enterprise
- Microservices

**Fördelar:**

- Multi-host clustering
- Automatisk skalning
- Self-healing
- High availability
- Rolling updates
- Load balancing built-in
- Används av alla stora tech-företag

**Utmaningar:**

- Större learning curve
- Mer komplex setup
- Behöver mer infrastruktur
- Overkill för små projekt

**När ska du välja vad?**

```
Projekt storlek / komplexitet
    |
    |                    K8s-zonen
    |              ___________________
    |             |                   |
    |             |  Microservices    |
    |             |  Produktion       |
    |             |  Auto-scaling     |
    |   Compose   |___________________| 
    |   zonen     
    |_________
    |         |
    |  Dev    |
    |  Test   |
    |  Små    |
    |_________|
    |
    +---------------------------------> Tid/Resurser
```

**Tumregel:** Börja med Docker Compose. När du behöver flera servrar, auto-scaling, eller har ett större team - då är det dags för Kubernetes.

## Kubernetes Arkitektur (Översikt)

Kubernetes körs som ett **cluster** - en samling maskiner som jobbar tillsammans. Vissa maskiner är "hjärnan" (control plane) och andra är "arbetare" (worker nodes).

```
                KUBERNETES CLUSTER

┌────────────────────────────────────────────────┐
│           CONTROL PLANE (Master)               │
│  ┌──────────────────────────────────────────┐  │
│  │  API Server  │  Scheduler  │  Controller │  │
│  │              │             │   Manager   │  │
│  └──────────────────────────────────────────┘  │
│              etcd (cluster data)               │
└─────────────────┬──────────────────────────────┘
                  │
     ┌────────────┼────────────┐
     │            │            │
┌────▼───┐   ┌────▼───┐   ┌────▼───┐
│ NODE 1 │   │ NODE 2 │   │ NODE 3 │
│        │   │        │   │        │
│ Pod    │   │ Pod    │   │ Pod    │
│ Pod    │   │ Pod    │   │ Pod    │
│ Pod    │   │        │   │ Pod    │
└────────┘   └────────┘   └────────┘
 Worker       Worker       Worker
```

### Control Plane (Hjärnan)

Control plane är "management layer" som bestämmer vad som ska hända i clustret:

**API Server**

- Entry point för allt
- När du kör `kubectl`-kommandon pratar du med API Server
- Alla komponenter kommunicerar genom den

**Scheduler**

- Bestämmer vilken worker node som ska köra nya pods
- Kollar: vilken node har plats? Vilka resurser behövs?
- Smart placering för optimal resursanvändning

**Controller Manager**

- Säkerställer att önskat tillstånd upprätthålls
- "Vi vill ha 3 replicas" - Controller ser till att det alltid är 3
- Om en pod dör, startar Controller en ny

**etcd**

- Distribuerad databas som lagrar hela cluster state
- Alla konfigurationer, secrets, tillstånd
- Kritisk komponent - utan etcd, inget cluster

### Worker Nodes (Arbetarna)

Worker nodes är de maskiner som faktiskt kör dina containers:

**Kubelet**

- Agent som körs på varje node
- Tar emot instruktioner från control plane
- Startar och övervakar containers
- Rapporterar status tillbaka

**Container Runtime**

- Den faktiska motor som kör containers
- Oftast containerd eller Docker
- Kubelet pratar med runtime för att starta containers

**Kube-proxy**

- Hanterar networking på noden
- Routing av trafik till rätt pods
- Implementerar Services (mer om det senare)

### Hur det fungerar tillsammans

1. Du skickar en request via `kubectl`: "Kör 3 instanser av min app"
2. API Server tar emot requesten
3. Scheduler bestämmer vilka nodes som ska köra pods
4. Kubelet på varje node startar containers
5. Controller Manager övervakar och säkerställer 3 pods alltid körs
6. etcd sparar allt detta

## Grundläggande Koncept

Nu när vi förstår arkitekturen, låt oss gå igenom de viktigaste koncepten i Kubernetes.

### Pod - Den Minsta Enheten

En **Pod** är den minsta deployable enheten i Kubernetes. Den är som ett "hus" för en eller flera containers som behöver köras tillsammans.

**Viktiga punkter:**

- Vanligtvis en container per pod
- Ibland flera containers som är tight coupled (sidecar pattern)
- Pods är **ephemeral** - de kan dö och skapas när som helst
- Varje pod får sin egen IP-adress
- Containers i samma pod delar nätverk och storage

**Analogi:**
Tänk på en Pod som ett hus. Vanligtvis bor en person (container) i huset, men ibland kan det bo flera som behöver vara nära varandra. Om huset rivs, byggs ett nytt - men det är inte samma hus.

```
┌─────────────────┐
│      POD        │
│  ┌───────────┐  │
│  │ Container │  │  <- Vanligast: en container
│  └───────────┘  │
│  IP: 10.0.1.5   │
└─────────────────┘

┌─────────────────┐
│      POD        │
│  ┌───────────┐  │
│  │   App     │  │  <- Ibland: flera containers
│  └───────────┘  │
│  ┌───────────┐  │
│  │  Sidecar  │  │  <- Ex: logging agent
│  └───────────┘  │
│  IP: 10.0.1.6   │
└─────────────────┘
```

**Exempel pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp
      image: myapp:1.0
      ports:
        - containerPort: 8080
```

Men du kommer sällan att skapa pods direkt. Istället använder du Deployments...

### Node - Arbetsmaskinen

En **Node** är en fysisk eller virtuell maskin i ditt cluster. Det är där pods faktiskt körs.

**Typer:**

- Master nodes (kör control plane)
- Worker nodes (kör dina pods)

**Du hanterar vanligtvis inte nodes direkt** - Kubernetes sköter placeringen. Du säger "kör 5 pods" och K8s bestämmer vilka nodes de ska köra på.

### Cluster - Hela Miljön

Ett **Cluster** är hela din Kubernetes-miljö:

- Control plane
- Alla worker nodes
- Allt som körs på dem

Ett cluster kan ha 3 nodes eller 1000 nodes - det är samma koncept.

## Deployment - Så Hanterar Du Pods

Okej, så pods är viktiga - men hur skapar och hanterar man dem? Svaret är **Deployment**.

### Vad är en Deployment?

En Deployment är ett **deklarativt sätt** att hantera pods. Du beskriver vad du vill ha, och Kubernetes gör det verkligt.

**Exempel:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3 # Jag vill ha 3 pods
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:1.0
          ports:
            - containerPort: 8080
```

Med detta säger du:

- "Jag vill ha 3 identiska pods"
- "De ska köra imagen myapp:1.0"
- "De ska lyssna på port 8080"

Kubernetes tar det härifrån och:

1. Skapar 3 pods
2. Fördelar dem över worker nodes
3. Övervakar dem kontinuerligt
4. Om en pod dör, skapar K8s en ny automatiskt

**Detta är self-healing!**

### Self-Healing i Aktion

```
Scenario:
Önskat tillstånd: 3 pods
Faktiskt tillstånd: 3 pods ✓

Node 2 kraschar!
Faktiskt tillstånd: 1 pod (2 pods dog) ✗

Kubernetes upptäcker:
"Hallå, jag ska ha 3 pods men har bara 1!"

Kubernetes agerar:
- Skapar 2 nya pods på andra nodes
- Faktiskt tillstånd: 3 pods ✓

Allt fixat - automatiskt!
```

### ReplicaSet - Bakom Kulisserna

När du skapar en Deployment skapar Kubernetes automatiskt en **ReplicaSet**. ReplicaSet är den som faktiskt säkerställer att rätt antal pods körs.

```
Deployment → ReplicaSet → Pods (3 st)
```

Du behöver sällan interagera med ReplicaSets direkt - Deployment hanterar det åt dig.

**Varför denna hierarki?**

- Deployment = hanterar updates och rollbacks
- ReplicaSet = hanterar antalet replicas
- Pods = kör faktiska containers

## Service - Stabil Nätverksåtkomst

Nu har vi 3 pods som kör vår app. Men det finns ett problem: **pods är ephemeral**.

### Problemet

- Pod 1 har IP 10.0.1.5
- Pod 1 kraschar
- Ny pod skapas med IP 10.0.1.9
- Alla som pratar med 10.0.1.5 kommer nu inte hitta appen!

Dessutom, vi har 3 pods - vilken ska man prata med?

### Lösningen - Service

En **Service** ger en stabil IP-adress och DNS-namn för att komma åt dina pods. Den fungerar också som en inbyggd load balancer.

```
          ┌─────────────┐
          │   SERVICE   │
          │ myapp-svc   │
          │ 10.100.0.5  │
          └──────┬──────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼──┐     ┌───▼──┐     ┌───▼──┐
│ Pod1 │     │ Pod2 │     │ Pod3 │
│10.0.1│     │10.0.2│     │10.0.3│
└──────┘     └──────┘     └──────┘
```

**Med en Service:**

- Alla pratar med servicens IP (10.100.0.5) eller DNS namn (myapp-svc)
- Service load balancerar automatiskt till alla pods
- Pods kan dö och skapas - service IP ändras aldrig
- Built-in service discovery!

### Service Types

**ClusterIP (default)**

- Åtkomst endast inom clustret
- Perfekt för interna services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp # Hittar alla pods med label app=myapp
  ports:
    - port: 80
      targetPort: 8080 # Pods lyssnar på 8080
```

**NodePort**

- Exponerar service på varje nodes IP på en specifik port
- Kan nås utifrån clustret
- Port range: 30000-32767

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080 # Åtkomst via <any-node-ip>:30080
```

**LoadBalancer**

- Skapar en cloud load balancer (AWS ELB, GCP LB, etc.)
- Extern IP-adress
- Används för att exponera apps till internet

```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
```

**ExternalName**

- DNS alias till extern service
- Används sällan

### Hur Service Hittar Pods

Services använder **labels** för att hitta pods:

```yaml
# Deployment skapar pods med label app=myapp
template:
  metadata:
    labels:
      app: myapp

# Service hittar dessa pods via selector
selector:
  app: myapp
```

Service skickar trafik till alla pods som matchar labeln. När nya pods skapas eller gamla tas bort, uppdateras Service automatiskt!

## Namespace - Virtuella Cluster

Ett **Namespace** är ett sätt att dela upp ditt cluster i virtuella sub-clusters. Det är som rum i ett stort hus.

### Varför Namespaces?

- **Organisera resurser**: Dev, staging, production i samma cluster
- **Isolera team**: Team A och Team B får egna namespaces
- **Resource quotas**: Begränsa hur mycket CPU/minne varje namespace kan använda
- **Access control**: Olika användare får access till olika namespaces

```
KUBERNETES CLUSTER
├── namespace: development
│   ├── myapp-deployment
│   ├── myapp-service
│   └── database
├── namespace: staging
│   ├── myapp-deployment
│   ├── myapp-service
│   └── database
└── namespace: production
    ├── myapp-deployment
    ├── myapp-service
    └── database
```

**Default namespaces:**

- `default` - om du inte anger namespace
- `kube-system` - Kubernetes system components
- `kube-public` - publikt tillgänglig data
- `kube-node-lease` - node heartbeats

**Skapa namespace:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

**Använda namespace:**

```bash
kubectl get pods -n development
kubectl apply -f deployment.yaml -n production
```

## ConfigMap och Secret

Din applikation behöver konfiguration. Men du vill inte baka in den i Docker imagen - då måste du bygga om imagen för varje miljö!

### ConfigMap - Icke-Känslig Konfiguration

**ConfigMap** lagrar konfigurationsdata som kan injiceras i pods.

**Exempel:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "postgres://db:5432/mydb"
  API_ENDPOINT: "https://api.example.com"
  FEATURE_FLAG_X: "true"
  log_level: "info"
```

**Använda i pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: myapp
      image: myapp:1.0
      envFrom:
        - configMapRef:
            name: app-config # Alla keys blir env variables
```

Nu har containern miljövariabler:

- `DATABASE_URL=postgres://db:5432/mydb`
- `API_ENDPOINT=https://api.example.com`
- osv.

**Fördelar:**

- Samma imagen i dev, staging, production
- Ändra config utan att rebuilda
- Versionshantera config separat

### Secret - Känslig Data

**Secret** är som ConfigMap men för känslig data: lösenord, API-nycklar, certifikat.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4= # base64 encoded
  password: c3VwZXJzZWNyZXQ= # base64 encoded
```

**Använda i pod:**

```yaml
spec:
  containers:
    - name: myapp
      image: myapp:1.0
      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

**OBS:** Secrets är base64-encodade, **inte krypterade** by default. För riktig säkerhet behöver du:

- Encryption at rest (konfigurera etcd encryption)
- RBAC för att begränsa access
- Eller externa secret managers (HashiCorp Vault, AWS Secrets Manager)

## Persistent Volumes - Lagring som Överlever

Containers är ephemeral - när de dör, försvinner all data. Men vad händer med din databas?

### Problemet

```
Pod med PostgreSQL:
- Sparar data i /var/lib/postgresql/data
- Pod kraschar
- Ny pod startas
- All data är borta! 😱
```

### Lösningen - Persistent Volumes

**PersistentVolume (PV)**

- Ett lagringsutrymme i clustret
- Kan vara: cloud disk (AWS EBS, GCP Persistent Disk), NFS, local disk, etc.
- Överlever pods

**PersistentVolumeClaim (PVC)**

- En "förfrågan" om lagring
- "Jag behöver 10GB disk"
- Kubernetes binder PVC till en PV

```
┌──────────────────┐
│       POD        │
│   PostgreSQL     │
│       │          │
│       ▼          │
│  ┌──────────┐    │
│  │   PVC    │    │  <- Pod använder PVC
│  │  10GB    │    │
│  └────┬─────┘    │
└───────┼──────────┘
        │
        ▼
   ┌─────────┐
   │   PV    │         <- PVC bunden till PV
   │  20GB   │
   │ AWS EBS │
   └─────────┘
```

**Exempel PVC:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce # Endast en node kan skriva
  resources:
    requests:
      storage: 10Gi # Behöver 10GB
```

**Använda i Deployment:**

```yaml
spec:
  containers:
    - name: postgres
      image: postgres:15
      volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumes:
    - name: postgres-storage
      persistentVolumeClaim:
        claimName: postgres-pvc
```

Nu överlever databasens data även om poden dör!

## Labels och Selectors

**Labels** är key-value pairs som du kan sätta på vilka Kubernetes-objekt som helst. De är fundamentala för hur K8s organiserar och hittar saker.

### Labels

```yaml
metadata:
  name: myapp-pod
  labels:
    app: myapp
    environment: production
    version: "1.0"
    team: backend
```

**Varför labels?**

- Organisera resurser
- Gruppera relaterade objekt
- Möjliggör selectors

### Selectors

**Selectors** används för att hitta objekt baserat på labels:

```yaml
# Service hittar alla pods med app=myapp
selector:
  app: myapp

# Deployment skapar pods med dessa labels
template:
  metadata:
    labels:
      app: myapp
```

**Exempel:**

```
Pods:
┌──────────────────┐
│ Pod: frontend-1  │
│ Labels:          │
│  app=frontend    │
│  tier=frontend   │
└──────────────────┘

┌──────────────────┐
│ Pod: frontend-2  │
│ Labels:          │
│  app=frontend    │
│  tier=frontend   │
└──────────────────┘

┌──────────────────┐
│ Pod: backend-1   │
│ Labels:          │
│  app=backend     │
│  tier=backend    │
└──────────────────┘

Service med selector app=frontend → hittar frontend-1 och frontend-2
Service med selector app=backend  → hittar backend-1
```

Labels är kraftfulla! Du kan välja vilka pods som helst:

- `app=myapp` - alla myapp pods
- `environment=production` - alla production pods
- `app=myapp,environment=production` - myapp pods i production

## Hur Kubernetes Upprätthåller Önskat Tillstånd

Kubernetes jobbar enligt **deklarativt paradigm**:

- Du deklarerar vad du vill ha
- Kubernetes figur out hur det ska göras
- Kubernetes övervakar kontinuerligt och korrigerar avvikelser

### Deklarativt vs Imperativt

**Imperativt (hur):**

```bash
docker run myapp:1.0
docker run myapp:1.0
docker run myapp:1.0
# Du säger exakt VAD som ska göras, steg för steg
```

**Deklarativt (vad):**

```yaml
replicas: 3
image: myapp:1.0
# Du säger VAD du vill ha, K8s figurer ut hur
```

### Control Loop

Kubernetes kör en kontinuerlig loop:

```
1. Läs önskat tillstånd (desired state)
   "replicas: 3"

2. Observera faktiskt tillstånd (actual state)
   "2 pods körs"

3. Jämför
   Desired: 3
   Actual: 2
   Diff: -1

4. Agera
   Skapa 1 ny pod

5. Repetera (konstant)
```

### Exempel Scenario: Self-Healing

```
Klockan 09:00:
✓ Önskat: 3 pods
✓ Faktiskt: 3 pods
✓ Status: OK

Klockan 14:32:
✓ Önskat: 3 pods
✗ Faktiskt: 2 pods (en krashade)
⚠ Controller upptäcker avvikelse
→ Skapar ny pod

Klockan 14:32:30:
✓ Önskat: 3 pods
✓ Faktiskt: 3 pods
✓ Status: OK igen
```

**Detta händer automatiskt, 24/7. Detta är self-healing!**

Du sover, men Kubernetes vaktar. En server går ner kl 03:00? Kubernetes flyttar pods till andra servrar. En pod kraschar? Kubernetes startar en ny. Allt utan att du lyfter ett finger.

## Scaling i Kubernetes

Skalning är en av Kubernetes största styrkor.

### Manual Scaling

**Ändra antal replicas:**

```bash
kubectl scale deployment myapp --replicas=5
```

Eller uppdatera YAML:

```yaml
spec:
  replicas: 5 # Var 3, nu 5
```

Kubernetes ser förändringen:

- "Jag har 3 pods men ska ha 5"
- Skapar 2 nya pods
- Fördelar dem över nodes
- Klart!

### Auto-Scaling (HPA - Horizontal Pod Autoscaler)

Varför ska du manuellt skala när Kubernetes kan göra det automatiskt?

**HPA** övervakar metrics och skalar automatiskt:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2 # Minst 2 pods
  maxReplicas: 10 # Max 10 pods
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70 # Skala om CPU > 70%
```

**Hur det fungerar:**

```
Normal trafik:
CPU usage: 40%
Pods: 2 (minReplicas)

Trafikspike! Black Friday!
CPU usage: 85%
HPA: "Över 70%! Skala upp!"
Pods: 2 → 4 → 6

CPU usage sjunker till 60%
Pods: 6 (behålls tills det sjunker mer)

Trafik normaliseras
CPU usage: 30%
HPA: "Under 70%, skala ner"
Pods: 6 → 4 → 2

Tillbaka till normal!
```

**Fördelar:**

- Hanterar trafikspikes automatiskt
- Sparar pengar (färre pods när det är lugnt)
- Du behöver inte övervaka konstant

### Cluster Auto-Scaling

Om alla nodes är fulla, vad då? **Cluster Autoscaler** kan lägga till fler nodes automatiskt (i cloud).

```
Scenario:
- 3 nodes, alla fulla
- HPA vill starta fler pods
- Men det finns inget utrymme!

Cluster Autoscaler:
- Upptäcker att pods är "pending" (väntar på plats)
- Begär ny node från cloud provider
- AWS/GCP startar ny VM
- Ny node joins cluster
- Pods schemaläggs på nya noden
- Problem löst!
```

Detta är **true auto-scaling** - både pods OCH infrastruktur skalar automatiskt.

## Rolling Updates - Zero Downtime Deployments

En av Kubernetes coolaste features: uppdatera din app utan downtime.

### Problemet med Traditionella Updates

```
Traditionellt:
1. Stoppa gammal version
2. Deploy ny version
3. Starta ny version
   
   ⏸ DOWNTIME under step 2-3!
```

### Kubernetes Rolling Update

```yaml
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1 # Max 1 extra pod under update
      maxUnavailable: 1 # Max 1 pod kan vara nere
```

**Process:**

```
Initial: v1.0
[v1] [v1] [v1] [v1]

Step 1: Starta en v2 pod
[v1] [v1] [v1] [v1] [v2]  <- maxSurge: 1

Step 2: Vänta tills v2 är ready
[v1] [v1] [v1] [v1] [v2✓]

Step 3: Ta bort en v1 pod
[v1] [v1] [v1] [v2✓]

Step 4: Starta ytterligare en v2
[v1] [v1] [v1] [v2✓] [v2]

Step 5: v2 ready, ta bort v1
[v1] [v1] [v2✓] [v2✓]

Step 6-8: Fortsätt...
[v1] [v2✓] [v2✓] [v2]
[v2✓] [v2✓] [v2✓] [v2]

Slutresultat:
[v2✓] [v2✓] [v2✓] [v2✓]
```

**Under hela processen:**

- Minst 3 pods är alltid running (maxUnavailable: 1)
- Users upplever ingen downtime
- Service load balancerar mellan v1 och v2 pods

### Rollback

Vad händer om v2 har en bugg?

```bash
kubectl rollout undo deployment myapp-deployment
```

Kubernetes rullar tillbaka till föregående version - samma rolling update process, men baklänges!

```
[v2] [v2] [v2] [v2]
→ Rolling back...
[v2] [v2] [v2] [v1]
[v2] [v2] [v1] [v1]
[v2] [v1] [v1] [v1]
[v1] [v1] [v1] [v1]
Rollback komplett!
```

## Health Checks - Så Kubernetes Vet Om Du Är OK

Hur vet Kubernetes om en container är healthy? Health checks!

### Liveness Probe - Är du vid liv?

**Liveness probe** kollar om containern är vid liv. Om den failar, startar Kubernetes om containern.

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30 # Vänta 30s efter start
  periodSeconds: 10 # Kolla var 10:e sekund
  timeoutSeconds: 5 # Timeout efter 5s
  failureThreshold: 3 # 3 misslyckanden → restart
```

**Exempel scenario:**

```
App startar → OK
Health check: GET /health → 200 OK ✓
Health check: GET /health → 200 OK ✓
Health check: GET /health → 200 OK ✓

App får deadlock, hänger sig
Health check: GET /health → Timeout ✗ (1)
Health check: GET /health → Timeout ✗ (2)
Health check: GET /health → Timeout ✗ (3)

failureThreshold nått!
Kubernetes: "Container är död, restartar!"
Container restartas
App startar → OK igen
```

### Readiness Probe - Är du redo för trafik?

**Readiness probe** kollar om containern är redo att ta emot requests. Om den failar, tar Kubernetes bort poden från Service (ingen trafik skickas).

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Varför är detta annorlunda än liveness?**

```
Pod startar:
- Liveness: OK (process körs)
- Readiness: FAIL (databas connection inte etablerad än)
→ Pod är alive men får INGEN trafik

5 sekunder senare:
- Liveness: OK
- Readiness: OK (databas connection OK nu)
→ Pod får nu trafik från Service
```

**Användningsfall:**

- Vänta på databas connection
- Ladda cache vid uppstart
- Vänta på dependencies

### Startup Probe - För Långsamma Uppstarter

Vissa apps tar lång tid att starta (JVM anyone?). **Startup probe** ger mer tid för uppstart.

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30 # 30 försök
  periodSeconds: 10 # Var 10:e sekund
  # = Max 300 sekunder (5 min) för uppstart
```

Efter startup probe lyckas, tar liveness probe över.

## Load Balancing - Inbyggt i Kubernetes

Med Docker Compose måste du sätta upp nginx eller liknande för load balancing. I Kubernetes? Inbyggt!

### Service = Load Balancer

När du skapar en Service får du automatiskt load balancing:

```
Request → Service
              ↓
    ┌─────────┼─────────┐
    ↓         ↓         ↓
  Pod 1     Pod 2     Pod 3
```

**Load balancing algorithms:**

- Round-robin (default) - turnerar mellan pods
- Session affinity - samma client → samma pod (optional)

**Exempel:**

```
Request 1 → Pod 1
Request 2 → Pod 2
Request 3 → Pod 3
Request 4 → Pod 1
Request 5 → Pod 2
...
```

### Kube-proxy

**Kube-proxy** körs på varje node och implementerar load balancing:

- Lyssnar på Service-ändringar
- Uppdaterar iptables/ipvs rules
- Dirigerar trafik till rätt pods

Du behöver inte tänka på det - det funkar bara!

## Ingress - HTTP Routing

Service ger dig load balancing, men vad om du vill:

- Routea baserat på URL path?
- SSL/TLS termination?
- En entry point för många services?

Det är här **Ingress** kommer in!

### Vad är Ingress?

Ingress är som en API Gateway eller reverse proxy för ditt cluster.

```
               INTERNET
                  ↓
           ┌──────────────┐
           │   INGRESS    │  <- En extern IP
           │  Controller  │
           └──────┬───────┘
                  │
     ┌────────────┼────────────┐
     ↓            ↓            ↓
[Service]    [Service]    [Service]
frontend     user-api     product-api
```

### Path-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
          - path: /api/users
            pathType: Prefix
            backend:
              service:
                name: user-service
                port:
                  number: 8080
          - path: /api/products
            pathType: Prefix
            backend:
              service:
                name: product-service
                port:
                  number: 8080
```

**Routing:**

```
myapp.com/              → frontend-service
myapp.com/api/users     → user-service
myapp.com/api/products  → product-service
```

En IP-adress, flera services - snyggt!

### SSL/TLS Termination

```yaml
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: tls-secret    # Certificate i Secret
  rules:
  - host: myapp.com
    ...
```

HTTPS termineras vid Ingress, intern trafik kan vara HTTP.

### Ingress Controller

För att Ingress ska fungera behöver du en **Ingress Controller**:

- nginx-ingress (populärast)
- Traefik
- HAProxy
- Cloud-specific (AWS ALB, GCP LB)

Ingress Controller är den faktiska load balancern som implementerar Ingress rules.

## Resource Management - CPU och Minne

Hur förhindrar du att en app tar alla resurser?

### Requests och Limits

**Requests** = minimum som garanteras
**Limits** = maximum som tillåts

```yaml
resources:
  requests:
    cpu: 100m # 100 millicores = 0.1 CPU
    memory: 128Mi # 128 megabytes
  limits:
    cpu: 500m # Max 0.5 CPU
    memory: 512Mi # Max 512 megabytes
```

### Hur Kubernetes Använder Detta

**Scheduler:**

```
Pod behöver: requests.cpu = 100m, requests.memory = 128Mi

Node 1: 
  Kapacitet: 2 CPU, 4Gi memory
  Använt: 1.9 CPU, 3.5Gi memory
  Ledigt: 0.1 CPU, 0.5Gi memory
  → NEJ, inte nog med CPU

Node 2:
  Kapacitet: 4 CPU, 8Gi memory
  Använt: 1 CPU, 2Gi memory
  Ledigt: 3 CPU, 6Gi memory
  → JA! Schemalägg här
```

**Runtime:**

- Container använder upp till `limits.cpu` och `limits.memory`
- Om den försöker använda mer minne → OOMKilled (killed)
- Om den försöker använda mer CPU → throttled (bromsas)

### Best Practices

```yaml
# Development pod - inga hårda limits
resources:
  requests:
    cpu: 100m
    memory: 128Mi

# Production pod - requests = limits för förutsägbarhet
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

## Kubernetes i Praktiken - Workflow

Hur jobbar man egentligen med Kubernetes dagligen?

### Development Workflow

```
1. Utveckla kod lokalt
   ↓
2. Bygg Docker image
   docker build -t myapp:1.1 .
   ↓
3. Pusha till registry
   docker push myregistry.com/myapp:1.1
   ↓
4. Uppdatera Kubernetes Deployment
   # Ändra image: myapp:1.0 → myapp:1.1 i YAML
   ↓
5. Applya ändring
   kubectl apply -f deployment.yaml
   ↓
6. Kubernetes gör rolling update
   - Startar pods med nya imagen
   - Tar bort gamla pods gradvis
   ↓
7. Verifiera
   kubectl get pods
   kubectl logs <pod-name>
```

### Infrastructure as Code

Alla Kubernetes-resurser definieras i YAML-filer:

```
my-app/
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── configmap.yaml
└── secret.yaml
```

**Fördelar:**

- Versionshantera i Git
- Code review av infrastruktur
- Reproducerbar miljö
- Disaster recovery - replaya alla YAML-filer

**Deploy allt:**

```bash
kubectl apply -f my-app/
```

### GitOps Workflow

Modern approach:

1. Commit YAML till Git
2. CI/CD pipeline detekterar ändring
3. Automatisk deploy till K8s
4. Git = single source of truth

```
Git commit → CI/CD → kubectl apply → Kubernetes
```

## kubectl - Kommandoradsverktyget

**kubectl** är ditt verktyg för att interagera med Kubernetes.

### Vanliga Kommandon

**Lista resurser:**

```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes
kubectl get all    # Allt i current namespace
```

**Detaljerad info:**

```bash
kubectl describe pod myapp-pod-abc123
kubectl describe deployment myapp-deployment
```

**Logs:**

```bash
kubectl logs myapp-pod-abc123
kubectl logs -f myapp-pod-abc123    # Follow (live)
kubectl logs myapp-pod-abc123 -c container-name    # Specific container
```

**Exec (shell into pod):**

```bash
kubectl exec -it myapp-pod-abc123 -- /bin/bash
kubectl exec -it myapp-pod-abc123 -- sh
```

**Apply/Create/Delete:**

```bash
kubectl apply -f deployment.yaml      # Create eller update
kubectl create -f deployment.yaml     # Endast create
kubectl delete -f deployment.yaml     # Delete
```

**Scaling:**

```bash
kubectl scale deployment myapp --replicas=5
```

**Namespace:**

```bash
kubectl get pods -n production              # Specific namespace
kubectl get pods --all-namespaces          # Alla namespaces
kubectl config set-context --current --namespace=production  # Sätt default
```

**Debugging:**

```bash
kubectl get events                          # Cluster events
kubectl top pods                            # Resource usage
kubectl port-forward myapp-pod-abc123 8080:8080    # Local access
```

### Kubectl Tips

```bash
# Alias för snabbhet
alias k=kubectl
k get pods

# Watch mode
kubectl get pods --watch

# Output formats
kubectl get pods -o wide              # Mer info
kubectl get pods -o yaml              # YAML format
kubectl get pods -o json              # JSON format

# Imperative commands (snabbt test)
kubectl run nginx --image=nginx       # Snabb pod
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
```

## Managed Kubernetes Services

Att köra eget Kubernetes cluster är komplext - control plane, etcd, networking, upgraderingar...

Därför använder de flesta **managed Kubernetes** från cloud providers.

### Google Kubernetes Engine (GKE)

**Fördelar:**

- Skapades av Google (som uppfann K8s)
- Enklast att komma igång
- Automatiska upgrades
- God integration med Google Cloud

```bash
gcloud container clusters create my-cluster --num-nodes=3
```

### Amazon Elastic Kubernetes Service (EKS)

**Fördelar:**

- AWS's K8s-tjänst
- Integration med AWS services (RDS, S3, IAM, etc.)
- Stor community

```bash
eksctl create cluster --name my-cluster --nodes=3
```

### Azure Kubernetes Service (AKS)

**Fördelar:**

- Microsoft Azure's K8s
- Bra för företag med Microsoft-stack
- Integration med Azure services

```bash
az aks create --resource-group myRG --name myCluster --node-count 3
```

### Fördelar med Managed K8s

- **Control plane hanteras för dig** - du slipper sköta master nodes
- **Automatiska updates** - K8s uppdateras säkert
- **Enklare setup** - klart på minuter istället för dagar
- **Bättre säkerhet** - patching, RBAC, network policies
- **Du betalar bara för worker nodes** - control plane ofta gratis

**Men:**

- Vendor lock-in (lite)
- Kostar mer än self-hosted (men du sparar tid)
- Mindre kontroll över control plane

## Local Kubernetes för Utveckling

Du vill testa K8s lokalt innan du kör i produktion? Det finns flera alternativ.

### Minikube

**Minikube** = Single-node K8s cluster på din laptop.

```bash
# Installera
brew install minikube    # macOS
choco install minikube   # Windows

# Starta
minikube start

# Använd
kubectl get nodes
```

**Fördelar:**

- Lätt att komma igång
- Perfekt för learning
- Addons för vanliga verktyg

### Kind (Kubernetes in Docker)

**Kind** = Kör K8s cluster i Docker containers.

```bash
# Installera
brew install kind

# Skapa cluster
kind create cluster

# Multi-node cluster
kind create cluster --config multi-node.yaml
```

**Fördelar:**

- Supersnabbt
- Lightweight
- Multi-node support
- Perfekt för CI/CD testing

### Docker Desktop

**Docker Desktop** inkluderar single-node Kubernetes.

- Enable Kubernetes i settings
- Klart!

**Fördelar:**

- Enklast - redan installerat med Docker Desktop
- Bra för basic testing

### K3s

**K3s** = Lightweight Kubernetes för edge, IoT, utveckling.

```bash
curl -sfL https://get.k3s.io | sh -
```

**Fördelar:**

- Mindre än 100MB binary
- Kör på Raspberry Pi
- Snabb startup

## När Behöver Du FAKTISKT Kubernetes?

Kubernetes är coolt, men är det rätt för dig?

### Du BEHÖVER Troligtvis K8s Om:

✅ **Du kör microservices i stor skala**

- 10+ services
- Varje service behöver skalas oberoende

✅ **Flera team deployer oberoende**

- Team A deployer sin service utan att vänta på Team B

✅ **High availability är kritiskt**

- 99.9%+ uptime requirement
- Kan inte ha downtime

✅ **Multi-environment på många servrar**

- Dev, staging, production
- Varje miljö har 10+ servrar

✅ **Auto-scaling är requirement**

- Trafiken varierar mycket
- Black Friday, rush hours, etc.

✅ **Du har DevOps-expertis**

- Team som kan hantera K8s komplexitet

### Du behöver INTE K8s Om:

❌ **Liten applikation eller team**

- Monolith eller 1-3 services
- 1-5 utvecklare

❌ **Docker Compose räcker**

- Kör på 1-2 servrar
- Simpel setup

❌ **Saknar K8s-expertis**

- Ingen har kört K8s innan
- Ingen tid att lära sig

❌ **Kan inte motivera complexity**

- Operational overhead
- Monitoring, logging setup
- Network complexity
- Learning curve

### Kostnad-Nytta Analys

**Kubernetes kostar:**

- Learning time (veckor/månader)
- Operational complexity
- Monitoring/logging infrastructure
- CI/CD pipelines
- Network complexity
- Felsökning svårare

**Kubernetes ger:**

- Auto-scaling
- Self-healing
- Zero-downtime deployments
- Service discovery
- Multi-cloud flexibility
- Industry standard

**Fråga dig själv:** Är fördelarna värda kostnaden?

### Tomrumregel

```
1-3 servers + monolith/few services
  → Docker Compose
  
4-10 servers + microservices
  → Kanske K8s, men tänk noga
  
10+ servers + många microservices
  → Definitivt K8s
```

**Viktig poäng:** Använd inte Kubernetes bara för att det är coolt eller "industry standard". Använd det när du faktiskt behöver det!

## Complexity Cost - Var Medveten

Kubernetes är powerful, men det kommer med pris.

### Complexity Factors

**Learning Curve**

- Nya koncept: Pods, Services, Deployments, etc.
- kubectl commands
- YAML configurations
- Networking concepts
- = Veckor/månader för teamet

**Operational Overhead**

- Cluster management
- Upgrades och patching
- Monitoring setup (Prometheus, Grafana)
- Logging setup (ELK/EFK stack)
- Backup och disaster recovery

**Network Complexity**

- CNI plugins (Calico, Flannel, etc.)
- Service mesh (Istio, Linkerd) om du behöver det
- Ingress controllers
- Network policies
- = Networking-expertis krävs

**Security**

- RBAC konfiguration
- Pod security policies
- Network policies
- Secret management
- Image scanning
- = Security-expertis krävs

**Debugging**

- Svårare än Docker Compose
- Fler lager att debugga
- Pods kan vara på olika nodes
- Logs över flera pods

### Är Det Värt Det?

**Om du är ett stort företag med microservices:**

- Ja! Fördelarna överväger complexity

**Om du är en startup med 3 utvecklare:**

- Troligen nej. Fokusera på produkt, använd enklare lösningar

**Om du är mellan:**

- Överväg managed K8s (GKE, EKS, AKS)
- De abstractar bort mycket complexity
- Du får fördelarna utan full operational overhead

## Kubernetes och Microservices

Kubernetes och microservices är som gjorda för varandra.

### Varför Passar De Perfekt?

**Microservices behöver:**

- Oberoende deployment ✓ K8s Deployments
- Individuell skalning ✓ K8s auto-scaling
- Service discovery ✓ K8s Services
- Load balancing ✓ K8s Services
- Health monitoring ✓ K8s probes
- Resilience ✓ K8s self-healing

**Kubernetes ger:**

- Allt ovan!

### Microservice Architecture på K8s

```
                     INGRESS
                        ↓
              ┌─────────┴─────────┐
              ↓                   ↓
        API GATEWAY          FRONTEND
              ↓                   
    ┌─────────┼─────────┐
    ↓         ↓         ↓
USER-SVC  PRODUCT-SVC  ORDER-SVC
    ↓         ↓         ↓
  REDIS   POSTGRES   POSTGRES
```

**Varje service:**

- Egen Deployment (skalas oberoende)
- Egen Service (intern communication)
- Egna ConfigMaps/Secrets
- Egen databas (database per service pattern)

**Detta är där Kubernetes shinar!**

## Monitoring och Logging i Kubernetes

I produktion måste du övervaka ditt cluster.

### Monitoring - Prometheus + Grafana

**Prometheus** = Metrics collection
**Grafana** = Visualization

```
Kubernetes Cluster
    ↓ (metrics)
Prometheus
    ↓ (query)
Grafana Dashboards
    ↓ (view)
DevOps Team
```

**Vad monitora:**

- **Cluster-level**: CPU, memory, disk per node
- **Pod-level**: CPU, memory per pod
- **Application metrics**: Request rate, error rate, latency
- **Custom metrics**: Business metrics

**Viktiga dashboards:**

- Cluster overview
- Resource usage per namespace
- Pod status och restarts
- Application performance

### Logging - EFK Stack

**EFK** = Elasticsearch + Fluentd + Kibana

```
Pods (loggar) 
    ↓
Fluentd (samlar logs från alla pods)
    ↓
Elasticsearch (lagrar och indexerar)
    ↓
Kibana (söker och visualiserar)
```

**Fördelar:**

- Centraliserad logging
- Sök över alla pods samtidigt
- Loggarna överlever även om pods dör

**Exempel queries:**

- "Visa alla ERROR logs senaste timmen"
- "Hitta logs från user-service i production namespace"
- "Vilka 500 errors har vi haft?"

### Varför Detta Är Viktigt

Utan monitoring och logging:

- Du vet inte vad som händer i ditt cluster
- Du kan inte debugga problem
- Du ser inte performance issues
- Du är blind!

Med monitoring och logging:

- Proaktiv övervakning
- Snabb felsökning
- Performance optimization
- Alerts när något är fel

**I produktion: MÅSTE HAVES!**

## Deployment Strategies

Kubernetes stödjer flera deployment-strategier.

### Rolling Update (Default)

Vi har redan pratat om detta - gradvis ersättning av pods.

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

**När:** Standard, fungerar för de flesta use cases.

### Recreate

Stoppa alla gamla pods, starta nya.

```yaml
strategy:
  type: Recreate
```

**Process:**

```
[v1] [v1] [v1]
    ↓
[ ] [ ] [ ]  ← DOWNTIME
    ↓
[v2] [v2] [v2]
```

**När:** När du inte kan ha både v1 och v2 samtidigt (databas-migrations, etc.).

### Blue-Green Deployment

Kör båda versionerna, switcha trafik när redo.

```
1. Deploy v2 (green) parallellt med v1 (blue)
   Blue: [v1] [v1] [v1]  ← Får trafik
   Green: [v2] [v2] [v2] ← Inget trafik

2. Testa green

3. Switch trafik från blue till green
   Blue: [v1] [v1] [v1]  ← Inget trafik
   Green: [v2] [v2] [v2] ← Får trafik

4. Om problem: switch tillbaka direkt!

5. Om OK: ta bort blue
```

**Implementering i K8s:**

- Två Deployments: myapp-blue, myapp-green
- Service selector bytts för att switcha trafik

**När:** Behöver snabb rollback, zero downtime kritiskt.

### Canary Deployment

Rulla ut till en liten % användare först.

```
1. 100% trafik till v1
   [v1] [v1] [v1] [v1] [v1]

2. Deploy en v2 pod (10% av pods)
   [v1] [v1] [v1] [v1] [v2]
   
3. Service load balancerar:
   90% requests → v1
   10% requests → v2

4. Monitora metrics från v2

5. Om OK: öka till 50%
   [v1] [v1] [v2] [v2]

6. Om fortfarande OK: 100%
   [v2] [v2] [v2] [v2]
```

**När:** High-risk deployments, behöver validera med verklig trafik.

## Konceptuellt Exempel - E-handel på Kubernetes

Låt oss sätta ihop allt vi lärt oss med ett verkligt exempel.

### Scenario

E-handelsplattform med microservices:

- Frontend (React SPA)
- API Gateway
- User Service
- Product Service
- Order Service
- Payment Service
- Notification Service
- PostgreSQL databaser
- Redis cache

### Kubernetes Resources

**1. Frontend Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: myregistry.com/frontend:1.2.0
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi
```

**2. User Service Deployment + Service**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: myregistry.com/user-service:2.1.0
          ports:
            - containerPort: 8080
          env:
            - name: DATABASE_URL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: user_db_url
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: user_db_password
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            requests:
              cpu: 200m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: user-service
  ports:
    - port: 80
      targetPort: 8080
```

**3. API Gateway (LoadBalancer)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  replicas: 3
  # ... template spec ...
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  type: LoadBalancer # Extern åtkomst
  selector:
    app: api-gateway
  ports:
    - port: 80
      targetPort: 8080
```

**4. Order Service + PostgreSQL**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  # ... template spec ...
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  type: ClusterIP
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-db
  template:
    metadata:
      labels:
        app: order-db
    spec:
      containers:
        - name: postgres
          image: postgres:15
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: order_db_password
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: order-db-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: order-db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: v1
kind: Service
metadata:
  name: order-db
spec:
  type: ClusterIP
  selector:
    app: order-db
  ports:
    - port: 5432
```

**5. Redis Cache**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7
          ports:
            - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  type: ClusterIP
  selector:
    app: redis
  ports:
    - port: 6379
```

**6. ConfigMap**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  user_db_url: "postgres://order-db:5432/userdb"
  product_db_url: "postgres://product-db:5432/productdb"
  order_db_url: "postgres://order-db:5432/orderdb"
  redis_url: "redis://redis:6379"
  payment_api_url: "https://api.stripe.com"
```

**7. Secrets**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
  namespace: production
type: Opaque
data:
  user_db_password: c3VwZXJzZWNyZXQxMjM=
  product_db_password: cGFzc3dvcmQxMjM=
  order_db_password: b3JkZXJwYXNzMTIz
  stripe_api_key: c2tfdGVzdF94eHh4eHg=
```

**8. HPA för Auto-Scaling**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**9. Ingress för Routing**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - myshop.com
      secretName: tls-secret
  rules:
    - host: myshop.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-gateway
                port:
                  number: 80
```

### Arkitekturdiagram

```
               INTERNET
                  │
                  ↓
             ┌────────┐
             │INGRESS │ (myshop.com)
             └───┬────┘
                 │
     ┌───────────┼───────────┐
     ↓                       ↓
┌─────────┐            ┌──────────┐
│FRONTEND │            │   API    │
│ (3 rep) │            │ GATEWAY  │
└─────────┘            │ (3 rep)  │
                       └────┬─────┘
                            │
       ┌────────────────────┼────────────────┐
       ↓                    ↓                ↓
  ┌────────┐          ┌─────────┐     ┌─────────┐
  │  USER  │          │PRODUCT  │     │ ORDER   │
  │SERVICE │          │SERVICE  │     │SERVICE  │
  │(3-20)  │←HPA      │(5 rep)  │     │(3 rep)  │
  └───┬────┘          └────┬────┘     └────┬────┘
      │                    │               │
      ↓                    ↓               ↓
 ┌────────┐          ┌─────────┐     ┌─────────┐
 │USER DB │          │PROD DB  │     │ORDER DB │
 │Postgres│          │Postgres │     │Postgres │
 └────────┘          └─────────┘     └────┬────┘
                                           │
                                     ┌─────▼────┐
                                     │   PVC    │
                                     │  20GB    │
                                     └──────────┘
      
      ┌───────────────────────────────┐
      │         REDIS CACHE           │
      │          (shared)             │
      └───────────────────────────────┘
```

### Hur Det Fungerar

**1. Request Flow:**

```
User → myshop.com 
     → Ingress 
     → Frontend Service 
     → Frontend Pod
     
User → myshop.com/api/users
     → Ingress
     → API Gateway Service
     → API Gateway Pod
     → User Service
     → User Service Pod
     → User DB
```

**2. Auto-Scaling:**

```
Normal dag:
- User Service: 3 pods
- CPU usage: 40%

Black Friday:
- Trafik spike!
- CPU usage: 85%
- HPA triggar
- User Service: 3 → 6 → 10 → 15 pods
- CPU usage: 65%

Efter Black Friday:
- Trafik normaliseras
- CPU usage: 30%
- HPA skalar ner
- User Service: 15 → 10 → 5 → 3 pods
```

**3. Self-Healing:**

```
Order Service Pod 2 kraschar:
- Deployment märker: "Jag ska ha 3 men har 2!"
- Startar ny pod
- Readiness probe väntar tills pod är ready
- Service börjar skicka trafik till nya poden
- Ingen downtime - andra 2 pods hanterade trafiken
```

**4. Rolling Update:**

```
Deploy User Service v2.2.0:
- kubectl apply -f user-service-deployment.yaml
- Rolling update startar:
  [v2.1] [v2.1] [v2.1] [v2.1] [v2.1]
       ↓
  [v2.1] [v2.1] [v2.1] [v2.1] [v2.2]
       ↓
  [v2.1] [v2.1] [v2.1] [v2.2] [v2.2]
       ↓
  ... gradvis ...
       ↓
  [v2.2] [v2.2] [v2.2] [v2.2] [v2.2]
- Zero downtime!
```

**Detta är Kubernetes i praktiken!**

## Nästa Steg för Lärande

Det här dokumentet har gett dig konceptuell förståelse för Kubernetes. För att lära dig på riktigt behöver du:

### 1. Hands-On Practice

**Installera lokalt:**

```bash
# Minikube
minikube start

# Eller Docker Desktop
# Enable Kubernetes i settings
```

**Grundläggande övningar:**

1. Deploy en enkel app (nginx)
2. Skapa en Service
3. Skala upp och ner
4. Gör en rolling update
5. Kolla logs
6. Exec into pods

### 2. Officiella Tutorials

**Kubernetes official docs:**

- [kubernetes.io/docs/tutorials](https://kubernetes.io/docs/tutorials/)
- Interactive browser-based tutorials
- Katacoda Kubernetes scenarios

### 3. Bygg Riktiga Projekt

**Börja smått:**

- Deploy en Spring Boot app
- Lägg till databas
- Lägg till Redis
- Sätt upp Ingress
- Implementera ConfigMaps/Secrets

**Bygg större:**

- Microservices-arkitektur
- Auto-scaling
- Monitoring med Prometheus/Grafana
- Logging med EFK

### 4. Lär Dig kubectl

Bli bekväm med kubectl - det är ditt viktigaste verktyg.

```bash
# Öva på:
kubectl get, describe, logs, exec
kubectl apply, delete
kubectl scale
kubectl rollout
```

### 5. Förstå Networking

Kubernetes networking är komplext:

- Pod networking
- Services och kube-proxy
- Ingress
- Network policies
- DNS

Ta tid att förstå hur det fungerar.

### 6. Certifieringar (Optional)

Om du vill jobba med K8s professionellt:

**CKAD (Certified Kubernetes Application Developer)**

- Fokus: Att deploya och hantera apps
- Praktiskt exam
- Bra för developers

**CKA (Certified Kubernetes Administrator)**

- Fokus: Cluster administration
- Mer infrastruktur-fokuserat
- Bra för DevOps/SRE

### 7. Community och Resurser

- **Kubernetes Slack** - community support
- **Reddit r/kubernetes** - diskussioner och frågor
- **YouTube** - tutorials och talks
- **Blogs** - många bra K8s blogs
- **Books** - "Kubernetes in Action", "Kubernetes Up & Running"

## Sammanfattning

**Kubernetes** är container orchestration-plattformen som gör det möjligt att köra containers i produktion på stor skala.

### Nyckelpunkter:

**Vad är K8s:**

- Container orchestration system
- Multi-host clustering
- Open source, industry standard
- Ursprung: Google Borg

**Varför K8s:**

- Auto-scaling (både pods och nodes)
- Self-healing (pods startas automatiskt om)
- Rolling updates (zero downtime deployments)
- Service discovery (inbyggd)
- Load balancing (inbyggd)
- Multi-server management
- Declarative configuration

**Grundkoncept:**

- **Pod** - minsta deployable unit (1+ containers)
- **Deployment** - hanterar pods, rolling updates
- **Service** - stabil IP/DNS för pods, load balancing
- **Namespace** - virtuella sub-clusters
- **ConfigMap/Secret** - konfiguration och känslig data
- **PersistentVolume** - storage som överlever pods
- **Labels/Selectors** - organisera och hitta resurser

**Arkitektur:**

- **Control Plane** - API Server, Scheduler, Controller Manager, etcd
- **Worker Nodes** - Kubelet, Container Runtime, Kube-proxy
- **kubectl** - CLI för att interagera med cluster

**K8s Features:**

- Declarative state management
- Continuous monitoring och correction
- Health checks (liveness, readiness, startup)
- Resource management (requests/limits)
- Auto-scaling (HPA, Cluster Autoscaler)
- Multiple deployment strategies

**Docker Compose vs K8s:**

- Compose: Single host, enkel, perfekt för dev/test
- K8s: Multi-host, komplex, perfekt för produktion

**Managed Services:**

- GKE (Google)
- EKS (AWS)
- AKS (Azure)
- → Lättare än self-hosted

**När använda K8s:**
✓ Microservices i stor skala
✓ Flera team/services
✓ High availability krävs
✓ Auto-scaling behövs
✗ Liten app/team
✗ Docker Compose räcker
✗ Ingen K8s-expertis

**Complexity Cost:**

- Learning curve
- Operational overhead
- Networking complexity
- Monitoring/logging setup
- → Väg fördelar mot kostnader

**K8s + Microservices:**

- Perfekt kombination
- Oberoende deployment och scaling
- Built-in service discovery
- Detta är där K8s shinar!

**Nästa steg:**

- Hands-on practice (Minikube, Docker Desktop)
- Officiella tutorials
- Bygg riktiga projekt
- Lär kubectl
- Överväg certifiering (CKAD, CKA)

### Vad Kommer Härnäst?

Nu när du förstår Kubernetes konceptuellt är nästa steg **CI/CD pipelines**. Hur deployer du automatiskt till Kubernetes när du pushar kod till Git? Det är där continuous integration och continuous deployment kommer in!

## Övningsuppgifter

**1. Konceptuell Förståelse**

Förklara relationen mellan Pod, ReplicaSet och Deployment. Varför har Kubernetes denna hierarki istället för att bara ha "Pods med replicas"?

**2. Self-Healing Scenario**

Du har en Deployment med 5 replicas. Beskriv steg-för-steg vad som händer i Kubernetes när:
a) En pod kraschar
b) En hel worker node går ner (med 2 pods på den)

Vilka Kubernetes-komponenter är involverade i varje steg?

**3. Arkitekturdiagram**

Rita ett diagram över Kubernetes cluster-arkitekturen. Inkludera:

- Control Plane (och dess komponenter)
- Worker Nodes (och dess komponenter)
- Hur de kommunicerar
- Var pods körs

**4. Docker Compose vs Kubernetes**

Du har en applikation som består av:

- En Node.js API (Express)
- En React frontend
- En PostgreSQL databas
- En Redis cache

Just nu kör du på en server med Docker Compose.

a) När skulle du överväga att migrera till Kubernetes?
b) Lista minst 5 konkreta tecken/triggers som indikerar att det är dags
c) Vilka är riskerna med att migrera för tidigt?

**5. E-handel på Kubernetes**

Design en microservices e-handelsplattform på Kubernetes. Din design ska inkludera:

**Services:**

- Frontend (React)
- API Gateway
- Authentication Service
- Product Service
- Cart Service
- Order Service
- Payment Service
- Email Service

**För varje service, specificera:**
a) Antal replicas (med motivering)
b) Resource requests/limits (rough estimates)
c) Vilka Services behöver vara exponerade externt vs internt
d) Vilka services behöver persistent storage
e) Vilka services skulle du auto-scala (HPA)

**Rita också:**

- Arkitekturdiagram med alla services
- Request flow från användare till backend
- Vilka Kubernetes resources behövs (Deployments, Services, PVCs, etc.)

**6. Scaling Analys**

Din applikation har följande behavior:

- Normal dag: 1000 requests/sekund, varje pod klarar 200 req/s
- Lunch rush (12-13): 3000 requests/sekund
- Kväll (18-20): 5000 requests/sekund
- Natt (00-06): 200 requests/sekund

a) Designa en HPA-konfiguration för detta. Hur många min/max replicas?
b) Beräkna ungefär hur många pods som kommer köra vid olika tider
c) Förklara skillnaden mellan manual scaling och auto-scaling för detta use case
d) Vilka metrics skulle du övervaka (förutom CPU)?

**7. Service Discovery**

Förklara hur Service discovery fungerar i Kubernetes:

a) Du har en `order-service` som behöver prata med `user-service`. Hur hittar order-service user-service utan hardcoded IPs?
b) Vad händer när user-service pods får nya IP-adresser efter en restart?
c) Hur implementeras load balancing mellan de 5 user-service pods automatiskt?
d) Skriv ett exempel på HTTP-anrop från order-service till user-service med rätt URL

---

**Bonus: Hands-On (Om du har miljö setup)**

8. Installera Minikube eller Docker Desktop K8s och:
   - Deploy nginx med 3 replicas
   - Skapa en Service för att exponera den
   - Skala upp till 5 replicas
   - Gör en rolling update till annan nginx-version
   - Dokumentera varje kommando och output

Lycka till! Kubernetes är komplext men otroligt kraftfullt när du väl förstår det. 🚀
