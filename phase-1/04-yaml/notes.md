# 📘 មេរៀនទី 4: YAML Syntax

> មេរៀនចុងក្រោយនៃ Phase 1 — ភាសាដែល Docker Compose និង Kubernetes ប្រើ

---

## 1. YAML ជាអ្វី?

**YAML** = "YAML Ain't Markup Language" — format សម្រាប់សរសេរ **configuration** ដែលងាយអានបំផុតសម្រាប់មនុស្ស។

### ប្រើនៅឯណា?

| Tool | File |
|------|------|
| **Docker Compose** | `docker-compose.yml` ⭐ |
| **Kubernetes** | `deployment.yaml` |
| **GitHub Actions** | `.github/workflows/ci.yml` |
| **Ansible** | `playbook.yml` |

> DevOps ទាំងមូល = YAML គ្រប់ទីកន្លែង! ដូច្នេះមេរៀននេះសំខាន់ខ្លាំងណាស់ 🔑

### YAML vs JSON — ទិន្នន័យដូចគ្នា

```json
// JSON
{
  "name": "my-app",
  "port": 8080,
  "tags": ["web", "docker"]
}
```

```yaml
# YAML — ស្អាតជាង, គ្មាន {} "" ,
name: my-app
port: 8080
tags:
  - web
  - docker
```

---

## 2. ច្បាប់ទី 1 (សំខាន់បំផុត): Indentation ⚠️

YAML ប្រើ **ការចូលបន្ទាត់ (spaces)** ដើម្បីបង្ហាញរចនាសម្ព័ន្ធ — ដូច Python។

### ច្បាប់ដែក 3 ចំណុច

1. ប្រើ **spaces ប៉ុណ្ណោះ** — **TAB ហាមដាច់ខាត!** ❌ (YAML error ភ្លាម)
2. ស្តង់ដារ: **2 spaces** ក្នុងមួយកម្រិត
3. កម្រិតដូចគ្នា ត្រូវចូលបន្ទាត់**ស្មើគ្នា**

```yaml
# ✅ ត្រឹមត្រូវ
server:
  host: localhost
  port: 8080

# ❌ ខុស — indentation មិនស្មើគ្នា
server:
  host: localhost
    port: 8080
```

> 90% នៃ YAML errors = indentation ខុស ឬប្រើ TAB!

---

## 3. Key-Value Pairs (មូលដ្ឋាន)

```yaml
# key: value (ត្រូវមាន space ក្រោយ colon!)
name: docker-lab
version: 1.0
port: 8080
debug: true
description: null        # គ្មានតម្លៃ (ឬសរសេរ ~)
```

### Data Types (YAML ស្មានប្រភេទដោយស្វ័យប្រវត្តិ)

```yaml
string1: hello                 # string
string2: "hello world"         # string (មាន quotes ក៏បាន)
number: 42                     # integer
decimal: 3.14                  # float
boolean1: true                 # boolean
boolean2: false
empty: null                    # null

# ⚠️ អន្ទាក់ល្បីៗ — ត្រូវប្រើ quotes:
version: "1.10"     # បើគ្មាន quotes → ក្លាយជាលេខ 1.1 !
answer: "yes"       # បើគ្មាន quotes → ក្លាយជា boolean true !
port_str: "8080"    # បង្ខំជា string
country: "NO"       # Norway → បើគ្មាន quotes ក្លាយជា false ! 😱
```

---

## 4. Nested Objects (Maps ក្នុង Maps)

```yaml
server:
  host: localhost
  port: 8080
  database:
    name: mydb
    user: admin
    password: secret123
```

ស្មើនឹង JSON:

```json
{"server": {"host": "localhost", "port": 8080, "database": {"name": "mydb", "user": "admin", "password": "secret123"}}}
```

---

## 5. Lists (Arrays)

```yaml
# របៀបទី 1: dash (-) — ប្រើច្រើនបំផុត
fruits:
  - apple
  - banana
  - mango

# របៀបទី 2: inline — សម្រាប់ list ខ្លីៗ
fruits: [apple, banana, mango]
```

### List នៃ Objects (ជួបញឹកញាប់ក្នុង Docker/K8s!)

