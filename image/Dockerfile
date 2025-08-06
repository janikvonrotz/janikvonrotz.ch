# --- Stage 1: Build the Hugo site ---
FROM alpine:3.14 as builder

RUN apk add --no-cache hugo git

WORKDIR /app

COPY . .

RUN git submodule update --init

RUN hugo --minify -d /public

# --- Stage 2: Serve with Nginx ---
FROM alpine:3.14

RUN apk add --no-cache nginx

COPY --from=builder /public /var/www/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
