.PHONY: help build push dev test test-env

# Docker image name and tag
IMAGE:=sanori/fah-gpu-bastet
TAG?=latest
VERSION:=25.11.0
# Shell that make should use
SHELL:=bash

help:
# http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html
	@grep -E '^[a-zA-Z0-9_%/-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

clean: ## Make clean the build environment
	docker image prune

image: DARGS?=--pull --build-arg version=$(VERSION)
image: ## Make the latest build of the image
	docker build $(DARGS) --rm --force-rm -t $(IMAGE):$(TAG) -t $(IMAGE):$(VERSION) .

shell: ## Run the image and enter to the bash
	docker run -it --rm --gpus all --device=/dev/kfd --device=/dev/dri --security-opt seccomp=unconfined \
	--group-add video --group-add $$(getent group render | cut -d: -f3) \
	--user "$$(id -u):$$(id -g)" --entrypoint="" $(IMAGE):$(VERSION) bash

push: ## Push the latest image
	docker push $(IMAGE):$(VERSION)
	docker push $(IMAGE):latest
