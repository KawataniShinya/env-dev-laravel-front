# syntax = docker/dockerfile:1.2

FROM jwilder/nginx-proxy:1.3.0-alpine

COPY nginx-proxy /usr/bin/nginx-proxy
RUN chmod a+x /usr/bin/nginx-proxy

ENTRYPOINT ["nginx-proxy"]
