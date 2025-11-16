# 💻 리눅스 셋팅
> MAC OS 환경 기반입니다.

## ✅ 가상화 머신 설치
![virtual_machine1.png](../res/virtual_machine1.png)
1. 가상화(Virtualize)
   - 의미: 가상화는 호스트 머신(실제 컴퓨터)의 CPU 아키텍처와 동일한 게스트 머신을 실행하는 방식이다. 즉, CPU 명령어를 직접적으로 실행하는 하드웨어 가속을 이용한다.
   - 조건: 호스트와 게스트 CPU 아키텍처가 동일해야한다.(예를 들어, ARM64 기반의 Mac에서 ARM64 기반의 게스트 OS를 실행해야한다.)
   - 속도: 매우 빠름. 하드웨어 가속을 이용하므로 거의 네이티브 성능에 가까운 속도를 제공한다.
2. 에뮬레이션(Emulate)
   - 의미: 에뮬레이션은 호스트 CPU와 다른 아키텍처의 게스트 OS를 실행하기 위해 CPU 명령어를 소프트웨어적으로 변환하는 방식이다.
   - 조건: 호스트와 게스트의 CPU 아키텍처가 달라도 동작한다.(예를 들어, ARM64 기반의 Mac에서 x86-64 기반의 게스트 OS를 실행할 수 있다.)
   - 속도: 느림. 소프트웨어를 통해 CPU 명령어를 변환해야하므로 성능이 저하된다.

호스트 환경에 맞는 우분투(나의 경우 ARM64)를 준비한 상황이라면 가상화를 선택하면 된다.

<br>

![virtual_machine2.png](../res/virtual_machine2.png)  
나의 경우, 2048로 세팅했다.

<br>

![virtual_machine3.png](../res/virtual_machine3.png)
부탕 이미지 종류를 선택하고, 사전에 다운받은 우분투 이미지 파일을 지정한다.

<br>

![virtual_machine4.png](../res/virtual_machine4.png)
나의 경우, 20GiB로 세팅했다.

<br>

![virtual_machine5.png](../res/virtual_machine5.png)
![virtual_machine6.png](../res/virtual_machine6.png)

쭉쭉 기본 세팅으로 넘어간다.

<br>

![virtual_machine7.png](../res/virtual_machine7.png)

생성한 가상머신을 실행시킨다.

<br>

쭉쭉 크게 별다른 세팅 변경 없이 진행했다.  
![virtual_machine8.png](../res/virtual_machine8.png)
![virtual_machine9.png](../res/virtual_machine9.png)
![virtual_machine10.png](../res/virtual_machine10.png)
![virtual_machine11.png](../res/virtual_machine11.png)
![virtual_machine12.png](../res/virtual_machine12.png)
![virtual_machine13.png](../res/virtual_machine13.png)
![virtual_machine14.png](../res/virtual_machine14.png)
![virtual_machine15.png](../res/virtual_machine15.png)
![virtual_machine16.png](../res/virtual_machine16.png)
![virtual_machine17.png](../res/virtual_machine17.png)
레디스를 공부하기 위해서 나는 계정을 레디스 관련으로 지었다.  
자유롭게 username 및 password 등을 작성하면 된다.
![virtual_machine18.png](../res/virtual_machine18.png)
![virtual_machine19.png](../res/virtual_machine19.png)

<br>

![virtual_machine20.png](../res/virtual_machine20.png)
위와 같이 설치가 완료되면 Reboot Now를 클릭하면 절대 안 된다.  
사진을 참고해서 우측 위에 아이콘을 클릭해야한다.  
![virtual_machine21.png](../res/virtual_machine21.png)  
CD를 클릭하고 꺼내기를 누른다.  

![virtual_machine22.png](../res/virtual_machine22.png)  
그리고 Restarts the VM을 클릭하여 가상머신을 재구동 시킨다.

<br>

![virtual_machine23.png](../res/virtual_machine23.png)
성공이다.


<br>

**참고 자료**  
[[ARM Mac] 맥북에 리눅스 설치하기(UTM 사용)](https://ruyagames.tistory.com/49)  



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