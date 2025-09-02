
#	Устанавка PGBouncer
apt -y install pgbouncer

### Сохраняем исходный вариант файла конфигурации:
mv /etc/pgbouncer/pgbouncer.ini /etc/pgbouncer/pgbouncer.ini.origin

### Редактируем файл конфигурации:
nao /etc/pgbouncer/pgbouncer.ini

### Вставляем текст:
[databases]
postgres = host=127.0.0.1 port=5432 dbname=postgres
* = host=127.0.0.1 port=5432

[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = *
listen_port = 6432
unix_socket_dir = /var/run/postgresql
auth_type = md5
#auth_type = trust
auth_file = /etc/pgbouncer/userlist.txt
auth_user = postgres
auth_query = SELECT usename, passwd FROM pg_shadow WHERE usename=$1
#admin_users = pgbouncer, postgres
admin_users = postgres
ignore_startup_parameters = extra_float_digits,geqo,search_path

pool_mode = session
#pool_mode = transaction
server_reset_query = DISCARD ALL
max_client_conn = 10000
#default_pool_size = 20
reserve_pool_size = 1
reserve_pool_timeout = 1
max_db_connections = 1000
#max_client_conn = 900
default_pool_size = 500
pkt_buf = 8192
listen_backlog = 4096
log_connections = 1
log_disconnections = 1

### Создаем файл userlist.txt со списком пользователей для работы через PGBouncer:
mcedit /etc/pgbouncer/userlist.txt

"postgres" "password"
"pgbouncer" "password"

### Перезапуск PGBouncer:
systemctl restart pgbouncer

### Проверка:
su - postgres
psql -p 6432 -h 127.0.0.1 -U postgres postgres
psql -p 6432 -h 201.37.89.216 -U postgres postgres #подключаемся к Leader.
