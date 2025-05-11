# Распределенный вычислитель арифметических выражений

## Описание проекта

Пользователь хочет считать арифметические выражения. Он вводит строку `2 + 2 * 2` и хочет получить в ответ `6`. Но наши операции сложения и умножения (также деление и вычитания) выполняются "очень-очень" долго. Поэтому вариант, при котором пользователь делает http-запрос и получает в качестве ответа результат, невозможна. Более того: вычисление каждой такой операции в нашей "альтернативной реальности" занимает "гигантские" вычислительные мощности. Соответственно каждое действие мы должны уметь выполнять отдельно и масштабировать эту систему можем добавлением вычислительных мощностей в нашу систему в виде новых "машин". Поэтому пользователь может с какой-то периодичностью уточнять у сервера "не посчиталось ли выражение"? Если выражение наконец будет вычислено - то он получит результат. Помните, что некоторые части арифметического выражения можно вычислять параллельно.

## Установка и настройка

Проект ставится в несколько шагов:

0. Устанавливаем Docker (если нет)
1. Клонируем репозиторий
2. Переходим в папку с программой в командной строке
3. Запускаем Docker для создания образов и запуска контейнеров

### 2.1. Установка Docker

Для работы с сервисом необходимо установить на компьютер Docker

Версии для Mac, Windows, Linux доступны по следующему адресу:
https://docs.docker.com/get-docker/

1. Скачиваем установочный файл под вашу операционную систему
2. Запускаем / Устанавливаем / Перезагружаем компьютер
### 2.2. Клонирование репозитория

Клонируем себе на компьютер репозиторий. Набираем в терминале:
```bash
git clone https://github.com/psych0kid/dcalculate.git
```

Переходим в папку с проектом на компьютере
```bash
cd dcalculate
```
## 2.3. Запуск проекта

Для создания образов и запуска контейнеров набираем:
```bash
docker-compose up --build
```
## 3. Пример использования

В проекте реализован GUI. Удобнее всего посмотреть работу из него. Но первым пунктом описания сделаны примеры запросов, через `curl`. 
### 3.1. Работа в GUI
<a name="работа-в-GUI"></a>
После запуска проекта он доступен по адресу:
```bash
http://localhost:3000/
```
### 3.2. Регистрация пользователя

Страница отвечает на GET-запрос, приветствием. Пример запроса:
```bash
curl -X GET -H "Content-Type: application/json" http://127.0.0.1:8080/registration
```

Отправляем POST-запрос по адресу:
```bash
http://localhost:8080/registration
```
Тело запроса в формате JSON, структура:
```bash
{
  "login": "user",
  "password": "password"
}
```

Пример запроса:
```bash
curl -X POST -H "Content-Type: application/json" -d "{\"login\": \"user\", \"password\":\"password\"}" http://127.0.0.1:8080/registration
```

### 3.3. Авторизация. Получение токена

Страница отвечает на GET-запрос, приветствием. Пример запроса:
```bash
curl -X GET -H "Content-Type: application/json" http://127.0.0.1:8080/login
```

Отправляем POST-запрос по адресу:
```bash
http://localhost:8080/login
```

Пример запроса:
```bash
curl -X POST -H "Content-Type: application/json" -d "{\"login\": \"user\", \"password\":\"password\"}" http://127.0.0.1:8080/login
```

В ответ сервер присылает Token. Пример токена:
```bash
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTMyNzYwNzcsImlhdCI6MTcxMzI3NTQ3NywibG9naW4iOiJ1c2VyMSIsIm5iZiI6MTcxMzI3NTQ3N30.zseZouXKkOlNL7X7mmOrVIFtu-Ekp2nb0lZ7Wnw_SVE"}
```
### 3.4. Добавление вычисления арифметического выражения.

Отправляем POST-запрос по адресу:
```bash
http://localhost:8080/add-expression
```

Тело запроса в формате JSON, структура:
```json
{"text": "2+2+2"}
```

Пример запроса:
```bash
curl -X POST -H "Content-Type: application/json" -d "{\"text\": \"3 + 1\"}" http://127.0.0.1:8080/add-expression
```
### Получение списка выражений со статусами

Отправляем GET-запрос по адресу:
```bash
http://localhost:8080/get-operations
```

