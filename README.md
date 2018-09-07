## HYPER LEDGER FABIC BLOCKCHAIN - Multinode

A Hyperledger Fabric distributed blockchain application which query and invoke the asset stored in the blockchain

## Docker and Swarm Setup

### Run the following commands to make the nodes as distributed docker setup
```
cd /mnt && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -


sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update

apt-cache policy docker-ce

sudo apt-get install docker-ce

sudo systemctl status docker

docker --version

docker run hello-world
```

### Install Docker Compose Latest version
```
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/bin/docker-compose

sudo chmod +x /usr/bin/docker-compose

docker-compose --version
```

### Init docker swarm
```
docker swarm init

Swarm initialized: current node (98553banw4boq29wg29ukgjat) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1j1uxr7o7pb9zsla95wvk21z6y2z6smjxa1hmppz5h80ded6zu-1c8iapuuh9f3vhwfckr7vx7y7 172.31.40.85:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### To add a manager to this swarm, run the following command in the other machines which wanted to join as manager:

```
    docker swarm join --token SWMTKN-1-1j1uxr7o7pb9zsla95wvk21z6y2z6smjxa1hmppz5h80ded6zu-f3iabcecigealzsjruki5qp7k 172.31.40.85:2377
```

### Create overlay network
```
docker network create --attachable --driver overlay hlf-net
```

## Fabric Setup in Multi node

### Download Platform-specific Binaries
```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.0
```

### Clone this repo on both the PCs i.e PC1 and PC2
```
git clone https://github.com/pranoygn/hyperledger_fabric.git

cd hyperledger_fabric/
./byn.sh
```

### CA server - PC1
```
docker run --rm -it --network="hlf-net" --name ca.e42.in -p 7054:7054 -e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server -e FABRIC_CA_SERVER_CA_NAME=ca.e42.in -e FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.e42.in-cert.pem -e FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/2fad2760d568752fe0f294bec5025a41070339ddb0d1f54637b620d9855b0153_sk -v $(pwd)/crypto-config/peerOrganizations/org1.e42.in/ca/:/etc/hyperledger/fabric-ca-server-config -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net hyperledger/fabric-ca sh -c 'fabric-ca-server start -b admin:adminpw -d'
```

### Orderer - PC1
docker run --rm -it --network="hlf-net" --name orderer.e42.in -p 7050:7050 -e ORDERER_GENERAL_LOGLEVEL=debug -e ORDERER_GENERAL_LISTENADDRESS=0.0.0.0 -e ORDERER_GENERAL_LISTENPORT=7050 -e ORDERER_GENERAL_GENESISMETHOD=file -e ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block -e ORDERER_GENERAL_LOCALMSPID=OrdererMSP -e ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp -e ORDERER_GENERAL_TLS_ENABLED=false -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net -v $(pwd)/channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block -v $(pwd)/crypto-config/ordererOrganizations/e42.in/orderers/orderer.e42.in/msp:/var/hyperledger/orderer/msp -w /opt/gopath/src/github.com/hyperledger/fabric hyperledger/fabric-orderer orderer
```
### CouchDB 0 - PC1
```
docker run --rm -it --network="hlf-net" --name couchdb0 -p 5984:5984 -e COUCHDB_USER= -e COUCHDB_PASSWORD= -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net hyperledger/fabric-couchdb
```
### Peer 0 - PC1
```
docker run --rm -it --network="hlf-net" --link orderer.e42.in:orderer.e42.in --name peer0.org1.e42.in -p 8051:7051 -p 8053:7053 -e CORE_LEDGER_STATE_STATEDATABASE=CouchDB -e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984 -e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME= -e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD= -e CORE_PEER_ADDRESSAUTODETECT=true -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_LOGGING_LEVEL=DEBUG -e CORE_PEER_NETWORKID=peer0.org1.e42.in -e CORE_NEXT=true -e CORE_PEER_ENDORSER_ENABLED=true -e CORE_PEER_ID=peer0.org1.e42.in -e CORE_PEER_PROFILE_ENABLED=true -e CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer.e42.in:7050 -e CORE_PEER_GOSSIP_IGNORESECURITY=true -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net -e CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.e42.in:7051 -e CORE_PEER_TLS_ENABLED=false -e CORE_PEER_GOSSIP_USELEADERELECTION=false -e CORE_PEER_GOSSIP_ORGLEADER=true -e CORE_PEER_LOCALMSPID=Org1MSP -v /var/run/:/host/var/run/ -v $(pwd)/crypto-config/peerOrganizations/org1.e42.in/peers/peer0.org1.e42.in/msp:/etc/hyperledger/fabric/msp -w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-peer peer node start

docker run --rm -it --network="hlf-net" --link orderer.e42.in:orderer.e42.in --link peer0.org1.e42.in:peer0.org1.e42.in --name peer1.org1.e42.in -p 9051:7051 -p 9053:7053 -e CORE_LEDGER_STATE_STATEDATABASE=CouchDB -e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984 -e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME= -e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD= -e CORE_PEER_ADDRESSAUTODETECT=true -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_LOGGING_LEVEL=DEBUG -e CORE_PEER_NETWORKID=peer1.org1.e42.in -e CORE_NEXT=true -e CORE_PEER_ENDORSER_ENABLED=true -e CORE_PEER_ID=peer1.org1.e42.in -e CORE_PEER_PROFILE_ENABLED=true -e CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer.e42.in:7050 -e CORE_PEER_GOSSIP_ORGLEADER=true -e CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.e42.in:7051 -e CORE_PEER_GOSSIP_IGNORESECURITY=true -e CORE_PEER_LOCALMSPID=Org1MSP -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net -e CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.e42.in:7051 -e CORE_PEER_GOSSIP_USELEADERELECTION=false -e CORE_PEER_TLS_ENABLED=false -v /var/run/:/host/var/run/ -v $(pwd)/crypto-config/peerOrganizations/org1.e42.in/peers/peer1.org1.e42.in/msp:/etc/hyperledger/fabric/msp -w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-peer peer node start
```
### CouchDB 1 - PC2
```
docker run --rm -it --network="hlf-net" --name couchdb1 -p 6984:5984 -e COUCHDB_USER= -e COUCHDB_PASSWORD= -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net hyperledger/fabric-couchdb
```
### Peer 1 - PC2
```
docker run --rm -it --network="hlf-net" --link orderer.e42.in:orderer.e42.in --link peer0.org1.e42.in:peer0.org1.e42.in --name peer1.org1.e42.in -p 9051:7051 -p 9053:7053 -e CORE_LEDGER_STATE_STATEDATABASE=CouchDB -e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984 -e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME= -e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD= -e CORE_PEER_ADDRESSAUTODETECT=true -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_LOGGING_LEVEL=DEBUG -e CORE_PEER_NETWORKID=peer1.org1.e42.in -e CORE_NEXT=true -e CORE_PEER_ENDORSER_ENABLED=true -e CORE_PEER_ID=peer1.org1.e42.in -e CORE_PEER_PROFILE_ENABLED=true -e CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer.e42.in:7050 -e CORE_PEER_GOSSIP_ORGLEADER=true -e CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.e42.in:7051 -e CORE_PEER_GOSSIP_IGNORESECURITY=true -e CORE_PEER_LOCALMSPID=Org1MSP -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net -e CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.e42.in:7051 -e CORE_PEER_GOSSIP_USELEADERELECTION=false -e CORE_PEER_TLS_ENABLED=false -v /var/run/:/host/var/run/ -v $(pwd)/crypto-config/peerOrganizations/org1.e42.in/peers/peer1.org1.e42.in/msp:/etc/hyperledger/fabric/msp -w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-peer peer node start
```
### CLI - PC2
```
docker run --rm -it --network="hlf-net" --name cli --link orderer.e42.in:orderer.e42.in --link peer0.org1.e42.in:peer0.org1.e42.in --link peer1.org1.e42.in:peer1.org1.e42.in -p 12051:7051 -p 12053:7053 -e GOPATH=/opt/gopath -e CORE_PEER_LOCALMSPID=Org1MSP -e CORE_PEER_TLS_ENABLED=false -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_LOGGING_LEVEL=DEBUG -e CORE_PEER_ID=cli -e CORE_PEER_ADDRESS=peer0.org1.e42.in:7051 -e CORE_PEER_NETWORKID=cli -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/users/Admin@org1.e42.in/msp -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net  -v /var/run/:/host/var/run/ -v $(pwd)/chaincode/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go -v $(pwd)/crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ -v $(pwd)/scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/ -v $(pwd)/channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts -w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-tools /bin/bash -c './scripts/script.sh'
```

