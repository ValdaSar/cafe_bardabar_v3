# 🎯 Примеры использования системы оптимизации изображений

## 🏠 Главная страница (Hero секция)

```tsx
// components/sections/HeroSection.tsx
import { OptimizedImage } from '@/components/ui/OptimizedImage';
import { preloadCriticalImages } from '@/lib/imagePreloader';
import { useEffect } from 'react';

export function HeroSection() {
  useEffect(() => {
    // Предзагружаем критические изображения
    preloadCriticalImages();
  }, []);

  return (
    <section className="relative h-screen flex items-center justify-center">
      {/* Фоновое изображение с высоким приоритетом */}
      <OptimizedImage
        src="/images/hero-bg.jpg"
        alt="Кафе Бардабар - уютная атмосфера"
        width={1920}
        height={1080}
        priority={true}
        className="absolute inset-0 w-full h-full object-cover"
        sizes="100vw"
        trackPerformance={true}
        adaptiveQuality={true}
      />
      
      {/* Контент поверх изображения */}
      <div className="relative z-10 text-center text-white">
        <h1 className="text-5xl font-bold mb-4">Добро пожаловать в Бардабар</h1>
        <p className="text-xl">Лучший кофе и атмосфера в городе</p>
      </div>
    </section>
  );
}
```

## 🍽️ Секция меню

```tsx
// components/sections/MenuSection.tsx
import { LazyImage } from '@/components/ui/LazyImage';
import { OptimizedImage } from '@/components/ui/OptimizedImage';
import { useState, useEffect } from 'react';
import { preloadImages } from '@/lib/imagePreloader';

interface MenuItem {
  id: string;
  name: string;
  description: string;
  price: number;
  image: string;
  category: string;
}

interface MenuSectionProps {
  items: MenuItem[];
}

export function MenuSection({ items }: MenuSectionProps) {
  const [selectedCategory, setSelectedCategory] = useState('coffee');
  
  // Предзагружаем изображения активной категории
  useEffect(() => {
    const categoryImages = items
      .filter(item => item.category === selectedCategory)
      .map(item => item.image)
      .slice(0, 6); // Первые 6 изображений
    
    preloadImages(categoryImages);
  }, [selectedCategory, items]);

  const filteredItems = items.filter(item => item.category === selectedCategory);

  return (
    <section className="py-16 bg-gray-50">
      <div className="container mx-auto px-4">
        <h2 className="text-4xl font-bold text-center mb-12">Наше меню</h2>
        
        {/* Категории */}
        <div className="flex justify-center mb-8">
          {['coffee', 'food', 'desserts'].map(category => (
            <button
              key={category}
              onClick={() => setSelectedCategory(category)}
              className={`px-6 py-2 mx-2 rounded-full ${
                selectedCategory === category
                  ? 'bg-brown-600 text-white'
                  : 'bg-white text-brown-600'
              }`}
            >
              {category === 'coffee' && 'Кофе'}
              {category === 'food' && 'Еда'}
              {category === 'desserts' && 'Десерты'}
            </button>
          ))}
        </div>

        {/* Сетка меню */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {filteredItems.map((item, index) => (
            <div key={item.id} className="bg-white rounded-lg shadow-lg overflow-hidden">
              {/* Изображение блюда */}
              <LazyImage
                src={item.image}
                alt={item.name}
                width={400}
                height={300}
                className="w-full h-48 object-cover"
                priority={index < 3} // Первые 3 изображения приоритетные
                rootMargin="100px" // Загружаем за 100px до появления
                threshold={0.1}
                placeholder="blur"
                onLoad={() => console.log(`Loaded: ${item.name}`)}
              />
              
              <div className="p-6">
                <h3 className="text-xl font-semibold mb-2">{item.name}</h3>
                <p className="text-gray-600 mb-4">{item.description}</p>
                <div className="flex justify-between items-center">
                  <span className="text-2xl font-bold text-brown-600">
                    {item.price} ₽
                  </span>
                  <button className="bg-brown-600 text-white px-4 py-2 rounded-lg hover:bg-brown-700">
                    Заказать
                  </button>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </section>
  );
}
```

## 📸 Галерея фотографий

