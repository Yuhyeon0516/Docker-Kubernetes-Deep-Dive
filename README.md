# Docker-Kubernetes-Deep-Dive

-   Udemy의 https://www.udemy.com/course/docker-kubernetes-2022 강의를 참고하여 공부하려고 한다.
-   편안하게 공부한것을 적는 일기장임
-   내부의 코드들은 대부분 강의에서 제공되었고 Docker의 구축 및 배포에 중점을 두었다.

### Section 1 Tutorial

-   Docker란

    -   Docker는 Container를 생성하고 관리하는 도구
        -   Container란 표준화된 소프트웨어 유닛

-   Docker 설정

    -   https://docs.docker.com/desktop/ 에서 OS별 요구사항을 충족한다면 Docker Desktop을 다운로드
    -   OS별 요구사항을 충족하지 못한다면 Docker Toolbox를 다운로드
    -   Linux는 Docker Engine을 설치하는 과정이 있음

-   first-demo

    -   Dockerfile 구성

        ```dockerfile
        # 다음 강의에 명령어에 대해 더 자세히 배워 볼 예정

        FROM node:14 # Node.js를 14버전 사용

        WORKDIR /app # 실행 Directory를 설정

        COPY package.json . # package.json을 docker container root에 카피

        RUN npm install # npm install을 진행

        COPY . . # root에 있는 모든 내용을 container root에 copy

        EXPOSE 3000 # 3000번 포트에서 실행

        CMD [ "node", "app.mjs" ] # node app.mjs 명령어를 실행
        ```

### Section 2 Docker Image & Container(Core Building Block)

-   Image & Container?

    -   Image는 Container를 위한 Templates/Blueprints임
        -   Container는 케이스, Image는 실제 내용물이라고 보면 편할듯

-   사전 빌드된 Image의 사용 & 실행

    -   https://hub.docker.com 에는 사전에 빌드된 Docker에서 관리하고있는 Image가 있음
    -   `docker run [image]` 명령어를 진행하면 local에 해당 image가 없을 시 library에서 가져온다고 알람이 발생하면서 다운로드되어 현재 실행중인 container에 추가되면서 실행됨(`docker ps -a` 명령어로 실행 중임을 확인할 수 있음)
    -   `docker run -it [image]` 명령어를 진행하면 앞에 받아두어 container에서 실행 중인 image로 실행됨

