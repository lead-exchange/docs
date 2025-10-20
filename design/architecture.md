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
- Управление профилями риэлторов
- Согласие с условиями платформы

#### 2.2. Lead Management Service

- CRUD операции для лидов (клиентов-покупателей)
- Валидация и нормализация данных лидов
- Архивация/активация лидов

#### 2.3. Property Management Service  

- CRUD операции для объектов недвижимости
- Управление фотографиями объектов
- Настройка долей комиссии

#### 2.4. Matching Service

- Алгоритмы автоматического мэтчинга
- Управление лентой рекомендаций
- Обработка лайков/дизлайков
- Управление статусами мэтчей

#### 2.5. Notification Service

- Внутренние уведомления о мэтчах
- Управление push-уведомлениями (при интеграции с Telegram)

### 3. База данных

- **Технология**: PostgreSQL
- **Основные сущности**:
  - Пользователи (риэлторы)
  - Лиды (покупатели)
  - Объекты недвижимости
  - Мэтчи
  - Уведомления

## Формат взаимодействия между компонентами

### Взаимодействие Frontend <-> Backend

```mermaid
sequenceDiagram
    participant F as Frontend
    participant API as Backend API
    participant DB as Database
    
    F->>API: HTTP REST/GraphQL запрос
    API->>DB: Запрос данных
    DB->>API: Результат
    API->>F: JSON ответ
    
    Note over F,API: Аутентификация через JWT токены
```

### Взаимодействие между сервисами бэкенда

```mermaid
flowchart TD
    A[User Service] --> B[Lead Service]
    A --> C[Property Service]
    B --> D[Matching Service]
    C --> D
    D --> E[Notification Service]
    E --> F[Frontend]
```

### Детальное описание ключевых процессов

#### Процесс мэтчинга

```mermaid
flowchart LR
    A[Новый лид/объект] --> B[Matching Service]
    B --> C[Поиск подходящих пар]
    C --> D[Ранжирование по релевантности]
    D --> E[Добавление в ленту]
    E --> F[Показ пользователю]
    F --> G{Лайк/Дизлайк}
    G --> H[Мэтч!]
    G --> I[Пропуск]
```

#### Процесс сделки после мэтча

```mermaid
sequenceDiagram
    participant R1 as Риэлтор 1
    participant R2 as Риэлтор 2
    participant S as System
    
    par 
        R1->>S: Лайкает объект
    and 
        R2->>S: Лайкает лида
    end
    Note over S: Мэтч
    
    par 
        S->>R1: Контакты R2
    and 
        S->>R2: Контакты R1
    end

    R1<<->>R2: Обсуждение деталей сделки
```

## Технические решения

### API Design

- **RESTful API** с JSON форматом
- **Error Handling:** Стандартизированные коды ошибок

### База данных

```mermaid
erDiagram
    USERS ||--o{ LEADS : creates
    USERS ||--o{ PROPERTIES : creates
    LEADS ||--o{ MATCHES : included_in
    PROPERTIES ||--o{ MATCHES : included_in
    
    USERS {
        bigint id PK
        varchar telegram_id
        varchar phone
        varchar email
        timestamp created_at
    }
    
    LEADS {
        bigint id PK
        bigint user_id FK
        jsonb requirements
        varchar status
        timestamp created_at
    }
    
    PROPERTIES {
        bigint id PK
        bigint user_id FK
        jsonb attributes
        decimal commission_share
        varchar status
        timestamp created_at
    }
    
    MATCHES {
        bigint id PK
        bigint lead_id FK
        bigint property_id FK
        varchar status
        decimal lead_user_commission
        decimal property_user_commission
        timestamp matched_at
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

