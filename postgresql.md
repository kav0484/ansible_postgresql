## POSTGRESQL

####Инициализация база данных
от имени пользователя postgres

```bash
postgres=# initdb -D /usr/local/pgsql/DATABASE
```

###Запуск сервиса

[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)

[Service]
Type=notify
User=postgres
ExecStart=/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT
TimeoutSec=0

[Install]
WantedBy=multi-user.target


####Созадние базы 

```bash
postgres=# CREATE DATABASE dbname;
```
####Создание пользователя и прав
```bash
postgres=# CREATE USER youruser WITH ENCRYPTED PASSWORD 'yourpass';
GRANT ALL PRIVILEGES ON DATABASE yourdbname TO youruser;
```

####Бекап и восстановление
#####1. с помощью утилиты pg_dump
бекап
```bash
pg_dump dbname > outfile
```
восстановление
```bash
psql dbname < outfile
```
#####2. Трансляция журналов
######ведущий сервер
в postgresql.conf 
   wal_level= replica or logical
   archive_mode=on
   archive_command = 'test ! -f /backup/demo/wal/%f && cp %p /backup/demo/wal/%f'
Установить параметр max_wal_senders > 0 для того, чтобы смогла отработать утилита pg_basebackup
Включить full_page_writes
разрешить подключение серверу в pg_hba.conf с которого будет делаться резервное копирование

######Ведомый сервер
pg_pasebackup -D /var/lib/postgresql/10/main -h
master -U postgres

создать файл recovery.conf в каталоге данных. В нем описывается, как
сервер должен выполнять восстановление и должен ли он работать
в качестве ведомого сервера. Чтобы сервер выступал в роли ведомого,
в файле должны присутствовать как минимум такие две строки:
standby_mode = on
restore_command = 'cp /wal_archive_location/%f %p'

#####2. Потоковая репликация
#####2.1 Слоты ркпликации
создание - SELECT * FROM pg_create_physical_replication_slot('node_a_slot');
Просмотр - SELECT slot_name, slot_type, active FROM pg_replication_slots;
 Для настройки резервного сервера на использование этого слота primary_slot_name должно быть настроено в конфигурации recovery.conf резервного.
 Вот простейший пример: 
   standby_mode = 'on'
   primary_conninfo = 'host=192.168.1.50 port=5432 user=foo password=foopass'
   primary_slot_name = 'node_a_slot'


####Регламентные задачи



####Роли

У ролей имеются следующие атрибуты.

1. Superuser: эта роль обходит все проверки, кроме атрибута login.
2. Login: роль с атрибутом login позволяет подключаться к базе данных.
3. Createdb: роль с таким атрибутом позволяет создавать базы данных.
4. Createrole: роль с этим атрибутом дает возможность создавать, изменять и удалять другие роли.
5. Replication: роль с данным атрибутом позволяет выполнять потоковую
репликацию.
6. Password: пароль роли используется при аутентификации методом md5.
Его можно зашифровать и задать для него срок действия. Отметим, что
этот пароль отличается от пароля в операционной системе.
7. Connection limit: задает максимальное число одновременных подключений. Каждое подключение потребляет аппаратные ресурсы, поэтому рекомендуется использовать средства организации пула подключений, например pgpool-II или PgBouncer, либо такие API, как Apache DBCP или c3p0.
8. Inherit: роль с таким атрибутом наследует разрешения от ролей, членом
которых является. Атрибут Inherit подразумевается по умолчанию.
9. Bypassrls: роль с этим атрибутом обходит механизм безопасности на
уровне строк (RLS).

```bash
postgres=# CREATE ROLE car_portal_app LOGIN;
```

####Схема
Схема содержит все именованные объекты базы данных: таблицы, представ-
ления, функции, агрегаты, индексы, последовательности, триггеры, типы дан-
ных, домены и
####Применение схем
У схем есть несколько применений:

1. управление авторизацией – в многопользовательской среде схемы
можно использовать для группировки объектов по ролям;
2. организация объектов базы данных – можно сгруппировать объек-
ты в соответствии с бизнес-логикой. Например, выделить в отдельную
группу исторические данные и данные аудита и завести для них спе­
циальную схему;
3. хранение стороннего SQL-кода – расширения, входящие в состав
сторонних пакетов, могут использоваться в нескольких приложениях.Компоненты базы данных PostgreSQL  83
Хранение их в отдельных схемах упрощает повторное использование
и обновление.
Чтобы создать схему car_portal_app, принадлежащую роли car_portal_app,
нужно выполнить команду:
```bash
postgres=# CREATE SCHEMA car_portal_app AUTHORIZATION car_portal_app;
```
-- Если имя схемы не указано, то оно совпадает с именем владельца
CREATE SCHEMA AUTHORIZATION car_portal_app;