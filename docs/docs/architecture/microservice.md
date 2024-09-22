## 1. Декомпозиция на микросервисы
   Основываясь на функциональных блоках и доменах, определенных ранее (управление отоплением, мониторинг температуры), предлагаем следующую структуру микросервисов:

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
Обеспечивает маршрутизацию запросов к нужным микросервисам и безопасность.
* Message Broker (Kafka)
Используется для асинхронного взаимодействия микросервисов, передачи сообщений между сервисами, например, уведомления о температурных событиях.
PostgreSQL База данных
Хранение данных пользователей, настроек устройств, журналов температуры.
## 2. Определение взаимодействия
   Основное взаимодействие между микросервисами будет происходить через следующие компоненты:

API Gateway: Обрабатывает внешние запросы и маршрутизирует их в нужные микросервисы.
Kafka: Используется для передачи событий между микросервисами (например, события изменения температуры или обновления состояния устройств).
База данных: Каждая служба работает с отдельной базой данных (паттерн базы данных на микросервис), однако Kafka будет использоваться для синхронизации между микросервисами.
## 3. Визуализация архитектуры
   C4 — Уровень контейнеров (Containers)

Создадим диаграмму контейнеров, которая отображает, как микросервисы взаимодействуют с внешними системами и друг с другом.

```puml
@startuml
!define RECTANGLE "rect"
!define SYSTEM "system"

title C4 Diagram - Container Level (Microservices Architecture)

actor Пользователь as User

rectangle "API Gateway" as APIGateway {
[User Service] as UserService
[Heating Control Service] as HeatingService
[Temperature Monitoring Service] as TempService
[Device Management Service] as DeviceService
[Notification Service] as NotificationService
}

cloud "Kafka" as Kafka

database "PostgreSQL БД Пользователей" as UserDB
database "PostgreSQL БД Устройств" as DeviceDB
database "PostgreSQL БД Температуры" as TempDB

User --> APIGateway : HTTP Запросы

APIGateway --> UserService : Маршрутизация к сервису пользователей
APIGateway --> HeatingService : Управление отоплением
APIGateway --> TempService : Мониторинг температуры
APIGateway --> DeviceService : Управление устройствами
APIGateway --> NotificationService : Уведомления

UserService --> UserDB : CRUD Операции
HeatingService --> DeviceDB : Команды управления устройствами
TempService --> TempDB : Хранение данных о температуре

TempService --> Kafka : Публикация данных о температуре
HeatingService --> Kafka : Подписка на обновления температуры

NotificationService --> Kafka : Публикация событий уведомлений

@enduml
```

### C4 — Уровень компонентов (Components)

На уровне компонентов детализируем, как организован один из ключевых микросервисов, например, Heating Control Service. Этот сервис управляет устройствами отопления и взаимодействует с другими микросервисами через Kafka.
```puml
@startuml
!define RECTANGLE "rect"

title C4 Diagram - Component Level (Heating Control Service)

package "Heating Control Service" {
[API] as HeatingAPI
[Command Processor] as CommandProcessor
[Device Manager] as DeviceManager
[Kafka Producer] as KafkaProducer
[Database Connector] as HeatingDB
}

actor Пользователь as User

User --> HeatingAPI : Команды управления отоплением
HeatingAPI --> CommandProcessor : Обработка команд
CommandProcessor --> DeviceManager : Управление устройствами
CommandProcessor --> KafkaProducer : Публикация событий в Kafka
CommandProcessor --> HeatingDB : Обновление состояния устройств

@enduml
```
### C4 — Уровень кода (Code)

На уровне кода можно показать, как реализован, например, Command Processor в микросервисе Heating Control Service. Ниже приведена упрощенная UML-диаграмма классов.

```puml
@startuml

class CommandProcessor {
+processCommand(command: HeatingCommand)
}

class HeatingCommand {
+deviceId: String
+temperature: Double
+execute()
}

class DeviceManager {
+setTemperature(deviceId: String, temperature: Double)
}

class KafkaProducer {
+publish(event: HeatingEvent)
}

CommandProcessor --> HeatingCommand
CommandProcessor --> DeviceManager
CommandProcessor --> KafkaProducer

@enduml
```