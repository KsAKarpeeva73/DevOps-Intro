## Task 1 — Operating System Analysis

Environment: Ubuntu VPS. All outputs below are copied as-is.

### 1.1 Boot Performance Analysis

Command:

```sh
systemd-analyze
```

Output:

```text
Startup finished in 2.434s (kernel) + 23.970s (userspace) = 26.405s
graphical.target reached after 22.857s in userspace.
```

Observation:
Система грузится примерно за ~26.4 секунды суммарно. Большая часть времени — userspace (почти 24 секунды), то есть уже после загрузки ядра. До graphical.target дошло за ~22.9 секунды в userspace (видимо, поднимаются сервисы и сеть).

Command:

```sh
systemd-analyze blame --no-pager
```

Output:

```text
16.106s cloud-init-local.service
 5.473s apt-daily.service
 2.035s cloud-init.service
 1.997s systemd-networkd-wait-online.service
 1.477s cloud-config.service
 1.399s dev-vda1.device
 1.091s cloud-final.service
  959ms apt-daily-upgrade.service
  872ms docker.service
  436ms ldconfig.service
  424ms tuned.service
  319ms user@0.service
  309ms systemd-udev-trigger.service
  281ms systemd-journal-catalog-update.service
  241ms fstrim.service
  213ms systemd-machine-id-commit.service
  145ms dev-hugepages.mount
  141ms dev-mqueue.mount
  137ms kmod-static-nodes.service
  136ms sys-kernel-debug.mount
  136ms systemd-fsck-root.service
  136ms modprobe@drm.service
  135ms systemd-logind.service
  134ms modprobe@configfs.service
  134ms modprobe@fuse.service
  132ms sys-kernel-tracing.mount
  116ms systemd-resolved.service
  111ms systemd-tmpfiles-setup-dev-early.service
  109ms systemd-modules-load.service
  107ms containerd.service
  107ms rsyslog.service
   93ms polkit.service
   88ms dbus.service
   84ms systemd-udevd.service
   79ms systemd-firstboot.service
   67ms systemd-networkd.service
   60ms systemd-journal-flush.service
   59ms systemd-remount-fs.service
   59ms systemd-update-utmp.service
   55ms ssh.service
   55ms systemd-binfmt.service
   55ms systemd-sysusers.service
   53ms systemd-tmpfiles-setup.service
   52ms systemd-timesyncd.service
   47ms packagekit.service
   44ms sys-fs-fuse-connections.mount
   41ms grub-common.service
   37ms sys-kernel-config.mount
   37ms systemd-random-seed.service
   36ms grub-initrd-fallback.service
   36ms systemd-update-utmp-runlevel.service
   31ms modprobe@dm_mod.service
   28ms systemd-tmpfiles-clean.service
   27ms systemd-journald.service
   26ms modprobe@loop.service
   25ms systemd-update-done.service
   23ms modprobe@efi_pstore.service
   20ms systemd-sysctl.service
   20ms e2scrub_reap.service
   18ms dpkg-db-backup.service
   16ms docker.socket
   15ms systemd-tmpfiles-setup-dev.service
   13ms user-runtime-dir@0.service
   10ms e2scrub_all.service
   10ms proc-sys-fs-binfmt_misc.mount
    8ms systemd-user-sessions.service
    2ms motd-news.service
```

Observation:
Топ по времени — cloud-init-local.service (16 сек), потом apt-daily.service (5.5 сек). Похоже это виртуалка/облако, и cloud-init реально заметно тормозит старт. Ещё видно systemd-networkd-wait-online.service ~2 секунды — ожидание сети.

Command:

```sh
uptime
```

Output:

```text
 20:08:42 up 63 days,  3:31,  1 user,  load average: 0.06, 0.02, 0.00
```

Observation:
Аптайм большой (63 дня), средняя нагрузка очень маленькая (0.06/0.02/0.00), то есть сейчас система почти простаивает.

Command:

```sh
w
```

Output:

```text
 20:08:42 up 63 days,  3:31,  1 user,  load average: 0.06, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU  WHAT
root              31.57.61.235     20:05    3days  0.00s  0.08s sshd: root@pts/0
```

Observation:
Сейчас активен 1 пользователь (root), подключение по SSH с IP 31.57.61.235. По IDLE странно показывает 3 дня, но при этом логин в 20:05 — возможно особенность/глюк учёта idle или сессия держится долго.

### 1.2 Process Forensics

Command:

```sh
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -n 6
```

Output:

