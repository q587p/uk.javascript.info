
# Отримання

JavaScript може надсилати мережеві запити на сервер і завантажувати нову інформацію, коли це потрібно.

Наприклад, ми можемо використовувати мережевий запит для:

- Зробити замовлення,
- Завантажити інформацію про користувача,
- Отримувати останні оновлення від сервера,
- …тощо.

…І все це без перезавантаження сторінки!

Існує зонтичний термін “AJAX” (скорочення <b>A</b>synchronous <b>J</b>avaScript <b>A</b>nd <b>X</b>ML) для мережевих запитів від JavaScript. Однак нам не обов’язково використовувати XML: термін походить із давніх часів, тому там це слово є. Можливо, ви вже чули цей термін.

Існує декілька способів надіслати мережевий запит та отримати інформацію з сервера.

Метод `fetch ()` є сучасним та універсальним, тому ми почнемо з нього. Він не підтримується старими браузерами (можна поліфільнути), але дуже добре підтримується новими.

Основний синтаксис:

```js
let promise = fetch(url, [options])
```

- **`url`**  - URL-адреса для доступу.
- **`options`** - необов'язкові параметри: метод, заголовки тощо.

Без `options` — це простий запит GET, що завантажує вміст `url`.

Браузер одразу запускає запит і повертає обіцянку, яку повинен використовувати код, що викликається для отримання результату.

Отримання відповіді — це, як правило, двоетапний процес.

