---
layout: post
title:  Docker In Action. Часть 1
date:   2017-10-26 15:43:36 +0000
categories: docker
---
> по мотивам [**«Docker In Action»**][dia_book]{:target="_blank"}

* TOC:
{:toc}

### Введение {#intro}

_Docker_ представляет собой набор из:

* [консольной программы][docker_cli]{:target="_blank"}
* [демона][docker_daemon]{:target="_blank"}
* набора служб

Основная функция _Docker_'а в том, что он упрощает использование таких технологий ядра _Linux_ как [простанства имен][namespaces]{:target="_blank"} и [cgroups][cgroups]{:target="_blank"}. В отличие от виртуальных машин _Docker_-контейнеры не используют аппаратную виртуализацию и запускаемые в них программы напрямую взаимодействуют с _Linux_-ядром хоста. Контейнеры запускаются как дочерние процессы _Docker_-демона.

_Docker_-контейнеры изолированы в отношении следующих 8 аспектов:

* пространство имен [PID][pid_ns]{:target="_blank"}
* пространство имен [UTS][uts_ns]{:target="_blank"}
* пространство имен [MNT][mnt_ns]{:target="_blank"}
* пространство имен [IPC][ipc_ns]{:target="_blank"}
* пространство имен [NET][net_ns]{:target="_blank"}
* пространство имен [USR][usr_ns]{:target="_blank"}
* [chroot()][chroot]{:target="_blank"}
* [cgroups][cgroups_ns]{:target="_blank"}

Контейнеры распространяются ввиде [образов][docker_image]{:target="_blank"}.

Сам _Docker_ устанавливается либо [напрямую][installation]{:target="_blank"} (при наличии поддерживающей технологии), либо с помощью [docker-toolbox][docker_toolbox]{:target="_blank"}.

Протестировать свежеустановленный _Docker_ можно запустив следующую команду:

~~~ bash
docker run --rm hello-world
~~~

Данная команда запускает новый контейнер. Сначала производится поиск образа с именем _hello-world_ на локальной машине. Если образ отсутствует, то _Docker_ выполняет поиск на [_Docker Hub_][docker_hub]{:target="_blank"}, который является реестром контейнеров по-умолчанию. После этого образ скачивается и устанавливается на локальной машине. Затем _Docker_ создает новый контейнер из этого образа и запускает его. Флаг `--rm` задаёт автоматическое удаление контейнера после его останова.

### Основы работы с контейнерами {#containers-basic}

Контейнеры запускаются в двух режимах:

* [detached][detached_mode]{:target="_blank"}
* [foreground][fg_mode]{:target="_blank"}

В _detached_-режиме контейнер запускается в фоне и не подсоединяется к потокам ввода-вывода терминала.

Пример:

~~~ bash
docker run --detach -p 8080:80 --name web nginx:latest
~~~