-   nodejs-app

    -   Dockerfile

        ```dockerfile
        FROM node # node image를 docker hub에서 가져옴

        WORKDIR /app # 아래 명령들의 작업 디렉토리를 /app으로 설정

        COPY . /app # 앞에 .은 Dockerfile을 기준으로 Copy해야할 대상의 경로, 뒤에 /app 은 docker image 내부의 경로 image 내부에 폴더가 존재하지 않는다면 생성됨

        RUN npm install # 실행하고 싶은 명령어를 실행 WORKDIR을 기준으로 실행됨

        EXPOSE 80 # 컨테이너를 실행 할 떄 이미지에 80번 포트가 필요하니 열어달라고 요청하는 느낌

        CMD ["node", "server.js"] # RUN 명령어와 유사하게 동작하지만 RUN은 이미지가 생성될 때 실행되고 CMD는 컨테이너가 시작될 때 실행됨
        ```

    -   `docker build .` 명령어를 진행
        -   `docker build [경로]` 명령어를 실행하면 커스텀 이미지가 생성됨
    -   `docker run -p 3000:80 c943d040fa68540871c75b241edc6c6a7956c0a7bd22a6bd531b5342b4d104c8` 명령어를 진행

        -   `docker run -p [외부 노출 포트]:[내부 컨테이너 포트] [Image Id]` 명령어를 실행하면 외부 노출 포트에 내부 컨테이너 포트에 맞는 이미지를 연결함

    -   Image에 변경 사항이 생겼을때 `docker run` 명령어를 입력하면 반영이 안됨
        -   Docker는 build가 진행되지 않으면 이전 결과물이 반영되어 변경점이 반영이 안됨
        -   그래서 변경점이 생기면 `docker build` 명령어를 진행해줘야함
    -   Docker는 cache를 사용하기 때문에 이전 사항에서 큰 변경점이 없다면 cache를 이용하여 build를 진행하여 속도의 차이가 보임

        -   추가로 이 cache 전략을 잘 활용하면 build 속도를 개선 시킬 수 있음

            -   아래와 같이 npm install 단계에서 cache를 사용하는 전략을 사용하면 build 속도를 개선 시킬 수 있음

            <table>
            <tr>
            <td> 변경 전 </td> <td> 변경 후 </td>
            </tr>
            <tr>
            <td>

            ```dockerfile
            FROM node

            WORKDIR /app

            COPY . /app
            # 파일 전체를 덮어쓰기하면서 package-lock.json과 node_modules가 삭제됨

            RUN npm install
            # package-lock.json과 node_modules가 삭제되면서 package에 변동이 있다고 docker에서 감지하고 npm install을 진행함

            EXPOSE 80

            CMD ["node", "server.js"]
            ```

            </td>
            <td>

            ```dockerfile
            FROM node

            WORKDIR /app

            COPY package.json /app
            # package.json만 먼저 복사해서 변경 사항이 있는지 체크함

            RUN npm install
            # 위의 package.json이 변경사항이 없다면 npm install도 cache가 사용되어 시간이 단축됨

            COPY . /app
            # 이후 나머지 파일을 전부 복사함

            EXPOSE 80

            CMD ["node", "server.js"]
            ```

            </td>
            </tr>
            </table>

-   Image & Container 관리

    -   `docker --help` 을 실행하면 어떤 명령어가 있는지 확인할 수 있음
    -   Container 중지
        -   `docker ps` 로 실행중인 docker container id 또는 name을 확인하여 `docker stop [id or name]`을 실행하면 container가 중지됨
    -   Container 재시작
        -   `docker ps -a` 로 docker history 중 재시작하고 싶은 container id 또는 name을 확인하여 `docker start [id or name]`을 실행하면 container가 재시작됨
        -   `docker run` 과의 다른점은 `docker run`은 새로운 container를 생성하고 실행하지만 `docker start`는 생성되어있는 container를 실행함
            -   그래서 `docker start`는 dockerfile이나 code의 변경이 없을때 주로 사용됨
    -   Attached & Detached Container

        -   위에서 알아본 `docker start` 로 docker container를 실행하는 경우 detached mode가 default이며 background에서 실행됨

            -   만약 `docker start` 로 실행된 container를 attached mode로 변경하여 foreground에서 실행하고 싶을때는 2가지 방법이 있다,

                1.  `docker ps`로 docker container id 또는 name을 확인하고 `docker attach [id or name]` 을 실행하면 attached mode로 foreground에서 실행된다.
                2.  `docker start` 를 실행할 때 -a flag를 이용하여 실행하면 attached mode로 foreground에서 실행된다.

                    (`docker start -a [id or name]`)

        -   반대로 `docker run` 으로 실행하는 경우 attached mode가 default이며 foreground에서 실행됨

            -   그래서 `docker run` 에서도 detached mode로 background에서 실행하고 싶을때는 -d flag를 이용하면 된다.

                (`docker run -p [외부 노출 포트]:[내부 컨테이너 포트] -d [Image Id]`)

        -   보통 Debug를 할떄 detached -> attached로 변경할 것으로 예상이 됨

-   python-app

    -   Dockerfile을 아래와 같이 구성하고 `docker build`와 `docker run`을 실행하면 에러가 발생한다. 이럴떄 사용하여야 하는것이 interactive mode이다.

        ```dockerfile
        FROM python

        WORKDIR /app

        COPY . /app

        CMD ["python", "rng.py"]

        '''Error
        Please enter the min number: Traceback (most recent call last):
        File "/app/rng.py", line 3, in <module>
            min_number = int(input('Please enter the min number: '))
                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        EOFError: EOF when reading a line
        '''
        ```

    -   Interactive mode
        -   사용자에게 입력을 받거나 어떠한 피드백에 대해 interactive하게 움직이려면 해당 모드를 사용하여야한다.
        -   `docker run`을 실행할때 -i flag를 이용하여 사용할 수 있다.

