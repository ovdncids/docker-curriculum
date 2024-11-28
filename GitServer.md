# Git Server
## 설치
```sh
sudo apt install git-core

# 사용자 git 추가
sudo useradd -s /bin/bash -m git
sudo passwd git
su - git
```

## Repository 만들기
```sh
mkdir /home/git/Repositories
mkdir /home/git/Repositories/repository-a
cd /home/git/Repositories/repository-a
git init --bare
```

## Project 클론
```sh
mkdir /home/git/Projects
cd /home/git/Projects
git clone ssh://git@localhost:/home/git/Repositories/repository-a
```

## Project에 remote 추가
```sh
cd /home/git/Projects
mkdir repository-a
cd repository-a
git init
git remote add origin ssh://git@localhost:/home/git/Repositories/repository-a
echo "# README" >> README.md
git add .
git commit -m "README.md"
git push
## git push --set-upstream origin main
```
* `git clone ssh://git@[ip]:[포트번호]/home/git/Repositories/repository-a`

## Push, Pull 할때 비밀번호 묻지 않기
* https://hackernoon.com/a-guide-on-how-to-host-your-own-git-server-with-raspberry-pi
* https://www.lainyzine.com/ko/article/handling-ssh-permission-denied-errors
* [비밀번호 등록](https://jjuke-brain.tistory.com/entry/Git-Push-%EB%98%90%EB%8A%94-Pull-%ED%95%A0-%EB%95%8C-id-password-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0-Github-Personal-Access-Token)
```sh
# SSH 키 파일 만들기
ssh-keygen
## Enter file in which to save the key (/Users/사용자/.ssh/id_rsa): /Users/사용자/.ssh/git_rsa
## Jenkins 계정의 경우 id_rsa 파일이 없으므로 id_rsa 파일로 생성

# SSH 키 파일에 연결 정보 넣기
ssh-copy-id -i ~/.ssh/git_rsa.pub -p 포트 git@아이피
## 로그인

# 셀에 등록 (진행 하지 않아도 되는듯 함)
ssh-agent zsh && ssh-add ~/.ssh/git_rsa
# ssh-agent bash && ssh-add ~/.ssh/git_rsa
## 셸이 새로 열린다.
exit
git pull
```

# Github CLI
* [설치 방법](https://velog.io/@mythos/Raspberry-Pi-github-CLI-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)
```sh
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo gpg --dearmor -o /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

## 로그인
```sh
gh auth login

# What account do you want to log into? GitHub.com
# What is your preferred protocol for Git operations on this host? HTTPS
# Authenticate Git with your GitHub credentials? Yes
# How would you like to authenticate GitHub CLI? Paste an authentication token
https://github.com/settings/tokens < 토큰 생성 후 붙여 넣기

git clone https://...
```
