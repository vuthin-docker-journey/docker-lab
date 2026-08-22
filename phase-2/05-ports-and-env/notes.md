# Phase 2 — មេរៀនទី 5: Port mapping (`-p`) និង Environment variables (`-e`)

> 📁 `phase-2/05-ports-and-env/notes.md`
> 🎯 គោលដៅ: ភ្ជាប់ពិភពខាងក្រៅទៅ container (ports) និងកំណត់ config ឲ្យ container ដោយមិនកែ image (env) — ហើយដោះស្រាយ postgres ដែល crash ពីមេរៀនទី 4

---

## 1. បញ្ហាពីរដែលមេរៀននេះដោះស្រាយ

```
បញ្ហា 1: docker run -d nginx  → http://localhost មិនចេញ        → -p
បញ្ហា 2: docker run -d postgres → Exited (1) password missing   → -e
```

---

## 2. Port mapping — រូបភាពធំ

Container នីមួយៗមាន **network namespace ផ្ទាល់ខ្លួន** — IP របស់វា (ឧ. `172.17.0.2`) និង ports របស់វា។ Port 80 ក្នុង container **មិនមែន** port 80 របស់ host ទេ។

```
 Browser (Windows)
      │  http://localhost:8080
      ▼
 ┌─────────────────────────── Host (WSL2 VM) ───────────────┐
 │  port 8080 ──── Docker NAT (iptables) ────┐               │
 │                                           ▼               │
 │                          ┌──── Container web ─────┐       │
 │                          │ 172.17.0.2 : 80 (nginx) │       │
 │                          └─────────────────────────┘       │
 └────────────────────────────────────────────────────────────┘
```

`-p 8080:80` = "អ្វីដែលមកដល់ host port **8080** បញ្ជូនទៅ container port **80**"។

### Syntax

```
-p [HOST_IP:]HOST_PORT:CONTAINER_PORT[/PROTOCOL]
```

| ទម្រង់ | ន័យ |
|---|---|
| `-p 8080:80` | host 8080 → container 80 (ស្តាប់គ្រប់ interface `0.0.0.0`) |
| `-p 127.0.0.1:8080:80` | តែ localhost ប៉ុណ្ណោះ (សុវត្ថិភាពជាង — ម៉ាស៊ីនផ្សេងក្នុង LAN ចូលមិនបាន) |
| `-p 80:80` | port ដូចគ្នា (ត្រូវការ port 80 ទំនេរនៅ host) |
| `-p 5432:5432/tcp` | បញ្ជាក់ protocol (default tcp) |
| `-p 8080:80 -p 8443:443` | ច្រើន ports |
| `-P` (ធំ) | map `EXPOSE` ports ទាំងអស់ទៅ host ports ចៃដន្យ (32768+) |
| `-p 80` | container 80 → host port ចៃដន្យ |

### `EXPOSE` vs `-p`

| | `EXPOSE 80` (ក្នុង Dockerfile) | `-p 8080:80` (ពេល run) |
|---|---|---|
| ធ្វើអ្វី | **ឯកសារ** — ប្រាប់ថា app ស្តាប់ port 80 | **បើក** port ពិត |
| បើក port ទេ? | ❌ | ✅ |
| ប្រើដោយ | `-P`, `docker inspect`, មនុស្សអាន | NAT rule ពិត |

💡 Container ក្នុង network ដូចគ្នាអាចនិយាយគ្នាដោយ **មិនត្រូវការ `-p`** — `-p` សម្រាប់តែ host/ខាងក្រៅចូលមក (Phase 4)។

### មើល port mapping

```bash
docker ps                        # PORTS column: 0.0.0.0:8080->80/tcp
docker port web                  # 80/tcp -> 0.0.0.0:8080
docker inspect -f '{{json .NetworkSettings.Ports}}' web
```

---

## 3. Environment variables — រូបភាពធំ

Image តែមួយ, config ផ្សេងគ្នា (dev/prod, password, port) → **មិនគួរ build image ថ្មី** សម្រាប់ config នីមួយៗ។ ដំណោះស្រាយ: env variables ពេល run។

```
Image (ដូចគ្នា)          Container                 Process ខាងក្នុងអាន
postgres:16  ──run -e──►  POSTGRES_PASSWORD=abc ──► os.environ / $POSTGRES_PASSWORD
```

### Syntax

| ទម្រង់ | ន័យ |
|---|---|
| `-e KEY=value` | កំណត់មួយ |
| `-e KEY` (គ្មាន `=`) | យកតម្លៃពី shell របស់ host |
| `--env-file .env` | អានពី file (មួយបន្ទាត់ `KEY=value`, `#` = comment) |
| `-e A=1 -e B=2` | ច្រើន |

### លំដាប់អាទិភាព (ខ្ពស់ → ទាប)

```
-e KEY=value  >  --env-file  >  ENV ក្នុង Dockerfile  >  gមាន
```

### មើល env

```bash
docker exec web env
docker exec web sh -c 'echo $NGINX_VERSION'
docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}' web
```

