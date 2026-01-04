FROM node:25.2.1

WORKDIR "/var/www/app"

COPY initContainerNode /usr/bin/initContainerNode
RUN chmod a+x /usr/bin/initContainerNode

ENTRYPOINT ["initContainerNode"]