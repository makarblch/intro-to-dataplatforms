## Кулишенко Макар
### Построение пайплайнов данных. Домашнее задание 4

**На чем мы остановились?**

Мы развернули Hive таким образом, чтобы была возможность одновременного его использования более чем одним клиентом 

**Что надо сделать сейчас?**

1. Запуск сессии Apache Spark под управлением YARN, развернутого на кластере из предыдущих заданий
2. Подключение к кластеру HDFS развернутому в предыдущих заданиях
3. Использование созданной ранее сессии Spark для чтения данных, которые были предварительно загружены на HDFS
4. Применение нескольких трансформаций данных (например, агрегаций или преобразований типов)
5. Применение партиционирования при сохранении данных
6. Сохранение преобразованных данных как таблицы
7. Проверку возможности чтения данных стандартным клиентом hive

**Решение:**

1. Начнем с подключения к своей jn.
```bash
ssh -L 9870:127.0.0.1:9870 -L 8088:127.0.0.1:8088 -L 19888:127.0.0.1:19888 team@176.109.91.8
```
176.109.91.8 - адрес моего узла входа. 

2. Установим все необходимое для корректной работы питон-окружения:
```bash
sudo apt install python3-venv
sudo apt install python3-pip
```

3. Зайдем в аккаунт hadoop
```bash
sudo -i -u hadoop
```

И перейдем в сессию tmux
```bash
tmux
```

4. Скачаем spark:
```bash
wget https://archive.apache.org/dist/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz
```
Распакуем его:
```bash
tar -xzvf spark-3.5.3-bin-hadoop3.tgz
```

5. Добавим переменные окружения:
```bash
export HADOOP_CONF_DIR="/home/hadoop/hadoop-3.4.0/etc/hadoop"
export HIVE_HOME="/home/hadoop/apache-hive-4.0.0-alpha-2-bin"
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib*
export PATH=$PATH:$HIVE_HOME/bin
export SPARK_LOCAL_IP=192.168.1.26
export SPARK_DIST_CLASSPATH="home/hadoop/spark-3.5.3-bin-hadoop3/jars/*:/home/hadoop/hadoop-3.4.0/etc/hadoop/*:/home/hadoop/hadoop-3.4.0/share/hadoop/common/lib/*:/home/hadoop/hadoop-3.4.0/share/hadoop/common/*:/home/hadoop/hadoop-3.4.0/share/hadoop/hdfs/*:/home/hadoop/hadoop-3.4.0/share/hadoop/hdfs/lib/*:/home/hadoop/hadoop-3.4.0/share/hadoop/yarn/*:/home/hadoop/hadoop-3.4.0/share/hadoop/yarn/lib/*:/home/hadoop/hadoop-3.4.0/share/hadoop/mapreduce/*:/home/hadoop/hadoop-3.4.0/share/hadoop/mapreduce/lib/*:/home/hadoop/apache-hive-4.0.0-alpha-2-bin/*:/home/hadoop/apache-hive-4.0.0-alpha-2-bin/lib/*"
export SPARK_HOME="/home/hadoop/spark-3.5.3-bin-hadoop3"
export PYTHONPATH=$(ZIPS=("$SPARK_HOME"/python/lib/*.zip); IFS=:; echo "${ZIPS[*]}"):PYTHONPATH
export PATH=$SPARK_HOME/bin:$PATH
```
192.168.1.26 - ip моей jn.

6. Перейдем обратно и создадим новый venv:
```bash
python3 -m venv venv
source venv/bin/activate
```

7. Обновим pip и установим ipython
```bash
pip install -U pip
pip install ipython
pip install onetl[files]
```

8. Создадим папку "input" в hdfs
```bash
hdfs dfs -mkdir /input
```


9. Загружу сокращенную версию датасета про депрессию у студентов с kaggle :
```bash
pip install gdown
gdown --id 1rMDv-TId0sth6ZkiCFEO2_u7f9u7bZ_a -O students.csv
```

10. Перейдем в ipython:
```bash
ipython
```

11. Там добавим следующий код:
```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from onetl.connection import SparkHDFS
from onetl.connection import Hive
from onetl.file import FileDFReader
from onetl.file.format import CSV
from onetl.db import DBWriter

spark = (
        SparkSession.builder.master("yarn")
        .appName("spark-with-yarn")
        .config("spark.sql.warehouse.dir", "/user/hive/warehouse")
        .config("spark.hive.metastore.uris", "thrift://tmpl-jn:9083")
        .config("spark.yarn.jars", "/home/hadoop/spark-3.5.3-bin-hadoop3/jars/*")
        .enableHiveSupport()
        .getOrCreate()
)

hdfs = SparkHDFS(host="tmpl-nn", port=9000, spark=spark, cluster="test")
reader = FileDFReader(connection=hdfs, format=CSV(delimiter=",", header=True), source_path="/input")
```

12. Загрузим наш датасет с песнями:
```python
df = reader.run(["students.csv"])
print(*[df.count(), df.printSchema(), df.rdd.getNumPartitions()])

hive = Hive(spark=spark, cluster="test")
print(hive.check())
```