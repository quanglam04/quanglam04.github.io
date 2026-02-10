---
title: "Docker hoạt động như thế nào ở tầng OS"
date: 2025-12-26 01:17:00  +0700
categories: [os, docker]
tags: [os, docker]
---


## Giới thiệu

Docker là một ứng dụng containerization, nó giúp tạo ra một môi trường ảo hóa độc lập cho ứng dụng. Docker được viết ban đầu trên Linux và cho tới tận bây giờ, Linux vẫn là môi trường mà Docker chạy "native" nhất.

Bài viết này sẽ đi sâu vào cách Docker hoạt động ở tầng hệ điều hành, giúp bạn hiểu rõ bản chất và lý do tại sao Docker lại phổ biến đến vậy.

## 1. User Space và Kernel Space

Để hiểu được Docker hoạt động như thế nào, bạn cần nắm vững khái niệm về hai tầng quan trọng trong kiến trúc hệ điều hành:

### User Space (Không gian người dùng)

User space là tầng chứa các ứng dụng và các dịch vụ liên quan tới người dùng. Đây là nơi các chương trình như:
- Web browsers (Chrome, Firefox)
- Text editors (VSCode, Vim)
- Command-line tools (bash, grep, ls, apt)
- Application servers (Nginx, Apache, Node.js)

hoạt động và tương tác với người dùng.

### Kernel Space (Không gian nhân)

Kernel space là tầng chứa các dịch vụ và các ứng dụng liên quan tới hệ thống, bao gồm:
- **Kernel code**: Nhân hệ điều hành
- **Drivers**: Trình điều khiển phần cứng
- **Memory management**: Quản lý bộ nhớ
- **Network stack**: Xử lý mạng
- **File system**: Quản lý tập tin
- **Process scheduling**: Lập lịch tiến trình

```
┌─────────────────────────────────────────┐
│         USER SPACE                      │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐    │
│  │ App1 │ │ App2 │ │ App3 │ │ App4 │    │
│  └──────┘ └──────┘ └──────┘ └──────┘    │
├─────────────────────────────────────────┤
│         KERNEL SPACE                    │
│  ┌────────────────────────────────┐     │
│  │  Kernel, Drivers, Memory Mgmt  │     │
│  │  Network Stack, File System    │     │
│  └────────────────────────────────┘     │
├─────────────────────────────────────────┤
│         HARDWARE                        │
│  CPU, RAM, Disk, Network Card...        │
└─────────────────────────────────────────┘
```

## 2. Docker Image: Đóng gói User Space

### Bản chất của Docker Image

Các image của Docker bản chất là **đóng gói của tầng User Space**. Điều này có nghĩa là:

**Ví dụ với image `ubuntu:20.04`:**
-  **Có chứa**: apt, bash, ls, grep, và các công cụ user-space của Ubuntu
-  **Không chứa**: Kernel code của Ubuntu

```dockerfile
# Khi bạn pull một image Ubuntu
docker pull ubuntu:20.04

# Image này chỉ khoảng 72MB
# Trong khi Ubuntu ISO đầy đủ thường > 2GB
```

### Tại sao Image nhẹ hơn OS đầy đủ?

| Thành phần | OS đầy đủ | Docker Image |
|------------|-----------|--------------|
| Kernel |  Có |  Không |
| Drivers |  Có |  Không |
| User-space tools |  Có |  Có |
| Desktop Environment |  Có |  Không |
| Dung lượng | 2-4 GB | 50-200 MB |

## 3. Container: Chia sẻ Kernel với Host

### Cơ chế hoạt động

Khi các image được deploy lên, nó sẽ tạo ra một container mới và **SỬ DỤNG CHUNG kernel với hệ thống host** để chạy các ứng dụng.

```
┌─────────────────────────────────────────────────┐
│                    HOST OS (Linux)              │
│                                                 │
│  ┌──────────────┐  ┌──────────────┐             │
│  │ Container 1  │  │ Container 2  │             │
│  │              │  │              │             │
│  │ Ubuntu       │  │ Alpine       │             │
│  │ User Space   │  │ User Space   │             │
│  └──────────────┘  └──────────────┘             │
│                                                 │
│  ────────────────────────────────────────────   │
│           SHARED KERNEL (Linux Kernel)          │
│  ────────────────────────────────────────────   │
│                                                 │
│                    HARDWARE                     │
└─────────────────────────────────────────────────┘
```

