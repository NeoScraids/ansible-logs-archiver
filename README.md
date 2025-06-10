# ansible-logs-archiver

> Un proyecto ligero de Ansible para renombrar, comprimir y archivar logs incorporando hostname, IP y fecha, organizado por carpetas de fecha.

```
ansible-logs-archiver/
├── .gitignore
├── LICENSE
├── README.md
├── inventory/
│   └── hosts.yml
├── playbook.yml
└── roles/
    └── logs_archiver/
        ├── defaults/
        │   └── main.yml
        ├── meta/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        └── handlers/
            └── main.yml
```

---

## .gitignore
```gitignore
# Evitar archivos de retry de Ansible
env.retry
# Editores\ n.vscode/
``` 

---

## LICENSE (MIT)
```text
MIT License

Copyright (c) 2025 Brandon Mendieta

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction...
``` 

---

## README.md
```markdown
# ansible-logs-archiver

> Archiva logs renombrados con hostname, IP y fecha, y organiza los `.tar.gz` por carpetas con fecha.

## Características
- Busca archivos `.log` en el directorio fuente.
- Renombra a `<hostname>_<IP>_<original_filename>.tar.gz`.
- Organiza en `<archive_base_dir>/YYYY-MM-DD/`.
- Limpieza opcional de logs originales.

## Requisitos Previos
- Ansible 2.9+ y Python 3.x
- Conexión SSH con sudo habilitado.

## Uso
```bash
git clone https://github.com/USERNAME/ansible-logs-archiver.git
cd ansible-logs-archiver
```
1. Edita `inventory/hosts.yml` con tus servidores.
2. Ajusta variables en `roles/logs_archiver/defaults/main.yml`.
3. Ejecuta:
```bash
ansible-playbook -i inventory/hosts.yml playbook.yml
```

## Variables
| Variable            | Descripción                                    | Valor por defecto            |
|---------------------|------------------------------------------------|------------------------------|
| `logs_source_dir`   | Ruta donde buscar logs                         | `/var/log/myapp`             |
| `archive_base_dir`  | Carpeta base de destino de los `.tar.gz`       | `/var/log/archived_logs`     |
| `file_pattern`      | Patrón de búsqueda de archivos (`glob`)        | `*.log`                      |
| `cleanup`           | Eliminar logs originales tras archivar         | `true`                       |

## Estructura del Role
Mira en `roles/logs_archiver` para más detalles.

## Autor
Brandon Mendieta — [GitHub](https://github.com/USERNAME)

## Licencia
MIT — véase [LICENSE](LICENSE).
```

---

## inventory/hosts.yml
```yaml
all:
  hosts:
    servidor1.example.com:
    servidor2.example.com:
``` 

---

## playbook.yml
```yaml
- name: Archive logs en todos los servidores
  hosts: all
  become: true
  vars_files:
    - roles/logs_archiver/defaults/main.yml
  roles:
    - logs_archiver
```

---

## roles/logs_archiver/defaults/main.yml
```yaml
---
# Directorio fuente de logs\ nlogs_source_dir: /var/log/myapp
# Directorio base para archivar\ narchive_base_dir: /var/log/archived_logs
# Patrón de archivos\ nfile_pattern: '*.log'
# ¿Eliminar logs originales? \ ncleanup: true
``` 

---

## roles/logs_archiver/meta/main.yml
```yaml
---
galaxy_info:
  role_name: logs_archiver
  author: Brandon Mendieta
  description: "Archiva logs renombrando con hostname, IP y fecha."
  company: TuEmpresa
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: EL
      versions:
        - 7
        - 8
dependencies: []
```

---

## roles/logs_archiver/tasks/main.yml
```yaml
---
- name: Encontrar archivos de log
  ansible.builtin.find:
    paths: "{{ logs_source_dir }}"
    patterns: "{{ file_pattern }}"
  register: found_logs

- name: Set fact de fecha
  ansible.builtin.set_fact:
    archive_date: "{{ ansible_date_time.date }}"

- name: Crear carpeta de archivo\ n  ansible.builtin.file:
    path: "{{ archive_base_dir }}/{{ archive_date }}"
    state: directory
    mode: '0755'

- name: Comprimir y renombrar cada log
  loop: "{{ found_logs.files }}"
  loop_control:
    label: "{{ item.path | basename }}"
  vars:
    base_name: "{{ ansible_hostname }}_{{ ansible_default_ipv4.address }}_{{ item.path | basename }}.tar.gz"
    dest_path: "{{ archive_base_dir }}/{{ archive_date }}/{{ base_name }}"
  ansible.builtin.archive:
    path: "{{ item.path }}"
    dest: "{{ dest_path }}"
  when: found_logs.matched > 0

- name: Eliminar log original tras archivar
  ansible.builtin.file:
    path: "{{ item.path }}"
    state: absent
  when: cleanup and found_logs.matched > 0
``` 

---

## roles/logs_archiver/handlers/main.yml
```yaml
---
- name: Recargar rsyslog (opcional)
  ansible.builtin.service:
    name: rsyslog
    state: reloaded
```
