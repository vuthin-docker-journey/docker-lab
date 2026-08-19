# 📘 មេរៀនទី 1: Linux Basics

> ផ្នែកមួយនៃ Phase 1 — មូលដ្ឋានគ្រឹះមុនរៀន Docker

---

## 1. Navigation Commands

| Command | អត្ថន័យ | ឧទាហរណ៍ |
|---------|---------|----------|
| `pwd` | បង្ហាញ directory បច្ចុប្បន្ន (print working directory) | `pwd` |
| `ls` | list files ក្នុង directory | `ls` |
| `ls -la` | list ទាំងអស់ រួមទាំង hidden files + លម្អិត | `ls -la` |
| `cd <dir>` | ចូលទៅ directory | `cd phase-1` |
| `cd ..` | ថយក្រោយមួយកម្រិត | `cd ..` |
| `cd ~` | ត្រឡប់ទៅ home directory | `cd ~` |

---

## 2. File Operations

| Command | អត្ថន័យ |
|---------|---------|
| `touch file.txt` | បង្កើត file ទទេ |
| `echo "text" > file.txt` | សរសេរ text ចូល file (ជំនួសមាតិកាចាស់) |
| `echo "text" >> file.txt` | បន្ថែម text ចូល file (មិនលុបចាស់) |
| `cat file.txt` | បង្ហាញមាតិកា file |
| `cp a.txt b.txt` | copy file |
| `mv a.txt folder/` | ផ្លាស់ទី ឬប្តូរឈ្មោះ file |
| `rm file.txt` | លុប file |
| `mkdir folder` | បង្កើត directory |
| `rm -r folder` | លុប directory ទាំងមូល ⚠️ ប្រយ័ត្ន! |

---

## 3. Permissions (chmod) ⭐ សំខាន់បំផុតសម្រាប់ Docker

### Permission 3 ប្រភេទ

| Symbol | ឈ្មោះ | អត្ថន័យ | តម្លៃលេខ |
|--------|-------|---------|----------|
| `r` | read | អាចអានមាតិកា file | **4** |
| `w` | write | អាចកែប្រែ/លុប file | **2** |
| `x` | execute | អាច run file ដូច program | **1** |

### របៀបអាន `ls -l`

```
-rwxr-xr-x  1  USER  ...  test.txt
```

```
-    rwx    r-x    r-x
│     │      │      │
│     │      │      └── Others (អ្នកផ្សេងទៀត)
│     │      └───────── Group (ក្រុម)
│     └──────────────── Owner (ម្ចាស់ file)
└────────────────────── File type (- = file, d = directory)
```

- **Owner: `rwx`** → read ✅ write ✅ execute ✅
- **Group: `r-x`** → read ✅ write ❌ execute ✅
- **Others: `r-x`** → read ✅ write ❌ execute ✅
- សញ្ញា `-` = **គ្មាន** permission នោះ

### Octal Notation (លេខ)

បូកតម្លៃក្នុងក្រុមនីមួយៗ (r=4, w=2, x=1):

```
rwx    r-x    r-x
4+2+1  4+0+1  4+0+1
  7      5      5     →  chmod 755
```

### លេខដែលប្រើញឹកញាប់

| Command | លទ្ធផល | ប្រើសម្រាប់ |
|---------|--------|-------------|
| `chmod 644` | `rw-r--r--` | config files (owner កែបាន, អ្នកផ្សេងអានបានតែប៉ុណ្ណោះ) |
| `chmod 755` | `rwxr-xr-x` | scripts/programs (owner ពេញសិទ្ធិ, អ្នកផ្សេង run បាន) |
| `chmod 777` | `rwxrwxrwx` | ⚠️ គ្រោះថ្នាក់! គ្រប់គ្នាពេញសិទ្ធិ — កុំប្រើលើ production |
| `chmod +x` | បន្ថែម execute | ធ្វើឱ្យ script អាច run បាន |
| `chmod -x` | ដក execute ចេញ | បិទការ run |

### ទំនាក់ទំនងជាមួយ Docker 🐳

```dockerfile
COPY entrypoint.sh /app/
RUN chmod +x /app/entrypoint.sh    # ← បើភ្លេច → container crash: "permission denied"
```

Error `permission denied` ក្នុង Docker ភាគច្រើនកើតឡើងព្រោះ script គ្មាន `x` permission!

---

## 4. Process Management

| Command | អត្ថន័យ |
|---------|---------|
| `ps` | មើល processes ដែលកំពុងដំណើរការ |
| `command &` | run process ក្នុង background (ឧ. `sleep 100 &`) |
| `kill <PID>` | បញ្ឈប់ process តាម PID |
| `kill -9 <PID>` | បង្ខំបញ្ឈប់ភ្លាមៗ (force kill) |

**PID** = Process ID — លេខសម្គាល់ process នីមួយៗ

### ការពិសោធន៍

```bash
sleep 100 &     # run ក្នុង background → បង្ហាញ PID
ps              # ឃើញ sleep ក្នុងបញ្ជី
kill <PID>      # បញ្ឈប់វា
ps              # sleep បាត់ហើយ
```

> ទំនាក់ទំនងជាមួយ Docker: `docker ps` មើល containers ដែលកំពុង run, `docker kill` បញ្ឈប់ container — គំនិតដូចគ្នា!

---

## 5. ចំណាំសំខាន់ៗ

- Git Bash (MINGW64) មិនមែន Linux ពេញលេញទេ — commands ខ្លះ (`top`, `systemctl`) មិនដំណើរការ
- Permission នៅ Windows មិនដូច Linux 100% — លទ្ធផល `ls -l` អាចមើលទៅចម្លែកបន្តិច
- នៅក្នុង Docker container (Linux ពិត) ច្បាប់ permission ដំណើរការត្រឹមត្រូវតាមតារាងខាងលើ

---

## ✅ Checklist លំហាត់

- [ ] Run navigation commands ទាំងអស់
- [ ] បង្កើត, copy, move, លុប file
- [ ] សាកល្បង `chmod +x` និង `chmod -x` លើ script.sh
- [ ] សាកល្បង `chmod 755` vs `chmod 644` ហើយប្រៀបធៀបលទ្ធផល `ls -l`
- [ ] Run `sleep 100 &` ហើយ kill វាតាម PID
- [ ] Commit ចូល Git: `git commit -m "Phase 1: linux basics notes"`

