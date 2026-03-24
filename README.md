# Linux Administration: Zero to Hero
### หลักสูตร 3 วัน + บทเสริม · AlmaLinux 9 Based

---

> **วิธีใช้เอกสารนี้**
> แต่ละบทมีคำอธิบายแนวคิดและ Assignment ให้ทำ ผู้เรียนเป็นคนเพิ่ม Step การปฏิบัติด้วยตัวเอง เพื่อให้เกิดการค้นคว้าและเรียนรู้อย่างแท้จริง

> **สภาพแวดล้อมที่ใช้ในหลักสูตรนี้**
> - OS: **AlmaLinux 9** (RHEL-compatible)
> - Package Manager: **dnf**
> - Firewall: **firewalld**
> - Init System: **systemd**

---

## Day 1 — Linux Fundamentals & File System

---

### บทที่ 1.1 — Linux Architecture & Distributions

Linux ไม่ใช่ระบบปฏิบัติการเดียว แต่เป็น "ครอบครัว" ของระบบที่ใช้ **Kernel** ตัวเดียวกัน โดย Kernel ทำหน้าที่เป็นตัวกลางระหว่าง Hardware (CPU, RAM, Disk) กับ Software ที่เราใช้งาน

ลำดับชั้นของระบบ Linux มีดังนี้:

```
[ Hardware ]
     ↓
[ Kernel ]   ← หัวใจของ Linux
     ↓
[ Shell ]    ← ช่องทางที่เราสั่งงาน
     ↓
[ Applications ]
```

**Distribution (Distro)** คือ Linux ที่ถูกบรรจุพร้อมเครื่องมือและ Package Manager ที่ต่างกัน เช่น:

| Distro | ใช้ที่ไหน | Package Manager | Family |
|--------|-----------|-----------------|--------|
| **AlmaLinux** | Enterprise Server, Production | `dnf` | RHEL |
| Rocky Linux | Enterprise Server | `dnf` | RHEL |
| CentOS Stream | Upstream testing | `dnf` | RHEL |
| Ubuntu | Server, Desktop, Cloud | `apt` | Debian |
| Debian | Stable Server | `apt` | Debian |

**ทำไมต้องใช้ AlmaLinux?**

AlmaLinux เกิดขึ้นหลังจาก Red Hat ประกาศยุติ CentOS 8 ในปี 2021 เป็น distribution ที่ binary-compatible กับ RHEL (Red Hat Enterprise Linux) 100% หมายความว่าโปรแกรมที่ทำงานได้บน RHEL จะทำงานได้บน AlmaLinux เหมือนกันทุกประการ เหมาะมากสำหรับ Enterprise Server ที่ต้องการความเสถียรระยะยาว (10 ปี support)

```
RHEL (ต้นทาง, มีค่าใช้จ่าย)
   ↓  binary-compatible
AlmaLinux / Rocky Linux (ฟรี, community-driven)
```

#### 📝 Assignment 1.1

> ค้นหาความแตกต่างระหว่าง AlmaLinux และ Ubuntu ในแง่ของการใช้งาน Production Server ว่ามีข้อดีข้อเสียต่างกันอย่างไร และค้นหาว่า AlmaLinux 9 จะได้รับ support ถึงปีไหน เขียนสรุปสั้นๆ ความยาวไม่เกิน 10 บรรทัด

---

### บทที่ 1.2 — Shell Basics & Navigation

**Shell** คือ Command-Line Interface (CLI) ที่รับคำสั่งจากเราแล้วส่งต่อให้ Kernel ทำงาน Shell ที่พบบ่อยที่สุดคือ **Bash (Bourne Again Shell)**

เมื่อเปิด Terminal จะเห็นสัญลักษณ์แบบนี้:

```
[username@hostname ~]$
```

> บน AlmaLinux จะใช้รูปแบบ `[user@host dir]$` ซึ่งต่างจาก Ubuntu ที่ใช้ `user@host:dir$`

ความหมายของแต่ละส่วน:
- `username` — ชื่อ user ที่ login อยู่
- `hostname` — ชื่อเครื่อง
- `~` — ตำแหน่งปัจจุบัน (`~` แทน Home Directory)
- `$` — user ทั่วไป (ถ้าเป็น `#` คือ root)

คำสั่งพื้นฐานที่ต้องรู้จัก:

| คำสั่ง | ความหมาย | ตัวอย่าง |
|--------|----------|---------|
| `pwd` | แสดง path ปัจจุบัน | `pwd` |
| `ls` | แสดงรายการไฟล์ | `ls -la` |
| `cd` | เปลี่ยน directory | `cd /home` |
| `man` | เปิด manual ของคำสั่ง | `man ls` |
| `history` | ดูคำสั่งที่ใช้ไป | `history` |
| `clear` | ล้างหน้าจอ | `clear` |
| `whoami` | แสดงชื่อ user ปัจจุบัน | `whoami` |

**เคล็ดลับ:** กด `Tab` เพื่อ Auto-complete ชื่อไฟล์หรือคำสั่ง และกด `↑ ↓` เพื่อเรียกคำสั่งที่ใช้ไปแล้ว

#### 📝 Assignment 1.2

> เปิด Terminal แล้วสำรวจระบบด้วยคำสั่งพื้นฐาน จากนั้นตอบคำถามต่อไปนี้:
> 1. Home Directory ของ user ของคุณอยู่ที่ path ไหน?
> 2. คำสั่ง `ls -la` แตกต่างจาก `ls` อย่างไร?
> 3. ทดลองใช้ `man ls` แล้วบอกว่า option `-h` ทำอะไร?

---

### บทที่ 1.3 — Filesystem Hierarchy Standard (FHS)

Linux มีโครงสร้างไฟล์เป็น **ต้นไม้** (Tree Structure) ที่เริ่มจาก Root `/` และแตกกิ่งออกไป ทุก directory มีวัตถุประสงค์ที่ชัดเจน:

```
/
├── bin/       ← คำสั่งพื้นฐาน (symlink ไปยัง /usr/bin บน AlmaLinux 9)
├── etc/       ← ไฟล์ config ของระบบทั้งหมด
├── home/      ← Home directory ของ user แต่ละคน
├── var/       ← ข้อมูลที่เปลี่ยนแปลงบ่อย (log, cache, spool)
├── tmp/       ← ไฟล์ชั่วคราว (ถูกลบหลัง reboot)
├── usr/       ← โปรแกรมและ library ที่ติดตั้งเพิ่ม
├── root/      ← Home ของ root user
├── proc/      ← ข้อมูล process ที่ run อยู่ (virtual filesystem)
├── dev/       ← Device files (disk, terminal)
├── mnt/       ← จุด mount disk หรือ storage ชั่วคราว
└── sys/       ← ข้อมูล kernel และ hardware (virtual)
```

> **AlmaLinux 9 เฉพาะ:** `/bin`, `/sbin`, `/lib`, `/lib64` เป็น symlink ไปยัง `/usr/bin`, `/usr/sbin`, `/usr/lib`, `/usr/lib64` ตามมาตรฐาน UsrMerge ซึ่งทำให้ระบบมีความสะอาดและง่ายต่อการจัดการ

**หลักการสำคัญ:** ใน Linux ทุกอย่างคือไฟล์ แม้แต่ Hardware Device ก็ถูกแสดงเป็นไฟล์ใน `/dev`

#### 📝 Assignment 1.3

> เข้าไปสำรวจแต่ละ directory ด้วยคำสั่ง `ls` และตอบคำถาม:
> 1. ไฟล์ config ของ network อยู่ใน directory ไหน? (บน AlmaLinux 9 ใช้ NetworkManager)
> 2. Log ของ system อยู่ที่ path อะไร?
> 3. `/proc/cpuinfo` บอกข้อมูลอะไรเกี่ยวกับ CPU?
> 4. ทดสอบว่า `/bin` บน AlmaLinux 9 เป็น symlink ไปที่ไหน โดยใช้คำสั่ง `ls -la /bin`

---

### บทที่ 1.4 — File Operations & Text Processing

การจัดการไฟล์คือทักษะพื้นฐานที่ใช้ทุกวัน ทั้งการสร้าง คัดลอก ย้าย ลบ และค้นหา

**คำสั่งจัดการไฟล์:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `touch` | สร้างไฟล์เปล่า |
| `mkdir` | สร้าง directory |
| `cp` | คัดลอกไฟล์ |
| `mv` | ย้าย/เปลี่ยนชื่อไฟล์ |
| `rm` | ลบไฟล์ |
| `find` | ค้นหาไฟล์ในระบบ |

**คำสั่ง Text Processing:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `cat` | แสดงเนื้อหาไฟล์ |
| `less` | แสดงไฟล์แบบ scroll ได้ |
| `grep` | ค้นหาข้อความในไฟล์ |
| `sed` | แก้ไขข้อความแบบ batch |
| `awk` | ประมวลผลข้อความแบบ column |
| `wc` | นับบรรทัด/คำ/ตัวอักษร |

