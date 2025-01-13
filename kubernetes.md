# Kubernetes

Pracujemy na deploymencie. \
Deployment zarządza ReplicaSet. \
ReplicaSet zarządza Pod.
Pod jest warstwą abstrackji dla kontenera.

## minikube

Lokalny Kubernetes, skupiający się na ułatwieniu nauki i rozwoju dla Kubernetes.

### Komendy

- `minikube start` - startuje lokalny klaster Kubernetesa (lokalny czyli nody master i slave są na jednym urządzeniu)

## kubectl

Narzędzie wiersza poleceń do komunikacji z płaszczyzną kontrolną klastra Kubernetes przy użyciu interfejsu API Kubernetes.

### Komendy

- `kubectl get nodes` - zwraca status nodów
- `kubectl get pod` - zwraca status podów
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

### Pliki konfiguracyjne
```yaml
apiVersion: apps/v1
kind: Deployment #rodzaj konfiguracji
metadata:
  name: nginx-deployment #nazwa deploymentu
  labels:
    app: nginx
spec: # spec jest dla deploymentu
  replicas: 2 #liczba replik
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec: #specyfikacja dla podów
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 8080

```