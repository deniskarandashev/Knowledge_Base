# 📖 JavaScript & Angular — Содержание курса

> Полноценное учебное пособие для Java/Spring-разработчиков, переходящих во фронтенд.
> 🎯 — тема, популярная на собеседованиях.

---

## Модуль 1. Основы JavaScript

- Переменные: `let`, `const`, `var` — различия и подводные камни
- 🎯 Hoisting — подъём объявлений
- Типы данных: примитивы, объекты, `typeof`
- 🎯 Приведение типов (Type Coercion) — неявные преобразования
- 🎯 Операторы сравнения: `==` vs `===`
- Логические операторы: `&&`, `||`, `??` (Nullish Coalescing)
- Truthy / Falsy значения
- Strict mode

## Модуль 2. Функции

- Три способа создать функцию: declaration, expression, arrow
- Области видимости и лексическое окружение
- 🎯 Замыкания (Closures) — как и зачем
- 🎯 Контекст `this` — правила определения, потеря контекста
- `call`, `apply`, `bind` — явное управление `this`
- 🎯 Стрелочные функции vs обычные — полный разбор
- IIFE и Module Pattern
- Higher-order functions, мемоизация

## Модуль 3. Объекты

- Объекты: создание, свойства, вычисляемые ключи, shorthand
- 🎯 Ссылки vs значения: примитивы и объекты
- 🎯 Копирование объектов: shallow vs deep clone, `structuredClone`
- Методы объекта и `this` в методах
- Опциональная цепочка `?.`
- 🎯 Прототипы и `__proto__` — прототипная цепочка
- 🎯 Прототипное наследование, `Object.create`
- 🎯 Дескрипторы свойств: `enumerable`, `configurable`, `writable`
- Геттеры и сеттеры
- `Symbol`, итераторы, `Symbol.iterator`

## Модуль 4. Классы

- Синтаксис классов ES6+: `class`, `constructor`, методы
- 🎯 Классы как синтаксический сахар над прототипами
- Наследование: `extends`, `super`
- Статические методы и свойства
- Приватные поля (`#`) и методы
- 🎯 `instanceof`, проверка типов
- Mixins — альтернатива множественному наследованию
- 🔗 Классы в Angular: компоненты, сервисы, DI, декораторы

## Модуль 5. Структуры данных

- Массивы: полный обзор методов (мутирующие vs иммутабельные)
- 🎯 Деструктуризация массивов и объектов
- 🎯 Spread и Rest операторы
- `Map` и `Set` (vs объекты и массивы)
- 🎯 `WeakMap` и `WeakSet` — слабые ссылки и GC
- Итераторы и `Symbol.iterator`
- Строки: методы, шаблонные литералы, tagged templates
- JSON: `parse`, `stringify`, `toJSON`, `reviver`
- `Date` и `Intl` (основы)
- 🆕 Иммутабельные методы: `toSorted`, `toReversed`, `Object.groupBy`

## Модуль 6. Асинхронность

- Однопоточность JavaScript — почему это не проблема
- 🎯🎯🎯 Event Loop — macrotasks, microtasks, очереди
- `setTimeout` / `setInterval` — macrotasks
- `requestAnimationFrame` — таймер для визуальных обновлений
- Callback Hell и зачем нужны промисы
- 🎯🎯 Promises — создание, `then`/`catch`/`finally`, цепочки
- `Promise.all`, `Promise.race`, `Promise.allSettled`, `Promise.any`
- 🎯🎯 `async/await` — синтаксический сахар
- Error handling: `try/catch`, `.catch()`, unhandled rejection
- `AbortController` и `AbortSignal.timeout`
- 🔗 От Promise к Observable (мост в Angular/RxJS)

## Модуль 7. Модули и сборка

