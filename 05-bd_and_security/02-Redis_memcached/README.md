# Домашнее задание к занятию 11.2 "Кеширование Redis/memcached" - Погорелый Даниил

[Memcached Telnet Commands Example](https://www.digitalocean.com/community/tutorials/memcached-telnet-commands-example)

[Шпаргалка по Redis](https://habr.com/ru/articles/204354/)

---

### Задание 1

Приведите примеры проблем, которые может решить кеширование.

#### Требования к результату
1. Приведите ответ в свободной форме.

#### Решение

1. Улучшенная производительность: Кеширование позволяет снизить нагрузку на ресурсы сервера, такие как база данных или внешние сервисы,
путем сохранения результатов выполнения дорогостоящих операций в более быстром хранилище, доступ к которому осуществляется намного быстрее.
Это позволяет ускорить обработку запросов и сократить время отклика приложения.
2. Снижение нагрузки на сервер и сеть: Кеширование уменьшает количество запросов к базе данных или внешним сервисам,
что снижает нагрузку на сервер и сеть.
3. Улучшение надежности: Кеш можно использовать для временного хранения данных и обеспечения их доступности в случае сбоя или потери данных.

---

### Задание 2

Установите и запустите memcached.

#### Требования к результату
1. Приведите скриншот systemctl status memcached, где будет видно, что memcached запущен

#### Решение

```bash
sudo apt update && apt install memcached
```

![systemctl status memcached](https://github.com/DanPogorelyi/devops/blob/main/05-bd_and_security/02-Redis_memcached/images/systemctl_status_memcached.png)

---

### Задание 3

Запишите в memcached несколько ключей с любыми именами и значениями, для которых выставлен TTL 5.

#### Требования к результату
1. Приведите скриншот, на котором видно, что спустя 5 секунд ключи удалились из базы.

#### Решение

```bash
telnet localhost 11211

set key1 0 5 3
dan

set key2 0 5 3
mac

quit
```

![memcached set get](https://github.com/DanPogorelyi/devops/blob/main/05-bd_and_security/02-Redis_memcached/images/memcached_set_get.png)

---

### Задание 4

Запишите в Redis несколько ключей с любыми именами и значениями.

#### Требования к результату
1. Через redis-cli достаньте все записанные ключи и значения из базы, приведите скриншот этой операции.

#### Решение

```bash
sudo apt update && apt install redis

redis-cli

set test:1:string "my binary safe string"
set test:1:string2 "my binary safe string2222"

keys test:1:*

get test:1:string2
get test:1:string

```

![redis set get](https://github.com/DanPogorelyi/devops/blob/main/05-bd_and_security/02-Redis_memcached/images/redis_set_get.png)

---