**Pipe (`|`)** คือการนำ Output ของคำสั่งหนึ่งส่งเป็น Input ให้อีกคำสั่ง:

```bash
# ตัวอย่าง: หาบรรทัดที่มีคำว่า "error" ใน /var/log/messages
cat /var/log/messages | grep "error"
```

> บน AlmaLinux 9 system log หลักอยู่ที่ `/var/log/messages` ไม่ใช่ `/var/log/syslog` แบบ Ubuntu

**Redirection (`>`, `>>`)** คือการส่ง Output ไปยังไฟล์:

```bash
echo "hello" > file.txt    # เขียนทับ
echo "world" >> file.txt   # เพิ่มต่อท้าย
```

> ⚠️ **คำเตือน:** `rm` ลบถาวรโดยไม่มี Recycle Bin ใช้ด้วยความระมัดระวัง

#### 📝 Assignment 1.4

> ทำแบบฝึกหัดต่อไปนี้:
> 1. สร้าง directory ชื่อ `myproject` แล้วสร้างไฟล์ 3 ไฟล์ข้างใน
> 2. ใช้ `grep` ค้นหาคำว่า "root" ใน `/etc/passwd` และนับว่ามีกี่บรรทัด
> 3. เขียน command ที่แสดงเฉพาะชื่อไฟล์ใน `/etc` ที่ลงท้ายด้วย `.conf`

---

### บทที่ 1.5 — Permissions, Ownership & Special Bits

Linux เป็นระบบ Multi-user ทุกไฟล์จึงมี **เจ้าของ (Owner)** และ **สิทธิ์การเข้าถึง (Permission)** ที่ควบคุมว่าใครทำอะไรได้บ้าง

เมื่อใช้คำสั่ง `ls -l` จะเห็นข้อมูลแบบนี้:

```
-rwxr-xr-- 1 alice devs 4096 Jan 1 12:00 script.sh
│└─┬──┘└─┬─┘└─┬─┘
│  │     │    └── Other: r-- (อ่านได้อย่างเดียว)
│  │     └─────── Group: r-x (อ่านและ execute ได้)
│  └───────────── Owner: rwx (อ่าน เขียน execute ได้)
└─────────────── ประเภท: - ไฟล์, d directory, l symlink
```

**ค่า Permission แบบตัวเลข (Octal):**

| ตัวอักษร | ตัวเลข | ความหมาย |
|---------|--------|----------|
| `r` | 4 | Read — อ่านได้ |
| `w` | 2 | Write — เขียน/แก้ไขได้ |
| `x` | 1 | Execute — รันได้ (ไฟล์) / เข้าได้ (directory) |

ตัวอย่าง: `chmod 755 script.sh` = Owner(7=rwx) Group(5=r-x) Other(5=r-x)

**Special Bits:**

| Bit | ความหมาย |
|-----|---------|
| SUID | โปรแกรมรันด้วยสิทธิ์ Owner (เช่น `passwd`) |
| SGID | ไฟล์ใหม่ใน directory สืบทอด Group |
| Sticky Bit | ลบได้เฉพาะเจ้าของ (เช่น `/tmp`) |

#### 📝 Assignment 1.5

> 1. สร้างไฟล์และตั้งค่า permission ให้ Owner อ่านเขียนได้, Group อ่านได้, Other ทำอะไรไม่ได้เลย
> 2. ดู permission ของ `/etc/passwd` และ `/etc/shadow` แตกต่างกันอย่างไร และทำไม?
> 3. `umask` คืออะไร และค่า default ของ AlmaLinux 9 คือเท่าไหร่?

---

### บทที่ 1.6 — Text Editors: Vim & Nano

การแก้ไขไฟล์ config บน server ที่ไม่มี GUI ต้องใช้ Text Editor บน Terminal

**Nano** — เหมาะสำหรับผู้เริ่มต้น ใช้งานง่าย ด้านล่างจะแสดง shortcut ตลอดเวลา

```
^X = Ctrl+X (ออก)    ^O = Ctrl+O (บันทึก)    ^W = Ctrl+W (ค้นหา)
```

> บน AlmaLinux 9 minimal install อาจไม่มี nano ต้องติดตั้งเพิ่ม: `dnf install nano`

**Vim** — ทรงพลังกว่า มักติดมากับระบบ RHEL-based โดย default:

```
Normal Mode  → พิมพ์คำสั่ง (default เมื่อเปิด)
Insert Mode  → กด i เพื่อพิมพ์ข้อความ
Visual Mode  → กด v เพื่อเลือกข้อความ
Command Mode → กด : เพื่อรันคำสั่ง
```

**Vim คำสั่งที่ต้องรู้:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `i` | เข้า Insert mode |
| `Esc` | กลับ Normal mode |
| `:w` | บันทึก |
| `:q` | ออก |
| `:wq` | บันทึกและออก |
| `:q!` | ออกโดยไม่บันทึก |
| `dd` | ลบทั้งบรรทัด |
| `/word` | ค้นหาคำ |

#### 📝 Assignment 1.6

> 1. สร้างไฟล์ชื่อ `practice.txt` ด้วย Vim แล้วพิมพ์ข้อความ 5 บรรทัด บันทึกและออก
> 2. เปิดไฟล์เดิมด้วย Vim แล้วค้นหาคำหนึ่ง และลบบรรทัดที่ 3 ออก
> 3. ลอง Vim tutor โดยพิมพ์คำสั่ง `vimtutor` แล้วทำแบบฝึกหัดอย่างน้อย 2 บทแรก

---

## Day 2 — System Administration & Networking

---

### บทที่ 2.1 — User & Group Management

Linux รองรับผู้ใช้หลายคนพร้อมกัน แต่ละคนมี **UID (User ID)** และ **GID (Group ID)** ที่ระบบใช้ระบุตัวตน

**ไฟล์สำคัญ:**

| ไฟล์ | เก็บอะไร |
|------|---------|
| `/etc/passwd` | ข้อมูล user ทุกคน (ชื่อ, UID, Home, Shell) |
| `/etc/shadow` | Password ที่เข้ารหัส (อ่านได้แค่ root) |
| `/etc/group` | ข้อมูล group ทั้งหมด |

**คำสั่งจัดการ User:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `useradd` | สร้าง user ใหม่ |
| `usermod` | แก้ไข user |
| `userdel` | ลบ user |
| `passwd` | เปลี่ยน password |
| `id` | ดู UID/GID ของ user |

**sudo บน AlmaLinux 9:**

user ที่อยู่ใน group `wheel` จะได้สิทธิ์ sudo โดยอัตโนมัติ (ต่างจาก Ubuntu ที่ใช้ group `sudo`)

```bash
# เพิ่ม user เข้า group wheel เพื่อให้ใช้ sudo ได้
usermod -aG wheel username
```

การควบคุม sudo อยู่ที่ไฟล์ `/etc/sudoers` (แก้ไขด้วยคำสั่ง `visudo` เท่านั้น)

#### 📝 Assignment 2.1

> 1. สร้าง user ใหม่ชื่อ `devuser` พร้อม Home Directory และตั้ง password
> 2. สร้าง group ชื่อ `developers` แล้วเพิ่ม `devuser` เข้าไป
> 3. ให้ `devuser` สามารถใช้ `sudo` ได้ โดยเพิ่มเข้า group `wheel` แล้วทดสอบ

---

### บทที่ 2.2 — Process Management

**Process** คือโปรแกรมที่กำลัง run อยู่ในขณะนั้น แต่ละ process มี **PID (Process ID)** ที่ไม่ซ้ำกัน

**States ของ Process:**

```
Running → Sleeping → Stopped → Zombie
  (R)        (S)       (T)       (Z)
```

**Zombie Process** คือ process ที่จบการทำงานแล้ว แต่ Parent Process ยังไม่ได้รับทราบ ถ้ามีมากเกินไปอาจทำให้ระบบมีปัญหา

**คำสั่งตรวจสอบ Process:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `ps aux` | แสดง process ทั้งหมดในระบบ |
| `top` | แสดง process แบบ real-time |
| `htop` | เหมือน top แต่สวยกว่า (ติดตั้งผ่าน `dnf install htop` จาก EPEL) |
| `pgrep` | หา PID ของ process ตามชื่อ |

**Signals ที่ใช้บ่อย:**

| Signal | ตัวเลข | ความหมาย |
|--------|--------|---------|
| SIGTERM | 15 | ขอให้ process หยุดอย่างสุภาพ (default) |
| SIGKILL | 9 | บังคับหยุดทันที (ไม่สามารถ ignore ได้) |
| SIGHUP | 1 | ให้ reload config ใหม่ |

**Background / Foreground:**

```bash
command &      # รัน command ใน background
Ctrl+Z         # หยุด process ชั่วคราว
bg             # ส่งไป background
fg             # เรียกกลับมา foreground
jobs           # ดู background jobs ทั้งหมด
```

