FROM alpine:3.14 as builder
RUN apk add --no-cache hugo git
WORKDIR /app
COPY . .
RUN git clone https://codeberg.org/janikvonrotz/hugo-new-css-theme.git themes/hugo-new-css-theme

RUN hugo --minify -d /public

FROM caddy:2-alpine
COPY --from=builder /public /srv
COPY <<EOF /etc/caddy/Caddyfile
:80 {
    root * /srv
    file_server
    try_files {path} {path}/ /index.html

    # Compression
    encode gzip

    # Security headers
    header {
        X-Content-Type-Options nosniff
        X-Frame-Options DENY
        X-XSS-Protection "1; mode=block"
        Referrer-Policy strict-origin-when-cross-origin
    }

    # Cache static assets
    @static {
        file
        path *.css *.js *.png *.jpg *.jpeg *.gif *.ico *.svg *.woff *.woff2 *.ttf *.eot
    }
    header @static Cache-Control "public, max-age=31536000, immutable"
}
EOF

EXPOSE 80
