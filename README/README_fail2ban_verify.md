# Linux Hardening — Fail2ban Verify

Проверяет, что служба fail2ban запущена и секция `[sshd]` настроена согласно ожидаемым значениям.

## Что делает

- Проверяет состояние `fail2ban.service` через `service_facts`
- Читает конфигурацию в порядке приоритета переопределений: `jail.conf` → `jail.d/*.conf`/`*.local` (по алфавиту) → `jail.local` — последнее найденное значение параметра побеждает, как это работает у самого fail2ban
- Парсит секции (`[sshd]` и т.д.) из объединённого текста построчно, отбрасывая комментарии (`#`/`;`)
- Сравнивает найденные параметры секции `[sshd]` с `fail2ban_sshd_settings` из `group_vars` (та же переменная, что использует `fail2ban_hardening`)

## Структура

```
roles/fail2ban_verify/
├── defaults/main.yml
├── tasks/
│ ├── main.yml
│ └── fail2ban.yml
└── templates/
└── fail2ban_report.j2
```

## Запуск

```bash
ansible-playbook playbooks/verify_fail2ban.yml
```

## Результат

```
reports/test1/fail2ban.txt
```
