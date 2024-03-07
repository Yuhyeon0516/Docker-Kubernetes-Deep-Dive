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

-   먼저 MongoDB를 도커화 해보겠음

    ```shell
    docker run --name mongodb --rm -d -p 27017:27017 mongo # 현재는 backend를 도커화 안하였기때문에 localhost:27017로 mongodb를 노출시키기 위해 -p flag를 이용함
    cd backend
    npm install
    node app.js # CONNECTED TO MONGODB log 확인
    ```

-   Backend를 도커화 해보겠음

    -   먼저 Dockerfile을 구성

        ```dockerfile
        FROM node

        WORKDIR /app

        COPY package.json .

        RUN npm install

        COPY . .

        EXPOSE 80

        CMD [ "node", "app.js" ]
        ```

    -   그리고 Backend도 이제 docker container에서 실행할 것이기 때문에 app.js의 mongodb 주소를 localhost에서 host.docker.internal로 변경

        ```javascript
        mongoose.connect(
            //"mongodb://localhost:27017/course-goals", <- 이전
            "mongodb://host.docker.internal:27017/course-goals",
            {
                useNewUrlParser: true,
                useUnifiedTopology: true,
            },
            (err) => {
                if (err) {
                    console.error("FAILED TO CONNECT TO MONGODB");
                    console.error(err);
                } else {
                    console.log("CONNECTED TO MONGODB");
                    app.listen(80);
                }
            }
        );
        ```

    -   이후 `docker build`, `docker run` 진행

        ```shell
        cd backend
        docker build -t goals-node .
        docker run --name goals-backend --rm -d goals-node
        ```

-   Frontend를 도커화

    -   먼저 Dockerfile을 구성

        ```dockerfile
        FROM node

        WORKDIR /app

        COPY package.json .

        RUN npm install

        COPY . .

        EXPOSE 3000

        CMD [ "npm", "run", "start" ]
        ```

    -   이후 `docker build`, `docker run` 진행

        ```shell
        docker build -t goals-react .
        docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react # React 앱의 경우 대화형 상호작용을 위해 -it flag를 추가하여야한다
        ```

-   효율적인 container 간 통신을 위한 docker network 추가

    -   먼저 3개의 container를 연결할 network를 생성

        ```shell
        docker network create goals-net
        ```

    -   이후 mongodb를 --network flag를 이용하여 `docker run` 진행

        ```shell
        docker run --name mongodb --rm -d --network goals-net mongo
        ```

    -   Backend도 동일하게 --network flag를 이용하여 `docker run` 진행해야하나 그전에 mongodb 주소를 변경 후 `docker build`, `docker run` 진행

        ```javascript
        mongoose.connect(
            // "mongodb://host.docker.internal:27017/course-goals",
            "mongodb://mongodb:27017/course-goals",
            {
                useNewUrlParser: true,
                useUnifiedTopology: true,
            },
            (err) => {
                if (err) {
                    console.error("FAILED TO CONNECT TO MONGODB");
                    console.error(err);
                } else {
                    console.log("CONNECTED TO MONGODB");
                    app.listen(80);
                }
            }
        );
        ```

        ```shell
        cd backend
        docker build -t goals-node .
        docker run --name goals-backend --rm -d --network goals-net goals-node
        ```

    -   마지막으로 frontend도 --network flag를 이용하여 `docker run` 진행해야하나 backend와 동일하게 주소를 변경해야한다.(localhost -> goals-backend)

        ```shell
        docker build -t goals-react .
        docker run --name goals-frontend --network goals-net --rm -p 3000:3000 -it goals-react
        ```

        -   그러나 위와 같이 하면 goals-backend를 찾을 수 없다고 에러가 발생함
        -   왜냐면 backend는 container에서 직접 node 명령어로 동작시켰기 때문에 network에 관련된 사항을 알고있으나 React의 javascript의 경우 browser에서 직접 컨트롤하기 때문에 network 관련사항을 알 수 없음
        -   이와 같은 상황을 해결하려면 backend부터 다시 진행하여야한다.

            -   Backend에서 `docker run`을 진행할때 -p flag를 이용하여 80번 port를 외부에 열어둔다.

                ```shell
                docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node
                ```

            -   이후 frontend에서 goals-backend로 변경하였던 주소를 다시 localhost로 변경

                -   backend가 localhost:80에서 실행중이기 때문에 localhost로 접근이 가능.
                -   이후 `docker build`, `docker run`을 진행

                    ```shell
                    docker build -t goals-react .
                    docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react # localhost:80으로 접근할 예정이라 --network flag는 이제 필요 없어짐
                    ```

