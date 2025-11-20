.PHONY: help build push dev test test-env

# Docker image name and tag
IMAGE:=foldingathome/fah-gpu-bastet
TAG?=latest
VERSION:=25.11.0
CUDA_RELEASE=11.7.1
ROCM_RELEASE=6.4.4
# Shell that make should use
SHELL:=bash

help:
# http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html
	@grep -E '^[a-zA-Z0-9_%/-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

clean: ## Make clean the build environment
	docker image prune

image: DARGS?=--pull --build-arg CUDA_RELEASE=$(CUDA_RELEASE) --build-arg ROCM_RELEASE=$(ROCM_RELEASE)
image: ## Make the latest build of the image
	docker build $(DARGS) --rm --force-rm -t $(IMAGE):$(TAG) -t $(IMAGE):$(VERSION) \
	-t $(IMAGE):cuda$(CUDA_RELEASE)-rocm$(ROCM_RELEASE) \
	-t $(IMAGE):$(VERSION)-cuda$(CUDA_RELEASE)-rocm$(ROCM_RELEASE) .

nvidia-image: DARGS?=--pull --build-arg CUDA_RELEASE=$(CUDA_RELEASE)
nvidia-image: ## Make nVidia image
	docker build -f Dockerfile.nvidia $(DARGS) --rm --force-rm \
	-t $(IMAGE):cuda$(CUDA_RELEASE) -t $(IMAGE):$(VERSION)-cuda$(CUDA_RELEASE) .

amd-image: DARGS?=--pull --build-arg ROCM_RELEASE=$(ROCM_RELEASE) 
amd-image: ## Make AMD image
	docker build -f Dockerfile.amd $(DARGS) --rm --force-rm \
	-t $(IMAGE):rocm$(ROCM_RELEASE) -t $(IMAGE):$(VERSION)-rocm$(ROCM_RELEASE) .

shell: ## Run the image and enter to the bash
	docker run -it --rm --gpus all --device=/dev/kfd --device=/dev/dri --security-opt seccomp=unconfined \
	--group-add video --group-add $$(getent group render | cut -d: -f3) \
	--user "$$(id -u):$$(id -g)" --entrypoint="" $(IMAGE):$(VERSION) bash

push: ## Push the latest image
	docker push $(IMAGE):$(VERSION)
	docker push $(IMAGE):latest
