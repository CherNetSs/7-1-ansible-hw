# Домашнее задание к занятию «Ansible»

Выполнил: Артем Чернецов

---

## Задание 1

### Playbook 1 — скачивание и распаковка архива

Файл: `playbook_archive.yml`

```yaml
---
- name: Download and unpack archive
  hosts: localhost
  become: true

  tasks:
    - name: Create directory
      ansible.builtin.file:
        path: /opt/example
        state: directory
        mode: '0755'

    - name: Download archive
      ansible.builtin.get_url:
        url: https://github.com/git/git/archive/refs/heads/master.zip
        dest: /tmp/git-master.zip

    - name: Unpack archive
      ansible.builtin.unarchive:
        src: /tmp/git-master.zip
        dest: /opt/example
        remote_src: true
```

### Проверка синтаксиса

```bash
ansible-playbook playbook_archive.yml --syntax-check
```

### Запуск playbook

```bash
ansible-playbook playbook_archive.yml -c local --ask-become-pass
```

### Результат выполнения

```text
PLAY RECAP ****************************************************************

localhost : ok=4 changed=3 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

## Playbook 2 — установка и запуск tuned

Файл: `playbook_tuned.yml`

```yaml
---
- name: Install and enable tuned
  hosts: localhost
  become: true

  tasks:
    - name: Install tuned
      ansible.builtin.package:
        name: tuned
        state: present

    - name: Start and enable tuned
      ansible.builtin.service:
        name: tuned
        state: started
        enabled: true
```

### Запуск playbook

```bash
ansible-playbook playbook_tuned.yml -c local --ask-become-pass
```

### Результат выполнения

```text
PLAY RECAP ****************************************************************

localhost : ok=3 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

## Playbook 3 — изменение MOTD

Файл: `playbook_motd.yml`

```yaml
---
- name: Change system MOTD
  hosts: localhost
  become: true

  vars:
    motd_text: "Welcome to my Ansible-managed server!"

  tasks:
    - name: Set MOTD
      ansible.builtin.copy:
        content: "{{ motd_text }}\n"
        dest: /etc/motd
        mode: '0644'
```

### Запуск playbook

```bash
ansible-playbook playbook_motd.yml -c local --ask-become-pass
```

### Результат выполнения

```text
PLAY RECAP ****************************************************************

localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```
## Задание 2

### Модифицированный playbook MOTD

Файл: `playbook_motd.yml`

```yaml id="q0j2r8"
---
- name: Change system MOTD
  hosts: localhost
  become: true

  vars:
    admin_message: "Have a good day, system administrator!"

  tasks:
    - name: Set MOTD
      ansible.builtin.copy:
        dest: /etc/motd
        mode: '0644'
        content: |
          Hostname: {{ ansible_hostname }}
          IP address: {{ ansible_default_ipv4.address }}

          {{ admin_message }}
```

### Проверка синтаксиса

```bash id="9d4v8l"
ansible-playbook playbook_motd.yml --syntax-check
```

### Запуск playbook

```bash id="o4bxbe"
ansible-playbook playbook_motd.yml -c local --ask-become-pass
```

### Результат выполнения

```text id="jlwm0w"
PLAY RECAP ****************************************************************

localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

### Проверка результата

```bash id="jlwm8c"
cat /etc/motd
```

Пример вывода:

```text id="jlwm5w"
Hostname: chernez-Virtual-Machine
IP address: 192.168.1.145

Have a good day, system administrator!
```
# Задание 3

## Создание роли для установки и настройки Apache

Была создана роль `apache_info`, выполняющая:

* установку Apache;
* настройку веб-страницы с информацией о системе;
* запуск и добавление Apache в автозагрузку;
* проверку доступности сайта через модуль `uri`.

---

# Структура проекта

```text
ansible-hw/
├── playbook_apache_role.yml
└── roles/
    └── apache_info/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        └── templates/
            ├── apache-default.conf.j2
            └── index.html.j2
```

---

# Playbook

Файл: `playbook_apache_role.yml`

```yaml
---
- name: Install Apache and publish host info
  hosts: localhost
  become: true

  roles:
    - apache_info
```

---

# Tasks роли

Файл: `roles/apache_info/tasks/main.yml`

```yaml
---
- name: Install Apache
  ansible.builtin.package:
    name: apache2
    state: present

- name: Configure Apache default site
  ansible.builtin.template:
    src: apache-default.conf.j2
    dest: /etc/apache2/sites-available/000-default.conf
    mode: '0644'
  notify: Restart Apache

- name: Create index.html with system information
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    mode: '0644'

- name: Allow HTTP through UFW
  community.general.ufw:
    rule: allow
    port: '80'
    proto: tcp
  ignore_errors: true

- name: Start and enable Apache
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: true

- name: Check website availability
  ansible.builtin.uri:
    url: http://localhost
    status_code: 200
```

---

# Handler

Файл: `roles/apache_info/handlers/main.yml`

```yaml
---
- name: Restart Apache
  ansible.builtin.service:
    name: apache2
    state: restarted
```

---

# Шаблон Apache

Файл: `roles/apache_info/templates/apache-default.conf.j2`

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

---

# Шаблон index.html

Файл: `roles/apache_info/templates/index.html.j2`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Host information</title>
</head>
<body>
    <h1>Host information</h1>

    <p><strong>Hostname:</strong> {{ ansible_hostname }}</p>
    <p><strong>CPU:</strong> {{ ansible_processor_vcpus }}</p>
    <p><strong>RAM:</strong> {{ ansible_memtotal_mb }} MB</p>
    <p><strong>First HDD:</strong> {{ ansible_devices.keys() | list | first }}</p>
    <p><strong>IP address:</strong> {{ ansible_default_ipv4.address }}</p>
</body>
</html>
```

---

# Проверка синтаксиса

```bash
ansible-playbook playbook_apache_role.yml --syntax-check
```

---

# Запуск playbook

```bash
ansible-playbook playbook_apache_role.yml -c local --ask-become-pass
```

---

# Результат выполнения

```text
PLAY RECAP ****************************************************************

localhost : ok=6 changed=4 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

# Проверка работы Apache

```bash
systemctl status apache2
```

---

# Проверка доступности сайта

```bash
curl -I http://localhost
```

Результат:

```text
HTTP/1.1 200 OK
```

---

# Проверка содержимого страницы

```bash
curl http://localhost
```

На странице отображаются:

* hostname;
* количество CPU;
* объём RAM;
* первый HDD;
* IP-адрес.

---

# Архив роли

Роль была упакована в архив:

```bash
tar -czf apache_info_role.tar.gz roles/apache_info
```
