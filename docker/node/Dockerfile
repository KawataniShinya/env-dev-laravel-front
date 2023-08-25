FROM node:19.9.0

WORKDIR "/var/www/app"

COPY initContainerNode /usr/bin/initContainerNode
RUN chmod a+x /usr/bin/initContainerNode

ENTRYPOINT ["initContainerNode"]