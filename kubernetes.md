# Kubernetes

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
- `kubectl create deployment` [nazwa deploymentu] --image=[nazwa obrazu]