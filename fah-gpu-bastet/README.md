# Folding@home GPU Container for Bastet (FaH client v8.x)

## TODOs
- [x] How to set fah-client directory to /fah
- [x] https://client.foldingathome.org/
  - By adding https://client.foldingathom.org to allow-origins
- [x] https://v8-4.foldingathome.org/
  - [x] How to connect with login
  - [x] How to connect without login
- [ ] Writing documents
  - [ ] Bastet concepts (User, Machine, web-client)
  - [ ] Clean run
  - [ ] Bastet test
- [x] Refactor ROCm installation
  - [x] Investigate ROCm-docker
  - [x] Investigate whether the image size can be further reduced
- [ ] docker-compose.yml instead of docker run
- [ ] Make rocm only image support OpenCL

## Goals
- Latest Docker image for the new FAH client, bastet (v8.x)
- Support for nVidia, AMD or both
- Provides a sanbox testing environment for the new FAH client and core

## Notes
- GPU drivers must be installed on the host machine since they are part of the OS kernel.
- If using an AMD GPU, pay attention to the `render` group ID associated with `/dev/dri/renderD*` permissions.

## How to run

For nVidia host machine
```bash
# Run container with GPUs, name it "fah0", map user and /fah volume
docker run --gpus all --name fah0 -d --user "$(id -u):$(id -g)" \
  --volume $HOME/fah:/fah foldingathome/fah-gpu-bastet:cuda
```

For AMD GPU host machine
```bash
docker run --device=/dev/kfd --device=/dev/dri \
    --security-opt seccomp=unconfined \
    --group-add video --group-add $(getent group render | cut -d: -f3) \
    --name fah0 -d --user "$(id -u):$(id -g)" \
    --volume $HOME/fah:/fah foldingathome/fah-gpu-bastet:rocm
```

`fah-client` option to connect web client after `client.db` were created
```bash
fah-client --http-addresses="0.0.0.0:7396" --allow="127.0.0.1 172.17.0.1" \
  --allowed-origins="https://v8-4.foldingathome.org/ https://client.foldingathome.org/"
```
