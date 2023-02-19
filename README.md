# yamdb_final

Учебный проект Яндекс.Практикум курса Python-разработчик(backend) с функцией workflow, выполняющей
автоматическое тестирование после пуша на Github, создание и отправку образа приложения на DockerHub, разворачивание проекта на удаленном сервере и отправки сообщения в телеграмм.

## Описание

Проект YaMDb собирает отзывы пользователей на произведения. Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Список категорий может быть расширен администратором.

Сами произведения в YaMDb не хранятся, в нём можно поделиться впечатлением.

В каждой категории есть произведения: книги, фильмы или музыка. Например, в категории «Книги» могут быть произведения «Винни-Пух и все-все-все» и «Марсианские хроники», а в категории «Музыка» — песня «Давеча» группы «Насекомые» и вторая сюита Баха. Произведению может быть присвоен жанр из списка предустановленных (например, «Сказка», «Рок» или «Артхаус»). Новые жанры может создавать только администратор. Благодарные или возмущённые пользователи оставляют к произведениям текстовые отзывы и ставят произведению оценку в диапазоне от одного до десяти (целое число); из пользовательских оценок формируется усреднённая оценка произведения — рейтинг (целое число). На одно произведение пользователь может оставить только один отзыв.

## Технологии

 - Python 3.7
 - Django 3.2.15
 - Django REST Framework 3.12.4
 - и многое другое... (см. файл requrements.txt)

## Шаблон наполнения .env
```
  # указываем, с какой БД работаем
  DB_ENGINE=django.db.backends.postgresql
  # имя базы данных
  DB_NAME=postgres
  # логин для подключения к базе данных
  POSTGRES_USER=postgres
  # пароль для подключения к БД (установите свой)
  POSTGRES_PASSWORD=postgres
  # название сервиса (контейнера)
  DB_HOST=db
  # порт для подключения к БД
  DB_PORT=5432
 ```
## Автоматизация развертывания серверного ПО

Для автоматизации развертывания проектов на боевых серверах используется среда виртуализации Docker, а также Docker-compose - инструмент для запуска многоконтейнерных приложений. Docker позволяет «упаковать» приложение со всем его окружением и зависимостями в контейнер, который может быть перенесён на любую Linux -систему, а также предоставляет среду по управлению контейнерами. 
Таким образом, для разворачивания проекта достаточно чтобы на сервере с ОС семейства Linux были установлены среда Docker и инструмент Docker-compose.

Ниже представлен Dockerfile - файл с инструкцией по разворачиванию Docker-контейнера веб-приложения:

```
# используемая версия python
# slim - образ имеет минимально необходимые компоненты для запуска
FROM python:3.7-slim
# создать дирректорию /app и сделать ее рабочей
WORKDIR /app
# копирование содержимого папки в рабочую дирректорию
COPY . .
# установка зависимостей из файла .requirements.txt
RUN pip3 install -r ./requirements.txt --no-cache-dir
# запуск сервера gunicorn на порту 8000
CMD ["gunicorn", "api_yamdb.wsgi:application", "--bind", "0:8000"]
```
В файле «docker-compose.yaml» описываются запускаемые контейнеры: веб-приложения, СУБД PostgreSQL и сервера Nginx
```
version: '3.8'
services:
  db:
    image: postgres:13.0-alpine
    volumes:
      - database:/var/lib/postgresql/data/
    env_file:
      - ./.env
  web:
    image: backndserj/yamdb_final:latest
    restart: always
    volumes:
      - static_value:/app/static/
      - media_value:/app/media/
    depends_on:
      - db
    env_file:
      - ./.env

  nginx:
    image: nginx:1.21.3-alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - static_value:/var/html/static/
      - media_value:/var/html/media/
    depends_on:
      - web

volumes:
  database:
  static_value:
  media_value:
```

## Описание команд для запуска приложения в контейнерах

Для запуска проекта в контейнерах - выполнить команду  `docker-compose up -d --build`, находясь в директории с docker-compose.yaml (infra)

```
После сборки контейнеров выполяем следующие команды:
# Выполнение миграций
docker-compose exec web python manage.py migrate
# Создание суперппользователя
docker-compose exec web python manage.py createsuperuser
# Собрать статику со всего проекта
docker-compose exec web python manage.py collectstatic --no-input
# Для дампа данных из БД
docker-compose exec web python manage.py dumpdata > fixtures.json
```
## Установка и запуск в dev-режиме

 1. Установите виртуальное окружение (команда: `python -m venv venv`).
 2. Активируйте виртуальное окружение (команда: `source venv/Scripts/activate`).
 3. Установите зависимости из файла requirements.txt (команда: `pip install -r requirements.txt`).
 4. Заполните базу данных (команда: `python manage.py loaddb`)
 5. Запустите dev-сервер (команда: `python manage.py runserver`).

## Ссылки

Проект доступен по ссылке http://yambd-f.ddns.net/api/v1/
Подробная документация http://yambd-f.ddns.net/redoc/


![example workflow](https://github.com/sergo-bckndNN/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)