# Base docker-compose.yaml
```yaml
# версия docker-compose
version: '3.8'

# имя директории для хранения данных
volumes:
  postgres_data:  # это название директории для хранения данных БД
  static_value:
  media_value:

# имена и описания контейнеров, которые должны быть развёрнуты
services:
  # описание контейнера db
  db:
    # образ, из которого должен быть запущен контейнер
    image: postgres:12.4
    # volume и связанная с ним директория в контейнере
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    # адрес файла, где хранятся переменные окружения
    env_file:
      - ./.env

  web:
    build: .
    restart: always
    command: gunicorn api_yamdb.wsgi:application --bind 0.0.0.0:8000
    ports:
      - "8000:8000"
    volumes:
      # Контейнер web будет работать с данными, хранящиеся в томе static_value, 
      # через свою директорию /code/static/
      - static_value:/code/static/

      # Данные, хранящиеся в томе media_value, будут доступны в контейнере web 
      # через директорию /code/media/
      - media_value:/code/media/
    # "зависит от", 
    depends_on:
      - db
    env_file:
      - ./.env

  nginx:
    # образ, из которого должен быть запущен контейнер
    image: nginx:1.19.3

    # запросы с внешнего порта 80 перенаправляем на внутренний порт 80
    ports:
      - "80:80"

    volumes:
      # При сборке скопировать созданный нами конфиг nginx из исходной директории 
      # в контейнер и сохранить его в директорию /etc/nginx/conf.d/
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf

      # Контейнер nginx будет работать с данными, хранящиеся в томе static_value, 
      # через свою директорию /var/html/static/
      - static_value:/var/html/static/

      # Данные, хранящиеся в томе media_value, будут доступны в контейнере nginx
      # через директорию /var/html/media/
      - media_value:/var/html/media/

    depends_on:
      # Контейнер nginx должен быть запущен после контейнера web
      - web
```

## Запуск docker-compose
```shell
docker-compose up -d --build
```
Для пересборки команда `up` выполняется с параметром `--build`.

Чтобы логи не мешали управлять контейнерами через терминал — развёртывание контейнеров можно выполнить в «фоновом режиме»: для этого применяется ключ `-d`.

## Запуск команд внутри контейнеров
```shell
docker-compose exec web python manage.py migrate --noinput
docker-compose exec web python manage.py createsuperuser
docker-compose exec web python manage.py collectstatic --no-input
```
