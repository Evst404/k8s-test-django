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

## Применить Service

```sh
kubectl apply -f service.yaml -n edu-aleksandr-evstigneev
```

## Применить Pod

```sh
kubectl apply -f pod.yaml -n edu-aleksandr-evstigneev
```
