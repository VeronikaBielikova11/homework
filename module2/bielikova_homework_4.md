# Домашнє завдання 4: Інфраструктура та мережі в DevOps

Команди виконано в Docker-контейнері `ubuntu:22.04`, запущеному з прапорцем `--privileged`. Усередині нього я запустила окремий Docker, щоб працювати з `iptables` і контейнерами в чистому середовищі:

```bash
apt-get update
apt-get install -y docker.io iptables iproute2 curl jq sudo
nohup dockerd > /var/log/dockerd.log 2>&1 &
```

## Завдання 1. Аналіз таблиці маршрутизації

Команда:

```bash
ip route
```

Відповідь:

```
default via 172.17.0.1 dev eth0
172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.4
172.18.0.0/16 dev docker0 proto kernel scope link src 172.18.0.1 linkdown
```

- Default route: `default via 172.17.0.1 dev eth0`, тобто Default Gateway: 172.17.0.1.
- Пояснення: `default via` означає маршрут за замовчуванням. Якщо для адреси призначення немає точнішого маршруту в таблиці, пакет відправляється на шлюз 172.17.0.1, а він передає його далі. Адреси 172.17.0.0/16 та 172.18.0.0/16 доступні напряму, тому для них є окремі записи.
- Мережевий інтерфейс: eth0.

## Завдання 2. Дослідження ланцюгів Firewall

Команда:

```bash
sudo iptables -L -n
```

Відповідь:

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DOCKER-USER  all  --  0.0.0.0/0            0.0.0.0/0
DOCKER-FORWARD  all  --  0.0.0.0/0            0.0.0.0/0

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain DOCKER (1 references)
target     prot opt source               destination
DROP       all  --  0.0.0.0/0            0.0.0.0/0

Chain DOCKER-BRIDGE (1 references)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0

Chain DOCKER-CT (1 references)
target     prot opt source               destination
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED

Chain DOCKER-FORWARD (1 references)
target     prot opt source               destination
DOCKER-CT  all  --  0.0.0.0/0            0.0.0.0/0
DOCKER-INTERNAL  all  --  0.0.0.0/0            0.0.0.0/0
DOCKER-BRIDGE  all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0

Chain DOCKER-INTERNAL (1 references)
target     prot opt source               destination

Chain DOCKER-USER (1 references)
target     prot opt source               destination
```

- Три основні ланцюги таблиці filter: INPUT, FORWARD, OUTPUT. Ланцюги `DOCKER-*` створив сам Docker.
- Default Policy для INPUT: ACCEPT.
- INPUT обробляє пакети, адресовані самому хосту, наприклад підключення до його SSH-сервера.
- FORWARD обробляє транзитні пакети, які проходять через хост на іншу адресу. Саме через нього йде трафік до контейнерів, тому Docker додав свої правила в цей ланцюг.

## Завдання 3. Мережева ізоляція в Docker

Команди:

```bash
docker run -d --name test-net nginx
docker inspect test-net | jq '.[0].NetworkSettings'
```

Вивід `docker inspect` дуже довгий, тому за допомогою `jq` я залишила лише секцію NetworkSettings.

Відповідь:

```
ed61d7c7032fa8febb4b14b0ba7472df936254ec330bfd760a9a859f340aaec2
{
  "SandboxID": "7c8ccc4a76f0498ba30badc718072c40e0b948e8c155533e7889b8ef8576eeaf",
  "SandboxKey": "/var/run/docker/netns/7c8ccc4a76f0",
  "Ports": {
    "80/tcp": null
  },
  "Networks": {
    "bridge": {
      "IPAMConfig": null,
      "Links": null,
      "Aliases": null,
      "DriverOpts": null,
      "GwPriority": 0,
      "NetworkID": "bc98f6927cc0ed0679f68466a81a623e60a7a02ee6173e8bf2dff2baa26c6986",
      "EndpointID": "92e52b6f22097ee68b446ec2edf78fe7538b7cd711c493a6c1f5c50bd1cc7a95",
      "Gateway": "172.18.0.1",
      "IPAddress": "172.18.0.2",
      "MacAddress": "66:51:52:b0:1d:9e",
      "IPPrefixLen": 16,
      "IPv6Gateway": "",
      "GlobalIPv6Address": "",
      "GlobalIPv6PrefixLen": 0,
      "DNSNames": null
    }
  }
}
```

- IPAddress: 172.18.0.2.
- Docker network: bridge.
- Чому IP недоступний з інтернету: 172.18.0.2 належить до приватного діапазону 172.16.0.0/12, а такі адреси не маршрутизуються в інтернеті. Ця адреса існує лише у віртуальній мережі bridge всередині хоста. Контейнер має власний мережевий простір імен (`SandboxKey` показує шлях до нього). Зовні до нього можна дістатися тільки через опублікований порт хоста, а в цьому контейнері порт не опубліковано (`"80/tcp": null`).

## Завдання 4. Port Mapping та NAT на практиці

Запуск контейнера:

```bash
docker run -d -p 8080:80 --name web-server nginx
docker ps
```

Відповідь:

```
d202ae8e6f921bcd89d6d0bb6932e8ea0de4dc023198d988e31f46f6c9448b8e
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
d202ae8e6f92   nginx     "/docker-entrypoint.…"   2 seconds ago   Up 2 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web-server
ed61d7c7032f   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 4 seconds   80/tcp                                    test-net
```

Перевірка доступності:

```bash
curl -I localhost:8080
```

Відповідь:

```
HTTP/1.1 200 OK
Server: nginx/1.31.8
Date: Tue, 10 Nov 2026 22:34:12 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Wed, 21 Oct 2026 14:24:51 GMT
Connection: keep-alive
ETag: "6ad8cb33-380"
Accept-Ranges: bytes
```

```bash
curl localhost:8080
```

Відповідь:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

NAT-правила:

```bash
sudo iptables -t nat -L -n
```

Відповідь:

```
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.18.0.0/16        0.0.0.0/0

