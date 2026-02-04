# Level 2 — Deployments & Self-Healing

## Doel
Leren hoe Kubernetes applicaties automatisch herstelt door middel van Deployments en de gewenste staat (desired state).

## Wat is er gebouwd
- Een Deployment (`hello-deployment`) in de namespace `dev`
- De Deployment draait een `nginx` container
- Het aantal replicas is configureerbaar via de Deployment

## Belangrijkste concepten
- **Deployment**: bewaakt de gewenste staat van een applicatie
- **ReplicaSet**: zorgt ervoor dat het juiste aantal Pods bestaat
- **Pod**: een vervangbare runtime-instantie

Relatie:
`Deployment → ReplicaSet → Pod`


## Wat is geobserveerd
- Bij het aanmaken van een Deployment maakt Kubernetes automatisch:
    - een ReplicaSet
    - één of meerdere Pods
- Wanneer een Pod handmatig wordt verwijderd:
    - wordt automatisch een nieuwe Pod aangemaakt
- De nieuwe Pod krijgt een andere naam, maar:
    - gebruikt dezelfde image
    - heeft dezelfde labels
    - voldoet aan dezelfde gewenste staat

## Belangrijk inzicht
In Kubernetes is de Deployment het primaire object van een applicatie.
Pods zijn tijdelijke instanties die automatisch worden vervangen wanneer ze verdwijnen.

## Conclusie
Self-healing in Kubernetes ontstaat niet automatisch.
Het werkt alleen omdat de gewenste staat expliciet is vastgelegd in een controller zoals een Deployment.
