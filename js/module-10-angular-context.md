# Модуль 10: JavaScript в контексте Angular

## Для кого этот модуль

Это финальный модуль курса. До этого ты уже прошёл длинную дорогу: в **Модуле 1** разобрался с типами, переменными и странностями приведения; в **Модуле 2** приручил функции, `this` и замыкания; в **Модуле 3** понял объекты, ссылки и прототипы; в **Модуле 4** увидел, что `class` в JavaScript — это красивая панель над старым прототипным мотором; в **Модуле 5** научился думать структурами данных и потоками значений; в **Модуле 6** прожевал event loop, Promise и `async/await`; в **Модуле 7** понял модули и сборку; в **Модуле 8** спустился в браузер, DOM и события; в **Модуле 9** разобрал сетевые запросы и хранилища. Теперь мы собираем всё это в одну инженерную картину.

Angular часто кажется магией ровно до того момента, пока не увидишь: под ним живёт всё тот же JavaScript. Никакого тайного второго языка нет. Есть браузер, DOM, события, очереди задач, ссылки на объекты, асинхронность, функции, модули и немного очень умной инфраструктуры поверх этого. Angular не отменяет механику JavaScript. Он её дисциплинирует.

Для Java/Spring-разработчика этот модуль особенно важен эмоционально. До него фронтенд может ощущаться как шумная комната, где всё происходит одновременно: шаблоны, биндинги, Observable, DI, Change Detection, странные ошибки с `ExpressionChangedAfterItHasBeenCheckedError`, а теперь ещё и Signals. Хорошая новость: это не хаос. Это система. И если ты поймёшь причинно-следственные связи, Angular начнёт восприниматься не как набор заклинаний, а как очень знакомая инженерная архитектура — просто в браузере.

Главная мысль модуля такая: **Angular — это JavaScript, организованный как фреймворк**. TypeScript даёт контракты. Декораторы размечают роли. DI поставляет зависимости. Zone.js подслушивает асинхронность. Change Detection решает, когда и что проверять. RxJS описывает потоки событий. Signals делают реактивность более точной и показывают, куда Angular движется дальше.

## 🎯 Interview Focus модуля

На собеседованиях из этого модуля особенно часто и особенно больно спрашивают пять тем:

1. **Zone.js** — что это такое и как оно связано с асинхронностью;
2. **Change Detection** — когда он запускается, чем `OnPush` отличается от `Default`;
3. **Observable vs Promise** — не только табличка, а понимание, почему RxJS вообще появился;
4. **Angular DI vs Spring DI** — показать, что ты видишь знакомую IoC-модель;
5. **Signals** — зачем Angular двигается в эту сторону и что это меняет.

Если ты умеешь рассказать эти темы спокойно, причинно-следственно и без магических фраз, ты уже звучишь как разработчик, который понимает Angular под капотом.

---

## 1. TypeScript как дисциплина для большого UI-кода

Если смотреть очень честно, Angular мог бы существовать и без TypeScript. Браузер ведь всё равно исполняет JavaScript. Но вопрос не в том, «можно ли», а в том, **какой ценой**. Когда приложение вырастает до десятков экранов, сотен компонентов и сервисов, код без контрактов начинает напоминать разговоры на кухне: пока все рядом, всё понятно; как только команда выросла, начинаются недопонимания, догадки и ошибки в рантайме.

Первая метафора: JavaScript без TypeScript — это стройка, где опытный мастер действительно может что-то собрать «на глаз». TypeScript — это не другой бетон и не другой кирпич. Это чертёж, смета и подписи на документах. Строить можно теми же руками, но вероятность поставить стену не туда резко ниже. Для Java-разработчика здесь очень родное ощущение: сигнатура — это договор.

Java-аналогия почти прямая: TypeScript даёт фронтенду ту же психологическую опору, которую Java даёт бэкенду. Но есть важное различие с миром JVM: в Java типы живут и в compiled-модели, и в reflection-runtime. В TypeScript большая часть типов исчезает после компиляции. Это не runtime-валидация. Это **compile-time discipline**. Поэтому Модуль 1 про типы и Модуль 9 про сеть всё ещё важны: сервер может прислать что угодно, и TypeScript не спасёт от кривых данных сам по себе.

```typescript
interface Task {
  id: number;
  title: string;
  done: boolean;
}

class TaskService {
  getOpenTitles(tasks: Task[]): string[] {
    return tasks
      .filter(task => !task.done)
      .map(task => task.title);
  }
}
```

### Разбор по шагам

1. `interface Task` задаёт контракт формы: у задачи должны быть `id`, `title`, `done`.
2. Этот интерфейс помогает редактору, компилятору и тебе самому понимать, что приходит в метод `getOpenTitles`.
3. Внутри метода используются знания из **Модуля 5**: `filter` и `map` строят маленький конвейер обработки данных.
4. Стрелочные функции внутри `filter` и `map` — это уже фундамент из **Модуля 2**.
5. После компиляции браузер увидит не `interface`, а обычный JavaScript-код с массивом и функциями.
6. Если сервер из **Модуля 9** пришлёт `{ id: "oops" }`, сам интерфейс не остановит браузер. Он только предупредит тебя на этапе разработки, если ты честно описал входные данные.

📚 **Источники:**
- [TypeScript Handbook: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript Handbook: Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)
- [Angular docs: Components](https://angular.dev/guide/components)


### 🎯 Interview Focus

Хороший ответ на интервью звучит так: **TypeScript нужен Angular не потому, что без него невозможно, а потому что он делает большие UI-приложения предсказуемыми: даёт контракты, автодополнение, безопасный рефакторинг и лучшее понимание API компонентов и сервисов. Но runtime-проверку данных он не делает автоматически.**

### ✅ Проверь себя

**1. Что главное даёт TypeScript поверх JavaScript?**
- A) Новый браузерный runtime
- B) Статические проверки и контракты на этапе разработки
- C) Многопоточность
- D) Встроенный HTTP-сервер

<details><summary>Ответ</summary>B. TypeScript усиливает этап разработки, а не создаёт новый runtime.</details>

**2. Почему TypeScript не спасает от кривого JSON с сервера сам по себе?**
- A) Потому что интерфейсы и типы в основном стираются после компиляции
- B) Потому что Angular не умеет читать JSON
- C) Потому что браузер запрещает типы
- D) Потому что `fetch` ломает generics

<details><summary>Ответ</summary>A. В runtime исполняется JavaScript, а не TypeScript-типы.</details>

---

## 2. Декораторы: наклейки, метаданные и язык для фреймворка

Когда человек впервые видит `@Component`, у него возникает ощущение, будто Angular по одной «собачке» сразу понимает всё на свете. На деле декоратор — это способ **разметить** класс или его части дополнительной информацией. Не магия, а ярлык. Сам по себе класс не превращается в кнопку, форму или экран. Но Angular получает карту: «Этот класс используй как компонент, вот его шаблон, вот селектор, вот какие зависимости ему нужны».

Первая метафора: класс — это чемодан, а декоратор — наклейка аэропорта. Наклейка не меняет форму чемодана и не делает его легче. Но благодаря ей система понимает, куда чемодан отправить. Точно так же `@Component` не делает рендер сам по себе. Он сообщает Angular, что этот класс — часть UI.

Вторая метафора: представь музей. Сам объект — это экспонат. Декоратор — табличка рядом: название, автор, зал, правила показа. Посетитель смотрит не только на статую, но и на контекст. Angular так же смотрит не только на класс, но и на прикреплённые метаданные. Для Java-разработчика это почти то же ощущение, что при чтении `@Service`, `@RestController`, `@Configuration` в Spring.

```typescript
import { Component, Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class LoggerService {
  log(message: string): void {
    console.log('[LOG]', message);
  }
}

@Component({
  selector: 'app-task-counter',
  standalone: true,
  imports: [],
  template: `<p>Открытых задач: {{ openCount }}</p>`
})
export class TaskCounterComponent {
  openCount = 3;

  constructor(private logger: LoggerService) {
    this.logger.log('TaskCounterComponent created');
  }
}
```

### Разбор по шагам

1. `@Injectable({ providedIn: 'root' })` сообщает Angular: этот класс можно создавать через DI, а его root-scoped экземпляр можно держать на уровне приложения.
2. `LoggerService` — обычный TypeScript-класс. Никакого наследования от «магического базового класса» нет.
3. `@Component(...)` приклеивает к `TaskCounterComponent` метаданные о селекторе и шаблоне.
4. `selector: 'app-task-counter'` говорит, под каким HTML-тегом компонент можно использовать.
5. `template` — это шаблон, где Angular связывает состояние класса и DOM. Это напрямую опирается на DOM-модель из **Модуля 8**.
6. Конструктор получает `LoggerService` через DI. Это мост к разделу про Angular DI и прямая параллель со Spring constructor injection.
7. Когда компонент создаётся, Angular использует метаданные и DI, чтобы собрать нужный объект и связать его с шаблоном.

### История типичной ошибки

Разработчик из Java видит `@Component`, радуется знакомому синтаксису и думает: «А, значит это просто как аннотация, ничего особенного». Потом он создаёт обычный класс-хелпер, пытается инжектить его в конструктор и удивляется, почему Angular не умеет его выдавать. История заканчивается фразой: «Но ведь это же просто класс». Именно. **Просто класс** — и чтобы Angular включил его в свою систему ролей и DI, ему нужно это сообщить.

📚 **Источники:**
- [TypeScript Handbook: Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)


📚 **Источники:**
- [TypeScript Handbook: Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)


### 🎯 Interview Focus

На интервью хороший ответ такой: **декораторы в Angular прикрепляют к классам и их частям метаданные, по которым фреймворк понимает, как использовать сущность. `@Component` описывает компонент, `@Injectable` — участника DI, `@Input` и `@Output` — точки связи снаружи. Это не магия браузера, а инфраструктурный слой Angular и TypeScript.**

### ✅ Проверь себя

**1. Что делает `@Component` по сути?**
- A) Создаёт поток для компонента
- B) Прикрепляет метаданные, чтобы Angular понял роль класса
- C) Сразу рендерит DOM без шаблона
- D) Заменяет DI

<details><summary>Ответ</summary>B. Декоратор даёт Angular информацию о роли и конфигурации класса.</details>

**2. Почему полезно сравнивать декораторы Angular с аннотациями Spring?**
- A) Потому что у них одинаковый байткод
- B) Потому что это похожий способ маркировать сущности для инфраструктуры
- C) Потому что Angular работает на JVM
- D) Потому что иначе нельзя использовать TypeScript

<details><summary>Ответ</summary>B. Идея маркировки ролей очень похожа, хотя среда исполнения разная.</details>

---

## 3. 🎯 Zone.js: невидимый шпион, прослушка проводов и ThreadLocal для асинхронности

Сейчас будет одна из самых трудных тем всего курса. Если Zone.js раньше казался тебе туманом, это нормально. Ты не один. Очень многие разработчики годами используют Angular и внутри чувствуют: «Я как будто знаю, что Zone.js существует, но объяснить не могу». Давай разложим это по-человечески.

### Зачем Angular вообще нужен такой механизм

Angular должен как-то понять, **когда после асинхронной работы стоит проверить интерфейс**. Пользователь кликнул кнопку, пришёл HTTP-ответ, сработал `setTimeout`, завершился `Promise`, произошёл event из DOM — всё это меняет состояние приложения. Если бы Angular ничего не знал про эти моменты, ему пришлось бы либо вообще не обновлять экран автоматически, либо заставлять тебя вручную после каждого async-кусочка говорить: «Дорогой Angular, проверь интерфейс».

