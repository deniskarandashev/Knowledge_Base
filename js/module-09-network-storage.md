# Модуль 9: Сетевые запросы и хранение данных

## Для кого этот модуль

Этот модуль для тебя, если HTTP, REST и Spring для тебя уже не тёмный лес, но браузер пока кажется странной территорией со своими законами. На сервере ты обычно думаешь так: есть endpoint, есть клиент, есть статус, есть тело ответа. В браузере к этому внезапно добавляются ещё охрана, правила хранения данных, автоматическая отправка cookies, ограничения на чтение чужих ответов и куча UX-следствий.

Представь два мира. Сервер — это сотрудник офиса с пропуском во внутренние комнаты. Браузер — это посетитель в большом бизнес-центре: он быстрый, удобный, но везде турникеты и контроль доступа. Именно из-за этого даже знакомые слова вроде `fetch`, cookies и HTTP-статусов в браузере начинают вести себя чуть иначе.

Я буду объяснять это как ментор рядом: медленно в сложных местах, особенно в CORS, и с постоянными связями с Java `HttpClient`, Spring и повседневными аналогиями. Наша задача — не просто выучить API, а перестать чувствовать, что браузер “творит магию”.

## Каким должен быть результат

После этого модуля ты сможешь уверенно объяснить, как работают `XMLHttpRequest`, `fetch`, CORS, preflight, `AbortController`, `FormData`, WebSocket, SSE, `localStorage`, `sessionStorage`, cookies, IndexedDB и Service Worker. Причём не на уровне “я видел такой код”, а на уровне “я понимаю, зачем это существует, какие риски и когда это выбирать”.

Отдельная цель — научиться связывать темы между собой. Например, понимать, как CORS связан с cookies и `credentials`, как `fetch` связан с UX при ошибках, почему `localStorage` удобен для кэша, но плох для чувствительных данных, и почему WebSocket — не универсальная замена обычному HTTP.

## Главная установка модуля

**В браузере важно не только “как отправить запрос”, но и “имеет ли код право прочитать ответ”, “должны ли данные уйти на сервер автоматически”, “как долго они живут” и “какой риск для безопасности ты создаёшь”.**

Если держать эту мысль в голове, все темы модуля перестают быть набором разрозненных API и складываются в одну систему.

## 🎯 Interview-темы в этом модуле

На собеседовании особенно часто проверяют `fetch`, CORS, preflight, различия между `localStorage`, `sessionStorage` и cookies, а также базовое понимание WebSocket и Service Worker. Это те темы, где быстро видно, понимаешь ли ты браузерную среду или просто умеешь копировать шаблонный код.

Поэтому здесь мы будем много говорить не только “как”, но и “почему”. Именно на этом обычно сыпятся сильные backend-разработчики, которые отлично знают HTTP, но ещё не привыкли к браузерной модели безопасности.

## 9.1. XMLHttpRequest — коротко, чтобы понимать legacy

`XMLHttpRequest`, или **XHR**, — старый браузерный API для HTTP-запросов. Думай о нём как о факсе: он до сих пор умеет передавать документы, но рядом с современным мессенджером выглядит шумно и тяжеловесно. Для Java-разработчика это похоже на более низкоуровневый стиль клиента, где нужно самому следить за этапами запроса, а не просто описать намерение и получить результат.

Почему полезно знать XHR сегодня? Потому что он встречается в legacy-коде, старых библиотеках и проектах, которые пережили не одну миграцию. Плюс через XHR хорошо видно, почему `fetch` настолько полюбили: он лучше дружит с `Promise` и `async/await`, а код вокруг него меньше похож на ритуал.

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/tasks');
xhr.onload = () => {
  if (xhr.status >= 200 && xhr.status < 300) {
    console.log(JSON.parse(xhr.responseText));
  }
};
xhr.send();
```

### Разбор по шагам

1. Создаём объект XHR.
2. `open(...)` задаёт метод и URL, но не отправляет запрос.
3. Через `onload` подписываемся на момент получения ответа.
4. Внутри обработчика вручную проверяем HTTP-статус.
5. `responseText` приходит строкой, поэтому JSON нужно парсить вручную.
6. `send()` запускает запрос.

История ошибки очень типичная: разработчик проверяет только `onload`, забывает про отдельную обработку сетевой ошибки и потом смешивает в одну кучу `404`, таймаут и реальное отсутствие сети. Это одна из причин, почему в новом коде обычно выбирают `fetch`: с ним проще держать в голове поток выполнения.

Главное, что нужно унести: XHR — не “запрещённый” API, а просто более шумный. Концепции HTTP там те же, но современный код почти всегда читается лучше на `fetch`.

📚 **Источники:**
- [MDN: XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- [javascript.info: XMLHttpRequest](https://learn.javascript.ru/xmlhttprequest)
- [WHATWG: XMLHttpRequest Standard](https://xhr.spec.whatwg.org/)
- [w3schools: AJAX XMLHttpRequest](https://www.w3schools.com/js/js_ajax_http.asp)


## 9.2. 🎯 Fetch API — полный разбор

`fetch` — это стандартный API для HTTP-запросов, доступный в браузерах и Node.js 18+ (без полифиллов). Если XHR был похож на факс с кучей ручных действий, то `fetch` — это уже аккуратная служба доставки: ты формулируешь адрес, способ отправки, метаданные и потом получаешь понятный объект ответа. Для Java-мышления это ближе всего к `HttpClient` из Java 11+ или к аккуратному клиентскому слою поверх Spring API.

Важно сразу принять базовую мысль: `fetch` не возвращает тебе “готовый JSON”. Он возвращает **`Promise<Response>`**. Это как талон в кафе: блюдо ещё не у тебя на столе, но есть обещание, что ты его получишь. А сам `Response` — это не только тело, а ещё статус, заголовки и информация о том, как этот ответ читать.

### Самый простой GET

```javascript
const response = await fetch('/api/tasks');
const tasks = await response.json();
console.log(tasks);
```

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### Разбор по шагам

1. `fetch('/api/tasks')` отправляет GET-запрос.
2. `await` ждёт завершения промиса.
3. В `response` попадает объект `Response`, а не сразу массив задач.
4. `response.json()` — отдельный асинхронный шаг чтения тела.
5. После второго `await` мы получаем уже обычные данные JavaScript.

Это место часто удивляет новичков. Хочется думать, что “я сходил в API и сразу получил JSON”. Но HTTP-ответ — это не только тело. Иногда тебе нужен статус, иногда заголовки, иногда текст, иногда файл, иногда поток. Поэтому браузер сначала даёт конверт, а уже потом ты решаешь, как читать письмо внутри.

### Почему сначала “почему”, а потом код

Перед тем как писать `method`, `headers` и `body`, полезно задать себе четыре вопроса:

- я читаю, создаю, обновляю или удаляю?
- есть ли у запроса тело?
- в каком формате сервер ждёт это тело?
- какие служебные данные нужны: токен, cookies, тип контента, желаемый формат ответа?

Без этих вопросов `fetch` превращается в копирование заклинаний. С ними — в осмысленное описание запроса.

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### `method`, `headers`, `body`

#### `method`

HTTP method — это глагол запроса. В быту это как разный тип обращения в одну и ту же организацию: “покажи”, “создай”, “замени”, “исправь”, “удали”. В Spring это тот же мир `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`.

#### `headers`

Headers — это служебные наклейки на коробке. Они не являются самой коробкой, но помогают всем участникам понять, что внутри и как это обрабатывать. Самые частые герои: `Content-Type`, `Accept`, `Authorization`.

#### `body`

Body — это содержимое запроса. Если отправляешь JSON, объект нужно сериализовать через `JSON.stringify(...)`. Это очень похоже на то, как DTO в Java превращается в JSON перед отправкой.

```javascript
await fetch('/api/tasks', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    title: 'Изучить fetch',
    done: false
  })
});
```

📚 **Источники:**
- [MDN: Request](https://developer.mozilla.org/en-US/docs/Web/API/Request)
- [MDN: Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
- [MDN: Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Разбор по шагам

1. Выбираем `POST`, потому что хотим создать новую задачу.
2. Через `Content-Type` сообщаем серверу, что тело — JSON.
3. Обычный объект нельзя просто положить в `body`.
4. `JSON.stringify(...)` превращает объект в строку JSON.
5. Сервер на стороне Spring обычно примет это как `@RequestBody`.

Очень частая ошибка — машинально писать `Content-Type: application/json` вообще везде. Но если тела нет, заголовок может быть просто бессмысленным. Ещё одна ошибка — положить в `body` обычный объект и удивляться, что сервер “ничего не понял”. То есть нам важен не шаблон, а причина каждого поля.

### Обработка ответа

После запроса жизнь не заканчивается. Ответ надо правильно интерпретировать. Главная привычка: **сначала проверь статус, потом читай тело**. Это спасает и от пустых ответов, и от кривых контрактов, и от менее понятных ошибок.

```javascript
const response = await fetch('/api/tasks');

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}

