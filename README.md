# Docker 맛보기

### $ cd practice-docker/frontend
### $ docker run -it node
- 로컬에서 node 이미지가 있는지 확인 후 없으면 node 이미지를 다운 받아서 실행
- 그 뒤 다시 명령어를 입력하면 이미지가 있기 때문에 바로 실행됨

### $ docker images
- 도커 이미지들 확인 명령어

### $ docker ps
- 임의의 이름을 가진 도커 컨테이너를 확인할 수 있음

### $ docker exec -it {names} bash
- 해당 컨테이너로 들어가서 bash shell을 실행시킴
- 가상의 리눅스 환경으로 들어옴

### https://www.yalco.kr/36_docker/
: 도커 명령어 정리

### $ docker ps -a
: 중지된 컨테이너도 확인 가능 (언제 작동이 멈췄는지도 확인 가능)

### $ docker stop $(docker ps -aq)
### $ docker system prune -a
: 도커 모든 컨테이너 중지, 사용되지 않는 모든 도커 요소 삭제


# DockerFile
- 나만의 이미지를 만들기 위한 설계도
(http-server를 이용하고 싶을 때, node 이미지는 http-server는 없는 상태이기 때문에 이를 컨테이너로실행할 때 마다 http-server를 다운받아줘야 함. node와 http-server가 모두 깔린 상태가 이미지로 있으면 컨테이너로 서비스를 실행하기 훨씬 수월해짐)
- 원래의 node 이미지를 개조, 튜닝해서 만드는 거라고 생각하면 됨

<pre><code>
FROM node:12.18.4

# 이미지 생성 과정에서 실행할 명령어
RUN npm install -g http-server 

# 이미지 내에서 명령어를 실행할(현 위치로 잡을) 디렉토리 설정
WORKDIR /home/node/app

# 컨테이너 실행시 실행할 명령어
CMD ["http-server", "-p", "8080", "./public"]

# 이미지 생성 명령어 (현 파일과 같은 디렉토리에서)
# docker build -t {이미지명} .

# 컨테이너 생성 & 실행 명령어
# docker run --name {컨테이너명} -v $(pwd):/home/node/app -p 8080:8080 {이미지명}
</code></pre>

## RUN 명령어
: 이미지를 생성하는 과정에서 실행되는 것 (이미지에서 컨테이너를 실행하는 시점에서는 이미 http-server가 설치되어 있음)

## CMD 명령어
: 이미지로부터 컨테이너가 만들어져 가동될 때 기본적으로 바로 실행되는 명령어

# dockerfile 이미지 생성 방법
- 해당 dockerfile이 있는 폴더로 디렉토리 이동
### $ docker build -t frontend-img .
- (-t 뒤에 원하는 이미지명을 적고 Dockerfile로의 상대경로를 적어줌)

### docker images
: 위에 만든 frontend-img 라는 이미지가 생성된 걸 볼 수 있음 (node + http-server)

### docker run --name frontend-con -v $(pwd):/home/node/app -p 8080:8080 frontend-img
- 생성될 컨테이너의 이름을 --name 뒤에 지정
-v : 볼륨 (컨테이너와 특정 폴더를 공유하는 것, 해당 폴더 위치에 존재하는 내용들이 frontend-con 컨테이너의 /home/node/app 폴더에 들어감)
-p : 컨테이너의 포트와 연결 (CMD로 미리 8080을 설정해줬음)
(같은 이미지의 다른 컨테이너를 실행할 때는 $(pwd):/home/node/app -p 8080:8081)

## COPY 명령어
- RUN처럼 이미지를 생성하는 과정에서 미리 해당 이미지 안에 특정 파일을 넣어두는 것
- volume은 CMD 처럼 컨테이너가 생성되어 실행될 때 그 내부의 폴더를 외부의 것과 연결

### docker run --name frontend-con -v $(pwd):/home/node/app -p 8080:8080 -d frontend-img
: -d 명령을 붙이면 데몬으로 실행


## 해당 작업들을 하나하나 처리하는 것은 굉장히 비생산적인 일이고 각각의 컨테이너를 서비스적으로 연결하는 것도 쉽지 않음
## docker-compose.yml 파일을 통해 하나의 파일에서 모든 설정을 입력하고
### docker-compose up 
: 명령어를 입력하면 한번에 연결되어 사용이 가능해짐