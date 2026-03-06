# k8s-study
My personal repo mainly written in Polish for learning and studying Kubernetes while at the same time strengthening my git/github skills.
## Architektura Klastra Kubernetesowego:
![Architektura](/images/kubernetes-cluster-architecture.svg)<br>

# Deploymenty
### Podstawowa konfiguracja Deploymentu w formacie .yaml :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 80
```
Execujemy plik używając Komendy:`k apply -f <nazwa-pliku>`<br>
### Komendy
`k delete deployment <nazwa-deploymentu>` - usuwa deployment *Replicaset zostaje*<br>
`k edit deployment <nazwa-deploymentu>` - otwiera pole tekstowe w $vim$ z plikiem konfiguracyjnym deploymentu<br>
`k create deployment <nazwa-deploymentu> --image=OBRAZ` - tworzy deployment z plikiem konfiguracyjnym zgodnie z wybranym obrazem<br>
## Debugowanie Podów
### Komendy
`k get logs <nazwa-poda>` - żeby komenda działała prawidłowo Kontener musi być Running<br>
`k describe pod <nazwa-poda>` - pokazuje eventy które zachodzą/zaszły w podzie, Kontener nie musi być running<br>
`k exec -it <nazwa-poda> -- bin/bash` - otwiera terminal Kontenera aplikacji<br>