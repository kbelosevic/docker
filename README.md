# Simple dockerized python application and Redis

With next instructions we will:
1) Build python docker image to run simple python app
2) Create python application which will run simple HTTP request counter.
3) Run dockerized Redis
4) Trough use of Docker Swarm we will scale up our Python application

**Requirements**
Linux (deb/rpm based)

## Installation steps

**1)Installing docker on Ubuntu by following instrcutions from [official Docker website](https://docs.docker.com/install/linux/docker-ce/ubuntu/) or quick and "dirty way":**
```
sudo apt-get -y remove docker docker-engine docker.io containerd runc && \
sudo apt-get update && \
sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo apt-key fingerprint 0EBFCD88 && \
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
sudo apt-get update && \
sudo apt-get -y install docker-ce docker-ce-cli containerd.io
```
**2)Let's make it easy to use docker commands without typing sudo everytime**
```
sudo usermod -aG docker $USER
```
**3)Clone this repository**
```
mkdir -pv ~/test/ && cd ~/test/ && git clone https://github.com/kbelosevic/docker.git
```
**4)Building Docker image**
```
cd ~/test/docker && docker build --tag=testing_app:test .
```
**5)Let's veryify that image is there:**
```
docker images |grep testing_app
testing_app               test                825f5d26be99        4 hours ago         131MB
```
**6)Initialize swarm**
```
docker swarm init
Swarm initialized: current node (m2deayreefe8zgrnk3ioi9zwj) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-00iirt2bgj29qlnkruj6gfmhcoevjwcrj1ddz05uzhpaxbucd1-9ed9yqpnz5bt7vvw6yom8hqv6 1.2.3.4:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
**7)Let's deploy out application**
```
cd ~/test/docker && docker stack deploy -c docker-compose.yml testapp
Ignoring unsupported options: build

Creating network testapp_webnet
Creating service testapp_web
Creating service testapp_redis
```
**8)Test if everything is ok**
```
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ww1gy3havd5d        testapp_redis       replicated          1/1                 redis:latest        *:6379->6379/tcp
ww1aqirtxwum        testapp_web         replicated          10/10               testing_app:test    *:80->80/tcp
```
```
docker stack ps testapp
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE         ERROR               PORTS
vihbzoror5f0        testapp_web.1       testing_app:test    zadatak             Running             Running 2 hours ago
phvbzs0medxq        testapp_redis.1     redis:latest        zadatak             Running             Running 2 hours ago
rf2044vpmmji        testapp_web.2       testing_app:test    zadatak             Running             Running 2 hours ago
tm2e5v1sw5te        testapp_web.3       testing_app:test    zadatak             Running             Running 2 hours ago
le5eomjvfabr        testapp_web.4       testing_app:test    zadatak             Running             Running 2 hours ago
x0su81iamf5t        testapp_web.5       testing_app:test    zadatak             Running             Running 2 hours ago
otzztp17ko17        testapp_web.6       testing_app:test    zadatak             Running             Running 2 hours ago
e3f3sxqa5b39        testapp_web.7       testing_app:test    zadatak             Running             Running 2 hours ago
wvxssb5urbqx        testapp_web.8       testing_app:test    zadatak             Running             Running 2 hours ago
p21slo9ep4m7        testapp_web.9       testing_app:test    zadatak             Running             Running 2 hours ago
xsb358534g9j        testapp_web.10      testing_app:test    zadatak             Running             Running 2 hours ago
```
```
curl http://127.0.0.1
<h3>Hello World!</h3><b>Hostname:</b> 5ea2dda49b48<br/><b>Visits:</b> 38095
```

## Stress testing the application
**1)Creating SSH forwarding**

We will forward our local port 45667 to servers port 80, so that when we visit 127.0.0.1:45667 it will actually show "localhost of remote server on port 80 127.0.0.1:80"
```
ssh -N -L 45667:127.0.0.1:80 zadatak@62.138.6.28 -p 47522
```
**2)Starting first ab test on our local machine when we have 1 service defined (replicas: 1) in docker-compose.yml**
```
ab -k -c 200 -n 2000 http://127.0.0.1:45667/
```
**3)Starting second ab test on our local machine when we have 10 services defined (replicas: 10) in docker-compose.yml - requires redeploy of our application first by using**

On server:
```
cd ~/test/docker && docker stack deploy -c docker-compose.yml testapp
```

Locally:
```
ab -k -c 200 -n 2000 http://127.0.0.1:45667/
```
Then we compare the tests.

**4)Simultaneosly of our tests we can also record HTTP traffic by using tcpdump on the server**
```
sudo tcpdump -nnvvSA port 80 -w captured_http_traffic.pcap
```
Reading it:
```
sudo tcpdump -nnvvSA -r captured_http_traffic.pcap
```
### Dirty one liner for initial docker install/setup and app deploy
```
sudo apt-get -y remove docker docker-engine docker.io containerd runc && \
sudo apt-get update && \
sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo apt-key fingerprint 0EBFCD88 && \
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
sudo apt-get update && \
sudo apt-get -y install docker-ce docker-ce-cli containerd.io && \
sudo usermod -aG docker $USER && \
mkdir -pv ~/test/ && cd ~/test/ && git clone https://github.com/kbelosevic/docker.git && \
cd ~/test/docker && docker build --tag=testing_app:test . && \
docker swarm init && \
cd ~/test/docker && docker stack deploy -c docker-compose.yml testapp && \
curl http://127.0.0.1 && echo
```
