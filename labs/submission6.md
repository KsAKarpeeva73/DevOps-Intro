## Task 1 — Container Lifecycle & Image Management

### 1.1 Basic Container Operations

Сначала я посмотрела, какие контейнеры уже есть в системе.

#### Command
```sh
docker ps -a
```

#### Output
```text
CONTAINER ID   IMAGE                   COMMAND                  CREATED       STATUS      PORTS                                         NAMES
f5831db4fae7   amnezia-openvpn-cloak   "dumb-init /opt/amne…"   2 weeks ago   Up 7 days   0.0.0.0:43->443/tcp, [::]:43->443/tcp         amnezia-openvpn-cloak
4ab953316938   amnezia-xray            "dumb-init /opt/amne…"   2 weeks ago   Up 7 days   0.0.0.0:321->321/tcp, [::]:321->321/tcp       amnezia-xray
9c88e1508a41   amnezia-awg2            "dumb-init /opt/amne…"   2 weeks ago   Up 7 days   0.0.0.0:433->433/udp, [::]:433->433/udp       amnezia-awg2
9542101897b0   pdf-web                 "bash -c 'uvicorn ap…"   7 weeks ago   Up 7 days   0.0.0.0:8081->8000/tcp, [::]:8081->8000/tcp   pdf-cleaner-web
4b91238d14c4   pdf-bot                 "bash -c 'python -u …"   7 weeks ago   Up 7 days                                                 pdf-cleaner-bot
```

Дальше я скачала образ Ubuntu и посмотрела его в списке образов.

#### Command
```sh
docker pull ubuntu:latest
```

#### Output
```text
latest: Pulling from library/ubuntu
01d7766a2e4a: Pull complete
fd8cda969ed2: Download complete
Digest: sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

#### Command
```sh
docker images ubuntu
```

#### Output
```text
                                                                                                    i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
ubuntu:latest   d1e2e92c075e        119MB         31.7MB
```

После этого я посмотрела ID образа, его размер и число слоев.

#### Command
```sh
docker image inspect ubuntu:latest --format='Image ID: {{.Id}}
Image size (bytes): {{.Size}}
Layer count: {{len .RootFS.Layers}}'
```

#### Output
```text
Image ID: sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
Image size (bytes): 29737017
Layer count: 1
```

Дальше я запустила контейнер и посмотрела внутри версию ОС и процессы.

#### Command
```sh
docker run --name ubuntu_container ubuntu:latest bash -lc 'cat /etc/os-release; echo; ps aux || (apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y procps && ps aux)'
```

#### Output
```text
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1 35.7  0.0   4324  3456 ?        Ss   17:07   0:00 bash -lc cat /etc/os-release; echo; ps aux || (apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y procps && ps aux)
root          10  0.0  0.0   7888  3968 ?        R    17:07   0:00 ps aux
```

### 1.2 Image Export and Dependency Analysis

После этого я сохранила образ в tar-файл и посмотрела его размер.

#### Command
```sh
docker save -o ubuntu_image.tar ubuntu:latest
```

#### Output
```text
```

#### Command
```sh
ls -lh ubuntu_image.tar
```

#### Output
```text
-rw------- 1 root root 31M Mar 12 17:07 ubuntu_image.tar
```

Дальше я попробовала удалить образ, пока контейнер `ubuntu_container` еще существует.

#### Command
```sh
docker rmi ubuntu:latest
```

#### Output
```text
Error response from daemon: conflict: unable to delete ubuntu:latest (must be forced) - container 98b5aa610810 is using its referenced image d1e2e92c075e
```

После этого я удалила контейнер и повторила удаление образа.

#### Command
```sh
docker rm ubuntu_container
```

#### Output
```text
ubuntu_container
```

#### Command
```sh
docker rmi ubuntu:latest
```

#### Output
```text
Untagged: ubuntu:latest
Deleted: sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
```

В конце я еще раз посмотрела список контейнеров.

#### Command
```sh
docker ps -a
```

#### Output
```text
CONTAINER ID   IMAGE                   COMMAND                  CREATED       STATUS      PORTS                                         NAMES
f5831db4fae7   amnezia-openvpn-cloak   "dumb-init /opt/amne…"   2 weeks ago   Up 7 days   0.0.0.0:43->443/tcp, [::]:43->443/tcp         amnezia-openvpn-cloak
4ab953316938   amnezia-xray            "dumb-init /opt/amne…"   2 weeks ago   Up 7 days   0.0.0.0:321->321/tcp, [::]:321->321/tcp       amnezia-xray
9c88e1508a41   amnezia-awg2            "dumb-init /opt/amne…"   2 weeks ago   Up 7 days   0.0.0.0:433->433/udp, [::]:433->433/udp       amnezia-awg2
9542101897b0   pdf-web                 "bash -c 'uvicorn ap…"   7 weeks ago   Up 7 days   0.0.0.0:8081->8000/tcp, [::]:8081->8000/tcp   pdf-cleaner-web
4b91238d14c4   pdf-bot                 "bash -c 'python -u …"   7 weeks ago   Up 7 days                                                 pdf-cleaner-bot
```

### Analysis

Размер образа по `docker image inspect` — `29737017` байт. Размер файла `ubuntu_image.tar` — `31M`. Они близкие, но могут не совпадать точно.

Количество слоев у образа — `1`.

Первая попытка удалить образ завершилась ошибкой, потому что контейнер `ubuntu_container` все еще был создан на основе этого образа. Пока контейнер существует, Docker считает, что образ используется.

То есть сначала нужно удалить контейнер, и только потом можно удалить образ.

В `ubuntu_image.tar` входят слои образа, его конфигурация и метаданные, которые нужны для последующей загрузки через `docker load`.

---

## Task 2 — Custom Image Creation & Analysis

### 2.1 Deploy and Customize Nginx

Сначала я убрала старые контейнеры и старый образ, чтобы не было конфликтов.

#### Command
```sh
docker rm -f nginx_container my_website_container web web_new 2>/dev/null || true
```

#### Output
```text
```

#### Command
```sh
docker rmi -f my_website:latest 2>/dev/null || true
```

#### Output
```text
```

Дальше я скачала образ `nginx`.

#### Command
```sh
docker pull nginx
```

#### Output
```text
Using default tag: latest
latest: Pulling from library/nginx
df9da45c1db2: Pull complete
18a071c04bd1: Pull complete
a9d395129dce: Pull complete
75a1d70aee50: Pull complete
206356c42440: Pull complete
79697674b897: Pull complete
9eef040df109: Pull complete
d99947bc9177: Download complete
23abb0f9ce55: Download complete
Digest: sha256:bc45d248c4e1d1709321de61566eb2b64d4f0e32765239d66573666be7f13349
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

