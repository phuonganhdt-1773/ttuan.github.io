# Dockerizing Rails application

1. Basic version

Create Docker file
```
# Choose ruby version, you can choose another version
FROM ruby:2.5.1
MAINTAINER tuantv.nhnd@gmail.com

# Language (Optional)
ENV LANG C.UTF-8

# Configure the main working directory.
ENV APP_DIR /app
RUN mkdir -p $APP_DIR
WORKDIR $APP_DIR

# Install apt based dependencies required to run Rails as well as RubyGems.
RUN apt-get update && apt-get install -y build-essential curl nodejs
RUN apt-get clean

# Copy the Gemfile as well as the Gemfile.lock and install the RubyGems.
COPY Gemfile /bundle/Gemfile
COPY Gemfile.lock /bundle/Gemfile.lock
RUN echo "gem: --no-rdoc --no-ri" > ~/.gemrc
RUN cd /bundle && bundle install -j4

# Copy the main application.
COPY . $APP_DIR
RUN chmod +x $APP_DIR/bin/*

# Expose port 3000 to the Docker host, so we can access it from the outside.
EXPOSE 3000

CMD rails server -b 0.0.0.0 -p 3000
```

and run this command to build: `docker build -t docker-demo .`

Run this command to launch: `docker run -p 3000:3000 -it docker-demo`.

2. Use docker-compose for ez management

docker-compose.yml
Tìm hiểu về các option trong này: build (context, docker file), command, volumes, ports, links, env_file, image, depend_on, environment, .... - docker directive
sự khác nhau giữa depends_on với link

```
version: "3"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    command: rails server -b 0.0.0.0
    volumes:
      - .:/app
      - bundle:/bundle # cache bundle data, map cai thu muc bundle(trong may local) vơi thư mục /bundle (mà mình cd xong chạy bundle install trong Dockerfile trên)
    ports:
      - 3000:3000
    # Connect with container db
    links:
      - db
    env_file: .env
    # Puts 2 lines to debug :D
    stdin_open: true
    tty: true
  db:
    image: mysql:5.6
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --innodb-file-format=Barracuda --innodb-file-per-table=true --innodb-large-prefix=true
    volumes:
      - db-data:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      - TZ=Japan
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
volumes:
  db-data:
    driver: local
  bundle:
    driver: local

```
Chú ý set password của mysql = ""
sau đó sửa file `dâtabase.yml`, thêm cái `host: db` vào.
```
docker-compose run app rake db:create
docker-compose run app rake db:migrate
```

Them muc xoa volume nua, neu k can than se dan den viec k connect duoc den db vi co password

3. Cache data

Trước mình có xe thằng busybox có thể cache được data, nhưng sau méo biết làm thế nào ;v

https://medium.com/@cristian_rivera/cache-rails-bundle-w-docker-compose-45512d952c2d

Vấn đề: Mỗi khi Gemfile thay đổi, hash của file sẽ bị thay đổi => Docker container nghĩ là sẽ cần build lại. Thời gian bundle install rất lâu, nên chúng ta cần tìm cách để cache lại gems data.

Solution:
+ Đưa logic của phần `bundle install` ra khỏi build process và đưa nó vào trong docker up process.
+ Set biến môi trường của bundle trong Dockerfile.
+ Tạo file docker-entrypoint.sh để check xem có cần cài thêm gem mới gì không, đã có gem gì rồi.
+ Trong docker-compose.yml file, mount cái volume chứa bundle data vào, sẽ load data từ đó chứ k cần cài lại mỗi lần container rebuild.

Trong Dockerfile:

```

COPY ./docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

ENV BUNDLE_PATH=/bundle \
    BUNDLE_BIN=/bundle/bin \
    GEM_HOME=/bundle
ENV PATH="${BUNDLE_BIN}:${PATH}"
```

docker-entrypoint.sh script này sẽ chạy khi container được `up`.

