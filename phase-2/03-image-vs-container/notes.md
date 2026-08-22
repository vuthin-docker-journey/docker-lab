# Phase 2 — មេរៀនទី 3: Image vs Container

> 📁 `phase-2/03-image-vs-container/notes.md`
> 🎯 គោលដៅ: យល់ច្បាស់ថា Image ជាអ្វី, Container ជាអ្វី, Layer ដំណើរការយ៉ាងណា និងគ្រប់គ្រង image ដោយ `pull / images / tag / rmi / push / history / inspect`

---

## 1. រូបភាពធំ (Big Picture)

```
        Docker Hub (Registry)
        ┌──────────────────────┐
        │  nginx:1.27          │
        │  postgres:16         │
        │  vuthin/test-regi    │
        └──────────┬───────────┘
                   │ docker pull          ▲ docker push
                   ▼                      │
        ┌──────────────────────┐          │
        │   Image (local)      │ ◄── docker build / docker tag
        │   read-only layers   │
        └──────────┬───────────┘
                   │ docker run (ច្រើនដង ក៏បាន)
          ┌────────┼────────┐
          ▼        ▼        ▼
     Container  Container  Container
       web1       web2       web3
    (writable layer រៀងៗខ្លួន)
```

**ពាក្យប្រៀបធៀប:**

| ការពិត | Docker |
|---|---|
| ពុម្ពនំ (mold) | **Image** — read-only, ប្រើម្តងហើយម្តងទៀត |
| នំដែលចាក់ចេញពីពុម្ព | **Container** — មាន state ផ្ទាល់ខ្លួន, អាចខូច/លុបបាន |
| ហាងលក់ពុម្ព | **Registry** (Docker Hub) |
| Class ក្នុង OOP | Image |
| Object / instance | Container |

---

## 2. Image ជាអ្វី?

Image = **បណ្តុំ read-only layers** + **metadata** (CMD, ENV, EXPOSE, WORKDIR...)។

- មិនអាចកែបាន (immutable) — បើចង់ប្តូរ ត្រូវ build image ថ្មី
- កំណត់ដោយ **ID (sha256 digest)** និង **tag** (ឈ្មោះងាយចាំ)
- ធ្វើពី layers ដែល **share គ្នាបាន** រវាង image ផ្សេងៗ

### ឈ្មោះ image ពេញលេញ

```
docker.io/vuthin/test-regi:latest
└──┬───┘ └──┬─┘ └───┬───┘ └──┬─┘
registry  user   repository  tag
```

- បើមិនសរសេរ registry → `docker.io` (Docker Hub)
- បើមិនសរសេរ user → official image (`library/nginx`)
- បើមិនសរសេរ tag → `latest` ⚠️ (`latest` មិនមែនមានន័យថា "ថ្មីបំផុត" ទេ — វាគ្រាន់តែជា tag default)

---

## 3. Container ជាអ្វី?

Container = **Image + writable layer មួយនៅលើកំពូល + process កំពុង run (ឬបានឈប់)**។

```
┌────────────────────────┐
│  Container layer (R/W) │ ← ឯកសារដែលអ្នកសរសេរក្នុង container ទៅទីនេះ
├────────────────────────┤
│  Layer 3: COPY app     │ ┐
│  Layer 2: RUN apt ...  │ ├ Image (read-only)
│  Layer 1: FROM debian  │ ┘
└────────────────────────┘
```

**ចំណុចសំខាន់:**
- `docker rm container` → លុបតែ writable layer ប៉ុណ្ណោះ, image នៅដដែល
- Container 10 ក្នុង image តែមួយ → share layers read-only ទាំងអស់, ចំណាយ disk តែ writable layer × 10
- ទិន្នន័យក្នុង writable layer **បាត់** ពេល container ត្រូវបានលុប (ដំណោះស្រាយ: Volumes — Phase 4)

---

## 4. Layers និង Copy-on-Write (CoW)

Layer នីមួយៗ = ការផ្លាស់ប្តូរ filesystem មួយជំហាន (diff)។

- **អាន** file → Docker រកពីលើចុះក្រោម, layer ណាមាន file នោះ → ប្រើ
- **សរសេរ/កែ** file ដែលស្ថិតក្នុង image layer → Docker **copy** file នោះឡើងមក writable layer រួចទើបកែ (Copy-on-Write)
- **លុប** file ក្នុង image layer → Docker ដាក់ "whiteout" marker ក្នុង writable layer (file នៅក្នុង image ដដែល តែមើលមិនឃើញ)

💡 នេះហើយមូលហេតុដែល `docker pull` image ទី 2 លឿនជាងទី 1 — layers ដែលមានរួចហើយ (`Already exists`) មិនត្រូវ download ទេ។

---

## 5. Commands សំខាន់

