# cert-manager-demo

Das Ziel der Demo ist es unsere Chat-App über einen Webbrowser zugänglich zu machen und mittels TLS Terminierung abzusichern.
Dabei deployen wir unsere Chat-App in einem Kubernetes-Cluster richten wir den von Kubernetes verwalteten Nginx-Ingress-Controller ein und erstellen eine Ingress-Ressourcen, 
um den Datenverkehr an unseren Chat-Dienste zu leiten. Sobald wir den Ingress eingerichtet haben, werden wir **cert-manager** in unserem Cluster installieren, 
um TLS-Zertifikate für die Verschlüsselung des HTTP-Verkehrs zum Ingress zu verwalten und bereitzustellen.

### Vorbedingungen
* Docker Image der Chat-App über Docker Hub zugänglich machen: [Pushing a Docker container image to Docker Hub](https://docs.docker.com/docker-hub/repos/#:~:text=To%20push%20an%20image%20to,docs%2Fbase%3Atesting%20)
* Installieren von kubectl: [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
* Kubernetes Cluster: Ich verwende [DigitalOcean](https://www.digitalocean.com/products/kubernetes), andere Tools wie [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) sind auch möglich
* Eigene Domain registrieren: Bspw. bei [Google Domains](https://domains.google/intl/de_de/) !!! ACHTUNG: Kostenpflichtig !!!

## 1. Deploy Chat-App
Zuallererst erstellen wir einen Namespace, in dem unsere Chat-App eingebettet ist.
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: chat
```
```
kubectl apply -f namespace.yaml
```

Wir definieren ein Deployment, das unseren Pod managed, in dem unser Chat-App Container läuft. Das Docker Hub image `nidigeser/chatapp:latest` wird verwendet.
Die Chat-App hört auf den Port 3000.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chat
  namespace: chat
  labels:
    app: chat
spec:
  replicas: 1
  selector:
    matchLabels:
      app: chat
  template:
    metadata:
      labels:
        app: chat
    spec:
      containers:
      - name: chat
        image: nidigeser/chatapp:latest
        ports:
        - containerPort: 3000
```

Zum Pod definieren wir den dazugehörigen Service, der über Port 80 ansprechbar ist und Anfragen an unsere Chat-App auf den Port 3000 weiterleitet.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: chat
  namespace: chat
spec:
  selector:
    app: chat
  ports:
    - protocol: TCP
      name: http
      port: 80
      targetPort: 3000
```

Wir erzeugen die Ressourcen
```
kubectl apply -f chat-app.yaml
```
und überprüfen, dass der Pod läuft
```
kubectl get pod --namespace chat
```
```
NAME                   READY   STATUS    RESTARTS   AGE
chat-d6455c79b-2tncn   1/1     Running   0          45s
```

## 2. Deploy Ingress Controller
Der Ingress Controller ist eine Anwendung, die in einem Cluster ausgeführt wird und einen HTTP Load-Balancer entsprechend den Ingress-Ressourcen konfiguriert. Ein solcher Load Balancer ist erforderlich, um Anwendungen an Clients außerhalb des Kubernetes-Clusters zu liefern. Der Ingress Controller wird zusammen mit dem Load Balancer in einem Pod bereitgestellt.
Wir verwenden als Ingress Controller NGINX und installieren diesen über das [Digital Ocean Manifest](https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md#digital-ocean). Weitere Installationsmöglichkeiten findet ihr [hier](https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/do/deploy.yaml
```
```
kubectl get pods --namespace ingress-nginx
```
```
kubectl get svc --namespace ingress-nginx
```
```
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.245.187.242   157.245.26.212   80:32726/TCP,443:32528/TCP   1m52s
```