const tasks = await response.json();
```

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### Разбор по шагам

1. Получаем `Response`.
2. Смотрим на `response.ok`.
3. Если статус не в диапазоне `200–299`, сами бросаем ошибку.
4. Только потом читаем JSON.
5. Так код явно разделяет транспорт и бизнес-результат.

`response.ok` — это короткая лампочка “зелёный/красный”. `response.status` — конкретный номер. Полезно помнить словарь: `200`, `201`, `204`, `400`, `401`, `403`, `404`, `409`, `500`. Для Java-разработчика это тот же словарь `HttpStatus`, только теперь он живёт в браузерном клиенте.

### Самая важная ловушка `fetch`

**`fetch` не бросает ошибку на `404` и `500`.**

Почему? Потому что с точки зрения сети обмен состоялся. Курьер доехал, дверь открыли, бумагу отдали. Просто на бумаге написано “не найдено” или “внутренняя ошибка сервера”. То есть транспорт успешен, а бизнес-результат плохой. Поэтому промис обычно `resolve`, а не `reject`.

```javascript
try {
  const response = await fetch('/api/missing-resource');
  console.log(response.status); // 404
} catch (error) {
  console.log('На 404 мы сюда обычно не попадём');
}
```

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### Разбор по шагам

1. Запрос реально отправляется.
2. Сервер отвечает `404`.
3. Браузер считает, что HTTP-обмен завершён нормально.
4. Поэтому промис завершается успешно.
5. Ошибку нужно определить по `response.ok` или `response.status`.

В `catch` чаще попадают совсем другие истории: отсутствует сеть, запрос отменён, браузер не дал прочитать ответ из-за CORS, произошёл иной низкоуровневый сбой.

### Мини-обёртка над `fetch`

Когда приложение становится живее одного учебного примера, удобно централизовать проверку статуса.

```javascript
async function apiRequest(url, options = {}) {
  const response = await fetch(url, options);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response;
}
```

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### Разбор по шагам

1. Функция принимает URL и настройки.
2. Внутри использует обычный `fetch`.
3. После ответа сразу проверяет `ok`.
4. Если статус плохой, бросает ошибку один раз, в одном месте.
5. Если всё хорошо, отдаёт `Response` дальше.

```javascript
async function loadTasks() {
  const response = await apiRequest('/api/tasks');
  return response.json();
}
```

### Разбор по шагам

1. `loadTasks()` больше не знает про детали проверки статуса.
2. Она использует готовый сетевой шаблон.
3. Если ошибка была, она уже проброшена.
4. Если нет — остаётся просто прочитать JSON.

Это похоже на клиентский сервисный слой в Java: низкоуровневые соглашения собраны в одном месте, а сценарии выше остаются короткими.

### POST с человекочитаемой ошибкой

Иногда сервер на ошибке отдаёт JSON вроде `{ message: "Title is required" }`. Было бы обидно проигнорировать это и показать пользователю только `HTTP 400`.

```javascript
// ✅ Надёжная проверка "есть ли тело ответа"
async function safeJson(response) {
  // Статусы без тела
  if (response.status === 204 || response.status === 304) {
    return null;
  }

  const text = await response.text();
  if (!text) return null;

  return JSON.parse(text);
}

