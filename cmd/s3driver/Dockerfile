FROM golang:1.16-alpine as gobuild

WORKDIR /build
ADD . /build

RUN go get -d -v ./...
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o ./s3driver ./cmd/s3driver

FROM debian:buster-slim
LABEL maintainers="Cyrill Troxler <cyrilltroxler@gmail.com>"
LABEL description="csi-s3 slim image"

COPY --from=gobuild /build/s3driver /s3driver

# some other dependencies
RUN apt-get update && \
  apt-get install -y curl unzip git make && \
  rm -rf /var/lib/apt/lists/*

# install s3fs
RUN cd /tmp \
    && git clone -b ignore_change_owner_err https://github.com/arugal/s3fs-fuse.git \
    && cd s3fs-fuse \
    && .github/workflows/linux-ci-helper.sh debian:stretch \
    && ./autogen.sh && ./configure --prefix=/usr/local/s3fs \
    && make && make install \
    && cd /tmp && rm -r /tmp/s3fs-fuse

# install rclone
ARG RCLONE_VERSION=v1.54.1
RUN cd /tmp \
  && curl -O https://downloads.rclone.org/${RCLONE_VERSION}/rclone-${RCLONE_VERSION}-linux-amd64.zip \
  && unzip /tmp/rclone-${RCLONE_VERSION}-linux-amd64.zip \
  && mv /tmp/rclone-*-linux-amd64/rclone /usr/bin \
  && rm -r /tmp/rclone*

ENV PATH=$PATH:/usr/local/s3fs/bin

ENTRYPOINT ["/s3driver"]
