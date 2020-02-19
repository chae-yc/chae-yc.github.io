---
title: "Docker - useful commands"
categories:
  - blogging
  - human
tags:
  - Docker
last_modified_at: 2020-02-01T16:11:00+09:00
toc: true
---

##  Check container's IP address

1. Use Docker *inspect*
    - docker inspect { CONTAINER_ID } | grep IPAddress
2. Use Docker *exec*
    - docker exec { CONTAINER_ID } ip addr show eth0


## Error (only in Window, when using docker toolbox)

> C:\Program Files\Docker Toolbox\docker.exe: Error response from daemon: cgroups: cannot find cgroup mount destination: unknown.

**Solution**:

in Docker toolbox

```sh
docker-machine ssh default
sudo -i
mkdir /sys/fs/cgroup/systemd
mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
exit
exit
```

## Delete all except one container

`docker rm $(docker ps -a | grep -v "container_name" | awk 'NR>1 {print $1}')`


### REF

https://docs.docker.com/compose/compose-file/