# API Endpoints для мобильного приложения "Места"

Этот документ описывает REST API эндпоинты для взаимодействия фронтенда и бэкенда мобильного приложения. Все эндпоинты используют JSON для запросов и ответов. Аутентификация осуществляется через JWT-токен в заголовке `Authorization: Bearer <token>` для защищенных запросов.

## Общие правила
- **Базовый URL**: `https://api.example.com/v1`
- **Формат данных**: JSON
- **Коды ошибок**: Стандартные HTTP (200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 404 Not Found, 500 Internal Server Error)
- **Пагинация**: Для списков используйте параметры `page` (номер страницы, начиная с 1) и `limit` (количество элементов, по умолчанию 20). Ответ включает `meta: { totalCount, page, limit }`
- **Кеширование**: GET-запросы кешируются на 5 минут (`Cache-Control: max-age=300`)

---

## 1. Places (Места)

### GET /places
Получить список мест с фильтрами.

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| category | string | Опционально | ID категории (например, "food", "entertainment") |
| lat | float | Опционально | Широта для поиска по радиусу |
| lon | float | Опционально | Долгота для поиска по радиусу |
| radius | integer | Опционально | Радиус поиска в метрах (по умолчанию 1000) |
| features | string | Опционально | Список особенностей, разделенных запятой (например, `wheelchair,parking,wifi`) |
| minRating | number | Опционально | Минимальный рейтинг для фильтра (например, 4.5) |
| page | integer | Опционально | Номер страницы (по умолчанию 1) |
| limit | integer | Опционально | Количество элементов на странице (по умолчанию 20, максимум 100) |
| sort | string | Опционально | Сортировка: "rating", "distance", "name", "newest" |

**Пример запроса**:
```
GET /v1/places?category=food&lat=55.7558&lon=37.6173&radius=1000&features=wheelchair,parking&minRating=4.0&page=1&limit=10&sort=rating
Authorization: Bearer <token> (если требуется для персонализации)
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "id": 1,
      "name": "Кафе 'Уют'",
      "category": "food",
      "coordinates": { "lat": 55.7558, "lon": 37.6173 },
      "rating": 4.5,
      "address": "ул. Ленина, 10",
      "isOpen": true
    },
    {
      "id": 2,
      "name": "Ресторан 'Вкус'",
      "category": "food",
      "coordinates": { "lat": 55.7560, "lon": 37.6180 },
      "rating": 4.2,
      "address": "ул. Пушкина, 5",
      "isOpen": false
    }
  ],
  "meta": {
    "totalCount": 150,
    "page": 1,
    "limit": 10
  }
}
```

**Коды ошибок**:
- **400 Bad Request** - Неверные параметры запроса
- **401 Unauthorized** - Требуется аутентификация
- **422 Unprocessable Entity** - Некорректное значение параметра
- **429 Too Many Requests** - Превышен лимит запросов
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Неверные координаты (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Параметры lat и lon должны быть в диапазоне от -90 до 90 для lat и от -180 до 180 для lon",
  "details": {
    "field": "lat",
    "value": "95.5",
    "issue": "значение вне диапазона"
  }
}
```

*Пример 2: Некорректное значение limit (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Значение limit не может быть больше 100",
  "details": {
    "field": "limit",
    "value": 150,
    "constraint": "максимум 100"
  }
}
```

*Пример 3: Превышен лимит запросов (429 Too Many Requests)*
```json
{
  "error": "Too Many Requests",
  "code": 429,
  "message": "Вы отправили слишком много запросов. Попробуйте позже",
  "retryAfter": 60
}
```

---

### GET /places/{id}
Получить детали места по ID.

**Таблица параметров пути**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| id | integer | Обязательно | Уникальный идентификатор места |

**Пример запроса**:
```
GET /v1/places/1
```

**Пример ответа (200 OK)**:
```json
{
  "id": 1,
  "name": "Кафе 'Уют'",
  "description": "Уютное кафе с домашней кухней",
  "category": "food",
  "coordinates": { "lat": 55.7558, "lon": 37.6173 },
  "rating": 4.5,
  "address": "ул. Ленина, 10",
  "phone": "+7 (495) 123-45-67",
  "photos": ["https://example.com/photo1.jpg"],
  "workingHours": {
    "mon": "9:00-18:00",
    "tue": "9:00-18:00",
    "wed": "9:00-18:00",
    "thu": "9:00-18:00",
    "fri": "9:00-20:00",
    "sat": "10:00-20:00",
    "sun": "закрыто"
  },
  "isOpen": true
}
```

