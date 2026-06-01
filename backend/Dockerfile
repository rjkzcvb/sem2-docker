# ========== ЭТАП 1: СБОРКА ==========
FROM golang:1.21-alpine AS builder

# Устанавливаем необходимые системные пакеты
RUN apk add --no-cache git

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем файлы с зависимостями
COPY go.mod go.sum ./

# Скачиваем зависимости
RUN go mod download

# Копируем весь остальной код
COPY . .

# Компилируем из папки cmd/api (где лежит main.go)
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o backend ./cmd/api/

# ========== ЭТАП 2: ФИНАЛЬНЫЙ ОБРАЗ ==========
FROM alpine:latest

# Устанавливаем ca-certificates для HTTPS-запросов
RUN apk --no-cache add ca-certificates

# Создаём непривилегированного пользователя
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем скомпилированный бинарник из стадии builder
COPY --from=builder /app/backend .

# Даём права на выполнение и меняем владельца
RUN chmod +x /app/backend && \
    chown appuser:appgroup /app/backend

# Переключаемся на непривилегированного пользователя
USER appuser

# Открываем порт (бэкенд слушает 8081)
EXPOSE 8081

# Healthcheck
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8081/health || exit 1

# Запуск
CMD ["./backend"]