### Ứng dụng trong Container

Khi các ứng dụng chạy trong container:
- Chúng hoạt động giống như các ứng dụng bình thường ở máy host
- Các lời gọi hệ thống (system calls) từ ứng dụng trong Docker có thể gọi **trực tiếp xuống kernel** như các ứng dụng ở máy host
- Không có overhead từ việc giả lập phần cứng hay chạy kernel riêng

```bash
# Ví dụ: Process trong container
docker run -d nginx

# Trên host, bạn có thể thấy process nginx
ps aux | grep nginx

# Nginx trong container gọi system calls trực tiếp xuống host kernel
```

## 4. Ba nhiệm vụ chính của Docker

Docker thực hiện ba công việc chính để tạo ra môi trường container độc lập:

### 4.1. Quản lý isolation bằng Namespace và Cgroup

#### Namespace - Cô lập tiến trình

Namespace được sử dụng để quản lý các tiến trình, giúp các ứng dụng:
- Độc lập với nhau
- Không thể vượt quyền để ảnh hưởng tới máy host
- Có "view" riêng về hệ thống

**Các loại namespace:**

```bash
# PID Namespace - Process ID isolation
# Container có PID namespace riêng
docker exec my-container ps aux
# PID 1 trong container ≠ PID 1 trên host

# Network Namespace - Network isolation
# Container có network stack riêng
docker exec my-container ip addr

# Mount Namespace - File system isolation
# Container có mount points riêng

# UTS Namespace - Hostname isolation
# Container có hostname riêng
docker exec my-container hostname

# IPC Namespace - Inter-Process Communication isolation
# User Namespace - User ID isolation
```

**Ví dụ minh họa:**

```bash
# Trên host, bạn có process với PID 12345
ps aux | grep nginx

# Trong container, process này có PID 1
docker exec my-nginx ps aux
# USER  PID  COMMAND
# root   1   nginx: master process
```

#### Cgroup - Giới hạn tài nguyên

Cgroup (Control Groups) được sử dụng để quản lý tài nguyên tối đa mà một container có thể sử dụng:

```bash
# Giới hạn CPU
docker run --cpus="1.5" my-app
# Container chỉ được dùng tối đa 1.5 CPU cores

# Giới hạn RAM
docker run --memory="512m" my-app
# Container chỉ được dùng tối đa 512MB RAM

# Giới hạn I/O
docker run --device-read-bps /dev/sda:1mb my-app
# Giới hạn tốc độ đọc disk
```

**Kiểm tra giới hạn của container:**

```bash
# Xem thông tin cgroup của container
docker stats my-container

CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT     MEM %
abc123         my-container  10.5%     256MiB / 512MiB      50%
```

### 4.2. Đóng gói thư viện

Trong file image, Docker đã bao gồm đầy đủ các thư viện cần thiết để chạy ứng dụng, **độc lập với host**.

**Ví dụ:**

```dockerfile
# Dockerfile cho ứng dụng Python
FROM python:3.9

# Các thư viện Python cần thiết được đóng gói trong image
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .
CMD ["python", "app.py"]
```

```
┌────────────────────────────────────┐
│        Container                   │
│                                    │
│  App Code: app.py                  │
│  Python Runtime: 3.9               │
│  Dependencies: Flask, requests...  │
│  System libs: libc, libssl...      │
│                                    │
└────────────────────────────────────┘
         ↓ Sử dụng kernel
┌────────────────────────────────────┐
│         Host Linux Kernel          │
└────────────────────────────────────┘
```

**Lợi ích:**
- Không conflict với thư viện trên host
- "Works on my machine" → "Works everywhere"
- Dễ dàng di chuyển giữa các môi trường

### 4.3. Quản lý tập tin bằng OverlayFS

Docker sử dụng **OverlayFS** (hoặc các storage drivers tương tự) để quản lý file system hiệu quả.

#### Cấu trúc Layers

File image Docker được chia thành nhiều layers (lớp):

```
┌─────────────────────────────────────┐
│  Layer 4: Writable Container Layer  │  ← Ghi file mới
├─────────────────────────────────────┤
│  Layer 3: Copy app.py               │  ← Read-only
├─────────────────────────────────────┤
│  Layer 2: pip install packages      │  ← Read-only
├─────────────────────────────────────┤
│  Layer 1: FROM python:3.9           │  ← Read-only
└─────────────────────────────────────┘
```

