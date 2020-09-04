# 准备

```bash
export PATH=/home/jck/fabric-network/fabric-samples/bin:$PATH

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

export COMPOSE_PROJECT_NAME=net
```

# CA

## TLS CA

```bash
mkdir -p /tmp/hyperledger/docker-compose/fabric-ca-tls && cd /tmp/hyperledger/docker-compose/fabric-ca-tls
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:
services:
   ca-tls:
     container_name: ca-tls
     image: hyperledger/fabric-ca
     command: sh -c 'fabric-ca-server start -d -b tls-ca-admin:tls-ca-adminpw --port 7052'
     environment:
       - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
       - FABRIC_CA_SERVER_TLS_ENABLED=true
       - FABRIC_CA_SERVER_CSR_CN=ca-tls
       - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
       - FABRIC_CA_SERVER_PORT=7052
       - FABRIC_CA_SERVER_DEBUG=true
     volumes:
       - /tmp/hyperledger/fabric-ca-tls:/tmp/hyperledger/fabric-ca
     networks:
       - fabric-ca
     ports:
       - 7052:7052
```

```bash
docker-compose up -d
sudo chown jck /tmp/hyperledger/
```

```bash
#设置环境变量指定根证书的路径(如果工作目录不同的话记得指定自己的工作目录,以下不再重复说明)
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem
#设置环境变量指定CA客户端的HOME文件夹
export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/fabric-ca-tls/admin
#登录管理员用户用于之后的节点身份注册
fabric-ca-client enroll -d -u https://tls-ca-admin:tls-ca-adminpw@0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem


fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem

fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem

fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem

fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem

fabric-ca-client register -d --id.name orderer1-org0 --id.secret ordererPW --id.type orderer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem

fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type admin -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem

fabric-ca-client register -d --id.name admin-org2 --id.secret org2AdminPW --id.type admin -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem
```

## Ogr0 CA

```bash
mkdir -p /tmp/hyperledger/org0/ca

mkdir -p /tmp/hyperledger/docker-compose/org0/ca && cd /tmp/hyperledger/docker-compose/org0/ca

touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:

services:

  org0:
    container_name: org0
    image: hyperledger/fabric-ca:latest
    command: sh -c 'fabric-ca-server start -d -b org0-admin:org0-adminpw --port 7053'
    environment:
      - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_CSR_CN=org0
      - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
      - FABRIC_CA_SERVER_PORT=7053
      - FABRIC_CA_SERVER_DEBUG=true
    volumes:
      - /tmp/hyperledger/org0/ca:/tmp/hyperledger/fabric-ca  ##重要！！！记得修改这里的路径为自己的工作目录
    networks:
      - fabric-ca
    ports:
      - 7053:7053
```

```bash
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org0/ca/crypto/ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org0/ca/admin

fabric-ca-client enroll -d -u https://org0-admin:org0-adminpw@0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem
```

```bash
fabric-ca-client register -d --id.name orderer1-org0 --id.secret ordererpw --id.type orderer -u https://0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem

fabric-ca-client register -d --id.name admin-org0 --id.secret org0adminpw --id.type admin --id.attrs "hf.Registrar.Roles=client,hf.Registrar.Attributes=*,hf.Revoker=true,hf.GenCRL=true,admin=true:ecert,abac.init=true:ecert" -u https://0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem
```


## Org1 CA

```bash
mkdir -p /tmp/hyperledger/org1/ca

mkdir -p /tmp/hyperledger/docker-compose/org1/ca && cd /tmp/hyperledger/docker-compose/org1/ca

touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:

services:

  org1:
    container_name: org1
    image: hyperledger/fabric-ca:latest
    command: sh -c 'fabric-ca-server start -d -b org1-admin:org1-adminpw --port 7054'
    environment:
      - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_CSR_CN=org1
      - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
      - FABRIC_CA_SERVER_PORT=7054
      - FABRIC_CA_SERVER_DEBUG=true
    volumes:
      - /tmp/hyperledger/org1/ca:/tmp/hyperledger/fabric-ca  ##重要！！！记得修改这里的路径为自己的工作目录
    networks:
      - fabric-ca
    ports:
      - 7054:7054
```

