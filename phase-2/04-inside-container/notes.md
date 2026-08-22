# Phase 2 — មេរៀនទី 4: ចូលក្នុង Container (logs, exec, inspect, cp, stats)

> 📁 `phase-2/04-inside-container/notes.md`
> 🎯 គោលដៅ: អាច "មើល" និង "ចូល" ទៅក្នុង container ដែលកំពុង run — ដើម្បី debug, ពិនិត្យ config, ចម្លង file, និងតាមដាន resource

---

## 1. ហេតុអ្វីត្រូវរៀនមេរៀននេះ?

Container ជាប្រអប់បិទជិត។ ពេល app មិនដើរ អ្នកត្រូវការឧបករណ៍ 5 យ៉ាង៖

```
               ┌──────────────────────────────┐
  docker logs ─► stdout / stderr              │
  docker exec ─► run command នៅខាងក្នុង       │  Container
  docker inspect ► metadata (IP, env, mounts) │
  docker cp   ─► ចម្លង file ចេញ/ចូល          │
  docker stats ► CPU / RAM / Net / IO         │
               └──────────────────────────────┘
```

---

## 2. `docker logs` — មើល output

Container មិនមាន terminal ផ្ទាល់; អ្វីដែល process PID 1 សរសេរទៅ **stdout/stderr** ត្រូវ Docker ចាប់ទុក។

| Flag | ន័យ |
|---|---|
| `docker logs NAME` | logs ទាំងអស់តាំងពីចាប់ផ្តើម |
| `-f` / `--follow` | តាមដានផ្ទាល់ (ដូច `tail -f`) — ចុច `Ctrl+C` ចេញ (container នៅ run) |
| `--tail 50` | តែ 50 បន្ទាត់ចុងក្រោយ |
| `-t` / `--timestamps` | បង្ហាញម៉ោង |
| `--since 10m` | តែ 10 នាទីចុងក្រោយ |

💡 **Best practice:** app ក្នុង container គួរសរសេរ log ទៅ stdout, **មិនមែន** ទៅ file ក្នុង container ទេ។ Nginx official image ធ្វើដូច្នេះដោយ symlink `/var/log/nginx/access.log → /dev/stdout`។

---

## 3. `docker exec` — run command នៅខាងក្នុង

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARGS]
```

| Flag | ន័យ |
|---|---|
| `-i` | interactive — រក្សា stdin បើក |
| `-t` | ផ្តល់ pseudo-TTY (terminal) |
| `-it` | ទាំងពីរ → បាន shell ដែលប្រើបាន |
| `-u USER` | run ជា user ផ្សេង (`-u root`) |
| `-w DIR` | working directory |
| `-e KEY=VAL` | env variable សម្រាប់ command នេះប៉ុណ្ណោះ |
| `-d` | run នៅ background |

```bash
docker exec web ls /etc/nginx                 # command ម្តង
docker exec -it web sh                        # ចូល shell (alpine → sh, debian → bash)
docker exec -it web bash                      # error បើ image គ្មាន bash!
```

⚠️ **Git Bash (MINGW64) gotcha:** បើ `-it` ឡើង `the input device is not a TTY` → ដាក់ `winpty` ខាងមុខ៖
```bash
winpty docker exec -it web sh
```

### `exec` vs `attach`

| | `exec -it` | `attach` |
|---|---|---|
| ភ្ជាប់ទៅ | process **ថ្មី** (shell) | PID 1 ដែលមានស្រាប់ |
| `exit` ធ្វើអ្វី | បិទ shell, container នៅ run ✅ | អាចឈប់ container (Ctrl+C ជា PID 1) ⚠️ |
| ប្រើពេល | debug ប្រចាំថ្ងៃ | កម្រ; ចេញដោយ `Ctrl+P, Ctrl+Q` |

---

## 4. `docker inspect` — មើលអ្វីៗទាំងអស់

Return JSON ធំ (config, state, network, mounts)។ ប្រើ **`--format`** (Go template) ដើម្បីទាញតែផ្នែកដែលត្រូវការ៖

```bash
docker inspect web                                            # JSON ទាំងមូល
docker inspect -f '{{.State.Status}}' web                     # running
docker inspect -f '{{.State.Pid}}' web                        # PID នៅ host
docker inspect -f '{{.NetworkSettings.IPAddress}}' web        # 172.17.0.2
docker inspect -f '{{.Config.Image}}' web                     # nginx:1.27-alpine
docker inspect -f '{{json .Config.Env}}' web                  # env list
docker inspect -f '{{json .HostConfig.PortBindings}}' web     # port mapping
docker inspect -f '{{json .Mounts}}' web                      # volumes
docker inspect -f '{{.RestartCount}}' web
```

💡 `docker inspect` ប្រើបានទាំង container, image, volume, network — វាជា "ប្រភពពិត" ពេល `docker ps` មិនគ្រប់គ្រាន់។

---

## 5. `docker cp` — ចម្លង file

```bash
docker cp CONTAINER:/path/in/container  ./local/path    # ចេញ
docker cp ./local/file  CONTAINER:/path/in/container    # ចូល
```

- ធ្វើការបានសូម្បី container **ឈប់** (stopped)
- ចម្លង folder ទាំងមូលបាន
- ⚠️ Git Bash: path ដូច `/etc/nginx` អាចត្រូវ MINGW បំប្លែង → បើមានបញ្ហា ដាក់ `MSYS_NO_PATHCONV=1` ខាងមុខ ឬសរសេរ `//etc/nginx`