#### Cơ chế Copy-on-Write

**Khi ghi file (từ ứng dụng):**
- Docker tạo một layer mới (writable layer)
- File mới được lưu vào layer này
- Các layer cũ vẫn giữ nguyên (read-only)

**Khi đọc file:**
- Docker đọc từ trên xuống (top-down)
- Nếu layer trên cùng có file → dùng file đó
- Nếu không → tìm ở layer dưới
- Tiếp tục cho đến khi tìm thấy

```bash
# Ví dụ: Modify file trong container
docker run -it ubuntu bash

# Trong container
echo "Hello" > /test.txt
# File này được ghi vào writable layer

# File /etc/hosts từ base image
cat /etc/hosts
# File này đọc từ read-only layer
```

#### Tiết kiệm bộ nhớ và tăng tốc độ

Với cơ chế layer, Docker đạt được hiệu quả cao:

```bash
# Chạy 1000 containers từ cùng 1 image
for i in {1..1000}; do
  docker run -d nginx
done

# Không cần copy 1000 lần image gốc!
# Image gốc là read-only và được share
# Chỉ writable layer cho mỗi container là khác nhau
```

**So sánh dung lượng:**

```bash
# Image nginx: 142MB
docker images nginx
REPOSITORY   TAG       SIZE
nginx        latest    142MB

# 1 container nginx chạy
docker run -d nginx
docker ps -s
CONTAINER ID   SIZE
abc123         2MB (virtual 142MB)
# Size thực tế chỉ 2MB (writable layer)
# Virtual size 142MB (shared với image)

# 1000 containers
# Tổng dung lượng ≈ 142MB + (1000 × 2MB) = ~2.1GB
# Thay vì 142GB nếu copy đầy đủ!
```

## 5. Docker vs Virtual Machine (VM)

### So sánh kiến trúc

**Virtual Machine:**

```
┌───────────────┐  ┌───────────────┐
│     VM 1      │  │     VM 2      │
│               │  │               │
│ App A         │  │ App B         │
│ Libs          │  │ Libs          │
│ Guest OS      │  │ Guest OS      │
│ Kernel        │  │ Kernel        │
└───────────────┘  └───────────────┘
        ↓                  ↓
┌─────────────────────────────────┐
│        Hypervisor               │
├─────────────────────────────────┤
│        Host OS                  │
│        Host Kernel              │
├─────────────────────────────────┤
│        Hardware                 │
└─────────────────────────────────┘
```

**Docker Containers:**

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│Container1│  │Container2│  │Container3│
│          │  │          │  │          │
│ App A    │  │ App B    │  │ App C    │
│ Libs     │  │ Libs     │  │ Libs     │
└──────────┘  └──────────┘  └──────────┘
        ↓           ↓            ↓
┌─────────────────────────────────────┐
│         Docker Engine               │
├─────────────────────────────────────┤
│         Host OS                     │
│         Shared Kernel               │
├─────────────────────────────────────┤
│         Hardware                    │
└─────────────────────────────────────┘
```

### Bảng so sánh chi tiết

| Tiêu chí | Virtual Machine | Docker Container |
|----------|----------------|------------------|
| **Kernel** | Mỗi VM có kernel riêng | Share kernel với host |
| **Dung lượng lưu trữ** | Rất lớn (GBs) | Nhỏ (MBs) |
| **Thời gian khởi động** | Phút | Giây |
| **Memory usage** | Cao (mỗi VM cần RAM riêng) | Thấp (share kernel) |
| **Performance overhead** | Cao (giả lập phần cứng) | Thấp (native) |
| **Isolation** | Hoàn toàn (có kernel riêng) | Process-level |
| **Boot process** | Phải khởi động cả kernel, BIOS | Chỉ start process |
| **Portability** | Khó (phụ thuộc hypervisor) | Dễ (image portable) |

### Ví dụ thực tế

**Khởi động thời gian:**

```bash
# VM
time VBoxManage startvm "Ubuntu VM"
# Real time: ~45 seconds

# Docker
time docker run -d nginx
# Real time: ~2 seconds
```

**Dung lượng:**

```bash
# VM: Ubuntu Server
# Virtual disk: 10-20GB
# RAM allocated: 2GB
# Total: 2GB RAM + 10-20GB Disk

