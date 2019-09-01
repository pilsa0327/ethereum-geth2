# geth를 활용한 이더리움 네트워크 구축2
[참고] 링크-> [도커문법 tutorial] https://github.com/pilsa0327/docker-tutorial  
[참고] 링크-> [geth를 활용한 이더리움 네트워크 구축2] https://github.com/pilsa0327/ethereum-geth1  

## geth를 이용한 멀티노드 운영


### A. 단일 컨테이너에서 멀티노드 운영

#### 1. 컨테이너 및 계정 생성  
`$ docker run -it --name ethereum.single.geth.com -p 8545:8545 -p 30303:30303 pjt3591oo/ethereum-geth:1.90 /bin/bash`  
-> ethereum.single.geth.com 이름을 가진 컨테이너 생성  
`$ mkdir node1`  
`$ mkdir node2`  
-> node1, node2 이름의 노드가 될 디렉터리 생성  
`$ cd node1`  
`$ geth --datadir $PWD account new`  
-> 현재 디렉터리(node1)에서 계정 생성

#### 2. 제네시스 파일 생성   
node1 디렉터리에 제네시스 파일 생성(singlecontainer_genesis.json)  
-> alloc 에 초기 이더 할당을 위해 앞서 생성한 계정 추가  
`$ cp singlecontainer_genesis.json /node2/`  
-> node2 디렉터리에 node1 디렉터리에 있는 제네시스 파일 복사  

#### 3. 노드1 구동  
`$ docker exec -it ethereum.single.geth.com /bin/bash # 컨테이너 실행`  
`$ cd node1   # 노드1 디렉터리 이동`      
`$ geth -datadir $PWD init singlecontainer_genesis.json`  
`$ geth --networkid 1234 --datadir $PWD --rpc --rpcport 8545 --rpcaddr "0.0.0.0" --rpccorsdomain "*" --rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" --port 30303 console`  
-> 제네시스 블록 생성 및 노드1 구동  

cf) 옵션
- rpcport : 추후 web3나 외부 지갑 프로그램에 연동 할 때 사용되는 포트
- port : 노드끼리 연결할 때 사용하는 포트 
-> 하나의 컨테이너에서 동작하기 때문에  노드2 구동시 중복되면 안됨.  
단, networkid 는 같아야함
- rpccorsdomain : 자신의 노드에 RPC로 접속할 IP를 지정 "", "*" 값은 모든 IP 접속 가능
- rpcapi : RPC에서 사용가능한 모듈 지정

#### 4. 노드2 구동
`새 터미널 창 실행`  
`$ docker exec -it ethereum.single.geth.com /bin/bash # 컨테이너 실행`  
`$ cd node2 #   노드2 디렉터리 이동`  
`$ geth -datadir $PWD init singlecontainer_genesis.json`  
`$ geth --networkid 1234 --datadir $PWD --rpc --rpcport 8546 --rpcaddr "0.0.0.0" --rpccorsdomain "*" --rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" --port 30304 console`  
-> 제네시스 블록 생성 및 노드2 구동  

#### 5. 연결  
`$ net`  
-> net 모듈을 이용해 연결 확인 가능  
-> peerCount : 현재 노드에 연결된 노드의 수  
`$ admin`  
-> admin 모듈을 통해 노드의 정보를 확인할 수 있음  
-> 여기서 nodeInfo의 enode 값과 admin.addPeer() 모듈을 통해 노드들을 연결시킬 수 있음  
`$ cd node2`  
`$ admin.addPeer(노드1의 enode값)`  
`$ net`  
-> 노드2에서 노드1로 연결시도  
-> enode 값을 전달할 떄는 그대로 넘기지 않고 주소 부분만 변경(로컬에서는 127.0.0.1:30303)  
-> 양쪽 노드에서 net 모듈을 통해 peerCount가 1로 바뀐 것을 확인 가능  

cf) 현재 키스토어 파일은 노드1에만 있음에도, 노드2에서 동일하게 계정 조회 가능  
-> 다만 노드 1에서만 키스토어 파일이 있기 때문에, 노드1에서만 계정 잠금 혹은 트랜잭션을 발생할 수 있음  

cf) 다수의 노드 구축시 일일히 admin.addPeer()를 입력하지 않고 static-nodes.json 파일을 만들어서,   
각 노드의 enode 값을 넣고, 각 노드 디렉터리에서 keystore 디렉터리와 같은 위치에 위치에 두고 실행하면 자동으로 연결함.


### B. 다중 컨테이너에서 멀티노드 운영

#### 1. 컨테이너 생성  
`$ docker run -it --name ethereum.multi1.geth.com -p 8545:8545 -p 30303:30303 pjt3591oo/ethereum-geth:1.90 /bin/bash`    
`# 컨테이너1 생성`      
`$ docker run -it --name ethereum.multi2.geth.com -p 8546:8545 -p 30304:30303 pjt3591oo/ethereum-geth:1.90 /bin/bash`  
`# 컨테이너2 생성`  

#### 2. 제네시스 파일 생성  
각 컨테이너에의 node 디렉토리 안에 multicontainer_genesis.json 파일 생성  

#### 3. 각 노드 구동  
노드 1 구동  
`$ geth -datadir $PWD init multicontainer_genesis.json`  
`$ geth --networkid 1234 --datadir $PWD --rpc --rpcport 8545 --rpcaddr "0.0.0.0" --rpccorsdomain "*" --rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" --port 30303 console`  

노드2 구동  
`$ geth -datadir $PWD init multicontainer_genesis.json`  
`$ geth --networkid 1234 --datadir $PWD --rpc --rpcport 8546 --rpcaddr "0.0.0.0" --rpccorsdomain "*" --rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" --port 30304 console`   
-> 각 컨테이너의 node 디렉토리로 이동 후 실행  

#### 4. 연결
단일 컨테이너에서 멀티노드 운영할 떄의 방법과 동일  
-> admin 모듈을 통해 노드의 enode 값을 가져온 후, admin.addPeer()을 이용해 연결 시도  


### C. 도커컴포즈를 이용하여 다수 컨테이너 관리  
멀티노드를 운영할 컨테이너를 일일히 생성할 시, 노드의 수가 많아질수록 비효율적  
-> 이런 문제를 해결하기위해 도커컴포즈를 이용  

`# genesis2.json 과 docker-compose.yaml 파일을 같은 디렉토리에 둠`  
`$ docker-compose -f [docker-compose.yaml 파일] up -d`  
-> docker-compose로 컨테이너를 띄울 때, docker-compose.yaml과 같은 파일 이름일 경우  
`$ docker-compose up -d` 로 실행 가능

-> 그 이후 과정은 이전과 동일  
-> 모든 컨테이너를 없애고 싶을 경우  
`$ docker-compose down`  

cf) 노드의 IP와 RPC PORT를 알고 있다면 geth 의 attach 명령어를 이용하면   
원격 시스템에서 특정 노드에 접속 가능










