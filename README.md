# AWS EC2에 Spring 애플리케이션 배포 및 JMeter를 이용한 성능 테스트
<br/>

## 🚪 들어가기
 이 프로젝트는 AWS EC2 인스턴스에서 Ubuntu를 실행 중인 Spring 애플리케이션을 배포하고, 로컬 Windows 머신에서 JMeter를 사용하여 성능 테스트를 수행하는 방법을 보여줍니다. 애플리케이션이 예상되는 부하 수준을 처리할 수 있는지 확인하고 성능 병목 현상을 식별하는 것이 목적입니다. <br>
 스프링 애플리케이션을 개발하면서 사용자가 직접적으로 들어올 때의 성능 지표를 파악하고 싶었습니다. JMeter는 Apache 재단에서 제공하는 오픈 소스 툴로, 무료로 사용할 수 있으며 성능이 뛰어나기 때문에 이번 테스트에 활용해 보고자 합니다. 이러한 접근을 통해 실제 운영 환경에서의 애플리케이션 성능을 더 깊이 이해하고, 연구 및 취업 목표를 달성하는 데 도움이 될 것으로 기대하고 있습니다.

<br/>

## ❓ JMeter란
JMeter는 성능 테스트와 부하 테스트에 널리 사용되는 오픈 소스 도구입니다.

### 1. **오픈 소스 및 무료**
- JMeter는 Apache 재단에서 제공하는 오픈 소스 도구로, 무료로 사용할 수 있으며 상업적인 사용에도 제한이 없습니다.

### 2. **다양한 프로토콜 지원**
- HTTP, HTTPS, FTP, JDBC, JMS, SOAP, RESTful API 등 다양한 프로토콜에 대한 성능 테스트를 지원합니다.
- 웹 애플리케이션, 데이터베이스, 메시징 서비스 등 여러 애플리케이션을 테스트할 수 있습니다.

### 3. **확장성**
- 플러그인 시스템을 통해 기능을 확장할 수 있습니다. 사용자는 필요한 경우 추가적인 플러그인을 설치하거나 커스텀 기능을 개발할 수 있습니다.
- 분산 테스트를 지원하여 여러 컴퓨터에서 동시에 부하 테스트를 수행할 수 있습니다.

### 4. **성능 분석 기능**
- 실시간 모니터링을 통해 테스트 실행 중에 성능 데이터를 수집하고 분석할 수 있습니다.
- 결과 데이터를 시각화할 수 있는 다양한 리포트 기능을 제공하며, 그래프와 표로 결과를 분석할 수 있습니다.

### 5. **대규모 부하 테스트 가능**
- 동시에 수천에서 수십만 명의 가상 사용자를 시뮬레이션하여 대규모 부하 테스트를 수행할 수 있습니다.

<br/>

## 💻 시스템 환경 및 소프트웨어
- **소프트웨어**: Mobaxterm  
- **클라우드 서비스**: AWS EC2  
- **운영 체제**: Ubuntu, Windows  
- **성능 테스트 도구**: JMeter  

<br/>

