# 💻 Kafka 실습 환경 세팅하기

> AWS EC2(Elastic Compute Cloud)에서 인스턴스를 발급받아 실행해볼 것이다.  
> 카프카를 안전하게 서비스로 운영하기 위해서는 최소 3대의 서버로 카프카 클러스터를 구축해야 하지만,  
> 카프카 클라이언트 개발 실습을 위해 1대만 설치한다. 

<br>

![EC2_AMI_selection.png](../res/EC2_AMI_selection.png)  
EC2 인스턴스를 설치하는데, 다음과 같이 Amazon Linux 2023 kernel-6.1 AMI로 선택했다.  
Amazon에서 직접 만든 템플릿으로, EC2 성능에 최적화가 되어있다.  


<br>

![EC2_instance_type.png](../res/EC2_instance_type.png)
인스턴스 유형에는 다음과 같이 `t2.micro`로 설정하고, 메모리는 1GB로 설정한다.  
➡ 주키퍼와 카프카를 실행시킬건데, 두 프로세스에 각각 400MB의 힙 메모리를 설정하기 위해서다.  
그리고 프리티어인지 꼭 확인해둔다.  

<br>

![keypair_setting.png](../res/keypair_setting.png)  
키 페어를 생성한다.  
키 페어 이름은 `test-kafka-server-key`로 설정했다.  

<br>


![inbound_rules.png](../res/inbound_rules.png)
보안 그룹에서 Inbound 규칙을 설정한다.
- `9092` port: 카프카 브로커의 기본 포트
- `2181` port: 주키퍼의 기본 포트

소스 IP를 `0.0.0.0/0`으로 설정하면 모든 IP 주소에서 접속하는 것을 허용한다.  
운영이 아니니 그렇게 해도 되지만, 나의 경우 우리집 와이파이 IP를 설정했다.

<br>

> Inbound: 발급 받은 인스턴스 외부로부터 들어오는 트래픽  
> ➡ 기본으로 ssh(22번 포트)만 설정돼 있다.
> <br>
> 
> Outbound: 인스턴스로부터 나가는 트래픽  
> ➡ 기본으로 모든 IP, 모든 port 접속 가능으로 설정돼 있다.

<br>

```shell
# read 권한만 가지고 있도록 설정
Kyeongchanui-MacBookPro:~ kyeongchanwoo$ chmod 400 test-kafka-server-key.pem

# read 권한만 있는 거 확인
Kyeongchanui-MacBookPro:~ kyeongchanwoo$ ll test-kafka-server-key.pem
-r--------@ 1 kyeongchanwoo  staff  1674 11월 26 06:31 test-kafka-server-key.pem

# 기본 유저명은 ec2-user다.
Kyeongchanui-MacBookPro:~ kyeongchanwoo$ ssh -i test-kafka-server-key.pem ec2-user@13.209.68.207
The authenticity of host '13.209.68.207 (13.209.68.207)' can't be established.
ED25519 key fingerprint is SHA256:vjA0FCdQgDQKu2ataJT6nVLysPVXXN9xtCXMh7DL03k.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '13.209.68.207' (ED25519) to the list of known hosts.
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
```