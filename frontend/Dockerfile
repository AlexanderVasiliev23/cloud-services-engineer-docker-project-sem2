# используем облегченную (alpine) версию node
FROM node:18-alpine AS builder

# задаем рабочую директорию
WORKDIR /app

# сначала собираем зависимости, т.к. они меняются редко и долго. Так сохранится слой, который не нужно будет каждый раз пересобирать
COPY package.json package-lock.json ./
RUN npm ci

# копируем остальные файлы приложения
COPY . .

# пробрасываем переменные окружения:
# NODE_OPTIONS - флаг для решения проболемы совместимости
# NODE_ENV - флаг окружения для сборки
ENV NODE_OPTIONS=--openssl-legacy-provider \
    NODE_ENV=production

# VUE_APP_API_URL - префикс пути для обращения к backend
ARG VUE_APP_API_URL_ARG
ENV VUE_APP_API_URL=${VUE_APP_API_URL_ARG:-/api}

# собираем приложение (готовые файлы будут лежать в каталоге /app/dist)
RUN npm run build

# для работы с файлами, которые сбилдили, будет использоваться nginx (также используем облегченную alpine версию образа)
FROM nginx:alpine

# создаем своего пользователя, чтобы не использовать root
RUN addgroup -S appuser && adduser -S appuser -G appuser -G nginx

# даем нашему пользователю права на файлы, связанные с nginx
RUN chown -R appuser:nginx /var/cache/nginx && \
    chown -R appuser:nginx /var/log/nginx && \
    chown -R appuser:nginx /etc/nginx/conf.d && \
    chmod -R 755 /var/cache/nginx && \
    touch /var/run/nginx.pid && \
    chown appuser:nginx /var/run/nginx.pid

# переносим собранные файлы приложения и конфиг nginx, попутно давая права на них нашему пользователю
COPY --from=builder --chown=appuser:nginx /app/dist /usr/share/nginx/html
COPY --chown=appuser:nginx nginx.conf /etc/nginx/conf.d/default.conf

# переключаемся на нашего пользователя
USER appuser

# проверка работоспособности приложения (почти то же, что и в docker-compose)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://127.0.0.1/ || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]