- Эволюция модульности: IIFE → CommonJS → AMD → ESM
- 🎯 ES Modules: `import`/`export`, named vs default
- Live bindings — живые привязки (read-only для потребителя)
- 🎯 Dynamic `import()` и code splitting
- CommonJS: `require`/`module.exports` (Node.js)
- Import Maps — native module resolution в браузере
- Bundlers: концепции Webpack, Vite (dev ≠ prod)
- 🎯 Tree-shaking — как работает и почему зависит от ESM
- 🔗 Современный Angular build system: application builder, esbuild
- Lazy loading в Angular: `loadComponent` / standalone routes

## Модуль 8. Браузер и DOM

- Браузерное окружение: `window`, `document`, `navigator`
- DOM-дерево: узлы, типы, навигация (`children` vs `childNodes`)
- Поиск элементов: `getElementById`, `querySelector`, `querySelectorAll`
- Модификация DOM: `createElement`, `append`, `innerHTML` vs `textContent`
- Атрибуты vs свойства — различия HTML и DOM
- Стили: `classList`, `style`, `getComputedStyle`
- 🎯 События: `addEventListener`, всплытие и погружение (bubbling/capturing)
- 🎯 Делегирование событий (event delegation)
- 🎯 Debounce и Throttle
- Passive listeners, `{ passive: true }`
- ♿ Доступность (a11y): семантический HTML, ARIA, фокус
- Shadow DOM и `composedPath()`
- `MutationObserver`, `IntersectionObserver`, `ResizeObserver`

## Модуль 9. Сетевые запросы и хранение данных

- 🎯 Fetch API: `fetch()`, `Request`, `Response`, `Headers`
- 🎯 CORS: Same-Origin Policy, preflight, simple requests
- Credentials и cookies в fetch-запросах
- `AbortController` / `AbortSignal.timeout` — отмена запросов
- `FormData`, `URL`, `URLSearchParams`
- `XMLHttpRequest` (legacy, для понимания)
- WebSocket — полнодуплексное соединение
- Server-Sent Events (SSE) — ограничения `EventSource`
- 🎯 `localStorage` / `sessionStorage` / cookies — различия
- IndexedDB — клиентская БД
- Cache API и Service Workers (обзор)
- 🛠️ DevTools Network tab: диагностика CORS, preflight, cookies
- 🔗 Angular `HttpClient` и functional interceptors

## Модуль 10. JavaScript в контексте Angular

- Зачем Angular-разработчику глубокий JavaScript
- TypeScript как надмножество JS
- Декораторы: `@Component`, `@Injectable`, `@Input` — метаданные для DI
- 🎯 Zone.js — патчинг асинхронных API
- 🎯 Change Detection: Default vs OnPush
- `ExpressionChangedAfterItHasBeenCheckedError` — разбор
- 🎯 Dependency Injection — аналогия со Spring IoC
- Lifecycle hooks: `ngOnInit`, `ngOnDestroy`, `ngOnChanges`
- 🎯 RxJS: Observable, операторы, `pipe`, `switchMap`, `takeUntilDestroyed`
- 🎯 Signals: `signal()`, `computed()`, `effect()`
- Signal inputs/outputs: `input()`, `output()`, `model()`
- Современный control flow: `@if`, `@for`, `@defer`
- Standalone components, `imports`, `provideHttpClient`
- Zoneless Angular — будущее Change Detection
- SSR / Hydration (`@angular/ssr`)
- 🛠️ Angular DevTools — профайлер и отладка CD

---

## 📊 Статистика курса

| | Значение |
|---|---|
| Модулей | 10 |
| Общий объём | ~1.37 MB (~760 страниц) |
| 📚 Блоков с источниками | 327 |
| 🎯 Interview-тем | 30+ |
| Источники | MDN, javascript.info, ECMA-262, w3schools, Angular docs, RxJS docs, web.dev |
| Целевая аудитория | Java/Spring → Frontend (Angular) |

---

*Создано с помощью SKILL_Educational_Materials_Universal.md*
