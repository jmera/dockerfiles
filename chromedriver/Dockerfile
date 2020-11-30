FROM alpine:20200917
MAINTAINER Julich Mera <julich.mera@gmail.com>

RUN apk add --upgrade --no-cache chromium chromium-chromedriver
RUN adduser -D driver

USER driver
WORKDIR /home/driver/
EXPOSE 9515

CMD ["/usr/bin/chromedriver"]
