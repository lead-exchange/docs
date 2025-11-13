# Архитектура системы "Биржа лидов"

## Общее описание архитектуры

Система представляет собой веб-приложение с клиент-серверной архитектурой, ориентированное на мобильные устройства

## Основные компоненты системы

### 1. Клиентское приложение (Frontend)

- **Формат**: Мобильно-ориентированное веб-приложение (PWA) или Telegram Mini App
- **Технологии**: React + TypeScript
- **Основные функции**:
    - Интерфейс регистрации/авторизации
    - Управление лидами и объектами
    - Лента мэтчей (Tinder-like интерфейс)
    - Уведомления о мэтчах
    - Личный кабинет с базовой статистикой

### 2. Бэкенд сервисы (Backend)

- **Технологии**: Java с Spring Boot
- **Архитектура**: Монолит с четким разделением на модули

Рассмотрим основные модули

#### 2.1. User Service

- Регистрация и аутентификация пользователей
- Согласие с условиями платформы

#### 2.2. Lead Management Service

- CRUD операции для лидов (клиентов-покупателей)
- Валидация данных лидов
- Настройка долей комиссии
- Архивация/активация лидов

#### 2.3. Property Management Service

- CRUD операции для объектов недвижимости
- Валидация данных объектов
- Управление фотографиями объектов
- Настройка долей комиссии
- Архивация/активация объектов

#### 2.4. Matching Service

- Алгоритмы автоматического мэтчинга
- Управление лентой рекомендаций
- Обработка лайков/дизлайков
- Управление статусами мэтчей

#### 2.5. Notification Service

- Внутренние уведомления о мэтчах
- Управление push-уведомлениями (в интеграции с Telegram)

### 3. База данных

- **Технология**: PostgreSQL
- **Основные сущности**:
    - Пользователи (риэлторы)
    - Лиды (покупатели)
    - Объекты недвижимости
    - Мэтчи

## Формат взаимодействия между компонентами

### Взаимодействие Frontend <-> Backend

```mermaid
sequenceDiagram
    participant F as Frontend
    participant API as Backend API
    participant DB as Database

    Note over F,API: Аутентификация через JWT токены
    F->>API: HTTP REST запрос
    API->>DB: Запрос данных
    DB->>API: Результат
    API->>F: JSON ответ
```

### Детальное описание ключевых процессов

#### Процесс мэтчинга

## Статусы матчей

Матчи могут находиться в одном из следующих статусов:

- `UNDEFINED`
- `LIKED`
- `DISLIKED`
- `COMMISSION`
- `ACCEPTED`
- `DECLINED`

---

### Правила перехода между статусами мэтча

> ⚠️ **Общее правило:**  
> Статус `UNDEFINED` — начальное состояние. После первого изменения статуса **обратно в `UNDEFINED` перейти нельзя**.

#### Доступные переходы в зависимости от текущего статуса коллеги:

| Текущий статус коллеги | Доступные действия для меня |
|------------------------|-----------------------------|
| `UNDEFINED`            | → `LIKED`, `DISLIKED`, `COMMISSION` |
| `COMMISSION`           | → `COMMISSION`, `ACCEPTED`, `DECLINED` |
| `LIKED`                | → `LIKED`, `DISLIKED`, `COMMISSION` |
| `DISLIKED`             | ❌ Нельзя изменить (блокировка) |
| `ACCEPTED`             | ❌ Нельзя изменить (блокировка) |
| `DECLINED`             | ❌ Нельзя изменить (блокировка) |

> ✅ **Редактирование последней записи:**  
> Пользователь может изменить статус **дважды подряд**, но при этом **должны соблюдаться указанные выше правила**.  
> Только последняя запись статуса редактируема — предыдущие фиксируются и не изменяются.

---

### Примеры

#### Пример 1: Редактирование
1. Я ставлю: `LIKED` (последняя запись)
2. Могу изменить её на `DISLIKED` или `COMMISSION` (в рамках правил)
3. После этого новое значение становится последним — можно снова редактировать, пока не будет установлен финальный статус

### Правила создания лога и обновление таблицы matches

- Любое обновление статуса матча ведет к записи в журнал
- В таблице матчес храниться только послдений актуальный статус


### Правила перехода между статусами

```mermaid
flowchart LR
    A[Новый лид/объект] --> B[Matching Service]
    B --> C[Поиск подходящих пар]
    C --> D[Ранжирование по релевантности]
    D --> E[Добавление в ленту]
    E --> F[Показ пользователю]
    F --> G{Лайк/Дизлайк}
    G --> H[Ответный лайк/лайк с условием]
    H --> J[Мэтч!]
    G --> I[Пропуск]
```


## Технические решения

### API Design

- **RESTful API** с JSON форматом
- **Error Handling:** Стандартизированные HTTP коды ошибок

### База данных

```mermaid
erDiagram
    USERS ||--o{ LEADS : creates
    USERS ||--o{ ESTATES : creates
    LEADS ||--o{ MATCHES : included_in
    ESTATES ||--o{ MATCHES : included_in
    MATCHES ||--o{ MATCHES_LOG : included_in
    USERS ||--o{ MATCHES_LOG : included_in
    USERS ||--o{ MATCHES : included_in
    
    USERS {
        uuid id PK
        varchar telegram_id
        timestamp created_at
        timestamp updated_at
    }
    
    LEADS {
        uuid id PK
        uuid user_id FK
        jsonb requirements
        varchar status
        decimal commission_share
        timestamp created_at
        timestamp updated_at
    }
    
    ESTATES {
        uuid id PK
        uuid user_id FK
        jsonb attributes
        decimal total_commission_rate
        decimal commission_share
        varchar status
        timestamp created_at
        timestamp updated_at
    }
    
    MATCHES {
        uuid id PK
        uuid lead_id FK
        uuid property_id FK
        decimal lead_commission
        uuid updated_by FK
        text comment
        varchar lead_status
        varchar estate_status
        timestamp matched_at
        timestamp created_at
        timestamp updated_at
    }
    
    MATCHES_LOG {
        uuid id PK
        uuid match_id FK
        varchar status
        decimal lead_commission
        uuid updated_by FK
        text comment
        timestamp created_at
    }
```

## Масштабируемость и ограничения

### Текущие ограничения (MVP)

- До 1000 активных пользователей
- До 10000 активных лидов/объектов
- Простой алгоритм мэтчинга

### Возможности для масштабирования

- Кэширование частых запросов (Redis)
- Микросервисная архитектура
- Улучшенные алгоритмы мэтчинга (ML/DL)
- Горизонтальное масштабирование БД

