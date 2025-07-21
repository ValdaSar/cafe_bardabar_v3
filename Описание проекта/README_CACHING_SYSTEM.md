# 🚀 Система кэширования для Next.js приложения

## 📋 Обзор

Эта система кэширования предоставляет мощные инструменты для оптимизации производительности React/Next.js приложения через интеллектуальное кэширование API запросов, предварительную загрузку данных и управление состоянием загрузки.

## 🎯 Основные возможности

- ✅ **Автоматическое кэширование** API запросов с настраиваемым TTL
- ✅ **Предварительная загрузка** данных на основе пользовательского поведения
- ✅ **Умное управление состоянием** загрузки с красивыми скелетонами
- ✅ **Мониторинг производительности** в режиме разработки
- ✅ **TypeScript поддержка** с полной типизацией
- ✅ **Оптимизация памяти** с автоматической очисткой
- ✅ **Intersection Observer** для ленивой загрузки
- ✅ **Централизованное управление** через React Context

## 📁 Структура файлов

```
src/
├── lib/
│   ├── cache.ts              # Основная логика кэширования
│   ├── cachedFetch.ts        # Обертка для fetch с кэшированием
│   ├── preloader.ts          # Утилиты предварительной загрузки
│   └── utils.ts              # Вспомогательные функции
├── hooks/
│   ├── useApiState.ts        # Хук для управления состоянием API
│   └── useStrapi.ts          # Специализированные хуки для Strapi
├── components/
│   └── LoadingStates.tsx     # Компоненты состояний загрузки
├── providers/
│   └── CacheProvider.tsx     # React провайдер для кэширования
├── app/
│   └── layout-with-cache.tsx # Пример интеграции в layout
└── docs/
    └── CACHING_GUIDE.md      # Подробная документация
```

## 🚀 Быстрый старт

### 1. Интеграция в приложение

```tsx
// app/layout.tsx
import { CacheProvider } from '@/providers/CacheProvider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ru">
      <body>
        <CacheProvider
          autoPreload={true}
          showMonitor={process.env.NODE_ENV === 'development'}
          cleanupInterval={5 * 60 * 1000} // 5 минут
          maxCacheSize={150}
        >
          {children}
        </CacheProvider>
      </body>
    </html>
  );
}
```

### 2. Использование в компонентах

```tsx
// components/MenuList.tsx
import { useMenuItems } from '@/hooks/useStrapi';
import { LoadingState, ErrorState } from '@/components/LoadingStates';

export function MenuList() {
  const { data: menuItems, isLoading, error } = useMenuItems();

  if (isLoading) return <LoadingState type="menu" />;
  if (error) return <ErrorState message={error} onRetry={() => window.location.reload()} />;

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {menuItems?.map(item => (
        <MenuCard key={item.id} item={item} />
      ))}
    </div>
  );
}
```

### 3. Предварительная загрузка данных

```tsx
// components/Navigation.tsx
import { dataPreloader } from '@/lib/preloader';

export function Navigation() {
  const handleMenuHover = () => {
    // Предзагружаем данные меню при наведении
    dataPreloader.preloadMultiple({
      dataTypes: ['menu-items', 'categories'],
      background: true,
      delay: 500
    });
  };

  return (
    <nav>
      <a 
        href="/menu" 
        onMouseEnter={handleMenuHover}
      >
        Меню
      </a>
    </nav>
  );
}
```

## 🔧 Конфигурация

### Настройки кэша

```typescript
// lib/cache.ts - настройки по умолчанию
const DEFAULT_CONFIG = {
  defaultTTL: 5 * 60 * 1000,     // 5 минут
  maxSize: 100,                   // максимум элементов
  cleanupInterval: 60 * 1000,     // очистка каждую минуту
  enableLogging: false            // логирование операций
};
```

### Настройки провайдера

```tsx
<CacheProvider
  // Автоматическая предзагрузка основных данных
  autoPreload={true}
  
  // Показать монитор кэша в dev режиме
  showMonitor={process.env.NODE_ENV === 'development'}
  
  // Интервал очистки кэша (мс)
  cleanupInterval={5 * 60 * 1000}
  
  // Максимальный размер кэша
  maxCacheSize={150}
  
  // Включить логирование
  enableLogging={false}
>
```

## 📊 Мониторинг производительности

### CacheMonitor компонент

В режиме разработки автоматически отображается монитор кэша:

- 📈 **Размер кэша** - количество закэшированных элементов
- 🎯 **Hit Rate** - процент попаданий в кэш
- 🔄 **Общие запросы** - общее количество запросов
- ✅ **Попадания** - количество успешных попаданий в кэш
- 🧹 **Последняя очистка** - время последней очистки кэша

### Программный доступ к статистике

```tsx
import { useCache } from '@/providers/CacheProvider';

function MyComponent() {
  const { 
    cacheSize, 
    cacheHitRate, 
    totalRequests, 
    cacheHits,
    clearCache,
    invalidateCache 
  } = useCache();

  return (
    <div>
      <p>Размер кэша: {cacheSize}</p>
      <p>Hit Rate: {cacheHitRate}%</p>
      <button onClick={clearCache}>Очистить кэш</button>
    </div>
  );
}
```

