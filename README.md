# Service discovery with Consul

Follow article [https://www.airpair.com/scalable-architecture-with-docker-consul-and-nginx](https://www.airpair.com/scalable-architecture-with-docker-consul-and-nginx)

Tested on Mac OS X

## Steps

1. Up boot2docker
```
brew install boot2docker
boot2docker init  
boot2docker up
export DOCKER_IP=`boot2docker ip`  
export DOCKER_HOST=`boot2docker socket`
```

2. Build Web service
```
cd app
docker build -t python/server .
```

3. Run Consul
```
docker run -it -h node \
 -p 8500:8500 \
 -p 8600:53/udp \
 progrium/consul \
 -server \
 -bootstrap \
 -advertise $DOCKER_IP \
 -log-level debug
```

4. Run Registrator
```
docker run -it \
-v /var/run/docker.sock:/tmp/docker.sock \
-h $DOCKER_IP progrium/registrator \
consul://$DOCKER_IP:8500
```

5. Build & run Nginx with consul template
```
cd drcon
docker build -t drcon .
docker run -it \
-e "CONSUL=$DOCKER_IP:8500" \
-e "SERVICE=simple" \
-p 80:80 drcon
```

6. Start Web services
```
docker run -it \
-e "SERVICE_NAME=simple" \
-p 8000:8000 python/server

docker run -it \
-e "SERVICE_NAME=simple" \
-p 8001:8000 python/server
```

7. Test load balancing
```
while true; do curl $DOCKER_IP:80; sleep 1; done
```

You will see requests are proxied to 2 web services above. If want more web service, just run command
```
docker run -it \
-e "SERVICE_NAME=simple" \
-p <external port>:8000 python/server
```