```bash
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/ca/crypto/ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org1/ca/admin

fabric-ca-client enroll -d -u https://org1-admin:org1-adminpw@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem
```

```bash
fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem

fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem

fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type admin -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem

fabric-ca-client register -d --id.name user-org1 --id.secret org1UserPW --id.type client -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem
```

## Org2 CA

```bash
mkdir -p /tmp/hyperledger/org2/ca

mkdir -p /tmp/hyperledger/docker-compose/org2/ca && cd /tmp/hyperledger/docker-compose/org2/ca

touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:

services:

  org2:
    container_name: org2
    image: hyperledger/fabric-ca:latest
    command: sh -c 'fabric-ca-server start -d -b org2-admin:org2-adminpw --port 7055'
    environment:
      - FABRIC_CA_SERVER_HOME=/tmp/hyperledger/fabric-ca/crypto
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_CSR_CN=org2
      - FABRIC_CA_SERVER_CSR_HOSTS=0.0.0.0
      - FABRIC_CA_SERVER_PORT=7055
      - FABRIC_CA_SERVER_DEBUG=true
    volumes:
      - /tmp/hyperledger/org2/ca:/tmp/hyperledger/fabric-ca  ##重要！！！记得修改这里的路径为自己的工作目录
    networks:
      - fabric-ca
    ports:
      - 7055:7055
```

```bash
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/ca/crypto/ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org2/ca/admin

fabric-ca-client enroll -d -u https://org2-admin:org2-adminpw@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem
```

```bash
fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem

fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem

fabric-ca-client register -d --id.name admin-org2 --id.secret org2AdminPW --id.type admin -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem

fabric-ca-client register -d --id.name user-org2 --id.secret org2UserPW --id.type client -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem
```

# Org1

## Peer1

Org1 CA

```bash
mkdir -p /tmp/hyperledger/org1/peer1/assets/ca/
cp /tmp/hyperledger/org1/ca/crypto/ca-cert.pem /tmp/hyperledger/org1/peer1/assets/ca/org1-ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org1/peer1
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/peer1/assets/ca/org1-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem
```

TLS

```bash
mkdir -p /tmp/hyperledger/org1/peer1/assets/tls-ca
cp /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem  /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem

export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem

fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org1 --tls.certfiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem
```

修改文件名

```bash
mv /tmp/hyperledger/org1/peer1/tls-msp/keystore/*_sk /tmp/hyperledger/org1/peer1/tls-msp/keystore/key.pem
```

## Peer2

Org1 CA

```bash
mkdir -p /tmp/hyperledger/org1/peer2/assets/ca/
cp /tmp/hyperledger/org1/ca/crypto/ca-cert.pem /tmp/hyperledger/org1/peer2/assets/ca/org1-ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org1/peer2
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/peer2/assets/ca/org1-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem
```

TLS

```bash
mkdir -p /tmp/hyperledger/org1/peer2/assets/tls-ca/
cp /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem  /tmp/hyperledger/org1/peer2/assets/tls-ca/tls-ca-cert.pem

export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/peer2/assets/tls-ca/tls-ca-cert.pem

fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org1 --tls.certfiles /tmp/hyperledger/org1/peer2/assets/tls-ca/tls-ca-cert.pem
```

修改文件名

```bash
mv /tmp/hyperledger/org1/peer2/tls-msp/keystore/*_sk /tmp/hyperledger/org1/peer2/tls-msp/keystore/key.pem
```

## Admin

Org1 CA

```bash
export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org1/admin
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/peer1/assets/ca/org1-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://admin-org1:org1AdminPW@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/peer1/assets/ca/org1-ca-cert.pem
```

TLS CA

```bash
export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem

fabric-ca-client enroll -d -u https://admin-org1:org1AdminPW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts admin-org1 --tls.certfiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem
```

复制证书到 admincerts 文件夹