## 🎨 Компоненты состояний загрузки

### Доступные компоненты

```tsx
import { 
  LoadingState,     // Универсальный компонент загрузки
  ErrorState,       // Состояние ошибки
  EmptyState,       // Пустое состояние
  ProgressIndicator // Индикатор прогресса
} from '@/components/LoadingStates';

// Использование
<LoadingState type="menu" />        // Скелетон для меню
<LoadingState type="events" />      // Скелетон для событий
<LoadingState type="reviews" />     // Скелетон для отзывов
<LoadingState type="categories" />  // Скелетон для категорий

<ErrorState 
  message="Ошибка загрузки данных" 
  onRetry={() => refetch()}
/>

<EmptyState 
  title="Нет данных"
  description="Попробуйте изменить фильтры"
  action={<button>Сбросить фильтры</button>}
/>
```

### Хук управления состояниями

```tsx
import { useLoadingState } from '@/components/LoadingStates';

function MyComponent() {
  const { 
    isLoading, 
    error, 
    isEmpty, 
    setLoading, 
    setError, 
    clearError 
  } = useLoadingState();

  // Автоматическое управление состояниями
}
```

## 🔄 API хуки

### Базовый хук useApiState

```tsx
import { useApiState } from '@/hooks/useApiState';

function useCustomData() {
  return useApiState({
    queryKey: 'custom-data',
    queryFn: () => fetch('/api/custom').then(res => res.json()),
    ttl: 10 * 60 * 1000, // 10 минут
    staleTime: 5 * 60 * 1000, // 5 минут
    retryAttempts: 3,
    retryDelay: 1000
  });
}
```

### Специализированные хуки для Strapi

```tsx
import { 
  useMenuItems,
  useMenuCategories,
  useMenuItemsByCategory,
  useEvents,
  useEventBySlug,
  useReviews 
} from '@/hooks/useStrapi';

// Использование
const { data: menuItems, isLoading, error, refetch } = useMenuItems();
const { data: categories } = useMenuCategories();
const { data: categoryItems } = useMenuItemsByCategory('desserts');
const { data: events } = useEvents();
const { data: event } = useEventBySlug('summer-party');
const { data: reviews } = useReviews();
```

## 🚀 Оптимизация производительности

### 1. Предварительная загрузка

```tsx
// Предзагрузка при наведении
const handleHover = () => {
  dataPreloader.preload('menu-items', { background: true });
};

// Предзагрузка при появлении в viewport
const { ref } = useIntersectionPreloader({
  dataTypes: ['upcoming-events'],
  threshold: 0.1,
  rootMargin: '100px'
});

// Массовая предзагрузка
dataPreloader.preloadMultiple({
  dataTypes: ['menu-items', 'categories', 'reviews'],
  background: true,
  delay: 1000
});
```

### 2. Настройка TTL

```tsx
// Короткий TTL для часто изменяющихся данных
const { data } = useApiState({
  queryKey: 'live-data',
  queryFn: fetchLiveData,
  ttl: 30 * 1000 // 30 секунд
});

// Длинный TTL для статичных данных
const { data } = useApiState({
  queryKey: 'static-data',
  queryFn: fetchStaticData,
  ttl: 60 * 60 * 1000 // 1 час
});
```

### 3. Управление размером кэша

```tsx
// Ограничение размера кэша
const cache = new MemoryCache({
  maxSize: 50,              // максимум 50 элементов
  cleanupInterval: 60000,   // очистка каждую минуту
  enableLogging: true       // логирование операций
});
```

## 🐛 Отладка

### Включение логирования

```tsx
// В CacheProvider
<CacheProvider enableLogging={true}>

// Или в отдельном кэше
const cache = new MemoryCache({ enableLogging: true });
```

### Мониторинг в DevTools

```tsx
// Добавить в компонент для отладки
if (process.env.NODE_ENV === 'development') {
  console.log('Cache stats:', {
    size: cache.size(),
    hitRate: cache.getHitRate(),
    keys: cache.keys()
  });
}
```

### Проверка состояния кэша

```tsx
import { useCache } from '@/providers/CacheProvider';

function DebugPanel() {
  const { cache } = useCache();
  
  const debugInfo = {
    cacheSize: cache.size(),
    cacheKeys: cache.keys(),
    hitRate: cache.getHitRate(),
    stats: cache.getStats()
  };
  
  return <pre>{JSON.stringify(debugInfo, null, 2)}</pre>;
}
```

## 📝 Лучшие практики

### 1. Структура ключей кэша

```tsx
// ✅ Хорошо - описательные ключи
'menu-items'
'menu-categories'
'event-summer-party'
'reviews-page-1'

// ❌ Плохо - неописательные ключи
'data1'
'items'
'stuff'
```

### 2. Управление TTL

