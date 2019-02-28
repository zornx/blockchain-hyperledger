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

## Examples
- config.json - Hyperledger Explorer config example
- config_v0.3.4.json - Hyperledger Explorer config example (v0.3.4)
- model.cto - Network models example

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

### Setup Hyperledger Explorer
Versions tested:
- Latest - Not working
- v0.3.7.1 - Not working
- v0.3.7 - Not working
- v0.3.6 - Not working
- v0.3.5.1 - Working (Tests failed)
- v0.3.5 - Working (Tests failed)
- v0.3.4 - Working (Current used)
- v0.3.3 - Working

```sh
$ sudo apt update && sudo apt upgrade
$ git clone --branch v0.3.4 https://github.com/hyperledger/blockchain-explorer.git
```

#### Install JQ (v1.5) (v1.6 not working)
```sh
$ wget https://github.com/stedolan/jq/releases/download/jq-1.5/jq-1.5.tar.gz
$ tar -xvzf jq-1.5.tar.gz
$ cd jq-1.5
$ ./configure && make && sudo make install
$ jq --version
```

#### Install PostgreSQL
```sh
$ sudo apt install postgresql postgresql-contrib
$ sudo pg_createcluster 10 main --start
$ psql --version
```

#### Start Hyperledger Fabric Network
```sh
$ cd ~/fabric-dev-servers
$ ./startFabric.sh
```

#### Configure PostgreSQL DB
```sh
$ sudo -u postgres psql
$ \i app/persistence/postgreSQL/db/explorerpg.sql
$ \i app/persistence/postgreSQL/db/updatepg.sql
$ \l
$ \d
$ \q
```

#### Configure Explorer
Save backup:
```sh
$ cd blockchain-explorer/app/platform/fabric
$ cp config.json config_backup.json
```
Check info:
```sh
$ nano ~/fabric-dev-servers/DevServer_connection.json
```
Change on:
```sh
$ cd blockchain-explorer/app/platform/fabric
$ nano config.json
```
Add:
"channel": "mychannel",
"pg": {
  "host": "127.0.0.1",
  "port": "5432",
  "database": "fabricexplorer",
  "username": "hppoc",
  "passwd": "password"
}

#### Run Explorer tests
Save backup:
```sh
$ cd blockchain-explorer
$ npm install
$ npm audit fix
$ cd app/test
$ npm install
$ npm audit fix
$ npm run test
$ cd ../../client
$ npm install
$ npm audit fix
$ npm test -- -u --coverage
```
Fix error: getBlockActivity is not a function
- https://stackoverflow.com/questions/52174042/error-in-hyperledger-explorer-configuration
```sh
$ cd blockchain-explorer
$ nano ~/blockchain-explorer/client/src/components/View/LandingPage.spec.js
```
Add entry on file client\src\components\View\LandingPage.spec.js:
- Add: getBlockActivity:jest.fn(),
```sh
$ npm run build
``

#### Start/Stop Explorer
Save backup:
```sh
$ cd blockchain-explorer
$ ./start.sh
$ ./stop.sh
```
Check Explorer: http://localhost:8080


## References
https://hyperledger-fabric.readthedocs.io/en/release-1.4/

https://hyperledger.github.io/composer/latest/introduction/introduction.html

https://github.com/hyperledger/blockchain-explorer