```bash
mkdir /tmp/hyperledger/org1/peer1/msp/admincerts
cp /tmp/hyperledger/org1/admin/msp/signcerts/cert.pem /tmp/hyperledger/org1/peer1/msp/admincerts/org1-admin-cert.pem


mkdir /tmp/hyperledger/org1/peer2/msp/admincerts
cp /tmp/hyperledger/org1/admin/msp/signcerts/cert.pem /tmp/hyperledger/org1/peer2/msp/admincerts/org1-admin-cert.pem
```

## 启动节点

### Peer1

```bash
mkdir -p /tmp/hyperledger/docker-compose/org1/peer1 && cd /tmp/hyperledger/docker-compose/org1/peer1
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:

services:
  peer1-org1:
    container_name: peer1-org1
    image: hyperledger/fabric-peer:2.0.0
    environment:
      - CORE_PEER_ID=peer1-org1
      - CORE_PEER_ADDRESS=peer1-org1:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer1-org1:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1-org1:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1-org1:7051
      - CORE_PEER_LOCALMSPID=org1MSP
      - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/peer1/msp
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_fabric-ca
      - FABRIC_LOGGING_SPEC=debug
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org1/peer1/tls-msp/signcerts/cert.pem
      - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org1/peer1/tls-msp/keystore/key.pem
      - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org1/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org1/peer1
    volumes:
      - /var/run:/host/var/run
      - /tmp/hyperledger/org1/peer1:/tmp/hyperledger/org1/peer1
    networks:
      - fabric-ca
```

### Peer2

```bash
mkdir -p /tmp/hyperledger/docker-compose/org1/peer2 && cd /tmp/hyperledger/docker-compose/org1/peer2
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:

services:
  peer2-org1:
    container_name: peer2-org1
    image: hyperledger/fabric-peer:2.0.0
    environment:
      - CORE_PEER_ID=peer2-org1
      - CORE_PEER_ADDRESS=peer2-org1:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer2-org1:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1-org1:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2-org1:7051
      - CORE_PEER_LOCALMSPID=org1MSP
      - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/peer2/msp
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_fabric-ca
      - FABRIC_LOGGING_SPEC=debug
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org1/peer2/tls-msp/signcerts/cert.pem
      - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org1/peer2/tls-msp/keystore/key.pem
      - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org1/peer2/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org1/peer2
    volumes:
      - /var/run:/host/var/run
      - /tmp/hyperledger/org1/peer2:/tmp/hyperledger/org1/peer2
    networks:
      - fabric-ca
```

# Org2

## Peer1

Org2 CA

```bash
mkdir -p /tmp/hyperledger/org2/peer1/assets/ca 
cp /tmp/hyperledger/org2/ca/crypto/ca-cert.pem /tmp/hyperledger/org2/peer1/assets/ca/org2-ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org2/peer1
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/peer1/assets/ca/org2-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/peer1/assets/ca/org2-ca-cert.pem
```

TLS

```bash
mkdir /tmp/hyperledger/org2/peer1/assets/tls-ca
cp /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem  /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem

export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem

fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org2 --tls.certfiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem
```

修改文件

```bash
mv /tmp/hyperledger/org2/peer1/tls-msp/keystore/*_sk /tmp/hyperledger/org2/peer1/tls-msp/keystore/key.pem
```

## Peer2

Org2 CA

```bash
mkdir -p /tmp/hyperledger/org2/peer2/assets/ca 
cp /tmp/hyperledger/org2/ca/crypto/ca-cert.pem /tmp/hyperledger/org2/peer2/assets/ca/org2-ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org2/peer2
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/peer2/assets/ca/org2-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/peer2/assets/ca/org2-ca-cert.pem
```

TLS

```bash
mkdir /tmp/hyperledger/org2/peer2/assets/tls-ca
cp /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem  /tmp/hyperledger/org2/peer2/assets/tls-ca/tls-ca-cert.pem

export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/peer2/assets/tls-ca/tls-ca-cert.pem

fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org2 --tls.certfiles /tmp/hyperledger/org2/peer2/assets/tls-ca/tls-ca-cert.pem
```

