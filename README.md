# cert-manager-demo

Das Ziel der Demo ist es unsere Chat-App über einen Webbrowser zugänglich zu machen und mittels TLS Terminierung abzusichern.
Dabei deployen wir unsere Chat-App in einem Kubernetes-Cluster richten wir den von Kubernetes verwalteten Nginx-Ingress-Controller ein und erstellen eine Ingress-Ressourcen, 
um den Datenverkehr an unseren Chat-Dienste zu leiten. Sobald wir den Ingress eingerichtet haben, werden wir **cert-manager** in unserem Cluster installieren, 
um TLS-Zertifikate für die Verschlüsselung des HTTP-Verkehrs zum Ingress zu verwalten und bereitzustellen.

## Vorbedingungen
* Docker Image der Chat-App über Docker Hub zugänglich machen: [Pushing a Docker container image to Docker Hub](https://docs.docker.com/docker-hub/repos/#:~:text=To%20push%20an%20image%20to,docs%2Fbase%3Atesting%20)
* Installieren von kubectl: [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
* Kubernetes Cluster: Ich verwende [DigitalOcean](https://www.digitalocean.com/products/kubernetes), andere Tools wie [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) sind auch möglich
* Eigene Domain registrieren: Bspw. bei [Google Domains](https://domains.google/intl/de_de/) !!! ACHTUNG: Kostenpflichtig !!!

