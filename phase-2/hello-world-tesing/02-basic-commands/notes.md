# មេរៀនទី 2: Docker Commands សំខាន់ៗ

> ដំណាក់កាលទី 2 — Docker ដំបូង | Lesson 02

---

## 1. Container Lifecycle

Container មានវដ្តជីវិតច្បាស់លាស់ — commands ទាំងអស់គ្រាន់តែរំកិល container ពីស្ថានភាពមួយទៅមួយ៖

```
                 docker create
  ┌─────────┐ ──────────────────▶ ┌──────────┐
  │  Image  │                     │ Created  │
  └─────────┘ ──┐                 └────┬─────┘
                │                      │ docker start
                │ docker run           ▼
                └──────────────▶ ┌──────────┐ ◀──── docker unpause
                                 │ Running  │ ────▶ Paused (docker pause)
                                 └────┬─────┘
                     docker stop      │      docker kill
                     (SIGTERM, 10s)   │      (SIGKILL ភ្លាម)
                                      ▼
                                 ┌──────────┐
                                 │  Exited  │ ──── docker start ──▶ Running
                                 └────┬─────┘
                                      │ docker rm
                                      ▼
                                   (លុបចោល)
```

---

## 2. Commands គ្រប់គ្រង Container

### Run

```bash
docker run nginx                     # foreground — terminal ជាប់ ឃើញ logs
docker run -d nginx                  # detached — run background, print container ID
docker run -d --name web nginx       # ដាក់ឈ្មោះ (ជំនួស ID វែង)
docker run -it alpine sh             # interactive + tty — ចូល shell
docker run --rm alpine echo hi       # លុប container ស្វ័យប្រវត្តិពេលចប់
```

| Flag | អត្ថន័យ |
|---|---|
| `-d` | detach — run background |
| `--name` | ដាក់ឈ្មោះ container |
| `-i` | interactive — បើក stdin |
| `-t` | tty — terminal ពិត (ជាធម្មតាប្រើជាមួយ `-i` → `-it`) |
| `--rm` | លុបពេល exit |

### មើល

```bash
docker ps                # containers កំពុង run
docker ps -a             # ទាំងអស់ រួមទាំង Exited
docker ps -q             # តែ IDs (មានប្រយោជន៍សម្រាប់ script)
docker ps -a --filter "status=exited"
```

Columns សំខាន់នៅ `docker ps`៖

| Column | អត្ថន័យ |
|---|---|
| CONTAINER ID | 12 តួអក្សរដំបូង (ប្រើ 3-4 តួដំបូងក៏បាន បើមិនជាន់គ្នា) |
| IMAGE | image ដែលបង្កើតពី |
| COMMAND | process PID 1 |
| STATUS | Up / Exited (code) |
| PORTS | port mapping (មេរៀនទី 4) |
| NAMES | ឈ្មោះ — បើមិនដាក់ Docker ចៃដន្យ (ឧ. `happy_turing`) |

### Stop / Start / Restart

```bash
docker stop web          # SIGTERM → រង់ចាំ 10s → SIGKILL
docker stop -t 30 web    # រង់ចាំ 30s
docker kill web          # SIGKILL ភ្លាម (មិនឱ្យ app សម្អាត)
docker start web         # start ឡើងវិញ (container ដដែល, data ក្នុងវានៅដដែល)
docker restart web       # stop + start
```

### លុប

```bash
docker rm web                        # លុប container (ត្រូវ stop មុន)
docker rm -f web                     # force — stop + rm
docker rm $(docker ps -aq)           # លុបទាំងអស់
docker container prune               # លុប containers Exited ទាំងអស់
```

---

## 3. Commands គ្រប់គ្រង Image

```bash
docker pull nginx                    # ទាញ (latest)
docker pull nginx:1.27-alpine        # ទាញ tag ជាក់លាក់
docker images                        # list images ក្នុងម៉ាស៊ីន
docker images -q                     # តែ IDs
docker rmi nginx:1.27-alpine         # លុប image
docker rmi $(docker images -q)       # លុបទាំងអស់
docker image prune                   # លុប dangling images (<none>)
docker image prune -a                # លុប images ដែលគ្មាន container ប្រើ
```

> **ចំណាំ:** មិនអាច `rmi` image ដែលមាន container (ទោះ Exited) កំពុងប្រើទេ — ត្រូវ `rm` container មុន។

### ទំនាក់ទំនង Image ↔ Container

```
nginx:latest (image 1 ដុំ)
    ├── web1   (container)
    ├── web2   (container)
    └── web3   (container)
```

image មួយ → containers ច្រើន។ លុប container មិនប៉ះ image។ លុប image ត្រូវគ្មាន container ប្រើ។

---

## 4. មើលខាងក្នុង Container

### Logs

```bash
docker logs web            # stdout/stderr ទាំងអស់
docker logs -f web         # follow — ដូច tail -f
docker logs --tail 20 web  # 20 បន្ទាត់ចុងក្រោយ
docker logs -t web         # បង្ហាញ timestamp
```

Docker ចាប់ **stdout/stderr របស់ PID 1** — នេះជាហេតុផលដែល apps ក្នុង container គួរ log ទៅ stdout មិនមែនទៅ file។

### Exec — run command ក្នុង container ដែលកំពុង run

```bash
docker exec web ls /usr/share/nginx/html   # run command មួយ
docker exec -it web sh                     # ចូល shell (alpine/busybox)
docker exec -it web bash                   # ចូល bash (debian/ubuntu)
docker exec -u root -it web sh             # ចូលជា root
```

**`run -it` vs `exec -it` ខុសគ្នា៖**

