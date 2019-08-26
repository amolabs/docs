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

## 환경 준비
### 서버 머신
Validator 노드를 실행하기 위해서는 인터넷 연결이 안정적인 물리적인 서버나
클라우드 서비스(Amazon AWS, Google cloud, Microsoft Azure 또는 유사한 서비스들)
상의 가상머신을 준비해야 한다. 이 가이드에서는 흔히 쓰이는 우분투 리눅스가
서버에 설치돼 있다고 가정한다.

### 필요한 패키지 설치
서버의 콘솔에 접속하여 root 권한으로 Docker를 설치한다:
```bash
sudo apt install docker.io
```

### 준비
amoabci 문서에서
[실행준비](https://github.com/amolabs/amoabci#prepare-for-launch) 섹션을
참조하여 정보를 모으고 데이터 디렉토리를 준비한다. 이 문서에서는 다음을
가정한다:
- 데이터 디렉토리가 `/mynode`에 있음
- `config.toml`이 `/mynode/tendermint/config/config.toml`에 있음
- `genesis.json`이 `/mynode/tendermint/config/genesis.json`에 있음

### Validator 키 준비
다음 명령을 통해 validator 키를 생성한다:
```bash
docker -it --rm -v /mynode/tendermint:/tendermint:Z -v /mynode/amo:/amo:Z amolabs/amod:latest tendermint init
```
위 명령으로 노드 키와 validator 키가 생성된다.
`/mynode/tendermint/config/priv_validator_key.json` 파일을 다른 안전한 장소에
백업해 놓는다. Validator 키가 다음과 같다고 가정한다:
```
{
	"address": "9A8F09C644941B5A526B19641A3D7C8805E312B9",
	"pub_key": {
		"type": "tendermint/PubKeyEd25519",
		"value": "+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM="
	}
}
```

## Validator 노드 실행
데몬들을 실행하기 위해 다음 명령을 수행한다:
```bash
docker run -it --rm -p 26656-26657 -v /mynode/tendermint:/tendermint:Z -v /mynode/amo:/amo:Z --name mynode -d amolabs/amod:latest
```
잠시 docker container의 출력을 살펴서 데몬들이 정상 동작하는지 확인한다.
```bash
docker logs -f mynode
```

다음 명령을 실행해서 노드의 상태를 확인한다:
```bash
apt install curl
curl localhost:26657/status
```
출력에서 `"sync_info"` 부분을 확하여 블록체인 네트워크의 다른 노드들과의 동기화
과정이 제대로 수행되고 있는지 확인한다. `"catching_up"`이 `true`일 경우
블록체인의 정보를 계속 수신하고 있는 중인 것이다. 그러니 동기화 과정이 끝날
때까지 기다린다.

## Stake 생성
코인을 stake하기 위해서는 AMO 클라이언트가 필요하다.
```bash
apt install golang
go get github.com/amolabs/amo-client-go/cmd/amocli
```

이 문서에서는 계정 키(amocli 사용자명 중 `myval`)와 충분한 AMO 코인을 확보한
상태라고 가정한다. 이제 `stake` 거래를 블록체인에 전송해야 한다. `stake` 거래를
위해서는 validator 공개키가 필요하다. 이 거래를 전송하는 행위는 이 공개키가
로드되어 실행되고 있는 validator 노드에 대해서 당신이 제어권을 행사하고 있다고
세상에 선언하는 의미가 있다. docker로 실행된 노드의 validator 공개키를 알아내기
위해서는 다음 명령을 실행한다:
```bash
docker exec -it <container_name> tendermint show_validator
```
이 명령은 [Prepare validator key](#prepare-validator-key) 섹션에서 본 것과 같은
형태로 validator 공개키를 표시한다. 이 공개키를 알아내는 작업을 자동화 스크립트
등에서 수행하려는 경우 다음과 같이 할 수 있다:
```bash
docker exec -it <container_name> tendermint show_validator | python -c "import sys, json; print json.load(sys.stdin)['value']"
```

이제 validator 공개키가 준비되었다. 이 공개키에 대해 100 AMO를 stake하려면
다음과 같이 명령한다:
```bash
amocli tx stake --user myval '+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM=' 100000000000000000000
```

모든 validator의 목록과 그들의 voting power등을 확인하기 위해서는 다음 명령을
실행한다:
```bash
curl localhost:26657/validators
```
