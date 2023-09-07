---
title: "Linux Docker Log Files"
date: 2023-09-07T14:23:20+02:00
draft: false
tags:
- Linux
- Docker
---

When using docker on linux I did notice the default folder located at /var/lib/docker/overlay2 got pretty big. This is mostly the fault due to logging (health checks etc..) from the containers.
These commands does help me to clean this up, **remember all containers you need to keep has to be running at this point!**

```
$> docker system prune --all --volumes --force
$> docker builder prune --all
$> truncate -s 0 /var/lib/docker/containers/*/*-json.log  #truncate instead of delete to keep logging possible and not breaking anything..
```

With this command you can link the container ID's inside the overlay2 directory to the actually container:
```
$> docker image inspect $(docker image ls -q)  --format '{{ .GraphDriver.Data.MergedDir}} -> {{.RepoDigests}}' | sed 's|/merged||g'
```

&nbsp;
### Changing docker globally for logging settings
We can tell docker globally to change logging settings for all containers, also newly created containers will be using this setting:
```
$> echo '{"log-driver": "json-file", "log-opts": {"max-size": "10m", "max-file": "3"}}' | jq . > /etc/docker/daemon.json
$> systemctl restart docker
```
So from now on log files shouldn't get bigger then expected.