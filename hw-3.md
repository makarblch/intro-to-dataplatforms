## Кулишенко Макар
### Построение пайплайнов данных. Домашнее задание 3

**На чем мы остановились?**

Мы развернули YARN и опубликовали веб-интерфейсы основных и вспомогательных демонов кластера для внешнего использования.

**Что надо сделать сейчас?**

Развернуть Hive таким образом, чтобы была возможность одновременного его использования более чем одним клиентом (т.е. при установке не использовать embedded подход), описать или реализовать автоматизированную загрузку данных в партиционированную таблицу Hive.

**Пререквезиты:**

- Развернутый YARN на кластере
- Доступ к jn, nn, а также дата-нодам
- Хорошее настроение 


1. Возобновим сессию с прошлой домашки:
```bash
ssh -L 9870:127.0.0.1:9870 -L 8088:127.0.0.1:8088 -L 19888:127.0.0.1:19888 team@176.109.91.8
```

2. Перейдем на namenode (`ssh tmpl-nn`) и установим postgresql.
```shell
ssh tmpl-nn
```
```shell
sudo apt install postgresql
```

3. Переключимся на пользователя postgres
```shell
sudo -i -u postgres
```

4. Создадим базу данных, которая будет использоваться нашим метастором. Для этого перейдем в psql-формат:
```shell
psql
```
5. Выполним следующие SQL-скрипты. Назовем таблицу для метаданных metastore (как на занятии):
```shell
CREATE DATABASE metastore;  
```

6. Создадим пользователя и дадим ему права:
```shell
CREATE USER hive with password 'тут нужен пароль, я взял пароль от своего узла для входа';
```
Важно: пароль надо передавать именно в одинарных(!) кавычках. Я уже успел помучиться с этой проблемой :(

Теперь раздадим права:
```shell
grant all privileges on database "metastore" to hive;
```

7. Передадим владение этой базы этому пользователю:
```shell
alter database metastore owner to hive;
```

8. Выйдем из режима psql, нажав `\q` + enter и `exit`

9. В vim отредактируем конфиг postgresql, чтобы он принимал изменения не только с локального хоста, но и извне.

Войдем в vim в режим редактирования (`a`):
```shell
sudo vim /etc/postgresql/16/main/postgresql.conf
```

Вместо `listen_addresses = 'localhost'` (в середине файла) напишем следующее:
```shell
listen_addresses = 'tmpl-nn'
```

10. Исправим конфиг безопасности:
```shell
sudo vim /etc/postgresql/16/main/pg_hba.conf
```

Здесь уже есть существующие конфиги. Их удалять не нужно. Вместо этого, между `IPv4 local connections` и `IPv6 local connections` добавим:
```shell
host   metastore   hive    192.168.1.1/32   password
host   metastore   hive    192.168.1.26/32   password
```
Где вместо 192.168.1.26/32 - ip моей jn (у меня 192.168.1.26).

password - здесь это метод аутентификации 

И рестартнем postgresql:
```shell
sudo systemctl restart postgresql
```

11. Вернемся на jn (введем пароль, такой же, как от узла входа,  если нужно)
```shell
ssh tmpl-jn
```
и установим клиента postgres:
```shell
sudo apt install postgresql-client-16
```

12. Можно убедиться, что все работает (с jump-node):
```shell
psql -h tmpl-nn -p 5432 -U hive -W -d metastore
```

13. Зайдем в пользователя hadoop:
```shell
sudo -i -u hadoop
```
и установим дистрибутив hive:
```shell
wget https://archive.apache.org/dist/hive/hive-4.0.0-alpha-2/apache-hive-4.0.0-alpha-2-bin.tar.gz
```

14. Распакуем Hive:
```shell
tar -xzvf apache-hive-4.0.0-alpha-2-bin.tar.gz
```

15. Перейдем в папку для скачивания драйвера постгреса
```shell
cd apache-hive-4.0.0-alpha-2-bin/lib/
```

и скачаем соответствующий драйвер:
```shell
wget https://jdbc.postgresql.org/download/postgresql-42.7.4.jar
```

16. Отредактируем конфиги:
```shell
vim ../conf/hive-site.xml
```
Туда вставим:
```xml
<configuration>
    <property>
        <name>hive.server2.authentication</name>
        <value>NONE</value>
    </property>
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    <property>
        <name>hive.server2.thrift.port</name>
        <value>5433</value>
        <description>TCP port number to listen on, default 10000</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:postgresql://tmpl-nn:5432/metastore</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.postgresql.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>i9o0#lR_VZ=d</value>
    </property>
</configuration>
```
Здесь 5433 - порт, по которому мы будем принимать соединение, а tmpl-nn:5432 - порт для подключения к бд postgres

17. Добавим переменные окружения в наш профиль:
```shell
vim ~/.profile
```

В конец добавим:
```shell
export HIVE_HOME=/home/hadoop/apache-hive-4.0.0-alpha-2-bin
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib/*
export PATH=$PATH:$HIVE_HOME/bin
```

HIVE_HOME - место, где живет hive
HIVE_CONF_DIR - где хранятся конфиги
HIVE_AUX_JARS_PATH - где хранятся библиотеки

18. Применим переменные окружения и проверим, что все работает:
```shell
source ~/.profile
```

```shell
hive --version
```

Увидим, что нам вернулась версия:
```
apache-hive-4.0.0-alpha-2
```

Это соответствует тому, что мы установили.

19. Создадим папку для dwh (на которую мы ссылаемся в конфиге)
```shell
hdfs dfs -mkdir -p /user/hive/warehouse
```

20. Выдадим права к этой папке и dwh
```shell
hdfs dfs -chmod g+w /tmp
hdfs dfs -chmod g+w /user/hive/warehouse
```

21. Перейдем в корневую папку:
```shell
cd ..
```

И инициализируем БД командой:
```shell
bin/schematool -dbType postgres -initSchema
```

На этом этапе мы инициализировали БД.


23. Запустим такую команду:
```shell
hive --hiveconf hive.server2.enable.doAs=false --hiveconf hive.security.authorization.enable=false --service hiveserver2 1>> /tmp/hs2.log 2>> /tmp/hs2.10g &
```

И выйдем из tmux.

24. Запустим hive следующей стандартной утилитой и попробуем подключиться:
```shell
beeline -u jdbc:hive2://tmpl-jn:5433 -n scott -p tiger
```

На этом все, спасибо!