修改文件

```bash
mv /tmp/hyperledger/org2/peer2/tls-msp/keystore/*_sk /tmp/hyperledger/org2/peer2/tls-msp/keystore/key.pem
```

## Admin

Org2 CA

```bash
export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org2/admin
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/peer1/assets/ca/org2-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://admin-org2:org2AdminPW@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/peer1/assets/ca/org2-ca-cert.pem
```

TLS

```bash
export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem

fabric-ca-client enroll -d -u https://admin-org2:org2AdminPW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts admin-org2 --tls.certfiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem
```

复制证书到 admincerts 文件夹

```bash
mkdir /tmp/hyperledger/org2/peer1/msp/admincerts
cp /tmp/hyperledger/org2/admin/msp/signcerts/cert.pem /tmp/hyperledger/org2/peer1/msp/admincerts/org2-admin-cert.pem

mkdir /tmp/hyperledger/org2/peer2/msp/admincerts
cp /tmp/hyperledger/org2/admin/msp/signcerts/cert.pem /tmp/hyperledger/org2/peer2/msp/admincerts/org2-admin-cert.pem
```

## 启动节点

### Peer1

```bash
mkdir -p /tmp/hyperledger/docker-compose/org2/peer1 && cd /tmp/hyperledger/docker-compose/org2/peer1
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:

services:
  peer1-org2:
    container_name: peer1-org2
    image: hyperledger/fabric-peer:2.0.0
    environment:
      - CORE_PEER_ID=peer1-org2
      - CORE_PEER_ADDRESS=peer1-org2:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer1-org2:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1-org2:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1-org2:7051
      - CORE_PEER_LOCALMSPID=org2MSP
      - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/peer1/msp
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_fabric-ca
      - FABRIC_LOGGING_SPEC=debug
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org2/peer1/tls-msp/signcerts/cert.pem
      - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org2/peer1/tls-msp/keystore/key.pem
      - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org2/peer1
    volumes:
      - /var/run:/host/var/run
      - /tmp/hyperledger/org2/peer1:/tmp/hyperledger/org2/peer1
    networks:
      - fabric-ca
```

### Peer2

```bash
mkdir -p /tmp/hyperledger/docker-compose/org2/peer2 && cd /tmp/hyperledger/docker-compose/org2/peer2
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:

services:
  peer2-org2:
    container_name: peer2-org2
    image: hyperledger/fabric-peer:2.0.0
    environment:
      - CORE_PEER_ID=peer2-org2
      - CORE_PEER_ADDRESS=peer2-org2:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer2-org2:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1-org2:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2-org2:7051
      - CORE_PEER_LOCALMSPID=org2MSP
      - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/peer2/msp
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_fabric-ca
      - FABRIC_LOGGING_SPEC=debug
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org2/peer2/tls-msp/signcerts/cert.pem
      - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org2/peer2/tls-msp/keystore/key.pem
      - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org2/peer2/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
      - CORE_PEER_PROFILE_ENABLED=true
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org2/peer2
    volumes:
      - /var/run:/host/var/run
      - /tmp/hyperledger/org2/peer2:/tmp/hyperledger/org2/peer2
    networks:
      - fabric-ca
```

# Org0

## Orderer

Org0 CA

```bash
mkdir -p /tmp/hyperledger/org0/orderer/assets/ca/
cp /tmp/hyperledger/org0/ca/crypto/ca-cert.pem /tmp/hyperledger/org0/orderer/assets/ca/org0-ca-cert.pem

export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org0/orderer
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org0/orderer/assets/ca/org0-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://orderer1-org0:ordererpw@0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/orderer/assets/ca/org0-ca-cert.pem
```

TLS

```bash
mkdir /tmp/hyperledger/org0/orderer/assets/tls-ca/
cp /tmp/hyperledger/fabric-ca-tls/crypto/ca-cert.pem  /tmp/hyperledger/org0/orderer/assets/tls-ca/tls-ca-cert.pem

export FABRIC_CA_CLIENT_MSPDIR=tls-msp
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org0/orderer/assets/tls-ca/tls-ca-cert.pem

fabric-ca-client enroll -d -u https://orderer1-org0:ordererPW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts orderer1-org0 --tls.certfiles /tmp/hyperledger/org0/orderer/assets/tls-ca/tls-ca-cert.pem
```

