# kivitendo-docker

## Build image

Customize docker-compose.yml to your needs and create an image. 

```
cp docker-compose.yml-template docker-compose.yml
vi docker-compose.yml
docker compose build
#docker compose --progress plain build --no-cache
#docker buildx bake 
```

## Create and run a container from the image.

```
sudo mkdir -p /wwws/postgresql # create directory for named bind volume (see docker-compose.yml)
docker compose up
```

## start from scratch

```
docker compose down
docker volume rm kivitendo-docker_postgresql
sudo rm -Rvf /wws/postgresql
sudo mkdir /wws/postgresql
```

