## 1. Декомпозиция на микросервисы
* User Service (Сервис Пользователей) Аутентификация и авторизация пользователей. Управление учетными записями пользователей и настройками.
* Heating Control Service (Сервис Управления Отоплением) Управление устройствами отопления (включение/выключение, установка температуры).
Обработка команд пользователей и отправка их на устройства.
* Temperature Monitoring Service (Сервис Мониторинга Температуры)
Получение данных с датчиков температуры.
Хранение и предоставление данных пользователям для мониторинга текущей температуры.
* Device Management Service (Сервис Управления Устройствами)
Управление устройствами IoT (регистрация устройств, обновление прошивки, диагностика).
* Notification Service (Сервис Уведомлений)
Отправка уведомлений пользователям (например, о выходе температуры за заданные рамки).
* API Gateway
Центральная точка доступа для всех внешних запросов.
* Message Broker (Kafka)
Используем для асинхронного взаимодействия микросервисов, передачи сообщений между сервисами.
## 2. Определение взаимодействия
API Gateway: Обрабатывает внешние запросы и маршрутизирует их в нужные микросервисы.
Kafka: Используется для передачи событий между микросервисами (например, события изменения температуры или обновления состояния устройств).
База данных: Каждая служба работает с отдельной базой данных (паттерн базы данных на микросервис), однако Kafka будет использоваться для синхронизации между микросервисами.
## 3. Визуализация архитектуры
   C4 — Уровень контейнеров (Containers)
```puml
@startuml
!define RECTANGLE "rect"
!define SYSTEM "system"

title C4 Diagram - Container Level (Microservices Architecture)

!include C4_Component.puml
!include C4_Context.puml
!include C4_Container.puml

title System Context Diagram - Управление отоплением

Person(user, "Пользователь", "Пользователь системы")
Person(iot, "Датчики", "Умные устройства", $sprite="robots")

Container(APIGateway,"Шлюз")
Container(UserService,"Сервис пользователей")
Container(HeatingService,"Управление отоплением")
Container(TempService,"Мониторинг температуры")
Container(DeviceService,"Управление устройствами")
Container(NotificationService,"Сервис уведомлений")

ContainerQueue(Kafka,"Kafka")

ContainerDb(UserDB, "База данных", "PostgreSQL", "PostgreSQL БД Пользователей", $sprite="pgsql_server")
ContainerDb(DeviceDB, "База данных", "PostgreSQL", "БД Устройств", $sprite="pgsql_server")
ContainerDb(TempDB, "База данных", "PostgreSQL", "БД Температуры", $sprite="pgsql_server")

Rel_D(user, APIGateway, "Запросы")
Rel_D(iot, APIGateway, "Запросы")

Rel(APIGateway, UserService, "Запросы")
Rel(APIGateway, HeatingService, "Запросы")
Rel(APIGateway, TempService, "Запросы")
Rel(APIGateway, DeviceService, "Запросы")

Rel(UserService, UserDB, "CRUD Операции")
Rel(DeviceService, DeviceDB, "CRUD Операции")
Rel(HeatingService, DeviceService, "Команды управления устройствами")
Rel(TempService, TempDB, "Хранение данных о температуре")
Rel(TempService, Kafka, "Публикация данных о температуре")
Rel(HeatingService, Kafka, "Подписка на обновления температуры")
Rel(Kafka,NotificationService, "Отправка уведомлений")

@enduml
```

### C4 — Уровень компонентов (Components)

Heating Control Service. Этот сервис управляет устройствами отопления и взаимодействует с другими микросервисами через Kafka.
```puml
@startuml
!define RECTANGLE "rect"

title C4 Diagram - Component Level (Heating Control Service)

!include C4_Component.puml
!include C4_Context.puml
!include C4_Container.puml

Person(user, "Пользователь", "Пользователь сервиса")

Container_Boundary(HeatingControlService, "Сервис управление отоплением") {
  Container(HeatingAPI, "API", "")
  Container(CommandProcessor, "Метод обработки", "")
  Container(DeviceClient, "Клиент сервиса устройств", "")
  Container(KafkaProducer, "Продьюсер в кафку", "")
  Container(HeatingDB, "DbLink в базу данных", "")
}

Rel_D(user, HeatingAPI, "Команды управления отоплением")
Rel(HeatingAPI, CommandProcessor, "Обработка команд")
Rel(CommandProcessor, DeviceClient, "Вызов API для устройств")
Rel(CommandProcessor, KafkaProducer, "Публикация событий в Kafka")
Rel(CommandProcessor, HeatingDB, "Обновление состояния устройств")

@enduml
```
### C4 — Уровень кода (Code)

Command Processor в микросервисе Heating Control Service.

```puml
@startuml

!include C4_Component.puml
!include C4_Context.puml
!include C4_Container.puml

class CommandProcessor {
+processCommand(command: HeatingCommand)
}

class HeatingCommand {
+id: long
+temperature: Double
+execute()
}

class DeviceClient {
+setTemperature(id: long, temperature: Double)
}

class KafkaProducer {
+publish(event: HeatingEvent)
}

CommandProcessor --> HeatingCommand
CommandProcessor --> DeviceClient
CommandProcessor --> KafkaProducer

@enduml
```