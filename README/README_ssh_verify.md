k# Linux Hardening — SSH Verify Module

Playbook для проверки того, что `ssh_hardening` реально применился и работает: конфиг, реальный доступ и факт логирования.

## Что проверяет

**1. sshd_config — эффективная конфигурация (`sshd -T`)**
Каждый управляемый параметр (см. [README_ssh_hardening.md](README_ssh_hardening.md)) сравнивается с ожидаемым значением. `AllowUsers`/`AcceptEnv` (многозначные директивы) сравниваются как списки — `sshd -T` печатает их отдельной строкой на каждое значение.

**2. Директивы сверх управляемого набора**
Сырой `/etc/ssh/sshd_config` построчно сверяется со списком `ssh_hardening_managed_directives` (в `roles/ssh_verify/defaults/main.yml`). Всё, что найдено сверх этого списка — выводится в отчёт как есть, без дополнительной интерпретации (включая `Include`, если он где-то прописан).

**3. Фактический доступ**
- права на `.ssh` (0700)
- публичный ключ реально в `authorized_keys`
- вход под root — отклонён
- вход по паролю — отклонён
- вход по ключу — работает

Все три блока — в одном файле отчёта, `ssh_config.txt`, потому что проверяют связанные, но не тождественные вещи: конфиг может быть верным, а реальный доступ — сломан (например, из-за прав на ключ), и наоборот.

**4. Логирование аутентификации (`/var/log/auth.log`)**
Не содержимое лога (оно может быть большим и чувствительным), а три факта:
- файл существует
- в нём есть записи от sshd
- файл обновлялся за последние 24 часа

## Зависимости

`sshpass` на control-ноде — для теста "вход по паролю должен быть отклонён":
```bash
sudo apt install sshpass
```

Ключ должен быть в `ssh-agent` перед запуском (ключ с passphrase — см. [README_generate_ssh_key.md](README_generate_ssh_key.md)):
```bash
ssh-add -l   # проверить, что ключ загружен
```

## Структура

```
roles/ssh_verify/
├── defaults/main.yml
├── tasks/
│ ├── main.yml
│ ├── sshd_config.yml # конфиг + доступ, единый отчёт
│ └── auth_log.yml
└── templates/
├── ssh_config_report.j2
└── ssh_auth_log_report.j2
```

## Запуск

```bash
ansible-playbook playbooks/verify_ssh_hardening.yml
ansible-playbook playbooks/verify_ssh_hardening.yml --tags sshd_config
ansible-playbook playbooks/verify_ssh_hardening.yml --tags auth_log
```

## Результат

```
reports/test1/ssh_config.txt # параметры, лишние директивы, доступ
reports/test1/ssh_auth_log.txt # факт логирования
```
