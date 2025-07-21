# Backend для проекта "Бар-да-бар"

Централизованный бэкенд на Express.js для обслуживания legacy и Next.js лендингов кафе "Бар-да-бар".

## Функциональность

- Обслуживание статического контента legacy-лендинга
- Проксирование запросов к Next.js лендингу
- API для бронирования столиков, отзывов и других функций
- Интеграция с Telegram для уведомлений
- Взаимодействие со Strapi CMS для получения данных

## Структура проекта

```
backend/
├── src/
│   ├── config/         # Конфигурационные файлы
│   ├── middleware/     # Middleware функции
│   ├── models/         # Модели данных
│   ├── public/         # Статические файлы
│   ├── routes/         # Маршруты API
│   │   ├── api.js      # API маршруты
│   │   ├── index.js    # Основные маршруты
│   │   ├── legacy.js   # Маршруты для legacy-лендинга
│   │   └── next.js     # Маршруты для Next.js лендинга
│   ├── utils/          # Вспомогательные функции
│   ├── app.js          # Настройка Express приложения
│   └── server.js       # Точка входа
├── Dockerfile          # Конфигурация Docker
└── package.json        # Зависимости и скрипты
```

## API Endpoints

### Бронирование столиков

- `POST /api/reservation` - Создание бронирования
  ```json
  {
    "name": "Иван Петров",
    "phone": "+7 999 123 45 67",
    "date": "2025-06-26",
    "time": "19:00",
    "guests": "2",
    "comment": "Столик у окна, пожалуйста"
  }
  ```

### Telegram уведомления

- `POST /api/telegram/send` - Отправка уведомлений
  ```json
  {
    "message": "Новое бронирование: Иван Петров, 2 гостя, 26.06.2025 19:00"
  }
  ```

### Контактные формы

- `POST /api/contact` - Отправка сообщений из контактной формы
  ```json
  {
    "name": "Мария",
    "email": "maria@example.com",
    "message": "Хочу узнать о корпоративных мероприятиях"
  }
  ```

### Получение данных

- `GET /api/menu` - Получение меню
- `GET /api/events` - Получение событий
- `GET /api/reviews` - Получение отзывов
- `GET /api/team` - Получение информации о команде
- `GET /api/info` - Получение общей информации о кафе

## Запуск

### Разработка

```bash
npm run dev:backend
```

### Продакшн

```bash
npm run build
npm run start
```

## Переменные окружения

Создайте файл `.env` в корне проекта:

```
PORT=3001
NODE_ENV=development
STRAPI_URL=http://localhost:1337
NEXT_URL=http://localhost:3000
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_CHAT_ID=your_telegram_chat_id
```

## Зависимости

- Express.js - Веб-фреймворк
- http-proxy-middleware - Проксирование запросов к Next.js
- axios - HTTP-клиент для запросов к Strapi CMS
- dotenv - Загрузка переменных окружения
- cors - Настройка CORS
- helmet - Безопасность HTTP-заголовков
- morgan - Логирование запросов
- winston - Логирование приложения 