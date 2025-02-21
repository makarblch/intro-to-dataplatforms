## Кулишенко Макар
### Построение пайплайнов данных. Домашнее задание 1

1. Войдем в среду и введем пароль:
```bash
ssh team@176.109.91.8
```
2. Установим менеджер сессий tmux, чтобы ssh держался дольше:
```bash
sudo apt install tmux
```
3. Сохраним ключ, чтобы повторно его не вводить распространим его по всем узлам
Bash
```bash
ssh-keygen
cat .ssh/id_ed25519.pub >> .ssh/authorized_keys
scp .ssh/authorized_keys 192.168.1.26:/home/team/.ssh/ 
scp .ssh/authorized_keys 192.168.1.27:/home/team/.ssh/ 
scp .ssh/authorized_keys 192.168.1.28:/home/team/.ssh/ 
scp .ssh/authorized_keys 192.168.1.29:/home/team/.ssh/
```
4. Отредактируем hosts с помощью VIM
```bash
sudo vim /etc/hosts
```
Добавим в окошко для редактирования строчки:
```text
# 127.0.0.1 localhost
127.0.0.1 tmpl-jn
# 192.168.1.26 tmpl-jn
192.168.1.27 tmpl-nn 
192.168.1.28 tmpl-dn-00 
192.168.1.29 tmpl-dn-01
```
И закомментируем этим строчки
```
\# The following lines are desirable for IPv6 capable hosts

\#::1     ip6-localhost ip6-loopback
\#fe00::0 ip6-localnet
\#ff00::0 ip6-mcastprefix
\#ff02::1 ip6-allnodes
\#ff02::2 ip6-allrouters
```
Проверим, что обращение к узлам работает:
```bash
team@team-6-jn:~$ ping tmpl-nn
PING tmpl-nn (192.168.1.27) 56(84) bytes of data.
64 bytes from tmpl-nn (192.168.1.27): icmp_seq=1 ttl=64 time=0.384 ms
64 bytes from tmpl-nn (192.168.1.27): icmp_seq=2 ttl=64 time=0.316 ms
64 bytes from tmpl-nn (192.168.1.27): icmp_seq=3 ttl=64 time=0.259 ms
64 bytes from tmpl-nn (192.168.1.27): icmp_seq=4 ttl=64 time=0.319 ms
```
5. Создадим пользователя, который будет выполнять сервисы hadoop
```bash
sudo adduser hadoop
```

6.  Теперь добавим hadoop для всех хостов. Создадим пользователя и в vim отредактируем файл hosts
```text
ssh {тут обращаемся к хосту}
sudo adduser hadoop
```

Для tmpl-nn /hosts:
```text
# 127.0.0.1 localhost
# 127.0.0.1 tmpl-nn

192.168.1.26 tmpl-jn
192.168.1.27 tmpl-nn
192.168.1.28 tmpl-dn-00
192.168.1.29 tmpl-dn-01
```

Для tmpl-dn-00 /hosts:
```text
#127.0.0.1 localhost
127.0.0.1 tmpl-jn

192.168.1.26 tmpl-jn
192.168.1.27 tmpl-nn
192.168.1.29 tmpl-dn-01

\# The following lines are desirable for IPv6 capable hosts

\#::1     ip6-localhost ip6-loopback
\#fe00::0 ip6-localnet
\#ff00::0 ip6-mcastprefix
\#ff02::1 ip6-allnodes
\#ff02::2 ip6-allrouters
```

Для tmpl-dn-01 /hosts:
```text
#127.0.0.1 localhost
127.0.0.1 tmpl-dn-01

192.168.1.26 tmpl-jn
192.168.1.27 tmpl-nn
192.168.1.28 tmpl-dn-00

\# The following lines are desirable for IPv6 capable hosts

\#::1     ip6-localhost ip6-loopback
\#fe00::0 ip6-localnet
\#ff00::0 ip6-mcastprefix
\#ff02::1 ip6-allnodes
\#ff02::2 ip6-allrouters
```
7. Вернемся обратно на jn
```bash
exit
exit
exit
```

8. Мы сделали так, чтобы все узлы знали друг друга по именам, теперь можем переходить в tmux.
Переключимся в пользователя hadoop
```bash
sudo -i -u hadoop tmux
```
Скачаем дистрибудив с hadoop с сайта Apache Hadoop
```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
```
Временно выходим из tmux, пока дистрибутив скачивается