```
#!/bin/bash

# Tell docker exit if any command fails with non-zero exit code, return it's error message
set -e

# Check xem có gem nào thiếu không, nếu có, cài vào thư mục binstubs (chỗ chứa gem của mình)
bundle check || bundle install --binstubs="$BUNDLE_BIN"

# Chạy lệnh được truyền vào từ ban đầu sau khi ta đã bundle install xong
exec "$@"
```

Cuối cùng, ta config trong docker-compose.yml để tạo volumne để lưu bundle, sau đó map nó vào trong app service.
file docker-compose.yml sẽ giống với bên trên, map volume bundle vào.

Lúc muốn build lần đầu thì gõ: `docker-compose build`
Những lần sau thì chỉ cần gọi: `docker-compose up`

4. Use wait-for-it script, docker tag

Vấn đề: Khi chúng ta sử dụng `docker-compose` để quản lý nhiều container 1 lúc(ít nhất là sẽ có app và db), chúng ta sẽ cần quy định thứ tự chạy các container này.
Ví dụ ở đây, chúng ta dùng từ khoá `depends_on` hoặc `links` trong config của container app, điều đó có nghĩa là container app sẽ khởi động sau container db. Tuy nhiên, k có gì chắc chắn là container app sẽ connect được tới container db.

Vì container db sẽ mất thời gian để khởi động, init file, load existed database, ... sau đó mới "ready for connecting". Nên nếu app container connect tới db khi nó chưa ready, thì sẽ bị báo lỗi (ví dụ bên app mà chạy rake db:migrate, thì sẽ báo lỗi vì lúc đó chưa connect được tới db mà).

Có 1 giải pháp là cho bên app đợi 30s rồi sau đó mới connect tới db. tuy nhiên thế thì vẫn chưa chắc chắn lắm, hoặc sẽ lãng phí thời gian.

Đó là lý do `wait-for-it.sh` ra đời. https://github.com/vishnubob/wait-for-it

Solution:
+ Download wait-for-it.sh từ repo trên: `curl https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh`