```yaml
users:
  - name: sok
    role: admin
  - name: dara
    role: developer
```

> ចំណាំ: `-` ចាប់ផ្តើម object ថ្មីមួយក្នុង list

---

## 6. ធាតុផ្សេងទៀតដែលត្រូវស្គាល់

```yaml
# Comment ចាប់ផ្តើមដោយ #
name: my-app    # comment នៅចុងបន្ទាត់ក៏បាន

# Multi-line string (| រក្សា newlines)
description: |
  បន្ទាត់ទី 1
  បន្ទាត់ទី 2

# Multi-line string (> បញ្ចូលជាបន្ទាត់តែមួយ)
summary: >
  ប្រយោគវែងមួយ
  ដែលសរសេរច្រើនបន្ទាត់
  តែក្លាយជាមួយបន្ទាត់

# Environment variable style (ជួបក្នុង docker-compose)
environment:
  - DB_HOST=db
  - DB_PORT=5432
```

---

## 7. ឧទាហរណ៍ពិត: docker-compose.yml 🐳

នេះជា file ដែលអ្នកនឹងសរសេរនៅ Phase ក្រោយ — សាកអានមើល ឥឡូវអ្នកគួរយល់រចនាសម្ព័ន្ធវាហើយ:

```yaml
version: "3.8"                # string (quotes ការពារក្លាយជាលេខ!)

services:                     # nested object
  web:                        # object ឈ្មោះ "web"
    image: nginx:latest       # key-value
    ports:                    # list
      - "8080:80"             # host:container (ចាំមេរៀនទី 3?)
    depends_on:
      - db

  db:                         # object ទី 2
    image: postgres:16
    environment:              # nested object
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

**អានចេញទេ?** services មាន 2: `web` (nginx, port mapping 8080→80) និង `db` (postgres ជាមួយ credentials) — អ្នកអាន YAML បានហើយ! 🎉

---

## 8. Validate YAML

មុន run ជានិច្ច គួរ check syntax:

- Online: **yamllint.com** ឬ **yamlchecker.com** — paste ចូល → ឃើញ error ភ្លាម
- VS Code: install extension **"YAML"** (Red Hat) — highlight error ផ្ទាល់ពេលវាយ

---

## 9. អន្ទាក់ល្បីៗត្រូវចាំ ⚠️

| អន្ទាក់ | ដំណោះស្រាយ |
|---------|-------------|
| ប្រើ TAB | ប្រើ 2 spaces ជំនួស |
| ភ្លេច space ក្រោយ `:` | `port: 8080` ✅ / `port:8080` ❌ |
| `version: 1.10` ក្លាយជា 1.1 | ដាក់ quotes: `"1.10"` |
| `yes/no/on/off` ក្លាយជា boolean | ដាក់ quotes បើចង់បាន string |
| Indentation មិនស្មើ | កម្រិតដូចគ្នា = spaces ស្មើគ្នា |

---

## ✅ Checklist លំហាត់

- [ ] បង្កើត file `practice.yml` ក្នុង `04-yaml/` ដែលមាន: string, number, boolean, nested object, list, list of objects
- [ ] Paste ចូល yamllint.com — ត្រូវ pass ដោយគ្មាន error
- [ ] សាកដាក់ TAB មួយក្នុង file → validate ម្តងទៀត → មើល error
- [ ] សរសេរ profile ខ្លួនឯងជា YAML (name, skills list, education object)
- [ ] អាន docker-compose.yml ខាងលើ ហើយពន្យល់ដោយពាក្យខ្លួនឯងថា បន្ទាត់នីមួយៗមានន័យអ្វី
- [ ] Commit: `git commit -m "Phase 1: yaml syntax notes - phase 1 complete!"`

---

## 🎉 Phase 1 ចប់ហើយ!

អ្នកបានរៀន:
1. ✅ Linux basics — commands, permissions (chmod), processes
2. ✅ Virtualization vs Containerization — VM vs Container
3. ✅ Networking — IP, Port, DNS, localhost, web server
4. ✅ YAML — ភាសា config របស់ DevOps

**បន្ទាប់: Phase 2 — ដំឡើង Docker Desktop ហើយ run container ដំបូង!** 🐳