async function createTask(task) {
  const response = await fetch('/api/tasks', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(task)
  });

  if (!response.ok) {
    const errorBody = await safeJson(response).catch(() => null);
    const message = errorBody?.message ?? `HTTP ${response.status}`;
    throw new Error(message);
  }

  return safeJson(response);
}
```

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### Разбор по шагам

1. Отправляем JSON через `POST`.
2. Если статус плохой, пробуем прочитать тело ошибки.
3. `.catch(() => null)` нужен на случай пустого или не-JSON тела.
4. Если сервер дал поле `message`, используем его.
5. Иначе падаем обратно на общий текст по статусу.
6. `safeJson(...)` сначала смотрит на статус-коды без тела (`204`, `304`).
7. Потом читает **фактический текст ответа**, а не верит слепо заголовку.
8. Если текста нет — возвращает `null`, если есть — парсит JSON.

Почему так надёжнее: `content-length` может отсутствовать (например, при chunked encoding), искажаться прокси или просто не отражать реальность. В браузерном коде безопаснее опираться на статус-код и реальное содержимое ответа.

### GET с токеном авторизации

```javascript
async function loadProfile(token) {
  const response = await fetch('/api/profile', {
    headers: {
      Authorization: `Bearer ${token}`,
      Accept: 'application/json'
    }
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}
```

📚 **Источники:**
- [MDN: Request](https://developer.mozilla.org/en-US/docs/Web/API/Request)
- [MDN: Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
- [MDN: Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Разбор по шагам

1. В `Authorization` кладём bearer-токен.
2. `Accept` подсказывает желаемый формат ответа.
3. Дальше идёт тот же жизненный цикл: ответ, проверка, чтение тела.

### Потоки: когда ответ приходит кусками

Иногда ответ не нужен целиком и сразу. Это может быть длинный лог, экспорт, генерация текста, прогресс операции. В быту это как если тебе не прислали книгу целиком, а читают её фрагментами по телефону. Если ждать самого конца, интерфейс будет “мертвым”. Если читать части по мере поступления, можно показывать промежуточный результат.

```javascript
const response = await fetch('/api/stream');
const reader = response.body.getReader();
const decoder = new TextDecoder();

let done = false;
let result = '';

while (!done) {
  const chunk = await reader.read();
  done = chunk.done;

  if (!done) {
    result += decoder.decode(chunk.value, { stream: true });
    console.log(result);
  }
}

result += decoder.decode(); // финальный flush для многобайтовых символов
```

📚 **Источники:**
- [MDN: Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
- [MDN: Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [MDN: Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Разбор по шагам

1. Получаем `Response` со stream body.
2. `getReader()` даёт ридер для чтения по кускам.
3. `TextDecoder` переводит байты в текст.
4. В цикле читаем очередной chunk.
5. Пока поток не завершён, склеиваем новый текст с предыдущим.
6. Можно обновлять UI прямо по мере поступления данных.

Для типичного CRUD это избыточно, но полезно понимать, что `fetch` умеет не только “дай мне JSON”.

### Частые ошибки как маленькие истории

**История 1.** Лена пишет `const data = response.json();` и получает не данные, а промис. Причина проста: чтение тела ответа — это тоже асинхронная операция.

**История 2.** Саша вызывает `response.json()` на `204 No Content` и ловит `Unexpected end of JSON input`. Проблема не в браузере, а в том, что он ожидал тело там, где его по контракту нет.

**История 3.** Максим кладёт в `body` обычный объект, не делает `JSON.stringify(...)`, а потом спорит с backend, что “API не работает”.

**История 4.** На `404` разработчик ждёт `catch` и не понимает, почему ошибка “проглатывается”. Но она не проглатывается — её просто нужно распознать по статусу.

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### Fetch vs XHR и связь с Java

`fetch` и XHR решают одну фундаментальную задачу: HTTP-запросы из браузера. Но XHR живёт событиями и ручным контролем, а `fetch` — промисами и композицией. Для Java-разработчика это примерно разница между шумной низкоуровневой работой с процессом и более современным, сервисным стилем.

Полезные соответствия:

- `fetch(url, options)` ↔ формирование `HttpRequest`;
- `Response` ↔ `HttpResponse`;
- `headers` ↔ `HttpHeaders`;
- `response.status` ↔ `statusCode()`;
- `response.json()` ↔ отдельный шаг десериализации.

Но не забывай ключевую разницу: серверный Java-код не живёт под Same-Origin Policy и CORS. Браузерный клиент — живёт.

📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [javascript.info: XMLHttpRequest](https://learn.javascript.ru/xmlhttprequest)


📚 **Источники:**
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [javascript.info: Fetch](https://learn.javascript.ru/fetch)
- [w3schools: JavaScript Fetch API](https://www.w3schools.com/js/js_api_fetch.asp)


### 🎯 Interview Focus: как рассказать про `fetch`

Хороший ответ звучит так:

> `fetch` — это стандартный API для HTTP-запросов, доступный в браузерах и Node.js 18+ (без полифиллов). Он принимает URL и `options`, где обычно указывают `method`, `headers`, `body`, `credentials`, `signal`. Возвращает `Promise<Response>`. Важно, что HTTP-ошибки вроде `404` и `500` не переводят промис в rejected, потому что сетевой обмен состоялся, поэтому нужно вручную проверять `response.ok` или `response.status`. Тело ответа читается отдельно через `json()`, `text()`, `blob()` или потоковые API.

### ✅ Проверь себя

**1. Что возвращает `fetch`?**  
A) Сразу JSON  
B) `Promise<Response>`  
C) Только статус-код  
D) Всегда строку

<details><summary>Ответ</summary>B) Это его базовая форма работы.</details>

**2. Что произойдёт при `404`?**  
A) Промис обязательно уйдёт в `catch`  
B) Браузер закроет вкладку  
C) Промис обычно зарезолвится, но `response.ok` будет `false`  
D) Ошибка превратится в `null`

<details><summary>Ответ</summary>C) Это самая частая ловушка.</details>

**3. Зачем нужен `Content-Type: application/json`?**  
A) Чтобы сервер понял формат body  
B) Чтобы ускорить рендер  
C) Чтобы отключить CORS  
D) Чтобы `fetch` стал синхронным

<details><summary>Ответ</summary>A) Этот заголовок объясняет серверу, как читать тело запроса.</details>

## 9.3. 🎯 CORS — Interview Focus!

CORS — тема, на которой спотыкаются даже сильные разработчики. И почти всегда проблема не в том, что человек “слабый”, а в том, что он пытается рассуждать о браузере как о сервере. А браузер — не свободный сетевой клиент. Это среда, которая защищает пользователя от слишком смелых страниц.

Если запомнишь только одну мысль, пусть это будет она: **CORS начинается не с CORS, а с Same-Origin Policy**. Сначала есть общее правило “нельзя свободно читать чужие ответы”, и только потом появляется механизм контролируемого разрешения.

### Что такое origin

Origin — это тройка:

- протокол;
- хост;
- порт.

`http://localhost:3000` и `http://localhost:8080` — разные origin из-за порта. `http://example.com` и `https://example.com` — разные origin из-за протокола. `https://app.example.com` и `https://api.example.com` — разные origin из-за хоста.

Бытовая метафора: origin — это не просто улица, а полный адрес “город + улица + дом”. Меняешь одну часть — приезжаешь уже в другое место.

📚 **Источники:**
- [MDN: Origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Same-Origin Policy: почему она вообще существует

**Same-Origin Policy (SOP)** — это правило браузера, которое не позволяет JavaScript-коду страницы свободно читать ответы другого origin без явного разрешения. Ключевое слово здесь — **читать**.

Представь сайт банка и вредоносный сайт, открытые у пользователя в разных вкладках. Если бы браузер разрешал любой странице без ограничений читать ответы любого сайта, вредоносная вкладка могла бы тихо сходить в банк от имени пользователя и прочитать баланс, профиль и историю операций. Именно от таких сценариев SOP и защищает.

📚 **Источники:**
- [MDN: Origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Паспортный контроль и клубный bouncer

Самая полезная метафора для CORS — **паспортный контроль**. Страница — это пассажир. Другой origin — это другая страна. Сам факт существования самолёта не означает, что тебя пустят через границу. Нужно разрешение.

Вторая метафора — **клубный bouncer**. Ты подошёл ко входу. Музыка слышна, дверь рядом, но охранник спрашивает: “Ты в списке?” CORS — это как раз список гостей, который сервер передаёт браузеру-охраннику.

Java/Spring-аналогия: на сервере разрешение часто выражается через `@CrossOrigin` или глобальную CORS-конфигурацию. То есть список гостей пишет сервер, не фронтенд.

📚 **Источники:**
- [MDN: Origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Что такое CORS

**CORS** — это Cross-Origin Resource Sharing: набор HTTP-заголовков, которыми сервер говорит браузеру, можно ли конкретной странице читать его ответ.

Самый известный заголовок:

```http
Access-Control-Allow-Origin: https://app.example.com
```

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Разбор по шагам

1. Страница открыта с `https://app.example.com`.
2. Она делает запрос на другой origin.
3. Сервер отвечает.
4. Браузер проверяет CORS-заголовки.
5. Если origin страницы разрешён, браузер отдаёт ответ JavaScript-коду.
6. Если нет — блокирует чтение ответа.

И вот здесь backend-разработчикам особенно важно остановиться: **сервер мог реально ответить, но браузер всё равно не даст JS прочитать тело**. Именно поэтому CORS кажется мистическим, если не держать в голове роль браузера как охраны.

### Простой кросс-origin GET

```javascript
const response = await fetch('http://localhost:8080/api/tasks');
const tasks = await response.json();
```

📚 **Источники:**
- [MDN: Origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Разбор по шагам

1. Скрипт работает, например, на `http://localhost:3000`.
2. Запрос уходит на `http://localhost:8080`.
3. Для браузера это другой origin.
4. Если сервер вернёт `Access-Control-Allow-Origin: http://localhost:3000`, браузер даст прочитать ответ.
5. Если не вернёт — в консоли появится CORS-ошибка.

### Ключевая мысль: запрет не на “отправить”, а на “прочитать”

Это суперважно. Браузер может физически отправить запрос. Сервер может даже выполнить действие. Но потом браузер может не отдать результат JavaScript-коду.

Представь курьера, которого пустили в архив, но запретили вынести копии документов наружу. Он дошёл. Разговор состоялся. Но вернуть содержимое клиентскому коду нельзя.

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Что такое preflight

Вот мы и пришли к самому страшному слову.

**Preflight** — это предварительный `OPTIONS`-запрос, который браузер отправляет перед основным кросс-origin запросом, если хочет заранее уточнить правила.

Браузер как будто спрашивает:

> Я сейчас приду вот с таким методом, вот с такими заголовками. Ты вообще это разрешаешь?

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Когда preflight появляется

Чаще всего, когда:

- метод не “простой” (`PUT`, `PATCH`, `DELETE` и т.п.);
- используются нестандартные заголовки;
- есть `Content-Type: application/json`;
- участвуют более сложные cross-origin условия.

**Простой (simple) запрос** — тот, который **не вызывает preflight**. Условия:

1. Метод: `GET`, `HEAD`, или `POST`
2. Заголовки — только из "безопасного" списка: `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`
3. `Content-Type` (если есть): только `text/plain`, `multipart/form-data`, или `application/x-www-form-urlencoded`
4. Нет event listeners на `XMLHttpRequest.upload` *(актуально только для XMLHttpRequest, не для обычного `fetch`-примера)*
5. Нет `ReadableStream` в теле

Всё остальное → preflight (`OPTIONS` запрос перед основным).

Это не магия. Это просто отдельный этап проверки.

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Пример запроса, который почти наверняка вызовет preflight

```javascript
await fetch('https://api.example.com/tasks', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: 'Bearer token-123'
  },
  body: JSON.stringify({ title: 'Новая задача' })
});
```

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Разбор по шагам

1. Запрос к другому origin.
2. Используется `POST`.
3. Есть `application/json`.
4. Есть `Authorization`.
5. Браузер решает не идти вслепую и сначала отправляет `OPTIONS`.

### Как выглядит preflight-запрос

```http
OPTIONS /tasks HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization, content-type
```

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Разбор по шагам

1. `OPTIONS /tasks` — это ещё не реальный `POST`.
2. `Origin` сообщает, какая именно страница хочет доступ.
3. `Access-Control-Request-Method` сообщает, какой метод будет позже.
4. `Access-Control-Request-Headers` перечисляет заголовки, которые клиент хочет использовать.

Это почти анкета на входе в клуб: “я пришёл отсюда, хочу сделать вот это, с собой у меня вот это”.

### Как должен ответить сервер

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 3600
```

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Разбор по шагам

1. `204` означает, что серверу не нужно возвращать тело.
2. `Allow-Origin` говорит, кому можно.
3. `Allow-Methods` говорит, какие методы допустимы.
4. `Allow-Headers` говорит, какие заголовки допустимы.
5. `Max-Age` позволяет браузеру какое-то время не повторять один и тот же preflight.

Если сервер не дал нужного разрешения, браузер **не отправит основной запрос**. Поэтому иногда кажется, что “мой `POST` даже не доходит”. Всё верно: он умер раньше, на этапе проверки у двери.

### Preflight шаг за шагом: медленно, как на доске

Допустим:

- фронт живёт на `http://localhost:3000`;
- API живёт на `http://localhost:8080`;
- ты хочешь отправить `POST /api/tasks` с JSON и `Authorization`.

#### Шаг 1. Пользователь нажал “Создать задачу”

Код вызывает `fetch(...)`.

#### Шаг 2. Браузер смотрит на запрос

Он видит другой origin, метод `POST`, JSON и заголовок авторизации.

#### Шаг 3. Браузер ещё не отправляет реальный `POST`

Сначала уходит `OPTIONS`.

```http
OPTIONS /api/tasks HTTP/1.1
Origin: http://localhost:3000
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization, content-type
```

#### Шаг 4. Сервер проверяет свою CORS-настройку

Он сравнивает origin, метод и заголовки с тем, что у него разрешено.

#### Шаг 5. Если всё ок, сервер отвечает

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
```

#### Шаг 6. Браузер читает ответ на preflight

Твоему JavaScript этот ответ не показывают. Это служебный разговор браузера с сервером.

#### Шаг 7. Только теперь браузер отправляет настоящий `POST`

Вот теперь уходит JSON-тело с задачей.

#### Шаг 8. Сервер отвечает на основной запрос

И на этом ответе тоже должен быть корректный `Access-Control-Allow-Origin`, иначе браузер снова не даст JS прочитать результат.

Это место часто упускают: **успешный preflight не отменяет CORS-требования к основному ответу**.

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Почему одинаковый код иногда “вдруг” начинает требовать preflight

- раньше был `GET`, стал `PATCH`;
- раньше не было `Authorization`, потом появился токен;
- раньше фронт и API жили на одном origin через прокси, потом их разнесли по разным портам;
- раньше заголовки были простыми, потом добавили кастомный `X-Trace-Id`.

То есть проблема не случайная. Изменились условия, браузер включил дополнительную проверку.

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Credentials: cookies и ещё одна большая путаница

Если ты хочешь, чтобы браузер в cross-origin запросе отправлял cookies, нужно явно это указать:

У `fetch` три режима credentials: `'omit'` (никогда не отправлять cookies), `'same-origin'` (по умолчанию — только для same-origin), `'include'` (всегда, включая cross-origin). Для cross-origin с cookies нужно И `credentials: 'include'` на клиенте, И `Access-Control-Allow-Credentials: true` на сервере.

```javascript
const response = await fetch('https://api.example.com/profile', {
  credentials: 'include'
});
```

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests (#credentials)](https://learn.javascript.ru/fetch-crossorigin#credentials)
- [MDN: Request.credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials)
- [MDN: RequestInit.credentials](https://developer.mozilla.org/en-US/docs/Web/API/RequestInit#credentials)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Разбор по шагам

1. По умолчанию браузер не обязан тащить cross-origin cookies в произвольный `fetch`.
2. `credentials: 'include'` говорит: “если есть подходящие cookies, отправь их тоже”.
3. Но этого мало: сервер тоже должен согласиться.

Серверу нужно вернуть примерно так:

```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```

### Разбор по шагам

1. `Allow-Credentials: true` сообщает, что сервер допускает cookies/credentials в таком сценарии.
2. `Allow-Origin` при этом должен быть **конкретным**.
3. `*` вместе с credentials использовать нельзя.

### Почему `*` нельзя вместе с credentials

Потому что это означало бы “любой сайт может прислать пользовательские cookies и читать ответ”. С точки зрения безопасности это слишком щедро. Возвращаясь к клубу: VIP-гостей нельзя пускать по принципу “да хоть кто”. Список должен быть точным.

📚 **Источники:**
- [javascript.info: Fetch: Cross-Origin Requests (#credentials)](https://learn.javascript.ru/fetch-crossorigin#credentials)
- [MDN: Request.credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials)
- [MDN: RequestInit.credentials](https://developer.mozilla.org/en-US/docs/Web/API/RequestInit#credentials)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Четыре очень частые ошибки как истории

#### История 1. “Поставлю `mode: 'no-cors'` и всё исчезнет”

Разработчик радуется, что ошибка в консоли исчезла, но потом понимает, что ответ читать нельзя. Он не починил CORS, а перевёл запрос в режим, где результат для JS почти бесполезен.

#### История 2. “Но в Postman же работает”

Работает, потому что Postman — не браузер. На него не действует Same-Origin Policy. Это ничего не доказывает про поведение фронта в реальной вкладке.

#### История 3. “Я добавил `Access-Control-Allow-Origin` в request headers”

Это всё равно что прийти в клуб с запиской “я сам себе разрешил войти”. Этот заголовок должен прийти **от сервера в ответе**, а не от клиента в запросе.

#### История 4. “Почему backend получает `OPTIONS`, кто это придумал?”

Это придумал браузер. И если сервер не умеет корректно обработать preflight, реальный запрос не случится.

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### Как чинить CORS правильно

Правильная формулировка очень простая:

**CORS чинится на сервере или в прокси.**

Что можно сделать:

- настроить `Access-Control-Allow-Origin`;
- разрешить нужные методы;
- разрешить нужные заголовки;
- включить `Access-Control-Allow-Credentials`, если нужны cookies;
- использовать dev proxy, чтобы локально жить как будто на одном origin;
- не плодить лишние кастомные заголовки без необходимости.

Чего фронт сделать не может:

- сам себе выдать разрешение на чтение чужого ответа;
- исправить отсутствие серверных CORS-заголовков;
- честно обойти Same-Origin Policy без участия сервера.

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)


### Как разговаривать с backend-командой

Вместо расплывчатого “у меня CORS не работает” полезно говорить так:

- фронт идёт с `http://localhost:3000`;
- endpoint такой-то;
- браузер отправляет preflight `OPTIONS`;
- сервер не возвращает `Access-Control-Allow-Origin`/`Methods`/`Headers`;
- если используем cookies — нужен `Allow-Credentials: true` и конкретный origin.

Такой разговор мгновенно переводит проблему из эмоций в инженерную плоскость.

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)


### Мини-диагностика по симптомам

- Ошибка про `Access-Control-Allow-Origin` → сервер не разрешил origin.
- Ошибка про preflight → проблема в ответе на `OPTIONS`.
- Cookies не отправляются → проверь `credentials: 'include'`, `Allow-Credentials`, конкретный origin и атрибуты самих cookies.
- В `curl` ок, в браузере нет → почти наверняка дело в CORS/SOP.

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)


### 🛠️ DevTools: диагностика сетевых проблем

**Network tab — ваш главный инструмент:**

1. **Preflight запросы**: фильтр `Method: OPTIONS` — увидите CORS preflight
2. **CORS ошибки**: в консоли "blocked by CORS policy", но **реальная причина** — в Response Headers OPTIONS запроса
3. **Как отличить network error от CORS block**: 
   - Network error → запрос не дошёл (нет строки в Network tab)
   - CORS block → запрос ушёл, ответ пришёл, но **браузер заблокировал доступ к телу**
4. **Cookies**: вкладка Cookies в деталях запроса — видны отправленные и полученные
5. **Replay**: правый клик → "Replay XHR" для повторного запроса

💡 **Типичная ловушка**: `fetch` с CORS-ошибкой выбрасывает `TypeError: Failed to fetch` — без деталей! Всегда смотрите Network tab.

📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)


### Кросс-связь с cookies

Очень важно увидеть, что CORS и cookies — не отдельные острова. Как только в игру входят сессии и `credentials`, темы из разделов 9.3 и 9.9 сцепляются. Именно поэтому взрослое понимание браузера строится не по темам, а по связям между ними.

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [javascript.info: Fetch: Cross-Origin Requests (#credentials)](https://learn.javascript.ru/fetch-crossorigin#credentials)
- [MDN: Request.credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)


📚 **Источники:**
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [javascript.info: Fetch: Cross-Origin Requests](https://learn.javascript.ru/fetch-crossorigin)
- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [Fetch Standard](https://fetch.spec.whatwg.org/)


### 🎯 Interview Focus: готовый ответ

Хороший ответ звучит так:

> Same-Origin Policy — это базовое браузерное ограничение, которое не позволяет JavaScript-коду страницы свободно читать ответы другого origin. Origin — это протокол, хост и порт. CORS — это механизм, с помощью которого сервер через `Access-Control-Allow-*` заголовки разрешает конкретным origin читать его ответы. Для непростых запросов браузер сначала отправляет preflight `OPTIONS`, где спрашивает разрешение на метод и заголовки. Если используются cookies, на клиенте нужен `credentials: 'include'`, а сервер должен вернуть `Access-Control-Allow-Credentials: true` и конкретный `Access-Control-Allow-Origin`, не `*`. Чинят CORS на сервере или в прокси, не на фронте.

### ✅ Проверь себя

**1. Что такое origin?**  
A) Только домен  
B) Протокол + хост + порт  
C) Любой полный URL  
D) Только IP

<details><summary>Ответ</summary>B) Именно эта тройка и определяет origin.</details>

**2. Что такое preflight?**  
A) Повторная загрузка страницы  
B) Предварительный `OPTIONS`-запрос браузера  
C) Авто-ретрай `fetch`  
D) Тип cookie

