# Docker - Spring boot

## Docker Compose
* https://seosh817.tistory.com/387
* [복잡한 로컬 구성](https://dev.gmarket.com/72)

### MaraiDB
* [Docker - MaraiDB](https://velog.io/@jkjan/Docker-MySQL-%EC%9B%90%EA%B2%A9-%EC%A0%91%EC%86%8D)
* 볼륨 생성: docker desktop > Volumes > Create > mariadb_volume

docker-composes/mariadb/docker-compose.yml
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
cd docker-composes/mariadb
docker-compose up -d
mysql -u root -p -P 43306
docker exec -it mariadb_container /bin/bash
```

## Spring Boot
* https://umanking.github.io/2021/07/11/spring-boot-docker-starter

Dockerfile
```Dockerfile
FROM adoptopenjdk/openjdk11
CMD ["./mvnw", "clean", "package"]
# CMD ["./gradle", "clean", "build", "bootJar"]
ARG JAR_FILE_PATH=target/*.jar
COPY ${JAR_FILE_PATH} app.jar
ENTRYPOINT ["java", "-jar", "app.jar", "--spring.profiles.active=production"]
EXPOSE 8080
```
* `./mvnw` 실행시 `JAVA_HOME` 필요
* EXPOSE = 8080 포트 오픈

```sh
# 로컬 이미지 생성
docker build --no-cache -t spring_image:0.0.1 ./
## --no-cache = 빌드를 캐시에 넣지 않는다.
## -t = 이름:버전 형식

# 컨테이너 생성
docker create --name spring_container -P spring_image:0.0.1
## -P = 컨테이너가 실행될때 EXPOSE에서 열린 포트를 랜덤 호스트 포트와 연결한다.
```

```sh
docker network ls
```

### Error response from daemon: pull access denied for spring_image
* `docker-compose.yml` 파일에서 `spring_image:0.0.1` 버전이 도커 데스크톱과 같은지 확인
