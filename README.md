# kivitendo-docker

## Installation

```
DDIR=./wws
git clone https://github.com/unimock/kivitendo-docker.git $DDIR
cd $DDIR
```

## Build image
Customize docker-compose.yml to your needs and create an image. 

```
cp docker-compose.yml-template docker-compose.yml
# edit
  vi docker-compose.yml
  docker compose build
  #docker compose --progress plain build --no-cache
  #docker buildx bake 
# or
  sed -i -e "s/myhost/<HOSTNAME>/g" docker-compose.yml
  sed -i -e "s/test.com/<DOMAIN>/g" docker-compose.yml

```

## Create and run a container from the image.
```
sudo mkdir -p /wwws/postgresql # create directory for named bind volume (see docker-compose.yml)
docker compose up # -d
```