После этого я запустила контейнер `nginx_container`.

#### Command
```sh
docker run -d -p 80:80 --name nginx_container nginx
```

#### Output
```text
0c748e25536040e71cf97d4543d028e5d0d7ff3b093f81ce1867673f2cb5bae6
```

Сразу после запуска я проверила страницу через `curl`.

#### Command
```sh
curl http://localhost
```

#### Output
```text
curl: (56) Recv failure: Connection reset by peer
```

С первого раза ответ не пришел, потому что Nginx еще не успел нормально подняться.

Теперь я создала свой `index.html`.

#### Command
```sh
cat > index.html <<'EOF'
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
EOF
```

#### Output
```text
```

#### Command
```sh
cat index.html
```

#### Output
```text
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

Дальше я скопировала файл в контейнер.

#### Command
```sh
docker cp index.html nginx_container:/usr/share/nginx/html/
```

#### Output
```text
Successfully copied 2.05kB to nginx_container:/usr/share/nginx/html/
```

После этого страница уже открывалась с моим содержимым.

#### Command
```sh
curl http://localhost
```

#### Output
```text
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

### 2.2 Create and Test Custom Image

После этого я сохранила текущее состояние контейнера в новый образ `my_website:latest`.

#### Command
```sh
docker commit nginx_container my_website:latest
```

#### Output
```text
sha256:ece26b94835d6c1341b8d170572c4882a8c9caf5bd4f9d1543cd826b1a135565
```

#### Command
```sh
docker images my_website
```

#### Output
```text
                                                                                                    i Info →   U  In Use
IMAGE               ID             DISK USAGE   CONTENT SIZE   EXTRA
my_website:latest   ece26b94835d        237MB           63MB
```

Дальше я удалила исходный контейнер и запустила новый уже из своего образа.

#### Command
```sh
docker rm -f nginx_container
```

#### Output
```text
nginx_container
```

#### Command
```sh
docker run -d -p 80:80 --name my_website_container my_website:latest
```

#### Output
```text
7695b2a0bb0962a1111892e80f460600d505f0d3710a66b4050352ca31b54b72
```

Сразу после запуска я еще раз проверила страницу.

#### Command
```sh
curl http://localhost
```

#### Output
```text
curl: (56) Recv failure: Connection reset by peer
```

Дальше я посмотрела изменения в файловой системе контейнера.

#### Command
```sh
docker diff my_website_container
```

#### Output
```text
C /run
C /run/nginx.pid
C /etc
C /etc/nginx
C /etc/nginx/conf.d
C /etc/nginx/conf.d/default.conf
```

### Analysis

Я взяла стандартный образ `nginx`, заменила `index.html` внутри контейнера и проверила через `curl`, что теперь отдается мой HTML:

