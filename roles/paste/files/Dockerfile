FROM golang:alpine3.8 AS build

COPY . /go/src/github.com/virtualsnow/linx-server
WORKDIR /go/src/github.com/virtualsnow/linx-server

RUN set -ex \
        && apk add --no-cache --virtual .build-deps git \
        && go get -v . \
        && apk del .build-deps

FROM alpine:3.8

COPY --from=build /go/bin/linx-server /usr/local/bin/linx-server

ENV GOPATH /go
ENV SSL_CERT_FILE /etc/ssl/cert.pem

COPY static /go/src/github.com/virtualsnow/linx-server/static/
COPY templates /go/src/github.com/virtualsnow/linx-server/templates/
COPY conf.ini /go/src/github.com/virtualsnow/linx-server/conf.ini

RUN mkdir -p /data/files && mkdir -p /data/meta && chown -R 65534:65534 /data

VOLUME ["/data/files", "/data/meta"]

EXPOSE 8080
USER nobody
ENTRYPOINT ["/usr/local/bin/linx-server", "-config=/go/src/github.com/virtualsnow/linx-server/conf.ini"]

