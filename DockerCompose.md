# Docker Compose
* https://seosh817.tistory.com/387
* [복잡한 로컬 구성](https://dev.gmarket.com/72)

## React - 이미지 생성
* [React Compose](https://night-knight.tistory.com/entry/Docker-1-React%EB%A1%9C-Docker-%EC%8B%A4%ED%96%89%EC%8B%9C%EC%BC%9C%EB%B3%B4%EA%B8%B0-Docker-compose)

{React 프로젝트}/Dockerfile
```Dockerfile
FROM node:16.20.2-alpine
WORKDIR /react-server
COPY . ./
RUN npm install
EXPOSE 3000
```
* WORKDIR = 이미지의 작업 경로
* COPY = {React 프로젝트}에서 모든 파일(.)을 이미지의 작업 경로(./)로 복사
* RUN = 이미지에서 실행하는 명령
* EXPOSE = 컨테이너의 8080 포트 오픈

```sh
# 도커 데스크톱에 로컬 이미지 생성
docker build --no-cache -t react_image:0.0.1 ./
## 생성한 Dockerfile 파일을 읽어서 이미지를 만든다.
## --no-cache = 현 빌드에 이전 빌드의 캐시를 사용하지 않는다. (현 빌드 후에는 현 빌드에 대한 캐시를 생성한다.)
## -t = 이름:버전 형식

# 컨테이너 생성
docker create --name react_container -P react_image:0.0.1
## -P = 컨테이너가 실행될때 EXPOSE에서 열린 포트를 랜덤 호스트 포트와 연결한다.

# 컨테이너 삭제 후 재생성
docker create --name react_container -P react_image:0.0.1 npm start
## 이미지 뒤에 붙는 npm = [COMMAND], start = [ARG...]
```
* 소스 파일, .git등 불필요한 파일이 서버에 복사 되고 이미지가 `1 GB`정도로 크므로 `NginX` 방식을 사용한다.
* [빌드 캐시 삭제](https://github.com/ovdncids/docker-curriculum/blob/master/ETC.md#build-cache-%EC%82%AD%EC%A0%9C)

## NGINX - 이미지 생성
* [NGINX React 연동](https://stackoverflow.com/questions/45598779/react-router-browserrouter-leads-to-404-not-found-nginx-error-when-going-to)
```sh
cp Dockerfile React-Dockerfile
npm run build
```

{React 프로젝트}/Dockerfile
```Dockerfile
FROM nginx
WORKDIR /usr/share/nginx/html
COPY ./build ./
EXPOSE 80
```
```sh
# 도커 데스크톱에 로컬 이미지 생성
docker build --no-cache -t nginx_react_image:0.0.1 ./

# 컨테이너 생성
docker run -d --name nginx_react_container -p 40080:80 nginx_react_image:0.0.1
```
* http://localhost:40080
* http://localhost:40080/users (`404 Not Found` 발생한다.)

{React 프로젝트}/Dockerfile.nginx.conf
```conf
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```
{React 프로젝트}/Dockerfile
```Dockerfile
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

<details><summary>Dockerfile로 default.conf 파일 수정하는 방법</summary>

  {React 프로젝트}/Dockerfile
  ```Dockerfile
  RUN echo '\
  server {\n\
      listen 80;\n\
      location / {\n\
          root /usr/share/nginx/html;\n\
          index index.html;\n\
          try_files $uri $uri/ /index.html;\n\
      }\n\
  }'\
  > /etc/nginx/conf.d/default.conf
  
  EXPOSE 80
  ```
  * `>> /etc/nginx/conf.d/default.conf` 이렇게 수정하면 이전 파일에 추가하고
  * `> /etc/nginx/conf.d/default.conf` 이렇게 하면 이전 파일을 덮어씌운다.
</details>

## Next.js - 이미지 생성
* [모범적인 Dockerfile](https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile)
* [적용 사례](https://velog.io/@jadenkim5179/Next.js-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-docker-%EB%B0%B0%ED%8F%AC-%EC%9D%B4%EB%AF%B8%EC%A7%80-%ED%81%AC%EA%B8%B0-%EC%A4%84%EC%9D%B4%EA%B8%B0)

{Next.js 프로젝트}/Dockerfile
```Dockerfile
FROM node:16.20.2-alpine AS base

# 의존성 설치 (700 MB)
FROM base AS dependencies
RUN apk add --no-cache libc6-compat
WORKDIR /app-dependencies
COPY package.json package-lock.json ./
RUN npm ci

# 빌더 (2 GB)
FROM base AS builder
WORKDIR /app-builder
COPY --from=dependencies /app-dependencies/node_modules ./node_modules
COPY . .
RUN npm run build

# 러너 (150 MB)
FROM base AS runner
WORKDIR /app-runner
ENV HOSTNAME "0.0.0.0"
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
RUN mkdir .next
RUN chown nextjs:nodejs .next
COPY --from=builder --chown=nextjs:nodejs /app-builder/public ./public
COPY --from=builder --chown=nextjs:nodejs /app-builder/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app-builder/.next/static ./.next/static

USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```
* `npm install`은 `npm outdated` 기준 `Wanted`를 설치한다.
* `npm ci`는 `package-lock.json` 기준으로 `npm install` 한다. (npm outdated - Current)

```sh
# 도커 데스크톱에 로컬 이미지 생성
docker build --no-cache -t nextjs_image:0.0.1 ./

# 컨테이너 생성
docker run --name nextjs_container -p 43000:3000 nextjs_image:0.0.1
```
* 단계 별로 `이미지 용량`과 `/app-?` 확인 한다.

next.config.js
```js
const nextConfig = {
  output: 'standalone'
}
```
* [Next.js는 standalone 모드로 빌드 해야 한다.](https://nextjs.org/docs/advanced-features/output-file-tracing)
* `CMD`는 docker create --name nextjs_container -p 43000:3000 nextjs_image:0.0.1 `node server.js`와 같다.

### 환경 변수 전달
* [Next.js - 환경 설정](https://github.com/ovdncids/react-curriculum/blob/master/Next-js.md#%ED%99%98%EA%B2%BD-%EC%84%A4%EC%A0%95)
```Dockerfile
ARG ENV
RUN npm run build:${ENV}
```
```sh
docker build --build-arg ENV=production -t nextjs_image:0.0.1 ./
```

## MariaDB - 컴포즈 생성
* [Docker - MariaDB](https://velog.io/@jkjan/Docker-MySQL-%EC%9B%90%EA%B2%A9-%EC%A0%91%EC%86%8D)
* [MariaDB - 권한, 백업](https://github.com/ovdncids/mysql-curriculum/blob/master/GrantDump.md)
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

# 컨테이너 생성
docker-compose up -d

mysql -u root -p -P 43306
## 기본 패스워드 0000
docker exec -it mariadb_container /bin/bash
```

## MariaDB와 Next.js 연결
```sh
docker network ls
```

docker-composes/{프로젝트}/docker-compose.yml
```yml
networks:
  compose_network:
    driver: bridge

services:
  mariadb_service:
    ...
    networks:
      - compose_network

  nextjs_service:
    image: nextjs_image:0.0.1
    container_name: nextjs_container
    ports:
      - "43000:3000"
    environment:
      - ENV=production
    networks:
      - compose_network
```
* networks 설정을 `bridge`로 맞추면 컨테이너 끼리 `IP` 대신 컴포즈의 `컨테이너 이름` 또는 `서비스 이름`으로 연결 가능하다.

.env.production
```env
MYSQL_HOST=mariadb_container
MYSQL_DATABASE=docker_database
MYSQL_USER=docker_user
MYSQL_PASSWORD=docker_password
```
* `HOST`를 컴포즈의 `컨테이너 이름` 또는 `서비스 이름`으로 맞춘다.
* [MySQL2 연결](https://github.com/ovdncids/react-curriculum/blob/master/Next-js-13.4.md#mysql2-%EC%97%B0%EA%B2%B0)
```sh
# `libraries/mysql2Pool.js` 생성, `page.js` 또는 `route.js`에서 `await mysql2Pool()` 호출
# `npm run build`에서 오류가 발생하면 `try catch` 사용
## (빌드에서는 `mariadb_container` 네트워크를 찾을 수 없으므로 당연히 오류 발생한다. docker build 동일)

docker build --no-cache -t nextjs_image:0.0.1 ./
```

```sh
cd docker-composes/{프로젝트}

# 기존의 nextjs_container, mariadb_container 우선 삭제 후 실행
docker-compose up -d
```

## Spring Boot - 이미지 생성
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

```sh
# 도커 데스크톱에 로컬 이미지 생성
docker build --no-cache -t spring_image:0.0.1 ./

# 컨테이너 생성
docker create --name spring_container -P -e PROFILE=production spring_image:0.0.1
## -e = 환경변수를 전달한다.
```

### Error response from daemon: pull access denied for spring_image
* `docker-compose.yml` 파일에서 `spring_image:0.0.1` 버전이 도커 데스크톱과 같은지 확인

## MariaDB와 Spring Boot 연결
docker-composes/{프로젝트}/docker-compose.yml
```yml
networks:
  compose_network:
    driver: bridge

services:
  mariadb_service:
    ...
    networks:
      - compose_network

  spring_service:
    image: spring_image:0.0.1
    container_name: spring_container
    ports:
      - "48080:8080"
    environment:
      - PROFILE=production
    networks:
      - compose_network
```
* networks 설정을 `bridge`로 맞추면 컨테이너 끼리 `IP` 대신 컴포즈의 `컨테이너 이름` 또는 `서비스 이름`으로 연결 가능하다.

src/main/resources/application-production.properties
```properties
spring.datasource.url=jdbc:mysql://mariadb_service:3306/docker_database
```
* `HOST`를 컴포즈의 `컨테이너 이름` 또는 `서비스 이름`으로 맞춘다.

```sh
cd docker-composes/{프로젝트}

# 기존의 spring_container, mariadb_container 우선 삭제 후 실행
docker-compose up -d
```
