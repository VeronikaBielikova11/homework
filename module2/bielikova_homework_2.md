# Домашнє завдання 2: Транспортний рівень та служби мережі

Команди виконано в Docker-контейнері `ubuntu:22.04`. Система була «чиста», тому сервіси для завдань 1–2 я запустила вручну:

```bash
python3 -m http.server 8080 > /dev/null 2>&1 &
nc -lk 9000 < /dev/null > /dev/null 2>&1 &
nc -ulk 9001 < /dev/null > /dev/null 2>&1 &
```

## Завдання 1. Аналіз відкритих портів

```
root@0e863d13a192:/# ss -tuln
Netid  State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
udp    UNCONN  0       0              0.0.0.0:9001        0.0.0.0:*
tcp    LISTEN  0       1              0.0.0.0:9000        0.0.0.0:*
tcp    LISTEN  0       5              0.0.0.0:8080        0.0.0.0:*
```

- TCP, порт 8080, стан LISTEN: веб-сервер Python
- TCP, порт 9000, стан LISTEN: netcat
- UDP, порт 9001, стан UNCONN: netcat

UDP не встановлює з'єднань, тому його порт має стан UNCONN, а не LISTEN. Порт при цьому відкритий і приймає дані.

## Завдання 2. Аналіз процесів, що слухають порти

```
root@0e863d13a192:/# ss -tulpn
udp   UNCONN  0  0  0.0.0.0:9001  0.0.0.0:*  users:(("nc",pid=640,fd=3))
tcp   LISTEN  0  1  0.0.0.0:9000  0.0.0.0:*  users:(("nc",pid=639,fd=3))
tcp   LISTEN  0  5  0.0.0.0:8080  0.0.0.0:*  users:(("python3",pid=638,fd=3))
```

- Процес: python3, PID: 638, порт: 8080
- Процес: nc, PID: 639, порт: 9000
- Процес: nc, PID: 640, порт: 9001

## Завдання 3. Перевірка TCP-з'єднання

```
root@0e863d13a192:/# nc -vz google.com 80
Connection to google.com (142.250.120.138) 80 port [tcp/*] succeeded!
```

- Тип з'єднання: TCP
- Порт: 80
- Результат: з'єднання встановлено

## Завдання 4. Порівняння TCP vs UDP

```
root@0e863d13a192:/# nc -u -vz 8.8.8.8 53
Connection to 8.8.8.8 53 port [udp/*] succeeded!
root@0e863d13a192:/# nc -vz 8.8.8.8 53
Connection to 8.8.8.8 53 port [tcp/*] succeeded!
```

Обидві команди показали `succeeded`, але це означає різне:

- TCP: `succeeded` означає, що тристороннє рукостискання (SYN, SYN-ACK, ACK) завершилося і сервер справді відповів.
- UDP: рукостискання немає, тому `succeeded` лише означає, що помилки не надійшло. Це не гарантує, що сервер отримав дані чи що порт відкритий.
- Різниця: TCP встановлює сесію і підтверджує з'єднання, а UDP працює без встановлення з'єднання (connection-less) і підтвердження не дає.

## Завдання 5. DNS-резолвінг

```
root@0e863d13a192:/# nslookup github.com
Server:		192.168.65.7
Address:	192.168.65.7#53

Non-authoritative answer:
Name:	github.com
Address: 140.82.121.4
```

- Домен: github.com
- IP: 140.82.121.4
- DNS-сервер: 192.168.65.7 (порт 53)

Адреса 192.168.65.7 належить внутрішньому DNS Docker, бо команду виконано в контейнері. `Non-authoritative answer` означає, що відповів не авторитетний сервер домену github.com, а проміжний.