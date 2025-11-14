# Folding@home GPU Container for Bastet (FaH client v8.x)

## TODOs
- [x] How to set fah-client directory to /fah
- [ ] https://client.foldingathome.org/
  - Is there no interface to old one?
- [ ] https://v8-4.foldingathome.org/
  - [x] How to connect with login
  - [ ] How to connect without login
- [ ] Writing documents

## Goals
- Latest Docker image for the new FAH client, bastet (v8.x)
- Support for nVidia, AMD and both
- Provides a sanbox testing environment for the new FAH client and core

## Notes
- GPU drivers must be installed on the host machine since they are part of the OS kernel.
- If using an AMD GPU, pay attention to the `render` group ID associated with `/dev/dri/renderD*` permissions.

## How to run

For nVidia host machine
```bash
# Run container with GPUs, name it "fah0", map user and /fah volume
docker run --gpus all --name fah0 -d --user "$(id -u):$(id -g)" \
  --volume $HOME/fah:/fah foldingathome/fah-gpu-bastet
```

For AMD GPU host machine
```bash
docker run --device=/dev/kfd --device=/dev/dri \
    --security-opt seccomp=unconfined \
    --group-add video --group-add $(getent group render | cut -d: -f3) \
    --name fah0 -d --user "$(id -u):$(id -g)" \
    --volume $HOME/fah:/fah foldingathome/fah-gpu-bastet
```
