# Домашнє завдання 3: Веб-протоколи та прикладний рівень

Команди виконано в Docker-контейнері `ubuntu:22.04`. Система була «чиста», тому потрібні утиліти я встановила вручну:

```bash
apt-get update
apt-get install -y curl openssl ca-certificates nodejs npm
```

## Завдання 1. Аналіз HTTP-заголовків та статус-кодів

Команда:

```bash
curl -I http://github.com
```

Відповідь:

```
HTTP/1.1 301 Moved Permanently
Content-Length: 0
Location: https://github.com/
```

- Статус-код: 301 Moved Permanently, тобто ресурс постійно переїхав на іншу адресу.
- Location: `https://github.com/`. Запит прийшов по незахищеному HTTP (порт 80), а GitHub працює тільки через HTTPS, тому сервер одразу перенаправляє на захищену адресу.
- Протокол: HTTP/1.1. Після переходу на HTTPS сервер відповідає вже по HTTP/2, це видно у відповіді в завданні 4.

## Завдання 2. Робота з методами HTTP

Тестовий API я розгорнула локально в контейнері за допомогою json-server:

```bash
mkdir /api && cd /api
echo '{"movies":[{"id":1,"title":"Inception","year":2010}]}' > db.json
npm i json-server@0.17.4
npx json-server db.json --port 3000 &
```

Команда для створення ресурсу (POST):

```bash
curl -i -X POST -H "Content-Type: application/json" -d '{"title":"Interstellar","year":2014}' http://localhost:3000/movies
```

Відповідь:

```
HTTP/1.1 201 Created
X-Powered-By: Express
Vary: Origin, X-HTTP-Method-Override, Accept-Encoding
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Expires: -1
Access-Control-Expose-Headers: Location
Location: http://localhost:3000/movies/2
X-Content-Type-Options: nosniff
Content-Type: application/json; charset=utf-8
Content-Length: 56
ETag: W/"38-YkUgfqH3uGpU9Xk1jSul9LuCNKE"
Date: Tue, 10 Nov 2026 22:27:46 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{
  "title": "Interstellar",
  "year": 2014,
  "id": 2
}
```

Перевірка, що ресурс створено (GET):

```bash
curl -i "http://localhost:3000/movies?year=2014"
```

Відповідь:

```
HTTP/1.1 200 OK
X-Powered-By: Express
Vary: Origin, Accept-Encoding
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Expires: -1
X-Content-Type-Options: nosniff
Content-Type: application/json; charset=utf-8
Content-Length: 70
ETag: W/"46-JVnEonDZJu3Hfa503WuoMl3nloU"
Date: Tue, 10 Nov 2026 22:27:46 GMT
Connection: keep-alive
Keep-Alive: timeout=5

[
  {
    "title": "Interstellar",
    "year": 2014,
    "id": 2
  }
]
```

- Статус-код успішного створення: 201 Created. У заголовку `Location` сервер повернув адресу нового ресурсу: `/movies/2`.
- Де передаються дані: GET передає параметри в URL (`?year=2014`), тому вони видні в адресному рядку, історії браузера та логах. POST передає дані в тілі запиту.
- Кешування: відповіді на GET браузери та проксі можуть кешувати за замовчуванням. Відповіді на POST зазвичай не кешуються, бо цей запит змінює дані на сервері.
- Призначення: GET тільки отримує дані і є безпечним та ідемпотентним. POST створює ресурс, і кожен повторний запит створить ще один запис.

## Завдання 3. Перевірка TLS-сертифіката

Я обрала сайт github.com. Вивід команди довгий, тому залишила потрібні рядки за допомогою `grep`:

```bash
echo | openssl s_client -connect github.com:443 2>&1 | grep -E "^issuer=|^subject=|^New,|^Verification|Verify return code"
```

Відповідь:

```
subject=CN = github.com
issuer=C = GB, O = Sectigo Limited, CN = Sectigo Public Server Authentication CA DV E36
Verification: OK
New, TLSv1.3, Cipher is TLS_AES_128_GCM_SHA256
Verify return code: 0 (ok)
```

- Issuer: Sectigo Limited, центр сертифікації Sectigo Public Server Authentication CA DV E36.
- Протокол: TLSv1.3.
- Cipher Suite: TLS_AES_128_GCM_SHA256.

`Verification: OK` означає, що ланцюжок сертифікатів успішно перевірено.

## Завдання 4. Аналіз безпеки Cookies

Команда:

```bash
curl -v -o /dev/null https://github.com 2>&1 | grep -iE '^< (HTTP|set-cookie)'
```

Відповідь (значення `_gh_sess` дуже довге, тому я його скоротила):

```
< HTTP/2 200
< set-cookie: _gh_sess=...; path=/; HttpOnly; secure; SameSite=Lax
< set-cookie: _octo=GH1.1.2001768220.1794349662; expires=Wed, 10 Nov 2027 22:27:42 GMT; domain=.github.com; path=/; secure; SameSite=Lax
< set-cookie: logged_in=no; expires=Wed, 10 Nov 2027 22:27:42 GMT; domain=.github.com; path=/; HttpOnly; secure; SameSite=Lax
```

| Cookie | HttpOnly | Secure | SameSite |
| --- | --- | --- | --- |
| `_gh_sess` | так | так | Lax |
| `_octo` | ні | так | Lax |
| `logged_in` | так | так | Lax |

- HttpOnly захищає від крадіжки cookie через XSS (Cross-Site Scripting). Якщо зловмиснику вдасться виконати свій JavaScript на сторінці, він не зможе прочитати таку cookie через `document.cookie`, а отже не вкраде сесію. Саму XSS-вразливість цей атрибут не усуває.
- Secure дозволяє браузеру надсилати cookie тільки по HTTPS. Вона не піде у відкритому вигляді по HTTP, тому її не вдасться перехопити в мережі, наприклад у відкритому Wi-Fi (атака Man-in-the-Middle).

## Завдання 5. Архітектурне моделювання

**Сценарій А: оновлення графіків цін у браузері. Обираю WebSocket.**

- Після одного рукостискання відкрите двостороннє з'єднання залишається активним, тому сервер надсилає нову ціну одразу, без окремого запиту від клієнта.
- Накладні витрати на кожне повідомлення малі. При оновленнях щосекунди HTTP-polling створював би багато зайвих запитів із повними заголовками і додавав би затримку.
- WebSocket підтримується браузерами напряму.
- Двосторонність корисна: клієнт може підписуватися на потрібні акції та відписуватися від них у тому самому з'єднанні.

SSE теж підходить для однонапрямленого потоку цін, але підписки довелося б надсилати окремими HTTP-запитами, тому для біржі зручніший WebSocket.

**Сценарій Б: взаємодія між мікросервісами (білінг <-> облік). Обираю gRPC.**

- Працює поверх HTTP/2: одне з'єднання обслуговує багато паралельних запитів (мультиплексування).
- Protobuf серіалізує дані в компактний бінарний формат, тому повідомлення менші й обробляються швидше, ніж JSON у REST. Це дає мінімальну затримку.
- Контракт описаний у `.proto`-файлі і строго типізований. Для фінансових даних це важливо, бо сервіси не зможуть обмінюватися даними неправильного формату.
- Є вбудовані дедлайни та стрімінг. Обмеження gRPC у браузері тут не заважає, бо зв'язок відбувається лише між сервісами всередині системи.
