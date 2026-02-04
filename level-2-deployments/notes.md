## Mijn notities voor dezel level

### Uitleg Deployment.yaml bestand

```apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

### Uitleg
```
apiVersion: apps/v1
```
- Welke API-groep/versie Kubernetes moet gebruiken om dit object te begrijpen.
- Deployments horen bij de apps API group.
- V1 is de stabiele versie.

👉 Zonder juiste apiVersion: Kubernetes snapt het object niet.

```
kind: Deployment
```
**Wat voor type object je aanmaakt**
- Deployment = controller die gewenste staat bewaakt voor Pods.
- Deployment maakt (meestal) een ReplicaSet aan die de Pods beheert.

👉 Dit is het “ik wil dat het blijft draaien”-object.

---

```
metadata:
```

*Algemene info over het object (denk: naamkaartje + labels + namespace).*

```
name: hello-deployment
```
**De naam van dit Deployment-object.**
- Uniek binnen de namespace.
- Deze naam gebruik je bij ``kubectl get deployment hello-deployment -n dev.``

👉 Dit is NIET de naam van de Pod. Pods krijgen suffixen.

```
namespace: dev
```
**Waar dit object leeft.**
- Deployment komt in namespace ``dev``.
- Alles wat het Deployment aanmaakt (ReplicaSet + Pods) komt ook in ``dev``.

👉 Als dev niet bestaat: apply faalt.

---

```
spec:
```
*De “desired state” — hier beschrijf je wat je wilt, niet wat er nu is.*

```
replicas: 1
```
**Hoeveel Pods je tegelijk wilt hebben.**
- `1` = één draaiende instantie
- `2` = twee Pods parallel (redundancy, load)
- Kubernetes gaat continu proberen dit getal werkelijkheid te maken.

👉 Dit is letterlijk de self-healing knop.

```
selector:
```
*Dit is superbelangrijk en vaak verkeerd begrepen.*

```
matchLabels:
```
**Hier zeg je:**

*"Deze Deployment is verantwoordelijk voor Pods met deze labels.”*

```
matchLabels:
    app: hello
```
👉 De Deployment “claimt” Pods met label app=hello.

**Waarom bestaat dit?**
- Kubernetes moet kunnen weten welke Pods bij dit Deployment horen.
- Het voorkomt dat Deployments elkaars Pods per ongeluk overnemen.

**❗ Belangrijke regel:**

`selector.matchLabels` moet matchen met `template.metadata.labels` (zie hieronder), anders werkt het niet zoals je denkt.

---

```
template:
```
*Dit is de pod template: een blauwdruk voor Pods die gemaakt moeten worden.*

`template.metadata.labels:`
```
labels:
  app: hello
