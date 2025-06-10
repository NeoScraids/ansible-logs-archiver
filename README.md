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
├── roles/
│   └── logs_archiver/
│       ├── defaults/
│       │   └── main.yml
│       ├── meta/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       └── handlers/
│           └── main.yml
└── .github/
    └── workflows/
        ├── refresh-readme.yml
        ├── ansible-lint.yml
        └── molecule-test.yml
```

---

## Configurar Git globalmente
Para que tus commits siempre usen tu usuario **NeoScraids**, añade esto a tu `~/.gitconfig` o ejecútalo en tu terminal:
```bash
git config --global user.name "NeoScraids"
git config --global user.email "NeoScraids@users.noreply.github.com"
```

---

## .gitignore
```gitignore
# Archivos de retry de Ansible
env.retry
# Configuración de editores
.vscode/
``` 

---

## LICENSE (MIT)
```text
MIT License

Copyright (c) 2025 NeoScraids

Permission is hereby granted, free of charge, to any person obtaining a copy
...
``` 

---

## README.md
```markdown
# ansible-logs-archiver

> Archiva logs renombrados con hostname, IP y fecha; organiza los `.tar.gz` por carpetas de fecha.

## Características
- Busca archivos `.log` en el directorio fuente.
- Renombra a `<hostname>_<IP>_<original_filename>.tar.gz`.
- Organiza en `<archive_base_dir>/YYYY-MM-DD/`.
- Elimina logs originales si `cleanup: true`.

## Requisitos Previos
- Ansible 2.9+ y Python 3.x.
- SSH con sudo.

## Uso
```bash
git clone https://github.com/NeoScraids/ansible-logs-archiver.git
cd ansible-logs-archiver
ansible-playbook -i inventory/hosts.yml playbook.yml
```

## Estructura
detallada arriba.

## Variables
| Variable           | Descripción                                  | Valor por defecto            |
|--------------------|----------------------------------------------|------------------------------|
| `logs_source_dir`  | Ruta de los logs                             | `/var/log/myapp`             |
| `archive_base_dir` | Ruta destino de los `.tar.gz`                | `/var/log/archived_logs`     |
| `file_pattern`     | Patrón de archivos (`glob`)                  | `*.log`                      |
| `cleanup`          | ¿Eliminar logs originales tras archivar?      | `true`                       |

## Autor
NeoScraids — [GitHub](https://github.com/NeoScraids)

## Licencia
MIT — véase [LICENSE](LICENSE)
```

---

## Workflows GitHub Actions
Coloca estos archivos en `.github/workflows/` para CI/CD:

### 1. Actualizar README (`refresh-readme.yml`)
```yaml
name: 👉 Actualizar README
on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

jobs:
  update-readme:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Regenerar estadísticas
        run: echo "Nothing to build; badges vienen de GitHub Readme Stats"
      - name: Commit README actualizado
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: "chore: Actualizar GitHub README"
          author_name: "NeoScraids"
          author_email: "NeoScraids@users.noreply.github.com"
```

### 2. Lint Ansible (`ansible-lint.yml`)
```yaml
name: Lint Ansible Role
on:
  push:
    paths:
      - 'roles/logs_archiver/**'
  pull_request:
jobs:
  ansible-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      - run: pip install ansible-lint
      - run: ansible-lint roles/logs_archiver
```

### 3. Tests con Molecule (`molecule-test.yml`)
```yaml
name: Molecule Test
on:
  push:
    paths:
      - 'roles/logs_archiver/**'
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      - run: |
          pip install molecule[docker] docker ansible
      - run: molecule test -s default
```
