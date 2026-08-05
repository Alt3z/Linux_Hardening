# Linux Hardening — Wazuh Agent

Устанавливает и настраивает Wazuh-агент, подключающий хост к Wazuh-менеджеру для сбора событий безопасности.

## Что делает

**Установка** (`wazuh_agent_hardening`):
- скачивает `.deb`-пакет нужной версии один раз на control-ноду в `files/wazuh-agent-installer/` (не качается заново, если файл уже есть локально — устойчиво к сетевым проблемам на самих хостах, та же схема, что и у GLPI)
- копирует пакет на целевой хост через Ansible-канал
- устанавливает через `apt: deb:`, передавая `WAZUH_MANAGER` и `WAZUH_AGENT_NAME` переменными окружения — постинсталл-скрипт пакета сам генерирует `/var/ossec/etc/ossec.conf` с этими значениями
- включает и запускает службу

Идемпотентность обеспечена вручную через `dpkg-query -W` — обычная проверка apt здесь не подходит, так как пакет ставится из локального файла, а не из репозитория.

## Настраиваемые параметры

`inventory/group_vars/linux_hosts/vars.yml`:
```yaml
wazuh_manager_ip: "192.168.56.105"
wazuh_agent_version: "4.7.5-1"
wazuh_agent_deb_url: "https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_{{ wazuh_agent_version }}_amd64.deb"
```

При обновлении версии — поменять `wazuh_agent_version` и либо удалить старый `.deb` из `files/wazuh-agent-installer/`, чтобы скачался новый, либо положить новый файл вручную.

## Структура

```
files/
└── wazuh-agent-installer/
└── wazuh-agent_4.7.5-1_amd64.deb # скачивается автоматически либо кладётся вручную
roles/
├── wazuh_agent_hardening/
│ ├── tasks/main.yml
│ └── handlers/main.yml
└── wazuh_agent_verify/
├── defaults/main.yml
├── tasks/
│ ├── main.yml
│ └── wazuh_agent.yml
└── templates/
└── wazuh_agent_report.j2
```

## Запуск

```bash
ansible-playbook playbooks/wazuh_agent_hardening.yml
ansible-playbook playbooks/verify_wazuh_agent.yml
```

## Что проверяет verify

- служба `wazuh-agent.service` в состоянии `running`
- `ossec.conf` содержит правильный адрес менеджера
- статус подключения к менеджеру через штатную утилиту `agent-control -i 000` (локальная проверка с самого агента, без необходимости доступа к серверу Wazuh с control-ноды)

## Результат

```
reports/test1/wazuh_agent.txt
```

