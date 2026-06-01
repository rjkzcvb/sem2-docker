# ========== ЭТАП 1: СБОРКА ==========
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENV NODE_OPTIONS=--openssl-legacy-provider
RUN npm run build

# ========== ЭТАП 2: NGINX ==========
FROM nginx:stable-alpine

# Удаляем дефолтные файлы
RUN rm -rf /usr/share/nginx/html/* && \
    rm -f /etc/nginx/conf.d/default.conf

# Копируем статику
COPY --from=builder /app/dist /usr/share/nginx/html

# Создаем все необходимые директории
RUN mkdir -p /var/run/nginx /var/log/nginx && \
    chown -R nginx:nginx /var/run/nginx /var/log/nginx && \
    chown -R nginx:nginx /usr/share/nginx/html && \
    chmod -R 755 /usr/share/nginx/html

# Временно запускаем от root (для диагностики)
# USER nginx  # Временно закомментируем

EXPOSE 80

# Запускаем nginx
CMD ["nginx", "-g", "daemon off;"]
