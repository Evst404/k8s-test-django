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

Создайте секрет для Django (подставьте свой SECRET_KEY и DATABASE_URL):

```sh
kubectl apply -f django-secret.yaml -n edu-aleksandr-evstigneev
```

Подсказка: DATABASE_URL можно взять из секрета `postgres` (dsn или собрать из host/port/user/password).

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

## Как задеплоить на prod

1) Обновите Docker-образ и пушьте его в Docker Hub:

```sh
cd backend_main_django
COMMIT_SHA=$(git rev-parse --short HEAD)
docker build -t DOCKERHUB_USER/evst404-k8s-test-django:${COMMIT_SHA} .
docker push DOCKERHUB_USER/evst404-k8s-test-django:${COMMIT_SHA}
```

2) Обновите тег в `django-deployment.yaml` и примените манифесты:

```sh
kubectl apply -f django-secret.yaml -n edu-aleksandr-evstigneev
kubectl apply -f django-service.yaml -n edu-aleksandr-evstigneev
kubectl apply -f django-deployment.yaml -n edu-aleksandr-evstigneev
```

3) Проверьте rollout и логи:

```sh
kubectl rollout status deployment/django-web -n edu-aleksandr-evstigneev
kubectl logs deployment/django-web -n edu-aleksandr-evstigneev
```

4) Запустите management-команды:

```sh
kubectl exec -it deployment/django-web -n edu-aleksandr-evstigneev -- python manage.py migrate
kubectl exec -it deployment/django-web -n edu-aleksandr-evstigneev -- python manage.py createsuperuser
```

5) Проверка доступа через port-forward:

```sh
kubectl port-forward service/django 8080:80 -n edu-aleksandr-evstigneev
```
