# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```


## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```


## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```


## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)


Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию 

```bash
cd terraform
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```


## Delete minikube

```bash
minikube delete
```

# Задание 1 
Смотреть через MkDocs. Установить плагины из уроков.

# Задание 2.1

https://github.com/shdwIsaac/DeviceService
https://github.com/shdwIsaac/TelemetryService

Собирать образы через docker build -t <servicename> .

Использовать docker compose в сервисе DeviceService что б развернуть сервисы

Для пуша в ContainerRegistry

```bash
docker login ghcr.io
docker tag <servicename> ghcr.io/<your-github-username>/<servicename>:latest
docker push ghcr.io/<your-github-username>/myservice:latest
```

# Задание 2.2

docker compose up -d Для кафки файл ниже

```yaml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```
Создание топика

```bash
docker exec -it kafka /bin/bash
kafka-topics --create --topic telemetry-data --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

docker compose up -d Для API GATEWAY файл ниже

```yaml
version: '3'
services:
  kong:
    image: kong:latest
    environment:
      KONG_DATABASE: "off"
      KONG_DECLARATIVE_CONFIG: /kong/kong.yml
    volumes:
      - ./kong.yml:/kong/kong.yml
    ports:
      - "8000:8000"
      - "8443:8443"
      - "8001:8001"
      - "8444:8444"
```

настройка

```yaml
_format_version: "2.1"
services:
- name: device-management-service
  url: http://device-management:8080
  routes:
    - name: device
      paths:
        - /devices
- name: telemetry-service
  url: http://telemetry-management:8080
  routes:
    - name: telemetry
      paths:
        - /telemetry
```