```html
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

В выводе `docker diff` у всех строк стоит `C`, то есть `Changed`. Это значит, что эти файлы или директории были изменены по сравнению с исходным состоянием контейнера.

Обозначения такие:
- `A` — added;
- `C` — changed;
- `D` — deleted.

Плюсы `docker commit`:
- быстро;
- удобно для разового эксперимента;
- можно сразу сохранить текущее состояние контейнера.

Минусы `docker commit`:
- плохо воспроизводится;
- не видно шагов сборки;
- неудобно поддерживать.

Плюсы Dockerfile:
- все шаги сборки описаны явно;
- сборку легко повторить;
- удобно хранить в репозитории;
- лучше подходит для нормальной разработки.

Минусы Dockerfile:
- нужно заранее описывать шаги;
- для быстрых экспериментов он менее удобен.

Для нормальной работы лучше Dockerfile, а `docker commit` удобен для быстрого эксперимента.

---

## Task 3 — Container Networking & Service Discovery

### 3.1 Create Custom Network

Сначала я удалила старые контейнеры и сеть.

#### Command
```sh
docker rm -f container1 container2 2>/dev/null || true
docker network rm lab_network 2>/dev/null || true
```

#### Output
```text
```

Дальше я скачала образ `alpine`.

#### Command
```sh
docker pull alpine
```

#### Output
```text
Using default tag: latest
latest: Pulling from library/alpine
589002ba0eae: Pull complete
9e595aac14e0: Download complete
caa817ad3aea: Download complete
Digest: sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c3f659
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

После этого я создала сеть `lab_network`.

#### Command
```sh
docker network create lab_network
```

#### Output
```text
303794bd28bd1cfadd649fb3b93f504577d0f6821b71bf401e05deebd4419a35
```

Потом я посмотрела список сетей.

#### Command
```sh
docker network ls
```

#### Output
```text
NETWORK ID     NAME              DRIVER    SCOPE
9da3e9efa4fa   amnezia-dns-net   bridge    local
c4865cfd80ed   bridge            bridge    local
b1db0e87d9dd   host              host      local
303794bd28bd   lab_network       bridge    local
4bbb14d9c1d5   none              null      local
e05dd8261414   pdf_default       bridge    local
```

Дальше я запустила два контейнера в этой сети.

#### Command
```sh
docker run -dit --network lab_network --name container1 alpine ash
```

#### Output
```text
6323ed23d60c908ba2af2d9ae67c2a26f066e025505133cbd4d97d4fe3935cb9
```

#### Command
```sh
docker run -dit --network lab_network --name container2 alpine ash
```

#### Output
```text
e8d7b61eb9f2c6b91ca102022bfd2c30fefeaea6c229c60587470febec19e82e
```

### 3.2 Test Connectivity and DNS

После этого я проверила связь между контейнерами.

#### Command
```sh
docker exec container1 ping -c 3 container2
```

#### Output
```text
PING container2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.127 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.099 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.132 ms

--- container2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.099/0.119/0.132 ms
```

Потом я посмотрела параметры сети и IP-адреса контейнеров.

#### Command
```sh
docker network inspect lab_network
```

#### Output
```text
[
    {
        "Name": "lab_network",
        "Id": "303794bd28bd1cfadd649fb3b93f504577d0f6821b71bf401e05deebd4419a35",
        "Created": "2026-03-12T17:13:35.825908407Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "IPRange": "",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Options": {},
        "Labels": {},
        "Containers": {
            "6323ed23d60c908ba2af2d9ae67c2a26f066e025505133cbd4d97d4fe3935cb9": {
                "Name": "container1",
                "EndpointID": "c40eb543f448e8c0f956ee361675a907c8d7b396824c007ab58cd0d546c4fff8",
                "MacAddress": "f6:50:5b:43:34:4b",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "e8d7b61eb9f2c6b91ca102022bfd2c30fefeaea6c229c60587470febec19e82e": {
                "Name": "container2",
                "EndpointID": "ca3ac2ccc400e058cb9ea4da7f0de784af9bfdccd5be982f9ac3bdff9ad785da",
                "MacAddress": "d6:3b:37:e3:d3:07",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Status": {
            "IPAM": {
                "Subnets": {
                    "172.19.0.0/16": {
                        "IPsInUse": 5,
                        "DynamicIPsAvailable": 65531
                    }
                }
            }
        }
    }
]
```

После этого я проверила DNS-разрешение имени `container2`.

#### Command
```sh
docker exec container1 nslookup container2
```

#### Output
```text
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:
Name:   container2
Address: 172.19.0.3

Non-authoritative answer:
```

### Analysis

Пинг прошел успешно, значит контейнеры в одной сети видят друг друга.

По `docker network inspect` видно, что:
- `container1` получил `172.19.0.2/16`;
- `container2` получил `172.19.0.3/16`.