```tsx
// components/sections/GallerySection.tsx
import { LazyImage } from '@/components/ui/LazyImage';
import { useState } from 'react';
import { useAdaptiveImageLoading } from '@/hooks/useImagePerformance';

interface GalleryImage {
  id: string;
  src: string;
  alt: string;
  category: 'interior' | 'food' | 'events';
}

interface GallerySectionProps {
  images: GalleryImage[];
}

export function GallerySection({ images }: GallerySectionProps) {
  const [selectedImage, setSelectedImage] = useState<string | null>(null);
  const [filter, setFilter] = useState<string>('all');
  const { shouldLoadHighQuality, getOptimalQuality } = useAdaptiveImageLoading();

  const filteredImages = images.filter(
    image => filter === 'all' || image.category === filter
  );

  return (
    <section className="py-16">
      <div className="container mx-auto px-4">
        <h2 className="text-4xl font-bold text-center mb-12">Галерея</h2>
        
        {/* Фильтры */}
        <div className="flex justify-center mb-8">
          {['all', 'interior', 'food', 'events'].map(category => (
            <button
              key={category}
              onClick={() => setFilter(category)}
              className={`px-4 py-2 mx-2 rounded ${
                filter === category
                  ? 'bg-brown-600 text-white'
                  : 'bg-gray-200 text-gray-700'
              }`}
            >
              {category === 'all' && 'Все'}
              {category === 'interior' && 'Интерьер'}
              {category === 'food' && 'Еда'}
              {category === 'events' && 'События'}
            </button>
          ))}
        </div>

        {/* Сетка изображений */}
        <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
          {filteredImages.map((image, index) => (
            <div
              key={image.id}
              className="relative aspect-square cursor-pointer group overflow-hidden rounded-lg"
              onClick={() => setSelectedImage(image.src)}
            >
              <LazyImage
                src={image.src}
                alt={image.alt}
                width={300}
                height={300}
                className="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                priority={index < 8} // Первые 8 изображений
                rootMargin="50px"
                threshold={0.1}
                quality={shouldLoadHighQuality ? 90 : getOptimalQuality()}
              />
              
              {/* Оверлей при наведении */}
              <div className="absolute inset-0 bg-black bg-opacity-0 group-hover:bg-opacity-30 transition-all duration-300 flex items-center justify-center">
                <span className="text-white opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  Увеличить
                </span>
              </div>
            </div>
          ))}
        </div>

        {/* Модальное окно для увеличенного изображения */}
        {selectedImage && (
          <div
            className="fixed inset-0 bg-black bg-opacity-80 flex items-center justify-center z-50"
            onClick={() => setSelectedImage(null)}
          >
            <div className="relative max-w-4xl max-h-4xl">
              <OptimizedImage
                src={selectedImage}
                alt="Увеличенное изображение"
                width={1200}
                height={800}
                className="max-w-full max-h-full object-contain"
                priority={true}
                quality={95}
              />
              
              <button
                onClick={() => setSelectedImage(null)}
                className="absolute top-4 right-4 text-white text-2xl bg-black bg-opacity-50 rounded-full w-10 h-10 flex items-center justify-center"
              >
                ×
              </button>
            </div>
          </div>
        )}
      </div>
    </section>
  );
}
```

## 👥 Секция отзывов