-   Image & Container 삭제하기

    -   `docker rm [container id or name]`을 입력하면 container를 삭제할 수 있음
        -   그러나 현재 container가 실행중이면 삭제가 안되어 `docker stop`을 진행 후 삭제하여야 한다.
    -   `docker rmi [image id]`를 실행하면 해당 image를 삭제할 수 있음
        -   그러나 사용중인 container가 있다면 삭제가 안됨(중지되어 있어도 존재한다면 동일)
    -   `docker run` 진행 시 --rm flag를 사용하여 container를 실행하면 해당 container가 중지될 때 자동으로 삭제됨

        (`docker run -p [외부 노출 포트]:[내부 컨테이너 포트] -d --rm [Image Id]`)

-   Container에서 Container로 복사하기

    -   `docker cp [복사하고 싶은 파일] [container id or name][:경로]`
        -   Ex1) `docker cp dummy/. abc:/test`
            -   Local의 dummy 폴더 내의 모든 내용을 abc container의 test 폴더로 복사
        -   Ex2) `docker cp abc:/test dummy`
            -   abc container의 test 폴더를 local의 dummy안으로 복사

-   Container와 Imagee에 name, tag 지정하기

    -   `docker run`을 진행 시 --name flag를 사용하면 container name을 지정할 수 있음

        -   `docker run -p [외부 노출 포트]:[내부 컨테이너 포트] -d --rm -- name [name] [Image Id]`

    -   `docker build`를 진행 시 -t flag를 이용하면 image에 name과 tag를 지정할 수 있음

        -   `docker build -t [name]:[tag] .`

-   Docker hub에 image push

    -   일단 docker hub에 가입하여야함
    -   `docker push [repository]:[tag name]`을 통해 push 할 수 있음
        -   repository와 local의 image name이 같아야함

-   Docker hub에서 image pull
    -   `docker pull [repository]:[tag name]`을 통해 pull 할 수 있음

### Section 3 데이터 관리 및 볼륨으로 작업하기

-   데이터 카테고리/다양한 종류의 데이터 이해하기

    -   Docker image는 읽기 전용이라 데이터의 수정이 필요하면 `docker build`를 다시 진행해야함
        -   모든 데이터가 docker container 내부 snapshot data로 저장되기 떄문에 container가 삭제되면 데이터가 모두 손실됨(단순 stop/start는 괜찮음)
        -   그래서 Docker container에서 volume라는 기능으로 읽기와 쓰기 모두 가능하게 설계가 되어 volume에 동적인 데이터를 저장할 수 있음
            -   이때는 docker container가 삭제되었다가 다시 실행되어도 data가 남아있음

