# 🖼️ Система оптимизации изображений

## 📋 Обзор

Комплексная система оптимизации изображений для Next.js приложения, включающая:
- Автоматическую оптимизацию форматов (WebP, AVIF)
- Ленивую загрузку с Intersection Observer
- Предзагрузку критических изображений
- Адаптивные размеры и качество
- Мониторинг производительности
- Кэширование изображений

## 🏗️ Архитектура

### Компоненты

#### 1. OptimizedImage
Основной компонент для отображения оптимизированных изображений:

```tsx
import { OptimizedImage } from '@/components/ui/OptimizedImage';

<OptimizedImage
  src="/images/hero.jpg"
  alt="Главное изображение"
  width={800}
  height={600}
  priority={true}
  trackPerformance={true}
  adaptiveQuality={true}
/>
```

**Особенности:**
- Автоматический выбор лучшего формата (WebP/AVIF)
- Генерация placeholder'ов
- Обработка ошибок загрузки
- Интеграция с системой мониторинга

#### 2. LazyImage
Компонент для ленивой загрузки изображений:

```tsx
import { LazyImage } from '@/components/ui/LazyImage';

<LazyImage
  src="/images/gallery/photo.jpg"
  alt="Фото галереи"
  width={400}
  height={300}
  rootMargin="50px"
  threshold={0.1}
/>
```

**Особенности:**
- Intersection Observer API
- Настраиваемые пороги загрузки
- Плавные переходы
- Обработка ошибок

#### 3. ImagePerformanceMonitor
Инструмент для мониторинга производительности (только в development):

```tsx
import { ImagePerformanceMonitor } from '@/components/dev/ImagePerformanceMonitor';

<ImagePerformanceMonitor
  enabled={process.env.NODE_ENV === 'development'}
  position="bottom-right"
/>
```

**Метрики:**
- Общее количество изображений
- Успешно загруженные/неудачные
- Среднее время загрузки
- Процент кэшированных изображений
- Рекомендации по оптимизации

### Утилиты

#### imagePreloader.ts
Утилиты для предзагрузки и кэширования:

```tsx
import {
  preloadImage,
  preloadImages,
  preloadCriticalImages,
  createOptimizedImageUrl,
  generateSrcSet
} from '@/lib/imagePreloader';

// Предзагрузка одного изображения
await preloadImage('/images/hero.jpg');

// Предзагрузка критических изображений
await preloadCriticalImages();

// Создание оптимизированного URL
const optimizedUrl = createOptimizedImageUrl(
  '/images/photo.jpg',
  800, // width
  600, // height
  85   // quality
);

// Генерация srcSet для адаптивности
const srcSet = generateSrcSet('/images/photo.jpg', [320, 640, 1024]);
```

#### useImagePerformance.ts
Хуки для управления производительностью:

```tsx
import {
  useImagePerformance,
  usePageImagePerformance,
  useAdaptiveImageLoading
} from '@/hooks/useImagePerformance';

// Мониторинг отдельного изображения
const { isLoading, metrics, loadImage } = useImagePerformance(src, {
  preload: true,
  trackMetrics: true,
  onLoad: (metrics) => console.log('Loaded:', metrics),
});

// Мониторинг всех изображений на странице
const { stats, addMetrics, getPerformanceReport } = usePageImagePerformance();

// Адаптивная загрузка по типу соединения
const { shouldLoadHighQuality, getOptimalQuality } = useAdaptiveImageLoading();
```

## ⚙️ Конфигурация Next.js

### next.config.js
```javascript
module.exports = {
  images: {
    unoptimized: false, // Включаем оптимизацию
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'your-cdn.com',
      },
    ],
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  
  // Экспериментальные функции
  experimental: {
    optimizeCss: true,
    scrollRestoration: true,
  },
  
  // Оптимизация бандла
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production' ? {
      exclude: ['error']
    } : false,
  },
};
```

## 🚀 Использование

### 1. Замена стандартных изображений

**Было:**
```tsx
<img src="/images/photo.jpg" alt="Фото" />
```

**Стало:**
```tsx
<OptimizedImage
  src="/images/photo.jpg"
  alt="Фото"
  width={400}
  height={300}
  priority={false}
/>
```

### 2. Критические изображения

Для изображений "above the fold":
```tsx
<OptimizedImage
  src="/images/hero.jpg"
  alt="Главное изображение"
  width={1920}
  height={1080}
  priority={true} // Приоритетная загрузка
  sizes="100vw"
/>
```

### 3. Галерея изображений

Для множественных изображений:
```tsx
{images.map((image, index) => (
  <LazyImage
    key={image.id}
    src={image.url}
    alt={image.alt}
    width={300}
    height={200}
    priority={index < 4} // Первые 4 изображения приоритетные
  />
))}
```

### 4. Предзагрузка в useEffect

```tsx
useEffect(() => {
  // Предзагружаем критические изображения
  preloadCriticalImages();
  
  // Предзагружаем изображения следующей страницы
  preloadImages([
    '/images/about-hero.jpg',
    '/images/menu-bg.jpg',
  ]);
}, []);
```

