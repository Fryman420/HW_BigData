Допущения в которых написана инструкция:
Мы считаем что мы подключены к каждой машине по ssh и инструкции ниже будут написаны исходя из того, что мы уже в соответствующей ноде.

Пункты i) и ii) выполняем на всех 3х машинах

i)  Создаем юзера под hadoop

1) sudo adduser hadoop 
- указываем какой-то пароль 12qwaszx, везде enter в личной информации
2) sudo usermod -aG sudo hadoop
- Даём этому пользователю права на выполнение команд через sudo 

(В задании нет требований по безопасности, так что для удобства даем рута, понимая что по-хорошему этого делать нельзя)

3) su - hadoop
Переходим в юзера hadoop
4) $ whoami
hadoop
-- проверяем на всякий что все норм 

ii)
Под пользователем hadoop (либо с sudo там, где нужно):
1) sudo apt-get update 
 sudo apt-get install -y ssh rsync net-tools
 - база
2) sudo apt-get install -y openjdk-11-jdk
ставим java
3) $ java -version
openjdk version "11.0.25" 2024-10-15
OpenJDK Runtime Environment (build 11.0.25+9-post-Ubuntu-1ubuntu124.04)
OpenJDK 64-Bit Server VM (build 11.0.25+9-post-Ubuntu-1ubuntu124.04, mixed mode, sharing)

Проверяем, должны увидеть что-то такое

iii) 
У нас 3 вм (буду называть их VM1, VM2, VM3; Их IP <IP_VM1>, <IP_VM2>, <IP_VM3>)
Пусть на 1й вм у нас будет NameNode, SecondaryNameNode + одна DataNode
На 2 и 3 вм по одной датаноде соответственно.

1) VM1$ ssh-keygen -t rsa -b 4096 -C "hadoop@VM1" 
При запросе пути сохранения ключа  Enter для использования пути по умолчанию (~/.ssh/id_rsa).
2) Скопируем публичный ключ (файл ~/.ssh/id_rsa.pub) на VM1 (саму себя):
VM1$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
VM1$ chmod 600 ~/.ssh/authorized_keys


3) Скопируем публичный ключ (файл ~/.ssh/id_rsa.pub) на VM2 и VM3

VM2$ mkdir -p ~/.ssh
VM2$ chmod 700 ~/.ssh
VM3$ mkdir -p ~/.ssh
VM3$ chmod 700 ~/.ssh


VM1$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2E.....hadoop@VM1

Копируем это безобразие и на других машинах:
VM2$ nano  ~/.ssh/authorized_keys
VM3$ nano  ~/.ssh/authorized_keys

(Считаем что человек читающий инструкцию знает как вставить ssh-ключ в nano и выйти из nano)


iv)

Ставим hadoop на всех машинах. Инструкции ниже надо выполнить на каждой ВМ

1) wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
Качаем версию 3.3.6
2) tar -xzf hadoop-3.3.6.tar.gz
распаковываем
3) mv hadoop-3.3.6 ~/hadoop
4) Настроим переменные среды, добавив в ~/.bashrc:
 
NB:
На всякий случай можно выполнить команду вроде 
$ readlink -f $(which java) | sed "s:bin/java::"
/usr/lib/jvm/java-11-openjdk-amd64/

Чтобы проверить совместимость версий

$echo "export HADOOP_HOME=\$HOME/hadoop" >> ~/.bashrc
$echo 'export PATH=\$PATH:\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin' >> ~/.bashrc
$echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
$source ~/.bashrc

v) Настройка конфигурационных файлов Hadoop

1) VM1$ nano $HADOOP_HOME/etc/hadoop/core-site.xml

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://<IP_VM1>:9000</value> ##### NB: <IP_VM1> -- надо заменить на hostname/api
        <!-- Либо используйте IP, например: hdfs://192.168.0.10:9000 -->
    </property>

    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoop_tmp</value>
        <description>Common directory for temporary files.</description>
    </property>
</configuration>

То же самое делаем на VM2, VM3

2) VM1$ nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
<configuration>
    <!-- Указываем где хранить метаданные NameNode -->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/hadoop/hdfs/namenode</value>
    </property>

    <!-- Указываем где хранить данные DataNode на каждой машине -->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/hadoop/hdfs/datanode</value>
    </property>

    <!-- Фактор репликации (число копий блоков) -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>

    <!-- Порт, на котором слушает NameNode -->
    <property>
        <name>dfs.namenode.http-address</name>
        <value>0.0.0.0:9870</value>
    </property>
</configuration>

3) От себя добавлю что намучился с security groups в aws

vi) форматирование nameNode
0) У меня вылезла какая-то бага, которую я на всякий решил через добавление в файл ~/hadoop/etc/hadoop/hadoop-env.sh
строки:
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
1) VM1$ cd $HADOOP_HOME
2) VM1$ hdfs namenode -format
3) В файле $HADOOP_HOME/etc/hadoop/workers на VM1
Надо добавить хосты остальных двух нод

vii) 
VM1$ start-dfs.sh

