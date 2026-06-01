# ========== ЭТАП 1: СБОРКА ==========
# Используем официальный образ Go на Alpine (легковесный)
FROM golang:1.21-alpine AS builder

# Устанавливаем необходимые системные пакеты (например, git для некоторых зависимостей)
# И сразу удаляем кэш apk, чтобы не раздувать слой
RUN apk add --no-cache git

# Устанавливаем рабочую директорию внутри контейнера
WORKDIR /app

# Копируем файлы с зависимостями (этот слой закэшируется, если go.mod не менялся)
COPY go.mod go.sum ./

# Скачиваем зависимости
RUN go mod download

# Копируем весь остальной код
COPY . .

# Компилируем статический бинарник без отладочной информации и с отключенным CGO
# CGO_ENABLED=0 позволяет получить полностью статический бинарь, который будет работать в scratch/alpine
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o backend .

# ========== ЭТАП 2: ФИНАЛЬНЫЙ ОБРАЗ ==========
# Используем минимальный образ Alpine (можно также distroless, но для начала alpine проще)
FROM alpine:latest

RUN apk --no-cache add ca-certificates

# Создаём непривилегированного пользователя
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем скомпилированный бинарник из стадии builder
COPY --from=builder /app/backend .

# Даём права на выполнение (бинарник должен быть исполняемым)
RUN chmod +x /app/backend

# Меняем владельца файла на нашего непривилегированного пользователя
RUN chown appuser:appgroup /app/backend

# Переключаемся на непривилегированного пользователя
USER appuser

# Открываем порт ( для бэкенда указан порт 8081)
EXPOSE 8081

# Healthcheck: проверяем, отвечает ли приложение на запрос (например, GET /health)
# Интервал, таймаут, порог ошибок — подбираются под приложение
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8081/health || exit 1

# Запуск бинарника
CMD ["./backend"]
