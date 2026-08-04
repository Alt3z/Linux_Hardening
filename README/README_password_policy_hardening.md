# Linux Hardening — Password Policy Hardening

Настраивает `/etc/login.defs` и `pam_pwquality` в `/etc/pam.d/common-password` по значениям из `group_vars`.

## Что делает

**`login.defs`** (через `lineinfile`, идемпотентно):
- `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE`, `ENCRYPT_METHOD` — приводятся к значениям из `login_defs_target`

**`common-password`**:
- ставит `libpam-pwquality`, если ещё не установлен
- строит строку `password requisite pam_pwquality.so retry=... minlen=... ucredit=... lcredit=... dcredit=... ocredit=...` из `pam_pwquality_target`
- если строка с `pam_pwquality.so` уже есть — заменяет её целиком; если нет — вставляет **перед** строкой `pam_unix.so` (порядок в PAM-стеке критичен: проверка сложности должна идти раньше установки пароля)

## Целевые значения

`inventory/group_vars/linux_hosts/vars.yml`:
```yaml
login_defs_target:
  PASS_MAX_DAYS: 90
  PASS_MIN_DAYS: 7
  PASS_WARN_AGE: 14
  ENCRYPT_METHOD: SHA512

pam_pwquality_target:
  control: requisite
  retry: 3
  minlen: 12
  ucredit: -1
  lcredit: -1
  dcredit: -1
  ocredit: -1
```

Эти значения независимы от `login_defs_expected`/`pam_pwquality_expected` в `password_policy_verify` (там заданы допустимые диапазоны для проверки, здесь — конкретные целевые числа для накатки). При изменении политики проверь оба места.

## Известное ограничение

Если параметр в `login.defs` сейчас закомментирован (`#PASS_MAX_DAYS 99999`), `lineinfile` не трогает закомментированную строку и добавляет новую активную в конец файла — работает корректно (последнее вхождение побеждает), но выглядит не идеально аккуратно в файле.

## Запуск

```bash
ansible-playbook playbooks/password_policy_hardening.yml
ansible-playbook playbooks/password_policy_hardening.yml --tags login_defs
ansible-playbook playbooks/password_policy_hardening.yml --tags pam_common_password
```

## Проверка результата

```bash
ansible-playbook playbooks/verify_password_policy.yml
```
Подробности — [README_password_policy_verify.md](README_password_policy_verify.md).
