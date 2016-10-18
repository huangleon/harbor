FROM golang:1.6.2

MAINTAINER jiangd@vmware.com

ENV http_proxy="http://16.216.235.20:8080" https_proxy="http://16.216.235.20:8080"

RUN apt-get update \
    && apt-get install -y libldap2-dev \
    && rm -r /var/lib/apt/lists/*

COPY . /go/src/github.com/vmware/harbor

WORKDIR /go/src/github.com/vmware/harbor/jobservice

RUN go build -v -a -o /go/bin/harbor_jobservice \
    && chmod u+x /go/bin/harbor_jobservice 
WORKDIR /go/bin/

ENV http_proxy="" https_proxy=""

ENTRYPOINT ["/go/bin/harbor_jobservice"]
