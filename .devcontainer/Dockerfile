FROM quay.io/ascend/cann:8.5.0-910b-ubuntu22.04-py3.11

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    wget curl git vim pre-commit gh tmux openssh-client ca-certificates gnupg \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 20
RUN curl -fsSL https://deb.nodesource.com/setup_20.x -o /tmp/nodesource_setup.sh \
    && bash /tmp/nodesource_setup.sh \
    && apt-get update \
    && apt-get install -y nodejs \
    && node --version \
    && npm --version \
    && rm -f /tmp/nodesource_setup.sh \
    && rm -rf /var/lib/apt/lists/*

# for on-device execution
RUN pip install --no-cache-dir torch==2.9.0 --index-url https://download.pytorch.org/whl/cpu \
    && pip install --no-cache-dir torch-npu==2.9.0 \
    && pip install --no-cache-dir numpy pyyaml \
    && pip install --no-cache-dir black prospector

# extra util
RUN pip install --no-cache-dir \
    pytest pybind11 nanobind setuptools wheel \
    ipython jupyterlab matplotlib pandas

RUN npm install -g @anthropic-ai/claude-code
RUN npm install -g @openai/codex
npm install -g @github/copilot
# NOTE: only for claude-code CLI, no need to install this if using remote vscode extensions

# change this ununsed arg if need to force rebuild later lines
ARG CACHE_BURST=1

WORKDIR /sources

#  ==== tilelang-ascend install ====
# (pin commit on 2026/04/16) https://github.com/tile-ai/tilelang-ascend/commit/19cde6d91cfa01fd262b2faa6345c7bcec5a81c2
# includes the new PTO backend example: https://github.com/tile-ai/tilelang-ascend/pull/367
ARG TILELANG_ASCEND_COMMIT=19cde6d91cfa01fd262b2faa6345c7bcec5a81c2
RUN git clone --recursive https://github.com/tile-ai/tilelang-ascend.git
RUN cd /sources/tilelang-ascend && git checkout $TILELANG_ASCEND_COMMIT && bash install_ascend.sh
RUN echo "source /sources/tilelang-ascend/set_env.sh" >> ~/.bashrc
#  ==== end of tilelang-ascend install ====


# ==== pto-dsl dependencies ====
# mirros https://github.com/huawei-csl/pto-dsl/blob/main/docker/Dockerfile

# certain operations need latest isa header, not CANN 8.5.0 default
# (pin commit on 2026/04/16)
ARG PTOISA_COMMIT=ef7ba7181de5c5db6bdb4867d87c0257fdc2e84f
WORKDIR /sources
RUN git clone https://gitcode.com/cann/pto-isa.git \
    && cd pto-isa && git checkout $PTOISA_COMMIT

# ARG ARCH=x86_64
ARG ARCH=aarch64
ARG RELEASE_REPO=zhangstevenunity/PTOAS
ARG RELEASE_VER=0.9
ARG RELEASE_TAG=v${RELEASE_VER}
ARG WHEEL_NAME=ptoas-${RELEASE_VER}-cp311-none-manylinux_2_34_${ARCH}.whl
ARG CLI_TAR_NAME=ptoas-bin-${ARCH}.tar.gz

WORKDIR /installers/
RUN wget https://github.com/${RELEASE_REPO}/releases/download/${RELEASE_TAG}/${WHEEL_NAME}
RUN pip install ./${WHEEL_NAME}
RUN python -c "import mlir.ir; from mlir.dialects import pto"

RUN wget https://github.com/${RELEASE_REPO}/releases/download/${RELEASE_TAG}/${CLI_TAR_NAME}

# set up correct lib path for `ptoas` cli
ARG CLI_DIR=/installers/ptoas-cli
RUN mkdir $CLI_DIR && tar zxf ./${CLI_TAR_NAME} -C $CLI_DIR
RUN chmod +x $CLI_DIR/bin/ptoas
ENV PATH=$CLI_DIR/bin:$PATH
ENV LD_LIBRARY_PATH=$CLI_DIR/lib:$LD_LIBRARY_PATH
# ==== end of pto-dsl dependencies =====

WORKDIR /sources