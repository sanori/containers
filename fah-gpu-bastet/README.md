# Folding@home GPU Container for Bastet (FaH client v8.x)

## Goals

- Latest Docker image for the new FAH client, Bastet (v8.x).
- Support for nVidia, AMD, or both.
- Provides a sandbox testing environment for the new FAH client and core.


## Notes

- GPU drivers must be installed on the host machine since they are part of the OS kernel.
    - nVidia: A package with a name similar to `nvidia-driver-???`
    - AMD: In most cases, these are already included in the base kernel.
- If using an AMD GPU, pay attention to the `render` group ID associated with `/dev/dri/renderD*` permissions.
- Unlike FAH client v7, fah-client Bastet does not read `config.xml` after creating `client.db`.
- You should control fah-client via the [v8-4.foldingathome.org](https://v8-4.foldingathome.org/) web control or `fahctl` command.
    - Fah-client does not start folding by default.
    - You must enable GPUs and configure the number of CPU cores to use for folding.
    - For more details, see [V8.4 Client Guide](https://foldingathome.org/guides/v8-4-client-guide/#v8-software-interface).
- The V8 client recognizes *users* and user *machines* separately.
    - User data: username, team, passkey
    - User machine data: `/etc/machine-id`, machine name, account token
    - If `/etc/machine-id` is changed, previous folding works will be **invalid**.
- `Passkey` and `token` are different.
    - Passkey: evidence of who the **user** is
    - Token: **machine** password used to connect to foldingathome.org


## How to run on your desktop

### Using `docker run`

For the nVidia host machine
```bash
# Run container with GPUs, name it "fah0", map user and /fah volume
docker run --gpus all --name fah0 -d -p 7396:7396 --user "$(id -u):$(id -g)" \
  -v $HOME/fah:/fah -v /etc/machine-id:/etc/machine-id:ro \
  foldingathome/fah-gpu-bastet:cuda
```

For the AMD GPU host machine
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

Once the container starts, you can access it with [v8-4.foldingathome.org](https://v8-4.foldingathome.org/)


### Using `docker compose`

Before you begin, check the `compose-{platform}.yml` file and uncomment any option or change any options.

For the nVidia host machine
```bash
docker compose -f compose-cuda.yml up -d
```

For the AMD GPU host machine
```bash
export RENDER_GID="$(getent group render | cut -d: -f3)"
docker compose -f compose-rocm.yml up -d
```

For both
```bash
export RENDER_GID="$(getent group render | cut -d: -f3)"
docker compose up -d
```

**Note:** You can set the `RENDER_GID` variable in your `.env` file.
```sh
RENDER_GID=999  # replace 999 with your machine's render group ID
```


## How to build a custom image

You may set the CUDA and ROCm versions by creating the `.env` file, as `docker compose` consults it. Please check the `.env-example` file as an example.

For the nVidia host machine
```bash
docker compose -f compose-cuda.yml build --pull
```

For the AMD GPU host machine
```bash
docker compose -f compose-rocm.yml build --pull
```

For both
```bash
docker compose build --pull
```


## Maintenance Tips

You can reduce the size of the `client.db` file **when fah-client is not running** by running this command:
```bash
sqlite3 client.db "VACUUM;"
```
