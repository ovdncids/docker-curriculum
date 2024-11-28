# 라즈베리파이 4B

## 설치
* [Raspberry Pi Imager](https://www.raspberrypi.com/software)
```sh
운영체제: Raspberry Pi OS (LEGACY)
저장소: SD카드 선택

# 설정
hostname 설정: raspberrypi
SSH 사용
사용자 이름: pi
Wifi: 자동 입력
Wifi 국가: GB
시간대: Asia/Seoul
키보드 레이아웃: us

# 쓰기
모든 데이터가 지워집니다. 예
```

## 원격 접속 (putty 또는 ssh)
```sh
# 접속
ssh pi@아이피

## IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY! (https://cpuu.postype.com/post/30065)
ssh-keygen -R 아이피

# 설정
sudo nano /boot/config.txt

## HDMI 최소 해상도
hdmi_safe=1

## HDMI 모니터 안켜질 경우 (https://rasino.tistory.com/287)
hdmi_group=2
hdmi_mode=82
### 82 = 1920x1080 (1080p)
### 85 = 1280x700 (720p)

# VNC
sudo raspi-config
Display Options > VNC Resolution > 1920x1080 또는 1280x720
Interface Options > VNC > Would you like the VNC server > Yes

sudo reboot
# 재부팅 후에 상단 Task에 VNC 아이콘이 있어야 Server로 동작한다.
```

### VNC Server 32-bit RasPiOS issue
* https://github.com/raspberrypi/bookworm-feedback/issues/41

### VNC Client
* https://www.realvnc.com/en/connect/download/viewer/macos  

## Update
* 상단 Task의 업데이트 아이콘과 동일
```sh
sudo apt update
sudo apt upgrade
```

## 한글
* https://velog.io/@thdusdl4767/Raspberry-Pi-Raspberry-Pi-%ED%95%9C%EA%B8%80-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EC%84%A4%EC%B9%98
```sh
sudo apt install fonts-nanum
sudo apt install ibus-hangul

Preferences > Raspberry Pi Configuration > Localisation > Set Locale... > ko (Korean), Character Set: UTF-8
리부팅

sudo ls –ld ~/.config/ibus
sudo rm -fr ~/.config/ibus
Preferences > IBus 기본 설정 > 바로 가기 키 > <Shift>space
                           > 입력 방식 > 한국어 - Hangul
상단 Task > 한글 - Hangul > 한글 상태 체크
```

## Anydesk
* https://anydesk.com/en/downloads/raspberry-pi
* https://remnant24c.tistory.com/330
* http://www.acronet.kr/29221
```sh
# 설치
wget https://download.anydesk.com/rpi/anydesk_6.1.1-1_armhf.deb
sudo dpkg -i anydesk_6.1.1-1_armhf.deb
# 리스트
sudo dpkg -l
# 삭제
sudo dpkg -r anydesk
```

## RDP (원격 데스트톱 연결)
```sh
sudo apt install xrdp
```

## Utils
```sh
# 용량 (https://power-of-optimism.tistory.com/143)
## 디렉토리별 용량 
df -h
## 전체 용량
df .
```

## Node.js
* https://it-jm.tistory.com/19
```sh
sudo curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install nodejs
리부팅
node -v
```

### NVM
* https://github.com/nvm-sh/nvm
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
source ~/.bashrc
nvm list
```

## MariaDB
* https://velog.io/@dogakday/%EB%9D%BC%EC%A6%88%EB%B2%A0%EB%A6%AC%ED%8C%8C%EC%9D%B4-Maria-DB-%EC%84%A4%EC%B9%98
```sh
sudo apt install mariadb-server
systemctl is-active mysql
sudo mysql -u root
```

## Java
* https://backendcode.tistory.com/262
```sh
# 설치
sudo apt install openjdk-8-jdk
# or 
sudo apt install openjdk-11-jdk
java -version

sudo nano ~/.bashrc
# $JAVA_HOME
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH=$PATH:$JAVA_HOME/bin

source ~/.bashrc
echo $JAVA_HOME
```
* [Jenkins](https://github.com/ovdncids/raspberrypi-curriculum/blob/master/Jenkins.md)를 사용하려면 `java 11 버전`을 사용해야 한다.
* `$JAVA_HOME`이 없어도 `Jenkins`는 실행 된다.