Первая метафора — **невидимый шпион**. Zone.js сидит в офисе, ничего не решает по бизнес-логике, никому не мешает работать, но подслушивает, когда звонит телефон, приходит письмо, срабатывает будильник или кто-то нажимает кнопку вызова. Он не делает работу сотрудников. Он просто замечает: «О, сейчас что-то произошло. Возможно, менеджеру стоит ещё раз посмотреть на доску задач».

Вторая метафора — **wiretapping, то есть прослушка проводов**. Monkey-patching в контексте Zone.js — это не «сломать API», а аккуратно врезаться в провод. Телефонная линия остаётся той же, разговор идёт как шёл, но теперь в линии стоит оборудование, которое может зафиксировать: звонок начался, звонок закончился, вот кто его инициировал. `setTimeout`, `addEventListener`, `Promise.then`, XHR — это провода, в которые врезается Zone.js.

Третья метафора — **ThreadLocal для асинхронной логической цепочки**. В Java ты можешь держать контекст в `ThreadLocal` и знать, что пока работа идёт в рамках потока, контекст течёт вместе с ней. В JavaScript поток обычно один, но асинхронные куски работы разнесены во времени. Zone.js пытается сохранить ощущение логического контекста поверх этих прыжков: вот этот callback относится к той зоне, которая была активна в момент его регистрации.

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)


### Что такое monkey-patching здесь по-человечески

Monkey-patching — это оборачивание существующей функции другой функцией, чтобы добавить наблюдение или дополнительное поведение. Не переписывание браузера с нуля, а «надевание чехла» на API.

Представь обычный `setTimeout`. В нативном мире браузер получает callback, время ожидания и потом когда-то ставит этот callback в очередь задач. Zone.js говорит: «Отлично, но перед тем как передать callback дальше, я заверну его в свою обёртку, чтобы потом понять, кто и когда его выполняет».

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)


### Пошагово: что Zone.js делает с `setTimeout`

```javascript
setTimeout(() => {
  console.log('Timer finished');
}, 1000);
```

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)


### Разбор по шагам

1. Твой код вызывает `setTimeout`.
2. Если Zone.js подключён, то вызывается уже **patch-версия** `setTimeout`, а не исходная напрямую.
3. Patch-версия запоминает текущую зону — то есть контекст, внутри которого был вызов.
4. Она берёт твой callback `() => console.log(...)` и оборачивает его в другой callback.
5. Уже этот новый, обёрнутый callback передаётся настоящему браузерному `setTimeout`.
6. Браузер ждёт 1000 мс и потом помещает обёрнутый callback в task queue, как мы разбирали в **Модуле 6**.
7. Когда event loop добирается до callback, сначала исполняется обёртка Zone.js.
8. Обёртка восстанавливает нужную зону, запускает твой исходный callback, а потом сообщает системе: «Асинхронная задача завершилась».
9. Angular, услышав это уведомление, получает повод решить, надо ли запускать Change Detection.

Теперь посмотрим на псевдокод, чтобы прочувствовать идею.

```javascript
const originalSetTimeout = window.setTimeout;

window.setTimeout = function patchedSetTimeout(callback, delay) {
  const currentZone = Zone.current;

  const wrappedCallback = function () {
    return currentZone.run(callback);
  };

  return originalSetTimeout(wrappedCallback, delay);
};
```

### Разбор по шагам

1. `originalSetTimeout` сохраняет ссылку на настоящий браузерный API. Это важно: Zone.js не хочет потерять исходную функцию.
2. Затем `window.setTimeout` заменяется на patch-версию. Это и есть monkey-patching.
3. При каждом вызове patch-функция запоминает `Zone.current` — логический асинхронный контекст.
4. Оригинальный callback не уходит в браузер как есть. Он заворачивается в `wrappedCallback`.
5. Когда таймер срабатывает, вызывается сначала `wrappedCallback`.
6. `currentZone.run(callback)` исполняет твой callback уже внутри нужного контекста зоны.
7. В реальной Zone.js внутри происходит больше bookkeeping: учёт task'ов, ошибок, вложенных зон и событий завершения.
8. Но ментальная модель именно такая: **обернуть, запомнить, восстановить, уведомить**.

### Как это связано с Angular

Angular создаёт свою зону — `NgZone`. Когда асинхронная задача, связанная с Angular-зоной, заканчивается, Angular получает сигнал, что система может стать «стабильной», и в подходящий момент запускает Change Detection. Важно: Zone.js **не обновляет DOM сам**. Он не сравнивает значения и не ищет, что изменилось. Он просто даёт Angular возможность вовремя сказать: «Пора проверить».

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)


### История типичной ошибки

Один разработчик услышал фразу «Zone.js следит за асинхронностью» и сделал вывод: «Значит он запускает код вместо event loop». Дальше началась путаница: почему же тогда `setTimeout` всё ещё зависит от браузера? Почему microtask order такой же, как в чистом JS? Ответ простой и важный: **Zone.js не заменяет платформу, он подслушивает платформу**. Как камера в коридоре не начинает двигать людей по зданию, так и Zone.js не начинает управлять event loop за браузер.

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)


📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)


### 🎯 Interview Focus

Хороший короткий ответ такой: **Zone.js monkey-patch'ит async API браузера (`setTimeout`, `Promise`, DOM events, XHR и др.), чтобы Angular мог замечать завершение асинхронной работы и после этого при необходимости запускать Change Detection. Это не многопоточность и не прямое обновление DOM, а инфраструктурное наблюдение за асинхронностью.**

### ✅ Проверь себя

**1. Что такое monkey-patching в контексте Zone.js?**
- A) Создание новых браузерных потоков
- B) Оборачивание существующих API ради наблюдения за их вызовами
- C) Замена DOM на Virtual DOM
- D) Сжатие скриптов

<details><summary>Ответ</summary>B. Смысл в том, чтобы встроиться в существующий API и добавить наблюдение.</details>

**2. Что делает Zone.js после завершения `setTimeout` callback?**
- A) Сам обновляет DOM
- B) Даёт Angular сигнал, что после async-активности стоит проверить UI
- C) Блокирует event loop
- D) Превращает macrotask в microtask

<details><summary>Ответ</summary>B. Он не рисует экран сам, а сообщает инфраструктуре о завершении async-работы.</details>

**3. Какая Java-аналогия ближе всего к Zone.js?**
- A) `ThreadLocal` + AOP вокруг асинхронных границ
- B) `ArrayList`
- C) JPA EntityManager
- D) `StringBuilder`

<details><summary>Ответ</summary>A. Идея логического контекста и инфраструктурного перехвата очень близка.</details>

---

## 4. 🎯 Change Detection: как Angular проверяет мир и почему это похоже на Hibernate dirty checking

Если Zone.js отвечает на вопрос «когда вообще стоит насторожиться после async-активности», то Change Detection отвечает на другой вопрос: **что именно теперь нужно синхронизировать с экраном**. Это сердце Angular-рендеринга, и без него всё остальное будет звучать кусками.

### Сначала интуиция, потом детали

Ключевая метафора здесь — Java-аналогия с **Hibernate dirty checking**. В JPA ты меняешь состояние entity в памяти, а потом в подходящий момент инфраструктура смотрит: «Что изменилось? Нужно ли это синхронизировать с базой?» Angular делает не то же самое буквально, но идея очень похожа: есть уже известная модель состояния, есть фаза проверки, есть синхронизация внешнего мира — только вместо базы данных у нас DOM.

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: ChangeDetectionStrategy](https://angular.dev/api/core/ChangeDetectionStrategy)
- [Angular API: ChangeDetectorRef](https://angular.dev/api/core/ChangeDetectorRef)


### Глубокая аналогия с Hibernate dirty checking

Представь JPA-сессию. Ты загрузил `TaskEntity`, поменял `task.setDone(true)`, но SQL не улетел в базу в ту же наносекунду. Сначала состояние изменилось **в памяти**. Потом, на `flush` или commit, Hibernate проверяет managed entities и решает, какие SQL надо отправить.

В Angular очень похожая логика мышления:

- у компонента есть состояние в памяти JavaScript;
- шаблон — это декларация, как из этого состояния получается DOM;
- при Change Detection Angular проходит по компонентам и проверяет выражения в шаблоне;
- если видит, что результат отличается от того, что был раньше, обновляет DOM.

Сходство важно не в деталях реализации, а в способе думать. **Изменение состояния и синхронизация внешнего мира — это не одна и та же секунда.** Сначала ты меняешь объект, потом инфраструктура решает, как и когда синхронизировать представление.

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: ChangeDetectionStrategy](https://angular.dev/api/core/ChangeDetectionStrategy)
- [Angular API: ChangeDetectorRef](https://angular.dev/api/core/ChangeDetectorRef)


### Когда Angular запускает Change Detection

На этом месте нужно быть очень конкретным. Change Detection обычно запускается после событий, которые Angular умеет отследить:

- DOM-события: клик, input, submit;
- таймеры и callback'и вроде `setTimeout`, если они произошли внутри Angular zone;
- завершение `Promise` и других async-операций;
- HTTP-ответы через механизмы, которые Angular наблюдает;
- явные вызовы `markForCheck()`, `detectChanges()`;
- эмиссии Observable через `async` pipe;
- обновления signal-зависимостей в современной модели.

Связь с **Модулем 8** и **Модулем 6** здесь железная: DOM-события и async-задачи не исчезают, Angular просто навешивает сверху свою систему проверки.

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: ChangeDetectionStrategy](https://angular.dev/api/core/ChangeDetectionStrategy)
- [Angular API: ChangeDetectorRef](https://angular.dev/api/core/ChangeDetectorRef)


### `Default` vs `OnPush` на реальном дереве компонентов

Допустим, у нас есть дерево:

- `AppComponent`
  - `TaskPageComponent`
    - `TaskToolbarComponent`
    - `TaskListComponent`
      - `TaskItemComponent`
      - `TaskItemComponent`
      - `TaskItemComponent`
  - `SettingsPanelComponent`

При стратегии **Default** Angular после подходящего триггера охотнее проходит по широкому куску дерева. Это как очень старательный администратор, который после любого звонка идёт проверять весь этаж: и переговорку, и кухню, и склад, и ресепшен. Это просто, надёжно и удобно для старта, но на большом приложении может быть дороже, чем нужно.

При стратегии **OnPush** компонент ведёт себя как сотрудник, который говорит: «Меня не дёргай без ясного повода». Поводы такие:

- изменилась ссылка входного `@Input`;
- произошло событие внутри этого компонента или его view;
- Observable через `async` pipe прислал новое значение;
- вызван `markForCheck()`;
- сигнал, прочитанный шаблоном, изменился.

Ключевое слово тут — **reference**. Это мост прямо в **Модуль 3** про ссылки и объекты. Если ты мутируешь поле у старого объекта, ссылка не меняется. Если создаёшь новый объект через spread, ссылка новая. Для `OnPush` это огромная разница.

```typescript
@Component({
  selector: 'app-task-item',
  standalone: true,
  imports: [],
  template: `{{ task.title }} - {{ task.done ? 'done' : 'open' }}`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class TaskItemComponent {
  @Input() task!: Task;
}

@Component({
  selector: 'app-task-page',
  standalone: true,
  imports: [TaskItemComponent],
  template: `
    <app-task-item [task]="task"></app-task-item>
    <button (click)="mutateTask()">Mutate</button>
    <button (click)="replaceTask()">Replace</button>
  `
})
export class TaskPageComponent {
  task: Task = { id: 1, title: 'Learn Angular', done: false };

  mutateTask(): void {
    this.task.done = true;
  }

  replaceTask(): void {
    this.task = { ...this.task, done: true };
  }
}
```

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: ChangeDetectionStrategy](https://angular.dev/api/core/ChangeDetectionStrategy)
- [Angular API: ChangeDetectorRef](https://angular.dev/api/core/ChangeDetectorRef)


### Разбор по шагам

1. `TaskItemComponent` использует `OnPush`, значит он предпочитает проверяться по явным сигналам, а не на каждый широкий обход.
2. В `TaskPageComponent` есть объект `task`, который передаётся вниз через `@Input`.
3. Метод `mutateTask()` меняет внутреннее поле `done`, но сам объект остаётся тем же объектом в памяти.
4. Для `OnPush` это проблема: новая ссылка не появилась, а значит один из главных триггеров не сработал.
5. Метод `replaceTask()` создаёт новый объект через spread из **Модуля 5**.
6. Теперь ссылка у `task` новая, и `OnPush`-компонент получает ясный сигнал: входное значение изменилось.
7. Именно поэтому `OnPush` очень любит immutable-подход: он делает изменение видимым на уровне ссылок.

### ExpressionChangedAfterItHasBeenCheckedError

Самая "загадочная" ошибка Angular (только в dev mode):

**Когда возникает:** вы изменяете данные **после** того, как Angular уже проверил binding:

```typescript
// ❌ Вызовет ошибку
ngAfterViewInit() {
  this.title = 'Changed!'; // Angular уже прочитал title для шаблона!
}
```

**Почему Angular это делает:** в dev mode Angular проверяет дважды (чтобы убедиться, что binding стабилен). Если между первой и второй проверкой значение изменилось — ошибка.

**Решения:**
1. `ChangeDetectorRef.detectChanges()` — принудительная перепроверка
2. `setTimeout(() => this.title = 'Changed!')` — перенос на следующий CD cycle
3. **Лучше всего**: переосмыслить архитектуру. Обычно ошибка означает, что данные текут "не туда"

Signals уменьшают вероятность этой ошибки благодаря реактивной модели, но **не устраняют её полностью** — запись в signal в поздних lifecycle hooks (например, `ngAfterViewInit`) может вызвать ту же ошибку.

📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: ChangeDetectionStrategy](https://angular.dev/api/core/ChangeDetectionStrategy)
- [Angular API: ChangeDetectorRef](https://angular.dev/api/core/ChangeDetectorRef)


📚 **Источники:**
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: ChangeDetectionStrategy](https://angular.dev/api/core/ChangeDetectionStrategy)
- [Angular API: ChangeDetectorRef](https://angular.dev/api/core/ChangeDetectorRef)


### 🎯 Interview Focus

Сильный ответ звучит так: **Change Detection — это механизм синхронизации состояния компонентов с DOM. Angular обычно запускает его после событий и async-операций, замеченных через Zone.js или другие реактивные механизмы. `Default` проверяет шире и чаще. `OnPush` проверяет точечнее: при новой ссылке `@Input`, событии внутри компонента, эмиссии через `async` pipe, `markForCheck()` и signal-изменениях. Лучшая Java-аналогия — Hibernate dirty checking: изменение состояния и синхронизация внешнего мира разделены по времени.**

### ✅ Проверь себя

**1. Какая Java-аналогия лучше всего помогает понять Change Detection?**
- A) Maven lifecycle
- B) Hibernate dirty checking
- C) JDBC connection pool
- D) `StringBuilder`

<details><summary>Ответ</summary>B. В обоих случаях инфраструктура синхронизирует внешний мир на основе уже изменённого состояния в памяти.</details>

**2. Почему `OnPush` любит immutable-обновления?**
- A) Потому что Angular не умеет читать поля объекта
- B) Потому что новый reference — явный сигнал об изменении входа
- C) Потому что immutable отключает DOM
- D) Потому что иначе TypeScript ломается

