## Observaties
- Toen ik de Pod verwijderde, kwam hij niet terug
- Kubernetes maakte geen nieuwe Pod aan

## Inzicht
Kubernetes houdt niets automatisch in stand.
Als er geen regel (controller) is die zegt dat iets moet blijven bestaan,
dan accepteert Kubernetes dat het verdwijnt.

## Belangrijk inzicht
- Pods zijn uitvoerders
- Controllers (zoals Deployments) bewaken de gewenste staat
- Een Pod wordt niet automatisch opnieuw aangemaakt als hij wordt verwijderd,
  tenzij er een controller aanwezig is die de gewenste staat afdwingt.