Ответ получаемый от сервера в формате JSON, структура:
```json
{
    "data": [
        {
            "unique_id": "abb1f26f-2014-43f6-9f88-22fd6bf7aedc",
            "query_text": "2 + 1 + 3 + 1",
            "creation_time": {
                "Time": "2025-03-05T21:17:24.764203Z",
                "Valid": true
            },
            "completion_time": {
                "Time": "2025-03-05T21:17:36.870463Z",
                "Valid": true
            },
            "execution_time": "00:00:12.10626",
            "server_name": {
                "String": "worker3",
                "Valid": true
            },
            "result": {
                "String": "7",
                "Valid": true
            },
            "status": "Done"
        }
    ],
    "total_items": 4,
    "total_pages": 1,
    "current_page": 1,
    "items_per_page": 5
}
```

Ответ содержит информацию о пагинации, которую мы высчитываем на стороне backend и возвращаем для frontend.

Данные берутся из БД Postgresql - таблица requests 

Так как во второй части задания у нас появилась авторизация. Простой GET-запрос, будет возвращать `Unauthorized`

Пример запроса без авторизации:
```bash
curl -X GET -H "Content-Type: application/json" http://127.0.0.1:8080/get-operations
```

Пример запроса с авторизацией
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTMyNzY1MTEsImlhdCI6MTcxMzI3NTkxMSwibG9naW4iOiJ1c2VyMSIsIm5iZiI6MTcxMzI3NTkxMX0.GL1bbHRijfDo1BortkJMCMrxQwRxexHP8sHEX98bDaA" http://localhost:8080/get-operations
```
Так как у этого запроса есть пагинация, следует указать как получать следующие страницы:
```bash
http://localhost:8080/get-operations?page=2
```
### Получение значения выражения по его идентификатору.

Информацию о каждой операции также можно получить по GET-запросу по адресу:
```bash
http://localhost:8080/get-request-by-id/{uuid}
```

Ответ получаемый от сервера в формате JSON, структура:
```json
{
    "unique_id":"abb1f26f-2014-43f6-9f88-22fd6bf7aedc",
    "query_text":"2 + 1 + 3 + 1",
    "server_name":"worker3",
    "result":"7",
    "status":"Done"
}
```

Пример запроса:
```bash
curl -X GET -H "Content-Type: application/json" http://localhost:8080/get-request-by-id/abb1f26f-2014-43f6-9f88-22fd6bf7aedc
```
### Получение списка доступных операций со временем их выполнения.

Данный эндпойнт умеет обрабатывать как GET так и POST запросы.
Если мы хотим узнать настройки воркеров, делаем GET-запрос по адресу:
```bash
http://localhost:8080/setup-workers
```

Ответ получаемый от сервера в формате JSON, структура:
```json
[
    {
        "worker_name": "worker2",
        "last_task": "70d9df9f-84ec-426a-b408-31fe7fe8380f",
        "status": "ready",
        "last_timeout_setup": "2025-03-05T21:10:21.870463Z",
        "current_timeout": 11
    },
    {
        "worker_name": "worker3",
        "last_task": "abb1f26f-2014-43f6-9f88-22fd6bf7aedc",
        "status": "ready",
        "last_timeout_setup": "2025-03-05T21:11:37.870463Z",
        "current_timeout": 12
    },
    {
        "worker_name": "worker1",
        "last_task": "f3715129-63f5-44a6-ac42-adbfb186ba60",
        "status": "ready",
        "last_timeout_setup": "2025-03-05T21:16:31.870463Z",
        "current_timeout": 10
    }
]
```

Пример GET - запроса
```bash
curl -X GET -H "Content-Type: application/json" http://localhost:8080/setup-workers
```

Если мы хотим поменять настройку у воркера, делаем POST-запрос по адресу:
```bash
http://localhost:8080/setup-workers
```

Тело запроса в формате JSON, структура:
```json
{
 "worker_name":"worker1",
 "timeout_data":10
}
```

Пример POST - запроса
```bash
curl -X POST -H "Content-Type: application/json" -d "{\"worker_name\": \"worker1\", \"timeout_data\": 19}" http://localhost:8080/setup-workers
```

# The end. Good luck!