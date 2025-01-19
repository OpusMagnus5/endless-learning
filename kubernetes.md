# Kubernetes

Pracujemy na deploymencie. \
Deployment zarządza ReplicaSet. \
ReplicaSet zarządza Pod.
Pod jest warstwą abstrackji dla kontenera.

## Namespaces
W Kubernetesie można organizować zasoby w namespacy w klastrze. To taki wirtualny klaster w klastrze.\
Z automatu tworzą sie namespaces:
* kube-system - namesapce dla procesów kubectl i master, nie należy nic tam ruszać
* kube-public - publicznie dostępne dane, configmap z danymi o klastrze
* kube-node-lease - node heartbeats, informacje na temat dostępności nodów
* default - przechowywuje zasoby tworzone przez nowe namespaces

Namespace tworzymy komendą:
`kubectl create namespace [nazwa namespacu]` \
lub poprzez plik configuracyjny np dla ConfigMapy:
```yaml
apiVersion: apps/v1
kind: ConfigMap
metadata:
  name: mongodb
  namespace: my-namespace #nazwa namespace
```

Można ograniczać zasoby po namespacach.
Każdy namesapce ma swoje ConfigMapy nie widzi je w innych ale może używać servisów z innych namespaców. \
Node i volume nie może być przydzielony do namespacu.

## minikube

Lokalny Kubernetes, skupiający się na ułatwieniu nauki i rozwoju dla Kubernetes.

### Komendy

- `minikube start` - startuje lokalny klaster Kubernetesa (lokalny czyli nody master i slave są na jednym urządzeniu)

## kubectl

Narzędzie wiersza poleceń do komunikacji z płaszczyzną kontrolną klastra Kubernetes przy użyciu interfejsu API Kubernetes.

### Komendy

- `kubectl get nodes` - zwraca status nodów
- `kubectl get pod -o wide` - zwraca status podów z większą ilościa szczegółów
- `kubectl get services` - zwraca status serwisów
- `kubectl get replicaset` - zwraca status replica set
- `kubectl create deployment [nazwa deploymentu] --image=[nazwa obrazu]` - tworzy deployment
- `kubectl delete deployment [nazwa deploymentu]` -  usuwa deployment
- `kubectl apply -f [file name.yaml]` - uruchamia to co jest w pliku konfiguracyjnym zamiast pisać wszystkie opcje np w create deployment, jeśli coś jest uruchomione, zmienimy plik i jeszcze raz wywołamy komendę to zaktualizuje obecną instalacje podów
- `kubectl delete -f [file name.yaml]` - usuwa to co stworzył plik przez apply
- `kubectl edit deployment [nazwa deploymentu]` - otwiera plik z konfiguracją deploymentu, który możemy skonfigurować
- `kubectl logs [nazwa podu]` - wchodzi w logi poda
- `kubectl describe pod [pod name]` - szczegółowe informacje na temat poda w tym historie statusów
- `kubectl exec -it [pod name] -- bin/bash` - wchodzi w terminal aplikacji
- `kubectl get deployment [nazwa deployemntu] -o yaml` - zwraca plik konfiguracyjny deployment
- `kubectl get namespaces` - zwraca namepsaces
- `kubectl get [np delpoyment] -n my-namespace` - gdy jakiś zasób jest przydzielony do konkretnego zasobu

### Pliki konfiguracyjne

* deployment
```yaml
apiVersion: apps/v1
kind: Deployment #rodzaj konfiguracji
metadata:
  name: mongodb
  labels:
    app: mongodb
spec: #spec jest dla deploymentu
  replicas: 1 #liczba replik
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec: #specyfikacja dla podów
      containers:
        - name: mongodb
          image: mongo
          ports:
            - containerPort: 27017
          env: #zmienne środowiskowe
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef: #użycie pliku z sekretem
                  name: mongodb-secret
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: password

```

* możemy tez kilka dokumentów w jednym pliku yaml rozdzielonych ---

```yaml
apiVersion: apps/v1
kind: Deployment #rodzaj konfiguracji
metadata:
  name: mongodb
  labels:
    app: mongodb
spec: #spec jest dla deploymentu
  replicas: 1 #liczba replik
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec: #specyfikacja dla podów
      containers:
        - name: mongodb
          image: mongo
          ports:
            - containerPort: 27017
          env: #zmienne środowiskowe
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef: #użycie pliku z sekretem
                  name: mongodb-secret
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: password
            - name: ME_CONFIG_MONGODB_SERVER
              valueFrom:
                configMapKeyRef: #użycie configmapy
                  name: mongodb-configmap
                  key: db_host

---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  type: LoadBalancer #dla external service, default nie musimy podawać i jest ClusterIp
  #Po uruchomieniu musimy odpalic komende - minikube service [nazwa servisu] | aby nadać publiczne ip
  ports:
    - protocol: TCP
      port: 27017 #service port
      targetPort: 27017 #container port
      nodePort: #port dla external service, musi być miedzy 30000 - 32767

```

* secret

```yaml
apiVersion: v1
kind: Secret #typ pliku Secret
metadata:
  name: mongodb-secret #nazwa sekretu
type: Opaque #default type dla pary klucz - wartość
data: #podajemy klucz i wartość dla sekretu
  username: dXNlcm5hbWU= #wartość musi zostać zakodowana w base64
  password: cGFzc3dvcmQ=
```

* service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

```

* config map

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  db_host: mongodb-service
```

## Ingress

Ingres służy do routowania połączeń między wystawionymi serwisami.
Do tego trzeba zainstalować implementacje Ingresa czy ingres controller który 
będzie wykonywał reguły zapisane w pliku.

Plik konfiguracyjny:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: my-host.com
    http:
      paths:
      - path: /
        pathType: Prefix  
        backend:
          service:
            name: mongodb-service
            port: 
              number: 80
```

`minikube addons enable ingress` - automatycznie instaluje nginx controller dla ingresa

## Helm

Helm jest package manager dla Kubernetesa. Pakowanie plików yaml i dystrybucja w reposytoriach. 

### Helm Charts
Jest to paczka plików yaml, można stworzyć własny helm charts za pomocą Helma.

Struktura:
katalog z nawą chartu
chart.yaml - meta info, name, zależności, versja
values.yaml - deafultowe wartości dla templatu
charts - katalog z zależnościami
templates - katalog z szablonami chartsów

### Template engine
Obsługuje silnik templatów, gdzie możemy parametryzować konfiguracje za pomocą placholderów np:
```yaml
name: {{ .Values.name }}
```

W takim przypadku zmienne są brane z dodatkowego pliku konfiguracyjnego - `values.yaml`

Wartości można nadpisać innym plikiem values.yaml, nowe wartości zostaną dopisane a jeśli się powtarzają zostaną nadpisane.

`helm install --values=my-values.yaml [chart name]`

Helm w wersji 3 nie przechowuje hisotrii deploymentu