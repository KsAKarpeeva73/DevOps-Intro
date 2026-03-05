
## Task 1 — Установка VirtualBox

### Операционная система хоста

Информация об операционной системе была получена с помощью команды PowerShell.

Использованная команда:

```

systeminfo

```

Соответствующий вывод:

```

Operating system: Windows 11
Operating system version: 10.0.22631.5189

```

---

### Версия VirtualBox

VirtualBox был установлен с официального сайта с использованием стандартного установщика и настроек по умолчанию.

Использованная команда:

```

"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version

```

Вывод команды:

```

7.2.4r170995

```

### Примечания по установке

VirtualBox был успешно установлен с использованием стандартных параметров установщика.  
Во время установки никаких ошибок не возникло.



## Task 2 — Ubuntu VM и анализ системы

### Конфигурация виртуальной машины

При создании виртуальной машины были заданы следующие параметры:

- RAM: 10 GB
- CPU: 6 cores
- Storage: 25+ GB
- OS: Ubuntu 24.04 LTS



## CPU information

Использованная команда:

```

lscpu

```

Вывод команды:

```

Architecture:                x86_64
CPU op-mode(s):            32-bit, 64-bit
Address sizes:             39 bits physical, 48 bits virtual
Byte Order:                Little Endian
CPU(s):                      6
On-line CPU(s) list:       0-5
Vendor ID:                   GenuineIntel
Model name:                11th Gen Intel(R) Core(TM) i7-11800H @ 2.30GHz
CPU family:              6
Model:                   141
Thread(s) per core:      1
Core(s) per socket:      6
Socket(s):               1
Stepping:                1
BogoMIPS:                4608.00
Flags:                   fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pg
e mca cmov pat pse36 clflush mmx fxsr sse sse2 ht s
yscall nx rdtscp lm constant_tsc rep_good nopl xtop
ology nonstop_tsc cpuid tsc_known_freq pni pclmulqd
q ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe po
pcnt aes xsave avx f16c rdrand hypervisor lahf_lm a
bm 3dnowprefetch fsgsbase bmi1 avx2 bmi2 invpcid rd
seed adx clflushopt sha_ni arat md_clear flush_l1d
arch_capabilities
Virtualization features:
Hypervisor vendor:         KVM
Virtualization type:       full
Caches (sum of all):
L1d:                       288 KiB (6 instances)
L1i:                       192 KiB (6 instances)
L2:                        7.5 MiB (6 instances)
L3:                        144 MiB (6 instances)
NUMA:
NUMA node(s):              1
NUMA node0 CPU(s):         0-5

```

## Memory information

Использованная команда:

```

free -h

```

Вывод:


```
           total        used        free      shared  buff/cache   available


Mem:           9.9Gi       1.9Gi       351Mi       216Mi       7.9Gi       8.0Gi
Swap:             0B          0B          0B

```

Дополнительно использована команда:

```

cat /proc/meminfo

```

Вывод:

```

MemTotal:       10386876 kB
MemFree:          359508 kB
MemAvailable:    8364116 kB
Buffers:           11920 kB
Cached:          8222964 kB
SwapCached:            0 kB
Active:          2617848 kB
Inactive:        6976056 kB
Active(anon):    1340080 kB
Inactive(anon):    61988 kB
Active(file):    1277768 kB
Inactive(file):  6914068 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB

```

---

## Network configuration

Использованная команда:

```

ip a

```

Вывод:

```

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
inet 127.0.0.1/8 scope host lo

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3

```

Дополнительно использована команда:

```

ip route

```

Вывод:

```

default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100

```

---

## Storage information

Использованная команда:

```

df -h

```

Вывод:

```

Filesystem      Size  Used Avail Use% Mounted on
tmpfs          1015M  1.8M 1013M   1% /run
/dev/sr0        6.2G  6.2G     0 100% /cdrom
/cow            5.0G  173M  4.8G   4% /
tmpfs           5.0G  8.0K  5.0G   1% /dev/shm

```

Дополнительно использована команда:

```

lsblk

```

Вывод:

```

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0  62.9G  0 disk
sr0     11:0    1   6.2G  0 rom  /cdrom

```

---

## Operating system information

Использованная команда:

```

lsb_release -a

```

Вывод:

```

Distributor ID: Ubuntu
Description:    Ubuntu 24.04.4 LTS
Release:        24.04
Codename:       noble

```

Дополнительно:

```

uname -a

```

Вывод:

```

Linux ubuntu 6.17.0-14-generic #14~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Thu Jan 15 15:52:10 UTC 2 x86_64 x86_64 x86_64 GNU/Linux

```

---

## Virtualization detection

Использованная команда:

```

systemd-detect-virt

```

Вывод:

```

oracle

```

Это подтверждает, что система работает внутри VirtualBox.

---

## System summary

Использованная команда:

```

hostnamectl

```

Вывод:

```

Static hostname: ubuntu
Virtualization: oracle
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.17.0-14-generic
Architecture: x86-64
Hardware Model: VirtualBox

```

---

## Additional system information

Использованные команды:

```

uptime
whoami

```

Вывод:

```

19:01:46 up 34 min,  1 user,  load average: 0.00, 0.00, 0.02
ubuntu

