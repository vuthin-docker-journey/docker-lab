# 📘 មេរៀនទី 3: Networking Basics

> ផ្នែកមួយនៃ Phase 1 — មូលដ្ឋាន network មុននឹងយល់ `docker run -p 8080:80`

---

## 1. IP Address — អាសយដ្ឋានរបស់ម៉ាស៊ីន

**IP (Internet Protocol) address** = លេខសម្គាល់ម៉ាស៊ីននីមួយៗនៅក្នុង network — ដូចអាសយដ្ឋានផ្ទះ 🏠

### IPv4 Format

```
192.168.1.10
 │   │  │ │
 └───┴──┴─┴── 4 ផ្នែក (octets), ផ្នែកនីមួយៗ 0-255
```

### ប្រភេទ IP សំខាន់ៗ

| ប្រភេទ | Range/ឧទាហរណ៍ | អត្ថន័យ |
|--------|----------------|---------|
| **Public IP** | ឧ. 203.144.x.x | IP ដែលមើលឃើញពី Internet (ISP ផ្តល់ឱ្យ) |
| **Private IP** | 192.168.x.x, 10.x.x.x, 172.16-31.x.x | ប្រើក្នុង network ផ្ទៃក្នុង (ផ្ទះ, ការិយាល័យ) |
| **Localhost** | 127.0.0.1 | ម៉ាស៊ីនខ្លួនឯង — "ខ្ញុំផ្ទាល់" |
| **0.0.0.0** | 0.0.0.0 | "គ្រប់ interface ទាំងអស់" — សំខាន់ក្នុង Docker! |

### 🔑 localhost (127.0.0.1)

- `localhost` = ឈ្មោះដែលចង្អុលទៅ `127.0.0.1` = **ម៉ាស៊ីនខ្លួនឯង**
- ពេល run web server លើម៉ាស៊ីនខ្លួនឯង → បើក browser ទៅ `http://localhost:3000`
- Traffic ទៅ localhost **មិនចេញពីម៉ាស៊ីនទេ** — វិលក្នុងម៉ាស៊ីនតែម្តង

> ⚠️ **ចំណាំសម្រាប់ Docker:** នៅក្នុង container, `localhost` = **container ខ្លួនឯង** មិនមែនម៉ាស៊ីន host ទេ! នេះជាកំហុសដែល beginners ជួបញឹកញាប់បំផុត។

---

## 2. Port — ទ្វារចូលទៅ service

IP = អាសយដ្ឋានអគារ 🏢 / **Port = លេខបន្ទប់ក្នុងអគារ** 🚪

ម៉ាស៊ីនមួយអាច run services ច្រើនក្នុងពេលតែមួយ — port ជាអ្នកបំបែកថា traffic ទៅ service ណា។

```
192.168.1.10:3000
      │       │
      IP     Port (0-65535)
```

### Ports ល្បីៗដែលត្រូវចាំ

| Port | Service | ចំណាំ |
|------|---------|-------|
| **80** | HTTP | web ធម្មតា |
| **443** | HTTPS | web មានសុវត្ថិភាព |
| **22** | SSH | remote login |
| **3306** | MySQL | database |
| **5432** | PostgreSQL | database |
| **6379** | Redis | cache |
| **27017** | MongoDB | database |
| **3000, 8080, 5000** | Dev servers | ប្រើញឹកញាប់ពេល develop |

### ច្បាប់សំខាន់

- Port range: **0 - 65535**
- Port 0-1023 = "well-known ports" — ត្រូវការ admin/root ដើម្បីប្រើ
- **Port មួយ ប្រើបានតែ process មួយ** ក្នុងពេលតែមួយ — បើជាន់គ្នា → error `port already in use` (កំហុសល្បីក្នុង Docker!)

### ទំនាក់ទំនងជាមួយ Docker 🐳

```bash
docker run -p 8080:80 nginx
#            │    │
#            │    └── port ក្នុង container (nginx ស្តាប់លើ 80)
#            └─────── port លើ host machine (អ្នកចូលតាម localhost:8080)
```

នេះហៅថា **port mapping** — "បើកទ្វារ 8080 លើ host ភ្ជាប់ទៅទ្វារ 80 ក្នុង container"

---

## 3. DNS — សៀវភៅទូរស័ព្ទរបស់ Internet

**DNS (Domain Name System)** = ប្រព័ន្ធបកប្រែ **ឈ្មោះ → IP** ☎️

មនុស្សចាំឈ្មោះងាយជាងលេខ:

