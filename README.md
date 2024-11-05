# Docker (24.0.7)
* 가상 컨테이너를 활용하여 원하는 서비스(웹서버, DB등)를 최소화 하여 설치 할 수 있다.
* https://www.youtube.com/watch?v=Ps8HDIAyPD0&list=PLuHgQVnccGMDeMJsGq2O-55Ymtx0IdKWf

# Docker hub
* Docker Image들의 모음
* https://hub.docker.com

## 설치
https://docs.docker.com/get-docker

## 아파치 Image 설치
```sh
# 설치된 image 보기
docker images

# 아파치 image 설치
docker pull httpd

# 아파치 image 삭제
docker rmi httpd
```

## 설치된 아파치 Image를 바탕으로 컨테이너 생성
```sh
# 모든 컨테이너 보기
docker ps -a
# 실행중인 컨테이너만 보기
docker ps

# 아파치 컨테이너를 httpd1 이름으로 생성
docker run --name httpd1 httpd
## 컨테이너 생성과 동시에 현재 터미널에서 컨테이너 실행
docker run --name httpd1 -d httpd
## 컨테이너 생성 후 데몬으로 실행

# 컨테이너 log 보기
docker logs -f httpd1

# 컨테이너 httpd1 중지
docker stop httpd1

# 컨테이너 httpd1 실행
docker start httpd1

# 컨테이너 httpd1 삭제
docker rm httpd1
```

## 호스트(내 컴퓨터)와 연결 하며 컨테이너 생성
```sh
# 호스트와 컨테이너 사이에 포트 포워딩 하는 httpd2 컨테이너 생성
docker run --name httpd2 -p 8880:80 httpd

# 호스트와 컨테이너 사아에 포트 포워딩과 파일 연결 하는 httpd3 컨테이너 생성
docker run --name httpd3 -p 8880:80 -v {호스트 경로}:/usr/local/apache2/htdocs httpd
## run = create, start, attach 또는 exec 한번에 실행
## {호스트 경로} = 파일을 복사하지 않고, 심볼릭 링크처럼 공유 된다.
```

## 컨테이너에 터미널 연결
```sh
docker exec -it httpd3 /bin/sh
docker exec -it httpd3 /bin/bash
```

### 아파치 proxy
```sh
# 주석 유지
#Include conf/extra/httpd-vhosts.conf

# 주석 해제 또는 <VirtualHost *:80> 위에 선언
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
<VirtualHost *:80>
    <Location /api>
        ProxyPass http://{ip}:{port}/api
    </Location>
</VirtualHost>
```

## Mac M1 - Docker 초기화
* https://github.com/docker/for-mac/issues/6145
```sh
Docker 종료

# 지금 실행 중이지 않은 컨테이너 삭제
docker system prune -a
# 볼륨 삭제
docker volume prune

# brew uninstall docker

rm -rf ~/.docker
rm -rf ~/Library/Caches/com.docker.docker
rm -rf ~/Library/Group Containers/group.com.docker
rm -rf ~/Library/Containers/com.docker.docker
rm -rf ~/Library/Application\ Support/Docker\ Desktop

open --background -a Docker
# brew install --cask docker
## --cask는 GUI 기반 프로그램 설치
```

## Ubuntu 22.04.3
* https://sleepyeyes.tistory.com/67
* https://lucas-owner.tistory.com/61
```sh
# Ubuntu 이미지 설치
docker pull ubuntu:latest

# 컨테이너 생성
docker create -it --name ubuntu_container -p 33000:3000 -p 33306:3306 -p 38080:8080 ubuntu
## -it = 컨테이너 내부로 진입 (attach 가능, i = 상호 입출력, t = tty를 활성화하여 bash 쉘을 사용)
### -it 옵션을 설정 하지 않으면 `docker start ubuntu_container` 후에 자동 종료 된다.
## -d = 데몬 서비스 (detached 모드, exec으로 터미널 가능)
## -p = 포트 바인딩

# 컨테이너 실행
docker start ubuntu_container

# 컨테이너 Shell 접속 (root로 접속됨)
docker attach ubuntu_container
```

### Shell 접속
```sh
# 버전 보기
cat /etc/os-release
# 사용자 보기
cat /etc/passwd
# 현재 사용중인 SHELL 보기
echo $SHELL

# apt update를 하지 않으면 sudo, net-tools 등을 설치 할 수 없다.
apt update

# sudo 설치
apt install sudo
# 사용자 생성
sudo adduser [사용자]
# root 비밀번호 변경
sudo passwd root
# 생성한 사용자에게 sudo 권한 주기
sudo usermod -aG sudo [사용자]
# 해당 사용자로 로그인
su - [사용자]
## 이제 `docker exec -it -u [사용자] ubuntu_container /bin/bash` 접속도 가능하다.

# 네트워크 설치
sudo apt install net-tools
sudo apt install iputils-ping
ifconfig
ping [호스트 ip]

# MariaDB
sudo apt install mariadb-server
sudo service --status-all
sudo service mariadb status
sudo service mariadb start
sudo mysql -u root
```
* [MariaDB - 권한](https://github.com/ovdncids/mysql-curriculum/blob/master/GrantDump.md)
* [Raspberry Pi - Java](https://github.com/ovdncids/raspberrypi-curriculum#java)

## 컨테이너 생성시 포트를 설정하지 않은 경우
* https://stackoverflow.com/questions/19335444/how-do-i-assign-a-port-mapping-to-an-existing-docker-container
* [컨테이너 파일의 위치](https://yooloo.tistory.com/188)
* [chroot 설명](https://www.44bits.io/ko/post/change-root-directory-by-using-chroot)
```sh
# 윈도우 Docker 설정 파일 위치
C:\Users\[사용자]\AppData\Local\Docker\wsl\data\ext4.vhdx

docker info
## Docker Root Dir: /var/lib/docker
## 해당 경로를 ubuntu에서 접근해야 한다.
docker inspect ubuntu_container
## 설정 정보를 확인
docker run -v/:/data -it --name docker_ubuntu ubuntu /bin/bash
## -v = 볼륨, /:/data = `Docker Root Dir`에 접근 가능하게 해준다.

# `Docker Root Dir`에 접근
chroot /data

# 컨테이너 inspect 설정 경로 이동
cd /var/lib/docker/containers/[컨테이너 해시값]
mv hostconfig.json hostconfig.json.ori
cat hostconfig.json.ori
mv config.v2.json config.v2.json.ori
cat config.v2.json.ori

# hostconfig.json 파일 새로 생성 ("ExposedPorts" 부분에 포트 추가)
cat << EOF > hostconfig.json
{hostconfig.json.ori 파일을 수정해서 넣음}
EOF
# config.v2.json 파일 새로 생성 ("PortBindings" 부분에 포트 추가)
cat << EOF > config.v2.json
{config.v2.json.ori 파일을 수정해서 넣음}
EOF
```