-   Volume을 이용한 MongoDB 데이터 지속성 추가

    -   Docker hub의 mongo docs를 보면 data store는 docker container 내부의 /data/db에 있다고 설명되어있음(https://hub.docker.com/_/mongo)

        ![where to stor data](https://github.com/Yuhyeon0516/Docker-Kubernetes-Deep-Dive/assets/120432007/97efe17d-1abd-4499-8421-4090622f8b42)

    -   그래서 mongodb의 데이터에 지속성을 추가하려면 -v flag를 이용하여 :/data/db를 유지시켜주면된다.

        ```shell
        docker run --name mongodb -v data:/data/db --rm -d --network goals-net mongo
        ```

-   MongoDB Database 보안

    -   Docker hub의 mongo docs를 보면 환경변수로 username과 password를 설정할 수 있게 되어있음
    -   이는 보안에 중점을 둔 옵션이며 실제 제품에서는 필수적으로 적용해야함
    -   MongoDB를 `docker run`할 때 -e flag로 username과 password를 전달해줌

        ```shell
        docker run --name mongodb -v data:/data/db --rm -d --network goals-net -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=secret mongo
        ```

    -   이후 backend에서도 주소를 입력 시 `[username:password@]`양식으로 전달해줌(추후 제품에서는 당연히 env로 관리해야하나 지금은 test 환경이니 주소에 바로 적음)

        ```javascript
        mongoose.connect(
            // "mongodb://mongodb:27017/course-goals", <- 이전
            "mongodb://admin:secret@mongodb:27017/course-goals?authSource=admin",
            {
                useNewUrlParser: true,
                useUnifiedTopology: true,
            },
            (err) => {
                if (err) {
                    console.error("FAILED TO CONNECT TO MONGODB");
                    console.error(err);
                } else {
                    console.log("CONNECTED TO MONGODB");
                    app.listen(80);
                }
            }
        );
        ```

        ```shell
        docker build -t goals-node .
        docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node
        ```

-   Backend container의 volume, bind mount, polishing 설정

    -   현재 backend의 code를 보면 logs 폴더에 access log가 남도록 설정되어있는데 이 부분도 logs volume으로 설정.

        ```shell
        docker run --name goals-backend -v logs:/app/logs --rm -d -p 80:80 --network goals-net goals-node
        ```

    -   이후 code의 live update를 확인하기 위해 bind mount를 설정

        ```shell
        docker run --name goals-backend -v $(pwd):/app:ro -v logs:/app/logs --rm -d -p 80:80 --network goals-net goals-node
        ```

    -   Container 내부에서 의존성으로 인한 충돌을 방지하기 위해 node_modules도 volume으로 설정해줌

        -   아래와 같이 설정하면 bind mount때 사용된 node_modules가 container 내부의 /app/node_modules를 덮어씌우지 않음(우선순위로 인해)

            ```shell
            docker run --name goals-backend -v $(pwd):/app:ro -v logs:/app/logs -v /app/node_modules --rm -d -p 80:80 --network goals-net goals-node
            ```

    -   이후 nodemon으로 code의 live update를 감지하도록 변경

        -   package.json의 devDependencies와 start script 추가

            ```json
            {
                "name": "backend",
                "version": "1.0.0",
                "description": "",
                "main": "index.js",
                "scripts": {
                    "test": "echo \"Error: no test specified\" && exit 1",
                    "start": "nodemon app.js"
                },
                "author": "Maximilian Schwarzmüller / Academind GmbH",
                "license": "ISC",
                "dependencies": {
                    "body-parser": "^1.19.0",
                    "express": "^4.17.1",
                    "mongoose": "^5.10.3",
                    "morgan": "^1.10.0"
                },
                "devDependencies": {
                    "nodemon": "^2.0.4"
                }
            }
            ```

        -   이후 Dockerfile에서 마지막 CMD 구문을 npm run start로 변경

            ```dockerfile
            FROM node

            WORKDIR /app

            COPY package.json .

            RUN npm install

            COPY . .

            EXPOSE 80

            CMD [ "npm", "run", "start" ]
            ```

        -   Dockerfile과 package.json이 변경되었으니 `docker build`, `docker run` 재진행

            ```shell
            docker build -t goals-node .
            docker run --name goals-backend -v $(pwd):/app:ro -v logs:/app/logs -v /app/node_modules --rm -d -p 80:80 --network goals-net goals-node
            ```

    -   마지막으로 기존에 backend에 mongodb username과 password를 hard coding 해두었는데 그것을 env로 변경

        -   Dockerfile에 ENV를 추가

            ```dockerfile
            FROM node

            WORKDIR /app

            COPY package.json .

            RUN npm install

            COPY . .

            EXPOSE 80

            ENV MONGODB_USERNAME=root
            ENV MONGODB_PASSWORD=root

            CMD [ "npm", "run", "start" ]
            ```

        -   MongoDB 주소에 env를 참고하도록 설정

            ```javascript
            mongoose.connect(
                `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`,
                {
                    useNewUrlParser: true,
                    useUnifiedTopology: true,
                },
                (err) => {
                    if (err) {
                        console.error("FAILED TO CONNECT TO MONGODB");
                        console.error(err);
                    } else {
                        console.log("CONNECTED TO MONGODB");
                        app.listen(80);
                    }
                }
            );
            ```

        -   Dockerfile이 변경되었으니 `docker build`를 재진행하고 `docker run` 진행 시 -e flag로 환경변수를 전달

            ```shell
            docker build -t goals-node .
            docker run --name goals-backend -v $(pwd):/app:ro -v logs:/app/logs -v /app/node_modules -e MONGODB_USERNAME=admin -e MONGODB_PASSWORD=secret --rm -d -p 80:80 --network goals-net goals-node
            ```

-   Frontend(React) container에 대한 code live update 추가

    -   다른 설정은 필요없고 bind mount를 이용하여 src 폴더 변경을 감지하도록 추가

        ```shell
        docker run -v $(pwd)/src:/app/src:ro --name goals-frontend --rm -p 3000:3000 -it goals-react
        ```

### Section 6 Docker Compose: 우아한 다중 컨테이너 오케스트레이션

-   사실 앞서 설정한 network를 이용한 docker 다중 container는 commend를 입력하기 힘들고 반복적인 작업이 많다는 단점이 있음
-   이를 극복하기 위해 docker compose 기능을 배워보려고함
-   Docker compose란?

    -   지금까지 진행한 `docker build`, `docker run`의 반복적인 작업을 자동화하여 터미널이 아닌 파일로 설정하고 실행함

-   Docker compose를 설정하는 방법

    -   Root 경로에 `docker-compose.yaml` 파일을 생성한다.
    -   이후 `docker-compose.yaml`에 아래와 같은 내용을 작성

        ```yaml
        version: "3.8"
        # https://docs.docker.com/compose/compose-file/compose-versioning/
        # 위 링크에서 docker compose version에 대한 정보를 알 수 있음

        services: # services는 각각의 service의 집합체 즉 최상위 container이다.
            mongodb: # services 하위에 mongodb라는 service가 있다.
                image: mongo # mongo image를 사용
                volumes: # volume을 설정
                    - data:/data/db
                # 환경 설정에는 아래와 같이 두개의 방법이 있음
                # 그러나 실제 프로덕트 환경에서 env를 노출시키면 안되니 env_file로 관리하는게 좋을듯
                # environment:
                #     MONGO_INITDB_ROOT_USERNAME: admin
                #     MONGO_INITDB_ROOT_PASSWORD: secret
                env_file:
                    - ./env/mongo.env

            backend: # services 하위에 backend라는 service가 있다.
                # local image를 이용하는 경우 기존에는 build 후 run을 진행하였음
                # docker compose에서는 위 build 과정까지 자동으로 도와줌
                # build가 필요한 경로를 전달해주면 거기서 Dockerfile을 찾아 build를 진행함
                # build:
                #     context: ./backend
                #     dockerfile: Dockerfile
                build: ./backend
                # "외부포트:내부포트"로 설정
                ports:
                    - "80:80"
                volumes:
                    - logs:/app/logs
                    - ./backend:/app
                    - /app/node_modules
                env_file:
                    - ./env/backend.env
                # mongodb가 먼저 구축이 되어야 backend를 동작시킬 수 있기때문에 의존성을 추가
                depends_on:
                    - mongodb

            frontend: # services 하위에 frontend라는 service가 있다.
                build: ./frontend
                ports:
                    - "3000:3000"
                volumes:
                    - ./frontend/src:/app/src
                # 기존에 -it flag를 이용하여 docker run을 진행하였고
                # 그것은 아래와 같은 옵션으로 정의할 수 있음
                stdin_open: true
                tty: true
                depends_on:
                    - backend

        # 위 services에 사용되는 명명된 볼륨을 전부 나열해야 사용할 수 있음
        volumes:
            data:
            logs:
        ```

    -   `docker-compose up`을 실행하면 `docker-compose.yaml`을 확인하여 필요한 volume 및 network를 생성하고 container를 실행시킴
        -   `docker-compose up -d`을 진행하면 dettach mode로 실행됨
            -   그래서 각각의 service 설정에서 dettach mode는 설정이 필요없음
        -   `docker-compose up --build`를 진행하면 image를 강제로 re-build 할 수 있음
    -   `docker-compose down`을 실행하면 실행되고있는 container가 중지되고 사용하던 network, container 모두 삭제됨(volume은 유지됨)

        -   그래서 --rm flag의 설정이 필요없음

    -   -network flag 설정은 docker compose 안에 있는 모든 service는 동일한 네트워크에 소속되도록 자동으로 설정됨

### Section 7 유틸리티 컨테이너로 작업하기 & 컨테이너에서 명령 실행하기

-   Utility Container란?

    -   기존의 Application Container는 환경과 앱이 같이 들어있었으나, Utility Container는 환경에 대한것만 담아두는 container라고 생각하면 된다.

-   Utility Container를 왜 사용하는가?

    -   Host machine 즉 내 PC에 특정 환경을 설치할 필요가 없이 docker를 이용하여 환경 구축이 가능함
    -   추가로 Laravel이나 PHP와 같은 프레임워크에서 특정 환경을 요구하는 경우가 있어서 이때 PC에 셋업하는 과정이 복잡하고 용량도 크기에 Utility Container를 사용하여 구축 후 사용

-   Utility Container 구축 실습

    -   Dockerfile을 아래와 같이 작성

        ```dockerfile
        FROM node:21-alpine

        WORKDIR /app
        ```

    -   이후 bind mount를 이용하여 내 PC의 node 환경이 아닌 docker의 node 환경을 이용하여 npm init이 가능해진다.

        ```shell
        docker run -it -v $(pwd):/app node-util npm init
        ```

-   ENTRYPOINT 활용

    -   Dockerfile에서 ENTRYPOINT를 활용하면 아래와 같이 사용이 가능하다

        ```dockerfile
        FROM node:21-alpine

        WORKDIR /app

        ENTRYPOINT [ "npm" ]
        ```

        ```shell
        # npm 명령어를 생략 가능
        docker run -it -v $(pwd):/app node-util init
        docker run -it -v $(pwd):/app node-util install express
        ```

    -   CMD와 차이점은 CMD는 입력된 명령어로 기존 명령어가 덮히는 방면에 ENTRYPOINT는 입력된 명령어 앞에 진입점처럼 사용됨

-   Docker compose 활용

    -   docker-compose.yaml을 아래와 같이 설정하여 docker compose에서도 활용이 가능함

        ```yaml
        version: "3.8"

        services:
            npm:
                build: ./
                stdin_open: true
                tty: true
                volumes:
                    - ./:/app
        ```

        ```shell
        # docker-compose run (service name) (script)
        docker-compose run npm init
        ```

### Section 8 더 복잡한 설정: Laravel & PHP 도커화

-   Laravel & PHP의 특별한점이 무엇이 있을까?

    -   PHP가 셋업이 생각보다 엄청 까다롭다.
    -   심지어 Request 요청에 따라 처리해줄 웹 서버가 필요하고 그를 구성해줘야함(생각보다 더 복잡함)
    -   이번 실습은 아래와 같은 구성으로 이루어짐

        ![target](https://github.com/Yuhyeon0516/Docker-Kubernetes-Deep-Dive/assets/120432007/8f9354af-2f39-4ab6-93a7-ebe2f8ba27ed)

    -   먼저 웹 서버용 Nginx container를 추가

        ```yaml
        version: "3.8"

        services:
            server:
                image: "nginx:stable-alpine"
                ports:
                    - "8000:80"
                volumes:
                    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
        ```

        ```nginx
        server {
            listen 80;
            index index.php index.html;
            server_name localhost;
            root /var/www/html/public;
            location / {
                try_files $uri $uri/ /index.php?$query_string;
            }
            location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass php:3000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
            }
        }
        ```

    -   이후 PHP container 추가

        ```yaml
        version: "3.8"

        services:
            server:
                image: "nginx:stable-alpine"
                ports:
                    - "8000:80"
                volumes:
                    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
            php:
                build:
                    context: ./dockerfiles
                    dockerfile: php.dockerfile
                volumes:
                    - ./src:/var/www/html:delegated
                ports:
                    - "3000:9000
        ```

        ```dockerfile
        FROM php:8.2-fpm-alpine

        WORKDIR /var/www/html

        COPY src .

        RUN docker-php-ext-install pdo pdo_mysql
        ```

    -   MySQL container 추가

        ```yaml
        version: "3.8"

        services:
            server:
                image: "nginx:stable-alpine"
                ports:
                    - "8000:80"
                volumes:
                    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
            php:
                build:
                    context: ./dockerfiles
                    dockerfile: php.dockerfile
                volumes:
                    - ./src:/var/www/html:delegated
                ports:
                    - "3000:9000"
            mysql:
                image: mysql
                env_file:
                    - ./env/mysql.env
        ```

    -   Compose utility container 추가

        ```yaml
        version: "3.8"

        services:
            server:
                image: "nginx:stable-alpine"
                ports:
                    - "8000:80"
                volumes:
                    - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
            php:
                build:
                    context: ./dockerfiles
                    dockerfile: php.dockerfile
                volumes:
                    - ./src:/var/www/html:delegated
                ports:
                    - "3000:9000"
            mysql:
                image: mysql
                env_file:
                    - ./env/mysql.env
            composer:
                build:
                    context: ./dockerfiles
                    dockerfile: composer.dockerfile
                volumes:
                    - ./src:/var/www/html
        ```

        ```dockerfile
        FROM composer:latest

        WORKDIR /var/www/html

        ENTRYPOINT [ "composer", "--ignore-platform-reqs" ]
        ```

    -   이후 composer를 이용하여 laravel 앱을 생성

        ```shell
        docker-compose run --rm composer create-project --prefer-dist laravel/laravel .
        ```

    -   server(nginx), php(laravel), mysql을 실행

        ```shell
        docker-compose up -d server php mysql server
        ```

    -   다른 utility container 추가

        -   artisan container 추가(artisan이란 laravel에서 database test를 진행할 수 있는 tool)

            ```yaml
            version: "3.8"

            services:
                server:
                    image: "nginx:stable-alpine"
                    ports:
                        - "8000:80"
                    volumes:
                        - ./src:/var/www/html
                        - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
                    depends_on:
                        - php
                        - mysql
                php:
                    build:
                        context: .
                        dockerfile: dockerfiles/php.dockerfile
                    volumes:
                        - ./src:/var/www/html:delegated
                    ports:
                        - "3000:9000"
                mysql:
                    image: mysql
                    env_file:
                        - ./env/mysql.env
                composer:
                    build:
                        context: ./dockerfiles
                        dockerfile: composer.dockerfile
                    volumes:
                        - ./src:/var/www/html
                artisan:
                    build:
                        context: .
                        dockerfile: dockerfiles/php.dockerfile
                    volumes:
                        - ./src:/var/www/html
                    entrypoint: ["php", "/var/www/html/artisan"]
            ```

            -   이후 database test를 위해 migrate 명령어 진행

                ```shell
                docker-compose run --rm artisan migrate

                '''출력결과
                Migration table created successfully.
                Migrating: 2014_10_12_000000_create_users_table
                Migrated:  2014_10_12_000000_create_users_table (14.00ms)
                Migrating: 2014_10_12_100000_create_password_resets_table
                Migrated:  2014_10_12_100000_create_password_resets_table (11.06ms)
                Migrating: 2019_08_19_000000_create_failed_jobs_table
                Migrated:  2019_08_19_000000_create_failed_jobs_table (12.59ms)
                '''
                ```

        -   node container 추가

            ```yaml
            version: "3.8"

            services:
                server:
                    image: "nginx:stable-alpine"
                    ports:
                        - "8000:80"
                    volumes:
                        - ./src:/var/www/html
                        - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
                    depends_on:
                        - php
                        - mysql
                php:
                    build:
                        context: .
                        dockerfile: dockerfiles/php.dockerfile
                    volumes:
                        - ./src:/var/www/html:delegated
                    ports:
                        - "3000:9000"
                mysql:
                    image: mysql
                    env_file:
                        - ./env/mysql.env
                composer:
                    build:
                        context: ./dockerfiles
                        dockerfile: composer.dockerfile
                    volumes:
                        - ./src:/var/www/html
                artisan:
                    build:
                        context: .
                        dockerfile: dockerfiles/php.dockerfile
                    volumes:
                        - ./src:/var/www/html
                    entrypoint: ["php", "/var/www/html/artisan"]
                npm:
                    image: node
                    working_dir: /var/www/html
                    entrypoint: ["npm"]
                    volumes:
                        - ./src:/var/www/html
            ```

### Section 9 Docker 컨테이너 배포하기

-   대망의 배포 단계이다
-   배포에는 가장 대표적인 AWS를 사용할 것이다
    -   당연히 약간의 비용이 발생할 수 있으니 주의.
-   배포 프로세스는 간단히 아래와 같다

    1. Docker hub에 image를 push
    2. Remote Machine(AWS, Azure, Google Cloud와 같은 Provider들)에서 image를 pull
    3. SSH와 같은 기타 설정을 한후 www로 접속할 수 있게 제공

-   Bind mount는 배포환경에서 변경점이 생김

    -   배포환경에서는 실시간 업데이트와 같은 유연성이 필요하지 않기 떄문에 Bind mount를 COPY로 변경하여야한다.
        -   Docker에서는 외부에 의존하지 않고 image로 모든것을 얻을 수 있다는 사상임
        -   그래서 지금까지 bind mount는 최대한 명령어 선에서 해결하려 하였음(나중에 배포환경에서는 명령어 안쓰면되니까)

-   첫 실습

    -   AWS EC2를 이용하여 실습예정

        -   AWS EC2 환경구성을 해야함

            1. Amazon Machine Instance(AMI)를 선택하여야 하는데 이번에는 기본적인 Amazon Linux를 사용해보려한다.
            2. 당연히 인스턴스 유형(Instance Type)은 Free tier를 제공해주는 t2.micro로 구성하려한다.
            3. 이후 키페어를 생성하고 .pem을 받아둔다.(나는 이전에 쓰던 키페어가 있어서 이것을 재활용)
            4. VPC 설정한다(아마 기본으로 생성되어있을거고 서비스별 방화벽에 대한 설정이 별도로 필요하다면 보안 그룹도 생성하여 사용하면 됨)
            5. 설정을 다 했다면 인스턴스 시작을 누르면 약 2~3분 후 인스턴스가 시작된다.
            6. SSH를 설정한다
                - SSH란 Secure Shell로 네트워크 상 다른 컴퓨터의 쉘을 사용할 수 있게 해 주는 프로그램 혹은 그 프로토콜을 의미함
                - 자세한 내용은 아래 링크에 잘 정리되어있으니 참고하고 설정에 들어가겠다
                  (https://heekangpark.github.io/ssh/01-introduction)
                - 앞에 받아둔 .pem 키페어에 권한을 먼저 부여한다.
                    ```shell
                    chmod 400 [키페어]
                    ```
                - 이후 ssh 명령어를 이용해 인스턴스 shell로 접속한다
                    ```shell
                    ssh -i [키페어] [인스턴스 주소]
                    ```
                - 이렇게하면 SSH를 통해 인스턴스 shell에서 명령을 수행할 수 있음
            7. 이제 docker를 설치한다
                ```shell
                sudo yum update -y
                sudo yum -y install docker
                sudo service docker start
                sudo usermod -a -G docker ec2-user
                # 이후 SSH Logout후 재접속
                sudo systemctl enable docker
                # docker version 명령어를 이용하여 docker 명령어가 잘 동작하는지 확인
                docker version
                ```

    -   Docker를 이용하요 배포하는 방법은 크게 2가지 있다.

        1. Source code를 push/pull하여 docker run과 build를 진행하는법
        2. Docker image를 push/pull하여 docker run만 진행하는법

        -   이번에는 2번을 사용해보려고한다 1번은 Git등 설정을 좀 더 해줘야함

    -   먼저 제공된 code를 가지고 local machine에서 `docker build`를 진행

        ```shell
        docker build -t single-node-app-deploy .
        ```

    -   이후 `docker run`을 진행하면 localhost에서 html을 확인할 수 있고 이러한 html을 보여주는 환경을 remote machine으로 옮기는 작업을 할 것임

        ```shell
        docker run -d --rm --name single-node-app -p 80:80 single-node-app-deploy
        ```

    -   Docker hub에 repository를 생성

        -   repository 이름과 local에 있는 dockerimage 이름이 같아야하기 때문에 처음부터 맞추는걸 추천
        -   다르면 추후에 `docker tag [현재 image name] [repository name]`을 진행하면 됨

    -   이후 `docker push [repository name]`을 진행하면 docker hub에 push됨
        -   당연히 `docker login`을 통해 권한이 있는 상태여야함
    -   이제 docker hub에 image가 있으니 SSH로 다시 넘어가서 해당 image를 pull해서 run하면됨

        -   `sudo docker run -d --rm -p 80:80 [repository name]`하면 repository는 public으로 되어있을 것이기 때문에 pull한 후 dettach mode로 run 할것임
        -   그러나 여기서 local 환경이 arm64가 아닌 PC를 사용한다면 아래와 같은 에러를 마주하게 될 것임
            ```shell
            WARNING: The requested image's platform (linux/arm64/v8) does not match the detected host platform (linux/amd64/v3) and no specific platform was requested
            exec /usr/local/bin/docker-entrypoint.sh: exec format error
            ```
        -   나를 예시로 들자면 현재 M2 arm64 맥북을 사용하고 있고, Amazon Linux는 amd64 기반의 환경인 상태에서 ARM에서 작성한 docker image를 AMD 기반의 환경에서 실행했기 때문에 발생하게 되는 문제이다.
        -   이러한 문제를 해결하려면 multi platform을 지원하게 build를 진행하여야한다. 방법은 아래와 같다.

            ```shell
            # buildx에 desktop-linux 환경을 사용
            docker buildx use desktop-linux
            # buildx build를 통해 linux/amd64, linux/arm64를 지원하는 image를 build하고 repository로 push
            docker buildx build --platform=linux/amd64,linux/arm64 -t [repository name] --push .
            ```

        -   이후 다시 SSH에서 `sudo docker run`을 진행하면 완료

    -   인스턴스의 IPv4주소를 찾아 웹에서 접속하면 아까 local에서 보았던 웹이 동일하게 나타날 것임
        -   만약 접속이 안된다면 인스턴스의 인바운드 보안규칙에 HTTP 80번 Port가 있는지 확인해봐야함

-   컨테이너/이미지 관리 & 업데이트

    -   만약 source code가 변경되면 업데이트를 어떻게?

        1. 당연히 docker image를 다시 build하면서 docker hub에 올리고
        2. 그 image를 AWS SSH에서 다시 `docker run`을 해주면된다.

            ```shell
            docker stop [container name]
            docker pull [repository name]
            docker run [repository name]
            ```

### Section 10 요약

### Section 11 Kubernetes 시작하기

### Section 12 실전 Kubernetes - 핵심 개념 자세히 알아보기

### Section 13 Kubeernetes로 데이터 & 볼륨 관리하기

### Section 14 Kubernetes 네트워킹

### Section 15 Kubernetes - 배포(AWS EKS)

### Section 16 마무리 정리 & 다음 단계
