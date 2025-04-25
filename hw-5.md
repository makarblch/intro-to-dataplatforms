## Кулишенко Макар
### Построение пайплайнов данных. Домашнее задание 5

**На чем мы остановились?**

Мы настроили Spark, проверили партиционирование данных, научились в питоне настраивать SparkSession

**Что надо сделать сейчас?**

Развернуть Airflow (мой выбор) для настройки DAG-ов по крону.

**Решение:**
Airflow будет запущен на порту 8080. Заранее прокинем этот порт, чтобы с локальной машины можно было тоже подсоединиться.

1. Начнем с подключения к своей jn.
```bash
ssh -L 9870:127.0.0.1:9870 -L 8088:127.0.0.1:8088 -L 19888:127.0.0.1:19888 -L 8080:127.0.0.1:8080 team@176.109.91.8
```
176.109.91.8 - адрес моего узла входа. 

2. Заходим в аккаунт hadoop:
```bash
sudo -i -u hadoop
```

3. Проверим, что нужные переменные окружения на месте. Добавим еще раз их на всякий пожарный:
```bash
wget https://archive.apache.org/dist/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz
tar -xzvf spark-3.5.3-bin-hadoop3.tgz
```

4. Добавим переменные окружения. 192.168.1.26 -- адрес моей jumpnode, Вы должны сюда поместить свой адрес:
```bash
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

export HIVE_HOME=/home/hadoop/apache-hive-4.0.0-alpha-2-bin
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib/*
export PATH=$PATH:$HIVE_HOME/bin

export SPARK_HOME=/home/hadoop/spark-3.5.3-bin-hadoop3
export HADOOP_CONF_DIR="/home/hadoop/hadoop-3.4.0/etc/hadoop"
export SPARK_LOCAL_IP=192.168.1.26
export PYTHONPATH=$(ZIPS=("$SPARK_HOME"/python/lib/*.zip); IFS=:; echo "${ZIPS[*]}"):$PYTHONPATH
export PATH=$SPARK_HOME/bin:$PATH
```

5. Перейдем обратно и создадим новый venv, обновим pip и установим все для airflow:
```bash
python3 -m venv venv
source venv/bin/activate

pip install "apache-airflow[celery]==2.10.3" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.10.3/constraints-3.12.txt"
```

6. Создадим пользователя в Airflow
```bash
airflow users create --role Admin --username admin --email admin --firstname admin --lastname admin --password admin
```

7. Напишем даг и сохраним его:
```bash
touch ~/airflow/dags/hw5.py
```

8. Запустим Airflow в режиме разработки, будем использовать tmux, чтобы не блокировать сессию:
```bash
tmux
airflow standalone
```

Проверим, что все работает