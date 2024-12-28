# Установка и настройка Hive на Ubuntu VM

## i) Установка MySQL на VM1
1. Установите MySQL:
   ```bash
   VM1$ sudo apt-get update
   VM1$ sudo apt-get install -y mysql-server
   ```
2. Запуститить и активировать MySQL:
   ```bash
   VM1$ sudo systemctl start mysql
   VM1$ sudo systemctl enable mysql
   ```

## ii) Создание базы данных и пользователя для Hive
1. Зайдите в MySQL:
   ```bash
   VM1$ sudo mysql -u root -p
   ```
2. В MySQL выполните следующие команды:
   ```sql
   CREATE DATABASE metastore;
   CREATE USER 'hiveuser'@'localhost' IDENTIFIED BY '12qwaszx';
   GRANT ALL PRIVILEGES ON metastore.* TO 'hiveuser'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

## iii) Скачивание Hive
1. Скачать и разархивировать Hive:
   ```bash
   VM1$ wget https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz
   VM1$ tar -xzf apache-hive-4.0.1-bin.tar.gz
   VM1$ mv apache-hive-4.0.1-bin ~/hive
   ```
2. Настройте переменные окружения:
   ```bash
   VM1$ echo "export HIVE_HOME=\$HOME/hive" >> ~/.bashrc
   VM1$ echo "export PATH=\$PATH:\$HIVE_HOME/bin" >> ~/.bashrc
   VM1$ source ~/.bashrc
   ```

## iv) Конфигурация Hive
1. Редактируйте `hive-env.sh`:
   ```bash
   nano $HIVE_HOME/conf/hive-env.sh
   ```
   Туда надо добавить:
   ```bash
   export HADOOP_HOME=/home/hadoop/hadoop
   export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
   export HIVE_CONF_DIR=$HIVE_HOME/conf
   ```

2. Настройте `hive-site.xml`:
   ```bash
   nano $HIVE_HOME/conf/hive-site.xml
   ```
   Вставьте:
   ```xml
   <configuration>
       <property>
           <name>javax.jdo.option.ConnectionURL</name>
           <value>jdbc:mysql://localhost:3306/metastore?createDatabaseIfNotExist=true&amp;useSSL=false&amp;allowPublicKeyRetrieval=true</value>
       </property>
       <property>
           <name>javax.jdo.option.ConnectionUserName</name>
           <value>hiveuser</value>
       </property>
       <property>
           <name>javax.jdo.option.ConnectionPassword</name>
           <value>12qwaszx</value>
       </property>
       <property>
           <name>datanucleus.autoCreateSchema</name>
           <value>true</value>
       </property>
       <property>
           <name>datanucleus.fixedDatastore</name>
           <value>false</value>
       </property>
       <property>
           <name>hive.metastore.uris</name>
           <value>thrift://localhost:9083</value>
       </property>
   </configuration>
   ```

## v) Установка MySQL Connector
1. Скачайте MySQL Connector:
   ```bash
   VM1$ wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-9.1.0.tar.gz
   VM1$ tar -xzf mysql-connector-j-9.1.0.tar.gz
   VM1$ cp mysql-connector-j-9.1.0/mysql-connector-j-9.1.0.jar $HIVE_HOME/lib/
   ```
2. Проверьте, что файл установлен:
   ```bash
   VM1$ ls -lh $HIVE_HOME/lib | grep mysql
   ```
<img width="529" alt="Снимок экрана 2024-12-29 в 03 09 47" src="https://github.com/user-attachments/assets/2eb80a1e-9f4a-4b89-ae0d-58c088b34b23" />

## vi) Настройка HDFS
1. Создайте необходимые директории:
   ```bash
   $HADOOP_HOME/bin/hdfs dfs -mkdir -p /user/hive/warehouse
   $HADOOP_HOME/bin/hdfs dfs -chmod g+w /user/hive/warehouse
   $HADOOP_HOME/bin/hdfs dfs -mkdir -p /tmp/hive
   $HADOOP_HOME/bin/hdfs dfs -chmod g+w /tmp/hive
   ```

## vii) Решение конфликта библиотек

У меня возникла проблема, которую я решил так:
0.
   ```bash
   VM1$ rm /home/hadoop/hadoop/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar
   ```

## viii) Запуск служб Hive
1. Запустите Hive Metastore и HiveServer2:

Чтобы не морочиться с обьяснением tmux использовал nohup
   ```bash
   VM1$ > metastore.log
   VM1$ nohup hive --service metastore > metastore.log 2>&1 &
   VM1$ > hiveserver2.log
   VM1$ nohup hive --service hiveserver2 > hiveserver2.log 2>&1 &
   VM1$ tail -n 50 metastore.log
   VM1$ tail -n 50 hiveserver2.log
   ```

2. Загрузить небольшие тестровые данные (не стал выгружать что-то огромное т.к. ВМ которые я арендовал на aws и так были маленькие и им едва хватало оперативы. Заниматься конфигурированием хранилищ дополнительно не хотелось, в задании тредобваний к размеру таблицы не было):
   ```bash
   echo -e "1\tJohn\tEngineering\t60000\n2\tJane\tMarketing\t55000" > employees.txt
   hdfs dfs -mkdir -p /user/hadoop/employees
   hdfs dfs -put employees.txt /user/hadoop/employees/
   ```

3. Подключитесь к Hive:
   ```bash
   $ beeline -u jdbc:hive2://localhost:10000 -n hadoop -p 12qwaszx --verbose=true
   ```

4. Создайте таблицу:
   ```sql
   CREATE DATABASE IF NOT EXISTS company;
   USE company;

   CREATE EXTERNAL TABLE IF NOT EXISTS employees_raw (
       id INT,
       name STRING,
       department STRING,
       salary INT
   )
   ROW FORMAT DELIMITED
   FIELDS TERMINATED BY '\t'
   LOCATION '/user/hadoop/employees/';
   
5. Проверяем
```sql
 jdbc:hive2://localhost:10000> SELECT * FROM employees_raw LIMIT 1;
   ```

Должно быть похоже на это:
<img width="1728" alt="Снимок экрана 2024-12-29 в 03 13 48" src="https://github.com/user-attachments/assets/d991fe93-4483-4aa3-ab4c-3e4f19a025e9" />

Если что-то в процессе не встало -- перезагружаем вм и все шаги заново.

