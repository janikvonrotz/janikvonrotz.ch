FROM alpine:3.14

RUN apk add --no-cache hugo git

WORKDIR /app

COPY . .

RUN git submodule update --init

RUN hugo --minify

EXPOSE 80

CMD ["busybox", "httpd", "-f", "-p", "80", "-h", "/app/public"]
