# 🐳 Roadmap — ផ្លូវសិក្សា Docker ពី 0 ដល់ Master

> ផែនទីបង្ហាញផ្លូវពេញលេញ។ វឌ្ឍនភាពបច្ចុប្បន្នមើលនៅ [README.md](README.md)។

---

## ដំណាក់កាលទី 1: មូលដ្ឋានគ្រឹះ (1-2 សប្តាហ៍)

មុននឹងចាប់ផ្តើម Docker អ្នកគួរយល់ដឹងអំពី៖

- **Linux basics** — commands ដូចជា `ls`, `cd`, `chmod`, `ps`, `kill`, process management
- **Virtualization vs Containerization** — ភាពខុសគ្នារវាង VM និង Container
- **Networking basics** — IP, Port, DNS, localhost
- **YAML syntax** — ព្រោះនឹងប្រើច្រើននៅពេលក្រោយ

## ដំណាក់កាលទី 2: Docker ដំបូង (2-3 សប្តាហ៍)

- ដំឡើង Docker Desktop / Docker Engine
- យល់ពី Architecture: Docker Daemon, Docker Client, Registry (Docker Hub)
- Commands សំខាន់ៗ៖
  - `docker run`, `docker ps`, `docker stop`, `docker rm`
  - `docker pull`, `docker push`, `docker images`, `docker rmi`
  - `docker logs`, `docker exec -it`
- យល់ពី **Image vs Container** — Image ជាពុម្ព ចំណែក Container ជា instance ដែលកំពុងដំណើរការ
- Port mapping (`-p 8080:80`) និង Environment variables (`-e`)

## ដំណាក់កាលទី 3: Dockerfile និង Image Building (2-3 សប្តាហ៍)

- សរសេរ Dockerfile ដោយខ្លួនឯង៖ `FROM`, `RUN`, `COPY`, `WORKDIR`, `CMD`, `ENTRYPOINT`, `EXPOSE`, `ENV`, `ARG`
- យល់ពី **Layer caching** — របៀបដែល Docker cache layers ដើម្បី build លឿន
- **Multi-stage builds** — កាត់បន្ថយទំហំ image (សំខាន់ណាស់!)
- `.dockerignore` — មិនអោយ copy files ដែលមិនចាំបាច់
- Best practices៖ ប្រើ base image តូច (alpine, slim), ដាក់ layer ដែលផ្លាស់ប្តូរញឹកញាប់នៅក្រោម
- **លំហាត់៖** Dockerize កម្មវិធីផ្ទាល់ខ្លួន (Node.js, Python, Go, Java...)

## ដំណាក់កាលទី 4: Data និង Networking (2 សប្តាហ៍)

- **Volumes** — `docker volume create`, bind mounts vs named volumes vs tmpfs
- Data persistence សម្រាប់ Database (PostgreSQL, MySQL, MongoDB ក្នុង container)
- **Docker Networks**៖
  - `bridge` (default), `host`, `none`, custom networks
  - Container-to-container communication តាមរយៈ network name
  - `docker network create`, `docker network inspect`

## ដំណាក់កាលទី 5: Docker Compose (2-3 សប្តាហ៍)

- សរសេរ `docker-compose.yml` — services, volumes, networks, `depends_on`
- `docker compose up -d`, `docker compose down`, `docker compose logs`
- Environment files (`.env`)
- Healthchecks និង restart policies
- **លំហាត់ជាក់ស្តែង៖** បង្កើត full-stack app (Frontend + Backend + Database + Redis) ជាមួយ Compose

## ដំណាក់កាលទី 6: កម្រិតមធ្យមខ្ពស់ (3-4 សប្តាហ៍)

- **Docker Registry** — private registry, Docker Hub, GitHub Container Registry (GHCR), AWS ECR
- Image tagging strategy — semantic versioning, `latest` tag pitfalls
- **Security**៖
  - មិន run ជា root user (`USER` directive)
  - Image scanning (Trivy, Docker Scout)
  - Secrets management — មិនដាក់ password ក្នុង image
  - Read-only filesystems, capabilities dropping
- Resource limits — `--memory`, `--cpus`
- Logging drivers និង monitoring (cAdvisor, Prometheus + Grafana)

## ដំណាក់កាលទី 7: CI/CD Integration (2-3 សប្តាហ៍)

- Build និង push image ដោយស្វ័យប្រវត្តិជាមួយ GitHub Actions ឬ GitLab CI
- BuildKit និង `docker buildx` — multi-platform builds (amd64, arm64)
- Automated testing ក្នុង container
- Deployment pipeline៖ build → test → scan → push → deploy

## ដំណាក់កាលទី 8: Orchestration (4-6 សប្តាហ៍)

- Docker Swarm (ស្វែងយល់ខ្លីៗ ដើម្បីយល់ concept)
- **Kubernetes** (ផ្លូវបន្តដ៏សំខាន់)៖
  - Pods, Deployments, Services, Ingress
  - ConfigMaps, Secrets
  - minikube ឬ kind សម្រាប់រៀននៅ local
- យល់ពីពេលណាគួរប្រើ Compose vs Swarm vs Kubernetes

## ដំណាក់កាលទី 9: កម្រិត Master (បន្តរហូត)

- **Container internals** — namespaces, cgroups, union filesystems (OverlayFS)
- Distroless images និង scratch images
- Docker in Docker (DinD) និង alternatives (Podman, containerd, Buildah)
- Debugging techniques — `docker inspect`, `docker stats`, `docker events`
- Performance tuning និង production troubleshooting
- ចូលរួម contribute ឬអានករណីសិក្សា production ជាក់ស្តែង

---

## 📚 ធនធានសិក្សា (Resources)

- [Docker Official Docs](https://docs.docker.com) — ល្អបំផុត
- [Play with Docker](https://labs.play-with-docker.com) — អនុវត្តដោយឥតគិតថ្លៃក្នុង browser
- YouTube: TechWorld with Nana, NetworkChuck
- សៀវភៅ: *Docker Deep Dive* ដោយ Nigel Poulton
