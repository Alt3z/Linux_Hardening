# Linux Hardening — Password Policy Verify Module

Ansible-плейбук для проверки политики паролей на целевых хостах.  
Не меняет конфигурацию — только читает состояние системы и формирует отчёты.

## Что делает playbook

Роль `password_policy_verify` прогоняет три блока проверок на каждом хосте:

**1. lastlog / faillog / chage** (`tags: lastlog`)
- Полный вывод `lastlog`
- Список пользователей, которые хотя бы раз входили в систему
- Для каждого такого пользователя:
  - `faillog` — неудачные попытки входа
  - `chage -l` — параметры срока действия пароля

**2. /etc/login.defs** (`tags: login_defs`)
Проверяет ключевые параметры политики паролей:

| Параметр        | Ожидание                          |
|-----------------|-----------------------------------|
| `PASS_MAX_DAYS` | = 90                              |
| `PASS_MIN_DAYS` | в диапазоне 1–7                   |
| `PASS_WARN_AGE` | в диапазоне 7–14                  |
| `ENCRYPT_METHOD`| = SHA512                          |

**3. PAM common-password** (`tags: pam_common_password`)
- Наличие модуля `pam_pwquality.so`
- Строка с `pam_pwquality` в `/etc/pam.d/common-password`
- Параметры сложности пароля:

| Параметр   | Ожидание        | Смысл                          |
|------------|-----------------|--------------------------------|
| `control`  | `requisite`     | жёсткая проверка               |
| `retry`    | ≥ 3             | число попыток ввода            |
| `minlen`   | ≥ 12            | минимальная длина              |
| `ucredit`  | ≤ -1            | минимум 1 заглавная буква      |
| `lcredit`  | ≤ -1            | минимум 1 строчная буква       |
| `dcredit`  | ≤ -1            | минимум 1 цифра                |
| `ocredit`  | ≤ -1            | минимум 1 спецсимвол           |

Результаты каждой проверки пишутся в текстовые отчёты на control-ноде.

## Структура

```
roles/
└── password_policy_verify/
├── defaults/
│   └── main.yml              # ожидаемые значения + путь к отчётам
├── tasks/
│   ├── main.yml              # точка входа (import_tasks)
│   ├── lastlog.yml
│   ├── login_defs.yml
│   └── pam_common_password.yml
└── templates/
├── report.j2             # отчёт lastlog/faillog/chage
├── login_defs_report.j2
└── pam_common_password_report.j2
playbooks/
└── verify_password_policy.yml    # точка входа
```


## Куда сохраняются отчёты

```
reports/
└── <inventory_hostname>/
├── password_policy.txt       # lastlog + faillog + chage
├── login_defs.txt            # проверка /etc/login.defs
└── pam_common_password.txt   # проверка PAM
```


Путь задаётся переменной `password_policy_reports_dir`  
(по умолчанию: `{{ playbook_dir }}/../reports`).

## Ожидаемые значения

Все пороги лежат в `roles/password_policy_verify/defaults/main.yml`.  
Их можно переопределить через `group_vars` / `host_vars` без правки роли.

```yaml
login_defs_expected:
  PASS_MAX_DAYS: 90
  PASS_MIN_DAYS_MIN: 1
  PASS_MIN_DAYS_MAX: 7
  PASS_WARN_AGE_MIN: 7
  PASS_WARN_AGE_MAX: 14
  ENCRYPT_METHOD: SHA512

pam_pwquality_expected:
  module: pam_pwquality.so
  control: requisite
  retry: 3
  minlen: 12
  ucredit: -1
  lcredit: -1
  dcredit: -1
  ocredit: -1
```

## Запуск

Полная проверка

```bash
ansible-playbook playbooks/verify_password_policy.yml
```

Только нужные части

```bash
ansible-playbook playbooks/verify_password_policy.yml --tags lastlog
ansible-playbook playbooks/verify_password_policy.yml --tags login_defs
ansible-playbook playbooks/verify_password_policy.yml --tags pam_common_password
```
