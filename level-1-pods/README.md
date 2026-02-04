# Level 1 — Pods & Namespaces

## Doel
Begrijpen wat een Pod is, hoe namespaces werken en waarom Pods zonder controller geen self-healing hebben.

## Wat ik heb gedaan
- Een namespace `dev` aangemaakt
- Een Pod `hello-pod` gestart met de image `nginx:latest`
- De Pod geïnspecteerd met `kubectl describe`
- De Pod verwijderd om te observeren wat Kubernetes doet

## Gebruikte commando’s
```bash
kubectl create namespace dev
kubectl run hello-pod --image=nginx:latest --restart=Never -n dev
kubectl get pods -n dev
kubectl describe pod hello-pod -n dev
kubectl exec -n dev hello-pod -- curl localhost
kubectl delete pod hello-pod -n dev
```


## Wat ik heb geleerd

- Een Pod is de kleinste runtime-eenheid in Kubernetes
- Pods bestaan altijd binnen een namespace
- Een namespace is een soort van map
- Een pod is hetzelfde als een bestand die zich plaats vindt in een namespace (map)
- Een Pod herstelt zichzelf niet als hij wordt verwijderd
- Kubernetes doet alleen wat expliciet is gedefinieerd
- Zonder een controller is er geen self-healing

# Conclusie

Pods zijn wegwerpbaar en niet geschikt om direct in productie te gebruiken zonder een controller zoals een Deployment.

