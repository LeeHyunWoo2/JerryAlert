
# JerryAlert
학원에서 선생님이 원격제어 프로그램으로 내 화면을 볼때, 그에 맞춰 GIF 이미지가 출력되도록 시도한 프로젝트에서 발전함



## Screenshots

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)


## 주요 기능

&nbsp;1. **사용자 설정 인터페이스**:
   - 사용자가 GUI를 통해 네트워크 어댑터와 포트 번호를 선택하고, 트래픽 갱신 간격과 임계값을 설정할 수 있음.
   - UI에 표기되는 수치에는 EMA를 적용시켜 트래픽 변화 폭이 끊기거나 날뛰지 않고, 매우 부드럽게 전환됨.

&nbsp;2. **네트워크 트래픽 모니터링**:
   - 지정한 네트워크 어댑터의 실시간 트래픽 상태를 모니터링하고, 업로드 및 다운로드 트래픽 수치를 제공.
   
&nbsp;3. **알림 및 팝업 경고 시스템**:
   - 트래픽이 지정한 임계값이 넘어설 경우 애니메이션 GIF가 포함된 경고 팝업을 띄워 사용자에게 알림.

&nbsp;4. **버퍼를 통한 네트워크 트래픽 관리**:
   - 네트워크 신호가 매우 짧은 간격으로 수신되기 때문에 버퍼 전환 및 스케줄러를 사용하여 최적화.


## 사용된 기술 및 라이브러리
- **Java 17**
- **Swing:** 인터페이스 구현
- **Pcap4j:** 네트워크 패킷 캡처 라이브러리
- **Wireshark:** 네트워크 트래픽 분석 도구


## Requirements
1.&nbsp;**Npcap** (필수)

&nbsp; 이 파일이 있어야 패킷 캡처가 작동합니다.  
&nbsp; [Npcap 다운로드 링크](https://npcap.com)에서 설치해주세요.

&nbsp;2. **JRE** (Java Runtime Environment)

&nbsp; exe파일이 JRE를 포함하고 있을 수도 있으나 만약 JRE가 필요할 경우,  
&nbsp; [Java 다운로드 링크](https://www.java.com/ko/download/)에서 설치를 해야 합니다.

### 시행착오 및 배운점

네트워크 트래픽이 예상보다 너무 촘촘하고 빠른 간격으로 수신되었고, 그 때문에 GIF 애니메이션이 멈추는 상황을 굉장히 오랫동안 해결하지 못했습니다.   

여러 기능을 동시에 구현하려고 스레드 풀을 늘려갈수록 경합이 매우 쉽게 발생하면서 로직 전체가 락에 걸려 멈추는 일도 굉장히 많이 겪었고, 그래서 스레드 관리는 굉장히 철저해야 한다는걸 배웠습니다. 민감한것 뿐만 아니라, 증상도 굉장히 치명적으로 다가오기 때문에 더더욱 그렇게 느꼈습니다.  

아래의 항목은 현재 기억에 남는 시행착오중 일부를 적은겁니다. 만든지 두달정도 지나고 나서야 작성할 여유가 생긴지라 일단 이거라도 기록해봅니다. 나머진 나중에...

- 스케줄러 줄이기
UI가 갱신되는 간격과 GIF이미지가 한번 출력되면 최소한 3초는 출력되도록 하기 위해서 스케줄러를 추가했었는데, 그 부분을 System.currentTimeMillis(); 를 이용해 비교하는 방식으로 하여 두개 정도의 스케줄러를 줄이는데 성공했습니다. 이 방식을 채택하는김에 겹치는게 없게 대부분의 로직을 순서대로 이어지게 하였는데, 너무 한계가 명확해서 다음엔 절대 이런식으로 설계 안할겁니다.

- 버퍼 로직 충돌문제
패킷캡처 자체가 버퍼를 사용하고 있지만, 이것만으로 부족해 버퍼를 따로 마련하고 활용하려 했으나, 버퍼를 비우는 로직과, 버퍼를 채우는 로직이 한 버퍼에 동시에 접근하게 될 경우 프로그램이 뻗어버리고, 그렇다고 synchronized를 걸어봤더니 로직이 한번 작동하고 굳어서 고민하던중, 물레방아가 떠올라서 설마 번갈아 쓰는 방법이 있을까 하고 검색해본 결과 정말로 사용하는 방법이라서 그걸 이용해 놓치는 패킷 없이 충돌문제를 해결하는데 성공했습니다.

- EMA
이 부분은 최적화에 성공하고나서 마지막으로 직면한 과제인데, UI의 트래픽 수치가 너무 잘 갱신되면서 순간적으로 0바이트인것까지 캐치해버려 수치가 바뀔때 점멸수준으로 날뛰는바람에 정신사나워지는 문제였습니다. 하지만 코인을 하면서 얻은 얼마 안되는 쓸만한 지식으로 이렇게 쓰일줄은 몰랐네요. 이동평균 자체가 이런용도로 쓰는거라서 이게 JAVA로 가능하다는건 예전에 어떤 블로그를 지나가다 봤던지라 바로 이용해봤습니다.
