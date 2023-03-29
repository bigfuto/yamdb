# yamdb_final

![yamdb_final](https://github.com/bigfuto/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)


**Описание**  
Проект YaMDb является базой данных по сбору отзывов пользователей на произведения.  
Произведения делятся на категории, такие как «Книги», «Фильмы», «Музыка». Список категорий может быть расширен, например, можно добавить категорию «Изобразительное искусство» или «Ювелирка», произведению может быть присвоен жанр из списка предустановленных, например, «Сказка», «Рок» или «Артхаус».  
Добавлять произведения, категории и жанры может только администратор. Пользователи оставляют к произведениям текстовые отзывы и ставят произведению оценку в диапазоне от одного до десяти, из пользовательских оценок формируется усреднённая оценка произведения — рейтинг. На одно произведение пользователь может оставить только один отзыв.Добавлять отзывы, комментарии и ставить оценки могут только аутентифицированные пользователи.  

**Технологии:**  
- [Python](https://www.python.org/doc/) 
- [Django](https://docs.djangoproject.com/en/4.1/releases/2.2.16/)
- [Django REST framework](https://www.django-rest-framework.org/)
- [DRF Simple JWT](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/)
- [Django Filter](https://django-filter.readthedocs.io/en/main/)
- [Gunicorn](https://gunicorn.org/)
- [PostgreSQL](https://www.postgresql.org/)
- [Docker](https://www.docker.com/)
- [Yandex.Cloud](https://cloud.yandex.ru/)


## Краткая история разработки
После получения технического задания от Заказчика наша команда провела мозговой штурм и мы решили разбить проект на 3 сегмента, над которыми можно было бы работать параллельно:  

**Первый разработчик:** [Даур Павликов](https://github.com/Koloyojik), коллективным решением был назначен тимлидом, приступил к разработке моделей, view-функций и эндпойнтов для произведений, категорий и жанров.  

**Второй разработчик:** [Иванов Илья](https://github.com/bigfuto), приступил к написанию системы управления пользователями, системы регистрации и аутентификации, права доступа, работу с токеном, системы подтверждения пользователь через e-mail, упаковку в Docker контейнер, подключение ngnix и PostgreSQL, автоматические тесты и развертывание проекта в Yandex.Cloud c помощью GitHub actions.

**Третий разработчик:** [Егор Дубихин](https://github.com/Kot9lpa99), приступил к разработке системы социальных взаимодействий, а именно отзывами, комментариями, рейтингом произведений.  

После утверждения архитектуры и разбиения установленных задач на подзадачи при помощи виртуальной доски тасков Trello команда приступила к написанию проекта.

---
## Документация  
**Подготовка:**  
- **Для начала необходимо задать переменные окружения в `./infra/.env`:**  
    ```bash
    DB_ENGINE=django.db.backends.postgresql # указываем, что работаем с postgresql
    DB_NAME=postgres # имя базы данных
    POSTGRES_USER=postgres # логин для подключения к базе данных
    POSTGRES_PASSWORD=postgres # пароль для подключения к БД (установите свой)
    DB_HOST=db # название сервиса (контейнера)
    DB_PORT=5432 # порт для подключения к БД
    ```
- **Если планируете разворачивать проект на удалённом сервере:**  
`./infra/nginx/default.conf`
    ```
    здесь должен быть указан IP или доменное имя этого сервера:
    server_name 127.0.0.1;
    ```  
**Запуск проекта в Docker:**
- **Переходим в коталог с docker-compose.yaml `./infra/`:**
    ```bash
    cd infra/
    ```  
- **Запускаем контейнеры:**  
    ```bash
    docker-compose up -d
    ```  
- **Применяем миграции:**  
    ```bash
    docker-compose exec web python manage.py migrate
    ```
- **Если требуется, то заполняем БД даными из фикстур:**  
    ```bash
    docker-compose exec web python manage.py loaddata fixtures.json
    ```
- **Собираем статику:**  
    ```bash
    docker-compose exec web python manage.py collectstatic --no-input 
    ```
- **Создаем суперпользователя:**  
    ```bash
    docker-compose exec web python manage.py createsuperuser
    ```
**Проект доступен по адресу:**  
```bash
http://127.0.0.1/ 
```
**Админка:**  
```bash
http://127.0.0.1/admin/ 
```

- **Админка:**
***login:*** `superuser`
***password:*** `superuser`

- **Получение Bearer токена:**
    ```bash
    {
    "confirmation_code": "689-be705501f183a92500e8",
    "username": "superuser"
    }
    ```

**Остановка проекта:**
- **С сохранением данных:**
    ```bash
    docker-compose down 
    ```
- **С удалением данных:**
    ```bash
    docker-compose down -v 
    ```
**Проект**  

---
## Документация API:  
**Алгоритм регистрации пользователей**  
Пользователь отправляет POST-запрос с параметрами email и username на эндпоинт `/api/v1/auth/signup/`.  
YaMDB отправляет письмо с кодом подтверждения (confirmation_code) на адрес email.
Пользователь отправляет POST-запрос с параметрами username и confirmation_code на эндпоинт `/api/v1/auth/token/`, в ответе на запрос ему приходит token (JWT-токен).
При желании пользователь отправляет PATCH-запрос на эндпоинт `/api/v1/users/me/` и заполняет поля в своём профайле (описание полей — в документации).  

### Пользовательские роли:
**Аноним** — может просматривать описания произведений, читать отзывы и комментарии.  

**Аутентифицированный пользователь (user)** — может, как и Аноним, читать всё, дополнительно он может публиковать отзывы и ставить оценку произведениям (фильмам/книгам/песенкам), может комментировать чужие отзывы; может редактировать и удалять свои отзывы и комментарии. Эта роль присваивается по умолчанию каждому новому пользователю.  

**Модератор (moderator)** — те же права, что и у Аутентифицированного пользователя плюс право удалять любые отзывы и комментарии.  

**Администратор (admin)** — полные права на управление всем контентом проекта. Может создавать и удалять произведения, категории и жанры. Может назначать роли пользователям.  

**Суперюзер Django** — обладет правами администратора (admin)  

---
### Расширение базовой модели пользователя
**Регистрации и аутентификация:**  
Для регистрации и аутентификации испульзуется один эндпоинт для POST запроса, `/api/v1/auth/signup/`.  
Была реализована функция, которая сначала проверяет наличие пользователя в базе данных, если его нет, то вызывает соответствующий реализатор, который проверяет поля и создаёт пользователя в базе данных, на основании найденного или вновь созданного пользователя, с помощью системы аутентификации django, создается токен восстановления (confirmation_code) и отправляется на email.  
Права доступа были разделены на три типа:
1. Для отзывов, комментариев и рейтинга - просмотр доступен всем, изменение/удаление автору/модератору/админу
2. Для произведений, категорий, жанров - просмотр доступен всем, изменение/удаление админу
3. Для ресурса users доступ открыт только админу.  

Для доступа к личной странице пользователя применяются стандартные права IsAuthenticated.  

**Работа с токеном:**  
После получения POST запроса на эндпоинт `/api/v1/auth/token/`, функция смотрит наличие обязательных полей (username и confirmation_code), с помощью системы аутентификации Django, проеверяет их и в случае успеха, используя модуль Simple JWT, выдает JWT токен. За авторизацию отвечает модуль Simple JWT, который ожидает JWT токен в заголовке Authorization.  

**Система подтверждения через e-mail:**  
С помощью функции send_mail после удачного POST запроса на `/api/v1/auth/signup/` отправляется письмо на e-mail пользователя с confirmation_code, который вместе с username пердается на `/api/v1/auth/token/` для получения токена.  

**User model:**  
Модель пользователя изменена следующим образом: Добавлено поле с биографией, изменено поле password на не обязательное(т.к. не используется при данной авторизации), добавлен список ролей. Для проверки ролей в модель добавлены соответствующие функции.  

---
### Регистрация и авторизация
**Получить код подтверждения на переданный email.**  
Права доступа: Доступно без токена.  
Использовать имя 'me' в качестве username запрещено.  
Поля email и username должны быть уникальными.  
**Пример запроса:**  
`http://127.0.0.1/api/v1/auth/signup/`  
```
{
"email": "string",
"username": "string"
}
```

**Пример ответа:** 
```
{
"email": "string",
"username": "string"
}
```
---
### Отзывы
**Получение списка всех отзывов**  
Получить список всех отзывов с помощью GET запроса  
Права доступа: Доступно без токена.  
`http://127.0.0.1/api/v1/titles/{title_id}/reviews/`  
**Добавление нового отзыва**  
Добавить новый отзыв. Пользователь может оставить только один отзыв на произведение.  
Права доступа: Аутентифицированные пользователи.  
**Пример POST запроса**  
```
{
"text": "string",
"score": 1
}
```
**Пример ответа**  
```
{
"id": 0,
"text": "string",
"author": "string",
"score": 1,
"pub_date": "2019-08-24T14:15:22Z"
}
```
---
### Комментарии
**Получение списка всех комментариев к отзыву. GET запрос**  
Получить список всех комментариев к отзыву по id  
Права доступа: Доступно без токена.  
`http://127.0.0.1/api/v1/titles/{title_id}/reviews/{review_id}/comments/`  
**Пример ответа**  
```
[
{
"count": 0,
"next": "string",
"previous": "string",
"results": []
}
]
```
**Добавление комментария к отзыву** 
Добавить новый комментарий для отзыва.  
Права доступа: Аутентифицированные пользователи.  
`http://127.0.0.1/api/v1/titles/{title_id}/reviews/{review_id}/comments/`  
**Пример запроса:**  
```
{
"text": "string"
}
```
**Пример ответа**  
```
{
"id": 0,
"text": "string",
"author": "string",
"pub_date": "2019-08-24T14:15:22Z"
}
```
Это лишь малая часть возможных примеров запросов. Полный список возможных запросов к API можно посмотреть вот по этому адресу: **http://127.0.0.1/redoc/**

---
**Команда #23:**  
:steam_locomotive:[**Даур Павликов**](https://github.com/Koloyojik)  
:railway_car:[**Иванов Илья**](https://github.com/bigfuto)  
:railway_car:[**Егор Дубихин**](https://github.com/Kot9lpa99)  