#### 📝 Assignment 2.2

> 1. รันคำสั่ง `sleep 300 &` แล้วหา PID ของมัน และ kill ด้วย SIGTERM
> 2. ใช้ `top` แล้วบอกว่า process ไหนกิน CPU สูงสุดในขณะนั้น
> 3. `nice` และ `renice` คืออะไร ใช้เมื่อไหร่? ลองยกตัวอย่างการใช้งาน

---

### บทที่ 2.3 — Package Management with DNF

**DNF (Dandified YUM)** คือ Package Manager หลักของ AlmaLinux 9 และ distro ตระกูล RHEL ทั้งหมด พัฒนาต่อจาก YUM เดิม มีความเร็วและจัดการ dependency ดีกว่า

**DNF vs APT เปรียบเทียบ:**

| งาน | AlmaLinux (DNF) | Ubuntu (APT) |
|-----|-----------------|--------------|
| อัปเดต repo list | `dnf check-update` | `apt update` |
| ติดตั้ง package | `dnf install nginx` | `apt install nginx` |
| ลบ package | `dnf remove nginx` | `apt remove nginx` |
| ค้นหา package | `dnf search nginx` | `apt search nginx` |
| ดูรายละเอียด | `dnf info nginx` | `apt show nginx` |
| อัปเกรดทั้งระบบ | `dnf upgrade` | `apt upgrade` |
| ล้าง cache | `dnf clean all` | `apt clean` |

**EPEL (Extra Packages for Enterprise Linux)** คือ repository เสริมที่มี package ที่ไม่ได้อยู่ใน repo หลักของ RHEL เช่น `htop`, `redis`, `certbot` ฯลฯ

```bash
# เปิดใช้งาน EPEL บน AlmaLinux 9
dnf install epel-release
```

**DNF Groups** — ติดตั้ง package หลายตัวพร้อมกันตามหมวดหมู่:

```bash
dnf grouplist                          # ดู group ทั้งหมด
dnf groupinstall "Development Tools"   # ติดตั้งเครื่องมือ development
```

**Module Streams** — AlmaLinux 9 รองรับ Application Stream ทำให้ติดตั้ง PHP หรือ Node.js หลาย version ได้:

```bash
dnf module list php        # ดู PHP version ที่มี
dnf module enable php:8.2  # เลือก version ที่ต้องการ
```

#### 📝 Assignment 2.3

> 1. ค้นหาว่า package `nginx` มีอยู่ใน repo ไหม แล้วดูรายละเอียดก่อนติดตั้ง
> 2. เปิดใช้งาน EPEL แล้วติดตั้ง `htop` และ `tree`
> 3. ค้นหาว่า PHP version อะไรบ้างที่มีให้เลือกใน AlmaLinux 9 ผ่าน Module Streams

---

### บทที่ 2.4 — Networking Fundamentals

การเข้าใจ networking พื้นฐานเป็นสิ่งจำเป็นสำหรับ Linux Admin เพราะ server เกือบทุกตัวต้องเชื่อมต่อเครือข่าย

**แนวคิดสำคัญ:**

- **IP Address** — ที่อยู่ของเครื่องในเครือข่าย (เช่น `192.168.1.10`)
- **Subnet Mask** — กำหนดขอบเขตของ network (เช่น `/24` = 255 IP ใน subnet นั้น)
- **Gateway** — เครื่องที่ทำหน้าที่เป็นประตูออกสู่ internet
- **DNS** — แปลงชื่อ domain เป็น IP address
- **Port** — ช่องทางเฉพาะของ service เช่น HTTP=80, HTTPS=443, SSH=22

**คำสั่ง Networking:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `ip addr` | ดู IP address ของทุก interface |
| `ip route` | ดู routing table |
| `ss -tuln` | ดู port ที่เปิดอยู่ในระบบ |
| `ping` | ทดสอบการเชื่อมต่อ |
| `traceroute` | ดูเส้นทาง packet ไปยังปลายทาง |
| `nslookup` | ค้นหา DNS record |
| `curl` | ดึงข้อมูลจาก URL |
| `nmcli` | จัดการ network ผ่าน NetworkManager (AlmaLinux) |

**NetworkManager บน AlmaLinux 9:**

AlmaLinux 9 ใช้ **NetworkManager** เป็น default แทนการแก้ไฟล์ config โดยตรง การตั้งค่า network ทำผ่าน `nmcli` หรือ `nmtui` (TUI interface)

```bash
nmcli device status          # ดูสถานะ network interface
nmcli connection show        # ดู connection ทั้งหมด
nmtui                        # เปิด text UI สำหรับตั้งค่า network
```

ไฟล์ config network อยู่ที่ `/etc/NetworkManager/system-connections/`

#### 📝 Assignment 2.4

> 1. ดู IP address ของเครื่องตัวเองและ Gateway ของ network โดยใช้ทั้ง `ip addr` และ `nmcli`
> 2. ใช้ `ss -tuln` แล้วบอกว่า service อะไรกำลัง listen อยู่ที่ port ไหน
> 3. ใช้ `traceroute` ไปยัง `8.8.8.8` แล้วอธิบายว่าแต่ละ hop คืออะไร

---

### บทที่ 2.5 — Systemd & Service Management

**Systemd** คือระบบ init ของ Linux สมัยใหม่ ทำหน้าที่เป็น process แรกที่รันเมื่อระบบบูต (PID 1) และจัดการ **Service** (daemon) ทั้งหมด

**Unit File** คือไฟล์ config ของแต่ละ service เก็บอยู่ที่ `/etc/systemd/system/` หรือ `/usr/lib/systemd/system/`

**คำสั่ง systemctl:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `systemctl start nginx` | เริ่ม service |
| `systemctl stop nginx` | หยุด service |
| `systemctl restart nginx` | restart service |
| `systemctl reload nginx` | reload config โดยไม่หยุด service |
| `systemctl status nginx` | ดูสถานะ service |
| `systemctl enable nginx` | ตั้งให้ auto-start เมื่อ reboot |
| `systemctl disable nginx` | ปิด auto-start |
| `systemctl list-units --type=service` | ดู service ทั้งหมด |

**journalctl** — ดู log ของ systemd:

```bash
journalctl -u nginx              # log เฉพาะ nginx
journalctl -f                    # ดู log แบบ real-time (follow)
journalctl --since "1 hour ago"  # log ย้อนหลัง 1 ชั่วโมง
journalctl -xe                   # log ล่าสุดพร้อม context (ใช้ debug)
```

#### 📝 Assignment 2.5

> 1. ติดตั้ง nginx แล้วตรวจสอบ status และดู log ด้วย journalctl
> 2. ทดสอบ restart nginx แล้วสังเกตว่า log แสดงอะไร
> 3. สร้าง Unit File ของ service อย่างง่ายที่รัน script ตอนบูต (ค้นคว้าโครงสร้าง Unit File)

---

### บทที่ 2.6 — SSH & Remote Access

**SSH (Secure Shell)** คือ protocol สำหรับ remote login อย่างปลอดภัย ใช้การเข้ารหัสทุก packet ต่างจาก Telnet ที่ส่งแบบ plain text

**การ Authentication มี 2 วิธี:**

```
1. Password Authentication  → ง่าย แต่เสี่ยง brute-force
2. Key-based Authentication → ปลอดภัยกว่ามาก (แนะนำ)
```

**Key-based Authentication ทำงานอย่างไร:**

```
[Client]                    [Server]
  สร้าง Key Pair
  ├── Private Key (เก็บไว้)
  └── Public Key  ─────────→ ~/.ssh/authorized_keys

  เชื่อมต่อ: SSH ใช้ Private Key พิสูจน์ตัวตนกับ Public Key
```

**คำสั่ง SSH ที่ใช้บ่อย:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `ssh user@host` | เชื่อมต่อ server |
| `ssh-keygen` | สร้าง Key Pair |
| `ssh-copy-id user@host` | คัดลอก Public Key ไปยัง server |
| `scp file user@host:/path` | คัดลอกไฟล์ผ่าน SSH |
| `rsync -avz src user@host:/dst` | sync ไฟล์แบบ incremental |

**SSH Config File** ที่ `~/.ssh/config` ช่วยให้ตั้งค่า default ได้:

```
Host myserver
    HostName 192.168.1.10
    User alice
    IdentityFile ~/.ssh/id_rsa
    Port 22
```

> **SELinux บน AlmaLinux:** ถ้าเปลี่ยน SSH port จาก 22 เป็นอื่น ต้องแจ้ง SELinux ด้วย: `semanage port -a -t ssh_port_t -p tcp 2222`

#### 📝 Assignment 2.6

> 1. สร้าง SSH Key Pair และ copy Public Key ไปยัง server (หรือ VM อีกตัว)
> 2. ทดลอง login โดยไม่ใช้ password (Key-based) ให้สำเร็จ
> 3. แก้ไข SSH Config ของ server (`/etc/ssh/sshd_config`) เพื่อปิด Password Authentication และอนุญาตเฉพาะ Key-based — ระวังอย่า lock ตัวเองออกจากระบบ!

