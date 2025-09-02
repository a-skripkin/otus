
# Устанавка HAProxy:
apt -y install haproxy

### Сохраняем исходный файл конфигурации:
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.origin

### Редактируем файл конфигурации:
nano /etc/haproxy/haproxy.cfg

global

        maxconn 10000
        log     127.0.0.1 local2

defaults
        log global
        mode tcp
        retries 2
        timeout client 30m
        timeout connect 4s
        timeout server 30m
        timeout check 5s

listen stats
    mode http
    bind *:7000
    stats enable
    stats uri /

listen postgres
    bind *:7432
    option httpchk
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 192.168.112.240:6432 maxconn 10000 check port 8008
    server node2 192.168.112.241:6432 maxconn 10000 check port 8008
    server node3 192.168.112.242:6432 maxconn 10000 check port 8008

### Перезагрузка и проверка HAProxy:
sudo systemctl restart haproxy
sudo systemctl status haproxy
psql -p 7432 -h 201.37.88.148 -U postgres postgres
