# Установка и настройка Hadoop на трёх виртуальных машинах

### Допущения:
Мы считаем, что подключены к каждой машине по SSH, и инструкции ниже написаны исходя из того, что мы уже находимся на соответствующей ноде.

---

## Шаги i) и ii): Выполняем на всех трёх машинах

### i) Создаем пользователя под Hadoop

1. Выполняем:
   ```bash
   sudo adduser hadoop
   ```
   Указываем пароль `12qwaszx`, остальную личную информацию оставляем пустой.

2. Даём этому пользователю права на выполнение команд через `sudo`:
   ```bash
   sudo usermod -aG sudo hadoop
   ```

   **Примечание:** Для упрощения даём root-доступ, понимая, что это небезопасно.

3. Переходим в пользователя `hadoop`:
   ```bash
   su - hadoop
   ```

4. Проверяем:
   ```bash
   whoami
   ```
   Результат: `hadoop`.

---

### ii) Под пользователем `hadoop` (или с `sudo`, где необходимо)

1. Обновляем пакеты:
   ```bash
   sudo apt-get update
   sudo apt-get install -y ssh rsync net-tools
   ```

2. Устанавливаем Java:
   ```bash
   sudo apt-get install -y openjdk-11-jdk
   ```

3. Проверяем версию Java:
   ```bash
   java -version
   ```
   Вывод:
   ```
   openjdk version "11.0.25" 2024-10-15
   OpenJDK Runtime Environment (build 11.0.25+9-post-Ubuntu-1ubuntu124.04)
   OpenJDK 64-Bit Server VM (build 11.0.25+9-post-Ubuntu-1ubuntu124.04, mixed mode, sharing)
   ```

---

## iii) Настройка SSH между машинами

На каждой ВМ (`VM1`, `VM2`, `VM3`) указываем IP-адреса: `<IP_VM1>`, `<IP_VM2>`, `<IP_VM3>`.

### Шаги для VM1:

1. Генерируем SSH-ключ:
   ```bash
   VM1$ ssh-keygen -t rsa -b 4096 -C "hadoop@VM1"
   ```
   При запросе пути нажимаем Enter (используем путь по умолчанию: `~/.ssh/id_rsa`).

2. Копируем публичный ключ на саму `VM1`:
   ```bash
   VM1$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   VM1$ chmod 600 ~/.ssh/authorized_keys
   ```

3. Копируем публичный ключ на `VM2` и `VM3`:
   На каждой из `VM2` и `VM3` выполняем:
   ```bash
   VM2$ mkdir -p ~/.ssh
   VM2$ chmod 700 ~/.ssh
   VM3$ mkdir -p ~/.ssh
   VM3$ chmod 700 ~/.ssh
   ```

   Затем с `VM1`:
   ```bash
   VM1$ cat ~/.ssh/id_rsa.pub
   ```
   Копируем содержимое и вставляем на `VM2` и `VM3` в файл `~/.ssh/authorized_keys` с помощью `nano`.

---

## iv) Установка Hadoop на всех машинах

1. Скачиваем Hadoop версии 3.3.6:
   ```bash
   VM1$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
   ```

2. Распаковываем:
   ```bash
   VM1$ tar -xzf hadoop-3.3.6.tar.gz
   ```

3. Перемещаем:
   ```bash
   VM1$ mv hadoop-3.3.6 ~/hadoop
   ```

4. Настраиваем переменные среды, добавив в `~/.bashrc`:
   ```bash
   VM1$ echo "export HADOOP_HOME=\$HOME/hadoop" >> ~/.bashrc
   VM1$ echo 'export PATH=\$PATH:\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin' >> ~/.bashrc
   VM1$ echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
   VM1$ source ~/.bashrc
   ```

---

## v) Настройка конфигурационных файлов Hadoop

### На всех ВМ:

#### 1. Редактируем `core-site.xml`:
   ```bash
   VM1$ nano $HADOOP_HOME/etc/hadoop/core-site.xml
   ```

   Вставляем:
   ```xml
   <configuration>
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://<IP_VM1>:9000</value>
       </property>
       <property>
           <name>hadoop.tmp.dir</name>
           <value>/home/hadoop/hadoop_tmp</value>
           <description>Common directory for temporary files.</description>
       </property>
   </configuration>
   ```

#### 2. Редактируем `hdfs-site.xml`:
   ```bash
   VM1$ nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
   ```

   Вставляем:
   ```xml
   <configuration>
       <property>
           <name>dfs.namenode.name.dir</name>
           <value>file:///home/hadoop/hdfs/namenode</value>
       </property>
       <property>
           <name>dfs.datanode.data.dir</name>
           <value>file:///home/hadoop/hdfs/datanode</value>
       </property>
       <property>
           <name>dfs.replication</name>
           <value>3</value>
       </property>
       <property>
           <name>dfs.namenode.http-address</name>
           <value>0.0.0.0:9870</value>
       </property>
   </configuration>
   ```

   **Примечание:** Обратите внимание на `security groups`, если используете AWS.

---

## vi) Форматирование NameNode

### 1. Добавляем Java-путь в `hadoop-env.sh` (если нужно):
   ```bash
   VM1$ export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
   ```

### 2. Форматируем NameNode:
   ```bash
   VM1$ cd $HADOOP_HOME
   VM1$ hdfs namenode -format
   ```

### 3. Редактируем файл `workers` на `VM1`:
   ```bash
   VM1$ nano $HADOOP_HOME/etc/hadoop/workers
   ```
   Добавляем хосты остальных двух нод (`VM2` и `VM3`).

---

## vii) Запуск Hadoop

На `VM1`:
```bash
VM1$ start-dfs.sh
```
<img width="467" alt="Снимок экрана 2024-12-28 в 03 37 54" src="https://github.com/user-attachments/assets/32a8cc16-cbef-48c8-93fd-38955b0ad543" />