<details><summary>Ответ</summary>B) Он проверяет, можно ли делать основной запрос.</details>

**3. Где обычно чинят реальную CORS-проблему?**  
A) На сервере или в прокси  
B) В CSS  
C) В `localStorage`  
D) В `JSON.parse`

<details><summary>Ответ</summary>A) Клиент не может сам себе разрешить доступ.</details>

## 9.4. AbortController — отмена запросов

`AbortController` нужен для ситуаций, когда запрос технически можно продолжать, но тебе он уже не нужен. Бытовая метафора — заказ такси, который ты отменяешь до приезда. Или заказ блюда в ресторане: кухня уже могла начать работу, но ты говоришь официанту “всё, отмена”.

В Java-мышлении это похоже на `Future.cancel(...)`: клиент перестаёт ждать результат. И вот здесь очень важно не спутать две вещи: **отмена ожидания на клиенте** и **гарантированное прекращение работы на сервере**. Первое `AbortController` делает, второе — не обещает.

```javascript
const controller = new AbortController();

fetch('/api/tasks', {
  signal: controller.signal
});

controller.abort();
```

### Разбор по шагам

1. Создаём контроллер.
2. `signal` передаём в `fetch`.
3. `abort()` сообщает, что операция больше не нужна.
4. `fetch` обычно завершается ошибкой `AbortError`.

Главные сценарии — live-search, уход со страницы, переключение вкладок интерфейса, клиентский таймаут. Во всех них важно не показывать пользователю старые или уже неактуальные результаты.

```javascript
const controller = new AbortController();

try {
  const response = await fetch('/api/tasks', {
    signal: controller.signal
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  const tasks = await response.json();
  console.log(tasks);
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Запрос отменён сознательно');
  } else {
    console.error(error);
  }
}
```

### Разбор по шагам

1. Запускаем `fetch` под управлением `signal`.
2. Если ответ придёт, идём обычным путём.
3. Если произойдёт отмена, попадаем в `catch`.
4. По `error.name === 'AbortError'` отличаем отмену от обычной ошибки сети или сервера.

История ошибки очень жизненная: разработчик делает поиск по мере ввода, не отменяет старые запросы и потом видит, как старый ответ приезжает позже нового и затирает интерфейс. В такой момент кажется, что “сеть ведёт себя хаотично”, хотя на самом деле просто не был настроен жизненный цикл запросов.

Современный бонус: браузеры постепенно дают более удобные способы задавать таймаут прямо через `AbortSignal`, без ручного `setTimeout`.

```javascript
// Современный способ — AbortSignal.timeout() (Web API, поддерживается в современных браузерах и Node.js 18+)
const response = await fetch('/api/data', {
  signal: AbortSignal.timeout(5000) // автоматический abort через 5 секунд
});

// Комбинация: ручная отмена + таймаут
const controller = new AbortController();
const responseWithFallback = await fetch('/api/data', {
  signal: AbortSignal.any([
    controller.signal,
    AbortSignal.timeout(5000)
  ])
});
// controller.abort() — ручная отмена
// Через 5 сек — автоматическая отмена
```

📚 **Источники:**
- [javascript.info: Fetch: Abort](https://learn.javascript.ru/fetch-abort)
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [MDN: AbortSignal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)
- [DOM Standard: AbortController](https://dom.spec.whatwg.org/#interface-abortcontroller)


### ✅ Проверь себя

**1. Что делает `AbortController`?**  
A) Сериализует JSON  
B) Позволяет отменить поддерживаемую операцию  
C) Хранит cookies  
D) Чинит CORS

<details><summary>Ответ</summary>B) Это стандартный механизм отмены.</details>

**2. Как выглядит ошибка отмены?**  
A) `AbortError`  
B) `HttpError`  
C) `CorsError`  
D) `StorageError`

<details><summary>Ответ</summary>A) Её полезно отличать от других ошибок.</details>

## 9.5. FormData и загрузка файлов

Как только в приложении появляется файл, привычная JSON-модель начинает скрипеть. `FormData` нужен ровно для этого: упаковать в один запрос и обычные поля формы, и файл. Представь прозрачную папку с кармашками: в одном лежит `title`, в другом `description`, в третьем сам файл. Браузер знает, как превратить такую папку в `multipart/form-data`.

