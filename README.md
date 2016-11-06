# kivitendo-docker


## Requirements
 * docker-engine version >= 1.10.0 
 * user with sudo access
 * user belongs to "docker" group

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
  docker-compose build
# or
  sed -i -e "s/myhost/<HOSTNAME>/g" docker-compose.yml
  sed -i -e "s/test.com/<DOMAIN>/g" docker-compose.yml
./do build
```

## Create and run a container from the image.
```
./do up
./do log
```

## Create a new image
 This is needed after changes in the BUILD section in docker-compose.yml. or for updates.
 
```
./do stop                                           # stop ispconfig
./do rm                                             # remove the container
git pull                                            # update ispconfig-docker
diff docker-compose.yml-template docker-compose.yml # check for new options docker-compose.yml
./do build
./do up                                             # create a new container
./do log                                            # show start up console
```

## useful commands (examples):

```
./do                                                # help
./do start|stop|restart                             # start/stop the container
./do console                                        # attach to the console
./do supervisor                                     # attach to supervisord to start/stop/restart daemons in the container
./do login bash                                     # login and run commands in the container
```

