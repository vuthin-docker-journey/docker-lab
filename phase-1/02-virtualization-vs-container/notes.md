# 📘 មេរៀនទី 2: Virtualization vs Containerization

> ផ្នែកមួយនៃ Phase 1 — ទ្រឹស្តីសំខាន់បំផុតមុននឹងយល់ Docker

---

## 1. បញ្ហាដើម: ហេតុអ្វីត្រូវការទាំងពីរនេះ?

កាលពីមុន (Traditional Deployment):

```
┌─────────────────────────────┐
│   App A    App B    App C   │  ← apps ទាំងអស់នៅលើ OS តែមួយ
├─────────────────────────────┤
│      Operating System       │
├─────────────────────────────┤
│      Physical Server        │
└─────────────────────────────┘
```

**បញ្ហា:**
- App A ត្រូវការ Python 2, App B ត្រូវការ Python 3 → ប៉ះទង្គិចគ្នា (dependency conflict)
- App មួយ crash អាចទាញ server ទាំងមូលដួល
- Server ប្រើ resource មិនអស់ — ខ្ជះខ្ជាយ
- "It works on my machine!" 😤 — ដំណើរការលើម៉ាស៊ីនខ្ញុំ តែ deploy ទៅ server មិនដើរ

---

## 2. Virtualization (Virtual Machines)

### គំនិត

បំបែក physical server មួយ ជា **ម៉ាស៊ីននិម្មិត (VM) ច្រើន** — VM នីមួយៗមាន OS ពេញលេញរៀងៗខ្លួន។

```
┌───────────┐ ┌───────────┐ ┌───────────┐
│   App A   │ │   App B   │ │   App C   │
├───────────┤ ├───────────┤ ├───────────┤
│ Bins/Libs │ │ Bins/Libs │ │ Bins/Libs │
├───────────┤ ├───────────┤ ├───────────┤
│ Guest OS  │ │ Guest OS  │ │ Guest OS  │  ← OS ពេញលេញរៀងៗខ្លួន (GB ច្រើន!)
│ (Ubuntu)  │ │ (CentOS)  │ │ (Windows) │
└───────────┘ └───────────┘ └───────────┘
┌─────────────────────────────────────────┐
│            Hypervisor                   │  ← កម្មវិធីគ្រប់គ្រង VMs
├─────────────────────────────────────────┤
│            Host OS (ឬគ្មាន)             │
├─────────────────────────────────────────┤
│          Physical Server                │
└─────────────────────────────────────────┘
```

### Hypervisor ជាអ្វី?

កម្មវិធីដែលបង្កើត និងគ្រប់គ្រង VMs ដោយបែងចែក hardware (CPU, RAM, Disk) ឱ្យ VM នីមួយៗ។

| ប្រភេទ | ពិពណ៌នា | ឧទាហរណ៍ |
|--------|---------|----------|
| Type 1 (Bare-metal) | ដំណើរការផ្ទាល់លើ hardware | VMware ESXi, Hyper-V, KVM |
| Type 2 (Hosted) | ដំណើរការលើ Host OS | VirtualBox, VMware Workstation |

### ចំណុចខ្លាំង / ខ្សោយ

✅ **ខ្លាំង:**
- Isolation ខ្លាំងបំផុត — VM មួយ crash មិនប៉ះ VM ផ្សេង
- Run OS ខុសគ្នាបានលើម៉ាស៊ីនតែមួយ (Linux + Windows ជាមួយគ្នា)
- សុវត្ថិភាពខ្ពស់

❌ **ខ្សោយ:**
- ធ្ងន់ — VM នីមួយៗត្រូវការ OS ពេញលេញ (ច្រើន GB)
- Boot យឺត (វិនាទីរាប់សិប ដល់នាទី)
- ស៊ី resource ច្រើន (RAM, CPU, Disk)

---

## 3. Containerization (Containers)

### គំនិត

Container ទាំងអស់ **ចែករំលែក Kernel របស់ Host OS ជាមួយគ្នា** — មិនត្រូវការ Guest OS ទេ។ Container ផ្ទុកតែ app + dependencies ប៉ុណ្ណោះ។

```
┌───────────┐ ┌───────────┐ ┌───────────┐
│   App A   │ │   App B   │ │   App C   │
├───────────┤ ├───────────┤ ├───────────┤
│ Bins/Libs │ │ Bins/Libs │ │ Bins/Libs │  ← តូច (MB ប៉ុណ្ណោះ!)
└───────────┘ └───────────┘ └───────────┘
┌─────────────────────────────────────────┐
│      Container Engine (Docker)          │  ← ជំនួស Hypervisor
├─────────────────────────────────────────┤
│         Host OS (Linux Kernel)          │  ← ចែករំលែក kernel តែមួយ
├─────────────────────────────────────────┤
│          Physical Server                │
└─────────────────────────────────────────┘
```