```text
    PID    PPID CMD                         %MEM %CPU
 387992  387960 python -u bot_pdf_cleaner.p 20.9  0.0
   5367       1 /usr/lib/systemd/systemd-jo  1.7  0.0
  13352       1 /usr/bin/python3 /usr/bin/f  1.0  0.1
  14276       1 /usr/bin/dockerd -H fd:// -  0.9  0.1
 388027  388002 /usr/local/bin/python3.12 /  0.6  0.3
```

Observation:
Самый прожорливый по памяти процесс — python -u bot_pdf_cleaner.p (20.9% RAM). Остальные сильно меньше (journald ~1.7%, какие-то python сервисы ~1%, dockerd ~0.9%).

Command:

```sh
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6
```

Output:

```text
    PID    PPID CMD                         %MEM %CPU
 388027  388002 /usr/local/bin/python3.12 /  0.6  0.3
 748516       1 /usr/libexec/packagekitd     0.2  0.2
  13352       1 /usr/bin/python3 /usr/bin/f  1.0  0.1
 748296       1 /usr/lib/systemd/systemd --  0.1  0.1
  14276       1 /usr/bin/dockerd -H fd:// -  0.9  0.1
```

Observation:
По CPU сейчас вообще тихо: максимум 0.3% у python3.12 процесса (PID 388027). В целом это совпадает с маленьким load average.

### 1.3 Service Dependencies

Command:

```sh
systemctl list-dependencies --no-pager
```

Output:

```text
default.target
* |-display-manager.service
* |-systemd-update-utmp-runlevel.service
* `-multi-user.target
*   |-acpid.service
*   |-containerd.service
*   |-cron.service
*   |-dbus.service
*   |-dmesg.service
*   |-docker.service
*   |-e2scrub_reap.service
*   |-fail2ban.service
*   |-grub-common.service
*   |-grub-initrd-fallback.service
*   |-networkd-dispatcher.service
*   |-qemu-guest-agent.service
*   |-rsyslog.service
*   |-systemd-ask-password-wall.path
*   |-systemd-logind.service
*   |-systemd-networkd.service
*   |-systemd-update-utmp-runlevel.service
*   |-systemd-user-sessions.service
*   |-tuned.service
*   |-ufw.service
*   |-basic.target
*   | |--.mount
*   | |-tmp.mount
*   | |-paths.target
*   | | `-acpid.path
*   | |-slices.target
*   | | |--.slice
*   | | `-system.slice
*   | |-sockets.target
*   | | |-acpid.socket
*   | | |-dbus.socket
*   | | |-docker.socket
*   | | |-iscsid.socket
*   | | |-ssh.socket
*   | | |-systemd-initctl.socket
*   | | |-systemd-journald-dev-log.socket
*   | | |-systemd-journald.socket
*   | | |-systemd-pcrextend.socket
*   | | |-systemd-sysext.socket
*   | | |-systemd-udevd-control.socket
*   | | `-systemd-udevd-kernel.socket
*   | |-sysinit.target
*   | | |-dev-hugepages.mount
*   | | |-dev-mqueue.mount
*   | | |-kmod-static-nodes.service
*   | | |-ldconfig.service
*   | | |-open-iscsi.service
*   | | |-proc-sys-fs-binfmt_misc.automount
*   | | |-sys-fs-fuse-connections.mount
*   | | |-sys-kernel-config.mount
*   | | |-sys-kernel-debug.mount
*   | | |-sys-kernel-tracing.mount
*   | | |-systemd-ask-password-console.path
*   | | |-systemd-binfmt.service
*   | | |-systemd-firstboot.service
*   | | |-systemd-hwdb-update.service
*   | | |-systemd-journal-catalog-update.service
*   | | |-systemd-journal-flush.service
*   | | |-systemd-journald.service
*   | | |-systemd-machine-id-commit.service
*   | | |-systemd-modules-load.service
*   | | |-systemd-pcrmachine.service
*   | | |-systemd-pcrphase-sysinit.service
*   | | |-systemd-pcrphase.service
*   | | |-systemd-pstore.service
*   | | |-systemd-random-seed.service
*   | | |-systemd-repart.service
*   | | |-systemd-resolved.service
*   | | |-systemd-sysctl.service
*   | | |-systemd-sysusers.service
*   | | |-systemd-timesyncd.service
*   | | |-systemd-tmpfiles-setup-dev-early.service
*   | | |-systemd-tmpfiles-setup-dev.service
*   | | |-systemd-tmpfiles-setup.service
*   | | |-systemd-tpm2-setup-early.service
*   | | |-systemd-tpm2-setup.service
*   | | |-systemd-udev-trigger.service
*   | | |-systemd-udevd.service
*   | | |-systemd-update-done.service
*   | | |-systemd-update-utmp.service
*   | | |-cryptsetup.target
*   | | |-integritysetup.target
*   | | |-local-fs.target
*   | | | |--.mount
*   | | | |-systemd-fsck-root.service
*   | | | `-systemd-remount-fs.service
*   | | |-swap.target
*   | | | `-swapfile.swap
*   | | `-veritysetup.target
*   | `-timers.target
*   |   |-apt-daily-upgrade.timer
*   |   |-apt-daily.timer
*   |   |-dpkg-db-backup.timer
*   |   |-e2scrub_all.timer
*   |   |-fstrim.timer
*   |   |-motd-news.timer
*   |   `-systemd-tmpfiles-clean.timer
*   |-cloud-init.target
*   | |-cloud-config.service
*   | |-cloud-final.service
*   | |-cloud-init-hotplugd.socket
*   | |-cloud-init-local.service
*   | `-cloud-init.service
*   |-getty.target
*   | |-getty-static.service
*   | |-getty@tty1.service
*   | `-serial-getty@ttyS0.service
*   `-remote-fs.target
```

