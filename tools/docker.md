## docker

### install

```shell
# for centos
yum install docker
systemctl start docker

# /etc/docker/daemon.json
{
	"registry-mirrors": ["http://hub-mirror.c.163.com"]
}

# to test
docker run hello-world
docker search ubuntu
docker pull ubuntu
docker run -t -i ubuntu /bin/bash

# check docker status
docker images
docker ps

```

