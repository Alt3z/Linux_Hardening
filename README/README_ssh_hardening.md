# Linux Hardening — SSH Hardening Module

Ansible-проект для базового hardening'а Linux-хостов. Первый реализованный модуль — автоматизация SSH: генерация ключевой пары, раскатка публичного ключа на хосты, отключение входа под root и парольной аутентификации.

## Что делает playbook

1. **`ssh_keygen`** — генерирует ED25519-ключевую пару на control-ноде (там, где запускается Ansible), если её ещё нет. Идемпотентно: при повторных запусках ключ не пересоздаётся.
2. **`ssh_hardening`**:
   - раскатывает публичный ключ в `~/.ssh/authorized_keys` пользователя на каждом целевом хосте;
   - деплоит захардненный `/etc/ssh/sshd_config` (запрет входа под root, запрет парольной аутентификации, только вход по ключу и ряд других ограничений);
   - валидирует конфиг через `sshd -t` перед применением — если синтаксис некорректен, файл не подменяется и sshd не падает;
   - рестартует `sshd` только если конфиг реально изменился.

Хосты обрабатываются последовательно (`serial: 1`), а не параллельно — если что-то пойдёт не так на первом хосте, второй останется нетронутым для диагностики.

## Структура проекта

```
Linux_Hardening/
├── ansible.cfg # глобальный конфиг: инвентарь, vault, sudo
├── requirements.yml # зависимости — коллекции Ansible Galaxy
├── vault_pass # пароль от vault, НЕ коммитится
├── vault_pass.example # пример файла
├── inventory/
│ ├── hosts.yml # список хостов и топология групп
│ └── group_vars/
│   └── linux_hosts/
│     ├── vars.yml # переменные подключения (открытые)
│     └── vault.yml # секреты, зашифровано ansible-vault, не коммитится
│     └── vault.yml.example # пример файла
├── files/
│ └── ssh_keys/ # сюда генерируется ключевая пара, в .gitignore для нормальных проектов
├── roles/
│ ├── ssh_keygen/ # генерация ключа на control-ноде
│ │ ├── defaults/main.yml
│ │ └── tasks/main.yml
│ └── ssh_hardening/ # раскатка ключа + hardening sshd
│ ├── defaults/main.yml
│ ├── tasks/main.yml
│ ├── handlers/main.yml
│ └── templates/sshd_config.j2
└── playbooks/
└── ssh_hardening.yml # точка входа
```

## Почему group_vars лежит внутри inventory/, а не в корне

Ansible ищет каталоги `group_vars/` и `host_vars/` только в двух местах:

1. рядом с файлом **инвентаря**;
2. рядом с файлом **playbook**.

Если положить `group_vars/` в корень проекта (не рядом ни с тем, ни с другим), Ansible его молча не найдёт при запуске через `ansible-playbook` — переменные типа `ansible_user`, `ansible_password`, путь к ключу окажутся не определены, и подключение к хостам развалится с `Permission denied`, при этом команда `ansible-inventory --host <host>`, запущенная из корня проекта, переменные всё равно покажет — потому что она резолвит `group_vars` относительно текущей рабочей директории, а не относительно playbook.

## Что нужно заменить под себя

### 1. IP-адреса и имена хостов — `inventory/hosts.yml`

```yaml
all:
  children:
    linux_hosts:
      hosts:
        test1:
          ansible_host: 192.168.56.181
        test2:
          ansible_host: 192.168.56.182
```
Нужно поменять `test1`/`test2` и IP на свои. Хостов может быть больше двух — просто добавь их под `hosts:`.

### 2. Имя пользователя — `inventory/group_vars/linux_hosts/vars.yml`

```yaml
ansible_user: user
```
Заменить `user` на реальный логин, под которым Ansible будет заходить и от имени которого раскатывается SSH-ключ.

### 3. Пароли — `inventory/group_vars/linux_hosts/vault.yml`

Файл зашифрован, редактируется только через ansible-vault, руками открывать/редактировать нельзя:

```bash
ansible-vault edit inventory/group_vars/linux_hosts/vault.yml
```

Внутри:
```yaml
vault_ansible_password: "qwe"          # SSH-пароль на первый прогон, пока ключа ещё нет на хосте
vault_ansible_become_password: "qwe"   # пароль sudo
```

### 4. Пароль от vault — файл `vault_pass`

```bash
echo "твой_секретный_пароль_от_vault" > vault_pass
chmod 600 vault_pass
```
Этот файл — мастер-ключ ко всем секретам проекта. **Никогда не коммитить в реальных проектах**

### 5. Разрешённый пользователь в sshd_config — `roles/ssh_hardening/defaults/main.yml`

```yaml
ssh_hardening_allow_users: "user"
```
Заменить на имя реального пользователя — иначе после hardening'а вход по SSH не получит вообще никто, кроме указанного здесь.

### 6. Имя SSH-сервиса — `roles/ssh_hardening/handlers/main.yml`

```yaml
- name: Restart sshd
  ansible.builtin.service:
    name: ssh          # Debian/Ubuntu — "ssh", RHEL/CentOS/Rocky — "sshd"
    state: restarted
```

## Первый запуск с нуля

```bash
# 1. Установить зависимости-коллекции
ansible-galaxy collection install -r requirements.yml

# 2. Создать файл с паролем от vault (см. пункт 4 выше)
echo "мастер_пароль" > vault_pass
chmod 600 vault_pass

# 3. Создать/отредактировать секреты
ansible-vault edit inventory/group_vars/linux_hosts/vault.yml

# 4. Проверить, что инвентарь и переменные резолвятся корректно
ansible-inventory --host test1

# 5. Прогнать playbook
ansible-playbook playbooks/ssh_hardening.yml
```

## Проверка после прогона

Открой **новый** терминал (не тот, где работал Ansible) и проверь:

```bash
ssh -i files/ssh_keys/linux_hardening_key user@192.168.56.181   # должно пустить без пароля
ssh root@192.168.56.181                                          # должно быть отклонено
```

## Повторные запуски и идемпотентность

Playbook безопасно перезапускать сколько угодно раз — при уже применённой конфигурации ничего не изменится (`changed=0` во всех задачах). Ключ не перегенерируется, пока файл `files/ssh_keys/linux_hardening_key` существует на control-ноде; ротация ключа — отдельная ручная процедура (удаление файла + повторный прогон + чистка старого ключа из `authorized_keys` на хостах).
