# Linux Hardening — Fail2ban Hardening

Устанавливает fail2ban и настраивает jail `[sshd]` через `jail.local`.

## Что делает

- Ставит пакет `fail2ban` (идемпотентно, если уже стоит через `install_packages.yml` — ничего не меняет)
- Деплоит `/etc/fail2ban/jail.local` с секцией `[sshd]` — пишет именно в `.local`, а не в `.conf`, так как `.conf` считается заводским файлом и может быть перезаписан при обновлении пакета
- Включает и запускает службу

## Настраиваемые параметры

`inventory/group_vars/linux_hosts/vars.yml`:
```yaml
fail2ban_sshd_settings:
  enabled: "true"
  port: "ssh"
  filter: "sshd"
  logpath: "/var/log/auth.log"
  maxretry: "5"
  bantime: "3600"
```
Та же переменная используется в `fail2ban_verify` для сравнения — единый источник правды.

## Структура

```
roles/fail2ban_hardening/
├── tasks/main.yml
├── handlers/main.yml
└── templates/jail.local.j2
```

## Запуск

```bash
ansible-playbook playbooks/fail2ban_hardening.yml
```

## Известное ограничение

У fail2ban нет штатной команды для сухой проверки синтаксиса конфига (аналога `sshd -t`) — если конфиг окажется битым, это будет видно постфактум через `fail2ban_verify` (служба не в `running`), а не заранее.

## Проверка результата

```bash
ansible-playbook playbooks/verify_fail2ban.yml
```
Подробности — [README_fail2ban_verify.md](README_fail2ban_verify.md).