修改文件

```bash
mv /tmp/hyperledger/org0/orderer/tls-msp/keystore/*_sk /tmp/hyperledger/org0/orderer/tls-msp/keystore/key.pem
```

## Admin

Org0 CA

```bash
export FABRIC_CA_CLIENT_HOME=/tmp/hyperledger/org0/admin
export FABRIC_CA_CLIENT_TLS_CERTFILES=/tmp/hyperledger/org0/orderer/assets/ca/org0-ca-cert.pem
export FABRIC_CA_CLIENT_MSPDIR=msp

fabric-ca-client enroll -d -u https://admin-org0:org0adminpw@0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/orderer/assets/ca/org0-ca-cert.pem
```

复制证书到 admincerts 文件夹

```bash
mkdir /tmp/hyperledger/org0/orderer/msp/admincerts
cp /tmp/hyperledger/org0/admin/msp/signcerts/cert.pem /tmp/hyperledger/org0/orderer/msp/admincerts/orderer-admin-cert.pem
```

# 添加配置文件

在 Org0，Org1, Org2 所有 msp 目录下添加 config.yaml

```yaml
NodeOUs:
  Enable: true
  ClientOUIdentifier:
    #修改对应的证书名称
    Certificate: cacerts/0-0-0-0-7053.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/0-0-0-0-7053.pem
    OrganizationalUnitIdentifier: peer
  AdminOUIdentifier:
    Certificate: cacerts/0-0-0-0-7053.pem
    OrganizationalUnitIdentifier: admin
  OrdererOUIdentifier:
    Certificate: cacerts/0-0-0-0-7053.pem
    OrganizationalUnitIdentifier: orderer
```

```bash
cd /tmp/hyperledger
touch config.yaml

echo org0/admin/msp org0/orderer/msp | xargs -n 1 cp config.yaml
echo org1/admin/msp org1/peer1/msp org1/peer2/msp | xargs -n 1 cp config.yaml
echo org2/admin/msp org2/peer1/msp org2/peer2/msp | xargs -n 1 cp config.yaml
``` 

# 网络配置

## msp

```bash
# Org0
mkdir -p /tmp/hyperledger/configtx && cd /tmp/hyperledger/configtx

mkdir org0

cp -r ../org0/admin/msp org0/

cd  org0/msp

mkdir tlscacerts && cd tlscacerts

cp  /tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem ./

# Org1
cd /tmp/hyperledger/configtx
mkdir org1 

cp -r ../org1/admin/msp org1/

cd org1/msp
mkdir tlscacerts && cd tlscacerts

cp /tmp/hyperledger/org1/admin/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem ./

# Org2
cd /tmp/hyperledger/configtx
mkdir org2 

cp -r ../org2/admin/msp org2/

cd org2/msp
mkdir tlscacerts && cd tlscacerts

cp /tmp/hyperledger/org2/admin/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem ./
```

## 创世区块和通道配置

```bash
cd /tmp/hyperledger/configtx
touch configtx.yaml
```

## 生成创世区块和通道信息

```bash
cd /tmp/hyperledger/configtx
mkdir system-genesis-block 
mkdir channel-artifacts

# 生成创世区块文件
configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block

# 生成通道
export CHANNEL_NAME=mychannel
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/${CHANNEL_NAME}.tx -channelID ${CHANNEL_NAME}

# 锚节点更新配置
export orgmsp=org1MSP
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/${orgmsp}anchors.tx -channelID ${CHANNEL_NAME} -asOrg ${orgmsp}

#锚节点更新配置
export orgmsp=org2MSP
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/${orgmsp}anchors.tx -channelID ${CHANNEL_NAME} -asOrg ${orgmsp}
```

## 启动 Orderer 节点