**Коды ошибок**:
- **404 Not Found** - Место с указанным ID не найдено
- **400 Bad Request** - ID не является числом
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Место не найдено (404 Not Found)*
```json
{
  "error": "Not Found",
  "code": 404,
  "message": "Место с ID 999 не найдено",
  "details": {
    "resourceType": "place",
    "id": 999
  }
}
```

*Пример 2: Неверный ID (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "ID должен быть числом",
  "details": {
    "field": "id",
    "value": "abc",
    "expected": "integer"
  }
}
```

---

### GET /places/search
Поиск мест по запросу.

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| q | string | Обязательно | Поисковый запрос (минимум 1, максимум 100 символов) |
| category | string | Опционально | ID категории для фильтра поиска |
| lat | float | Опционально | Широта для поиска по радиусу |
| lon | float | Опционально | Долгота для поиска по радиусу |
| radius | integer | Опционально | Радиус поиска в метрах (по умолчанию 1000) |
| page | integer | Опционально | Номер страницы (по умолчанию 1) |
| limit | integer | Опционально | Количество элементов на странице (по умолчанию 20) |

**Пример запроса**:
```
GET /v1/places/search?q=кафе&category=food&lat=55.7558&lon=37.6173
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "id": 1,
      "name": "Кафе 'Уют'",
      "category": "food",
      "coordinates": { "lat": 55.7558, "lon": 37.6173 },
      "rating": 4.5,
      "address": "ул. Ленина, 10"
    }
  ],
  "meta": {
    "totalCount": 1,
    "page": 1,
    "limit": 20
  }
}
```

**Коды ошибок**:
- **400 Bad Request** - Параметр q отсутствует или пуст
- **422 Unprocessable Entity** - Некорректное значение параметра
- **429 Too Many Requests** - Превышен лимит запросов
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Отсутствует обязательный параметр q (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Параметр q обязателен",
  "details": {
    "field": "q",
    "issue": "отсутствует"
  }
}
```

*Пример 2: Поисковый запрос слишком длинный (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Длина поискового запроса не может быть больше 100 символов",
  "details": {
    "field": "q",
    "length": 150,
    "maxLength": 100
  }
}
```

---

### GET /places/{id}/reviews
Получить отзывы места.

**Таблица параметров пути**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| id | integer | Обязательно | ID места |

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| page | integer | Опционально | Номер страницы (по умолчанию 1) |
| limit | integer | Опционально | Количество отзывов (по умолчанию 20, максимум 100) |
| sort | string | Опционально | Сортировка: "newest", "oldest", "helpful" (по умолчанию "newest") |

**Пример запроса**:
```
GET /v1/places/1/reviews?page=1&limit=5&sort=newest
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "id": 101,
      "userId": 5,
      "userName": "Алексей",
      "rating": 5,
      "text": "Отличное кафе, рекомендую!",
      "createdAt": "2023-10-01T12:00:00Z"
    },
    {
      "id": 102,
      "userId": 6,
      "userName": "Мария",
      "rating": 4,
      "text": "Хорошее место, но дорого",
      "createdAt": "2023-10-02T14:30:00Z"
    }
  ],
  "meta": {
    "totalCount": 25,
    "page": 1,
    "limit": 5
  }
}
```

**Коды ошибок**:
- **404 Not Found** - Место не найдено
- **400 Bad Request** - Неверные параметры запроса
- **422 Unprocessable Entity** - Некорректное значение параметра
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Место не найдено (404 Not Found)*
```json
{
  "error": "Not Found",
  "code": 404,
  "message": "Место с ID 999 не найдено",
  "details": {
    "resourceType": "place",
    "id": 999
  }
}
```

*Пример 2: Неверное значение limit (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Значение limit не может быть больше 100",
  "details": {
    "field": "limit",
    "value": 150
  }
}
```

