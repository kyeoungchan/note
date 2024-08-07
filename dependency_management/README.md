# 빌드 / 의존성 관리 도구

## 빌드 / 의존성 관리의 정의
### 빌드란
소스코드 파일을 컴파일을 통하여 실행할 수 있는 파일로 변환하는 과정 혹은 결과물을 말한다.

### 빌드 / 의존성 관리 도구란
소스코드를 빌드하면서 여러가지 라이브러리를 사용하는데, 프로젝트가 어떤 외부 라이브러리를 사용하고 있는지를 별도로 관리하는 도구를 말한다.

### 빌드 / 의존성 관리 도구가 수행하는 작업
- 종속성 다운로드
- 소스코드를 바이너리코드로 컴파일(Java 기준)
- 바이너리 코드를 패키징
- 테스트 실행
- 프로덕션 시스템에 배포

### 의존성 관리 도구가 중요한 이유
- 같은 팀에서 협업을 하면서 의존성이 제대로 설치되어 있지 않으면 애플리케이션을 실행할 수 없다.
  - 만약 혼자서 개발한다면, 그냥 필요한 라이브러리들을 컴퓨터에 미리 설치해두기만 하면 된다.
- 오픈 소스로 관리되는 라이브러리의 소스코드를 그대로 붙여넣거나, 어떤 라이브러리를 쓴다고 문서화하는 방법도 있긴하다.
  - 의존성의 의존성과 같은 재귀 의존 현상과 라이브러리 각각의 버전 업데이트 대응을 별도의 도구 없이 해결하기에는 시간 소모가 심하다.
> 언어 차원에서 제공되는(built-in) 라이브러리는 이미 설치되어 있으므로 관리할 필요가 없다.

## Maven
### 특징
- Life Cycle: 정해진 라이프사이클에 의하여 작업을 수행하며, 전반적인 프로젝트 관리 기능을 포함하고 이다.
- 프로젝트 모델링: Maven은 필요한 라이브러리를 pom.xml에 정의한다.
- 플러그인을 통한 전역적인 재사용
  - 빌드에 대한 대부분의 책임을 각 플러그인에 위임한다.
  - 플러그인들은 Maven 저장소(Repository)에 저장된다.
### Maven 설정 파일
- pom: Maven의 기능을 이용하기 위해 사용하는 xml 문서
  - 프로젝트마다 1개씩 갖는다.
  - 필요한 라이브러리들을 pom.xml에 정의한다.
    - => 라이브러리 실행에 필요한 다른 라이브러리 관리 및 네트워크를 통한 다운로드를 해준다.
- pom.xml의 주요 기능
  - 프로젝트 정보: 프로젝트의 이름, 라이센스 등
  - 빌드 설정: 소스, 리소스, 라이프사이클 별 실행한 플러그인 등 빌드와 관련되 성질
  - 빌드 환경: 사용자 환경 별로 달라질 수 있는 프로파일 정보
  - pom 연관 정보: 의존 프로젝트(모듈), 상위 프로젝트, 포함하고 있는 하위 모듈

## Gradle
### Groovy란?
- JVM에서 사용되는 스크립트 언어
- Java와 호환이 되며, Java 클래스 파일을 그대로 Groovy 클래스 파일로 사용 가능

### Gradle 특징
- 간결함: xml에 비해 간결한 정의가 가능하다.
- 지원: Maven과 Ivy 레포지토리를 완전히 지원한다.

### Gradle 설정 파일
- build.gradle
  - 빌드에 대한 모든 기능을 정의
  - 환경 설정, 빌드 방법, 라이브러리 정보를 기술하여 프로젝트의 관리환경을 구성
- setting.gradle
  - 프로젝트 구성을 설정할 때 작성하는 파일
  - 프로젝트 간의 의존성 및 멀티 프로젝트를 구성할 때 사용
  - 싱글 프로젝트의 경우 생략 가능


출처  
[백엔드가 이정도는 해줘야 함 - 8. 의존성 관리 도구 결정](https://velog.io/@city7310/%EB%B0%B1%EC%97%94%EB%93%9C%EA%B0%80-%EC%9D%B4%EC%A0%95%EB%8F%84%EB%8A%94-%ED%95%B4%EC%A4%98%EC%95%BC-%ED%95%A8-8.-%EC%9D%98%EC%A1%B4%EC%84%B1-%EA%B4%80%EB%A6%AC-%EB%8F%84%EA%B5%AC-%EA%B2%B0%EC%A0%95)  
[빌드 관리 도구 - 메이븐(Maven)과 그래들(Gradle)](https://scoring.tistory.com/entry/%EB%B9%8C%EB%93%9C-%EA%B4%80%EB%A6%AC-%EB%8F%84%EA%B5%AC-%EB%A9%94%EC%9D%B4%EB%B8%90Maven%EA%B3%BC-%EA%B7%B8%EB%9E%98%EB%93%A4Gradle)