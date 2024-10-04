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
Нужно установить Helm!!! https://helm.sh/docs/intro/install/

choco install kubernetes-helm

cd charts
cd smart-home-monolith
helm dependency update

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

Удалить .terraform.lock.hcl и другие такие файлы что б сделать terraform init а потом уже apply


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

Картинки отоброжаются только в IDE JetBrains 

![img.png](img.png)

# Задание 1 
Смотреть через MkDocs. Установить плагины из уроков.

Через терминал
```bash
cd docs
mkdocs build
mkdocs serve
```


# Задание 2.1

После развертывания по инструкции выше до задания 1

Пробросить на внешний порт
kubectl port-forward smart-home-monolith-postgresql-0 5433:5432

подключится к бд, логин и пароль можно найти в файле values.yaml

Сделал по 1 строке записи в каждой таблице

База данных 
![img_1.png](img_1.png)

Ответ с монолита

![img_2.png](img_2.png)

на этом этапе переобулся на docker compose

подключение к бд через порт localhost:5433 смотреть кредиты в env 

надо положить в бд какие-нить данные. Можно воспользоваться pgAdmin

https://github.com/shdwIsaac/DeviceService
https://github.com/shdwIsaac/TelemetryService

Собирать образы через docker build -t <servicename> .

подменить на свои image в файле compose (находится здесь).

docker compose up -d

Пример вызова
![img_3.png](img_3.png)

![img_4.png](img_4.png)

Для пуша в ContainerRegistry

```bash
docker login ghcr.io
docker tag <servicename> ghcr.io/<your-github-username>/<servicename>:latest
docker push ghcr.io/<your-github-username>/myservice:latest
```

# Задание 2.2

Kong в docker compose

http://localhost:8000/api/heating/1
настройка на этот адрес в kong.yml

![img_5.png](img_5.png)

![img_6.png](img_6.png)
