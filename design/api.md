# API Specification - Биржа лидов

## Аутентификация

Все ручки, кроме тех, которые используются для аутентификации и авторизации, требуют JWT токен в заголовке:
```
Authorization: Bearer <token>
```


## Endpoints

### Аутентификация и пользователи

#### Регистрация пользователя

```http
POST /auth/register
```

**Request Body:**

```json
{
    "telegram_id": "string",
    "phone": "string",
    "email": "string?",
    "agency_name": "string?"
}
```

**Response:**

```json
{
    "access_token": "string"
}
```

#### Авторизация через Telegram

```http
POST /auth/telegram
```

**Request Body:**

```json
{
    "telegram_auth_data": "object"
}
```

**Response:**

```json
{
    "access_token": "string"
}
```

### Управление лидами

#### Создание лида
```http
POST /leads
```

**Request Body:**

```json
{
    "client_name": "string?",
    "requirements": {
        "location": {
            "city": "string",
            "districts": "string[]",
            "metro_stations": "string[]"
        },
        "budget": {
            "min": "number",
            "max": "number"
        },
        "area": {
            "min": "number",
            "max": "number"
        },
        "rooms": "number[]",
        "property_type": "enum [APARTMENT | HOUSE | TOWNHOUSE]",
        "property_class": "enum [ECONOMY | COMFORT | BUSINESS | ELITE]",
        "condition": "enum [ROUGH | PRE_FINISHING | FINISHING | DESIGN]",
        "payment_method": "enum [CASH | MORTGAGE | EXCHANGE]",
        "additional_requirements": "string?"
    },
    "client_contacts": "string?",
    "notes": "string?"
}
```

**Response:**

```json
{
    "lead_id": "string",
    "created_at": "timestamp"
}
```

#### Получение списка лидов пользователя

```http
GET /leads
```

Query Parameters:
- `status` (optional): `enum [ACTIVE | ARCHIVED]`

**Response:**

```json
{
    "leads": [
        {
            "lead_id": "string",
            "client_name": "string?",
            "requirements": "object",
            "status": "enum [ACTIVE | ARCHIVED]",
            "match_count": "number",
            "created_at": "timestamp"
        }
    ]
}
```

#### Обновление лида

```http
PUT /leads/{lead_id}
```

**Request Body:** аналогично созданию

#### Архивация лида

```http
POST /leads/{lead_id}/archive
```

### Управление объектами

#### Создание объекта

```http
POST /properties
```

**Request Body:**

```json
{
    "address": {
        "city": "string",
        "district": "string",
        "street": "string",
        "house_number": "string",
        "metro_station": "string?"
    },
    "details": {
        "price": "number",
        "area": "number",
        "rooms": "number",
        "floor": "number",
        "total_floors": "number",
        "year_built": "number?",
        "property_type": "enum [APARTMENT | HOUSE | APARTMENTS | TOWNHOUSE]",
        "property_class": "enum [ECONOMY | COMFORT | BUSINESS | ELITE]",
        "condition": "enum [ROUGH | PRE_FINISHING | FINISHING | DESIGN]"
    },
    "commission_share": "number",
    "photos": "string[]?",
    "description": "string?"
}
```

**Response:**

```json
{
    "property_id": "string",
    "created_at": "timestamp"
}
```

#### Получение списка объектов пользователя

```http
GET /properties
```

Query Parameters:
- `status` (optional): `enum [ACTIVE | ARCHIVED]`

**Response:**

```json
{
  "properties": [
    {
      "property_id": "string",
      "address": "object",
      "details": "object",
      "status": "string",
      "commission_share": "number",
      "match_count": "number",
      "created_at": "timestamp"
    }
  ]
}
```

#### Обновление объекта

```http
PUT /properties/{property_id}
```

**Request Body:** аналогично созданию

#### Архивация объекта

```http
PUT /properties/{property_id}/archive
```

### Мэтчинг

#### Получение рекомендаций объектов для конкретного лида

```http
GET /leads/{lead_id}/property-recommendations
```

Query Parameters:
- `page` (optional): `number`
- `limit` (optional): `number`

**Response:**

