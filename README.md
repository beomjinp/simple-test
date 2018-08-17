Test
===
이더리움 계좌의 잔고를 확인하는 간단한 어플리케이션을 작성한다. 
잔고 확인을 위해 이더리움 라이트노드를 직접 올리고, 잔고 정보를 관리하는 간단한 API서버도 개발한다.
안드로이드 앱에선 계좌를 등록하고 잔고를 확인한다.

# 서버 요구사항
- aws 프리티어 기본셋팅. ubuntu 추천.
- geth를 통해 계정의 잔고와 거래개수를 조회한다. geth 가이드문서 제공(문서 하단 참조)

# api 서버 앱 요구사항.
- 계정정보를 등록한다.
- 등록된 계정의 잔고, 거래개수를 매 10초마다 업데이트한다.
- 주소별로 잔고와 거래개수를 조회할 수 있다.  외부로 노출하는 http-api는 이것 하나이다.
## 선택사항
- 데이터는 디스크에 기록하지 않아도 된다. 그러므로 굳이 DBMS를 사용할 필요는 없다.
- 아무나 접속할 수 없게 간단한 인증을 추가한다.

# 안드로이드 앱 요구사항.
- UI 설계. https://ovenapp.io/view/nnkvygcnktvBOXFU8U1AW5lsoQX7m3Tq/JVwn1
- 첫 화면에서 등록된 계좌의 정보를 서버로부터 업데이트한다.
- 주소와 이름을 입력해 계좌를 등록할 수 있다.
- 계좌를 선택하면 잔고와 거래개수가 포함된 상세내역을 조회할 수 있다.
## 선택사항.
- 상세내역에선 최근 거래내역도 확인할 수 있다. https://rinkeby.etherscan.io/apis 참고.
- 아래 기술들을 사용하라.
- http://square.github.io/retrofit/
- https://developer.android.com/topic/libraries/architecture/
- https://github.com/ReactiveX/RxJava
- https://google.github.io/dagger/

# 제출
- 완료 후 서버 소스와 안드로이드 앱 소스가 있는 깃헙 주소를 공유한다.

# 문의
기술적 문의를 제외한 요구사항에 대한 문의는 다음 이메일로 부탁드립니다. smoh@pentasecurity.com

geth 가이드.
====
## 설치
https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu

## 실행
* --rinkeby 는 테스트넷으로 접속하는 옵션이다.
* --light 풀노드가 아닌 라이트노드로 동작하는 옵션이다. aws 프리티어 ec2의 경우 기본 저장공간이 8GB밖에 안되니 풀노드 동작은 불가능하다. 실행시 꼭 light노드로 실행해야한다.
* --rpc JsonRPC를 활성화하는 옵션이다. 잔고 조회, 거래개수 조회에 필요하다.
### 포어그라운드에서 콘솔로 붙을 때.
```
$ geth --rinkeby --light --rpc console
```
### 백그라운드로 실행시킬 때.
```
$ nohup geth --rinkeby --light --rpc &
```

## 블록임포트 확인
초기 실행 후 블록 임포트가 완료될 때 까지 약 30분에서 1시간정도 소요된다.
최신블록은 https://rinkeby.etherscan.io/ 에서 확인할 수 있다.
```
INFO [07-23|03:16:00.769] Block synchronisation started 
INFO [07-23|03:16:02.100] Imported new block headers               count=192 elapsed=77.684ms number=192 hash=8c570c…ba360c ignored=0
INFO [07-23|03:16:02.344] Imported new block headers               count=192 elapsed=78.181ms number=384 hash=6d95fa…a59e49 ignored=0
..........
INFO [07-23|03:53:52.426] Imported new block headers               count=2048 elapsed=9.041s    number=2682624 hash=e6ee7d…3a49c9 ignored=0
INFO [07-23|03:53:55.095] Imported new block headers               count=448  elapsed=2.220s    number=2683072 hash=423939…f173c8 ignored=0
INFO [07-23|03:53:55.755] Imported new block headers               count=119  elapsed=495.997ms number=2683191 hash=4d18b1…637c54 ignored=0
INFO [07-23|03:53:55.796] Imported new block headers               count=1    elapsed=501.277µs number=2683192 hash=479c9d…b04538 ignored=0
INFO [07-23|03:53:55.877] Imported new block headers               count=3    elapsed=1.235ms   number=2683195 hash=07b879…c4aa01 ignored=0
```

## 테스트
### 거래개수 정상출력.
```
$ curl http://127.0.0.1:8545 -X POST --data '{"id":7,"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0x47071CDF9615df13ae128C1915487390d48DdED6","latest"]}' -H "Content-Type: application/json" 
{"jsonrpc":"2.0","id":7,"result":"0xc3"}
```

### 잔고 정상출력.
```
$ curl http://127.0.0.1:8545 -X POST --data '{"id":7,"jsonrpc":"2.0","method":"eth_getBalance","params":["0x47071CDF9615df13ae128C1915487390d48DdED6","latest"]}' -H "Content-Type: application/json" 
{"jsonrpc":"2.0","id":7,"result":"0x379f5b5e6ff1b24"}
```

### 비정상 출력.
geth에 연결된 peer가 없거나, 네트워크 문제 등 다양한 이상상황에선 다음과 같은 에러메시지가 출력될 수 있다.
peer가 없는 경우엔 시간을 두고 기다린 후 다시 시도해본다.
```
$ curl http://127.0.0.1:8545 -X POST --data '{"id":7,"jsonrpc":"2.0","method":"eth_getBalance","params":["0x47071CDF9615df13ae128C1915487390d48DdED6","latest"]}' -H "Content-Type: application/json" 
{"jsonrpc":"2.0","id":7,"error":{"code":-32000,"message":"no suitable peers available"}}
```