По `nslookup` видно, что имя `container2` разрешается во внутренний IP `172.19.0.3`.

Это работает за счет внутреннего DNS Docker. В пользовательской сети Docker сам связывает имена контейнеров с их IP-адресами, поэтому можно обращаться по имени, а не по IP.

Плюсы user-defined bridge network по сравнению с обычной `bridge`:
- контейнеры могут обращаться друг к другу по имени;
- сеть лучше изолирована;
- удобнее управлять контейнерами;
- не нужно вручную искать IP.

---

## Task 4 — Data Persistence with Volumes

### 4.1 Create and Use Volume

Сначала я удалила старые контейнеры, которые могли занимать порт `80`.

#### Command
```sh
docker rm -f my_website_container web web_new 2>/dev/null || true
```

#### Output
```text
my_website_container
```

Дальше я пересоздала volume `app_data`.

#### Command
```sh
docker volume rm app_data 2>/dev/null || true
docker volume create app_data
```

#### Output
```text
app_data
```

После этого я посмотрела список volume.

#### Command
```sh
docker volume ls
```

#### Output
```text
DRIVER    VOLUME NAME
local     8e6851f21fdb7bd9a3d02d66359a0086b9d34a2217abb2f638970854c805927d
local     34b72800effbac4db81f08e2cee5e842fd46ecf21234b2bafc9571b3e350eef4
local     685e3588d2909acf742627298325ce90dc7932c37a7302282170f89e2f1c3632
local     1810c51bd27622450c5f5ba61be8a6435b2991fc30230b2bc942fd6d20db9478
local     9465cecc5c6a85bfc766c6b63b978738b3eb2b3f1d328134dc94868cba496143
local     app_data
local     b57fab918fe65891a1fa47bf6163b57d784a12ce24dff885a351c67b8cffd173
local     d28e526c4b534b7370955145ec0a3c27ab9a62f4e60efde667ed1cafd3db3c2e
local     fa7264ed72506c0edd3d1100596f0169cb03d631603537c902c2280315bf4855
```

Дальше я запустила контейнер `web` с volume `app_data`.

#### Command
```sh
docker run -d -p 80:80 -v app_data:/usr/share/nginx/html --name web nginx
```

#### Output
```text
237bb57ee87581bfc2ccacb1835d933bf0184f14f48a148635aa713cb8a2c158
```

После этого я создала свой `index.html`.

#### Command
```sh
cat > index.html <<'EOF'
<html><body><h1>Persistent Data</h1></body></html>
EOF
```

#### Output
```text
```

#### Command
```sh
cat index.html
```

#### Output
```text
<html><body><h1>Persistent Data</h1></body></html>
```

Потом я скопировала этот файл в контейнер.

#### Command
```sh
docker cp index.html web:/usr/share/nginx/html/
```

#### Output
```text
Successfully copied 2.05kB to web:/usr/share/nginx/html/
```

После этого я проверила содержимое через `curl`.

#### Command
```sh
curl http://localhost
```

#### Output
```text
<html><body><h1>Persistent Data</h1></body></html>
```

### 4.2 Verify Persistence

Дальше я остановила и удалила контейнер `web`.

#### Command
```sh
docker stop web && docker rm web
```

#### Output
```text
web
web
```

После этого я создала новый контейнер `web_new` с тем же volume `app_data`.

#### Command
```sh
docker run -d -p 80:80 -v app_data:/usr/share/nginx/html --name web_new nginx
```

#### Output
```text
da08d824cd7b222447ade7692984677ab5e221b7d11176e34baf640ecee3b667
```

Потом я снова проверила страницу.

#### Command
```sh
curl http://localhost
```

#### Output
```text
<html><body><h1>Persistent Data</h1></body></html>
```

В конце я посмотрела информацию о volume.

#### Command
```sh
docker volume inspect app_data
```

#### Output
```text
[
    {
        "CreatedAt": "2026-03-12T17:15:27Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/app_data/_data",
        "Name": "app_data",
        "Options": null,
        "Scope": "local"
    }
]
```

### Analysis

Для проверки я использовала такой HTML:

```html
<html><body><h1>Persistent Data</h1></body></html>
```

После удаления контейнера `web` и запуска `web_new` данные не исчезли. Значит, файл сохранился не внутри контейнера, а во внешнем volume `app_data`.

Смысл volume в том, что данные живут отдельно от контейнера. Контейнер можно удалить и создать заново, а данные останутся.

Это важно для приложений, которым нужно сохранять файлы, конфиги, базу данных и другие данные между перезапусками.

Разница такая:
- `volumes` — отдельное хранилище Docker, подходит для постоянных данных;
- `bind mounts` — папка с хоста, удобно в разработке;
- `container storage` — обычный слой контейнера, исчезает вместе с ним.

Для постоянных данных правильнее использовать volume.