```
#!/usr/bin/env bash
#   Use this script to test if a given TCP host/port are available

WAITFORIT_cmdname=${0##*/}

echoerr() { if [[ $WAITFORIT_QUIET -ne 1 ]]; then echo "$@" 1>&2; fi }

usage()
{
    cat << USAGE >&2
Usage:
    $WAITFORIT_cmdname host:port [-s] [-t timeout] [-- command args]
    -h HOST | --host=HOST       Host or IP under test
    -p PORT | --port=PORT       TCP port under test
                                Alternatively, you specify the host and port as host:port
    -s | --strict               Only execute subcommand if the test succeeds
    -q | --quiet                Don't output any status messages
    -t TIMEOUT | --timeout=TIMEOUT
                                Timeout in seconds, zero for no timeout
    -- COMMAND ARGS             Execute command with args after the test finishes
USAGE
    exit 1
}

wait_for()
{
    if [[ $WAITFORIT_TIMEOUT -gt 0 ]]; then
        echoerr "$WAITFORIT_cmdname: waiting $WAITFORIT_TIMEOUT seconds for $WAITFORIT_HOST:$WAITFORIT_PORT"
    else
        echoerr "$WAITFORIT_cmdname: waiting for $WAITFORIT_HOST:$WAITFORIT_PORT without a timeout"
    fi
    WAITFORIT_start_ts=$(date +%s)
    while :
    do
        if [[ $WAITFORIT_ISBUSY -eq 1 ]]; then
            nc -z $WAITFORIT_HOST $WAITFORIT_PORT
            WAITFORIT_result=$?
        else
            (echo > /dev/tcp/$WAITFORIT_HOST/$WAITFORIT_PORT) >/dev/null 2>&1
            WAITFORIT_result=$?
        fi
        if [[ $WAITFORIT_result -eq 0 ]]; then
            WAITFORIT_end_ts=$(date +%s)
            echoerr "$WAITFORIT_cmdname: $WAITFORIT_HOST:$WAITFORIT_PORT is available after $((WAITFORIT_end_ts - WAITFORIT_start_ts)) seconds"
            break
        fi
        sleep 1
    done
    return $WAITFORIT_result
}

wait_for_wrapper()
{
    # In order to support SIGINT during timeout: http://unix.stackexchange.com/a/57692
    if [[ $WAITFORIT_QUIET -eq 1 ]]; then
        timeout $WAITFORIT_BUSYTIMEFLAG $WAITFORIT_TIMEOUT $0 --quiet --child --host=$WAITFORIT_HOST --port=$WAITFORIT_PORT --timeout=$WAITFORIT_TIMEOUT &
    else
        timeout $WAITFORIT_BUSYTIMEFLAG $WAITFORIT_TIMEOUT $0 --child --host=$WAITFORIT_HOST --port=$WAITFORIT_PORT --timeout=$WAITFORIT_TIMEOUT &
    fi
    WAITFORIT_PID=$!
    trap "kill -INT -$WAITFORIT_PID" INT
    wait $WAITFORIT_PID
    WAITFORIT_RESULT=$?
    if [[ $WAITFORIT_RESULT -ne 0 ]]; then
        echoerr "$WAITFORIT_cmdname: timeout occurred after waiting $WAITFORIT_TIMEOUT seconds for $WAITFORIT_HOST:$WAITFORIT_PORT"
    fi
    return $WAITFORIT_RESULT
}

# process arguments
while [[ $# -gt 0 ]]
do
    case "$1" in
        *:* )
        WAITFORIT_hostport=(${1//:/ })
        WAITFORIT_HOST=${WAITFORIT_hostport[0]}
        WAITFORIT_PORT=${WAITFORIT_hostport[1]}
        shift 1
        ;;
        --child)
        WAITFORIT_CHILD=1
        shift 1
        ;;
        -q | --quiet)
        WAITFORIT_QUIET=1
        shift 1
        ;;
        -s | --strict)
        WAITFORIT_STRICT=1
        shift 1
        ;;
        -h)
        WAITFORIT_HOST="$2"
        if [[ $WAITFORIT_HOST == "" ]]; then break; fi
        shift 2
        ;;
        --host=*)
        WAITFORIT_HOST="${1#*=}"
        shift 1
        ;;
        -p)
        WAITFORIT_PORT="$2"
        if [[ $WAITFORIT_PORT == "" ]]; then break; fi
        shift 2
        ;;
        --port=*)
        WAITFORIT_PORT="${1#*=}"
        shift 1
        ;;
        -t)
        WAITFORIT_TIMEOUT="$2"
        if [[ $WAITFORIT_TIMEOUT == "" ]]; then break; fi
        shift 2
        ;;
        --timeout=*)
        WAITFORIT_TIMEOUT="${1#*=}"
        shift 1
        ;;
        --)
        shift
        WAITFORIT_CLI=("$@")
        break
        ;;
        --help)
        usage
        ;;
        *)
        echoerr "Unknown argument: $1"
        usage
        ;;
    esac
done

if [[ "$WAITFORIT_HOST" == "" || "$WAITFORIT_PORT" == "" ]]; then
    echoerr "Error: you need to provide a host and port to test."
    usage
fi

WAITFORIT_TIMEOUT=${WAITFORIT_TIMEOUT:-15}
WAITFORIT_STRICT=${WAITFORIT_STRICT:-0}
WAITFORIT_CHILD=${WAITFORIT_CHILD:-0}
WAITFORIT_QUIET=${WAITFORIT_QUIET:-0}

# check to see if timeout is from busybox?
WAITFORIT_TIMEOUT_PATH=$(type -p timeout)
WAITFORIT_TIMEOUT_PATH=$(realpath $WAITFORIT_TIMEOUT_PATH 2>/dev/null || readlink -f $WAITFORIT_TIMEOUT_PATH)
if [[ $WAITFORIT_TIMEOUT_PATH =~ "busybox" ]]; then
        WAITFORIT_ISBUSY=1
        WAITFORIT_BUSYTIMEFLAG="-t"

else
        WAITFORIT_ISBUSY=0
        WAITFORIT_BUSYTIMEFLAG=""
fi

if [[ $WAITFORIT_CHILD -gt 0 ]]; then
    wait_for
    WAITFORIT_RESULT=$?
    exit $WAITFORIT_RESULT
else
    if [[ $WAITFORIT_TIMEOUT -gt 0 ]]; then
        wait_for_wrapper
        WAITFORIT_RESULT=$?
    else
        wait_for
        WAITFORIT_RESULT=$?
    fi
fi

if [[ $WAITFORIT_CLI != "" ]]; then
    if [[ $WAITFORIT_RESULT -ne 0 && $WAITFORIT_STRICT -eq 1 ]]; then
        echoerr "$WAITFORIT_cmdname: strict mode, refusing to execute subprocess"
        exit $WAITFORIT_RESULT
    fi
    exec "${WAITFORIT_CLI[@]}"
else
    exit $WAITFORIT_RESULT
fi
```

