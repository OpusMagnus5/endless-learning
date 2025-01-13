# Docker

## Budowanie obrazu

- `docker images` - zwraca liste obrazów
- `docker image rm [id obrazu]` - usuwa obraz
- `docker build . -t [nazwa:tag]` - buduje obraz za pomocą Dockerfile w bieżącym katalogu

## Minikube i docker

- `eval $(minikube docker-env)` - używanie dockera z mimikube