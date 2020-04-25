FROM alpine:3.11

RUN apk update && \
  apk add python py-pip py-setuptools git ca-certificates && \
  pip install python-dateutil

RUN git clone --depth=1 --branch=v2.1.0 https://github.com/s3tools/s3cmd.git /opt/s3cmd && \
    ln -s /opt/s3cmd/s3cmd /usr/bin/s3cmd

ENTRYPOINT ["s3cmd"]