## Testing the Network

### Bin/Bash CLI - PC2
```
docker run --rm -it --network="hlf-net" --name cli --link orderer.e42.in:orderer.e42.in --link peer0.org1.e42.in:peer0.org1.e42.in --link peer1.org1.e42.in:peer1.org1.e42.in -p 12051:7051 -p 12053:7053 -e GOPATH=/opt/gopath -e CORE_PEER_LOCALMSPID=Org1MSP -e CORE_PEER_TLS_ENABLED=false -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_LOGGING_LEVEL=DEBUG -e CORE_PEER_ID=cli -e CORE_PEER_ADDRESS=peer0.org1.e42.in:7051 -e CORE_PEER_NETWORKID=cli -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/users/Admin@org1.e42.in/msp -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=hlf-net  -v /var/run/:/host/var/run/ -v $(pwd)/chaincode/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go -v $(pwd)/crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ -v $(pwd)/scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/ -v $(pwd)/channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts -w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-tools /bin/bash
```

### Instantiate Chaincode on Peer0
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/users/Admin@org1.e42.in/msp
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/peers/peer0.org1.e42.in/tls/ca.crt
CORE_PEER_ADDRESS=peer0.org1.e42.in:7051

peer chaincode instantiate -o orderer.e42.in:7050 -C mychannel -n assetcc -v 1.0 -c '{"Args":["init","asset_id_1","{"ID":1, "name":"Asset 1"}"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```

### Query the Chaincode on Peer1
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/users/Admin@org1.e42.in/msp
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/peers/peer0.org1.e42.in/tls/ca.crt
CORE_PEER_ADDRESS=peer1.org1.e42.in:7051

peer chaincode query -C mychannel -n assetcc -c '{"Args":["query","asset_id_1"]}'

Query Result: {"ID":1, "name":"Asset 1"}
```

### Invoke the Chaincode on Peer0
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/users/Admin@org1.e42.in/msp
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.e42.in/peers/peer0.org1.e42.in/tls/ca.crt
CORE_PEER_ADDRESS=peer0.org1.e42.in:7051

peer chaincode invoke -o orderer.e42.in:7050 -C mychannel -n assetcc -c '{"Args":["invoke","asset_id_2","{"ID":2, "name":"Asset 2"}"]}'
```

### Query the Chaincode
```
peer chaincode query -C mychannel -n assetcc -c '{"Args":["query","asset_id_2"]}'

Query Result: {"ID":2, "name":"Asset 2"}
```
