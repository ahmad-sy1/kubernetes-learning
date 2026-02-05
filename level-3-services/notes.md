# LEVEL 3 — Services (stabiel netwerk & “switching”)

---
**Directory voor Level 3**
```
level-3-services/
├── service.yaml
├── README.md
└── notes.md
Service.yaml is leeg, nog niks ermee gedaan
```
---

## Inspecteer je huidige situatie
```
kubectl get pods -n dev -o wide
```
**Wat zien we:**

| NAME                            | READY | STATUS  | RESTARTS | AGE | IP           | NODE       | NOMINATED NODE | READINESS GATES |
|---------------------------------|-------|---------|----------|-----|--------------|------------|----------------|-----------------|
| hello-deployment-c9f85d78-hvx2q | 1/1   | Running | 0        | 19h | 10.244.0.5   | minikube   | <none>         | <none>          |
| hello-deployment-c9f85d78-rhhng | 1/1   | Running | 0        | 19h | 10.244.0.6   | minikube   | <none>         | <none>          |
| hello-deployment-c9f85d78-wr47s | 1/1   | Running | 0        | 19h | 10.244.0.7   | minikube   | <none>         | <none>          |

**Wat weten we:**
- elke Pod heeft een IP
- **die IP’s zijn:** intern, tijdelijk, en verdwijnen bij Pod replace

👉 Dit is waarom Pods nooit je endpoint mogen zijn.

---

## Maak een Service

```
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  namespace: dev
spec:
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP

```

### Uitleg per onderdeel

```
apiVersion: v1
```
➡️ Geeft aan dat dit een core Kubernetes-object is (Services zitten in de core API).

```
kind: Service
```
➡️ Dit object is een Service, bedoeld om netwerkverkeer naar Pods te routeren.

```
metadata:
```
➡️ Algemene informatie over het Service-object.

```
name: hello-service
```
➡️ De vaste naam waarmee andere onderdelen (of jij) deze Service kunnen bereiken.

```
namespace: dev
```
➡️ De Service bestaat in de namespace `dev` en kan alleen Pods in die namespace selecteren.


```
spec:
```
➡️ Beschrijft hoe de Service zich moet gedragen (desired state).

```
spec:
```
➡️ Beschrijft hoe de Service zich moet gedragen (desired state).

```
selector:
```
➡️ Bepaalt welke Pods verkeer mogen ontvangen via deze Service.


```
app: hello
```
➡️ Alle Pods met label app=hello worden automatisch onderdeel van deze Service.


```
ports:
```
➡️ Definieert op welke poorten de Service verkeer accepteert en doorstuurt.


```
- port: 80
```
➡️ De poort waarop de Service bereikbaar is.

```
targetPort: 80
```
➡️ De poort op de Pod/container waar het verkeer naartoe wordt gestuurd.

```
type: ClusterIP
```
➡️ Maakt de Service alleen intern bereikbaar binnen het cluster (standaard type).


## Samenvatting in één zin (zoals bij Level 2)

Deze Service creëert een stabiel intern endpoint dat verkeer verdeelt over alle Pods met label app=hello in de namespace dev.

**Belangrijk:**

- selector = koppeling met je Deployment
- Service kent geen Pods bij naam
- Alleen labels tellen

### Mini-vergelijking

- Deployment → zorgt dát Pods bestaan
- Service → zorgt dát Pods bereikbaar zijn

---

## Apply de Service

```
kubectl apply -f service.yaml
```

**Check**
```
kubectl get services -n dev
```

**Je ziet iets als:**

| NAME          | TYPE      | CLUSTER-IP      | EXTERNAL-IP | PORT(S) | AGE  |
|---------------|-----------|-----------------|-------------|---------|------|
| hello-service | ClusterIP | 10.105.254.100  | <none>      | 80/TCP  | 102s |

*👉 Dit IP verandert niet, zelfs niet als Pods verdwijnen.*


## Bewijs dat Service werkt

We gaan port-forwarden (simpel, geen Ingress nog).
```
kubectl port-forward service/hello-service 8080:80 -n dev
```
**Ga in je browser naar:**
```
http://localhost:8080
```
**Je ziet nginx 🎉**

**Let op:**
- Je praat niet met een Pod
- Je praat met de **Service**
- Serevice kiest een Pod

---

## Het “switch”-moment
Laat port-forward draaien.

Open eventueel 2 terminals.

**In terminal 2:**
```
kubectl get pods -n dev
kubectl delete pod <POD_NAAM> -n dev
```

## Wat gebeurde er bij het verwijderen van een Pod?

Tijdens het verwijderen van een Pod viel de port-forward verbinding weg.
Dit kwam niet doordat de Service stopte, maar omdat `kubectl port-forward`
een tunnel opent naar één specifieke Pod.

Toen deze Pod werd verwijderd:
- werd de tunnel verbroken (vergelijkbaar met het lostrekken van een stekker)
- bleef de Service bestaan
- bleef de Deployment actief
- werd automatisch een nieuwe Pod aangemaakt door de ReplicaSet

Na het opnieuw starten van `kubectl port-forward` werkte de verbinding weer,
omdat kubectl een nieuwe Pod koos achter de Service.

---

## Hoe reageert een Service wanneer een Pod verdwijnt?

Een Service houdt zelf geen verbindingen open naar Pods.
In plaats daarvan houdt de Service een lijst bij van alle Pods die
`Ready` zijn en voldoen aan de selector.

Wanneer een Pod:
- wordt verwijderd
- of `NotReady` wordt

dan:
- verdwijnt deze automatisch uit de endpoint-lijst van de Service
- stuurt de Service nieuw verkeer alleen naar andere beschikbare Pods

Dit proces gebeurt automatisch en vereist geen handmatige actie
van een developer.

**De Service en de Deployment werken onafhankelijk van elkaar:**

- De Service bepaalt waar verkeer naartoe gaat
- De Deployment bepaalt hoeveel Pods er bestaan

Als een Pod verdwijnt:
- de Service stopt met verkeer daarheen
- de Deployment zorgt dat er een nieuwe Pod komt

---

## Conclusie

In Kubernetes praat verkeer nooit direct met Pods.
De Service fungeert als stabiel netwerk-endpoint en verdeelt verkeer
over alle beschikbare Pods.

Het wegvallen van een Pod heeft geen impact op de Service zelf.
Alleen tijdelijke debug-tools zoals `kubectl port-forward`
kunnen hun verbinding verliezen.



