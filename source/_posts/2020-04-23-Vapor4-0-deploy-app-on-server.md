title: '[Vapor4.0]deploy app on server'
date: 2020-04-23 17:04:49

tags:
categories: Vapor 4.0
---
Just record the steps how I deploy my vapor project on the server.

Step 1: Install Docker

```swift
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```

Step2: Start Docker

```swift
sudo systemctl enable docker
sudo systemctl start docker
sudo groupadd docker
sudo usermod -aG docker $USER
```

Step3: Install docker compose

```swift
sudo curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

Step4: copy the source code into the server.

```swift
scp Dreamoji-server root@144.202.101.94:~
```

Step5: unzip the source code.

```swift
apt install unzip
unzip Dreamoji-server.zip
```

Step6: build & run

```swift
cd Dreamoji-server
docker-compose build
docker-compose up app
```



