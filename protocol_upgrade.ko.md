# 프로토콜 업그레이드 가이드
이 문서의 대상은 validator 여부와 상관 없이 AMO blockchain node를 운영하는 모든
주체이다. 이 문서에서는 독자가 이미 운영중인 AMO blockchain node를 보유하고
있다고 가정한다. 이 문서는 AMO protocol이 업그레이드 될 때마다 수행해야 하는
작업들을 기술한다.

## Protocol v4 &rarr; v5 (2021년 5월에 실행 예정)
### 요약
AMO blockchain node `amod v1.7.x`를 `amod v1.8.3`으로 교체한다.
### Docker 사용시 작업 내용
1. 새로운 docker image `amolabs/amod:1.8.3`을 다운로드
   ```
   docker pull amolabs/amod:1.8.3
   ```
1. docker container 재시작
   ```
   docker stop amod
   docker rm amod
   docker run -d -name amod -v <data_root>/amo:/amo amolabs/amod:1.8.3
   ```
   `<data_root>`는 실제 데이터 디렉토리를 절대경로로 입력한다
1. 실행 상태 확인
   ```
   docker logs -f --tail 10 amod
   ```
   로그 메시지가 지속적으로 생성되는 것을 확인한 후 &lt;CTRL-C&gt;를 눌러 로그
   보기를 종료

### 직접 컴파일한 바이너리 사용시 작업 내용
1. https://github.com/amolabs/amoabci 로부터 source code를 체크아웃
   ```
   # 이미 다운로드 받은 소스코드가 있는 경우
   cd <amoabci_root>
   git remote update
   git checkout v1.8.3
   # 처음 다운로드 받는 경우
   git clone https://github.com/amolabs/amoabci
   cd amoabci
   git checkout v1.8.3
   ```
1. https://github.com/amolabs/amoabci/blob/master/README.ko.md 문서를 참고하여
   컴파일

## Protocol v3 &rarr; v4 (2020년 8월에 실행)
TBA