---

## Day 3 — Scripting, Security & Production Skills

---

### บทที่ 3.1 — Bash Scripting Essentials

**Bash Script** คือไฟล์ที่รวมคำสั่ง Shell หลายๆ คำสั่งไว้ด้วยกัน เพื่อทำงานซ้ำๆ ได้อัตโนมัติโดยไม่ต้องพิมพ์ทีละบรรทัด

Script ทุกไฟล์ต้องเริ่มต้นด้วย **Shebang** เพื่อบอกว่าใช้ interpreter ไหน:

```bash
#!/bin/bash
```

**โครงสร้างพื้นฐาน:**

```bash
#!/bin/bash

# Variables
NAME="Linux Admin"
echo "Hello, $NAME"

# Conditional
if [ $1 -gt 10 ]; then
    echo "มากกว่า 10"
else
    echo "ไม่มากกว่า 10"
fi

# Loop
for i in {1..5}; do
    echo "รอบที่ $i"
done

# Function
backup() {
    echo "กำลัง backup $1..."
}
backup /etc
```

**Special Variables:**

| Variable | ความหมาย |
|---------|---------|
| `$0` | ชื่อ script |
| `$1, $2...` | argument ที่ 1, 2... |
| `$#` | จำนวน argument ทั้งหมด |
| `$?` | Exit code ของคำสั่งก่อนหน้า (0=สำเร็จ) |
| `$$` | PID ของ script ปัจจุบัน |

**Cron** — ตั้งเวลารันคำสั่งอัตโนมัติ ตั้งค่าด้วย `crontab -e`:

```
# ┌─── นาที (0-59)
# │ ┌──── ชั่วโมง (0-23)
# │ │ ┌───── วันที่ (1-31)
# │ │ │ ┌────── เดือน (1-12)
# │ │ │ │ ┌─────── วันในสัปดาห์ (0-7, 0=อาทิตย์)
  0 2 * * * /home/alice/backup.sh
```

#### 📝 Assignment 3.1

> 1. เขียน script ที่รับชื่อ directory เป็น argument แล้ว backup โดย tar+gzip และตั้งชื่อไฟล์ตาม date
> 2. เพิ่ม error handling ให้ script (ตรวจสอบว่า directory มีอยู่จริงก่อน backup)
> 3. ตั้ง cron ให้รัน script backup ทุกคืน 2 ทุ่ม และ log output ไปยังไฟล์

---

### บทที่ 3.2 — Storage & Disk Management

การจัดการ storage เป็นงานประจำของ Admin ตั้งแต่การเพิ่ม disk ใหม่ ไปจนถึงการแก้ปัญหา disk full

**Disk → Partition → Filesystem → Mount** คือ flow ของการใช้งาน storage:

```
Physical Disk (/dev/sdb)
      ↓
   Partition (/dev/sdb1)
      ↓
   Format Filesystem (xfs หรือ ext4)
      ↓
   Mount ไปยัง Directory (/data)
```

> **AlmaLinux 9:** Filesystem เริ่มต้นของระบบคือ **XFS** (ไม่ใช่ ext4 แบบ Ubuntu) XFS เหมาะกับงาน high-performance และไฟล์ขนาดใหญ่มาก

**คำสั่งสำคัญ:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `lsblk` | แสดง disk และ partition ทั้งหมด |
| `fdisk -l` | รายละเอียด disk และ partition |
| `df -h` | พื้นที่ใช้งานของ filesystem ที่ mount อยู่ |
| `du -sh /path` | ขนาดของ directory |
| `mount` | mount filesystem |
| `umount` | unmount filesystem |
| `xfs_info` | ดูข้อมูล XFS filesystem |

**LVM (Logical Volume Manager)** คือ abstraction layer ที่ทำให้จัดการ storage ได้ยืดหยุ่นขึ้น AlmaLinux 9 ใช้ LVM เป็น default สำหรับ root filesystem:

```
Physical Volumes (PV) → Volume Group (VG) → Logical Volumes (LV)
   [/dev/sdb]              [vg_data]             [lv_home]
   [/dev/sdc]                                    [lv_var]
```

**fstab** — ไฟล์ `/etc/fstab` กำหนดว่าจะ auto-mount อะไรตอนบูต ผิดพลาดอาจทำให้ระบบบูตไม่ขึ้น

#### 📝 Assignment 3.2

> 1. เพิ่ม disk ใหม่ใน VM แล้ว partition, format เป็น XFS และ mount ไปยัง `/data`
> 2. ตั้งค่าใน `/etc/fstab` ให้ auto-mount เมื่อบูต แล้วทดสอบด้วย `mount -a`
> 3. ค้นหาว่า directory ไหนกิน disk มากที่สุดใน `/var` และหาสาเหตุ

---

### บทที่ 3.3 — Logging & Monitoring

**Log** คือบันทึกเหตุการณ์ในระบบ เป็นเครื่องมือสำคัญที่สุดในการแก้ไขปัญหา Log ที่ดีสามารถบอกได้ว่าเกิดอะไรขึ้น เมื่อไหร่ และทำไม

**แหล่ง Log สำคัญบน AlmaLinux 9:**

| Log File | เก็บอะไร |
|----------|---------|
| `/var/log/messages` | System log ทั่วไป (หลักของ RHEL-based) |
| `/var/log/secure` | การ login, sudo, SSH (แทน auth.log ของ Ubuntu) |
| `/var/log/kern` | Kernel messages |
| `/var/log/nginx/` | Web server log |
| `journalctl` | Log ของ systemd (รวมศูนย์) |
| `/var/log/audit/audit.log` | Security audit log (SELinux) |

**logrotate** — หมุนเวียน log ไฟล์ไม่ให้ disk เต็ม ตั้งค่าได้ที่ `/etc/logrotate.d/`

**คำสั่ง Monitoring:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `vmstat` | CPU, Memory, I/O, Swap statistics |
| `iostat` | Disk I/O statistics |
| `sar` | System Activity Reporter (historical) |
| `free -h` | Memory ที่ใช้งาน |
| `uptime` | ระยะเวลาที่ server run และ load average |

**Load Average** คือค่าเฉลี่ยจำนวน process ที่รอ CPU ในช่วง 1, 5, 15 นาที ถ้าเกินจำนวน CPU core = system กำลัง overload

#### 📝 Assignment 3.3

> 1. ดู log ของการ login เข้าระบบใน `/var/log/secure` หาว่ามีการพยายาม login ที่ล้มเหลวหรือไม่
> 2. ตั้งค่า logrotate ให้หมุน log ของ nginx ทุกสัปดาห์ และเก็บไว้ 4 สัปดาห์
> 3. เขียน script ง่ายๆ ที่ monitor disk usage และส่ง alert (echo ไปยัง log file) เมื่อเกิน 80%

---

### บทที่ 3.4 — Security & Hardening (SELinux + firewalld)

Security ของ Linux Server ไม่ใช่สิ่งที่ตั้งครั้งเดียวแล้วจบ AlmaLinux 9 มีระบบ security หลายชั้นที่แตกต่างจาก Ubuntu อย่างมีนัยสำคัญ

**firewalld — Firewall ของ AlmaLinux:**

AlmaLinux ใช้ `firewalld` ซึ่งทำงานบน concept ของ **Zone** แทนที่จะเป็น rule-based แบบ iptables ตรงๆ Zone กำหนดระดับความเชื่อถือของ network interface

```bash
firewall-cmd --state                                              # ดูสถานะ
firewall-cmd --get-default-zone                                   # ดู zone เริ่มต้น
firewall-cmd --zone=public --list-all                             # ดู rule ใน zone
firewall-cmd --zone=public --add-service=http --permanent         # อนุญาต HTTP
firewall-cmd --zone=public --add-service=https --permanent        # อนุญาต HTTPS
firewall-cmd --zone=public --add-port=8080/tcp --permanent        # เปิด port เฉพาะ
firewall-cmd --reload                                             # apply การเปลี่ยนแปลง
```

> ⚠️ ต้องใส่ `--permanent` เสมอ ไม่งั้น rule จะหายหลัง reload

**SELinux — Security ระดับ Kernel:**

**SELinux (Security-Enhanced Linux)** คือ Mandatory Access Control (MAC) ที่บังคับ policy ให้ทุก process ระบบนี้เป็นเอกลักษณ์ของ RHEL family และมักทำให้ผู้เริ่มต้นสับสน

```
SELinux มี 3 Mode:
Enforcing  → บังคับใช้ policy และ block ที่ผิด (production)
Permissive → แจ้งเตือนแต่ไม่ block (debugging)
Disabled   → ปิด SELinux (ไม่แนะนำ)
```