---

### POST /places/{id}/reviews
Добавить отзыв к месту (требует аутентификации).

**Таблица параметров пути**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| id | integer | Обязательно | ID места |

**Таблица параметров тела запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| rating | integer | Обязательно | Оценка от 1 до 5 |
| text | string | Обязательно | Текст отзыва (минимум 10, максимум 1000 символов) |

**Пример запроса**:
```
POST /v1/places/1/reviews
Authorization: Bearer <token>
Content-Type: application/json

{
  "rating": 5,
  "text": "Отлично! Вкусное кофе и уютная обстановка!"
}
```

**Пример ответа (201 Created)**:
```json
{
  "id": 103,
  "userId": 7,
  "placeId": 1,
  "rating": 5,
  "text": "Отлично! Вкусное кофе и уютная обстановка!",
  "createdAt": "2023-10-03T10:00:00Z"
}
```

**Коды ошибок**:
- **400 Bad Request** - Отсутствуют обязательные параметры или неверный формат
- **401 Unauthorized** - Отсутствует или неверный токен аутентификации
- **404 Not Found** - Место не найдено
- **422 Unprocessable Entity** - Некорректное значение параметра
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Отсутствует обязательный параметр (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Параметр rating обязателен",
  "details": {
    "field": "rating",
    "issue": "отсутствует"
  }
}
```

*Пример 2: Неверная оценка (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Rating должен быть числом от 1 до 5",
  "details": {
    "field": "rating",
    "value": 10,
    "minValue": 1,
    "maxValue": 5
  }
}
```

*Пример 3: Отсутствует токен аутентификации (401 Unauthorized)*
```json
{
  "error": "Unauthorized",
  "code": 401,
  "message": "Требуется аутентификация. Отсутствует заголовок Authorization"
}
```

---

## 2. Routes (Маршруты)

### GET /routes
Получить варианты маршрутов от точки A до точки B с опциональными промежуточными точками.

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| origin | string | Обязательно | Координаты начала в формате `lat,lon` или адрес |
| destination | string | Обязательно | Координаты конца в формате `lat,lon` или адрес |
| waypoints | string | Опционально | Список промежуточных точек, разделенных `\|`, например `lat1,lon1\|lat2,lon2` |
| mode | string | Опционально | Режим передвижения: `car`, `bike`, `walk` (по умолчанию `car`) |
| alternatives | boolean | Опционально | Вернуть несколько вариантов маршрутов (по умолчанию `true`) |

**Пример запроса**:
```
GET /v1/routes?origin=59.9410,30.3609&destination=59.9630,30.3110&mode=car&alternatives=true
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "id": "route-1",
      "mode": "car",
      "duration": 1380,
      "distance": 10000,
      "waypointsCount": 0,
      "summary": "23 мин, 10 км",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [30.3609, 59.9410],
          [30.3110, 59.9630]
        ]
      }
    },
    {
      "id": "route-2",
      "mode": "car",
      "duration": 2520,
      "distance": 13000,
      "waypointsCount": 2,
      "summary": "42 мин, 13 км, через 2 места",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [30.3609, 59.9410],
          [30.3500, 59.9500],
          [30.3300, 59.9600],
          [30.3110, 59.9630]
        ]
      }
    }
  ]
}
```

**Коды ошибок**:
- **400 Bad Request** - Отсутствуют обязательные параметры или неверный формат координат
- **404 Not Found** - Маршрут не найден или адрес не распознан
- **422 Unprocessable Entity** - Некорректное значение параметра
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Отсутствует обязательный параметр (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Параметры origin и destination обязательны",
  "details": {
    "field": "origin",
    "issue": "отсутствует"
  }
}
```

*Пример 2: Неверный формат координат (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Координаты должны быть в формате lat,lon",
  "details": {
    "field": "origin",
    "value": "not-coordinates",
    "format": "lat,lon"
  }
}
```

*Пример 3: Маршрут не найден (404 Not Found)*
```json
{
  "error": "Not Found",
  "code": 404,
  "message": "Не удалось найти маршрут между указанными точками"
}
```

---