Nhớ chạy `chmod +x wait-for-it.sh` đấy.

+ Sửa file docker-compose.yml để có thể gọi script này lúc khởi động:

docker-compose.yml

```
version: "3"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    command: ./wait-for-it.sh db:3306 -- ./docker-entrypoint.sh
    volumes:
      - .:/app
      - bundle:/bundle
    ports:
      - 3000:3000
    # Connect with container db
    links:
      - db
    env_file: .env
    # Puts 2 lines to debug :D
    stdin_open: true
    tty: true
  db:
    image: mysql:5.6
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --innodb-file-format=Barracuda --innodb-file-per-table=true --innodb-large-prefix=true
    volumes:
      - db-data:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      - TZ=Japan
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
volumes:
  db-data:
    driver: local
  bundle:
    driver: local

```
Trong docker-entrypoint.sh bổ sung thêm vài lệnh: (vì giờ mình move cái command run rails s đi rồi mà)

```
#!/bin/bash
set -e
bundle check || bundle install --binstubs="$BUNDLE_BIN"
bundle exec rake db:create
bundle exec rake db:migrate
bundle exec rails s -b 0.0.0.0
exec "$@"
```
Sau khi sửa xong nhớ chạy lại: `chmod +x docker-entrypoint.sh` để cấp quyền execute lại cho file này.

Trong file Dockerfile, xoá dòng : `ENTRYPOINT ["/docker-entrypoint.sh"]` đi, vì giờ mình chỉ chạy lệnh này sau khi db:3306 ready, nên đã move nó vào trong docker-compose.yml rồi

Bây giờ chạy `dc build` và `dc up` sẽ thấy đoạn:

```
app_1  | wait-for-it.sh: waiting 15 seconds for db:3306
app_1  | wait-for-it.sh: db:3306 is available after 0 seconds
```

5. Move to folder, use script to copy and build
6. Deployment with docker (bao ca nginx, whenever) https://code.likeagirl.io/how-to-dockerize-your-ruby-on-rails-development-environment-cede019905cd
7. Config SSL for deployed app
8. Add more with Redis, ELK stack
9. Debug on docker development
10. Docker network, volume,...
11. Khi nào thì mình cần build lại? khi thêm 1 gem mới có cần build lại không?
12. Cache mysql data, redis, ....
13. Remove tmp/pid khi khởi động lại server
14. magic of layer: https://steveltn.me/deploy-rails-applications-using-docker-4eccdcf05da0   https://medium.com/@jessgreb01/digging-into-docker-layers-c22f948ed612
15. None image: https://www.projectatomic.io/blog/2015/07/what-are-docker-none-none-images/
16. Khi nào thì cần build lại docker? chỉ khi có thay đổi gì đó trong image, tất cả các thao tác như: mount 1 file gì đó vào trong image, thay đổi trong docker compose, chekc gì đó trong volume, ... đều k cần build lại docker image or gọi docker-compose build ;) -> ngon
17. Dockerizing any application: https://hackernoon.com/how-to-dockerize-any-application-b60ad00e76da