Observation:
default.target ведёт к multi-user.target (и ещё есть display-manager.service, но судя по серверу он может быть просто в конфиге/пакетах). В multi-user.target видно типичный серверный набор: docker, containerd, ufw, fail2ban, cron, rsyslog, плюс qemu-guest-agent (ещё один признак виртуалки).

Command:

```sh
systemctl list-dependencies multi-user.target --no-pager
```

Output:

```text
multi-user.target
* |-acpid.service
* |-containerd.service
* |-cron.service
* |-dbus.service
* |-dmesg.service
* |-docker.service
* |-e2scrub_reap.service
* |-fail2ban.service
* |-grub-common.service
* |-grub-initrd-fallback.service
* |-networkd-dispatcher.service
* |-qemu-guest-agent.service
* |-rsyslog.service
* |-systemd-ask-password-wall.path
* |-systemd-logind.service
* |-systemd-networkd.service
* |-systemd-update-utmp-runlevel.service
* |-systemd-user-sessions.service
* |-tuned.service
* |-ufw.service
* |-basic.target
* | |--.mount
* | |-tmp.mount
* | |-paths.target
* | | `-acpid.path
* | |-slices.target
* | | |--.slice
* | | `-system.slice
* | |-sockets.target
* | | |-acpid.socket
* | | |-dbus.socket
* | | |-docker.socket
* | | |-iscsid.socket
* | | |-ssh.socket
* | | |-systemd-initctl.socket
* | | |-systemd-journald-dev-log.socket
* | | |-systemd-journald.socket
* | | |-systemd-pcrextend.socket
* | | |-systemd-sysext.socket
* | | |-systemd-udevd-control.socket
* | | `-systemd-udevd-kernel.socket
* | |-sysinit.target
* | | |-dev-hugepages.mount
* | | |-dev-mqueue.mount
* | | |-kmod-static-nodes.service
* | | |-ldconfig.service
* | | |-open-iscsi.service
* | | |-proc-sys-fs-binfmt_misc.automount
* | | |-sys-fs-fuse-connections.mount
* | | |-sys-kernel-config.mount
* | | |-sys-kernel-debug.mount
* | | |-sys-kernel-tracing.mount
* | | |-systemd-ask-password-console.path
* | | |-systemd-binfmt.service
* | | |-systemd-firstboot.service
* | | |-systemd-hwdb-update.service
* | | |-systemd-journal-catalog-update.service
* | | |-systemd-journal-flush.service
* | | |-systemd-journald.service
* | | |-systemd-machine-id-commit.service
* | | |-systemd-modules-load.service
* | | |-systemd-pcrmachine.service
* | | |-systemd-pcrphase-sysinit.service
* | | |-systemd-pcrphase.service
* | | |-systemd-pstore.service
* | | |-systemd-random-seed.service
* | | |-systemd-repart.service
* | | |-systemd-resolved.service
* | | |-systemd-sysctl.service
* | | |-systemd-sysusers.service
* | | |-systemd-timesyncd.service
* | | |-systemd-tmpfiles-setup-dev-early.service
* | | |-systemd-tmpfiles-setup-dev.service
* | | |-systemd-tmpfiles-setup.service
* | | |-systemd-tpm2-setup-early.service
* | | |-systemd-tpm2-setup.service
* | | |-systemd-udev-trigger.service
* | | |-systemd-udevd.service
* | | |-systemd-update-done.service
* | | |-systemd-update-utmp.service
* | | |-cryptsetup.target
* | | |-integritysetup.target
* | | |-local-fs.target
* | | | |--.mount
* | | | |-systemd-fsck-root.service
* | | | `-systemd-remount-fs.service
* | | |-swap.target
* | | | `-swapfile.swap
* | | `-veritysetup.target
* | `-timers.target
* |   |-apt-daily-upgrade.timer
* |   |-apt-daily.timer
* |   |-dpkg-db-backup.timer
* |   |-e2scrub_all.timer
* |   |-fstrim.timer
* |   |-motd-news.timer
* |   `-systemd-tmpfiles-clean.timer
* |-cloud-init.target
* | |-cloud-config.service
* | |-cloud-final.service
* | |-cloud-init-hotplugd.socket
* | |-cloud-init-local.service
* | `-cloud-init.service
* |-getty.target
* | |-getty-static.service
* | |-getty@tty1.service
* | `-serial-getty@ttyS0.service
* `-remote-fs.target
```

Observation:
Тут проще видно, что multi-user.target реально тянет почти все основные сервисы (сеть, ssh socket, docker, firewall). То есть “нормальный” серверный профиль.

### 1.4 User Sessions

Command:

```sh
who -a
```

Output:

```text
           system boot  Dec 25 16:36
           run-level 5  Dec 25 16:37
