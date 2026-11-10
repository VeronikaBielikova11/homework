# Домашнє завдання 5: Діагностика та безпека мереж

Команди виконано в Docker-контейнері `ubuntu:22.04`. Система була «чиста», тому потрібні утиліти я встановила вручну:

```bash
apt-get update
apt-get install -y tcpdump curl sudo ca-certificates
```

## Завдання 1. Діагностика трафіку з tcpdump

Термінал 1, запуск перехоплення:

```bash
sudo tcpdump -i any port 80 -c 10
```

Термінал 2, запит під час роботи `tcpdump`:

```bash
curl -s -D - http://example.com
```

Відповідь у терміналі 2:

```
HTTP/1.1 200 OK
Date: Tue, 10 Nov 2026 22:37:14 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: cloudflare
Last-Modified: Sun, 08 Nov 2026 18:05:34 GMT
Allow: GET, HEAD
Accept-Ranges: bytes
Age: 1850
cf-cache-status: HIT
CF-RAY: a5888a66a84d5a89-VIE
alt-svc: h3=":443"; ma=86400

<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content="width=device-width,initial-scale=1"><title>Example Domain</title><style>html{color-scheme:light dark;background:light-dark(#eee,#222)}body{font:16px/1.6 system-ui,sans-serif;max-width:26em;margin:auto;padding:25vh 2em 2em;text-align:center}</style></head><body><p>This domain is for use in documentation examples without needing permission. This is not a service; avoid relying on it for testing and monitoring purposes.</p><script src=/s.js></script></body></html>
```

Відповідь у терміналі 1:

```
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
22:37:14.599914 eth0  Out IP 7bb4ea386db2.59024 > 104.20.23.154.80: Flags [S], seq 599557472, win 65495, options [mss 65495,sackOK,TS val 1608773759 ecr 0,nop,wscale 10], length 0
22:37:14.627355 eth0  In  IP 104.20.23.154.80 > 7bb4ea386db2.59024: Flags [S.], seq 769837655, ack 599557473, win 65408, options [mss 65495,sackOK,TS val 1461725113 ecr 1608773759,nop,wscale 7], length 0
22:37:14.627403 eth0  Out IP 7bb4ea386db2.59024 > 104.20.23.154.80: Flags [.], ack 1, win 64, options [nop,nop,TS val 1608773787 ecr 1461725113], length 0
22:37:14.627570 eth0  Out IP 7bb4ea386db2.59024 > 104.20.23.154.80: Flags [P.], seq 1:76, ack 1, win 64, options [nop,nop,TS val 1608773787 ecr 1461725113], length 75: HTTP: GET / HTTP/1.1
22:37:14.627801 eth0  In  IP 104.20.23.154.80 > 7bb4ea386db2.59024: Flags [.], ack 76, win 510, options [nop,nop,TS val 1461725114 ecr 1608773787], length 0
22:37:14.663652 eth0  In  IP 104.20.23.154.80 > 7bb4ea386db2.59024: Flags [P.], seq 1:937, ack 76, win 4096, options [nop,nop,TS val 1461725149 ecr 1608773787], length 936: HTTP: HTTP/1.1 200 OK
22:37:14.663735 eth0  Out IP 7bb4ea386db2.59024 > 104.20.23.154.80: Flags [.], ack 937, win 64, options [nop,nop,TS val 1608773823 ecr 1461725149], length 0
22:37:14.664016 eth0  Out IP 7bb4ea386db2.59024 > 104.20.23.154.80: Flags [F.], seq 76, ack 937, win 64, options [nop,nop,TS val 1608773823 ecr 1461725149], length 0
22:37:14.664456 eth0  In  IP 104.20.23.154.80 > 7bb4ea386db2.59024: Flags [.], ack 77, win 4095, options [nop,nop,TS val 1461725150 ecr 1608773823], length 0
22:37:14.697505 eth0  In  IP 104.20.23.154.80 > 7bb4ea386db2.59024: Flags [F.], seq 937, ack 77, win 4096, options [nop,nop,TS val 1461725183 ecr 1608773823], length 0
10 packets captured
11 packets received by filter
0 packets dropped by kernel
```

- Послідовність прапорців перших трьох пакетів: `[S]`, `[S.]`, `[.]`.
- TCP handshake відбувся успішно: клієнт надіслав SYN, сервер відповів SYN-ACK, клієнт підтвердив ACK. Після цього почалася передача даних.
- `[P.]` означає PSH + ACK. Прапорець Push каже отримувачу одразу передати дані застосунку, не чекаючи заповнення буфера, а крапка підтверджує отримані дані. Тут із `[P.]` пішли HTTP-запит `GET / HTTP/1.1` (75 байтів) і відповідь `HTTP/1.1 200 OK` (936 байтів).

