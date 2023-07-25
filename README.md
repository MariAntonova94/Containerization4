Урок 4. Dockerfile и слои
Задание: необходимо собрать образ и запустить из него контейнер.
основой образа должна быть alpine
установить необходимо mariaDB
уменьшить размер образа (способ обсуждался на лекции)
необходимо открыть порт для коммуникации с другими сущностями
для проверки решения необходимо подключить к такому контейнеру phpmyadmin 
(нужно, чтобы в нем вы увидели данные из вашей БД)
необходимо смонтировать внешнюю папку для хранения данных БД вне контейнера

Для начала необходимо создать Dockerfile, который будет описывать наш образ.

# Используем alpine как базовый образ

FROM alpine

# Устанавливаем и настраиваем mariaDB

RUN apk update && apk add mariadb mariadb-client

RUN mkdir /run/mysqld && chown mysql /run/mysqld

COPY my.cnf /etc/mysql/my.cnf

# Уменьшаем размер образа

RUN rm -rf /var/cache/apk/*

# Открываем порт 3306 для коммуникации с другими сущностями

EXPOSE 3306

# Запускаем mariaDB

CMD ["mysqld_safe"]

Далее создаем файл my.cnf, который будет содержать настройки mariaDB.

[mysqld]

datadir=/data/mysql

socket=/run/mysqld/mysqld.sock

Теперь сборка и запуск контейнера.

```
$ docker build -t my-mariadb .
$ docker run -d -p 3306:3306 -v /path/to/external/folder:/data/mysql --name my-mariadb my-mariadb
```

Пояснения:

- Сначала мы создаем образ с тегом `my-mariadb` с помощью команды `docker build`.
- Затем мы запускаем контейнер из созданного образа с помощью команды `docker run`.
- Флаг `-d` запускает контейнер в фоновом режиме.
- Флаг `-p 3306:3306` открывает порт 3306 в контейнере, чтобы мы могли подключаться извне.
- Флаг `-v /path/to/external/folder:/data/mysql` монтирует внешнюю папку `/path/to/external/folder`
 для хранения данных БД вне контейнера.
- Флаг `--name my-mariadb` задает имя контейнера.

Далее устанавливаем phpMyAdmin и подключаем его к нашему контейнеру.

```
$ docker run -d -p 8080:80 --link my-mariadb:db phpmyadmin/phpmyadmin
```

Пояснения:

- Мы запускаем контейнер phpMyAdmin из официального образа `phpmyadmin/phpmyadmin`.
- Флаг `-d` запускает контейнер в фоновом режиме.
- Флаг `-p 8080:80` открывает порт 8080 в контейнере, чтобы мы могли получить доступ к phpMyAdmin извне.
- Флаг `--link my-mariadb:db` устанавливает связь между контейнером phpMyAdmin и контейнером my-mariadb,
 чтобы они могли взаимодействовать друг с другом.

Теперь мы можем открыть веб-браузер и перейти по адресу `http://localhost:8080`,
 чтобы открыть phpMyAdmin. При входе вам потребуется ввести имя пользователя и пароль для mariaDB.
  После этого вы увидите интерфейс phpMyAdmin, где вы сможете просматривать данные в вашей БД.

  *Подготовила студентка GeekBrains* [*`Антонова Мария`*]