# Домашнее задание к занятию "08.03 Работа с Roles"

1. Создать в старой версии playbook файл `requirements.yml` и заполнить его следующим содержимым:
   ```yaml
   ---
     - src: git@github.com:netology-code/mnt-homeworks-ansible.git
       scm: git
       version: "1.0.1"
       name: java 
   ```
2. При помощи `ansible-galaxy` скачать себе эту роль. Запустите  `molecule test`, посмотрите на вывод команды.
```bash
admin1@pc:~/mnt-homework/08-ansible-03-role/playbook$ ansible-galaxy install -r roles/requirements.yml -p roles
- extracting java to /home/admin1/mnt-homework/08-ansible-03-role/playbook/roles/java
- java (1.0.1) was installed successfully
admin1@pc:~/mnt-homework/08-ansible-03-role/playbook/roles/java$ molecule test
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - image: docker.io/pycontribs/centos:8
    name: centos8
    pre_build_image: true
  - image: docker.io/pycontribs/ubuntu
    name: centos7
    pre_build_image: true
  - image: docker.io/pycontribs/ubuntu:latest
    name: ubuntu
    pre_build_image: true
provisioner:
  inventory:
    group_vars:
      all:
        java_home: /opt/jdk/{{ jdk_folder }}
        jdk_distr_name: openjdk-11-linux.tar.gz
        jdk_distr_type: remote
        jdk_folder: '{{ jdk_distr_name.split(''-'')[:2] | join(''-'')  }}'
        jdk_url: '{{ openjdk[11] }}'
  name: ansible
verifier:
  name: ansible

CRITICAL Failed to pre-validate.

{'driver': [{'name': ['unallowed value docker']}]}
```
3. Перейдите в каталог с ролью elastic-role и создайте сценарий тестирования по умолчаню при помощи `molecule init scenario --driver-name docker`.
```bash
admin1@pc:~/mnt-homework/08-ansible-03-role/playbook/roles/elastic$ molecule init scenario -r elastic elastic
INFO     Initializing new scenario elastic...
INFO     Initialized scenario in /home/admin1/mnt-homework/08-ansible-03-role/playbook/roles/elastic/molecule/elastic successfully.
```
4. Добавьте несколько разных дистрибутивов (centos:8, ubuntu:latest) для инстансов и протестируйте роль, исправьте найденные ошибки, если они есть.
```bash
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: centos:8
    image: docker.io/pycontribs/centos:8
    pre_build_image: true
  - name: ubuntu
    image: docker.io/pycontribs/ubuntu
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
```
```bash
admin1@pc:~/mnt-homework/08-ansible-03-role/playbook/roles/elastic$ molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun with role_name_check=0...
INFO     Set ANSIBLE_LIBRARY=/home/admin1/.cache/ansible-compat/986051/modules:/home/admin1/.ansible/plugins/modules:/usr/share/ansible/plugins/modules
INFO     Set ANSIBLE_COLLECTIONS_PATH=/home/admin1/.cache/ansible-compat/986051/collections:/home/admin1/.ansible/collections:/usr/share/ansible/collections
INFO     Set ANSIBLE_ROLES_PATH=/home/admin1/.cache/ansible-compat/986051/roles:/home/admin1/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
INFO     Using /home/admin1/.cache/ansible-compat/986051/roles/netology.kibana symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Running default > dependency
INFO     Running ansible-galaxy collection install -v --force --pre community.docker:>=3.0.0-a2
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > lint
INFO     Lint is disabled.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'docker'
[WARNING]: Collection community.docker does not support Ansible version 2.10.8
...
PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory

```
5. Создайте новый каталог с ролью при помощи `molecule init role --driver-name docker kibana-role`. Можете использовать другой драйвер, который более удобен вам.
```bash
admin1@pc:~/mnt-homework/08-ansible-03-role/playbook/roles/kibana$ molecule init role 'netology.kibana' -d docker
INFO     Initializing new role kibana...
Invalid -W option ignored: unknown warning category: 'CryptographyDeprecationWarning'
No config file found; using defaults
- Role kibana was created successfully
Invalid -W option ignored: unknown warning category: 'CryptographyDeprecationWarning'
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | CHANGED => {"backup": "","changed": true,"msg": "line added"}
INFO     Initialized role in /home/admin1/mnt-homework/08-ansible-03-role/playbook/roles/kibana/kibana successfully.
```
6. На основе tasks из старого playbook заполните новую role. Разнесите переменные между `vars` и `default`. Проведите тестирование на разных дистрибитивах (centos:7, centos:8, ubuntu).
```bash
---
  kibana_version: "8.3.3"
  kibana_home: "/opt/kibana/{{ kibana_version }}" 
```
8. Добавьте roles в `requirements.yml` в playbook.
```bash
- src: git@github.com:kofe88/elastic-role.git
  scm: git
  version: "1.0.1"
  name: elastic-role

- src: git@github.com:kofe88/kibana_role.git
  scm: git
  version: "1.0.1"
  name: kibana_role
```
9. Переработайте playbook на использование roles
```bash
---
- name: Install Java
  hosts: all
  roles:
    - java
- name: Install Elastic
  hosts: all
  roles:
    - elastic
- name: Install Kibana
  hosts: all
  roles:
    - kibana
```
[elastic](https://github.com/TedFak/elastic)
[kibana](https://github.com/TedFak/kibana)
