include ../config
NAME = ${PUSH_URL}/mdd-wws
VERSION=0.2.0

.PHONY: build
build:
	docker build --rm -t $(NAME):$(VERSION) .

.PHONY: rmi
rmi:
	docker rmi  $(NAME):$(VERSION)

#.PHONY: run
#run:
#	docker run  -d $(docker_run_flags) --name logstash $(NAME):$(VERSION)
#
#.PHONY: shell
#shell:
#	docker run -t -i --rm $(NAME):$(VERSION) /bin/bash
#
#.PHONY: test
#test:
#	/bin/bash tests/logstash.sh
#
#.PHONY: tag
#tag:
#	docker tag $(NAME):$(VERSION) $(NAME):latest
#
.PHONY: push
push:
	#docker tag -f $(NAME):$(VERSION)  $(PUSH_URL)/$(NAME):$(VERSION)
	docker push                          $(NAME):$(VERSION)
	#docker rmi                        $(PUSH_URL)/$(NAME):$(VERSION)