### GET /routes/preview
Получить краткую информацию о маршрутах без полного геометрического пути.

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| origin | string | Обязательно | Координаты начала в формате `lat,lon` или адрес |
| destination | string | Обязательно | Координаты конца в формате `lat,lon` или адрес |
| mode | string | Опционально | Режим передвижения: `car`, `bike`, `walk` |
| alternatives | boolean | Опционально | Вернуть несколько вариантов (по умолчанию `true`) |

**Пример запроса**:
```
GET /v1/routes/preview?origin=59.9410,30.3609&destination=59.9630,30.3110&mode=bike
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "id": "route-1",
      "mode": "bike",
      "duration": 3480,
      "distance": 12000,
      "summary": "58 мин, 12 км"
    },
    {
      "id": "route-2",
      "mode": "walk",
      "duration": 6240,
      "distance": 18000,
      "summary": "1 ч 44 мин, 18 км"
    }
  ]
}
```

**Коды ошибок**:
- **400 Bad Request** - Отсутствуют обязательные параметры
- **404 Not Found** - Маршрут не найден
- **422 Unprocessable Entity** - Некорректное значение параметра
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Неверный режим передвижения (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Режим должен быть одним из: car, bike, walk",
  "details": {
    "field": "mode",
    "value": "helicopter",
    "allowedValues": ["car", "bike", "walk"]
  }
}
```

---

## 3. Categories (Категории)

### GET /categories
Получить список категорий с количеством мест.

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| (нет параметров) | - | - | - |

**Пример запроса**:
```
GET /v1/categories
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "id": 1,
      "name": "Еда",
      "count": 150
    },
    {
      "id": 2,
      "name": "Развлечения",
      "count": 80
    },
    {
      "id": 3,
      "name": "Магазины",
      "count": 200
    }
  ]
}
```

**Коды ошибок**:
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Ошибка сервера (500 Internal Server Error)*
```json
{
  "error": "Internal Server Error",
  "code": 500,
  "message": "Ошибка при получении списка категорий",
  "details": {
    "requestId": "req-12345"
  }
}
```

---

## 4. User (Пользователь)

### POST /auth/login
Вход пользователя.

**Таблица параметров тела запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| email | string | Обязательно | Email пользователя (валидный формат) |
| password | string | Обязательно | Пароль (минимум 6 символов) |

**Пример запроса**:
```
POST /v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Пример ответа (200 OK)**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 7,
    "name": "Иван",
    "email": "user@example.com"
  }
}
```

**Коды ошибок**:
- **400 Bad Request** - Отсутствуют обязательные параметры или неверный формат email
- **401 Unauthorized** - Неверный email или пароль
- **422 Unprocessable Entity** - Пароль слишком короткий
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Неверный email или пароль (401 Unauthorized)*
```json
{
  "error": "Unauthorized",
  "code": 401,
  "message": "Неверный email или пароль"
}
```

*Пример 2: Отсутствует обязательный параметр (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Параметр email обязателен",
  "details": {
    "field": "email",
    "issue": "отсутствует"
  }
}
```

*Пример 3: Неверный формат email (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Email должен быть в формате example@domain.com",
  "details": {
    "field": "email",
    "value": "invalid-email",
    "format": "email"
  }
}
```

---

### POST /auth/register
Регистрация нового пользователя.

**Таблица параметров тела запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| name | string | Обязательно | Имя пользователя (минимум 2, максимум 50 символов) |
| email | string | Обязательно | Email пользователя (валидный формат) |
| password | string | Обязательно | Пароль (минимум 6 символов) |

**Пример запроса**:
```
POST /v1/auth/register
Content-Type: application/json

{
  "name": "Иван",
  "email": "user@example.com",
  "password": "password123"
}
```

**Пример ответа (201 Created)**:
```json
{
  "id": 7,
  "name": "Иван",
  "email": "user@example.com",
  "createdAt": "2023-10-03T10:00:00Z"
}
```

**Коды ошибок**:
- **400 Bad Request** - Отсутствуют обязательные параметры или неверный формат
- **409 Conflict** - Email уже зарегистрирован
- **422 Unprocessable Entity** - Некорректное значение параметра (слишком короткий пароль, etc.)
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Email уже зарегистрирован (409 Conflict)*
```json
{
  "error": "Conflict",
  "code": 409,
  "message": "Пользователь с этим email уже зарегистрирован",
  "details": {
    "field": "email",
    "value": "user@example.com"
  }
}
```

*Пример 2: Пароль слишком короткий (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Пароль должен содержать минимум 6 символов",
  "details": {
    "field": "password",
    "length": 3,
    "minLength": 6
  }
}
```