<details><summary>Ответ</summary>B. `OnPush` прекрасно работает там, где изменение видно на уровне ссылок.</details>

**3. Что обычно запускает Change Detection?**
- A) Любая строка кода в редакторе
- B) События, таймеры, Promise, HTTP и другие замеченные Angular триггеры
- C) Только вызов `detectChanges()`
- D) Только изменение CSS

<details><summary>Ответ</summary>B. Angular реагирует на события и async-активность, а не на абстрактное «изменение кода».</details>

---

## 5. 🎯 Observable vs Promise: история взросления UI-мышления

Если объяснять Observable и Promise только таблицей «одно значение vs много значений», это будет правда, но очень сухая и слишком поверхностная. Нам нужен путь мышления. Представь историю.

### История, с которой начинает почти каждый

Ты приходишь из мира **Модуля 6**, уже знаешь `Promise`, `async/await`, умеешь делать HTTP-запрос из **Модуля 9** и думаешь: «Ну отлично, этого ведь достаточно». И для первого шага это правда. Нужно загрузить список задач один раз? Promise звучит нормально. Ты запросил данные, получил результат, показал на экране, пошёл дальше.

Java-аналогия: Promise близок к упрощённому `CompletableFuture`. Он представляет одно будущее завершение. Но UI редко живёт одной точкой во времени. UI — это поток. Пользователь печатает, удаляет, возвращается, вводит снова, меняет фильтр, уходит со страницы, приходит новый HTTP-ответ. И вот тут начинается взросление.

