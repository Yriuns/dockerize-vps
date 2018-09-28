# How to deploy multiple independent websites with docker

## Introduction

It is common for individual developers and small businesses to run multiple independent websites on a single VPS. However, if they need to listen on the same port, such as `80`, we had to merge their configuration files together, which is not clean.

The main idea comes from [How to set your VPS with docker containers for multiple websites](https://medium.com/@rokerkony/how-to-set-your-vps-with-docker-containers-for-multiple-websites-55524e59cae1) and corresponding repo [rokerkony/dockerize-vps](https://github.com/rokerkony/dockerize-vps). However, his method doesn't support communication between containers. To solve this problem, I make some modifications based on his repo.

[jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy) is a automated nginx reverse proxy for docker.
> nginx-proxy sets up a container running nginx and docker-gen. docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped. 

We can use it as a unified frontend to listen on `80` and `443` port, and then dispatch request to different websites according to the url.

## Usage

### Clone this repo

```bash
git clone git@github.com:Yriuns/dockerize-vps.git
```

### Start the nginx-proxy

```bash
cd dockerize-vps/nginx_proxy
docker-compose up -d
```

### Integrate into your project

You need to follow the structure and files in `/template` directory.

1. If you have already run nginx in a docker container, just wrap it in a `/nginx` folder.
2. Otherwise, create a `/nginx` forlder, write `Dockerfile` and `default.conf` like `/template` do.
3. Copy or edit your project `docker-compose.yml` file.

```yaml
version: "2.0"
services:
  nginx:
    build: ./nginx
    ports:
      # any port that has not been used
      - "8081:80"
    links:
      - backend
    environment:
      # your domain
      - VIRTUAL_HOST=example.com
    networks:
      - default
      # connect to nginx-proxy network
      - nginx_proxy_default
  backend:
    build: ./backend
    volumes:
      - ./backend:/opt/workspace

# connect to nginx-proxy network
networks:
  nginx_proxy_default:
    external: true
```
4. Run
```bash
docker-compose up -d
```

## Example

Start `nginx_proxy`

```bash
cd nginx_proxy
docker-compose up -d
```

Start `site1`

```bash
cd example/site1
docker-compose up -d
```

Start `site2`

```bash
cd example/site2
docker-compose up -d
```

Test `sie1`

```bash
curl -H "Host: site1.com" localhost
```

```html
<html>
<head>
    <meta charset="utf-8">
    <title>Site 1</title>
</head>

<body>
    <h1>Hello, World!</h1>
</body>
</html>
```

Test `site2`

```bash
curl -H "Host: site2.com" localhost
```

```html
<html>
<head>
    <meta charset="utf-8">
    <title>Site 2</title>
</head>

<body>
    <h1>Hello, World!</h1>
</body>
</html>
```