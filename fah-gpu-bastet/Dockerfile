# Folding@home GPU Container

# CUDA 11.7.1 is the lowest version of ubuntu22.04 jammy
ARG CUDA_RELEASE=11.7.1

FROM nvidia/cuda:${CUDA_RELEASE}-base-ubuntu22.04
LABEL description="Official Folding@home GPU Container for Bastet"

ARG ROCM_RELEASE=6.4.4
ARG FAH_CLIENT_VERSION
ARG FAH_CLIENT_SHA256SUM
# Check if mandatory arguments have been provided
RUN : "${FAH_CLIENT_VERSION:?FAH_CLIENT_VERSION is required!}"
RUN : "${FAH_CLIENT_SHA256SUM:?FAH_CLIENT_SHA256SUM is required!}"

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      kmod \
      polkitd \
      python3 \
      python3-websocket \
      curl \
    # setup OpenCL ICD for nVidia
    && mkdir -p /etc/OpenCL/vendors \
    && echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd \
    # get ROCm OpenCL runtime packages
    && mkdir -p --mode=0755 /etc/apt/keyrings \
    && curl -sL https://repo.radeon.com/rocm/rocm.gpg.key | gpg --dearmor > /etc/apt/keyrings/rocm.gpg \
    && printf "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/${ROCM_RELEASE}/ jammy main" \
        | tee /etc/apt/sources.list.d/rocm.list \
    && printf 'Package: *\nPin: release o=repo.radeon.com\nPin-Priority: 600' \
        | tee /etc/apt/preferences.d/rocm-pin-600 \
    && apt-get update && apt-get install -y --no-install-recommends \
      rocm-opencl-runtime \
      rocm-smi-lib \
      rocminfo \
    # create libOpenCL.so instead of install ocl-icd-opencl-dev
    && ln -s /usr/lib/x86_64-linux-gnu/libOpenCL.so.1 /usr/lib/x86_64-linux-gnu/libOpenCL.so \
    # add users and groups to properly install fah-client.deb
    && useradd -ms /bin/bash fah-client \
    && groupadd -r render \
    && usermod -aG render,video fah-client \
    && mkdir /fah \
    && chown fah-client:fah-client /fah \
    # download and verify checksum
    && curl -fsSL \
      https://download.foldingathome.org/releases/public/fah-client/debian-10-64bit/release/fah-client_${FAH_CLIENT_VERSION}_amd64.deb \
      -o fah-client.deb \
    && echo "${FAH_CLIENT_SHA256SUM} fah-client.deb" \
      | sha256sum -c --strict - \
    # install and cleanup
    && DEBIAN_FRONTEND=noninteractive dpkg --install --force-depends fah-client.deb \
    && rm -rf fah-client.deb \
    && apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

ENV LD_LIBRARY_PATH=/opt/rocm/lib
ENV PATH=$PATH:/opt/rocm/bin

USER fah-client
WORKDIR "/fah"
VOLUME ["/fah"]
EXPOSE 7396

# allow access outside container (allow option accepts just one entry)
ENTRYPOINT ["fah-client", "--allow", "0/0", "--http-addresses", "0.0.0.0:7396"]
