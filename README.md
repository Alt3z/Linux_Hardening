# Linux Hardening

Ansible-проект для базовой безопасной настройки (hardening) Linux-хостов.

Тестовое окружение в этом проекте: 2 виртуалки (`test1`, `test2`), поднятые через [Fast_VM_Deployment](https://github.com/Alt3z/Fast_VM_Deployment) (Vagrant + Ansible), сеть `192.168.56.0/24`.

## Что реализовано сейчас

**SSH hardening** — полностью готовый модуль:
- автогенерация SSH-ключевой пары на control-ноде
- раскатка публичного ключа на хосты
- отключение входа под root
- отключение парольной аутентификации, вход только по ключу
- автоматическая проверка (`ssh_verify`), что всё применилось и реально работает

Подробности — [README_ssh_hardening.md](README/README_ssh_hardening.md) и [README_ssh_verify.md](README_ssh_verify.md).

## Быстрый старт

```bash
ansible-galaxy collection install -r requirements.yml
echo "пароль_от_vault" > vault_pass && chmod 600 vault_pass
ansible-vault edit inventory/group_vars/linux_hosts/vault.yml
ansible-playbook playbooks/ssh_hardening.yml
ansible-playbook playbooks/verify_ssh_hardening.yml
```
