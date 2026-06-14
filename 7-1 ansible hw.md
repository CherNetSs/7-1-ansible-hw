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