### ចំណុចខ្លាំង / ខ្សោយ

✅ **ខ្លាំង:**
- ស្រាល — គ្មាន Guest OS (MB ជំនួស GB)
- ចាប់ផ្តើមលឿន (មិល្លីវិនាទី ដល់វិនាទី)
- Run containers រាប់សិប/រយ លើម៉ាស៊ីនតែមួយ
- Portable — "Build once, run anywhere" ដោះស្រាយបញ្ហា "works on my machine"

❌ **ខ្សោយ:**
- Isolation ខ្សោយជាង VM (ចែករំលែក kernel)
- Container Linux ត្រូវការ Linux kernel — មិនអាច run Windows container លើ Linux host ផ្ទាល់ទេ
- Kernel មានបញ្ហា → ប៉ះ containers ទាំងអស់

---

## 4. តារាងប្រៀបធៀប ⭐

| លក្ខណៈ | Virtual Machine | Container |
|--------|-----------------|-----------|
| **OS** | Guest OS ពេញលេញរៀងៗខ្លួន | ចែករំលែក Host kernel |
| **ទំហំ** | GB (ធំ) | MB (តូច) |
| **ល្បឿន boot** | នាទី/វិនាទីច្រើន | មិល្លីវិនាទី/វិនាទី |
| **Isolation** | ខ្លាំងបំផុត (hardware-level) | ល្អ (process-level) |
| **Resource** | ស៊ីច្រើន | ស៊ីតិច |
| **ចំនួនលើ server** | រាប់ (តិច) | រាប់សិប/រយ (ច្រើន) |
| **គ្រប់គ្រងដោយ** | Hypervisor | Container Engine (Docker) |
| **Portability** | មធ្យម (image ធំ) | ខ្ពស់បំផុត |
| **ប្រើពេលណា** | ត្រូវការ OS ខុសគ្នា, isolation ខ្លាំង | Microservices, CI/CD, deploy លឿន |

---

## 5. ការប្រៀបធៀបជាឧទាហរណ៍ 🏠

**VM = ផ្ទះឯកជន (House)**
- មានគ្រឹះ, ជញ្ជាំង, ប្រព័ន្ធទឹកភ្លើងរៀងៗខ្លួន (Guest OS)
- ឯកជនភាពខ្ពស់ តែថ្លៃ ហើយសាងសង់យូរ

**Container = បន្ទប់ក្នុងអគារ Apartment**
- ចែករំលែកគ្រឹះ, ប្រព័ន្ធទឹកភ្លើងរួមគ្នា (Host Kernel)
- តូច, ថោក, ចូលនៅបានភ្លាមៗ តែជញ្ជាំងស្តើងជាង (isolation ខ្សោយជាង)

---

## 6. ចំណេះដឹងបន្ថែម: Docker លើ Windows ដំណើរការយ៉ាងណា?

Container Linux ត្រូវការ Linux kernel — ប៉ុន្តែ Windows គ្មាន Linux kernel ទេ!

**ដំណោះស្រាយ:** Docker Desktop លើ Windows ប្រើ **WSL2** (Windows Subsystem for Linux 2) — ដែលជា Linux VM ស្រាលមួយនៅក្នុង Windows។

```
Windows 11
   └── WSL2 (Linux VM ស្រាល)
          └── Docker Engine
                 └── Containers...
```

គួរឱ្យអស់សំណើច: Docker លើ Windows = **Containerization នៅក្នុង Virtualization** 😄 — ទាំងពីរធ្វើការជាមួយគ្នា មិនមែនជាសត្រូវគ្នាទេ!

---

## 7. ពាក្យគន្លឹះត្រូវចាំ

| ពាក្យ | អត្ថន័យ |
|-------|---------|
| **Kernel** | ស្នូលរបស់ OS — គ្រប់គ្រង hardware, memory, processes |
| **Hypervisor** | កម្មវិធីបង្កើត/គ្រប់គ្រង VMs |
| **Container Engine** | កម្មវិធី run containers (Docker, Podman, containerd) |
| **Image** | គំរូ (template) សម្រាប់បង្កើត container |
| **Isolation** | ការបំបែកមិនឱ្យ apps ប៉ះពាល់គ្នា |
| **WSL2** | Linux kernel ស្រាលនៅក្នុង Windows |

---

## ✅ Checklist លំហាត់

- [x] គូរ diagram VM vs Container ដោយខ្លួនឯង (លើក្រដាស ឬ draw.io)
- [x] ឆ្លើយសំណួរ: ហេតុអ្វី container boot លឿនជាង VM?
- [x] ឆ្លើយសំណួរ: ហេតុអ្វី container Linux មិនអាច run ផ្ទាល់លើ Windows?
- [x] ឆ្លើយសំណួរ: ពេលណាគួរប្រើ VM ជំនួស Container?
- [x] Commit: `git commit -m "Phase 1: virtualization vs containerization notes"`