📚 **Источники:**
- [RxJS: Overview](https://rxjs.dev/guide/overview)
- [RxJS: Operators](https://rxjs.dev/guide/operators)
- [RxJS: Subscription](https://rxjs.dev/guide/subscription)
- [Angular docs: RxJS interop with signals](https://angular.dev/guide/signals/rxjs-interop)


### Шаг 1. Тебе внезапно нужна отмена

Ты делаешь live-search по задачам. Пользователь вводит: `a`, потом `an`, потом `ang`, потом `angular`. Если на каждый ввод делать Promise-запрос, ты быстро видишь проблему: старый запрос всё ещё может вернуться позже нового. В результате экран может показать **устаревший ответ**.

Тут у разработчика случается маленькое фронтенд-прозрение: проблема не в том, что Promise «плохой». Проблема в том, что модель «один запрос, один ответ, и дальше всё закончено» не описывает реальную жизнь поисковой строки.

```typescript
search(term: string): Promise<Task[]> {
  return fetch(`/api/tasks?query=${encodeURIComponent(term)}`)
    .then(response => response.json());
}
```

📚 **Источники:**
- [RxJS: Overview](https://rxjs.dev/guide/overview)
- [RxJS: Operators](https://rxjs.dev/guide/operators)
- [RxJS: Subscription](https://rxjs.dev/guide/subscription)
- [Angular docs: RxJS interop with signals](https://angular.dev/guide/signals/rxjs-interop)


### Разбор по шагам

1. Метод `search` принимает одну строку `term`.
2. `fetch` отправляет один HTTP-запрос — здесь всё ещё живёт модель из **Модуля 9**.
3. Возвращается Promise: он когда-то завершится одним значением `Task[]` или ошибкой.
4. Пока что всё выглядит нормально.
5. Но если пользователь быстро вызовет `search('a')`, `search('an')`, `search('ang')`, у нас появятся три независимых Promise.
6. Promise сам по себе не даёт удобной композиции потока пользовательского ввода и не решает красиво задачу «старое больше не интересно».

### Шаг 2. Потом тебе нужны не одно, а многие значения

В UI полно источников, которые по природе потоковые:

- `input`-события из DOM;
- клики;
- изменения route params;
- websocket-сообщения;
- обновления формы;
- таймеры;
- комбинации всего этого.

Promise здесь начинает напоминать попытку налить реку в бутылку. Бутылка хорошая. Просто объект не тот. Observable нужен не потому, что «так модно в Angular», а потому что сам интерфейс живёт **во времени**, а не в одной точке.

📚 **Источники:**
- [RxJS: Overview](https://rxjs.dev/guide/overview)
- [RxJS: Operators](https://rxjs.dev/guide/operators)
- [RxJS: Subscription](https://rxjs.dev/guide/subscription)
- [Angular docs: RxJS interop with signals](https://angular.dev/guide/signals/rxjs-interop)


### Шаг 3. Потом тебе нужны операторы

Допустим, ты уже понял, что источников много. Но этого мало. Нужно ещё уметь говорить:

- «подожди, пока пользователь перестанет печатать 300 мс» — `debounceTime`;
- «не запускай новый запрос, пока есть более актуальный» — `switchMap`;
- «отфильтруй одинаковые подряд значения» — `distinctUntilChanged`;
- «склей поток route params и поток current user» — `combineLatest`;
- «повтори запрос при временной ошибке» — `retry`.

Вот тут RxJS раскрывается по-настоящему. Observable — это не просто «много значений». Это **композиция потока во времени**.

📚 **Источники:**
- [RxJS: Overview](https://rxjs.dev/guide/overview)
- [RxJS: Operators](https://rxjs.dev/guide/operators)
- [RxJS: Subscription](https://rxjs.dev/guide/subscription)
- [Angular docs: RxJS interop with signals](https://angular.dev/guide/signals/rxjs-interop)


### Observable как история про поток

Первая метафора: Observable — это не курьер с конвертом, а новостной канал. Подписался — и тебе могут приходить новые выпуски. Можно отключиться. Можно фильтровать. Можно комбинировать с другими каналами.

Вторая метафора: Observable — это конвейер на фабрике. На вход подаются события, дальше стоят фильтры, сортировщики, упаковщики, дубликатоуловители, отмена бракованных партий и объединение с соседней линией. Именно так ощущаются операторы RxJS.

Java-аналогия: Observable ближе к `Flux` из Project Reactor, чем к `CompletableFuture`. Если Promise — это скорее `Mono`/Future-подобное одно завершение, то Observable — это поток, который может жить долго и давать много значений.

```typescript
results$ = this.searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  filter((term): term is string => term != null && term.length >= 2),
  switchMap(term =>
    this.http.get<Task[]>(`/api/tasks?query=${encodeURIComponent(term)}`)
  )
);
```

📚 **Источники:**
- [RxJS: Overview](https://rxjs.dev/guide/overview)
- [RxJS: Operators](https://rxjs.dev/guide/operators)
- [RxJS: Subscription](https://rxjs.dev/guide/subscription)
- [Angular docs: RxJS interop with signals](https://angular.dev/guide/signals/rxjs-interop)


### Разбор по шагам

1. `valueChanges` — это поток значений из формы. Не одно значение, а серия изменений во времени.
2. `debounceTime(300)` говорит: «Не реагируй на каждую букву мгновенно. Подожди короткую паузу».
3. `distinctUntilChanged()` отсеивает одинаковые подряд запросы.
4. `filter(...)` отсеивает `null` и слишком короткие строки, чтобы не шуметь запросами на каждый пустой чих формы.
5. `switchMap(...)` делает самый важный трюк: если пришло новое значение, предыдущий внутренний запрос становится неактуальным, и поток переключается на новый.
6. `encodeURIComponent(...)` защищает query string от пробелов, `&`, `?` и других спецсимволов.
7. Это решает ту самую человеческую проблему live-search: экрану нужен **последний актуальный** результат, а не любой пришедший позже.
8. Здесь соединяются темы из **Модуля 8** (DOM input events), **Модуля 9** (HTTP) и **Модуля 6** (асинхронность), но теперь уже через реактивный язык RxJS.

### Так значит Promise не нужен?

Нужен. Promise остаётся отличным инструментом для одноразовых асинхронных операций. Но Angular исторически глубоко интегрирован с RxJS, потому что UI — это почти всегда поток изменений, а не один раз и навсегда. Хороший разработчик не ругает Promise. Он понимает **границы применимости**.

#### Отписка: как не создать memory leak

```typescript
// Способ 1: async pipe (рекомендуется для шаблонов)
// В template: {{ tasks$ | async }}

// Способ 2: takeUntilDestroyed (Angular 16+)
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

constructor() {
  this.dataService.getData()
    .pipe(takeUntilDestroyed())
    .subscribe(data => this.data = data);
}

// Способ 3: DestroyRef (более гибкий)
private destroyRef = inject(DestroyRef);
ngOnInit() {
  this.sub = this.data$.subscribe(...);
  this.destroyRef.onDestroy(() => this.sub.unsubscribe());
}
```

📚 **Источники:**
- [RxJS: Overview](https://rxjs.dev/guide/overview)
- [RxJS: Operators](https://rxjs.dev/guide/operators)
- [RxJS: Subscription](https://rxjs.dev/guide/subscription)
- [Angular docs: RxJS interop with signals](https://angular.dev/guide/signals/rxjs-interop)


📚 **Источники:**
- [RxJS: Overview](https://rxjs.dev/guide/overview)
- [RxJS: Operators](https://rxjs.dev/guide/operators)
- [RxJS: Subscription](https://rxjs.dev/guide/subscription)
- [Angular docs: RxJS interop with signals](https://angular.dev/guide/signals/rxjs-interop)


### 🎯 Interview Focus

Сильный ответ: **Promise представляет одно будущее значение и хорошо подходит для разовой асинхронной операции. Observable представляет поток значений во времени, умеет ленивость, отмену через отписку и богатую композицию операторов. Для UI это особенно важно, потому что пользовательский ввод, события, роутинг и HTTP часто образуют длительные и комбинируемые потоки. Поэтому Angular глубоко использует RxJS.**

### ✅ Проверь себя

**1. Почему Promise часто перестаёт хватать для live-search?**
- A) Потому что браузер не умеет Promise
- B) Потому что нужен поток ввода, отмена/переключение устаревших запросов и операторы времени
- C) Потому что Promise нельзя возвращать из функций
- D) Потому что Angular запрещает `async/await`

<details><summary>Ответ</summary>B. Проблема в потоковой природе UI, а не в «плохом» Promise.</details>

**2. Что делает `switchMap` в типичном поисковом сценарии?**
- A) Запускает все запросы и выводит любой поздний ответ
- B) Переключается на новый внутренний поток и делает старый неактуальным
- C) Блокирует ввод пользователя
- D) Удаляет DOM-элемент поиска

<details><summary>Ответ</summary>B. Для живого поиска это один из самых полезных операторов.</details>

---

## 6. Subjects: `BehaviorSubject` и `ReplaySubject` как мост между потоком и состоянием

Когда ты уже понял Observable, быстро появляется новый вопрос: «А что делать, если мне нужен не просто поток событий, а ещё и **текущее состояние**, доступное нескольким частям приложения?» Здесь в кадр выходят Subject'ы.

Первая метафора: обычный Observable похож на радиоэфир, который ты только слушаешь. Subject — это уже рация. В неё можно и слушать, и говорить. Удобно? Очень. Опасно? Тоже очень. Если дать всем сотрудникам офиса право кричать в одну рацию, через день никто не будет понимать, откуда прилетело состояние.

### `BehaviorSubject`: текущее состояние прямо сейчас

`BehaviorSubject` всегда хранит **последнее значение** и немедленно отдаёт его новому подписчику. Это как электронное табло на вокзале. Ты только вошёл в зал — и сразу видишь текущее время отправления, не дожидаясь следующего объявления.

Это очень удобно для auth-state, loading-state, списка задач, текущего фильтра. Именно потому, что новый компонент или новый подписчик не должен ждать следующего изменения, чтобы понять «а что сейчас происходит». Он получает текущую картину сразу.

📚 **Источники:**
- [RxJS: Subject](https://rxjs.dev/guide/subject)
- [RxJS API: BehaviorSubject](https://rxjs.dev/api/index/class/BehaviorSubject)
- [RxJS API: ReplaySubject](https://rxjs.dev/api/index/class/ReplaySubject)


### `ReplaySubject`: когда важна не только текущая точка, но и недавняя история

`ReplaySubject` идёт дальше и может воспроизвести несколько последних значений. Это уже не табло, а маленький видеорегистратор. Ты пришёл позже, но можешь быстро увидеть, что происходило незадолго до тебя.

Полезно? Да, если новому подписчику нужен контекст. Но за это платишь памятью и иногда сложностью reasoning. Поэтому `ReplaySubject` нельзя использовать бездумно, иначе получится ситуация: «Мы зачем-то храним вагон старых событий, которые никому не нужны».

```typescript
@Injectable({ providedIn: 'root' })
export class TaskStore {
  private tasksSubject = new BehaviorSubject<Task[]>([]);
  readonly tasks$ = this.tasksSubject.asObservable();

  setTasks(tasks: Task[]): void {
    this.tasksSubject.next(tasks);
  }

  toggleTask(id: number): void {
    const updated = this.tasksSubject.value.map(task =>
      task.id === id ? { ...task, done: !task.done } : task
    );

    this.tasksSubject.next(updated);
  }
}
```

📚 **Источники:**
- [RxJS: Subject](https://rxjs.dev/guide/subject)
- [RxJS API: BehaviorSubject](https://rxjs.dev/api/index/class/BehaviorSubject)
- [RxJS API: ReplaySubject](https://rxjs.dev/api/index/class/ReplaySubject)


### Разбор по шагам

1. `tasksSubject` — приватный `BehaviorSubject`, внутри которого хранится текущее состояние списка задач.
2. Начальное значение `[]` означает: даже до загрузки у нас есть корректное текущее состояние — пустой массив.
3. Снаружи публикуется только `tasks$ = this.tasksSubject.asObservable()`. Это важно для инкапсуляции.
4. Метод `setTasks` централизованно меняет состояние через `next`.
5. В `toggleTask` используется immutable-подход: через `map` и spread создаются новые объекты и новый массив.
6. Это сочетает идеи из **Модуля 3** (ссылки), **Модуля 5** (массивные трансформации) и раздела про `OnPush`.
7. Такой сервис очень похож на аккуратный service layer в Spring: внутрь можно писать, снаружи — только читать.

📚 **Источники:**
- [RxJS: Subject](https://rxjs.dev/guide/subject)
- [RxJS API: BehaviorSubject](https://rxjs.dev/api/index/class/BehaviorSubject)
- [RxJS API: ReplaySubject](https://rxjs.dev/api/index/class/ReplaySubject)


### 🎯 Interview Focus

Коротко: **`BehaviorSubject` хранит текущее значение и сразу отдаёт его новому подписчику. `ReplaySubject` может воспроизвести несколько прошлых значений. Оба являются и Observable, и источником эмиссии, поэтому лучше держать их приватно внутри сервиса, а наружу отдавать только `asObservable()`.**

### ✅ Проверь себя

**1. В чём главная особенность `BehaviorSubject`?**
- A) Он не хранит состояние
- B) Он всегда имеет текущее значение и сразу отдаёт его новому подписчику
- C) Он работает только с HTTP
- D) Он не поддерживает `next`

<details><summary>Ответ</summary>B. Именно это делает его удобным для state-сценариев.</details>

**2. Когда разумнее выбрать `ReplaySubject`?**
- A) Когда новому подписчику нужна часть недавней истории
- B) Когда нельзя хранить значения
- C) Когда нужен только CSS
- D) Когда сервисов в приложении нет

<details><summary>Ответ</summary>A. ReplaySubject полезен именно там, где важен контекст недавних событий.</details>

---

## 7. Signals в Angular 17+: будущее, в котором Angular знает точные зависимости

(Signals появились как developer preview в Angular 16, но стали stable в Angular 17.)

Если Zone.js и широкий Change Detection были способом сказать: «После значимой async-активности давай ещё раз пройдёмся и посмотрим, что изменилось», то Signals предлагают другой стиль мышления: **мы знаем, какая часть UI зависит от какого состояния, значит можем обновлять точнее**.

### Почему Angular вообще пошёл в Signals

Потому что эпоха «подслушай всю асинхронность, потом пройди по дереву и проверь» работает, но имеет потолок. Особенно когда приложение большое, а разработчики хотят более предсказуемую, локальную и тонко отслеживаемую реактивность.

Первая метафора: старый подход похож на администратора склада, который после каждого звонка в офис идёт обходить весь ряд полок и смотреть, не надо ли что-то переставить. Signals — это датчики на конкретных полках. Изменилась полка A — не нужно ради этого проверять полку Z.

Вторая метафора: если Zone.js — это шпион, который говорит «что-то где-то произошло», то Signal — это умная лампа, которая знает, какие выключатели с ней связаны. Изменился конкретный выключатель — лампа и только нужные связанные вещи реагируют напрямую.

Третья метафора: Excel. Формула в одной ячейке знает, от каких ячеек она зависит. Меняется одна исходная ячейка — пересчитываются только зависимые формулы. Это и есть **fine-grained reactivity**.

Java-аналогия тут менее буквальная, но полезная: представь смесь `AtomicReference`, observer pattern и автоматического графа зависимостей. Signal — это реактивная ячейка состояния. `computed` — вычисляемое значение на основе других сигналов. `effect` — побочный эффект, который запускается, когда использованные сигналы меняются.

📚 **Источники:**
- [Angular docs: Signals](https://angular.dev/guide/signals)
- [Angular docs: RxJS interop](https://angular.dev/guide/signals/rxjs-interop)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)


### Signals не заменяют всё подряд

Очень важный взрослый момент: Signals — не «убийца RxJS» и не кнопка «выключить мозг». Они особенно хороши для **локального состояния и точных зависимостей view-слоя**. Observable по-прежнему прекрасно подходит для потоков времени: HTTP, websockets, пользовательский ввод, внешние асинхронные события. На практике они часто работают вместе.

```typescript
import { Component, computed, effect, signal } from '@angular/core';

@Component({
  selector: 'app-task-stats',
  standalone: true,
  imports: [],
  template: `
    <p>Всего: {{ tasks().length }}</p>
    <p>Выполнено: {{ completedCount() }}</p>
  `
})
export class TaskStatsComponent {
  tasks = signal<Task[]>([]);

  completedCount = computed(() =>
    this.tasks().filter(task => task.done).length
  );

  constructor() {
    effect(() => {
      console.log('Completed count changed:', this.completedCount());
    });
  }

  loadTasks(tasks: Task[]): void {
    this.tasks.set(tasks);
  }
}
```

📚 **Источники:**
- [Angular docs: Signals](https://angular.dev/guide/signals)
- [Angular docs: RxJS interop](https://angular.dev/guide/signals/rxjs-interop)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)


### Разбор по шагам

1. `tasks = signal<Task[]>([])` создаёт реактивную ячейку состояния со стартовым значением пустого массива.
2. Чтение сигнала происходит как вызов функции: `tasks()`.
3. `completedCount` — это `computed`, то есть вычисляемое значение на основе `tasks`.
4. Когда `computed` впервые вычисляется, Angular запоминает, что он зависит от `tasks`.
5. `effect` читает `completedCount()`, значит тоже становится зависимым от него.
6. Когда `loadTasks` вызывает `this.tasks.set(tasks)`, меняется сигнал `tasks`.
7. Angular знает точную цепочку зависимостей: нужно пересчитать `completedCount` и заново выполнить `effect`.
8. Шаблон, который читает `tasks()` и `completedCount()`, тоже знает свои зависимости. Это и есть шаг к более точным обновлениям.

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

// Observable → Signal
tasks = toSignal(this.http.get<Task[]>('/api/tasks'), { initialValue: [] });

// Signal → Observable
tasks$ = toObservable(this.tasksSignal);
```

Сигналы в современном Angular заходят ещё глубже: теперь ими можно описывать не только локальное состояние, но и сам контракт компонента снаружи.

```typescript
// Signal-based inputs (Angular 17.1+)
@Component({
  selector: 'app-task-detail',
  standalone: true,
  imports: [],
  template: `
    <h2>{{ task().title }}</h2>
    <p>Status: {{ status() }}</p>
    <button (click)="complete()">Done</button>
  `
})
export class TaskDetailComponent {
  // Signal inputs — реактивные, типобезопасные
  task = input.required<Task>();          // обязательный
  priority = input<number>(0);            // с дефолтом
  
  // Function-based output (не signal!)
  completed = output<Task>();
  
  // Two-way binding с model()
  status = model<string>('pending');      // [(status)]="taskStatus"
  
  // Computed от input signal
  isUrgent = computed(() => this.priority() > 8);
  
  complete() {
    this.status.set('done');
    this.completed.emit(this.task());
  }
}
```

Это важный сдвиг мышления: signal inputs постепенно заменяют декоратор `@Input()` там, где тебе нужна более естественная реактивность. Они сразу дружат с `computed()` и `effect()`, без дополнительной обвязки и ручного мостика между «входом компонента» и локальным реактивным состоянием. `output()` при этом возвращает `OutputEmitterRef` — это emitter, а не signal. В отличие от `input()` и `model()`, он не создаёт реактивную привязку для чтения состояния.

### Почему говорят, что эпоха Zone.js заканчивается

Не в смысле «завтра всё удалят», а в смысле **смены центра тяжести**. Раньше главная идея была: «Наблюдай все async-границы и после этого проверяй UI». Теперь всё сильнее становится идея: «Точно отслеживай зависимости состояния и обновляй только нужное».

То есть Signals лечат несколько старых болей:

- уменьшают зависимость от широких проходов по дереву;
- делают локальное состояние более прямым и читаемым;
- лучше объясняют, почему именно обновился конкретный кусок UI;
- приближают Angular к более fine-grained reactive-модели.

📚 **Источники:**
- [Angular docs: Signals](https://angular.dev/guide/signals)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)


### Zoneless Angular — будущее Change Detection

Angular движется к **полному отказу от Zone.js**:

```typescript
// Включение zoneless mode (Angular 18+ experimental)
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```

**Что это значит на практике:**
- Нет патчинга `setTimeout`/`Promise`/`addEventListener`
- Change Detection запускается не «на каждый async-случай подряд», а по конкретным уведомлениям, которые Angular умеет видеть без Zone.js
- Меньше bundle size (`zone.js` ~13KB gzipped)
- Более предсказуемая производительность

**Что нужно знать о zoneless:**
- `AsyncPipe` **совместим** с zoneless — он сам вызывает `markForCheck()`
- `OnPush` **рекомендован**, но не строго обязателен — Angular уведомляет CD через несколько механизмов:
  - изменение signal, используемого в шаблоне
  - template/host event listeners
  - `AsyncPipe`
  - программный вызов `markForCheck()` / `detectChanges()`
  - `setInput()` на компоненте
- `toSignal()` — удобен, но не обязательная замена `| async`
- Signals делают модель CD чище, но не требуют мгновенно переписать всё приложение
- Главное преимущество: предсказуемость — CD запускается только по конкретным причинам, а не «на каждый setTimeout»

💡 Это НЕ означает, что Zone.js "сломан" — он работает. Но понимание Signals + OnPush — это инвестиция в будущее Angular.

📚 **Источники:**
- [Angular docs: Signals](https://angular.dev/guide/signals)
- [Angular docs: Zoneless](https://angular.dev/guide/experimental/zoneless)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)


📚 **Источники:**
- [Angular docs: Signals](https://angular.dev/guide/signals)
- [Angular docs: RxJS interop](https://angular.dev/guide/signals/rxjs-interop)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)


### 🎯 Interview Focus

Хороший ответ: **Signals — это точечные реактивные ячейки состояния и зависимостей. Они особенно полезны для локального state и view-слоя, потому что позволяют Angular отслеживать зависимости fine-grained-образом и обновлять UI точнее. Они не отменяют RxJS, а дополняют его. Именно поэтому их часто называют будущим Angular и признаком движения away from heavy zone-based change detection.**

### ✅ Проверь себя

**1. Что лучше всего описывает Signal?**
- A) Реактивную ячейку состояния
- B) Новый вид HTTP-протокола
- C) Замену DOM
- D) Аналог `setInterval`

<details><summary>Ответ</summary>A. Signal — это маленький реактивный контейнер состояния.</details>

**2. Какую проблему Signals решают особенно хорошо?**
- A) Fine-grained отслеживание зависимостей view и локального state
- B) Компиляцию Java в браузере
- C) Хранение паролей
- D) Замену всех network API

<details><summary>Ответ</summary>A. Их сила именно в точном знании зависимостей и адресных обновлениях.</details>

---

## 8. 🎯 Angular DI vs Spring DI: почти тот же ментальный контейнер, только в браузере

Если ты пришёл из Spring, то DI — это место, где Angular должен внезапно стать эмоционально близким. Не «отдалённо похожим», а почти родным. И это очень приятный момент: оказывается, фронтенд тоже может быть не хаотичной свалкой new-объектов, а системой с IoC-контейнером, constructor injection и управляемыми зависимостями.

### Сначала большая идея

**Dependency Injection** — это способ сказать: объект не должен собирать своё окружение вручную. Ему дают зависимости снаружи. Это снижает связанность, делает код тестируемее и переносит ответственность за сборку графа объектов в инфраструктуру.

Первая метафора: сотрудник не тащит домой отвертки, монитор и сетевой кабель, чтобы утром самому собрать рабочее место. Этим занимается офисная инфраструктура. Сотрудник просто приходит и работает. Так и сервис не должен сам создавать `new HttpClient()`, `new Logger()` и `new Config()` внутри себя.

📚 **Источники:**
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)
- [Angular API: inject()](https://angular.dev/api/core/inject)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Importing components](https://angular.dev/guide/components/importing)


### Подробная карта соответствий Spring ↔ Angular

| Spring / Java | Angular | Что это значит |
|---|---|---|
| `ApplicationContext` | `Injector` | Контейнер, который знает, как выдавать зависимости |
| `@Service`, `@Component` | `@Injectable`, `@Component` | Маркировка роли класса для инфраструктуры |
| Constructor Injection | Constructor Injection / `inject()` | Предпочтительный способ получить зависимости |
| Singleton bean | `providedIn: 'root'` | Один экземпляр на всё приложение |
| Scope | Hierarchical injector scope | Область жизни может зависеть от дерева компонентов |
| `@Qualifier`, named beans | `InjectionToken`, multi providers | Явный ключ зависимости, когда класса недостаточно |
| `@Configuration` + `@Bean` | `providers`, `useClass`, `useFactory`, `useValue` | Конфигурация способа создания зависимости |
| Test doubles / mocks | TestBed providers override | Подмена зависимостей в тестах |

Самое главное здесь — не механически запомнить таблицу, а почувствовать ментальную близость. Ты уже умеешь мыслить контейнером. Просто контейнер переехал из серверного мира в браузер.

📚 **Источники:**
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)
- [Angular API: inject()](https://angular.dev/api/core/inject)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Importing components](https://angular.dev/guide/components/importing)


### Что особенно отличается

Но нельзя делать вид, что различий нет. Они есть, и они очень важны.

Главное отличие — **иерархичность injector'ов, связанная с деревом компонентов**. В Spring ты часто думаешь категориями singleton / prototype / request / session. В Angular ты дополнительно думаешь категориями: «всё приложение», «эта feature», «эта ветка экрана», «этот конкретный компонент и его потомки».

Это очень браузерное мышление. Потому что UI — дерево. И зависимости могут жить вместе с веткой этого дерева. Это не хуже и не лучше серверных scope'ов. Это просто другой ландшафт.

📚 **Источники:**
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)
- [Angular API: inject()](https://angular.dev/api/core/inject)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Importing components](https://angular.dev/guide/components/importing)


### Конкретный Angular-пример

```typescript
import { InjectionToken, Injectable, inject } from '@angular/core';

export const API_URL = new InjectionToken<string>('API_URL');

@Injectable({ providedIn: 'root' })
export class ApiService {
  private apiUrl = inject(API_URL);

  getTasksUrl(): string {
    return `${this.apiUrl}/tasks`;
  }
}
```

📚 **Источники:**
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)
- [Angular API: inject()](https://angular.dev/api/core/inject)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Importing components](https://angular.dev/guide/components/importing)


### Разбор по шагам

1. `API_URL` — это `InjectionToken`, специальный ключ зависимости.
2. Он нужен, когда зависимость не выражается удобным runtime-классом или когда мы хотим внедрять конфигурационное значение.
3. `ApiService` объявлен как `providedIn: 'root'`, значит его можно считать application-wide singleton'ом.
4. Внутри сервиса вызывается `inject(API_URL)`, чтобы получить значение токена из контейнера.
5. ⚠️ `inject()` работает только в **injection context**: конструктор, field initializer, или factory-функция. Вызов в обычном методе (`ngOnInit`, event handler) даст `NG0203: inject() must be called from an injection context`.
6. Метод `getTasksUrl` использует это значение для сборки URL.
7. По смыслу это очень похоже на Spring-конфигурацию со значением свойства или именованным бином.

А теперь конфигурация provider'а:

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    { provide: API_URL, useValue: 'https://api.example.com' }
  ]
});
```

### Разбор по шагам

1. При старте приложения Angular получает список provider'ов.
2. `{ provide: API_URL, useValue: 'https://api.example.com' }` говорит контейнеру: если кто-то просит `API_URL`, отдай эту строку.
3. Это аналог конфигурационного биндинга значения в Spring.
4. В результате `ApiService` не жёстко зашивает URL в коде, а получает его через DI.
5. Такой код легче тестировать, менять между средами и переиспользовать.

### Та же идея на языке Spring

```java
// Spring — через @Value
@Value("${api.url}") private String apiUrl;
// или через @ConfigurationProperties
@ConfigurationProperties(prefix = "api")
public class ApiConfig { private String url; ... }
```

📚 **Источники:**
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)
- [Angular API: inject()](https://angular.dev/api/core/inject)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Importing components](https://angular.dev/guide/components/importing)


### Разбор по шагам

1. В Spring конфигурационные значения обычно приходят через `@Value` или `@ConfigurationProperties`.
2. Это ближе к роли Angular `InjectionToken` + `useValue`, чем строковый `@Bean`.
3. По ментальной модели это очень близко к Angular provider + token.
4. Среда другая, синтаксис другой, но идея одна: **объекты получают зависимости извне через контейнер**.

📚 **Источники:**
- [Angular docs: Dependency Injection](https://angular.dev/guide/di)
- [Angular API: inject()](https://angular.dev/api/core/inject)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Importing components](https://angular.dev/guide/components/importing)


### 🎯 Interview Focus

Сильный ответ: **Angular DI концептуально почти идентичен Spring DI: есть контейнер (`Injector`), который знает, как создавать и выдавать зависимости, а объекты получают их через constructor injection или `inject()`. Главное отличие — Angular часто использует иерархические injector'ы, связанные с деревом компонентов. `InjectionToken`, `providers`, `useValue`, `useFactory`, `useClass` — ключевые инструменты конфигурации.**

#### Standalone-first: современный Angular

С Angular 17 standalone-компоненты стали default. NgModule больше не нужен для нового кода:

```typescript
@Component({
  standalone: true,
  selector: 'app-task-list',
  imports: [CommonModule, TaskItemComponent],
  template: `...`
})
export class TaskListComponent { }
```

Регистрация сервисов — через `providedIn: 'root'` или `providers` в bootstrapApplication.

### ✅ Проверь себя

**1. Что общего у Angular DI и Spring DI?**
- A) Оба требуют JVM
- B) Оба реализуют IoC и внедрение зависимостей через контейнер
- C) Оба работают только через XML
- D) Оба запрещают конструкторы

<details><summary>Ответ</summary>B. Идея контейнера и ослабления связанности у них общая.</details>

**2. Что заметно отличает Angular DI от Spring DI?**
- A) Angular часто привязывает область жизни зависимости к дереву компонентов
- B) Angular не умеет фабрики
- C) Spring не знает singleton
- D) Angular не умеет конфигурационные значения

<details><summary>Ответ</summary>A. Иерархические injector'ы — одна из самых важных особенностей Angular.</details>

---

## 9. Lifecycle Hooks и их связь с event loop, шаблоном и фазами жизни компонента

Когда разработчик только знакомится с hooks, возникает опасная привычка: выучить список методов как мантру и подставлять их почти случайно. Это не понимание. Hook — это не «магический способ написать код внутри Angular». Hook — это **правильный момент жизни компонента**.

- `ngOnInit` близок к `@PostConstruct`;
- `ngOnDestroy` близок к `@PreDestroy`;
- `ngOnChanges` — это реакция на новые входные значения;
- `ngAfterViewInit` — момент, когда view уже собрана.

И снова связь с **Модулем 6**: hooks не бегают в отдельном потоке и не живут вне общего исполнения JavaScript. Они вызываются в рамках работы Angular, который сам живёт внутри обычного браузерного event loop.

Порядок на первом цикле: `constructor` → `ngOnChanges` *(только если изменился `@Input`)* → `ngOnInit` *(только первый раз)* → `ngDoCheck` → `ngAfterContentInit` *(только первый раз)* → `ngAfterContentChecked` → `ngAfterViewInit` *(только первый раз)* → `ngAfterViewChecked`. На последующих циклах: `ngOnChanges` *(только при изменении inputs)* → `ngDoCheck` → `ngAfterContentChecked` → `ngAfterViewChecked`.

```typescript
@Component({
  selector: 'app-task-detail',
  standalone: true,
  imports: [],
  template: `<p>{{ taskId }}</p>`
})
export class TaskDetailComponent implements OnInit, OnChanges, OnDestroy {
  @Input() taskId!: number;
  private timerId?: number;

  constructor(private ngZone: NgZone) {}

  ngOnChanges(changes: SimpleChanges): void {
    console.log('Input changed:', this.taskId);
    if (changes['taskId'] && !changes['taskId'].firstChange) {
      this.loadTask();
    }
  }

  ngOnInit(): void {
    this.loadTask();

    this.ngZone.runOutsideAngular(() => {
      this.timerId = window.setInterval(() => {
        console.log('Polling task', this.taskId);

        this.ngZone.run(() => {
          this.updateFromPoll();
        });
      }, 5000);
    });
  }

  ngOnDestroy(): void {
    if (this.timerId) {
      window.clearInterval(this.timerId);
    }
  }

  private loadTask(): void {
    console.log('Loading task', this.taskId);
  }

  private updateFromPoll(): void {
    this.loadTask();
  }
}
```

### Разбор по шагам

1. `@Input() taskId` означает, что значение приходит снаружи, из родителя.
2. `ngOnChanges()` срабатывает, когда Angular видит изменение входного значения.
3. `ngOnInit()` вызывается один раз после начальной инициализации компонента.
4. `loadTask()` — это уже реальный метод класса, а не абстрактный `reload()`, которого нигде нет.
5. ⚠️ `setInterval` внутри Angular zone будил бы Change Detection каждые 5 секунд, даже если UI почти не меняется.
6. Поэтому таймер вынесен в `ngZone.runOutsideAngular(...)`, а внутрь Angular мы возвращаемся только в момент, когда реально обновляем состояние.
7. `ngOnDestroy()` нужен для cleanup: мы очищаем таймер, чтобы не оставить утечку и лишнюю активность.
8. Это очень похоже на серверную дисциплину управления ресурсами: открыл — закрой, подписался — отпишись, создал таймер — почисти.

📚 **Источники:**
- [Angular docs: Component Lifecycle](https://angular.dev/guide/components/lifecycle)
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)


### 🎯 Interview Focus

Коротко: **Lifecycle hooks — это точки жизненного цикла компонента. `ngOnInit` — начальная инициализация, `ngOnChanges` — реакция на новые `@Input`, `ngAfterViewInit` — момент после сборки view, `ngOnDestroy` — cleanup. Hooks вызываются внутри обычной работы Angular и JavaScript, а не в отдельном параллельном механизме.**

### ✅ Проверь себя

**1. Какой hook ближе всего к `@PreDestroy`?**
- A) `ngOnInit`
- B) `ngAfterViewInit`
- C) `ngOnDestroy`
- D) `trackBy`

<details><summary>Ответ</summary>C. Именно здесь обычно освобождают ресурсы и подписки.</details>

**2. Почему hooks нельзя считать отдельным параллельным механизмом?**
- A) Потому что они вызываются в рамках обычного исполнения Angular и JavaScript
- B) Потому что hooks есть только в Spring
- C) Потому что hooks работают до загрузки браузера
- D) Потому что hooks заменяют event loop

<details><summary>Ответ</summary>A. Hook — это правильный момент жизни компонента внутри общего исполнения.</details>

---

## 10. Производительность: `trackBy`, `OnPush`, `detach`, `runOutsideAngular`

Когда люди говорят «оптимизация Angular», часто звучит что-то абстрактное и пугающее. Но большая часть реальной оптимизации очень земная: **делать меньше лишней работы**. Не заставлять Angular пересоздавать DOM там, где можно использовать существующие узлы. Не будить Change Detection на каждый шорох. Не проверять тяжёлый кусок UI без необходимости.

### `trackBy`: не терять identity элементов списка

Первая метафора: гардероб. Если у каждого пальто есть номерок, гардеробщик не будет каждый раз перебирать весь зал и гадать, чьё это пальто. `trackBy` даёт Angular такой номерок для списка.

```typescript
@Component({
  standalone: true,
  selector: 'app-task-list',
  imports: [AppSpinnerComponent, HeavyChartComponent],
  template: `
    @if (isLoading()) {
      <app-spinner />
    } @else if (error()) {
      <p class="error">{{ error() }}</p>
    } @else {
      <ul>
        @for (task of tasks(); track task.id) {
          <li>{{ task.title }}</li>
        } @empty {
          <li>Нет задач</li>
        }
      </ul>
    }

    @defer (on viewport) {
      <app-heavy-chart [data]="chartData()" />
    } @placeholder {
      <p>График загрузится при прокрутке...</p>
    }
  `
})
export class TaskListComponent {
  tasks = signal<Task[]>([]);
  chartData = signal<Task[]>([]);
  isLoading = signal(false);
  error = signal<string | null>(null);
}
```

📚 **Источники:**
- [Angular docs: Templates control flow](https://angular.dev/guide/templates/control-flow)
- [Angular API: TrackByFunction](https://angular.dev/api/core/TrackByFunction)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)


### Разбор по шагам

1. `@if / @else` — это современный control flow Angular 17+, который читается ближе к обычной логике языка.
2. `@for (task of tasks(); track task.id)` — современный способ итерироваться по списку и сразу указать identity элемента.
3. `track task.id` — это та же идея, что старый `trackBy`, только компактнее и встроено прямо в синтаксис цикла.
4. `@empty` помогает элегантно показать пустой список без дополнительных `*ngIf`.
5. `@defer` откладывает тяжёлый кусок UI до нужного момента, например до появления во viewport.
6. Это особенно важно для больших списков и частых обновлений.

🏚️ **Legacy (structural directives, до Angular 17):**

```typescript
@Component({
  standalone: true,
  selector: 'app-task-list',
  imports: [CommonModule],
  template: `
    <div *ngIf="isLoading; else content">Loading...</div>
    <ng-template #content>
      <li *ngFor="let task of tasks; trackBy: trackByTaskId">
        {{ task.title }}
      </li>
    </ng-template>
  `
})
export class LegacyTaskListComponent {
  @Input() tasks: Task[] = [];
  isLoading = false;

  trackByTaskId(index: number, task: Task): number {
    return task.id;
  }
}
```

### `runOutsideAngular`: не будить охранника без причины

Если у тебя есть частые scroll-события, drag-and-drop, тяжёлая анимация или внешняя библиотека, которая стреляет событиями десятки раз в секунду, Angular-zone может слишком часто запускать проверки. Тогда полезно вынести шумную работу наружу.

```typescript
constructor(private ngZone: NgZone) {}

private scrollHandler = () => {
  console.log('scrolling...');
};

ngOnInit() {
  this.ngZone.runOutsideAngular(() => {
    window.addEventListener('scroll', this.scrollHandler);
  });
}

ngOnDestroy() {
  window.removeEventListener('scroll', this.scrollHandler);
}
```

📚 **Источники:**
- [Angular API: NgZone](https://angular.dev/api/core/NgZone)
- [Angular best practices: Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)


### Разбор по шагам

1. Через DI мы получаем `NgZone`.
2. `runOutsideAngular` выполняет переданный код вне Angular zone.
3. Обработчик `scroll` регистрируется так, что его частые срабатывания не будут каждый раз автоматически будить Angular Change Detection.
4. Это полезно, когда событие очень частое, а UI не нужно обновлять на каждый тик.
5. Если позже всё же надо обновить состояние экрана, можно вернуться внутрь Angular через `ngZone.run(...)`.

### `detach`: ручная коробка передач

Иногда есть тяжёлый компонент, который обновляется редко и предсказуемо. Тогда можно отсоединить его от автоматической проверки и включать её вручную.

```typescript
constructor(private cdr: ChangeDetectorRef) {}

ngOnInit(): void {
  this.cdr.detach();
}

refreshView(): void {
  this.cdr.detectChanges();
}
```

📚 **Источники:**
- [Angular API: ChangeDetectorRef](https://angular.dev/api/core/ChangeDetectorRef)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular API: ChangeDetectionStrategy](https://angular.dev/api/core/ChangeDetectionStrategy)


### Разбор по шагам

1. Через `ChangeDetectorRef` мы получаем контроль над проверкой текущего компонента.
2. `detach()` выводит компонент из обычной автоматической цепочки Change Detection.
3. Теперь Angular не будет проверять его в стандартном режиме так же, как раньше.
4. Когда мы сами считаем нужным обновить view, вызываем `detectChanges()`.
5. Это мощный инструмент, но он требует дисциплины. Если забыть вручную синхронизировать view, интерфейс будет «застывшим».

📚 **Источники:**
- [Angular docs: Templates control flow](https://angular.dev/guide/templates/control-flow)
- [Angular API: TrackByFunction](https://angular.dev/api/core/TrackByFunction)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)


### 🎯 Interview Focus

Хороший ответ: **`trackBy` уменьшает лишнее пересоздание DOM в списках, `OnPush` уменьшает число проверок за счёт явных триггеров, `runOutsideAngular` не даёт шумной активности будить Change Detection без причины, `detach` отключает компонент от автоматической проверки до ручного контроля. Это четыре разных инструмента оптимизации, а не один и тот же приём.**

### ✅ Проверь себя

**1. Зачем нужен `trackBy`?**
- A) Чтобы Angular стабильно сопоставлял элементы списка и DOM-узлы
- B) Чтобы TypeScript вывел тип массива
- C) Чтобы отключить DI
- D) Чтобы заменить CSS

<details><summary>Ответ</summary>A. `trackBy` помогает сохранить identity элементов списка и уменьшить DOM churn.</details>

**2. Когда полезен `runOutsideAngular`?**
- A) Когда есть частая шумная активность, которая не должна запускать Change Detection на каждом шаге
- B) Когда нужно создать новый сервис
- C) Когда хочется заменить Promise
- D) Когда надо отключить TypeScript

<details><summary>Ответ</summary>A. Это способ уменьшить лишние срабатывания проверки на шумных сценариях.</details>

**3. Что делает `detach`?**
- A) Удаляет компонент навсегда
- B) Выводит компонент из автоматического Change Detection до ручного управления
- C) Превращает компонент в сервис
- D) Создаёт новый поток

<details><summary>Ответ</summary>B. Это ручной режим контроля обновлений view.</details>

---

## 11. Как всё соединяется: путь от клика пользователя до обновления DOM

Теперь сделаем то, ради чего вообще существовал весь курс: **соберём одну непрерывную цепочку**. Представь, что пользователь кликает кнопку «Завершить задачу» в Angular-приложении.

### Шаг 1. Событие из браузера

Из **Модуля 8** мы знаем: браузер генерирует DOM-событие. Никакого «angular click» как физического явления в природе нет. Есть обычный browser event.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 2. JavaScript-функция-обработчик

Из **Модуля 2** мы знаем: вызывается функция, и у неё есть контекст, замыкания и доступ к состоянию компонента. Там же живут стрелочные функции, `this` и методы.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 3. Изменение объекта или массива

Из **Модуля 3** и **Модуля 5** мы знаем: состояние в компоненте — это объекты, массивы и ссылки. Если это `OnPush`, вопрос новой ссылки становится критическим.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 4. Если действие асинхронное — включается история из event loop

Из **Модуля 6** знаем: таймеры, Promise, HTTP-ответы и другие async-операции идут через event loop, task queue и microtask queue.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 5. Если есть HTTP — подключается сетевой слой

Из **Модуля 9** знаем: запрос уходит через браузерный сетевой API, ответ приходит позже, а не в ту же секунду.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 6. Zone.js замечает завершение async-работы

Это уже тема текущего модуля: если операция прошла внутри Angular zone, инфраструктура получает сигнал, что произошло нечто значимое для возможного обновления UI.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 7. Change Detection или Signals решают, что нужно обновить

Если приложение живёт в классической модели, Angular запускает Change Detection. Если активно используется signal-модель, Angular может точнее отследить зависимости состояния. Здесь встречаются миры старой и новой реактивности.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 8. Шаблон пересчитывается и синхронизируется с DOM

Шаблон — это не магический HTML, а декларация того, как значения состояния превращаются в DOM. В этот момент опыт из **Модуля 7** про сборку и шаблонный компайлинг тоже имеет значение: Angular заранее готовит инфраструктуру, чтобы такие обновления были системными, а не хаотичными.

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


### Шаг 9. Пользователь видит обновление

📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


📚 **Источники:**
- [Angular docs: Components](https://angular.dev/guide/components)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular docs: Signals](https://angular.dev/guide/signals)


---

## 12. Итоги курса: все 10 модулей как одна карта мышления

Давай проговорим это почти по-менторски, как старший коллега в конце хорошего обучения.

В **Модуле 1** ты узнал, что JavaScript не стыдится динамичности, а значит нельзя относиться к нему как к «бедной Java». Там были типы, приведение, `var/let/const`, `===` и фундаментальная осторожность. Это было нужно, чтобы дальше не наступать на минные поля языка.

В **Модуле 2** ты увидел, что функция в JavaScript — это не просто кусок кода, а код плюс окружение. Замыкания, `this`, стрелочные функции — всё это потом всплывёт в обработчиках событий, RxJS-операторах, сервисах и компонентах Angular.

В **Модуле 3** ты понял, что объекты в JS живут через ссылки и прототипы. Без этого нельзя объяснить ни `OnPush`, ни мутацию vs immutable-update, ни то, почему состояние иногда «как будто не обновилось».

В **Модуле 6** ты разобрал event loop, Promise, `async/await`, microtasks и macrotasks. Без этого Zone.js и Change Detection кажутся мистикой. С этим — становятся просто инфраструктурой поверх знакомой async-механики.

В **Модуле 7** ты понял, как код организуется по модулям и как сборка доставляет его в браузер. Angular CLI, lazy loading, tree-shaking и структура приложения опираются именно на это мышление.

В **Модуле 8** ты спустился в браузерное окружение: `window`, `document`, DOM, события, bubbling, делегирование, `debounce`, `throttle`. Angular не отменяет браузер. Он в нём живёт. И тот, кто знает браузер, понимает Angular намного глубже.

В **Модуле 9** ты увидел сетевой слой и хранение данных: `fetch`, CORS, storage, cookies, IndexedDB, WebSocket. Это важно, потому что Angular-приложение почти всегда связано с сетью и состоянием данных, а значит не существует отдельно от этих реалий.

И вот **Модуль 10** наконец собрал всё вместе. TypeScript дал контракты. Декораторы разметили роли. DI связал объекты. Zone.js объяснил, как Angular подслушивает асинхронность. Change Detection объяснил, как интерфейс синхронизируется с состоянием. RxJS показал, как жить в мире потоков событий. Signals показали, куда Angular движется дальше.

Если собрать всё в одну фразу, получится так:

**Пользователь действует в браузере; JavaScript-функции обрабатывают событие; состояние живёт в объектах и массивах; асинхронность координируется event loop; Angular через DI, декораторы, Zone.js, Change Detection, RxJS и Signals превращает это в системное приложение; а DOM обновляется как следствие понятной цепочки, а не магии.**

И это, честно говоря, очень красивый момент. Потому что теперь Angular уже не выглядит как мешок отдельных терминов. Он становится инженерной системой, которую можно объяснить, а значит — контролировать.

### 🎯 Interview Focus

Самый сильный кандидат в конце курса умеет не только отвечать на вопросы по кускам, но и собирать их в историю причин и следствий. Если ты можешь связно рассказать путь от клика пользователя до обновления DOM и при этом объяснить роль Zone.js, Change Detection, RxJS, Signals и DI, твоя база уже очень крепкая.

### ✅ Проверь себя

**1. Что связывает JavaScript и Angular сильнее всего?**
- A) Angular полностью заменяет механику языка
- B) Angular опирается на механику JavaScript и браузера и организует её
- C) Angular не использует асинхронность
- D) Angular работает только через jQuery

<details><summary>Ответ</summary>B. Angular — это инженерный слой поверх уже существующей платформы.</details>

**2. Какая мысль важнее всего после всего курса?**
- A) Фреймворк важнее языка
- B) Магия Angular необъяснима
- C) Фреймворк понятен через базовую механику JavaScript и браузера
- D) Достаточно помнить только команды CLI

