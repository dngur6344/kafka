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

    문제를 해결했다.
<img src="/image/kafka zookeeper fix.png"></img>

    이 부분을 실제 ip주소가 아니라 localhost로 변경하였더니 실행이 잘된다. 이유는 아직 뭔지 모르겠다.
    그럼 netstat - lntp | grep 9092 를 이용하여 제대로 실행되는지 확인해보자.
<img src="/image/kafka success.png"></img>

    제대로 실행이 되고 있다!
---------------------------------------------------------------------------------------
### Kafka topic 설정.
    현재 broker는 id 0이며 해당 broker에는 아직 topic이 없다. 
    topic이란 consumer가 구독하는 단위인데 여러 consumer들이 모인 
    consumer group 하나가 하나의 topic을 담당한다. 
    그렇다면 하나의 topic은 여러 consumer group에서 메세지를 가져갈 수 있고,
    하나의 consumer group은 하나의 topic만 담당하므로
    topic : consumer group = 1 : N 
    관계라고 할 수 있겠다.
    (이건 잘 몰랐는데 stack overflow에서 찾아봤더니 있다!
    https://stackoverflow.com/questions/57753211/kafka-use-common-consumer-group-to-access-multiple-topics)
    
    또한 topic은 여러 partition으로 나뉘는데 한 partition에 
    같은 consumer group 내의 여러 consumer가 접근할 수 없다.
    하지만 partition의 수보다 consumer group내의 consumer의 수가 적으면,
    하나의 consumer가 여러 partition에서 메세지를 읽어온다.
    
    아무튼 다시 실습을 해보면
<img src="/image/topic initial.png"></img>
    
    처음 실행시킨 것이기에 topic이 존재하지 않는다. 여기에 topic을 하나 만들자.
<img src="/image/topic create.png"></img>
    
    이건 test라는 topic을 만들고 그 안에 partition을 두개를 할당한 것을 의미한다.
    topic을 만들었으니 제대로 만들었는지 확인해보자.
    
<img src="/image/topic check.png"></img>
    
    제대로 만들어진 것을 확인할 수 있다!
    아까 설정한 2개의 partition도 0번 broker에 나뉘어졌는지 확인해보자
    
<img src="/image/partition check.png"></img>

    0번 broker에 partition이 0 과 1 두개가 위치함을 알 수 있다.
    
--------------------------------------------------------------------------------------------------------------
### Producer - Consumer console 

    topic에 대한 setting이 끝났으니 기본으로 주어지는 producer console 과
    consumer console을 실행시켜서 올바르게 메시지를 주고받는지 확인해보자.
    
    먼저 명령어를 이용하여 producer console을 실행시키자
    bin/kafka-console-producer.sh --broker-list {ip address}:9092 --topic {topic name}
    
    그런데 웬걸 에러가 뜨면서 실행이 되질 않는다.
<img src="/image/producer error.png"></img>

    그래서 구글링을 해본 결과 stack overflow에서 해답을 찾을 수 있었다.
    https://stackoverflow.com/questions/47677549/kafka-zookeeper-connection-to-node-1-could-not-be-established-broker-may-no
    
    이유는 single node만을 이용해서 서버를 구성할 경우 server.properties의
    listeners 항목에 localhost를 적어줘야 한다는 것이다.
    
<img src="/image/kafka re setting.png"></img>
    
    이유는 모르겠지만 시키는 대로 하고 producer와 consumer console을 각각 띄워본다.
    그리고 몇 개의 단어를 입력해봤더니
<img src="/image/comm success.png"></img>
    
    왼쪽 화면이 producer이고 오른쪽 화면이 consumer인데 producer에서 보내는 메시지가
    consumer에서 제대로 출력되는 것을 볼 수 있다.
    
    kafka 서버가 제대로 구동되었으므로 이젠 Spring boot를 이용해서 
    각각의 producer와 consumer를 구현하기로 계획을 세웠다.
    
    일단 계획은 partition이 두개니까 consumer project를 두개 작성을 할 것이고, 
    partition 두개를 하나의 project에서 받을 경우와 두개의 project에 각각 하나씩
    partition을 담당시켰을 떄 메시지를 읽는 시간을 비교해볼 계획이다.
