# 📘 Команды в Docker CLI
<!--markdownlint-disable MD046-->

## 📌 Логические значения в командах Docker

Перед изучением команд более подробно стоит знать, что во многих командах **Docker** используются флаги с логическими значениями.

Особенности

1. `-f`, `-f=true`, `--force=true` — всё это эквивалентно.
2. Значение `false` указывается явно: `--rm=false`
3. Некоторые флаги по умолчанию включены (`true`), другие — выключены (`false`), чтобы узнать значение по-умолчанию используйте `--help`.
4. Лучше всегда указывать явно, особенно в скриптах или CI/CD:

```bash
# Лучше явно:
docker rm -f=true mycontainer
```

---

## 🆘 Опция `--help`

Каждая команда в **Docker CLI** поддерживает флаг `--help`, который выводит краткую справку:

```bash
docker run --help
docker build --help
```

Данная команда показывает:

- Список доступных флагов и опций, и их значения по-умолчанию
- Формат вызова
- Краткие описания

!!! info

    **Совет**: используйте `--help` вместо поиска в интернете.
    Таким образом вы быстрее запомните команды и начнете разбираться в них.

---

## 🚀 Команда `docker run`

`docker run` — основная команда для запуска контейнеров в Docker.
Она поддерживает множество опций, позволяющих гибко настраивать поведение контейнера.

Ниже приведены ключевые флаги, которые чаще всего используются при работе с **Docker**.

### `-a` / `--attach`

Флаг `--attach` подключает стандартные потоки (`stdin`, `stdout`, `stderr`) контейнера к текущему терминалу.
Это поведение по умолчанию, если не используется флаг `--detach`.

**Что это значит**: вы видите вывод контейнера прямо в своём терминале, и можете взаимодействовать с ним (если запущен в интерактивном режиме).

```bash
docker run --rm ubuntu echo "Hello from container"
# Вы увидете в терминале: "Hello from container"
```

### `-d` / `--detach`

Флаг `--detach` запускает контейнер в фоновом режиме.
Вы не будете видеть вывод контейнера в терминале, но получите его `container ID`.

Это удобно для фоновых сервисов (например, баз данных, веб-серверов и т. д.).

```bash
docker run -d nginx
# Вы увидете в терминале: e6ddc7e8afbe9a2a2c4d108e7e1767c02f5b5fcb3f172cf7b3d3ff87d7585b8f
```

Это ID работающего контейнера.
Вывод самого контейнера (например, логов nginx) вы не увидите, но сможете посмотреть их позже с помощью:

```bash
docker logs <container_id>
```

### `-i` / `--interactive`

Флаг `--interactive` поддерживает доступность открытого потока `stdin`.
Обычно используется с флагом `--tty` для запуска контейнера в интерактивном режиме.

```bash
docker run -it debian /bin/bash
```

### `-t` / `--tty`

Флаг `--tty` создает псевдоустройство TTY(терминал).
Обычно используется с флагом `--interactive` для запуска контейнера в интерактивном режиме.

```bash
docker run -it debian /bin/bash
```

### `--restart`

Флаг `--restart` определяет политику перезапуска контейнера при его завершении.

Возможные варианты:

- `no` (не перезапускать).
- `on-failure` (перезапускать при ошибке).
- `always` (всегда перезапускать).
- `unless-stopped` (перезапускать, пока не остановлен вручную).

```bash
docker run --restart always nginx
```

Также может быть задан дополнительный необязательный аргумент - количество перезапусков.

```bash
docker run --restart on-failure:10 postgres
```

### `--rm`

Флаг `--rm` автоматически удаляет контейнер после его остановки.
Несовместим с флагом `--detach`.

```bash
docker run --rm alpine echo "Hello"
```

### `-e` / `--env` / `--env-file`

Флаг `--env` устанавливает переменные окружения внутри контейнера.

```bash
docker run -e MY_VAR=value ubuntu env | grep MY_VAR
# Вывод: MY_VAR=value
```

Также стоит отметить что есть флаг `--env-file`, которому можно передать заданный файл.

```bash
docker run --env-file .env ubuntu env
```

### `-h` / `--hostname`

Флаг `--hostname` задает hostname внутри контейнера.

```bash
docker run -h mycontainer ubuntu hostname
# Вывод: mycontainer
```

### `--name`

Флаг `--name` присваивает контейнеру имя (по умолчанию генерируется случайное).

```bash
docker run --name my_nginx nginx
```

### `-v` / `--volume` / `--volumes-from`

Флаг `--volume` подключает том (volume) или директорию хоста в контейнер.

```bash
docker run -v /host/path:/container/path ubuntu
```

А при помощи флага `--volumes-from` можно подключить другой контейнер как контейнер данных.

```bash
docker run -d --volumes-from dbdata --name db1 postgres
```

### `--expose`

Флаг `--expose` открывает порт внутри контейнера (но не публикует его на хост).

```bash
docker run --expose 8080 nginx
```

### `-p` / `--publish`

Флаг `--publish` публикует порт контейнера на хост.

```bash
docker run -p 8080:80 nginx
# Хост - 8080
# Контейнер - 80
```

