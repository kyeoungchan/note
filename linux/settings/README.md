# 💻 리눅스 셋팅
> MAC OS 환경 기반입니다.

## ✅ 우분투 데스크탑 설치
가상 머신에 로그인을 한 후, Terminal 환경에서 다음의 명령어 실행  
`apt`: **Advanced Package Tooll**의 약자로, 우분투와 같은 데비안(Debian) 기반 리눅스 시스템에서 패키지(소프트웨어)를 설치, 업데이트, 삭제 및 관리하는 데 사용되는 명령어다.  
```shell
sudo apt update
// 리눅스 시스템에서 현재 사용할 수 있는 패키지 목록을 최신 정보로 갱신하는 명령어다.
```
```shell
sudo apt intall ubuntu-desktop
// 오래 걸린다.
```
```shell
sudo reboot
```

화면 캡처  
![linux_window1.png](../res/linux_window1.png)  
![linux_window2.png](../res/linux_window2.png)  

<br>

## ✅ SSH로 접속하기
Mac에서 제공하는 터미널에서 직접 리눅스 SSH에 접근하는 방법도 있다.

<br>

1. UTM 리눅스에 SSH 서버 설치
`systemctl`: Linux 운영체제에서 시스템 서비스(daemon)을 관리하는 명령어다.  
```shell
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

2. 리눅스 VM의 IP 확인
```shell
ip addr
// 혹은 hostname -I
// 혹은 ifconfig
```

3. Mac 터미널 환경에서 SSH 접속
```shell
ssh username@192.***.**.*
```

## ✅ 우분투 데스크탑 삭제
가상 머신에 로그인을 한 후, Terminal 환경에서 다음의 명령어 실행  
```shell
# 1. Ubuntu Desktop 패키지 제거
sudo apt remove ubuntu-desktop

# 2. 관련 의존성 패키지도 함께 제거
sudo apt autoremove

# 3. GUI 관련 패키지 완전 제거 (선택사항, 더 깨끗하게 제거하고 싶다면)
sudo apt purge ubuntu-desktop
sudo apt autoremove --purge

# 4. 그래픽 디스플레이 매니저 제거 (GDM3)
sudo apt remove gdm3
# 또는
sudo apt purge gdm3

# 5. 부팅 타겟을 CLI로 변경
sudo systemctl set-default multi-user.target

# 6. 재부팅 후 확인
sudo reboot
```
재부팅 후에는 자동으로 CLI 환경으로 부팅된다.

### 💡 추가 정리(선택 사항)
더 깨끗하게 정리하고 싶다면
```shell
# 사용하지 않는 패키지 정리
sudo apt autoclean
sudo apt clean

# 고아 패키지 제거
sudo apt autoremove
```
> `ubuntu-desktop`을 설치하면 많은 GUI 관련 패키지들이 함께 설치되는데, `autoremove`로 대부분 제거된다.