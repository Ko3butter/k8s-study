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
# Plik konfiguracyjny K8s
Kazdy plik konfiguracyjny ma 3 części:<br>
1) Metadata - name, label
2) Specification (spec) - wpisujesz jakiekolwiek specyfikacje jakie chcesz zeby skonfigurował: apiVersion: (dla kazdego komponentu jest inny apiVersion np. v1); kind: np. Deployment, Service, Pod;
3) Status - kubernetes zawsze bedzie porownywal desired state z actual state, jezeli obydwa stany nie są równe to k8s wie ze jest cos do naprawienia, wiec bedzie próbował to naprawić (basis of self-healing feature)<br>
*etcd trzyma aktualny status kazdego komponenta K8s*<br><br>
Zachowuj pliki konfiguracyjne razem ze swoim kodzie, ale mogą tez być na oddzielnym Repo gdzie tylko są pliki konfiguracyjne.
### Warstwy Abstrakcji:
- Deployment zarządza Replicasetem
- Replicaset zarządza Podem
- Pod jest abstrakcją Kontenera<br>
*Wszystko ponizej Deploymenta powinno być zarządzane przez K8s. (zwykle nie musisz się o to martwić)*
<br><br>
W przykładowym pliku [nginx-deployment.yaml](nginx-deployment.yaml)
```yaml
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
Ta część kodu dotyczy do jaki ma powstać w Node, blueprint Poda.
### Łączenie komponentów (Labels, Selectors, Ports)
#### Podpinanie Deploymentów do Podów
- Metadata przechowuje Labels<br>
- Spec przechowuje Selectors i Ports<br>
- Pody dostają Label poprzez Template blueprint<br>
- Label jest matched razem z selectorem (nazywają się tak samo.)<br>
#### Podpinanie Services do Deploymentów
Service łączą się do Deploymentów przez Selector który robi połączenie pomiędzy Service a Deploymentem lub jego Podami.<br>
<br>
Poniewaz Service musi wiedzieć które Pody są zarejestrowane/dotyczą tego Service.<br>
- Lokalizacja label na pliku konfiguracyjnym Deployment

```yaml
metadata:
   labels:
   app: nginx
```
- Lokalizacja selector i ports na pliku konfiguracyjnym Service
```yaml
metadata:
  name: nginx-service
spec:
  selector:
  app: nginx
  ports:
    - protocol: TCP
      port: 80 // port na którym Service jest dostępny
      targetPort: 8080 // port do którego ma wysyłać dane (forwardować request) aka. port na którym nasłuchuje docelowy Pod
```
- *Porty zawsze muszą zostać uwzględnione i w plikach konfiguracyjnych Serviceów i w plikach konfiguracyjnych Podów*
