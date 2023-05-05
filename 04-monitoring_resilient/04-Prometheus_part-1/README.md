# Домашнее задание к занятию 9.4. "Система мониторинга Prometheus Часть 1" - Погорелый Даниил

[Ansible Prometheus Playbook](https://github.com/mesaguy/ansible-prometheus)

[How install Prometheus on Ubuntu 22.04](https://itslinuxfoss.com/how-to-install-prometheus-on-ubuntu-22-04-lts/)

---

### Задание 1
Установите Prometheus.

#### Процесс выполнения
1. Выполняя задание, сверяйтесь с процессом, отражённым в записи лекции
2. Создайте пользователя prometheus
3. Скачайте prometheus и в соответствии с лекцией разместите файлы в целевые директории
4. Создайте сервис как показано на уроке
5. Проверьте что prometheus запускается, останавливается, перезапускается и отображает статус с помощью systemctl

#### Требования к результату
1. Прикрепите к файлу README.md скриншот systemctl status prometheus, где будет написано: prometheus.service — Prometheus Service Netology Lesson 9.4 — [Ваши ФИО]

#### Решение

Create a user and Prometheus system’s group
```bash
sudo groupadd --system prometheus
```

Add the Prometheus user and assign it the created group
```bash
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

Create the primary directory
```bash
sudo mkdir /etc/prometheus
```

Create sub directory
```bash
sudo mkdir /var/lib/prometheus
```

Download
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.37.7/prometheus-2.37.7.linux-386.tar.gz
```

Extract
```bash
tar xvf prometheus*.tar.gz
```



Navigate to directory

```bash
cd prometheus*/
```

Move the binary files
```bash
sudo mv prometheus promtool /usr/local/bin/
```

Confirm the installed version
```bash
prometheus --version
```

Move Prometheus configuration template
```bash
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

Move console libraries
```bash
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

Configure Prometheus
```bash
sudo vim /etc/prometheus/prometheus.yml
```

Create Prometheus systemd service
```bash
sudo nano /etc/systemd/system/prometheus.service
```

```bash
[Unit]
Description=Prometheus Service Netology Lesson 9.4 — Pogorelyi Daniil
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Reload and enable the Prometheus
```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable --now prometheus
```

```bash
sudo ufw allow 9090/tcp
```

Provide permissions to Prometheus files
```bash
chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
```

![systemctl status prometheus](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-1/images/systemctl-status-prometheus.png)

---

### Задание 2
Установите Node Exporter.

#### Процесс выполнения
1. Выполняя ДЗ сверяйтесь с процессом отражённым в записи лекции.
2. Скачайте node exporter приведённый в презентации и в соответствии с лекцией разместите файлы в целевые директории
3. Создайте сервис для как показано на уроке
4. Проверьте что node exporter запускается, останавливается, перезапускается и отображает статус с помощью systemctl

#### Требования к результату
1. Прикрепите к файлу README.md скриншот systemctl status node-exporter, где будет написано: node-exporter.service — Node Exporter Netology Lesson 9.4 — [Ваши ФИО]

#### Решение

Download
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-386.tar.gz
```

Extract
```bash
tar xvf node_exporter*.tar.gz
```

Copy
```bash
mkdir /etc/prometheus/node-exporter
cp ./* /etc/prometheus/node-exporter
```

Provide permissions
```bash
chown -R prometheus:prometheus /etc/prometheus/node-exporter/
```

Create service
```bash
vim /etc/systemd/system/node-exporter.service
```

```bash
[Unit]
Description=Node Exporter Netology Lesson 9.4 — Pogorelyi Daniil
After=network.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/etc/prometheus/node-exporter/node_exporter
[Install]
WantedBy=multi-user.target
```


Reload and enable the Node Exporter
```bash
sudo systemctl enable node-exporter
sudo systemctl start node-exporter
sudo systemctl status node-exporter
```

![systemctl status node exporter](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-1/images/systemctl-status-node-exporter.png)

---

### Задание 3
Подключите Node Exporter к серверу Prometheus.

#### Процесс выполнения
1. Выполняя ДЗ сверяйтесь с процессом отражённым в записи лекции.
2. Отредактируйте prometheus.yaml, добавив в массив таргетов установленный в задании 2 node exporter
3. Перезапустите prometheus
4. Проверьте что он запустился

#### Требования к результату
1. Прикрепите к файлу README.md скриншот конфигурации из интерфейса Prometheus вкладки Status > Configuration
2. Прикрепите к файлу README.md скриншот из интерфейса Prometheus вкладки Status > Targets, чтобы было видно минимум два эндпоинта

#### Решение

Configure Prometheus
```bash
sudo vim /etc/prometheus/prometheus.yml
```

```yaml
scrape_configs:
  — job_name: 'prometheus'
  scrape_interval: 5s
  static_configs:
  — targets: ['localhost:9090', 'localhost:9100']
```

![prometheus status configuration](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-1/images/prometheus-status-configuration.png)

![prometheus status target](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-1/images/prometheus-status-target.png)

---

### Задание 4*
Установите Grafana.

#### Требования к результату
1. Прикрепите к файлу README.md скриншот левого нижнего угла интерфейса, чтобы при наведении на иконку пользователя были видны ваши ФИО

#### Решение

Download
```bash
wget https://dl.grafana.com/oss/release/grafana_9.2.4_amd64.deb
dpkg -i grafana_9.2.4_amd64.deb
```

```bash
systemctl enable grafana-server
systemctl start grafana-server
systemctl status grafana-server
```

![grafana user](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-1/images/grafana-user.png)

---

### Задание 5*
Интегрируйте Grafana и Prometheus.

#### Требования к результату
1. Прикрепите к файлу README.md скриншот дашборда (ID:11074) с поступающими туда данными из Node Exporter

#### Решение

![grafana dashboard](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-1/images/grafana-dashboard.png)

---