# 🔑 대칭키 암호화(Symmetric-key Encryption)
- 암복호화에 사용하는 키가 동일하다.
- 장점
  - 암호화방식에 **속도가 빠르다.**
  - **대용량 Data 암호화에 적합**하다.
- 단점
  - **키를 교환**해야 하는 문제
  - **탈취** 관리 걱정
  - **사람이 증가할수록** 키관리가 어려워짐
  - **확장성 떨어짐**
- Session Key, Secret Key, Shared Key, 대칭키, 단용키라고도 함
- 기밀성을 제공하나, 무결성/인증/부인방지를 보장하지 않음
- 대표적 알고리즘 : 공인인증서의 암호화방식으로 유명한 SEED, DES, 3DES, AES, ARIA, 최근 주목받고 있는 암호인 ChaCha20

## 💡 대칭키 암호화 알고리즘의 종류
1. DES(Data Encryption Standard)
   - 1975년 미국의 연방 표준국(NIST)에서 발표한 대칭 키 기반의 블록 암호화 알고리즘
   - 블록 크기는 64bit, 키 길이는 56bit인 페이스텔(Feistel) 구조
   - DES를 3번 적용하여 보안을 더욱 강화한 3 DES(Triple DES)도 활용됨
2. SEED
   - 1999년 국대 한국인터넷진흥원(KISA)이 개발한 블록 암호화 알고리즘
   - 128bit 비밀키로부터 생성된 16개의 64bit 라운드 키를 사용하여 총 16회의 라운드를 거쳐 128bit의 평문 블록을 128bit 암호문 블록으로 암호화하여 출력하는 방식
   - 블록 크기는 128bit이며, 키 길이에 따라 128bit, 256bit로 분류
3. AES(Advanced Encryption Standard)
   - 2001년 미국 표준 기술 연구소(NIST)에서 발표한 블록 암호화 알고리즘
   - DES의 개인 키에 대한 전사적 공격이 가능해지고, 3 DES의 성능문제를 극복하기 위해 개발
   - 블록 크기는 128bit이며, 키 길이에 따라 128bit, 192bit, 256bit로 분류
   - AES의 라운드 수는 10, 12, 14 라운드로 분류되며, 한 라운드는 SubBytes, ShiftRows, MixColumns, AddRoundKey의 4가지 계층으로 구성
4. ARIA(Academy, Research Institute, Agency)
   - 2004년 국가정보원과 산학연구협회가 개발한 블록 암호화 알고리즘
   - ARIA는 학계(Academy), 연구기관(Research), 정부(Agency)의 영문 앞글자로 구성
   - 블록 크기는 128bit이며, 키 길이에 따라 128bit, 192bit, 256bit로 분류
   - ARIA는 경량 환경 및 하드웨어에서의 효율성 향상을 위해 개발되었으며, ARIA가 사용하는 대부분의 연산은 XOR과 같은 단순한 바이트 단위 연산으로 구성
5. IDEA(International Data Encryption Algorithm)
   - DES를 대체하기 위해 스위스 연방기술 기관에서 개발한 블록 암호화 알고리즘
   - 128bit의 키를 사용하여 64bit 평문을 8라운드에 거쳐 64bit의 암호문을 만듦
6. LFSR(Linear Feedback Shift Register)
   - 시프트 레지스터의 일종으로, 레지스터에 입력되는 값이 이전 상태 값들의 선형 함수로 계산되는 구조로 되어 있는 스트림 암호화 알고리즘
   - LFSR에 사용되는 선형 함수는 주로 배타적 논리합(XOR)이고, LFSR의 초기 비트 값은 시드(Seed)라고 한다.

출처  
[대칭키 vs 공개키(비대칭키)](https://velog.io/@gs0351/%EB%8C%80%EC%B9%AD%ED%82%A4-vs-%EA%B3%B5%EA%B0%9C%ED%82%A4%EB%B9%84%EB%8C%80%EC%B9%AD%ED%82%A4)