```bash
mkdir -p /tmp/hyperledger/docker-compose/org0/orderer && cd /tmp/hyperledger/docker-compose/org0/orderer
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:
services:
  orderer1-org0:
    container_name: orderer1-org0
    image: hyperledger/fabric-orderer:2.0.0
    environment:
      - ORDERER_HOME=/tmp/hyperledger/orderer
      - ORDERER_HOST=orderer1-org0
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/tmp/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=org0MSP
      - ORDERER_GENERAL_LOCALMSPDIR=/tmp/hyperledger/org0/orderer/msp
      - ORDERER_GENERAL_TLS_ENABLED=true

      - ORDERER_GENERAL_TLS_PRIVATEKEY=/tmp/hyperledger/org0/orderer/tls-msp/keystore/key.pem
      - ORDERER_GENERAL_TLS_CERTIFICATE=/tmp/hyperledger/org0/orderer/tls-msp/signcerts/cert.pem
      - ORDERER_GENERAL_TLS_ROOTCAS=[/tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem]

      - ORDERER_KAFKA_TOPIC_REPLICATIONFACTOR=1
      - ORDERER_KAFKA_VERBOSE=true
      - ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=/tmp/hyperledger/org0/orderer/tls-msp/signcerts/cert.pem
      - ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=/tmp/hyperledger/org0/orderer/tls-msp/keystore/key.pem
      - ORDERER_GENERAL_CLUSTER_ROOTCAS=[/tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem]

      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_DEBUG_BROADCASTTRACEDIR=data/logs
    volumes:
      - /tmp/hyperledger/org0/orderer:/tmp/hyperledger/org0/orderer/
      - /tmp/hyperledger/configtx/system-genesis-block/genesis.block:/tmp/hyperledger/orderer/orderer.genesis.block

    networks:
      - fabric-ca
```

## 启动 Cli

### Org1

```bash
mkdir -p /tmp/hyperledger/docker-compose/org1/cli && cd /tmp/hyperledger/docker-compose/org1/cli
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:
services:
    cli-org1:
      container_name: cli-org1
      image: hyperledger/fabric-tools:2.0.0
      tty: true
      stdin_open: true
      environment:
        - SYS_CHANNEL=testchainid
        - GOPATH=/opt/gopath
        - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
        - FABRIC_LOGGING_SPEC=DEBUG
        - CORE_PEER_ID=cli-org1
        - CORE_PEER_ADDRESS=peer1-org1:7051
        - CORE_PEER_LOCALMSPID=org1MSP
        - CORE_PEER_TLS_ENABLED=true
        - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org1/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
        - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org1/peer1/tls-msp/signcerts/cert.pem
        - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org1/peer1/tls-msp/keystore/key.pem
        - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/peer1/msp
      working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org1
      command: /bin/bash
      volumes:
        - /tmp/hyperledger/org1:/tmp/hyperledger/org1/
        - /tmp/hyperledger/org2:/tmp/hyperledger/org2/
        - /tmp/hyperledger/org1/peer1/assets/chaincode:/opt/gopath/src/github.com/hyperledger/fabric-samples/chaincode
        - /tmp/hyperledger/org1/admin:/tmp/hyperledger/org1/admin
        - /tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem:/tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
        - /tmp/hyperledger/org1/peer1/tls-msp/tlscacerts:/tmp/hyperledger/org1/admin/msp/tlscacerts
        - /tmp/hyperledger/configtx/channel-artifacts:/tmp/hyperledger/configtx/channel-artifacts
      networks:
        - fabric-ca
```

### Org2

```bash
mkdir -p /tmp/hyperledger/docker-compose/org2/cli && cd /tmp/hyperledger/docker-compose/org2/cli
touch docker-compose.yaml
```

