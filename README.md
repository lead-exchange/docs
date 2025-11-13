# Документация проекта

Этот репозиторий содержит всю документацию по проекту. Здесь вы найдете требования, архитектурные решения, стандарты разработки и другую важную информацию

## Структура документации

```bash
docs/
├── requirements/ # Различные требования (бизнес, функциональные и тд)
├── analysis/     # Анализ рынка и рисков
├── design/       # Архитектура и API
└── guidelines/   # Стандарты и руководства
```

## Быстрый доступ
- [Презентация](https://docs.google.com/presentation/d/13-437cfUWmEjm1xPf43afObIOPyLnWYpTucjpRch9Bg/edit?usp=sharing)
- [Бот] (https://t.me/lead_exchange_bot)
### Требования
- [Бизнес-требования](requirements/business.md)
- [Пользовательские, функциональные, нефункциональные требования](requirements/user_functional_non-functional.md)

### Стандарты разработки
- [Формат веток и коммитов](guidelines/branch-commit-format.md)

### Анализ рынка и рисков
- [Анализ конкурентов](analytics/competition-analysis.md)
- [Карта рисков](analytics/risk-map.md)

### Архитектура и дизайн
- [API приложения](design/api.md)
- [Архитектура приложения](design/architecture.md)
- [Дизайн](https://www.figma.com/design/t3tf8xmkJHUU9YporI17TJ/Birzha-Lidov?node-id=26-509&p=f&t=hcREp6OTSLighx9C-0)

## Процесс работы с документацией

### Внесение изменений
1. Создайте issue с описанием необходимых изменений
2. Создайте ветку по шаблону:
   ```bash
   git checkout -b feature/{issue-number}
   ```
3. Внесите изменения и создайте PR
4. Получите ревью от команды (необходим минимум от одного)
5. После получения approve - merge в main
