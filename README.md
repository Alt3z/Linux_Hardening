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

Требования к окружению — [system_requirements](https://github.com/Alt3z/Linux_Hardening/blob/main/system_requirements).

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
```

## Быстрый старт

```bash
# см. system_requirements для полного списка пакетов на control-ноде
ansible-galaxy collection install -r requirements.yml
echo "пароль_от_vault" > vault_pass && chmod 600 vault_pass
ansible-vault edit inventory/group_vars/linux_hosts/vault.yml

# SSH
ansible-playbook playbooks/generate_ssh_key.yml --ask-vault-pass
eval "$(ssh-agent -s)" && ssh-add files/ssh_keys/linux_hardening_key
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
```

## Отчёты

Каждая `*_verify`-роль пишет текстовый отчёт на control-ноду, не в терминал: 

```
reports/<host>/<модуль>.txt
```