<details><summary>Ответ</summary>C. Без понимания платформы фреймворк кажется магией, с пониманием — становится системой.</details>

---

## 📝 Квиз по модулю

**1. Почему TypeScript называют надстройкой над JavaScript?**
- A) Потому что он компилируется в JavaScript и усиливает этап разработки контрактами и проверками
- B) Потому что он заменяет браузер
- C) Потому что он запрещает JavaScript
- D) Потому что он работает только в Angular

<details><summary>Ответ</summary>A. Конечное исполнение всё равно происходит в JavaScript, но разработка становится безопаснее.</details>

**2. Что делает `@Component`?**
- A) Включает второй поток
- B) Даёт Angular метаданные о компоненте и его конфигурации
- C) Заменяет HTML на SQL
- D) Отключает DI

<details><summary>Ответ</summary>B. Это ярлык роли класса для фреймворка.</details>

**3. За что отвечает Zone.js?**
- A) За наблюдение за async-активностью и уведомление Angular о её завершении
- B) За хранение CSS
- C) За построение БД
- D) За generics TypeScript

<details><summary>Ответ</summary>A. Zone.js подслушивает асинхронные границы, а не обновляет DOM сам.</details>

**4. Почему `OnPush` выигрывает на больших приложениях?**
- A) Потому что делает проверки более точечными и любит явные сигналы
- B) Потому что отключает браузер
- C) Потому что запрещает события
- D) Потому что не использует DOM