---

## 6. `docker stats` និង `docker top`

```bash
docker stats                 # live, containers ទាំងអស់ (Ctrl+C ចេញ)
docker stats --no-stream web # snapshot ម្តង
docker top web               # processes ក្នុង container (មើលពី host)
```

Output `stats`: `CPU %`, `MEM USAGE / LIMIT`, `NET I/O`, `BLOCK I/O`, `PIDS`។
🔗 `LIMIT` ដែលឃើញ = RAM នៃ WSL2 VM ទាំងមូល (ព្រោះមិនទាន់កំណត់ `--memory`) — Phase 6 និង 9 (cgroups) នឹងពន្យល់។

---

## 7. Lab — អនុវត្តន៍

### Lab 1: Logs

```bash
docker run -d --name web -p 8080:80 nginx:1.27-alpine
curl -s localhost:8080 > /dev/null          # ឬបើក browser http://localhost:8080
curl -s localhost:8080/notfound > /dev/null
docker logs web                              # ឃើញ 200 និង 404
docker logs -f -t --tail 5 web               # refresh browser ពីរបីដង → Ctrl+C
```

### Lab 2: ចូល shell និងរុករក

```bash
winpty docker exec -it web sh      # ឬ docker exec -it web sh បើមិន error
# នៅក្នុង container:
ps aux                 # សង្កេត: PID 1 = nginx master, មាន worker ពីរបី
cat /etc/os-release    # Alpine Linux — ខុសពី host!
ls /usr/share/nginx/html
cat /etc/hostname      # = container ID ខ្លី
ip addr 2>/dev/null || cat /etc/hosts
whoami                 # root
exit
docker ps              # web នៅ running ✅
```

### Lab 3: កែ web page ពីខាងក្នុង (ឃើញ CoW ម្តងទៀត)

```bash
docker exec web sh -c 'echo "<h1>Hello from vuthin</h1>" > /usr/share/nginx/html/index.html'
curl localhost:8080
docker diff web
```

### Lab 4: inspect

```bash
docker inspect -f '{{.State.Status}} pid={{.State.Pid}} ip={{.NetworkSettings.IPAddress}}' web
docker inspect -f '{{json .HostConfig.PortBindings}}' web
docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}' web
```

### Lab 5: cp ចេញ–ចូល

```bash
mkdir -p ~/Desktop/docker-lap/phase-2/04-inside-container/lab
cd ~/Desktop/docker-lap/phase-2/04-inside-container/lab

docker cp web:/etc/nginx/nginx.conf ./nginx.conf      # ចេញ — អាន config ពិត
cat nginx.conf

echo "<h1>Copied in from host</h1>" > index.html
docker cp index.html web:/usr/share/nginx/html/index.html   # ចូល
curl localhost:8080
```

### Lab 6: stats & top

```bash
docker stats --no-stream web
docker top web
# បង្កើត load បន្តិច:
for i in $(seq 1 200); do curl -s localhost:8080 > /dev/null; done
docker stats --no-stream web
```

### Lab 7: Debug ស្ថានការណ៍ពិត — container ដែល crash

```bash
docker run -d --name db postgres:16
docker ps                      # db មិនមាន!
docker ps -a                   # Exited (1)
docker logs db                 # "Database is uninitialized and superuser password is not specified"
docker inspect -f '{{.State.ExitCode}} {{.State.Error}}' db
docker rm db
```
👉 នេះជាលំដាប់ debug ស្តង់ដារ: `ps -a` → `logs` → `inspect`។ ដំណោះស្រាយ (`-e POSTGRES_PASSWORD`) នៅមេរៀនទី 5។

