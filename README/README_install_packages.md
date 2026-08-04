# Linux Hardening — Install Packages

Playbook для установки пакетов, необходимых остальным модулям проекта. Без отчёта — просто устанавливает и всё, проверка отдельно через `packages_verify`.

## Что делает

Обновляет apt-кэш и ставит:
- `libpam-pwquality` — для модуля политики паролей
- `rsyslog` — логирование, включая `/var/log/auth.log`
- `fail2ban`
- `iptables`
- `iptables-persistent`

Для `iptables-persistent` заранее форсируется ответ на debconf-диалог (не сохранять текущие правила автоматически при установке), чтобы установка не зависала в интерактивном режиме.

## Запуск

```bash
ansible-playbook playbooks/install_packages.yml
```

## Проверка результата

```bash
ansible-playbook playbooks/verify_packages.yml
cat reports/test1/packages.txt
```
Подробности — [README_packages_verify.md](README_packages_verify.md).

## Добавление новых пакетов

Список задан прямо в `playbooks/install_packages.yml`, задача `Install required packages`. Добавь имя пакета в список — и не забудь продублировать его в `roles/packages_verify/defaults/main.yml` (`packages_expected`), иначе verify не будет знать о новом пакете.


