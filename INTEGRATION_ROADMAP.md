# Дорожная карта интеграции Frontend, Backend и Strapi

## Обзор
Интеграция включает соединение Strapi (CMS и API), Backend (дополнительная логика) и Frontend (Next.js интерфейс). Цель - обеспечить обмен данными, аутентификацию и функциональность (резервы, меню, события).

## Шаг 1: Подготовка окружения
- Убедиться, что Strapi запущен на http://localhost:1337.
- Запустить Backend сервер (npm run start в backend).
- Запустить Frontend (npm run dev в frontend).
- Настроить .env файлы для API ключей и URL (STRAPI_URL, BACKEND_URL).

## Шаг 2: Интеграция Strapi с Backend
- В Backend создать сервисы для запросов к Strapi API (axios или fetch).
- Реализовать endpoints в Backend для проксирования данных из Strapi (например, /api/reservations).
- Добавить обработку ошибок и валидацию.

## Шаг 3: Интеграция Backend с Frontend
- В Frontend использовать API routes или lib для запросов к Backend (например, fetch('/api/backend/reservations')).
- Обновить компоненты (ReservationForm, MenuList) для динамического получения данных.
- Добавить состояние (React Query или Context) для кэширования данных.

## Шаг 4: Полная интеграция (Telegram, Auth)
- Восстановить Telegram уведомления в Strapi/Backend без ошибок.
- Настроить аутентификацию (JWT из Strapi в Backend и Frontend).
- Интегрировать Yandex Maps в Frontend с данными из Strapi.

## Шаг 5: Тестирование и Оптимизация
- Тестировать E2E сценарии (резерв стола, просмотр меню).
- Добавить кэширование (Redis в Backend).
- Оптимизировать изображения и запросы.
- Развернуть в production (Docker).

После каждого шага тестировать и фиксировать изменения.