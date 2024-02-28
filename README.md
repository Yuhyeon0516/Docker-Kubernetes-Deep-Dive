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

-   first-demo-starting-setup

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
        -   Container에서 가져다 쓸 수 있는 작은 코드? 도구? 조각? 이라고 생각하면 될듯

-   사전 빌드된 Image의 사용 & 실행

    -   https://hub.docker.com 에는 사전에 빌드된 Docker에서 관리하고있는 Image가 있음
    -   `docker run [image]` 명령어를 진행하면 local에 해당 image가 없을 시 library에서 가져온다고 알람이 발생하면서 다운로드되어 현재 실행중인 container에 추가되면서 실행됨(`docker ps -a` 명령어로 실행 중임을 확인할 수 있음)
    -   `docker run -it [image]` 명령어를 진행하면 앞에 받아두어 container에서 실행 중인 image로 실행됨

-   nodejs-app-starting

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

            COPY . /app # 파일 전체를 덮어쓰기하면서 파일이 변경되었다고 docker에서 인지함

            RUN npm install # 파일이 변경되었기에 npm install도 진행함

            EXPOSE 80

            CMD ["node", "server.js"]
            ```

            </td>
            <td>

            ```docker
            FROM node

            WORKDIR /app

            COPY package.json /app # package.json만 먼저 복사해서 변경 사항이 있는지 체크함

            RUN npm install # 위의 package.json이 변경사항이 없다면 npm install도 cache가 사용되어 시간이 단축됨

            COPY . /app # 이후 나머지 파일을 전부 복사함

            EXPOSE 80

            CMD ["node", "server.js"]
            ```

            </td>
            </tr>
            </table>

### Section 3

### Section 4

### Section 5

### Section 6

### Section 7

### Section 8

### Section 9

### Section 10

### Section 11

### Section 12

### Section 13

### Section 14

### Section 15