```tsx
// components/sections/ReviewsSection.tsx
import { OptimizedImage } from '@/components/ui/OptimizedImage';
import { LazyImage } from '@/components/ui/LazyImage';
import { useImagePerformance } from '@/hooks/useImagePerformance';

interface Review {
  id: string;
  name: string;
  avatar: string;
  rating: number;
  text: string;
  photos?: string[];
  date: string;
}

interface ReviewsSectionProps {
  reviews: Review[];
}

export function ReviewsSection({ reviews }: ReviewsSectionProps) {
  return (
    <section className="py-16 bg-gray-50">
      <div className="container mx-auto px-4">
        <h2 className="text-4xl font-bold text-center mb-12">Отзывы наших гостей</h2>
        
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {reviews.map((review, index) => (
            <ReviewCard key={review.id} review={review} priority={index < 3} />
          ))}
        </div>
      </div>
    </section>
  );
}

function ReviewCard({ review, priority }: { review: Review; priority: boolean }) {
  const { loadImage, metrics } = useImagePerformance(review.avatar, {
    preload: priority,
    trackMetrics: true,
  });

  return (
    <div className="bg-white rounded-lg shadow-lg p-6">
      {/* Заголовок с аватаром */}
      <div className="flex items-center mb-4">
        <OptimizedImage
          src={review.avatar}
          alt={`Аватар ${review.name}`}
          width={60}
          height={60}
          className="rounded-full mr-4"
          priority={priority}
          onLoad={() => loadImage()}
        />
        
        <div>
          <h3 className="font-semibold text-lg">{review.name}</h3>
          <div className="flex items-center">
            {[...Array(5)].map((_, i) => (
              <span
                key={i}
                className={`text-lg ${
                  i < review.rating ? 'text-yellow-400' : 'text-gray-300'
                }`}
              >
                ★
              </span>
            ))}
          </div>
        </div>
      </div>

      {/* Текст отзыва */}
      <p className="text-gray-700 mb-4">{review.text}</p>

      {/* Фотографии к отзыву */}
      {review.photos && review.photos.length > 0 && (
        <div className="grid grid-cols-2 gap-2 mb-4">
          {review.photos.slice(0, 4).map((photo, photoIndex) => (
            <LazyImage
              key={photoIndex}
              src={photo}
              alt={`Фото от ${review.name}`}
              width={150}
              height={150}
              className="rounded-lg object-cover"
              priority={priority && photoIndex < 2}
              rootMargin="100px"
            />
          ))}
        </div>
      )}

      {/* Дата */}
      <p className="text-sm text-gray-500">{review.date}</p>
      
      {/* Метрики производительности (только в development) */}
      {process.env.NODE_ENV === 'development' && metrics && (
        <div className="mt-2 text-xs text-gray-400">
          Загрузка аватара: {metrics.loadTime}ms
        </div>
      )}
    </div>
  );
}
```

## 🏢 Страница "О нас"

```tsx
// pages/about.tsx или app/about/page.tsx
import { OptimizedImage } from '@/components/ui/OptimizedImage';
import { LazyImage } from '@/components/ui/LazyImage';
import { useEffect } from 'react';
import { preloadImages } from '@/lib/imagePreloader';
import { usePageImagePerformance } from '@/hooks/useImagePerformance';

export default function AboutPage() {
  const { addMetrics, getPerformanceReport } = usePageImagePerformance();

  useEffect(() => {
    // Предзагружаем изображения для этой страницы
    preloadImages([
      '/images/about/team-photo.jpg',
      '/images/about/history-1.jpg',
      '/images/about/history-2.jpg',
    ]);

    // Отправляем отчет о производительности при уходе со страницы
    return () => {
      const report = getPerformanceReport();
      if (report.totalImages > 0) {
        console.log('About page image performance:', report);
        // Здесь можно отправить метрики в аналитику
      }
    };
  }, []);

  return (
    <div className="min-h-screen">
      {/* Hero секция */}
      <section className="relative h-96 flex items-center justify-center">
        <OptimizedImage
          src="/images/about/hero-bg.jpg"
          alt="История кафе Бардабар"
          width={1920}
          height={600}
          priority={true}
          className="absolute inset-0 w-full h-full object-cover"
          sizes="100vw"
        />
        
        <div className="relative z-10 text-center text-white">
          <h1 className="text-5xl font-bold mb-4">Наша история</h1>
          <p className="text-xl">Более 10 лет создаем уютную атмосферу</p>
        </div>
      </section>

      {/* Основной контент */}
      <section className="py-16">
        <div className="container mx-auto px-4">
          <div className="grid grid-cols-1 lg:grid-cols-2 gap-12 items-center">
            {/* Текст */}
            <div>
              <h2 className="text-3xl font-bold mb-6">Как все начиналось</h2>
              <p className="text-lg text-gray-700 mb-6">
                В 2013 году мы открыли небольшое кафе с большой мечтой - 
                создать место, где каждый гость чувствует себя как дома.
              </p>
              <p className="text-lg text-gray-700 mb-6">
                За годы работы мы выросли, но наши ценности остались прежними: 
                качественный кофе, свежая выпечка и теплая атмосфера.
              </p>
            </div>

            {/* Изображение */}
            <div>
              <LazyImage
                src="/images/about/history-1.jpg"
                alt="Первое кафе Бардабар в 2013 году"
                width={600}
                height={400}
                className="rounded-lg shadow-lg"
                priority={false}
                rootMargin="50px"
                onLoad={(metrics) => addMetrics(metrics)}
              />
            </div>
          </div>
        </div>
      </section>

      {/* Команда */}
      <section className="py-16 bg-gray-50">
        <div className="container mx-auto px-4">
          <h2 className="text-3xl font-bold text-center mb-12">Наша команда</h2>
          
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
            {teamMembers.map((member, index) => (
              <div key={member.id} className="text-center">
                <LazyImage
                  src={member.photo}
                  alt={member.name}
                  width={250}
                  height={250}
                  className="rounded-full mx-auto mb-4"
                  priority={index < 3}
                  rootMargin="100px"
                />
                
                <h3 className="text-xl font-semibold mb-2">{member.name}</h3>
                <p className="text-gray-600 mb-2">{member.position}</p>
                <p className="text-sm text-gray-500">{member.description}</p>
              </div>
            ))}
          </div>
        </div>
      </section>
    </div>
  );
}

const teamMembers = [
  {
    id: '1',
    name: 'Анна Петрова',
    position: 'Основатель и шеф-повар',
    photo: '/images/team/anna.jpg',
    description: 'Создатель уникальных рецептов и вдохновитель команды',
  },
  {
    id: '2',
    name: 'Михаил Сидоров',
    position: 'Бариста',
    photo: '/images/team/mikhail.jpg',
    description: 'Мастер кофейного искусства с 8-летним опытом',
  },
  {
    id: '3',
    name: 'Елена Козлова',
    position: 'Менеджер',
    photo: '/images/team/elena.jpg',
    description: 'Заботится о том, чтобы каждый гость остался доволен',
  },
];
```

