# 1. install-go.sh

```bash
#!/bin/bash
set -x
set -e

# default version
VERSION=$1
VERSION=${VERSION:-1.14.6}

PLATFORM=$2
PLATFORM=${PLATFORM:-linux}

GOROOT="/usr/local/go"
GOPATH=$HOME/gopath
GO_DOWNLOAD_URL="https://golang.org/dl"

# download and install
case ${PLATFORM} in
"linux")
    wget ${GO_DOWNLOAD_URL}/go${VERSION}.${PLATFORM}-amd64.tar.gz
    tar -C /usr/local -xzf go${VERSION}.${PLATFORM}-amd64.tar.gz
    ;;
"mac")
    PLATFORM="darwin"
    wget ${GO_DOWNLOAD_URL}/go${VERSION}.${PLATFORM}-amd64.tar.gz
    tar -C /usr/local -xzf go${VERSION}.${PLATFORM}-amd64.tar.gz
    ;;
*)
    echo "platform not found"
    ;;
esac    

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

# 2. 安装

```bash
chmod +x install-go.sh
./install-go.sh 1.14.6 linux
```

> 更多版本号可参考：https://golang.org/dl/

参考：
 - https://golang.org/doc/install
