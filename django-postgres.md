## Подключение Postgres к Django

Установите в виртуальное окружение драйвер для работы с *postgres*: **psycopg2-binary**

```shell
pip install psycopg2-binary
```

Секретные данные (токены, пароли, ключи) нужно хранить отдельно от кода, для управления файлами `.env` потребуется пакет **django-environ**:

```shell
pip install django-environ
```

Добавьте настройки в файл `settings.py`:
```python
import environ

env = environ.Env(DEBUG=(bool, False))
env.read_env(env.str('../', '.env')) # если файл .env лежит на уровень выше

DATABASES = {
    'default': {
        'ENGINE': env('DB_ENGINE'),
        'NAME': env('DB_NAME'),
        'USER': env('POSTGRES_USER'),
        'PASSWORD': env('POSTGRES_PASSWORD'),
        'HOST': env('DB_HOST'),
        'PORT': env('DB_PORT'),
    }
}
```

Теперь создайте файл `.env` в корневой директории проекта и добавьте в него настройки подключения к базе данных:
```shell
DB_ENGINE=django.db.backends.postgresql
DB_NAME=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
DB_HOST=db
DB_PORT=5432
```
Чтобы изменения вступили в силу — выполните миграции:
```shell
python manage.py migrate
```

Готово! Теперь Django настроен для взаимодействия с базой данных.

## Заполнить базу
В базе *yatube* пока что нет данных, но они есть в вашем проекте, который установлен локально, на вашем компьютере.
Пора перенести их на сервер, в *PostgreSQL*. Выполните команду:
```shell
python manage.py dumpdata > dump.json
# выполнить локально, данные сохранятся в dump.json
```

Для копирования файлов с локального компьютера на сервер есть утилита **scp** (от англ. secure copy).
Она копирует файлы на сервер по протоколу *SSH*. Пользоваться ей весьма просто:
```shell
# scp my_file username@host:<путь-на-сервере>

# укажите IP своего сервера и путь до своей домашней директории на сервере
scp dump.json praktikum@84.201.161.196:/home/praktikum/
```

После выполнения этой команды файл `dump.json` появится в вашей домашней директории на сервере.

Теперь нужно выполнить команду для переноса данных:
```shell
python manage.py loaddata dump.json 
```
Готово. Теперь все данные перенесены на сервер и доступны посетителям вашего проекта.
Откройте свой проект в браузере и убедитесь в этом.

