# Linux Hardening — GLPI Agent

Устанавливает и настраивает GLPI-агент (инвентаризация, версия 1.19), связывающий хост с GLPI-сервером.

## Что делает

**Установка** (`glpi_agent_hardening`):
- ставит зависимости `perl`, `wget`
- скачивает perl-инсталлятор указанной версии в `files/glpi-agent-installer/` на control-ноде (единожды — если файл уже там лежит, повторно не скачивается, даже если сеть недоступна)
- копирует его на целевой хост через Ansible-канал (SSH), не требуя от самого хоста доступа в интернет
- запускает установку, автоматически отвечая пустой строкой на все интерактивные запросы инсталлятора
- деплоит `/etc/glpi-agent/conf.d/00-install.cfg` с параметрами подключения к серверу
- включает и запускает службу `glpi-agent`

Установка идемпотентна не через apt (агент ставится не пакетом, а perl-скриптом) — проверяется наличие `/usr/bin/glpi-agent`; если бинарник уже есть на хосте, весь процесс скачивания/копирования/установки пропускается. Конфиг при этом обновляется всегда, независимо от того, ставился ли агент заново.

## Настраиваемые параметры

`inventory/group_vars/linux_hosts/vars.yml`:

```yaml
glpi_agent_version: "1.19"
glpi_agent_installer_url: "https://github.com/glpi-project/glpi-agent/releases/download/{{ glpi_agent_version }}/glpi-agent-{{ glpi_agent_version }}-linux-installer.pl"

glpi_agent_server: "192.168.56.10"
glpi_agent_httpd_trust:
  - "192.168.56.10"
  - "localhost"
glpi_agent_no_ssl_check: 1
```

При обновлении версии агента достаточно поменять `glpi_agent_version` — URL и путь к локальному файлу пересчитаются автоматически, но не забудь либо удалить старый `.pl`-файл из `files/glpi-agent-installer/` (чтобы скачался новый), либо положить новый файл под новым именем самостоятельно.

## Структура

```
files/
└── glpi-agent-installer/
└── glpi-agent-1.19-linux-installer.pl # скачивается автоматически либо кладётся вручную
roles/
├── glpi_agent_hardening/
│ ├── tasks/main.yml
│ ├── handlers/main.yml
│ └── templates/00-install.cfg.j2
└── glpi_agent_verify/
├── defaults/main.yml
├── tasks/
│ ├── main.yml
│ └── glpi_agent.yml
└── templates/
└── glpi_agent_report.j2
```

## Запуск

```bash
ansible-playbook playbooks/glpi_agent_hardening.yml
ansible-playbook playbooks/verify_glpi_agent.yml
```

## Что проверяет verify

- бинарник `/usr/bin/glpi-agent` установлен
- служба `glpi-agent.service` в состоянии `running`
- конфиг `00-install.cfg` содержит актуальные `server`/`httpd-trust`
- полное содержимое конфига — в отчёт для ручной сверки

## Результат

```
reports/test1/glpi_agent.txt
```