Java/Spring-аналогия здесь очень прямая: на клиенте `FormData`, на сервере `MultipartFile` и поля формы. Это естественная пара.

```javascript
const formData = new FormData();
formData.append('title', 'Новая задача');
formData.append('attachment', fileInput.files[0]);

await fetch('/api/tasks/upload', {
  method: 'POST',
  body: formData
});
```

### Разбор по шагам

1. Создаём пустой контейнер `FormData`.
2. Добавляем обычное поле `title`.
3. Добавляем файл из input.
4. Отправляем всё одним `POST`.
5. Браузер сам формирует multipart-тело.

Самая важная ловушка: **не ставь `Content-Type` руками**, если отправляешь `FormData`. Браузер сам добавляет корректный `multipart/form-data` вместе с `boundary`. Если выставить заголовок вручную, легко сломать формат.

```javascript
const formData = new FormData();
formData.append('title', 'Отчёт');
formData.append('description', 'Файл за июнь');
formData.append('file', fileInput.files[0]);

const response = await fetch('/api/reports', {
  method: 'POST',
  body: formData
});

if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}
```

### Разбор по шагам

1. В контейнер кладём несколько полей и файл.
2. Отправляем их одним запросом.
3. После ответа всё равно проверяем `response.ok` — правила `fetch` не исчезают.
4. Если сервер вернул ошибку, обрабатываем её обычным путём.

Типичная история ошибки: разработчик по привычке добавляет `Content-Type: multipart/form-data`, сервер “внезапно перестаёт видеть файл”, и начинается охота на мнимый баг в backend. На деле сломан был именно клиентский заголовок. И да: если файл летит на другой origin, CORS из раздела 9.3 никуда не делся — `FormData` не отменяет браузерную безопасность.

📚 **Источники:**
- [javascript.info: FormData](https://learn.javascript.ru/formdata)
- [MDN: FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [MDN: Using FormData Objects](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects)


### ✅ Проверь себя

**1. Когда `FormData` особенно полезен?**  
A) Для файлов и multipart-форм  
B) Для WebSocket  
C) Для cookies  
D) Для `setTimeout`

<details><summary>Ответ</summary>A) Это его главный сценарий.</details>

**2. Нужно ли вручную задавать `Content-Type` для `FormData`?**  
A) Да, всегда  
B) Нет, браузер обычно делает это сам  
C) Да, только `application/json`  
D) Только при `GET`

<details><summary>Ответ</summary>B) Это частая и важная ловушка.</details>

## 9.6. WebSocket — основы

WebSocket нужен там, где обычные письма уже слишком медленные и формальные. Если `fetch` — это обмен письмами: запрос, потом ответ, потом новый запрос, то **WebSocket** — это телефонный звонок. Соединение открыли один раз, и дальше обе стороны могут говорить почти в реальном времени.

Это очень полезная метафора, потому что сразу становится видно, когда WebSocket нужен, а когда нет. Для чата телефонный звонок логичен. Для одноразовой загрузки списка задач — уже нет, там обычное письмо через `fetch` проще и дешевле.

Вторая метафора: `fetch` — это каждый раз подойти к стойке информации с новым вопросом. WebSocket — это открыть постоянную линию с оператором. Java-аналогия — Spring WebSocket/STOMP и любая инфраструктура real-time событий.

Интересный факт: WebSocket **не подчиняется** обычному CORS-механизму. Вместо preflight-запроса браузер отправляет заголовок `Origin` в HTTP Upgrade handshake, а сервер сам решает, принять ли соединение. Это упрощает настройку, но и перекладывает ответственность на серверную сторону.

```javascript
const socket = new WebSocket('wss://example.com/ws');

socket.addEventListener('open', () => {
  console.log('Соединение открыто');
});

socket.addEventListener('message', event => {
  console.log('Пришло сообщение:', event.data);
});
```

### Разбор по шагам

1. Создаём WebSocket-соединение по `wss://` URL.
2. Ждём событие `open`.
3. Когда сервер присылает данные, срабатывает `message`.
4. `event.data` содержит полезную нагрузку.

```javascript
socket.addEventListener('open', () => {
  socket.send(JSON.stringify({
    type: 'chat-message',
    text: 'Привет!'
  }));
});
```

### Разбор по шагам

1. Сначала убеждаемся, что соединение реально открыто.
2. Через `send(...)` отправляем сообщение серверу.
3. Часто данные сериализуются в JSON.
4. Сервер обрабатывает уже не обычный HTTP-body, а сообщение в постоянном канале.

Когда WebSocket уместен: чат, live-данные, совместное редактирование, игры, торговые экраны. Когда он обычно лишний: обычный CRUD, загрузка страницы, форма регистрации, “получить список и успокоиться”.

История ошибки очень типичная: разработчик тащит WebSocket в pet-project “список книг”, чтобы было “современно”, а потом получает reconnection-логику, heartbeat, баги при потере сети и ноль реальной пользы. Более мощный инструмент — не всегда лучший. Именно поэтому следующий раздел про SSE тоже важен: иногда нужен поток событий, но не двусторонний разговор.

📚 **Источники:**
- [javascript.info: WebSocket](https://learn.javascript.ru/websocket)
- [MDN: WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [WHATWG: WebSockets Standard](https://websockets.spec.whatwg.org/)


### ✅ Проверь себя

**1. Как лучше всего описать WebSocket?**  
A) Постоянный двусторонний канал  
B) Новый тип cookies  
C) Локальное хранилище  
D) Вид JSON

<details><summary>Ответ</summary>A) Именно двусторонность делает WebSocket особенным.</details>

**2. Что чаще выбрать для разовой загрузки списка задач?**  
A) WebSocket  
B) Обычно `fetch`  
C) Только SSE  
D) Только IndexedDB

<details><summary>Ответ</summary>B) Для одноразового CRUD-запроса HTTP проще.</details>

## 9.7. Server-Sent Events

SSE, или **Server-Sent Events**, полезно понимать как облегчённый real-time сценарий. Если WebSocket — это телефонный звонок, где говорят обе стороны, то SSE — это радио или живое табло: сервер говорит, клиент слушает.

Такой инструмент уместен, когда тебе нужен поток уведомлений, статусов, логов или прогресса, но не нужен полноценный двусторонний разговор. В этом его сила: иногда он проще WebSocket и по ментальной модели, и по инфраструктуре.

```javascript
const source = new EventSource('/api/events');

source.onmessage = event => {
  console.log('Новое событие:', event.data);
};
```

### Разбор по шагам

1. `EventSource` открывает соединение на endpoint событий.
2. Сервер держит его открытым и периодически присылает данные.
3. Когда приходит новое сообщение, срабатывает `onmessage`.
4. `event.data` содержит полезную нагрузку.

История ошибки простая: команде нужен только поток логов деплоя, но она тащит WebSocket “потому что real-time”. А потом поддерживает больше сложности, чем требуется. Если клиенту не нужно постоянно говорить обратно, SSE может быть намного естественнее.

**Ограничения EventSource:**
- Только `GET` запросы (нельзя `POST`/`PUT`)
- Нельзя добавить кастомные headers (проблема с `Authorization`!)
- Встроенный auto-reconnect с заголовком `Last-Event-ID`
- Для auth: используйте cookies или query-параметры (token в URL — плохая практика, но иногда единственный вариант)

**Если нужны custom headers** → используйте `fetch` + `ReadableStream` или библиотеку вроде `eventsource-parser`.

📚 **Источники:**
- [javascript.info: Server-Sent Events](https://learn.javascript.ru/server-sent-events)
- [MDN: EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)
- [HTML Standard: Server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html)


### ✅ Проверь себя

**1. Что лучше всего описывает SSE?**  
A) Двусторонний канал  
B) Односторонний поток событий от сервера к клиенту  
C) Локальная база данных  
D) Вид cookie

<details><summary>Ответ</summary>B) Это его главная идея.</details>

**2. Какой API используют для SSE?**  
A) `EventSource`  
B) `AbortController`  
C) `FormData`  
D) `document.cookie`

<details><summary>Ответ</summary>A) Это стандартный браузерный интерфейс для SSE.</details>

## 9.8. 🎯 localStorage и sessionStorage — Interview Focus

После сети почти всегда приходит вопрос: а где жить данным на клиенте? Не “где-нибудь”, а с каким сроком жизни, для каких сценариев и с какими рисками. Именно тут новичок часто выбирает хранилище по привычке, а зрелый разработчик — по истории использования.

`localStorage` и `sessionStorage` — это простые key-value хранилища браузера. Представь шкафчик с ячейками: у каждой ячейки есть ключ, а внутри лежит строка. Java-аналогия — очень маленький `Map<String, String>`, который живёт не на сервере, а в браузере пользователя.

### `localStorage`: долгоживущий шкафчик

`localStorage` сохраняет данные надолго. Пользователь может закрыть вкладку, браузер, вернуться завтра — и данные обычно останутся. Это хорошо подходит для темы интерфейса, языка, небольшого кэша, черновика, последнего успешного состояния списка.

Бытовая метафора: это ящик рабочего стола, куда ты кладёшь блокнот и находишь его там же завтра.

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### `sessionStorage`: состояние одной вкладки

`sessionStorage` живёт, пока жива конкретная вкладка. Закрыл вкладку — данные исчезли. Это удобно для шагов формы, временного мастера регистрации, состояния экрана, которое не должно пережить закрытие. Нюанс: `sessionStorage` **переживает** перезагрузку страницы (F5/refresh) — данные исчезают только при закрытии вкладки. При дупликации вкладки (Ctrl+Click) новая вкладка получает КОПИЮ sessionStorage, но дальше живёт независимо.

Бытовая метафора: листок на столе переговорной. Пока встреча идёт, он полезен. Встреча закончилась — листок выбросили.

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### Оба хранилища строковые

