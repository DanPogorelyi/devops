# Домашнее задание к занятию 9.5. "Система мониторинга Prometheus Часть 2" - Погорелый Даниил

[Ansible Prometheus Playbook](https://github.com/mesaguy/ansible-prometheus)

[How install Prometheus on Ubuntu 22.04](https://itslinuxfoss.com/how-to-install-prometheus-on-ubuntu-22-04-lts/)

[How install Prometheus Alert Manager](https://linuxhint.com/install-configure-prometheus-alert-manager-ubuntu/)

[How To Install and Use Docker on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

---

### Задание 1
Создайте файл с правилом оповещения, как в лекции, и добавьте его в конфиг Prometheus.

#### Требования к результату
1. Погасите node exporter, стоящий на мониторинге, и прикрепите скриншот раздела оповещений Prometheus, где оповещение будет в статусе Pending

#### Решение

Download
```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.dragonfly-amd64.tar.gz
```

Extract
```bash
sudo tar xvf alertmanager*.tar.gz
```

Move to /opt/alertmanager
```bash
sudo mv -v alertmanager*.tar.gz /opt/alertmanager
```

Change the user and group of all the files and directories of the /opt/alertmanager/ directory to root as follows:
```bash
sudo chown -Rfv root:root /opt/alertmanager
```

Create the data/ directory
```bash
sudo mkdir -v /opt/alertmanager/data
```

Change the owner and group of the /opt/alertmanager/data/ directory to prometheus with the following command:
```bash
sudo chown -Rfv prometheus:prometheus /opt/alertmanager/data
```

Create a systemd service file alertmanager.service
```bash
sudo nano /etc/systemd/system/alertmanager.service
```

```bash
[Unit]
Description=Alertmanager for prometheus

[Service]
Restart=always
User=prometheus
ExecStart=/opt/alertmanager/alertmanager --config.file=/opt/alertmanager/alertmanager.yml --storage.path=/opt/alertmanager/data            
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager.service
sudo systemctl enable alertmanager.service
sudo systemctl status alertmanager.service
```

Add Alert Manager config
```bash
sudo nano /etc/prometheus/prometheus.yml
```

```yaml
alerting:
  alertmanagers:
    - static_configs:
      - targets:
      - localhost:9093 
```

```bash
sudo systemctl restart prometheus
systemctl status prometheus
```

Create rule
```bash
sudo nano /etc/prometheus/netology-test.yml
```

```yaml
groups: # Список групп
  - name: netology-test # Имя группы
    rules: # Список правил текущей группы
  - alert: InstanceDown # Название текущего правила
    expr: up == 0 # Логическое выражение
    for: 1m # Сколько ждать отбоя предупреждения перед отправкой оповещения
    labels:
    severity: critical # Критичность события
    annotations: # Описание
    description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.' # Полное описание алерта
    summary: Instance {{ $labels.instance }} down # Краткое описание алерта
```

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```yaml
rule_files:
  - "netology-test.yml"
```

```bash
sudo nano /etc/prometheus/alertmanager.yml
```

```yaml
global:
route:
  group_by: ['alertname'] # Параметр группировки оповещений - по имени
  group_wait: 30s # Сколько ждать восстановления, перед тем как отправить первое оповещение.
  group_interval: 10m # Сколько ждать перед тем как дослать оповещение о новых сработках по текущему алерту.
  repeat_interval: 60m # Сколько ждать перед тем как отправить повторное оповещение
  receiver: 'email' # Способ которым будет доставляться текущее оповещение
receivers: # Настройка способов оповещения
  - name: 'email'
    email_configs:
      - to: 'yourmailto@todomain.com'
        from: 'yourmailfrom@fromdomain.com'
        smarthost: 'mailserver:25'
        auth_username: 'user'
        auth_identity: 'user'
        auth_password: 'paS$w0rd'
```

```bash
sudo systemctl restart alertmanager
systemctl status alertmanager
sudo systemctl stop node-exporter
systemctl status node-exporter
```

![prometheus alerts pending](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-2/images/prometheus-alerts-pending.png)

---

### Задание 2
Установите Alertmanager и интегрируйте его с Prometheus.

#### Требования к результату
1. Прикрепите скриншот Alerts из Prometheus, где правило оповещения будет в статусе Fireing, и скриншот из Alertmanager, где будет видно действующее правило оповещения

#### Решение

![prometheus alerts firing](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-2/images/prometheus-alerts-firing.png)

---

### Задание 3
Активируйте экспортёр метрик в Docker и подключите его к Prometheus.

#### Требования к результату
1. Приложите скриншот браузера с открытым эндпоинтом, а также скриншот списка таргетов из интерфейса Prometheus.*

#### Решение

```bash
sudo apt-get update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Add key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Add the Docker repository to APT sources:
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker
```bash
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
sudo systemctl status docker
```

Configure docker daemon.json
```bash
sudo nano /etc/docker/daemon.json
```

```json
{
  "metrics-addr" : "0.0.0.0:9323",
  "experimental" : true
}
```

```bash
sudo systemctl restart docker && systemctl status docker
```

Add endpoint Docker in Prometheus

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```yaml
static_configs:
  - targets: ["84.201.163.21:9090", "84.201.163.21:9100", "84.201.163.21:9323"]
```

```bash
sudo systemctl restart prometheus
```

![docker metrics](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-2/images/docker-metrics.png)

![prometheus targets](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/04-Prometheus_part-2/images/prometheus-targets.png)

---

### Задание 4*
Создайте свой дашборд Grafana с различными метриками Docker и сервера, на котором он стоит.

#### Требования к результату
1. Приложите скриншот, на котором будет дашборд Grafana с действующей метрикой

#### Решение

---