<details><summary>Ответ</summary>A. Меньше лишних проверок — меньше лишней работы.</details>

**5. Когда Observable логичнее Promise?**
- A) Когда перед нами поток событий, отмена, время и композиция операторов
- B) Когда нужен один синхронный ответ
- C) Когда нельзя писать TypeScript
- D) Когда нужен только CSS-grid

<details><summary>Ответ</summary>A. Именно там раскрывается реактивная природа Observable.</details>

**6. Чем `BehaviorSubject` отличается от `ReplaySubject`?**
- A) `BehaviorSubject` хранит текущее значение, `ReplaySubject` может воспроизводить часть истории
- B) Ничем
- C) `ReplaySubject` не поддерживает подписки
- D) `BehaviorSubject` не хранит состояние

<details><summary>Ответ</summary>A. Один про «сейчас», другой ещё и про «недавнее до этого».</details>

**7. Для чего Angular Signals?**
- A) Для точного отслеживания state-зависимостей и fine-grained реактивности
- B) Для генерации SQL
- C) Для удаления всех Observable из Angular
- D) Для компиляции CSS

<details><summary>Ответ</summary>A. Signals усиливают точность реактивной модели view-слоя.</details>

**8. Что общего у Angular DI и Spring DI?**
- A) Контейнер создаёт объекты и внедряет зависимости
- B) Оба требуют Tomcat
- C) Оба работают только через XML
- D) Оба создают HTTP-сессию на каждый компонент