LOGIN      ttyS0        Dec 25 16:37               577 id=tyS0
LOGIN      tty1         Dec 25 16:37               576 id=tty1
root     - pts/0        Feb 26 20:05   .        748319 (31.57.61.235)
           pts/1        Feb 26 20:08            748525 id=ts/1  term=0 exit=0
           pts/2        Dec 25 20:14             14787 id=ts/2  term=0 exit=0
           pts/3        Dec 25 21:02             16140 id=ts/3  term=0 exit=0
           pts/4        Dec 25 21:00             23743 id=ts/4  term=0 exit=0
           pts/5        Dec 25 21:22             24005 id=ts/5  term=0 exit=0
```

Observation:
Видно system boot 25 Dec 16:36 и run-level 5 (как “графический”, хотя это сервер — возможно просто уровень по умолчанию). Активная сессия root сейчас на pts/0. Есть старые pts/2..pts/5 с Dec 25 (уже завершены).

Command:

```sh
last -n 5
```

Output:

```text
root     pts/0        31.57.61.235     Thu Feb 26 20:05   still logged in
root     pts/0        31.57.61.235     Mon Feb 23 14:48 - 14:49  (00:01)
root     pts/0        31.57.61.235     Mon Feb 23 14:28 - 14:28  (00:00)
root     pts/0        95.182.115.130   Tue Jan 20 11:06 - 12:21  (01:14)
root     pts/0        95.182.115.130   Mon Jan 19 11:39 - 12:15  (00:36)

wtmp begins Thu Dec 25 16:36:59 2025
```

Observation:
Последние входы — root по SSH. Сейчас залогинен с 31.57.61.235. До этого были короткие сессии 23 Feb. В январе заходили с 95.182.115.130. Лог начался с момента поднятия системы (25 Dec 2025).

### 1.5 Memory Analysis

Command:

```sh
free -h
```

Output:

```text
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       2.2Gi       294Mi       3.8Mi       5.6Gi       5.5Gi
Swap:            9Gi        32Mi         9Gi
```

Observation:
RAM всего 7.8 GiB, при этом “used” 2.2 GiB, но большая часть в buff/cache (5.6 GiB) — это норм для Linux (кэш под файловую систему). available 5.5 GiB, то есть реально памяти достаточно. Swap почти не используется (32 MiB из 9 GiB).

Command:

```sh
cat /proc/meminfo | grep -e MemTotal -e SwapTotal -e MemAvailable
```

Output:

```text
MemTotal:        8132324 kB
MemAvailable:    5813852 kB
SwapTotal:      10485756 kB
```

Observation:
MemAvailable ~5.8 GB (в kB), что совпадает с free -h (5.5 GiB available, разница из-за округления).

### Answers / Summary

What is the top memory-consuming process?
python -u bot_pdf_cleaner.p (PID 387992) — 20.9% MEM.

Resource utilization patterns:
Система почти не нагружена по CPU (load average ~0.06). По памяти всё спокойно: много ушло в кэш, но available большой. Самый заметный потребитель RAM — один python-процесс (похоже мой сервис/бот), остальные процессы на фоне.


## Task 2 — Networking Analysis

### 2.1 Network Path Tracing

Command:
```sh
traceroute github.com
```

Output:

```text
traceroute to github.com (140.82.113.4), 30 hops max, 60 byte packets
 1  _gateway (31.57.63.1)  1.976 ms  1.889 ms  1.846 ms
 2  * * *
 3  * * *
 4  * * *
 5  * * *
 6  eqix-dc5.github-2.com (206.126.237.205)  61.269 ms  60.060 ms  60.001 ms
 7  * * *
 8  * * *
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
```

Observation:
Первый хоп — это локальный шлюз внутри сети провайдера/виртуалки (31.57.63.1) с небольшой задержкой ~2 мс. Дальше почти все хопы скрыты (звёздочки), то есть промежуточные маршрутизаторы не отвечают на ICMP/UDP traceroute или режут TTL exceeded. При этом на 6-м хопе виден узел GitHub на Equinix (eqix-dc5.github-2.com, 206.126.237.205) с RTT около 60 мс, то есть путь реально доходит до сети GitHub, просто большая часть трассы не светится.

Command:

```sh
dig github.com
```

Output:

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> github.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49515
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;github.com.                    IN      A

;; ANSWER SECTION:
github.com.             9       IN      A       140.82.113.4

;; Query time: 1 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Thu Feb 26 20:19:30 UTC 2026
;; MSG SIZE  rcvd: 55
```

