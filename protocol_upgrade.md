# Protocol Upgrade Guide
This document is available in [Korean](protocol_upgrade.ko.md) also.

The intended audience of this document are all entities running an AMO
blockchain node either validator node or not. This document assumes the reader
posesses a already running AMO blockchain node. This document explains the
steps required to be performed whenever there is a AMO protocol upgrade.

## Protocol v4 &rarr; v5 (To be executed in May 2021)
### Summary
Replace AMO blockchain node `amod v1.7.x` with `amod v1.8.3`.
### Steps when using Docker
1. pull new docker image `amolabs/amod:1.8.3`
   ```
   docker pull amolabs/amod:1.8.3
   ```
1. restart `amod` docker container
   ```
   docker stop amod
   docker rm amod
   docker run -d -name amod -v <data_root>/amo:/amo amolabs/amod:1.8.3
   ```
   replace `<data_root>` with the actual data root as an absolute path
1. check execution status
   ```
   docker logs -f --tail 10 amod
   ```
   confirm that the log message are generated continuously, and press
   &lt;CTRL-C&gt; to stop viewing logs

### Steps when using custom-built binary
1. Install necessary libraries (snappy, cleveldb, rocksdb)
   * ubuntu
	  ```
	  sudo apt-get install libsnappy-dev libleveldb-dev librocksdb-dev
	  ```
   * centos
      ```
	  sudo yum install snappy-devel leveldb-devel
	  wget https://github.com/facebook/rocksdb/archive/v5.18.4.zip
	  unzip v5.18.4.zip
	  cd rocksdb-5.18.4
	  make shared_lib && sudo make install-shared
	  ```
1. checkout source code from https://github.com/amolabs/amoabci
   ```
   # when there is source code checked out previously
   cd <amoabci_root>
   git remote update
   git checkout v1.8.3
   # when checking out for the first time
   git clone https://github.com/amolabs/amoabci
   cd amoabci
   git checkout v1.8.3
   ```
1. compile the binary according to
   https://github.com/amolabs/amoabci/blob/master/README.md
   ```
   make build_c
   ```
1. restart `amod`
   ```
   killall amod
   ./amod run --home <data_root>/amo
   ```

## Protocol v3 &rarr; v4 (Executed in Aug 2020)
TBA
