# Домашнее задание к занятию 10.1 "Keepalived/vrrp" - Погорелый Даниил

---

### Задание 1
Разверните топологию из лекции и выполните установку и настройку сервиса Keepalived.

```bash
vrrp_instance test {
  state "name_mode"
  interface "name_interface"
  virtual_router_id "number id"
  priority "number priority"
  advert_int "number advert"
  authentication {
      auth_type "auth type"
      auth_pass "password"
  }
  unicast_peer {
    "ip address host"
  }
  virtual_ipaddress {
    "ip address host" dev "interface" label "interface":vip
  }
}
```

#### Требования к результату
В качестве решения предоставьте:
1. рабочую конфигурацию обеих нод, оформленную как блок кода в вашем md-файле;
2. скриншоты статуса сервисов, на которых видно, что одна нода перешла в MASTER, а вторая в BACKUP state.

#### Решение

```bash
vrrp_instance main {
  state BACKUP
  interface eth0
  virtual_router_id 11
  priority 50
  advert_int 4
  authentication {
    auth_type AH
    auth_pass 123
  }
  unicast_peer {
    10.129.0.9
  }
  virtual_ipaddress {
    10.129.0.200 dev eth0 label MainNew
  }
}

vrrp_instance main {
  state MASTER
  interface eth0
  virtual_router_id 11
  priority 50
  advert_int 4
  authentication {
    auth_type AH
    auth_pass 123
  }
  unicast_peer {
    10.129.0.6
  }
  virtual_ipaddress {
    10.129.0.200 dev eth0 label MainNew
  }
}
```

![keepalived](https://github.com/DanPogorelyi/devops/blob/main/04-monitoring_resilient/05-Keepalive/images/keepalived-status.png)

---