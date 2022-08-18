# Домашнее задание к занятию "08.01 Введение в Ansible"

## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.
```bash
TASK [Print fact] **********************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}
```
2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.
```bash
admin1@pc:~/mnt-homework/08-ansible-01-base$ cat playbook/group_vars/all/examp.yml
---
  some_fact: all default fact
```
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.
```bash
TASK [Print fact] **********************************************************************************************************************************************************************************************************************
ok: [server2] => {
    "msg": "el"
}
ok: [server1] => {
    "msg": "deb"
}

```
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.
```bash
TASK [Print fact] **********************************************************************************************************************************************************************************************************************
ok: [server2] => {
    "msg": "el default fact"
}
ok: [server1] => {
    "msg": "deb default fact"
}

``` 
6. Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
```bash
PLAY RECAP *****************************************************************************************************************************************************************************************************************************
server1                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server2                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
```bash
admin1@pc:~/mnt-homework/08-ansible-01-base$ ansible-vault  create playbook/group_vars/el/
New Vault password: 
Confirm New Vault password: 
```
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
```bash
admin1@pc:~/mnt-homework/08-ansible-01-base$ ansible-playbook -i playbook/inventory/prod.yml playbook/site.yml -v
No config file found; using defaults

PLAY [Print os facts] ******************************************************************************************************************************************************************************************************************
ERROR! Attempting to decrypt but no vault secrets found
```
```bash
admin1@pc:~/mnt-homework/08-ansible-01-base$ ansible-playbook -i playbook/inventory/prod.yml playbook/site.yml --ask-vault-pass
Vault password: 
```
9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
```bash
admin1@pc:~/mnt-homework/08-ansible-01-base$ ansible-doc  -t connection -l
...                                                                                                                                                       
local                          execute on controller                                                                                                                                                                               
...
```
10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
```bash
  all:
    hosts:
      localhost:
        ansible_connection: local
```
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
```bash
TASK [Print fact] **********************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [server2] => {
    "msg": "el default fact"
}
ok: [server1] => {
    "msg": "deb default fact"
}
```


