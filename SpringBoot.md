# Docker - Spring boot

## Docker Compose
* https://seosh817.tistory.com/387
* [복잡한 로컬 구성](https://dev.gmarket.com/72)

### MaraiDB
* [Docker - MaraiDB](https://velog.io/@jkjan/Docker-MySQL-%EC%9B%90%EA%B2%A9-%EC%A0%91%EC%86%8D)
* 볼륨 생성: docker desktop > Volumes > Create > MariaDB_Volumn

docker-compose/MariaDB/docker-compose.yml
```yml
version: "3.9"

volumes:
  MariaDB_Volumn:
    external: true
    name: MariaDB_Volumn

services:
  db:
    image: mariadb
    container_name: con_mariadb
    ports: 
      - "43306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=dockerDB
      - MYSQL_USER=dockerUSER
      - MYSQL_PASSWORD=dockerPASSWORD
      - TZ=Asia/Seoul
    command: 
      - --default-authentication-plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --skip-character-set-client-handshake
    volumes:
      - MariaDB_Volumn:/var/lib/mysql
```

```sh
cd docker-compose/MariaDB
docker-compose up -d
mysql -u root -p -P 43306
docker exec -it con_mariadb /bin/bash
```

## Spring boot
* https://umanking.github.io/2021/07/11/spring-boot-docker-starter

Dockerfile
```Dockerfile
FROM adoptopenjdk/openjdk11
CMD ["./mvnw", "clean", "package"]
# CMD ["./gradle", "clean", "build", "bootJar"]
ARG JAR_FILE_PATH=target/*.jar
COPY ${JAR_FILE_PATH} app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
EXPOSE 8080 
```
* `./mvnw` 실행시 `JAVA_HOME` 필요
* EXPOSE = 8080 포트 오픈

```sh
# 로컬 이미지 생성
docker build -t image-spring-test:0.0.1 ./
## -t = 이름:버전 형식

# 컨테이너 생성
docker create --name con_spring -P image-spring-test:0.0.1
## -P = 컨테이너가 실행될때 EXPOSE에서 열린 포트를 랜덤 호스트 포트와 연결한다.
```
