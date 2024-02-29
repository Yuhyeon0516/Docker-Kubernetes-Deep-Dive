# Docker-Kubernetes-Deep-Dive

-   Udemy의 https://www.udemy.com/course/docker-kubernetes-2022 강의를 참고하여 공부하려고 한다.
-   편안하게 공부한것을 적는 일기장임
-   내부의 코드들은 대부분 강의에서 제공됨

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

        ```docker
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

        ```docker
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

            ```docker
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

            ```docker
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

        ```docker
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

### Section 4 네트워킹: (교차) 컨테이너 통신

### Section 5 Docker로 다중 컨테이너 애플리케이션 구축하기

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
