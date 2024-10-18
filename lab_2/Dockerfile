# Используем лёгкий образ Nginx статичной версии 
FROM nginx:1.27.2-alpine-slim

# устанавливаем рабочую директорию
WORKDIR /usr/src/app/

# Открываем порт 80
EXPOSE 80

# Копируем только нужные файлы сайта в директорию Nginx
ARG SITE_PATH
COPY $SITE_PATH /usr/share/nginx/html

# Используем не root пользователя для безопасности (не требуется в nginx образе)
# RUN adduser -D myuser && chown -R myuser ./
# USER myuser

# Запускаем Nginx
CMD ["nginx", "-g", "daemon off;"]
