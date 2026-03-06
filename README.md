# Kubernetes Study Repo
My personal repo mainly written in Polish for learning and studying Kubernetes while at the same time strengthening my git/github skills.<br>
Główne źródło: [👆](https://www.youtube.com/watch?v=X48VuDVv0do)<br>Left at 1:26:45
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
Ta część kodu dotyczy jaki ma powstać Pod w Node, blueprint Poda.
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

Wynik komendy: `k describe service nginx-service`, uzywając pliku konfiguracyjnego z [nginx-service.yaml](nginx-service.yaml)<br>
```
Name:                     nginx-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.24.131
IPs:                      10.43.24.131
Port:                     <unset>  80/TCP
TargetPort:               8080/TCP
Endpoints:                10.42.0.12:8080,10.42.0.13:8080
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```
Aby zweryfikować adresy IP zeby upewnić się ze nalezą do poprawnych Podów nalezy sprawdzić komendą:<br>
`k get pod -o wide` - "o" jest skrótem "output", "wide" uzywamy aby dostać więcej informacji.<br>
Wynik komendy:
```
NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE                        NOMINATED NODE   READINESS GATES
nginx-deployment-7f65fcf556-46wqj   1/1     Running   0          4h16m   10.42.0.12   k3d-main-cluster-server-0   <none>           <none>
nginx-deployment-7f65fcf556-w9bbx   1/1     Running   0          12m     10.42.0.13   k3d-main-cluster-server-0   <none>           <none>
```
- Jak widać adresy IP zgadzają się z Endpointami w Service.<br>

`k get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml` - komenda która spisuje zaupdateowany output deploymentu pliku konfiguracyjnego, i zapisuje go na nowy plik [nginx-deployment-result.yaml](nginx-deployment-result.yaml)
<br>
<br>
Na pliku mozna zaobserować automatycznie dodane przez K8s zmiany od czasu zaimplementowania go, które są aktualizowane na bieząco przez K8s. Utworzenie takiego pliku jest bardzo uzyteczne w debugowaniu.
- W przypadku kiedy chcemy skopiować Deployment który juz mamy uzywając np. zautomatyzowanego skryptu, nalezy usunąć większość wygenerowanego przez K8s rzeczy. Dopiero wtedy mozemy stworzyć następny Deployment z tego blueprintu konfiguracyjnego.<br>

Aby usunąć Deployment lub Service nalezy uzyć komendy:<br>
`k delete -f <nazwa-deploymentu>`<br>
`k delete -f <nazwa-service>`
# Demo Project: MongoDB and MongoExpress
Przegląd komponentów:
- Stworzymy Poda MongoDB, aby rozmawiać z podem nalezy utworzyć Service.
- Service będzie Internal Service, czyli zaden zewnętrzny request jest dozwolony do Poda, tylko komponenty w tym samym Klastrze mogą z nim rozmawiać.
- Stworzymy MongoExpress Deployment, będziemy potrzebać: DB Url (przez ConfigMap) MongoDB zeby MongoExpress się z nim komunikował; DB User, DB Pwd (przez Secret) MongoDB zeby MongoExpress mógł autoryzować.
- Zeby przekazać tę informacje Deploymentowi MongoExpress uzyjemy deployment.yaml przez Env variables (zmienne środowiskowe)
- Jak zrobimy setup będziemy potrzebować MongoExpress zeby był dostępny przez przeglądarke, do tego stworzymy External Service, co pozwoli zewnętrzne requesty aby mówiły do Poda.

Url:
- IP Address of Node
- Port of external Service

#### Browser request flow przez komponenty K8s
- Request przychodzi z przeglądarki
- Przechodzi do External Service MongoExpress który zforwarduje go do Poda MongoExpress
- Pod połączy się z Internal Service MongoDB który zforwarduje go do Poda MongoDB
- Pod MongoDB autoryzuje request uzywając Credentials w Secrecie (Secret zyje w K8s, nie w Repo)

#### Tworzymy plik konfiguracyjny Deploymentu MongoDB
- Wszystkie potrzebne informacje znajduje na stronie [DockerHub](https://hub.docker.com/_/mongo)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
specs:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
  spec:
    containers:
    - name: mongodb
      image: mongo
      ports:
      - containerPort: 27017
        value:
      - name: MONGODB_INITDB_ROOT_PASSWORD
        value:
```
- Aby zachować bezpieczeństwo Bazy Danych tworzymy plik konfiguracyjny obiektu Secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
    mongo-root-username: bmF6d2E=
    mongo-root-password: aGFzbG8=
```
- Hasła w Secrecie są szyfrowane w base64, dlatego aby zrobić bezpieczniejesze hasło i nazwe zrobimy podwójną enkrypcje uzywając komendy:<br>
`echo -n 'nazwa' | base64`
- Secret musi zostać utworzony przed Deploymentem
- Tworzymy obiekt Secret, mozemy go sprawdzić komendą: `k get secret`<br>
Wynik komendy:
```
NAME             TYPE     DATA   AGE
mongodb-secret   Opaque   2      57s
```