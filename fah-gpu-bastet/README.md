# Folding@home GPU Container for Bastet (FaH client v8.x)

## Goals

- Latest Docker image for the new FAH client, bastet (v8.x)
- Support for nVidia, AMD, or both
- Provides a sandbox testing environment for the new FAH client and core

## Notes

- GPU drivers must be installed on the host machine since they are part of the OS kernel
- If using an AMD GPU, pay attention to the `render` group ID associated with `/dev/dri/renderD*` permissions
- Different from FAH client v7, fah-client Bastet does not read `config.xml` once it creates `client.db`
- You should control fah-client through [v8-4.foldingathome.org](https://v8-4.foldingathome.org/) web control or `fahctl` command
    - Fah-client does not start folding by default.
    - You should enable GPU's and set up how many CPU cores work for the folding.
    - See [V8.4 Client Guide](https://foldingathome.org/guides/v8-4-client-guide/#v8-software-interface)
- V8 client recognizes *you* and *your machines* separately
    - Your data: username, team, passkey
    - Your machine's data: `/etc/machine-id`, machine name, account token
    - If `/etc/machine-id` is changed, previous folding works are **abandoned**
- `Passkey` and `token` are different
    - Passkey: evidence of who **you** are
    - Token: a password of your **machine** to connect to foldingathome.org

## How to run in your desktop

### Using `docker run`

For nVidia host machine
```bash
# Run container with GPUs, name it "fah0", map user and /fah volume
docker run --gpus all --name fah0 -d -p 7396:7396 --user "$(id -u):$(id -g)" \
  -v $HOME/fah:/fah -v /etc/machine-id:/etc/machine-id:ro \
  foldingathome/fah-gpu-bastet:cuda
```

For AMD GPU host machine
```bash
docker run --device=/dev/kfd --device=/dev/dri \
  --security-opt seccomp=unconfined \
  --group-add video --group-add "$(getent group render | cut -d: -f3)" \
  --name fah0 -d -p 7396:7396 --user "$(id -u):$(id -g)" \
  -v $HOME/fah:/fah -v /etc/machine-id:/etc/machine-id:ro \
  foldingathome/fah-gpu-bastet:rocm
```

For both
```bash
docker run --gpus all --device=/dev/kfd --device=/dev/dri \
  --security-opt seccomp=unconfined \
  --group-add video --group-add "$(getent group render | cut -d: -f3)" \
  --name fah0 -d -p 7396:7396 --user "$(id -u):$(id -g)" \
  -v $HOME/fah:/fah -v /etc/machine-id:/etc/machine-id:ro \
  foldingathome/fah-gpu-bastet:cuda-rocm
```

After the container is set up, you can access through [v8-4.foldingathome.org](https://v8-4.foldingathome.org/)

### Using `docker compose`

Before start, you should check `compose-{platform}.yml` file and uncomment or change options.

For nVidia host machine
```bash
docker compose -f compose-cuda.yml up -d
```

For AMD GPU host machine
```bash
export RENDER_GID="$(getent group render | cut -d: -f3)"
docker compose -f compose-rocm.yml up -d
```

You can set `RENDER_GID` variable in the `.env` file.
```sh
RENDER_GID=999  # You should replace 999 to the render's group ID of your machine
```

## How to build custom image

You may set CUDA and ROCm versions by setting `.env` file because `docker compose` consults the file. Please check `.env-example` file as an example.

For nVidia host machine
```bash
docker compose -f compose-cuda.yml build --pull
```

For AMD GPU host machine
```bash
docker compose -f compose-rocm.yml build --pull
```

For both
```bash
docker compose build --pull
```

## Maintenance Tips

You can reduce the size of `client.db` file **when fah-client is not running** by running this command:
```bash
sqlite3 client.db "VACUUM;"
```
