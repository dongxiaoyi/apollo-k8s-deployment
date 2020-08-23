# Dockerfile for apollo-admin-server

# Build with:
# docker build -t apollo-admin-server:v1.0.0 .

FROM openjdk:8-jre-alpine3.8

RUN \
    echo "http://mirrors.aliyun.com/alpine/v3.8/main" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/v3.8/community" >> /etc/apk/repositories && \
    apk update upgrade && \
    apk add --no-cache procps curl bash tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    mkdir -p /apollo-admin-server

ADD . /apollo-admin-server/
RUN addgroup -S apollo && adduser -S apollo -G apollo


RUN mkdir /opt && chown -R apollo:apollo /opt
ENV APOLLO_ADMIN_SERVICE_NAME="service-apollo-admin-server.sre"
RUN chgrp -R 0 /opt && \
    chmod -R g=u /opt

#RUN addgroup -S -g 1000 apollo && adduser -S -G apollo -u 999 apollo

EXPOSE 8090
RUN chown -R apollo:apollo /apollo-admin-server
RUN chgrp -R 0 /apollo-admin-server && \
    chmod -R g=u /apollo-admin-server
USER apollo
CMD ["/apollo-admin-server/scripts/startup-kubernetes.sh"]
