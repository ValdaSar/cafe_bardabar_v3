# Strapi CMS для проекта "Бар-да-бар"

Headless CMS для управления контентом сайта кафе "Бар-да-бар".

## Функционал

### Content Types (Типы контента)
- **Homepage** - Контент главной страницы
- **Menu Items** - Блюда и напитки
- **Menu Categories** - Категории меню
- **Events** - Афиша мероприятий
- **Reviews** - Отзывы гостей (с модерацией)
- **Reservations** - Бронирования столиков
- **Team Members** - Команда кафе

### API Endpoints

#### Меню
- `GET /api/menu-items` - Получение всех блюд
- `GET /api/menu-items/:id` - Получение конкретного блюда
- `GET /api/menu-categories` - Получение категорий меню

#### События
- `GET /api/events` - Получение всех событий
- `GET /api/events/:id` - Получение конкретного события
- `GET /api/events?filters[isActive][$eq]=true` - Получение только активных событий

#### Отзывы
- `GET /api/reviews` - Опубликованные отзывы
- `POST /api/reviews` - Создание отзыва (требует модерации)
- `GET /api/reviews/pending` - Отзывы на модерации (требует авторизации)
- `PUT /api/reviews/:id/moderate` - Модерация отзыва (требует авторизации)

#### Бронирования
- `POST /api/reservations` - Создание бронирования
- `GET /api/reservations/today` - Бронирования на сегодня (требует авторизации)
- `PUT /api/reservations/:id/status` - Изменение статуса бронирования (требует авторизации)

## Запуск

### Разработка
```bash
npm run develop
```

### Продакшн
```bash
npm run build
npm run start
```

## База данных

### PostgreSQL (рекомендуется)
Настроена для работы с PostgreSQL через переменные окружения в `.env`:

```env
DATABASE_CLIENT=postgres
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=cafe_bardabar_strapi
DATABASE_USERNAME=dev
DATABASE_PASSWORD=dev
```

### Запуск с Docker
```bash
# Из корневой директории проекта
docker-compose up -d postgres strapi
```

## Первый запуск

1. Запустите Strapi: `npm run develop`
2. Откройте http://localhost:1337/admin
3. Создайте администратора
4. Запустится скрипт bootstrap.js, который создаст тестовые данные

## Тестовые данные

При первом запуске Strapi автоматически создаст следующие тестовые данные:
- 5 категорий меню (Бургеры, Закуски, Напитки, Десерты, Спецпредложения)
- 9 блюд меню с описаниями и ценами
- 4 события с датами и описаниями
- 4 отзыва с рейтингами
- 4 члена команды с описаниями
- Контент для главной страницы

## API ключи

Для доступа к API нужно:
1. Зайти в Settings → API Tokens
2. Создать новый токен
3. Использовать его в заголовке: `Authorization: Bearer YOUR_TOKEN`

## Структура проекта

```
strapi-cms/
├── config/           # Конфигурация
│   ├── database.ts   # Настройки БД
│   ├── server.ts     # Настройки сервера
│   ├── middlewares.ts # CORS и безопасность
│   └── plugins.ts    # Плагины
├── src/
│   ├── api/          # API endpoints
│   │   ├── reservation/  # Бронирования
│   │   ├── review/       # Отзывы
│   │   ├── menu-item/    # Меню
│   │   └── ...
│   ├── bootstrap.js  # Скрипт для заполнения тестовыми данными
│   ├── components/   # Переиспользуемые компоненты
│   └── index.ts      # Инициализация данных
└── types/            # TypeScript типы
```

## Кастомизация

### Добавление нового content type
1. Создать в `src/api/your-type/content-types/your-type/schema.json`
2. Добавить контроллер в `src/api/your-type/controllers/`
3. Настроить роуты в `src/api/your-type/routes/`

### Кастомные API
Примеры кастомных контроллеров и роутов смотрите в:
- `src/api/reservation/controllers/reservation.ts`
- `src/api/review/controllers/review.ts`

## Безопасность

- CORS настроен для локальной разработки
- Rate limiting включен
- Helmet для HTTP заголовков
- Validation для всех полей

## Мониторинг

Все действия логируются в консоль:
- Новые бронирования
- Новые отзывы
- Модерация контента

## Интеграция

Strapi интегрирован с:
- Express.js backend - через API
- Next.js frontend - через API
- Legacy frontend - через API
- PostgreSQL - для хранения данных

---

## 📚 Документация Strapi

- [Strapi Documentation](https://docs.strapi.io)
- [API Reference](https://docs.strapi.io/dev-docs/api/rest)
- [Plugin Development](https://docs.strapi.io/dev-docs/plugins)

- [Discord](https://discord.strapi.io) - Come chat with the Strapi community including the core team.
- [Forum](https://forum.strapi.io/) - Place to discuss, ask questions and find answers, show your Strapi project and get feedback or just talk with other Community members.
- [Awesome Strapi](https://github.com/strapi/awesome-strapi) - A curated list of awesome things related to Strapi.

---

<sub>🤫 Psst! [Strapi is hiring](https://strapi.io/careers).</sub>