---

# 📝 ចម្លើយ Checklist

## លំហាត់ទី 1: Diagram VM vs Container

```
        VIRTUAL MACHINE                      CONTAINER
┌───────────┐ ┌───────────┐      ┌───────┐ ┌───────┐ ┌───────┐
│   App A   │ │   App B   │      │ App A │ │ App B │ │ App C │
├───────────┤ ├───────────┤      └───────┘ └───────┘ └───────┘
│ Guest OS  │ │ Guest OS  │      ┌───────────────────────────┐
│   (GB!)   │ │   (GB!)   │      │   Docker Engine           │
└───────────┘ └───────────┘      ├───────────────────────────┤
┌─────────────────────────┐      │   Host OS (shared kernel) │
│      Hypervisor         │      ├───────────────────────────┤
├─────────────────────────┤      │   Physical Server         │
│      Host OS            │      └───────────────────────────┘
├─────────────────────────┤
│   Physical Server       │       Guest OS រៀងខ្លួន → ធ្ងន់, boot យឺត
└─────────────────────────┘       ចែករំលែក kernel → ស្រាល, boot លឿន
```

---

## សំណួរទី 1: ហេតុអ្វី Container boot លឿនជាង VM?

**ចម្លើយ:** ព្រោះ Container **មិនត្រូវ boot OS ទេ**។

- **VM boot** = ដំណើរការ Guest OS ពេញលេញ: BIOS/firmware និម្មិត → kernel loading → system services → រួចទើប app ចាប់ផ្តើម។ ដូចបើកកុំព្យូទ័រថ្មីមួយ — ចំណាយពេលរាប់សិបវិនាទីដល់នាទី។
- **Container start** = គ្រាន់តែ**ចាប់ផ្តើម process ថ្មីមួយ**លើ kernel ដែលកំពុងដំណើរការស្រាប់។ Kernel មិនចាំបាច់ boot ឡើងវិញទេ ព្រោះ Host OS boot រួចហើយ។ លឿនដូចបើក program ធម្មតា — មិល្លីវិនាទីប៉ុណ្ណោះ។

> 🏗️ VM = សាងសង់ផ្ទះថ្មីមុននឹងចូលនៅ / 🔑 Container = ចូលបន្ទប់ apartment ដែលអគារមានស្រាប់

---

## សំណួរទី 2: ហេតុអ្វី Container Linux មិនអាច run ផ្ទាល់លើ Windows?

**ចម្លើយ:** ព្រោះ Container **មិនមាន OS ផ្ទាល់ខ្លួន** — វា**ខ្ចី kernel ពី Host** មកប្រើ។

1. Container Linux ផ្ទុកតែ app + libraries ហើយពឹងផ្អែកលើ **Linux kernel** ដើម្បីហៅ system calls (គ្រប់គ្រង process, memory, network...)
2. Windows ប្រើ **NT kernel** ដែលមាន system calls ខុសពី Linux ទាំងស្រុង — ដូចនិយាយភាសាខុសគ្នា 🗣️
3. App ក្នុង container Linux ហៅ Linux system call → NT kernel ស្តាប់មិនយល់ → មិនដំណើរការ

**ដំណោះស្រាយ = WSL2:** Docker Desktop ដាក់ Linux kernel ស្រាលមួយ (VM តូច) ចូលក្នុង Windows ដើម្បីឱ្យ containers Linux មាន kernel ត្រឹមត្រូវប្រើ។

---

## សំណួរទី 3: ពេលណាគួរប្រើ VM ជំនួស Container?

| ស្ថានភាព | ហេតុផល |
|----------|---------|
| **ត្រូវការ OS ខុសគ្នា** | ឧ. run Windows Server លើ Linux host — container ធ្វើមិនបានព្រោះចែករំលែក kernel |
| **ត្រូវការ isolation ខ្លាំងបំផុត** | Banking, multi-tenant hosting — VM បំបែកនៅកម្រិត hardware, សុវត្ថិភាពជាង |
| **Legacy applications** | Apps ចាស់ៗត្រូវការ OS version ជាក់លាក់ ឬ kernel modules ពិសេស |
| **ត្រូវការ kernel ផ្ទាល់ខ្លួន** | Testing kernel, drivers, OS-level software |
| **Compliance/ច្បាប់តម្រូវ** | ធនាគារ, សុខាភិបាល ខ្លះតម្រូវ VM-level isolation |

**ចម្លើយខ្លី:**
- Container = **ល្បឿន + ស្រាល + portability** (microservices, CI/CD)
- VM = **isolation ខ្លាំង ឬ OS ខុសគ្នា**
- ការពិត: ក្រុមហ៊ុនធំៗប្រើ**ទាំងពីរ** — containers run ក្នុង VMs លើ cloud (AWS EC2 = VM → Docker containers ក្នុងនោះ)!