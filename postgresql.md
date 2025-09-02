# ��������� Postgres 17

### �� ���� �����

### ���������� ����������� � ���������:
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

apt install postgresql-17

### ������� ������������� � ������ ������:
sudo su postgres
psql
\password

create user replicator replication login encrypted password 'password';

create user pgbouncer login encrypted password 'password';

### ����� �� psql:
\q
Ctrl+D

### ����������� ����� ������������ ��� ����������� ���������� �����������:
nano /etc/postgresql/17/main/pg_hba.conf

��������
host all all 127.0.0.1/32 scram-sha-256
host replication all 127.0.0.1/32 scram-sha-256
�� 
host all all 0.0.0.0/0 scram-sha-256
host replication all 0.0.0.0/0 scram-sha-256


nano /etc/postgresql/17/main/postgresql.conf
#listen_address = 'localhost' - �������������� � �������� localhost �� *


### �� ����� 2 � 3
### ������� ���������� pgdata , ������ ����� ��������������� � ������ ��������:
systemctl stop postgresql
rm -rf /var/lib/postgresql/17/main/*
