
# CA | Peer节点

```bash
## 搭建 TLS CA
docker-compose up -d ca-tls

## 登记 TLS CA 管理员
sudo fabric-ca-client enroll -d -u https://tls-ca-admin:tls-ca-adminpw@0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/tls-ca/admin

## 向 TLS CA 进行注册
sudo fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/tls-ca/admin
sudo fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/tls-ca/admin
sudo fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/tls-ca/admin
sudo fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/tls-ca/admin
sudo fabric-ca-client register -d --id.name orderer1-org0 --id.secret ordererPW --id.type orderer -u https://0.0.0.0:7052 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/tls-ca/admin

## 搭建 ORG CA
docker-compose up -d rca-org0 rca-org1 rca-org2

# 登记 Org0 CA 管理员
sudo fabric-ca-client enroll -d -u https://rca-org0-admin:rca-org0-adminpw@0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org0/ca/admin

# 向 Org0 CA 进行注册
sudo fabric-ca-client register -d --id.name orderer1-org0 --id.secret ordererpw --id.type orderer -u https://0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org0/ca/admin
sudo fabric-ca-client register -d --id.name admin-org0 --id.secret org0adminpw --id.type admin --id.attrs "hf.Registrar.Roles=client,hf.Registrar.Attributes=*,hf.Revoker=true,hf.GenCRL=true,admin=true:ecert,abac.init=true:ecert" -u https://0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org0/ca/admin

# 登记 Org1 CA 管理员
sudo fabric-ca-client enroll -d -u https://rca-org1-admin:rca-org1-adminpw@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/ca/admin

# 向 Org1 进行注册
sudo fabric-ca-client register -d --id.name peer1-org1 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/ca/admin
sudo fabric-ca-client register -d --id.name peer2-org1 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/ca/admin
sudo fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type user -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/ca/admin
sudo fabric-ca-client register -d --id.name user-org1 --id.secret org1UserPW --id.type user -u https://0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/ca/admin

# 登记 Org2 管理员
sudo fabric-ca-client enroll -d -u https://rca-org2-admin:rca-org2-adminpw@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/ca/admin

# 向 Org2 进行注册
sudo fabric-ca-client register -d --id.name peer1-org2 --id.secret peer1PW --id.type peer -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/ca/admin
sudo fabric-ca-client register -d --id.name peer2-org2 --id.secret peer2PW --id.type peer -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/ca/admin
sudo fabric-ca-client register -d --id.name admin-org2 --id.secret org2AdminPW --id.type user -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/ca/admin
sudo fabric-ca-client register -d --id.name user-org2 --id.secret org2UserPW --id.type user -u https://0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/ca/admin

# Org1-Peer1
# 向 Org1 CA 进行登记，获得所属组织的 ORG CA 证书
sudo fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/peer1 --mspdir msp
# 向 TLS CA 进行登记，获得 TLS 证书
sudo fabric-ca-client enroll -d -u https://peer1-org1:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org1 --home /tmp/hyperledger/org1/peer1 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --mspdir tls-msp
# 拷贝文件
sudo su
mv /tmp/hyperledger/org1/peer1/msp/keystore/*_sk /tmp/hyperledger/org1/peer1/msp/keystore/key.pem

# Org1-Peer2
# 向 Org1 CA 进行登记，获得所属组织的 ORG CA 证书
sudo fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/peer2 --mspdir msp
# 向 TLS CA 进行登记，获得 TLS 证书
sudo fabric-ca-client enroll -d -u https://peer2-org1:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org1 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/peer2 --mspdir tls-msp
# 拷贝文件
sudo su
mv /tmp/hyperledger/org1/peer2/msp/keystore/*_sk /tmp/hyperledger/org1/peer2/msp/keystore/key.pem


# Org1 Admin
# 向 Org1 CA 进行登记，获得所属组织的 ORG CA 证书
sudo fabric-ca-client enroll -d -u https://admin-org1:org1AdminPW@0.0.0.0:7054 --tls.certfiles /tmp/hyperledger/org1/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org1/admin --mspdir msp
# 拷贝文件
sudo su
mkdir /tmp/hyperledger/org1/peer1/msp/admincerts
cp /tmp/hyperledger/org1/admin/msp/signcerts/cert.pem /tmp/hyperledger/org1/peer1/msp/admincerts/org1-admin-cert.pem
mkdir /tmp/hyperledger/org1/peer2/msp/admincerts
cp /tmp/hyperledger/org1/admin/msp/signcerts/cert.pem /tmp/hyperledger/org1/peer2/msp/admincerts/org1-admin-cert.pem

# Org2-Peer1
sudo fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/peer1 --mspdir msp 
# TLS
sudo fabric-ca-client enroll -d -u https://peer1-org2:peer1PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer1-org2 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/peer1 --mspdir tls-msp
# 拷贝
sudo su
mv /tmp/hyperledger/org2/peer1/msp/keystore/*_sk /tmp/hyperledger/org2/peer1/msp/keystore/key.pem

# Org2-Peer2
sudo fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/peer2 --mspdir msp
# TLS
sudo fabric-ca-client enroll -d -u https://peer2-org2:peer2PW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts peer2-org2 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/peer2 --mspdir tls-msp
# 拷贝
sudo su
mv /tmp/hyperledger/org2/peer2/msp/keystore/*_sk /tmp/hyperledger/org2/peer2/msp/keystore/key.pem

# Org2 Admin
sudo fabric-ca-client enroll -d -u https://admin-org2:org2AdminPW@0.0.0.0:7055 --tls.certfiles /tmp/hyperledger/org2/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org2/admin --mspdir msp
# 拷贝
sudo su
mkdir /tmp/hyperledger/org2/peer1/msp/admincerts
cp /tmp/hyperledger/org2/admin/msp/signcerts/cert.pem /tmp/hyperledger/org2/peer1/msp/admincerts/org2-admin-cert.pem
mkdir /tmp/hyperledger/org2/peer2/msp/admincerts
cp /tmp/hyperledger/org2/admin/msp/signcerts/cert.pem /tmp/hyperledger/org2/peer2/msp/admincerts/org2-admin-cert.pem

# 启动 Peer 节点
docker-compose up -d peer1-org1 peer2-org1 peer1-org2 peer2-org2

# Org0-Order
sudo fabric-ca-client enroll -d -u https://orderer1-org0:ordererpw@0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org0/orderer --mspdir msp
# TLS
sudo fabric-ca-client enroll -d -u https://orderer1-org0:ordererPW@0.0.0.0:7052 --enrollment.profile tls --csr.hosts orderer1-org0 --tls.certfiles /tmp/hyperledger/tls-ca/crypto/ca-cert.pem --home /tmp/hyperledger/org0/orderer --mspdir tls-msp
# 拷贝
sudo su
mv /tmp/hyperledger/org0/order/tls-msp/keystore/*_sk /tmp/hyperledger/org0/order/tls-msp/keystore/key.pem

# Org0 Admin
sudo fabric-ca-client enroll -d -u https://admin-org0:org0adminpw@0.0.0.0:7053 --tls.certfiles /tmp/hyperledger/org0/ca/crypto/ca-cert.pem --home /tmp/hyperledger/org0/admin --mspdir msp
# 拷贝
sudo su
mkdir /tmp/hyperledger/org0/orderer/msp/admincerts
cp /tmp/hyperledger/org0/admin/msp/signcerts/cert.pem /tmp/hyperledger/org0/orderer/msp/admincerts/org0-admin-cert.pem
```

