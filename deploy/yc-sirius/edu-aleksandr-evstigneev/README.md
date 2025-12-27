# Как задеплоить код

В этой папке лежат рабочие манифесты для окружения yc-sirius-dev.

## Как подготовить dev окружение

Создайте секрет с SSL-сертификатом PostgreSQL (root.crt):

```sh
kubectl apply -f pg-root-crt-secret.yaml -n edu-aleksandr-evstigneev
```

Шаблон секрета (вставьте root.crt из секрета `postgres`):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pg-root-crt
  namespace: edu-aleksandr-evstigneev
type: Opaque
stringData:
  root.crt: |
    -----BEGIN CERTIFICATE-----
    ...paste certificate...
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    ...paste certificate...
    -----END CERTIFICATE-----
```

После создания секрета можно запускать pod с уже примонтированным сертификатом:

```sh
kubectl apply -f psql-client.yaml -n edu-aleksandr-evstigneev
```

## Как собрать и опубликовать Docker-образ

1) Соберите образ из `backend_main_django`:

```sh
cd backend_main_django
COMMIT_SHA=$(git rev-parse --short HEAD)
docker build -t DOCKERHUB_USER/k8s-test-django:${COMMIT_SHA} .
```

2) Авторизуйтесь и отправьте образ в Docker Hub:

```sh
docker login
docker push DOCKERHUB_USER/k8s-test-django:${COMMIT_SHA}
```

3) Для старого коммита используйте его хэш как тег:

```sh
git checkout <old_commit>
COMMIT_SHA=$(git rev-parse --short HEAD)
docker build -t DOCKERHUB_USER/k8s-test-django:${COMMIT_SHA} .
docker push DOCKERHUB_USER/k8s-test-django:${COMMIT_SHA}
git checkout main
```

## Применить Service

```sh
kubectl apply -f service.yaml -n edu-aleksandr-evstigneev
```

## Применить Pod

```sh
kubectl apply -f pod.yaml -n edu-aleksandr-evstigneev
```
