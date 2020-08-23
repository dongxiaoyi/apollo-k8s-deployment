# Dockerfile for apollo-portal-server

# Build with:
# docker build -t apollo-portal-server:v1.0.0 .

FROM openjdk:8-jre-alpine3.8
#RUN apk add --update sudo

RUN apk add shadow

RUN \
    echo "http://mirrors.aliyun.com/alpine/v3.8/main" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/v3.8/community" >> /etc/apk/repositories  && \
    apk update upgrade && \
    apk add --no-cache procps curl bash tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    mkdir -p /apollo-portal-server

ADD . /apollo-portal-server/
RUN addgroup -S apollo && adduser -S apollo -G apollo

ENV APOLLO_PORTAL_SERVICE_NAME="service-apollo-portal-server.sre"

RUN mkdir /opt && chown -R apollo:apollo /opt
RUN chgrp -R 0 /opt && \
    chmod -R g=u /opt


EXPOSE 8070
RUN chown -R apollo:apollo /apollo-portal-server
RUN chmod 0755 /apollo-portal-server/apollo-portal.jar && chmod +x /apollo-portal-server/apollo-portal.jar

RUN chgrp -R 0 /apollo-portal-server && \
    chmod -R g=u /apollo-portal-server


USER apollo

CMD ["/apollo-portal-server/scripts/startup-kubernetes.sh"]
