FROM golang:1.16-alpine as gobuild

WORKDIR /build
ADD . /build

RUN go get -d -v ./...
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o ./s3driver ./cmd/s3driver

FROM debian:stretch as s3fsbuild

WORKDIR /build

RUN apt-get update && apt-get install -y git make \
    && git clone -b ignore_change_owner_err https://github.com/arugal/s3fs-fuse.git \
    && cd s3fs-fuse \
    && .github/workflows/linux-ci-helper.sh debian:stretch \
    && ./autogen.sh \
    && ./configure --prefix=/usr/local/s3fs \
    && make \
    && make install

FROM debian:buster-slim
LABEL maintainers="Cyrill Troxler <cyrilltroxler@gmail.com>"
LABEL description="csi-s3 slim image"

COPY --from=gobuild /build/s3driver /s3driver
COPY --from=s3fsbuild /usr/local/s3fs /usr/local/s3fs

# some other dependencies
RUN apt-get update && \
  apt-get install -y curl unzip && \
  rm -rf /var/lib/apt/lists/*

# s3fs env
RUN export PATH=$PATH:/usr/local/s3fs/bin

# install rclone
ARG RCLONE_VERSION=v1.54.1
RUN cd /tmp \
  && curl -O https://downloads.rclone.org/${RCLONE_VERSION}/rclone-${RCLONE_VERSION}-linux-amd64.zip \
  && unzip /tmp/rclone-${RCLONE_VERSION}-linux-amd64.zip \
  && mv /tmp/rclone-*-linux-amd64/rclone /usr/bin \
  && rm -r /tmp/rclone*

ENTRYPOINT ["/s3driver"]