| Command | ធ្វើអ្វី | ឧទាហរណ៍ |
|---|---|---|
| `docker pull IMAGE[:TAG]` | ទាញ image ពី registry | `docker pull nginx:1.27-alpine` |
| `docker images` / `docker image ls` | បង្ហាញ images local | `docker images` |
| `docker image inspect IMAGE` | មើល metadata (JSON) | `docker image inspect nginx` |
| `docker history IMAGE` | មើល layers និងទំហំរបស់វា | `docker history nginx` |
| `docker tag SRC TARGET` | ដាក់ឈ្មោះ/tag ថ្មី (មិន copy ទេ) | `docker tag nginx vuthin/web:v1` |
| `docker rmi IMAGE` / `docker image rm` | លុប image | `docker rmi nginx:1.27-alpine` |
| `docker image prune` | លុប dangling images (`<none>`) | `docker image prune` |
| `docker image prune -a` | លុប image ទាំងអស់ដែលគ្មាន container ប្រើ | ⚠️ ប្រយ័ត្ន |
| `docker push IMAGE:TAG` | បញ្ជូនទៅ registry | `docker push vuthin/web:v1` |
| `docker search TERM` | ស្វែងរកនៅ Docker Hub | `docker search redis` |
| `docker system df` | មើល disk ដែល images/containers/volumes ប្រើ | `docker system df` |
| `docker save / load` | export/import image ជា `.tar` | `docker save nginx -o nginx.tar` |

### អាន output `docker images`

```
REPOSITORY          TAG            IMAGE ID       CREATED        SIZE
nginx               latest         3b25b682ea82   2 weeks ago    192MB
nginx               1.27-alpine    a9f1d2c3e4f5   2 weeks ago    43MB
vuthin/test-regi    latest         3b25b682ea82   2 weeks ago    192MB
<none>              <none>         7c8d9e0f1a2b   3 days ago     120MB
```

- `nginx:latest` និង `vuthin/test-regi` មាន **IMAGE ID ដូចគ្នា** → image តែមួយ, ឈ្មោះពីរ (tag មិនចំណាយ disk)
- `<none>` = **dangling image** — layer ចាស់ដែលបាត់ tag (ជាធម្មតាបន្ទាប់ពី rebuild)
- `SIZE` = ទំហំ logical; layers ដែល share គ្នារាប់ម្តងប៉ុណ្ណោះលើ disk ពិត

---

## 6. Lab — អនុវត្តន៍ (Git Bash)

### Lab 1: Pull និងសង្កេត layers

```bash
docker pull nginx:1.27-alpine
# សង្កេត: Pull complete / Already exists, digest sha256:...

docker pull nginx:1.27
# សង្កេត: តើមាន layer "Already exists" ទេ? (alpine vs debian ខុស base → ប្រហែលគ្មាន)

docker images nginx
docker history nginx:1.27-alpine
# បន្ទាត់នីមួយៗ = layer មួយ; <missing> មានន័យថា layer ត្រូវបាន build នៅម៉ាស៊ីនផ្សេង
```

### Lab 2: Image មួយ → Container ច្រើន

```bash
docker run -d --name web1 nginx:1.27-alpine
docker run -d --name web2 nginx:1.27-alpine
docker run -d --name web3 nginx:1.27-alpine

docker ps
docker system df        # មើល "Containers" size vs "Images" size
```

### Lab 3: Copy-on-Write ដោយផ្ទាល់

```bash
# សរសេរ file ក្នុង web1
docker exec web1 sh -c 'echo "hello from web1" > /usr/share/nginx/html/test.txt'

# web2 មើលឃើញទេ?
docker exec web2 cat /usr/share/nginx/html/test.txt
# → No such file  ✅ writable layer ដាច់ដោយឡែក

# មើលអ្វីដែលផ្លាស់ប្តូរក្នុង web1 ធៀបនឹង image
docker diff web1
# A = Added, C = Changed, D = Deleted
```

### Lab 4: Tag មិន copy

```bash
docker tag nginx:1.27-alpine vuthin/myweb:v1
docker tag nginx:1.27-alpine vuthin/myweb:latest
docker images | grep -E "nginx|myweb"
# IMAGE ID ដូចគ្នាទាំង 3 បន្ទាត់
```

### Lab 5: Push ទៅ Docker Hub

```bash
docker login
docker push vuthin/myweb:v1
# សង្កេត: "Mounted from library/nginx" — Hub ដឹងថា layers មានរួចហើយ, មិន upload ម្តងទៀត!
```

### Lab 6: rmi និងកំហុសដែលនឹងជួប

```bash
docker rmi nginx:1.27-alpine
# Error: image is referenced in multiple repositories  → ប្រើ ID ឬលុប tag ម្តងមួយ
docker rmi vuthin/myweb:latest     # Untagged (image នៅដដែល ព្រោះ tag ផ្សេងនៅមាន)

docker rmi vuthin/myweb:v1
# Error: container web1 is using its referenced image → ត្រូវលុប container មុន

docker rm -f web1 web2 web3
docker rmi vuthin/myweb:v1 nginx:1.27-alpine
docker images
```

### Lab 7: Cleanup

```bash
docker image prune          # dangling only
docker system df
```

---

## 7. Image ID vs Digest vs Tag