## 📱 Адаптивная загрузка для мобильных устройств

```tsx
// components/MobileOptimizedGallery.tsx
import { LazyImage } from '@/components/ui/LazyImage';
import { useAdaptiveImageLoading } from '@/hooks/useImagePerformance';
import { useState, useEffect } from 'react';

export function MobileOptimizedGallery({ images }: { images: string[] }) {
  const { shouldLoadHighQuality, getOptimalQuality, connectionType } = useAdaptiveImageLoading();
  const [loadedCount, setLoadedCount] = useState(0);
  const [isMobile, setIsMobile] = useState(false);

  useEffect(() => {
    setIsMobile(window.innerWidth < 768);
  }, []);

  // Определяем количество изображений для загрузки в зависимости от соединения
  const getLoadLimit = () => {
    if (connectionType === 'slow-2g' || connectionType === '2g') return 4;
    if (connectionType === '3g') return 8;
    return 12; // 4g или wifi
  };

  const loadLimit = getLoadLimit();
  const visibleImages = images.slice(0, Math.min(loadedCount + 4, loadLimit));

  return (
    <div className="space-y-4">
      {/* Информация о соединении */}
      {process.env.NODE_ENV === 'development' && (
        <div className="text-sm text-gray-500 p-2 bg-gray-100 rounded">
          Соединение: {connectionType}, Качество: {getOptimalQuality()}%, 
          Лимит: {loadLimit} изображений
        </div>
      )}

      {/* Сетка изображений */}
      <div className={`grid gap-2 ${
        isMobile ? 'grid-cols-2' : 'grid-cols-3 md:grid-cols-4'
      }`}>
        {visibleImages.map((image, index) => (
          <LazyImage
            key={index}
            src={image}
            alt={`Галерея изображение ${index + 1}`}
            width={isMobile ? 150 : 200}
            height={isMobile ? 150 : 200}
            className="w-full aspect-square object-cover rounded-lg"
            priority={index < 2}
            quality={shouldLoadHighQuality ? 85 : getOptimalQuality()}
            rootMargin="100px"
            onLoad={() => {
              if (index === visibleImages.length - 1 && loadedCount < loadLimit - 4) {
                setLoadedCount(prev => prev + 4);
              }
            }}
          />
        ))}
      </div>

      {/* Кнопка "Загрузить еще" */}
      {loadedCount < images.length && loadedCount < loadLimit && (
        <div className="text-center">
          <button
            onClick={() => setLoadedCount(prev => prev + 4)}
            className="bg-brown-600 text-white px-6 py-2 rounded-lg hover:bg-brown-700"
          >
            Загрузить еще
          </button>
        </div>
      )}

      {/* Предупреждение о медленном соединении */}
      {(connectionType === 'slow-2g' || connectionType === '2g') && (
        <div className="text-center text-sm text-orange-600 bg-orange-50 p-3 rounded">
          ⚠️ Обнаружено медленное соединение. Изображения загружаются в сниженном качестве.
        </div>
      )}
    </div>
  );
}
```

