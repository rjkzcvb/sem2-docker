# ========== ЭТАП 1: СБОРКА ФРОНТЕНДА ==========
FROM node:18-alpine AS builder

WORKDIR /app

# Копируем файлы зависимостей
COPY package*.json ./

# Устанавливаем зависимости (--frozen-lockfile для CI-надёжности)
RUN npm ci

# Копируем исходный код
COPY . .

# Собираем статические файлы
RUN npm run build

# ========== ЭТАП 2: СЕРВИНГ СТАТИКИ ЧЕРЕЗ NGINX ==========
# Используем официальный легковесный образ nginx на Alpine
FROM nginx:stable-alpine

# Удаляем стандартную конфигурацию и статику
RUN rm -rf /usr/share/nginx/html/* && \
    rm /etc/nginx/conf.d/default.conf

# Копируем собранные файлы из предыдущего этапа
COPY --from=builder /app/dist /usr/share/nginx/html

# (Опционально) Создаём свой конфиг для nginx, если нужно кастомное поведение
# COPY nginx.conf /etc/nginx/conf.d/default.conf

# Создаём непривилегированного пользователя (в образе nginx-alpine уже есть пользователь nginx)
# Просто убедимся, что он владеет файлами
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chmod -R 755 /usr/share/nginx/html

# Переключаемся на непривилегированного пользователя nginx
USER nginx

# Открываем порт 80 
EXPOSE 80

# Healthcheck для nginx (проверяем, доступна ли index.html)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

# Запускаем nginx в foreground (так nginx и работает внутри контейнера)
CMD ["nginx", "-g", "daemon off;"]
