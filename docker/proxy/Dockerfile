FROM jwilder/nginx-proxy:1.9.0-alpine

COPY nginx-proxy /usr/bin/nginx-proxy
RUN chmod a+x /usr/bin/nginx-proxy

ENTRYPOINT ["nginx-proxy"]
