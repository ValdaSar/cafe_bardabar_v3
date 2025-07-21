# Инструкция по разработке фронтенда на Next.js

Эта инструкция поможет вам настроить и запустить фронтенд-часть проекта на Next.js, пока бэкенд, база данных и CMS работают в Docker-контейнерах.

## Предварительные требования

1. Node.js (рекомендуется версия 18 или выше)
2. npm (устанавливается вместе с Node.js)
3. Запущенные Docker-контейнеры с бэкендом, базой данных и Strapi CMS (см. инструкцию в `DOCKER_README.md`)

## Настройка и запуск фронтенда

### 1. Установка зависимостей

Перейдите в директорию `next-landing` и установите зависимости:

```bash
cd next-landing
npm install
```

### 2. Настройка переменных окружения

Создайте файл `.env.local` в директории `next-landing` со следующим содержимым:

```
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_STRAPI_URL=http://localhost:1337
```

Эти переменные окружения указывают на запущенные Docker-контейнеры с бэкендом и Strapi CMS.

### 3. Запуск сервера разработки

Запустите сервер разработки Next.js:

```bash
npm run dev
```

Фронтенд будет доступен по адресу: http://localhost:3000

## Структура проекта

Основная структура директории `next-landing`:

- `src/pages` - страницы приложения
- `src/components` - компоненты React
- `src/styles` - стили CSS/SCSS
- `src/lib` - вспомогательные функции и утилиты
- `src/utils` - утилиты и хелперы
- `public` - статические файлы (изображения, шрифты и т.д.)

## Работа с API

Для взаимодействия с бэкендом и Strapi CMS используйте модули из директории `shared`:

```javascript
import { StrapiClient } from '../../shared/api/strapiClient';

// Пример использования
const strapiClient = new StrapiClient();
const menuItems = await strapiClient.getMenuItems();
```

## Разработка компонентов

Проект использует Tailwind CSS для стилизации компонентов. Пример создания компонента:

```jsx
// src/components/Button.tsx
import React from 'react';

interface ButtonProps {
  text: string;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

const Button: React.FC<ButtonProps> = ({ text, onClick, variant = 'primary' }) => {
  const baseClasses = 'px-4 py-2 rounded font-medium';
  const variantClasses = variant === 'primary' 
    ? 'bg-blue-600 text-white hover:bg-blue-700' 
    : 'bg-gray-200 text-gray-800 hover:bg-gray-300';
  
  return (
    <button 
      className={`${baseClasses} ${variantClasses}`}
      onClick={onClick}
    >
      {text}
    </button>
  );
};

export default Button;
```

## Сборка для продакшена

Для сборки приложения для продакшена выполните:

```bash
npm run build
```

Для запуска собранного приложения:

```bash
npm run start
```

## Линтинг и форматирование кода

Для проверки кода с помощью ESLint:

```bash
npm run lint
```

## Отладка

Для отладки Next.js приложения можно использовать инструменты разработчика в браузере. Также можно использовать отладку в VS Code, настроив конфигурацию в файле `.vscode/launch.json`.

## Полезные ресурсы

- [Документация Next.js](https://nextjs.org/docs)
- [Документация React](https://reactjs.org/docs/getting-started.html)
- [Документация Tailwind CSS](https://tailwindcss.com/docs)
- [Документация Framer Motion](https://www.framer.com/motion/)