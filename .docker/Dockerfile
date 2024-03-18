FROM alpine:3.15 as builder

ARG HAVEN_VERSION=v3.0.8
ARG TARGETOS
ARG TARGETARCH
ARG TARGETVARIANT
ARG DEBIAN_FRONTEND=noninteractive

RUN apk --no-cache add git
RUN apk --no-cache add bash
RUN apk --no-cache add build-base
RUN apk --no-cache add patch
RUN apk --no-cache add cmake
RUN apk --no-cache add openssl-dev
RUN apk --no-cache add linux-headers
RUN apk --no-cache add zeromq-dev
RUN apk --no-cache add libexecinfo-dev
RUN apk --no-cache add libunwind-dev
RUN apk --no-cache add boost-dev
RUN apk --no-cache add boost-static
RUN apk --no-cache add curl-static
RUN apk --no-cache add zip

WORKDIR /explorer
COPY . .
RUN git submodule update --init --recursive

# Building Zephyr
WORKDIR src/haven-main/build/Linux/master/release/
RUN cmake -D CMAKE_BUILD_TYPE=Release -D STACK_TRACE:BOOL=OFF ../../../..
RUN cmake --build . --target wallet --
RUN cmake --build . --target daemonizer --
RUN cmake --build . --target cryptonote_protocol --

WORKDIR /explorer/build

RUN cmake -D STACK_TRACE:BOOL=OFF -DMONERO_DIR=$PWD/../src/haven-main ..
RUN make -j2

RUN zip /lib.zip $(ldd xmrblocks | grep -E '/[^\ ]*' -o)

FROM alpine:3.15 as runner

RUN apk add zip

COPY --from=builder /lib.zip .
RUN unzip -o lib.zip && rm -rf lib.zip

WORKDIR /explorer
COPY --from=builder /explorer/build/xmrblocks /explorer/
COPY --from=builder /explorer/build/templates /explorer/templates/