### `-P` / `--publish-all`

Флаг `--publish-all` публикует все открытые порты контейнера на случайные порты хоста.

```bash
docker run -P nginx
```

### `--entrypoint`

Флаг `--entrypoint` переопределяет `ENTRYPOINT` образа.

```bash
docker run --entrypoint /bin/bash ubuntu
```

### `-u` / `--user`

Флаг `--user` задает пользователя для запуска процесса.
Заменяет `USER` из Dockerfile.

```bash
docker run -u 1000 ubuntu whoami
# Вывод: неизвестный пользователь (если UID 1000 не существует)
```

### `-w` / `--workdir`

Флаг `--workdir` устанавливает рабочую директорию внутри контейнера.
Заменяет любые значения из Dockerfile.

```bash
docker run -w /app ubuntu pwd
# Вывод: /app
```

---

## 🧱 Команды для управления контейнерами

Ниже приведены ключевые команды для управления контейнерами.

### `docker attach`

Подключается к работающему контейнеру, чтобы видеть его вывод и взаимодействовать с ним.

```bash
ID=$(docker run -d debian sh -c "while true; do echo 'tick'; sleep 1; done;")
docker attach $ID
```

!!! warning

    Используйте с осторожностью — завершение сессии может остановить контейнер, если не указан флаг `-d`.
    То есть использование комбинации клавиш **Ctrl + C** для выхода завершит процесс и приведет к завершению работы контейнера.

### `docker create`

Создает новый контейнер из образа, но не запускает его.
Флаги этой команды в основном те же, что для команды `docker run`.

```bash
docker create --name mycontainer ubuntu
```

### `docker cp`

Копирует файлы между файловыми системами хоста и контейнера.

```bash
docker cp mycontainer:/file.txt ./file.txt
docker cp ./script.sh mycontainer:/script.sh
```

### `docker exec`

Выполняет команду внутри уже работающего контейнера.

Может использоваться для замены `ssh` при входе.

```bash
ID=$(docker run -d debian sh -c "while true; do sleep 1; done;")
docker exec $ID echo "Hello" # Hello
docker exec -it $ID /bin/bash
```

### `docker kill`

Посылает сигнал основному процессу в контейнере.
По умолчанию - `SIGKILL`, который немедлено завершает работу контейнера.

```bash
docker kill mycontainer
```

Однако при помощи флага `-s` можно передать другой сигнал.

```bash
ID=$(docker run -d debian bash -c \
    "trap 'echo caught' SIGTRAP; while true; do sleep 1; done;")
docker kill -s SIGTRAP $ID # SIGTRAP
docker logs $ID # caught
docker kill $ID # SIGKILL
```

### `docker pause` / `docker unpause`

Приостанавливает / возобновляет все процессы в контейнере.

!!! info

    Данная команда не посылает никаких сигналов завершения, поэтому и процессы не могут быть полностью завершены.
    Вместо этого данная команда использует механизм приостановки `cgroups` внутри Linux ядра.

```bash
docker pause mycontainer
docker unpause mycontainer
```

### `docker restart`

Перезапускает контейнер, можно считать приблизительным аналогом последовательности команд `docker stop` и `docker start`.

```bash
docker restart mycontainer
```

Имеется флаг `-t`, который определяет интервал времени ожидания, необходимого для завершения работы контейнера.

```bash
docker restart -t 10 mycontainer
```

### `docker rm`

Удаляет остановленные контейнеры.

```bash
docker rm mycontainer
```

Чтобы удалить работающий контейнер можно использовать флаг `-f`.

```bash
docker rm -f mycontainer-running
```

При необходимости также можно удалить все тома, созданные данным контейнером.

```bash
docker rm -v mycontainer-with-volumes
```

!!! info

    **Из интересного**: командой ниже можно удалить все остановленные контейнеры.

    ```bash
    docker rm $(docker ps -aq)
    ```

### `docker start` / `docker stop`

Запускает / останавливает контейнер.

```bash
docker start mycontainer
docker stop mycontainer
```

Для команды остановки можно также как и для `docker restart` использовать флаг `-t`.

```bash
docker stop -t 10 mycontainer
```

---

## 🔍 Команды для получения информации о Docker

### `docker info`

Выводит общую информацию о системе **Docker**.

```bash
docker info
```

### `docker help`

Показывает справку по всем командам **Docker**.

```bash
docker help
```

### `docker version`

Выводит информацию о версии клиента и сервера **Docker**.

```bash
docker version
```

---

## 🔎 Команды для получения информации о контейнерах

### `docker diff`

Показывает изменения в файловой системе контейнера по сравнению с файловой системой образа.

```bash
ID=$(docker run -d ubuntu touch /NEW-FILE)
docker diff $ID
```

### `docker events`

Выводит в реальном времени события от демона.

```bash
docker events
```

### `docker inspect`

Предоставляет подробную информацию о заданных контейнерах или образах.

```bash
ID=$(docker run -d ubuntu touch /NEW-FILE)
docker inspect $ID
```

Можно отифильтровать вывод через флаг `-f`, который принимает шаблоны Go.

