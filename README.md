# docker-ambari

## Prerequisites
An Overlay network is needed. <br/>
In every docker-machine(vm or real-machine), use following commands to build a swarm: <br/>
```shell
#start docker deamon with using consul key-value store 
docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=consul://${consul.host.ip}:8500 --cluster-advertise=${network-interface}:2375

#start consul (first)
docker run -d \
    -v /data \
    -p 8300:8300 \
    -p 8301:8301 \
    -p 8301:8301/udp \
    -p 8302:8302 \
    -p 8302:8302/udp \
    -p 8400:8400 \
    -p 8500:8500 \
    -h consul_m01 \
    --name consul_m01 \
    --restart=always \
    progrium/consul -server -advertise ${consul.host.ip} -bootstrap-expect 2
    
#start consul (others)
  docker run -d \
    -v /data \
    -p 8300:8300 \
    -p 8301:8301 \
    -p 8301:8301/udp \
    -p 8302:8302 \
    -p 8302:8302/udp \
    -p 8400:8400 \
    -p 8500:8500 \
    -h consul_m02 \
    --name consul_m02 \
    --restart=always \
    progrium/consul -server -advertise ${this.machine.ip} -join ${consul.host.ip}
    
#start swarm-agent
docker run --name=agent -d \
  --restart=always \
  swarm join --advertise=${this.machine.ip}:2375 consul://${consul.host.ip}:8500

#--
#start a swarm-maanger on any machine in this swarm cluster
docker run -d --name=manager_01 -p 2376:2375 \
  --restart=always \
  swarm manage consul://${consul.host.ip}:8500

#Create an overlay network
docker network create --driver overlay net99

