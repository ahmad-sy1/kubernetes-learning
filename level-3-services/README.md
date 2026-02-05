# Level 3 — Services (stabiel netwerk & “switching”)

## Doel
Begrijpen hoe Kubernetes netwerkverkeer stabiel naar Pods routeert,
ook wanneer Pods verdwijnen of opnieuw worden aangemaakt.

## Wat is er gebouwd
- Een Service (`hello-service`) in de namespace `dev`
- De Service routeert verkeer naar Pods met label `app=hello`
- De Service gebruikt het type `ClusterIP`

## Belangrijkste concepten
- **Service**: stabiel netwerk-endpoint binnen het cluster
- **Selector**: koppelt een Service aan Pods via labels
- **Pod IP’s**: tijdelijk en ongeschikt als vast endpoint

**Relatie:**

Service → Pods


(De Deployment blijft verantwoordelijk voor het bestaan van Pods.)

## Wat is geobserveerd
- Elke Pod heeft een eigen, tijdelijk IP-adres
- De Service krijgt één stabiel ClusterIP
- Verkeer wordt automatisch verdeeld over beschikbare Pods
- Wanneer een Pod wordt verwijderd:
    - verdwijnt deze uit de Service
    - blijft de Service bestaan
    - wordt verkeer automatisch naar andere Pods gestuurd

## Belangrijk inzicht
In Kubernetes praat netwerkverkeer nooit direct met Pods.
De Service fungeert als stabiel aanspreekpunt en verdeelt verkeer
over alle beschikbare Pods op basis van labels.

## Nuance (belangrijk)
Tijdens het testen viel `kubectl port-forward` weg bij het verwijderen
van een Pod. Dit kwam niet doordat de Service stopte, maar omdat
`kubectl port-forward` een tijdelijke verbinding opent naar één Pod.

In een echte productie-opzet (bijvoorbeeld via Ingress of een andere client)
blijft de Service functioneren zonder onderbreking.

### Architectuuroverzicht

<img alt="#" src="img.png" width="500">

Dit diagram toont de samenhang tussen de belangrijkste Kubernetes-componenten
binnen één cluster.

Binnen het cluster draaien meerdere Pods waarin de containers van de applicatie
worden uitgevoerd. Deze Pods zijn tijdelijk en vervangbaar.

De Deployment beschrijft de gewenste staat van de applicatie en gebruikt een
ReplicaSet om ervoor te zorgen dat het juiste aantal Pods actief blijft.
Wanneer een Pod verdwijnt, zorgt de ReplicaSet ervoor dat er automatisch
een nieuwe Pod wordt aangemaakt.

De Service staat los van de Deployment en ReplicaSet en selecteert Pods uitsluitend
op basis van labels. De Service fungeert als een stabiel netwerk-endpoint en houdt
een lijst bij van alle Pods die klaar zijn om verkeer te ontvangen.

Wanneer een Pod niet meer beschikbaar is, wordt deze automatisch uit de Service
verwijderd en wordt verkeer naar andere beschikbare Pods gestuurd.


## Conclusie
Services lossen het probleem van dynamische en vervangbare Pods op
door een stabiel netwerk-endpoint te bieden.
“Switching” tussen Pods gebeurt automatisch binnen de Service,
zonder handmatige acties van een developer.