⚠️ **សុវត្ថិភាព:** `docker inspect` បង្ហាញ env **ទាំងអស់ជា plain text** (រួមទាំង password)។ ក្នុង production ប្រើ Docker secrets / vault (Phase 6)។ កុំ commit `.env` ចូល git — ដាក់ក្នុង `.gitignore`, commit `.env.example` ជំនួស។

### Official images ប្រើ env សម្រាប់ config

| Image | Env សំខាន់ |
|---|---|
| `postgres` | `POSTGRES_PASSWORD` (ចាំបាច់), `POSTGRES_USER`, `POSTGRES_DB` |
| `mysql` | `MYSQL_ROOT_PASSWORD` (ចាំបាច់), `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD` |
| `redis` | គ្មានចាំបាច់ (config តាម command args) |
| `mongo` | `MONGO_INITDB_ROOT_USERNAME`, `MONGO_INITDB_ROOT_PASSWORD` |
| `nginx` | `NGINX_HOST`, `NGINX_PORT` (សម្រាប់ templates) |

👉 រកបាននៅ Docker Hub → tab **Overview** របស់ image នីមួយៗ — ទម្លាប់អាន Overview មុន run គឺចាំបាច់។

---

## 4. Lab — អនុវត្តន៍

### Lab 1: ឃើញបញ្ហា ហើយដោះស្រាយ

```bash
docker run -d --name web-noport nginx:1.27-alpine
curl localhost            # ❌ Connection refused (ឬ browser ចូលមិនបាន)
docker inspect -f '{{.NetworkSettings.IPAddress}}' web-noport   # 172.17.0.x

docker run -d --name web -p 8080:80 nginx:1.27-alpine
curl localhost:8080       # ✅
docker ps                 # ប្រៀបធៀប PORTS column ទាំងពីរ
docker port web
docker rm -f web-noport
```

### Lab 2: Port conflict

```bash
docker run -d --name web2 -p 8080:80 nginx:1.27-alpine
# ❌ Error: port is already allocated
docker run -d --name web2 -p 8081:80 nginx:1.27-alpine     # ✅ host port ផ្សេង
curl localhost:8081
```
👉 Container ពីរអាចស្តាប់ port **80** ខាងក្នុងដូចគ្នា — តែ host port ត្រូវខុសគ្នា។

### Lab 3: `-P` និង localhost-only

```bash
docker run -d --name web3 -P nginx:1.27-alpine
docker port web3                            # 80/tcp -> 0.0.0.0:32768 (ចៃដន្យ)

docker run -d --name web4 -p 127.0.0.1:8082:80 nginx:1.27-alpine
docker ps                                   # 127.0.0.1:8082->80/tcp
docker rm -f web2 web3 web4
```

### Lab 4: ដោះស្រាយ postgres ពីមេរៀនទី 4

```bash
docker run -d --name db \
  -e POSTGRES_PASSWORD=secret123 \
  -e POSTGRES_USER=vuthin \
  -e POSTGRES_DB=learn \
  -p 5432:5432 \
  postgres:16

docker ps                                   # Up ✅
docker logs db | tail -5                    # "database system is ready to accept connections"
docker exec -it db psql -U vuthin -d learn  # (winpty បើត្រូវការ)
```
ក្នុង psql:
```sql
\l
CREATE TABLE notes(id serial, body text);
INSERT INTO notes(body) VALUES ('hello docker');
SELECT * FROM notes;
\q
```

### Lab 5: `--env-file`

```bash
mkdir -p ~/Desktop/docker-lap/phase-2/05-ports-and-env/lab
cd ~/Desktop/docker-lap/phase-2/05-ports-and-env/lab

cat > .env <<'EOF'
# MySQL config
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=shop
MYSQL_USER=app
MYSQL_PASSWORD=apppass
EOF

cat > .env.example <<'EOF'
MYSQL_ROOT_PASSWORD=
MYSQL_DATABASE=shop
MYSQL_USER=app
MYSQL_PASSWORD=
EOF

echo ".env" >> ~/Desktop/docker-lap/.gitignore

docker run -d --name mysql --env-file .env -p 3306:3306 mysql:8
docker logs -f mysql          # រង់ចាំ "ready for connections" (port 3306) → Ctrl+C
docker exec -it mysql mysql -uapp -papppass shop -e "SHOW DATABASES;"
```

### Lab 6: env override និង inspect

```bash
docker run --rm -e GREETING=hello alpine sh -c 'echo $GREETING'
export GREETING=from-host
docker run --rm -e GREETING alpine sh -c 'echo $GREETING'      # យកពី host shell

docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}' db
# សង្កេត: password ឃើញជា plain text ⚠️
```

### Lab 7: ទិន្នន័យបាត់? (ត្រៀមសម្រាប់ Phase 4)

```bash
docker rm -f db
docker run -d --name db -e POSTGRES_PASSWORD=secret123 -e POSTGRES_USER=vuthin -e POSTGRES_DB=learn -p 5432:5432 postgres:16
sleep 5
docker exec db psql -U vuthin -d learn -c 'SELECT * FROM notes;'
# ❌ relation "notes" does not exist → ទិន្នន័យបាត់ជាមួយ container
```
👉 **Volumes (Phase 4)** ដោះស្រាយនេះ។