```bash
docker inspect -f "{{.Config}}" $ID
```

### `docker logs`

Выводит журналы контейнера, выводится все, что было записано в потоках `STDERR` и `STDOUT` внутри контейнера.

```bash
docker logs $ID
```

### `docker port`

Показывает, как порты контейнера сопоставлены с портами хоста.
Часто используется после `docker run -P` для получения назначенных портов.

```bash
ID=$(docker run -d -P redis)
docker port $ID
```

### `docker ps`

Предоставляет обшую информацию о работающих контейнерах.

```bash
docker ps
```

У данной команды огромное количество флагом, но самые важные:

- `-a`: позволяет вывести информацию о всех контейнерах, а не только о работающих.
- `-q`: возвращает только идентификаторы контейнеров.

```bash
docker ps -a
docker ps -aq
```

### `docker top`

Предоставляет информацию о процессах, выполняющихся внутри контейнера.

```bash
ID=$(docker run -d redis)
docker top $ID
docker top $ID -axZ
```

!!! info

    В действительности эта команда вызывает внутри контейнера утилиту `ps`.
    Также флаги `docker top` совпадают с флагами утилиты `ps`. Флаги `-ef` стоят по умолчанию.

---

## 📦 Команды для работы с образами

### `docker build`

Создает образ из **Dockerfile**.

```bash
docker build -t myimage .
```

### `docker commit`

Создает образ из указонного контейнера.

```bash
ID=$(docker run -d redis touch /new-file)
docker commit -a "Arthur Lokhov" -m "Comment" $ID commit:test
# sha256:93786e8672017a42fa4514a3055ce37b25b109e873ecd5477693472d957b19b3
docker images commit
# REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
# commit       test      93786e867201   34 seconds ago   211MB
docker run commit:test ls /new-file
# /new-file
```

### `docker export`

Экспортирует содержимое файловой системы контейнера в виде tar-архива, направляя его в стандартный `STDOUT` поток.

```bash
docker export mycontainer > container.tar
```

!!! warning

    Экспортируется только файловая система, а вот мета-данные(порты, CMD, ENTRYPOINT и так далее) будет потеряны.
    Также в файловую систему не будут включены какие-либо тома.

### `docker history`

Выводит информацию о каждоме слое(уровне) в образе.

```bash
docker history redis
```

### `docker images`

Выводит список локальных образов.

```bash
docker images
```

Можно вывести в том числе промежуточные образы, с использованием флага `-a`.

```bash
docker images -a
```

Также стоит выделить флаг `-q`, который возвращает только идентификаторы.

```bash
docker images -q
```

### `docker import`

Создает образ из архива, содержащего файловую систему.
Архив может быть передан как через `STDIN` поток, так и через URL.
Возвращает идентификатор нового созданного образа.

```bash
docker export $ID | docker import - flatten:test
docker history flatten:test 
```

!!! info

    Образ созданный из `docker import` состоит **только из одного уровня**.

### `docker load`

Загружает репозиторий из tar-архива, передаваемого через `STDIN` поток.

Репозиторий может содержать несколько образов и тегов.

Также в отличии от `docker import`, в загружаемыые образы включены метаданные и история.

```bash
docker load < image.tar
```

### `docker rmi`

Удаляет заданный образ.

Если передать команде только имя репозитория без тега, то предполагается тег `latest`.

```bash
docker rmi myimage
```

### `docker save`

Сохраняет именованные образы или репозитории в tar-архив, передаваемый в `STDOUT` поток.

Если задано только имя репозитория, то в архиве будут сохранены все образы, а не только с тегом `latest`.

```bash
docker save -o /tmp/redis.tar redis:latest
docker rmi redis:latest
docker load -i /tmp/redis.tar
docker images redis
```

### `docker tag`

Связывает имя репозитория и тега с заданным образом.

Если для нового имени тег не задан, то автоматически присваевается `latest`.

```bash
docker tag faa2b75ce09a newname # newname:latest
docker tag newname:latest amount/newname # amount/newname:latest
docker tag newname:latest amount/newname:newtag # amount/newname:newtag
```

---

## 🌐 Команды для работы с Docker Hub и другими реестрами

### `docker login`

Выполняют процедуру входа в **Docker Hub**(значение по умолчанию) или другой реестр.

```bash
docker login
```

### `docker logout`

Выполняет процедуру выхода из реестра. **Docker Hub** является значением по-умолчанию.

```bash
docker logout
```

### `docker pull`

Загружает образ из реестра. Реестр определяется по имени образа.

Также если имя тега не задано, то будет использоваться тег `latest`.

```bash
docker pull redis # Только :latest
```

С флагом `-a` происходит загрузка всех образов.

```bash
docker pull -a # Все образы со всеми тегами
```

### `docker push`

Выгружает образ или репозиторий в заданный реестр. При отсутствии тега выгружаются все образы, а не только с тегом `latest`.

```bash
docker push myrepo/myimage:latest
```

### `docker search`

Выводит список всех найденных репозиторием, соответствующих заданному шаблону.
Результат будет содержать не более 25 записей.

```bash
docker search mongo
```
