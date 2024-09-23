## 1. Выбор типов API
   REST API: Подходит для получения информации о состоянии устройства и управлении устройствами. Например, синхронные запросы для включения/выключения устройства или получения последних данных телеметрии.
   AsyncAPI (Kafka): Используем для асинхронного взаимодействия, когда данные телеметрии передаются через шину сообщений в реальном времени, например, для передачи новых измерений температуры.
## 2. Проектирование API для микросервиса «Управление устройствами»
### 1. Получение информации об устройстве

- Эндпойнт: api/v1/devices/{id:long}
- Метод: GET
- Описание: Возвращает подробную информацию о конкретном устройстве по его ID.

Пример запроса:

```bash
GET api/v1/devices/1
```

Пример ответа:

```json
{
"id": 1,
"type": "thermostat",
"serial_number": "ABC1",
"status": "active",
"house_id": 67890,
"created_at": "2023-09-01T10:00:00Z"
}
```
Коды ответа:

- 200 — Успех
- 404 — Устройство не найдено
- 500 — Ошибка сервера

### 2. Обновление состояния устройства

- Эндпойнт: api/v1/devices/{id:long}/status
- Метод: PUT
- Описание: Позволяет изменить состояние устройства (например, включить/выключить).

Пример запроса:

```bash
PUT api/v1/devices/1/status
```
Тело запроса:

```json
{
"status": "off"
}
```
Пример ответа:

```json
{
"id": 12345,
"status": "off",
"updated_at": "2023-09-20T12:30:00Z"
}
```
Коды ответа:

- 200 — Успех
- 404 — Устройство не найдено
- 500 — Ошибка сервера

### 3. Отправка команды устройству

- Эндпойнт: api/v1/devices/{id:long}/commands
- Метод: POST
- Описание: Отправляет команду устройству (например, установить температуру).
Пример запроса:

```bash
POST /devices/{id:long}/commands
```
Тело запроса:

```json
{
"command": "set_temperature",
"value": 22
}
```
Пример ответа:

Swagger Response Status 200

Коды ответа:

- 200 — Команда успешно отправлена
- 400 — Неверный формат команды
- 404 — Устройство не найдено
- 500 — Ошибка сервера
## 3. Проектирование API для микросервиса «Телеметрия»
### 1. Получение последних данных телеметрии

Эндпойнт: api/v1/devices/{device_id:long}/telemetry/latest
Метод: GET
Описание: Возвращает последнее полученное значение телеметрии для устройства.
Пример запроса:

```bash
GET api/v1/devices/1/telemetry/latest
```
Пример ответа:

```json
{
"device_id": 1,
"temperature": 21.5,
"timestamp": "2023-09-20T12:30:00Z"
}
```
Коды ответа:

- 200 — Успех
- 404 — Данные телеметрии не найдены
- 500 — Ошибка сервера
### 2. Получение исторических данных телеметрии

Эндпойнт: api/v1/devices/{device_id:long}/telemetry
Метод: GET
Описание: Возвращает исторические данные телеметрии для устройства за определённый период времени.
Пример запроса:

```bash
GET api/v1/devices/1/telemetry?start=2023-09-19T00:00:00Z&end=2023-09-20T00:00:00Z
```
Пример ответа:

```json
[
{
"device_id": 1,
"temperature": 22.1,
"timestamp": "2023-09-19T10:00:00Z"
},
{
"device_id": 2,
"temperature": 21.5,
"timestamp": "2023-09-19T12:00:00Z"
}
]
```
Коды ответа:

- 200 — Успех
- 400 — Неверные параметры запроса
- 404 — Данные телеметрии не найдены
- 500 — Ошибка сервера

## 4. Асинхронное взаимодействие через Kafka (AsyncAPI)
Топик: telemetry-data
Описание события: Передача новых данных телеметрии.

```json
{
"device_id": 1,
"module_id": 231245,
"temperature": 21.7,
"timestamp": "2023-09-20T12:45:00Z"
}
```

## 5. Документирование API с использованием Swagger/OpenAPI

```yaml
openapi: 3.0.0
info:
   title: Device Management API
   version: 1.0.0
paths:
   '/devices/{id}':
      get:
         summary: Get device information
         parameters:
            - name: id
              in: path
              required: true
              schema:
                 type: number
              description: ID of the device
         responses:
            '200':
               description: Success
               content:
                  application/json:
                     schema:
                        type: object
                        properties:
                           id:
                              type: number
                           type:
                              type: string
                           serial_number:
                              type: string
                           status:
                              type: string
                           house_id:
                              type: number
                           created_at:
                              type: string
                              format: date-time
            '404':
               description: Device not found
            '500':
               description: Server error
```