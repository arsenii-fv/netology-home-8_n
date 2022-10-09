# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного playbook. 
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`. 

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.
```bash

all:
  hosts:
    kibana_h:
      ansible_connection: docker
    elasticsearch_h:
      ansible_connection: docker
kibana: 
   hosts:
     kibana_h:
elasticsearch:
   hosts:
     elasticsearch_h:
```
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
```bash

- name: Install Kibana
  hosts: kibana
  remote_user: root
  gather_facts: false
  tasks:
    - name: Create directrory for Kibana
      ansible.builtin.file:
        state: directory
        path: "{{ kibana_home }}"
        # mode: 0644
      tags: kibana
    - name: Upload .tar.gz file kibana containing binaries from local storage
      ansible.builtin.copy:
        src: "{{ kibana_tar_package }}"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        mode: 0755
      register: download_kibana_binaries
      until: download_kibana_binaries is succeeded
    - name: Extract Kibana in the installation directory
      # become: true
      ansible.builtin.unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      ansible.builtin.template:
        src: templates/kbn.yml
        dest: "{{ kibana_home }}/config/kibana.yml"
      tags: kibana
```
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
```bash

arsen@lite:~/08-ansible/08_ansible_02/playbook$ ansible-lint site.yml
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
WARNING  Listing 5 violation(s) that are fatal
risky-file-permissions: File permissions unset or incorrect.
site.yml:43 Task/Handler: Ensure installation dir exists

risky-file-permissions: File permissions unset or incorrect.
site.yml:61 Task/Handler: Export environment variables

risky-file-permissions: File permissions unset or incorrect.
site.yml:86 Task/Handler: Create directrory for Elasticsearch

risky-file-permissions: File permissions unset or incorrect.
site.yml:113 Task/Handler: Set environment Elastic

risky-file-permissions: File permissions unset or incorrect.
site.yml:151 Task/Handler: Set environment Kibana

You can skip specific rules or tags by adding them to your configuration file:
# .config/ansible-lint.yml
warn_list:  # or 'skip_list' to silence them completely
  - experimental  # all rules tagged as experimental

Finished with 0 failure(s), 5 warning(s) on 1 files.
```
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
```bash

arsen@lite:~/08-ansible/08_ansible_02/playbook$ ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Docker create and started] **********************************************************************************************************************

TASK [Centos1_kibana Docker] **************************************************************************************************************************
changed: [localhost]

TASK [Centos2_elastic Docker] *************************************************************************************************************************
changed: [localhost]

PLAY [Install Java] ***********************************************************************************************************************************

TASK [Set facts for Java 19 vars] *********************************************************************************************************************
ok: [kibana_h]
ok: [elasticsearch_h]

TASK [Upload .tar.gz file containing binaries from local storage] *************************************************************************************
fatal: [elasticsearch_h]: UNREACHABLE! => {"changed": false, "msg": "Failed to create temporary directory. In some cases, you may have been able to authenticate and did not have permissions on the target directory. Consider changing the remote tmp path in ansible.cfg to a path rooted in \"/tmp\", for more error information use -vvv. Failed command was: ( umask 77 && mkdir -p \"` echo /tmp/.ansible_${USER}/tmp `\"&& mkdir \"` echo /tmp/.ansible_${USER}/tmp/ansible-tmp-1665348152.3490233-14944-27351388849488 `\" && echo ansible-tmp-1665348152.3490233-14944-27351388849488=\"` echo /tmp/.ansible_${USER}/tmp/ansible-tmp-1665348152.3490233-14944-27351388849488 `\" ), exited with result 1", "unreachable": true}
fatal: [kibana_h]: UNREACHABLE! => {"changed": false, "msg": "Failed to create temporary directory. In some cases, you may have been able to authenticate and did not have permissions on the target directory. Consider changing the remote tmp path in ansible.cfg to a path rooted in \"/tmp\", for more error information use -vvv. Failed command was: ( umask 77 && mkdir -p \"` echo /tmp/.ansible_${USER}/tmp `\"&& mkdir \"` echo /tmp/.ansible_${USER}/tmp/ansible-tmp-1665348152.3411815-14943-211416905840257 `\" && echo ansible-tmp-1665348152.3411815-14943-211416905840257=\"` echo /tmp/.ansible_${USER}/tmp/ansible-tmp-1665348152.3411815-14943-211416905840257 `\" ), exited with result 1", "unreachable": true}

