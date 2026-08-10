# =========================
# Build Stage
# =========================

FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build -- --no-progress


# =========================
# Runtime Stage
# =========================

FROM nginxinc/nginx-unprivileged:alpine

COPY --from=builder /app/dist/tutorial-ui/ /usr/share/nginx/html

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
