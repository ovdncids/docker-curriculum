# 안드로이드폰 서버 만들기
* https://www.youtube.com/watch?v=trjUbvSXfmc&t=12s0

# Termux@3.0.9 (Android 10 이상 필요)
```sh
# Termux 앱 설치 (Ubuntu 설치전에 가볍게 Terminal을 사용 할 수 있다. 제약이 큼)
설정 > 애플리케이션 > Termux > 배터리 > 제한 없음

# Android 10보다 낮을 경우 `F-Droid APK 설치` (https://f-droid.org)
# F-Droid > Termux 검색 후 설치
# Android 8~10에서는 Ubuntu의 su 명령어가 정상동작 하지 않을 수 있다.

# 버전 확인
termux-info

apt update
apt upgrade
apt install openssh    # 1024보다 이하의 포트는 사용 할 수 없으므로 'Termux의 openssh 라이브러리'가 8022로 설정함
passwd
whoami    # 사용자 아이디 u0_a348

apt install termux-tools
# Termux 애플리케이션이 백그라운드에서도 계속 돌게 만듬 (이미 설치 되어 있다.)
termux-wake-lock
ifconfig
sshd
pkill sshd

# 테더링 네트워크에서 Mac에서 ssh에 접근 할 수 없을 경우 (포트 포워딩, 하지만 USB-C 케이블을 뽑으면 포트 포워딩 종료됨)
adb forward tcp:8022 tcp:8022
ssh u0_a348@localhost -p 8022    # 접속 후에 폰을 닫아도 접속 유지됨, 포트 포워딩 때문인지 인터넷도 유지됨

# Ubuntu 설치 (25.10)
apt install proot-distro
proot-distro install ubuntu
# root 권한으로 로그인
proot-distro login ubuntu --user root
# 버전 확인
lsb_release -a

# 백그라운드 Shell 관리
apt install tmux
# 새로운 Shell 생성
tmux new -s shell1
# 현재 Shell 백그라운드로 이동
Ctrl + b d
# 백그라운드 Shell 보기
tmux list-sessions
# 해당 백그라운드로 돌아가기
tmux attach -t shell1
```
* [Ubuntu - user 생성](https://github.com/ovdncids/docker-curriculum/tree/master?tab=readme-ov-file#shell-%EC%A0%91%EC%86%8D)
* https://github.com/nvm-sh/nvm

### Next.js dev 실행
```sh
npm run dev
# Unhandled Rejection: NodeError [SystemError]: A system error occurred: uv_interface_addresses returned Unknown system error 13 (Unknown system error 13)
# 이런 오류가 발생하면
npx next dev -H 127.0.0.1
```

### VSCode server 설치 (웹에서 해당 사용자의 home 폴더 기준으로 VSCode 사용)
```sh
su - {사용자}
curl -fsSL https://code-server.dev/install.sh | sh
code-server --bind-addr 127.0.0.1:8000

adb forward tcp:8000 tcp:8000
http://localhost:8000

# 패스워드 확인
/home/{사용자}/.config/code-server/config.yaml

# 실행 파일 (tmux은 nohup 같은것 사용 불가)
vi code-server.sh

#!/usr/bin/env bash
tmux new -s code-server "/usr/bin/code-server --bind-addr 127.0.0.1:8000"  > code-server.log

chmod +x code-server.sh
./code-server.sh
```
* ❕ `tmux session`은 ssh 접속해서 생성하면 ssh 종료와 함께 꺼지므로 Android쪽에서 사용해야 한다.
* ❕ `code-server`에 접속해서 Terminal을 쓰면 Android의 터미널 환경이다.
