# Домашнее задание к занятию "08.03 Использование Yandex Cloud"

## Подготовка к выполнению

1. Подготовьте в Yandex Cloud три хоста: для `clickhouse`, для `vector` и для `lighthouse`.

Ссылка на репозиторий LightHouse: https://github.com/VKCOM/lighthouse

## Основная часть

1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает lighthouse.
```bash

- name: Install lighthouse
  hosts: lighthouse
  tasks:
    - name: "Ensure installation dir lighthouse"
      become: true
      # become_user: root
      ansible.builtin.file:
        path: "{{ item }}"
        state: directory
        mode: 0777
      with_items:
        - "{{ lightdir }}"
        - "/opt/lighthouse"

    - name: Download lighthouse
      # command: chdir=/opt/lighthouse
      # command: ""
      ansible.builtin.get_url:
        url: "https://github.com/VKCOM/lighthouse/archive/refs/heads/master.zip"
        dest: "{{ lightdir }}/master.zip"
        mode: 0644
        force: false

    - name: Extract lighthouse
      become: true
      ansible.builtin.unarchive:
        # remote_src: yes
        src: "{{ lightdir }}/master.zip"
        dest: "/opt/lighthouse"
        # extra_opts: [--strip-components=2]
    #     creates: "/opt/lighthouse/bin"
    # - name: Start lighthouse
    #   command: "/opt/lighthouse/node-v18.6.0-linux-arm64/bin/node lighthouse-cli https://webdevblog.ru/"
    #   #changed_when: false
    - name: Ensure nginx
      become: true
      ansible.builtin.apt:
        name: nginx
        state: stable

    - name: Add domain
      become: true
      ansible.builtin.template:
        # remote_src: yes
        src: default.t
        dest: /etc/nginx/sites-enabled/default
        mode: 0644

    - name: Start nginx
      become: true
      ansible.builtin.service:
        name: nginx
        state: restarted

```
2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.
3. Tasks должны: скачать статику lighthouse, установить nginx или любой другой webserver, настроить его конфиг для открытия lighthouse, запустить webserver.
4. Приготовьте свой собственный inventory файл `prod.yml`.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
