FROM node:24.0.2

WORKDIR "/var/www/app"

COPY initContainerNode /usr/bin/initContainerNode
RUN chmod a+x /usr/bin/initContainerNode

ENTRYPOINT ["initContainerNode"]