Observation:
Резолвится A-запись github.com -> 140.82.113.4, статус NOERROR. DNS-сервер указан 127.0.0.53 — это локальный stub от systemd-resolved, который дальше уже форвардит запрос наружу. Время запроса 1 мс, похоже ответ пришёл быстро (возможно из кэша).

### 2.2 Packet Capture

Command:

```sh
sudo timeout 10 tcpdump -c 5 -i any 'port 53' -nn
```

Output:

```text
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
20:20:09.147672 lo    In  IP 127.0.0.1.11386 > 127.0.0.53.53: 16119+ [1au] A? github.com. (51)
20:20:09.148136 eth0  Out IP 31.57.63.237.28411 > 1.1.1.1.53: 22587+ [1au] A? github.com. (39)
20:20:09.150723 eth0  In  IP 1.1.1.1.53 > 31.57.63.237.28411: 22587 1/0/1 A 140.82.113.4 (55)
20:20:09.150993 lo    In  IP 127.0.0.53.53 > 127.0.0.1.11386: 16119 1/0/1 A 140.82.113.4 (55)
20:20:11.304994 lo    In  IP 127.0.0.1.41099 > 127.0.0.53.53: 27926+ [1au] A? github.com. (51)
5 packets captured
11 packets received by filter
0 packets dropped by kernel
```

Analysis of DNS query/response patterns:
Тут видно типичный паттерн systemd-resolved: приложение делает запрос на 127.0.0.53 (через lo), затем systemd-resolved отправляет реальный DNS-запрос наружу (eth0) на 1.1.1.1:53, получает ответ обратно от 1.1.1.1 и возвращает ответ приложению обратно на loopback. То есть локальный stub выступает прокси между приложением и внешним резолвером.

One example DNS query from packet capture (sanitized):

```text
20:20:09.148136 eth0  Out IP <server_ip>.28411 > 1.1.1.1.53: 22587+ [1au] A? github.com. (39)
```

### 2.3 Reverse DNS

Command:

```sh
dig -x 8.8.4.4
```

Output:

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> -x 8.8.4.4
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61260
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;4.4.8.8.in-addr.arpa.          IN      PTR

;; ANSWER SECTION:
4.4.8.8.in-addr.arpa.   78896   IN      PTR     dns.google.

;; Query time: 5 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Thu Feb 26 20:20:21 UTC 2026
;; MSG SIZE  rcvd: 73
```

Command:

```sh
dig -x 1.1.2.2
```

Output:

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> -x 1.1.2.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 40008
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;2.2.1.1.in-addr.arpa.          IN      PTR

;; AUTHORITY SECTION:
1.in-addr.arpa.         3600    IN      SOA     ns.apnic.net. read-txt-record-of-zone-first-dns-admin.apnic.net. 23597 7200 1800 604800 3600

;; Query time: 197 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Thu Feb 26 20:20:21 UTC 2026
;; MSG SIZE  rcvd: 137
```

Comparison of reverse lookup results:
Для 8.8.4.4 PTR-запись есть и нормально резолвится в dns.google (NOERROR). Для 1.1.2.2 PTR-записи нет (NXDOMAIN) — в ответе пришёл SOA для зоны 1.in-addr.arpa, что выглядит как “официально не существует записи” и поэтому обратное имя не выдаётся. Также заметно, что NXDOMAIN отвечал дольше (197 мс), чем успешный PTR для Google (5 мс).

