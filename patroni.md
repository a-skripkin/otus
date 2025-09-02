
#	”становка Patroni


### ”станавка необходимых пакетов дл€ работы с Python:
apt -y install python3 python3-pip python3-dev python3-psycopg2 libpq-dev

### ѕакеты Python:
pip3 install psycopg2 --break-system-packages
pip3 install psycopg2-binary --break-system-packages
pip3 install patroni --break-system-packages
pip3 install python-etcd --break-system-packages

### —оздаем каталог и редактируем конфигурацию Patroni:
mkdir /etc/patroni/
nano /etc/patroni/patroni.yml

patroni.yml дл€ 192.168.112.240:

scope: postgres-cluster # одинаковое значение на всех узлах
name: 192.168.112.240 # разное значение на всех узлах
namespace: /service/ # одинаковое значение на всех узлах

restapi:
  listen: 192.168.112.240:8008 # разное значение на всех узлах
  connect_address: 192.168.112.240:8008 # разное значение на всех узлах
  authentication:
    username: patroni
    password: 'password'

etcd:
  hosts: 192.168.112.240:2379, 192.168.112.241:2379, 192.168.112.242:2379 # список всех узлов, на которых установлен etcd

bootstrap:
  method: initdb
  dcs:
    ttl: 60
    loop_wait: 10
    retry_timeout: 27
    maximum_lag_on_failover: 2048576
    master_start_timeout: 300
    synchronous_mode: true
    synchronous_mode_strict: false
    synchronous_node_count: 1
    # standby_cluster:
      # host: 127.0.0.1
      # port: 1111
      # primary_slot_name: patroni
    postgresql:
      use_pg_rewind: false
      use_slots: true
      parameters:
        max_connections: 100
        superuser_reserved_connections: 5
        max_locks_per_transaction: 64
        max_prepared_transactions: 0
        huge_pages: try
        shared_buffers: 512MB
        work_mem: 128MB
        maintenance_work_mem: 256MB
        effective_cache_size: 4GB
        checkpoint_timeout: 15min
        checkpoint_completion_target: 0.9
        min_wal_size: 2GB
        max_wal_size: 4GB
        wal_buffers: 32MB
        default_statistics_target: 1000
        seq_page_cost: 1
        random_page_cost: 4
        effective_io_concurrency: 2
        synchronous_commit: on
        autovacuum: on
        autovacuum_max_workers: 5
        autovacuum_vacuum_scale_factor: 0.01
        autovacuum_analyze_scale_factor: 0.02
        autovacuum_vacuum_cost_limit: 200
        autovacuum_vacuum_cost_delay: 20
        autovacuum_naptime: 1s
        max_files_per_process: 4096
        archive_mode: on
        archive_timeout: 1800s
        archive_command: cd .
        wal_level: replica
        wal_keep_segments: 130
        max_wal_senders: 10
        max_replication_slots: 10
        hot_standby: on
        hot_standby_feedback: True
        wal_log_hints: on
        shared_preload_libraries: pg_stat_statements,auto_explain
        pg_stat_statements.max: 10000
        pg_stat_statements.track: all
        pg_stat_statements.save: off
        auto_explain.log_min_duration: 10s
        auto_explain.log_analyze: true
        auto_explain.log_buffers: true
        auto_explain.log_timing: false
        auto_explain.log_triggers: true
        auto_explain.log_verbose: true
        auto_explain.log_nested_statements: true
        track_io_timing: on
        log_lock_waits: on
        log_temp_files: 0
        track_activities: on
        track_counts: on
        track_functions: all
        log_checkpoints: on
        logging_collector: on
        log_statement: mod
        log_truncate_on_rotation: on
        log_rotation_age: 1d
        log_rotation_size: 0
        log_line_prefix: '%m [%p] %q%u@%d '
        log_filename: 'postgresql-%a.log'
        log_directory: /var/log/postgresql

  initdb:  # List options to be passed on to initdb
    - encoding: UTF8
    - locale: en_US.UTF-8
    - data-checksums

  pg_hba:  # должен содержать адреса ¬—≈’ машин, используемых в кластере
    - host all all 0.0.0.0/0 scram-sha-256
    - host replication replicator scram-sha-256

postgresql:
  listen: 192.168.112.240,127.0.0.1:5432 # разное значение на всех узлах
  connect_address: 192.168.112.240:5432 # разное значение на всех узлах
  use_unix_socket: true
  data_dir: /var/lib/postgresql/17/main
  bin_dir: /usr/lib/postgresql/17/bin
  config_dir: /etc/postgresql/17/main
  pgpass: /var/lib/postgresql/.pgpass_patroni
  authentication:
    replication:
      username: replicator
      password: password
    superuser:
      username: postgres
      password: password
  parameters:
    unix_socket_directories: /var/run/postgresql
    stats_temp_directory: /var/lib/pgsql_stats_tmp

  remove_data_directory_on_rewind_failure: false
  remove_data_directory_on_diverged_timelines: false
  
#callbacks:
#on_start:
#on_stop:
#on_restart:
#on_reload:
#on_role_change:

  create_replica_methods:
    - basebackup
  basebackup:
    max-rate: '100M'
    checkpoint: 'fast'

watchdog:
  mode: off  # Allowed values: off, automatic, required
  device: /dev/watchdog
  safety_margin: 5

tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false

  #specify a node to replicate from (cascading replication)
#replicatefrom: (node name)

### Ќазначение права на каталоги:
chown postgres:postgres -R /etc/patroni
chmod 700 /etc/patroni
mkdir /var/lib/pgsql_stats_tmp
chown postgres:postgres /var/lib/pgsql_stats_tmp

### ¬ыполнить проверку работоспособности:
sudo -u postgres patroni /etc/patroni/patroni.yml

### —делать Patroni как службу (на всех трех нодах одинаково):
nano /etc/systemd/system/patroni.service

### “екст конфигурации:
[Unit]
Description=High availability PostgreSQL Cluster
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres

#Read in configuration file if it exists, otherwise proceed
EnvironmentFile=-/etc/patroni_env.conf

#Start the patroni process
ExecStart=/usr/local/bin/patroni /etc/patroni/patroni.yml

#Send HUP to reload from patroni.yml
ExecReload=/bin/kill -s HUP $MAINPID

#only kill the patroni process, not it's children, so it will gracefully stop postgres
KillMode=process

#Give a reasonable amount of time for the server to start up/shut down
TimeoutSec=60

#Do not restart the service if it crashes, we want to manually inspect database on failure
Restart=no

[Install]
WantedBy=multi-user.target

### ѕеревод Patroni в автозапуск, стартуем и провер€ем:
systemctl daemon-reload
systemctl enable patroni
systemctl start patroni
systemctl status patroni

### ѕросмотреть состо€ние кластера:  
patronictl -c /etc/patroni/patroni.yml list