## Завдання 2. Налаштування Reverse Proxy (Nginx)

```nginx
location /api {
    proxy_pass http://localhost:3000;
    proxy_set_header X-Real-IP $remote_addr;
}
```

**Навіщо потрібен `proxy_set_header X-Real-IP`.** З'єднання з бекендом встановлює Nginx, тому IP реального клієнта бекенд сам не дізнається. Ця директива додає до запиту заголовок `X-Real-IP` зі справжньою адресою клієнта (`$remote_addr`). Вона потрібна для логів, обмеження кількості запитів, блокування за IP та геолокації.

Без неї бекенд побачить як адресу клієнта IP Nginx. У цій конфігурації це `127.0.0.1`, тобто всі запити виглядатимуть так, ніби прийшли з одного локального джерела.

**Перевага безпеки `proxy_pass`.** Порт 3000 можна не відкривати назовні, наприклад слухати лише на `127.0.0.1` і закрити його у firewall. Назовні доступні тільки 80 та 443 порти Nginx, тому клієнт не може звернутися до застосунку напряму. Nginx стає єдиною точкою входу: тут можна налаштувати TLS, обмеження швидкості, фільтрацію запитів і доступ лише до потрібних шляхів, як `/api` у цьому прикладі. Також приховуються внутрішня адреса та порт застосунку.

## Завдання 3. Вибір алгоритму балансування

**Обираю IP Hash.**

Балансувальник рахує хеш від IP клієнта, тому запити того самого клієнта потрапляють на той самий сервер, де лежить його сесія. Цей спосіб закріплення називається sticky sessions.

**Проблема Round Robin.** Запити розподіляються по черзі, тому наступний запит користувача потрапить на інший сервер, де його сесії немає. Користувача викидатиме з акаунта, а стан, наприклад кошик, буде губитися. Least Connections має ту саму проблему, бо обирає сервер за навантаженням, а не за клієнтом.

IP Hash має й обмеження: користувачі за одним NAT потрапляють на один сервер, а при падінні сервера його сесії зникають. Тому в довгостроковій перспективі краще винести сесії у спільне сховище, наприклад Redis, і зробити застосунок stateless.

## Завдання 4. Мікросегментація в Kubernetes

```yaml
spec:
  podSelector:
    matchLabels:
      app: database
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
```

1. **До яких Pod застосовується правило.** До Pod з міткою `app: database` у тому namespace, де створено цю політику (`spec.podSelector`).

2. **Яким Pod дозволено доступ.** Лише Pod з міткою `app: backend` із того самого namespace, бо `namespaceSelector` не вказано. Увесь інший вхідний трафік до бази блокується, зокрема від фронтенду. Порти в правилі не вказані, тому бекенду дозволено звертатися до всіх портів бази.

3. **Як це обмежує Blast Radius.** Якщо зловмисник зламає Pod фронтенду, він не зможе напряму підключитися до бази, бо мережа відхилить таке з'єднання. Щоб дістатися даних, йому доведеться зламати ще й бекенд. Отже, наслідки злому обмежуються одним компонентом, а рух мережею всередині кластера (lateral movement) ускладнюється.

Політика працює лише тоді, коли мережевий плагін кластера (CNI) підтримує NetworkPolicy, наприклад Calico або Cilium.

## Завдання 5. Аналіз системи виявлення вторгнень (IDS)

Подія: `[Alert] ET EXPLOIT POSSIBLE SQL Injection (UNION SELECT)`.

**Різниця між IDS та IPS.** IDS працює пасивно: аналізує копію трафіку і створює сповіщення, а сам пакет доходить до застосунку. Реагувати на подію має адміністратор або інша система.

IPS стоїть безпосередньо на шляху трафіку (inline). Вона б заблокувала цей пакет (drop), записала подію в лог і за потреби розірвала б з'єднання, тому запит із `UNION SELECT` не дійшов би до застосунку. Ризик у тому, що при хибному спрацюванні IPS заблокує і легітимний запит.

**Ще один рівень захисту (Defense in Depth).** WAF (Web Application Firewall) на рівні edge, наприклад Cloudflare WAF або AWS WAF. Він аналізує HTTP-запити ще до входу в нашу мережу, розпізнає SQL Injection у параметрах та блокує такі запити на межі мережі.

Якщо атака все ж пройде далі, останнім рівнем лишається сам застосунок: параметризовані запити (prepared statements) не дають виконати введені дані як SQL.
