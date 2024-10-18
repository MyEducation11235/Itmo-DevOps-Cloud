# Используем лёгкий образ Nginx статичной версии 
FROM nginx:1.27.2-alpine-slim

# устанавливаем рабочую директорию
WORKDIR /usr/src/app/

# Открываем порт 80
EXPOSE 80

# Копируем только нужные файлы сайта в директорию Nginx
COPY site /usr/share/nginx/html

# Используем не root пользователя для безопасности 
RUN adduser -D myuser && chown -R myuser /myapp-data
USER myuser

# Запускаем Nginx
CMD ["nginx", "-g", "daemon off;"]
