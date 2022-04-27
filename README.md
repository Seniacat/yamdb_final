# API_YaMDB
![yamdb_workflow](https://github.com/Seniacat/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)

### Описание
Проект сервиса API для YaMDB - социальной сети, которая собирает отзывы (Review) и оценки пользователей
на произведения (Title) в разных категориях и жанрах, а так же комментарии к отзывам.
Произведения делятся на категории (Category) и жанры (Genres), список которых может быть расширен, но правами на добавление
новых жанров, категорий и произведений обладает только администратор. 
Для авторизации пользователей используется код подтверждения.Для аутентификации пользователей используются JWT-токены. 

Реализован REST API CRUD для моделей проекта, для аутентификации примненяется JWT-токен. В проекте реализованы пермишены, фильтрации, сортировки и поиск по запросам клиентов, реализована пагинация ответов от API, установлено ограничение количества запросов к API. Проект разворачивается в трех Docker контейнерах: web-приложение, postgresql-база данных и nginx-сервер.

Проект развернут на боевом сервере Yandex.Cloud. Реализовано CI и CD проекта. При push изменений в главную ветку проект автоматические тестируется на соотвествие требованиям PEP8 и проверяется внутренними автотестами. После успешного прохождения тестов, на git-платформе собирается обзраз web-контейнера Docker и автоматически размешается в облачном хранилище DockerHub. Размещенный образ автоматически разворачивается на боевом сервере вмете с контейнером веб-сервера nginx и базы данных PostgreSQL.

### Системные требования
- Python 3.7+
- Docker
- Works on Linux, Windows, macOS

### Стек технологий:
- Python 3.8
- Django 3.2
- Django Rest Framework
- Simple-JWT
- PostreSQL
- Nginx
- Gunicorn
- Docker
- GitHub Actions (CI/CD)

### Запуск проекта в Docker - контейнерах:
Клонируйте репозиторий и перейдите в него в командной строке.
Создайте и активируйте виртуальное окружение:
```
git clone https://github.com/Seniacat/API_YaMDB.git
cd API_YaMDB/
```
Должен быть свободен порт 8000. PostgreSQL поднимается на 5432 порту, он тоже должен быть свободен.
Cоздать и открыть файл .env с переменными окружения:
```
cd infra
touch .env
```
Заполнить .env файл с переменными окружения по примеру (SECRET_KEY см. в файле settings.py). 
Необходимые для работы проекта переменные окружения можно найти в файле .env.example в текущей директории:
```
echo DB_ENGINE=django.db.backends.postgresql >> .env

echo DB_NAME=postgres >> .env

echo POSTGRES_PASSWORD=postgres >> .env

echo POSTGRES_USER=postgres  >> .env

echo DB_HOST=db  >> .env

echo DB_PORT=5432  >> .env

echo SECRET_KEY=************ >> .env
```
Установить и запустить приложения в контейнерах (образ для контейнера web загружается из DockerHub):
```
docker-compose up -d
```
Запустить миграции, создать суперюзера, собрать статику и заполнить БД:
```
docker-compose exec web python manage.py migrate

docker-compose exec web python manage.py createsuperuser

docker-compose exec web python manage.py collectstatic --no-input 

docker-compose exec web python manage.py loaddata fixtures.json
```
### Документация к проекту
Документация для API [доступна по ссылке](http://127.0.0.1/redoc/) после установки приложения.

### Работа с API для всех пользователей
Для неавторизованных пользователей работа с API доступна только в режиме чтения
```
Права доступа: Доступно без токена.
GET /api/v1/categories/ - Получение списка всех категорий
GET /api/v1/genres/ - Получение списка всех жанров
GET /api/v1/titles/ - Получение списка всех произведений
GET /api/v1/titles/{title_id}/reviews/ - Получение списка всех отзывов
GET /api/v1/titles/{title_id}/reviews/{review_id}/comments/ - Получение списка всех комментариев к отзыву
```
### Пользовательские роли
- Anonymous — может просматривать описания произведений, читать отзывы и комментарии.
- Аутентифицированный пользователь (user) — может, как и Аноним, читать всё, дополнительно он может публиковать отзывы и ставить оценку произведениям (фильмам/книгам/песенкам), может комментировать чужие отзывы; может редактировать и удалять свои отзывы и комментарии. Роль присваивается по умолчанию каждому новому пользователю.
- Модератор (moderator) — облаадет правами аутентифицированного пользователя + право удалять любые отзывы и комментарии.
- Администратор (admin) — полные права на управление всем контентом проекта. Может создавать и удалять произведения, категории и жанры, а так же назначать роли пользователям.

### Регистрация нового пользователя:
Получение кода подтверждения на переданный email. Поля email и username должны быть уникальными.
```
POST /api/v1/auth/signup/

{
  "email": "string",
  "username": "string"
}
```
 Получение JWT-токена для аутентификации
```
POST /api/v1/auth/token/

{
  "username": "string",
  "confirmation_code": "string"
}
```
### Работа с API для авторизованных пользователей
 Получение данных своей учетной записи:
```
Права доступа: user
GET api/v1/users/me/
```
Добавление категорий:
```
Права доступа: admin
POST /api/v1/categories/

{
  "name": "string",
  "slug": "string"
}
```
Добавление жанров:
```
Права доступа: admin
POST /api/v1/genres/

{
  "name": "string",
  "slug": "string"
}
```
Добавление произведений и обновление информации о произведении:
```
Права доступа: admin
POST /api/v1/titles/
PATCH /api/v1/titles/{titles_id}/

{
  "name": "string",
  "year": 0,
  "description": "string",
  "genre": [
    "string"
  ],
  "category": "string"
}
```
Удаление произведений
```
DELETE /api/v1/titles/{titles_id}/
```
Полный список эндпойнтов, методы и параметры запросов описаны в докуметации:
```
/redoc/ 
```
