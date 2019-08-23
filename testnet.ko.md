# AMO 테스트넷
AMO 테스트넷의 목적은 실제 금전 자산을 소비하지 않으면서도 AMO 블록체인의
기능들을 활용해 보는 것이다. 메인넷과 테스트넷 사이에는 이런 차이 외에 다른
기능상의 차이는 없다. 주요한 차이점은 다음과 같다:
- AMO 테스트넷의 모든 데이터는 주기적으로 삭제되고 새로운 초기상태로부터
  블록체인이 다시 생성되기 때문에, 테스트넷에서의 AMO 코인은 실제 세상에서의
  금전 자산과 같은 가치를 갖지 않는다.
- 어떤 사용자든 *faucet 서버*로부터 *공짜로* AMO 코인을 얻을 수 있다.

## 테스트 가이드
### AMO 블록체인 탐색기
[block explorer](http://explorer.amolabs.io) 웹사이트에서 AMO 블록체인
테스트넷의 내부 데이터들을 탐색할 수 있다: 블록들의 목록, 블록 내의 거래 목록,
개별 거래 정보, 그리고 AMO 블록체인 기타 내부 상태 정보를 볼 수 있다.

또한, 탐색기 사이트의 [데모 페이지](http://explorer.amolabs.io/demo)에서
테스트넷의 각종 기능을 자유롭게 테스트해 볼 수 있다.

### 클라이언트 프로그램
AMO 블록체인 탐색기의 데모 페이지와 같이 제한된 환경을 활용하는 것 외에 AMO
블록체인 네트워크와 직접 상호작용할 수도 있다. [AMO client RPC
specification](https://github.com/amolabs/docs/blob/master/rpc.md)을 준수하는
소프트웨어 프로그램이나 하드웨어 장치를 AMO 클라이언트라 부른다. AMO 블록체인
탐색기도 특별한 종류의 AMO 클라이언트이다. AMO Labs에서는 범용으로 사용 가능한
[`amo-client-go`](https://github.com/amolabs/amo-client-go)라는 CLI 프로그램을
제공하며, 이를 통해 사용자가 원하는 방식으로 테스트넷과 같은 AMO 블록체인에
접근할 수 있다.

`amo-client-go`를 통해 AMO 클라이언트로 참여하기 위해서는 AMO 클라이언트의
소스코드를 AMO Labs의 github
[저장소](https://github.com/amolabs/amo-client-go)에서 다운로드 받는다. AMO
클라인트 CLI를 다운로드하고 컴파일하는 방법에 대한 자세한 사항은
[README.md](https://github.com/amolabs/amo-client-go/blob/master/README.md)를
참조한다.

AMO CLI 프로그램(`amocli`)를 설치하고 나면 AMO 블록체인 노드 중 하나에 다음과
같은 방식으로 접속할 수 있다:
```bash
amocli --rpc <rpc_node_ip:port> <command> <args>
```
로컬 호스트에서 실행되고 있는 노드에 접속할 때에는 `--rpc` 옵션을 생략할 수
있다. `amocli`의 자세한 사용법에 대해서는
[README.md](https://github.com/amolabs/amo-client-go/blob/master/README.md)를
참조한다.

### Validator 노드 실행
Validator 노드로 참여하기 위해 validator 노드를 준비하고 실행하는 방법은 별도의
[빠른 실행 가이드](https://github.com/amolabs/docs/blob/master/qs_val.md)를
참조한다.

### Non-validator 노드 실행
Non-validator 노드로 참여하기 위해서는 
[README.md](https://github.com/amolabs/testnet/blob/master/README.md)를
참조한다.

## 네트워크 정보
### AMO 블록체인 노드
테스트넷은 최초에 6개의 validator 노드들과 하나의 non-validator 노드로
시작한다(이는 나중에 변경될 수 있음):
- seed: 172.104.88.12
- validator 1: 139.162.116.176 (genesis validator)
- validator 2: 96.126.125.11
- validator 3: 139.162.180.153
- validator 4: 172.104.82.146
- validator 5: 45.33.24.148
- validator 6: 172.105.64.192

각각의 노드는 2 개의 포트를 연다: 포트 26656은 P2P 연결, 포트 26657은 RPC
연결에 사용된다. 직접 블록체인 노드를 실행하는 경우 seed 노드에 연결할 것을
권장한다.

### AMO 스토리지 서비스
현재 실행되고 있는 AMO 기본 스토리지 서비스는 다음과 같다.
- http://139.162.111.178:5000

### P2P 연결
모든 AMO 블록체인 노드는 *node_id*를 가지고 있으며 어떤 노드에 P2P 연결을
하고자 할 때는 이 *node_id*를 같이 명시해야 한다. 테스트넷의 경우 매 노드들의
노드 ID는 새로운 체인이 시작할 때 바뀔 수 있다. 이때 어떤 노드의 *노드 ID*를
알아내려면 다음과 같이 RPC 질의를 하면 된다:
```bash
curl http://<ip>:26657/status
```
`result.node_info.id`를 확인한다. `config.toml` 파일에 연결할 노드의 주소를
적을 때는 `<node_id>@<ip>:26656`와 같은 형식으로 적는다:
```toml
[p2p]
psersistent_peers = "node address"
```

### RPC 연결
모든 AMO 클라이언트는 AMO 블록체인과 상호작용하기 위해 RPC 연결점이 필요하다.
RPC 서버 주소는 `http://<ip>:26657`와 같이 지정한다. 단, `amocli`의 `--rpc`
플래그에 사용할 때는 `http://` 접두사 없이 `<ip>:26657` 만을 지정한다.

### AMO 스토리지 연결
`amocli`의 `--sto` 플래그에 스토리지 연결점을 지정할 때는 `http://` 접두사 없이
`<ip>:5000` 만을 지정한다.

### Genesis 파일
모든 AMO 블록체인 노드는 RPC 연결을 통해 자신이 사용한 genesis 파일을 알려준다.
다음과 같이 해서 `genesis.json` 파일을 얻을 수 있다:
```bash
curl http://<ip>:26657/genesis
```
이 결과물을 `genesis.json` 파일로 저장하여 사용한다.