```json
{
    "recommendations": [
        {
            "property_id": "string",
            "property_data": {
                "address": "object",
                "details": "object",
                "commission_share": "number",
                "photos": "string[]"
            },
            "realtor_info": {
                "realtor_id": "string",
                "name": "string?",
                "agency": "string?"
            },
            "match_score": "number",
            "commission_details": {
                "proposed_share": "number"
            }
        }
    ],
    "total": "number",
    "page": "number"
}
```

#### Получение рекомендаций лидов для конкретного объекта

```http
GET /properties/{property_id}/lead-recommendations
```

Query Parameters:
- `page` (optional): `number`
- `limit` (optional): `number`

**Response:**

```json
{
    "recommendations": [
        {
            "lead_id": "string",
            "lead_data": {
                "client_name": "string?",
                "requirements": "object",
                "client_contacts": "string?",
                "notes": "string?"
            },
            "realtor_info": {
                "realtor_id": "string",
                "name": "string?",
                "agency": "string?"
            },
            "match_score": "number",
            "commission_details": {
                "proposed_share": "number"
            }
        }
    ]
}
```

#### Лайк/дизлайк объекта для лида

```http
POST /leads/{lead_id}/property-recommendations/{property_id}/action
```

**Request Body:**

```json
{
    "action": "enum [LIKE | DISLIKE]",
    "proposed_commission": "number?",
    "notes": "string?"
}
```

#### Лайк/дизлайк лида для объекта

```http
POST /properties/{property_id}/lead-recommendations/{lead_id}/action
```

**Request Body:**

```json
{
    "action": "enum [LIKE | DISLIKE]",
    "proposed_commission": "number?",
    "notes": "string?"
}
```

#### Получение мэтчей для конкретного лида

```http
GET /leads/{lead_id}/matches
```

Query Parameters:
- `status`: `enum [PENDING | ACCEPTED | REJECTED | COMPLETED]`
- `page` (optional): `number`
- `limit` (optional): `number`

**Response:**

```json
{
    "matches": [
        {
            "match_id": "string",
            "property": {
                "property_id": "string",
                "address": "object",
                "details": "object",
                "photos": "string[]"
            },
            "status": "enum [PENDING | ACCEPTED | REJECTED | COMPLETED]",
            "commission": {
                "proposed_by_lead": "number?",
                "negotiation_status": "enum [PENDING | AGREED | REJECTED]"
            },
            "realtor_info": {
                "realtor_id": "string",
                "name": "string?",
                "agency": "string?",
                "telegram": "string?"
            },
            "timeline": {
                "created_at": "timestamp",
                "contact_shared_at": "timestamp?"
            }
        }
    ]
}
```

#### Получение мэтчей для конкретного объекта

```http
GET /properties/{property_id}/matches
```

Query Parameters:
- `status`: `enum [PENDING | ACCEPTED | REJECTED | COMPLETED]`
- `page` (optional): `number`
- `limit` (optional): `number`

**Response:**

```json
{
    "matches": [
        {
            "match_id": "string",
            "lead": {
                "lead_id": "string",
                "client_name": "string?",
                "requirements": "object"
            },
            "status": "enum [PENDING | ACCEPTED | REJECTED | COMPLETED]",
            "commission": {
                "proposed_by_property": "number?",
                "negotiation_status": "enum [PENDING | AGREED | REJECTED]"
            },
            "realtor_info": {
                "realtor_id": "string",
                "name": "string?",
                "agency": "string?",
                "telegram": "string?"
            },
            "timeline": {
                "created_at": "timestamp",
                "contact_shared_at": "timestamp?"
            }
        }
    ]
}
```

### Уведомления

#### Получение уведомлений

```http
GET /notifications
```

Query Parameters:
- `unread_only` (optional): `boolean`
- `page` (optional): `number`
- `limit` (optional): `number`

**Response:**

```json
{
    "notifications": [
        {
            "id": "string",
            "type": "enum [NEW_MATCH | CONTACT_SHARED]",
            "title": "string",
            "message": "string",
            "data": "object",
            "is_read": "boolean",
            "created_at": "timestamp"
        }
    ],
    "unread_count": "number"
}
```

#### Отметка уведомления как прочитанного

```http
POST /notifications/{notification_id}/read
```

