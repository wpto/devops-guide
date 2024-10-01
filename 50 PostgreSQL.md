Начнем с установки СУБД - Postgresql. Добавим официальный репозиторий postgresql. После добавления необходимо выполнить обновление, чтобы пакетный менеджер yum подтянул свежий репозиторий. На вопросы о импортировании PGP ключа нажимаем y.
```bash
yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-aarch64/pgdg-redhat-repo-latest.noarch.rpm
```

```bash
Last metadata expiration check: 0:06:44 ago on Tue 01 Oct 2024 03:11:40 AM MSK.
pgdg-redhat-repo-latest.noarch.rpm               12 kB/s |  15 kB     00:01
Dependencies resolved.
================================================================================
 Package                Architecture Version           Repository          Size
================================================================================
Installing:
 pgdg-redhat-repo       noarch       42.0-45PGDG       @commandline        15 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 15 k
Installed size: 17 k
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : pgdg-redhat-repo-42.0-45PGDG.noarch                    1/1
  Verifying        : pgdg-redhat-repo-42.0-45PGDG.noarch                    1/1

Installed:
  pgdg-redhat-repo-42.0-45PGDG.noarch

Complete!
```

Проверим список всех репозиториев.
```bash
yum repolist
```

```bash
repo id        repo name
appstream      CentOS Stream 9 - AppStream
baseos         CentOS Stream 9 - BaseOS
extras-common  CentOS Stream 9 - Extras packages
pgdg-common    PostgreSQL common RPMs for RHEL / Rocky / AlmaLinux 9 - aarch64
pgdg12         PostgreSQL 12 for RHEL / Rocky / AlmaLinux 9 - aarch64
pgdg13         PostgreSQL 13 for RHEL / Rocky / AlmaLinux 9 - aarch64
pgdg14         PostgreSQL 14 for RHEL / Rocky / AlmaLinux 9 - aarch64
pgdg15         PostgreSQL 15 for RHEL / Rocky / AlmaLinux 9 - aarch64
pgdg16         PostgreSQL 16 for RHEL / Rocky / AlmaLinux 9 - aarch64
pgdg17         PostgreSQL 17 for RHEL / Rocky / AlmaLinux 9 - aarch64
```

Ищем postgresql server 17-ой версии среди всех пакетов.
```bash
yum -y search postgresql17
```

```bash
...
====================== Name Exactly Matched: postgresql17 ======================
postgresql17.aarch64 : PostgreSQL client programs and libraries
========================== Name Matched: postgresql17 ==========================
postgresql17-contrib.aarch64 : Contributed source and binaries distributed with
                             : PostgreSQL
postgresql17-devel.aarch64 : PostgreSQL development header files and libraries
postgresql17-docs.aarch64 : Extra documentation for PostgreSQL
postgresql17-libs.aarch64 : The shared libraries required for any PostgreSQL
                          : clients
postgresql17-llvmjit.aarch64 : Just-in-time compilation support for PostgreSQL
postgresql17-odbc.aarch64 : PostgreSQL ODBC driver
postgresql17-plperl.aarch64 : The Perl procedural language for PostgreSQL
postgresql17-plpython3.aarch64 : The Python3 procedural language for PostgreSQL
postgresql17-pltcl.aarch64 : The Tcl procedural language for PostgreSQL
postgresql17-server.aarch64 : The programs needed to create and run a PostgreSQL
                            : server
postgresql17-tcl.aarch64 : A Tcl client library for PostgreSQL
postgresql17-test.aarch64 : The test suite distributed with PostgreSQL
```

Нам нужен сервер и набор пакетов для работы с ним, устанавливаем.
```bash
yum -y install postgresql17 postgresql17-server
```

После установки необходимо инициализировать базу данных скриптом, запускаем скрипт.
```bash
/usr/pgsql-17/bin/postgresql-17-setup initdb
```

Запускаем Postgresql и добавляем в автозагрузку.
```bash
systemctl start postgresql-17
systemctl enable postgresql-17
```

Проверяем статус службы Postgresql.
```bash
systemctl status postgresql-17
```

```bash
● postgresql-17.service - PostgreSQL 17 database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql-17.service; enabled; preset: disabled)
     Active: active (running) since Tue 2024-10-01 03:27:37 MSK; 1min 33s ago
       Docs: https://www.postgresql.org/docs/17/static/
    Process: 6916 ExecStartPre=/usr/pgsql-17/bin/postgresql-17-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 6921 (postgres)
      Tasks: 7 (limit: 3791)
     Memory: 20.0M
        CPU: 92ms
     CGroup: /system.slice/postgresql-17.service
             ├─6921 /usr/pgsql-17/bin/postgres -D /var/lib/pgsql/17/data/
             ├─6922 "postgres: logger "
             ├─6923 "postgres: checkpointer "
             ├─6924 "postgres: background writer "
             ├─6926 "postgres: walwriter "
             ├─6927 "postgres: autovacuum launcher "
             └─6928 "postgres: logical replication launcher "

Oct 01 03:27:37 localhost.localdomain systemd[1]: Starting PostgreSQL 17 database server...
Oct 01 03:27:37 localhost.localdomain postgres[6921]: 2024-10-01 03:27:37.174 MSK [6921] LOG:  redirecting log output to logging>
Oct 01 03:27:37 localhost.localdomain postgres[6921]: 2024-10-01 03:27:37.174 MSK [6921] HINT:  Future log output will appear in>
Oct 01 03:27:37 localhost.localdomain systemd[1]: Started PostgreSQL 17 database server.
```

Попробуем подключиться к базе данных при помощи утилиты psql, также есть встроенный пользователь postgres. Данная конструкция выполняет команду psql от пользователя postgres.

```bash
sudo -u postgres psql
```

```bash
psql (17.0)
Type "help" for help.

postgres=#
```

> Не путаем пользователя postgres в операционной системе Linux, список всех пользователей на сервере хранится в файле /etc/passwd. Пользователи, они же роли это внутренняя абстракция Postgresql, посмотреть пользователей можно подключившись к базе данных, например утилитой psql и набрать `\du`

Для выхода из psql вводим `\q`. Чтобы посмотреть список всех команд в psql вводим `\?`. Необходимо внести изменения в пару конфиг файлов, для работы нашего приложения с Postgresql. Открываем текстовым редактором vim файл `/var/lib/pgsql/17/data/postgresql.conf`. Ищем в этом файле `#listen_addresses`. Для поиска в редакторе vim нажимаем / в командном режиме и вводим искомую строку, после чего переходим в режим вставки, убираем комментарий(знак #) и указываем IP-адрес виртуальной машины, сохраняем и выходим.

```bash
# - Connection Settings -

listen_addresses = '192.168.0.103'      # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
#port = 5432                            # (change requires restart)
max_connections = 100                   # (change requires restart)
```

После этого перезагружаем сервис Postgresql.
```bash

```

Далее открываем файл `/var/lib/pgsql/13/data/pg_hba.conf`, данный файл отвечает за разграничение доступа для подключений к базам данных. В нем указываем тип подключения, базу данных, пользователя, ip-адрес источника подключения и метод, например по паролю(md5).

В конце файла добавим подключения откуда угодно, в любую базу данных, любым пользователем, но по паролю.

```bash
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
host    all             all             0.0.0.0/0               md5
```

После правки `pg_hba.conf` нет нужды рестартить базу данных, это может привести к потере данных при активной записи, мы можем перечитать конфигурационные файлы без остановки. Используем `systemctl reload`. То есть reload это вполне безопасно, но не каждая служба его поддерживает.

```bash
systemctl reload postgresql-17
```