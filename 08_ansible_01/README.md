# Домашнее задание "08.01 Введение в Ansible"

## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-playbook -i inventory/test.yml site.yml
TASK [Print fact]***********************************************************************
ok: [localhost] => {
    "msg": 12
```
2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.
```bash
TASK [Print fact] ***********************************************************************
ok: [localhost] => {
    "msg": "all default fact"
```
3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.


4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-playbook -i inventory/prod.yml site.yml 
TASK [Print fact] *************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
```
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-playbook -i inventory/prod.yml site.yml 
TASK [Print fact] *****************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
```
6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.

7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-vault encrypt group_vars/deb/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful

arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-vault encrypt group_vars/el/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] ***************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [centos7]
ok: [ubuntu]

TASK [Print OS] *********************************************************************************************************************
ok: [centos7] => {
    "msg": "Debian"
}
ok: [ubuntu] => {
    "msg": "Debian"
}

TASK [Print fact] *******************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP **************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-doc -t connection -l 
community.general.zone         Run tasks in a zone instance                                                                     
community.libvirt.libvirt_lxc  Run tasks in lxc containers via libvirt                                                          
community.libvirt.libvirt_qemu Run tasks on libvirt/qemu virtual machines                                                       
community.okd.oc               Execute tasks in pods running on OpenShift                                                       
community.vmware.vmware_tools  Execute tasks inside a VM via VMware Tools                                                       
community.zabbix.httpapi       Use httpapi to run command on network appliances                                                 
containers.podman.buildah      Interact with an existing buildah container                                                      
containers.podman.podman       Interact with an existing podman container                                                       
kubernetes.core.kubectl        Execute tasks in pods running on Kubernetes                                                      
local                          execute on controller                                                                            
paramiko_ssh                   Run tasks via python ssh (paramiko)                                                              
psrp                           Run tasks over Microsoft PowerShell Remoting Protocol                                            
ssh                            connect via SSH client binary                                                                    
winrm                          Run tasks over Microsoft's WinRM   
```
10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
```bash
  el:
    hosts:
      centos7:
        ansible_connection: local 
  deb:
    hosts:
      ubuntu:
        ansible_connection: local 
  local:
    hosts:
      local-host:
        ansible_connection: local
```
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] *************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************
ok: [centos7]
ok: [ubuntu]
ok: [local-host]

TASK [Print OS] *******************************************************************************************************************
ok: [centos7] => {
    "msg": "Debian"
}
ok: [ubuntu] => {
    "msg": "Debian"
}
ok: [local-host] => {
    "msg": "Debian"
}

TASK [Print fact] *****************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [local-host] => {
    "msg": "local default fact"
}

PLAY RECAP ************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
local-host                 : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.

## Необязательная часть

1. При помощи `ansible-vault` расшифруйте все зашифрованные файлы с переменными.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-vault decrypt group_vars/deb/examp.yml
Vault password: 
Decryption successful

arsen@lite:~/08-ansible/work8_1.1/iplaybook$ ansible-vault decrypt group_vars/el/examp.yml
Vault password: 
Decryption successful
```
2. Зашифруйте отдельное значение `PaSSw0rd` для переменной `some_fact` паролем `netology`. Добавьте полученное значение в `group_vars/all/exmp.yml`.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-vault encrypt_string --vault-id @prompt PaSSw0rd
New vault password (default): 
Confirm new vault password (default): 
Encryption successful
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          35633966386661386434343737663166613930646230323331353464393766363033376439323131
          3331623035653537303535356463363831616636376435340a396561626664383066623431376533
          30613631623736393531313431383766383430363632306531323965353834346562313863346162
          6637616663343137630a356263343466656363653737373936333837376630623334646266383136
          6432

---
  some_fact: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          35633966386661386434343737663166613930646230323331353464393766363033376439323131
          3331623035653537303535356463363831616636376435340a396561626664383066623431376533
          30613631623736393531313431383766383430363632306531323965353834346562313863346162
          6637616663343137630a356263343466656363653737373936333837376630623334646266383136
          6432
```
3. Запустите `playbook`, убедитесь, что для нужных хостов применился новый `fact`.
```bash
arsen@lite:~/08-ansible/work8_1.1/playbook$ ansible-playbook -i inventory/test.yml --vault-id @prompt site.yml 
Vault password (default): 

PLAY [Print os facts] *******************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [localhost]

TASK [Print OS] *************************************************************************************************************************
ok: [localhost] => {
    "msg": "Debian"
}

TASK [Print fact] ***********************************************************************************************************************
ok: [localhost] => {
    "msg": "PaSSw0rd"
}

PLAY RECAP ******************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
4. Добавьте новую группу хостов `fedora`, самостоятельно придумайте для неё переменную. В качестве образа можно использовать [этот](https://hub.docker.com/r/pycontribs/fedora).
5. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.
6. Все изменения должны быть зафиксированы и отправлены в вашей личный репозиторий.


### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.