```javascript
localStorage.setItem('tasks', JSON.stringify(tasks));
const saved = JSON.parse(localStorage.getItem('tasks') ?? '[]');
```

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### Разбор по шагам

1. Сохраняем значение по ключу `tasks`.
2. Перед сохранением сериализуем массив задач в JSON-строку.
3. `getItem(...)` возвращает строку или `null`.
4. Через `?? '[]'` задаём запасной вариант.
5. `JSON.parse(...)` превращает строку обратно в массив.

### Не начинай с вопроса “что лучше?”

Начинай с вопроса “что это за данные?”.

- Это тема сайта?
- Это временный шаг формы?
- Это офлайн-кэш?
- Это чувствительный токен?
- Это что-то, что браузер должен сам отправлять на сервер?

Пока ты не ответил на эти вопросы, выбор между storage и cookies будет гаданием.

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### История: когда что выбирать

Представь Аню и её `task-manager`.

**Сцена 1.** Пользователь выбрал тёмную тему. Серверу это знание не нужно на каждый запрос. Идеальный кандидат — `localStorage`.

**Сцена 2.** Пользователь заполняет форму из трёх шагов в одной вкладке. Если обновил страницу — хорошо бы сохранить промежуточное состояние. Если завтра откроет новую вкладку, старые шаги уже не нужны. Тут часто уместен `sessionStorage`.

**Сцена 3.** Теперь нужно поддержать серверную сессию. Браузер должен автоматически отправлять идентификатор на backend. Здесь естественно появляются cookies, а не storage.

Вот поэтому одной таблицы различий мало. Решение рождается из сценария, а не из голого API.

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### Размер, срок жизни и безопасность

- `localStorage` и `sessionStorage` обычно дают несколько мегабайт.
- Cookies заметно меньше и не подходят для хранения больших кусков данных.
- Storage не отправляется на сервер автоматически.
- Cookies могут отправляться автоматически вместе с HTTP-запросами.
- Storage доступен JavaScript-коду страницы, а значит уязвим для чтения при XSS.

И вот здесь рождается самая болезненная тема: **почему хранить чувствительные токены в `localStorage` рискованно**. Не потому что storage “плохой”, а потому что любой скрипт на странице, включая вредоносный при XSS, сможет его прочитать.

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### Мини-таблица с правильным смыслом

| Хранилище | Срок жизни | Уходит на сервер автоматически | Доступно JS | Типичный сценарий |
|---|---|---:|---:|---|
| `localStorage` | Долго | Нет | Да | тема, язык, локальный кэш |
| `sessionStorage` | Пока жива вкладка | Нет | Да | шаги формы, временное состояние |
| Cookies | По `Max-Age`/`Expires` | Да | Иногда, если не `HttpOnly` | серверная сессия, auth |

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### Три частые ошибки как истории

**История 1.** Разработчик кладёт JWT в `localStorage`, потому что “так проще”, а потом не думает про XSS. Простота API не отменяет угрозы.

**История 2.** Кто-то тащит в cookie тяжёлый JSON профиля. В результате каждый запрос носит с собой лишний груз.

**История 3.** Человек хранит важный черновик в `sessionStorage` и удивляется, что после закрытия вкладки всё исчезло. Но именно так это хранилище и должно работать.

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### Связь с IndexedDB и Service Worker

Если нужен серьёзный офлайн-кэш, `localStorage` часто маловат и слишком примитивен. Тогда смотри на IndexedDB и Service Worker. Это важная перекрёстная связь: простые storage решают не все клиентские сценарии хранения.

📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [javascript.info: IndexedDB](https://learn.javascript.ru/indexeddb)
- [MDN: IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)


📚 **Источники:**
- [javascript.info: localStorage, sessionStorage](https://learn.javascript.ru/localstorage)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [w3schools: Web Storage API](https://www.w3schools.com/js/js_api_web_storage.asp)


### 🎯 Interview Focus: storage vs cookies

На собеседовании удобно отвечать так:

> `localStorage` и `sessionStorage` — это браузерные key-value хранилища, доступные JavaScript-коду страницы. `localStorage` живёт долго, `sessionStorage` — до закрытия вкладки, и ни одно из них не отправляется на сервер автоматически. Cookies отличаются тем, что браузер может прикладывать их к HTTP-запросам, а ещё у них есть атрибуты безопасности вроде `HttpOnly`, `Secure`, `SameSite`. Поэтому storage подходит для локального UI-состояния и небольшого кэша, а cookies часто используют для серверной сессии и аутентификации.

### ✅ Проверь себя

**1. Что живёт дольше?**  
A) `sessionStorage`  
B) `localStorage`  
C) Переменная в функции  
D) `AbortController`

<details><summary>Ответ</summary>B) `localStorage` обычно переживает закрытие вкладки и браузера.</details>

**2. Что происходит с `sessionStorage` после закрытия вкладки?**  
A) Очищается  
B) Становится cookie  
C) Уходит на сервер  
D) Переезжает в IndexedDB

<details><summary>Ответ</summary>A) Это его главный жизненный цикл.</details>

**3. Почему опасно хранить секреты в `localStorage`?**  
A) Потому что он слишком маленький  
B) Потому что любой JS-код страницы может его прочитать  
C) Потому что он вызывает preflight  
D) Потому что он работает только в Angular

<details><summary>Ответ</summary>B) Риск XSS здесь главный.</details>

## 9.9. Cookies

Cookies лучше понимать не как “ещё одно маленькое хранилище”, а как **механизм, через который браузер автоматически приносит серверу нужную записку**. Самая удобная метафора — гардеробный номерок или пропуск. Тебя узнают не по длинному рассказу, а по жетону, который ты показываешь снова и снова.

Java-аналогия здесь прямая: `Set-Cookie` в ответе сервера, потом `Cookie` в следующем запросе, работа с сессией, Spring Security и весь привычный мир аутентификации.

### Почему cookies отличаются от storage по смыслу

Главная сила cookie не в том, что она “лежит у пользователя”, а в том, что браузер умеет автоматически прикладывать её к подходящим HTTP-запросам. Именно поэтому cookie так естественно подходят для серверной сессии.

Если `localStorage` — это блокнот в столе пользователя, то cookie — это пропуск, который охрана смотрит при каждом входе.

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [w3schools: JavaScript Cookies](https://www.w3schools.com/js/js_cookies.asp)


### Как сервер задаёт cookie

```http
Set-Cookie: SESSION=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
```

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [w3schools: JavaScript Cookies](https://www.w3schools.com/js/js_cookies.asp)


### Разбор по шагам

1. `SESSION=abc123` — имя и значение.
2. `HttpOnly` запрещает чтение cookie из JavaScript.
3. `Secure` разрешает отправку только по HTTPS.
4. `SameSite=Lax` ограничивает межсайтовые сценарии.
5. `Path=/` задаёт область действия по пути.

### Почему `HttpOnly` так важна

Это один из самых полезных защитных флагов. Если cookie `HttpOnly`, JavaScript не может украсть её через `document.cookie`. Это не магическая защита от всего, но очень сильное снижение риска при XSS.

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Почему `Secure` и `SameSite` тоже важны

`Secure` не даёт чувствительной cookie гулять по незащищённому HTTP. `SameSite` управляет тем, когда cookie участвует в cross-site запросах, и помогает в защите от CSRF. То есть мир cookies — это не только удобство, но и очень плотная связь с безопасностью.

| SameSite | Поведение | Когда использовать |
|----------|-----------|-------------------|
| `Strict` | Cookie не отправляется при cross-site навигации | Критичные действия (перевод денег) |
| `Lax` (default) | Не отправляется в POST/iframe cross-site, но отправляется при навигации по ссылке | Большинство случаев |
| `None` | Всегда отправляется (ТРЕБУЕТ `Secure`) | Виджеты, cross-site API |

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)


### Работа через `document.cookie`

```javascript
document.cookie = 'theme=dark; Path=/';
console.log(document.cookie);
```

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)


### Разбор по шагам

1. Создаём обычную JS-доступную cookie.
2. `Path=/` делает её доступной на всём сайте.
3. `document.cookie` возвращает строку со всеми доступными JS cookies.
4. Поэтому использовать cookies как удобный мини-storage не слишком приятно — API строковый и ограниченный.

### Story mode: когда cookie, когда storage

Тема сайта? Обычно `localStorage`. Серверная сессия? Обычно cookie. Тяжёлый клиентский кэш? Ни то ни другое, смотри на IndexedDB. Это не соревнование “что лучше”, это выбор инструмента по роли.

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [w3schools: JavaScript Cookies](https://www.w3schools.com/js/js_cookies.asp)


### Связь с CORS и credentials

Если фронт и API находятся на разных origin и ты хочешь, чтобы браузер отправлял cookies, нужно помнить весь комплект из раздела 9.3:

- на клиенте `credentials: 'include'`;
- на сервере `Access-Control-Allow-Credentials: true`;
- `Access-Control-Allow-Origin` должен быть конкретным, не `*`.

Именно здесь очень хорошо видно, что CORS и cookies — это не две отдельные темы, а один связанный разговор про доверие и безопасность.

Типичная история ошибки: разработчик честно включил `credentials: 'include'`, но не проверил серверную CORS-конфигурацию и атрибуты cookie. В итоге “сессия не работает”, хотя проблема вообще не в самом `fetch`.

📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [javascript.info: Fetch: Cross-Origin Requests (#credentials)](https://learn.javascript.ru/fetch-crossorigin#credentials)
- [MDN: Request.credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)


📚 **Источники:**
- [javascript.info: Cookies](https://learn.javascript.ru/cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [w3schools: JavaScript Cookies](https://www.w3schools.com/js/js_cookies.asp)


### ✅ Проверь себя

**1. Какой флаг делает cookie недоступной для JavaScript?**  
A) `Secure`  
B) `HttpOnly`  
C) `Path`  
D) `Max-Age`

