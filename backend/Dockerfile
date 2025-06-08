# используем облегченную (alpine) версию golang
FROM golang:1.23-alpine AS builder

# задаем рабочую директорию
WORKDIR /app

# сначала собираем зависимости, т.к. они меняются редко и долго. Так сохранится слой, который не нужно будет каждый раз пересобирать
COPY go.mod go.sum ./
RUN go mod download

# копируем остальные файлы приложения
COPY . .

# собираем приложение (готовые файлы будут лежать в каталоге /app/dist)
# CGO_ENABLED=0 - делаем итоговый бинарник полностью статическим (не зависим от системных библиотек, меньше размер)
# -ldflags="-w -s" - отключаем отладочную информацию для облегчения бинарника
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /app/myapp /app/cmd/api

# используем облегченный образ "без всего"
FROM alpine:3.18

# создаем своего пользователя, чтобы не использовать root
RUN addgroup -S appuser && adduser -S appuser -G appuser

# переносим готовый бинарник
COPY --from=builder /app/myapp /app/myapp

# даем нашему пользователю права на каталог, в котором будет находиться приложение
RUN chown -R appuser:appuser /app

# переключаемся на нашего пользователя
USER appuser

# проверка работоспособности приложения (почти то же, что и в docker-compose)
HEALTHCHECK --interval=2s --timeout=10s --start-period=2s \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8081/health || exit 1

EXPOSE 8081

CMD ["/app/myapp"]