# Validator 노드의 빠른 실행 가이드
This document is available in [English](qs_val.md) also.

## 소개
이 가이드는 validator 노드를 준비하고 실행하기 위한 방법을 기술한다. 이
가이드는 [amoabci README](https://github.com/amolabs/amoabci/README.md)에
설명된 방법을 활용한 것이다.

## 화폐 자산
블록체인 노드가 합의 과정에 제대로 참여하기 위해서는 stake가 있어야 한다.
따라서, validator 노드를 운영하기 위해서는 사전에 AMO 코인을 충분히 확보하는
것이 필요하다. 테스트넷의 경우에는 다양한 경로로 AMO Labs에 요청하여 validator
노드를 위한 충분한 양의 AMO 코인을 얻을 수 있다. 그러나 메인넷의 경우 필요한
AMO 코인을 획득하는 것은 전적으로 사용자에게 달려 있다. 이 가이드는 메인넷을
위해 어떻게 코인을 확보할 수 있는지는 설명하지 않는다. 테스트넷과 메인넷 모두
`stake` 거래를 전송(이 가이드의 가장 마지막 단계)하기 전에 코인을 확보해 놓아야
한다.

## 서버 환경 준비
### 서버 머신
Validator 노드를 실행하기 위해서는 인터넷 연결이 안정적인 물리적인 서버나
클라우드 서비스(Amazon AWS, Google cloud, Microsoft Azure 또는 유사한 서비스들)
상의 가상머신을 준비해야 한다. 이 가이드에서는 흔히 쓰이는 Ubuntu Linux나
MacOS가 서버에 설치돼 있다고 가정한다.

### 필요한 패키지 설치
서버의 터미널에 접속하여 root 권한으로 `git`, `wget` 그리고 `curl`을 설치한다:
```bash
sudo apt install git wget curl
```

### `amod` 데몬 설치
amoabci 문서의 [Getting
Started](https://github.com/amolabs/amoabci#getting-started) 섹션을 참조하여
컴파일된 바이너리 혹은 소스파일을 이용하여 `amod` 데몬을 설치한다.

### 설정 스크립트 설치
Validator 노드 설정을 도와주는 스크립트를 사용하기 위해서는 서버의 터미널에
접속하여 다음 명령을 실행한다:
```bash
cd $HOME
git clone https://github.com/amolabs/testnet
cd testnet
```
해당 스크립트는 데이터 디렉토리를 준비하고, `config.toml` 파일을 생성하고,
`systemd`에 `amod.service`를 등록하고, 데이터 디렉토리에 필요한 파일들을
복사해준다. 만약 당신이 `node_key.json` 과 `priv_validator_key.json` 파일을
가지고 있다면, 그것들을 `testnet` 디렉토리 아래에 위치시킨다. 해당 파일들은
데이터 디렉토리에 자동으로 복사된다. 만약 가지고 있지 않다면, 스트립트가 당신을
위하여 해당 파일들을 생성한다.

## Mainnet/Testnet에서 실행
### 노드 정보

| | `node_id` | `node_ip_addr` | `node_p2p_port` | `node_rpc_port` |
|-|-|-|-|-|
| mainnet | `f5123e0f663fe8e0662b82de8f6a1d843a9d4fbd` | `172.104.88.12` | `26656` | `26657` |
| testnet | `a944a1fa8259e19a9bac2c2b41d050f04ce50e51` | `172.105.213.114` | `26656` | `26657` |

### `genesis.json` 얻기
`genesis.json` 파일을 얻기 위하여 다음 명령을 실행한다.
```bash
cd testnet
wget <node_ip_addr>:<node_rpc_port>/genesis -O genesis.json
```
[노드 정보](#노드-정보) 섹션을 참조하여, 당신이 어떠한 네트워크에 접속하고
싶은지에 따라, 알맞은 `node_ip_addr`과 `node_rpc_port`를 입력한다.

예를 들어, **mainnet**을 위한 `genesis.json` 파일을 다운로드 하기 위해서는 다음
명령을 실행한다:
```bash
cd testnet
wget 172.104.88.12:26657/genesis -O genesis.json
```

### 설정
먼저, 데이터 디렉토리 위치를 선정하고, 현재 서버의 외부 ip 주소와 seed 노드의
정보를 파악한다. 그리고, 다음 명령을 실행한다: 
```bash
sudo ./setup.sh -e <ext_ip_addr> <data_root> <moniker> <node_id>@<node_ip_addr>:<node_p2p_port>
```
[노드 정보](#노드-정보) 섹션을 참조하여, 당신이 어떠한 네트워크에 접속하고
싶은지에 따라, 알맞은 `ext_ip_addr`, `data_root`, `moniker`(노드 이름) 그리고
`node_*` 정보를 입력한다.

예를 들어, 현재 서버의 외부 ip 주소가 `123.456.789.0`(불가능 ip 주소)이고
데이터 디렉토리가 `/mynode`이고 노드 이름이 `mynodename`이고 당신이 mainnet에
접속하고 싶다면, root 권한으로 다음 명령을 실행한다: 
```bash
sudo ./setup.sh -e 123.456.789.0 /mynode mynodename f5123e0f663fe8e0662b82de8f6a1d843a9d4fbd@172.104.88.12:26656
```

### 키 백업
`/mynode/amo/config/priv_validator_key.json` 파일을 안전한 곳에
보관한다.

### 노드 실행
노드를 실행하기 위하여 root 권한으로 다음 명령을 실행한다:
```bash
sudo systemctl start amod
```

### 노드 상태 확인
노드 상태를 확인하기 위하여 root 권한으로 다음 명령을 실행한다:
```bash
sudo systemctl status amod
```

### 노드 정지
노드를 정기하기 위하여 root 권한으로 다음 명령을 실행한다:
```bash
sudo systemctl stop amod
```

### 정보 수집
다음 명령으로 노드의 validator 주소와 공개키를 확인하여 편리한 곳에 기록해
둔다.
```bash
curl localhost:26657/status
```
<p align="center"><img src="images/node_status.png"/></p>

출력에서 `"sync_info"` 부분을 확하여 블록체인 네트워크의 다른 노드들과의 동기화
과정이 제대로 수행되고 있는지 확인한다. `"catching_up"`이 `true`일 경우
블록체인의 정보를 계속 수신하고 있는 중인 것이다. 그러니 동기화 과정이 끝날
때까지 기다린다.

## Stake 생성
**NOTE:** 메인넷인 경우는 아래의 방법 등과 같이 보다 통제된 방법을 사용해야
하지만, 테스트넷인 경우는 <a href="http://explorer.amolabs.io/wallet">AMO
블록체인 탐색기</a>에 접속하여 안내에 따른다.

코인을 stake하기 위해서는 `amocli`(AMO 클라이언트)가 필요하다. amo-client-go
문서에서 [설치](https://github.com/amolabs/amo-client-go#installation) 섹션을
참조하여 적절한 방법으로 `amocli`를 설치한다. 

이 문서에서는 계정 키(amocli 사용자명 중
`myval`, 지갑 주소 `D2CC7F160874AF06027A09DC0E8DC67E85E6D704`)와 충분한 AMO
코인을 확보한 상태라고 가정한다. 이제 `stake` 거래를 블록체인에 전송해야 한다.
`stake` 거래를 위해서는 validator 공개키가 필요하다. 이 거래를 전송하는 행위는
이 공개키가 로드되어 실행되고 있는 validator 노드에 대해서 당신이 제어권을
행사하고 있다고 세상에 선언하는 의미가 있다. validator 공개키를 알아내기
위해서는 다음 명령을 실행한다:
```bash
amod --home <data_root>/amo tendermint show_validator
```
알맞은 `data_root`를 입력한다.

예를 들어, 데이터 디렉토리가 `/mynode`이면 다음 명령을 실행한다:
```bash
amod --home /mynode/amo tendermint show_validator
```

Validator 키가 다음과 같다고 가정한다:
```json
{
  "address": "9A8F09C644941B5A526B19641A3D7C8805E312B9",
  "pub_key": {
    "type": "tendermint/PubKeyEd25519",
    "value": "+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM="
  }
}
```

이제 validator 공개키가 준비되었다. 이 공개키에 대해 1000000 AMO를 stake 하기
위해서 다음 명령을 실행한다:
```bash
amocli tx --user myval stake '+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM=' 1000000000000000000000000 
```

1000000 AMO가 제대로 stake 되었는지 확인하기 위하여 다음 명령을 실행한다:
```bash
amocli query stake 'D2CC7F160874AF06027A09DC0E8DC67E85E6D704'
```

모든 validator의 목록과 그들의 voting power등을 확인하기 위해서 다음 명령을
실행한다:
```bash
curl localhost:26657/validators
```