```bash
getenforce              # ดู mode ปัจจุบัน
setenforce 0            # เปลี่ยนเป็น Permissive ชั่วคราว
sestatus                # ดูสถานะละเอียด
ausearch -m avc -ts recent  # ดู SELinux denial ล่าสุด
```

> **สำคัญ:** เมื่อ service ไม่ทำงานบน AlmaLinux ให้ตรวจ SELinux ก่อนเสมอ มักเป็นสาเหตุหลัก

**fail2ban** — monitor log หา pattern การโจมตี แล้ว ban IP อัตโนมัติ:

```bash
dnf install fail2ban
systemctl enable --now fail2ban
```

**Security Checklist สำหรับ AlmaLinux:**

- อัปเดต package สม่ำเสมอ (`dnf upgrade`)
- ปิด service ที่ไม่ใช้งาน
- ใช้ SSH Key แทน Password
- ปิด root login ผ่าน SSH
- ตั้งค่า firewalld ให้ถูกต้อง
- ไม่ปิด SELinux — ควรเรียนรู้และทำงานร่วมกับมัน
- ดู log ใน `/var/log/secure` สม่ำเสมอ

#### 📝 Assignment 3.4

> 1. ตั้งค่า firewalld ให้อนุญาตเฉพาะ SSH, HTTP, HTTPS และ deny port อื่นทั้งหมด
> 2. ทดสอบติดตั้ง nginx แล้วดูว่า SELinux มีผลต่อการทำงานอย่างไร
> 3. ติดตั้งและตั้งค่า fail2ban สำหรับ SSH protection บน AlmaLinux 9

---

### บทที่ 3.5 — Automation & DevOps Intro

Linux Admin สมัยใหม่ไม่ได้แค่ดูแล server ด้วยมือ แต่ต้องรู้จัก **Automation** เพื่อจัดการ server จำนวนมากได้อย่างมีประสิทธิภาพ

**Ansible** — Agentless Configuration Management tool ที่ใช้ SSH เชื่อมต่อ server:

```
[Control Node]                    [Managed Nodes]
  Ansible ─── SSH ──→  server1
             ─── SSH ──→  server2
             ─── SSH ──→  server3
```

**Ansible Concepts:**

| Concept | ความหมาย |
|---------|---------|
| Inventory | รายชื่อ server ที่จะจัดการ |
| Playbook | ไฟล์ YAML ที่กำหนด task |
| Module | เครื่องมือสำเร็จรูปสำหรับงานต่างๆ |
| Role | การจัดกลุ่ม task ที่ใช้ซ้ำได้ |

**Podman บน AlmaLinux 9:**

AlmaLinux 9 มี **Podman** ติดมาด้วย ซึ่งเป็น container runtime ที่ compatible กับ Docker แต่ไม่ต้องใช้ daemon และ rootless โดย default

```
Docker   = ต้องมี daemon รันอยู่ตลอดเวลา (root required)
Podman   = ไม่มี daemon, rootless ได้, command เหมือน Docker
```

```bash
# Podman ใช้ command เดียวกับ Docker เกือบทั้งหมด
podman run -d -p 80:80 nginx
podman ps
podman images
```

#### 📝 Assignment 3.5

> 1. ทดลองใช้ Podman (ที่ติดมากับ AlmaLinux 9) รัน nginx container
> 2. ติดตั้ง Ansible แล้วสร้าง Playbook ที่ติดตั้ง nginx บน localhost
> 3. ค้นคว้าว่า `podman-compose` คืออะไร และใช้แทน `docker-compose` ได้หรือไม่

---

### บทที่ 3.6 — Capstone Project: Deploy a LEMP Stack บน AlmaLinux 9

**LEMP Stack** คือชุดซอฟต์แวร์ยอดนิยมสำหรับ web hosting:

```
L = Linux (AlmaLinux 9)
E = Nginx         (Web Server)
M = MariaDB       (Database — แทน MySQL บน RHEL ecosystem)
P = PHP           (Programming Language)
```

> บน AlmaLinux 9 นิยมใช้ **MariaDB** มากกว่า MySQL เพราะเป็น default ใน RHEL ecosystem และ fully compatible

**Architecture ที่จะ deploy:**

```
Internet
   ↓
[firewalld] :80/:443        ← Firewall + SELinux
   ↓
[Nginx] :80/:443            ← Web Server
   ↓
[PHP-FPM] unix socket       ← PHP Processor
   ↓
[MariaDB] :3306 (internal)  ← Database (ปิดจากภายนอก)
```

**Checklist สิ่งที่ต้องทำ:**

- [ ] ติดตั้งและ configure Nginx
- [ ] เลือก PHP version ผ่าน DNF Module Streams แล้วติดตั้ง PHP-FPM
- [ ] ติดตั้ง MariaDB และรัน `mysql_secure_installation`
- [ ] สร้าง Database และ User สำหรับ application
- [ ] ตั้งค่า SELinux context ให้ Nginx อ่าน/เขียน web files ได้
- [ ] ตั้งค่า firewalld อนุญาตเฉพาะ port ที่จำเป็น
- [ ] ทดสอบด้วย PHP info page
- [ ] ตั้งค่า logrotate สำหรับ Nginx log
- [ ] เขียน script backup database อัตโนมัติ

#### 📝 Assignment 3.6 (Final Project)

> Deploy LEMP Stack บน AlmaLinux 9 และส่งรายงานที่มีเนื้อหาดังนี้:
>
> 1. **Architecture Diagram** — วาดแผนผังการทำงานของระบบ
> 2. **Installation Log** — บันทึก command ทุกขั้นตอนที่ใช้ติดตั้ง
> 3. **Security Measures** — อธิบาย security ที่ตั้งค่าไว้ (firewalld rules, SELinux context, SSH config)
> 4. **Automation Script** — script สำหรับ backup MariaDB และ log ที่รันผ่าน cron
> 5. **Troubleshooting** — บันทึกปัญหาที่เจอ โดยเฉพาะปัญหา SELinux และวิธีแก้ไข

---

## บทเสริม A — Deploy Laravel บน AlmaLinux 9 (Bare Metal)

---

### A.1 — ภาพรวมและ Architecture

**Laravel** คือ PHP Framework ที่ได้รับความนิยมสูงที่สุดในโลก มีโครงสร้างที่ชัดเจนและ ecosystem ที่ครบครัน การ deploy บน bare metal หมายถึงการติดตั้งทุกอย่างลงบน OS โดยตรงโดยไม่มี layer ของ container คั่นกลาง

**สิ่งที่ต้องการสำหรับ Laravel บน AlmaLinux 9:**

```
[Internet]
    ↓
[firewalld + SELinux]    ← Security layer ของ AlmaLinux
    ↓
[Nginx]                  ← Web Server (reverse proxy ไปยัง PHP-FPM)
    ↓
[PHP-FPM 8.2+]           ← ประมวลผล PHP
    ├── Extensions: pdo_mysql, mbstring, xml, curl, zip, bcmath, tokenizer
    ↓
[MariaDB 10.11]          ← Database
    ↓
[Redis]                  ← Cache / Queue / Session (optional แต่แนะนำ)

[Supervisor]             ← จัดการ Queue Worker process
[Composer]               ← PHP Package Manager
[Node.js/NPM]            ← Build frontend assets (Vite)
```

**ทำไมถึงเลือก Bare Metal แทน Docker?**

| ข้อดี Bare Metal | ข้อดี Docker |
|----------------|-------------|
| Performance สูงกว่า | Reproducible ทุก environment |
| ง่ายต่อการ debug โดยตรง | Deploy ง่ายและเร็ว |
| ไม่มี overhead ของ container | Isolation ระหว่าง service |
| เหมาะกับ single server | Scaling ง่าย |

---

### A.2 — เตรียมระบบและติดตั้ง Dependencies

**AlmaLinux 9 ต้องเพิ่ม repository พิเศษ** เพราะ repo หลักมี PHP version เก่าเกินไปสำหรับ Laravel ใหม่ๆ

**Repository ที่ต้องใช้:**

```
EPEL      → package ทั่วไปที่ไม่อยู่ใน base
Remi      → PHP version ล่าสุด (8.2, 8.3)
```

ขั้นตอนการเพิ่ม Remi repo บน AlmaLinux 9:

```bash
# เพิ่ม EPEL ก่อน (Remi ต้องการ EPEL)
dnf install epel-release

# เพิ่ม Remi repository
dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm

# เปิดใช้งาน PHP 8.2 stream จาก Remi
dnf module reset php
dnf module enable php:remi-8.2

# ติดตั้ง PHP และ extensions ที่ Laravel ต้องการ
dnf install php php-fpm php-cli php-mysqlnd php-mbstring \
    php-xml php-curl php-zip php-bcmath php-tokenizer \
    php-ctype php-fileinfo php-openssl php-redis
```

**Composer** คือ Dependency Manager ของ PHP ทำงานคล้าย `npm` ของ Node.js หรือ `pip` ของ Python