*Пример 3: Имя слишком короткое (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Имя должно содержать минимум 2 символа",
  "details": {
    "field": "name",
    "length": 1,
    "minLength": 2
  }
}
```

---

### GET /user/favorites
Получить избранные места пользователя (требует аутентификации).

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| page | integer | Опционально | Номер страницы (по умолчанию 1) |
| limit | integer | Опционально | Количество элементов (по умолчанию 20) |

**Пример запроса**:
```
GET /v1/user/favorites?page=1&limit=10
Authorization: Bearer <token>
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "id": 1,
      "name": "Кафе 'Уют'",
      "category": "food",
      "coordinates": { "lat": 55.7558, "lon": 37.6173 },
      "rating": 4.5
    },
    {
      "id": 3,
      "name": "Парк 'Центральный'",
      "category": "entertainment",
      "coordinates": { "lat": 55.7565, "lon": 37.6190 },
      "rating": 4.0
    }
  ],
  "meta": {
    "totalCount": 5,
    "page": 1,
    "limit": 10
  }
}
```

**Коды ошибок**:
- **401 Unauthorized** - Отсутствует или неверный токен аутентификации
- **400 Bad Request** - Неверные параметры запроса
- **422 Unprocessable Entity** - Некорректное значение параметра
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Отсутствует токен аутентификации (401 Unauthorized)*
```json
{
  "error": "Unauthorized",
  "code": 401,
  "message": "Требуется аутентификация. Отсутствует заголовок Authorization"
}
```

*Пример 2: Неверный токен (401 Unauthorized)*
```json
{
  "error": "Unauthorized",
  "code": 401,
  "message": "Неверный или истекший токен аутентификации"
}
```

---

### POST /user/favorites
Добавить место в избранное (требует аутентификации).

**Таблица параметров тела запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| placeId | integer | Обязательно | ID места |

**Пример запроса**:
```
POST /v1/user/favorites
Authorization: Bearer <token>
Content-Type: application/json

{
  "placeId": 1
}
```

**Пример ответа (201 Created)**:
```json
{
  "message": "Место добавлено в избранное"
}
```

**Коды ошибок**:
- **400 Bad Request** - Отсутствует параметр placeId или неверный формат
- **401 Unauthorized** - Отсутствует или неверный токен
- **404 Not Found** - Место не найдено
- **409 Conflict** - Место уже в избранном
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Место уже в избранном (409 Conflict)*
```json
{
  "error": "Conflict",
  "code": 409,
  "message": "Это место уже добавлено в избранное",
  "details": {
    "placeId": 1
  }
}
```

*Пример 2: Место не найдено (404 Not Found)*
```json
{
  "error": "Not Found",
  "code": 404,
  "message": "Место с ID 999 не найдено",
  "details": {
    "placeId": 999
  }
}
```

---

### DELETE /user/favorites/{placeId}
Удалить место из избранного (требует аутентификации).

**Таблица параметров пути**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| placeId | integer | Обязательно | ID места |

**Пример запроса**:
```
DELETE /v1/user/favorites/1
Authorization: Bearer <token>
```

**Пример ответа (204 No Content)**:
(Пустой ответ)

**Коды ошибок**:
- **401 Unauthorized** - Отсутствует или неверный токен
- **404 Not Found** - Место не найдено или не в избранном
- **400 Bad Request** - placeId не является числом
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Место не в избранном (404 Not Found)*
```json
{
  "error": "Not Found",
  "code": 404,
  "message": "Место с ID 1 не найдено в избранном"
}
```

*Пример 2: Неверный формат ID (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "placeId должен быть числом",
  "details": {
    "placeId": "abc",
    "expected": "integer"
  }
}
```

---

## 5. Working Mode (Режим работы)