# Docker: ubuntu:20.04
docker images ubuntu
# REPOSITORY   TAG       SIZE
# ubuntu       20.04     72.8MB
# RAM: share với host, chỉ dùng khi cần
```

## 6. Docker trên Windows và macOS

### Vấn đề cơ bản

Docker xuất phát từ Linux, các image được xây dựng dựa trên Linux, chứ không phải Windows hay macOS. Do đó, Docker trên các nền tảng này **không thể chạy native**.

### Docker trên Windows

#### Cách tiếp cận cũ: Hyper-V

Trước đây, Docker sử dụng Hyper-V để chạy máy ảo Linux:

```
┌─────────────────────────────────────┐
│          Windows Host               │
│                                     │
│  ┌──────────────────────────────┐   │
│  │   Hyper-V Virtual Machine    │   │
│  │                              │   │
│  │   ┌──────────────────────┐   │   │
│  │   │  Linux VM            │   │   │
│  │   │                      │   │   │
│  │   │  Docker Daemon       │   │   │ 
│  │   │  Containers          │   │   │ 
│  │   └──────────────────────┘   │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

**Nhược điểm:**
- Nặng nề (phải chạy cả VM)
- Chậm
- Tốn tài nguyên

#### Cách tiếp cận hiện tại: WSL 2

Windows hiện có tích hợp sẵn WSL 2 (Windows Subsystem for Linux):

```bash
# Kiểm tra WSL
wsl --list --verbose

# NAME                   STATE           VERSION
# * Ubuntu-20.04         Running         2
#   docker-desktop       Running         2
```

```
┌─────────────────────────────────────┐
│          Windows 11                 │
│                                     │
│  ┌──────────────────────────────┐   │
│  │   WSL 2                      │   │
│  │   (Lightweight VM)           │   │
│  │                              │   │
│  │   Docker Daemon              │   │
│  │   Containers                 │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

**Cải thiện:**
- Nhẹ hơn Hyper-V
- Tích hợp tốt hơn
- Performance tốt hơn

**Hạn chế:**
- Vẫn không thể native
- WSL 2 vẫn phải chạy trên môi trường Windows
- Có overhead so với Linux thuần

### Docker trên macOS

macOS sử dụng ảo hóa của Apple (Hypervisor.framework):

```
┌─────────────────────────────────────┐
│            macOS                    │
│                                     │
│  ┌──────────────────────────────┐   │
│  │   macOS Virtualization       │   │
│  │   (Hypervisor.framework)     │   │
│  │                              │   │
│  │   ┌──────────────────────┐   │   │
│  │   │  Linux VM            │   │   │
│  │   │  Docker Daemon       │   │   │
│  │   │  Containers          │   │   │
│  │   └──────────────────────┘   │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

Docker Daemon phải chạy trong máy ảo Linux, không chạy trực tiếp trên macOS.

**Hạn chế:**
- Không native
- Overhead từ virtualization
- Performance không bằng Linux

### Windows Containers

Docker cho phép chạy native với Windows thông qua **Windows Containers**:

```bash
# Switch sang Windows containers
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon

# Sử dụng Windows base images
docker pull mcr.microsoft.com/windows/nanoserver:ltsc2022
docker pull mcr.microsoft.com/windows/servercore/iis
```

**Đặc điểm:**
- Chạy native trên Windows
- Sử dụng Windows kernel
- Chỉ hoạt động với images được build cho Windows

**Use cases:**
- .NET Framework applications
- IIS web servers
- Windows-specific workloads

**Hạn chế:**
- Không thể chạy Linux containers
- Ít phổ biến hơn Linux containers
- Image size lớn hơn
- Ecosystem nhỏ hơn

### Kết luận về các nền tảng

```bash
# Performance comparison (relative)
Linux:   ████████████████████ 100% (Native)
WSL 2:   ████████████████░░░░ 80-85%
macOS:   ███████████████░░░░░ 75-80%
Hyper-V: ██████████░░░░░░░░░░ 50-60%
```

**Trong production:**
- Linux chiếm ưu thế tuyệt đối
- Hầu hết các cloud providers chạy Linux containers
- Windows containers chỉ dùng cho workloads đặc thù

**Đây cũng là lý do nhiều developers chuyển sang dùng Linux:**
- Native Docker experience
- Performance tốt nhất
- Đặc biệt phù hợp với máy yếu (không cần overhead của VM)


