## Base default.conf
```shell
server {
    # Слушаем порт 80
    listen 80;

    # Список IP, запросы к которым должен обрабатывать nginx
    # В этом уроке проект разворачивается локально, поэтому nginx
    # должен обрабатывать запросы к 127.0.0.1.
    # Если вы планируете разворачивать контейнеры на удалённом сервере,
    # здесь должен быть указан IP или доменное имя этого сервера
    server_name 127.0.0.1 localhost;
    server_tokens off;

    # Указываем директорию со статикой:
    # если запрос направлен к внутреннему адресу /static/ — 
    # nginx отдаст файлы из /var/html/static/
    location /static/ {
        root /var/html/;
    }

    # Указываем директорию с медиа: 
    # если запрос направлен к внутреннему адресу /media/,
    # nginx будет обращаться за файлами в свою директорию /var/html/media/
    location /media/ {
        root /var/media/;
    }

    # Все остальные запросы перенаправляем в Django-приложение,
    # на порт 8000 контейнера web
    location / {
        proxy_pass http://web:8000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
}
```

### Настройка `listen`
По умолчанию все HTTP-запросы приходят на 80-й порт сервера; в запросе этот порт не прописывается в явном виде: например адрес `praktikum.yandex.ru` — это тоже самое, что и `praktikum.yandex.ru:80`. В строке `listen 80` серверу **nginx** обрабатывать запросы, полученные на порт `80`.

### Настройки `location /static/` и `location /media/`
В конфиге указано, из какой директории nginx должен раздавать статику и медиа: запросы к адресу `/static/` будут перенаправлены в директорию `/var/html/static/`, а запросы к адресу `/media/` будут перенаправлены в `/var/html/media/`

### Настройка `location /`
В свою очередь Django-приложение ожидает запросы на порт 8000. Следовательно, nginx должен принимать запросы на 80 порте и перенаправлять их на 8000 порт в контейнер `web`.

## Настройка Django
Проверьте настройки Django-проекта: откройте `settings.py` и удостоверьтесь что настройки для статики и медиа указаны верно:
```shell

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

MEDIA_URL = "/media/"
MEDIA_ROOT = os.path.join(BASE_DIR, "media")
```
Больше от Django ничего не требуется.

## Предстартовая подготовка и запуск nginx
```shell
sudo nginx -t

# текст успешной проверки:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Если ошибок не найдено — перезапустите `nginx`, чтобы конфигурация вступила в силу. Это можно сделать двумя способами — командой `restart` или `reload`. Лучше использовать `reload`: эта команда проверит конфигурацию, дождется окончания обработки всех запросов, и после этого мягко перезагрузит сервер.
```shell
sudo nginx -s reload 
```