Chain DOCKER (2 references)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.18.0.3:80
```

Перевірка через зовнішню адресу хоста (не localhost):

```bash
curl -sI http://172.17.0.4:8080 | head -1
```

Відповідь:

```
HTTP/1.1 200 OK
```

- Результат curl: 200 OK, відкривається стартова сторінка nginx.
- Результат iptables: `DNAT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:8080 to:172.18.0.3:80`.
- Логіка: клієнт надсилає запит на IP хоста і порт 8080. У ланцюзі PREROUTING пакет потрапляє в ланцюг DOCKER, де правило DNAT замінює адресу призначення на 172.18.0.3:80. Далі хост маршрутизує пакет через bridge `docker0` у контейнер. Клієнт бачить тільки адресу хоста, а контейнер лишається в ізольованій мережі. Зворотні пакети система розпізнає через conntrack і автоматично підставляє назад адресу хоста.

Правило в ланцюзі OUTPUT не застосовується до 127.0.0.0/8, тому запит на `localhost:8080` обробляє `docker-proxy`. Запит на зовнішню адресу хоста проходить через DNAT, і він теж повернув 200 OK.

## Завдання 5. Архітектура Kubernetes: Pod vs Service

1. **Чому контейнери в одному Pod спілкуються через localhost.** Усі контейнери Pod мають спільний мережевий простір імен, який створює службовий pause-контейнер. Тому в них одна IP-адреса, один набір портів і спільний інтерфейс loopback. Контейнер просто звертається до `localhost:порт` сусіда, але два контейнери в одному Pod не можуть зайняти той самий порт.

2. **Навіщо потрібен Service (ClusterIP).** Pod ефемерні: при перезапуску або масштабуванні вони створюються заново і отримують нові IP, тому покладатися на адресу конкретного Pod не можна. Service дає стабільну віртуальну IP-адресу та DNS-ім'я, знаходить потрібні Pod за селектором міток і розподіляє між ними запити.

3. **Роль kube-proxy.** kube-proxy працює на кожному вузлі та стежить через API-сервер за змінами Service і EndpointSlice. На основі цих даних він налаштовує на вузлі правила (iptables, IPVS або nftables), які перенаправляють трафік із ClusterIP на IP одного з робочих Pod. Коли Pod змінюються, правила оновлюються, а адреса Service для клієнтів лишається тією самою.