PLAY RECAP ********************************************************************************************************************************************
elasticsearch_h            : ok=1    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0   
kibana_h                   : ok=1    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
```bash
arsen@lite:~/08-ansible/08_ansible_02/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Docker create and started] **********************************************************************************************************************

TASK [Centos1_kibana Docker] **************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
-    "exists": false,
-    "running": false
+    "exists": true,
+    "running": true
 }

changed: [localhost]

TASK [Centos2_elastic Docker] *************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
-    "exists": false,
-    "running": false
+    "exists": true,
+    "running": true
 }

changed: [localhost]

PLAY [Install Java] ***********************************************************************************************************************************

TASK [Set facts for Java 19 vars] *********************************************************************************************************************
ok: [kibana_h]
ok: [elasticsearch_h]

TASK [Upload .tar.gz file containing binaries from local storage] *************************************************************************************
diff skipped: source file size is greater than 104448
changed: [kibana_h]
diff skipped: source file size is greater than 104448
changed: [elasticsearch_h]

TASK [Ensure installation dir exists] *****************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/19",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elasticsearch_h]
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/19",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana_h]

TASK [Extract java in the installation directory] *****************************************************************************************************
changed: [elasticsearch_h]
changed: [kibana_h]

TASK [Export environment variables] *******************************************************************************************************************
--- before
+++ after: /home/arsen/.ansible/tmp/ansible-local-298084cwywvc/tmp9nec64cr/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/19
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [kibana_h]
--- before
+++ after: /home/arsen/.ansible/tmp/ansible-local-298084cwywvc/tmp66zofpzo/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/19
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [elasticsearch_h]

PLAY [Install Elasticsearch] **************************************************************************************************************************

TASK [Create directrory for Elasticsearch] ************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/elastic/8.4.3",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elasticsearch_h]

TASK [Upload .tar.gz file elasticsearch containing binaries from local storage] ***********************************************************************
diff skipped: source file size is greater than 104448
changed: [elasticsearch_h]

TASK [Extract Elasticsearch in the installation directory] ********************************************************************************************
changed: [elasticsearch_h]

TASK [Set environment Elastic] ************************************************************************************************************************
--- before
+++ after: /home/arsen/.ansible/tmp/ansible-local-298084cwywvc/tmp2s0493se/elk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export ES_HOME=/opt/elastic/8.4.3
+export PATH=$PATH:$ES_HOME/bin
\ No newline at end of file

changed: [elasticsearch_h]

PLAY [Install Kibana] *********************************************************************************************************************************

TASK [Create directrory for Kibana] *******************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/kibana/8.4.3",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana_h]

TASK [Upload .tar.gz file kibana containing binaries from local storage] ******************************************************************************
diff skipped: source file size is greater than 104448
changed: [kibana_h]

TASK [Extract Kibana in the installation directory] ***************************************************************************************************
changed: [kibana_h]

TASK [Set environment Kibana] *************************************************************************************************************************
--- before
+++ after: /home/arsen/.ansible/tmp/ansible-local-298084cwywvc/tmpego9kt_0/kbn.sh.j2
@@ -0,0 +1,7 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export KIBANA_HOME=/opt/kibana/8.4.3
+export PATH=$PATH:$KIBANA_HOME/bin
+
+#elasticsearch.url: "http://elasticsearch_h:9200"
\ No newline at end of file

changed: [kibana_h]

PLAY [Docker destroy] *********************************************************************************************************************************

TASK [Centos1_kibana Docker destroy] ******************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
-    "exists": true,
-    "running": true
+    "exists": false,
+    "running": false
 }

changed: [localhost]

TASK [Centos2_elastic Docker destroy] *****************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
-    "exists": true,
-    "running": true
+    "exists": false,
+    "running": false
 }

changed: [localhost]

PLAY RECAP ********************************************************************************************************************************************
elasticsearch_h            : ok=9    changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
kibana_h                   : ok=9    changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=4    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.

## Необязательная часть

1. Приготовьте дополнительный хост для установки logstash.
2. Пропишите данный хост в `prod.yml` в новую группу `logstash`.
3. Дополните playbook ещё одним play, который будет исполнять установку logstash только на выделенный для него хост.
4. Все переменные для нового play определите в отдельный файл `group_vars/logstash/vars.yml`.
5. Logstash конфиг должен конфигурироваться в части ссылки на elasticsearch (можно взять, например его IP из facts или определить через vars).
6. Дополните README.md, протестируйте playbook, выложите новую версию в github. В ответ предоставьте ссылку на репозиторий.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
