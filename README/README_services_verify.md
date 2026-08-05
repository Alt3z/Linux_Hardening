# Linux Hardening — Services Verify

Проверяет состояние ключевых служб на хосте: SSH, iptables/netfilter-persistent, Kaspersky (KESL, klnagent64) — и дополнительно показывает текущие правила iptables.

## Что делает

**`ssh_service`** — проверяет, что `ssh.service` в состоянии `running`.

**`iptables`** — проверяет состояние `netfilter-persistent.service` и `iptables.service` (если вообще существуют как юниты — сам iptables не является службой per se, правила это состояние ядра; персистентность обеспечивает именно `netfilter-persistent`, oneshot-юнит, штатно находящийся в состоянии `exited`, а не `running`, после успешного применения правил). Плюс выводит полный дамп текущих правил (`iptables -L -n -v --line-numbers`, и `ip6tables`, если IPv6 доступен).

**`kaspersky`** — проверяет состояние служб `kesl.service` и `klnagent64.service`.

## Все проверки собирают факты один раз

`service_facts` вызывается один раз в начале роли с тегом `always` — гарантированно выполняется независимо от того, какой тег запрошен через `--tags`, так что данные о службах доступны любому отдельному блоку проверки.

## Структура

```
roles/services_verify/
├── defaults/main.yml
├── tasks/
│ ├── main.yml
│ ├── verify_ssh_service.yml
│ ├── iptables.yml
│ └── kaspersky.yml
└── templates/
├── iptables_report.j2
└── kaspersky_report.j2
```

## Запуск

```bash
ansible-playbook playbooks/verify_services.yml
ansible-playbook playbooks/verify_services.yml --tags ssh_service
ansible-playbook playbooks/verify_services.yml --tags iptables
ansible-playbook playbooks/verify_services.yml --tags kaspersky
```

## Результат

```
reports/test1/iptables.txt # состояние службы + полный дамп правил
reports/test1/kaspersky.txt # состояние обеих служб, OK/FAIL по каждой
``