### Lab 8: Cleanup

```bash
docker rm -f web db mysql
docker ps -a
```

---

## 5. សង្ខេប flags `docker run` ដែលអ្នកស្គាល់ឥឡូវ

```bash
docker run -d \                # background
  --name api \                 # ឈ្មោះ
  -p 127.0.0.1:3000:3000 \     # host:container
  -e NODE_ENV=production \     # env
  --env-file .env \            # env file
  --rm \                       # លុបពេលឈប់
  -it \                        # interactive (សម្រាប់ shell)
  IMAGE:TAG [COMMAND]
```

---

## 6. អន្ទាក់ទូទៅ

1. `-p 80:8080` ឬ `-p 8080:80`? **ឆ្វេង = host, ស្តាំ = container** — ចាំ "ខាងក្រៅ:ខាងក្នុង"
2. `port is already allocated` → host port ជាប់ (container ផ្សេង ឬ app Windows) — ប្តូរ host port
3. `EXPOSE` មិនបើក port — ត្រូវ `-p`
4. App ស្តាប់ `127.0.0.1` ក្នុង container → `-p` ក៏ចូលមិនបាន; app ត្រូវស្តាប់ `0.0.0.0` (សំខាន់ណាស់ពេល Dockerize app ផ្ទាល់ខ្លួន Phase 3!)
5. Windows: port 80/443/3306/5432 អាចជាប់ដោយ IIS, MySQL, PostgreSQL ដែលដំឡើងលើ Windows — `netstat -ano | findstr :5432` (PowerShell) ដើម្បីពិនិត្យ
6. Env ជា **string ទាំងអស់** — `-e DEBUG=true` → app ទទួល `"true"` (string)
7. `.env` ក្នុង `--env-file` មិនគាំទ្រ quotes ដូច shell — `KEY="value"` → តម្លៃរួម quotes!

---

## 7. Checklist ✅

- [ ] ខ្ញុំអាចគូររូប host port → NAT → container port
- [ ] ខ្ញុំដឹង `-p` ឆ្វេង/ស្តាំ ដោយមិនចាំបាច់គិត
- [ ] ខ្ញុំបានជួប port conflict និងដោះស្រាយ
- [ ] ខ្ញុំដឹង `EXPOSE` ≠ `-p`
- [ ] ខ្ញុំបាន run postgres + mysql ដោយ env ត្រឹមត្រូវ និងភ្ជាប់ចូលបាន
- [ ] ខ្ញុំបានប្រើ `--env-file` និងដាក់ `.env` ក្នុង `.gitignore`
- [ ] ខ្ញុំឃើញទិន្នន័យបាត់ពេល rm container (Lab 7)

### សំណួរត្រួតពិនិត្យ

1. `docker run -p 3000:80 nginx` — browser បើក port ប៉ុន្មាន?
2. Container ពីរស្តាប់ port 80 ខាងក្នុងទាំងពីរ — ដើរបានទេ?
3. ហេតុអ្វី app ដែល bind `127.0.0.1` ក្នុង container ចូលមិនបានពី host ទោះមាន `-p`?
4. ខុសគ្នារវាង `-p 8080:80` និង `-p 127.0.0.1:8080:80`?
5. ហេតុអ្វីមិនគួរ bake password ចូល image ដោយ `ENV` ក្នុង Dockerfile?

### ចម្លើយ

1. **3000** — ឆ្វេង = host។
2. **បាន** — port ខាងក្នុងជា namespace ផ្ទាល់ខ្លួន; តែ host port ត្រូវខុសគ្នា (`-p 8080:80`, `-p 8081:80`)។
3. Traffic ពី `-p` មកដល់ interface `eth0` របស់ container (172.17.0.x), មិនមែន loopback; app bind តែ `127.0.0.1` → មិនទទួល។ ត្រូវ bind `0.0.0.0`។
4. ទីមួយស្តាប់គ្រប់ interface (ម៉ាស៊ីន LAN ចូលបាន); ទីពីរតែ localhost (សុវត្ថិភាពជាង សម្រាប់ dev)។
5. Image ចែករំលែក/push ទៅ Hub → password នៅក្នុង layer, អ្នកណាក៏ `docker history`/`inspect` ឃើញ; ហើយប្តូរ password ត្រូវ rebuild។ Env ពេល run ឬ secrets ល្អជាង។

---

## 8. ដាក់ចូល repo

```bash
mkdir -p ~/Desktop/docker-lap/phase-2/05-ports-and-env
mv ~/Downloads/notes-lesson5.md ~/Desktop/docker-lap/phase-2/05-ports-and-env/notes.md

cd ~/Desktop/docker-lap
git status                     # ពិនិត្យ: .env មិនគួរឃើញ, .env.example គួរឃើញ
git add .
git commit -m "Phase 2: lesson 5 - port mapping and env variables"
git push
```

**បន្ទាប់:** មេរៀនទី 6 — លំហាត់សរុប Phase 2 (Nginx + Redis + Postgres ដោយ flags ពេញលេញ) រួចបញ្ចប់ Phase 2 🎉