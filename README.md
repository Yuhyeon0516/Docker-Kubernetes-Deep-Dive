# Docker-Kubernetes-Deep-Dive

-   Udemy의 https://www.udemy.com/course/docker-kubernetes-2022 강의를 참고하여 공부하려고 한다.
-   편안하게 공부한것을 적는 일기장임

-   Docker란

    -   Docker는 Container를 생성하고 관리하는 도구
        -   Container란 표준화된 소프트웨어 유닛

-   Docker 설정

    -   https://docs.docker.com/desktop/ 에서 OS별 요구사항을 충족한다면 Docker Desktop을 다운로드
    -   OS별 요구사항을 충족하지 못한다면 Docker Toolbox를 다운로드
    -   Linux는 Docker Engine을 설치하는 과정이 있음

-   first-demo-starting-setup

    -   .mjs 코드는 강의에서 제공해주는것을 그대로 사용하였음.
    -   Dockerfile 구성

        ```docker
        FROM node:14 # Node.js를 14버전 사용

        WORKDIR /app # 실행 Directory를 설정

        COPY package.json . # package.json을 docker container root에 카피

        RUN npm install # npm install을 진행

        COPY . . # root에 있는 모든 내용을 container root에 copy

        EXPOSE 3000 # 3000번 포트에서 실행

        CMD [ "node", "app.mjs" ] # node app.mjs 명령어를 실행
        ```