<details><summary>Ответ</summary>A. Суть одна: IoC и ослабление связанности через контейнер.</details>

**9. Что особенно важно помнить про `OnPush` и объекты?**
- A) Мутация старого объекта и создание нового объекта — одно и то же
- B) Для `OnPush` новая ссылка часто важнее, чем тихая мутация старой
- C) `OnPush` запрещает массивы
- D) `OnPush` работает только с числами

<details><summary>Ответ</summary>B. Здесь напрямую всплывают знания про ссылки из Модуля 3.</details>

**10. Почему Signals называют будущим Angular?**
- A) Потому что они помогают уменьшить зависимость от грубого zone-based подхода и дают более точные обновления
- B) Потому что они создают новые браузеры
- C) Потому что они заменяют HTTP
- D) Потому что они делают TypeScript ненужным

<details><summary>Ответ</summary>A. Это движение в сторону fine-grained реактивности и более адресного обновления UI.</details>

---

## 🏋️ Мини-упражнения

**Упражнение 1.** Представь `task-card` с `OnPush`. Родитель мутирует `task.done = true`. Опиши вслух, почему изменение может быть незаметным для ребёнка, и затем объясни, почему `task = { ...task, done: true }` меняет ситуацию. Если можешь рассказать это без шпаргалки, значит ты действительно связал **Модуль 3** про ссылки, **Модуль 5** про spread и текущий раздел про Change Detection.