```bash
# ติดตั้ง Composer globally
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer
```

#### 📝 Assignment A.2

> 1. ติดตั้ง PHP 8.2 ผ่าน Remi Repository บน AlmaLinux 9 พร้อม extension ที่ Laravel ต้องการ
> 2. ติดตั้ง Composer และตรวจสอบ version
> 3. ค้นหาว่า Laravel 11 (version ล่าสุด) ต้องการ PHP version ขั้นต่ำเท่าไหร่

---

### A.3 — สร้าง Laravel Project และตั้งค่า

**โครงสร้างไฟล์ที่สำคัญใน Laravel:**

```
/var/www/myapp/
├── app/           ← โค้ดหลักของ application (Models, Controllers)
├── bootstrap/     ← Bootstrap files ของ Laravel
├── config/        ← ไฟล์ตั้งค่าทั้งหมด
├── database/      ← Migration, Seeder, Factory
├── public/        ← Document Root ของ Web Server (index.php)
├── resources/     ← Views, CSS, JS (ก่อน compile)
├── routes/        ← Route definitions
├── storage/       ← Log, Cache, Sessions, Uploaded files
│   ├── app/
│   ├── framework/
│   └── logs/
├── vendor/        ← Dependencies ที่ติดตั้งผ่าน Composer
└── .env           ← Environment Variables (ไม่ commit ไป Git!)
```

**ไฟล์ `.env`** คือหัวใจของการ config Laravel แต่ละ environment (dev/staging/production) จะมี `.env` ของตัวเองที่แตกต่างกัน:

```ini
APP_ENV=production
APP_DEBUG=false          # ปิดเสมอใน production
APP_URL=https://myapp.com
APP_KEY=base64:...       # สร้างด้วย php artisan key:generate

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp_db
DB_USERNAME=myapp_user
DB_PASSWORD=strong_password_here

CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

**Permission ที่ถูกต้อง** เป็นเรื่องสำคัญมากบน production:

```
Web Server user (nginx) ต้องอ่าน/เขียน storage/ และ bootstrap/cache/
ไฟล์อื่นควร readable แต่ไม่ต้องให้ web server write ได้
```

```bash
# ตั้ง ownership ให้ nginx user
chown -R nginx:nginx /var/www/myapp

