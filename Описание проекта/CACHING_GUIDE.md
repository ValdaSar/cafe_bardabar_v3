# Руководство по системе кэширования

Это руководство описывает систему кэширования, реализованную в проекте для оптимизации производительности и улучшения пользовательского опыта.

## Обзор системы

Система кэширования состоит из нескольких компонентов:

- **MemoryCache** - основной класс для управления кэшем в памяти
- **CachedFetch** - обертка для HTTP-запросов с кэшированием
- **DataPreloader** - система предварительной загрузки данных
- **CacheProvider** - React провайдер для управления кэшем
- **CacheMonitor** - компонент для мониторинга производительности

## Основные возможности

### 1. Автоматическое кэширование API запросов

```typescript
import { menuAPI } from '@/lib/strapi';

// Все запросы автоматически кэшируются
const menuItems = await menuAPI.getMenuItems();
const categories = await menuAPI.getCategories();
```

### 2. Настраиваемое время жизни (TTL)

```typescript
import { CACHE_TTL } from '@/lib/cache';

// Различные TTL для разных типов данных
CACHE_TTL.MENU = 15 * 60 * 1000;     // 15 минут
CACHE_TTL.EVENTS = 30 * 60 * 1000;   // 30 минут
CACHE_TTL.REVIEWS = 10 * 60 * 1000;  // 10 минут
```

### 3. Предварительная загрузка данных

```typescript
import { useDataPreloader } from '@/lib/preloader';

function MyComponent() {
  const { preloadEssentials, smartPreload } = useDataPreloader();
  
  useEffect(() => {
    // Загружаем основные данные
    preloadEssentials();
    
    // Умная предзагрузка на основе поведения пользователя
    smartPreload({
      visitedPages: ['/menu', '/events'],
      timeOnPage: 30000,
      scrollDepth: 75
    });
  }, []);
}
```

### 4. Управление кэшем через провайдер

```typescript
import { CacheProvider, useCache } from '@/providers/CacheProvider';

function App() {
  return (
    <CacheProvider autoPreload={true} showMonitor={true}>
      <MyApp />
    </CacheProvider>
  );
}

function MyComponent() {
  const { clearCache, invalidateCache, preloadData } = useCache();
  
  const handleRefresh = () => {
    invalidateCache('menu'); // Инвалидация кэша меню
    preloadData(['menu-items', 'categories']);
  };
}
```

## Компоненты состояния загрузки

### Скелетоны для различных типов контента

```typescript
import { 
  MenuItemSkeleton, 
  EventCardSkeleton, 
  ReviewSkeleton,
  GridLoadingState 
} from '@/components/LoadingStates';

function MenuPage() {
  const { data: menuItems, isLoading } = useMenuItems();
  
  if (isLoading) {
    return (
      <GridLoadingState 
        ItemSkeleton={MenuItemSkeleton}
        itemCount={6}
        columns={3}
      />
    );
  }
  
  return (
    <div className="grid grid-cols-3 gap-6">
      {menuItems.map(item => (
        <MenuItemCard key={item.id} item={item} />
      ))}
    </div>
  );
}
```

### Универсальные состояния

```typescript
import { LoadingState, ErrorState, EmptyState } from '@/components/LoadingStates';

function DataComponent() {
  const { data, isLoading, error } = useApiData();
  
  if (isLoading) {
    return <LoadingState message="Загружаем данные..." />;
  }
  
  if (error) {
    return (
      <ErrorState 
        message={error}
        onRetry={() => refetch()}
      />
    );
  }
  
  if (!data?.length) {
    return (
      <EmptyState 
        title="Нет данных"
        message="Попробуйте обновить страницу"
        action={{
          label: "Обновить",
          onClick: () => refetch()
        }}
      />
    );
  }
  
  return <DataList data={data} />;
}
```

## Мониторинг производительности

### CacheMonitor компонент

В режиме разработки автоматически отображается монитор кэша:

- Размер кэша
- Hit Rate (процент попаданий)
- Время последней очистки
- Кнопки управления (очистка, cleanup)
- Индикатор производительности

### Программный доступ к статистике

```typescript
import { cache } from '@/lib/cache';
import { dataPreloader } from '@/lib/preloader';

// Статистика кэша
console.log('Cache size:', cache.size());
console.log('Cache stats:', cache.getStats());

// Статистика предзагрузчика
console.log('Preloader stats:', dataPreloader.getStats());
```