```

*Dit zijn labels die elke Pod krijgt die door dit Deployment wordt gemaakt.*

**Waarom belangrijk?**

- Deze labels moeten matchen met de selector. 
- Later gaat je Service op labels selecteren, dus dit is je “koppelpunt”.

---

```
template.spec:
```
*Dit is de pod spec: hoe de Pod precies draait.*

---

```
containers:
```
*Een Pod kan meerdere containers hebben, maar meestal 1.*

*Hier definiëer je de containers die in de Pod moeten draaien.*

---

```
- name: nginx
```
*Naam van de container binnen de Pod.*
- Handig voor logs/exec
- `kubectl logs <pod> -c nginx`
- `kubectl exec -it <pod> -c nginx -- sh`

👉 Containernaam is lokaal binnen de Pod.

--- 

```
image: nginx:latest
```
*Welke Docker image gebruikt wordt voor die container.*
- Kubernetes trekt deze image (van Docker Hub tenzij anders).
- `latest` is een tag, niet “automatisch altijd nieuwste” (maar kan wel verwarrend zijn).

⚠️ Kleine maar belangrijke best practice:

In productie liever een vaste tag (bv `nginx:1.25.3`) of een digest, omdat `latest` kan veranderen.


## Samenvatting (wat dit hele bestand “zegt”)

Kubernetes, ik wil in namespace `dev` een Deployment genaamd `hello-deployment` die altijd 1 Pod draaiend houdt met label `app=hello`, en in die Pod draait een container `nginx` gebaseerd op `nginx:latest`.

---

## Uitgevoerde commandos
```
kubectl get pods -n dev
```

*Deze commando zegt letterlijk laat zien welke pods je hebt in de namespace `dev`*

---
```
kubectl apply -f deployment.yaml
```
**Wat doet dit commando:**
- Kubernetes API slaat jouw desired state op
- Deployment controller ziet:
    
    ``ik moet 1 replica hebben``
- Deployment maakt een ReplicaSet
- ReplicaSet maakt 1 Pod

--- 
```
kubectl get replicasets -n dev
```
of kort: `kubectl get rs -n dev)`

**Wat doet dit commando:**

👉 Het vraagt aan de Kubernetes API:

- “Laat mij alle ReplicaSet-objecten zien in de namespace `dev`.”
  
Het start niets, het verandert niets. 
Het is puur observatie.

**Wat is een ReplicaSet (in 1 zin)**

- *Een ReplicaSet zorgt ervoor dat er altijd precies X Pods bestaan met bepaalde labels.*
- ReplicaSet is ontstaan doordat we de commando `kubectl apply -f deployment.yaml` hebben gerunt, waardoor de API de `yaml` gebruikt.
- Aan de hand van de `yaml` bestand, is ReplicaSet ontstaan.

---

```
kubectl get pods -n dev
```
*Dit toont de huidige beschikbare pods*

**Return value:**

| NAME | READY | STATUS | RESTARTS | AGE |
|------|-------|--------|----------|-----|
| hello-deployment-c9f85d78-p4qcf | 1/1 | Running | 0 | 9m23s |

---
## Checken dat ik de pod n NIET zelf heb gemaakt:
```
kubectl describe pod hello-deployment-c9f85d78-p4qcf -n dev
```
**Zoek naar:**
``Controlled By:  ReplicaSet/hello-deployment-...
``

*Dit is het bewijs van self-healing*

De Pod “hoort” bij iets dat ’m bewaakt.

--- 
## Het moment van waarheid: kill de Pod
**Pak de Pod-naam en:**
```
kubectl delete pod <POD_NAAM> -n dev
```

**In dit geval:**
`kubectl delete pod hello-deployment-c9f85d78-p4qcf -n dev`

### Ik check daarna of de pod nog bestaat of niet:
```
kubectl get pods -n dev -w
```
---

## Begrijpen wie wat deed

- Deployment: bewaakt gewenste staat (replicas)
- ReplicaSet: bewaakt Pods
- Pod: uitvoerder, vervangbaar

`Deployment → ReplicaSet → Pod`

---
## Extra overservatie

Ik verander tijdelijk in `deployment.yaml`:

```
replicas: 3
```

**Dan:**
```
kubectl apply -f deployment.yaml
kubectl get pods -n dev
```
*Ik zie 3 Pods verschijnen, zonder dat ik Pods aanmaakt.*

---

## Test vragen:
1. Wie maakte de nieuwe Pod na delete?
```
Antwoord: “Na het verwijderen van een Pod maakt de ReplicaSet de pods weer aan.”
```

2. Waarom maakt Kubernetes geen Pod met exact dezelfde naam?

```
Antwoord: Omdat een Pod een tijdelijke instantie is, geen vaste identiteit.
    - Een Pod is vervangbaar
    - Kubernetes ziet een nieuwe Pod als een nieuwe instantie
    - Namen zijn uniek om:
    - events, logs, lifecycle, debugging, correct te kunnen volgen
```

3. Wat is het “echte” object van je applicatie?
```
In Kubernetes is de Deployment het primaire object van een applicatie;
Pods zijn vervangbare instanties die door de Deployment worden beheerd.
```