**Упражнение 2.** Вообрази строку поиска задач. Пользователь быстро вводит `a`, `an`, `ang`, `angular`. Сначала словами объясни плохой вариант с Promise без отмены, а потом хороший вариант с `debounceTime` и `switchMap`. Цель — не просто вспомнить RxJS-операторы, а рассказать историю, почему UI живёт как поток из **Модуля 8**, асинхронность идёт через механику **Модуля 6**, а HTTP приходит из **Модуля 9**.

**Упражнение 3.** Нарисуй Angular DI как офис. Root injector будет центральным складом, component providers — локальными кладовками этажей, сервисы — оборудованием. Затем рядом нарисуй Spring `ApplicationContext` и сравни, где scope больше похож на «всё приложение», а где — на «эта ветка экрана». Если можешь объяснить это без путаницы, ты уже хорошо видишь общую IoC-картину.

**Упражнение 4.** Возьми простой пример с `setTimeout` и проговори по памяти весь путь: кто регистрирует callback, кто ждёт таймер, когда callback попадает в очередь, что monkey-patch делает с ним, как Zone.js восстанавливает контекст, и почему после этого Angular может запустить Change Detection. Это упражнение специально нарабатывает уверенность в самой страшной interview-теме модуля.

**Упражнение 5.** Составь устный рассказ «Все 10 модулей за 2 минуты». Начни с типов и функций, дойди до объектов, event loop, DOM, сети и завершай Angular. Это очень полезное упражнение на интеграцию знаний. В реальном интервью именно такие связные рассказы отличают человека, который «учил темы», от человека, который действительно собрал систему.

---

## 🚀 Задание по проекту (FINAL!): mini task-manager на Angular

Теперь собери финальную учебную версию task-manager как небольшое Angular-приложение. Представь, что у тебя есть `AppComponent`, маршруты `tasks`, `tasks/:id`, `settings`, сервис `TaskService`, контейнерный экран `task-page`, список `task-list`, карточка `task-item`, форма `task-form` и слой API. Цель не просто показать список задач, а **доказать самому себе**, что все 10 модулей сложились в архитектуру.

### Каким должен быть state

`TaskService` должен стать сердцем данных. Если хочешь RxJS-модель, держи внутри `private tasksSubject = new BehaviorSubject<Task[]>([])`, наружу отдавай `tasks$`, а обновления делай через новые массивы. Если хочешь современный Angular-подход, заведи `tasks = signal<Task[]>([])`, `loading = signal(false)`, `error = signal<string | null>(null)` и `computed` для `completedCount` или `filteredTasks`. Идеален даже гибрид: сеть и события — через Observable, локальный state экрана — через Signals.

### Как здесь проявляются все модули

- **Модуль 1**: корректные типы значений, аккуратность с `null/undefined`, отсутствие сюрпризов coercion.
- **Модуль 2**: обработчики событий, стрелочные функции, замыкания в RxJS-операторах.
- **Модуль 3**: понимание ссылок, immutable-обновления и влияние мутаций на `OnPush`.
- **Модуль 4**: классы сервисов и компонентов как синтаксический сахар над объектной моделью JS.
- **Модуль 5**: `map/filter/reduce`, spread, структура массивов задач, вычисления статистики.
- **Модуль 6**: Promise, event loop, таймеры, асинхронный порядок выполнения.
- **Модуль 7**: структура фич, lazy loading, организация проекта.
- **Модуль 8**: DOM-события, формы, ввод пользователя, `debounce`.
- **Модуль 9**: HTTP-запросы, ошибки сети, хранилище токена или фильтра.
- **Модуль 10**: DI, Change Detection, Zone.js, RxJS, Signals, оптимизации.

### Конкретные технические ожидания

1. **Список задач** сделай через `OnPush` и `trackBy` по `task.id`.
2. **Фильтр задач** реализуй через реактивный поток ввода или signal-состояние.
3. **API-слой** инжектируй через DI, а базовый URL передай через `InjectionToken`.
4. **Loader и error banner** покажи явно: пользователь должен видеть не только happy path.
5. **Обновление статуса задачи** делай immutable-образом.
6. **Удаление и создание задачи** должны обновлять список без полного хаоса в рендере.
7. **`ngOnDestroy`** используй там, где нужен cleanup, если у тебя остаются ручные подписки или таймеры.
8. **Если есть очень шумная UI-активность**, попробуй вынести её в `runOutsideAngular` и возвращаться внутрь только когда реально меняется видимое состояние.

### Минимальный пользовательский сценарий

Пользователь открывает страницу, видит loader, получает список задач, фильтрует его, добавляет новую задачу, переключает статус, удаляет задачу, видит сообщение об ошибке при сбое API и не теряет отзывчивость даже на длинном списке. Если у тебя это получилось, значит курс действительно соединился в цельную картину.

## Контрольные вопросы с подробными ответами

### 1. Почему Angular нельзя понять глубоко без JavaScript?

Потому что Angular живёт не в вакууме, а внутри браузерного JavaScript runtime. События, промисы, DOM, ссылки на объекты, очереди задач и асинхронность никуда не исчезают. Фреймворк только организует их. Если не понимать базовую механику из **Модулей 1–9**, Angular будет казаться набором случайных правил и «магии».

### 2. Зачем Angular нужен TypeScript?

TypeScript даёт контрактность, безопасный рефакторинг и ясную архитектуру для большого UI-кода. Он делает компоненты, сервисы, модели и шаблоны предсказуемыми. Но важно помнить, что браузер исполняет уже не TypeScript, а JavaScript, поэтому проверка входящих данных в runtime остаётся отдельной задачей.

### 3. Что делают декораторы в Angular?

Они маркируют классы и элементы метаданными, по которым Angular понимает, как их использовать. `@Component` описывает компонент, `@Injectable` — сервис для DI, `@Input` и `@Output` — контракт связи компонента снаружи. Это похоже на аннотации Spring, только реализовано в экосистеме JavaScript/TypeScript.

### 4. Как объяснить Zone.js в одном ответе?

Zone.js оборачивает async API браузера и помогает Angular отслеживать завершение асинхронной работы. Через monkey-patching он сохраняет логический контекст вызова и уведомляет инфраструктуру, когда задача закончилась. Это невидимый шпион и прослушка проводов вокруг асинхронности, а не механизм прямого обновления DOM.

### 5. В чём разница между `Default` и `OnPush`?

`Default` проверяет дерево компонентов шире и чаще после релевантных триггеров. `OnPush` старается проверять компонент точечнее: когда приходит новая ссылка входного значения, происходит событие внутри компонента, Observable через `async` pipe эмитит новое значение, вызывается `markForCheck` или меняется signal-зависимость. `OnPush` особенно хорошо сочетается с immutable-подходом.

### 6. Почему Observable так важен для Angular?

Потому что UI по природе потоковый: пользователь вводит текст, кликает, маршрут меняется, HTTP отвечает позже, форма живёт долго, websocket приносит новые события. Observable естественно описывает такие процессы, умеет композицию, время, переключение и отмену. Поэтому Angular исторически глубоко связан с RxJS.

### 7. Когда лучше использовать `BehaviorSubject`, а когда `ReplaySubject`?

`BehaviorSubject` удобен, когда важно текущее состояние прямо сейчас: список задач, флаг загрузки, авторизация. `ReplaySubject` нужен, когда новому подписчику важно увидеть не только «сейчас», но и недавнюю историю событий. Первый — табло, второй — короткий видеорегистратор.

### 8. Зачем Angular добавил Signals, если уже был RxJS?

Потому что Signals решают другую задачу: точное отслеживание state-зависимостей внутри view-слоя. Observable хорош для потоков во времени и async-композиции, а Signal хорош для локального реактивного состояния и fine-grained обновлений. Вместе они закрывают разные части реактивной архитектуры.

### 9. Чем Angular DI похож на Spring DI и чем отличается?

Похож самой идеей IoC: контейнер создаёт зависимости и внедряет их в объекты. Отличается средой и scope'ами. Angular работает в браузере и часто использует иерархические injector'ы, связанные с деревом компонентов. Это делает область жизни сервиса ближе к экрану или ветке UI, чем к серверному запросу.

### 10. Как hooks связаны с event loop и rendering?

Hooks вызываются не отдельно от общего исполнения, а внутри циклов Angular и JavaScript. Они отражают фазы жизни компонента: создание, инициализация, реакция на изменения, уничтожение. Поэтому правильный hook — это выбор правильного момента в уже существующем жизненном цикле UI.

### 11. Зачем нужны `trackBy`, `runOutsideAngular` и `detach`?

Они уменьшают лишнюю работу. `trackBy` стабилизирует DOM в списках, `runOutsideAngular` не даёт шумной активности будить Change Detection без смысла, `detach` позволяет вручную управлять тяжёлым участком экрана. Это разные способы экономить ресурсы и повышать предсказуемость поведения.

### 12. Какая главная карта знаний остаётся после курса?

Ты должен видеть путь от события пользователя до обновления интерфейса. Пользователь действует, браузер шлёт событие, JavaScript меняет состояние, асинхронность координируется event loop, Angular через свои реактивные и инфраструктурные механизмы узнаёт об изменении, затем нужные части UI перерисовываются. Если эта цепочка стала у тебя в голове цельной, курс выполнил свою задачу.

### SSR и Hydration (Server-Side Rendering)

Angular 17+ имеет **полноценную SSR** с hydration:

```typescript
// main.server.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideHttpClient } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { provideServerRendering } from '@angular/platform-server';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideServerRendering(),
    provideRouter(routes),
    provideHttpClient()
  ]
});

// main.ts (клиент)
import { bootstrapApplication, provideClientHydration } from '@angular/platform-browser';
import { provideHttpClient } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(),  // ← включает hydration на клиенте
    provideRouter(routes),
    provideHttpClient()
  ]
});
```

Без `provideClientHydration()` Angular перерисует DOM с нуля, потеряв преимущество серверного рендеринга.

**Зачем Spring-разработчику?**
- Аналог Thymeleaf/JSP: сервер отдаёт готовый HTML
- Но Angular hydration **не пересоздаёт** DOM — переиспользует серверный
- SEO, производительность первого кадра, Web Vitals

📚 **Источники:**
- [Angular docs: Server-side & hybrid rendering](https://angular.dev/guide/ssr)
- [Angular docs: Routing](https://angular.dev/guide/routing)
- [Angular docs: Components](https://angular.dev/guide/components)


---

### 🛠️ Angular DevTools

Расширение для Chrome, встроенный профайлер:

1. **Component tree**: визуальное дерево компонентов (как React DevTools)
2. **Change Detection profiler**: какие компоненты перерисовываются и почему
3. **Signal debugging**: отслеживание зависимостей signal → computed → effect
4. **Dependency Injection**: визуализация DI-дерева провайдеров

💡 Для дебага производительности: запустите profiler, выполните действие, посмотрите — какие компоненты запустили CD и сколько это заняло.

📚 **Источники:**
- [Angular DevTools](https://angular.dev/tools/devtools)
- [Angular docs: Change Detection](https://angular.dev/guide/change-detection)
- [Angular docs: Signals](https://angular.dev/guide/signals)