## Оптимизация производительности

### 1. Настройка TTL для разных типов данных

```typescript
// Частообновляемые данные - короткий TTL
CACHE_TTL.REVIEWS = 5 * 60 * 1000;   // 5 минут

// Редкообновляемые данные - длинный TTL
CACHE_TTL.MENU = 60 * 60 * 1000;     // 1 час
```

### 2. Умная предварительная загрузка

```typescript
// Загрузка данных на основе пересечения элементов
import { intersectionPreloader } from '@/lib/preloader';

useEffect(() => {
  const element = document.getElementById('menu-section');
  if (element) {
    intersectionPreloader.observe(element, {
      dataTypes: ['menu-items', 'categories'],
      background: true
    });
  }
}, []);
```

### 3. Инвалидация кэша при изменениях

```typescript
import { cacheInvalidation } from '@/lib/cache';

// После создания нового отзыва
const createReview = async (reviewData) => {
  const result = await reviewsAPI.createReview(reviewData);
  
  // Инвалидируем кэш отзывов
  cacheInvalidation.invalidateByPattern('reviews');
  
  return result;
};
```

## Лучшие практики

### 1. Использование хуков с кэшированием

```typescript
// ✅ Правильно - используем оптимизированные хуки
const { data, isLoading, error, isStale } = useMenuItems();

// ❌ Неправильно - прямые вызовы API
const [data, setData] = useState(null);
useEffect(() => {
  menuAPI.getMenuItems().then(setData);
}, []);
```

### 2. Обработка устаревших данных

```typescript
function MenuComponent() {
  const { data, isLoading, isStale, refetch } = useMenuItems();
  
  return (
    <div>
      {isStale && (
        <div className="bg-yellow-100 p-2 rounded mb-4">
          Данные могут быть устаревшими.
          <button onClick={refetch}>Обновить</button>
        </div>
      )}
      {/* Контент */}
    </div>
  );
}
```

### 3. Предварительная загрузка критических данных

```typescript
// В корневом компоненте приложения
function App() {
  return (
    <CacheProvider 
      autoPreload={true}
      maxCacheSize={150}
      cleanupInterval={300000}
    >
      <Router>
        <Routes>
          {/* Маршруты */}
        </Routes>
      </Router>
    </CacheProvider>
  );
}
```

### 4. HOC для автоматической предзагрузки

```typescript
import { withPreload } from '@/providers/CacheProvider';

// Компонент с автоматической предзагрузкой
const MenuPageWithPreload = withPreload(MenuPage, [
  'menu-items',
  'categories',
  'special-menu'
]);
```

## Отладка и диагностика

### 1. Включение подробного логирования

```typescript
// В режиме разработки
if (process.env.NODE_ENV === 'development') {
  // Логирование всех операций с кэшем
  cache.enableDebugMode();
}
```

### 2. Анализ производительности

```typescript
// Измерение времени загрузки
const startTime = performance.now();
const data = await menuAPI.getMenuItems();
const loadTime = performance.now() - startTime;
console.log(`Load time: ${loadTime}ms`);
```

### 3. Мониторинг использования памяти

```typescript
// Проверка размера кэша
setInterval(() => {
  const size = cache.size();
  if (size > 100) {
    console.warn('Cache size is getting large:', size);
    cache.cleanup();
  }
}, 60000);
```

## Конфигурация

### Переменные окружения

```env
# .env.local
NEXT_PUBLIC_CACHE_ENABLED=true
NEXT_PUBLIC_CACHE_DEBUG=false
NEXT_PUBLIC_PRELOAD_ENABLED=true
```

### Настройка в коде

```typescript
// lib/cache.ts
export const CACHE_CONFIG = {
  MAX_SIZE: process.env.NODE_ENV === 'production' ? 200 : 50,
  CLEANUP_INTERVAL: 5 * 60 * 1000, // 5 минут
  DEBUG_MODE: process.env.NODE_ENV === 'development'
};
```

## Заключение

Система кэширования обеспечивает:

- **Улучшенную производительность** за счет кэширования API запросов
- **Лучший UX** благодаря предварительной загрузке и состояниям загрузки
- **Оптимизацию ресурсов** через умное управление кэшем
- **Простоту использования** с помощью готовых хуков и компонентов
- **Мониторинг** для отслеживания производительности

Используйте эти инструменты для создания быстрого и отзывчивого пользовательского интерфейса.