# Лабораторная работа №1 со звёздочкой

**Выполнили:** Недиков Михаил, Проскуряков Роман и Зюзин Владислав

---

**Техническое задание**

- Попробовать взломать nginx другой команды.
- Проверить минимум три уязвимости.
- В отчёт приложить скрины попыток взлома, описание уязвимостей, на которые проверяли, и итог — успешен взлом или нет.

---

**Шаги решения**

1. Договориться с командой о взломе их NGINX-конфига
2. Изменения конфига
3. Path traversal уязвимость
4. Перебор страниц через ffuf
5. Уязвимость CVE-2013-2028 (переполнение буфера)
6. Общий вывод

---

**Система**

- Windows 11
- Версия NGINX у противоположной команды: 1.27.1

---

**Договориться с командой о взломе их NGINX-конфига**

Первым делом мы просмотрели некоторые репозитории для определения нашей будущей "жертвы". Мы выбрали следующую работу: [репозиторий](https://github.com/cicada-pops/devOps-clouds-labs/tree/main/lab-1-nginx).

Также мы спросили разрешение и удостоверились, что никто пока что не "ломал" конфиг владельца лабораторной работы:

*—скрин—*

---

**Изменения конфига**

На момент получения конфиг выглядел так:

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    server {
        listen 80;
        server_name project1.com project2.com;

        location / {
            return 301 https://$host$request_uri;
        }
    }

    server {
        listen 443 ssl;
        server_name project1.com;

        ssl_certificate /usr/local/etc/nginx/ssl/project1.crt;
        ssl_certificate_key /usr/local/etc/nginx/ssl/project1.key;

        root /Users/cicada/project1/;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location /images/ {
            alias /Users/cicada/project1/images/;
        }

        location /assets/ {
            alias /Users/cicada/project1/assets/;
        }
    }

    server {
        listen 443 ssl;
        server_name project2.com;

        ssl_certificate /usr/local/etc/nginx/ssl/project2.crt;
        ssl_certificate_key /usr/local/etc/nginx/ssl/project2.key;

        root /Users/cicada/project2/;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location /images/ {
            alias /Users/cicada/project2/images/;
        }

        location /assets/ {
            alias /Users/cicada/project2/assets/;
        }
    }
}
```

Однако, я не могу запустить его на своей системе, поэтому изменил пути до сертификатов, папок хранения изображений и ассетов:
```nginx

#user  nobody;  
worker_processes  1;

#error_log  logs/error.log;  
#error_log  logs/error.log  notice;  
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {  
    worker_connections  1024;  
}

http {

    server {  
        listen 80;  
        server_name project1.com project2.com;

        location / {  
            return 301 https://$host$request_uri;  
        }  
    }

    server {  
        listen 443 ssl;  
        server_name project1.com;

        ssl_certificate "C:/Users/nedik/Downloads/server.crt";  
        ssl_certificate_key "C:/Users/nedik/Downloads/server.key";

        root C:/Users/nedik/Downloads/testSite/;  
        index index.html;

        location / {  
            try_files $uri $uri/ =404;  
        }

        location /images/ {  
            alias C:/Users/nedik/Downloads/testSite/images/;  
        }

        location /assets/ {  
            alias C:/Users/nedik/Downloads/testSite/assets/;  
        }  
    }

    server {  
        listen 443 ssl;  
        server_name project2.com;

        ssl_certificate "C:/Users/nedik/Downloads/server.crt";  
        ssl_certificate_key "C:/Users/nedik/Downloads/server.key";

        root C:/Users/nedik/Downloads/testSite2/;  
        index index.html;

        location / {  
            try_files $uri $uri/ =404;  
        }

        location /images/ {  
            alias C:/Users/nedik/Downloads/testSite2/images/;  
        }

        location /assets/ {  
            alias C:/Users/nedik/Downloads/testSite2/assets/;  
        }  
    }  
}
```

**Данные изменения никак не влияют на поиск уязвимостей. Однако сервер теперь успешно работает:**

**--скрин--**

---

**Path traversal уязвимость**

**Описание:** Path traversal уязвимость возникает, когда веб-приложение неправильно обрабатывает входные данные и позволяет злоумышленникам получать доступ к файлам и директориям на сервере, которые находятся за пределами корневой директории приложения. Это может привести к чтению конфиденциальных файлов, таких как конфигурационные файлы, базы данных, или пароли.

**Попытка взлома:** для проверки данной уязвимости мы разместим в папку `C:/Users/nedik/Downloads/testSite/images/` файл `test.png`, а в `C:/Users/nedik/Downloads/` файл `test.jpg`. Попробуем получить файл по URL `https://project1.com/images/test.png`:

**--скрин--**

Теперь попробуем получить доступ к файлу `test.jpg`, который находится на две папки выше исходного полученного файла по URL `https://project1.com/images/../../test.jpg`:

**--скрин--**