-   data-volumes

    -   feedback을 작성하고 저장하는 간단한 앱을 구성해보겠음
    -   Dockerfile을 아래와 같이 구성

        ```dockerfile
        FROM node

        WORKDIR /app

        COPY package.json .

        RUN npm install

        COPY . .

        EXPOSE 80

        CMD [ "node", "server.js" ]
        ```

    -   이후 `docker build`, `docker run` 진행

        ```shell
        docker build -t feedback-node .
        docker run -p 3000:80 -d --name feedback-app --rm feedback-node
        ```

    -   현재 Dockerfile 설정으로는 `docker stop`을 진행 시 container가 삭제되어 저장된 feedback이 모두 손실됨

        -   --rm flag를 제외하고 `docker run`을 재진행
        -   이후 `docker stop/start feedback-app`을 진행하면 저장된 feedback이 남아있음

    -   Volume
        -   잘 생각해보면 copy와 정말 비슷하다는 느낌을 주나 copy는 container내부에 volumes는 container 외부에 저장하는 것
    -   Dockerfile을 아래와 같이 수정

        ```dockerfile
        FROM node

        WORKDIR /app

        COPY package.json .

        RUN npm install

        COPY . .

        EXPOSE 80

        VOLUME [ "/app/feedback" ] # app/feedback을 volume에 맵핑

        CMD [ "node", "server.js" ]
        ```

    -   이후 `docker build`, `docker run` 진행

        ```shell
        docker build -t feedback-node:volumes .
        docker run -p 3000:80 -d --name feedback-app --rm feedback-node:volumes
        ```

    -   그러나 feedback을 save하고 `docker stop` 후 `docker run`을 다시 진행 시 저장된 feedback이 조회가 안됨
        -   그 이유는 Dockerfile을 VOLUME으로 설정하는것은 익명 volume으로써 docker container가 삭제될떄 같이 삭제됨
        -   그래서 명명된 volume으로 재설정을 하여 docker container가 삭제되어도 volume이 삭제되지 않도록 설정이 필요함
    -   Dockerfile을 이전과 동일하게 다시 변경

        ```dockerfile
        FROM node

        WORKDIR /app

        COPY package.json .

        RUN npm install

        COPY . .

        EXPOSE 80

        CMD [ "node", "server.js" ]
        ```

    -   `docker build`를 재진행

        ```shell
        docker build -t feedback-node:volumes .
        ```

    -   이후 `docker run`을 진행할 때 -v flag를 이용하여 명명된 volume을 생성

        ```shell
        docker run -d -p 3000:80 --name feedback-app --rm -v feedback:/app/feedback feedback-node:volumes # -v [명명]:[경로]
        ```

    -   위와 같이 진행하면 `docker stop`을 진행하면서 docker container가 삭제되어도 volume은 남아있음(docker volume ls로 확인가능)
    -   그러면 이제 `docker run`을 진행할때 동일한 volume을 -v flag로 설정해준다면 이전에 생성한 volume과 연결이 되어 저장된 data가 모두 남아있음을 확인할 수 있음

-   Bind Mount

    -   Bind Mount는 volume과 거의 유사한 기능을 보이나 Volume은 영구적이고 편집이 불가능하나 Bind Mount는 영구적이고 편집이 가능한 데이터를 저장할때 사용함
    -   주로 개발과정에서 사용이 됨
    -   `docker run`을 진행 할때 volume과 동일하게 -v flag를 이용하여 bind mount를 사용할 수 있음

        ```shell
        macOS / Linux : docker run -d -p 3000:80 --name feedback-app --rm -v feedback:/app/feedback -v $(pwd):/app:ro -v /app/node_modules feedback-node:volumes
        Windows: docker run -d -p 3000:80 --name feedback-app --rm -v feedback:/app/feedback -v "%cd%":/app:ro -v /app/node_modules feedback-node:volumes
        # 현재 이것을 실행하는 폴더를 /app에 read only로 mount하고 /app/node_modules는 내부 컨테이너에서 사용하라는 뜻
        # 익명 볼륨이 우선순위가 가장 높으며 그 다음 구체적인 경로를 가지는 볼륨이 우선순위가 높음
        # 이렇게하면 수정하는 내용을 실시간으로 볼 수 있음
        ```

-   중간 정리
    |명령어|이름|설명|우선순위|
    |------|---|---|---|
    |`docker run -v path`|익명 볼륨|컨테이너가 삭제되면 같이 삭제됨|1|
    |`docker run -v name:path`|명명 볼륨|컨테이너가 삭제되어도 동일한 볼륨의 이름으로 실행되면 이전 데이터가 유지됨|상세경로가 자세한 순|
    |`docker run -v path:path`|바인드 마운트|로컬 파일을 실시간으로 마운트함|-|

