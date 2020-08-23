# Dockerfile for apollo-config-server

# Build with:
# docker build -t apollo-config-server:v1.0.0 .

FROM openjdk:8-jre-alpine3.8

RUN \
    echo "http://mirrors.aliyun.com/alpine/v3.8/main" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/v3.8/community" >> /etc/apk/repositories && \
    apk update upgrade && \
    apk add --no-cache procps curl bash tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    mkdir -p /apollo-config-server
RUN addgroup -S apollo && adduser -S apollo -G apollo
ADD . /apollo-config-server/
#RUN echo "apollo:x:1000590000:1000590001::/home/apollo:" >> /etc/passwd
    ## thanks for http://stackoverflow.com/a/1094354/535203 to compute the creation date
#RUN     echo "apollo:!:$(($(date +%s) / 60 / 60 / 24)):0:99999:7:::" >> /etc/shadow
#RUN     echo "apollo:x:1000590000:" >> /etc/group
#RUN     mkdir /home/apollo && chown apollo:apollo /home/apollo
ENV APOLLO_CONFIG_SERVICE_NAME="service-apollo-config-server.sre"
RUN mkdir /opt && chown -R apollo:apollo /opt
RUN chgrp -R 0 /opt && \
    chmod -R g=u /opt


EXPOSE 8080
RUN chown -R apollo:apollo /apollo-config-server

RUN chgrp -R 0 /apollo-config-server && \
    chmod -R g=u /apollo-config-server
#USER apollo
USER apollo

CMD ["/apollo-config-server/scripts/startup-kubernetes.sh"]