**Вывод:** Как видим, мы получили 404 ошибку. Другие проверки также выдают данную ошибку для обоих доменов `project1.com` и `project2.com`, ибо у них почти идентичное строение, что говорит о том, что у сервера отсутствует Path traversal уязвимость. Если обратимся к коду, то увидим, что в строке `location /images/` имеется слеш на конце, если бы его не было, то в уязвимом примере Nginx будет сопоставлять любые URL-адреса, начинающиеся с `/images`. Это означает, что и запрос `/images/test.png`, и запрос `/imagestest.png` вернут один и тот же файл. Принимая во внимание, что мы можем получить доступ к целевой папке через любой URL-адрес запроса, начинающийся с `/img`, мы можем попытаться получить доступ к вездесущему каталогу `..`, тем самым достигая родительского каталога для целевого, отправив запрос к `/images..` для приведенного примера.

```markdown
**Перебор страниц через ffuf**

**Описание:**  
Использование инструмента **ffuf** (Fuzz Faster U Fool) для перебора страниц на сервере — это общий подход для тестирования безопасности, который может быть использован для обнаружения скрытых или несанкционированных страниц и директорий на веб-сервере.

**Попытка взлома:** Сначала необходимо скачать приложение ffuf.  
Для этого:

1. **Установим Go:** Загрузим и установим Go с [официального сайта](https://go.dev).  
2. **Установим ffuf:** Откроем командную строку и выполним команду:
   ```bash
   go install github.com/ffuf/ffuf/v2@latest
   ```

Далее создадим текстовый файл `wordlist.txt`, содержащий потенциальные названия директорий. Вот некоторые наши слова:
```
admin
login
dashboard
secret
```

Теперь запустим **ffuf** для перебора директорий:
```bash
ffuf -u https://project1.com/FUZZ -w C:\Users\nedik\Downloads\wordlist.txt -mc 200
```

**Что мы получим в консоли:**

--скрин--

**Вот что будет, если фильтровать ответы по коду 404:**
```bash
ffuf -u https://project1.com/FUZZ -w C:\Users\nedik\Downloads\wordlist.txt -mc 404
```

--скрин--

**Вывод:** К сожалению, данный способ тоже не привёл нас к нахождению уязвимости. Мы не нашли такие URI, которые по смыслу не были бы доступны через какой-нибудь веб-интерфейс, о которых обычный пользователь лучше бы не знал. Конечно, данный способ будет более актуален для больших проектов, где существует множество различных доступных URI. Для второго домена мы не делали тестов, так как структуры обоих серверов с разными доменами идентичны.

---

**Уязвимость CVE-2013-2028 (переполнение буфера)**

**Описание:**  
Уязвимость возникает в результате неправильно обработанной длины HTTP-запроса в модуле **ngx_http_parse_content_length**. Когда nginx обрабатывал специально сформированный HTTP-запрос с очень большим значением в заголовке `Content-Length`, это могло привести к переполнению буфера в памяти. Версии nginx до 1.4.1 включительно подвержены этой уязвимости.

**Попытка взлома:** Для проверки сервера напишем специальный код на Python, который будет отправлять запрос к нашему серверу с очень большим `Content-Length`:

```python
import socket

# Настройка подключения к серверу nginx
host = '127.0.0.1'  # IP-адрес сервера nginx
port = 80           # Порт, который прослушивает nginx

# Специально сформированный HTTP-запрос
request = (
    "GET / HTTP/1.1\r\n"
    "Host: {}\r\n"
    "Content-Length: 4294967295\r\n"  # Очень большое значение Content-Length для вызова переполнения
    "\r\n"
).format(host)

# Создание сокета и отправка запроса
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.connect((host, port))
    sock.sendall(request.encode())
    response = sock.recv(4096)
    print("Received response:")
    print(response.decode())
```

**После запуска получаем следующий вывод:**

--скрин--

**Вывод:** Ошибка `413 Request Entity Too Large` означает, что запрос, отправленный из нашего кода, превысил максимально допустимый размер на сервере. Это может произойти при попытке загрузить файл или отправить большое количество данных. Это подтверждает, что данная версия NGINX обладает лимитом для переданного body. Изменить его размер можно с помощью `client_max_body_size`, например:
```nginx
client_max_body_size 50M;
```

Таким образом, данная уязвимость отсутствует у проверяемой конфигурации сервера NGINX.

---

**Общий вывод**

Как мы увидели, многие уязвимости обнаруживаются и исправляются в новых версиях NGINX на постоянной основе. Однако стоит относиться более серьёзно к синтаксису и тестам конфигураций, поскольку, например, первая уязвимость может появиться буквально из-за недоставленного `/` в пути эндпоинта.

**Ресурсы**  
- [Habr: Тестирование безопасности](https://habr.com/ru/articles/745718/)
- [NGINX: Security Advisories](https://nginx.org/en/security_advisories.html)
