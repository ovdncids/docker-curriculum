# ETC

## Mac Docker 경로
* https://yooloo.tistory.com/188
```sh
~/Library/Containers/com.docker.docker/Data/vms/0
~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw (기본이 64 GB)
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

### Docker.raw 크기 줄이기
```sh
Docker Desktop > 도구 > Resources > Virtual disk limit > 40 GB
## 8 GB로 변경하여도 34 GB정도로 작아진다. (크기를 줄이면 기존 이미지들이 삭제 될 수 있다.)
## Docker.raw 파일 크기와 실제 디스크를 차지하는 크기가 다를 수 있다.
```

### Build cache 삭제
* 도커는 빌드 할때마다 cache를 생성한다. 빌드가 많으면 하드에 용량 많이 없어지므로 틈틈히 지우자.
```sh
docker system prune -a
# 또는
docker builder prune -f
```
