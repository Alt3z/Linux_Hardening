# Linux Hardening — Packages Verify

Проверяет, что пакеты, устанавливаемые `install_packages.yml`, реально присутствуют на хосте, и показывает установленную версию каждого.

## Что делает

Через `ansible.builtin.package_facts` (apt) собирает список установленных пакетов, сверяет с `packages_expected` (`roles/packages_verify/defaults/main.yml`), пишет отчёт с версией по каждому и итогом.

## Структура

```
roles/packages_verify/
├── defaults/main.yml # packages_expected — список ожидаемых пакетов
├── tasks/
│ ├── main.yml
│ └── packages.yml
└── templates/
└── packages_report.j2
```

## Запуск

```bash
ansible-playbook playbooks/verify_packages.yml
```

## Результат

```
reports/test1/packages.txt
```