### Lab 8: Cleanup

```bash
docker rm -f web
```

---

## 8. លំដាប់ Debug ស្តង់ដារ (ចាំទុក)

```
container មិនដើរ?
   │
   ├─ docker ps -a          → ស្ថានភាព? Exited code ប៉ុន្មាន?
   ├─ docker logs NAME      → app និយាយអ្វី?
   ├─ docker inspect NAME   → config/port/env ត្រឹមត្រូវទេ?
   ├─ docker exec -it NAME sh → ចូលមើលផ្ទាល់ (បើ running)
   └─ docker stats          → អស់ RAM? (OOMKilled = true ក្នុង inspect)
```

---

## 9. អន្ទាក់ទូទៅ

1. `docker exec -it web bash` → `executable file not found` — Alpine គ្មាន bash; ប្រើ `sh`
2. `exec` ចូល container ដែល **stopped** មិនបាន — start វាមុន, ឬប្រើ `docker cp` / `docker run` image ដោយ `sh`
3. `exit` ពី `exec` មិនឈប់ container; `Ctrl+C` ក្នុង `attach` អាចឈប់
4. Git Bash: `winpty` សម្រាប់ `-it`, `MSYS_NO_PATHCONV=1` សម្រាប់ path
5. `docker logs` មើលតែ stdout/stderr — បើ app សរសេរ log ទៅ file ត្រូវ `exec` ចូលអាន

---

## 10. Checklist ✅

- [ ] ខ្ញុំបានប្រើ `logs -f --tail` និងឃើញ request ផ្ទាល់
- [ ] ខ្ញុំបានចូល shell, ឃើញ PID 1 = nginx និង OS = Alpine
- [ ] ខ្ញុំដឹងភាពខុសគ្នា `exec` vs `attach`
- [ ] ខ្ញុំបានទាញ IP, PID, PortBindings ដោយ `inspect --format`
- [ ] ខ្ញុំបាន `cp` ចេញ និងចូល
- [ ] ខ្ញុំបាន debug postgres ដែល crash តាមលំដាប់ ps -a → logs → inspect

### សំណួរត្រួតពិនិត្យ

1. ហេតុអ្វី `exit` ពី `docker exec -it` មិនធ្វើឲ្យ container ឈប់?
2. ហេតុអ្វី app ក្នុង container គួរ log ទៅ stdout ជាជាង file?
3. `docker cp` ធ្វើការលើ container ដែល stopped ទេ? ហេតុអ្វី?
4. `docker logs db` ទទេ ប៉ុន្តែ container ឈប់ — ត្រូវមើលអ្វីបន្ទាប់?
5. `ps aux` ក្នុង container ឃើញ process តិចណាស់ ធៀបនឹង host — ពន្យល់។

### ចម្លើយ

1. `exec` បង្កើត **process ថ្មី** ក្នុង container; container រស់តាម **PID 1** ប៉ុណ្ណោះ។ shell ចេញ → PID 1 នៅដដែល។
2. Docker ចាប់ stdout/stderr ដោយស្វ័យប្រវត្តិ → `docker logs`, log drivers, Compose/Kubernetes សុទ្ធតែអានពីទីនោះ; file ក្នុង container បាត់ពេលលុប និងពិបាកចូលមើល។
3. **បាន** — វាអាន/សរសេរ filesystem (writable layer) ដោយផ្ទាល់, មិនត្រូវការ process run ទេ។
4. `docker inspect -f '{{.State.ExitCode}} {{.State.Error}} {{.State.OOMKilled}}'` — exit code (137 = killed/OOM, 126/127 = command មិនរក) និង error ពី Docker ផ្ទាល់។
5. **PID namespace** — container មើលឃើញតែ process របស់ខ្លួន (PID 1 = app របស់វា), មិនឃើញ host។ (Phase 9 ស៊ីជម្រៅ)

---

## 11. ដាក់ចូល repo

```bash
mkdir -p ~/Desktop/docker-lap/phase-2/04-inside-container
mv ~/Downloads/notes-lesson4.md ~/Desktop/docker-lap/phase-2/04-inside-container/notes.md

cd ~/Desktop/docker-lap
git add .
git commit -m "Phase 2: lesson 4 - logs, exec, inspect, cp, stats"
git push
```

**បន្ទាប់:** មេរៀនទី 5 — Port mapping (`-p`) និង Environment variables (`-e`) — ដោះស្រាយ postgres ដែល crash នៅ Lab 7 🐳