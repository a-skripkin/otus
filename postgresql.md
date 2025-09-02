# Установка Postgres 17

### На трех нодах

### Регистрция репозитория и установка:
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

apt install postgresql-17

### Создаем пользователей и задаем пароли:
sudo su postgres
psql
\password

create user replicator replication login encrypted password 'password';

create user pgbouncer login encrypted password 'password';

### Выход из psql:
\q
Ctrl+D

### Редактируем файлы конфигурации для возможности удаленного подключения:
nano /etc/postgresql/17/main/pg_hba.conf

Изменяем
host all all 127.0.0.1/32 scram-sha-256
host replication all 127.0.0.1/32 scram-sha-256
на 
host all all 0.0.0.0/0 scram-sha-256
host replication all 0.0.0.0/0 scram-sha-256


nano /etc/postgresql/17/main/postgresql.conf
#listen_address = 'localhost' - раскоментируем и заменяем localhost на *


### На нодах 2 и 3
### Удаляем содержимое pgdata , данные будут реплицироваться с лидера кластера:
systemctl stop postgresql
rm -rf /var/lib/postgresql/17/main/*
