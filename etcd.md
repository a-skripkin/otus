
#	Установка etcd
### Скачиваем и разархивируем дистрибутив etcd:
cd /tmp
wget https://github.com/etcd-io/etcd/releases/download/v3.5.5/etcd-v3.5.5-linux-amd64.tar.gz

tar xzvf etcd-v3.5.5-linux-amd64.tar.gz

### Перемещаем исполняемые файлы etcd в usr/local/bin:
sudo mv /tmp/etcd-v3.5.5-linux-amd64/etcd* /usr/local/bin/

### Создаем пользователя и каталоги для работы  etcd:
sudo groupadd --system etcd
sudo useradd -s /sbin/nologin --system -g etcd etcd

mkdir /opt/etcd
mkdir /etc/etcd
mkdir /var/lib/etcd

### Назначаем права:
chown -R etcd:etcd /opt/etcd /var/lib/etcd /etc/etcd
chmod -R 700 /opt/etcd/ /var/lib/etcd /etc/etcd

### Создание конфигурации etcd.
nano /etc/etcd/etcd.conf

ETCD_NAME="etcd2"
ETCD_LISTEN_CLIENT_URLS="http://201.37.89.217:2379,http://127.0.0.1:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://201.37.89.217:2379"
ETCD_LISTEN_PEER_URLS="http://201.37.89.217:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://201.37.89.217:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-postgres-cluster"
ETCD_INITIAL_CLUSTER="etcd1=http://201.37.89.216:2380,etcd2=http://201.37.89.217:2380,etcd3=http://201.37.89.218:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"


### Создаем службу etcd и правим конфиг:
nano /etc/systemd/system/etcd.service

Текст конфига:
[Unit]
Description=Etcd Server
Documentation=https://github.com/etcd-io/etcd
After=network.target
After=network-online.target
Wants=network-online.target
  
[Service]
User=etcd
Type=notify
#WorkingDirectory=/var/lib/etcd/
WorkingDirectory=/opt/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
#set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/local/bin/etcd"
Restart=on-failure
LimitNOFILE=65536
IOSchedulingClass=realtime
IOSchedulingPriority=0
Nice=-20
 
[Install]
WantedBy=multi-user.target

### Устанавливаем автозапуск службы etcd и стартуем:
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd

### Проверяем службу и доступность нод:
systemctl status etcd

ETCDCTL_API=2 etcdctl member list

etcdctl endpoint health --cluster -w table