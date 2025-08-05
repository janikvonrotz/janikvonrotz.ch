FROM alpine:3.14

RUN apk add --no-cache hugo

WORKDIR /app

COPY . .

RUN hugo

EXPOSE 1313

CMD ["hugo", "server", "--bind", "0.0.0.0"]
