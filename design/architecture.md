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
![Label](matches.drawio)

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

#### Процесс сделки после мэтча

##### При согласии с разделением комиссии

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

##### При несогласии с разделением комиссии

###### Со стороны лида

```mermaid
sequenceDiagram
    participant R1 as Риэлтор 1
    participant R2 as Риэлтор 2
    participant S as System
    
    par 
        R1->>S: Лайкает объект
    and 
        R2->>S: Лайкает лида (с условием)
    end
    S->>R1: Информация с предложением об изменении комиссии
    alt согласен
        R1->>S: согласие с изменившемися условиями сделки
        Note over S: Мэтч
        par
          S->>R1: Контакты R2
        and
          S->>R2: Контакты R1
        end
  
        R1<<->>R2: Обсуждение деталей сделки
    else
        R1->>S: несогласие с изменившемися условиями сделки
        Note over S: Мэтч не случился
    end
```

###### Со стороны объекта

```mermaid
sequenceDiagram
    participant R1 as Риэлтор 1
    participant R2 as Риэлтор 2
    participant S as System

    par
        R1->>S: Лайкает объект (с условием)
    and
        R2->>S: Лайкает лида
    end
    
    S->>R2: Информация с предложением об изменении комиссии
    
    alt согласен
        R2->>S: согласие с изменившемися условиями сделки
        Note over S: Мэтч
        
        par
            S->>R1: Контакты R2
        and
            S->>R2: Контакты R1
        end
    
        R1<<->>R2: Обсуждение деталей сделки
    else
        R2->>S: несогласие с изменившемися условиями сделки
        Note over S: Мэтч не случился
    end
```

## Технические решения

### API Design

- **RESTful API** с JSON форматом
- **Error Handling:** Стандартизированные HTTP коды ошибок

### База данных

```mermaid
erDiagram
    USERS ||--o{ LEADS : creates
    USERS ||--o{ PROPERTIES : creates
    LEADS ||--o{ MATCHES : included_in
    PROPERTIES ||--o{ MATCHES : included_in
    
    USERS {
        uuid id PK
        varchar telegram_id
        varchar phone
        varchar email
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
    
    PROPERTIES {
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
        varchar status
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

