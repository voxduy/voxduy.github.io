---
title: Docker Commands - Frequently Used
author: voxduy
date: 2024-04-14 12:00:00 +0700
categories: [Docker]
tags: [docker]
image:
  path: /posts/2024-04-14-docker-commands-frequently-used/docker.jpg
  width: 800
  height: 500
pin: true
---

This is a collection of docker commands that I have ever been. And I still keep it up-to-date currently.

# Init Docker

Install docker using the convenience script

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

# Run a new container

Start a new container

```bash
docker run IMAGE
```

Map a port

```bash
docker run -p HOST_PORT:CONTAINER_PORT IMAGE
```
> Ex: `docker run -p 8080:80 nginx`

Map all port

```bash
docker run -P IMAGE
```

# Manage containers

A list of running containers

```bash
docker ps
```

A list of all containers

```bash
docker ps -a
```

Remove a container

```bash
docker rm CONTAINER
```

Remove a running container

```bash
docker rm -f CONTAINER
```

Delete stopped containers

```bash
docker container prune
```

Stop a running container

```bash
docker stop CONTAINER
```

Start a stopped container

```bash
docker start CONTAINER
```

# Manage images

Download an image from a Docker registry

```bash
docker pull IMAGE
```

Delete an image

```bash
docker rmi IMAGE
```

Lists all docker images available on the system.

```bash
docker images
```

Build & tag an image from a Dockerfile

```bash
docker build -t IMAGE DIRECTORY
```

# Docker network

Lists all Docker networks

```bash
docker network list
```

 Create a new docker network

```
docker network create reverseproxy-nw
```

Remove a docker network

```bash
docker network rm reverseproxy-nw
```

# Info & Stats

Show installed docker version

```bash
docker version
```

Show stats of running containers

```bash
docker stats
```

Show the logs of a container

```bash
docker logs CONTAINER
```

Show cấu hình của container ở low-level (JSON format)

```bash
docker inspect CONTAINER
```

Show processes of container

```
docker top CONTAINER
```

# Docker compose

- Start containers which are defined in docker-compose.yml

```bash
docker-compose up -d
```
> Detached mode: Run containers in the background, print new container names

```bash
docker-compose up --build
```
> Rebuild the images for the services in the `docker-compose.yml` file before starting the containers


- Stop containers which are defined in docker-compose.yml

```
docker-compose down
```

```
docker-compose down -v
```
> Stops and removes containers, networks, and volumes

- See docker-compose logs

```bash
docker-compose logs <name-of-service>
```

> Options:
> --no-color Produce monochrome output.
> -f, --follow Follow log output.
> -t, --timestamps Show timestamps.
> --tail="all" Number of lines to show from the end of the logs for each container.

```bash
docker-compose logs -f -t >> myDockerCompose.log
```
> To save the output to a file

```bash
docker-compose logs -t -f --tail <no of lines>
```
> see output logs from all the services in your terminal

Ex: show logs của tất cả service docker, chỉ 5 dòng cuối

```bash
docker-compose logs -t -f --tail 5
```

```bash
docker-compose logs -t -f --tail <no of lines> <name-of-service1> <name-of-service2> ... <name-of-service N>
```
> log output from specific services

Ex: say you have API and portal services then you can do something like below. Where 5 represents last 5 lines from both logs.

```bash
docker-compose logs -t -f --tail 5 portal api
```

# Operation & Tshoot

1. **ERROR >>>  'ContainerConfig'**

```
docker compose up -d --force-recreate
```


2. **Upgrade hoặc restart một server có docker container**

Cần đảm bảo rằng các container sẽ được **tự động khởi động lại** sau khi hệ thống khởi động lại, hoặc lưu trạng thái để không mất dữ liệu quan trọng:

Bước 1: Kiểm tra các container đang chạy

```bash
docker ps
```

Bước 2: Cấu hình container tự động khởi động lại sau khi reboot
Để đảm bảo các container được tự động khởi động lại sau khi hệ thống restart, bạn có thể cấu hình **restart policy** cho các container. Sử dụng lệnh dưới đây để cấu hình cho container hiện tại:

```bash
docker update --restart unless-stopped <container_name>
```

Trong đó, `<container_name>` là tên hoặc ID của container. Tùy chọn **unless-stopped** sẽ đảm bảo rằng container chỉ không khởi động lại nếu trước đó nó đã được dừng thủ công.

Bước 3: Cập nhật hệ thống Ubuntu
Tiếp theo, bạn có thể thực hiện quá trình **upgrade** hệ thống. Điều này có thể được thực hiện thông qua các lệnh thông thường:

```bash
sudo apt update
sudo apt upgrade
```

Bước 4: Restart server (nếu cần)

```bash
sudo reboot
```

Bước 5: Kiểm tra lại các container sau khi khởi động lại

```bash
docker ps
```

Nếu các container đã được thiết lập đúng với chính sách **restart unless-stopped**, chúng sẽ tự động khởi động lại và chạy bình thường.

**Cách an toàn: chủ động stop tất cả các container trước khi upgrade hoặc restart**
Nếu bạn muốn dừng tất cả các container trước khi nâng cấp và sau đó khởi động lại chúng sau khi hoàn tất:

1. Dừng tất cả container

   ```bash
   docker stop $(docker ps -q)
   ```

2. Thực hiện upgrade

   ```bash
   sudo apt update
   sudo apt upgrade
   ```

3. Khởi động lại tất cả container sau khi nâng cấp xong

   ```bash
   docker start $(docker ps -a -q)
   ```

**Bonus**
Sử dụng command sau để kiểm tra xem một container Docker có được cấu hình để tự động khởi động lại sau khi restart server hay không:

```bash
docker inspect -f '{{ .HostConfig.RestartPolicy.Name }}' <container_name_or_id>
```

Trong đó:
- `<container_name_or_id>` là tên hoặc ID của container mà bạn muốn kiểm tra.

> Các giá trị có thể của Restart Policy:
> - **no**: Container sẽ không tự động khởi động lại.
> - **always**: Container sẽ tự động khởi động lại bất cứ khi nào nó dừng lại hoặc khi hệ thống khởi động lại.
> - **unless-stopped**: Container sẽ tự động khởi động lại trừ khi nó được dừng thủ công.
> - **on-failure**: Container sẽ chỉ khởi động lại nếu nó dừng lại do lỗi (với mã thoát khác 0).

Ví dụ:

```bash
docker inspect -f '{{ .HostConfig.RestartPolicy.Name }}' my_container
```

Kết quả có thể là `no`, `always`, `unless-stopped`, hoặc `on-failure` tùy thuộc vào chính sách restart của container đó.  
  
  
**Reference**  
[The Ultimate Docker Cheat Sheet | dockerlabs (collabnix.com)](https://dockerlabs.collabnix.com/docker/cheatsheet/)
