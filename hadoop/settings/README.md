# 🧑🏻‍💻 하둡 설치 가이드
```shell
# homebrew 자체 최신화
$ brew update

# 하둡 설치
$ brew install hadoop
```

<br>

하둡을 실행할 때 기본 파일 시스템을 하둡으로 설정하기 위해 `fs.defaultFS` 값을 `hdfs://localhost:9000`으로 설정하여 실행한다.  
하둡 경로의 `core-site.xml`에 다음과 같이 옵션을 넣으면 된다.  
경로: `/opt/homebrew/Cellar/hadoop/3.4.2(버전)/libexec/etc/hadoop`

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```