# Linux Hardening

Ansible-проект для базовой безопасной настройки (hardening) Linux-хостов.

Тестовое окружение в этом проекте: 2 виртуалки (`test1`, `test2`), поднятые через [Fast_VM_Deployment](https://github.com/Alt3z/Fast_VM_Deployment) (Vagrant + Ansible), сеть `192.168.56.0/24`.

## Модули

| Модуль | Настройка | Проверка |
|---|---|---|
| Пакеты | [README_install_packages.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_install_packages.md) | [README_packages_verify.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_packages_verify.md) |
| SSH-ключ | [README_generate_ssh_key.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_generate_ssh_key.md) | — |
| SSH | [README_ssh_hardening.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_ssh_hardening.md) | [README_ssh_verify.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_ssh_verify.md) |
| Политика паролей | [README_password_policy_hardening.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_password_policy_hardening.md) | [README_password_policy_verify.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_password_policy_verify.md) |
| Fail2ban | [README_fail2ban_hardening.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_fail2ban_hardening.md) | [README_fail2ban_verify.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_fail2ban_verify.md) |
| Службы (iptables, Kaspersky) | — | [README_services_verify.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_services_verify.md) |
| GLPI-агент | [README_glpi_agent.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_glpi_agent.md) | входит туда же |
| Zabbix-агент | [README_zabbix_agent.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_zabbix_agent.md) | входит туда же |
| Wazuh-агент | [README_wazuh_agent.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_wazuh_agent.md) | входит туда же |
| Сводный отчёт | — | [README_aggregate_reports.md](https://github.com/Alt3z/Linux_Hardening/blob/main/README/README_aggregate_reports.md) |

Требования к окружению — [system_requirements.md](https://github.com/Alt3z/Linux_Hardening/blob/main/system_requirements.md).

## Структура проекта

```
Linux_Hardening/
├── ansible.cfg
├── requirements.yml
├── vault_pass # НЕ комитится в реальных проектах
├── vault_pass.example # информация о файле vault_pass
├── system_requirements
├── inventory/
│ ├── hosts.yml
│ └── group_vars/
│ └── linux_hosts/
│ ├── vars.yml
│ └── vault.yml # зашифровано, НЕ комитится в реальных проектах
│ └── vault.yml.example # информация о файле vault.yml
├── files/
│ └── ssh_keys/ # генерируется, в .gitignore (здесь должен быть открытый и закрытый ключи, примеры есть внутри (НЕ комитится в реальных проектах))
│ ├── glpi-agent-installer/   # скачивается автоматически или кладётся вручную
│ └── wazuh-agent-installer/  # аналогично
├── reports/ # генерируется, в .gitignore
├── roles/
│ ├── ssh_keygen/
│ ├── ssh_hardening/
│ ├── ssh_verify/
│ ├── password_policy_hardening/
│ ├── password_policy_verify/
│ ├── fail2ban_hardening/
│ ├── fail2ban_verify/
│ ├── packages_verify/
│ └── services_verify/
│ ├── sysctl_hardening/
│ ├── sysctl_verify/
│ ├── glpi_agent_hardening/
│ ├── glpi_agent_verify/
│ ├── zabbix_agent_hardening/
│ ├── zabbix_agent_verify/
│ ├── wazuh_agent_hardening/
│ ├── wazuh_agent_verify/
│ └── aggregate_reports/
└── playbooks/
├── install_packages.yml
├── verify_packages.yml
├── generate_ssh_key.yml
├── ssh_hardening.yml
├── verify_ssh_hardening.yml
├── password_policy_hardening.yml
├── verify_password_policy.yml
├── fail2ban_hardening.yml
├── verify_fail2ban.yml
└── verify_services.yml
├── sysctl_hardening.yml
├── verify_sysctl.yml
├── glpi_agent_hardening.yml
├── verify_glpi_agent.yml
├── zabbix_agent_hardening.yml
├── verify_zabbix_agent.yml
├── wazuh_agent_hardening.yml
├── verify_wazuh_agent.yml
├── aggregate_reports.yml
├── hardening_all.yml.yml # вся настройка сразу
├── verify_all.yml    # все проверки сразу
└── all.yml            # site.yml + verify_all.yml
```

## Старт

1. Установка пакетов из system_requirements

2.1. Vault — пароль и секреты

```
echo "пароль_от_vault" > vault_pass && chmod 600 vault_pass
ansible-vault edit inventory/group_vars/linux_hosts/vault.yml
```

2.2. Vault Внутри: 

```
vault_ansible_password: "..."
vault_ansible_become_password: "..."
vault_ssh_key_passphrase: "..."
```

3. SSH-ключ
3.1. Автоматическое создание:
```
ansible-playbook playbooks/generate_ssh_key.yml --ask-vault-pass
```
3.2. Либо положить руками и выдать права:
```
chmod 600 files/ssh_keys/linux_hardening_key
chmod 644 files/ssh_keys/linux_hardening_key.pub
```
4. ssh-agent, если ключ с паролем

```
eval "$(ssh-agent -s)"
ssh-add files/ssh_keys/linux_hardening_key
ssh-add -l   # проверка
```

После работы:
```
ssh-agent -k
```

5. Инвентарь и переменные

```
inventory/hosts.yml — IP-адреса test1/test2.
inventory/group_vars/linux_hosts/vars.yml — проверить и поправить под реальные значения:
ssh_hardening_extra_allow_users: [...]
zabbix_server_ip: "..."
glpi_agent_server: "..."
wazuh_manager_ip: "..."
```

## Плейбуки

```bash
# SSH
ansible-playbook playbooks/ssh_hardening.yml
ansible-playbook playbooks/verify_ssh_hardening.yml

# пакеты на хостах
ansible-playbook playbooks/install_packages.yml
ansible-playbook playbooks/verify_packages.yml

# политика паролей
ansible-playbook playbooks/password_policy_hardening.yml
ansible-playbook playbooks/verify_password_policy.yml

# fail2ban
ansible-playbook playbooks/fail2ban_hardening.yml
ansible-playbook playbooks/verify_fail2ban.yml

# службы (iptables/Kaspersky)
ansible-playbook playbooks/verify_services.yml

# GLPI / Zabbix / Wazuh агенты
ansible-playbook playbooks/glpi_agent_hardening.yml
ansible-playbook playbooks/verify_glpi_agent.yml
ansible-playbook playbooks/zabbix_agent_hardening.yml
ansible-playbook playbooks/verify_zabbix_agent.yml
ansible-playbook playbooks/wazuh_agent_hardening.yml
ansible-playbook playbooks/verify_wazuh_agent.yml

# сводный отчёт по каждому хосту
ansible-playbook playbooks/aggregate_reports.yml

ansible-playbook playbooks/site.yml         # вся настройка
ansible-playbook playbooks/verify_all.yml    # все проверки + сборка отчёта
ansible-playbook playbooks/all.yml           # всё сразу
```

## Отчёты

Каждая `*_verify`-роль пишет текстовый отчёт на control-ноду, не в терминал: 

```
reports/<host>/<модуль>.txt
```