| | មកពីណា | ផ្លាស់ប្តូរបានទេ? | ប្រើពេលណា |
|---|---|---|---|
| **Tag** (`nginx:1.27`) | មនុស្សដាក់ | ✅ បាន (ចង្អុលទៅ image ថ្មីបាន) | ប្រើប្រចាំថ្ងៃ |
| **Image ID** (`3b25b682ea82`) | sha256 នៃ image config | ❌ | កំណត់ image local ច្បាស់ |
| **Digest** (`nginx@sha256:abc...`) | sha256 នៃ manifest ពី registry | ❌ | Production / CI ដែលត្រូវការភាពច្បាស់ 100% |

```bash
docker pull nginx@sha256:<digest>    # ទាញ image ពិតប្រាកដមួយ, មិនអាចប្តូរបាន
```

⚠️ `latest` អាចផ្លាស់ប្តូរនៅពេលណាក៏បាន — ក្នុង production ប្រើ tag ជាក់លាក់ (`1.27.2`) ឬ digest។

---

## 8. អន្ទាក់ទូទៅ (Gotchas)

1. **`latest` ≠ ថ្មីបំផុត** — គ្រាន់តែ tag default
2. **`docker rm` មិនលុប image** / **`docker rmi` មិនលុប container** — ពីរនេះដាច់ដោយឡែក
3. **Data ក្នុង container បាត់** ពេល `docker rm` — សម្រាប់ database ត្រូវប្រើ volume
4. **`<none>:<none>`** = dangling, លុបបានដោយ `docker image prune`
5. **`docker pull` ដដែលៗមិន download ទេ** បើ digest ដូចគ្នា — "Image is up to date"
6. **Image size** នៅ Docker Hub (compressed) តូចជាង `docker images` (uncompressed)

🔗 **ភ្ជាប់ទៅ Phase 3:** Dockerfile instruction នីមួយៗ (`FROM`, `RUN`, `COPY`) = បង្កើត layer មួយ។ យល់ layers ឥឡូវ → យល់ cache និង image size នៅ Phase 3 ភ្លាម។

---

## 9. Checklist ✅

- [ ] ខ្ញុំអាចពន្យល់ Image vs Container ដោយពាក្យប្រៀបធៀបផ្ទាល់ខ្លួន
- [ ] ខ្ញុំបាន run container 3 ពី image តែមួយ ហើយឃើញ `docker system df`
- [ ] ខ្ញុំបានបង្ហាញ Copy-on-Write ដោយ `docker exec` + `docker diff`
- [ ] ខ្ញុំដឹងថា `docker tag` មិន copy image
- [ ] ខ្ញុំបាន push `vuthin/myweb:v1` ហើយឃើញ "Mounted from"
- [ ] ខ្ញុំបានជួប error `rmi` ទាំងពីរ និងដោះស្រាយបាន
- [ ] ខ្ញុំអាចបែងចែក Tag / Image ID / Digest

### សំណួរត្រួតពិនិត្យ (ឆ្លើយក្នុង notes)

1. បើ `docker rm` container ហើយ, `docker images` នៅបង្ហាញ image ទេ? ហេតុអ្វី?
2. Container 3 ពី image 192MB ចំណាយ disk ប្រហែល 576MB ឬទេ? ពន្យល់។
3. `docker tag a b` ហើយ `docker images` បង្ហាញ 2 បន្ទាត់ — មាន image ប៉ុន្មាន?
4. ហេតុអ្វីបានជា `docker pull nginx:1.27` ជួនកាល "Already exists" ច្រើន layer?
5. ហេតុអ្វី production មិនគួរប្រើ `:latest`?

### ចម្លើយ

1. **នៅបង្ហាញ។** `docker rm` លុបតែ writable layer និង metadata របស់ container; image ជា object ដាច់ដោយឡែក។
2. **ទេ។** Layers read-only 192MB ត្រូវ share; container នីមួយៗបន្ថែមតែ writable layer (ជាធម្មតាពីរបី KB) → ≈192MB + តិចតួច។
3. **Image តែមួយ។** Tag ជា pointer ទៅ Image ID ដូចគ្នា — `docker images` បង្ហាញម្តងក្នុងមួយ tag។
4. ព្រោះ image ចែក **base layers** ដូចគ្នា (ឧ. debian base) ជាមួយ image ដែលមានរួចហើយ; layer កំណត់ដោយ sha256 ដូច្នេះ Docker ដឹងថាមានរួចហើយ។
5. `latest` អាចប្តូរទៅ version ថ្មីគ្រប់ពេល → deploy មិនដូចគ្នារវាង server, bug ដែលធ្វើឡើងវិញមិនបាន។ ប្រើ tag ជាក់លាក់ ឬ digest។

---

## 10. ដាក់ចូល repo

```bash
mkdir -p ~/Desktop/docker-lap/phase-2/03-image-vs-container
mv ~/Downloads/notes-lesson3.md ~/Desktop/docker-lap/phase-2/03-image-vs-container/notes.md

cd ~/Desktop/docker-lap
git add .
git commit -m "Phase 2: lesson 3 - image vs container, layers, tag/push/rmi"
git push
```

**បន្ទាប់:** មេរៀនទី 4 — ចូលក្នុង Container (`logs`, `exec -it`, `inspect`, `cp`, `stats`) 🐳