Флаг `-p` задает связывание порта _80_ контейнера с портом _8080_ хоста. Также контейнеру задается имя `web`.
При переходе на [http://localhost:8080](http://localhost:8080){:target="_blank"} должна отобразится стартовая страница _NGINX_.

_IP-адрес_ контейнера можно получить выполнив:

~~~ bash
docker inspect -f '{% raw %}{{ .NetworkSettings.IPAddress }}{% endraw %}' web
~~~

Опция `-f` (или `--format`) позволяет указать [шаблон][golang-template]{:target="_blank"} для вывода.

При запуске контейера _Docker_ присваивает ему уникальный идентификатор и, в случае _detach_-режима, выводит его на терминал.

Для установление связи между контейнерами используется флаг `--link` ([deprecated][dockerlinks]{:target="_blank"}).

~~~ bash
docker run --rm --interactive --tty --link web:web busybox:latest /bin/sh
~~~

Флаг `--link` задает добавление в _/etc/hosts_ соответствия для контейнера с именем _web_.

~~~ bash
cat /etc/hosts | grep web
~~~

Проверить полученную конфигурацию можно выполнив команду:

~~~ bash
wget -O - http://web:80/
~~~

Список запущенных контейнеров получаем вызвав:

~~~ bash
docker ps
~~~

Просмотреть логи контейнера можно выполнив:

~~~ bash
docker logs <container_name>
~~~

Останавливаем контейнер командой:

~~~ bash
docker stop <container_name>
~~~

Для переименования контейнера используется команда:

~~~ bash
docker rename <old_name> <new_name>
~~~

Создание нового контейнера (без запуска):

~~~ bash
docker create nginx:latest
~~~

Ниже приведена диаграмма переходов состояний docker-контейнера.

![Диаграмма переходов состояний docker-контейнеров]({{ site.url }}/images/2017-10-26-docker-in-action-p1/container-states.png)

Запуск контейнера и [внедрение переменной окружения][env_var]{:target="_blank"}:

~~~ bash
docker run --rm --env 'CUSTOM_ENV_VAR=custom variable' busybox:latest env
~~~

Для задания политики автоматического перезапуска контейнера, в случае завершения всех процессов, используется флаг `--restart`.
Значением по-умолчанию является `no`, т.е. контейнер не перезапускается.

Удаление остановленного контейнера выполняется с помощью команды:

~~~ bash
docker rm <container_name>
~~~

### Установка образов {#images-installation}

Образ является файлом в состав которого входят:

* файлы, которые будут доступны, созданному на его основе, контейнеру
* метаданные (взаимосвязь между образами, история команд, открытые порты, описания томов и пр.)

Репозитории представляют собой наборы образов. Имя репозитория состоит из

* _url_-адреса реестра
* имени пользователя
* короткого имени

и имеет вид `[[<registry_host>/]<user_name>/]<short_name>`.

Каждый образ в репозитории имеет уникальную метку(-и) ([_tag_][tag_glossary]{:target="_blank"}).

Примеры:

```
wordpress # :latest - по-умолчанию
wordpress:alpine
quay.io/dockerlibrary/wordpress:php5.6
quay.io/dockerlibrary/wordpress:php5.6-apache
```

Один и тот же образ может иметь несколько меток, например для уточнения версии (`4.8-php7.0-fpm` и `4.8.2-php7.0-fpm` для _wordpress_).

[Официальными репозиториями][official_repos]{:target="_blank"} называются репозитории курируемые _Docker Hub_.

Для регистрации в _Docker_-реестре используется команда `login`:

~~~ bash
docker login
~~~

Выход из _Docker_-реестра выполняется так:

~~~ bash
docker logout
~~~

Поиск репозиториев на _Docker Hub_ из командной строки выполняется командной `search`:

~~~ bash
docker search nginx
~~~

Удалить образ можно выполнив:

~~~ bash
docker rmi nginx:latest
~~~

Сохранение образа в файл:

~~~ bash
docker save -o nginx.latest.tar nginx:latest
~~~

Загрузка образа из файла:

~~~ bash
docker load -i nginx.latest.tar
~~~

Сборка (и установка) образа из _Dockerfile_:

~~~ bash
git clone https://github.com/nginxinc/docker-nginx.git
docker build -t my_nginx:stable docker-nginx/stable/alpine
~~~

Между образами имеет место отношение _родитель/потомок_, т.е. одни образы создаются на основе других. Слои - это образы, относящиеся к хотя бы одному другому образу. Доступные контейнеру файлы являются объединением всей цепочки зависимостей образа из которого создаётся контейнер.

### Постоянное хранение и разделяемое состояние {#persistent-storage}

_Docker_ предоставляет [несколько][mount_types]{:target="_blank"} способов внедрения в контейнер директорий хоста:

* [_volumes_][volumes]{:target="_blank"} (тома)
* [_bind mounts_][bind_mounts]{:target="_blank"} (монтирование)
* [_tmpfs_][tmpfs]{:target="_blank"} (память хоста)

#### Тома (_volumes_) {#volumes}

Тома создаются в управляемой _Docker_'ом части файловой системы хоста (_/var/lib/docker/volumes/_ на _Linux_).

[Создание тома][create_volume]{:target="_blank"}:

~~~ bash
docker volume create nginx-logs
~~~

Просмотр томов:

~~~ bash
docker volume ls
~~~

[Использование тома][start_container_with_volume]{:target="_blank"}:

~~~ bash
docker run -d --rm --name volume-test \
  --mount src=nginx-logs,target=/var/log/nginx nginx:latest
~~~

Если при запуске контейнера указан несуществующий том, то _Docker_ создаст его сам.

Также возможно копирование (метаданных) томов используемых одним контейнером в другой

~~~ bash
docker run --rm --name volumes-from-test \
  --volumes-from volume-test alpine:latest ls /var/log/nginx
~~~

Удаление тома:

~~~ bash
docker volume rm nginx-logs
~~~

#### Монтирование (_bind mounts_) {#bind-mounts}

Монтирование возможно для произвольного файла или директории файловой системы хоста.

~~~ bash
echo '<html><body>Hello, Docker!</body></html>' > bind.html
docker run -d --rm --name bind-mounts-test -p 8080:80 \
  --mount type=bind,src=$(pwd)/bind.html,target=/usr/share/nginx/html/index.html nginx:latest
~~~

#### Память хоста (tmpfs) {#tmpfs}

При использовании `tmpfs` данные сохраняются только в памяти хоста (или `swap`, если памяти не достаточно).
После останова контейнера содержимое будет удалено. При `commit`'е контейнера содержимое также не сохраняется.

~~~ bash
docker run -d --rm --name tmpfs-test \
    --mount type=tmpfs,destination=/usr/local/apache2/logs \
    --read-only httpd:2.4.29
~~~

### Настройка сети {#networking}

В ходе установки _Docker_ автоматически создает на машине хоста виртуальный интерфейс _docker0_ и конфигурирует его как _Ethernet_-мост.


Просмотр списка _Ethernet_-мостов:

~~~ bash
brctl show
~~~

При запуске контейнера доступны [следующие][network_settings_run]{:target="_blank"} варианты его подсоединения к сети:

* [`bridge`][bridge]{:target="_blank"} - создание настроек сети на основе _Ethernet_-моста по-умолчанию. См. [_linux network namespaces_][linux_network_namespaces]{:target="_blank"}
* `none` - не выполнять настройку (доступен только [_loopback_][loopback]{:target="_blank"})
* `container:<name|id>` - переиспользование настроек сети другого контейнера
* `host` - использование настроек сети хоста
* [`<network-name>|<network-id>`][user_defined_networks]{:target="_blank"} - подсоединение к пользовательской сети

Также доступны варианты [`macvlan`][macvlan]{:target="_blank"} и [`overlay`][overlay]{:target="_blank"}.

Создание пользовательской сети:

~~~ bash
docker network create --driver bridge custom_net
~~~

Использование пользовательской сети:

~~~ bash
docker run -d --rm --name nginx --network=custom_net nginx:latest
docker run --rm --network=custom_net alpine:latest ping -c 3 nginx
~~~

За разрешение имен отвечает встроенный _DNS_-сервер _Docker_'а.

Присоединение контейнера к пользовательской сети:

~~~ bash
docker run -d --rm --name nginx nginx:latest
docker network connect custom_net nginx
~~~

Существует [два][exposing_and_publishing]{:target="_blank"} различных механизма работы с портами _Docker_-контейнера:

* _exposing_
* _publishing_

Выставление (_exposing_) портов выполняется или с помощью ключевого слова `EXPOSE` в _Dockerfile_, или через указание флага `--expose` при запуске контейнера. Данный механизм используется для документирования.

При публикации (_publishing_) порты контейнера [отображаются][net_binding]{:target="_blank"} на порты хоста:

* заданные (`-p 8080:80`)
* или произвольные (`-p 80`) (см. [ip_local_port_range_][ip_local_port_range]{:target="_blank"})

Автоматическое отображение открытых (_exposed_) портов контейнера на произвольные порты хоста:

~~~ bash
docker run -d --rm -P --name nginx nginx:latest
~~~

Задание диапазона для отображения:

~~~ bash
docker run -d --rm -p 8080-8090:80 --name nginx nginx:latest
~~~

По-умолчанию, при указании флага `-p`, отображение будет выполнено для всех интерфейсов хоста.
Для указания интерфейса значение параметру задаётся в виде: `IP:host_port:container_port` или `IP::container_port`.

~~~ bash
docker run -d --rm -p 127.0.0.1:8080:80 --name nginx nginx:latest
docker port nginx 80 # проверка
~~~

### Изоляция {#isolation}

_Docker_ обеспечивает управление 2-мя видами, доступных контейнерам, ресурсов:

* [память][resource_constraints_memory]{:target="_blank"}
* [CPU][resource_constraints_cpu]{:target="_blank"}

Ограничение доступной памяти:

~~~ bash
docker run -d -P --rm --memory 256m nginx:latest
~~~

Задание __веса__, относительно других контейнеров, для предоставления времени _CPU_:

~~~ bash
docker run -d -P --rm --cpu-shares 512 nginx:latest
~~~

#### Устройства

Для настройки доступа к устройствам хоста используется флаг `--device`:

~~~ bash
docker -it --rm --device /dev/video0:/dev/video0 ubuntu:latest ls -al /dev
~~~

#### Разделяемая память

Для настройки взаимодействия между контейнерами на уровне памяти используется флаг [`--ipc`][ipc]{:target="_blank"}.

Пример использования: [Docker in Action: The Shared Memory Namespace - DZone Cloud][shared_memory_host]{:target="_blank"}.

#### Пользователи

Пользователем внутри контейнера, по-умолчанию, является _root_.
При совпадении UID пользователь контейнера имеет такие же права на файлы хоста как и этот пользователь.

Для указания пользователя при старте (`run`) или запуске (`create`) контейнера используется опция `-u` или `--user`.

~~~ bash
docker run --rm --user nobody:nogroup debian:latest id
~~~



### Полезные ссылки {#useful-links}

* [Resources Cleanup](https://gist.github.com/bastman/5b57ddb3c11942094f8d0a97d461b430){:target="_blank"}
* [Reference Architectures](https://success.docker.com/architectures){:target="_blank"}
* [Deep dive container networking][container_namespaces_deep]{:target="_blank"}

[dia_book]: https://www.manning.com/books/docker-in-action
[docker_cli]: https://docs.docker.com/engine/docker-overview/#the-docker-client
[docker_daemon]: https://docs.docker.com/engine/docker-overview/#the-docker-daemon
[namespaces]: https://en.wikipedia.org/wiki/Linux_namespaces
[cgroups]: https://en.wikipedia.org/wiki/Cgroups
[pid_ns]: http://man7.org/linux/man-pages/man7/pid_namespaces.7.html
[uts_ns]: https://en.wikipedia.org/wiki/Linux_namespaces#UTS
[mnt_ns]: http://man7.org/linux/man-pages/man7/mount_namespaces.7.html
[ipc_ns]: https://www.systutorials.com/docs/linux/man/7-namespaces/#lbAF
[net_ns]: http://man7.org/linux/man-pages/man8/ip-netns.8.html
[usr_ns]: http://man7.org/linux/man-pages/man7/user_namespaces.7.html
[chroot]: https://en.wikipedia.org/wiki/Chroot
[cgroups_ns]: http://man7.org/linux/man-pages/man7/cgroup_namespaces.7.html
[docker_image]: https://docs.docker.com/glossary/?term=image
[installation]: https://docs.docker.com/engine/installation
[docker_toolbox]: https://www.docker.com/products/docker-toolbox
[docker_hub]: https://hub.docker.com/
[detached_mode]: https://docs.docker.com/engine/reference/run/#detached--d
[fg_mode]: https://docs.docker.com/engine/reference/run/#foreground
[env_var]: https://docs.docker.com/engine/reference/run/#env-environment-variables
[tag_glossary]: https://docs.docker.com/glossary/?term=tag
[official_repos]: https://docs.docker.com/docker-hub/official_repos
[mount_types]: https://docs.docker.com/engine/admin/volumes/#more-details-about-mount-types
[volumes]: https://docs.docker.com/engine/admin/volumes/volumes
[bind_mounts]: https://docs.docker.com/engine/admin/volumes/bind-mounts
[tmpfs]: https://docs.docker.com/engine/admin/volumes/tmpfs
[create_volume]: https://docs.docker.com/engine/admin/volumes/volumes/#create-and-manage-volumes
[start_container_with_volume]: https://docs.docker.com/engine/admin/volumes/volumes/#start-a-container-with-a-volume
[network_settings_run]: https://docs.docker.com/engine/reference/run/#network-settings
[bridge]: https://docs.docker.com/engine/userguide/networking/#the-default-bridge-network
[linux_network_namespaces]: https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces
[loopback]: https://en.wikipedia.org/wiki/Loopback
[user_defined_networks]: https://docs.docker.com/engine/userguide/networking/#user-defined-networks
[macvlan]: https://docs.docker.com/engine/userguide/networking/get-started-macvlan
[overlay]: https://docs.docker.com/engine/userguide/networking/#overlay-networks-in-swarm-mode
[exposing_and_publishing]: https://docs.docker.com/engine/userguide/networking/#exposing-and-publishing-ports
[net_binding]: https://docs.docker.com/engine/userguide/networking/default_network/binding
[ip_local_port_range]: http://www.faqs.org/docs/securing/chap6sec70.html
[dockerlinks]: https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks
[container_namespaces_deep]: https://platform9.com/blog/container-namespaces-deep-dive-container-networking/
[resource_constraints_memory]: https://docs.docker.com/engine/admin/resource_constraints/#memory
[resource_constraints_cpu]: https://docs.docker.com/engine/admin/resource_constraints/#cpu
[shared_memory_namespace]: https://dzone.com/articles/docker-in-action-the-shared-memory-namespace
[ipc]: https://docs.docker.com/engine/reference/run/#ipc-settings-ipc
[shared_memory_host]: https://dzone.com/articles/docker-in-action-the-shared-memory-namespace
[golang-template]: https://golang.org/pkg/text/template