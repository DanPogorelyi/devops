# Домашнее задание к занятию 9.2. "Система мониторинга Zabbix" - Погорелый Даниил


---

### Задание 1

Установите Zabbix Server с веб-интерфейсом.

#### Процесс выполнения
1. Выполняя ДЗ сверяйтесь с процессом отражённым в записи лекции.
2. Установите PostgreSQL. Для установки достаточна та версия что есть в системном репозитороии Debian 11
3. Пользуясь конфигуратором комманд с официального сайта, составьте набор команд для установки последней версии Zabbix с поддержкой PostgreSQL и Apache
4. Выполните все необходимые команды для установки Zabbix Server и Zabbix Web Server

#### Требования к результаты
1. Прикрепите в файл README.md скриншот авторизации в админке
2. Приложите в файл README.md текст использованных команд в GitHub

#### Решение

Host server

```bash
vim /etc/hosts
```

![Скриншот-1](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/zabbix-server_hosts.png)


```bash
wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb
dpkg -i zabbix-release_6.4-1+debian11_all.deb
apt update
```

```bash
apt install postgresql
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

```bash
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
```

Импорт начальной схемы и данных:
```bash
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

Настройка базы данных для Zabbix сервера:
```bash
vim /etc/zabbix/zabbix_server.conf
```

![Скриншот-2](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/DBPassword.png)


```bash
systemctl restart  zabbix-server.service
systemctl status zabbix-server.service
```

![Скриншот-3](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/zabbix-server_status.png)


Настройка PHP для веб-интерфейса:
```bash
vim /etc/zabbix/nginx.conf
```

![Скриншот-4](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/nginx_conf.png)


```bash
systemctl restart  nginx
systemctl status nginx
```

![Скриншот-5](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/nginx_status.png)


### login
login: Admin

password: zabbix

![Скриншот-6](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/zabbix-server_adminka.png)

---

### Задание 2

Установите Zabbix Agent на два хоста.

#### Процесс выполнения
1. Выполняя ДЗ сверяйтесь с процессом отражённым в записи лекции.
2. Установите Zabbix Agent на 2 виртмашины, одной из них может быть ваш Zabbix Server
3. Добавьте Zabbix Server в список разрешенных серверов ваших Zabbix Agentов
4. Добавьте Zabbix Agentов в раздел Configuration > Hosts вашего Zabbix Servera
5. Проверьте что в разделе Latest Data начали появляться данные с добавленных агентов

#### Требования к результаты
1. Приложите в файл README.md скриншот раздела Configuration > Hosts, где видно, что агенты подключены к серверу
2. Приложите в файл README.md скриншот лога zabbix agent, где видно, что он работает с сервером
3. Приложите в файл README.md скриншот раздела Monitoring > Latest data для обоих хостов, где видны поступающие от агентов данные.
4. Приложите в файл README.md текст использованных команд в GitHub

#### Решение

```bash
wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb
dpkg -i zabbix-release_6.4-1+debian11_all.deb
apt update
```

```bash
apt install zabbix-agent2 zabbix-agent2-plugin-*
```

```bash
systemctl status zabbix-agent2.service
```

![Скриншот-7](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/host-1_zabbix_status.png)

![Скриншот-8](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/host-2_zabbix_status.png)


```bash
vim /etc/zabbix/zabbix_agent2.conf
```

![Скриншот-9](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/host-1_add_ip-server.png)

![Скриншот-10](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/host-2_add_ip-server.png)


```bash
systemctl restart zabbix-agent2.service
```


Создание хостов:
![Скриншот-11](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/host-1_create_host.png)

![Скриншот-12](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/host-2_create_host.png)


```bash
tail -f /var/log/zabbix/zabbix_agent2.log
```


![Скриншот-13](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/hosts.png)

![Скриншот-14](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/logs.png)

![Скриншот-15](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/02-zabbix_part-1/images/monitoring.png)

---
