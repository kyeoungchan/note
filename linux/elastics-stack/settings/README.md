# 💻 리눅스 환경 엘라스틱서치 설치
## ✅ wget을 이용한 설치
```shell
es@es-vm:~$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.1.5-linux-aarch64.tar.gz
```
```shell
es@es-vm:~$ tar -xzvf elasticsearch-9.1.5-linux-aarch64.tar.gz
```
> tar 옵션
> - x: 아카이브 풀기
> - z: gz 압출 파일 적용
> - v: 진행 과정 표시
> - f: 아카이브 해제 파일명을 지정

```shell
es@es-vm:~/elasticsearch-9.1.5$ ./bin/elasticsearch 
```

## ✅ 리눅스 패키지 매니저를 이용한 설치
> apt-key는 Ubuntu 22.04 이후로 deprecated(폐기 예정) 되었고,
이제는 GPG 키를 /usr/share/keyrings 에 직접 저장하고 signed-by 옵션을 사용해야 합니다.

```shell
# GPG 키를 받아서 keyrings 디렉토리에 저장
es@es-vm:~$ curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```
```shell
# apt-transport-https 패키지 설치
es@es-vm:~$ sudo apt install apt-transport-https
```
```shell
# 저장소 등록 (예: Elasticsearch 8.x, 9.x 모두 동일)
es@es-vm:~$ echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | \
sudo tee /etc/apt/sources.list.d/elastic-9.x.list
```
```shell
# 패지키 매니저 업데이트
es@es-vm:~$ sudo apt-get update
```
```shell
# 엘라스틱서치 최신 버전 설치
es@es-vm:~$ sudo apt-get install elasticsearch
```
```shell
# systemd를 이용해 엘라스틱서치 실행
es@es-vm:~$ sudo /bin/systemctl start elasticsearch.service
```
```shell
# systemd를 이용해 엘라스틱서치 실행 여부 확인
es@es-vm:~$ sudo /bin/systemctl status elasticsearch.service
```
```shell
# systemd를 이용해 엘라스틱서치 종료
es@es-vm:~$ sudo /bin/systemctl stop elasticsearch.service
```
> 리눅스 패키지 매니저를 이용할 경우 리눅스의 기본 설치 경로를 따른다.
> - 실행 파일 경로: /usr/share/elasticsearch
> - 로그 경로: /var/log/elasticsearch
> - 시스템 설정 파일 경로: /etc/default/elasticsearch
> - 설정 경로: /etc/elasticsearch
> - 데이터 저장 경로: /var/lib/elasticsearch

<br>

# 💻 리눅스 환경 키바나 설치
## ✅ wget을 이용한 설치
```shell
es@es-vm:~$ wget https://artifacts.elastic.co/downloads/kibana/kibana-9.1.5-linux-aarch64.tar.gz
```
```shell
es@es-vm:~$ tar -xzvf kibana-9.1.5-linux-aarch64.tar.gz
```
> tar 옵션
> - x: 아카이브 풀기
> - z: gz 압출 파일 적용
> - v: 진행 과정 표시
> - f: 아카이브 해제 파일명을 지정

```shell
es@es-vm:~/kibana-9.1.5$ ./bin/kibana 
```

## ✅ 리눅스 패키지 매니저를 이용한 설치
> 위에서 elasticsearch 설치 중 다음의 과정을 거쳤다면 생략 가능하다.
> - GPG 키를 받아서 keyrings 디렉토리에 저장
> - apt-transport-https 패키지 설치
> - 저장소 등록 (예: Elasticsearch 8.x, 9.x 모두 동일)
> - 패지키 매니저 업데이트

```shell
# 키바나 최신 버전 설치
es@es-vm:~$ sudo apt-get install kibana
```
```shell
# systemd를 이용해 키바나 실행
es@es-vm:~$ sudo /bin/systemctl start kibana.service
```
```shell
# systemd를 이용해 키바나 실행 여부 확인
es@es-vm:~$ sudo /bin/systemctl status kibana.service
```
```shell
# Kibana enrollment token 생성
es@es-vm:~$ sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```
위의 명령어 수행 후 localhost:5601에 들어가고 토큰을 요구하면 입력하면 된다.
```shell
# systemd를 이용해 키바나 종료
es@es-vm:~$ sudo /bin/systemctl stop kibana.service
```

