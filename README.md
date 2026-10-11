# Отчёт по практическому занятию 2: Docker Compose и сохранение данных
## Цель работы
Запустить блог из двух сервисов (WordPress + MariaDB), опубликовать запись, сохранить её при пересоздании контейнеров, а также восстановить работу сайта после остановки базы данных.

## Шаг 1. Подготовка образов
Заранее скачаны необходимые образы:

bash
docker pull wordpress:apache
docker pull mariadb:11.4
Результат: оба образа успешно загружены.

https://img/1.png
https://img/2.png

## Шаг 2. Создание файла Compose
Создан файл compose.yaml в директории ~/docker-lesson-02:

bash
mkdir -p ~/docker-lesson-02
cd ~/docker-lesson-02
nano compose.yaml
Содержимое файла:

yaml
name: docker-lesson-02
services:
  db:
    image: mariadb:11.4
    environment:
      MARIADB_DATABASE: wordpress
      MARIADB_USER: student
      MARIADB_PASSWORD: lab-db-pass
      MARIADB_ROOT_PASSWORD: lab-root-pass
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 5s
      timeout: 5s
      retries: 30
      start_period: 30s
  wordpress:
    image: wordpress:apache
    ports:
      - "127.0.0.1:8090:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: student
      WORDPRESS_DB_PASSWORD: lab-db-pass
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - wp_data:/var/www/html
volumes:
  db_data:
  wp_data:
https://img/3.png

## Шаг 3. Запуск сервисов
bash
docker compose config -q
docker compose up -d
docker compose ps
Результат:

db — статус Up (healthy)

wordpress — статус Up, порт 127.0.0.1:8090->80/tcp

https://img/4.png

Сайт открылся по адресу http://localhost:8090 — появился мастер установки WordPress.

## Шаг 4. Настройка блога и публикация записи
В мастере установки WordPress:

Язык: русский

Название сайта: «Учебный блог — Тимур WWW»

Имя пользователя: student

Пароль: задан отдельный учебный пароль

Email: student@example.com

https://img/5.png

Создана запись «Моя практика Docker 2 W» с текстом:

Тимур ИТ-9.23.3 (наверное, я честно хз не помню)
Данные должны сохраниться

Запись опубликована, открыта по адресу http://localhost:8090/?p=7.

https://img/6.png

## Шаг 5. Логи и остановка сайта
bash
docker compose logs --tail 20 wordpress
docker compose stop wordpress
docker compose ps -a
Результат: в логах найден запрос GET с кодом 200. После остановки:

wordpress — статус Exited (137)

db — остаётся Up (healthy)

https://img/7.png
https://img/8.png

При обновлении страницы сайт не загружался. После команды:

bash
docker compose start wordpress
сайт снова заработал, текст записи сохранился.

## Шаг 6. Пересоздание контейнеров с сохранением записи
bash
docker compose ps -q > containers-before.txt
docker compose down
docker compose ps -a
docker volume ls --filter label=com.docker.compose.project=docker-lesson-02
Результат: контейнеры удалены, тома docker-lesson-02_db_data и docker-lesson-02_wp_data сохранены.

https://img/9.png

Повторный запуск:

bash
docker compose up -d
docker compose ps
docker compose ps -q > containers-after.txt
cat containers-before.txt
cat containers-after.txt
Идентификаторы контейнеров изменились, но запись и учётная запись сохранились — повторная установка WordPress не потребовалась.

https://img/10.png

## Шаг 7. Восстановление сайта после остановки базы данных
Остановлен только сервис db:

bash
docker compose stop db
docker compose ps -a
Результат:

db — статус Exited (0)

wordpress — статус Up

https://img/11.png

При обновлении страницы записи через Ctrl+F5 появилось сообщение об ошибке подключения к базе данных.

Запуск существующего сервиса db:

bash
docker compose start db
docker compose ps
Результат: db — статус Up (healthy), wordpress — Up. Сайт снова заработал.

https://img/12.png

В запись добавлена фраза «Работа базы восстановлена» и сохранена.

https://img/13.png

Итог
В ходе практического занятия:

Запущен блог из двух сервисов через Docker Compose.

Опубликована запись в WordPress.

Проверено сохранение данных при пересоздании контейнеров (docker compose down без -v).

Остановлена и восстановлена база данных — сайт продолжил работу с прежними данными.

В запись добавлена фраза «Работа базы восстановлена».

Задание выполнено полностью.