# ตั้ง permission สำหรับ storage และ cache
chmod -R 775 /var/www/myapp/storage
chmod -R 775 /var/www/myapp/bootstrap/cache
```

#### 📝 Assignment A.3

> 1. Clone หรือสร้าง Laravel project ใหม่ใน `/var/www/myapp`
> 2. ตั้งค่า `.env` ให้ถูกต้องสำหรับ production (APP_DEBUG=false, APP_ENV=production)
> 3. รัน `composer install --no-dev --optimize-autoloader` และอธิบายว่า `--no-dev` ทำอะไร
> 4. ตั้งค่า permission ของ `storage/` และ `bootstrap/cache/` ให้ถูกต้อง

---

### A.4 — ตั้งค่า Nginx + SELinux สำหรับ Laravel

Nginx ทำหน้าที่เป็น Web Server ที่รับ HTTP/HTTPS request แล้วส่งต่อ PHP files ไปให้ PHP-FPM ประมวลผล

**หลักการสำคัญของ Nginx + Laravel:**

```
Request ทุกอย่าง → /public/index.php  (Laravel's entry point)
Static files (css, js, images) → Nginx serve ตรงโดยไม่ผ่าน PHP
```

**ตัวอย่าง Nginx Config สำหรับ Laravel:**

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name myapp.com www.myapp.com;
    root /var/www/myapp/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;
    charset utf-8;

    # ส่ง request ทุกอย่างไปที่ index.php
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # ปิดการเข้าถึงไฟล์ซ่อน (.env, .git)
    location ~ /\.(?!well-known).* {
        deny all;
    }

    # ส่ง PHP ไปยัง PHP-FPM
    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

**SELinux Context สำหรับ Nginx + Laravel บน AlmaLinux 9:**

นี่คือส่วนที่แตกต่างจาก Ubuntu มากที่สุด SELinux จะ block Nginx ไม่ให้อ่านไฟล์ที่ไม่มี context ถูกต้อง ต้องตั้งค่าเพิ่ม:

```bash
# ให้ Nginx อ่านไฟล์ web content ทั่วไปได้
chcon -R -t httpd_sys_content_t /var/www/myapp

# ให้ Nginx อ่าน/เขียน storage และ cache ได้
chcon -R -t httpd_sys_rw_content_t /var/www/myapp/storage
chcon -R -t httpd_sys_rw_content_t /var/www/myapp/bootstrap/cache

# ให้ Nginx connect ออก network ได้ (สำหรับ Redis, Mail, API)
setsebool -P httpd_can_network_connect on

# ให้ Nginx connect ไปยัง database ได้
setsebool -P httpd_can_network_connect_db on
```

> 💡 **เทคนิค Debug SELinux:** เมื่อ service ไม่ทำงานให้รัน `ausearch -m avc -ts recent | audit2why` เพื่อดูว่า SELinux block อะไรและควรแก้อย่างไร

#### 📝 Assignment A.4

> 1. สร้างไฟล์ config Nginx สำหรับ Laravel ใน `/etc/nginx/conf.d/myapp.conf`
> 2. ทดสอบว่า config ถูกต้องด้วย `nginx -t` แล้ว reload Nginx
> 3. แก้ไข SELinux context ที่จำเป็นและทดสอบว่า Laravel เข้าได้จาก browser
> 4. ติดตั้ง SSL Certificate ด้วย Let's Encrypt (Certbot จาก EPEL) สำหรับ HTTPS

---

### A.5 — Database, Migration และ Queue Worker

**การตั้งค่า MariaDB สำหรับ Laravel:**

ต้องสร้าง Database และ User ที่มีสิทธิ์เฉพาะ database นั้นเท่านั้น ไม่ควรให้ application ใช้ root user เด็ดขาด

```sql
CREATE DATABASE myapp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'StrongPassword!';
GRANT ALL PRIVILEGES ON myapp_db.* TO 'myapp_user'@'localhost';
FLUSH PRIVILEGES;
```

> `utf8mb4` สำคัญมาก เพราะรองรับ emoji และ unicode ทุกภาษา รวมถึงภาษาไทย

**Artisan Commands ที่ใช้บ่อยใน Production:**

| คำสั่ง | ความหมาย |
|--------|----------|
| `php artisan migrate` | รัน database migration |
| `php artisan migrate --seed` | migrate พร้อม seed ข้อมูล |
| `php artisan key:generate` | สร้าง APP_KEY |
| `php artisan storage:link` | สร้าง symlink สำหรับ public storage |
| `php artisan config:cache` | cache config ไฟล์ (เร็วขึ้น) |
| `php artisan route:cache` | cache route definitions |
| `php artisan view:cache` | compile Blade templates |
| `php artisan optimize` | รัน cache ทั้งหมดพร้อมกัน |
| `php artisan queue:work` | เริ่ม Queue Worker |

**Supervisor สำหรับ Queue Worker:**

Laravel Queue ต้องการ process ที่รันอยู่ตลอดเวลา Supervisor คือเครื่องมือที่ดูแลให้ process เหล่านี้ไม่ตายแม้เกิด error

ติดตั้งบน AlmaLinux 9:

```bash
dnf install supervisor
systemctl enable --now supervisord
```

Config file ที่ `/etc/supervisord.d/laravel-worker.ini`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/myapp/artisan queue:work redis --sleep=3 --tries=3
autostart=true
autorestart=true
user=nginx
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/myapp/storage/logs/worker.log
```

#### 📝 Assignment A.5

> 1. สร้าง MariaDB database และ user สำหรับ Laravel แล้วรัน migration
> 2. ทดสอบรัน `php artisan optimize` และอธิบายว่า cache แต่ละอย่างส่งผลอย่างไร
> 3. ติดตั้ง Supervisor และตั้งค่าให้รัน Queue Worker 2 processes อัตโนมัติ

---

### A.6 — Deploy Process และ Automation Script

**Deploy Flow มาตรฐานสำหรับ Laravel บน Production:**

```
1. Pull code ใหม่จาก Git
2. composer install --no-dev --optimize-autoloader
3. php artisan down          (maintenance mode)
4. php artisan migrate --force
5. php artisan optimize:clear
6. php artisan optimize
7. npm ci && npm run build   (ถ้ามี frontend)
8. php artisan storage:link
9. php artisan up            (ออกจาก maintenance mode)
10. supervisorctl restart laravel-worker:*
```

**ตัวอย่าง Deploy Script:**

```bash
#!/bin/bash
set -e   # หยุดทันทีถ้ามี error

APP_DIR="/var/www/myapp"
PHP="/usr/bin/php"
LOG="/var/log/deploy.log"

echo "=== Starting deployment $(date) ===" | tee -a $LOG
cd $APP_DIR

echo "--- Pulling latest code..." | tee -a $LOG
git pull origin main

echo "--- Installing dependencies..." | tee -a $LOG
$PHP /usr/local/bin/composer install --no-dev --optimize-autoloader

echo "--- Entering maintenance mode..." | tee -a $LOG
$PHP artisan down

echo "--- Running migrations..." | tee -a $LOG
$PHP artisan migrate --force

echo "--- Clearing and rebuilding cache..." | tee -a $LOG
$PHP artisan optimize:clear
$PHP artisan optimize

echo "--- Building frontend assets..." | tee -a $LOG
npm ci --production
npm run build

echo "--- Linking storage..." | tee -a $LOG
$PHP artisan storage:link

echo "--- Fixing SELinux context after deploy..." | tee -a $LOG
chcon -R -t httpd_sys_content_t $APP_DIR
chcon -R -t httpd_sys_rw_content_t $APP_DIR/storage
chcon -R -t httpd_sys_rw_content_t $APP_DIR/bootstrap/cache

echo "--- Restarting queue workers..." | tee -a $LOG
supervisorctl restart laravel-worker:*

echo "--- Exiting maintenance mode..." | tee -a $LOG
$PHP artisan up

echo "=== Deployment completed $(date) ===" | tee -a $LOG
```

> สังเกตว่ามีขั้นตอน re-apply SELinux context หลัง `git pull` เพราะ git อาจ reset context ของไฟล์ใหม่ที่ดึงมา

#### 📝 Assignment A.6

> 1. ปรับ deploy script ให้เหมาะกับ project ของคุณและทดสอบรัน
> 2. ตั้ง cron job สำหรับ Laravel Scheduler (`php artisan schedule:run`) ทุก 1 นาที
> 3. เขียน script backup ที่ backup ทั้ง MariaDB และ `storage/app/` ด้วย `mysqldump` + `tar`

---

## บทเสริม B — Deploy Laravel ด้วย Docker บน AlmaLinux 9

---

### B.1 — ภาพรวมและ Architecture

การ deploy Laravel ด้วย Docker ทำให้ environment เหมือนกันทุกที่ ตั้งแต่ developer machine ไปจนถึง production server ลดปัญหา "works on my machine" ได้อย่างมีประสิทธิภาพ

**Docker Stack สำหรับ Laravel:**

```
[Internet]
    ↓
[Nginx Container] :80, :443
    ↓
[PHP-FPM Container]         ← App Container หลัก
    ├── Laravel code
    ├── Composer dependencies
    └── Compiled assets
[MariaDB Container]         ← Database
[Redis Container]           ← Cache/Queue/Session
[Queue Worker Container]    ← รัน php artisan queue:work
[Scheduler Container]       ← รัน php artisan schedule:run
```

**Docker Concepts ที่ต้องเข้าใจก่อน:**

| Concept | ความหมาย |
|---------|---------|
| Image | Blueprint ของ container (read-only) |
| Container | Instance ที่รันจาก Image |
| Volume | Persistent storage ที่ data ไม่หายเมื่อ container ตาย |
| Network | Virtual network ที่ container ใช้คุยกัน |
| Dockerfile | Script สำหรับสร้าง Image |
| docker-compose | เครื่องมือจัดการหลาย container พร้อมกัน |

**ข้อแตกต่าง Docker vs Podman บน AlmaLinux 9:**

AlmaLinux 9 มี **Podman** ติดมาด้วย ซึ่ง compatible กับ Docker command หากต้องการใช้ Docker Engine จริงๆ ต้องติดตั้งเพิ่มจาก Docker's official repository ทั้งสองทางใช้ `docker-compose` หรือ `podman-compose` ได้เหมือนกัน

---

### B.2 — Dockerfile สำหรับ Laravel

**Dockerfile** คือ script ที่กำหนดว่า Image ของเราจะมีอะไรบ้าง ใช้ Base Image ของ PHP-FPM แล้ว install เพิ่มเติม

**ตัวอย่าง Dockerfile สำหรับ Laravel (Multi-stage Build):**

```dockerfile
# Stage 1: Build frontend assets
FROM node:20-alpine AS frontend
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: PHP Application
FROM php:8.2-fpm AS production

# ติดตั้ง system dependencies
RUN apt-get update && apt-get install -y \
    git curl zip unzip libpng-dev libxml2-dev \
    libzip-dev libonig-dev libfreetype6-dev \
    && docker-php-ext-install \
       pdo_mysql mbstring xml zip bcmath gd opcache \
    && rm -rf /var/lib/apt/lists/*

# ติดตั้ง Redis extension
RUN pecl install redis && docker-php-ext-enable redis

# ติดตั้ง Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Copy dependency files ก่อน (ใช้ Docker layer cache ให้เป็นประโยชน์)
COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts --no-autoloader --prefer-dist

# Copy source code
COPY . .

# รับ frontend assets จาก stage แรก
COPY --from=frontend /app/public/build ./public/build

# Generate autoload และ optimize
RUN composer dump-autoload --no-dev --optimize \
    && php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache \
    && php artisan storage:link

# ตั้ง permission
RUN chown -R www-data:www-data /var/www/html/storage \
    && chown -R www-data:www-data /var/www/html/bootstrap/cache \
    && chmod -R 775 /var/www/html/storage \
    && chmod -R 775 /var/www/html/bootstrap/cache
```

**Multi-stage Build** ทำให้ Image ขนาดเล็กลงเพราะ Node.js และ build tools จะไม่ติดไปกับ production image โดยจะ copy เฉพาะ output ที่ compile แล้วเท่านั้น

#### 📝 Assignment B.2

> 1. สร้าง Dockerfile สำหรับ Laravel project ของคุณและ build image
> 2. ทดสอบรัน container เดี่ยวๆ และตรวจสอบว่า PHP extensions ครบถ้วน
> 3. อธิบายว่าทำไม Multi-stage Build ถึงทำให้ Image ขนาดเล็กลง และขนาดลดลงเท่าไหร่

---

### B.3 — Docker Compose สำหรับ Laravel

**docker-compose.yml** กำหนดทุก service ที่ application ต้องการ พร้อม network และ volume config

**ตัวอย่าง docker-compose.yml สำหรับ Laravel Production:**

```yaml
version: '3.8'

services:
  # PHP-FPM Application
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: laravel_app
    restart: unless-stopped
    working_dir: /var/www/html
    volumes:
      - storage_data:/var/www/html/storage/app
      - logs_data:/var/www/html/storage/logs
    env_file:
      - .env.production
    networks:
      - app_network
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  # Nginx Web Server
  nginx:
    image: nginx:alpine
    container_name: laravel_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/conf.d:/etc/nginx/conf.d:ro
      - ./public:/var/www/html/public:ro
      - certbot_www:/var/www/certbot:ro
      - certbot_conf:/etc/letsencrypt:ro
    networks:
      - app_network
    depends_on:
      - app

  # MariaDB Database
  db:
    image: mariadb:10.11
    container_name: laravel_db
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MARIADB_DATABASE: ${DB_DATABASE}
      MARIADB_USER: ${DB_USERNAME}
      MARIADB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app_network
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache/Queue
  redis:
    image: redis:7-alpine
    container_name: laravel_redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app_network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # Queue Worker
  queue:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: laravel_queue
    restart: unless-stopped
    command: php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
    env_file:
      - .env.production
    volumes:
      - logs_data:/var/www/html/storage/logs
    networks:
      - app_network
    depends_on:
      - app
      - redis

  # Laravel Scheduler
  scheduler:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: laravel_scheduler
    restart: unless-stopped
    command: >
      sh -c "while true; do php artisan schedule:run --verbose --no-interaction; sleep 60; done"
    env_file:
      - .env.production
    networks:
      - app_network
    depends_on:
      - app

volumes:
  db_data:
  redis_data:
  storage_data:
  logs_data:
  certbot_www:
  certbot_conf:

networks:
  app_network:
    driver: bridge
```

**Nginx Config ภายใน Docker** ต้องส่ง request ไปยัง `app` container โดยใช้ชื่อ service แทน IP:

```nginx
upstream php-fpm {
    server app:9000;    # ชื่อ service ใน docker-compose แทน localhost
}

server {
    listen 80;
    server_name myapp.com;
    root /var/www/html/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

#### 📝 Assignment B.3

> 1. สร้าง `docker-compose.yml` ครบทุก service และ test รันในเครื่อง local ก่อน
> 2. ทดสอบว่า container ทุกตัว healthy และ Laravel เข้าถึงได้จาก browser
> 3. ค้นคว้าว่า `healthcheck` ใน docker-compose มีประโยชน์อย่างไรและทำงานอย่างไร

---

### B.4 — Deploy บน AlmaLinux 9 Production Server

การ deploy Docker บน production server AlmaLinux 9 มีขั้นตอนที่ต้องจัดการ SELinux และ firewalld เพิ่มเติม

**ติดตั้ง Docker Engine บน AlmaLinux 9:**

Docker ไม่ได้อยู่ใน default repo ต้องเพิ่ม Docker's official repository ก่อน:

```bash
# เพิ่ม Docker repository (ใช้ RHEL repo)
dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

# ติดตั้ง Docker Engine และ compose plugin
dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# เริ่ม Docker และตั้งให้ auto-start
systemctl enable --now docker

# เพิ่ม user เข้า docker group
usermod -aG docker $USER
```

> หรือใช้ **Podman** ที่ติดมากับระบบ ไม่ต้องติดตั้งเพิ่ม และปลอดภัยกว่า (rootless by default)

**SELinux กับ Docker Volume บน AlmaLinux 9:**

Docker mount volume เข้า container ต้องมี SELinux label ที่ถูกต้อง ใช้ `:z` หรือ `:Z` flag:

```yaml
volumes:
  - ./storage:/var/www/html/storage:z   # z = share label กับ container อื่น
  - ./public:/var/www/html/public:Z     # Z = private label เฉพาะ container นี้
```

**firewalld กับ Docker บน AlmaLinux 9:**

Docker จัดการ iptables เองโดยตรง ซึ่งอาจ bypass firewalld ได้บางกรณี ต้องตั้งค่าเพิ่ม:

```bash
# อนุญาต traffic ใน docker network
firewall-cmd --zone=trusted --add-interface=docker0 --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent
firewall-cmd --reload
```

**Zero-downtime Deploy Script:**

```bash
#!/bin/bash
set -e

echo "=== Docker Deploy Starting $(date) ==="

# Pull latest code
git pull origin main

# Build new image
docker compose build app

# Run migrations ก่อน switch traffic
docker compose run --rm app php artisan migrate --force

# Recreate containers with new image
docker compose up -d --no-deps app queue scheduler

# ทำความสะอาด image เก่า
docker image prune -f

echo "=== Deploy Completed $(date) ==="
```

#### 📝 Assignment B.4

> 1. ติดตั้ง Docker Engine (หรือใช้ Podman) บน AlmaLinux 9 และตั้งค่า firewalld ให้ถูกต้อง
> 2. Deploy Laravel application ด้วย docker-compose บน production server
> 3. ทดสอบ zero-downtime deploy โดยการ update code แล้ว rebuild และ restart container
> 4. ตั้งค่า SSL ด้วย Certbot container สำหรับ Let's Encrypt

---

### B.5 — Container Registry และ CI/CD Pipeline

**Container Registry** คือที่เก็บ Docker Image เหมือน GitHub แต่สำหรับ container image ช่วยให้ไม่ต้อง build image บน production server โดยตรง ทำให้ deploy เร็วขึ้นและ reproducible

**Flow ที่แนะนำสำหรับ Production:**

```
[Developer] → git push → [CI/CD Pipeline]
                               ↓
                    Build & Test Docker Image
                               ↓
                    Push Image to Registry
                               ↓
              [Production Server (AlmaLinux 9)]
                    docker pull → docker compose up
```

**ตัวอย่าง GitHub Actions Workflow:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to Container Registry
        run: echo ${{ secrets.REGISTRY_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin

      - name: Build and push Docker image
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .
          docker tag ghcr.io/${{ github.repository }}:${{ github.sha }} ghcr.io/${{ github.repository }}:latest
          docker push ghcr.io/${{ github.repository }}:latest

      - name: Deploy to AlmaLinux Server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/myapp
            docker compose pull
            docker compose run --rm app php artisan migrate --force
            docker compose up -d --no-deps app queue scheduler
            docker image prune -f
```

**เปรียบเทียบสรุป Bare Metal vs Docker บน AlmaLinux 9:**

| ด้าน | Bare Metal | Docker |
|------|-----------|--------|
| Performance | ดีที่สุด | ดี (overhead เล็กน้อย) |
| Setup ครั้งแรก | ซับซ้อนกว่า (SELinux, Remi repo) | ง่ายกว่าเมื่อ Dockerfile พร้อม |
| SELinux | ต้องตั้ง chcon ทุก path | ใช้ :z/:Z volume label |
| Deploy ใหม่ | รัน deploy script | `docker compose up -d` |
| Rollback | ยาก | ง่าย (pull image เก่า) |
| Scaling | ต้องตั้งค่าเยอะ | `docker compose scale` |
| Debugging | ง่าย (เข้าถึงไฟล์โดยตรง) | ต้อง `docker exec` เข้า container |
| Log | `/var/log/` และ `storage/logs/` | `docker logs` หรือ volume mount |

#### 📝 Assignment B.5 (Final Project บทเสริม)

> สร้าง complete deployment pipeline สำหรับ Laravel project บน AlmaLinux 9 โดยมีเนื้อหาครบดังนี้:
>
> 1. **Dockerfile** — Multi-stage build ที่ optimize สำหรับ production
> 2. **docker-compose.yml** — ครบทุก service (app, nginx, db, redis, queue, scheduler)
> 3. **SELinux & firewalld** — ตั้งค่าให้ถูกต้องและ document ไว้
> 4. **Deploy Script** — zero-downtime deployment script
> 5. **CI/CD** — GitHub Actions หรือ GitLab CI pipeline ที่ auto-deploy เมื่อ push to main
>
> **เปรียบเทียบ:** หลังจากทำทั้ง Bare Metal (บทเสริม A) และ Docker (บทเสริม B) แล้ว เขียนสรุปว่าแต่ละแบบเหมาะกับ use case ไหน และถ้าคุณต้องเลือกสำหรับ project จริงบน AlmaLinux 9 คุณจะเลือกอะไรและทำไม

---

## Appendix — Quick Reference สำหรับ AlmaLinux 9

### คำสั่งฉุกเฉินที่ต้องรู้

```bash
# เมื่อ disk เต็ม
df -h                              # หาว่า filesystem ไหนเต็ม
du -sh /var/* | sort -hr           # หา directory ที่ใหญ่ที่สุด
journalctl --vacuum-size=500M      # ล้าง journal log เหลือ 500MB

# เมื่อ server ช้า
top                                # ดู CPU/Memory usage
iostat -x 1 5                      # ดู disk I/O
free -h                            # ดู memory เหลือเท่าไหร่

# เมื่อ service ไม่ทำงาน
systemctl status service-name      # ดู status
journalctl -u service-name -n 50   # ดู log ล่าสุด 50 บรรทัด
ausearch -m avc -ts recent         # ดู SELinux denial (AlmaLinux!)
sealert -a /var/log/audit/audit.log  # วิเคราะห์ SELinux และแนะนำ fix

# เมื่อ firewall blocking
firewall-cmd --zone=public --list-all       # ดู rules ทั้งหมด
firewall-cmd --zone=public --add-port=PORT/tcp --permanent
firewall-cmd --reload

# เมื่อ login ไม่ได้
cat /var/log/secure | grep Failed          # ดู failed login (AlmaLinux)
```

### เปรียบเทียบ Command บน AlmaLinux vs Ubuntu

| งาน | AlmaLinux 9 | Ubuntu 22.04 |
|-----|-------------|--------------|
| ติดตั้ง package | `dnf install pkg` | `apt install pkg` |
| อัปเดต package list | `dnf check-update` | `apt update` |
| Firewall | `firewall-cmd` | `ufw` |
| SELinux/AppArmor | `getenforce`, `semanage` | `aa-status`, `aa-enforce` |
| System log | `/var/log/messages` | `/var/log/syslog` |
| Auth log | `/var/log/secure` | `/var/log/auth.log` |
| sudo group | `wheel` | `sudo` |
| Default filesystem | XFS | ext4 |
| Network config | NetworkManager / `nmcli` | Netplan |
| Container (built-in) | Podman | — |
| PHP repo | Remi + EPEL | ppa:ondrej/php |

### Resources สำหรับศึกษาต่อ

| แหล่งเรียนรู้ | URL |
|-------------|-----|
| AlmaLinux Documentation | wiki.almalinux.org |
| Red Hat Documentation | access.redhat.com/documentation |
| Arch Wiki (อ้างอิงที่ดีที่สุด) | wiki.archlinux.org |
| Laravel Documentation | laravel.com/docs |
| Docker Documentation | docs.docker.com |
| Remi Repository | rpms.remirepo.net |
| OverTheWire Wargames (Security) | overthewire.org |

### Certification เส้นทางต่อไป

```
Linux+ (CompTIA)
    ↓
LFCS (Linux Foundation Certified Sysadmin)
    ↓
RHCSA (Red Hat Certified System Administrator)  ← ตรงกับ AlmaLinux มากที่สุด
    ↓
RHCE (Red Hat Certified Engineer)
```

---

*เอกสารนี้จัดทำสำหรับหลักสูตร Linux Administration: Zero to Hero (AlmaLinux 9)*
*ผู้เรียนควรเพิ่ม Step และ Notes จากการปฏิบัติจริงลงในแต่ละบท*