## 1️⃣ 테스트 준비 전: JMeter 설치
1. [JMeter](https://jmeter.apache.org/download_jmeter.cgi) 접속
   

2. 해당 파일 다운로드 후 압축 해제
   
   <img src="https://github.com/user-attachments/assets/521bdf88-05b9-4242-8a0f-323498b8d43f" width="70%">

4.  JMeter 경로로 이동
   ```bash
   cd C:\apache-jmeter-5.6.3\bin
   ```

5. `jmeter.bat` 실행  
```bash
jmeter.bat
```

<br/>

## 2️⃣ 테스트 준비 전: Spring 애플리케이션 생성 및 EC2 서버 배포 
1. Spring 애플리케이션 생성
   
   - Spring 애플리케이션에서 화면에 "Click Me" 버튼을 클릭하면, 이 버튼이 `localhost:8088/test`로 `GET 요청`을 보냅니다. 서버는 요청을 수신하고, 컨트롤러 메서드에서 "요청 들어왔습니다"라는 로그를 남깁니다.
   
   <img src="https://github.com/user-attachments/assets/ccb9b9a0-df53-4f1a-a25f-4362981bc0b5" width="70%">

2. EC2 서버 배포
   ```bash
   # 로컬(윈도우)에서 EC2(우분투)로 Spring 애플리케이션 배포
   scp -i {pem key} {Spring 애플리케이션 경로}  {username}@{ip}:/home/ubuntu
   ```
   <img src="https://github.com/user-attachments/assets/2311ff85-a544-4c8c-8083-38552f59a128" width="55%">

  

<br/>

## 🧪 JMeter 시나리오 테스트

### 시나리오 예시
이 테스트는 웹 애플리케이션의 성능 및 안정성을 평가하기 위한 것입니다. 100명의 사용자가 동시에 애플리케이션에 요청을 보내고, 이를 통해 서버의 응답 시간 및 처리 능력을 측정합니다.


### 설정 세부사항
- **Number of Threads**:  
  동시에 100명의 사용자가 애플리케이션에 접속하도록 설정합니다.  

- **Ramp-Up Period**:  
  100명의 스레드가 60초에 걸쳐 순차적으로 시작됩니다. 즉, 매초 약 1.67개의 스레드가 시작됩니다. 이를 통해 갑작스러운 트래픽 스파이크를 피하고 서버의 부하를 점진적으로 증가시킬 수 있습니다.  

- **Loop Count**:  
  각 스레드는 5번 요청을 반복합니다. 총 요청 수는 100 스레드 × 5 루프 = 500 요청이 됩니다.  

  <img src="https://github.com/user-attachments/assets/beb21479-582b-46ec-bd6f-f0da9ac6f71e" width="550">

<br/>

## 🍂 성능 테스트 결과

#### 이미지 1: Summary Report (테스트의 전반적인 성능을 평가)
<img src="https://github.com/user-attachments/assets/76333f5a-dc54-4509-b30d-b3b493dfcd01" width="70%">

- **Sample (샘플 수)**:  
  총 5000개의 요청이 테스트되었습니다. 이는 테스트의 전반적인 부하를 나타냅니다.

- **Average (평균 응답 시간)**:  
  평균 응답 시간은 223ms로, 서버가 요청에 응답하는 데 걸리는 평균 시간입니다. 이 값이 낮을수록 서버의 응답 성능이 좋습니다.

- **Min (최소 응답 시간)**:  
  최소 응답 시간은 6ms로, 가장 빠른 요청의 응답 시간을 나타냅니다. 이는 서버가 최적의 조건에서 얼마나 빠르게 응답할 수 있는지를 보여줍니다.

- **Max (최대 응답 시간)**:  
  최대 응답 시간은 7632ms로, 가장 느린 요청의 응답 시간을 나타냅니다. 이 값은 서버에 부하가 많이 걸릴 때의 성능을 반영합니다.

- **Standard Deviation (표준 편차)**:  
  표준 편차는 529.71로, 응답 시간의 변동성을 나타냅니다. 값이 클수록 응답 시간이 불안정하다는 것을 의미하며, 일관성이 부족하다는 신호일 수 있습니다.

- **Error (오류 비율)**:  
  오류 비율이 0.0%로, 모든 요청이 성공적으로 처리되었음을 나타냅니다. 이는 애플리케이션이 안정적으로 작동하고 있다는 좋은 신호입니다.

- **Throughput (처리량)**:  
  처리량은 초당 83.51개의 요청을 처리할 수 있음을 의미합니다. 이 값은 서버의 부하 처리 능력을 나타내며, 높은 수치일수록 더 많은 요청을 처리할 수 있다는 것을 의미합니다.

- **Received (받은 바이트 수)**:  
  총 14.43 KB의 데이터가 클라이언트로부터 수신되었습니다. 이 값은 요청에 대한 서버의 응답을 포함한 데이터의 양을 나타냅니다.

- **Sent KB (전송된 데이터 크기)**:  
  총 14.03 KB의 데이터가 클라이언트에게 전송되었습니다. 이는 서버가 클라이언트에게 전송한 데이터의 양을 나타냅니다.

- **Avg (평균 전송 크기)**:  
  평균적으로 요청당 177.0 KB의 데이터가 전송되었습니다. 이는 각 요청이 클라이언트와 서버 간에 얼마나 많은 데이터를 주고받는지를 보여줍니다.

#### 이미지 2: Transactions per Second (주어진 시간 동안 처리된 트랜잭션 수 확인)
<img src="https://github.com/user-attachments/assets/bd8a94a0-d6ce-455c-b2a7-51ad8b8530fd" width="70%">

1. **6초까지 160 TPS**:  
   초기 부하가 급격히 증가하여 서버가 초당 160개의 요청을 처리할 수 있는 상태입니다. 이는 서버가 부하를 잘 처리하고 있다는 긍정적인 신호입니다.

2. **12초대 40 TPS**:  
   TPS가 급격히 감소하여 40으로 떨어진 것은 서버의 성능이 저하되었거나, 처리할 수 있는 요청의 수가 줄어들었다는 것을 의미합니다. 이 경우, 성능 병목 현상이 발생했을 가능성이 높습니다.

3. **14초에 160 TPS로 증가**:  
   TPS가 다시 160으로 증가하는 것은 서버가 부하를 다시 견딜 수 있는 상태로 복구되었다는 것을 나타냅니다. 이는 일시적인 문제였을 수도 있습니다.

4. **30초부터 54초까지 80 TPS 유지**:  
   TPS가 80으로 안정적으로 유지되고 있다는 것은 서버가 일정 수준의 부하를 지속적으로 처리할 수 있음을 의미합니다. 그러나 이전 160 TPS에 비해 낮은 수치이므로, 서버가 최적의 성능을 발휘하지 못하고 있다는 신호일 수 있습니다.

<br/>

## 💎 결론
이번 프로젝트를 통해 AWS EC2 인스턴스에 배포한 Spring 애플리케이션의 성능을 JMeter를 사용해 테스트하였습니다. 사용자 요청에 대한 응답 속도와 처리량을 측정함으로써, 애플리케이션이 예상 부하를 효과적으로 처리할 수 있는지를 확인했습니다. JMeter는 Apache 재단에서 제공하는 오픈 소스 도구로, 무료로 사용할 수 있으며 뛰어난 성능을 보이는 만큼, 향후 다른 프로젝트에서도 적극 활용할 계획입니다. 이 테스트를 통해 얻은 데이터는 저의 연구 및 취업 목표 달성에 중요한 기반이 될 것입니다.
