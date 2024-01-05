# Docker Compose
* https://seosh817.tistory.com/387
* [복잡한 로컬 구성](https://dev.gmarket.com/72)

## MaraiDB
* [Docker - MaraiDB](https://velog.io/@jkjan/Docker-MySQL-%EC%9B%90%EA%B2%A9-%EC%A0%91%EC%86%8D)
* 볼륨 생성: docker desktop > Volumes > Create > mariadb_volume

docker-composes/{프로젝트}/docker-compose.yml
```yml
version: "3.9"

volumes:
  mariadb_volume:
    external: true
    name: mariadb_volume

services:
  mariadb_service:
    image: mariadb
    container_name: mariadb_container
    ports:
      - "43306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=docker_database
      - MYSQL_USER=docker_user
      - MYSQL_PASSWORD=docker_password
      - TZ=Asia/Seoul
    command:
      - --default-authentication-plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --skip-character-set-client-handshake
    volumes:
      - mariadb_volume:/var/lib/mysql
```

```sh
cd docker-composes/{프로젝트}
docker-compose up -d
mysql -u root -p -P 43306
docker exec -it mariadb_container /bin/bash
```

## Spring Boot
* https://umanking.github.io/2021/07/11/spring-boot-docker-starter
* [CMD vs ENTRYPOINT](https://velog.io/@dachae/Docker-5-Dockerfile-%EC%9C%A0%EC%9D%98%EC%82%AC%ED%95%AD)
* [Docker for Spring Boot](https://spring.io/guides/topicals/spring-boot-docker)

Dockerfile
```Dockerfile
FROM adoptopenjdk/openjdk11

# Maven으로 .jar 파일을 로컬에서 우선 생성해야함
# ./mvnw clean package
ARG JAR_FILE_PATH=target/*-SNAPSHOT.jar

# Gradle으로 .jar 파일을 로컬에서 우선 생성해야함
# ./gradlew clean build
ARG JAR_FILE_PATH=build/libs/*-SNAPSHOT.jar

COPY ${JAR_FILE_PATH} app.jar
ENTRYPOINT ["java", "-jar", "app.jar", "--spring.profiles.active=${PROFILE}"]
EXPOSE 8080
```
* `./mvnw` 또는 `./gradlew` 실행시 윈도우의 경우 `JAVA_HOME` 필요
* EXPOSE = 컨테이너의 8080 포트 오픈

```sh
# 도커 데스크톱에 로컬 이미지 생성
docker build --no-cache -t spring_image:0.0.1 ./
## 생성한 Dockerfile 파일을 읽어서 이미지를 만든다.
## --no-cache = 빌드를 캐시에 넣지 않는다.
## -t = 이름:버전 형식

# 컨테이너 생성
docker create --name spring_container -P -e PROFILE=production spring_image:0.0.1
## -P = 컨테이너가 실행될때 EXPOSE에서 열린 포트를 랜덤 호스트 포트와 연결한다.
## -e = 환경변수를 전달한다.
```

### Error response from daemon: pull access denied for spring_image
* `docker-compose.yml` 파일에서 `spring_image:0.0.1` 버전이 도커 데스크톱과 같은지 확인

## MariaDB와 Spring Boot 연결
```sh
docker network ls
```

docker-composes/{프로젝트}/docker-compose.yml
```yml
networks:
  maria_network:
    driver: bridge

services:
  mariadb_service:
    ...
    networks:
      - maria_network

  spring_service:
    image: spring_image:0.0.1
    container_name: spring_container
    ports:
      - "48080:8080"
    environment:
      - PROFILE=production
    networks:
      - maria_network
```
* networks 설정을 `bridge`로 맞추면 컨테이너 끼리 `IP` 대신 `서비스 이름`으로 연결 가능하다.

src/main/resources/application-production.properties
```properties
spring.datasource.url=jdbc:mysql://mariadb_service:3306/docker_database
```
* IP를 컴포즈의 서비스 이름으로 맞춘다.

```sh
cd docker-composes/{프로젝트}
docker-compose up -d
```
