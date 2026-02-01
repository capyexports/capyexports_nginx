# nginx/Dockerfile - Build context: nginx/ (docker build -f nginx/Dockerfile nginx/)
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY conf.d/ /etc/nginx/conf.d/

HEALTHCHECK --interval=10s --timeout=3s --start-period=2s --retries=3 \
  CMD wget -q -O- http://127.0.0.1/__nginx_health__ || exit 1

EXPOSE 80 443