## 📊 Мониторинг производительности

### В режиме разработки

1. **Визуальный мониторинг**: Кнопка в правом нижнем углу показывает:
   - Общее количество изображений
   - Успешно загруженные/неудачные
   - Среднее время загрузки
   - Процент кэшированных
   - Рекомендации по оптимизации

2. **Метрики на изображениях**: Время загрузки отображается поверх каждого изображения

3. **Консольные логи**: Детальная информация о загрузке каждого изображения

### В продакшене

```tsx
// Сбор метрик для аналитики
const { getPerformanceReport } = usePageImagePerformance();

useEffect(() => {
  const report = getPerformanceReport();
  
  // Отправка в аналитику
  analytics.track('image_performance', {
    totalImages: report.totalImages,
    successRate: report.successRate,
    averageLoadTime: report.averageLoadTime,
  });
}, []);
```

## 🎯 Лучшие практики

### 1. Размеры изображений

```tsx
// ✅ Правильно - явные размеры
<OptimizedImage
  src="/images/photo.jpg"
  width={400}
  height={300}
  alt="Описание"
/>

// ❌ Неправильно - без размеров
<OptimizedImage
  src="/images/photo.jpg"
  alt="Описание"
/>
```

### 2. Приоритеты загрузки

```tsx
// ✅ Критические изображения
<OptimizedImage priority={true} /> // Hero, логотип

// ✅ Обычные изображения
<OptimizedImage priority={false} /> // Галерея, контент
```

### 3. Адаптивные размеры

```tsx
<OptimizedImage
  src="/images/responsive.jpg"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

### 4. Обработка ошибок

```tsx
<OptimizedImage
  src="/images/photo.jpg"
  alt="Фото"
  onError={() => {
    console.error('Ошибка загрузки изображения');
    // Логирование, fallback изображение
  }}
  onLoad={() => {
    console.log('Изображение загружено');
    // Аналитика, анимации
  }}
/>
```

## 🔧 Настройка

### Переменные окружения

```env
# .env.local
NEXT_PUBLIC_IMAGE_OPTIMIZATION=true
NEXT_PUBLIC_IMAGE_QUALITY=85
NEXT_PUBLIC_ENABLE_PERFORMANCE_MONITORING=true
```

### Кастомизация форматов

```tsx
// lib/imagePreloader.ts
export function getBestImageFormat(originalSrc: string): string {
  const formats = getSupportedImageFormats();
  
  // Приоритет: AVIF > WebP > оригинал
  if (formats.avif && shouldUseAVIF(originalSrc)) {
    return originalSrc.replace(/\.(jpg|jpeg|png)$/i, '.avif');
  }
  
  if (formats.webp && shouldUseWebP(originalSrc)) {
    return originalSrc.replace(/\.(jpg|jpeg|png)$/i, '.webp');
  }
  
  return originalSrc;
}
```

## 📈 Метрики производительности

### Целевые показатели

- **LCP (Largest Contentful Paint)**: < 2.5s
- **Время загрузки изображений**: < 1s
- **Процент успешных загрузок**: > 95%
- **Использование кэша**: > 70%

### Мониторинг

```tsx
// Интеграция с Web Vitals
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  // Отправка метрик в аналитику
  analytics.track('web_vital', {
    name: metric.name,
    value: metric.value,
    id: metric.id,
  });
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

## 🐛 Отладка

### Проблемы и решения

1. **Медленная загрузка**:
   - Проверьте размеры изображений
   - Используйте WebP/AVIF форматы
   - Включите предзагрузку для критических изображений

2. **Ошибки загрузки**:
   - Проверьте пути к файлам
   - Убедитесь в корректности remotePatterns в next.config.js
   - Добавьте fallback изображения

3. **Проблемы с кэшированием**:
   - Проверьте заголовки Cache-Control
   - Очистите кэш браузера
   - Проверьте размер кэша изображений

### Логирование

```tsx
// Включение детального логирования
if (process.env.NODE_ENV === 'development') {
  window.imageOptimizationDebug = true;
}
```

## 🔄 Миграция

### Пошаговая замена

1. **Установка зависимостей** (уже выполнено)
2. **Замена компонентов Image**:
   ```bash
   # Поиск всех использований
   grep -r "<Image" src/
   grep -r "next/image" src/
   ```

3. **Обновление импортов**:
   ```tsx
   // Было
   import Image from 'next/image';
   
   // Стало
   import { OptimizedImage } from '@/components/ui/OptimizedImage';
   ```

4. **Тестирование производительности**:
   - Включите мониторинг в development
   - Проверьте метрики в браузере
   - Сравните с предыдущими показателями

## 📚 Дополнительные ресурсы

- [Next.js Image Optimization](https://nextjs.org/docs/basic-features/image-optimization)
- [Web Vitals](https://web.dev/vitals/)
- [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [WebP Support](https://caniuse.com/webp)
- [AVIF Support](https://caniuse.com/avif)