# 配置 MSP & 交易通道 & 创世区块

将证书复制到 msp 文件夹下

```bash
sudo su

# Org0
mkdir -p /tmp/hyperledger/configtx && cd /tmp/hyperledger/configtx
mkdir org0
cp -r ../org0/admin/msp org0/

cd org0/msp
mkdir tlscacerts && cd tlscacerts
cp /tmp/hyperledger/org0/orderer/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem ./

# Org1
cd /tmp/hyperledger/configtx
mkdir org1 
cp -r ../org1/admin/msp org1/

cd org1/msp
mkdir tlscacerts && cd tlscacerts
cp /tmp/hyperledger/org1/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem ./

# Org2
cd /tmp/hyperledger/configtx
mkdir org2 
cp -r ../org2/admin/msp org2/

cd org2/msp
mkdir tlscacerts && cd tlscacerts
cp /tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem ./


# # org0
# cp -f /tmp/hyperledger/org0/admin/msp/signcerts/cert.pem /tmp/hyperledger/org0/msp/admincerts/admin-org0-cert.pem
# cp -f /tmp/hyperledger/org0/admin/msp/cacerts/*.pem /tmp/hyperledger/org0/msp/cacerts/org0-ca-cert.pem
# cp -f /tmp/hyperledger/org0/ca/crypto/ca-cert.pem /tmp/hyperledger/org0/msp/tlscacerts/tls-ca-cert.pem

# # org1
# cp -f /tmp/hyperledger/org1/admin/msp/signcerts/cert.pem /tmp/hyperledger/org1/msp/admincerts/admin-org1-cert.pem
# cp -f /tmp/hyperledger/org1/admin/msp/cacerts/*.pem /tmp/hyperledger/org1/msp/cacerts/org1-ca-cert.pem
# cp -f /tmp/hyperledger/org1/ca/crypto/ca-cert.pem /tmp/hyperledger/org1/msp/tlscacerts/tls-ca-cert.pem

# # org2
# cp -f /tmp/hyperledger/org2/admin/msp/signcerts/cert.pem /tmp/hyperledger/org2/msp/admincerts/admin-org2-cert.pem
# cp -f /tmp/hyperledger/org2/admin/msp/cacerts/*.pem /tmp/hyperledger/org2/msp/cacerts/org2-ca-cert.pem
# cp -f /tmp/hyperledger/org2/ca/crypto/ca-cert.pem /tmp/hyperledger/org2/msp/tlscacerts/tls-ca-cert.pem
```



```bash
# 配置通道文件
cd /tmp/hyperledger/confgtx
touch configtx.yaml
# 生成创世区块
mkdir -p /tmp/hyperfabric/output
configtxgen -profile OrgsOrdererGenesis -outputBlock /tmp/hyperledger/output/genesis.block -channelID syschannel
configtxgen -profile OrgsChannel -outputCreateChannelTx /tmp/hyperledger/output/channel.tx -channelID mychannel


# mkdir -p /tmp/hyperfabric/org1/peer1/assets
# mkdir -p /tmp/hyperfabric/org2/peer1/assets
# cp -f /tmp/hyperfabric/output/* /tmp/hyperfabric/org0/orderer
# cp -f /tmp/hyperfabric/output/* /tmp/hyperfabric/org1/peer1/assets
# cp -f /tmp/hyperfabric/output/* /tmp/hyperfabric/org2/peer1/assets

# mkdir -p /tmp/hyperfabric/org1/admin/msp/admincerts
# cp -f /tmp/hyperfabric/org1/msp/admincerts/admin-org1-cert.pem /tmp/hyperfabric/org1/admin/msp/admincerts

# mkdir -p /tmp/hyperfabric/org2/admin/msp/admincerts
# cp -f /tmp/hyperfabric/org2/msp/admincerts/admin-org2-cert.pem /tmp/hyperfabric/org2/admin/msp/admincerts
```

# Order 节点 | 客户端

```bash
docker-compose up -d orderer1-org0 cli-org1 cli-org2
```