9. Сгенерируем ключ:
```bash
ssh-keygen
cat .ssh/id_ed25519.pub > .ssh/authorized_keys 
cat .ssh/authorized_keys
```
10. И переложим его в файлики:
```bash
scp -r .ssh/ tmpl-nn:/home/hadoop 
scp -r .ssh/ tmpl-dn-00:/home/hadoop 
scp -r .ssh/ tmpl-dn-01:/home/hadoop
```
11. Подключаемся к уже запущенной сессии tmux 
```bash
tmux attach -t 0
```
12. Развернем этот же дистрибутив на других нодах:
```bash
scp hadoop-3.4.0.tar.gz tmpl-nn:/home/hadoop 
scp hadoop-3.4.0.tar.gz tmpl-dn-00:/home/hadoop 
scp hadoop-3.4.0.tar.gz tmpl-dn-01:/home/hadoop
```

13. Теперь дистрибутив разложен на все ноды. Распакуем hadoop.
```bash
tar -xpf hadoop-3.4.0.tar.gz
```

14. Заходим на NameNode и тоже распаковываем
```bash
ssh tmpl-nn
tar -xzvf hadoop-3.4.0.tar.gz
```
И то же самое на датанодах:
```bash
ssh tmpl-dn-00
tar -xzvf hadoop-3.4.0.tar.gz
```bash
ssh tmpl-dn-01
tar -xzvf hadoop-3.4.0.tar.gz
```
Вернемся на jn:
```bash
ssh tmpl-jn
```
А затем выйдем:
```bash
exit
exit
exit
exit
```
15. Проверим версию java
```bash
java -version which java
```
Узнаем реально положение файла:
```bash
readlink -f /usr/bin/java 
```
Выведено: /usr/lib/jvm/java-11-openjdk-amd64/bin/java.

Настроим профиль
```bash
vim .profile
```
16. Добавим переменные окружения с помощью `vim .profile`
```bash
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

17. Активируем профиль
```bash
source .profile
```

18. Проверим версию hadoop
```bash
hadoop --version
```

19. Раскидаем профиль по нодам
```bash
scp .profile tmpl-nn:/home/hadoop 
scp .profile tmpl-dn-00:/home/hadoop 
scp .profile tmpl-dn-01:/home/hadoop
```

Войдем в папку:
```
cd hadoop-3.4.0/etc/hadoop/
```

20. Поправим скрипт, который устанавливает переменные окружения
```bash
vim hadoop-env.sh
```

Нужно добавить: JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

21. 
```bash
vim core-site.xml Bash
```
Поправим:
```xml
<configuration> <property>
<name>fs.defaultFS</name>
<value>hdfs://tmpl-nn:9000</value> </property>
</configuration>
```

22.
```bash 
vim hdfs-site.xml
```
Поправим:
```xml
<configuration> <property>
<name>dfs.replication</name>
<value>3</value> </property>
</configuration>
```

23. Меняем имена узлов
```bash
vim workers
```
```bash
tmpl-nn 
tmpl-dn-00 
tmpl-dn-01
```

24. Узнаем текущую активную директорию в консоли:
```bash
pwd
```
Вернулось: /home/hadoop/hadoop-3.4.0/etc/hadoop

25. Теперь распространим команды выше на все ноды:
```bash
scp hadoop-env.sh tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp hadoop-env.sh tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp hadoop-env.sh tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp core-site.xml tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp core-site.xml tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp core-site.xml tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hdfs-site.xml tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp hdfs-site.xml tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp hdfs-site.xml tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp workers tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp workers tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop 
scp workers tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
```

Везде видим 100%, поэтому можно запускать hadoop.

26. 
```bash
ssh tmpl-nn
cd hadoop-3.4.0/
```

27. Создадим и отформатируем файловую систему
```bash
bin/hdfs namenode -format 
```
Запускаем кластер
```bash
sbin/start-dfs.sh
```

28. Проверим, какие сервисы запущены:
```bash
hadoop@team-6-nn:~/hadoop-3.4.0$ jps
35490 Jps
35367 SecondaryNameNode
34983 NameNode
35160 DataNode
```

Ура! Теперь перейдем на ноды и убедимся, что там все работает:
```bash
ssh tmpl-dn-00 
jps
```
Вернется:
```bash
33942 Jps
33802 DataNode
```
```bash
ssh tmpl-dn-01 
jps
```
Вернется:
```bash
33531 Jps
33389 DataNode
```

На этом все, спасибо!