```yaml
version: '2'

networks:
  fabric-ca:
services:
    cli-org2:
      container_name: cli-org2
      image: hyperledger/fabric-tools:2.0.0
      tty: true
      stdin_open: true
      environment:
        - SYS_CHANNEL=testchainid
        - GOPATH=/opt/gopath
        - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
        - FABRIC_LOGGING_SPEC=DEBUG
        - CORE_PEER_ID=cli-org2
        - CORE_PEER_ADDRESS=peer1-org2:7051
        - CORE_PEER_LOCALMSPID=org2MSP
        - CORE_PEER_TLS_ENABLED=true
        - CORE_PEER_TLS_ROOTCERT_FILE=/tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
        - CORE_PEER_TLS_CERT_FILE=/tmp/hyperledger/org2/peer1/tls-msp/signcerts/cert.pem
        - CORE_PEER_TLS_KEY_FILE=/tmp/hyperledger/org2/peer1/tls-msp/keystore/key.pem
        - CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/peer1/msp
      working_dir: /opt/gopath/src/github.com/hyperledger/fabric/org2
      command: /bin/bash
      volumes:
        - /tmp/hyperledger/org1:/tmp/hyperledger/org1/
        - /tmp/hyperledger/org2:/tmp/hyperledger/org2/
        - /tmp/hyperledger/org2/peer1:/tmp/hyperledger/org2/peer1
        - /tmp/hyperledger/org2/peer1/assets/chaincode:/opt/gopath/src/github.com/hyperledger/fabric-samples/chaincode
        - /tmp/hyperledger/org2/admin:/tmp/hyperledger/org2/admin
        - /tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem:/tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
        - /tmp/hyperledger/org2/peer1/tls-msp/tlscacerts:/tmp/hyperledger/org2/peer1/msp/tlscacerts
        - /tmp/hyperledger/configtx/channel-artifacts:/tmp/hyperledger/configtx/channel-artifacts
      networks:
        - fabric-ca
```

# 创建并加入通道

## Org1-CLI

```bash
docker exec -it cli-org1 bash

export CHANNEL_NAME=mychannel
export ORDERER_CA=/tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp

cd /tmp/hyperledger/configtx

peer channel create -o orderer1-org0:7050 -c ${CHANNEL_NAME} --ordererTLSHostnameOverride orderer1-org0 -f ./channel-artifacts/${CHANNEL_NAME}.tx --outputBlock ./channel-artifacts/${CHANNEL_NAME}.block --tls --cafile ${ORDERER_CA}


export CORE_PEER_ADDRESS=peer1-org1:7051
peer channel join -b ./channel-artifacts/mychannel.block

export CORE_PEER_ADDRESS=peer2-org1:7051
peer channel join -b ./channel-artifacts/mychannel.block


export CORE_PEER_LOCALMSPID=org1MSP
peer channel update -o orderer1-org0:7050 --ordererTLSHostnameOverride orderer1-org0 -c $CHANNEL_NAME -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors.tx --tls --cafile $ORDERER_CA
```

## Org2-CLI

```bash
docker exec -it cli-org2 bash

export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/admin/msp

export CORE_PEER_ADDRESS=peer1-org2:7051
peer channel join -b ./channel-artifacts/mychannel.block

 export CORE_PEER_ADDRESS=peer2-org2:7051
 peer channel join -b ./channel-artifacts/mychannel.block

cd /tmp/hyperledger/configtx

export CHANNEL_NAME=mychannel
export ORDERER_CA=/tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem
export CORE_PEER_LOCALMSPID=org2MSP

peer channel update -o orderer1-org0:7050 --ordererTLSHostnameOverride orderer1-org0 -c $CHANNEL_NAME -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors.tx --tls --cafile $ORDERER_CA
```

# 链码测试

## Org1-CLI

```bash
docker exec -it cli-org1 bash
cd /tmp/hyperledger/org1/peer1/assets/chaincode

export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org1/admin/msp
peer lifecycle chaincode install fabcar.tar.gz
```

## Org2-CLI

```bash
docker exec -it cli-org2 bash
cd /tmp/hyperledger/org2/peer1/assets/chaincode

export CORE_PEER_MSPCONFIGPATH=/tmp/hyperledger/org2/admin/msp
peer lifecycle chaincode install fabcar.tar.gz
```