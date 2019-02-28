# Blockchain Hyperledger
Tutorial for Hyperledger Fabric Network (v1.3) using Hyperledger Composer (v2.0) and Hyperledger Explorer (v0.3.4)

## Requirements
- VirtualBox
- Docker
- Docker Compose
- Node.js
- NPM
- GIT
- Python

## Notes
If installing Hyperledger Composer using Linux, be aware of the following advice:
- Login as a normal user, rather than root.
- Do not su to root.
- When installing prerequisites, use curl, then unzip using sudo.
- Run prereqs-ubuntu.sh as a normal user. It may prompt for root password as some of it's actions are required to be run as root.
- Do not use npm with sudo or su to root to use it.
- Avoid installing node globally as root.**

## Instructions
### Setup Environment
```sh
$ sudo apt update && sudo apt upgrade
$ sudo apt install git make gcc g++ libltdl-dev curl python pkg-config
$ sudo apt update && sudo apt upgrade
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt update
$ sudo apt install docker-ce
$ sudo systemctl status docker
$ docker -v
$ sudo su
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
$ su $USER
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose -v
$ wget https://nodejs.org/dist/v8.11.2/node-v8.11.2-linux-x64.tar.xz
$ tar -xf node-v8.11.2-linux-x64.tar.xz
$ mv node-v8.11.2-linux-x64/ node/
$ echo 'export PATH=~/node/bin:$PATH' >> ~/.profile
$ source .profile
$ npm install -g npm
$ npm install -g grpc
$ node -v
$ npm -v
```

### Setup Hyperledger Composer
```sh
$ npm install -g composer-cli@0.20
$ npm install -g composer-rest-server@0.20
$ npm install -g generator-hyperledger-composer@0.20
$ npm install -g yo
$ npm install -g composer-playground@0.20
```

### Install Hyperledger Fabric Dev Servers
```sh
$ mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers
$ curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
$ tar -xvf fabric-dev-servers.tar.gz
$ cd ~/fabric-dev-servers
$ export FABRIC_VERSION=hlfv12
$ ./downloadFabric.sh
```

### Install Hyperledger Fabric Binaries
```sh
$ git clone https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0
$ echo 'export PATH=~/fabric-samples/bin:$PATH' >> ~/.profile
$ source ~/.profile
```

### Controlling your Dev Environment
```sh
$ cd ~/fabric-dev-servers
$ export FABRIC_VERSION=hlfv12 (Old version hlfv11)
$ ./startFabric.sh
$ docker ps
$ ./createPeerAdminCard.sh
$ ~/fabric-dev-servers/startFabric.sh
```
Check CouchDB: http://localhost:5984/_utils/#

#### Start the web app ("Playground")
```sh
$ composer-playground
```
Check Composer Playground: http://localhost:8080/login

### Composer Setup Network
```sh
$ yo hyperledger-composer:businessnetwork
```
Network Configuration:
- Network Name: tutorial-network
- License: Apache-2.0
- Namespace: org.example.mynetwork
- Empty Network: No
Defining a business network
- Modelling network (.cto file)
- Defining JavaScript transaction logic (logic.js file)
- Defining access control (permissions.acl)

#### Generate a business network archive
```sh
$ cd tutorial-network
$ composer archive create -t dir -n .
```

#### Deploying the business network 
Note: Fabric network should be running (./startFabric.sh)
```sh
$ cd tutorial-network
$ composer network install --card PeerAdmin@hlfv1 --archiveFile tutorial-network@0.0.1.bna
$ composer network start --networkName tutorial-network --networkVersion 0.0.1 --networkAdmin admin --networkAdminEnrollSecret adminpw --card PeerAdmin@hlfv1 --file networkadmin.card
$ composer card import --file networkadmin.card
$ composer network ping --card admin@tutorial-network
```
#### Start Composer Playground
```sh
$ composer-playground
```
Check Composer Playground: http://localhost:8080

#### Generating a REST server
```sh
$ cd tutorial-network
$ composer-rest-server
```
REST Server Configuration:
- Card Name: admin@tutorial-network
- Namespaces: Never Use
- Secure the generated API with API Key: No
- Enable authenticatio using Passport: No
- Enable explorer test interface: Yes
- Enable dynamic logging: No
- Enable event publication over WebSockets: Yes
- Enable TLS security: No
Check REST API: http://localhost:3000/explorer/

#### Generating an Angular application
```sh
$ cd tutorial-network
$ yo hyperledger-composer:angular
```
Application Configuration (Change)
- Connect to running business network: Yes
- Enter standard package.json questions
- Business network card: admin@tutorial-network
- Select: Connect to an existing REST API
- REST Server address: http://localhost
- Server port: 3000
- Select: Namespaces are not usednom 

#### Run the application
Note: REST Server should be running
```sh
$ cd angular-app
$ npm start
```
Check Angular App: http://localhost:4200
Fix Error message: Invalid Host header
- In node_modules/webpack-dev-server/lib/Server.js change:
  - "if(this.disableHostCheck) return true;" to "return true;"

## References
https://hyperledger-fabric.readthedocs.io/en/release-1.4/

https://hyperledger.github.io/composer/latest/introduction/introduction.html

https://github.com/hyperledger/blockchain-explorer
