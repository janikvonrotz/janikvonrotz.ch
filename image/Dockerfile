FROM alpine:3.14

RUN apk add --no-cache hugo git nginx

WORKDIR /app

COPY . .

RUN git submodule update --init

RUN hugo --minify -d /var/www/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
