# Linux Hardening — SSH Key Generation

Отдельный, самостоятельный playbook для генерации SSH-ключевой пары, используемой всеми остальными модулями проекта. Выполняется вручную, отдельно от `ssh_hardening` — чтобы обычные прогоны hardening не трогали существующий ключ.

## Что делает

- Генерирует ED25519-ключевую пару **с passphrase** на control-ноде (не на целевых хостах)
- Идемпотентно: если файл ключа уже существует по указанному пути — не пересоздаёт его

## Где хранится ключ

```
files/ssh_keys/
├── linux_hardening_key # приватный, НЕ коммитится
└── linux_hardening_key.pub # публичный, тоже не коммитится (генерируется заново при необходимости)
```

## Passphrase

Задаётся в vault, не в открытом виде:
```bash
ansible-vault edit inventory/group_vars/linux_hosts/vault.yml
```
```yaml
vault_ssh_key_passphrase: "..."
```

## Зависимости

Генерация ключа с passphrase требует Python-пакет `bcrypt` на control-ноде:
```bash
pip install bcrypt --break-system-packages
```
Без него — ошибка `Need bcrypt module`. Подробности — [SYSTEM_REQUIREMENTS.md](SYSTEM_REQUIREMENTS.md).

## Запуск

```bash
ansible-playbook playbooks/generate_ssh_key.yml --ask-vault-pass
```

## Использование ключа после генерации — через ssh-agent

Так как ключ с passphrase, для работы Ansible (и обычного `ssh`) без ручного ввода пароля на каждое подключение — добавь ключ в agent один раз за сессию терминала:

```bash
eval "$(ssh-agent -s)"
ssh-add files/ssh_keys/linux_hardening_key
```

Спросит passphrase один раз, дальше все команды в этом терминале используют ключ автоматически.

Чтобы не делать это вручную при каждом новом терминале, можно добавить в `~/.bashrc`:
```bash
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)" > /dev/null
fi
ssh-add -l > /dev/null 2>&1 || ssh-add ~/Linux_Hardening/files/ssh_keys/linux_hardening_key
```

## ⚠️ Важно про ротацию ключа

**Никогда не удаляй существующий ключ до того, как новый подтверждённо работает на всех хостах.** Если старый ключ удалён локально, а `authorized_keys` на хостах ещё не обновлён Ansible'ом (например, `PasswordAuthentication no` уже применён, а прогнать `ssh_hardening` с новым ключом ещё не успел) — можно потерять доступ и по ключу, и по паролю одновременно. Восстановление в такой ситуации — только через консоль гипервизора (`VBoxManage`/VirtualBox GUI) или `vagrant ssh`, если VM поднята через Vagrant.

Безопасный порядок ротации:
1. Сгенерировать новый ключ под другим именем, не трогая старый файл
2. Раскатать новый публичный ключ через `ssh_hardening` (модуль `authorized_key` только добавляет, не удаляет существующие записи)
3. Вручную проверить вход по новому ключу на всех хостах
4. Только после подтверждения — переключить `ansible_ssh_private_key_file` на новый ключ и отдельно удалить старую запись из `authorized_keys`