-   Docker Volume 관리하기

    -   `docker volume ls` : 생성된 docker volume의 리스트를 확인(바인드 마운트는 docker에서 관리를 하지않아 리스트에 뜨지 않음)
    -   `docker volume create [name]` : docker volume을 생성함
    -   `docker volume inspect [name]` : docker volume의 상세 내용을 확인할 수 있음
    -   `docker volume rm [name]` : docker volume을 삭제
    -   `docker volume prune` : 사용하지 않는 volume을 삭제

-   Dockerignore

    -   `.dockerignore`에 `docker build`를 할 때 제외할 파일을 `.gitignore`처럼 작성하면 제외 후 build됨

-   환경변수(ENVironment)

    -   Dockerfile에 env에 대한 내용을 작성하고 `docker run`을 진행할때 값을 전달해줌

        -   Dockerfile 예시

            ```dockerfile
            FROM node

            WORKDIR /app

            COPY package.json .

            RUN npm install

            COPY . .

            ENV PORT 80 # PORT라는 ENV에 기본값을 80으로 설정

            EXPOSE $PORT # ENV에 PORT로 설정해준 값을 사용

            CMD [ "node", "server.js" ]
            ```

            1. PORT의 기본값을 Dockerfile에 작성하면서 그대로 사용하는 방법이며 이전과 동일하게 `docker run`을 진행하면 된다
            2. `docker run`을 진행할때 --env(-e) flag를 이용하여 입력이 가능
                ```shell
                docker run -d -p 3000:80 -e PORT=80 --name feedback-app --rm -v feedback:/app/feedback -v $(pwd):/app:ro -v /app/node_modules feedback-node:volumes
                ```
            3. .env를 생성하고 `docker run`을 진행 시 --env-file flag로 해당 파일을 참고
                ```env
                PORT=8000 # .env에서 PORT의 값을 설정
                ```
                ```shell
                docker run -d -p 3000:80 --env-file ./.env --name feedback-app --rm -v feedback:/app/feedback -v $(pwd):/app:ro -v /app/node_modules feedback-node:volumes
                ```

-   빌드인수(ARGuments)

    -   ARG는 code에서 가져다 쓰는건 불가능함
    -   ENV와 유사하나 ENV는 container를 생성할 때 값을 전달해주는 반면에 ARG는 image를 build할때 값을 전달해줌
    -   결국 ARG는 image를 생성하면 값이 잠기는 개념임
    -   예시

        ```dockerfile
        FROM node

        WORKDIR /app

        COPY package.json .

        RUN npm install

        COPY . .

        ARG DEFAULT_PORT=80

        ENV PORT ${DEFAULT_PORT}

        EXPOSE ${PORT}

        CMD [ "node", "server.js" ]
        ```

        ```shell
        docker build feedback-node:dev --build-arg DEFAULT_PORT=8000 .
        ```

### Section 4 네트워킹: (교차) 컨테이너 통신

-   이제 여기서부터가 가장 배우고 싶던 네트워크 통신에 대한 내용이다.
    -   Front단과 Back단을 Docker에서 어떻게 통신하는지 배워보고 싶었다.