```tsx
// ✅ Хорошо - разный TTL для разных типов данных
const TTL_CONFIG = {
  STATIC_DATA: 60 * 60 * 1000,    // 1 час
  DYNAMIC_DATA: 5 * 60 * 1000,    // 5 минут
  LIVE_DATA: 30 * 1000,           // 30 секунд
  USER_DATA: 10 * 60 * 1000       // 10 минут
};
```

### 3. Обработка ошибок

```tsx
// ✅ Хорошо - graceful degradation
const { data, isLoading, error } = useMenuItems();

if (error) {
  return (
    <ErrorState 
      message="Не удалось загрузить меню"
      onRetry={() => refetch()}
    />
  );
}
```

### 4. Предварительная загрузка

```tsx
// ✅ Хорошо - предзагрузка на основе пользовательского поведения
const handleNavigation = (route: string) => {
  if (route === '/menu') {
    dataPreloader.preload('menu-items');
    dataPreloader.preload('menu-categories');
  }
};
```

## 🔧 Расширение системы

### Добавление нового типа данных

```tsx
// 1. Добавить в useStrapi.ts
export function useRestaurantInfo() {
  return useApiState({
    queryKey: 'restaurant-info',
    queryFn: () => cachedFetch('/api/restaurant/info'),
    ttl: 60 * 60 * 1000 // 1 час для статичной информации
  });
}

// 2. Добавить в preloader.ts
const DATA_ENDPOINTS = {
  'restaurant-info': '/api/restaurant/info',
  // ... другие endpoints
};

// 3. Добавить скелетон в LoadingStates.tsx
function RestaurantInfoSkeleton() {
  return (
    <div className="animate-pulse space-y-4">
      <div className="h-6 bg-gray-200 rounded w-1/2"></div>
      <div className="h-4 bg-gray-200 rounded w-3/4"></div>
    </div>
  );
}
```

### Кастомные стратегии кэширования

```tsx
// Создание специализированного кэша
class ApiCache extends MemoryCache {
  constructor() {
    super({
      maxSize: 200,
      defaultTTL: 10 * 60 * 1000,
      cleanupInterval: 2 * 60 * 1000
    });
  }
  
  // Кастомная логика инвалидации
  invalidateByPattern(pattern: string) {
    const keys = this.keys().filter(key => key.includes(pattern));
    keys.forEach(key => this.delete(key));
  }
}
```

## 📈 Метрики производительности

### Отслеживание ключевых метрик

```tsx
// Компонент для отслеживания метрик
function PerformanceMetrics() {
  const { cache } = useCache();
  const [metrics, setMetrics] = useState({
    cacheHitRate: 0,
    averageResponseTime: 0,
    totalRequests: 0
  });
  
  useEffect(() => {
    const interval = setInterval(() => {
      setMetrics({
        cacheHitRate: cache.getHitRate(),
        averageResponseTime: cache.getAverageResponseTime(),
        totalRequests: cache.getTotalRequests()
      });
    }, 5000);
    
    return () => clearInterval(interval);
  }, [cache]);
  
  return (
    <div className="metrics-panel">
      <div>Hit Rate: {metrics.cacheHitRate}%</div>
      <div>Avg Response: {metrics.averageResponseTime}ms</div>
      <div>Total Requests: {metrics.totalRequests}</div>
    </div>
  );
}
```

## 🚀 Развертывание

### Production настройки

```tsx
// app/layout.tsx
const isProduction = process.env.NODE_ENV === 'production';

<CacheProvider
  autoPreload={true}
  showMonitor={!isProduction}  // Скрыть монитор в продакшене
  cleanupInterval={isProduction ? 10 * 60 * 1000 : 5 * 60 * 1000}
  maxCacheSize={isProduction ? 200 : 100}
  enableLogging={!isProduction}
>
```

### Environment переменные

```bash
# .env.local
NEXT_PUBLIC_CACHE_TTL=300000          # 5 минут
NEXT_PUBLIC_CACHE_MAX_SIZE=150        # максимум элементов
NEXT_PUBLIC_CACHE_CLEANUP_INTERVAL=300000  # интервал очистки
NEXT_PUBLIC_ENABLE_CACHE_LOGGING=false     # логирование
```

## 📚 Дополнительные ресурсы

- 📖 [Подробная документация](./docs/CACHING_GUIDE.md)
- 🎯 [Примеры использования](./src/app/layout-with-cache.tsx)
- 🔧 [API Reference](./src/lib/cache.ts)
- 🎨 [Компоненты UI](./src/components/LoadingStates.tsx)

## 🤝 Поддержка

Для вопросов и предложений по улучшению системы кэширования:

1. Проверьте [документацию](./docs/CACHING_GUIDE.md)
2. Изучите [примеры использования](./src/app/layout-with-cache.tsx)
3. Включите логирование для отладки
4. Используйте CacheMonitor в режиме разработки

---

**Система кэширования готова к использованию! 🎉**

Интегрируйте ее в ваше приложение для значительного улучшения производительности и пользовательского опыта.