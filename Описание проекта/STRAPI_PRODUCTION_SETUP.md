# Настройка Strapi для Production

Для корректной работы сайта в production окружении необходимо настроить Strapi CMS и обновить конфигурацию.

## 1. Переменные окружения для Next.js

### Development (.env.local)
Текущие настройки для разработки:
```env
NEXT_PUBLIC_STRAPI_API_URL=http://localhost:1337
NEXT_PUBLIC_STRAPI_API_TOKEN=your_strapi_api_token_here
```

### Production (.env.production или переменные сервера)
Для production необходимо обновить:
```env
# Production Strapi Configuration
NEXT_PUBLIC_STRAPI_API_URL=https://your-strapi-domain.com
NEXT_PUBLIC_STRAPI_API_TOKEN=your_production_strapi_api_token

# Internal URL для SSR (если Strapi и Next.js в одной сети)
STRAPI_INTERNAL_URL=http://strapi:1337
```

## 2. Настройка CORS в Strapi

### Текущая конфигурация (Development)
Файл: `strapi-cms/config/middlewares.ts`
```typescript
origin: [
  'http://localhost:3000', // Next.js frontend
  'http://localhost:3001', // Express.js backend  
  'http://localhost:1337', // Strapi admin
]
```

### Production конфигурация
Обновите `middlewares.ts` для поддержки production доменов:

```typescript
export default [
  'strapi::logger',
  'strapi::errors',
  'strapi::security',
  {
    name: 'strapi::cors',
    config: {
      origin: [
        // Development URLs
        'http://localhost:3000',
        'http://localhost:3001', 
        'http://localhost:1337',
        
        // Production URLs - ЗАМЕНИТЕ НА ВАШИ ДОМЕНЫ
        'https://your-website-domain.com',
        'https://www.your-website-domain.com',
        'https://your-strapi-domain.com',
        
        // Дополнительные домены при необходимости
        'https://staging.your-website-domain.com',
      ],
      methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS', 'PATCH'],
      headers: [
        'Content-Type',
        'Authorization',
        'Origin',
        'Accept',
        'X-Requested-With',
      ],
      keepHeaderOnError: true,
    },
  },
  'strapi::poweredBy',
  'strapi::query',
  'strapi::body',
  'strapi::session',
  'strapi::favicon',
  'strapi::public',
];
```

## 3. Переменные окружения для Strapi

### Создайте файл `.env` в папке `strapi-cms/`

```env
# Database Configuration
DATABASE_CLIENT=postgres
DATABASE_HOST=your-db-host
DATABASE_PORT=5432
DATABASE_NAME=strapi_production
DATABASE_USERNAME=your-db-user
DATABASE_PASSWORD=your-db-password
DATABASE_SSL=true

# Server Configuration
HOST=0.0.0.0
PORT=1337
APP_KEYS=your-app-keys-here
API_TOKEN_SALT=your-api-token-salt
ADMIN_JWT_SECRET=your-admin-jwt-secret
TRANSFER_TOKEN_SALT=your-transfer-token-salt
JWT_SECRET=your-jwt-secret

# Environment
NODE_ENV=production

# Security
CORS_ENABLED=true

# File Upload (если используете внешнее хранилище)
# AWS_ACCESS_KEY_ID=your-aws-key
# AWS_SECRET_ACCESS_KEY=your-aws-secret
# AWS_REGION=your-aws-region
# AWS_BUCKET=your-s3-bucket
```

## 4. Генерация секретных ключей

Для генерации безопасных ключей используйте:

```bash
# В папке strapi-cms
npm run strapi generate

# Или вручную генерируйте случайные строки
node -e "console.log(require('crypto').randomBytes(64).toString('base64'))"
```

## 5. База данных Production

### PostgreSQL (рекомендуется)
1. **Создайте базу данных PostgreSQL**
   - Используйте сервисы типа AWS RDS, Google Cloud SQL, или DigitalOcean
   - Или настройте собственный PostgreSQL сервер

2. **Настройте подключение**
   ```env
   DATABASE_CLIENT=postgres
   DATABASE_HOST=your-postgres-host
   DATABASE_PORT=5432
   DATABASE_NAME=strapi_production
   DATABASE_USERNAME=your-username
   DATABASE_PASSWORD=your-password
   DATABASE_SSL=true
   ```

### SQLite (только для тестирования)
```env
DATABASE_CLIENT=sqlite
DATABASE_FILENAME=.tmp/data.db
```

## 6. Деплой Strapi

