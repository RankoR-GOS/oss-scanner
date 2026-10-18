FROM debian:trixie
ENV DEBIAN_FRONTEND=noninteractive \
    CC=clang \
    CXX=clang++ \
    CFLAGS="-g -fno-omit-frame-pointer" \
    CXXFLAGS="-g -fno-omit-frame-pointer"
RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates build-essential clang make git python3 gdb \
    && rm -rf /var/lib/apt/lists/*

COPY . /src
WORKDIR /src

RUN make -j"$(nproc)" CONFIG_NATIVE=false \
    && make -j"$(nproc)" CONFIG_NATIVE=false VARIANT=light \
    && make -j"$(nproc)" CONFIG_NATIVE=false OUT=out-android CONFIG_N_ARENA=1 CONFIG_STATS=true CONFIG_SELF_INIT=false \
        CONFIG_CXX_ALLOCATOR=false
