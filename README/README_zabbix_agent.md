# Linux Hardening — Zabbix Agent

Устанавливает и настраивает Zabbix-агент, подключающий хост к Zabbix-серверу для мониторинга.

## Что делает

**Установка** (`zabbix_agent_hardening`):
- ставит пакет `zabbix-agent`
- создаёт директорию логов `/var/log/zabbix` с владельцем `zabbix:zabbix`
- деплоит `/etc/zabbix/zabbix_agentd.conf` с адресом сервера, включая `Hostname` (берётся автоматически из `ansible_hostname`)
- включает и запускает службу

## Настраиваемый параметр

`inventory/group_vars/linux_hosts/vars.yml`:
```yaml
zabbix_server_ip: "192.168.56.101"
```
Используется и в `Server`, и в `ServerActive` конфига — единый источник для настройки и проверки.

## Структура

```
roles/
├── zabbix_agent_hardening/
│ ├── tasks/main.yml
│ ├── handlers/main.yml
│ └── templates/zabbix_agentd.conf.j2
└── zabbix_agent_verify/
├── defaults/main.yml
├── tasks/
│ ├── main.yml
│ └── zabbix_agent.yml
└── templates/
└── zabbix_agent_report.j2
```

## Запуск

```bash
ansible-playbook playbooks/zabbix_agent_hardening.yml
ansible-playbook playbooks/verify_zabbix_agent.yml
```

## Что проверяет verify

- служба `zabbix-agent.service` в состоянии `running`
- агент реально слушает порт `10050` локально (не просто запущен как процесс)
- конфиг содержит актуальные `Server`/`ServerActive`

Если агент ещё не установлен — все проверки честно показывают `FAIL`/`не найдена`, без падения плейбука (используется `ignore_errors` на сетевой проверке и `when` на чтении конфига).

## Результат

```
reports/test1/zabbix_agent.txt
```