## 🔄 Предзагрузка для навигации

```tsx
// hooks/useRoutePreloading.ts
import { useRouter } from 'next/router';
import { useEffect } from 'react';
import { preloadImages } from '@/lib/imagePreloader';

// Карта маршрутов и их критических изображений
const ROUTE_IMAGES = {
  '/': [
    '/images/hero-bg.jpg',
    '/images/menu-preview.jpg',
  ],
  '/menu': [
    '/images/menu/coffee-1.jpg',
    '/images/menu/coffee-2.jpg',
    '/images/menu/food-1.jpg',
  ],
  '/about': [
    '/images/about/hero-bg.jpg',
    '/images/about/team-photo.jpg',
  ],
  '/gallery': [
    '/images/gallery/interior-1.jpg',
    '/images/gallery/interior-2.jpg',
  ],
};

export function useRoutePreloading() {
  const router = useRouter();

  useEffect(() => {
    // Предзагружаем изображения для текущего маршрута
    const currentImages = ROUTE_IMAGES[router.pathname as keyof typeof ROUTE_IMAGES];
    if (currentImages) {
      preloadImages(currentImages);
    }

    // Предзагружаем изображения при наведении на ссылки
    const handleLinkHover = (e: MouseEvent) => {
      const target = e.target as HTMLElement;
      const link = target.closest('a[href]') as HTMLAnchorElement;
      
      if (link && link.href) {
        const url = new URL(link.href);
        const pathname = url.pathname;
        const images = ROUTE_IMAGES[pathname as keyof typeof ROUTE_IMAGES];
        
        if (images) {
          // Предзагружаем с небольшой задержкой
          setTimeout(() => preloadImages(images), 100);
        }
      }
    };

    document.addEventListener('mouseover', handleLinkHover);
    return () => document.removeEventListener('mouseover', handleLinkHover);
  }, [router.pathname]);
}
```

## 📊 Интеграция с аналитикой

```tsx
// utils/imageAnalytics.ts
import { usePageImagePerformance } from '@/hooks/useImagePerformance';
import { useEffect } from 'react';

export function useImageAnalytics() {
  const { getPerformanceReport } = usePageImagePerformance();

  useEffect(() => {
    // Отправляем метрики при уходе со страницы
    const handleBeforeUnload = () => {
      const report = getPerformanceReport();
      
      if (report.totalImages > 0) {
        // Отправляем в Google Analytics
        if (typeof gtag !== 'undefined') {
          gtag('event', 'image_performance', {
            event_category: 'Performance',
            event_label: window.location.pathname,
            value: Math.round(report.averageLoadTime),
            custom_map: {
              total_images: report.totalImages,
              success_rate: Math.round(report.successRate * 100),
              cache_hit_rate: Math.round(report.cacheHitRate * 100),
            },
          });
        }

        // Отправляем в собственную аналитику
        fetch('/api/analytics/images', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            pathname: window.location.pathname,
            ...report,
            timestamp: Date.now(),
          }),
          keepalive: true, // Важно для отправки при закрытии страницы
        }).catch(console.error);
      }
    };

    window.addEventListener('beforeunload', handleBeforeUnload);
    return () => window.removeEventListener('beforeunload', handleBeforeUnload);
  }, [getPerformanceReport]);
}

// Использование в _app.tsx или layout.tsx
export function AppWithAnalytics({ children }: { children: React.ReactNode }) {
  useImageAnalytics();
  
  return <>{children}</>;
}
```

Эти примеры показывают, как эффективно использовать систему оптимизации изображений в различных сценариях, от простых галерей до сложных адаптивных интерфейсов с мониторингом производительности.