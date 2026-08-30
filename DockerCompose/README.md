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

## GitLab
```yml
networks:
  compose_network:
    driver: bridge

services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    hostname: gitlab.local
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://{호스트 IP}:8929'
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
```
