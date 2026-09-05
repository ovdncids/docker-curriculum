# DockerCompose
* https://seosh817.tistory.com/387
* [복잡한 로컬 구성](https://dev.gmarket.com/72)

## Networks
docker-composes/{프로젝트}/docker-compose.yml
```yml
networks:
  a_network:
    driver: bridge
  b_network:
    driver: bridge

services:
  gitlab:
    container_name: gitlab_service
    networks:
      - a_network

  nexus:
    container_name: nexus_service
    networks:
      - b_network

  jenkins:
    networks:
      - a_network
      - b_network
```
* `docker-compose.yml` 안에서는 기본적으로 `networks 설정 없이` `container_name` 또는 `서비스 이름`으로 서로 접속 가능하다.
* `networks 설정`은 `네트워크 격리`하고 싶을때 사용한다.

```sh
cd docker-composes/{프로젝트}

# docker-compose.yml 바탕으로 컨테이너 생성
docker-compose up -d
```

## GitLab@19.3
```yml
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    hostname: gitlab.local
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://{호스트_IP}:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2222
    ports:
      - "8929:8929"
      - "2222:22"
    volumes:
      - gitlab_config:/etc/gitlab
      - gitlab_logs:/var/log/gitlab
      - gitlab_data:/var/opt/gitlab
    shm_size: "256m"
volumes:
  gitlab_config:
  gitlab_logs:
  gitlab_data:
```
```sh
# 24시간 후에 root 초기 패스워드 파일이 삭제 됨
cat /etc/gitlab/initial_root_password

# 강제 패스워드 변경
docker exec -it gitlab gitlab-rake "gitlab:password:reset"
## Enter username: root

# http://{호스트 IP}:8929
## 사용자 계정 생성
## root 계정으로 로그인 후 사용자 계정 활성화
## 사용자 계정으로 프로젝트 생성

# 프로젝트 폴더
git remote add origin ssh://git@{호스트_IP}:2222/{사용자}/{프로젝트_명}.git
git push --set-upstream origin main
## The authenticity of host '[{호스트_IP}]:2222 ([{호스트_IP}]:2222)' can't be established.
## ED25519 key fingerprint is: SHA256:xxx...

ssh-keygen -t ed25519 -C "{사용자_이메일}"
cat ~/.ssh/id_ed25519.pub
## SHA256:xxx... 복사
## http://{호스트 IP}:8929 > 사용자 > 오른쪽 위 프로필 > Edit profile > Access > SSH Keys > 붙여 넣기

ssh -T -p 2222 git@{호스트_IP}
## Welcome to GitLab, @사용자

git push --set-upstream origin main
```

## Nexus@3.96.0-09
```yml
services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    restart: unless-stopped
    ports:
      - "8081:8081"
    volumes:
      - nexus-data:/nexus-data
volumes:
  nexus-data:
```

## Python@3.12.10, uv@0.12.7, FastAPI@0.141.1, uvicorn@0.52.4
* https://github.com/ovdncids/python-curriculum/blob/master/PythonInstall.md

docker-composes/{프로젝트}/{Python 프로젝트}/Dockerfile
```Dockerfile
# Build
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim AS builder
WORKDIR /app
COPY pyproject.toml uv.lock README.md ./
COPY src ./src
RUN uv build

# Runtime
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim
WORKDIR /app
COPY --from=builder /app/dist/*.whl /tmp/
RUN uv pip install --system /tmp/*.whl
CMD ["uvicorn", "fastapi_project.main:app", "--host", "0.0.0.0", "--port", "8000"]
```
```sh
docker build -t fastapi-project-image .
docker run --name fastapi-project -p 8000:8000 fastapi-project-image
```

### Python with Nexus
* pyproject.toml
```toml
[[tool.uv.index]]
name = "nexus"
url = "http://localhost:8081/repository/pypi-proxy/simple"
default = true
```
* Nexus에서 `pypi-proxy`를 생성하면 `pypi-proxy/simple`도 같이 생성해 준다. (simple은 pypi만의 관행 이다.)

## Jenkins@2.258.3
docker-composes/{프로젝트}/Dockerfile
```Dockerfile
FROM jenkins/jenkins:lts
USER root
RUN apt-get update \
    && apt-get install -y docker.io \
    && rm -rf /var/lib/apt/lists/*
RUN usermod -aG root jenkins
USER jenkins
```
```yml
services:
  jenkins:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock

# `./`는 호스트의 경로를 쓴다. (docker-composes/{프로젝트}/jenkins_home) 따라서 아래는 불필요 하다.
# volumes:
#   jenkins_home:
# `/var/run/docker.sock:/var/run/docker.sock` 호스트의 /var/run/docker.sock를 컨테이너가 그대로 쓴다. (권한: `RUN usermod -aG root jenkins`)
```
```sh
# Jenkins가 호스트의 docker를 사용할 수 있는지 확인
docker exec jenkins docker version
```
* http://localhost:8080
* 플러그인 없이 설치
* Jenkins 관리 > Plugins > Available plugins > `Folders Plugin`, `Pipeline`

Pipeline Script
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'ps -p $$'
                git branch: 'main', url: 'http://gitlab:8929/{사용자}/{프로젝트_명}.git'
            }
        }
    }
}
```
* `./jenkins_home/workspace/{폴더}/{파이프_명}`에 빌드된 파일 존재