<details><summary>Ответ</summary>B) Это основной защитный флаг против чтения cookie из JS.</details>

**2. Что делает `Secure`?**  
A) Шифрует JSON  
B) Разрешает отправку cookie только по HTTPS  
C) Делает cookie вечной  
D) Выключает CORS

<details><summary>Ответ</summary>B) Это транспортное ограничение для безопасности.</details>

**3. Что отличает cookies от `localStorage` концептуально?**  
A) Cookies могут автоматически отправляться на сервер  
B) Cookies умеют хранить гигабайты файлов  
C) Cookies создают WebSocket  
D) Cookies всегда безопаснее без настроек

<details><summary>Ответ</summary>A) Это главное поведенческое различие.</details>

## 9.10. IndexedDB — обзор

Когда данных на клиенте становится заметно больше, чем пара флагов и небольшой кэш, на сцену выходит IndexedDB. Если `localStorage` — это шкафчик, то **IndexedDB** — уже склад с полками и каталогом. Java-аналогия очень простая: локальная мини-БД внутри браузера.

Она нужна, когда данных много, нужен офлайн, коллекции, индексы, более серьёзный кэш или структурированное хранение. Для темы сайта это перебор, для богатого офлайн-приложения — вполне естественный выбор.

```javascript
const request = indexedDB.open('task-manager-db', 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  if (!db.objectStoreNames.contains('tasks')) {
    db.createObjectStore('tasks', { keyPath: 'id', autoIncrement: true });
  }
};
```

### Разбор по шагам

1. Открываем или создаём базу с именем `task-manager-db`.
2. `1` — версия схемы.
3. В `onupgradeneeded` создаём нужные object stores при первом запуске или смене версии схемы.
4. Дальше можно работать с записями.
5. На практике часто используют обёртки вроде Dexie, потому что нативный API не самый уютный.

Типичная ошибка — тащить IndexedDB ради одной галочки `darkMode=true`. Зеркальная ошибка — пытаться запихнуть серьёзный офлайн-кэш в `localStorage`. Смотри на масштаб задачи.

📚 **Источники:**
- [javascript.info: IndexedDB](https://learn.javascript.ru/indexeddb)
- [MDN: IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)


### ✅ Проверь себя

**1. Когда IndexedDB уместнее `localStorage`?**  
A) Когда данных много и нужна структура  
B) Когда нужен только один флаг темы  
C) Когда надо решить CORS  
D) Когда нужен WebSocket

<details><summary>Ответ</summary>A) Именно для этого она и существует.</details>

**2. Какой образ лучше всего подходит IndexedDB?**  
A) Мини-база данных в браузере  
B) Вид cookie  
C) HTTP-заголовок  
D) Таймер

<details><summary>Ответ</summary>A) Это самый полезный mental model.</details>

## 9.11. Service Workers и кэширование

Service Worker удобно представлять как умного посредника между страницей и сетью. Консьерж, диспетчер, вахтёр — любая из этих метафор работает. Он видит запрос и решает: отдать ответ из кэша, сходить в сеть, обновить кэш или скомбинировать стратегии.

Смысл кэширования тут очень житейский: хранить копию ресурса поближе, чтобы не бегать каждый раз наружу. В Java это похоже на локальный кэш поверх медленного внешнего сервиса.

```javascript
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(cached => cached || fetch(event.request))
  );
});
```

### Разбор по шагам

1. Service Worker слушает событие `fetch`.
2. Для каждого запроса сначала ищет ответ в кэше.
3. Если находит — отдаёт его.
4. Если нет — идёт в сеть через обычный `fetch`.

Сила Service Worker в офлайне, скорости и контроле над стратегией доставки ресурсов. Но это не “волшебная папка”, а логика, которую нужно поддерживать: версионировать кэши, обновлять ресурсы и не кормить пользователя бесконечно устаревшими данными.

📚 **Источники:**
- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [MDN: Cache](https://developer.mozilla.org/en-US/docs/Web/API/Cache)
- [MDN: CacheStorage](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage)
- [Service Workers spec](https://w3c.github.io/ServiceWorker/)


### ✅ Проверь себя

**1. Кто такой Service Worker по роли?**  
A) Сетевой посредник между страницей и сетью  
B) Вид `localStorage`  
C) CSS-компилятор  
D) SQL-сервер

<details><summary>Ответ</summary>A) Это самый точный высокий уровень.</details>

**2. Что такое кэширование здесь?**  
A) Повторная отправка того же запроса  
B) Сохранение копии ресурса для быстрого или офлайн-доступа  
C) Автоматическая отмена CORS  
D) Шифрование cookies

<details><summary>Ответ</summary>B) Именно это делает кэш полезным.</details>

## 9.12. Связь с Angular: HttpClient и interceptors

Angular `HttpClient` помогает увидеть, как браузерный HTTP превращается из россыпи вызовов в организованный сервисный слой. Если `fetch` — это хороший базовый инструмент, то `HttpClient` — уже удобная стойка обслуживания внутри приложения.

Для Java-разработчика тут особенно понятна аналогия со Spring: ты не хочешь раскидывать низкоуровневые HTTP-детали по всему коду, ты хочешь централизовать правила, заголовки, обработку ошибок и расширения.

### Почему это не просто “ещё один `fetch`”

Angular возвращает `Observable`, а не `Promise`. Это другой стиль композиции, ближе к Reactor/RxJava-мышлению: подписки, операторы, пайплайны.

📚 **Источники:**
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular API: HttpClient](https://angular.dev/api/common/http/HttpClient)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [Angular docs: Interceptors](https://angular.dev/guide/http/interceptors)


### Зачем нужны interceptors

Interceptor — это проходная, через которую идут все HTTP-запросы и ответы. Здесь удобно:

- добавлять `Authorization`;
- включать глобальный спиннер;
- логировать;
- централизованно ловить `401`;
- делать retry в нужных местах.

```typescript
// Angular 17+ — функциональный interceptor
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  const authReq = token
    ? req.clone({
        setHeaders: { Authorization: `Bearer ${token}` }
      })
    : req;
  return next(authReq);
};

// Регистрация в provideHttpClient:
provideHttpClient(withInterceptors([authInterceptor]))
```

Class-based interceptors (`implements HttpInterceptor`) — это старый, но по-прежнему поддерживаемый стиль. Функциональные interceptors (`HttpInterceptorFn`) — рекомендуемый современный подход, но старый стиль тоже полезно уметь читать:

```typescript
// 🏚️ Старый стиль, но всё ещё поддерживается
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private auth: AuthService) {}
  
  intercept(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    const token = this.auth.getToken();
    if (token) {
      req = req.clone({
        setHeaders: { Authorization: `Bearer ${token}` }
      });
    }
    return next.handle(req);
  }
}
```

📚 **Источники:**
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular API: HttpClient](https://angular.dev/api/common/http/HttpClient)
- [Angular docs: Interceptors](https://angular.dev/guide/http/interceptors)
- [Angular API: HttpInterceptor](https://angular.dev/api/common/http/HttpInterceptor)


### Разбор по шагам

1. Interceptor получает исходный запрос.
2. Запрос immutable, поэтому нужен `clone(...)`.
3. Через `setHeaders` добавляем `Authorization`.
4. Передаём модифицированный запрос дальше.
5. Таким образом общая логика не размазывается по компонентам.

И очень важное напоминание: Angular `HttpClient` не отменяет CORS, cookies, Same-Origin Policy и preflight. Он делает код удобнее, но живёт внутри тех же браузерных правил. Это та же реальность, только с лучшей архитектурной обёрткой.

📚 **Источники:**
- [Angular docs: HTTP Client](https://angular.dev/guide/http)
- [Angular API: HttpClient](https://angular.dev/api/common/http/HttpClient)
- [Angular docs: Interceptors](https://angular.dev/guide/http/interceptors)
- [Angular API: HttpInterceptor](https://angular.dev/api/common/http/HttpInterceptor)


### ✅ Проверь себя

**1. Что делает interceptor?**  
A) Централизованно перехватывает HTTP-запросы и ответы  
B) Создаёт IndexedDB  
C) Выключает CORS  
D) Хранит файлы

<details><summary>Ответ</summary>A) Это точка для общей сетевой логики.</details>

**2. Чем `HttpClient` полезен по сравнению с голым `fetch`?**  
A) Даёт более высокий уровень абстракции и расширения  
B) Отменяет необходимость знать HTTP  
C) Работает без браузерных ограничений  
D) Удаляет статус-коды

<details><summary>Ответ</summary>A) Это его главная практическая ценность.</details>

## 📝 Квиз по модулю

**1. Что лучше всего описывает `fetch`?**  
A) Legacy API на событиях  
B) Promise-based API для HTTP-запросов  
C) Только API для WebSocket  
D) Только Node.js API

<details><summary>Ответ</summary>B) Это современный стандартный браузерный API для HTTP.</details>

**2. Что такое Same-Origin Policy?**  
A) Браузерное ограничение на чтение кросс-origin ответов  
B) Тип cookie  
C) Вид JSON  
D) Метод HTTP

<details><summary>Ответ</summary>A) Это фундамент браузерной безопасности.</details>

**3. Что такое preflight?**  
A) Предварительный `OPTIONS`-запрос  
B) Парсинг JSON  
C) Отмена запроса  
D) Очистка `localStorage`

<details><summary>Ответ</summary>A) Браузер заранее проверяет разрешения.</details>

**4. Что лучше использовать для длительного локального UI-кэша?**  
A) `localStorage`  
B) `sessionStorage`  
C) `AbortController`  
D) `EventSource`