### GET /working-mode
Получить информацию о режиме работы.

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| placeId | integer | Опционально | ID места для конкретного режима. Если не указано, возвращает глобальный статус |

**Пример запроса**:
```
GET /v1/working-mode?placeId=1
```

**Пример ответа (200 OK)**:
```json
{
  "isOpen": true,
  "hours": {
    "mon": "9:00-18:00",
    "tue": "9:00-18:00",
    "wed": "9:00-18:00",
    "thu": "9:00-18:00",
    "fri": "9:00-20:00",
    "sat": "10:00-20:00",
    "sun": "закрыто"
  },
  "status": "Открыто до 18:00"
}
```

**Коды ошибок**:
- **404 Not Found** - Место не найдено
- **400 Bad Request** - placeId не является числом
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Место не найдено (404 Not Found)*
```json
{
  "error": "Not Found",
  "code": 404,
  "message": "Место с ID 999 не найдено",
  "details": {
    "placeId": 999
  }
}
```

---

## 6. Maps (Карта и кластеры)

### GET /places/clusters
Получить кластеры мест для карты.

**Таблица параметров запроса**:

| Параметр | Тип | Обязательность | Описание |
|----------|-----|---|----------|
| zoom | integer | Обязательно | Уровень зума карты (от 0 до 22) |
| bounds | string | Обязательно | Границы карты в формате "minLat,minLon,maxLat,maxLon" |

**Пример запроса**:
```
GET /v1/places/clusters?zoom=14&bounds=55.75,37.61,55.76,37.62
```

**Пример ответа (200 OK)**:
```json
{
  "data": [
    {
      "coordinates": { "lat": 55.7558, "lon": 37.6173 },
      "count": 5,
      "places": [1, 2, 3, 4, 5]
    },
    {
      "coordinates": { "lat": 55.7560, "lon": 37.6180 },
      "count": 3,
      "places": [6, 7, 8]
    }
  ]
}
```

**Коды ошибок**:
- **400 Bad Request** - Отсутствуют обязательные параметры или неверный формат
- **422 Unprocessable Entity** - Некорректное значение параметра (zoom вне диапазона, etc.)
- **500 Internal Server Error** - Ошибка сервера

**Примеры ошибок**:

*Пример 1: Отсутствует обязательный параметр (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Параметры zoom и bounds обязательны",
  "details": {
    "field": "bounds",
    "issue": "отсутствует"
  }
}
```

*Пример 2: Неверный уровень зума (422 Unprocessable Entity)*
```json
{
  "error": "Unprocessable Entity",
  "code": 422,
  "message": "Zoom должен быть числом от 0 до 22",
  "details": {
    "field": "zoom",
    "value": 25,
    "minValue": 0,
    "maxValue": 22
  }
}
```

*Пример 3: Неверный формат границ (400 Bad Request)*
```json
{
  "error": "Bad Request",
  "code": 400,
  "message": "Границы должны быть в формате minLat,minLon,maxLat,maxLon",
  "details": {
    "field": "bounds",
    "value": "invalid",
    "format": "minLat,minLon,maxLat,maxLon"
  }
}
```

---

## Стандартные коды ошибок

| Код | Описание |
|-----|----------|
| 200 | OK - Запрос выполнен успешно |
| 201 | Created - Ресурс успешно создан |
| 204 | No Content - Запрос выполнен успешно, содержимого нет |
| 400 | Bad Request - Неверный запрос (отсутствуют/неверные параметры) |
| 401 | Unauthorized - Требуется аутентификация или токен неверный |
| 404 | Not Found - Ресурс не найден |
| 409 | Conflict - Конфликт (например, дублирование) |
| 422 | Unprocessable Entity - Некорректное значение параметра |
| 429 | Too Many Requests - Превышен лимит запросов |
| 500 | Internal Server Error - Ошибка сервера |

---

## Форматы ошибок

Все ошибки возвращаются в следующем формате (где применимо):

```json
{
  "error": "Error Type",
  "code": 400,
  "message": "Описание ошибки на русском языке",
  "details": {
    "field": "имя_поля",
    "value": "переданное_значение",
    "issue": "описание проблемы",
    "constraint": "ограничение (если применимо)"
  }
}
```
