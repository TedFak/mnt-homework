# Домашнее задание к занятию "08.02 Работа с Playbook"

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.
```bash
---
all:
  hosts:
    server1:
      ansible_host: 192.168.56.11
    server2:
      ansible_host: 192.168.56.12
  vars:
      ansible_connection: ssh
      ansible_user: vagrant
      ansible_password: vagrant
```
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
```bash
- name: Install Kibana
  hosts: server2
  tasks:
    - name: Upload .tar.gz file containing binaries from local storage
      copy:
        src: "{{ kibana_package }}"
        dest: "/tmp/kibana-8.3.3-linux-x86_64.tar.gz"
      register: download_kibana_binaries
      until: download_kibana_binaries is succeeded
      tags: kibana
    - name: Ensure installation dir exists
      become: true
      file:
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/kibana-8.3.3-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - kibana
    - name: Set environment kibana
      become: true
      template:
        src: templates/kibana.sh.j2
        dest: /etc/profile.d/kibana.sh
        mode: 0644
      tags: kibana
```
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
```bash
admin1@pc:~/mnt-homework/08-ansible-02-playbook$ ansible-lint playbook/site.yml
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: playbook/site.yml
WARNING  Listing 8 violation(s) that are fatal
risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:5 Task/Handler: Upload .tar.gz file containing binaries from local storage

risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:12 Task/Handler: Ensure installation dir exists

risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:28 Task/Handler: Export environment variables

risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:37 Task/Handler: Upload .tar.gz file containing binaries from local storage

risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:44 Task/Handler: Ensure installation dir exists

risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:60 Task/Handler: Set environment Elastic

risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:69 Task/Handler: Upload .tar.gz file containing binaries from local storage

risky-file-permissions: File permissions unset or incorrect
playbook/site.yml:76 Task/Handler: Ensure installation dir exists

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - experimental  # all rules tagged as experimental

Finished with 0 failure(s), 8 warning(s) on 1 files.
```
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
```bash
admin1@pc:~/mnt-homework/08-ansible-02-playbook$ ansible-playbook -i playbook/inventory/hosts.yml playbook/site.yml --check
[WARNING]: Found both group and host with same name: server2
[WARNING]: Found both group and host with same name: server1
...
PLAY RECAP *****************************************************************************************************************************************************************************************************************************
server1                    : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
server2                    : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
```
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
```bash
admin1@pc:~/mnt-homework/08-ansible-02-playbook$ ansible-playbook -i playbook/inventory/hosts.yml playbook/site.yml --diff
[WARNING]: Found both group and host with same name: server1
[WARNING]: Found both group and host with same name: server2
...
PLAY RECAP *****************************************************************************************************************************************************************************************************************************
server1                    : ok=9    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
server2                    : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
```