### Вариант 1: Docker
1. **Создайте Dockerfile в папке strapi-cms/**
   ```dockerfile
   FROM node:18-alpine
   
   WORKDIR /app
   
   COPY package*.json ./
   RUN npm ci --only=production
   
   COPY . .
   
   RUN npm run build
   
   EXPOSE 1337
   
   CMD ["npm", "start"]
   ```

2. **Docker Compose**
   ```yaml
   version: '3.8'
   services:
     strapi:
       build: ./strapi-cms
       ports:
         - "1337:1337"
       environment:
         - NODE_ENV=production
       volumes:
         - ./strapi-cms/public/uploads:/app/public/uploads
   ```

### Вариант 2: Традиционный хостинг
1. **Загрузите код на сервер**
2. **Установите зависимости**
   ```bash
   cd strapi-cms
   npm ci --only=production
   ```
3. **Соберите проект**
   ```bash
   npm run build
   ```
4. **Запустите в production**
   ```bash
   npm start
   ```

### Вариант 3: Облачные платформы
- **Heroku**: Добавьте buildpack для Node.js
- **Vercel**: Не рекомендуется для Strapi (serverless ограничения)
- **Railway**: Хорошо подходит для Strapi
- **DigitalOcean App Platform**: Простой деплой

## 7. Настройка API токенов

1. **Войдите в админ-панель Strapi** (https://your-strapi-domain.com/admin)
2. **Перейдите в Settings → API Tokens**
3. **Создайте новый токен**:
   - Name: "Production Frontend Token"
   - Token type: "Read-only" (для безопасности)
   - Token duration: "Unlimited" или установите срок действия
4. **Скопируйте токен** и добавьте в переменные окружения Next.js

## 8. Миграция данных

### Экспорт данных из Development
```bash
cd strapi-cms
npm run strapi export -- --file backup.tar.gz
```

### Импорт данных в Production
```bash
cd strapi-cms
npm run strapi import -- --file backup.tar.gz
```

## 9. Мониторинг и логи

### PM2 (для традиционного хостинга)
```bash
npm install -g pm2
cd strapi-cms
pm2 start npm --name "strapi" -- start
pm2 save
pm2 startup
```

### Логирование
Добавьте в конфигурацию Strapi:
```javascript
// config/logger.js
module.exports = {
  level: 'info',
  format: 'combined',
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
};
```

## 10. Безопасность

### Обновите настройки безопасности
```javascript
// config/middlewares.js - добавьте security настройки
{
  name: 'strapi::security',
  config: {
    contentSecurityPolicy: {
      useDefaults: true,
      directives: {
        'connect-src': ["'self'", 'https:'],
        'img-src': ["'self'", 'data:', 'blob:', 'https:'],
        'media-src': ["'self'", 'data:', 'blob:', 'https:'],
        upgradeInsecureRequests: null,
      },
    },
  },
}
```

### Настройте HTTPS
- Используйте SSL сертификаты (Let's Encrypt)
- Настройте редирект с HTTP на HTTPS
- Обновите все URL в конфигурации на HTTPS

## 11. Проверка готовности

### Чек-лист перед запуском:
- [ ] База данных настроена и доступна
- [ ] Все переменные окружения установлены
- [ ] CORS настроен для production доменов
- [ ] API токены созданы и настроены
- [ ] SSL сертификаты установлены
- [ ] Данные мигрированы (если необходимо)
- [ ] Мониторинг и логирование настроены
- [ ] Backup стратегия определена

### Тестирование
1. **Проверьте доступность API**:
   ```bash
   curl https://your-strapi-domain.com/api/menu-items
   ```

2. **Проверьте админ-панель**:
   - Откройте https://your-strapi-domain.com/admin
   - Войдите с учетными данными администратора

3. **Проверьте интеграцию с Next.js**:
   - Обновите переменные окружения в Next.js
   - Проверьте загрузку данных на сайте

## 12. Troubleshooting

### Частые проблемы:

**CORS ошибки**
- Проверьте настройки origin в middlewares.ts
- Убедитесь, что домен точно совпадает (с/без www)

**База данных недоступна**
- Проверьте настройки подключения
- Убедитесь, что база данных запущена
- Проверьте firewall правила

**API токен не работает**
- Проверьте правильность токена
- Убедитесь, что токен имеет нужные разрешения
- Проверьте срок действия токена

**Файлы не загружаются**
- Проверьте права доступа к папке uploads
- Настройте внешнее хранилище (S3, Cloudinary)

---

**Важно**: Сохраните все пароли и ключи в безопасном месте. Не коммитьте production переменные окружения в репозиторий.