-   총 3개의 case로 study를 할 것이다.

    -   case 1 : www 통신(웹 API와 같은)
        -   기본적으로 container 내부에서 웹 API와 같은 http/https으로의 통신은 가능하다.
    -   case 2 : Container에서 로컬 호스트 머신으로의 통신(MongoDB와 유사한 외부 데이터베이스를 로컬 호스트에서 동작시킬때와 같은)
        -   Docker 내부의 localhost를 사용하려면 host.docker.internal을 사용하면 된다.(localhost:3000 = host.docker.internal:3000)
    -   case 3 : Container 간 통신

        -   Docker hub에 있는 공식 mongo image를 이용하여 새로운 컨테이너를 구축
            ```shell
            docker run -d --name mongodb mongo
            docker ps
            ''' 아래와 같이 mongo db가 docker container 27017 port에서 동작하고있음
            CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS       NAMES
            dac36adb4ab7   mongo     "docker-entrypoint.s…"   18 seconds ago   Up 17 seconds   27017/tcp   mongodb
            '''
            docker inspect mongodb
            ''' inspect를 실행하여 mongodb의 ip주소를 확인할 수 있음(현재는 172.17.0.2)
            "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "ce22d8a98c7d68ba28b39758a8b86825b2b460d59e03d35de543bfe97433bf52",
            "SandboxKey": "/var/run/docker/netns/ce22d8a98c7d",
            "Ports": {
                "27017/tcp": null
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "bfcaaa359f3184bba112f0896398e817f5587cbd1e2b761f597660f65087dbba",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:02",
                    "NetworkID": "871b5d200e5131b59ccb449219adb6b1f030104784c98f3d7d68baff61f0780c",
                    "EndpointID": "bfcaaa359f3184bba112f0896398e817f5587cbd1e2b761f597660f65087dbba",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DriverOpts": null,
                    "DNSNames": null
                }
            }
            '''
            ```
        -   이후 위에서 확인된 mongodb 주소를 가지고 앱에서 mongodb에 연결할 떄 주소를 아래와 같이 변경
            ```javascript
            mongoose.connect(
                "mongodb://172.17.0.2:27017/swfavorites",
                { useNewUrlParser: true },
                (err) => {
                    if (err) {
                        console.log(err);
                    } else {
                        app.listen(3000);
                    }
                }
            );
            ```
        -   `docker build`, `docker run`을 진행

            ```shell
            docker build -t favorites-node .
            docker run --name favorites -d --rm -p 3000:3000 favorites-node
            ```

        -   그러나 위와 같이 진행하면 매번 container를 inspect해서 ip address를 찾고 하는 과정이 반복된다.
        -   이를 개선하기 위해 network 기능이 있다.
            ```shell
            docker network create favorites-net # favorites-net이라는 network를 생성
            docker run -d --name mongodb --network favorites-net mongo # favorites-net network에 mongo를 연결
            ```
            ```javascript
            mongoose.connect(
                "mongodb://mongodb:27017/swfavorites",
                { useNewUrlParser: true },
                (err) => {
                    if (err) {
                        console.log(err);
                    } else {
                        app.listen(3000);
                    }
                }
            );
            // container간 동일한 network에 있다면 ip address대신 container 이름으로 접근이 가능하다.
            // 현재는 mongo를 mongodb라는 container에 실행시켜놨기 때문에 위와 같은 예시가 가능하다.
            ```
            ```shell
            docker build -t favorites-node .
            docker run --name favorites --network favorites-net -d --rm -p 3000:3000 favorites-node # --network flag로 mongodb와 동일한 network에 연결
            ```

### Section 5 Docker로 다중 컨테이너 애플리케이션 구축하기

-   보통의 앱들은 Database, Frontend, Backend 등 다양한 컨테이너가 필요한 경우가 많음
-   그 상황을 위해 Multi container를 구성해보려고 한다.
-   이 데모앱은 총 3개의 Building Blocks로 이루어져있다.
    -   Database
        -   MongoDB 이용
    -   Backend
        -   NodeJS 기반 REST API
    -   Frontend
        -   React 기반의 SPA 웹

### Section 6 Docker Compose: 우아한 다중 컨테이너 오케스트레이션

### Section 7 유틸리티 컨테이너로 작업하기 & 컨테이너에서 명령 실행하기

### Section 8 더 복잡한 설정: Laravel & PHP 도커화

### Section 9 Docker 컨테이너 배포하기

### Section 10 요약

### Section 11 Kubernetes 시작하기

### Section 12 실전 Kubernetes - 핵심 개념 자세히 알아보기

### Section 13 Kubeernetes로 데이터 & 볼륨 관리하기

### Section 14 Kubernetes 네트워킹

### Section 15 Kubernetes - 배포(AWS EKS)

### Section 16 마무리 정리 & 다음 단계
