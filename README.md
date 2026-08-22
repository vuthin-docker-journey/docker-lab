# 🐳 Docker Lab — ដំណើររៀន Docker ពី 0 ដល់ Master

> Repo កត់ត្រាការរៀន Docker ដោយខ្លួនឯង ជាភាសាខ្មែរ — notes, លំហាត់, និង projects ពិត។
> គោលដៅ៖ អាច Dockerize project ពិត និងយល់ container internals។

**Environment:** Windows 11 + WSL2 + Docker Desktop + Git Bash
**Docker Hub:** [hub.docker.com/u/vuthin](https://hub.docker.com/u/vuthin)

---

## 📍 វឌ្ឍនភាព

> ផែនទីផ្លូវពេញលេញ (ប្រធានបទលម្អិត + រយៈពេល) នៅ [ROADMAP.md](ROADMAP.md)

| Phase | ប្រធានបទ | ស្ថានភាព |
|:---:|---|:---:|
| 1 | Linux, VM vs Container, Networking, YAML | ✅ |
| 2 | Docker ដំបូង — Architecture, Commands, Image/Container, Ports | 🔄 |
| 3 | Dockerfile — build image ខ្លួនឯង, multi-stage | ⬜ |
| 4 | Volumes & Networks — data persistence, container-to-container | ⬜ |
| 5 | Docker Compose — multi-container apps | ⬜ |
| 6 | Production — registry, CI/CD, security, resource limits | ⬜ |
| 7 | Debugging & Optimization — logs, image size, healthcheck | ⬜ |
| 8 | Orchestration — Swarm, Kubernetes intro | ⬜ |
| 9 | Internals — namespaces, cgroups, containerd, runc | ⬜ |

---

## 📂 Structure

```
docker-lab/
├── README.md
├── phase-1/
│   ├── 01-linux-basics/notes.md
│   ├── 02-virtualization-vs-container/notes.md
│   ├── 03-networking/notes.md
│   └── 04-yaml/notes.md
├── ROADMAP.md
├── phase-2/
│   └── 02-basic-commands/
│       ├── notes.md
│       └── project-my-website/        ← project ដំបូង
│           ├── Dockerfile
│           └── index.html
├── phase-3/ ...
```

**ទម្លាប់:** `phase-X/NN-topic/notes.md` — មេរៀននីមួយៗមាន diagram, command table, លំហាត់, checklist + ចម្លើយ។ Project នៅក្នុង folder របស់មេរៀនដែលប្រើវា។

---

## 🚀 Projects

| # | Project | Stack | Image | Phase |
|:---:|---|---|---|:---:|
| 1 | **my-website** — static site | HTML + nginx:alpine | `vuthin/my-website` | 2 |
| 2 | todo-api — REST API + DB | Spring Boot + PostgreSQL | — | 3-4 |
| 3 | todo-app — full-stack | React + Spring Boot + Postgres + Compose | — | 5 |
| 4 | todo-app + cache/proxy | + Redis + nginx reverse proxy | — | 5-6 |
| 5 | app ពិត | VBola POS / Korean app → CI → VPS | — | 6+ |

### Run project 1

```bash
cd phase-2/02-basic-commands/project-my-website
docker build -t vuthin/my-website:2.1 .
docker run -d --name mysite -p 8080:80 vuthin/my-website:2.1
# http://localhost:8080
```

ឬទាញពី Hub ដោយផ្ទាល់៖
```bash
docker run -d -p 8080:80 vuthin/my-website:2.1
```

---

## 🧠 គំនិតស្នូលដែលរៀនរួច

- **Container = process** (PID 1) ក្នុង namespace — PID 1 ចប់ → container ចប់
- **Image immutable, Container ephemeral** — កែ code = build image ថ្មី, container លុប/បង្កើតបាន
- **Client → Daemon → Registry** — `docker` CLI គ្រាន់តែផ្ញើ REST API; ម៉ាស៊ីន local = cache, registry = ប្រភព
- **Image = layers** ចែករំលែក — `my-website` 62MB ប៉ុន្តែបន្ថែមលើ `nginx:alpine` តែ ~1KB
- **`-p host:container`** — containers ច្រើនស្តាប់ port 80 ដូចគ្នាបាន ព្រោះ network namespace ដាច់ពីគ្នា
- **ឈ្មោះ image** = `registry/namespace/repo:tag` — `nginx` ពិតជា `docker.io/library/nginx:latest`

---

## 🛠 Cheat sheet

```bash
# Container
docker run -d --name NAME -p H:C -e K=V IMAGE:TAG
docker ps -a  |  docker logs -f NAME  |  docker exec -it NAME sh
docker stop NAME  |  docker start NAME  |  docker rm -f NAME

# Image
docker build -t user/repo:tag .  |  docker images  |  docker rmi IMAGE
docker tag SRC DST  |  docker push user/repo:tag  |  docker history IMAGE

# Debug
docker inspect -f '{{.State.Status}}' NAME  |  docker stats  |  docker top NAME

# Cleanup
docker system df  |  docker system prune  |  docker image prune
```

---

## 📚 ឯកសារយោង

- [Docker Docs](https://docs.docker.com)
- [Play with Docker](https://labs.play-with-docker.com)
- [Docker Hub](https://hub.docker.com)

---

*រៀនជាមួយ Claude — notes ទាំងអស់ជាភាសាខ្មែរ។ Commit បន្ទាប់ពីគ្រប់មេរៀន។*