| | `docker run -it alpine sh` | `docker exec -it web sh` |
|---|---|---|
| បង្កើត container ថ្មី? | បាទ | ទេ — ចូល container ដែលមានស្រាប់ |
| sh ជា PID 1? | បាទ — ចេញ shell → container ចប់ | ទេ — ចេញ shell → container នៅតែ run |

### Inspect / Stats / Top

```bash
docker inspect web                         # JSON លម្អិតទាំងអស់
docker inspect -f '{{.State.Status}}' web  # ទាញតែ field មួយ
docker inspect -f '{{.NetworkSettings.IPAddress}}' web
docker stats                               # CPU/RAM live (ដូច top)
docker top web                             # processes ក្នុង container
docker port web                            # port mapping
```

---

## 5. Commands ប្រព័ន្ធ

```bash
docker system df           # ទំហំ disk ដែល images/containers/volumes ប្រើ
docker system prune        # សម្អាត containers Exited, networks មិនប្រើ, dangling images
docker system prune -a     # + images ទាំងអស់ដែលមិនប្រើ (ប្រយ័ត្ន!)
```

---

## 6. សន្លឹកបញ្ជា (Cheat Sheet)

```
Container:  run  ps  stop  start  restart  kill  rm  logs  exec  inspect  stats  top
Image:      pull  images  rmi  image prune
System:     info  version  system df  system prune
```

Syntax ថ្មី (ណែនាំ) vs ចាស់ — ដូចគ្នាទាំងពីរ៖

| ចាស់ | ថ្មី |
|---|---|
| `docker ps` | `docker container ls` |
| `docker rm` | `docker container rm` |
| `docker images` | `docker image ls` |
| `docker rmi` | `docker image rm` |

---

## 7. លំហាត់អនុវត្ត

```bash
# 1. Lifecycle ពេញ
docker run -d --name web nginx
docker ps
docker stop web
docker ps -a            # Exited (0)
docker start web
docker ps               # Up ម្តងទៀត
docker rm -f web

# 2. Interactive
docker run -it --name box alpine sh
  # ខាងក្នុង container:
  hostname               # ឃើញ container ID
  ps aux                 # ឃើញតែ sh — មិនឃើញ processes host!
  ls /
  echo "hello" > /test.txt
  exit                   # sh ចប់ → container ចប់
docker ps -a             # box: Exited
docker start box
docker exec box cat /test.txt    # file នៅដដែល!
docker rm -f box

# 3. Logs និង exec
docker run -d --name web nginx
docker logs -f web       # បើក terminal ថ្មី ហើយ run បន្ទាត់ខាងក្រោម
docker exec web curl -s localhost   # (បើ curl មាន) ឬ:
docker exec -it web bash
  curl localhost          # ឃើញ HTML "Welcome to nginx"
  exit
docker logs web          # ឃើញ access log ពី request ខាងលើ

# 4. ពិសោធន៍ PID 1
docker run -d --name sleeper alpine sleep 60
docker top sleeper
docker exec sleeper kill 1        # សម្លាប់ PID 1 → ?
docker ps -a                      # sleeper Exited

# 5. សម្អាត
docker rm -f $(docker ps -aq)
docker system df
```

---

## 8. Checklist

- [ ] អាចគូរ lifecycle Created → Running → Exited → លុប
- [ ] យល់ `-d`, `-it`, `--rm`, `--name`
- [ ] យល់ `stop` vs `kill`
- [ ] យល់ `run -it` vs `exec -it`
- [ ] យល់ហេតុអ្វី container ចប់ពេល PID 1 ចប់
- [ ] អាចមើល logs និងចូលក្នុង container បាន
- [ ] ដឹងថា `rmi` បរាជ័យពេលមាន container ប្រើ image

---

## 9. ចម្លើយ Checklist

**1. `stop` vs `kill`?**
`stop` ផ្ញើ SIGTERM ឱ្យ PID 1 ឱ្យឱកាស app សម្អាត (បិទ DB connection, សរសេរ file ចប់) រង់ចាំ 10 វិនាទី ទើបផ្ញើ SIGKILL។ `kill` ផ្ញើ SIGKILL ភ្លាម — process ស្លាប់ដោយមិនបានសម្អាត។ ប្រើ `stop` ជាធម្មតា `kill` តែពេល app គាំង។

**2. `run -it` vs `exec -it`?**
`run` បង្កើត container ថ្មី ហើយ `sh` ជា PID 1 — វាយ `exit` container ចប់។ `exec` run process បន្ថែមក្នុង container ដែលកំពុង run — `exit` គ្រាន់តែបិទ shell ប៉ុណ្ណោះ PID 1 ដើម (ឧ. nginx) នៅដដែល។ សម្រាប់ debug app ដែលកំពុង run ប្រើ `exec`។

**3. ហេតុអ្វី container ចប់ពេល PID 1 ចប់?**
Container = process PID 1 + namespace ព័ទ្ធជុំវិញវា។ គ្មាន process → គ្មានអ្វីត្រូវដាក់ក្នុង namespace → container ចប់។ លំហាត់ទី 4 បង្ហាញ៖ `kill 1` ខាងក្នុង → container Exited ភ្លាម។

**4. ហេតុអ្វី `rmi` បរាជ័យ?**
Container (ទោះ Exited) មាន writable layer ដែលអង្គុយលើ layers របស់ image។ លុប image = លុបគ្រឹះរបស់ container។ ដូច្នេះត្រូវ `rm` container មុន ឬ `rmi -f` (មិនណែនាំ)។

**5. data ក្នុង container នៅដដែលពេល stop/start?**
បាទ — stop មិនលុប writable layer ទេ (លំហាត់ទី 2: `/test.txt` នៅ)។ ប៉ុន្តែ `rm` លុបចោលទាំងអស់។ ចង់រក្សា data ជាអចិន្ត្រៃយ៍ត្រូវប្រើ Volume (Phase 4)។