```
អ្នកវាយ: google.com
   │
   ▼
DNS Server: "google.com = 142.250.190.78"
   │
   ▼
Browser ភ្ជាប់ទៅ 142.250.190.78
```

### ដំណើរការ DNS Lookup

1. Browser ពិនិត្យ **cache** ខ្លួនឯងជាមុន (ធ្លាប់សួររួច?)
2. ពិនិត្យ file **hosts** ក្នុងម៉ាស៊ីន (`C:\Windows\System32\drivers\etc\hosts`)
3. សួរ **DNS server** (ISP ឬ 8.8.8.8 របស់ Google, 1.1.1.1 របស់ Cloudflare)
4. ទទួលបាន IP → ភ្ជាប់

### ទំនាក់ទំនងជាមួយ Docker 🐳

Docker មាន **DNS ផ្ទៃក្នុងរបស់ខ្លួន** — containers អាចហៅគ្នាតាម**ឈ្មោះ**:

```yaml
# docker-compose.yml (នឹងរៀនពេលក្រោយ)
services:
  web:
    ...
  db:
    ...
# ក្នុង code របស់ web: ភ្ជាប់ទៅ "db:5432" — មិនចាំបាច់ដឹង IP ទេ!
```

---

## 4. អនុវត្តជាក់ស្តែង (Git Bash / CMD)

```bash
# 1. មើល IP របស់ម៉ាស៊ីនខ្លួនឯង
ipconfig                    # Windows (រក "IPv4 Address")

# 2. Test ថាម៉ាស៊ីនមួយ reachable ឬអត់
ping google.com             # ផ្ញើ packets ទៅ google
ping 127.0.0.1              # ping ខ្លួនឯង — គួរតែឆ្លើយភ្លាមៗ

# 3. DNS lookup — រក IP ពីឈ្មោះ
nslookup google.com         # សួរ DNS server
nslookup github.com

# 4. ទាញទិន្នន័យពី web server
curl https://api.github.com # HTTP request ពី command line
curl -I https://google.com  # -I = មើលតែ headers

# 5. មើល ports ដែលកំពុងបើក
netstat -an | grep LISTEN   # Git Bash
netstat -ano | findstr LISTENING   # CMD/PowerShell
```

### ការពិសោធន៍តូចមួយ 🧪

```bash
# បើមាន Python (ភាគច្រើន Windows មាន py launcher):
python -m http.server 8000
# ឬ: py -m http.server 8000

# បន្ទាប់មកបើក browser ទៅ:
#   http://localhost:8000
#   http://127.0.0.1:8000    ← ដូចគ្នា!

# ចុច Ctrl+C ដើម្បីបញ្ឈប់ server
```

អ្នកទើបតែ run web server លើ port 8000 ហើយចូលមើលតាម localhost — នេះជាគំនិតដូចគ្នានឹង Docker port mapping!

---

## 5. ពាក្យគន្លឹះត្រូវចាំ

| ពាក្យ | អត្ថន័យ |
|-------|---------|
| **IP address** | អាសយដ្ឋានម៉ាស៊ីនក្នុង network |
| **localhost / 127.0.0.1** | ម៉ាស៊ីនខ្លួនឯង |
| **0.0.0.0** | ស្តាប់គ្រប់ interfaces (សំខាន់ក្នុង Docker) |
| **Port** | ទ្វារ/លេខបន្ទប់សម្រាប់ service នីមួយៗ (0-65535) |
| **Port mapping** | ភ្ជាប់ port host ↔ port container (`-p 8080:80`) |
| **DNS** | បកប្រែឈ្មោះ → IP |
| **ping** | test ថាម៉ាស៊ីន reachable |
| **curl** | ផ្ញើ HTTP request ពី terminal |

---

## ✅ Checklist លំហាត់

- [ ] Run `ipconfig` ហើយរក Private IP របស់ម៉ាស៊ីនអ្នក (192.168.x.x)
- [ ] `ping 127.0.0.1` និង `ping google.com` — ប្រៀបធៀបល្បឿន (ms)
- [ ] `nslookup github.com` — កត់ត្រា IP ដែលទទួលបាន
- [ ] Run `python -m http.server 8000` ហើយបើក `http://localhost:8000` ក្នុង browser
- [ ] សាកល្បង run server 2 ដងលើ port ដូចគ្នា — មើល error `port already in use`
- [ ] ឆ្លើយសំណួរ: ភាពខុសគ្នារវាង `localhost` និង `0.0.0.0`?
- [ ] ឆ្លើយសំណួរ: ក្នុង `-p 8080:80` លេខមួយណាជា host port?
- [ ] Commit: `git commit -m "Phase 1: networking basics notes"`