<details><summary>Ответ</summary>A) Для такого сценария `localStorage` обычно удобнее.</details>

**5. Что делает `HttpOnly`?**  
A) Запрещает JS читать cookie  
B) Шифрует весь сайт  
C) Делает `fetch` синхронным  
D) Отправляет cookie только по WebSocket

<details><summary>Ответ</summary>A) Это один из ключевых защитных атрибутов cookie.</details>

**6. Когда особенно уместен WebSocket?**  
A) Для real-time двустороннего обмена  
B) Для темы сайта  
C) Для отправки формы  
D) Для локальной БД

<details><summary>Ответ</summary>A) Чаты и live-обновления — типичные примеры.</details>

**7. Что верно про `sessionStorage`?**  
A) Он живёт до закрытия вкладки  
B) Он автоматически уходит на сервер  
C) Он безопасен для паролей  
D) Он заменяет cookies

<details><summary>Ответ</summary>A) Это его основное отличие от `localStorage`.</details>

**8. Что делает Service Worker?**  
A) Выступает посредником для запросов и кэша  
B) Является видом XHR  
C) Хранит только cookies  
D) Отменяет CORS

<details><summary>Ответ</summary>A) Это его ключевая роль в PWA-модели.</details>

**9. Что лучше описывает IndexedDB?**  
A) Мини-БД в браузере  
B) Новый вид cookie  
C) Заголовок HTTP  
D) Таймер

<details><summary>Ответ</summary>A) Именно так её и стоит представлять.</details>

---

## 🏋️ Мини-упражнения

### Упражнение 1

Напиши функцию `loadTasks()`, которая делает `fetch('/api/tasks')`, проверяет `response.ok` и при ошибке бросает `Error` с текстом вида `HTTP 404`. Потом перепиши тот же сценарий через `XMLHttpRequest` и сравни, какой вариант легче читать без комментариев.

Подсказка ментора: не спеши писать сразу код. Сначала вслух проговори поток выполнения: “получаю `Response`, проверяю статус, потом читаю JSON”. Если ты умеешь проговорить это словами, код почти сам ложится на место.

### Упражнение 2

Сделай форму создания задачи с полем файла. Собери данные через `FormData`, отправь их на mock API и добавь кнопку отмены через `AbortController`. Отдельно обработай `AbortError`, чтобы в интерфейсе было видно: это не падение сети, а сознательная отмена.

Подсказка ментора: здесь специально пересекаются разделы 9.4 и 9.5. Это не случайно. В реальных приложениях темы не живут изолированно.

### Упражнение 3

Сохрани последний успешный список задач в `localStorage`. При старте страницы сначала попробуй сходить в API, а если сеть недоступна — покажи кэшированную копию. Потом сам сформулируй вслух, почему это удобно для UX, но не делает `localStorage` заменой базе данных.

Подсказка ментора: это отличный мост между разделами про `fetch`, ошибки сети и клиентское хранилище. Если сможешь красиво объяснить этот сценарий, ты уже думаешь как инженер, а не как человек, который просто выучил API.

---

## 🚀 Задание по проекту

Подключи pet-project `task-manager` к mock API. Сделай модуль `api.js` с функциями `getTasks`, `createTask`, `toggleTask`, `deleteTask`, которые внутри используют `fetch`, проверяют `response.ok` и возвращают JSON. Если сервер недоступен или отвечает ошибкой, превращай это в понятное сообщение для UI, а не просто в `console.error`.

Добавь fallback через `localStorage`. После успешной загрузки сохраняй снимок задач в `task-manager-cache`. При следующем старте сначала пытайся сходить в API, но если сеть упала — покажи последнюю сохранённую копию и текст, что пользователь сейчас работает с офлайн-данными. Так ты свяжешь `fetch`, обработку ошибок и клиентское хранилище в одном реальном сценарии.

Если хочешь усилить проект, вынеси фронт и mock API на разные порты, специально столкнись с CORS и настрой прокси или серверные заголовки. Это очень полезная практика: после неё CORS перестаёт быть теорией и превращается в понятную инженерную задачу.

Ещё один челлендж для смелых: подумай, где в твоём проекте действительно уместен WebSocket, а где его добавление будет просто красивой, но ненужной сложностью. Если сможешь аргументированно отказаться от модной технологии — это тоже признак зрелости.

---

## Контрольные вопросы

### 1. Чем `fetch` отличается от `XMLHttpRequest`?

`fetch` современнее, чище читается и работает через промисы, поэтому хорошо сочетается с `async/await`. XHR построен на событиях и требует больше ручной работы: проверять статус, читать `responseText`, держать в голове этапы. Оба API делают HTTP-запросы, но `fetch` обычно проще сопровождать. Если хочется образно: `fetch` — это аккуратная служба доставки, а XHR — старый факс, который всё ещё умеет работать, но просит больше ручного участия.

### 2. Почему `fetch` не бросает ошибку на `404` и `500`?

Потому что запрос доехал, сервер ответил, транспорт состоялся. Для `fetch` это успешный сетевой обмен, хотя бизнес-результат плохой. Поэтому HTTP-ошибки ты определяешь сам через `response.ok` или `response.status`. В `catch` чаще попадают сетевые проблемы, отмена запроса или браузерная блокировка чтения ответа.

### 3. Что такое Same-Origin Policy?

Это базовое правило браузера: код страницы не может свободно читать ответы другого origin без разрешения сервера. Origin — это комбинация протокола, хоста и порта. Политика защищает пользователя от межсайтового чтения приватных данных. CORS не отменяет SOP, а даёт управляемый механизм исключений.

### 4. Что такое preflight?

Это предварительный `OPTIONS`-запрос, которым браузер спрашивает у сервера, разрешён ли основной кросс-origin запрос с заданным методом и заголовками. Он нужен для непростых запросов и помогает браузеру не выполнять потенциально опасные сценарии вслепую. Если preflight не проходит, до основного запроса дело может вообще не дойти.

### 5. Как правильно чинить CORS-ошибку?

На сервере или в прокси. Нужно настроить корректные `Access-Control-Allow-*` заголовки, а при работе с cookies ещё и правильно разрешить credentials. На фронте нельзя самому выписать себе разрешение. `mode: 'no-cors'` обычно не решает задачу чтения ответа.

### 6. Чем `localStorage`, `sessionStorage` и cookies отличаются?

`localStorage` живёт долго и не отправляется на сервер автоматически. `sessionStorage` живёт до закрытия вкладки и тоже не отправляется автоматически. Cookies могут уходить на сервер вместе с запросами и имеют атрибуты безопасности вроде `HttpOnly`, `Secure`, `SameSite`. Storage хорош для локального состояния интерфейса и простого кэша, cookies — для серверной сессии и аутентификационных сценариев.

### 7. Почему `HttpOnly` cookie важна?

Потому что JavaScript не может её прочитать. Это уменьшает риск кражи сессионных данных через XSS. Поэтому для серверной аутентификации `HttpOnly` cookies часто безопаснее, чем хранение токенов в `localStorage`, если сама архитектура проекта с ними совместима.

### 8. Когда выбирать IndexedDB?

Когда данных уже слишком много для `localStorage`, нужна структура, индексы, офлайн-режим или более серьёзный клиентский кэш. `localStorage` — шкафчик, IndexedDB — склад с каталогом. Для пары строковых флагов она избыточна, для офлайн-приложения — может быть очень уместна.

### 9. Что делает `AbortController` и чего не гарантирует?

Он отменяет интерес клиента к результату и прерывает поддерживаемую операцию вроде `fetch`. Но он не гарантирует, что сервер уже бросил свою внутреннюю работу. Клиентская отмена и серверный откат — не одно и то же.

### 10. Чем WebSocket отличается от SSE?

WebSocket даёт постоянный двусторонний канал, где говорить могут обе стороны. SSE — это обычно однонаправленный поток событий от сервера к клиенту. Если нужна “телефонная линия” — WebSocket. Если нужна “радиостанция” или “живое табло” — иногда хватает SSE.

### 11. Что делает Service Worker?

Он стоит между страницей и сетью, может перехватывать запросы, отдавать ответы из кэша и помогать офлайн-режиму. Это активный посредник, а не просто место хранения файлов. Его сила — в управлении стратегией доставки ресурсов.

### 12. Как Angular `HttpClient` связан с этим модулем?

Он даёт более высокий уровень абстракции над браузерным HTTP, но не отменяет сам HTTP и правила браузера. CORS, cookies, Same-Origin Policy и безопасность никуда не исчезают. Просто работать с ними становится удобнее через сервисы, interceptors и более организованную клиентскую архитектуру.

---

## Финальное резюме модуля

Если собрать модуль в одну картину, то получится так. `fetch` — это главный современный способ делать HTTP-запросы в браузере, но понимать его нужно вместе со статусами, телом ответа, JSON, обработкой ошибок, потоками и отменой. CORS — это не каприз фронтенда, а следствие Same-Origin Policy и браузерной безопасности. Как только ты мысленно проходишь preflight шаг за шагом, тема перестаёт быть магией.

`localStorage`, `sessionStorage` и cookies — это не просто три места “куда что-то положить”. Это три разных истории с разным сроком жизни, разным уровнем связи с сервером и разными рисками. А WebSocket, SSE, IndexedDB и Service Worker нужны не всегда, но именно они делают браузерное приложение взрослым: real-time, офлайн, кэш, потоковые данные и более богатый UX становятся осознанными инструментами, а не случайными словами из документации.

Если после этого модуля у тебя стало меньше чувства “браузер ведёт себя странно” и больше чувства “ага, тут есть строгие правила, и я их понимаю”, значит мы с тобой пришли в правильную точку.
