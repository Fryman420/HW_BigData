Работаем под юзером hadoop из дз 1

i) Редактируем yarn-site.xml

На всех машинах под юзером hadoop:

```bash
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

NB: Возможно nano выдаст ошибку:
```
The command could not be located because '/usr/bin:/bin' is not included in the PATH environment variable.
nano: command not found.
```
Тогда надо юзать:
```bash
/bin/nano
```

Вставляем туда:
```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>ec2-54-242-88-143.compute-1.amazonaws.com</value>
    </property>

    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>0.0.0.0:8088</value>
    </property>

    <property>
        <name>yarn.resourcemanager.address</name>
        <value>0.0.0.0:8032</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>

    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>128</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>4096</value>
    </property>

    <property>
        <name>yarn.nodemanager.webapp.address</name>
        <value>0.0.0.0:8042</value>
    </property>
</configuration>
```

ii) Редактируем mapred-site.xml

```bash
cd $HADOOP_HOME/etc/hadoop
nano mapred-site.xml
```

Вставляем:
```xml
<configuration>
   
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

   
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>0.0.0.0:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>0.0.0.0:19888</value>
    </property>

    
    <property>
        <name>mapreduce.jobhistory.done-dir</name>
        <value>/home/hadoop/hadoop_tmp/mapreduce/done</value>
    </property>

    <property>
        <name>mapreduce.jobhistory.intermediate-done-dir</name>
        <value>/home/hadoop/hadoop_tmp/mapreduce/intermediate-done</value>
    </property>
</configuration>
```

iii) Проверяем переменную окружения JAVA_HOME

На всех машинах:
```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

iv) Запуск YARN

На VM1 выполняем:
```bash
VM1$ cd $HADOOP_HOME
```

Если были проблемы с запуском, добавляем в ~/.bashrc:
```bash
export HADOOP_HOME=~/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Применяем изменения:
```bash
source ~/.bashrc
```

Запускаем:
```bash
VM1$ start-yarn.sh
```

**NB:** Если что-то сломалось, имеет смысл убить процессы с помощью команды:
```bash
kill -9 <номер процесса>
```
Просмотреть запущенные процессы можно через:
```bash
jps
```

v) Запуск History Server

На VM1 выполняем:
```bash
VM1$ mapred --daemon start historyserver
```


P.S.
Т.к. я делал это на AWS добавлю, что в security group сделал такие настройки. В первом задании надо достаточно было открыть порты внутри security group, но для этого добавил возможность смотреть наружу на 0.0.0.0/0 хоть это и не безопасно (условий по безопасности, опять же, не было) 
<img width="1425" alt="Снимок экрана 2024-12-28 в 03 42 58" src="https://github.com/user-attachments/assets/e6fbdc01-3b00-4a2e-8400-b50066a3e98e" />


Еще для траблшутинга может быть полезно выполнить 
/bin/rm -f $HADOOP_HOME/logs/*.pid
/bin/rm -f $HADOOP_HOME/pids/*.pid
/bin/rm -f /tmp/hadoop-hadoop/*.pid

Итого:
<img width="1727" alt="Снимок экрана 2024-12-28 в 04 22 58" src="https://github.com/user-attachments/assets/bb639496-cc35-4e4f-9f9c-6204026862a1" />
<img width="1728" alt="Снимок экрана 2024-12-28 в 04 23 21" src="https://github.com/user-attachments/assets/deb67a57-b9b8-4ff8-a95f-e96f9dde8053" />
<img width="1727" alt="Снимок экрана 2024-12-28 в 04 23 34" src="https://github.com/user-attachments/assets/787acc1d-90dc-4c2f-975c-5270cd7b89a6" />