**По-перше, __обіцянка__, повернута функцією `fetch`, вирішується за допомогою об’єкта вбудованого класу [Response] (https://fetch.spec.whatwg.org/#response-class), як тільки сервер відповість заголовками.**

На цьому етапі ми можемо перевірити статус HTTP, щоб побачити його успішність чи ні, перевірити заголовки, але ще не маємо тіла.

Обіцянка відхиляється, якщо `fetch` не зміг зробити HTTP-запит, наприклад проблеми з мережею, або такого сайту немає. Ненормальні статуси HTTP, такі як 404 або 500, не викликають помилок.

Ми можемо побачити статус HTTP у властивостях відповіді:

- **`status`** - код стану HTTP, напр. 200.
- **`ok`** - логічне значення, `true` (так), якщо код стану HTTP в діапазоні 200-299.

Наприклад:

```js
let response = await fetch(url);

if (response.ok) { // якщо статус HTTP в межах 200-299
  // отримуємо тіло відповіді (метод, пояснений нижче)
  let json = await response.json();
} else {
  alert("Помилка HTTP: " + response.status);
}
```

**По-друге, щоб отримати тіло відповіді, нам потрібно використовувати додатковий виклик методу.**

`Response` надає безліч методів, заснованих на обіцянках, для доступу до тіла в різних форматах:

- **`response.text()`** - прочитати відповідь і повернути як текст,
- **`response.json()`** - розібрати відповідь як JSON,
- **`response.formData()`** - повернути відповідь як об’єкт `FormData` (пояснено в [наступному розділі](info:formdata)),
- **`response.blob()`** - повернути відповідь як [Блоб](info:blob) (двійкові дані з типом),
- **`response.arrayBuffer()`** - повернути відповідь як [Двійковий масив](info:arraybuffer-binary-arrays) (низькорівневе представлення двійкових даних),
- крім того, `response.body` є об’єктом [ReadableStream](https://streams.spec.whatwg.org/#rs-class) — він дозволяє вам читати тіло по частинах, як ми побачимо пізніше на прикладі.

Наприклад, давайте отримаємо JSON-об'єкт із останніми комітами від GitHub:

```js run async
let url = 'https://api.github.com/repos/javascript-tutorial/uk.javascript.info/commits';
let response = await fetch(url);

*!*
let commits = await response.json(); // читаємо тіло відповіді та розбираємо як JSON
*/!*

alert(commits[0].author.login);
```

Або те саме без `await`, використовуючи синтаксис чистих обіцянок:

```js run
fetch('https://api.github.com/repos/javascript-tutorial/uk.javascript.info/commits')
  .then(response => response.json())
  .then(commits => alert(commits[0].author.login));
```

Щоб отримати текст відповіді, `await response.text ()` замість `.json ()`:

```js run async
let response = await fetch('https://api.github.com/repos/javascript-tutorial/uk.javascript.info/commits');

let text = await response.text(); // читаємо тіло відповіді як текст

alert(text.slice(0, 80) + '...');
```

Як показовий приклад для читання у двійковому форматі, давайте дістанемо та покажемо зображення логотипу із [специфікації “fetch”] (https://fetch.spec.whatwg.org) (див. розділ [Блоб](info:blob) для подробиць щодо операцій на `Blob`):

```js async run
let response = await fetch('/article/fetch/logo-fetch.svg');

*!*
let blob = await response.blob(); // завантажити як об’єкт Blob
*/!*

// create <img> for it
let img = document.createElement('img');
img.style = 'position:fixed;top:10px;left:10px;width:100px';
document.body.append(img);

// show it
img.src = URL.createObjectURL(blob);

setTimeout(() => { // приховати через три секунди
  img.remove();
  URL.revokeObjectURL(img.src);
}, 3000);
```

````warn
Ми можемо вибрати лише один метод читання тіла.

Якщо ми вже отримали відповідь з `response.text()`, тоді `response.json()` не буде працювати, оскільки вміст основного тексту вже оброблений

```js
let text = await response.text(); // споживане тіло відповіді
let parsed = await response.json(); // не вдалося (вже спожито)
```
````

## Заголовки відповідей

The response headers are available in a Map-like headers object in `response.headers`.

It's not exactly a Map, but it has similar methods to get individual headers by name or iterate over them:

```js run async
let response = await fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits');

// get one header
alert(response.headers.get('Content-Type')); // application/json; charset=utf-8

// iterate over all headers
for (let [key, value] of response.headers) {
  alert(`${key} = ${value}`);
}
```

## Request headers

To set a request header in `fetch`, we can use the `headers` option. It has an object with outgoing headers, like this:

```js
let response = fetch(protectedUrl, {
  headers: {
    Authentication: 'secret'
  }
});
```

...But there's a list of [forbidden HTTP headers](https://fetch.spec.whatwg.org/#forbidden-header-name) that we can't set:

- `Accept-Charset`, `Accept-Encoding`
- `Access-Control-Request-Headers`
- `Access-Control-Request-Method`
- `Connection`
- `Content-Length`
- `Cookie`, `Cookie2`
- `Date`
- `DNT`
- `Expect`
- `Host`
- `Keep-Alive`
- `Origin`
- `Referer`
- `TE`
- `Trailer`
- `Transfer-Encoding`
- `Upgrade`
- `Via`
- `Proxy-*`
- `Sec-*`

These headers ensure proper and safe HTTP, so they are controlled exclusively by the browser.

## POST requests

To make a `POST` request, or a request with another method, we need to use `fetch` options:

- **`method`** -- HTTP-method, e.g. `POST`,
- **`body`** -- the request body, one of:
  - a string (e.g. JSON-encoded),
  - `FormData` object, to submit the data as `form/multipart`,
  - `Blob`/`BufferSource` to send binary data,
  - [URLSearchParams](info:url), to submit the data in `x-www-form-urlencoded` encoding, rarely used.

The JSON format is used most of the time.

For example, this code submits `user` object as JSON:

```js run async
let user = {
  name: 'John',
  surname: 'Smith'
};

*!*
let response = await fetch('/article/fetch/post/user', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json;charset=utf-8'
  },
  body: JSON.stringify(user)
});
*/!*

let result = await response.json();
alert(result.message);
```

Please note, if the request `body` is a string, then `Content-Type` header is set to `text/plain;charset=UTF-8` by default.

But, as we're going to send JSON, we use `headers` option to send `application/json` instead, the correct `Content-Type` for JSON-encoded data.

## Sending an image

We can also submit binary data with `fetch` using `Blob` or `BufferSource` objects.

In this example, there's a `<canvas>` where we can draw by moving a mouse over it. A click on the "submit" button sends the image to the server:

```html run autorun height="90"
<body style="margin:0">
  <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

  <input type="button" value="Submit" onclick="submit()">

  <script>
    canvasElem.onmousemove = function(e) {
      let ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };

    async function submit() {
      let blob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));
      let response = await fetch('/article/fetch/post/image', {
        method: 'POST',
        body: blob
      });

      // the server responds with confirmation and the image size
      let result = await response.json();
      alert(result.message);
    }

  </script>
</body>
```

Please note, here we don't set `Content-Type` header manually, because a `Blob` object has a built-in type (here `image/png`, as generated by `toBlob`). For `Blob` objects that type becomes the value of `Content-Type`.

The `submit()` function can be rewritten without `async/await` like this:

```js
function submit() {
  canvasElem.toBlob(function(blob) {        
    fetch('/article/fetch/post/image', {
      method: 'POST',
      body: blob
    })
      .then(response => response.json())
      .then(result => alert(JSON.stringify(result, null, 2)))
  }, 'image/png');
}
```

## Summary

A typical fetch request consists of two `await` calls:

```js
let response = await fetch(url, options); // resolves with response headers
let result = await response.json(); // read body as json
```

Or, without `await`:

```js
fetch(url, options)
  .then(response => response.json())
  .then(result => /* process result */)
```

Response properties:
- `response.status` -- HTTP code of the response,
- `response.ok` -- `true` is the status is 200-299.
- `response.headers` -- Map-like object with HTTP headers.

Methods to get response body:
- **`response.text()`** -- return the response as text,
- **`response.json()`** -- parse the response as JSON object,
- **`response.formData()`** -- return the response as `FormData` object (form/multipart encoding, see the next chapter),
- **`response.blob()`** -- return the response as [Blob](info:blob) (binary data with type),
- **`response.arrayBuffer()`** -- return the response as [ArrayBuffer](info:arraybuffer-binary-arrays) (low-level binary data),

Fetch options so far:
- `method` -- HTTP-method,
- `headers` -- an object with request headers (not any header is allowed),
- `body` -- the data to send (request body) as `string`, `FormData`, `BufferSource`, `Blob` or `UrlSearchParams` object.

In the next chapters we'll see more options and use cases of `fetch`.
