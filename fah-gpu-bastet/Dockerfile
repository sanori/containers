# Folding@home GPU Container

# CUDA 11.7.1 is the lowest version of ubuntu22.04 jammy
FROM nvidia/cuda:11.7.1-base-ubuntu22.04
LABEL description="Official Folding@home GPU Container"

ARG ROCM_RELEASE=7.1

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      ocl-icd-opencl-dev \
      clinfo \
      libexpat1 \
      kmod \
      polkitd-pkla \
      policykit-1 \
      python3 \
      python3-websocket \
      curl \
    # point at lib mapped in by container runtime
    && mkdir -p /etc/OpenCL/vendors \
    && echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd \
    # get ROCm OpenCL runtime packages
    && curl -sL https://repo.radeon.com/rocm/rocm.gpg.key | apt-key add - \
    && printf "deb [arch=amd64] https://repo.radeon.com/rocm/apt/${ROCM_RELEASE}/ jammy main" \
        | tee /etc/apt/sources.list.d/rocm.list \
    && apt-get update && apt-get install -y --no-install-recommends \
      rocm-opencl-runtime \
      rocm-smi-lib \
      rocminfo
    # next line gets past the fahclient.postinst
RUN mkdir -p /etc/fahclient && touch /etc/fahclient/config.xml \
    && useradd -ms /bin/bash fah-client \
    && groupadd -r render \
    && usermod -aG render,video fah-client \
    # download and verify checksum
    && curl -fsSL \
      https://download.foldingathome.org/releases/public/fah-client/debian-10-64bit/release/fah-client_8.4.9_amd64.deb \
      -o fah.deb \
    && echo "098f4abf7c172e543de4391e86eacd10655e67d53fd4b0ae05633e2b57f9e07c fah.deb" \
      | sha256sum -c --strict - \
    # install and cleanup
    && DEBIAN_FRONTEND=noninteractive dpkg --install --force-depends fah.deb \
    && rm -rf fah.deb \
    # && apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

ENV LD_LIBRARY_PATH=/opt/rocm/lib
ENV PATH=$PATH:/opt/rocm/bin:/opt/rocm/opencl/bin

WORKDIR "/fah"
VOLUME ["/fah"]
EXPOSE 7396

ENTRYPOINT ["fah-client"]
