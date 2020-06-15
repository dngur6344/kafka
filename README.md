### kafka에 대한 학습
    회사에서 Spring Boot를 이용하여 만드는 서비스를 개발 중인데, 내가 맡은 부분은
    Kafka Consumer를 구현하여 메세지를 읽은 뒤 저장소로 보내는 서비스를 개발하는 것이다.
    그래서 Kafka에 대해 공부를 하며 얻은 지식을 정리할겸 업무와는 별개로 간단한 환경을 구축해보기로 했다.
    
------------------------------------------------------------------------------------
#### zookeeper 환경 구축
    원하는 경로에 Zookeeper 알집을 풀어서 zookeeper를 먼저 설정해야 한다. 
    Kafka가 구동되기 위해서는 zookeeper가 필수로 구동되어야 하기 때문이다.
    
    zookeeper를 설치한 뒤 구동을 하기 위해  **config/zoo.cfg**  파일을 수정하였다.
    
<img src="/image/zookeeper setting.png"></img>

    포트가 가려졌지만 zookeeper의 포트는 2181이다.
    
    그리고 명령어를 입력하여 zookeeper를 실행시켜보자.
<img src="/image/zookeeper start.png"></img>

    이제 실행 중인 프로세스를 zookeeper 포트인 2181로 검색해서 실행되고 있는지 확인해보자.
<img src="/image/zookeeper port.png"></img>

    잘 실행되고 있다. 그럼 zookeeper를 띄웠으니 kafka를 실행시켜보자
-----------------------------------------------------------------------------------
#### Kafka 실행
    zookeeper와 마찬가지로 원하는 경로에 압축파일을 풀어 kafka를 설치하자.
    해당 경로에 들어간 뒤 config/server.properties 파일을 열어서 설정을 하자.
    해당 파일은 kafka를 실행할 때 읽어들이는 kafka 설정파일이다.
<img src="/image/kafka server.png"></img>

    여기를 자세히 보면 broker라는 친구가 있다. 이 broker는 메세지들을 담는 partition들을 모아놓은 창고...?라고
    봐도 무방할 것 같다. Broker 안에 Topic 안에 Partition이 있는거니까.
    
    그리고 listeners라는 항목도 있는데 이건 broker의 주소를 나타내는 것이라고 알고 있다.
    그 밑에 advertised.listeners 항목도 있는데 이건 설정을 해도되고 안해도 된다. 
    다른 client들이 해당 broker에 접근할 때 사용하는 주소인데
    설정을 안할 경우 listeners 항목과 똑같이 설정이 되고 
    좀 복잡한 네트워크 구조면 다르게 설정하는 것으로 알고 있다.
<img src="/image/kafka server zookeeper.png"></img>
    
    
    여기에는 zookeeper 서버들을 적으면된다. 
    나는 지금 하나의 zookeeper 인스턴스를 사용하고 있어서 해당 주소만 적어놓은 상태인데, 
    여러 개의 zookeeper 인스턴스를 사용한다면 각각의 주소를 다 적으면 된다.
    
<img src="/image/kafka daemon.png"></img>

    해당 명령어를 입력해 백그라운드로 kafka를 띄우려고 하는데 제대로 실행이 되지 않고 있다.
    문제를 좀 해결해야겠다.
