FROM alpine:3.14

RUN apk add --no-cache hugo git

WORKDIR /app

COPY . .

RUN git submodule update --init

RUN hugo

EXPOSE 1313

CMD ["hugo", "server", "--bind", "0.0.0.0"]
