## 1. install-go.sh

```shell
#!/bin/bash
# for linux
set -x
set -e

# default version: 1.10.3
VERSION=$1
VERSION=${VERSION:-1.10.3}
GOROOT="/usr/local/go"
GOPATH=$HOME/gopath

# download and install
wget https://dl.google.com/go/go${VERSION}.linux-amd64.tar.gz
tar -C /usr/local -xzf go${VERSION}.linux-amd64.tar.gz

# set golang env
cat >> $HOME/.bashrc << EOF 
# Golang env
export GOROOT=/usr/local/go
export GOPATH=\$HOME/gopath
export PATH=\$PATH:\$GOROOT/bin:\$GOPATH/bin
EOF

source $HOME/.bashrc

# mkdir gopath
mkdir -p $GOPATH/src $GOPATH/pkg $GOPATH/bin
```

## 2. 安装

```shell
chmod +x install-go.sh
./install-go.sh 1.10.3
```

> 更多版本号可参考：https://golang.org/dl/

参考：
 - https://golang.org/doc/install
