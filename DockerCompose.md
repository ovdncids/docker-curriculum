# Docker Compose

## MaraiDB
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
