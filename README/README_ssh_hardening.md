# Linux Hardening — SSH Hardening Module

Ansible-роль для настройки SSH: раскатка публичного ключа и hardened `sshd_config`. Генерация самого ключа вынесена в отдельный playbook — см. [README_generate_ssh_key.md](README_generate_ssh_key.md), выполняется отдельно и не входит в этот прогон.

## Что делает playbook

1. Создаёт `~/.ssh` на хосте с правами `0700`
2. Раскатывает публичный ключ (сгенерированный заранее) в `authorized_keys`
3. Деплоит захардненный `/etc/ssh/sshd_config` из шаблона, с валидацией `sshd -t` перед применением — если конфиг битый, файл не подменяется и sshd не падает
4. Рестартует `sshd` только если конфиг реально изменился

Хосты обрабатываются последовательно (`serial: 1`) — если что-то пойдёт не так на первом хосте, второй останется нетронутым.

## Настраиваемые параметры sshd_config

Единый источник значений — `inventory/group_vars/linux_hosts/vars.yml`, переменная `ssh_hardening_expected` (используется и при накатке, и при проверке — не расходятся):

| Параметр | Значение |
|---|---|
| LogLevel | VERBOSE |
| PermitRootLogin | no |
| PubkeyAuthentication | yes |
| AuthorizedKeysFile | .ssh/authorized_keys |
| PasswordAuthentication | no |
| PermitEmptyPasswords | no |
| KbdInteractiveAuthentication | no |
| UsePAM | yes |
| X11Forwarding | no |
| PrintMotd | no |
| AcceptEnv | LANG LC_* |
| Subsystem | sftp /usr/lib/openssh/sftp-server |
| AllowUsers | `ansible_user` + `ssh_hardening_extra_allow_users` |

Чтобы разрешить доступ дополнительным пользователям, помимо того, от чьего имени работает Ansible:
```yaml
# inventory/group_vars/linux_hosts/vars.yml
ssh_hardening_extra_allow_users: ["admin", "devops"]
```

## Структура

```
roles/ssh_hardening/
├── defaults/main.yml
├── tasks/main.yml
├── handlers/main.yml
└── templates/sshd_config.j2
```

## Запуск

```bash
ansible-playbook playbooks/ssh_hardening.yml
```

Повторные прогоны безопасны — при уже применённой конфигурации ничего не меняется (`changed=0`).

## Проверка после прогона

```bash
ansible-playbook playbooks/verify_ssh_hardening.yml
```
Подробности — [README_ssh_verify.md](README_ssh_verify.md).
